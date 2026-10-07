# Phase 2: Security Hardening

> Close all exploitable security holes. Every task here fixes a vulnerability that an attacker could use today.

**Priority:** P1 — Do before any external users
**Effort:** ~8 hours total
**Prerequisite:** Phase 1 complete
**Source:** Production Audit Critical + High (auth, CSRF, open redirect, race conditions)

---

## Critical Bugs Addressed

| Bug ID | Issue | Task |
|---|---|---|
| C-01 | CSRF state unvalidated — Google OAuth callback | T-2.1 |
| C-02 | CSRF state unvalidated — Microsoft OAuth callback | T-2.1 |
| C-03 | No user-level auth on Google token endpoint | T-2.2 |
| C-04 | No user-level auth on Microsoft token endpoint | T-2.2 |
| C-05 | Session fixation via OAuth callback | T-2.3 |
| C-11 | Unbounded event array OOM | T-2.4 |
| C-12 | Partial sync failures treated as success | T-2.5 |
| C-16 | Weak CSRF on Google Add Account | T-2.6 |
| C-17 | Weak CSRF on Microsoft Add Account | T-2.6 |

## High Bugs Addressed

| Bug ID | Issue | Task |
|---|---|---|
| H-01 | Refresh token family revocation not transactional | T-2.7 |
| H-04 | Non-atomic session read-modify-write on Redis | T-2.8 |
| H-05 | releaseRefreshLock silent verify fail | T-2.8 |
| H-09 | Open redirect — Google Add Account | T-2.9 |
| H-10 | Open redirect — Microsoft Add Account | T-2.9 |
| H-11 | OTP brute-force off-by-one | T-2.10 |
| H-12 | OTP comparison not constant-time | T-2.10 |
| H-13 | Google token expiry timing leak | T-2.11 |
| H-14 | Microsoft token expiry timing leak | T-2.11 |
| H-22 | OTP get/increment race — brute-force bypass | T-2.12 |

---

## Tasks

### T-2.1: Validate CSRF state parameter on OAuth callbacks (Google + Microsoft)

**Source:** C-01, C-02
**Files:**
- `core/arm-core-be/src/auth/auth.controller.ts:246` (`googleAuthCallback`)
- `core/arm-core-be/src/auth/auth.controller.ts:370` (`microsoftAuthCallback`)
**Bug:** `req.query.state` is passed through with no server-side validation.

- [ ] Generate a random state nonce before redirect, store in Redis with TTL (e.g., 5 min)
- [ ] On callback, verify `req.query.state` matches the stored nonce
- [ ] Delete the nonce after use (single-use)
- [ ] Return 403 if state is missing or doesn't match
- [ ] Apply to both `googleAuthCallback()` and `microsoftAuthCallback()`
- [ ] Add tests: callback with missing state returns 403, callback with wrong state returns 403, callback with correct state succeeds

**Effort:** 1 hour

---

### T-2.2: Add user-level authorization to token endpoints

**Source:** C-03, C-04
**Files:**
- `core/arm-core-be/src/auth/auth.controller.ts:275` (`getGoogleToken`)
- `core/arm-core-be/src/auth/auth.controller.ts:399` (`getMicrosoftToken`)
**Bug:** Any internal service with `SESSION_INTERNAL_SECRET` can fetch ANY user's OAuth tokens by passing any userId.

- [ ] Add a `requesting-service` header or similar mechanism to scope which service can request which user's tokens
- [ ] Alternatively: validate that the requesting userId matches a session context (session service passes the authenticated user's ID)
- [ ] At minimum: add an audit log entry when tokens are fetched, so exfiltration is detectable
- [ ] Add tests: request for userId that doesn't match requesting context returns 403

**Effort:** 1 hour

---

### T-2.3: Fix session fixation in session service oauthCallback

**Source:** C-05
**File:** `core/arm-session/src/session/session.controller.ts:468`
**Function:** `oauthCallback()`
**Bug:** Accepts `sid` query parameter from the browser and sets it as `SESSION_ID` cookie without verifying ownership.

- [ ] Remove acceptance of `sid` from query parameters
- [ ] If session handoff is needed, use a signed, single-use handoff token stored in Redis
- [ ] The handoff token should be generated server-side before OAuth redirect, not from user input
- [ ] Add test: crafted `?sid=attacker-sid` does NOT set that sid as the session cookie

**Effort:** 1 hour

---

### T-2.4: Add max size to event array in performInitialSync

**Source:** C-11
**File:** `apps/arm-app-calendar/src/backend/src/events/sync-orchestrator.service.ts:365`
**Function:** `performInitialSync()`
**Bug:** All pages of events accumulated into unbounded `allEvents` array. 100K events = OOM.

- [ ] Add a configurable max event count (e.g., `MAX_INITIAL_SYNC_EVENTS = 10000`)
- [ ] Process events in batches instead of accumulating all into memory
- [ ] If total exceeds max, log a warning and stop fetching — create blockers for what was fetched
- [ ] Add test: verify processing stops at MAX_INITIAL_SYNC_EVENTS

**Effort:** 1 hour

---

### T-2.5: Fail job on partial blocker failure in applyDeltaEvents

**Source:** C-12
**File:** `apps/arm-app-calendar/src/backend/src/events/webhook-processor.service.ts:215`
**Function:** `applyDeltaEvents()`
**Bug:** `Promise.allSettled()` logs rejections but returns `{ outcome: 'completed' }`. BullMQ treats as success, no retry.

- [ ] Check `Promise.allSettled()` results for rejections
- [ ] If any rejections, throw an error so BullMQ retries the job
- [ ] Include the count and IDs of failed blockers in the error message
- [ ] Add test: job with partial failures throws and is retried

**Effort:** 30 minutes

---

### T-2.6: Strengthen CSRF on Add Account flows (Google + Microsoft)

**Source:** C-16, C-17
**Files:**
- `apps/arm-app-calendar/src/backend/src/google/google-oauth.service.ts:54` (`exchangeAddAccountCode`)
- `apps/arm-app-calendar/src/backend/src/microsoft/microsoft-oauth.service.ts:58` (`exchangeAddAccountCode`)
**Bug:** State validation compares plain userId string (`state !== userId`). No encryption, no expiry. Vulnerable to interception and replay.

- [ ] Use the same encrypted+signed state pattern that `handleUpdateCallback()` already uses (line 109-159 in google-oauth.service.ts)
- [ ] Include a nonce and timestamp in the state, encrypt with `GOOGLE_TOKEN_ENCRYPTION_KEY`
- [ ] Validate decryption, nonce uniqueness, and expiry (5 min max) on callback
- [ ] Apply same pattern to Microsoft Add Account
- [ ] Add tests: expired state rejected, replayed state rejected, valid state accepted

**Effort:** 1 hour

---

### T-2.7: Make refresh token family revocation transactional

**Source:** H-01
**File:** `core/arm-core-be/src/auth/auth.service.ts:63`
**Function:** `refresh()`
**Bug:** Token family revocation is not atomic. TOCTOU race: between checking the token and revoking the family, a concurrent request can use the same token.

- [ ] Wrap the token lookup + validation + rotation + family revocation in a Postgres transaction
- [ ] Use `SELECT ... FOR UPDATE` on the token row to prevent concurrent reads
- [ ] Add test: two concurrent refresh requests with the same token — one succeeds, one gets family revoked

**Effort:** 30 minutes

---

### T-2.8: Fix session update race condition

**Source:** H-04, H-05
**Files:**
- `core/arm-session/src/session/session.service.ts:61` (`updateSession`)
- `core/arm-session/src/session/session.service.ts:126` (`releaseRefreshLock`)
**Bug (H-04):** Non-atomic read-modify-write on Redis session. Concurrent updates overwrite each other.
**Bug (H-05):** `releaseRefreshLock` returns silently if verify fails. Lock stuck for 5s.

- [ ] Use Redis WATCH/MULTI/EXEC for atomic session updates, or use a Lua script
- [ ] For the lock: use a Lua script that atomically checks the lock value before releasing (standard Redis distributed lock pattern)
- [ ] Add test: concurrent session updates don't lose data

**Effort:** 1 hour

---

### T-2.9: Validate redirect URLs in Add Account flows

**Source:** H-09, H-10
**Files:**
- `core/arm-session/src/session/session.controller.ts:204` (`calendarAddAccount`)
- `core/arm-session/src/session/session.controller.ts:328` (`calendarAddAccountMicrosoft`)
**Bug:** Location header from upstream not validated. Can redirect browser to any URL.

- [ ] Validate that redirect URL is on an allowed origin (same-origin or known OAuth providers)
- [ ] Maintain an allowlist: `['accounts.google.com', 'login.microsoftonline.com', 'localhost']`
- [ ] Reject redirects to non-allowed origins with 400
- [ ] Add test: redirect to `evil.com` returns 400

**Effort:** 30 minutes

---

### T-2.10: Fix OTP brute-force off-by-one + add constant-time comparison

**Source:** H-11, H-12
**File:** `core/arm-core-be/src/auth/email-login.service.ts:79`
**Function:** `verifyOtpAndLogin()`
**Bug (H-11):** `> MAX` instead of `>= MAX` — allows one extra brute-force attempt.
**Bug (H-12):** OTP comparison not constant-time — timing attack.

- [ ] Change `>` to `>=` for the attempt count check
- [ ] Replace `===` OTP comparison with `crypto.timingSafeEqual()`:
  ```typescript
  const expected = Buffer.from(storedOtp);
  const received = Buffer.from(submittedOtp);
  if (expected.length !== received.length || !crypto.timingSafeEqual(expected, received)) {
    // invalid
  }
  ```
- [ ] Add test: attempt count at exactly MAX is rejected

**Effort:** 15 minutes

---

### T-2.11: Fix token expiry timing information leaks

**Source:** H-13, H-14
**Files:**
- `core/arm-core-be/src/auth/google-auth.service.ts:104` (`getValidGoogleToken`)
- `core/arm-core-be/src/auth/microsoft-auth.service.ts:89` (`getValidMicrosoftToken`)
**Bug:** Token expiry check leaks timing information.

- [ ] Use constant-time operations for the expiry comparison
- [ ] Add a fixed sleep/delay to normalize response time regardless of token state
- [ ] Or: always perform the full token fetch path (expired or not) and only branch on the result

**Effort:** 30 minutes

---

### T-2.12: Atomize OTP get + increment in Redis

**Source:** H-22
**File:** `core/arm-core-be/src/redis.service.ts:142`
**Functions:** `getOtp()` / `incrementOtpAttempts()`
**Bug:** Separate Redis calls for get and increment. Race condition allows brute-force bypass: submit many requests simultaneously before any increment lands.

- [ ] Combine get + increment into a single Lua script:
  ```lua
  local otp = redis.call('GET', KEYS[1])
  local attempts = redis.call('INCR', KEYS[2])
  return {otp, attempts}
  ```
- [ ] The Lua script is atomic in Redis — no race window
- [ ] Add test: 100 concurrent OTP submissions don't exceed MAX attempts

**Effort:** 30 minutes

---

## Additional Security Tasks

### T-2.13: Fix session service timing-safe string comparison length leak

**Source:** M-12
**File:** `core/arm-session/src/session/session.controller.ts:574`
**Function:** `timingSafeStringEqual()`
**Bug:** Length check early-exit leaks secret length via timing.

- [ ] Pad both strings to the same length before comparing, or always compare fixed-length hashes

**Effort:** 15 minutes

---

### T-2.14: Fix H-02, H-03 — Non-atomic token delete + insert

**Source:** H-02, H-03
**Files:**
- `core/arm-core-be/src/auth/auth.service.ts:185` (`googleLogin`)
- `core/arm-core-be/src/auth/auth.service.ts:269` (`microsoftLogin`)
**Bug:** Token delete + insert not atomic. Crash between = two tokens.

- [ ] Wrap in a transaction (same pattern as T-2.7)

**Effort:** 30 minutes

---

### T-2.15: Fix H-16 — TRUST_PROXY_HEADERS config-dependent header spoofing

**Source:** H-16
**File:** `core/arm-core-be/src/auth/guards/kong-jwt.guard.ts:28`
**Bug:** Config-dependent header spoofing risk.

- [ ] Verify TRUST_PROXY_HEADERS defaults to `false` in all environments without Kong
- [ ] Add startup validation: if TRUST_PROXY_HEADERS is true and no Kong detected, log a warning

**Effort:** 15 minutes

---

### T-2.16: Add request body limits

**Source:** Infra Issue I-04, Failure Analysis Tier 2 #8
**Files:**
- `core/arm-session/src/main.ts` (Express body limit)
- Kong config template
**Bug:** No explicit request body size limit. One 500MB upload OOMKills Kong.

- [ ] Add `app.use(express.json({ limit: '1mb' }))` to session service
- [ ] Add `app.use(express.urlencoded({ limit: '1mb' }))` to session service
- [ ] Add `client_max_body_size 10m;` to Kong/Nginx config
- [ ] For upload endpoints (future Object Storage), add a separate higher limit

**Effort:** 30 minutes

---

### T-2.17: Fix H-30 — Generic controller default ownership check is no-op

**Source:** H-30
**File:** `core/arm-core-be/src/common/generic.controller.ts:65`
**Function:** `remove()`
**Bug:** Default ownership check is no-op — subclasses inherit open delete.

- [ ] Add a default ownership check that throws if not overridden
- [ ] Or: make the base class method abstract, forcing subclasses to implement

**Effort:** 30 minutes

---

### T-2.18: Fix H-31 — Admin cache invalidation race

**Source:** H-31
**File:** `apps/arm-admin/src/backend/src/users.service.ts:216`
**Function:** `revokeAdmin()`
**Bug:** Cache invalidation race — 5 min unauthorized access window if Redis delete fails.

- [ ] Treat cache delete failure as critical — retry or fail the operation
- [ ] Add `try/catch` with retry (1 attempt) on Redis cache delete
- [ ] If both fail, revert the admin revocation in Postgres

**Effort:** 30 minutes

---

## Completion Checklist

- [ ] All 18 tasks done
- [ ] CSRF validated on all OAuth callbacks (C-01, C-02, C-16, C-17)
- [ ] Token endpoints have user-level authorization (C-03, C-04)
- [ ] Session fixation closed (C-05)
- [ ] OOM from large calendars prevented (C-11)
- [ ] Partial sync failures properly reported (C-12)
- [ ] OTP brute-force + timing attacks closed (H-11, H-12, H-22)
- [ ] Open redirects validated (H-09, H-10)
- [ ] Session races fixed (H-04, H-05)
- [ ] All test suites pass
