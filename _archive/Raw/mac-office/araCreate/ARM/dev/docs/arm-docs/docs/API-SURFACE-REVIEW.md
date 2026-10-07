# ARM Platform — API Surface Review & Remediation List

**Date:** 2026-08-27
**Scope:** Every HTTP endpoint across core-be, arm-session, calendar-be, admin-be, notification — plus the Kong and Caddy routes that expose them
**Method:** Read from controllers, guards, `app.module.ts` provider lists, `kong.yml.tmpl`, `Caddyfile.prod`, and caller-side greps across all three frontends. No runtime probing.
**Status:** Report only — no code changes made

---

## How to read this

Each item is `API-nn` with **Risk / Current / Fix / Verify**. The `Verify` line is the success criterion — an item is not done until that check passes, not when the edit is made.

Items are marked:
- **LIVE** — exploitable or wrong today, in the server environment
- **LATENT** — correct today only because of an unrelated condition (a flag that is unset, a module import order); breaks the day that condition changes
- **DEBT** — no defect, but a cost that compounds

Two decisions gate roughly half the list. Settle those first — several items disappear entirely depending on the answer.

---

## 0. Blocking decisions

### DECISION-1: Is `/api/*` (Caddy → Kong) a product, or leftover?

**Findings that led here:**
- No frontend references `/api/core`, `/api/calendar`, or `/api/admin`. The browser only ever calls `/session/*`.
- East-west is direct, not via Kong: `arm-session` calls `CORE_BE_URL` for OTP request, OTP verify, signup complete, logout, and refresh (`session.controller.ts:85,121,166,241`, `proxy.service.ts:293`).
- Kong's own config concedes its rate-limit bucket is "a platform-wide ceiling, not per-tenant" because there is a single `arm-platform` JWT consumer (`kong.yml.tmpl:93-99`).

**If leftover:** remove `handle /api/*` from `Caddyfile.prod:35` except the provider-callback and webhook paths. This closes API-01, API-02 and API-03 in one change, and takes Kong out of the browser path entirely.

**If product:** it needs one Kong consumer per tenant (which needs per-tenant JWT claims from core-be), and every public route needs an inbound `X-User-Id` clear before anything else on this list matters.

**Until decided:** treat API-01 through API-05 as the minimum patch that makes the current shape safe.

### DECISION-2: Keep the `X-User-Id` header-trust path, or delete it?

**Current:** `TRUST_PROXY_HEADERS` appears in exactly one `.env.example` and one Joi default. It is set to `true` in **no** compose file, descriptor, or environment. So the header-trust branch in all three guards (`KongJwtGuard` in core-be and admin-be, `AuthGuard` in calendar-be) plus the throttler key derivation (`app-throttler.guard.ts:14-20`) is dead code in both environments.

**Delete it:** three guards get simpler, the header-trust bypass documented in `AUTH-ARCHITECTURE.md` §3 cannot return, one flag fewer to get wrong.

**Keep it:** Kong must clear inbound `X-User-Id` on **every** route including the public ones (it currently overwrites the header only on the three JWT-protected routes), and API-03 becomes urgent rather than latent.

---

## 1. Critical — live architectural violation

### API-01: core-be returns JWTs to the browser through Kong — **LIVE**
- **Risk:** The single rule the session service exists to enforce ("tokens never reach the browser") is bypassable from any browser on the server.
- **Current:** `POST /v1/auth/login/email/verify` and `POST /v1/auth/signup/complete` return `{ userId, accessToken, refreshToken }` in the response body (`core-be auth.controller.ts:113,188`). Kong's `core-auth-public-route` publishes the prefix `/api/core/v1/auth/login` with **no JWT plugin**, path-rewritten to core-be (`kong.yml.tmpl:46-57`, `:105-117`), and Caddy sends `/api/*` to Kong (`(secret removed):35`). `POST /api/core/v1/auth/refresh` is worse — it is also public, accepts the `refresh_token` cookie, and returns the rotated refresh token in the body, so any script on that origin can convert an httpOnly cookie into a readable credential.
- **Fix:** Remove `/api/core/v1/auth/login` and `/api/core/v1/auth/refresh` from the public route's path list. Neither has a legitimate caller — the session service reaches those handlers east-west. Separately, stop returning `refreshToken` in the body on any path the browser can reach; the session service reads it server-to-server and could take it from a header or a dedicated internal route.
- **Verify:** `curl -X POST https://<host>/api/core/v1/auth/login/email/request-otp` returns 404 from the edge. Login through the UI still works end to end.

### API-02: Internal token endpoints are reachable from the internet — **LIVE**
- **Risk:** Endpoints that hand out provider OAuth tokens are exposed publicly, defended only by a shared secret that lives in five services' `.env`.
- **Current:** `GET /v1/auth/google/token/:userId` and `/v1/auth/microsoft/token/:userId` (`core-be auth.controller.ts:358,528`) are `@Public()` + `SessionSecretGuard`. They sit under the public Kong prefix `/api/core/v1/auth/google`, and Kong never strips a client-supplied `x-session-secret`. The session proxy does strip that header (`proxy.service.ts:48-66`) — Kong does not.
- **Fix:** Narrow the public prefixes to the two exact provider callback paths (see API-04), and move these two handlers under an `/v1/internal/` prefix the edge never routes.
- **Verify:** `curl https://<host>/api/core/v1/auth/google/token/<uuid>` returns 404 at the edge, not 401 from the app.

### API-03: Public Kong routes accept a client-supplied `X-User-Id` — **LATENT**
- **Risk:** Full user impersonation on any endpoint reachable through a public Kong route.
- **Current:** The Lua `pre-function` that overwrites `X-User-Id` from the verified JWT is bound only to `core-api-route`, `calendar-api-route` and `admin-api-route`. On `core-auth-public-route` and `calendar-auth-public-route` (which covers the `AuthGuard`-protected `add-account` endpoints) a client-supplied header passes through untouched. Not exploitable today only because `TRUST_PROXY_HEADERS` is unset everywhere. The same header feeds the rate-limit key (`app-throttler.guard.ts:14-20`), so a spoofed value would also mint a fresh OTP bucket per request. The comment at `kong.yml.tmpl:50-53` claims the authenticated sub-paths "are still protected by NestJS KongJwtGuard (Passport JWT fallback)" — true only while the flag is off, which is the opposite of that flag's purpose.
- **Fix:** Per DECISION-2 — either delete the header-trust branch from all three guards, or add an inbound `X-User-Id` clear to every Kong route. Correct the misleading comment either way.
- **Verify:** With `TRUST_PROXY_HEADERS=true` set deliberately in a scratch environment, a request carrying `X-User-Id: <victim>` to a public route is rejected.

---

## 2. Edge configuration

### API-04: Narrow the public Kong path list to provider callbacks only
- **Risk:** Prefix-based public routes silently expose every sibling path added under them later.
- **Current:** `core-auth-public-route` publishes four prefixes; `/api/core/v1/auth/google` alone also exposes `/token/:userId`, `/status/:userId` and `/revoke`.
- **Fix:** Replace the prefixes with the exact callback paths. **Assumption to confirm before doing this:** that `GOOGLE_CORE_CALLBACK_URL` and `MICROSOFT_CORE_CALLBACK_URL` on the server point at `https://<host>/api/core/v1/auth/{google,microsoft}/callback`. Both are blank in `.env.example:191-197`, so this was not verifiable from the repo. If they point elsewhere, these prefixes can be removed outright.
- **Verify:** A full Google login and a full Microsoft login complete against the server after the change.

### API-05: Delete `calendar-auth-public-route`
- **Risk:** It is the route carrying API-03's latent hole, and appears to serve nothing.
- **Current:** `kong.yml.tmpl:186-190` publishes `/api/calendar/v1/auth` with no JWT plugin. Nothing at that prefix is a provider redirect target: the four `add-account` routes are `AuthGuard`-protected and called through the session proxy, and `/v1/auth/google/url` is dead (API-13).
- **Fix:** Remove the route and its two plugins.
- **Verify:** Add-account for Google and Microsoft still completes through the UI.

### API-06: Keep exactly three public edge routes
- **Current:** Four public routes exist across two services.
- **Fix:** The only paths a third party must reach are `calendar-reconnect-callback-route`, `calendar-webhook-route`, and whichever core OAuth callback paths API-04 confirms. Everything else at the edge should require auth or not be routed.
- **Verify:** Enumerate Kong routes after the change; every entry without a `jwt` plugin is on that list of three.

### API-07: Adopt a path convention for internal endpoints — **DEBT**
- **Risk:** Internal endpoints protected only by "the edge happens not to route this path" break when a prefix widens, which is exactly how API-02 happened.
- **Current:** calendar-be does this correctly — `/v1/internal/user/:userId` and `/v1/webhook/renew-all` are unroutable from the edge *and* secret-guarded. core-be's token endpoints sit under `/v1/auth/*`, mixed in with public ones.
- **Fix:** Standardize on an `/v1/internal/` prefix for every service-to-service route, guarded by the secret, never routed at the edge. Record it in `CODING-STANDARDS.md`.
- **Verify:** Every `SessionSecretGuard` / `InternalSecretGuard` route in the platform lives under that prefix.

---

## 3. Code correctness

### API-08: calendar-be reports errors with HTTP 200/201 — **LIVE**
- **Risk:** Any client branching on `res.ok` or `res.status` sees success for every failure. Corrupts retry logic, the session proxy's error handling, and every Kong and Prometheus error-rate metric.
- **Current:** Every `catch` in `account.controller.ts` returns `{ statusCode: 400, success: false }` in the body while the HTTP status stays 200 (201 for POST). Same pattern for `HttpStatus.NO_CONTENT` on empty lists (`:60`), `HttpStatus.GONE` on a missing account (`:216`), and `calendar-management.controller.ts:90`. core-be and admin-be throw properly, so the platform is inconsistent about this too.
- **Fix:** Throw the corresponding Nest exception; let the exception filter set the status.
- **Verify:** e2e assertions on `res.status` — not `res.body.statusCode` — for each failure path in `account.e2e-spec.ts`.

### API-09: Webhook ingress fails open — **LIVE**
- **Risk:** Unbounded queue growth from an unauthenticated caller during a database blip.
- **Current:** `events.controller.ts:278-289` sets `known = true` and enqueues when the channel lookup throws. `POST /v1/events/notifications` is the only unauthenticated state-changing endpoint on the platform and is `@SkipThrottle()`. On the server Kong caps it at 30/min; locally there is no Kong, so no limit at all. (The `channelToken` / `clientState` verification in the worker is timing-safe and correct — the gap is only this ingress write.)
- **Fix:** Return 503 on lookup failure so providers retry, or apply a bounded throttle to this route specifically.
- **Verify:** With the Mongo container stopped, a POST to the webhook path does not add a job to the queue.

### API-10: Session service checks its internal secret in the handler, not a guard — **DEBT**
- **Risk:** Invisible to any guard-level audit, and the next `internal/*` route added there is unprotected by default.
- **Current:** `session.controller.ts:628,655` call `this.validateInternalSecret(req)` inside the handler body.
- **Fix:** Extract to a guard, matching calendar-be's `InternalSecretGuard`.
- **Verify:** Grep for `@UseGuards` covers every `internal/` route in the platform.

### API-11: `GET /session/oauth/callback` still accepts a raw `sid` query param — **LIVE**
- **Risk:** A session ID in a URL lands in access logs, browser history and `Referer` headers.
- **Current:** `session.controller.ts:578-596` keeps `sid` as a fallback to the single-use handoff token. The code says "will be removed"; it has not been.
- **Fix:** Remove the fallback. The handoff token replaced it and is single-use with a 30s TTL.
- **Verify:** Google login and email login both complete; no `sid` in any redirect URL.

### API-12: The `trust proxy = 1` assumption holds on one path only — **LIVE (low impact)**
- **Risk:** Rate-limit buckets keyed on the wrong address.
- **Current:** `core-be main.ts:57-70` reasons explicitly about "that one proxy" — the session service, which writes a single `x-forwarded-for` entry, so `req.ip` is the client. Via Caddy → Kong there are two hops, so the header carries two entries and `req.ip` resolves to Caddy's address; all public-auth callers then share one 5/min bucket. Low impact today because the browser uses the session path.
- **Fix:** Moot if DECISION-1 closes `/api/*`. Otherwise set the hop count per environment, or key the OTP throttle on the email address rather than the IP.
- **Verify:** Two clients on different IPs hitting the public auth path get independent buckets.

---

## 4. Dead surface to delete

All four caller-checks below were done by grepping the three frontend repos, the session service, and the e2e specs. All returned zero callers.

### API-13: `PUT /v1/account/:email/tokens` — **LIVE**, highest-value deletion
- **Risk:** A live write path for provider credentials, reachable from the browser via `/session/proxy/calendar/v1/...`. `TokenDataDto` requires `accessToken` and accepts `refreshToken` (`account-event.dto.ts:32`). Contradicts the same rule as API-01.
- **Current:** `account.controller.ts:254`. No caller anywhere.
- **Fix:** Delete the route, the DTO, and `updateAccountTokens` if nothing else calls it.
- **Verify:** Add-account, reconnect, and sync all still work.

### API-14: `GET /v1/account/sync-from-core`
- **Risk:** A GET that mutates — it creates or updates an account (`account.controller.ts:182`). Unsafe under prefetch, caching and link-following.
- **Fix:** Delete. If the behaviour is ever needed, it is a POST.

### API-15: `GET /v1/auth/google/url`
- **Risk:** Unguarded route returning a hardcoded core-be URL (`calendar-be auth.controller.ts:76`). Also the only reason `calendar-auth-public-route` looks load-bearing.
- **Fix:** Delete.

### API-16: `GET /v1/account/accounts`
- **Risk:** Duplicate read — same `findAllByUser` call and paging as `GET /v1/account`, differing only in response envelope (`account.controller.ts:60` vs `:99`).
- **Fix:** Delete the duplicate.

### API-17: `GET /v1/:accountId` — root-level wildcard — **LATENT**
- **Risk:** A parameterised route at the root of the global prefix. It shadows nothing today *only* because `CalendarManagementModule` is imported before `HealthModule` (`app.module.ts:120` vs `:123`) and the health routes happen to be two segments deep. Any future single-segment `GET /v1/x` registered after it resolves to a calendar lookup instead.
- **Current:** `calendar-management.controller.ts:79`, under `@Controller()`, self-marked deprecated with "Frontend has fully migrated — safe to remove once confirmed no external callers remain".
- **Fix:** Delete. Route correctness must not depend on module import order.
- **Verify:** `GET /v1/health/live` and `/ready` still respond; no route in the calendar app is registered at the bare prefix root.

### API-18: `projects: 'PROJECTS_BE_URL'` in `SERVICE_ENV_MAP` — **DEBT**
- **Risk:** A reserved proxy target for a service that does not exist in `repos.conf`.
- **Current:** `proxy.service.ts:88`.
- **Fix:** Remove; add it when projects-be exists.

### API-19: `GenericController` — single-use abstraction — **DEBT**
- **Risk:** Three of its five inherited routes exist only to `throw ForbiddenException` — `GET /v1/users`, `POST /v1/users`, `DELETE /v1/users/:id` (`users.controller.ts:70-85`). A generic base was built, then most of it disabled. Its generics also erased the DTO type once and let unvalidated bodies reach TypeORM (recorded in the comment at `users.controller.ts:54-60`).
- **Current:** Exactly one consumer — `UsersController`.
- **Fix:** Inline the two real handlers; drop the three refusing routes from the surface rather than publishing routes that can only 403. Keep the `ParseUUIDPipe` and the explicit DTO.
- **Verify:** `GET /v1/users` returns 404, not 403. Self-read and self-update still work; another user's id still refused.

### API-20: Redundant `:userId` path params — **DEBT**
- **Risk:** An IDOR surface that has to be defended instead of not existing. Both handlers throw unless the param equals `caller.id` (`core-be auth.controller.ts:396,568`).
- **Fix:** Read identity from the token; drop the param.

---

## 5. Standardize (cross-repo, do once)

### API-21: One response envelope
calendar-be wraps everything in `{ statusCode, success, ... }`; core-be and admin-be return bare entities and throw. Two conventions behind one proxy. Pick one, write it into `CODING-STANDARDS.md`, and it becomes a review check instead of a per-PR argument. Depends on API-08 landing first.

### API-22: One health contract
Five services, four shapes: core-be `/v1/health` + `/live`; calendar-be `/live` + `/ready` (no aggregate); admin-be `/v1/health` only (no live/ready); notification hardcodes the prefix into the controller path (`@Controller('v1/health')`) instead of using `setGlobalPrefix`; session `/session/health`. Standardize on `live` + `ready` + aggregate.

Note: the generated compose healthchecks are all internally consistent with what each service actually serves — including notification's `localhost:4001`, which is correct because its `PORT` defaults to 4001 inside its own container. This is a consistency fix, not a live break.

### API-23: State where rate limiting lives
Three layers, three different keys: Kong per-route (one platform-wide bucket), Nest per-IP (core-be, admin-be, session), per-user (calendar-be). Two endpoints opt out entirely (`SkipThrottle` on the webhook and the sync-status poll). Decide the authority per environment and record it — local has no Kong at all, so anything that relies on the edge for limiting is unlimited in development.

### API-24: Correct two stale claims
- The repo-layout line in the root `CLAUDE.md` calls notification "email via BullMQ + SMTP"; its controllers are Kafka `@EventPattern` consumers (`app.controller.ts:28,52`). The table further down in the same file says Kafka.
- `kong.yml.tmpl:50-53` — see API-03.

---

## 6. Verified correct — do not change

Stated explicitly so a future pass does not "fix" these:

- **Per-user scoping is consistent.** Every user-scoped handler passes `user.sub` down rather than trusting a path param: `CalendarController` (all seven routes), `AccountController`, `CalendarManagementController`, and both core-be `status` endpoints.
- **The session proxy's header strip list** (`proxy.service.ts:48-66`) removes all seven trust headers. This is what keeps internal routes unreachable through the proxy, and why the proxy is safe while Kong is not.
- **admin-be applies `KongJwtGuard + AdminRoleGuard` at the controller level** on all three controllers, so no admin route can be added unprotected by omission.
- **Webhook `channelToken` / `clientState` comparison is timing-safe** (`webhook-processor.service.ts:144,403`).
- **`UsersController`'s `ParseUUIDPipe` and explicit DTO re-declaration** fix a real past bug — keep them when unwinding API-19.

---

## 7. Suggested order

1. **DECISION-1** — it determines whether API-01 through API-05 are five edits or one.
2. **API-01, API-02, API-04, API-05** — edge config only, no application code, closes the critical findings.
3. **API-13 through API-18** — deletions. Shrinking the surface before standardizing it means less to standardize.
4. **API-08** — widest blast radius of the code bugs, and API-21 depends on it.
5. **DECISION-2**, then API-03.
6. **API-09, API-10, API-11** — remaining correctness items.
7. **API-19 through API-24** — debt and consistency, at whatever pace suits.

---

## Appendix: endpoint inventory

Auth column: **none** = no guard; **secret** = `X-Session-Secret`; **jwt** = access token or trusted header; **session** = `SESSION_ID` cookie; **admin** = jwt + admin role. All four Nest backends use global prefix `v1`.

### core-be (`:4000`) — global `GlobalAuthGuard` + throttler

| Method | Path | Auth |
|---|---|---|
| POST | `/v1/auth/login/email/request-otp` | none (5/min) |
| POST | `/v1/auth/login/email/verify` | none (5/min) |
| POST | `/v1/auth/signup/complete` | none (5/min) |
| POST | `/v1/auth/refresh` | none (refresh token is the credential) |
| GET | `/v1/auth/google` · `/v1/auth/google/callback` | none (Passport) |
| GET | `/v1/auth/microsoft` · `/v1/auth/microsoft/callback` | none (Passport) |
| GET | `/v1/health` · `/v1/health/live` | none |
| GET | `/v1/metrics` | none — private-IP allowlist only |
| GET | `/v1/auth/google/token/:userId` · `/v1/auth/microsoft/token/:userId` | secret |
| POST | `/v1/registry/register` | secret |
| GET | `/v1/auth/me` | jwt |
| POST | `/v1/auth/logout` | jwt |
| POST | `/v1/auth/google/revoke` · `/v1/auth/microsoft/revoke` | jwt |
| GET | `/v1/auth/google/status/:userId` · `/v1/auth/microsoft/status/:userId` | jwt (self only) |
| GET | `/v1/registry` · `/v1/registry/type/:type` · `/v1/registry/:name` | jwt |
| GET/PUT | `/v1/users/:id` | jwt (self only) |
| GET/POST/DELETE | `/v1/users` · `/v1/users` · `/v1/users/:id` | jwt — hard-disabled, always 403 |
| POST | `/v1/media/upload/avatar` · `/upload/org-logo` · `/upload` | jwt (2/1/10 MB) |
| DELETE | `/v1/media/avatar` | jwt |

### arm-session (`:5001`) — throttler only, no global auth guard

| Method | Path | Auth |
|---|---|---|
| GET | `/session/health` | none |
| POST | `/session/login/email/request-otp` · `/login/email/verify` · `/signup/complete` | none (5/min each) |
| GET | `/session/oauth/callback` | none — single-use handoff token, or legacy `sid` |
| POST | `/session/activate` | none — single-use handoff token, 30s TTL |
| POST | `/session/internal/create-session` · `/internal/revoke-user-sessions` | secret (checked in handler, not a guard) |
| POST | `/session/logout` | session |
| GET | `/session/me` | session |
| GET | `/session/calendar/add-account` · `/calendar/oauth/callback` | session |
| GET | `/session/calendar/add-account/microsoft` · `/calendar/oauth/callback/microsoft` | session |
| ALL | `/session/proxy/*path` | session (300/min) |
| GET | `/metrics` | none — private-IP allowlist only |

### calendar-be (`:4001`) — per-route `AuthGuard`, global per-user throttler

| Method | Path | Auth |
|---|---|---|
| GET | `/v1/` | none |
| GET | `/v1/health/live` · `/v1/health/ready` | none |
| GET | `/v1/auth/google/url` | none |
| GET | `/v1/google/update/account/callback` | none — `state` is the credential |
| GET | `/v1/events/notifications` | none — validation echo, `SkipThrottle` |
| POST | `/v1/events/notifications` | none — channel lookup + worker-side token check, `SkipThrottle` |
| DELETE | `/v1/internal/user/:userId` | secret |
| POST | `/v1/webhook/renew-all` | secret |
| GET | `/v1/auth/google/add-account` · `/v1/auth/microsoft/add-account` | jwt |
| POST | `/v1/auth/google/add-account/callback` · `/v1/auth/microsoft/add-account/callback` | jwt |
| POST/GET | `/v1/account` | jwt |
| GET | `/v1/account/accounts` · `/available` · `/sync-from-core` · `/:accountId/sync-status` | jwt |
| POST | `/v1/account/connect` | jwt |
| PUT | `/v1/account/:accountId` · `/v1/account/:email/tokens` | jwt |
| DELETE | `/v1/account/:accountId` | jwt |
| GET | `/v1/calendar/events` · `/events/:eventId` · `/calendars` · `/calendars/by-account/:accountId` | jwt |
| POST/PUT/DELETE | `/v1/calendar/events` · `/events/:eventId` | jwt |
| GET | `/v1/account/all-with-calendars` · `/v1/account/:accountId/calendars` · `/v1/:accountId` | jwt |
| PUT | `/v1/account/:accountId/calendars/imported` | jwt |
| POST | `/v1/events/:targetCalendarId/sync` · `/sync/resync` · `/sync/update` | jwt |
| POST | `/v1/events/:accountId/sync/stop` | jwt |
| GET | `/v1/events/preview/calendar/:accountId` · `/active-syncs` · `/active-syncs-with-history` | jwt |
| GET | `/v1/events/sync-history/:targetCalendarId` · `/v1/events/:accountId` | jwt |
| GET (SSE) | `/v1/events/stream` | jwt |
| GET | `/v1/events/sync/status/:jobId` | jwt (`SkipThrottle`) |
| DELETE | `/v1/events/sync/:targetCalendarId` · `/v1/events/source/:eventId` | jwt |
| GET | `/v1/google/update/account/:email` | jwt |

### admin-be (`:10001`) — controller-level `KongJwtGuard + AdminRoleGuard`

| Method | Path | Auth |
|---|---|---|
| GET | `/v1/health` | none |
| GET | `/v1/admin/me` | admin |
| GET | `/v1/users` · `/v1/users/:id` | admin |
| PATCH | `/v1/users/:id` | admin |
| POST | `/v1/users/:id/disable` (10/min) · `/enable` · `/grant-admin` · `/revoke-admin` | admin |
| DELETE | `/v1/users/:id` | admin (10/min) |

### notification — no public HTTP API

| Method | Path | Auth |
|---|---|---|
| GET | `/v1/health/live` · `/v1/health/ready` | none |

Everything else is Kafka: `send_welcome_email`, `send_login_otp_email`, and the DLQ topic consumer.
