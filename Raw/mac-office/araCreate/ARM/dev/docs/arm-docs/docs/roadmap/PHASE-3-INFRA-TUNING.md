# Phase 3: Infrastructure Right-Sizing

> Automate resource tuning so the platform handles 50+ users without manual intervention. Build `make tune-infra` and apply the immediate infrastructure fixes from the Failure Analysis.

**Priority:** P2 — Do before 50 users
**Effort:** ~12 hours total
**Prerequisite:** Phase 1 complete (can run in parallel with Phase 2)
**Source:** Infra Auto-Tune Design Spec + Failure Analysis Tier 2

---

## Failure Scenarios Addressed

| Scenario | Risk | Task |
|---|---|---|
| 1: Kong OOM | Very High | T-3.1 (auto-tune Kong) |
| 2: Server-wide OOM | High | T-3.2 (Kafka), T-3.3 (Postgres), T-3.4 (MongoDB) |
| 4: Disk full | Certain | T-3.6 (log rotation via auto-tune) |
| 5: Redis fills up | High | T-3.5 (maxmemory + policy) |

## Infra Issues Addressed

| Issue ID | Description | Task |
|---|---|---|
| I-01 | Server overcommit — containers request 15.3 GB | T-3.1, T-3.2 |
| I-02 | DB pool exhaustion (250 > max_connections 200) | T-3.3 |
| I-04 | No request body limits (covered in Phase 2 T-2.16) | — |
| I-05 | Postgres 30s statement_timeout on migrations | T-3.3 |
| I-06 | Kafka topics auto-created with 1 partition | T-3.2 |

---

## Tasks

### T-3.1: Create `make/tune.mk` — the tune-infra target

**Source:** Infra Auto-Tune Design Spec Section 2
**File:** `deploy/arm-deploy-make/make/tune.mk` (new)
**Purpose:** Shell script that reads API surface signals and writes `generated/infra-tune.env`.

- [ ] Create `make/tune.mk` with the `tune-infra` target
- [ ] Implement input signal reading:
  - Route count from `generated/kong.yml` (`grep -c 'paths:'`)
  - Service count from `services.conf`
  - Topic count from `make/infra.mk` topic creation list
  - BullMQ queue count from calendar-be source
  - Server RAM from `/proc/meminfo` or `SERVER_RAM_MB` env var
- [ ] Implement formula calculations for Kong, Kafka, Postgres, MongoDB, Redis, Docker logging
- [ ] Write output to `generated/infra-tune.env`
- [ ] Print human-readable summary to stdout
- [ ] Add `generated/infra-tune.env` to `.gitignore`
- [ ] Verify: `make tune-infra` runs without errors and produces valid output

**Effort:** 3 hours

---

### T-3.2: Kong auto-tune — worker processes and memory cache

**Source:** Infra Auto-Tune Design Spec Section 5.1
**Formulas:**
- `KONG_NGINX_WORKER_PROCESSES = min(4, ceil(ROUTE_COUNT / 50))`
- `KONG_MEM_CACHE_SIZE = max(16, ROUTE_COUNT * 2)` MB
- Container `mem_limit = (WORKERS * 120) + CACHE_MB + 50` MB

- [ ] Add Kong formulas to `tune.mk`
- [ ] Output: `KONG_NGINX_WORKER_PROCESSES`, `KONG_MEM_CACHE_SIZE`, `TUNE_KONG_MEM_LIMIT`
- [ ] Modify `docker-compose.yml` — Kong service:
  ```yaml
  environment:
    KONG_NGINX_WORKER_PROCESSES: ${KONG_NGINX_WORKER_PROCESSES:-auto}
    KONG_MEM_CACHE_SIZE: ${KONG_MEM_CACHE_SIZE:-128m}
  mem_limit: ${TUNE_KONG_MEM_LIMIT:-512m}
  ```
- [ ] Verify: `docker compose config` resolves all `${TUNE_*}` variables
- [ ] Test: 7 routes -> 1 worker, 16MB cache, 186m limit
- [ ] Test: 100 routes -> 2 workers, 200MB cache, 490m limit

**Effort:** 1 hour

---

### T-3.3: Kafka auto-tune — broker count and heap

**Source:** Infra Auto-Tune Design Spec Section 5.2 + Section 8
**Formulas:**
- Broker count: `TOPIC_COUNT > 20 ? 3 : 1`
- Heap: `TOPIC_COUNT > 20 ? 512m : 256m`
- Replication factor: matches broker count

- [ ] Add Kafka formulas to `tune.mk`
- [ ] Output: `TUNE_KAFKA_BROKER_COUNT`, `KAFKA_HEAP_OPTS`, `KAFKA_DEFAULT_REPLICATION_FACTOR`, etc.
- [ ] Generate `generated/docker-compose.kafka.yml` with 1 or 3 broker definitions
- [ ] Modify `docker-compose.infra.prod.yml`:
  - Remove hardcoded 3 broker definitions
  - Add `include: [generated/docker-compose.kafka.yml]`
- [ ] Verify: with 3 topics, only 1 Kafka broker starts
- [ ] Test: 3 topics -> 1 broker, 256m heap, RF=1
- [ ] Test: 25 topics -> 3 brokers, 512m heap, RF=3

**Effort:** 2 hours

---

### T-3.4: Postgres auto-tune — shared_buffers, max_connections, work_mem

**Source:** Infra Auto-Tune Design Spec Section 5.3
**Formulas:**
- `shared_buffers = SERVER_RAM_MB / 8`
- `max_connections = SERVICE_COUNT * 25`
- `work_mem = max(4, shared_buffers / max_connections * 2)` MB
- `effective_cache_size = SERVER_RAM_MB / 4`

- [ ] Add Postgres formulas to `tune.mk`
- [ ] Output: `TUNE_PG_SHARED_BUFFERS`, `TUNE_PG_MAX_CONNECTIONS`, `TUNE_PG_WORK_MEM`, etc.
- [ ] Modify `docker-compose.infra.prod.yml` — Postgres command:
  ```yaml
  command:
    - postgres
    - -c
    - shared_buffers=${TUNE_PG_SHARED_BUFFERS:-128MB}
    - -c
    - max_connections=${TUNE_PG_MAX_CONNECTIONS:-200}
    # ... etc
  ```
- [ ] Verify: Postgres starts with tuned values
- [ ] Test: 7.6 GB server -> shared_buffers=972MB, max_connections=200

**Effort:** 1 hour

---

### T-3.5: MongoDB auto-tune — wiredTiger cache size

**Source:** Infra Auto-Tune Design Spec Section 5.4
**Formula:** `--wiredTigerCacheSizeGB = CONTAINER_LIMIT_MB * 0.4 / 1024`
**Risk addressed:** Without this, WiredTiger sees host RAM (7.6 GB), calculates cache as 3.3 GB, OOMKills against 1 GB container limit.

- [ ] Add MongoDB formula to `tune.mk`
- [ ] Output: `TUNE_MONGO_WIREDTIGER_CACHE_SIZE_GB`
- [ ] Modify `docker-compose.infra.prod.yml` — MongoDB command:
  ```yaml
  command: >
    mongod
    --wiredTigerCacheSizeGB ${TUNE_MONGO_WIREDTIGER_CACHE_SIZE_GB:-0.4}
    --bind_ip_all --auth
  ```
- [ ] Verify: MongoDB starts with explicit cache size
- [ ] Test: 1 GB container -> 0.4 GB cache

**Effort:** 30 minutes

---

### T-3.6: Redis auto-tune — maxmemory and eviction policy

**Source:** Infra Auto-Tune Design Spec Section 5.5
**Formula:** `maxmemory = 256 + (QUEUE_COUNT * 50) + (SERVICE_COUNT * 10)` MB
**Policy:** Change from `noeviction` to `volatile-lru`

- [ ] Add Redis formulas to `tune.mk`
- [ ] Output: `TUNE_REDIS_MAXMEMORY`, `TUNE_REDIS_MAXMEMORY_POLICY`
- [ ] Modify `docker-compose.infra.prod.yml` — Redis command:
  ```yaml
  command: >
    redis-server
    --requirepass ${REDIS_PASSWORD}
    --maxmemory ${TUNE_REDIS_MAXMEMORY:-768mb}
    --maxmemory-policy ${TUNE_REDIS_MAXMEMORY_POLICY:-noeviction}
    --appendonly yes
    --appendfsync everysec
  ```
- [ ] Verify: Redis reports `volatile-lru` policy
- [ ] Test: 5 queues, 8 services -> 586mb

**Effort:** 30 minutes

---

### T-3.7: Docker log rotation via auto-tune

**Source:** Infra Auto-Tune Design Spec Section 5.6
**Values:** `max-size: 50m`, `max-file: 3`

- [ ] Add logging variables to `tune.mk` output
- [ ] Output: `TUNE_DOCKER_LOG_MAX_SIZE`, `TUNE_DOCKER_LOG_MAX_FILE`
- [ ] Add `x-logging` YAML anchor to compose files:
  ```yaml
  x-logging: &default-logging
    driver: json-file
    options:
      max-size: ${TUNE_DOCKER_LOG_MAX_SIZE:-50m}
      max-file: ${TUNE_DOCKER_LOG_MAX_FILE:-3}
  ```
- [ ] Apply `logging: *default-logging` to all service definitions
- [ ] Verify: `docker inspect` shows log rotation on containers

**Effort:** 30 minutes

---

### T-3.8: Integrate tune-infra into deploy pipeline

**Source:** Infra Auto-Tune Design Spec Section 3
**Files:**
- `deploy/arm-deploy-make/make/ci.mk` — add step to `ci-deploy-prod`
- `deploy/arm-deploy-make/make/local-dev.mk` — add step to `local-dev`
- `deploy/arm-deploy-make/make/arm.mk` — add step to `arm-run`

- [ ] Add `tune-infra` after `generate-compose` and before `up` in `ci-deploy-prod`
- [ ] Add `tune-infra` to `local-dev` startup sequence
- [ ] Add `tune-infra` to `arm-run` startup sequence
- [ ] Verify: `make arm-run` calls `tune-infra` automatically

**Effort:** 30 minutes

---

### T-3.9: Update compose files with TUNE_* variable references

**Source:** Infra Auto-Tune Design Spec Section 7
**Files:** All compose files that reference infrastructure services

- [ ] Replace hardcoded values in `docker-compose.infra.prod.yml` with `${TUNE_*:-default}`
- [ ] Replace hardcoded values in `docker-compose.yml` with `${TUNE_*:-default}`
- [ ] Replace hardcoded values in `docker-compose.local.dev.yml` with `${TUNE_*:-default}`
- [ ] Ensure every `TUNE_*` variable has a `:-default` fallback matching current hardcoded value
- [ ] Verify: `docker compose config` without `tune-infra` having run still produces valid config

**Effort:** 1 hour

---

### T-3.10: Write tune-infra unit tests

**Source:** Infra Auto-Tune Design Spec Section 12
**File:** `deploy/arm-deploy-make/tests/test-tune-infra.sh` (new)

- [ ] Test: 7 routes -> `KONG_NGINX_WORKER_PROCESSES=1`
- [ ] Test: 100 routes -> `KONG_NGINX_WORKER_PROCESSES=2`
- [ ] Test: `SERVER_RAM_MB=7782` -> `TUNE_PG_SHARED_BUFFERS=972MB`
- [ ] Test: 3 topics -> `TUNE_KAFKA_BROKER_COUNT=1`
- [ ] Test: 25 topics -> `TUNE_KAFKA_BROKER_COUNT=3`
- [ ] Test: 1024 MB container -> `TUNE_MONGO_WIREDTIGER_CACHE_SIZE_GB=0.4`
- [ ] Test: output file is syntactically valid (no spaces around `=`)
- [ ] Test: all `TUNE_*` variables have `:-default` references in compose files

**Effort:** 1 hour

---

### T-3.11: Reduce Kafka to 1 broker (manual pre-tune step)

**Source:** Failure Analysis Tier 2 #6
**Risk addressed:** 3 brokers eat 1.8 GB for email OTPs. Overkill for current scale.
**Note:** This is the immediate manual fix. Once T-3.3 (auto-tune) is deployed, this is handled automatically.

- [ ] If auto-tune is not yet deployed, manually reduce to 1 broker in compose
- [ ] Verify: notification service still produces/consumes Kafka messages
- [ ] Verify: OTP emails still send

**Effort:** 1 hour (including testing)

---

### T-3.12: Increase Redis maxmemory to 1.5 GB (manual pre-tune step)

**Source:** Failure Analysis Tier 2 #9
**Risk addressed:** BullMQ job retention fills 768 MB with `noeviction` policy.

- [ ] If auto-tune is not yet deployed, manually increase Redis maxmemory
- [ ] Change policy from `noeviction` to `volatile-lru`
- [ ] Verify: `redis-cli CONFIG GET maxmemory` returns new value

**Effort:** 5 minutes

---

### T-3.13: Fix Postgres statement_timeout for migrations

**Source:** Infra Issue I-05
**File:** `deploy/arm-deploy-make/docker-compose.infra.prod.yml`
**Bug:** 30s `statement_timeout` applies to migrations. Large index creation killed mid-flight.

- [ ] Set `statement_timeout` only for runtime connections, not for migration connections
- [ ] Or: increase to 300s and add a separate migration-specific connection with no timeout
- [ ] Verify: migration that creates indexes completes without timeout

**Effort:** 30 minutes

---

### T-3.14: Fix M-26 — No validation of MONGODB_URI format or JWT secret length

**Source:** M-26
**File:** `apps/arm-app-calendar/src/backend/src/common/config/env.ts`
**Bug:** No validation that MONGODB_URI is a valid connection string or that JWT secrets are sufficient length.

- [ ] Add startup validation: MONGODB_URI starts with `mongodb://` or `mongodb+srv://`
- [ ] Add startup validation: JWT_ACCESS_SECRET is at least 32 characters
- [ ] Throw on startup if validation fails (fail-fast)

**Effort:** 30 minutes

---

## Completion Checklist

- [ ] All 14 tasks done
- [ ] `make tune-infra` runs and produces `generated/infra-tune.env`
- [ ] All compose files use `${TUNE_*:-default}` variables
- [ ] Kong right-sized (1 worker for current scale, ~186 MB)
- [ ] Kafka reduced to 1 broker (saves 1.5 GB RAM)
- [ ] Postgres tuned (shared_buffers, max_connections, work_mem)
- [ ] MongoDB has explicit wiredTiger cache size
- [ ] Redis uses volatile-lru policy with right-sized maxmemory
- [ ] Docker log rotation active on all containers
- [ ] tune-infra integrated into deploy pipeline
- [ ] Unit tests pass
- [ ] `docker compose config` validates with and without tune-infra
