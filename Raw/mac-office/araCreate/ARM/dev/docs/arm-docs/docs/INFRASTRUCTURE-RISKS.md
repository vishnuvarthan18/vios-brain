# ARM Platform — Infrastructure Risks & Multi-Server Plan

**Date:** 2026-08-24
**Current state:** All ~25 containers on a single server (~13.3 GB memory allocated)
**Target state:** 4-server split on Hetzner private network

---

## 1. Critical Risks (data loss / extended downtime)

### INFRA-01: Backups are local-only by default
- **Risk:** Disk failure destroys both production data and backups simultaneously
- **Current:** `make backup-all` writes to `backups/` on the same disk. `backup-sync` to S3 exists but requires `BACKUP_S3_BUCKET` to be configured.
- **Fix:** Configure `BACKUP_S3_BUCKET` pointing to Hetzner Object Storage, schedule `make backup-sync` after each backup run. Verify restore from S3 onto a separate machine.

### INFRA-02: No database replication
- **Risk:** PostgreSQL, MongoDB, or Redis crash means downtime until container restarts + potential data loss for unfsynced writes
- **Current:** All three are standalone single-instance. `restart: unless-stopped` brings them back, but in-flight transactions are lost.
- **Fix:** In the multi-server setup: PostgreSQL streaming replication (primary + standby), MongoDB replica set (minimum 3 members for automatic failover), Redis Sentinel (compose file already exists at `docker-compose.infra.redis-sentinel.yml`).

### INFRA-03: Redis eviction policy is `noeviction`
- **Risk:** When Redis hits 768 MB, **all writes fail** — sessions break, BullMQ jobs stop, the platform is effectively down
- **Current:** `docker-compose.infra.prod.yml` sets `noeviction`. Comments indicate `volatile-lru` was the intended policy.
- **Fix:** Change to `volatile-lru`. This evicts only keys with a TTL (completed BullMQ jobs, cached data) while preserving keys without TTL. Existing alerts trigger at 80% (warning) and 92% (critical).

### INFRA-04: Kafka single broker, replication factor 1
- **Risk:** Broker crash or disk corruption loses notification events permanently. No replay possible.
- **Current:** Single KRaft broker, 512 MB, auto-create topics, `min.insync.replicas=1`.
- **Fix:** For the multi-server setup, run 3 brokers with replication factor 3. Short-term: accept the risk but ensure the notification service handles missed events gracefully (idempotent re-send on recovery).

### INFRA-05: No swap configured
- **Risk:** Memory spike triggers OOM killer with no buffer — container is hard-killed, in-flight requests and jobs lost
- **Current:** No swap partition or `memory-swap` limit in compose files.
- **Fix:** Add 2-4 GB swap on each server. Not as a crutch — as a buffer that gives you time to react before OOM kills processes.

---

## 2. Important Risks (operational)

### INFRA-06: PostgreSQL connection pool at exact capacity
- **Risk:** 4 services x 50 connections = 200, which equals `max_connections=200`. Zero headroom. One extra connection (backup pg_dump, migration, manual psql) causes connection failures.
- **Current:** No PgBouncer. TypeORM pools directly.
- **Fix:** Either bump `max_connections` to 250+ or deploy PgBouncer on the DB server. PgBouncer is better long-term — multiplexes hundreds of app connections into fewer DB connections.

### INFRA-07: Rate limiting is per-platform, not per-tenant
- **Risk:** One tenant can exhaust the 90 req/min ceiling and starve all others
- **Current:** Kong rate-limit plugin uses a single `arm-platform` JWT consumer. All tenants share the same bucket.
- **Fix:** Issue per-tenant JWT claims (e.g., `tenant_id` in the token), configure Kong to rate-limit by that claim. This requires a core-be change to include tenant context in the JWT.

### INFRA-08: No automated restore verification
- **Risk:** Backups exist but may be corrupt or incomplete — discovered only during an actual disaster
- **Current:** `make restore-postgres-drill` and `make restore-mongodb-drill` exist as manual targets. Nothing runs them on a schedule.
- **Fix:** On Server 4 (Backup), schedule a weekly cron that pulls the latest backup, restores it, runs a basic integrity check (row count, collection count), and emails the result.

### INFRA-09: Kong config template drift
- **Risk:** Direct edits to `generated/kong.yml` are silently overwritten by next `make generate-kong-config` run. Already caused SEC-007 (literal `${VAR}` in declarative config = auth bypass).
- **Current:** `kong.yml.tmpl` → `generated/kong.yml` render pipeline. No guard against direct edits.
- **Fix:** Add a header comment to `generated/kong.yml` ("DO NOT EDIT — generated from kong.yml.tmpl"), and add a CI check that diffs the generated file against a fresh render.

### INFRA-10: No infrastructure-as-code
- **Risk:** If the server dies, rebuilding is a manual process. Nobody knows how long it takes.
- **Current:** No Terraform, Ansible, or cloud-init scripts. Server setup is undocumented.
- **Fix:** Write an Ansible playbook or cloud-init script that provisions a bare server to running state. Test it by rebuilding onto Server 4. This becomes your disaster recovery runbook.

---

## 3. Worth Monitoring

### INFRA-11: Observability stack is 26% of total memory
- Prometheus, Loki, Tempo, Grafana, Alloy, 5 exporters = ~3.5 GB
- Not a problem when isolated on its own server. On a single server, it competes with app/DB for resources.

### INFRA-12: No CDN / DDoS protection
- Static assets served directly by Caddy. No edge caching, no DDoS mitigation beyond Kong rate limits.
- Fix: Put Cloudflare (free tier) in front. Handles DDoS, caches static assets, reduces bandwidth on Server 2.

### INFRA-13: TLS certificate renewal untested
- Caddy auto-renews via ACME, but if DNS or port 80 challenge fails silently, the cert expires and the site goes down.
- Fix: Add a Prometheus alert for certificate expiry < 14 days.

### INFRA-14: No load testing baseline
- Unknown how many concurrent users the platform handles before something breaks.
- Fix: Run a basic k6 or Artillery load test against staging. Establish the ceiling. Know when you need to scale.

---

## 4. Multi-Server Plan (Hetzner)

### Target Architecture

```
                        ┌─────────────────────┐
                        │   Hetzner Cloud      │
                        │   Private Network    │
                        │   10.0.0.0/16        │
                        └─────────┬────────────┘
                                  │
          ┌───────────┬───────────┼───────────┬──────────────┐
          │           │           │           │              │
    ┌─────▼─────┐ ┌───▼───┐ ┌────▼────┐ ┌────▼────┐  ┌─────▼──────┐
    │ Server 1  │ │  S2   │ │   S3    │ │   S4    │  │  Hetzner   │
    │ Database  │ │  App  │ │ Observe │ │ Backup  │  │  Object    │
    │           │ │       │ │         │ │         │  │  Storage   │
    │ Postgres  │ │ Caddy │ │ Prom    │ │ Backup  │  │  (S3)      │
    │ MongoDB   │ │ Kong  │ │ Grafana │ │ dumps   │  │            │
    │ Redis+    │ │ Auth  │ │ Loki    │ │ Restore │  │ Offsite    │
    │  Sentinel │ │ session service   │ │ Tempo   │ │ drills  │  │ copy       │
    │ Kafka     │ │ Core  │ │ Alloy   │ │         │  │            │
    │           │ │ Cal   │ │ Export  │ │         │  │            │
    │           │ │ Admin │ │         │ │         │  │            │
    │           │ │ Notif │ │         │ │         │  │            │
    └───────────┘ └───────┘ └─────────┘ └─────────┘  └────────────┘
    10.0.1.1      10.0.1.2   10.0.1.3   10.0.1.4     (API endpoint)
    No public IP  Public IP  Optional*  No public IP
```

*Server 3 gets a public IP only if Grafana is accessed externally. Otherwise, access via SSH tunnel or VPN.

### Server Sizing

| Server | Purpose | Hetzner Type | RAM | Disk | Est. Monthly |
|---|---|---|---|---|---|
| S1 — Database | Postgres, Mongo, Redis Sentinel, Kafka | CX32 or dedicated | 16 GB | NVMe | €15–30 |
| S2 — Application | Caddy, Kong, Authelia, 8 app services | CX22 / CX32 | 8–16 GB | SSD | €10–20 |
| S3 — Observability | Prometheus, Grafana, Loki, Tempo, exporters | CX22 | 8 GB | SSD | €10 |
| S4 — Backup | Backup storage, restore drills | CX11 | 4 GB | Storage Volume | €5 + storage |
| — | Object Storage | Hetzner S3 | — | — | ~€5 |

**Estimated total: €45–70/month**

### Network Rules

| From | To | Ports | Purpose |
|---|---|---|---|
| S2 (App) | S1 (DB) | 5432, 27017, 6379, 9092 | App → database connections |
| S3 (Observe) | S1 (DB) | 9187, 9216, 9121 | Exporters scraping DB metrics |
| S3 (Observe) | S2 (App) | 8001, 9090 (per service) | Prometheus scraping app metrics |
| S4 (Backup) | S1 (DB) | 5432, 27017 | pg_dump, mongodump over private network |
| S4 (Backup) | Hetzner S3 | 443 | Offsite sync |
| Internet | S2 (App) | 80, 443 | Browser traffic via Caddy |
| Internet | S3 (Observe) | 443 (optional) | Grafana (if exposed, behind Authelia) |

All other ports: **deny from public**. Database ports (5432, 27017, 6379) are never exposed to the internet.

### What Changes in Configuration

| Config | Current | After Split |
|---|---|---|
| `POSTGRES_HOST` | `postgres` (Docker DNS) | `10.0.1.1` (S1 private IP) |
| `MONGODB_URI` | `mongodb://mongo:27017/...` | `mongodb://10.0.1.1:27017/...` |
| `REDIS_HOST` | `redis` (Docker DNS) | `10.0.1.1` |
| `KAFKA_BROKERS` | `kafka:9092` | `10.0.1.1:9092` |
| Prometheus targets | `localhost:*` | `10.0.1.1:*`, `10.0.1.2:*` |
| Backup cron | `docker exec postgres pg_dump` | `pg_dump -h 10.0.1.1` from S4 |

### Migration Order

1. Provision all 4 servers on Hetzner Cloud Network
2. Set up S4 (Backup) first — start receiving backups immediately
3. Set up S1 (Database) — migrate data from current single server
4. Set up S3 (Observability) — point exporters at S1
5. Set up S2 (Application) — point at S1 for databases, update env vars
6. DNS cutover — point domain to S2's public IP
7. Decommission old single server

---

## 5. Priority Checklist

**Do before the multi-server split (on current single server):**

- [ ] Configure `BACKUP_S3_BUCKET` and verify `make backup-sync` works
- [ ] Fix Redis eviction policy: `noeviction` → `volatile-lru`
- [ ] Bump PostgreSQL `max_connections` to 250 (or add PgBouncer)
- [ ] Add 2-4 GB swap
- [ ] Document server rebuild procedure

**Do during the multi-server split:**

- [ ] Enable Redis Sentinel on S1
- [ ] Deploy PgBouncer on S1
- [ ] Set up automated backup to S4 + S3 sync to Hetzner Object Storage
- [ ] Schedule weekly restore drills on S4
- [ ] Put Cloudflare in front of S2
- [ ] Add TLS expiry alert in Prometheus
- [ ] Write Ansible playbook for server provisioning
- [ ] Run load test to establish baseline

**Do after stable on multi-server:**

- [ ] Evaluate PostgreSQL streaming replication (primary on S1, standby on S4)
- [ ] Evaluate MongoDB replica set (requires 3 nodes minimum)
- [ ] Evaluate Kafka multi-broker (3 brokers for replication factor 3)
- [ ] Implement per-tenant rate limiting in Kong
