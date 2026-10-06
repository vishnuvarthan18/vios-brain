# ARM Platform — Failure Analysis & Infrastructure Risk Report

**Date:** 2026-08-21  
**Server:** Hetzner CX31 — 7.6 GB RAM, 4 CPU cores, 150 GB disk, no swap  
**Current state:** 1 user, 27 containers, 4.4 GB RAM used (58%), load average 1.41

---

## Current Resource Usage (1 User)

| Resource | Current | Limit | % Used | Concern |
|---|---|---|---|---|
| RAM | 4.4 GB | 7.6 GB | 58% | Over half consumed with 1 user |
| Container memory allocated | 3.6 GB | 8.4 GB (sum of limits) | 43% | Limits exceed physical RAM |
| Kong memory | 383 MB | 512 MB | **75%** | Near OOM with 1 user |
| Kafka (3 brokers) | 1.8 GB | 3 GB | 60% | 24% of total server RAM for email OTPs |
| CPU load average | 1.41 | 4 cores | 35% | Elevated for idle system |
| Disk | 31 GB | 150 GB | 21% | OK now, log growth unchecked |
| Swap | 0 | 0 | N/A | No safety net — OOMKiller fires immediately |
| Containers | 27 | — | — | All on one machine |

### Per-Container Memory Snapshot

| Container | Used | Limit | % | Risk |
|---|---|---|---|---|
| arm-kafka1-prod | 604 MB | 1 GB | 59% | High baseline |
| arm-kafka2-prod | 601 MB | 1 GB | 59% | High baseline |
| arm-kafka3-prod | 613 MB | 1 GB | 60% | High baseline |
| arm-kong | 383 MB | 512 MB | **75%** | Near OOM |
| arm-mongodb-prod | 179 MB | 1 GB | 18% | Will grow with data |
| arm-calendar-backend | 162 MB | 768 MB | 21% | Will grow with syncs |
| arm-loki | 134 MB | 512 MB | 26% | Will grow with logs |
| arm-core-backend | 115 MB | 768 MB | 15% | Will grow with users |
| arm-alloy | 93 MB | 256 MB | 36% | Steady |
| arm-prometheus | 91 MB | 512 MB | 18% | Grows with metric cardinality |
| arm-grafana | 87 MB | 256 MB | 34% | Steady |
| arm-postgres-prod | 76 MB | 1.5 GB | 5% | Grows with connections |
| arm-notification-service | 60 MB | 768 MB | 8% | Steady |
| arm-admin-backend | 58 MB | 768 MB | 8% | Steady |
| arm-session | 41 MB | 768 MB | 5% | Grows with sessions |
| arm-redis-prod | 11 MB | 1.5 GB | 1% | Grows with jobs + sessions |

---

## Failure Scenarios

### Scenario 1: Kong OOM — 10-20 Users

**Probability: Very High (near certain)**  
**Time to failure:** First traffic spike over 20 concurrent requests

Kong is at 383 MB / 512 MB with 1 user. Each concurrent HTTP request through Kong adds ~2-5 MB for request buffering.

```
383 MB (base) + 20 users × 3 MB avg = 443 MB → approaches 512 MB limit
```

**Chain of failure:**
1. Kong hits 512 MB container memory limit
2. Docker OOMKills Kong container
3. Kong restarts (restart policy: `unless-stopped`)
4. During restart (10-30 seconds), ALL `/api/*` routes return 502 from Caddy
5. Caddy marks Kong as unhealthy
6. Every user sees "Service Unavailable"
7. Kong comes back, immediately hit by queued requests, OOMs again
8. **Restart loop** — site is down indefinitely until traffic drops

**Why this is the #1 risk:** Kong is already at 75% memory doing nothing. No other component is this close to its limit. This will be the first thing to break.

---

### Scenario 2: Server-Wide OOM — 50-80 Users

**Probability: High**  
**Time to failure:** First time 50+ users are online simultaneously

Available RAM: 3.2 GB. Each additional active user adds ~30-50 MB across the stack (session handling, backend request processing, MongoDB working set growth, Redis entries, BullMQ jobs).

```
3.2 GB available ÷ 40 MB per user = ~80 users before system OOM
```

But spikes are worse than averages. A burst of 50 calendar syncs triggers:
- 50 BullMQ jobs in memory (~10 MB each = 500 MB)
- 50 Google API responses buffered (~5 MB each = 250 MB)
- Calendar-be's unbounded event array (audit bug C-11) can hit 100+ MB per sync

**Chain of failure:**
1. No swap configured — Linux OOMKiller activates immediately when RAM fills
2. OOMKiller picks the highest-memory process: typically a **Kafka broker** (600 MB each)
3. Kafka broker killed → remaining 2 brokers rebalance → 30 second unavailability
4. Notification service can't consume from Kafka → OTP emails stop → users can't log in
5. If OOMKiller picks **MongoDB** instead: all calendar data operations fail, possible corruption
6. If OOMKiller picks **Postgres** instead: all auth/user operations fail, possible WAL corruption
7. Killed container restarts → memory pressure returns → OOMKiller fires again
8. **Thrashing loop** until traffic drops or manual intervention

**Critical detail:** Container memory limits exceed physical RAM. Sum of all container limits = 8.4 GB, but the server only has 7.6 GB. Docker allows this (limits are per-container, not total), but it means the OOMKiller operates at the Linux kernel level, not the Docker level, choosing victims unpredictably.

---

### Scenario 3: Cascading Auth Failure — 30+ Users

**Probability: High**  
**Time to failure:** 15 minutes after 30+ users log in around the same time

JWT access tokens have a 15-minute TTL. When 30+ users log in around the same time, their tokens all expire in the same 15-minute window.

```
30 users × 2 requests/second = 60 req/s
session service silent refresh holds Redis lock for up to 5s
Core-be DB pool = 50 connections
```

**Chain of failure:**
1. 30 users' JWT access tokens expire within seconds of each other
2. 30 concurrent silent refresh requests hit session service simultaneously
3. session service acquires Redis lock for user A; users B-Z queue or wait
4. session service calls core-be `/auth/refresh` — core-be rotates tokens, writes to Postgres
5. Postgres pool handles 30 refresh requests + ongoing queries = pool pressure
6. If any refresh takes >5s (Postgres slow under load), the Redis lock expires
7. Second session service request starts a concurrent refresh for the same session (lock expired)
8. Core-be's single-use token rotation detects the "replay" → **revokes the entire token family**
9. User is silently logged out — sees "session expired" with no explanation
10. User logs in again → same pattern repeats in 15 minutes
11. At scale: 10-30% of users experience random logouts every 15 minutes

---

### Scenario 4: Disk Full — 2-4 Weeks

**Probability: Certain (given enough time)**  
**Time to failure:** ~11 days at 100 users without log rotation

Current disk: 31 GB used / 150 GB (114 GB available).

Growth sources:
- Docker container logs (no rotation configured): ~100-500 MB/day per container × 27 containers = **5-13 GB/day**
- Loki ingestion (30-day retention): accumulates continuously
- MongoDB data: ~10 MB/user/month for calendar events
- Postgres WAL: grows with write volume
- Kafka segments: grows with message volume (7-day default retention)

```
114 GB available ÷ 10 GB/day = ~11 days to fill
```

**Chain of failure:**
1. Disk hits 100%
2. Postgres can't write WAL → all writes fail → auth operations crash
3. MongoDB can't write journal → potential data corruption on next restart
4. Docker can't write container logs → containers crash or hang
5. Kafka can't write segments → brokers crash → email OTPs stop
6. Loki can't ingest → observability goes blind (no logs to diagnose the problem)
7. SSH still works (kernel reserves 5% for root) but every service is down
8. Recovery requires manual SSH, pruning logs/data, restarting all containers

---

### Scenario 5: Redis Fills Up — Weeks to Months

**Probability: High**  
**Time to failure:** Days to weeks depending on sync frequency

Redis: 11 MB / 768 MB with `noeviction` policy. The `noeviction` policy means when Redis is full, ALL write operations return errors — it does not evict old data to make room.

Growth sources:
- sessions: ~2 KB × users (small)
- BullMQ completed jobs: `removeOnComplete: { count: 50, age: 86400 }` retains up to 50 completed jobs per queue, each ~5-50 KB
- BullMQ failed jobs: retained indefinitely if DLQ is not draining

Worst case calculation:
```
100 users × 10 syncs/day × 30 KB avg job × 50 retained = 1.5 GB (exceeds 768 MB limit)
```

**Chain of failure:**
1. Redis hits 768 MB limit
2. `noeviction` policy: ALL write operations fail with `OOM command not allowed`
3. session service can't create new sessions → new logins fail
4. session service can't update existing sessions → token refresh fails → users logged out
5. Calendar-be can't enqueue BullMQ jobs → syncs stop
6. OTP codes can't be stored in Redis → email login fails
7. **Platform is completely inaccessible** — nobody can log in, nobody with a session can act

---

### Scenario 6: Google API Quota Exhaustion — 20+ Users with Calendar Sync

**Probability: High**  
**Time to failure:** First batch of 20+ users enabling calendar sync

Google Calendar API quota: 1,000,000 queries/day (default project), but per-user per-100-seconds limit is 500 queries. Calendar-be has no exponential backoff on 429 errors (audit finding H-24).

```
20 users × 5 calendars × initial sync (100 events each)
= 20 × 5 × 100 = 10,000 blocker creation API calls in burst
+ 20 × 5 = 100 calendar.list calls
+ 20 × 5 = 100 events.list calls
= ~10,200 API calls in a few minutes
```

**Chain of failure:**
1. Initial sync for 20 users triggers burst of 10,000+ Google API calls
2. Google returns 429 (per-user rate limit hit)
3. No exponential backoff in code → immediate retry → more 429s → quota consumed faster
4. Per-user quota exhausted → all syncs for that user fail
5. Project-wide daily quota may also be exhausted depending on burst size
6. All syncs fail permanently until Google quota resets (24 hours)
7. Webhook processing also uses the same quota → existing syncs degrade
8. Users see "Sync Failed" with no recovery for 24 hours
9. Users contact support → support can't fix it → must wait for quota reset

---

### Scenario 7: Single Redis Instance Restart — All Sessions Lost

**Probability: Medium (happens eventually)**  
**Time to failure:** Any Redis restart (OOM, manual maintenance, Docker update)

Redis is a single instance with no persistence of sessions (sessions are in-memory only). Redis Sentinel exists as a compose file but is not enabled.

**Chain of failure:**
1. Redis restarts (OOM from scenario 5, or Docker update, or hardware issue)
2. Redis comes back with empty state
3. Every user's `SESSION_ID` cookie points to a session that no longer exists
4. session service's `SessionGuard` reads cookie → queries Redis → session not found → 401
5. **100 users simultaneously see "session expired"** → all redirected to login page
6. 100 users try to log in at once → OTP email burst → Kafka/SMTP pressure
7. All 100 create new sessions → session service + core-be handle 100 concurrent auth flows
8. Combined with scenario 3: auth infrastructure under peak load, cascading failures

---

### Scenario 8: MongoDB Data Corruption on OOM Kill

**Probability: Medium**  
**Time to failure:** Any time MongoDB container is OOMKilled

MongoDB is a single instance, not a replica set. No explicit write concern is configured. Default write concern is `{ w: 1 }` (acknowledged by the primary — but there's only one node, so this is the minimum).

**Chain of failure:**
1. MongoDB is OOMKilled mid-write (during a sync that creates 50 blocker events)
2. MongoDB restarts, replays journal
3. Partial write: 30 of 50 blocker events are in MongoDB, 20 are lost
4. No way to detect which events were lost — no transaction, no checksum
5. User's calendar shows incomplete blockers — some meetings not blocked
6. Calendar sync shows "completed" (BullMQ job succeeded before MongoDB crash)
7. Only way to recover: full resync — but user doesn't know they need to

---

## Failure Timeline — When Things Break

| User Count | What Breaks | Probability | Downtime |
|---|---|---|---|
| **1** (now) | Nothing — but Kong at 75% memory is a warning | — | — |
| **10-20** | Kong OOM → all API routes 502 | Very High | 10-30s per restart, loops |
| **20-30** | Google API quota exhaustion on initial sync burst | High | 24 hours (quota reset) |
| **30-50** | Auth cascade — concurrent token refresh → mass logout | High | Ongoing (every 15 min cycle) |
| **50-80** | Server OOM — Linux OOMKiller takes out Kafka or Postgres | High | 1-10 min, may corrupt data |
| **100** | All of the above + Redis full + MongoDB connection exhaustion | Certain | Sustained outage |
| **2-4 weeks** | Disk full from logs (regardless of user count) | Certain | Total outage until manual cleanup |

---

## Single Points of Failure

Every infrastructure component is a single instance. Any one dying takes part or all of the platform with it.

| Component | If It Dies | Data Loss? | Recovery Time | User Impact |
|---|---|---|---|---|
| **Postgres** | All auth, users, admin ops fail | Possible WAL corruption | 30-60s restart + replay | All users: can't log in, can't use admin |
| **MongoDB** | All calendar data ops fail | Possible partial writes | 30-60s restart | Calendar: stale/incomplete data |
| **Redis** | All sessions gone, all jobs lost | **Yes — all sessions wiped** | 10s restart, empty state | **100% of users logged out instantly** |
| **Kafka broker 1** | Partition leader election | None (RF=3) | 5-10s auto-recovery | Brief email delay |
| **All 3 Kafka** | OTP emails permanently broken | None | 1-5 min manual restart | Users can't log in via email OTP |
| **Kong** | All /api/* routes 502 | None | 10-30s restart | All users: API calls fail |
| **Caddy** | HTTPS terminates, site unreachable | None | 10-30s restart | All users: site down |
| **session service** | All frontend→backend calls fail | None | 10-30s restart | All users: app non-functional |
| **core-be** | Auth, users, tokens — all fail | None (stateless) | 10-30s restart | All users: auth broken |
| **calendar-be** | Calendar sync, events — all fail | Jobs in-flight lost | 10-30s restart | Calendar users: sync fails |

**Redis is the scariest single point of failure.** A restart means every user's session is gone. No persistence, no replication, no failover.

---

## Resource Contention Map

All 27 containers compete for the same 4 CPUs, 7.6 GB RAM, and single disk:

```
                    ┌─────────────────────────────────────────┐
                    │         SINGLE HETZNER SERVER            │
                    │   7.6 GB RAM │ 4 CPUs │ 150 GB disk     │
                    │   No swap    │ No HA  │ No replication   │
                    ├─────────────────────────────────────────┤
                    │                                         │
   ┌────────────────┤  KAFKA (3 brokers)                      │
   │  1.8 GB RAM    │  CPU: 5% idle, spikes on produce/consume│
   │  (24% of total)│  Disk: segment storage + logs           │
   ├────────────────┤                                         │
   │                │  DATABASES                              │
   │  Postgres      │  76 MB now, grows with connections      │
   │  MongoDB       │  179 MB now, grows with calendar data   │
   │  Redis         │  11 MB now, grows with sessions + jobs  │
   ├────────────────┤                                         │
   │                │  APP SERVICES (8 containers)            │
   │  core-be       │  115 MB — auth, users, token CRUD       │
   │  session       │  41 MB — session proxy, every request   │
   │  calendar-be   │  162 MB — sync engine, BullMQ workers   │
   │  admin-be      │  58 MB — user management                │
   │  notification  │  61 MB — Kafka consumer, SMTP           │
   │  core-fe       │  5 MB — nginx static                    │
   │  calendar-fe   │  5 MB — nginx static                    │
   │  admin-fe      │  5 MB — nginx static                    │
   ├────────────────┤                                         │
   │                │  EDGE (2 containers)                    │
   │  Caddy         │  23 MB — TLS termination, reverse proxy │
   │  Kong          │  383 MB — API gateway, rate limiting    │
   ├────────────────┤                                         │
   │                │  OBSERVABILITY (10 containers)          │
   │  Prometheus    │  92 MB — metrics scraping every 15s     │
   │  Grafana       │  87 MB — dashboards, alerting           │
   │  Loki          │  134 MB — log ingestion                 │
   │  Alloy         │  93 MB — log collection agent           │
   │  Tempo         │  22 MB — trace storage                  │
   │  + 5 exporters │  ~70 MB combined                        │
   └────────────────┤                                         │
                    └─────────────────────────────────────────┘

   Observability alone: ~500 MB RAM, ~3% CPU
   Kafka alone: 1.8 GB RAM, ~5% CPU
   Together: 2.3 GB (30% of total RAM) for infrastructure, not app logic
```

---

## What Must Happen Before 100 Users

### Tier 1 — Do now (prevents first outage)

| # | Action | Why | Effort |
|---|---|---|---|
| 1 | **Raise Kong memory limit** from 512 MB to 1 GB | 75% used with 1 user — OOMs at ~20 | 5 min |
| 2 | **Add swap** (4-8 GB) | No safety net — OOMKiller fires immediately | 10 min |
| 3 | **Configure Docker log rotation** | Disk fills in ~11 days | 10 min |
| 4 | **Enable Redis AOF persistence** | Restart = all sessions wiped | 15 min |

### Tier 2 — Do before 50 users

| # | Action | Why | Effort |
|---|---|---|---|
| 5 | **Upgrade server to 16-32 GB RAM** | 7.6 GB can't sustain 27 containers + 50 users | 1 hour (Hetzner resize) |
| 6 | **Reduce Kafka to 1 broker** | 3 brokers eat 1.8 GB for email OTPs — overkill | 1 hour |
| 7 | **Add Google API exponential backoff** | Quota exhaustion at 20 users | 2 hours |
| 8 | **Add request body limits to Kong/session service** | One large upload OOMKills Kong | 1 hour |
| 9 | **Increase Redis maxmemory to 1.5 GB** | BullMQ job retention fills 768 MB | 5 min |

### Tier 3 — Do before 100 users

| # | Action | Why | Effort |
|---|---|---|---|
| 10 | **Reduce DB pool sizes** (50 → 25 per service) | 5 × 50 = 250 exceeds Postgres max_connections 200 | 30 min |
| 11 | **Add BullMQ retry config** to sync/webhook queues | Transient failures = permanent job loss | 1 hour |
| 12 | **Enable Redis Sentinel** | Single Redis = all sessions lost on restart | 2 hours |
| 13 | **Move Kafka to separate server** (or replace with direct SMTP) | Frees 1.8 GB RAM for app containers | 1 day |
| 14 | **Configure Loki disk alerts** | Fills silently until everything crashes | 30 min |
| 15 | **Automate backups** (`make backup-cron-install`) | No backup = no recovery from data loss | 15 min |
