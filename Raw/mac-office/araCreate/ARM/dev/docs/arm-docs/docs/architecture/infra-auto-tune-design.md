# Infrastructure Auto-Tune — Design Spec

> A `make` target that reads the ARM API surface (routes, services, topics, queues, server RAM) and generates optimized infrastructure configuration. Deterministic, reproducible, zero runtime agents.

**Status:** Proposed  
**Date:** 2026-08-21

---

## 1. Problem

ARM's infrastructure services (Kong, Kafka, Postgres, MongoDB, Redis) run with hardcoded defaults that are either too large (3 Kafka brokers with 1 GB heap each for 50 messages/day) or too small (Postgres `shared_buffers=128MB` on a 7.6 GB server). As the API surface grows — new routes, new services, more topics — nobody remembers to re-tune these values. The result is either wasted resources or outages.

**Current waste on the server (measured 2026-08-21):**

| Service | Current RAM | Right-Sized RAM | Wasted |
|---|---|---|---|
| Kafka (3 brokers) | 1.8 GB | 300 MB (1 broker) | 1.5 GB |
| Kong (4 workers) | 383 MB | 120 MB (1 worker) | 263 MB |
| Postgres | 76 MB | 76 MB (but shared_buffers too low) | Perf loss |
| MongoDB | 179 MB | 179 MB (but wiredTiger cache not set) | Future OOM |

**1.75 GB recoverable** from Kong + Kafka alone — 23% of total server RAM.

---

## 2. Solution

A new `make tune-infra` target in `deploy/arm-deploy-make/make/tune.mk` that:

1. Reads the current API surface from already-generated config files
2. Applies deterministic formulas to calculate optimal values per service
3. Writes results to `generated/infra-tune.env`
4. Compose files reference these values via `${VAR}` interpolation

Same inputs always produce same outputs. No runtime monitoring, no sidecars, no agents.

---

## 3. Pipeline Integration

`tune-infra` runs as part of the deploy pipeline, after the API surface is known and before containers start:

```
make ci-deploy-prod
  1. env-check
  2. clone-repos
  3. infra-up-prod
  4. infra-wait-prod
  5. generate-compose-prod       ← generates app compose files from descriptors
  6. tune-infra                  ← NEW: reads generated/ configs, writes tuned values
  7. generate-kong-config        ← renders kong.yml from template
  8. up                          ← starts all containers with tuned config
```

For local dev, `make local-dev` calls `tune-infra` after `generate-compose` and before starting containers.

For `make arm-run`, `tune-infra` runs as part of the startup sequence, after infra health checks pass.

---

## 4. Input Signals

`tune-infra` reads these signals from existing files and system state:

| Signal | Source | How Read |
|---|---|---|
| Route count | `generated/kong.yml` | `grep -c 'paths:'` |
| Plugin count | `generated/kong.yml` | `grep -c '- name:'` (under plugins) |
| Service count | `services.conf` | `grep -cv '^#\|^$$'` (non-comment, non-empty lines) |
| Topic count | `make/infra.mk` topic creation list | Count of `--topic` arguments |
| BullMQ queue count | `apps/arm-app-calendar/src/backend/src/**/*.ts` | `grep -r 'QUEUE.*=' \| grep -c 'const'` |
| SSE routes | `generated/kong.yml` | `grep -c 'stream'` in paths |
| Server RAM | `/proc/meminfo` or `SERVER_RAM_MB` env var | `free -m \| awk '/Mem:/{print $2}'` |
| Container memory limits | Generated compose files | Parse `mem_limit` per service |

All inputs are deterministic and available at deploy time. No runtime metrics needed.

---

## 5. What Gets Tuned

### 5.1 Kong Gateway

| Parameter | Formula | Example (7 routes) |
|---|---|---|
| `KONG_NGINX_WORKER_PROCESSES` | `min(4, ceil(ROUTE_COUNT / 50))` | 1 |
| `KONG_MEM_CACHE_SIZE` | `max(16, ROUTE_COUNT * 2)` MB | 16m |
| Kong container `mem_limit` | `(WORKERS * 120) + CACHE_MB + 50` MB | 186m |

**Rationale:** Kong docs recommend 1 worker per CPU matched to limit, ~500 MB per worker for large deployments. For small deployments (<50 routes), 1 worker handles hundreds of req/s. The 128 MB default `db_cache` is for database-backed mode — ARM uses dbless mode with a 10 KB config, so 16 MB is sufficient.

**Scaling:** At 50 routes → 1 worker, 100 MB cache, ~270 MB limit. At 200 routes → 4 workers, 400 MB cache, ~930 MB limit.

### 5.2 Kafka

| Parameter | Formula | Example (3 topics) |
|---|---|---|
| Broker count | `TOPIC_COUNT > 20 ? 3 : 1` | 1 |
| `KAFKA_HEAP_OPTS` | `TOPIC_COUNT > 20 ? "-Xmx512m -Xms512m" : "-Xmx256m -Xms256m"` | -Xmx256m -Xms256m |
| `KAFKA_DEFAULT_REPLICATION_FACTOR` | `BROKER_COUNT > 1 ? 3 : 1` | 1 |
| `KAFKA_MIN_INSYNC_REPLICAS` | `BROKER_COUNT > 1 ? 2 : 1` | 1 |
| `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR` | Same as replication factor | 1 |
| Kafka container `mem_limit` | `HEAP_MB + 256` MB (JVM overhead) | 512m |

**Rationale:** Kafka docs say combined mode (controller+broker in same process) is suitable for "small clusters." RF=3 on 1 server provides zero additional durability over RF=1 (same disk). JVM heap of 256 MB handles thousands of messages/day.

**Scaling:** At 20+ topics → 3 brokers, 512 MB heap, RF=3. This is the inflection point where multi-broker adds value (partition leadership distribution, parallel consumption).

**Upgrade path:** When ARM moves to a second server, add that server's broker to `KAFKA_BROKERS`. Topics are reassigned automatically by Kafka partition reassignment. No code changes.

### 5.3 PostgreSQL

| Parameter | Formula | Example (7.6 GB server, 8 services) |
|---|---|---|
| `shared_buffers` | `SERVER_RAM_MB / 8` | 972 MB |
| `max_connections` | `SERVICE_COUNT * 25` | 200 |
| `work_mem` | `max(4, shared_buffers / max_connections * 2)` MB | 8 MB |
| `effective_cache_size` | `SERVER_RAM_MB / 4` | 1945 MB |
| `maintenance_work_mem` | `max(64, SERVER_RAM_MB / 32)` MB | 243 MB |
| `wal_buffers` | `max(4, shared_buffers / 32)` MB, capped at 64 MB | 30 MB |

**Rationale:** Postgres docs recommend `shared_buffers` at 25% of RAM for a dedicated server. ARM shares the server with 26 other containers, so 12.5% (1/8) is the right tradeoff. `effective_cache_size` at 25% accounts for the OS page cache shared with other processes. `max_connections` at 25 per service prevents pool exhaustion (5 backends × 25 = 125, plus overhead for monitoring, migrations, admin).

**Scaling:** On a 32 GB server → `shared_buffers=4GB`, `effective_cache_size=8GB`, `work_mem=32MB`. The formulas scale linearly with server RAM.

### 5.4 MongoDB

| Parameter | Formula | Example (1 GB container limit) |
|---|---|---|
| `--wiredTigerCacheSizeGB` | `CONTAINER_LIMIT_MB * 0.4 / 1024` | 0.4 |

**Rationale:** MongoDB docs are explicit: *"When running in containers with restricted memory, you must manually configure the cache size."* Without this, WiredTiger sees host RAM (7.6 GB), calculates cache as 3.3 GB, and OOMKills against the 1 GB container limit. 40% of container limit leaves room for connections, indexes, and the journal.

**Scaling:** If container limit increases to 2 GB → cache becomes 0.8 GB. Formula stays the same.

### 5.5 Redis

| Parameter | Formula | Example (8 services, 5 queues) |
|---|---|---|
| `maxmemory` | `256 + (QUEUE_COUNT * 50) + (SERVICE_COUNT * 10)` MB | 586 MB |
| `maxmemory-policy` | Always `volatile-lru` | volatile-lru |

**Rationale:** Redis docs document 10 eviction policies. `noeviction` (current) fails ALL writes when full — including session writes. `volatile-lru` evicts only keys with TTL set (BullMQ completed jobs have TTL, sessions have TTL), keeping the most recently used. This matches ARM's mixed workload (sessions must persist, completed jobs can be evicted).

Base 256 MB covers sessions + OTP codes. Each BullMQ queue adds ~50 MB for job history retention. Each service adds ~10 MB for miscellaneous keys.

**Scaling:** At 20 queues + 15 services → `256 + 1000 + 150 = 1406 MB`. Scales linearly with workload.

### 5.6 Docker Log Rotation

| Parameter | Value | Rationale |
|---|---|---|
| `max-size` | `50m` | Per-container log file capped at 50 MB |
| `max-file` | `3` | 3 rotated files max = 150 MB per container |

**Rationale:** Without rotation, Docker's `json-file` driver writes unbounded logs. 27 containers × uncapped logs = disk fills in ~11 days at 100 users. With `50m × 3`, worst case is 27 × 150 MB = 4 GB total — bounded.

Applied via `generated/infra-tune.env` and referenced in compose files' `logging:` block, or via Docker daemon config (`/etc/docker/daemon.json`).

---

## 6. Output

`tune-infra` writes a single file: `generated/infra-tune.env`

```env
# ============================================================
# ARM Infrastructure Tune — Auto-Generated
# Do not hand-edit. Re-generated by: make tune-infra
#
# Inputs:
#   routes=7  services=8  topics=3  queues=5  ram=7782MB
#
# To override any value, set it in .env — .env takes precedence
# over generated/infra-tune.env in Docker Compose.
# ============================================================

# Kong
KONG_NGINX_WORKER_PROCESSES=1
KONG_MEM_CACHE_SIZE=16m
TUNE_KONG_MEM_LIMIT=186m

# Kafka
KAFKA_HEAP_OPTS=-Xmx256m -Xms256m
TUNE_KAFKA_BROKER_COUNT=1
KAFKA_DEFAULT_REPLICATION_FACTOR=1
KAFKA_MIN_INSYNC_REPLICAS=1
KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1
KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1
TUNE_KAFKA_MEM_LIMIT=512m

# Postgres
TUNE_PG_SHARED_BUFFERS=972MB
TUNE_PG_MAX_CONNECTIONS=200
TUNE_PG_WORK_MEM=8MB
TUNE_PG_EFFECTIVE_CACHE_SIZE=1945MB
TUNE_PG_MAINTENANCE_WORK_MEM=243MB
TUNE_PG_WAL_BUFFERS=30MB

# MongoDB
TUNE_MONGO_WIREDTIGER_CACHE_SIZE_GB=0.4

# Redis
TUNE_REDIS_MAXMEMORY=586mb
TUNE_REDIS_MAXMEMORY_POLICY=volatile-lru

# Docker logging
TUNE_DOCKER_LOG_MAX_SIZE=50m
TUNE_DOCKER_LOG_MAX_FILE=3
```

**Prefix convention:** Values that need compose-file changes use the `TUNE_` prefix to distinguish them from existing env vars. Values that Kong/Kafka read directly from env (like `KONG_NGINX_WORKER_PROCESSES`, `KAFKA_HEAP_OPTS`) use their native names.

**Override mechanism:** Docker Compose loads env files in order. `.env` is loaded first (user overrides). `generated/infra-tune.env` is loaded second. `.env` values take precedence, so a manual override always wins.

---

## 7. Compose File Changes

The generated compose files and infra compose files need to reference `TUNE_*` variables. These are minimal, mechanical changes:

### `docker-compose.infra.prod.yml` (Postgres)

```yaml
postgres:
  command:
    - postgres
    - -c
    - shared_buffers=${TUNE_PG_SHARED_BUFFERS:-128MB}
    - -c
    - max_connections=${TUNE_PG_MAX_CONNECTIONS:-200}
    - -c
    - work_mem=${TUNE_PG_WORK_MEM:-4MB}
    - -c
    - effective_cache_size=${TUNE_PG_EFFECTIVE_CACHE_SIZE:-4GB}
    - -c
    - maintenance_work_mem=${TUNE_PG_MAINTENANCE_WORK_MEM:-64MB}
    - -c
    - wal_buffers=${TUNE_PG_WAL_BUFFERS:-4MB}
```

### `docker-compose.infra.prod.yml` (MongoDB)

```yaml
mongodb:
  command: >
    mongod
    --wiredTigerCacheSizeGB ${TUNE_MONGO_WIREDTIGER_CACHE_SIZE_GB:-0.4}
    --bind_ip_all
    --auth
```

### `docker-compose.infra.prod.yml` (Redis)

```yaml
redis:
  command: >
    redis-server
    --requirepass ${REDIS_PASSWORD}
    --maxmemory ${TUNE_REDIS_MAXMEMORY:-768mb}
    --maxmemory-policy ${TUNE_REDIS_MAXMEMORY_POLICY:-noeviction}
    --appendonly yes
    --appendfsync everysec
```

### `docker-compose.yml` (Kong)

```yaml
kong:
  environment:
    KONG_NGINX_WORKER_PROCESSES: ${KONG_NGINX_WORKER_PROCESSES:-auto}
    KONG_MEM_CACHE_SIZE: ${KONG_MEM_CACHE_SIZE:-128m}
  mem_limit: ${TUNE_KONG_MEM_LIMIT:-512m}
```

### All containers (Docker log rotation)

```yaml
x-logging: &default-logging
  driver: json-file
  options:
    max-size: ${TUNE_DOCKER_LOG_MAX_SIZE:-50m}
    max-file: ${TUNE_DOCKER_LOG_MAX_FILE:-3}

services:
  core-backend:
    logging: *default-logging
  # ... all other services
```

**Defaults:** Every `${TUNE_*}` variable has a `:-default` fallback matching the current hardcoded value. If `tune-infra` hasn't run, the platform behaves exactly as it does today. Zero risk.

---

## 8. Kafka Broker Count — Dynamic Compose Generation

When `TUNE_KAFKA_BROKER_COUNT=1`, the infra compose should only define 1 broker instead of 3. This requires `tune-infra` to generate a Kafka-specific compose fragment:

- `generated/docker-compose.kafka.yml` — contains 1 or 3 broker service definitions
- `docker-compose.infra.prod.yml` removes the hardcoded 3 broker definitions
- `docker-compose.infra.prod.yml` includes `generated/docker-compose.kafka.yml` via the `include:` directive

This follows the same pattern as the app service compose generation — the orchestrator generates, the main compose includes.

---

## 9. CLI Output

`make tune-infra` prints a human-readable summary:

```
ARM Infrastructure Tune
========================
Inputs:  7 routes, 8 services, 3 topics, 5 queues, 7782 MB RAM

Kong:      1 worker, 16 MB cache, 186 MB limit (was: 4 workers, 128 MB cache, 512 MB)
Kafka:     1 broker, 256 MB heap (was: 3 brokers, 1 GB heap)
Postgres:  shared_buffers=972MB, max_connections=200, work_mem=8MB
MongoDB:   wiredTigerCacheSizeGB=0.4 (container limit: 1024 MB)
Redis:     maxmemory=586mb, policy=volatile-lru (was: 768mb, noeviction)
Logging:   max-size=50m, max-file=3

Estimated RAM savings: ~1.75 GB
Written to: generated/infra-tune.env
```

---

## 10. Files Changed / Created

| File | Action | Purpose |
|---|---|---|
| `make/tune.mk` | **New** | The `tune-infra` target and all formulas |
| `generated/infra-tune.env` | **Generated** (gitignored) | Tuned values output |
| `generated/docker-compose.kafka.yml` | **Generated** (gitignored) | 1 or 3 Kafka broker definitions |
| `docker-compose.infra.prod.yml` | **Modified** | Replace hardcoded values with `${TUNE_*:-default}` variables, include Kafka fragment |
| `docker-compose.yml` | **Modified** | Add `${TUNE_*}` for Kong, add `x-logging` anchor |
| `docker-compose.local.dev.yml` | **Modified** | Same `${TUNE_*}` variables for local dev consistency |
| `make/ci.mk` | **Modified** | Add `tune-infra` step to `ci-deploy-prod` |
| `make/local-dev.mk` | **Modified** | Add `tune-infra` step to `local-dev` |
| `make/arm.mk` | **Modified** | Add `tune-infra` step to `arm-run` |
| `Makefile` | **No change** | Already includes all `make/*.mk` files |

---

## 11. What Doesn't Change

- No application code changes in any service
- No new containers, agents, or sidecars
- The descriptor system (`make descriptor`, `generate-compose`) stays untouched
- `REQUIRED_ENV_VARS` in service descriptors are not affected
- Manual overrides in `.env` always take precedence
- If `tune-infra` is never run, all `${TUNE_*:-default}` fallbacks match current behavior

---

## 12. Testing

### Unit tests (`tests/test-tune-infra.sh`)

- Given a fixture `kong.yml` with 7 routes → assert `KONG_NGINX_WORKER_PROCESSES=1`
- Given a fixture `kong.yml` with 100 routes → assert `KONG_NGINX_WORKER_PROCESSES=2`
- Given `SERVER_RAM_MB=7782` → assert `TUNE_PG_SHARED_BUFFERS=972MB`
- Given 3 topics → assert `TUNE_KAFKA_BROKER_COUNT=1`
- Given 25 topics → assert `TUNE_KAFKA_BROKER_COUNT=3`
- Given container limit 1024 MB → assert `TUNE_MONGO_WIREDTIGER_CACHE_SIZE_GB=0.4`
- Verify `generated/infra-tune.env` is syntactically valid (no spaces around `=`, no unescaped special chars)
- Verify all `TUNE_*` variables have corresponding `:-default` references in compose files

### Integration test

- Run `make tune-infra` on the actual workspace
- Verify `generated/infra-tune.env` exists and is non-empty
- Run `docker compose config` to verify all variable substitutions resolve (no `${TUNE_*}` literals in output)
