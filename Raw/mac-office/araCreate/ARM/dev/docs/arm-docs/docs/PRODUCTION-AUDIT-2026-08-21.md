# ARM Platform — Deep Production Audit

**Date:** 2026-08-21  
**Scope:** Every file, every function across all 7 repos + infrastructure  
**Target:** 100 concurrent users on a single Hetzner server  
**Status:** Report only — no code changes made

---

## Grand Totals

| Severity | Core-be Auth | Core-be Other | session service | Calendar Sync | Calendar Acct/Google/MS | Admin + Notification | Frontends | **Total** |
|---|---|---|---|---|---|---|---|---|
| CRITICAL | 4 | 1 | 1 | 5 | 6 | 2 | 0 | **19** |
| HIGH | 9 | 4 | 4 | 7 | 8 | 3 | 0 | **35** |
| MEDIUM | 6 | 4 | 2 | 8 | 7 | 8 | 0 | **35** |
| LOW | 5 | 4 | 3 | 5 | 3 | 5 | 0 | **25** |
| **Total** | 24 | 13 | 10 | 25 | 24 | 18 | 0 | **114** |

Frontends (core-fe, calendar-fe, admin-fe) are clean — no critical issues found.

---

## Table of Contents

1. [Critical Bugs](#1-critical-bugs-19)
2. [High Bugs](#2-high-bugs-35)
3. [Medium Bugs](#3-medium-bugs-35)
4. [Low Bugs](#4-low-bugs-25)
5. [Infrastructure Issues](#5-infrastructure-issues)
6. [Top 10 Priority Fixes](#6-top-10-priority-fixes)

---

## 1. Critical Bugs (19)

### 1.1 Auth & Session

#### C-01: CSRF state parameter unvalidated on Google OAuth callback
- **File:** `core/arm-core-be/src/auth/auth.controller.ts:246`
- **Function:** `googleAuthCallback()`
- **Bug:** `req.query.state` is read and passed through with no server-side validation. The comment says "the frontend is the only party able to verify it" — but if the frontend nonce store is broken or missing, CSRF protection is bypassed entirely.
- **Impact:** Attacker can forge OAuth callbacks, potentially linking their Google account to a victim's ARM user.

#### C-02: CSRF state parameter unvalidated on Microsoft OAuth callback
- **File:** `core/arm-core-be/src/auth/auth.controller.ts:370`
- **Function:** `microsoftAuthCallback()`
- **Bug:** Same issue as C-01 for Microsoft.

#### C-03: No user-level authorization on Google token endpoint
- **File:** `core/arm-core-be/src/auth/auth.controller.ts:275`
- **Function:** `getGoogleToken()`
- **Bug:** Protected only by `SessionSecretGuard` (checks `X-Session-Secret` header). Any internal service with the shared secret can fetch ANY user's Google tokens by passing any userId. No check that the requesting service is authorized to access that specific user's tokens.
- **Impact:** Lateral movement — a compromised calendar-be instance can exfiltrate all users' Google OAuth tokens.

#### C-04: No user-level authorization on Microsoft token endpoint
- **File:** `core/arm-core-be/src/auth/auth.controller.ts:399`
- **Function:** `getMicrosoftToken()`
- **Bug:** Same as C-03 for Microsoft tokens.

#### C-05: Session fixation via OAuth callback
- **File:** `core/arm-session/src/session/session.controller.ts:468`
- **Function:** `oauthCallback()`
- **Bug:** Accepts `sid` query parameter from the browser and sets it as a `SESSION_ID` cookie without verifying ownership. An attacker can craft a link `?sid=attacker-sid` and trick a victim into clicking it, overwriting the victim's session cookie with the attacker's session.
- **Impact:** Session hijacking — victim is logged in as attacker, or attacker captures victim's subsequent actions.

### 1.2 Token Handling

#### C-06: Token expiry arithmetic on undefined produces NaN
- **File:** `apps/arm-app-calendar/src/backend/src/google/google-client.factory.ts:211`
- **Function:** `refreshTokenIfExpiredForAccount()`
- **Bug:** `credentials.expiry_date - 10_000` — if Google returns `undefined` for `expiry_date`, this produces `NaN`. `tokenExpiryAt` is set to `NaN`, meaning the token never appears expired and is never refreshed.
- **Impact:** Stale Google access tokens used indefinitely until Google revokes them (1 hour), then all API calls fail.

#### C-07: Microsoft token refresh logic is inverted
- **File:** `apps/arm-app-calendar/src/backend/src/microsoft/microsoft-client.factory.ts:57`
- **Function:** `refreshIfExpired()`
- **Bug:** `if (tokenExpiryAt > new Date())` returns early — this means tokens are refreshed when they're VALID, and skipped when EXPIRED. The condition is backwards.
- **Impact:** All Microsoft calendar sync is broken. Expired tokens are sent to Microsoft Graph, returning 401. Valid tokens trigger unnecessary refresh calls.

#### C-08: Microsoft token expires_in unit mismatch
- **File:** `apps/arm-app-calendar/src/backend/src/microsoft/microsoft-oauth.service.ts:126`
- **Function:** `exchangeAddAccountCode()`
- **Bug:** `expires_in` from Microsoft is in seconds (e.g., 3600). The code subtracts `10_000` (milliseconds) directly: `Date.now() + expires_in - 10_000`. This sets expiry to ~10 seconds from now instead of ~50 minutes.
- **Impact:** Microsoft tokens expire almost immediately after being stored. Every subsequent API call triggers a refresh.

#### C-09: Token encryption key padding silently produces collisions
- **File:** `apps/arm-app-calendar/src/backend/src/common/crypto/token-encryption.service.ts:24`
- **Function:** `getKey()`
- **Bug:** `padEnd(32, '0').slice(0, 32)` silently truncates keys longer than 32 chars and pads shorter ones. A 48-char key and a 64-char key that share the first 32 chars produce the same derived encryption key.
- **Impact:** Tokens encrypted with one key may be decryptable with a different (truncated) key. Key rotation is silently broken.

#### C-10: Account marked authenticated without token validation
- **File:** `apps/arm-app-calendar/src/backend/src/account/account-token.service.ts:60`
- **Function:** `updateAccountTokens()`
- **Bug:** Sets `isAuthenticated: true` unconditionally after accepting tokens. No validation that the tokens are non-empty, correctly formatted, or actually valid against Google/Microsoft.
- **Impact:** Fake or expired tokens stored as "authenticated" — subsequent API calls fail with confusing errors.

### 1.3 Sync Engine

#### C-11: Unbounded event array causes OOM on large calendars
- **File:** `apps/arm-app-calendar/src/backend/src/events/sync-orchestrator.service.ts:365`
- **Function:** `performInitialSync()`
- **Bug:** All pages of events are accumulated into a single `allEvents` array with no size limit. A source calendar with 100K+ events loads the entire list into Node.js memory.
- **Impact:** OOM crash. Calendar-be restarts. All concurrent sync jobs lost.

#### C-12: Partial sync failures silently treated as success
- **File:** `apps/arm-app-calendar/src/backend/src/events/webhook-processor.service.ts:215`
- **Function:** `applyDeltaEvents()`
- **Bug:** Uses `Promise.allSettled()` but only logs rejections — the job returns `{ outcome: 'completed' }` even when 50% of blockers failed to create/update. BullMQ treats it as success, no retry.
- **Impact:** Silent data loss. Users see incomplete sync with no error indication.

#### C-13: Race condition on teardown stoppingAt gate
- **File:** `apps/arm-app-calendar/src/backend/src/events/sync-lifecycle.service.ts:498`
- **Function:** `executeTeardown()`
- **Bug:** Checks `stoppingAt` is set but does not acquire a lock. Between the check and the blocker deletion loop, a concurrent teardown can read the same blockers, causing duplicate deletion attempts.
- **Impact:** Duplicate Google API calls, orphaned blockers if concurrency assumptions break.

#### C-14: Stale blocker sweep incomplete on large calendars
- **File:** `apps/arm-app-calendar/src/backend/src/events/webhook-processor.service.ts:275`
- **Function:** `sweepStaleBlockers()`
- **Bug:** Called after 410 syncToken expiry. Assumes `currentEventIds` is the complete event list, but doesn't verify pagination was fully consumed. On calendars with >2500 events, only the first page is checked.
- **Impact:** Orphaned blockers accumulate permanently on large calendars after syncToken expiry.

#### C-15: Orphaned blockers created during cancellation window
- **File:** `apps/arm-app-calendar/src/backend/src/events/sync-orchestrator.service.ts:385`
- **Function:** `performInitialSync()`
- **Bug:** Cancellation is checked every 10 events. With slow blocker creation (5s each), the sync runs for up to 50 seconds after unsync is requested, creating blockers that will never be cleaned up.
- **Impact:** Orphaned blocker events in Google Calendar that the user cannot remove.

### 1.4 OAuth

#### C-16: Weak CSRF on Google Add Account
- **File:** `apps/arm-app-calendar/src/backend/src/google/google-oauth.service.ts:54`
- **Function:** `exchangeAddAccountCode()`
- **Bug:** State validation compares plain userId string (`state !== userId`) with no encryption. Unlike `handleUpdateCallback()` (line 109-159) which uses encrypted+signed state with expiry, Add Account state is vulnerable to interception and replay.
- **Impact:** CSRF attack can link attacker's Google calendar to victim's ARM account.

#### C-17: Weak CSRF on Microsoft Add Account
- **File:** `apps/arm-app-calendar/src/backend/src/microsoft/microsoft-oauth.service.ts:58`
- **Function:** `exchangeAddAccountCode()`
- **Bug:** Same as C-16 for Microsoft.

### 1.5 Other Services

#### C-18: Registry DTO accepts javascript: URLs for Module Federation
- **File:** `core/arm-core-be/src/registry/registry.dto.ts:23`
- **Function:** `RegisterServiceDto`
- **Bug:** `@IsUrl({ require_tld: false })` accepts `javascript:`, `data:`, and `about:` URLs. The `remoteUrl` field is used by Module Federation host in arm-core-fe to dynamically load and execute remote code.
- **Impact:** Code injection — a compromised internal service can register a `javascript:` URL and execute arbitrary code in every user's browser.

#### C-19: Email template injection via Kafka payload
- **File:** `services/arm-service-notification/src/mail/mail.service.ts:15`
- **Function:** `sendWelcomeEmail()`
- **Bug:** `...data` spreads the entire Kafka payload into the Handlebars template context. A crafted payload with `{{#if (eq 1 1)}}INJECTED{{/if}}` executes as Handlebars code.
- **Impact:** Server-side template injection (SSTI). Can leak email variables, cause ReDoS, or potentially execute code depending on registered helpers.

---

## 2. High Bugs (35)

### 2.1 Race Conditions & Concurrency

| ID | File | Function | Bug |
|---|---|---|---|
| H-01 | `core-be/auth.service.ts:63` | `refresh()` | Refresh token family revocation not transactional — TOCTOU race |
| H-02 | `core-be/auth.service.ts:185` | `googleLogin()` | Token delete + insert not atomic — crash between = two tokens |
| H-03 | `core-be/auth.service.ts:269` | `microsoftLogin()` | Same race as H-02 |
| H-04 | `session/session.service.ts:61` | `updateSession()` | Non-atomic read-modify-write on Redis — concurrent updates overwrite each other |
| H-05 | `session/session.service.ts:126` | `releaseRefreshLock()` | Returns silently if verify fails — lock stuck for 5s, blocking other instances |
| H-06 | `calendar-be/sync-config.repository.ts:64` | `findBySourceCalendarIds()` | Missing `stoppingAt` filter — webhooks processed for syncs being torn down |
| H-07 | `calendar-be/webhook-processor.service.ts:310` | `applyDeltaEvents()` | Creates blockers for targets mid-teardown (uses H-06's unfiltered query) |
| H-08 | `calendar-be/sync-poll-processor.ts:48` | `process()` | Redis lock TTL 600s — crash = 10 min poll starvation |

### 2.2 Open Redirects

| ID | File | Function | Bug |
|---|---|---|---|
| H-09 | `session/session.controller.ts:204` | `calendarAddAccount()` | Location header from upstream not validated — redirects browser to any URL |
| H-10 | `session/session.controller.ts:328` | `calendarAddAccountMicrosoft()` | Same open redirect for Microsoft |

### 2.3 Token & Auth

| ID | File | Function | Bug |
|---|---|---|---|
| H-11 | `core-be/email-login.service.ts:79` | `verifyOtpAndLogin()` | Off-by-one: `> MAX` instead of `>= MAX` — allows 6th brute-force attempt |
| H-12 | `core-be/email-login.service.ts:79` | `verifyOtpAndLogin()` | OTP comparison not constant-time — timing attack |
| H-13 | `core-be/google-auth.service.ts:104` | `getValidGoogleToken()` | Token expiry check leaks timing information |
| H-14 | `core-be/microsoft-auth.service.ts:89` | `getValidMicrosoftToken()` | Same timing issue |
| H-15 | `core-be/session-secret.guard.ts:33` | `canActivate()` | `!== secret` not constant-time — use `crypto.timingSafeEqual()` |
| H-16 | `core-be/kong-jwt.guard.ts:28` | `canActivate()` | TRUST_PROXY_HEADERS config-dependent header spoofing risk |
| H-17 | `calendar-be/google-client.factory.ts:177` | `refreshTokenIfExpiredForAccount()` | Null tokenExpiryAt blocks refresh — self-healing path broken |
| H-18 | `calendar-be/core-auth.service.ts:41` | `getGoogleToken()` | SESSION_INTERNAL_SECRET falls back to empty string — the bug fixed earlier today |
| H-19 | `calendar-be/google-calendar-api.service.ts:465` | `listBlockerEvents()` | Searches `[ BLOCKER ]` title nobody writes — orphan sweep permanently inert |
| H-20 | `calendar-be/microsoft-calendar-api.service.ts:291` | `listBlockerEvents()` | Same inert orphan sweep for Microsoft |

### 2.4 Error Handling & Resilience

| ID | File | Function | Bug |
|---|---|---|---|
| H-21 | `core-be/kafka/notification-producer.service.ts:32` | `onModuleInit()` | Kafka connection failure doesn't block startup — OTP emails silently fail |
| H-22 | `core-be/redis.service.ts:142` | `getOtp()` / `incrementOtpAttempts()` | Separate Redis calls — brute-force bypass via race |
| H-23 | `calendar-be/sync-lifecycle.service.ts:533` | `executeTeardown()` | Partial deletion throws → BullMQ retry storm on already-deleted blockers |
| H-24 | `calendar-be/sync-orchestrator.service.ts:395` | `performInitialSync()` | No exponential backoff on Google 429 in event listing loop |
| H-25 | `calendar-be/webhook-processor.service.ts:650` | `handleTargetWebhook()` | Blindly recreates blockers for all syncs on target — mismatched blocker recreation |
| H-26 | `calendar-be/webhook-processor.service.ts:160` | `handleWebhook()` | No notification timestamp validation — stale webhooks apply outdated events |
| H-27 | `calendar-be/google-calendar-api.service.ts:105` | `listCalendarsByAccountId()` | Empty filtered list triggers deleteStaleForAccount with empty returnedIds — deletes ALL calendars |

### 2.5 Schema & Data

| ID | File | Function | Bug |
|---|---|---|---|
| H-28 | `calendar-be/microsoft-calendar-api.service.ts:234` | `listCalendarsByAccountId()` | Ownership check case-sensitive — mixed-case accountId breaks owner detection |
| H-29 | `calendar-be/calendar.schema.ts:18` | Calendar schema | `expiration` as string — lexicographic comparison breaks on size mismatch |
| H-30 | `core-be/generic.controller.ts:65` | `remove()` | Default ownership check is no-op — subclasses inherit open delete |

### 2.6 Admin & Notification

| ID | File | Function | Bug |
|---|---|---|---|
| H-31 | `admin-be/users.service.ts:216` | `revokeAdmin()` | Cache invalidation race — 5 min unauthorized access window if Redis delete fails |
| H-32 | `admin-be/users.service.ts:139` | `disableUser()` | Doesn't invalidate active access tokens — 15 min window |
| H-33 | `admin-be/users.service.ts:153` | `disableUser()` | Audit logs in same DB — admin with DB access can delete own audit trail |
| H-34 | `notification/mail.service.ts:28` | `sendLoginOtpEmail()` | No retry on SMTP failure — OTP emails go to DLQ permanently |
| H-35 | `notification/mail.module.ts:19` | `buildMailerConfig()` | No SMTP connection pooling or timeout — can hang indefinitely |

---

## 3. Medium Bugs (35)

### 3.1 Core-be Auth

| ID | File | Function | Bug |
|---|---|---|---|
| M-01 | `core-be/auth.controller.ts:103` | `refresh()` | Generic error return — client can't distinguish "no token" from "refresh failed" |
| M-02 | `core-be/auth.controller.ts:297` | `getGoogleStatus()` | Case-sensitive userId check may fail on format mismatch |
| M-03 | `core-be/auth.controller.ts:422` | `getMicrosoftStatus()` | Same as M-02 |
| M-04 | `core-be/auth.controller.ts:55` | `requestLoginOtp()` | Throttle is per-IP not per-email — ineffective against distributed brute-force |
| M-05 | `core-be/auth.controller.ts:92` | `verifyLoginOtp()` | Refresh token returned in JSON body — increases XSS impact |
| M-06 | `core-be/microsoft-auth.service.ts:157` | `refreshMicrosoftToken()` | No Content-Type validation on token response |
| M-07 | `core-be/google-auth.service.ts:52` | `saveGoogleTokens()` | Scope validation uses substring match instead of exact |

### 3.2 Core-be Other

| ID | File | Function | Bug |
|---|---|---|---|
| M-08 | `core-be/users/dto/update-user.dto.ts:38` | UpdateUserDto | No `@MaxLength()` on firstName/lastName — XSS via stored name |
| M-09 | `core-be/users/dto/is-avatar-data-url.validator.ts:60` | `checkAvatarDataUrl()` | No base64 string length check before decode — memory pressure |
| M-10 | `core-be/common/generic.service.ts:29` | `findOrFail()` | No UUID format validation — leaks user input in error messages |
| M-11 | `core-be/common/metrics/http-metrics.middleware.ts:24` | `httpMetricsMiddleware()` | `res.once('finish')` may not fire on aborted requests — metric inaccuracy |

### 3.3 session service

| ID | File | Function | Bug |
|---|---|---|---|
| M-12 | `session/session.controller.ts:574` | `timingSafeStringEqual()` | Length check early-exit leaks secret length via timing |
| M-13 | `session/main.ts` | `bootstrap()` | No explicit request body size limit — Express defaults may allow large payloads |

### 3.4 Calendar-be Sync Engine

| ID | File | Function | Bug |
|---|---|---|---|
| M-14 | `calendar-be/sync-orchestrator.service.ts:360` | `queueInitialSync()` | No backoff on duplicate sync detection — frontend must handle retries |
| M-15 | `calendar-be/webhook-processor.service.ts:485` | `pollCalendar()` | Full resync on syncToken invalid loads all events into memory |
| M-16 | `calendar-be/sync-lifecycle.service.ts:511` | `executeTeardown()` | Mongo delete BEFORE Google delete — Google failure = true orphans |
| M-17 | `calendar-be/event.repository.ts:81` | `findBlockersBySourceEvent()` | `$or` query without composite index — slow on large datasets |
| M-18 | `calendar-be/webhook-processor.service.ts:1048` | `fullyResyncCalendar()` | Unbounded event pagination — memory risk |
| M-19 | `calendar-be/sync-queue-events.service.ts:33` | `onSyncCompleted()` | JSON parse error swallowed — frontend never notified |
| M-20 | `calendar-be/teardown-queue-events.service.ts:37` | `onTeardownCompleted()` | SSE publish failure swallowed — frontend stale |
| M-21 | `calendar-be/dlq.service.ts:26` | `moveToDlq()` | Auto-removed jobs bypass DLQ — no audit trail |

### 3.5 Calendar-be Account/Google/Microsoft

| ID | File | Function | Bug |
|---|---|---|---|
| M-22 | `calendar-be/account-connection.service.ts:130` | `syncFromCoreAuth()` | Sets `onSync: false` — inconsistent with `connectGoogleAccount()` which sets `true` |
| M-23 | `calendar-be/common/circuit-breaker.ts:71` | `execute()` | HALF_OPEN doesn't reset failureCount — circuit can immediately reopen |
| M-24 | `calendar-be/common/crypto/token-encryption.service.ts:42` | `decrypt()` | No hex format validation — invalid hex silently produces garbage |
| M-25 | `calendar-be/auth/auth.guard.ts:50` | `canActivate()` | Documentation mismatch on token part count |
| M-26 | `calendar-be/common/config/env.ts` | (overall) | No validation of MONGODB_URI format or JWT secret length |
| M-27 | `calendar-be/webhook/webhook-renewal.service.ts:39` | `findExpiringChannels()` | Unnecessary parseInt on already-string expiration |

### 3.6 Admin-be + Notification

| ID | File | Function | Bug |
|---|---|---|---|
| M-28 | `admin-be/audit.service.ts:26` | `findAll()` | No RBAC — any admin can query any other admin's audit logs |
| M-29 | `admin-be/app.module.ts:81` | (global config) | 100 req/min global — no per-operation rate limiting for destructive ops |
| M-30 | `admin-be/users.service.ts:183` | `grantAdmin()` | Can promote disabled users to admin without validation |
| M-31 | `admin-be/users.service.ts:227` | `deleteUser()` | Dangling admin_users references — no cleanup on delete |
| M-32 | `admin-be/users.service.ts:167` | `enableUser()` | Admin role cache not cleared on re-enable |
| M-33 | `notification/dlq-consumer.controller.ts:14` | `handleDlqMessage()` | DLQ messages only logged — lost on restart, no alerting |
| M-34 | `notification/main.ts:31` | (consumer config) | Missing idempotency — duplicate emails on rebalance |
| M-35 | `notification/health.controller.ts:19` | `ready()` | SMTP connectivity not checked in health endpoint |

---

## 4. Low Bugs (25)

### 4.1 Core-be Auth

| ID | File | Function | Bug |
|---|---|---|---|
| L-01 | `core-be/auth.controller.ts:430` | `determineRedirectUrl()` | Dead code — never called |
| L-02 | `core-be/auth.controller.ts:92` | `verifyLoginOtp()` | Cookie AND body both return refresh token — redundant exposure |
| L-03 | `core-be/email-login.service.ts:107` | `verifyOtpAndLogin()` | firstName from email local part — `attacker+spam@x.com` → ugly name |
| L-04 | `core-be/auth.service.ts:112` | `logout()` | Token-not-found is silent — idempotent but could mask issues |
| L-05 | `core-be/jwt.strategy.ts:28` | `signedWithSecret()` | Broad catch on timingSafeEqual — may hide crypto failures |

### 4.2 Core-be Other

| ID | File | Function | Bug |
|---|---|---|---|
| L-06 | `core-be/libs/crypto/token-encryption.service.ts` | (overall) | No comments explaining why GCM, why random IV |
| L-07 | `core-be/health/health.controller.ts:43` | `check()` | Health endpoints not excluded from throttle |
| L-08 | `core-be/registry/registry.controller.ts:40` | `getAll()` | Missing input length validation on page/limit params |
| L-09 | `core-be/database/postgres-extra-options.ts:20` | `buildPostgresExtraOptions()` | No validation that timeout is a positive integer |

### 4.3 session service

| ID | File | Function | Bug |
|---|---|---|---|
| L-10 | `session/proxy.service.ts:88` | `forward()` | Unknown service returns 502 instead of 404 |
| L-11 | `session/config/env.validation.ts:15` | (overall) | SESSION_TTL_SECONDS not validated at startup |
| L-12 | `session/session.service.ts:90` | `createHandoffToken()` | Doesn't verify session exists before creating handoff |

### 4.4 Calendar-be Sync Engine

| ID | File | Function | Bug |
|---|---|---|---|
| L-13 | `calendar-be/events/blocker-content.util.ts:60` | `resolveBlockerContent()` | Null-safe visibility check could be clearer |
| L-14 | `calendar-be/events/webhook-processor.service.ts:180` | `handleWebhook()` | Channel token verification underdocumented |
| L-15 | `calendar-be/events/sync-poll-processor.ts:46` | `process()` | Redis client pattern suggests manual management (BullMQ handles it) |
| L-16 | `calendar-be/events/microsoft-delta-renewal.service.ts:56` | `renewAllDeltaChains()` | No error handling on queue.add — loop stops on first failure |
| L-17 | `calendar-be/events/sync-config.repository.ts:27` | `findByTargetCalendarId()` | Missing stoppingAt filter (inconsistent with other methods) |

### 4.5 Calendar-be Account/Google/Microsoft

| ID | File | Function | Bug |
|---|---|---|---|
| L-18 | `calendar-be/webhook/webhook-renewal.processor.ts:87` | `process()` | No logging of which validation failed — hard to debug |
| L-19 | `calendar-be/google/google.module.ts:17` | GoogleModule | AccountModule imported twice (direct + forwardRef) |
| L-20 | `calendar-be/google/google.module.ts` | (overall) | JWT_SECRET used directly, not via env helper |

### 4.6 Admin-be + Notification

| ID | File | Function | Bug |
|---|---|---|---|
| L-21 | `admin-be/users.service.ts:120` | `updateUser()` | Email uniqueness not validated — constraint error instead of 409 |
| L-22 | `notification/dlq.service.ts:37` | `send()` | No circuit breaker — Kafka outage hangs DLQ sends |
| L-23 | `notification/lib/kafka-brokers.ts:15` | `parseKafkaBrokers()` | Missing env var silently falls back to localhost in production |
| L-24 | `notification/mail.service.ts:15` | `sendWelcomeEmail()` | No email format validation before sending |
| L-25 | `notification/registry/registry.entity.ts:29` | RegistryEntity | No unique constraint on remoteUrl — duplicate registrations possible |

---

## 5. Infrastructure Issues

### 5.1 Critical

| ID | Issue | File | Impact at 100 Users |
|---|---|---|---|
| I-01 | **Server overcommit**: 25+ containers requesting 15.3 GB RAM + 16.4 CPU | `docker-compose.*.yml` | OOMKills, 2s+ latency |
| I-02 | **DB pool exhaustion**: Postgres `max_connections=200`, 5 backends × pool 50 = 250 | `docker-compose.infra.prod.yml` | Auth fails, cascading 503s |
| I-03 | **Single Redis (768MB)** for sessions + BullMQ jobs, `noeviction` policy | `docker-compose.infra.prod.yml` | Session writes fail, jobs dropped |
| I-04 | **No request body limits** on Kong/Caddy | `docker-compose.yml`, `Caddyfile.prod` | One 500MB upload OOMKills Kong |

### 5.2 High

| ID | Issue | File | Impact at 100 Users |
|---|---|---|---|
| I-05 | Postgres 30s `statement_timeout` applies to migrations | `docker-compose.infra.prod.yml` | Large index creation killed mid-flight |
| I-06 | Kafka topics auto-created with 1 partition | `docker-compose.infra.prod.yml` | Email sends serialized, throughput ceiling |
| I-07 | MongoDB single instance, no replica set, no write concern | `docker-compose.infra.prod.yml` | Silent data loss on container restart |
| I-08 | Health checks only verify process alive, not dependency readiness | `docker-compose.yml` | False positives, Kong routes to broken backends |

### 5.3 Medium

| ID | Issue | File |
|---|---|---|
| I-09 | No TLS cert expiration alert in Grafana | `docker-compose.yml` (Caddy) |
| I-10 | No backup automation by default — `make backup-cron-install` is manual | `make/backup.mk` |
| I-11 | Observability stack not HA — Prometheus/Loki single instance | `docker-compose.yml` |
| I-12 | Log disk space not monitored — 100 users = ~5GB/day, disk fills in 6 days | `observability/loki-config.yml` |
| I-13 | Rate limiting is platform-wide, not per-user — one user blocks all | `kong/kong.yml.tmpl` |
| I-14 | Single Redis (no Sentinel) — restart = all sessions lost | `docker-compose.infra.prod.yml` |

---

## 6. Top 10 Priority Fixes

| # | Bug ID | Fix | Impact | Effort |
|---|---|---|---|---|
| 1 | C-07 | Fix inverted condition in Microsoft `refreshIfExpired()` | All Microsoft sync broken now | 5 min |
| 2 | C-08 | Fix `expires_in` seconds → milliseconds conversion | Microsoft tokens expire in 10s | 5 min |
| 3 | C-09 | Reject mismatched key lengths in `getKey()` instead of padding | Different keys produce same encryption | 30 min |
| 4 | H-15 | Replace `!==` with `crypto.timingSafeEqual()` in SessionSecretGuard | Timing attack on service secret | 15 min |
| 5 | C-19 | Whitelist template fields in notification service | Email template injection (SSTI) | 30 min |
| 6 | C-18 | Reject `javascript:`/`data:` URLs in registry DTO | Module Federation code injection | 15 min |
| 7 | C-10 | Validate tokens before setting `isAuthenticated: true` | Fake tokens accepted as valid | 30 min |
| 8 | C-05 | Validate session ownership in session service `oauthCallback()` | Session fixation / hijacking | 1 hour |
| 9 | C-11 | Add max size to event array in `performInitialSync()` | OOM on large calendars | 1 hour |
| 10 | C-12 | Fail job on partial blocker failure in `applyDeltaEvents()` | Silent data loss | 30 min |

**Fixes 1-2 are 5 minutes each and likely fix all Microsoft calendar sync.** Fixes 3-6 are under 30 minutes and close the highest-severity security holes. The full top 10 is ~5 hours of work total.

---

## Frontends — Clean

All three frontends (arm-core-fe, arm-app-calendar/frontend, arm-admin/frontend) passed audit with no critical, high, or medium issues:

- Auth state management properly avoids token persistence
- Module Federation origin validation implemented (`isTrustedOrigin()`)
- Route guards correctly gate unauthenticated access
- All API calls go through session-service proxy with `credentials: 'include'`
- No `dangerouslySetInnerHTML` or eval-like patterns
- OAuth CSRF nonce stored in sessionStorage before redirect
- SSE connections properly cleaned up on unmount with exponential backoff
- No sensitive data in localStorage/sessionStorage
- No known vulnerable dependency versions
