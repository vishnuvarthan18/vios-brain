# Failure-Mode Runbook — calendar-be

Verified against `apps/backend` source at commit `7c08935` (2026-08-03). Every claim below cites the file (and line, where useful) it was verified against. Where the code doesn't answer the question, this doc says **"behavior not yet verified"** instead of guessing — do not treat those lines as safe assumptions during an incident.

This is a description of **current, actual** behavior, not a design spec. Several of these gaps are already tracked in `TASK_SHEET.md` (PROD-003 etc.) — this doc records what happens *today*, before those land.

---

## 1. MongoDB down/unreachable

**Pool/timeout config** — `src/app.module.ts:58-73` (`MongooseModule.forRootAsync`):
```
maxPoolSize: 50, minPoolSize: 10,
connectTimeoutMS: 10_000, socketTimeoutMS: 45_000,
serverSelectionTimeoutMS: 10_000, heartbeatFrequencyMS: 10_000,
writeConcern: { w: 'majority', wtimeout: 5_000 }
```

**At boot, if Mongo is unreachable:** the `useFactory` doesn't set `retryAttempts`/`retryDelay`, so `@nestjs/mongoose`'s own defaults apply — `retryAttempts: 9`, `retryDelay: 3000` (verified in the installed package, `@nestjs/mongoose/dist/common/mongoose.utils.js`, `handleRetry(retryAttempts = 9, retryDelay = 3000, ...)`). Each attempt can take up to `serverSelectionTimeoutMS` (10s) before it's counted as failed, then the module waits `retryDelay` (3s) before the next attempt — worst case roughly 9 × 13s ≈ 117s before Nest gives up. When retries are exhausted, `mongoose-core.module.js`'s `catchError` rethrows, which rejects the `NestFactory.create(AppModule)` promise in `src/main.ts:16`. `src/main.ts:73` is `void bootstrap();` with **no `.catch()`**, and no `process.on('unhandledRejection', ...)` handler exists anywhere in `src/` (grep confirmed). Node's default behavior (no `--unhandled-rejections` override in `Dockerfile`) is to crash the process on an unhandled rejection. **Net effect: the container crash-loops until Mongo becomes reachable at boot** — there's no graceful "start degraded" path.

**Mid-flight, if Mongo drops after boot:** in-flight operations are bounded by `socketTimeoutMS` (45s) at the driver level, and a new server selection (e.g. after a replica-set failover) is bounded by `serverSelectionTimeoutMS` (10s) — so requests fail rather than hang forever, but can take up to 45s to do so. No custom `ExceptionFilter`/`APP_FILTER` exists in `src/` (grep confirmed), so an uncaught Mongoose error becomes NestJS's default unhandled-exception response: HTTP 500 with a generic `{"statusCode":500,"message":"Internal server error"}` body. There is no repo-wide catch-and-retry wrapper around Mongoose calls in the service layer.

**Health surface today:** `src/app.controller.ts` exposes only `GET /` (served at `GET /v1/` under the global prefix set in `src/main.ts:19`), backed by `src/app.service.ts`'s `getHello()` → returns the static string `'Hello World!'`. It never touches Mongo or Redis. Per `TASK_SHEET.md:55` and `:1151-1155` (PROD-003), this is confirmed **"⏳ Not started"** — the Docker healthcheck hits exactly this endpoint, so the container reports "healthy" through a full Mongo outage. **No readiness probe exists yet.**

**What an on-call engineer sees:** container restarts repeatedly at boot with no clear crash log beyond an unhandled rejection stack trace (Mongo connection error) in stdout; if Mongo drops mid-flight, scattered HTTP 500s with generic bodies across arbitrary endpoints, with no distinguishing "Mongo is down" signal in the response — only in the pino log line for that request.

**What to do:** no automated recovery beyond Mongo's own retry loop at boot (bounded, then crash) — requires manual intervention (restore Mongo, then the container's restart policy will succeed on the next attempt). No readiness probe exists to catch a mid-flight outage automatically.

---

## 2. Redis down/unreachable

**Client wiring** — `src/common/redis/redis.module.ts`: a single shared Redis client is obtained via `queue.client` from a dedicated no-op BullMQ queue (`redis-bridge`), exported as `REDIS_CLIENT`. This is the same underlying `ioredis` connection used by `IdempotencyInterceptor`, `GoogleClientFactory`'s token-refresh cache, and `SseRedisService` (which additionally `.duplicate()`s it for a dedicated pub/sub subscriber — `sse-redis.service.ts:28`).

**Reconnect behavior (ioredis/BullMQ defaults, verified in installed packages, not overridden anywhere in `src/`):**
- `BullModule.forRootAsync` (`app.module.ts:75-95`) passes only a bare `url`/sentinel config — no `maxRetriesPerRequest`, `retryStrategy`, or `enableReadyCheck` override anywhere in `src/`.
- The `redis-bridge` queue (`RedisModule`) is a plain `Queue`, not a `Worker`, so it is a **non-blocking** connection (`hasBlockingConnection` defaults `false` in `bullmq`'s `queue-base.js`). BullMQ only forces `maxRetriesPerRequest = null` for *blocking* connections (`redis-connection.js`); this one keeps ioredis's own default, `maxRetriesPerRequest: 20`. So commands issued through `REDIS_CLIENT` while Redis is down sit in ioredis's offline queue (`enableOfflineQueue: true` by default) and are retried on reconnect attempts, but the calling promise **rejects** once 20 retries are exhausted with ioredis's "max retries per request" error — it does not wait forever.
- Reconnect attempts themselves use BullMQ's overridden `retryStrategy`: `Math.max(Math.min(Math.exp(times), 20_000), 1_000)` (`bullmq/dist/cjs/classes/redis-connection.js`) — starts near 1s, grows to a 20s cap, retries indefinitely.
- BullMQ **Worker** processors (e.g. `SyncPollProcessor`, `WebhookRenewalProcessor`) each open their own separate *blocking* connection where `maxRetriesPerRequest` is forced to `null` — those connections' blocking calls simply stall until Redis comes back rather than erroring out.

**What breaks:**
- **Job enqueue** (`queue.add(...)` calls) goes through the non-blocking connection — will eventually reject (not hang forever) if Redis is down long enough to exhaust 20 retries.
- **`sync-poll.processor.ts`'s lock** (`sync-poll.processor.ts:40-47`, `client.set(lockKey, ..., 'NX')`): if this `set` rejects because Redis is down, the exception propagates out of `process()` uncaught — no try/catch wraps the lock-acquire call itself. BullMQ's own Worker-level job-failure handling then applies (job marked failed; retried only if the job/queue was configured with `attempts`/`backoff` — none of the `BullModule.registerQueue(...)` call sites in `src/` set `defaultJobOptions`, so per-job retry behavior depends on options passed at the individual `.add()` call site, not traced further here — **behavior not yet verified** past "the process() call throws").
- **`SseRedisService`**: both `publish()` (`sse-redis.service.ts:81-86`) and the subscribe-on-first-listener call (`:61`) wrap their Redis calls in `.catch()` that only logs — a down Redis means SSE updates silently stop delivering, but doesn't throw back to the caller that triggered the publish (e.g. a sync completing). The initial `subscribe()` observable itself doesn't reject; a client just never receives the message.
- **`IdempotencyInterceptor`** (`idempotency.interceptor.ts`) — **fails closed on read, fails open on write**:
  - The cache-read, `await this.redis.get(redisKey)` at line 77, has **no try/catch** around it. If Redis is down and this rejects, the rejection propagates out of `intercept()` uncaught, which NestJS surfaces as an unhandled exception → HTTP 500 for that request. **Any mutating request that includes an `Idempotency-Key` header will fail outright while Redis is down**, even though the underlying handler (e.g. account-connect, sync-start) might otherwise have succeeded.
  - The cache-write, `this.redis.set(...)` at lines 93-100, **is** wrapped in `.catch()` that only logs — a failed cache write after a successful request doesn't affect the response the client sees; it just means that write isn't idempotency-protected.
- **`GoogleClientFactory.refreshAccountTokenIfExpired`** (`google-client.factory.ts:95-117`, the token-refresh pre-check used by `WebhookRenewalProcessor`): `await this.redis.get(cacheKey)` at line 97 similarly has no try/catch. If Redis is down, this call throws inside `webhook-renewal.processor.ts:84`'s `process()` (also unguarded there), which fails that BullMQ job. Note this is a **different** code path from the per-Google-API-call token check (`getGoogleToken` → `refreshTokenIfExpiredForAccount`), which does not touch Redis at all — so a live sync's own token refresh is unaffected by Redis being down; only the webhook-renewal cron's pre-check job is.

**What an on-call engineer sees:** HTTP 500s specifically on requests carrying `Idempotency-Key` (account-connect, sync-start); webhook-renewal jobs failing in BullMQ's failed-job set with a Redis connection error; SSE-driven UI updates going stale with no client-visible error; `[Poll] Skipped ...` warnings absent (the lock-acquire itself throws before that log line, so poll jobs error out instead of logging their usual skip message) — check the Nest/pino logs for ioredis "max retries per request" or connection-refused errors as the root-cause signature.

**What to do:** no automated recovery beyond ioredis's own reconnect loop (bounded per-command, unbounded for the connection itself) — requires manual intervention to restore Redis. No fallback/degraded mode exists for the idempotency read path or the webhook-renewal Redis pre-check.

---

## 3. core-be (arm-core-be) down/slow

`CoreAuthService` (`src/common/core-auth/core-auth.service.ts:14-19`) wraps every call in a `CircuitBreaker`:
```
failureThreshold: 5, successThreshold: 2, timeout: 30_000, name: 'core-be'
```
This is a process-wide singleton (default NestJS provider scope) — its state is shared across all users' requests, not per-account.

**What a caller sees when the breaker trips:** `getGoogleToken`, `getGoogleStatus`, and `revokeGoogleToken` (`core-auth.service.ts:28-158`) each wrap their body in `try { return await this.breaker.execute(...) } catch (error) { ...; throw new HttpException(..., HttpStatus.SERVICE_UNAVAILABLE) }`. When the breaker is `OPEN`, `CircuitBreaker.execute` (`circuit-breaker.ts:37-49`) throws `Error("Circuit breaker is OPEN — core-be is unavailable. Retry after Xs")` without attempting the call; this is caught by the same catch block and rethrown as `HttpException('Failed to retrieve Google authentication token' / 'status' / 'authentication token', HttpStatus.SERVICE_UNAVAILABLE)` — **HTTP 503** to the caller in all three cases. Before the breaker trips (< 5 consecutive failures), a core-be timeout/error surfaces the same way (503), just per-request rather than short-circuited.

**What an on-call engineer sees:** `this.logger.error('Failed to get Google token for user ${userId}:', error)` (and the status/revoke equivalents) in the pino logs — the logged `error` is either the raw fetch/network error (pre-trip) or the breaker's own "Circuit breaker is OPEN — core-be is unavailable..." message (post-trip, cheap and fast — no network call attempted). Client-facing: HTTP 503 from any endpoint that depends on `CoreAuthService`.

**What to do:** the breaker self-heals — after `timeout: 30_000` (30s) it moves to `HALF_OPEN` and probes; 2 consecutive successes (`successThreshold: 2`) close it again. No manual intervention needed for the breaker itself; if core-be stays down, 503s continue until core-be recovers.

---

## 4. Google OAuth token-refresh endpoint down/slow

`GoogleClientFactory` (`src/google/google-client.factory.ts:28-33`) wraps the actual `(secret removed)()` call in its own breaker:
```
failureThreshold: 5, successThreshold: 2, timeout: 30_000, name: 'google-token-refresh'
```
also a process-wide singleton — one bad patch of Google token-refresh failures trips it for every account, not just the one that triggered it.

**Briefly down (breaker still CLOSED):** `refreshTokenIfExpiredForAccount`'s catch block (`google-client.factory.ts:194-209`) inspects the error's status. A 400/401 (refresh token itself invalid/revoked) sets `doc.isAuthenticated = false` and returns. Anything else — including a network failure or the breaker's own "OPEN" error once tripped — falls into the `else` branch: **"Transient error ... log and continue with the stale token"** (comment and behavior both confirmed at line 204-208); the account stays `isAuthenticated: true` with its old access token, and `getGoogleToken` (`:69-90`) returns `hasToken: true` with that stale token. The request is **not** failed here.

**Downstream consequence (code-verified, not speculative):** because the stale token is handed back rather than the request being blocked, the actual Google Calendar API call made with it (in `google-calendar-api.service.ts`) will get a `401` from Google. Every method there that checks `isAuthError(error)` (`code === 401`) then calls `markAccountUnauthenticated(accountId)` (e.g. `listEvents`, `createEvent`, `updateEvent`, `listCalendarsByAccountId`, `listEventsIncremental`, `listBlockerEvents` — see their respective catch blocks in `google-calendar-api.service.ts`), which sets `isAuthenticated: false` in Mongo. **So a sustained-enough Google token-refresh outage can cause accounts to be marked unauthenticated even though their refresh token is fine** — the trigger is the outage, not a real credential problem. This is inferred by tracing the two files together, not a single comment stating it.

**Definitively down long enough to trip the breaker:** once `OPEN`, `refreshAccessToken()` isn't even attempted — `breaker.execute` throws immediately, which (per the previous paragraph) also lands in the "transient error, continue with stale token" branch. So the observable behavior doesn't change qualitatively between "occasionally failing" and "breaker OPEN" from a single sync's perspective — both proceed with a stale token and let the downstream Calendar API call fail. The breaker's only effect here is avoiding repeated slow network calls to Google's token endpoint while OPEN (fails fast internally), not changing what the caller/sync sees.

**What an on-call engineer sees:** `[TokenRefresh] Refresh attempt failed for account ${accountId} (proceeding with stale token): ...` warnings/errors in logs (`google-client.factory.ts:206-207`) during the outage; a wave of accounts subsequently flipping to `isAuthenticated: false` with `[Auth] Account ${accountId} marked unauthenticated after 401 from Google` (`google-calendar-api.service.ts:67-69`) shortly after, even though nothing is wrong with those accounts' actual Google grants.

**What to do:** the breaker self-heals the same way as core-be's (30s timeout → half-open → 2 successes to close). There is no automated re-authentication recovery for accounts that got marked `isAuthenticated: false` this way — that requires the user to reconnect, i.e. manual/user-driven intervention.

---

## 5. Google Calendar API (events.list/get/insert/update/delete/watch) slow/down

`getCalendarClient` (`google-client.factory.ts:42-67`) constructs the `googleapis` client with `timeout: 15_000` (line 65) — a 15s client-side timeout inherited by **every** call made through that client instance, since `GoogleCalendarApiService` (`calendar/google-calendar-api.service.ts`) never sets its own per-call timeout.

**Confirmed: no circuit breaker on any of these calls.** Every method in `google-calendar-api.service.ts` (`listCalendarsByAccountId`, `listEvents`, `getEvent`, `createEvent`, `updateEvent`, `deleteEvent`, `watchEvents`, `stopChannel`, `listEventsIncremental`, `listBlockerEvents`) wraps the call in a plain `try/catch` that classifies the error (404 → `NotFoundException`, 401 → mark unauthenticated, 429/quota → `(secret removed)(429)`, 412 on update → one etag-refresh retry, everything else → `BadRequestException`) — none of them go through `CircuitBreaker`. This matches the brief's framing exactly: `CoreAuthService` and `GoogleClientFactory`'s token-refresh both got breaker coverage; the actual Calendar API surface did not.

**This is a known, currently-unsolved gap, not a mitigated one.** During a sustained Google Calendar API outage, every sync/webhook/poll job that calls into `GoogleCalendarApiService` will individually wait up to 15s, time out, and get mapped to a generic `BadRequestException` (or one of the specific mappings above if the error carries a recognizable code) — there is no fast-fail path. Under load (many concurrent syncs), this means 15s of held concurrency per call rather than an immediate rejection, until `google-calendar-api.service.ts` gets its own breaker in a future task.

**What an on-call engineer sees:** a spike in `Failed to list events: ...` / `Failed to create event: ...` etc. log lines (each method's catch block logs via the thrown exception's message, not a dedicated logger call in most cases — `listCalendarsByAccountId`, `createEvent`, etc. rely on the exception message alone unless another catch path logs explicitly), all trending toward ~15s request latency before failing; HTTP 400 (`BadRequestException`) to the caller in the generic case, 429 for quota, 404 for missing events, 403 (`ForbiddenException`) for `watchEvents` access-denied.

**What to do:** no automated recovery — requires manual intervention (or waiting out the Google-side outage). No breaker, no automatic backoff/retry visible in this file beyond the single etag-conflict retry in `updateEvent`, which is unrelated to outage handling.

---

## 6. Microsoft Graph API slow/down

**There is currently no Microsoft calendar-sync integration at all** — this is the most important thing to know before assuming symmetry with Google. `src/providers/calendar-provider.registry.ts:52-59` (`forProviderName`) only has a `case` for `CALENDAR_PROVIDER_NAME.GOOGLE`; any other provider name (including `'microsoft'`) falls through to `default: throw new Error('Unsupported calendar provider: ${providerName}')`. The `src/microsoft/` directory contains only `microsoft.service.ts` (generates the Microsoft OAuth "Add Account" URL) and `microsoft-oauth.service.ts` (exchanges the OAuth code and fetches the user's profile) — there is no Microsoft equivalent of `google-calendar-api.service.ts`; no calendar-sync-time Graph API calls (events.list/insert/update/delete/subscribe) exist in this codebase today.

**The only live Microsoft Graph traffic is the account-connect flow** (`microsoft-oauth.service.ts:44-79`): two plain `fetch()` calls — token exchange (`https://login.microsoftonline.com/.../oauth2/v2.0/token`) and profile lookup (`https://graph.microsoft.com/v1.0/me`). **Neither has a timeout or a circuit breaker** — no `AbortController`/timeout option is passed to either `fetch()` call, and no `CircuitBreaker` wraps them (confirmed by reading the full file — this is not symmetric with Google's `google-client.factory.ts`, which does have both). If Microsoft's login/Graph endpoints hang, the request hangs for however long Node's default `fetch` behaves (no explicit cap in this code) — **this is a gap, not a documented/mitigated behavior**, and unlike the Google case, it affects both the OAuth token exchange and the profile lookup with no fallback.

**What an on-call engineer sees:** `Microsoft token exchange failed: ${status} ${body}` or `Failed to fetch Microsoft profile: ${status} ${body}` logs (`microsoft-oauth.service.ts:65-66`, `82-83`) on a hard failure with a response; on a hang, no clear signal until whatever upstream timeout (BFF proxy, load balancer) eventually cuts the connection — **behavior not yet verified** past "no application-level timeout exists."

**What to do:** no automated recovery, no breaker, no timeout — requires manual intervention if Microsoft's endpoints are degraded. Since there's no calendar-sync path for Microsoft yet, the blast radius today is limited to users attempting to connect a Microsoft account, not any ongoing sync.

---

## 7. Notification service (arm-service-notification) down

**Correction to a likely assumption:** `NOTIFICATION_WEBHOOK_URL` (`src/common/config/env.ts:60-63`) is **not** an outbound client call to `arm-service-notification`. Grepping all of `src/` for any reference to `arm-service-notification`, `NotificationService`, or a notification-service HTTP client turns up nothing. `NOTIFICATION_WEBHOOK_URL` is calendar-be's own **inbound** webhook receiver address — the URL calendar-be tells *Google* to POST push notifications to, passed as the `address` argument to `watchEvents`/`createSubscription`:
- `events/sync-orchestrator.service.ts:465` and `:533` (source-calendar and target-calendar webhook registration)
- `webhook/webhook-renewal.processor.ts:98` (channel renewal)

which both eventually flow into `google-calendar-api.service.ts:371-400`'s `watchEvents`, where `address` becomes `requestBody.address` in the Google `channels.watch` call. The variable's default value, `http://localhost:4001/events/notifications` (`env.ts:62`), even points at calendar-be's **own** default port (`PORT` also defaults to `4001` — `env.ts:37`) — confirming it's self-referential, not a pointer to another service.

**Conclusion: there is no code path in this repo where calendar-be calls out to `arm-service-notification`.** This dependency-failure section, as scoped in the brief, does not currently apply to calendar-be — if `arm-service-notification` goes down, it has no observable effect on calendar-be's own request/sync behavior, because calendar-be never talks to it. (It's plausible some other ARM service calls `arm-service-notification`, but that's outside `apps/backend`'s code and outside this task's scope to verify.)

**What an on-call engineer sees:** nothing calendar-be-specific — if you're paged for "arm-service-notification down" and calendar-be syncs/webhooks are behaving normally, that's expected; calendar-be has no dependency on it to break.

**What to do:** not applicable — no automated or manual recovery needed on the calendar-be side for this dependency, because it isn't one.

---

## Summary table

| Dependency | Breaker? | Timeout? | Fails open or closed? | Health probe today? |
|---|---|---|---|---|
| MongoDB | No | Driver-level (`serverSelectionTimeoutMS`/`socketTimeoutMS`) | Closed (boot crash-loops; mid-flight 500s) | No (PROD-003 not started) |
| Redis (general) | No | ioredis default (`maxRetriesPerRequest: 20` on non-blocking conn) | Mixed — SSE publish/subscribe fail open (logged, swallowed); idempotency-key read fails closed (500); idempotency-key write fails open | No |
| core-be | Yes (`'core-be'`) | N/A (breaker + underlying fetch) | Closed (503) | N/A |
| Google token-refresh | Yes (`'google-token-refresh'`) | N/A | Open — proceeds with stale token either way | N/A |
| Google Calendar API | **No** (known gap) | 15s (inherited from client construction) | Closed (per-call error after 15s) | N/A |
| Microsoft Graph (OAuth/profile only — no sync path exists) | **No** | **No** | Closed, but unbounded wait first | N/A |
| arm-service-notification | N/A | N/A | N/A — no such call exists in this codebase | N/A |
