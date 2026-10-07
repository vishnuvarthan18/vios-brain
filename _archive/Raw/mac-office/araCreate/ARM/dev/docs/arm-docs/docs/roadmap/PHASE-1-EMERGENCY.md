# Phase 1: Emergency Stabilization

> Prevent the first outage. Everything here is under 30 minutes per task and addresses imminent failure scenarios.

**Priority:** P0 — Do now
**Effort:** ~3 hours total
**Prerequisite:** None
**Source:** Failure Analysis Tier 1 + Production Audit Top 10 (items 1-4)

---

## Failure Scenarios Addressed

| Scenario | Risk | Task |
|---|---|---|
| 1: Kong OOM at 10-20 users | Very High | T-1.1 |
| 2: Server-wide OOM at 50-80 users | High | T-1.3 |
| 4: Disk full in ~11 days | Certain | T-1.2 |
| 5: Redis fills up | High | T-1.4 |

## Critical Bugs Addressed

| Bug ID | Issue | Task |
|---|---|---|
| C-07 | Microsoft token refresh inverted | T-1.5 |
| C-08 | Microsoft expires_in unit mismatch | T-1.6 |
| C-09 | Token encryption key padding | T-1.7 |
| C-06 | Token expiry arithmetic on undefined | T-1.8 |

---

## Tasks

### T-1.1: Raise Kong memory limit from 512 MB to 1 GB

**Source:** Failure Analysis Tier 1 #1, Infra Issue I-01
**File:** `deploy/arm-deploy-make/docker-compose.yml` (Kong service)
**Risk addressed:** Kong at 75% memory with 1 user. OOMs at ~20 concurrent requests.

- [ ] Find the Kong service `mem_limit` in compose
- [ ] Change `512m` to `1g`
- [ ] Verify: `docker compose config | grep -A2 mem_limit` shows `1g` for Kong
- [ ] Deploy: restart Kong container on server

**Effort:** 5 minutes

---

### T-1.2: Configure Docker log rotation

**Source:** Failure Analysis Tier 1 #3, Scenario 4
**File:** Server-side `/etc/docker/daemon.json` OR compose `x-logging` anchor
**Risk addressed:** 27 containers writing unbounded logs. Disk fills in ~11 days at 100 users.

- [ ] Add to `/etc/docker/daemon.json` on the server:
  ```json
  {
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "50m",
      "max-file": "3"
    }
  }
  ```
- [ ] Alternatively, add `x-logging` YAML anchor to compose and apply to all services
- [ ] Restart Docker daemon: `sudo systemctl restart docker`
- [ ] Verify: `docker inspect --format='{{.HostConfig.LogConfig}}' arm-kong` shows max-size

**Effort:** 10 minutes

---

### T-1.3: Add swap space (4-8 GB)

**Source:** Failure Analysis Tier 1 #2, Scenario 2
**Risk addressed:** No swap = OOMKiller fires immediately when RAM fills. Swap gives a buffer.

- [ ] On the server:
  ```bash
  sudo fallocate -l 4G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
  ```
- [ ] Set swappiness low: `sudo sysctl vm.swappiness=10`
- [ ] Persist: `echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf`
- [ ] Verify: `free -h` shows swap

**Effort:** 10 minutes

---

### T-1.4: Enable Redis AOF persistence

**Source:** Failure Analysis Tier 1 #4, Scenario 7, Infra Issue I-03
**File:** `deploy/arm-deploy-make/docker-compose.infra.prod.yml` (Redis service)
**Risk addressed:** Redis restart = all sessions wiped, all BullMQ jobs lost. 100% of users logged out instantly.

- [ ] Add `--appendonly yes --appendfsync everysec` to Redis command
- [ ] Ensure the Redis data volume is mounted (should already be)
- [ ] Restart Redis container
- [ ] Verify: `docker exec arm-redis-prod redis-cli CONFIG GET appendonly` returns `yes`

**Effort:** 15 minutes

---

### T-1.5: Fix Microsoft token refresh inverted condition

**Source:** Production Audit C-07 (Critical), Top 10 Fix #1
**File:** `apps/arm-app-calendar/src/backend/src/microsoft/microsoft-client.factory.ts:57`
**Function:** `refreshIfExpired()`
**Bug:** `if (tokenExpiryAt > new Date())` returns early — refreshes VALID tokens, skips EXPIRED ones.

- [ ] Change condition from `>` to `<` (or `<=` with a buffer)
- [ ] Before: `if (tokenExpiryAt > new Date()) return;` (skip when valid = wrong)
- [ ] After: `if (tokenExpiryAt > new Date()) return;` -> `if (tokenExpiryAt > new Date(Date.now() + 60_000)) return;` — return early only if token expires more than 60s from now
- [ ] Run tests: `cd apps/arm-app-calendar && npx jest microsoft-client.factory --no-coverage`
- [ ] Verify: Microsoft calendar sync actually works end-to-end

**Effort:** 5 minutes

---

### T-1.6: Fix Microsoft token expires_in unit mismatch

**Source:** Production Audit C-08 (Critical), Top 10 Fix #2
**File:** `apps/arm-app-calendar/src/backend/src/microsoft/microsoft-oauth.service.ts:126`
**Function:** `exchangeAddAccountCode()`
**Bug:** `expires_in` from Microsoft is in seconds (e.g., 3600). Code subtracts `10_000` milliseconds directly: `Date.now() + expires_in - 10_000`. Should be `Date.now() + expires_in * 1000 - 10_000`.

- [ ] Find the `expires_in` usage
- [ ] Change to: `Date.now() + (expires_in * 1000) - 10_000`
- [ ] Check for same pattern in other Microsoft token files
- [ ] Run tests: `cd apps/arm-app-calendar && npx jest microsoft-oauth --no-coverage`

**Effort:** 5 minutes

---

### T-1.7: Fix token encryption key padding — reject mismatched lengths

**Source:** Production Audit C-09 (Critical), Top 10 Fix #3
**File:** `apps/arm-app-calendar/src/backend/src/common/crypto/token-encryption.service.ts:24`
**Function:** `getKey()`
**Bug:** `padEnd(32, '0').slice(0, 32)` silently truncates/pads keys. Different keys can produce the same encryption key.

- [ ] Replace padEnd/slice with length validation:
  ```typescript
  const raw = Buffer.from(this.encryptionKey, 'hex');
  if (raw.length !== 32) {
    throw new Error(`GOOGLE_TOKEN_ENCRYPTION_KEY must be exactly 64 hex chars (32 bytes), got ${this.encryptionKey.length} chars`);
  }
  return raw;
  ```
- [ ] Run tests: `cd apps/arm-app-calendar && npx jest token-encryption --no-coverage`
- [ ] Verify `.env` key is exactly 64 hex chars

**Effort:** 30 minutes

---

### T-1.8: Guard against undefined expiry_date from Google

**Source:** Production Audit C-06 (Critical)
**File:** `apps/arm-app-calendar/src/backend/src/google/google-client.factory.ts:211`
**Function:** `refreshTokenIfExpiredForAccount()`
**Bug:** `credentials.expiry_date - 10_000` produces NaN if Google returns undefined for `expiry_date`. Token never appears expired.

- [ ] Add guard: `const expiryMs = credentials.expiry_date ?? Date.now();`
- [ ] Use `expiryMs - 10_000` instead of `credentials.expiry_date - 10_000`
- [ ] Run tests: `cd apps/arm-app-calendar && npx jest google-client.factory --no-coverage`

**Effort:** 10 minutes

---

### T-1.9: Fix session service secret timing-safe comparison

**Source:** Production Audit H-15 (High), Top 10 Fix #4
**File:** `core/arm-core-be/src/auth/guards/session-secret.guard.ts:33`
**Function:** `canActivate()`
**Bug:** `!== secret` is not constant-time. Timing attack can leak the SESSION_INTERNAL_SECRET byte-by-byte.

- [ ] Replace `!==` with `crypto.timingSafeEqual()`:
  ```typescript
  const expected = Buffer.from(this.secret);
  const received = Buffer.from(headerValue);
  if (expected.length !== received.length || !crypto.timingSafeEqual(expected, received)) {
    throw new UnauthorizedException();
  }
  ```
- [ ] Add `import * as crypto from 'crypto';` if not present
- [ ] Run tests: `cd core/arm-core-be && npx jest session-secret --no-coverage`

**Effort:** 15 minutes

---

### T-1.10: Whitelist template fields in notification service

**Source:** Production Audit C-19 (Critical), Top 10 Fix #5
**File:** `services/arm-service-notification/src/mail/mail.service.ts:15`
**Function:** `sendWelcomeEmail()`
**Bug:** `...data` spreads entire Kafka payload into Handlebars context. Server-side template injection.

- [ ] Destructure only expected fields instead of spreading:
  ```typescript
  const { name, email, otpCode } = data;
  // Pass only whitelisted fields to template
  ```
- [ ] Apply same fix to `sendLoginOtpEmail()` and any other template methods
- [ ] Run tests: `cd services/arm-service-notification && npx jest mail --no-coverage`

**Effort:** 30 minutes

---

### T-1.11: Reject javascript:/data: URLs in registry DTO

**Source:** Production Audit C-18 (Critical), Top 10 Fix #6
**File:** `core/arm-core-be/src/registry/registry.dto.ts:23`
**Function:** `RegisterServiceDto`
**Bug:** `@IsUrl({ require_tld: false })` accepts `javascript:`, `data:`, `about:` URLs. Module Federation loads and executes remote code from these URLs.

- [ ] Add protocol whitelist:
  ```typescript
  @IsUrl({ require_tld: false, protocols: ['http', 'https'], require_protocol: true })
  ```
- [ ] Add test: register with `javascript:alert(1)` should return 400
- [ ] Run tests: `cd core/arm-core-be && npx jest registry --no-coverage`

**Effort:** 15 minutes

---

### T-1.12: Validate tokens before setting isAuthenticated

**Source:** Production Audit C-10 (Critical), Top 10 Fix #7
**File:** `apps/arm-app-calendar/src/backend/src/account/account-token.service.ts:60`
**Function:** `updateAccountTokens()`
**Bug:** Sets `isAuthenticated: true` unconditionally. Fake or expired tokens accepted.

- [ ] Add validation before setting `isAuthenticated`:
  ```typescript
  if (!accessToken || accessToken.trim() === '') {
    this.logger.warn(`Empty access token for account ${accountId}`);
    return;
  }
  ```
- [ ] Check that `refreshToken` is also non-empty when expected
- [ ] Run tests: `cd apps/arm-app-calendar && npx jest account-token --no-coverage`

**Effort:** 30 minutes

---

## Completion Checklist

- [ ] All 12 tasks done
- [ ] Server: Kong at 1 GB, swap enabled, log rotation on, Redis AOF on
- [ ] Code: C-06, C-07, C-08, C-09, C-10, C-18, C-19 fixed
- [ ] Security: H-15 (timing-safe session service secret) fixed
- [ ] Microsoft calendar sync functional (C-07 + C-08 were blocking it entirely)
- [ ] All test suites pass
