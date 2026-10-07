# session service (Backend for Frontend) — Architecture Decision & Implementation Plan

> **Context**: `arm-core` is one of multiple frontend+backend repo pairs (e.g. `arm-calendar`, `arm-projects`, etc.).
> This document answers: _should the session service live inside `arm-core-be`, or somewhere else?_
>
> See [ADR-001](../adr/001-standalone-bff.md) for the short-form decision record distilled from this proposal, and [SESSION-FLOW.md](../SESSION-FLOW.md) for what was actually implemented.

---

## 1. Why NOT putting session service inside `arm-core-be`

If the session service lives inside `arm-core-be`, it only serves `arm-core-fe`.
When `arm-calendar-fe` loads inside the core shell (via Module Federation), it runs in the **same browser session** but the session cookie is owned by `arm-core-be`.
The calendar remote has no session service of its own — it would have to call `arm-core-be`'s session service as a proxy for its own backend (`arm-calendar-be`), which means:

- `arm-core-be` becomes a **god proxy** for every other service
- Every new repo (calendar, projects, vendors…) adds routes to `arm-core-be`
- Teams cannot deploy independently — calendar release requires touching core
- Token scope bleeds across services with no isolation
- Violates the core purpose of session service: **one session service per client, owned by that client's team**

**Verdict: session service inside `arm-core-be` does not scale past one repo.**

---

## 2. What the research says (2025 standards)

| Authority | Recommendation |
|---|---|
| **IETF** draft-ietf-oauth-browser-based-apps-26 | session service is the preferred architecture for browser-based apps. Token never reaches the browser. session service acts as the confidential OAuth client. |
| **OWASP** | Avoid keeping tokens in the browser. session service with encrypted httpOnly cookies is the minimum bar. |
| **Auth0** | session service is the gold standard. Browser-facing layer handles sessions; backends handle business logic. |
| **Microsoft Azure** | One session service per client type. API Gateway at the edge for cross-cutting concerns — session service sits behind the gateway. |
| **Curity** | Token Handler Pattern — Kong plugin decrypts session cookie and injects Bearer token. Complements session service but does not replace it. |
| **Kong / 2025 industry** | Kong handles north-south (client→gateway) traffic. session service-to-backend calls are east-west — route them direct via service mesh (Kong Mesh / Istio), not back through Kong. |

---

## 3. The Right Architecture for Multiple Repos

```
┌─────────────────────────────────────────────────────────────────────┐
│                        BROWSER                                      │
│                                                                     │
│   arm-core-fe (shell)                                               │
│   ┌──────────────────────────────────────────────────────────┐     │
│   │  calendar remote  │  projects remote  │  vendors remote  │     │
│   └──────────────────────────────────────────────────────────┘     │
│                │                                                     │
│   Only holds:  SESSION_ID cookie (httpOnly, SameSite=Strict)        │
└────────────────│────────────────────────────────────────────────────┘
                 │ HTTPS
                 ▼
┌────────────────────────────────────────────────────────────────────┐
│                     Kong API Gateway          [NORTH-SOUTH EDGE]   │
│                                                                    │
│  • SSL/TLS termination                                             │
│  • Rate limiting, WAF, CORS enforcement                            │
│  • Routes /session/* → arm-session (internal network)                      │
│  • Does NOT validate JWT — session management belongs to arm-session   │
└───────────────────────────┬────────────────────────────────────────┘
                            │ Trusted internal network
                            ▼
┌────────────────────────────────────────────────────────────────────┐
│                  arm-session  (standalone NestJS service)              │
│                                                                    │
│  • Issues SESSION_ID cookie to browser                             │
│  • Stores { accessToken, refreshToken, userId } in Redis           │
│  • Reads SESSION_ID → injects Authorization: Bearer into calls     │
│  • Handles silent token refresh (browser never involved)           │
│  • Calls backends DIRECTLY — no second Kong hop                    │
│  • Single source of truth for ALL frontend apps                    │
└────┬───────────────────────────────────────────────────────────────┘
     │ Direct internal calls — secured by Kong Mesh / Istio mTLS
     │ (EAST-WEST traffic — never routes back through Kong)
     ├─────────────────────────────────────────────┐
     ▼                                             ▼
arm-core-be                               arm-calendar-be
arm-projects-be                           arm-vendors-be
(any future service)                      (any future service)
```

### Key insights

1. **Kong is the single public entry point.** Every request from the browser hits Kong first. Kong handles edge concerns (TLS, rate limiting, WAF) and routes `/session/*` to `arm-session`.

2. **arm-session never calls Kong for backend requests.** Once inside the internal network, `arm-session` calls backend services directly using their internal hostnames. This eliminates a redundant gateway hop, reduces latency, and avoids Kong becoming a bottleneck for all proxied traffic.

3. **East-west security via service mesh.** Internal calls (`arm-session` → `arm-core-be`, etc.) are secured with mutual TLS provided by Kong Mesh or Istio — no need to route them back through Kong.

4. **Backends own their own JWT validation.** Each backend service validates the `Authorization: Bearer` token it receives from `arm-session`. Kong does not need to inspect tokens in the session service path.

---

## 4. Repository Structure

```
arm/
├── core/
│   ├── arm-core-fe/       ← shell app (Module Federation host)
│   └── arm-core-be/       ← core business API (users, auth, registry)
│
├── session/
│   └── arm-session/           ← standalone session service (shared by ALL frontends)
│
├── calendar/
│   ├── arm-calendar-fe/   ← calendar remote (Module Federation)
│   └── arm-calendar-be/   ← calendar business API
│
└── projects/              ← future repo, same pattern
```

`arm-session` is its own repo, its own deployment, its own Docker container. It does **one job**: manage sessions and proxy authenticated requests to backends.

---

## 5. What `arm-session` does — responsibilities

| Responsibility | Detail |
|---|---|
| **Login** | Receives credentials from frontend → calls `arm-core-be /v1/auth/login` directly → gets `{ accessToken, refreshToken }` → stores in Redis → issues `SESSION_ID` cookie → returns `{ userId }` only |
| **Logout** | Deletes Redis session → clears cookie |
| **Session check** | `GET /session/me` → reads Redis → returns `{ userId, isAuthenticated }` |
| **Silent refresh** | Detects access token near expiry (checks JWT `exp`) → calls `arm-core-be /v1/auth/refresh` directly → updates Redis → continues request transparently |
| **Proxy** | `ALL /session/proxy/{service}/*` → reads `SESSION_ID` → looks up `accessToken` from Redis → calls backend service **directly** with `Authorization: Bearer` header (no Kong hop) |
| **Google OAuth** | Intercepts the OAuth callback → creates session → sets cookie → redirects to frontend **with no token in URL** |

---

## 6. Redis Session Schema

```
Key:   session:data:{sessionId}           TTL: 30 days (match refresh token TTL)
Value: {
  accessToken:  "eyJ..."                 (full JWT — never sent to browser)
  refreshToken: "abc123..."              (plain refresh token)
  userId:       "uuid"
  userAgent:    "Mozilla/5.0..."         (fingerprint)
  ipHash:       "sha256(ip)"             (fingerprint)
  createdAt:    "ISO timestamp"
  refreshedAt:  "ISO timestamp"
}
```

---

## 7. Cookie Spec

```
Set-Cookie: SESSION_ID={signedSessionId};
  HttpOnly;
  Secure;
  SameSite=Strict;
  Path=/;
  Max-Age=2592000   (30 days in seconds)
```

`SESSION_ID` value is the UUID **signed** with `SESSION_COOKIE_SECRET` ((secret removed)).
Raw UUID is the Redis key. Signature prevents cookie forgery without Redis lookup.

---

## 8. Environment Variables (`arm-session`)

```env
# Server
PORT=5000
NODE_ENV=development

# Redis (shared with arm-core-be)
REDIS_HOST=localhost
REDIS_PORT=6379

# Session
(secret removed)<32-byte hex — generate: openssl rand -hex 32>
SESSION_TTL_SECONDS=2592000   # 30 days

# Upstream services — direct internal URLs (no Kong hop)
CORE_BE_URL=http://arm-core-be:4000
CALENDAR_BE_URL=http://arm-calendar-be:4001
PROJECTS_BE_URL=http://arm-projects-be:4002

# CORS — only the shell origin may send cookies
ALLOWED_ORIGIN=http://localhost:3000

# Token refresh threshold — refresh if token expires within N seconds
(secret removed)
```

> `KONG_GATEWAY_URL` is removed. `arm-session` calls backends directly using their internal hostnames.
> Kong is configured separately and routes public `/session/*` traffic to `arm-session` — that is Kong's only concern here.

---

## 9. Kong Configuration (edge layer)

Kong is configured with one route that forwards all session service traffic inward:

```
Service:  arm-session
URL:      http://arm-session:5000
Routes:   /session/*

Plugins applied at this service:
  - rate-limiting     (login brute-force protection)
  - cors              (enforce ALLOWED_ORIGIN)
  - ip-restriction    (block known bad actors)
  - bot-detection     (optional)
```

Kong does **not** apply JWT validation on the `/session/*` route — `arm-session` owns session validation via the `SESSION_ID` cookie, and backends own JWT validation for their own routes.

---

## 10. Changes required in existing repos

### `arm-core-be` — minimal changes

| File | Change |
|---|---|
| `auth.controller.ts` | Google OAuth callback → call `arm-session POST /session/internal/create-session` instead of redirecting with token in URL |
| `main.ts` | CORS: allow `arm-session` internal origin (e.g. `http://arm-session:5000`) in addition to frontend |
| No structural changes | All business logic, JWT signing, refresh token DB stays identical |

### `arm-core-fe` — frontend changes

| File | Change |
|---|---|
| `auth-client.ts` | Remove Bearer token injection interceptor. Remove `doTokenRefresh`. Both clients just send `withCredentials: true`. |
| `auth-store.ts` | Remove `accessToken` field. Keep `userId`, `isAuthenticated` only. |
| `auth-service.ts` | All calls point to `arm-session` endpoints via Kong (`/session/login`, `/session/logout`, `/session/me`) |
| `AUTH_ROUTES` | Add `SESSION_ROUTES` constants pointing to `/session/*` |
| `google-callback-page.tsx` | Remove token parsing from URL. Just read `userId`, navigate to `/home`. |
| `calendar-page.tsx` | Remove `accessToken` prop passed to `<Calendar />` |
| `vite.config.ts` | Remove `./auth-store` from federation exposes |
| `protected-route.tsx` | No change — already calls `/auth/me` → update URL to `/session/me` |

### `arm-calendar-fe` (and all future remotes)

| What | Detail |
|---|---|
| All API calls | Use `withCredentials: true` — session cookie is sent automatically |
| No token handling | Remove any accessToken prop usage |
| No auth-store import | Remove `import { useAuthStore } from "core/auth-store"` |
| CORS on calendar backend | Must allow `arm-session` internal origin |

---

## 11. Request flow — step by step

### Login
```
1. User submits login form
2. arm-core-fe  →  POST /session/login  { email, password }
3. Kong         →  routes to arm-session (rate-limit check passes)
4. arm-session      →  POST arm-core-be /v1/auth/login  (direct internal call)
5. arm-core-be  →  returns { accessToken, refreshToken, userId }
6. arm-session      →  stores { accessToken, refreshToken, userId } in Redis
7. arm-session      →  Set-Cookie: SESSION_ID=signed(uuid); HttpOnly; Secure
8. arm-session      →  returns { userId }  ← only this reaches the browser
9. arm-core-fe  →  stores userId in Zustand. Done.
```

### Authenticated API call (calendar fetches events)
```
1. calendar-fe  →  GET /session/proxy/calendar/events   (cookie sent automatically)
2. Kong         →  routes to arm-session (no JWT validation needed here)
3. arm-session      →  reads SESSION_ID from cookie
4. arm-session      →  looks up Redis → gets accessToken
5. arm-session      →  checks JWT exp → token fine
6. arm-session      →  GET arm-calendar-be /events  Authorization: Bearer eyJ...
                   (direct internal call — mTLS via service mesh)
7. arm-calendar-be → validates JWT → returns events
8. arm-session      →  forwards response to browser
   ← browser never saw the token at any step
   ← Kong was only involved at step 2 — one edge hop, not two
```

### Silent token refresh
```
1. arm-session reads session, sees token expires in 45 seconds (< 60s threshold)
2. arm-session → POST arm-core-be /v1/auth/refresh  (direct internal call)
3. arm-core-be → returns new { accessToken, refreshToken }
4. arm-session → updates Redis session with new tokens, resets TTL
5. arm-session → continues proxying the original request with new token
   ← browser never knew a refresh happened
```

### Google OAuth login
```
1. User clicks "Sign in with Google"
2. arm-core-fe → GET arm-core-be /v1/auth/google  (redirects to Google)
3. Google → callback → arm-core-be /v1/auth/google/callback
4. arm-core-be → POST arm-session /session/internal/create-session { accessToken, refreshToken, userId }
   (direct internal call — not via Kong)
5. arm-session → stores session in Redis → returns sessionId
6. arm-core-be → redirect to FRONTEND_URL/auth/callback?userId={userId}
   (NO accessToken in URL — it never left the server)
7. arm-core-fe → reads userId from URL, stores in Zustand, navigates to /home
```

---

## 12. Security properties gained

| Threat | Before (Zustand) | After (session service) |
|---|---|---|
| XSS reads access token | `useAuthStore.getState().accessToken` | Nothing to read — token is in Redis |
| XSS reads refresh token | Not in browser | Not in browser |
| Token in URL (Google OAuth) | Yes — briefly in browser history | Eliminated |
| Compromised remote reads token | Via `core/auth-store` import | Not possible — no token in browser |
| Token theft via devtools | Visible in Zustand devtools | Not present |
| CSRF on session cookie | Mitigated by `SameSite=Strict` | Same — `SameSite=Strict` |
| Session hijacking | N/A | Mitigated by IP+UA fingerprint check |
| session service → backend traffic intercepted | N/A | mTLS via Kong Mesh / Istio (east-west) |
| Double Kong hop / bottleneck | N/A | Eliminated — session service calls backends direct |
| Kong becomes JWT validator | N/A | Kong-free JWT path — each backend validates its own tokens |

---

## 13. Phased implementation plan

### Phase 0 — Prep (no user-facing changes)
- [ ] Create `arm-session` repo and NestJS project
- [ ] Set up Redis connection (reuse same Redis instance)
- [ ] Define `SessionService` interface
- [ ] Design and test Redis session schema
- [ ] Configure Kong route: `/session/*` → `arm-session:5000` with rate-limit + CORS plugins
- [ ] Set up Kong Mesh (or Istio) for internal mTLS between `arm-session` and backends
- [ ] Set up Docker container and CI for `arm-session`

### Phase 1 — session service core (no frontend changes yet)
- [ ] Implement `SessionService` (create/get/refresh/delete)
- [ ] Implement `SessionGuard` (reads SESSION_ID cookie, validates, attaches session)
- [ ] Implement `POST /session/login` endpoint
- [ ] Implement `POST /session/logout` endpoint
- [ ] Implement `GET /session/me` endpoint
- [ ] Implement `ALL /session/proxy/*` proxy handler (direct backend calls — no Kong)
- [ ] Implement silent token refresh inside proxy handler
- [ ] Implement session fingerprinting (IP hash + userAgent)
- [ ] Unit test all session operations
- [ ] Integration test full login → proxy → logout cycle

### Phase 2 — Google OAuth via session service
- [ ] Add `POST /session/internal/create-session` endpoint (called by `arm-core-be` after OAuth)
- [ ] Modify `arm-core-be` Google OAuth callback to call session service directly instead of URL redirect with token
- [ ] Test Google OAuth flow end to end
- [ ] Verify no token appears in URL or browser history

### Phase 3 — Frontend migration (`arm-core-fe`)
- [ ] Add `SESSION_ROUTES` constants
- [ ] Rewrite `auth-client.ts` — remove token injection, remove refresh logic
- [ ] Simplify `auth-store.ts` — remove `accessToken`
- [ ] Update `auth-service.ts` — point to session service endpoints (via Kong)
- [ ] Update `google-callback-page.tsx` — remove token parsing
- [ ] Update `calendar-page.tsx` — remove accessToken prop
- [ ] Update `vite.config.ts` — remove `./auth-store` expose
- [ ] Update `protected-route.tsx` — point `/auth/me` to `/session/me`
- [ ] Full E2E test: login, navigate all protected routes, logout, Google OAuth

### Phase 4 — Remotes migration
- [ ] `arm-calendar-fe`: remove token prop handling, add `withCredentials: true` to all API calls
- [ ] Repeat for each new remote as they are built

### Phase 5 — Hardening
- [ ] Session rotation on refresh (new SESSION_ID issued after each token refresh)
- [ ] Add Content Security Policy headers via `helmet` in `arm-session`
- [ ] Add rate limiting on `/session/login` and `/session/me` (via Kong plugin)
- [ ] Set up Redis session monitoring / alerting
- [ ] Penetration test the session service surface
- [ ] Document runbook for session invalidation (force logout all users)
- [ ] Verify mTLS is enforced on all east-west routes (arm-session ↔ backends)

---

## 14. What stays the same

- `arm-core-be` JWT signing logic — unchanged
- Each backend service validates its own JWT — unchanged
- Refresh token in DB (`refresh_tokens` table) — unchanged
- `SameSite=Strict` httpOnly cookie pattern — extended from refreshToken to SESSION_ID
- `useAuthSync` cross-tab logout — stays, watches `user-id` in localStorage
- All business APIs — no changes needed

---

## 15. Open questions to decide before implementation

| Question | Options |
|---|---|
| Kong Mesh vs Istio for east-west mTLS | Kong Mesh if already using Kong (simpler ops); Istio if cluster-wide mesh is planned |
| Where does `arm-session` live in production? | Same domain as shell behind Kong (recommended) — avoids cross-origin cookie issues |
| What happens when Redis goes down? | All sessions invalidated — users must re-login. Acceptable? Consider Redis Sentinel / Cluster. |
| Session TTL strategy | Fixed 30 days, or sliding (reset TTL on every active request)? |
| Internal session service endpoint auth (`/session/internal/create-session`) | Shared secret between `arm-core-be` and `arm-session`, or mTLS (preferred if service mesh is in place) |
| Service discovery for backend URLs | Static env vars (simple) or Consul/k8s DNS (recommended for production) |

---

## Sources

- [IETF draft-ietf-oauth-browser-based-apps-26](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps)
- [OWASP OAuth2 Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html)
- [Auth0 — Token Storage](https://auth0.com/docs/secure/security-guidance/data-security/token-storage)
- [Auth0 — The Backend For Frontend Pattern](https://auth0.com/blog/the-backend-for-frontend-pattern-bff/)
- [Curity — Token Handler Pattern](https://curity.io/resources/learn/the-token-handler-pattern/)
- [Curity — Kong OAuth Proxy Plugin](https://curity.io/resources/learn/kong-oauth-proxy/)
- [Microsoft Azure — Backends for Frontends Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends)
- [Kong — API Gateway + Service Mesh Architecture](https://developer.konghq.com/mesh/)
- [WunderGraph — 5 Best Practices for BFFs](https://wundergraph.com/blog/5-best-practices-for-backend-for-frontends)
- [FusionAuth — Guide to session service Auth](https://fusionauth.io/blog/backend-for-frontend)
- [microservices.io — Auth in microservice architecture (2025)](https://microservices.io/post/architecture/2025/05/28/microservices-authn-authz-part-2-authentication.html)
