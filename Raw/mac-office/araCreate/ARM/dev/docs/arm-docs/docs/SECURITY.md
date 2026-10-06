# ARM Platform — Security Architecture

> Trust boundaries, the token-never-reaches-the-browser model, per-service auth guard behavior, CORS/rate-limiting/input-validation posture across all four backends, the platform's fixed security-bug history, and an OWASP Top 10 coverage map with honest gaps.
>
> This is a **consolidation**, not new policy — every control described here already exists in code. What didn't exist before this document is a single place that ties root `CLAUDE.md`'s token rule, [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md)'s guard/deny-list detail, and each service's own security-relevant `TASK_SHEET.md` entries into one threat-model-shaped read, cross-checked against the actual guard/bootstrap source rather than trusted from any one doc.
>
> Verified against every backend's `main.ts` and `app.module.ts`, the four auth guards (`core-be/kong-jwt.guard.ts`, `calendar-be/auth.guard.ts`, `admin-be/kong-jwt.guard.ts`, `session.guard.ts`), `kong/kong.yml`, and the `SEC-*`/security-relevant entries across `apps/arm-app-calendar/TASK_SHEET.md`, `core/arm-session/TASK_SHEET.md`, and `AUTH-ARCHITECTURE.md` (2026-08-10).

---

## 1. Threat model summary

```text
┌─ Untrusted ──────────────────────────────────────────────────┐
│  Browser — holds only an opaque, HMAC-signed SESSION_ID       │
│  cookie. Never a JWT, never a raw token, never in a URL        │
│  (residual exception: §10).                                    │
└──────────────────────────┬──────────────────────────────────┘
                            │ SESSION_ID cookie only
┌─ Edge (environment-dependent — see ARCHITECTURE.md §3) ───────┐
│  Local dev: no edge. Kong's X-User-Id fallback path in         │
│  every guard is the ACTIVE code path here — see §3's warning.  │
│  Server: Caddy terminates TLS, Kong is the real API edge —      │
│  JWT-validates and injects X-User-Id before anything            │
│  downstream sees the request.                                   │
└──────────────────────────┬──────────────────────────────────┘
                            │
┌─ Session owner (trust boundary #1) ────────────────────────────┐
│  arm-session — the ONLY thing that ever holds a real JWT outside    │
│  of Redis. Injects Authorization: Bearer on every proxied call. │
└──────────────────────────┬──────────────────────────────────┘
                            │ Authorization: Bearer <JWT>  (+ X-Session-Secret on internal routes)
┌─ Backends (trust boundary #2 — each independently gated) ──────┐
│  core-be · calendar-be · admin-be — each verifies the JWT        │
│  itself; none trusts the session service blindly. See §3 for the guard      │
│  pattern shared (almost) identically across all three.          │
└──────────────────────────┬──────────────────────────────────┘
                            │
┌─ Data (trust boundary #3) ──────────────────────────────────────┐
│  PostgreSQL (arm_core, arm_admin) · MongoDB · Redis — reachable  │
│  only from the internal Docker network, never a host port in    │
│  production (INFRASTRUCTURE.md §8).                              │
└──────────────────────────────────────────────────────────────────┘
```

**Primary attack surface:** the browser-facing edge (session service's `/session/*` routes, the two frontend-facing OAuth flows) and the East-West header-trust boundary between the edge and each backend (§3) — since that boundary is implemented independently four times (core-be, calendar-be, admin-be, and implicitly the session service's own guard), it's the one class of bug (SEC-001, below) that has actually recurred across repos.

---

## 2. Token storage strategy

**Rule, stated once in root `CLAUDE.md` and enforced everywhere:** JWTs and Google/Microsoft OAuth tokens never reach the browser. Not in a response body, not in a URL, not in `localStorage`.

- The browser's only credential is `SESSION_ID` — an HMAC-signed (`SESSION_COOKIE_SECRET`) opaque UUID, not a JWT, verified timing-safely by `SessionGuard`.
- `Session` (access token, refresh token, `userId`, `ipHash`) lives server-side in Redis (`session:data:{uuid}`) — see [SESSION-FLOW.md](SESSION-FLOW.md) for the full lifecycle.
- Calendar Google/Microsoft OAuth tokens are (secret removed) at rest in MongoDB (`GOOGLE_TOKEN_ENCRYPTION_KEY`) — never returned in any calendar-be API response ([`apps/arm-app-calendar/GOOGLE-OAUTH.md`](calendar/GOOGLE-OAUTH.md)).
- **`SEC-005` (fixed):** a dead frontend file (`core-auth.service.ts`) once stored a JWT in `localStorage`, violating this rule. It's gone; the rule is deliberately checked for when new auth code is reviewed, per `AUTH-ARCHITECTURE.md`'s own text.

---

## 3. Auth guard pattern — verified across all four backends

Three backends (`core-be`, `calendar-be`, `admin-be`) implement the **same dual-path guard shape**, independently, in three different files. All three were read directly from source for this document rather than trusted from their own `CLAUDE.md` summaries:

```
if (TRUST_PROXY_HEADERS === 'true' && X-User-Id header present):
    trust the header directly, no signature check
else:
    fall through to full JWT verification (Passport / jwtService.verifyAsync)
```

| Service | Guard file | Header-trust path exists? | Gated by `TRUST_PROXY_HEADERS`? |
|---|---|---|---|
| `core-be` | `src/auth/guards/kong-jwt.guard.ts` | Yes | **Yes** — verified in source |
| `calendar-be` | `src/auth/auth.guard.ts` | Yes | **Yes** — verified in source, fixed 2026-07-02 (`SEC-001`) |
| `admin-be` | `src/auth/guards/kong-jwt.guard.ts` | Yes | **Yes** — verified in source |
| `session` | `session.guard.ts` | No — validates only the `SESSION_ID` cookie, never reads `X-User-Id` | N/A, doesn't participate in this pattern |

**`SEC-001` — fixed everywhere it exists, verified against current source, not just task-sheet claims.** The original bug (`calendar-be`'s `AuthGuard` trusted a client-supplied `X-User-Id` header unconditionally — full horizontal privilege escalation, any authenticated user could impersonate any other user by ID) is closed in all three backends that had the pattern. `apps/arm-app-calendar/TASK_SHEET.md`'s own SEC-001 write-up carries a cross-reference claiming "identical gap in `apps/arm-admin/backend`" — **this cross-reference is stale.** `apps/arm-admin/src/backend` has no `TASK_SHEET.md` at all, and its `kong-jwt.guard.ts` source (read directly for this document) already has the identical `TRUST_PROXY_HEADERS`-gated fix, with an explicit `SEC-001` comment of its own. Whether the cross-reference was wrong when written or the admin fix landed afterward without updating calendar's note, treat calendar's "identical gap" claim as outdated — admin-be is not currently vulnerable to this.

**Why the gate matters — this is the active code path locally, not a theoretical edge case.** Per root `CLAUDE.md`: local dev uses neither Caddy nor Kong. `TRUST_PROXY_HEADERS` is unset/`false` there, so every request falls through to full JWT verification — this is intentional and correct. The risk this gate exists for is a service becoming reachable **without** a real Kong in front while `TRUST_PROXY_HEADERS` is still `true` (a misconfigured server deploy, or a service exposed on a host port it shouldn't be). **`TRUST_PROXY_HEADERS` must never be `true` unless Kong is verified to be the only path to that service.**

**That gate is also what contained `SEC-007` (§11).** Kong spent an unknown period validating JWTs against a literal, repo-committed placeholder string instead of `JWT_ACCESS_SECRET`, so anyone able to read this repo could mint a token for any `sub` and Kong would accept it and inject that `X-User-Id`. It was **not** exploitable end to end purely because `TRUST_PROXY_HEADERS` is set nowhere in `.env`, `.env.example`, or any compose file — so the backends fell through to verifying the JWT themselves and rejected the forgery. The dual-path guard was the only thing standing in the way, and it would have become full impersonation the moment that flag was turned on for its intended purpose. Fixed 2026-08-19 by rendering Kong's config with the real secret ([INFRASTRUCTURE.md](INFRASTRUCTURE.md) §4.1). Worth reading as evidence for the rule above rather than an argument that it no longer matters.

**Kong's side of the contract, confirmed in Kong's config ([INFRASTRUCTURE.md](INFRASTRUCTURE.md) §4):** the `pre-function` plugin decodes the caller's JWT payload itself and sets `X-User-Id` from `payload.sub` — it does not merely relay a client-supplied header. Nothing downstream of Kong strips a client-supplied `X-User-Id` in transit (confirmed: `ProxyService.forward()` only strips `host`/`cookie`/`authorization`/`content-length`; Caddy only adds a response CSP header) — the entire safety of the header-trust path rests on `TRUST_PROXY_HEADERS` being false everywhere Kong isn't genuinely and exclusively in front.

**Verified, still-open gap — `Risk 13` from `AUTH-ARCHITECTURE.md`, restated here because it's squarely a security control, not just a doc gap.** Disabling a user in admin-be revokes their refresh tokens, writes `arm_admin.disabled_users`, and writes `disabled:{userId}` to Redis (900s TTL). **Neither `core-be`'s `KongJwtGuard` nor `calendar-be`'s `AuthGuard` reads that key** — confirmed by reading both guard files in full for this document (neither references `disabled:` or any deny-list service). A disabled user's still-valid access token keeps working against core-be and calendar-be for up to its remaining TTL (up to 15 minutes) after being disabled, not the near-immediate cutoff the deny-list's own TTL implies. This is a real, live gap, not a historical one — unlike `SEC-001`, it has not been fixed.

---

## 4. CORS policy per service

| Service | Mechanism | Allowed origins |
|---|---|---|
| `core-be` | Dynamic origin callback | Prod: `https://arametrics.app`, `https://dev.arametrics.app`, `process.env.FRONTEND_URL`, `process.env.CALENDAR_FRONTEND_URL`. Dev-only (added when `NODE_ENV` isn't `production`/`staging`): `localhost:5173`, `:3000`, `:3001`, `:4001`. Requests with no `Origin` header are allowed through (same-origin/non-browser callers) |
| `calendar-be` | Static array | `env.FRONTEND_URL`, `env.CORE_FRONTEND_URL`, `env.CORS_ORIGIN_1`, `env.CORS_ORIGIN_2` (all env-driven, `.filter(Boolean)`) |
| `admin-be` | Single origin | `config.get('CORS_ORIGIN')` — one value, no array, no dev-mode fallback list |
| `session` | Single origin | `process.env.ALLOWED_ORIGIN \|\| 'http://localhost:3000'` |

Every service sets `credentials: true` — required since the session service's `SESSION_ID` cookie and each backend's cookie-forwarding rely on cross-origin credentialed requests within the platform's own origins. `core-be` is the only service with a genuine allowlist-callback pattern (rejects with an explicit `Error` on mismatch); the other three rely on CORS's own single-origin-string behavior, which is equally strict but has no equivalent explicit-rejection logging.

---

## 5. Security headers (`helmet`)

| Service | `helmet()` applied? |
|---|---|
| `core-be` | Yes |
| `session` | Yes |
| `calendar-be` | Yes |
| **`admin-be`** | **No — verified absent.** Not called in `main.ts`, and `helmet` isn't even listed in `admin-be`'s `package.json` dependencies. |

**Verified gap.** Every other backend in the platform sets `helmet()`'s default header set (`X-Content-Type-Options: nosniff`, `X-Frame-Options`, a default CSP, HSTS when applicable, etc.) — `admin-be` sets none of them. This is worth weighing against the fact that `admin-be` is arguably the most sensitive backend in the platform (user disable/enable, admin role grants, audit log), and it's the one with the least out-of-the-box header hardening. Adding `app.use(helmet())` to `admin-be/src/main.ts` (matching the other three) is a low-effort fix.

---

## 6. Input validation

All four backends use NestJS's `ValidationPipe` globally with `class-validator` DTOs, but not identically:

| Service | `ValidationPipe` config |
|---|---|
| `core-be` | `{ whitelist: true, forbidNonWhitelisted: true, transform: true }` — unknown properties are **rejected** (400), not silently dropped |
| `calendar-be` | `{ whitelist: true, forbidNonWhitelisted: true, transform: true }` — same strict behavior |
| `session` | `{ whitelist: true, transform: true }` — unknown properties are silently **stripped**, not rejected |
| `admin-be` | `{ whitelist: true, transform: true }` — same silent-strip behavior as `session` |

The practical difference: against `core-be`/`calendar-be`, sending an extra unexpected field in a request body is a hard 400. Against `session`/`admin-be`, it's silently dropped and the request proceeds. Neither behavior is wrong (both prevent mass-assignment into unintended fields), but the inconsistency means a client integration test written against one service's tolerant behavior won't necessarily catch the same mistake against a stricter one — worth aligning if these ever get pulled into a shared NestJS boilerplate.

---

## 7. Rate limiting

Two layers: Kong (edge, server only) and each service's own NestJS `ThrottlerModule` (both environments).

### Kong (server edge — see [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §4 for the full route table)

| Route | Limit |
|---|---|
| `core-api` (main route) | 90/min, 2000/hr — **shared across the entire platform**, keyed to the single `arm-platform` JWT consumer, not per-tenant |
| `calendar-api` (main route) | Same 90/min, 2000/hr shared bucket |
| `calendar-api` auth route (`/api/calendar/v1/auth`) | 20/min, 300/hr |
| `calendar-api` webhook route | 30/min, 500/hr |

### Per-service (NestJS `@nestjs/throttler`, Redis-backed storage where configured)

| Service | Config | Tracking key |
|---|---|---|
| `core-be` | `{ ttl: 60000, limit: 120 }`, Redis storage | `AppThrottlerGuard` — keys by `X-User-Id` header when `TRUST_PROXY_HEADERS=true`, else falls back to `req.ip` |
| `calendar-be` | `{ ttl: 60000, limit: 20 }` (`BUG-007`) | `PerUserThrottlerGuard` — **decodes** (does not verify) the caller's JWT `sub` claim to key per-user, falling back to IP if no token is present. Deliberately decode-only: this guard runs as a global `APP_GUARD` ahead of route-level `AuthGuard`, so a forged token just lands in its own bucket and is still rejected downstream by the real auth check |
| `admin-be` | `{ ttl: 60000, limit: 100 }` | Base `ThrottlerGuard` — IP-based only, no per-user keying |
| `session` | Three named buckets: `login` 10/min, `otp` 5/min (mirrors core-be's OTP bucket), `proxy` 300/min | Redis-backed, standard IP-based tracking (per `session.controller.ts`'s route-level `@Throttle()` usage) |

**Note on the shared-bucket Kong limit:** the Kong config's own comment (quoted in [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §4) already flags this — the platform-wide 90/min ceiling is a single bucket for every user going through Kong, not per-tenant. True per-tenant limiting at the Kong layer would need per-tenant JWT claims and one Kong consumer per tenant; tracked as a known limitation, not solved by the current config. The per-service NestJS throttlers (this section) are the actual per-user rate control today.

---

## 8. Secrets management

Not duplicated here — see [ENV-VARS.md](ENV-VARS.md) (canonical list, critical constraints, known example/guide drifts) and [CICD.md](CICD.md) §7 (the `.env` generation flow: GitHub Environment secrets → written directly on the target server over SSH → `docker compose --env-file`). The short version: no secrets are committed to any repo, `.env`/`.arm.conf`/`.arm/` are gitignored everywhere, and `env-check`'s two-tier `REQUIRED_CORE_VARS`/`REQUIRED_OPTIONAL_VARS` split is enforced identically in local dev and CI.

---

## 9. `X-Session-Secret` — internal service-to-service auth

Used on routes that must be called only by another platform service, never by a browser: `POST /session/internal/create-session` (core-be → session service, after OAuth) and `CoreAuthService.getGoogleToken` (calendar-be → core-be). Validated with `timingSafeEqual` (per `arm-session/CLAUDE.md`) — a constant-time comparison specifically to prevent a timing side-channel from leaking the secret one byte at a time. `SESSION_INTERNAL_SECRET` must match wherever it's checked; this is the same shared-secret pattern as `JWT_ACCESS_SECRET`, just for internal S2S calls instead of user tokens.

---

## 10. Tokens in the OAuth URL — status

The documentation inventory that led to this document flagged this as `SEC-24`, an open item to investigate. It's already resolved, documented in [SESSION-FLOW.md](SESSION-FLOW.md): SSO redirects carry `sid=` — a **signed session id**, not a raw JWT — through the callback chain (`/session/oauth/callback?sid={signedId}&state={csrfNonce}`). The residual exposure is that `sid` briefly appears in browser history/server access logs before the cookie is set and the URL param becomes irrelevant — a signed opaque session pointer in a URL is a materially smaller risk than a bearer JWT would be (a leaked `sid` is a session hijack risk bounded by the session's own TTL and revocability, not an unbounded bearer credential), but it's not zero. No further action items beyond what `SESSION-FLOW.md` already tracks.

**OAuth CSRF (a related, separate concern):** the SSO login flow relays a `state` nonce through the callback chain. The calendar Add-Account/Update-Account callbacks separately validate `state` against `session.userId`, redirecting to an error state on mismatch. `SEC-002`/`SEC-003` (both fixed, `apps/arm-app-calendar/TASK_SHEET.md`) record two now-closed gaps here: a missing CSRF check on one OAuth callback, and an account-squatting bug where any authenticated user could claim another user's Google identity mid-link.

---

## 11. Fixed security bugs — cross-repo catalogue

All verified as currently fixed by reading the relevant source, not just the task sheet's own status field:

| ID | Repo | Bug | Severity |
|---|---|---|---|
| `SEC-001` | calendar-be, admin-be | Unconditional trust of client-supplied `X-User-Id` — full horizontal privilege escalation | High |
| `SEC-002` | calendar-be | OAuth "Add Account" callback missing CSRF `state` validation | Medium–High |
| `SEC-003` | calendar-be | Account squatting — any user could claim another user's Google identity mid-link | Medium |
| `SEC-004` | calendar-be | Webhook channel-token check ran after dispatch for target-only calendars | Medium |
| `SEC-005` | core-fe | Dead frontend file stored a JWT in `localStorage`, violating the token-never-reaches-browser rule | Medium (defense-in-depth violation, not directly exploited) |
| `SEC-006` | arm-deploy-make (Kong) | `pre-function` Lua called `require 'cjson'`, which Kong's sandbox forbids — every `/api/*` request carrying a token 500'd, so `X-User-Id` injection never once ran | Medium (availability of Kong's API surface; no data exposure) |
| `SEC-007` | arm-deploy-make (Kong) | Kong's JWT consumer secret was the literal, repo-committed string `${KONG_JWT_SECRET}` — declarative config is not interpolated. Genuine tokens 401'd; tokens signed with the public placeholder were accepted and their `sub` injected | High (latent) — see note below |
| `BFF-18`/`TASK-045` | session service, core-fe | OAuth `state` (CSRF nonce) wasn't relayed end-to-end through `/oauth/callback` | Medium |

---

## 12. OWASP Top 10 (2021) coverage map

| # | Category | Coverage |
|---|---|---|
| A01 | Broken Access Control | Dual-path guards on every backend (§3), `AdminRoleGuard` for admin-only routes, `SEC-001` fixed everywhere — **but Risk 13's deny-list gap (§3) is exactly this category, still open** |
| A02 | Cryptographic Failures | AES-256 for Google/Microsoft tokens at rest, (secret removed) signed session IDs, `timingSafeEqual` comparisons for both session verification and `X-Session-Secret`, no tokens ever transit to the browser (§2) |
| A03 | Injection | TypeORM (parameterized queries) for Postgres, Mongoose (parameterized) for MongoDB — no raw string-concatenated queries found in any guard/service read for this document. `class-validator` DTOs constrain input shape before it reaches a query (§6) |
| A04 | Insecure Design | The session service-owns-tokens architecture itself is the primary design-level control — token exposure is structurally prevented rather than merely policy-prevented. Per-service auth guards independently verify rather than transitively trust the session service (§3) |
| A05 | Security Misconfiguration | `helmet()` on 3/4 backends (§5 — `admin-be` is the gap), `TRUST_PROXY_HEADERS` defaulting to unset/false everywhere, Kong admin API loopback-only on the server ([INFRASTRUCTURE.md](INFRASTRUCTURE.md) §4) |
| A06 | Vulnerable and Outdated Components | Not assessed by this document — no dependency-scanning/SCA tooling was found wired into any CI workflow across the 8 repos ([CICD.md](CICD.md) §3's inventory) beyond `arm-app-calendar/ci.yml`'s `sast` job (static analysis, not the same as dependency CVE scanning) |
| A07 | Identification and Authentication Failures | OTP-only login (no password auth at all — removed platform-wide per `AUTH-ARCHITECTURE.md`), signed opaque session IDs, silent JWT refresh with a Redis-based refresh-lock to prevent concurrent-refresh races |
| A08 | Software and Data Integrity Failures | `npm publish --access public` for `arm-ui-library` with no documented artifact signing/provenance step; deploy pipeline builds fresh from `git clone` on the target server rather than from a signed/attested image ([CICD.md](CICD.md) §1, §6) |
| A09 | Security Logging and Monitoring Failures | Structured logging via `nestjs-pino` across all backends, centralized in Loki/Grafana in the CI/dev observability stack ([INFRASTRUCTURE.md](INFRASTRUCTURE.md) §6). **Correction (2026-08-10, while writing [MONITORING.md](MONITORING.md) §2):** the claim below that no backend configures pino `redact` was trusted from `deploy/arm-deploy-make/CLAUDE.md` without re-verifying against source — that CLAUDE.md text is stale. All 5 backends configure an identical `redact.paths` list (`req.headers.authorization`, `req.headers.cookie`, `*.password`, `*.accessToken`, `*.refreshToken`, `*.token`, `*.secret`), confirmed by reading each `LoggerModule.forRoot()` call directly. This control is in place |
| A10 | Server-Side Request Forgery (SSRF) | Not assessed by this document — no code path found during this pass that fetches an arbitrary user-supplied URL server-side (OAuth redirect URIs are env-configured, not user-supplied); worth a dedicated pass if a feature ever accepts a user-provided URL to fetch |

**A06 and A10 are marked "not assessed," not "no findings"** — this document verified what it read; it did not run a dependency audit or attempt SSRF probing. Treat those two rows as open scope for a follow-up pass, not a clean bill of health.

---

## 13. Known open security issues, with rating

| Issue | Severity | Where documented |
|---|---|---|
| Disabling a user doesn't revoke their already-issued access token for up to 15 minutes — deny-list check missing from `core-be`/`calendar-be` guards | High | `AUTH-ARCHITECTURE.md` Risk 13, restated §3 above |
| `admin-be` has no `helmet()` — the only backend without default security headers, on the most privilege-sensitive service in the platform | Medium | §5 above (new finding, this document) |
| No dependency/CVE scanning (SCA) wired into any of the 8 repos' CI | Low–Medium | §12/A06 above (new finding, this document) — `arm-app-calendar`'s `sast` job is static analysis, not the same control |
| Kong's rate limiting is a single platform-wide bucket, not per-tenant | Low | the Kong config's own comment, restated §7 above |
| `ValidationPipe` strictness (`forbidNonWhitelisted`) is inconsistent across backends | Low | §6 above (new finding, this document) |

---

## 14. Related documents

| Doc | Role |
|---|---|
| [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) | Full login flow, JWT structure, RBAC, multi-account model — this document assumes that one as background and focuses on the security-control cross-section |
| [SESSION-FLOW.md](SESSION-FLOW.md) | Session lifecycle detail behind §2 and §10 |
| [ENV-VARS.md](ENV-VARS.md) | Every secret referenced in §8 |
| [CICD.md](CICD.md) | The deploy pipeline referenced in §8/A08 |
| [INFRASTRUCTURE.md](INFRASTRUCTURE.md) | Kong config (§4/§7's basis), network boundaries (§1's edge layer) |
| [MONITORING.md](MONITORING.md) §2 | Confirms pino `redact` is implemented across all 5 backends (correction to this doc's earlier A09 finding) |
| [`apps/arm-app-calendar/GOOGLE-OAUTH.md`](calendar/GOOGLE-OAUTH.md) | OAuth flow detail behind §10's CSRF discussion |
