# ARM Platform — Authentication & Authorization Architecture

> Read the workspace-level [CLAUDE.md](../../../CLAUDE.md) first. This document covers every auth concern that spans more than one repo: how a user logs in, how a JWT becomes a session, how each backend decides whether to trust a request, and how admin access and account-disable actually work today — including where that last one doesn't fully work yet.
>
> Assembled from `core/arm-core-be/CLAUDE.md` ("Login", "Auth Guard"), `core/arm-session/CLAUDE.md` ("Google OAuth Flow"), `core/arm-core-fe/CLAUDE.md` ("Auth State"), and `apps/arm-admin/src/backend/CLAUDE.md` ("Auth Guard Stack", "Deny-List Mechanism") — with the JWT claims, the guard dual-path, and the deny-list wiring verified directly against source rather than taken as-is from those files (two corrections below).

---

## 1. Platform Login Flow

There are exactly two ways to log in. **Password login does not exist** — it was removed entirely, not just deprecated:

- **Email OTP** — `POST /v1/auth/login/email/request-otp` generates a 6-digit code, stores it in Redis (5 min TTL), and publishes it to Kafka for `arm-service-notification` to email. `POST /v1/auth/login/email/verify` checks the code (rate-limited, single-use) and finds-or-creates the user by email. Any existing account can log in this way, **including one created via Google/Microsoft SSO** — `authProvider` records how the account was created and is not consulted by any login path. The same holds in reverse and across providers: an email-created account can sign in with Google or Microsoft, and a Google-created account with Microsoft. An SSO login links its provider id onto the existing row, matched by the same verified email address. An emailed code is therefore a way into any account, which means the OTP's own hardening (6-digit CSPRNG code, 5-minute TTL, 5 attempts then invalidated, throttled requests, constant-time compare) is the security floor for every account rather than just email-only ones. There is deliberately no per-account or per-org switch to restrict this; if a tenant ever needs SSO-only enforcement, it belongs at the org-policy level, not per user.
- **Google / Microsoft SSO** — `GET /v1/auth/google` (or `/microsoft`) redirects to the provider's consent screen; the callback always auto-provisions on first login. There is no separate "signup" step for either method, and deliberately no domain/invite allowlist restricting who can self-provision.

### Admin approval gate

When `SIGNUP_REQUIRES_APPROVAL=true` on core-be, proving an identity is no longer enough to get in. A completed signup still creates the user row and records the terms acceptance, but issues **no session**; the account lands as `approval-status = 'pending'` and an admin releases it from the admin portal.

- **The gate** is `core/arm-core-be/src/auth/approval-gate.ts`, called from all four paths that could otherwise open a session: email OTP verify, `googleLogin`, `microsoftLogin`, and `refresh`. Only an explicit `'approved'` passes — it is written that way round so a query that fails to load the column fails **closed** rather than disabling the gate everywhere at once.
- **`pending`** returns an `awaiting-approval` outcome; the browser lands on `/auth/pending-approval`, which holds no state and polls nothing.
- **`rejected`** throws a 401 with a deliberately neutral message ("This account is not available. Please contact your administrator."). Rejection is reversible — an admin can approve afterwards.
- **The flag is not consulted by the gate.** It decides only what a *new* signup is written as; whether a status may log in is not negotiable, so an account already queued stays out even if the flag is switched off.
- **Approval and suspension are independent axes.** `approval-status` means "never let in yet"; `is-active` means "suspended by an admin" and drives `disabled_users` plus the Redis deny list. Approving does not un-suspend.

Emails on this path, all via Kafka → `arm-service-notification`:

| Event | Emitted when | To |
|---|---|---|
| `send_admin_new_signup_email` | a signup joins the queue | every admin with `notify-on-signup`, else `ADMIN_SIGNUP_NOTIFICATION_EMAIL` |
| `send_account_approved_email` | an admin approves | the approved user |

The approval email is triggered by admin-be calling `POST /v1/auth/internal/user-approved` (X-Session-Secret) — core-be owns both the users table and the Kafka producer, so it takes only a `userId` and reads the address itself. Emission is fire-and-forget, so the admin portal carries a **Resend** action for when a broker outage swallows it.

Both methods issue JWTs from core-be, but they **do not** share one session-creation path:

- **SSO** — core-be calls `POST /session/internal/create-session` (server-to-server, `X-Session-Secret`), then redirects the browser to `/session/oauth/callback?sid={signedId}&state={csrfNonce}` so the session service can set the cookie.
- **Email OTP** — the browser talks to `POST /session/login/email/verify`; the session service calls core-be to verify the code, then **creates the Redis session itself** and sets `SESSION_ID` on that response. It does **not** go through `/session/internal/create-session`.

SSO redirect sequence:

```text
core-be issues a JWT access/refresh pair
        ↓
core-be calls POST /session/internal/create-session (server-to-server, X-Session-Secret header)
        ↓
session service creates a Redis session (accessToken + refreshToken stored server-side, never sent to the browser)
        ↓
session service returns { signedId, handoffId } to core-be
        ↓
core-be redirects the browser to /session/oauth/callback?sid={signedId}&state={csrfNonce}
        ↓
session service sets the SESSION_ID cookie (httpOnly, sameSite=lax, secure only in production/staging) → redirects to the frontend
```

The session service is the only thing that ever sets a cookie on the browser, and `SESSION_ID` is the only token the browser ever holds. Full session/proxy lifecycle (including the OTP cookie path, silent refresh, and proxy rewrite): [`SESSION-FLOW.md`](SESSION-FLOW.md). See also [`core/arm-session/README.md`](../../../core/arm-session/README.md).

---

## 2. JWT Structure & Configuration

JWTs are issued by `arm-core-be` (`AuthService.{login,googleLogin,microsoftLogin,refresh}`) and are HMAC-signed, symmetric-key tokens — there is no public/private keypair, every verifying service needs the same secret.

**Claims** (`core/arm-core-be/src/auth/strategies/jwt.strategy.ts`):

| Claim | Meaning |
| --- | --- |
| `sub` | User ID. Required — a token without it is rejected. |
| `email` | User's email, carried for convenience. |
| `iss` | Issuer. |

**TTLs:** access tokens default to 15 minutes (`JWT_ACCESS_TOKEN_TTL`), refresh tokens to 30 days (`JWT_REFRESH_TOKEN_TTL`). Refresh tokens are stored as SHA-256 hashes (never plaintext) in `refresh_tokens`, grouped by `familyId` — reusing a revoked token kills the entire family, which is the stolen-refresh-token defense.

**Secret rotation:** `core-be`'s `JwtStrategy` supports a `JWT_ACCESS_SECRET_PREVIOUS` fallback. On verification failure against the current `JWT_ACCESS_SECRET`, it retries against the previous one before rejecting — so rotating the signing secret doesn't force a mass logout; tokens issued under the old secret keep working until they expire naturally.

**Correction to this section's source material:** `core-be/CLAUDE.md`'s "JWT Configuration" section states both `JWT_ACCESS_SECRET` *and* `JWT_REFRESH_SECRET` "must be identical to the values in `arm-app-calendar`." That's not what the code does — `apps/arm-app-calendar/apps/backend/src` only ever reads `JWT_ACCESS_SECRET` (confirmed via `grep` across the whole backend source and `.env.example`; `JWT_REFRESH_SECRET` appears nowhere in that repo). This matches the root [`CLAUDE.md`](../../../CLAUDE.md)'s existing guidance: only `JWT_ACCESS_SECRET` needs to match across `arm-core-be` and `arm-app-calendar`; `JWT_REFRESH_SECRET` is core-be-only. Worth fixing in `core-be/CLAUDE.md` — treat that file's claim as stale until it is.

---

## 3. Auth Guards Per Service

Every backend that isn't the session service (`arm-core-be`, `apps/arm-app-calendar/apps/backend`, `apps/arm-admin/src/backend`) uses the same dual-path guard pattern, even though it's named differently per repo (`KongJwtGuard` in core-be and admin-be, `AuthGuard` in calendar-be):

1. **Kong path** — only trusted when `TRUST_PROXY_HEADERS=true`. If Kong's `X-User-Id` header is present, it's trusted outright and no JWT verification happens. This is the server path — Kong verified the JWT at the edge.
2. **Direct path** — the default, and the only path in local dev, where Kong is not in the request path at all (local compose publishes services on host ports with no edge proxy; see [ARCHITECTURE.md](ARCHITECTURE.md) §3). Falls through to Passport JWT verification against `JWT_ACCESS_SECRET`.

`TRUST_PROXY_HEADERS` gating the header-trust path exists specifically because of a fixed bug: calendar-be's guard carries an explicit code comment referencing `SEC-001`, an auth bypass where the guard used to trust a client-supplied `X-User-Id` header unconditionally — any caller could impersonate any user by ID. The gate is the fix; it must stay `false` in every environment that isn't genuinely behind Kong.

**Calendar-be's guard has one extra behavior the others don't:** on an expired access token, if a `refresh_token` cookie or `x-refresh-token` header is present, it calls `POST {CORE_API_URL}/v1/auth/refresh` directly and retries verification with the new token — a second, independent refresh path alongside the session service's own silent refresh (see `core/arm-session/README.md`'s "Proxying to backends"). Both exist; a request that reaches calendar-be with an already-expired token that the session service didn't catch in time still has a chance to self-heal here.

**Internal service-to-service calls** (core-be → session service, calendar-be → core-be for token fetches) don't use JWTs at all — they use a shared `X-Session-Secret: (secret removed) header, checked with a timing-safe comparison (`SessionSecretGuard` / equivalent per service).

---

## 4. Role-Based Access Control — Admin

Admin access is layered on top of the same platform login, not a separate identity system:

1. `KongJwtGuard` runs first (same dual-path pattern as above) — establishes `request.user.userId` from a normal platform JWT.
2. `AdminRoleGuard` runs second — looks up `userId` in `arm_admin.admin_users`. Not found → `403`. Found → `request.admin = { userId, role }`, cached in Redis as `admin_role:{userId}` (5 min TTL, falls back to a DB query and a warning log if Redis is down — it never fails closed on Redis being unreachable).
3. Controllers read the resolved admin via `@CurrentAdmin()`.

So there's no separate "admin login" — any platform user who exists in `admin_users` gets elevated access on their normal session. `core-fe`'s `isAdmin` flag in the Zustand auth store is set the same way: a separate `adminMe()` check, gated on session verification finishing first (to avoid racing the session service's token refresh).

### Deny-list (disable/enable a user) — verified gap

`arm-admin/src/backend`'s `CLAUDE.md` documents disabling a user as a two-layer operation: revoke all refresh tokens in `arm_core.refresh_tokens`, write a row to `arm_admin.disabled_users`, then write `disabled:{userId}` to Redis (15 min TTL, matching the access-token TTL), and claims this Redis key is checked as a "hot-path check in core-be and calendar-be guards."

**That check does not exist.** A repo-wide search for the `disabled:` key pattern and for `DenyList`/`deny-list` across every backend found it written and read only inside `apps/arm-admin/src/backend` (`DenyListService`, `DenyListSeeder`, `UsersService`) — `core-core-be`, `calendar-be`, and `arm-session` never reference it. In practice this means: disabling a user revokes their **refresh** tokens (so they can't get a new access token once the current one expires), but does **not** invalidate an already-issued **access** token. A disabled user's existing session keeps working against core-be and calendar-be for up to 15 minutes (the default access-token TTL) after being disabled, not the near-immediate cutoff the deny-list's own TTL implies. This is worth a deliberate decision (wire the check into `KongJwtGuard`/`AuthGuard`, or shorten the access-token TTL, or accept the window) rather than leaving the documentation and the behavior silently disagreeing.

---

## 5. Multi-Account Model (Calendar)

A platform login (one `userId`) can have **multiple** linked Google/Microsoft calendar accounts — this is intentionally separate from the single identity used to log in. Two distinct provisioning paths exist for calendar-be's `Account` documents, and they behave differently:

- **Login-provisioned account** (`syncFromCoreAuth()`): calendar-be fetches the token core-be already holds from SSO login, via `CoreAuthService.getGoogleToken(userId)` → `GET {CORE_API_URL}/v1/auth/google/token/:userId` (internal, `X-Session-Secret`-guarded). This path's `Account.refreshToken` is described in `apps/arm-app-calendar/CLAUDE.md` as "almost never populated" — flagged in source as `TASK-037`, which is not tracked in the current `TASK_SHEET.md` and should be treated as an open, unverified-status gap rather than assumed fixed.
- **"Add Account" flow** (either frontend's "+ Add Account" button): a separate, calendar-be-native OAuth flow (`GoogleOAuthService`/`MicrosoftOAuthService`), routed through the session service's dedicated `/session/calendar/add-account(/microsoft)` endpoints specifically because the OAuth consent redirect is a `302` a generic axios-based proxy would swallow server-side (see `core/arm-session/README.md`'s "Adding a second calendar account"). This path does its own consent redirect and code exchange and *does* populate `refreshToken` correctly.

Do not assume the two paths behave the same way — code comments in calendar-be explicitly warn against conflating them. The full mechanics of the Add-Account flow (popup, dedicated session service routes, BroadcastChannel completion signal, CSRF `state` validation, and the separate Reconnect/Update Account path) are in [`apps/arm-app-calendar/GOOGLE-OAUTH.md`](calendar/GOOGLE-OAUTH.md).

---

## 6. Token Expiry and Reconnect

Two independent expiry clocks exist and are easy to conflate:

- **Platform JWT expiry** (15 min access / 30 day refresh) — handled transparently. The session service refreshes silently before every proxied call if the token is within `TOKEN_REFRESH_THRESHOLD_SECONDS` (60s) of expiry; calendar-be additionally self-heals on a `401` by calling core-be's refresh endpoint directly (§3). The browser never sees this happen.
- **Google/Microsoft calendar-account token expiry** — surfaced via `GET /v1/auth/google/status/:userId` (and the Microsoft equivalent), returning `{ isConnected, expiresAt, scope, needsReauth }`. Per `apps/arm-app-calendar/CLAUDE.md`, calendar-be's `isAuthenticated` flag on an `Account` "self-heals on 401/403" — i.e. a failed Google/Microsoft API call flips the account to needing reconnect, surfaced to the user rather than failing silently. Reconnecting uses the **Update Account** flow documented in [`GOOGLE-OAUTH.md`](calendar/GOOGLE-OAUTH.md) §8 (not a re-run of Add Account session service routes), scoped to the specific account email.

---

## 7. Security Constraints

- **Tokens never reach the browser.** Not in a response body, not in a URL, not in `localStorage` — the `SESSION_ID` cookie is the only thing the browser ever holds, and it's opaque (an HMAC-signed UUID, not a JWT). `apps/arm-app-calendar`'s `TASK_SHEET.md` (`SEC-005`) records a since-fixed instance of this rule being violated (a dead frontend file storing a JWT in `localStorage`) — the rule is deliberately checked for.
- **Cookie attributes:** `httpOnly`, `sameSite: 'lax'`, `secure` gated on `NODE_ENV=production` (verified in `core/arm-session/src/main.ts` — local dev runs over plain HTTP so `secure` would break it there).
- **CORS is a single allowed origin**, not a list — `ALLOWED_ORIGIN`, `credentials: true`. No separate CORS code path per environment; only the origin *value* changes.
- **`TRUST_PROXY_HEADERS` must be `false`** everywhere a service isn't genuinely deployed behind Kong (§3) — this is the fix for `SEC-001` and the single most important flag in this document to get wrong.
- **OAuth CSRF:** the SSO login flow relays a `state` nonce through the callback chain; the calendar Add-Account callbacks separately validate `state` against `session.userId` and redirect to an error state on mismatch. `apps/arm-app-calendar/TASK_SHEET.md`'s `SEC-002`/`SEC-003` record two now-fixed gaps in this area (a missing CSRF check on one OAuth callback, and an account-squatting bug where any authenticated user could claim another user's Google identity mid-link).
- **Internal auth uses a shared secret, not mTLS or JWTs** — `X-Session-Secret`, timing-safe compared. Anyone with `SESSION_INTERNAL_SECRET` can call any internal endpoint on any backend; treat it with the same care as a root credential.
- **The deny-list gap in §4 is a live constraint**, not a historical note — until it's wired into core-be's and calendar-be's guards, "disable a user" is a 15-minute-delayed operation for anyone holding a still-valid access token, not an immediate one.

---

## Related Documents

- [`SESSION-FLOW.md`](SESSION-FLOW.md) — session lifecycle, proxy rewrite, silent refresh, cookie/Redis schema
- [`core/arm-session/README.md`](../../../core/arm-session/README.md) — session service implementation guide
- [`ARCHITECTURE.md`](ARCHITECTURE.md) — platform map and edge-by-environment
- [`core/arm-core-be/CLAUDE.md`](../../../core/arm-core-be/CLAUDE.md), [`core/arm-core-fe/CLAUDE.md`](../../../core/arm-core-fe/CLAUDE.md), [`apps/arm-admin/src/backend/CLAUDE.md`](../../../apps/arm-admin/src/backend/CLAUDE.md) — AI-agent context for each service; more implementation detail than this doc, less "why"
- [`calendar/GOOGLE-OAUTH.md`](calendar/GOOGLE-OAUTH.md) — Add Account popup / session service callback / BroadcastChannel / Reconnect (Update Account) mechanics
- [`calendar/ARCHITECTURE.md`](calendar/ARCHITECTURE.md) §5 — calendar-be's token refresh/CoreAuth-fallback detail behind this doc's §5 summary
- [`troubleshooting/auth-failures.md`](troubleshooting/auth-failures.md) — symptom-first triage guide built on this document
- [KONG-CONFIG.md](KONG-CONFIG.md) §2–§3 — the concrete Kong-side mechanism (JWT plugin + Lua pre-function) behind this document's `TRUST_PROXY_HEADERS`/`X-User-Id` description
