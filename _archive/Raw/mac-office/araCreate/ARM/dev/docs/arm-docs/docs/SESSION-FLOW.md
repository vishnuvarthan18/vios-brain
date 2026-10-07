# ARM Platform — session service Session & Proxy Flow

> Runtime behavior of `arm-session`: how a session is created, stored, presented as a cookie, used to inject JWTs on proxied calls, refreshed, and destroyed.
>
> Distinct from [`docs/architecture/session-architecture.md`](architecture/session-architecture.md) (why a standalone session service) and [`core/arm-session/README.md`](../../../core/arm-session/README.md) (developer implementation guide). This document is the **lifecycle + proxy** reference.
>
> Verified against `core/arm-session/src/session/**` and `session/**` (2026-08-10).

---

## 1. Role in one sentence

The session service is the **only** process that holds access/refresh JWTs for the browser. The browser holds an opaque httpOnly `SESSION_ID` cookie; every MFE API call goes to `/session/proxy/...` and the session service injects `Authorization: Bearer …` toward backends.

Hard rules: [ARCHITECTURE.md](ARCHITECTURE.md) §2. Login product behavior: [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md).

---

## 2. Session lifecycle

```text
┌─ Create ──────────────────────────────────────────────────────────────┐
│  A. Email OTP (browser → session service)                                         │
│     POST /session/login/email/verify                                      │
│       → core-be /v1/auth/login/email/verify                           │
│       → SessionService.createSession → Redis                          │
│       → Set-Cookie: SESSION_ID                                        │
│                                                                       │
│  B. SSO Google/Microsoft (core-be → session service internal → redirect)          │
│     core-be POST /session/internal/create-session (X-Session-Secret)          │
│       → Redis session + { signedId, handoffId }                       │
│       → redirect SESSION_PUBLIC_URL/session/oauth/callback?sid=…              │
│       → Set-Cookie: SESSION_ID → redirect FRONTEND_URL/auth/callback  │
│                                                                       │
│  C. Handoff activate (implemented, currently unwired from FE)         │
│     POST /session/session/activate { handoffId } → cookie                 │
└───────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─ Use ─────────────────────────────────────────────────────────────────┐
│  Browser sends SESSION_ID on credentialed calls to session service host           │
│  SessionGuard → Redis getSession → req.session                         │
│  ProxyService.refreshIfNeeded → inject Bearer → upstream              │
└───────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─ End ─────────────────────────────────────────────────────────────────┐
│  POST /session/logout (SessionGuard)                                          │
│    → best-effort core-be /v1/auth/logout                              │
│    → Redis DEL session                                                │
│    → clearCookie SESSION_ID                                           │
│  Or Redis TTL expires → next request 401                              │
└───────────────────────────────────────────────────────────────────────┘
```

### Correction vs AUTH-ARCHITECTURE §1

AUTH-ARCHITECTURE says both OTP and SSO converge on `POST /session/internal/create-session`. **OTP does not** — the session service verify handler creates the Redis session itself and sets the cookie. Only SSO (and the unused handoff path) use `/session/internal/create-session`.

### Endpoint map (session-relevant)

| Method | Path | Auth | Role |
|---|---|---|---|
| `POST` | `/session/login/email/request-otp` | throttle 5/60s | Proxy OTP request to core-be |
| `POST` | `/session/login/email/verify` | throttle 5/60s | Verify OTP → create session + cookie |
| `GET` | `/session/oauth/callback` | none (uses `sid` query) | SSO cookie issue after internal create |
| `POST` | `/session/internal/create-session` | `X-Session-Secret` | S2S session create (SSO) |
| `POST` | `/session/session/activate` | none | Redeem `handoffId` → cookie |
| `GET` | `/session/me` | `SessionGuard` | Current session user |
| `POST` | `/session/logout` | `SessionGuard` | Destroy session |
| `ALL` | `/session/proxy/*` | throttle 300/60s + `SessionGuard` | Authenticated API proxy |

Controller: `core/arm-session/src/session/session.controller.ts`. Guard: `session.guard.ts`.

---

## 3. `SESSION_ID` cookie

| Attribute | Value |
|---|---|
| Name | `SESSION_ID` |
| Value | `{uuid}.{hmac-sha256-hex}` signed with `SESSION_COOKIE_SECRET` |
| `httpOnly` | `true` |
| `sameSite` | `'lax'` |
| `secure` | `true` when `NODE_ENV` is `production` **or** `staging`; else `false` |
| `path` | `'/'` |
| `domain` | **omitted** — host-only for the session service host (`localhost:5001` locally) |
| `maxAge` | Hardcoded **30 days** (`SESSION_MAX_AGE_MS`) |

Redis TTL is controlled separately by `SESSION_TTL_SECONDS` (default also 30d). If you change the env TTL without changing `SESSION_MAX_AGE_MS`, cookie lifetime and Redis lifetime can drift.

`SessionGuard` on IP hash mismatch: **warn only**, does not reject (session still accepted).

---

## 4. Redis schema

| Key | TTL | Contents |
|---|---|---|
| `session:data:{uuid}` | `SESSION_TTL_SECONDS` (default `2592000`); refreshed on `updateSession` | JSON `Session` |
| `session:handoff:{uuid}` | 30s | signedId string (one-time redeem) |
| `session:refresh-lock:{uuid}` | 5s | lock token for cross-instance refresh |

```typescript
interface Session {
  accessToken: string;
  refreshToken: string;
  userId: string;
  userAgent: string;
  ipHash: string;       // sha256 of client IP
  createdAt: string;    // ISO
  refreshedAt: string;  // ISO; updated on silent refresh
}
```

Cookie carries the **signed** id; Redis key uses the bare uuid after HMAC verify (`timingSafeEqual`).

---

## 5. Proxy path

```text
FE → ${VITE_SESSION_URL}/session/proxy/{service}/{rest}?query
  → SessionGuard (cookie → Redis)
  → ProxyService.refreshIfNeeded
  → GET/POST/… {SERVICE_URL}/{rest}?query
       Authorization: Bearer {accessToken}
       x-correlation-id: {req.id}
```

### `SERVICE_ENV_MAP`

| URL segment | Env var |
|---|---|
| `core` | `CORE_BE_URL` |
| `calendar` | `CALENDAR_BE_URL` |
| `admin` | `ADMIN_BE_URL` |
| `projects` | `PROJECTS_BE_URL` (reserved; optional) |

Unknown service → `502`.

### Rewrite

1. Strip `/session/proxy/`, split service vs path (query stripped from path string).
2. `buildTargetUrl(base, path, req.query)` — WHATWG `URL`; nested query flattened with bracket notation.
3. Example: `/session/proxy/calendar/v1/events?from=…` → `{CALENDAR_BE_URL}/v1/events?from=…`.

### Headers

| Direction | Behavior |
|---|---|
| Request strip | `host`, `cookie`, `authorization`, `content-length`, and the internal trust headers `x-session-secret`, `x-requesting-user-id`, `x-user-id`, `x-forwarded-*`, `x-real-ip` (all client-settable; backends read them as caller identity) |
| Request inject | `Authorization: Bearer …`, `x-correlation-id` |
| Response strip | `transfer-encoding`, `connection`, all `access-control-*` (upstream CORS must not override session service CORS) |

Request bodies take one of two paths: json/urlencoded are **buffered** and re-serialized (1MB parser cap), while anything the parsers skip that carries a `content-type` — multipart above all — is **streamed** through with `content-length` preserved, capped at 12MB (413 over, 411 for chunked). Responses stream. Timeout 30s.

Correlation id: prefer inbound `x-correlation-id` / `x-request-id`, else new UUID (`app.module.ts`).

---

## 6. Silent token refresh

Runs **before every proxied request** in `ProxyService.refreshIfNeeded`:

1. Decode JWT `exp` (no signature verify at this step).
2. If `exp - now <= TOKEN_REFRESH_THRESHOLD_SECONDS` (default **60**), refresh.
3. `POST {CORE_BE_URL}/v1/auth/refresh` with `{ refreshToken }`.
4. `sessions.updateSession` with new access/refresh (resets Redis TTL).

**Concurrency**

- Same instance: in-memory `refreshPromises` Map coalesces concurrent refreshes for one `signedId`.
- Cross-instance: Redis `SET NX PX` lock; losers poll Redis up to ~6s for the peer’s new token.

**Failure behavior:** refresh errors are logged; session service may forward the **stale** access token rather than fail the proxy call immediately — upstream then typically returns `401` if the token is already expired. This is intentional soft-fail, not a silent success.

---

## 7. Dedicated OAuth routes (not the proxy)

Calendar Add Account cannot use `/session/proxy/...`: axios would follow the `302` to Google server-side. Dedicated routes capture `Location` with `maxRedirects: 0` and redirect the browser.

Full flow: [`apps/arm-app-calendar/GOOGLE-OAUTH.md`](calendar/GOOGLE-OAUTH.md).

Those routes use `SessionGuard` + session JWT for S2S calls — **not** `X-Session-Secret`.

---

## 8. Internal auth (`X-Session-Secret`)

| Path | Auth |
|---|---|
| Browser → session service (proxy, me, logout, calendar OAuth) | `SESSION_ID` → `SessionGuard` |
| session service → backends (proxy) | Bearer access token from Redis |
| Service → `POST /session/internal/create-session` | Header `x-session-secret` vs `SESSION_INTERNAL_SECRET` (`timingSafeEqual`) |

**Not** “Kong → session service” for the normal proxy path. Local compose has no Kong on the browser path. Only **arm-core-be** OAuth callbacks currently call `/session/internal/create-session`.

Calendar-be → core-be token fetches use the same shared secret on core-be’s internal guards — that is separate from the session service proxy.

---

## 9. CORS & local cookies (`:3000` → `:5001`)

```typescript
app.enableCors({
  origin: process.env.ALLOWED_ORIGIN || 'http://localhost:3000',
  credentials: true,
});
```

- Single allowed origin (not a list).
- FE must use credentialed fetch (`withCredentials` / `credentials: 'include'`) — core-fe `AuthClient` does.
- Cookie is host-only on the session service origin; Lax + credentialed CORS is what makes shell → session service XHR work locally.
- `secure: false` in `development` so HTTP localhost works.

See [ARCHITECTURE.md](ARCHITECTURE.md) §3 for edge differences across environments.

---

## 10. Known bugs & limitations

| Item | Status |
|---|---|
| **Query-param duplication on proxy** | **Fixed** (2026-07). Historical bug was in `forward()` using both a concatenated URL and axios `params` — not in `parseUrl()` itself (inventory wording was imprecise). Fix: `buildTargetUrl()` + drop `params: req.query`; follow-up nested-query flatten. No workaround needed. |
| **Tokens in OAuth URL (SEC-24 inventory suggestion)** | Mitigated by design: SSO redirects carry `sid=` (signed session id), not JWTs. Residual: `sid` appears briefly in the URL before the cookie is set. |
| **Multipart / streamed uploads** | Open limitation — proxy buffers JSON `req.body` only. |
| **Cookie maxAge vs `SESSION_TTL_SECONDS`** | Possible drift if env TTL is changed without code change. |
| **`session/activate` / `handoffId`** | Implemented in session service; no live FE/core-be caller found — treat as unwired. |
| **Refresh soft-fail** | Stale token may be forwarded after refresh failure (§6). |

Open session service functional bugs in `core/arm-session/TASK_SHEET.md`: none listed at doc time (BFF-01–18 marked done).

---

## 11. Env vars (session / proxy)

Startup-required: `SESSION_COOKIE_SECRET`, `SESSION_INTERNAL_SECRET`, `CORE_BE_URL`, `REDIS_HOST`.

| Var | Role |
|---|---|
| `SESSION_COOKIE_SECRET` | HMAC for cookie signedId |
| `SESSION_TTL_SECONDS` | Redis TTL (default 30d) |
| `TOKEN_REFRESH_THRESHOLD_SECONDS` | Silent refresh window (default 60) |
| `SESSION_INTERNAL_SECRET` | Internal create-session |
| `CORE_BE_URL` / `CALENDAR_BE_URL` / `ADMIN_BE_URL` | Proxy map |
| `ALLOWED_ORIGIN` | CORS origin |
| `FRONTEND_URL` | Post-OAuth / error redirects |
| `REDIS_*` | Session store |
| `NODE_ENV` | Cookie `secure` + logging |
| `VITE_SESSION_URL` / `SESSION_PUBLIC_URL` | Browser-facing session service (`http://localhost:5001` local) |

Full catalog: [ENV-VARS.md](ENV-VARS.md) §3.

---

## Related docs

| Doc | Role |
|---|---|
| [`docs/architecture/session-architecture.md`](architecture/session-architecture.md) | Why a standalone session service |
| [`core/arm-session/README.md`](../../../core/arm-session/README.md) | Implementation guide (DOC-A03) |
| [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) | Platform login / JWT / guards |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Platform map + edge-by-environment |
| [`GOOGLE-OAUTH.md`](calendar/GOOGLE-OAUTH.md) | Dedicated calendar OAuth routes |
| [ENV-VARS.md](ENV-VARS.md) | Env reference |
| [`troubleshooting/auth-failures.md`](troubleshooting/auth-failures.md) | Symptom-first triage guide built on this document's session/refresh mechanics |
| [KONG-CONFIG.md](KONG-CONFIG.md) | Kong's separate, unrelated `/api/core`/`/api/calendar` API surface — not the browser's path this document covers |
