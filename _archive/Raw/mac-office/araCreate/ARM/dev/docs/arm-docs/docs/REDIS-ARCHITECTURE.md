# ARM Platform — Redis Architecture

> Every Redis use case across the platform — sessions, OTP storage, the admin deny-list/role cache, calendar-be's BullMQ queues, and its SSE pub/sub — with the full key-namespace inventory, TTLs, and which services are actually Sentinel-aware.
>
> [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §5 covers the Sentinel HA *topology* (containers, failover parameters). This document covers what's actually stored in Redis and — the one finding worth reading first — which of the four Redis-using services would and wouldn't survive a Sentinel failover.
>
> Verified against every service's Redis client/module source (`core-be`, `session`, `calendar-be`, `admin-be`), `apps/arm-app-calendar/src/backend/src/app.module.ts`'s BullMQ connection factory, every `BullModule.registerQueue()` call, and `deploy/arm-deploy-make/observability/prometheus.yml` (2026-08-10).

---

## 1. Redis instances — when each is used

| Environment | Topology | Reference |
|---|---|---|
| Local dev / CI-dev | Single instance (`docker-compose.infra.yml`'s `redis` service) | [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §1 |
| Production | Single instance by default (`docker-compose.infra.prod.yml`), **or** Sentinel HA selected with `REDIS_HA=yes` (replaces — not supplements — the single-instance service) | [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §5 |

**Sentinel HA is now selectable from the normal deploy flow**, which it wasn't when this document was first written: `make ci-deploy-prod REDIS_HA=yes` (or `infra-up-prod REDIS_HA=yes`) brings up the Sentinel topology and stops whichever topology isn't selected first, so the two can't run simultaneously. It previously required running the compose file by hand with nothing pointing an operator at it. **This raises the stakes of §8 rather than resolving it** — turning HA on is now a one-flag decision, while two of the four Redis-using services still can't survive the failover it enables.

All four Redis-using services (`core-be`, `session`, `calendar-be`, `admin-be`) point at the **same physical Redis instance** — there's no per-service Redis deployment. Logical separation between services' keys is achieved entirely through **key-prefix namespacing** (§2), not separate Redis instances or databases. `core-be`'s client is the only one that reads a `REDIS_DB` env var to select a logical Redis database index (defaults to `0`); none of the other three services support selecting a non-default DB index at all, and none of the `.env.example` files set `REDIS_DB` to anything but the default — in practice, every service shares logical DB 0. Namespace prefixing, not DB separation, is the real collision-prevention mechanism here.

---

## 2. Key namespace inventory

Every Redis key pattern in the platform, by owning service:

| Prefix | Owner | Purpose | TTL | §Ref |
|---|---|---|---|---|
| `session:data:{uuid}` | `session` | Session store — access/refresh JWTs, userId, ipHash | `SESSION_TTL_SECONDS`, default 2,592,000s (30 days) | §3 |
| `session:handoff:{uuid}` | `session` | Single-use handoff token (invite/setPassword session exchange) | 30s, hardcoded | §3 |
| `session:refresh-lock:{uuid}` | `session` | Coordinates concurrent JWT refresh across session service instances | 5,000ms, hardcoded | §3 |
| `otp:login-email:{userId}` | `core-be` | Email-OTP login code | 300s (`OTP_TTL_SECONDS`) | §4 |
| `otp:welcome-email:{userId}` | `core-be` | Same mechanism, different flow (welcome email) | 300s, same constant | §4 |
| `otp:attempts:login-email:{userId}` | `core-be` | Per-user OTP attempt counter, capped at 5 | 300s, set only on first increment (§4) | §4 |
| `disabled:{userId}` | `admin-be` (written) — **should be read by `core-be`/`calendar-be`, isn't** | Deny-list entry for a disabled user | 900s (15 min, matches access-token TTL) | §5, [SECURITY.md](SECURITY.md) §3 |
| `admin_role:{userId}` | `admin-be` | Cached admin role lookup, falls back to Postgres on miss | 300s (5 min) | §5 |
| `bull:initial-sync:*`, `bull:sync-poll:*`, `bull:sync-teardown:*`, `bull:dead-letter:*`, `bull:webhook-processing:*`, `bull:webhook-renewal:*`, `bull:microsoft-delta-renewal:*`, `bull:redis-bridge:*` | `calendar-be` | BullMQ's own internal key structure (job data, wait/active/completed/failed lists, etc.) — `bull:` + queue name is BullMQ's fixed convention, not something calendar-be chose | Per-job, governed by each queue's `removeOnComplete`/`removeOnFail` (§6) | §6 |
| `sse:{userId}` | `calendar-be` | Pub/sub channel — publishes `CALENDAR_UPDATED_EVENT` notifications to any subscribed SSE connection for that user | N/A (pub/sub, not a stored key) | §7 |

**No key ever collides across services** — every prefix is a distinct top-level namespace (`session:`, `otp:`, `disabled:`, `admin_role:`, `bull:`, `sse:`), and BullMQ's own `bull:` prefix is reserved by the library itself. If you're adding a new Redis use case anywhere in the platform, pick a new top-level prefix not in this table, or extend an existing owner's prefix — never write directly into another service's namespace.

---

## 3. Session store (`session`)

Full lifecycle detail is in [SESSION-FLOW.md](SESSION-FLOW.md) — this is the key-format summary:

- `session:data:{uuid}` stores the `Session` object (`accessToken`, `refreshToken`, `userId`, `userAgent`, `ipHash`, `createdAt`, `refreshedAt`) as JSON. The `SESSION_ID` cookie itself is `{uuid}.{hmac-sha256-sig}` — the Redis key only ever uses the bare `uuid` half.
- `session:handoff:{uuid}` — single-use, 30s TTL, created by `POST /session/internal/create-session` and redeemed via `getdel` by `POST /session/session/activate` (atomic read-and-delete, so a handoff can't be replayed).
- `session:refresh-lock:{uuid}` — 5-second TTL, coordinates `performRefresh`/`awaitPeerRefresh` across however many session service instances are running, so two instances don't both refresh the same user's JWT concurrently and race each other writing the result back to `session:data:{uuid}`.

---

## 4. OTP storage (`core-be`)

- `setOtp`/`getOtp`/`deleteOtp`/`getAndDeleteOtp` all key on `otp:{type}:{userId}` — `type` is `OtpType.LOGIN_EMAIL` (`'login-email'`) for the platform's actual login flow, or `OtpType.WELCOME_EMAIL` (`'welcome-email'`) for the separate welcome-email use case. `OTP_TTL_SECONDS = 300` (5 minutes) for both.
- `incrementOtpAttempts` keys on `otp:attempts:{type}:{userId}` — a plain Redis `INCR`, with `EXPIRE` set **only on the first increment** (`count === 1`) so the attempt window doesn't slide forward on every failed guess. Capped at `MAX_OTP_ATTEMPTS = 5` in `email-login.service.ts`; the 6th attempt invalidates the code outright (`deleteOtp`).
- `revokeOtpAndMarkVerified` is a single Redis pipeline (`DEL` + `SET ... EX`) that atomically deletes the OTP key and sets a `verified` marker in one round trip — the code comment is explicit about why: without pipelining, a crash between the two operations would either leave the OTP alive (replay risk) or leave the verified flag unset (user stuck).

---

## 5. Deny-list and admin role cache (`admin-be`)

- `disabled:{userId}` — written by `DenyListService` when an admin disables a user (`SET ... EX 900`), deleted on re-enable. `DISABLED_TTL_SECONDS = 900` is chosen to match the access-token TTL, per the code's own comment — the intent is that a disabled user's still-valid token stops being trusted no later than it would have expired anyway.
- **This key is written correctly but read by nobody who needs it.** Confirmed in [SECURITY.md](SECURITY.md) §3 and [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) Risk 13: neither `core-be`'s `KongJwtGuard` nor `calendar-be`'s `AuthGuard` ever checks this key. It's re-seeded from `arm_admin.disabled_users` on `admin-be` startup (`DenyListSeeder`) so it survives Redis restarts, but that only matters once something downstream actually reads it.
- `admin_role:{userId}` — cached admin-role lookup (5 min TTL), read by `AdminRoleGuard` on every admin-portal request to avoid a Postgres round trip per request. Falls back to a direct DB query (and logs a warning) if Redis is unavailable — never fails closed on a Redis outage.

---

## 6. BullMQ queues (`calendar-be`)

`calendar-be` is the only service in the platform that uses Redis as a job queue backend (BullMQ). Full inventory, cross-referenced with `apps/arm-app-calendar/CLAUDE.md`'s sync-architecture description:

| Queue name | Constant | Purpose | Retry/backoff | Cleanup |
|---|---|---|---|---|
| `initial-sync` | `SYNC_QUEUE` | Runs `performInitialSync` — first full sync when a `SyncConfig` is created | **None set on the job** — BullMQ's default (1 attempt, no retry) applies. No queue-level `defaultJobOptions` override exists either. | `removeOnComplete: {count: 50, age: 24h}`, `removeOnFail: {count: 100, age: 7d}` |
| `sync-poll` | `SYNC_POLL_QUEUE` | 5-minute repeating poll fallback for calendars without webhook support (Microsoft, or Google webhooks that failed to register) | Repeat job (`{ every: 5 * 60 * 1000 }`), not attempt/backoff-based | Not explicitly set at the repeat-job level |
| `sync-teardown` | `SYNC_TEARDOWN_QUEUE` | Tears down a sync when a user stops it | `attempts: 10`, exponential backoff (30s base... doubling), `jitter: 1` | `removeOnComplete: {count: 50, age: 24h}`, `removeOnFail: {count: 100, age: 7d}` |
| `dead-letter` | `DLQ_QUEUE` | Receives jobs that exhausted retries on `initial-sync`/`sync-poll` | N/A — destination queue, not itself retried the same way | — |
| `webhook-processing` | `WEBHOOK_PROCESSING_QUEUE` | Processes incoming Google webhook push notifications | — | — |
| `webhook-renewal` | `WEBHOOK_RENEWAL_QUEUE` | Renews Google push-notification channels before they expire (~7 days) | `attempts: 3`, exponential backoff (30s base), `jitter: 1` | `removeOnComplete: {count: 100}`, `removeOnFail: {count: 200}` |
| `microsoft-delta-renewal` | `MICROSOFT_DELTA_RENEWAL_QUEUE` | Renews Microsoft Graph delta-query subscriptions (Microsoft's equivalent housekeeping — no push notifications, so this is poll-chain renewal, not a webhook channel) | — | — |
| `redis-bridge` | (inline string, `common/redis/redis.module.ts`) | **Not a real job queue** — exists purely so `SseRedisService` (§7) can reuse BullMQ's already-established, Sentinel-aware Redis connection (`queue.client`) instead of opening a second raw `ioredis` connection that would need its own Sentinel-awareness code | N/A | N/A |

**Verified gap: `initial-sync` jobs have no retry policy at all.** Every other queue with retry semantics (`sync-teardown`, `webhook-renewal`) sets explicit `attempts`/`backoff`. The job that performs a user's very first calendar sync does not — a transient failure (a Google API blip, a momentary network issue) fails the job permanently on the first attempt, with no automatic retry, only landing in `dead-letter` via whatever explicit DLQ-routing logic exists in `dlq.service.ts`. Whether this is intentional (initial sync failures should surface to the user immediately rather than silently retry) or an oversight wasn't determined from this pass — worth confirming with whoever owns the sync engine.

**Webhook renewal's cron trigger:** `webhook-renewal.service.ts` has a `@Cron('0 0 * * *')` (daily, midnight) that scans for channels expiring within a 48-hour window and enqueues `webhook-renewal` jobs for each — this is what actually drives the 3-attempt/30s-backoff renewal jobs above, not a per-channel repeat job.

---

## 7. Pub/sub (`calendar-be`)

`SseRedisService` implements per-user real-time calendar-update push over Redis pub/sub, backing the frontend's SSE endpoint (`/session/proxy/calendar/v1/events/stream`, per `apps/arm-app-calendar/CLAUDE.md`):

- Channel pattern: `sse:{userId}`.
- **Two connections, deliberately:** the shared `REDIS_CLIENT` (from the `redis-bridge` BullMQ queue, §6) is used for `PUBLISH`, but subscribing duplicates that connection first (`this.pub.duplicate()`) — a Redis connection in subscribe mode can only run subscribe/unsubscribe commands, so it can't be the same connection used for normal `GET`/`SET`/queue operations.
- Publish is fire-and-forget — a failed `PUBLISH` is logged and swallowed, never thrown, since a missed real-time update isn't worth failing the operation that triggered it (the frontend's next poll/manual refresh still picks up the change from MongoDB directly).

---

## 8. Sentinel awareness — verified per service, and it's inconsistent

Every service's actual Redis client construction was read directly for this document, not assumed from a shared pattern. Two are Sentinel-aware; two are not:

| Service | Sentinel-aware? | Evidence |
|---|---|---|
| `core-be` | **Yes** | `libs/redis/redis.service.ts`'s own file header says "sentinel-aware client setup" — branches on `REDIS_SENTINELS` env var, connects via `{ sentinels, name: REDIS_SENTINEL_MASTER \|\| 'arm-master' }` when set |
| `calendar-be` | **Yes** | `app.module.ts`'s `BullModule.forRootAsync` factory has the identical branch — same `REDIS_SENTINELS` parsing, same `'arm-master'` default master name (matching `docker-compose.infra.redis-sentinel.yml`'s `sentinel monitor arm-master arm-redis-master 6379 2`) |
| `session` | **No — verified absent** | `redis/redis.service.ts` connects with a plain `{ host, port, password }` — no `REDIS_SENTINELS` branch, no sentinel-mode `ioredis` connection options anywhere in the file |
| `admin-be` | **No — verified absent** | `redis/redis.module.ts` connects with the same plain `{ host, port, password }` shape — no sentinel awareness |

**What this means concretely:** if Redis Sentinel HA is ever adopted in production ([INFRASTRUCTURE.md](INFRASTRUCTURE.md) §5's opt-in topology), `core-be` and `calendar-be` would follow a failover correctly — Sentinel tells them the new master and they reconnect there. **`session` and `admin-be` would not.** They're configured against a fixed `host`/`port` — if that host stops being the master (or stops responding at all) during a failover, both services keep trying to reach a Redis node that's no longer accepting writes (or is down entirely) until someone manually updates their `REDIS_HOST`/`REDIS_PORT` config and restarts them.

**The practical blast radius is severe, because of which two services these are:** `session` owns every user's session (`session:data:*`) — a failover event that `session` can't follow means **every active session breaks platform-wide** until `session` is manually pointed at the new master, not a graceful degradation. `admin-be` owns the deny-list write path and role cache — less immediately user-facing, but the admin portal's disable/enable flow would silently stop working.

This is a real gap worth closing before Sentinel HA is actually turned on for production, not after. `session`'s own `CLAUDE.md` documents a Redis Cluster/replica scaling roadmap (read replicas first, Cluster only if write throughput becomes the ceiling) — that roadmap is about *scaling*, and doesn't mention Sentinel *failover* awareness at all. This is a distinct, currently-undocumented gap from that scaling plan.

---

## 9. Memory policy and its operational implication

Server (`docker-compose.infra.prod.yml`, and all three data nodes of `docker-compose.infra.redis-sentinel.yml`): `--maxmemory 768mb --maxmemory-policy noeviction --appendonly yes --appendfsync everysec`, inside a 1.5g container limit. Local dev (`docker-compose.infra.yml`) stays at `--maxmemory 384mb` — a dev machine holds a handful of sessions and no real queue depth, and the smaller ceiling keeps the footprint off a laptop.

**maxmemory is deliberately held at ~50% of the container's cgroup limit, not just under it.** `appendonly` rewrites fork, and the copy-on-write child can transiently double the resident set; sizing `maxmemory` close to the cgroup limit would convert a recoverable Redis-level OOM error into an OOMKill of the whole container, which additionally loses the AOF write in flight.

**`noeviction` is a deliberate, meaningful choice, not a default left alone — worth being explicit about its consequence:** under `noeviction`, once Redis hits its memory ceiling, it **rejects write commands outright** rather than silently evicting old keys (which `allkeys-lru` or similar policies would do). This is the correct choice for this platform, because almost everything in Redis here is either a security-relevant control (deny-list, session store) or a durability-relevant job queue (BullMQ) — silently evicting a session key or a queued sync job to make room for a new write would be far worse than the write simply failing loudly. But it does mean a Redis memory exhaustion event surfaces as **write failures across every service simultaneously** (new sessions can't be created, OTPs can't be issued, new sync jobs can't be enqueued) rather than a slow, graceful degradation — which is why §10's memory-usage alerts exist.

AOF persistence (`appendonly yes`, `everysec` fsync) means at most ~1 second of writes is at risk on an unclean shutdown — sessions, OTPs, and queued jobs all survive a Redis restart, up to that ~1s window.

---

## 10. Monitoring

- **Infra-level metrics are collected, but not visualized in a dedicated dashboard.** `redis-exporter` (`oliver006/redis_exporter`) runs in `docker-compose.dev.yml`, scraped by Prometheus (`observability/prometheus.yml`'s `redis` job, target `arm-redis-exporter-dev:9121`) — memory usage, connected clients, commands/sec, evicted/expired keys are all being collected. **But** only one Grafana dashboard (`dependency-map.json`) references Redis at all, and only as a dependency-graph node — there's no dedicated Redis operations dashboard surfacing those exporter metrics for at-a-glance memory/eviction/connection-count monitoring, despite the metrics existing in Prometheus already.
- **Application-level Redis command latency is a separate, also-open gap.** `session/CLAUDE.md`'s own text: "there's currently no Redis-specific latency/throughput metric in the existing Prometheus setup — only `session_miss_total` and `session_proxy_duration_seconds`." This is not contradicted by the exporter existing — the exporter measures Redis server-side stats, not per-command latency as observed by each client. Both gaps are real and independent: no dashboard for the infra metrics that do exist, and no application-level command-latency instrumentation at all.
- **Redis liveness and memory alerting now exist.** `observability/grafana/provisioning/alerting/rules.yaml` carries `arm_redis_down` plus `arm_redis_memory_high` (`redis_memory_used_bytes / redis_memory_max_bytes` > 0.80 for 10m, warning) and `arm_redis_memory_critical` (> 0.92 for 2m, critical). Given §9's `noeviction` behavior these are the metrics that predict an imminent platform-wide write-failure event before it happens; the exporter data they read was already being scraped.

Still open: no dedicated Redis operations dashboard, and no application-level Redis command-latency instrumentation.

---

## 11. Known pitfalls (summary)

| Pitfall | Where | Severity |
|---|---|---|
| `session` and `admin-be` are not Sentinel-aware — a production failover would break every active session and the admin disable/enable flow | §8 | High — undocumented until now, matters before Sentinel HA is ever turned on |
| `initial-sync` BullMQ jobs have no `attempts`/`backoff` — a transient failure on a user's first sync doesn't retry | §6 | Medium — unclear if intentional |
| `disabled:{userId}` deny-list is written correctly but read by nobody (`core-be`/`calendar-be` guards don't check it) | §5 | High — already tracked as Risk 13 in [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) and [SECURITY.md](SECURITY.md), restated here for the Redis-usage angle |
| No dedicated Grafana dashboard for Redis, despite `redis_exporter` metrics already being scraped | §10 | Low — low-effort fix, data already exists |
| ~~No memory-usage alert, despite `noeviction` meaning a full Redis fails writes platform-wide, not gracefully~~ | §9, §10 | **Closed** — `arm_redis_memory_high`/`arm_redis_memory_critical` added; server `maxmemory` also raised 384mb → 768mb |

---

## 12. Related documents

| Doc | Role |
|---|---|
| [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §5 | Sentinel HA container topology and failover parameters |
| [SESSION-FLOW.md](SESSION-FLOW.md) | Full session lifecycle behind §3 |
| [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) | Risk 13 (deny-list gap), JWT/session model |
| [SECURITY.md](SECURITY.md) §3, §7 | The deny-list gap from the security-control angle; rate-limiting detail (also Redis-backed via `@nestjs/throttler`'s Redis storage on `core-be`/`session`) |
| [`apps/arm-app-calendar/CLAUDE.md`](../../../apps/arm-app-calendar/CLAUDE.md) | Sync architecture context behind §6's queue purposes |
| [KAFKA-ARCHITECTURE.md](KAFKA-ARCHITECTURE.md) | The platform's other queue system — Kafka, used only for email notifications, sharing no infrastructure with this document's Redis/BullMQ coverage |
| [ADR-004](adr/004-bullmq-vs-kafka.md), [ADR-008](adr/008-redis-sentinel-ha.md) | Why BullMQ over Kafka for calendar sync, and the Sentinel-HA rollout gap this document's §8 documents in full |
