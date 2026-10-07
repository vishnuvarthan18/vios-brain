# Phase 6: Operational Maturity

> Bring the platform to full 100-user readiness. Remaining High and Medium bugs, infrastructure HA, monitoring, and resilience.

**Priority:** P3 — Before scaling to 100 users
**Effort:** ~12 hours total
**Prerequisite:** Phase 3 complete (can run in parallel with Phase 4/5)
**Source:** Failure Analysis Tier 3 + Production Audit remaining High/Medium/Low bugs

---

## Failure Scenarios Addressed

| Scenario | Risk | Task |
|---|---|---|
| 3: Cascading auth failure (30+ users) | High | T-6.4 |
| 7: Redis restart = sessions lost | Medium | T-6.6 |
| 8: MongoDB data corruption on OOM | Medium | T-6.7 |

## Infra Issues Addressed

| Issue ID | Description | Task |
|---|---|---|
| I-07 | MongoDB single instance, no replica set | T-6.7 |
| I-08 | Health checks only verify process alive | T-6.8 |
| I-10 | No backup automation | T-6.9 |
| I-12 | Log disk space not monitored | T-6.10 |

---

## Tasks

### T-6.1: Fix H-21 — Kafka connection failure doesn't block startup

**Source:** H-21
**File:** `core/arm-core-be/src/kafka/notification-producer.service.ts:32`
**Function:** `onModuleInit()`
**Bug:** Kafka connection failure is caught but startup continues. OTP emails silently fail.

- [ ] Add retry logic (3 attempts with 5s delay) on Kafka connection
- [ ] If all retries fail, log a CRITICAL-level warning (don't crash — graceful degradation)
- [ ] Set a flag that email OTP is unavailable, surface in health check
- [ ] Add test: Kafka connection failure is detected in health check

**Effort:** 30 minutes

---

### T-6.2: Fix H-28 — Case-sensitive accountId breaks Microsoft owner detection

**Source:** H-28
**File:** `apps/arm-app-calendar/src/backend/src/calendar/microsoft-calendar-api.service.ts:234`
**Function:** `listCalendarsByAccountId()`
**Bug:** Ownership check case-sensitive. Mixed-case accountId breaks owner detection.

- [ ] Use `.toLowerCase()` on both sides of the comparison
- [ ] Add test: mixed-case email still resolves ownership correctly

**Effort:** 15 minutes

---

### T-6.3: Fix H-29 — Calendar expiration field stored as string

**Source:** H-29
**File:** `apps/arm-app-calendar/src/backend/src/calendar/calendar.schema.ts:18`
**Bug:** `expiration` stored as string. Lexicographic comparison breaks on size mismatch (noted in CLAUDE.md as harmless until year 2286, but still a code smell).

- [ ] Add validation: when writing `expiration`, assert it's a 13-digit millisecond epoch string
- [ ] Add a comment documenting why this is safe and the constraint it depends on
- [ ] Consider migration to Number type if future features need it

**Effort:** 15 minutes

---

### T-6.4: Reduce DB pool sizes to prevent connection exhaustion

**Source:** Failure Analysis Tier 3 #10, Infra Issue I-02
**Bug:** 5 backends x 50 pool = 250 > Postgres max_connections 200. Auth fails under load.

- [ ] Reduce pool size from 50 to 20 per service in TypeORM/Postgres config
- [ ] 5 services x 20 = 100, well within max_connections 200
- [ ] Leave headroom for monitoring, migrations, admin queries
- [ ] Apply to: core-be, admin-be (the two services that use Postgres)
- [ ] Verify: connection pool warnings don't appear in logs under load

**Effort:** 30 minutes

---

### T-6.5: Add BullMQ retry configuration to sync/webhook queues

**Source:** Failure Analysis Tier 3 #11
**Bug:** Transient failures in sync/webhook jobs = permanent job loss. No retry configured.

- [ ] Add retry config to all BullMQ queue registrations:
  ```typescript
  {
    attempts: 3,
    backoff: { type: 'exponential', delay: 5000 },
    removeOnFail: { count: 100 },
  }
  ```
- [ ] Apply to: `INITIAL_SYNC_QUEUE`, `WEBHOOK_PROCESSING_QUEUE`, `WEBHOOK_RENEWAL_QUEUE`, `POLL_QUEUE`, `TEARDOWN_QUEUE`
- [ ] Verify: failed jobs are retried (check BullMQ dashboard or logs)

**Effort:** 1 hour

---

### T-6.6: Enable Redis Sentinel

**Source:** Failure Analysis Tier 3 #12, Scenario 7, Infra Issue I-14
**Risk addressed:** Single Redis instance. Restart = all sessions wiped. 100% of users logged out.

- [ ] Enable Redis Sentinel compose configuration (already exists as a file)
- [ ] Configure session service, calendar-be, admin-be to connect via Sentinel
- [ ] Test failover: kill primary, verify Sentinel promotes replica, verify sessions survive
- [ ] Verify: session reads/writes work through Sentinel

**Effort:** 2 hours

---

### T-6.7: Configure MongoDB write concern and consider replica set

**Source:** Infra Issue I-07, Failure Analysis Scenario 8
**Risk addressed:** MongoDB single instance, no write concern. OOMKill mid-write = partial data loss.

- [ ] Add `w: 'majority'` write concern to Mongoose connection options (even with single instance, this ensures journal acknowledgment)
- [ ] Enable journaling explicitly if not already on
- [ ] Document the replica set upgrade path for future scaling
- [ ] Verify: write concern is reflected in `db.serverStatus()`

**Effort:** 30 minutes

---

### T-6.8: Improve health checks to verify dependency readiness

**Source:** Infra Issue I-08
**Bug:** Health checks only verify process alive, not dependency readiness. Kong routes to broken backends.

- [ ] core-be health: check Postgres connection + Kafka connection
- [ ] calendar-be health: check MongoDB connection + Redis connection
- [ ] session service health: check Redis connection
- [ ] admin-be health: check Postgres connection
- [ ] notification: check Kafka consumer group membership + SMTP reachability
- [ ] Verify: health endpoint returns unhealthy when a dependency is down

**Effort:** 1 hour

---

### T-6.9: Automate backups

**Source:** Failure Analysis Tier 3 #15, Infra Issue I-10
**Tool:** `make backup-cron-install` (already exists)

- [ ] Run `make backup-cron-install` on the server
- [ ] Verify cron job is installed and runs at scheduled time
- [ ] Verify backup files are created for Postgres, MongoDB, Redis
- [ ] Test restore: `make backup-restore` works from a backup file
- [ ] Set up retention: keep last 7 daily + last 4 weekly backups

**Effort:** 30 minutes

---

### T-6.10: Configure Loki disk alerts

**Source:** Failure Analysis Tier 3 #14, Infra Issue I-12
**Risk addressed:** 100 users = ~5 GB/day logs. Disk fills in 6 days without alerting.

- [ ] Add Grafana alert rule: disk usage > 80% (warning), > 90% (critical)
- [ ] Add Loki retention policy: 14-day max (not 30-day default)
- [ ] Add alert for Loki ingestion failure
- [ ] Configure alert notification channel (email or webhook)
- [ ] Verify: alert fires when disk usage crosses threshold

**Effort:** 30 minutes

---

### T-6.11: Fix admin-be remaining Medium bugs (batch)

**Source:** M-28, M-29, M-30, M-31, M-32

- [ ] **M-28:** Add RBAC to audit log queries — admins can only query their own audit logs (unless super-admin)
- [ ] **M-29:** Add per-operation rate limiting for destructive admin ops (disable/delete user: 10/min)
- [ ] **M-30:** Reject admin promotion for disabled users — validate `isActive` before `grantAdmin()`
- [ ] **M-31:** Clean up `admin_users` references on user delete (cascade or nullify)
- [ ] **M-32:** Clear admin role cache on user re-enable

**Effort:** 1.5 hours

---

### T-6.12: Fix notification service remaining bugs (batch)

**Source:** H-34, H-35, M-33, M-34, M-35

- [ ] **H-34:** Add retry on SMTP failure (3 attempts, exponential backoff) before sending to DLQ
- [ ] **H-35:** Add SMTP connection pooling and timeout (30s connect, 60s socket)
- [ ] **M-33:** DLQ messages: persist to disk or database (not just log) and add alerting
- [ ] **M-34:** Add idempotency key to email sends — skip if already sent (dedup on Kafka message key)
- [ ] **M-35:** Add SMTP connectivity check to health endpoint

**Effort:** 1.5 hours

---

### T-6.13: Fix H-32, H-33 — Admin disable user gaps

**Source:** H-32, H-33

- [ ] **H-32:** On user disable, publish a Kafka event so session service can invalidate active sessions immediately (not wait 15 min for token expiry)
- [ ] **H-33:** Move audit logs to a separate database/schema that admin users can't directly access. Or: add tamper detection (hash chain).

**Effort:** 1 hour

---

### T-6.14: Fix core-be remaining Medium bugs (batch)

**Source:** M-01, M-02, M-03, M-04, M-05, M-06, M-07, M-08, M-10, M-11

- [ ] **M-01:** Return specific error codes from refresh endpoint (distinguish "no token" vs "refresh failed")
- [ ] **M-02, M-03:** Normalize userId to lowercase before comparison in Google/Microsoft status checks
- [ ] **M-04:** Add per-email rate limiting for OTP requests (supplement per-IP throttle)
- [ ] **M-05:** Remove refresh token from JSON response body (cookie-only)
- [ ] **M-06:** Validate Content-Type on Microsoft token response
- [ ] **M-07:** Use exact scope comparison (split on spaces, compare sets) instead of substring
- [ ] **M-08:** Add `@MaxLength(100)` to firstName/lastName in UpdateUserDto
- [ ] **M-10:** Add UUID format validation in `findOrFail()` before query
- [ ] **M-11:** Handle aborted requests in metrics middleware (use 'close' event as fallback)

**Effort:** 2 hours

---

### T-6.15: Fix session service remaining Low bugs

**Source:** L-10, L-11, L-12, M-12, M-13

- [ ] **L-10:** Return 404 (not 502) for unknown service in proxy
- [ ] **L-11:** Add SESSION_TTL_SECONDS startup validation
- [ ] **L-12:** Verify session exists before creating handoff token
- [ ] **M-13 (if not done in Phase 2):** Add explicit body size limit in session service main.ts

**Effort:** 30 minutes

---

### T-6.16: Fix calendar-be remaining Low bugs

**Source:** L-13 through L-20, M-22 through M-27

- [ ] **L-16:** Add try/catch around `queue.add()` in `renewAllDeltaChains()` — continue loop on failure
- [ ] **L-17:** Add stoppingAt filter to `findByTargetCalendarId()` for consistency
- [ ] **L-18:** Add descriptive logging for which webhook renewal validation failed
- [ ] **L-19:** Remove duplicate `AccountModule` import in GoogleModule
- [ ] **L-20:** Use env helper for JWT_SECRET instead of direct `process.env`
- [ ] **M-23:** Fix HALF_OPEN not resetting failureCount in circuit breaker
- [ ] **M-24:** Add hex format validation in decrypt
- [ ] **M-25:** Fix documentation mismatch on token part count in auth guard
- [ ] **M-27:** Remove unnecessary parseInt on already-string expiration in webhook renewal

**Effort:** 1.5 hours

---

### T-6.17: Fix core-be remaining Low bugs

**Source:** L-01 through L-09

- [ ] **L-01:** Remove dead code `determineRedirectUrl()` (never called)
- [ ] **L-03:** Sanitize firstName from email local part (strip after `+`, limit length)
- [ ] **L-07:** Exclude health endpoints from throttle decorator
- [ ] **L-08:** Add input length validation on page/limit pagination params
- [ ] **L-09:** Validate timeout is a positive integer in Postgres extra options
- [ ] **L-02, L-04, L-05, L-06:** Low-priority code quality fixes

**Effort:** 1 hour

---

### T-6.18: Fix notification/admin remaining Low bugs

**Source:** L-21 through L-25

- [ ] **L-21:** Return 409 on email uniqueness violation instead of constraint error
- [ ] **L-22:** Add circuit breaker to Kafka DLQ sends
- [ ] **L-23:** Fail startup if KAFKA_BROKERS env var is missing (instead of falling back to localhost)
- [ ] **L-24:** Add basic email format validation before SMTP send
- [ ] **L-25:** Add unique constraint on `remoteUrl` in registry entity

**Effort:** 1 hour

---

### T-6.19: Configure per-user rate limiting in Kong

**Source:** Infra Issue I-13
**Bug:** Rate limiting is platform-wide, not per-user. One user can exhaust the rate limit for all users.

- [ ] Configure Kong rate-limiting plugin with `consumer` identifier (per-user) instead of global
- [ ] Set per-user limits: 100 req/min for API, 10 req/min for auth endpoints
- [ ] Update `kong.yml.tmpl` template
- [ ] Verify: one user's requests don't block another user's

**Effort:** 30 minutes

---

### T-6.20: Add TLS cert expiration alert

**Source:** Infra Issue I-09

- [ ] Add Grafana alert: TLS certificate expires within 14 days
- [ ] Caddy auto-renews via ACME, but the alert catches renewal failures
- [ ] Verify: alert fires for a test certificate

**Effort:** 15 minutes

---

## Completion Checklist

- [ ] All 20 tasks done
- [ ] DB pool sizes reduced (5x20=100 < max_connections 200)
- [ ] BullMQ retry configured on all queues
- [ ] Redis Sentinel enabled (session failover tested)
- [ ] MongoDB write concern configured
- [ ] Health checks verify dependencies (not just process alive)
- [ ] Backups automated and tested
- [ ] Disk alerts configured
- [ ] Admin bugs fixed (RBAC, cache, audit)
- [ ] Notification retry + idempotency in place
- [ ] Per-user rate limiting in Kong
- [ ] All 114 code bugs + 14 infra issues addressed
- [ ] All test suites pass
- [ ] Platform ready for 100 concurrent users
