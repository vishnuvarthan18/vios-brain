# Troubleshooting — Authentication Failures

Symptom-first triage for the platform's auth path. This document doesn't re-explain the architecture — [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) and [SESSION-FLOW.md](../SESSION-FLOW.md) already do that thoroughly — it maps observed symptoms to the specific mechanism most likely responsible, with the diagnostic steps to confirm.

There are two auth clocks on this platform that fail differently — **platform login (JWT/session)** and **linked Google/Microsoft calendar-account tokens** (see [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §6). Confirm which one you're actually looking at before diagnosing — "auth is broken" reported by a user could be either, and the fixes are unrelated.

---

## 1. Quick triage table

| Symptom | Most likely cause | Jump to |
|---|---|---|
| Logged out immediately after login, or every page load | `SESSION_ID` cookie not being set/sent | §2 |
| `401` from a proxied API call shortly after a working login | Silent refresh failed, stale token forwarded | §3 |
| Email OTP never arrives | Kafka/notification-service dependency down | §4 |
| SSO redirect loop, or lands on an error page after Google/Microsoft consent | CSRF `state` mismatch, or `X-Session-Secret` misconfigured | §5 |
| Admin disabled a user, but they can still use the app | **Expected, not a bug** — deny-list gap | §6 |
| `401` from calendar-be/admin-be/core-be specifically, core-fe still shows logged in | `TRUST_PROXY_HEADERS`/`JWT_ACCESS_SECRET` mismatch between services | §7 |
| Works in local dev, breaks on the server (or vice versa) | Kong/`X-User-Id` trust path misconfigured | §8 |
| "Reconnect your calendar" banner appears unexpectedly | Google/Microsoft account-token expiry — unrelated to platform login | §9 |

---

## 2. `SESSION_ID` cookie missing or not persisting

The session service is the only thing that ever sets a cookie on the browser ([AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §1), so this symptom always traces back to the session service's cookie-setting or the browser rejecting it.

**Check first — cookie attributes** ([SESSION-FLOW.md](../SESSION-FLOW.md) §3): `httpOnly`, `sameSite: 'lax'`, `secure` true only when `NODE_ENV` is `production`/`staging`, no `domain` set (host-only). The most common local-dev failure mode: if `secure: true` ever ships to a plain-HTTP local environment, the browser silently drops the cookie — no error surfaces, the request just looks like login never happened. Confirm `NODE_ENV` on the session service matches the environment's actual protocol.

**Check second — CORS** ([SESSION-FLOW.md](../SESSION-FLOW.md) §9): `ALLOWED_ORIGIN` is a **single** origin, not a list. If the frontend's actual origin doesn't exactly match `ALLOWED_ORIGIN`, credentialed requests (`withCredentials`/`credentials: 'include'`) fail CORS preflight — check the browser console for a CORS error specifically, not just a generic network failure. This is an easy trap when `core-fe` is accessed via a hostname other than exactly what `ALLOWED_ORIGIN` expects (e.g. `127.0.0.1:3000` vs `localhost:3000` — browsers treat these as different origins).

**Check third — popup-specific cookie issues**: the calendar Add-Account OAuth flow runs in a popup ([GOOGLE-OAUTH.md](../calendar/GOOGLE-OAUTH.md)) — some browser privacy settings (third-party cookie blocking, Safari ITP) can behave differently for popup-window cookies than main-window ones. If session issues are specifically reported *after* an OAuth popup flow, check that document's §6 (BroadcastChannel completion) and its local-dev origin gotcha before assuming this troubleshooting doc's general cookie guidance is the whole story.

**Diagnostic**: `curl -i` the login endpoint and inspect `Set-Cookie` directly — confirms whether the session service is even attempting to set the cookie, isolating "session service never set it" from "browser rejected it."

---

## 3. `401` shortly after a working login

The session service refreshes the access token silently before every proxied call if it's within `TOKEN_REFRESH_THRESHOLD_SECONDS` (60s default) of expiring ([SESSION-FLOW.md](../SESSION-FLOW.md) §6). This is a **soft-fail** design: if the refresh call itself fails, the session service logs the error and forwards the **stale** token anyway rather than blocking the request — the backend then returns `401` on that stale token, which is what the user/caller actually sees. The 401 is the visible symptom; the root cause is one layer up, in the refresh attempt.

**Check**: session service logs around the failing request's timestamp for a refresh-failure log line. Common underlying causes:
- **Refresh token family revoked** — [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §2: refresh tokens are grouped by `familyId`; reusing an already-revoked token kills the whole family (the stolen-token defense). If a user has multiple tabs/devices and one triggered a refresh that raced another, this can produce a false-positive family revocation — worth checking `refresh_tokens` for that user's family state directly in `arm_core`.
- **`core-be`'s `/v1/auth/refresh` itself down/erroring** — check core-be's own health and logs, not just the session service's.
- **Clock skew** between the session service and core-be — JWT `exp` decoding happens client-side (session service) without signature verification at the pre-check step; a sufficiently skewed clock could miscalculate the refresh threshold. Rare, but worth ruling out if refresh failures correlate with a specific host.

**Note**: calendar-be has its *own* independent refresh path (§3 of [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md)) — on a `401` with a `refresh_token` cookie/header present, it calls core-be's refresh endpoint directly and retries, as a second chance alongside the session service's own silent refresh. If calendar-be-proxied calls recover from a 401 that other services don't, this is why — it's not inconsistent behavior, it's an intentionally asymmetric extra safety net specific to that one service.

---

## 4. Email OTP never arrives

This is not actually an auth-guard problem — it's a dependency chain: `POST /v1/auth/login/email/request-otp` generates the code and publishes it to Kafka; `arm-service-notification` consumes it and sends the email via SMTP. A failure anywhere in that chain looks like "the user never got their code," not like an auth error.

**Check, in order**: Kafka broker health, `arm-service-notification`'s consumer (confirm it's actually consuming, not just alive — see [WEBHOOKS.md](../calendar/WEBHOOKS.md)'s §7 for a parallel example of "the container is up" not meaning "the thing it's supposed to do is happening"), SMTP credentials/connectivity. This is entirely outside the auth guard/JWT system — don't spend time in `core-be`'s auth code for this symptom.

**Also check**: the account might be SSO-provisioned. Per [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §1, an SSO-provisioned account is **rejected** at the OTP-request endpoint by design — "no email arrived" for such an account isn't a delivery failure, it's the endpoint correctly refusing to issue a fallback code around SSO/conditional-access policy. Confirm the user's account provisioning method before assuming this is a delivery bug.

---

## 5. SSO redirect loop, or error after Google/Microsoft consent

- **CSRF `state` mismatch**: the SSO flow relays a `state` nonce through the callback chain ([AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §7); a mismatch redirects to an error state rather than completing login. If this happens consistently (not just occasionally), check for a proxy/load-balancer stripping or rewriting query params on the callback URL.
- **`X-Session-Secret` mismatch**: core-be's SSO callback calls `POST /session/internal/create-session` server-to-server with `X-Session-Secret: (secret removed) ([AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §3, §7 in [SESSION-FLOW.md](../SESSION-FLOW.md)). If `SESSION_INTERNAL_SECRET` differs between core-be's and the session service's env, this call fails with a `401`/`403` from the session service side, and the user sees a generic error at the point where the redirect should have handed off to the cookie-setting step. Confirm the secret matches exactly across both services' `.env`.
- **This is a distinct failure class from the calendar Add-Account popup flow** ([GOOGLE-OAUTH.md](../calendar/GOOGLE-OAUTH.md)) — that flow uses `SessionGuard` + session JWT, not `X-Session-Secret`, and has its own dedicated troubleshooting content (popup blocking, BroadcastChannel, local-dev origin gotcha) in that document. Don't conflate platform-login SSO with the calendar-linking OAuth flow — they share "OAuth" in the name but are different mechanisms end to end.

---

## 6. User was disabled by an admin but can still use the app — expected, not a bug

**This is a documented, currently-open platform gap, not a troubleshooting mystery.** Per [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §4: disabling a user revokes their refresh tokens (blocking a *new* access token) and writes a Redis deny-list key — but **no backend guard actually checks that deny-list key**. A disabled user's already-issued access token keeps working against core-be and calendar-be for up to its remaining TTL (15 minutes by default), not immediately as the deny-list's own TTL implies.

**Do not spend time debugging this as if it were a defect in the disable flow itself** — the disable flow is working exactly as coded; the gap is that nothing downstream enforces it in real time. If a disabled user needs to be cut off *immediately* rather than within 15 minutes, that requires an engineering fix (wire the deny-list check into `KongJwtGuard`/`AuthGuard`, or shorten the access-token TTL) — not something resolvable by any runbook step. Escalate as a feature gap, not an incident, unless the 15-minute window itself is the active incident.

---

## 7. `401` from one specific backend, core-fe still shows logged in

Every backend except the session service verifies the same JWT independently ([AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §3) — a service-specific 401 while the session otherwise looks healthy points at that service's own configuration, not the session itself.

**Check**: `JWT_ACCESS_SECRET` must be byte-for-byte identical between `core-be` and the failing service (`calendar-be`, `admin-be`) — a drifted secret makes every token that service receives fail signature verification, regardless of how valid the session is everywhere else. This is exactly the constraint the root [CLAUDE.md](../../../../CLAUDE.md) flags under "Environment Variables — Critical Constraints."

**Also check `TRUST_PROXY_HEADERS`** on that specific service — if it's `true` but Kong isn't actually in front of that service in this environment (or is misconfigured to not inject `X-User-Id`), the guard's Kong-path branch is taken but finds no header, and behavior depends on the specific guard's fallthrough logic — worth confirming this flag's value matches whether Kong is genuinely fronting that service in the current environment (see §8).

---

## 8. Works in local dev, breaks on the server (or vice versa)

Local dev has **no Kong in the request path at all** — every guard's Kong-trust branch is dead code locally, and the JWT-verification fallback is the only path actually exercised ([AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §3, root [CLAUDE.md](../../../../CLAUDE.md)). The server *does* run behind Kong, with `TRUST_PROXY_HEADERS=true` expected to be set and Kong expected to inject `X-User-Id` after verifying the JWT at the edge.

**This asymmetry is the single most common source of "works here, not there" auth bugs on this platform.** If something depends on Kong actually being present and correctly configured (header injection, routing rules), it will pass silently in local dev (because the guard falls through to direct JWT verification, which still works) and only fail in an environment where Kong's behavior differs from expected — meaning a Kong misconfiguration on the server can hide behind a guard that "still works" via its fallback path, rather than failing loudly. When triaging an environment-specific auth failure, check Kong's actual routing/header-injection behavior directly (not just whether the app-level guard passed), and confirm `TRUST_PROXY_HEADERS` is set correctly for that specific environment before assuming the guard code itself is at fault.

---

## 9. "Reconnect your calendar" appears unexpectedly

This is the **Google/Microsoft calendar-account token clock**, not the platform login session — a completely separate expiry mechanism ([AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §6). calendar-be's `Account.isAuthenticated` self-heals to `false` on a `401` from a Google/Microsoft API call — note this only actually triggers on a real `401`, not a `403` (see [ARCHITECTURE.md](../calendar/ARCHITECTURE.md) §11.1, a verified drift from what the older docs claimed) — surfacing a reconnect prompt to the user. This is independent of whether the user's platform login is fine; a perfectly valid session can still show this banner if the *linked calendar account's* Google/Microsoft grant expired or was revoked externally (e.g. the user revoked app access from their Google account settings). Reconnecting uses the Update Account flow, not a platform re-login — see [GOOGLE-OAUTH.md](../calendar/GOOGLE-OAUTH.md) §8.

---

## 10. Diagnostic commands

```sh
# Confirm the session service is issuing a cookie at all
curl -i -X POST http://localhost:5001/session/login/email/verify -d '{...}'

# Confirm core-be's own health independent of the session service
curl -sf http://localhost:4000/v1/health

# Inspect a live session directly in Redis (bypasses the app entirely)
redis-cli GET "session:data:<uuid>"
```

See [restart-services.md](../runbooks/restart-services.md) §3 for the full per-service health-endpoint reference if the issue turns out to be a downstream service being unhealthy rather than an auth-logic problem specifically.

---

## Related documents

- [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) — the architecture this troubleshooting guide maps symptoms onto
- [SESSION-FLOW.md](../SESSION-FLOW.md) — session lifecycle, Redis schema, silent refresh mechanics in full
- [calendar/GOOGLE-OAUTH.md](../calendar/GOOGLE-OAUTH.md) — Add Account / Reconnect flow-specific troubleshooting (popup, BroadcastChannel, CSRF)
- [runbooks/restart-services.md](../runbooks/restart-services.md) — health-check reference if the root cause is a downstream dependency, not auth logic itself
- [SECURITY.md](../SECURITY.md) — the broader OWASP/threat-model context behind several of the constraints referenced above
