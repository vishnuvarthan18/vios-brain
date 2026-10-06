# arm-core-be — API Reference

> Every HTTP endpoint `arm-core-be` exposes — auth, users, registry, health — with guards, request/response shapes, and the platform's error format. This is the narrative companion to the service's own live OpenAPI spec, not a replacement for it.
>
> **Interactive spec:** `GET /api-docs` (Swagger UI), dev/local only — gated off in production/staging in `main.ts`. Base path for every route below is `http://localhost:4000/v1` in local dev (global prefix `v1`, set in `main.ts`; host port `4000`, container port `3000`). All routes in this platform are normally reached through the session service proxy (`/session/proxy/core/v1/...`), not called directly by a browser — see [SESSION-FLOW.md](../SESSION-FLOW.md).
>
> **Swagger coverage is real but partial — verified by counting `@Api*` decorators per controller.** `auth.controller.ts` is fully annotated (`@ApiOperation`/`@ApiResponse` on every route). `users.controller.ts` has only a bare `@ApiTags` — its actual routes are inherited from `GenericController`, which carries zero Swagger decoration. `registry.controller.ts` and `health.controller.ts` have none at all. In practice: `/api-docs` will show accurate, described auth endpoints, and bare undescribed route signatures for everything else. This document covers all four controllers at the same level of detail regardless of what Swagger currently shows.
>
> Verified against every controller (`auth`, `users`, `registry`, `health`), every DTO, `common/generic.controller.ts`, `common/filters/all-exceptions.filter.ts`, and `auth/strategies/jwt.strategy.ts` (2026-08-11).

---

## 1. Auth guard & JWT — enough to read the tables below

Full detail is in [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) and [SECURITY.md](../SECURITY.md) — this is the minimum needed to read "Guard" column entries in §2–§4.

- **`KongJwtGuard`** (wired app-wide via `GlobalAuthGuard`, an `APP_GUARD`) is the default on every route unless overridden. Dual path: trusts Kong's `X-User-Id` header only when `TRUST_PROXY_HEADERS=true`; otherwise verifies the JWT directly via Passport. In local dev, `TRUST_PROXY_HEADERS` is unset — every request falls through to full JWT verification.
- **`@Public()`** on a route bypasses `GlobalAuthGuard` entirely.
- **`SessionSecretGuard`** (used alongside `@Public()` on a handful of routes) validates the `X-Session-Secret` header instead of a user JWT — for calls from another platform service, not a logged-in user.
- **JWT payload** (`JwtStrategy`'s `JwtPayload` interface): `{ sub?: string, email?: string, iss?: string }` — no role/admin claim embedded. `req.user` after a successful guard check is `{ id: payload.sub, email: payload.email }`.
- **Verified, not previously documented elsewhere in this workspace:** `JwtStrategy` supports **secret rotation** — it tries `JWT_ACCESS_SECRET` first, and if the token's signature doesn't match, falls back to `JWT_ACCESS_SECRET_PREVIOUS` (both compared via `timingSafeEqual`, not simple string equality). This lets a secret be rotated without invalidating every in-flight access token — old tokens verify against the previous secret until they naturally expire. [ENV-VARS.md](../ENV-VARS.md) already lists `JWT_ACCESS_SECRET_PREVIOUS` as an optional rotation fallback not present in `.env.example`; this confirms the code path it refers to.

---

## 2. Auth endpoints (`/v1/auth/*`)

| Method | Path | Guard | Purpose |
|---|---|---|---|
| `POST` | `/auth/login/email/request-otp` | `@Public()`, `Throttle` (5/60s) | Step 1 — send a 6-digit login code. Body: `RequestLoginOtpDto { email }`. Always the same response whether or not the account exists. |
| `POST` | `/auth/login/email/verify` | `@Public()`, `Throttle` (5/60s) | Step 2 — verify the code. Body: `VerifyLoginOtpDto { email, code }` (`code` must match `/^\d{6}$/`). **Three outcomes:** an approved account sets an `httpOnly` `refresh_token` cookie (30d, `sameSite: strict`, `secure` in prod/staging) and returns `{ userId, accessToken, refreshToken }`; an address with no account yet returns `{ pendingId }` and no cookie; an account awaiting admin approval returns `{ status: "awaiting-approval" }` and no cookie. A rejected account gets a `401` with a neutral message. |
| `POST` | `/auth/signup/complete` | `@Public()`, `Throttle` (5/60s) | Turns a pending signup into an account — **the only path in the platform that creates a user row**. Body: `CompleteSignupDto { pendingId, termsVersion, acceptedTerms }`; `termsVersion` is validated against `KNOWN_TERMS_VERSIONS` **before** the single-use pending record is consumed. Returns tokens and a cookie as above, or `{ status: "awaiting-approval" }` with neither when `SIGNUP_REQUIRES_APPROVAL=true`. `401` if the id is unknown, expired, spent, or presented by a different browser (one message for all four); `409` if the email was registered meanwhile. |
| `GET` | `/auth/me` | `KongJwtGuard` | Returns `{ userId }` for the caller's own token. |
| `POST` | `/auth/refresh` | none (reads token from body or cookie) | Body: `{ refreshToken?: string }`, falls back to the `refresh_token` cookie if absent. Returns `{ ok: false }` (200, not an error) if no token is found anywhere; otherwise rotates the cookie and returns `{ ok: true, accessToken, refreshToken }`. |
| `POST` | `/auth/logout` | `KongJwtGuard` | Body: `{ refreshToken?: string }` (or cookie). Clears the cookie unconditionally; if a token+userId pair is available, revokes it server-side too — throws `500` if that revoke step fails (cookie is still cleared either way, so the browser session is gone regardless of the throw). |
| `GET` | `/auth/google` | `@Public()`, `GoogleAuthGuard` | Redirects to Google's consent screen. Same entry point for signup and login — always auto-provisions. |
| `GET` | `/auth/google/callback` | `@Public()`, Passport `google` | OAuth callback. Calls `POST /session/internal/create-session` server-to-server, then redirects the browser to `{SESSION_PUBLIC_URL}/session/oauth/callback?sid=...`. On any session service-call failure, redirects to `{FRONTEND_URL}/auth/login?error=oauth_failed` instead of hanging or leaking a partial session. |
| `POST` | `/auth/google/revoke` | `KongJwtGuard` | Revokes the caller's own Google token. |
| `POST` | `/auth/internal/user-approved` | `@Public()`, `SessionSecretGuard` | Called by admin-be when an admin approves an account, and by its **Resend** action. Body: `UserApprovedDto { userId }` — **the id only**: the address and name are read from the users table here, so a caller holding the internal secret cannot aim a platform email at an arbitrary address. Emits `send_account_approved_email`. Returns `{ queued: true }` — queued, not sent, because the producer is fire-and-forget. `404` for an unknown user. |
| `GET` | `/auth/google/token/:userId` | `@Public()`, `SessionSecretGuard` | Used by calendar-be to fetch a Google token for `userId`. **Requires an `X-Requesting-User-Id` header equal to `:userId`** — a missing header used to skip the comparison entirely, so the secret alone was enough to read any user's token. Returns `{ hasToken: false, message }` if none, or `{ hasToken: true, accessToken, expiresAt, scope }`. |
| `GET` | `/auth/google/status/:userId` | `KongJwtGuard` | Returns `{ isConnected, expiresAt, scope, needsReauth }`. Throws `403` if `caller.id !== userId` — this is a self-only lookup despite `userId` being a path param, not derived from the token. Deliberately uses the read-only status check, not the auto-refreshing token getter, so a status poll can't itself trigger a 401 on a stale token. |
| `GET` | `/auth/microsoft` | `@Public()`, `MicrosoftAuthGuard` | Same pattern as `/auth/google`. |
| `GET` | `/auth/microsoft/callback` | `@Public()`, Passport `microsoft` | Same pattern as `/auth/google/callback`. |
| `POST` | `/auth/microsoft/revoke` | `KongJwtGuard` | Deletes the local `microsoft_tokens` row only — Microsoft has no server-side per-token revoke API, so the upstream grant isn't invalidated. |
| `GET` | `/auth/microsoft/token/:userId` | `@Public()`, `SessionSecretGuard` | Same pattern as the Google token endpoint. |
| `GET` | `/auth/microsoft/status/:userId` | `KongJwtGuard` | Same self-only pattern as Google status. |

**Approval gate.** When `SIGNUP_REQUIRES_APPROVAL=true`, every path above that could open a session runs `approval-gate.ts` first — email verify, both OAuth callbacks, and `/auth/refresh`. The OAuth callbacks express the same three outcomes as redirects: `?pending=…` for a first-time identity, `?approval=awaiting-approval` for an unapproved account, and the normal session handoff otherwise. See [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §1.

**Deliberately absent:** `/auth/login`, `/signup`, `/verify-email`, `/set-password`, `/forgot-password`, `/verify-reset-otp`, `/reset-password`, `/change-password`, and the old login-only `google/login`/`microsoft/login` variants — all removed when password auth was dropped platform-wide and signup/login merged into one auto-provisioning flow. See [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md).

---

## 3. User endpoints (`/v1/users/*`)

`UsersController extends GenericController<User, CreateUserDto, UpdateUserDto>` — some routes are the generic base class's, some are overridden to permanently reject.

| Method | Path | Source | Behavior |
|---|---|---|---|
| `GET` | `/users` | Overridden | **Always throws `403 Forbidden`** — `getAll()` is permanently disabled, regardless of caller. |
| `POST` | `/users` | Overridden | **Always throws `403 Forbidden`** — direct account creation is not permitted; accounts are created only via OTP/SSO login. `CreateUserDto` is an intentionally empty class that exists solely to satisfy `GenericController`'s type parameter — it has no fields and nothing ever reads a body for this route. |
| `GET` | `/users/:id` | Inherited (`GenericController.getOne`) | `ParseUUIDPipe` on `:id`. `checkReadOwnership` is overridden to require `id === caller.id` — throws `403` otherwise. Self-only, not admin-bypassable at this layer. |
| `PUT` | `/users/:id` | Inherited (`GenericController.update`) | `ParseUUIDPipe` on `:id`. Body: `UpdateUserDto { email?, firstName?, lastName? }` (each field independently optional, validated with `class-validator` when present — `@IsEmail`/`@IsString`). `checkUpdateOwnership` is overridden the same way as read — self-only. |
| `DELETE` | `/users/:id` | Overridden | **Always throws `403 Forbidden`** — account deletion has no UI surface yet and is not exposed to regular users (code comment: "must not be exposed to regular users"). |

**Note the HTTP method for update is `PUT`, not `PATCH`**, despite `UpdateUserDto`'s fully-optional fields making it behave like a partial update — this is `GenericController`'s fixed convention (`@Put(':id')`), not a per-route choice `UsersController` made.

---

## 4. Registry endpoints (`/v1/registry/*`)

Service/MFE self-registration and lookup — backs `arm-core-fe`'s Module Federation host discovering remote URLs at runtime. **Corrected 2026-08-12** (was previously stated as never called by `core-fe` — that was wrong): `core-fe` does call `GET /registry/:name` for `calendar`, on every authenticated session, from `ProtectedRoute` (`core-fe/src/lib/remote-registry.ts`). What's actually true: nothing ever calls `POST /registry/register`, so the table is permanently empty and the read never has anything to return — remote discovery in practice is still the static `VITE_CALENDAR_REMOTE_URL`/`VITE_ADMIN_REMOTE_URL` env vars baked in at build time. Full detail in [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §2.4 and [`core/arm-core-be/REGISTRY.md`](../../../../core/arm-core-be/REGISTRY.md).

| Method | Path | Guard | Purpose |
|---|---|---|---|
| `POST` | `/registry/register` | `@Public()`, `SessionSecretGuard` | Registers/overwrites a service's remote URL. Body: `RegisterServiceDto { name, type: 'backend'\|'mfe', url?, remoteUrl? }` — `url`/`remoteUrl` validated with `@IsUrl({ require_tld: false })` so `http://host:port` local addresses pass. Returns `{ success: true, id }`. Deliberately not reachable by an ordinary authenticated user — code comment: the shell "trusts this value to load code at runtime." |
| `GET` | `/registry` | **`KongJwtGuard` (default — no `@Public()`)** | Paginated list of every registered entry (`page`/`limit` query params, `limit` clamped to 1–100, default 50). Any authenticated platform user can read the full registry — there's no additional authorization layer beyond "logged in." |
| `GET` | `/registry/type/:type` | `KongJwtGuard` | Same pagination, filtered by `type` (`'backend'` \| `'mfe'`) — the `:type` param isn't validated against the enum at the controller level, just cast. |
| `GET` | `/registry/:name` | `KongJwtGuard` | Single entry lookup by exact `name`. |

---

## 5. Health endpoint (`/v1/health*`)

| Method | Path | Guard | Checks |
|---|---|---|---|
| `GET` | `/health/live` | `@Public()` | Bare `{ status: 'ok' }` — no dependency checks. |
| `GET` | `/health` | `@Public()` | `@nestjs/terminus` aggregated check: Postgres (`TypeOrmHealthIndicator.pingCheck`, 2s timeout), Redis (`RedisService.isHealthy`), and a static `core` info block (`{ status: 'up', name: 'core-service', version: '1.0.0', startedAt }`). Returns `503` with per-indicator detail if either dependency check fails. |

Scraped by Prometheus at `/v1/metrics` (separate from these two — see [MONITORING.md](../MONITORING.md) §4), not documented further here since it's not a JSON API response in the same sense.

**Note this is the platform's one health controller *without* a genuine liveness/readiness split at the process level** — `/health` (the "full" check) still responds even though it's not literally named `/ready`, and `/health/live` is the only truly dependency-free route. Functionally equivalent to `calendar-be`'s `/health/live` + `/health/ready` pair, just named `/health/live` + `/health` instead of `/health/live` + `/health/ready`. See [MONITORING.md](../MONITORING.md) §8 for how this compares across all five backends.

---

## 6. Error response format

Every uncaught exception across every controller is normalized by `AllExceptionsFilter` (`@Catch()`, global) to the same JSON shape:

```ts
interface ErrorResponseBody {
  statusCode: number;
  timestamp: string;      // ISO 8601
  path: string;            // request.url
  correlationId: string | undefined;  // matches the pino request log's correlationId
  message: string | string[];
}
```

Special-cased mappings, verified in the filter itself:

| Thrown | Response |
|---|---|
| `QueryFailedError` (TypeORM/Postgres constraint violation) | `400`, message hardcoded to `"The request could not be completed due to a data constraint."` — the real Postgres error text is logged server-side only, never returned to the client |
| `ThrottlerException` | Whatever status the throttler set, message hardcoded to `"Too many requests. Please try again later."` — hides the internal exception class name. Other domain-specific `429`s (e.g. OTP brute-force lockout) keep their own actionable message, since only `ThrottlerException` specifically is intercepted here |
| Any other `HttpException` | Its own status and message (or `.getResponse().message` for `class-validator` array-of-messages bodies) pass through unchanged |
| Anything else (non-`HttpException`, non-`QueryFailedError`) | `500`, generic `"An unexpected error occurred."` — the real error is logged server-side with a stack trace, never returned to the client |

`5xx` responses (and only `5xx`) are logged via `Logger.error` with the stack trace; everything else is either not logged at this layer or logged without a stack.

---

## 7. Input validation

Global `ValidationPipe` (`main.ts`): `{ whitelist: true, forbidNonWhitelisted: true, transform: true }` — unknown body fields are **rejected outright** (400), not silently stripped. This is the stricter of the two patterns used across the platform's four backends; see [SECURITY.md](../SECURITY.md) §6 for the cross-service comparison (`session`/`admin-be` silently strip instead).

---

## 8. Rate limiting

Not repeated here in full — see [SECURITY.md](../SECURITY.md) §7 and [REDIS-ARCHITECTURE.md](../REDIS-ARCHITECTURE.md) §2 for the complete picture (Redis-backed `@nestjs/throttler`, `AppThrottlerGuard` keys by `X-User-Id` when `TRUST_PROXY_HEADERS=true`, else by IP; default bucket `{ ttl: 60000, limit: 120 }`; OTP-specific routes additionally carry their own `@Throttle({ limit: 5, ttl: 60000 })`).

---

## 9. Related documents

| Doc | Role |
|---|---|
| [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) | Full login flow, JWT structure, guard implementation across all 4 backends |
| [SECURITY.md](../SECURITY.md) | Guard pattern verification, rate limiting, input validation comparison across services |
| [SESSION-FLOW.md](../SESSION-FLOW.md) | How these endpoints are actually reached from the browser (via the session service proxy, not directly) |
| [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §2.4 | The registry API's actual read/write consumption status — read path live, write path unused |
| [MONITORING.md](../MONITORING.md) §8 | Health-endpoint comparison across all 5 backends |
| [ENV-VARS.md](../ENV-VARS.md) | `JWT_ACCESS_SECRET`/`JWT_ACCESS_SECRET_PREVIOUS`/etc. |
| [`CLAUDE.md`](../../../../core/arm-core-be/CLAUDE.md) | Source structure, PostgreSQL table reference, what-not-to-do |
