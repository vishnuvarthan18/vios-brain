# 01 — Pre-Split Fixes

> Risks that exist on the current single server. Fix before migration.

---

## 1.1 Fix Redis Eviction Policy

**File:** `deploy/arm-deploy-make/docker-compose.infra.prod.yml:141`

`noeviction` → `volatile-lru`

When Redis hits 768 MB with `noeviction`, all writes fail — sessions break, BullMQ stops. `volatile-lru` evicts only keys with TTL (completed jobs, cached data) while preserving non-TTL keys.

```
--maxmemory-policy ${TUNE_REDIS_MAXMEMORY_POLICY:-noeviction}
→
--maxmemory-policy ${TUNE_REDIS_MAXMEMORY_POLICY:-volatile-lru}
```

Also fix in `docker-compose.infra.redis-sentinel.yml` (lines 19, 51, 83) — hardcoded `noeviction` → `volatile-lru`.

**Verify:** `docker exec arm-redis-prod redis-cli -a $REDIS_PASSWORD CONFIG GET maxmemory-policy` returns `volatile-lru`.

---

## 1.2 Bump PostgreSQL max_connections

**File:** `deploy/arm-deploy-make/docker-compose.infra.prod.yml:55`

`200` → `250`

4 services x 50 pool = 200 connections = zero headroom. A `pg_dump`, migration, or manual `psql` session fails.

```
max_connections=${TUNE_PG_MAX_CONNECTIONS:-200}
→
max_connections=${TUNE_PG_MAX_CONNECTIONS:-250}
```

**Verify:** `docker exec arm-postgres-prod psql -U admin -c "SHOW max_connections;"` returns `250`.

---

## 1.3 Configure Offsite Backups

**File:** `deploy/arm-deploy-make/.env` (not committed)

Set these to point at Hetzner Object Storage:
- `BACKUP_S3_BUCKET`
- `BACKUP_S3_ENDPOINT`
- `BACKUP_S3_ACCESS_KEY_ID`
- `(secret removed)`
- `BACKUP_S3_REGION`

The GitHub Actions workflow already has these as `(secret removed)` secrets/vars.

**Verify:** `make backup-all` completes (local dump + S3 sync + prune). Check the bucket in Hetzner console.
