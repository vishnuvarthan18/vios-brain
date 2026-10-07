# ARM Platform — Kong API Gateway Configuration

Kong is edge-only, DB-less, and **not in the default local dev path** — local dev runs neither Kong nor Caddy, per root [CLAUDE.md](../../../CLAUDE.md)'s Kong/edge routing section and [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §3. It *can* be run locally on demand with `make edge-up`, which puts Kong and Caddy in front of the running local stack using the same config the server mounts — see [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md). This document covers every route, plugin, and consumer in `deploy/arm-deploy-make/kong/kong.yml.tmpl` (the platform's entire Kong config, rendered to `generated/kong.yml` at deploy time) and what each one is actually for — verified directly against that file, 2026-08-11, **re-verified 2026-08-19** (the `admin-api` route in §1 and the metrics-listener split in §6 are both new since the first pass).

**Kong's routes are a separate, direct-to-backend API surface — not the browser's path.** The browser never talks to Kong directly; it goes through the session service proxy (`/session/proxy/...`, see [SESSION-FLOW.md](SESSION-FLOW.md)). Kong instead fronts `core-be`/`calendar-be` at `/api/core`/`/api/calendar` for callers presenting a raw JWT directly — **a repo-wide search found no code anywhere in this workspace that actually calls either path.** Treat these routes as a configured-but-currently-unconsumed external API surface (a plausible third-party-integration or mobile-client entry point) rather than something any existing frontend depends on. If you're troubleshooting a browser-facing auth or routing issue, this document is very likely not where the problem lives — check [SESSION-FLOW.md](SESSION-FLOW.md) and [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) instead.

---

## 1. Route inventory

| Route | Path | Service / upstream | `strip_path` | JWT-protected at Kong? |
|---|---|---|---|---|
| `core-api-route` | `/api/core` | `core-backend:3000` | `true` | Yes |
| `calendar-api-route` | `/api/calendar` | `calendar-backend:4001` | `true` | Yes |
| `calendar-auth-public-route` | `/api/calendar/v1/auth` | `calendar-backend:4001` | `false` (custom rewrite instead — §4) | **No** |
| `calendar-webhook-route` | `/api/calendar/events/notifications` | `calendar-backend:4001` | `false` (custom rewrite instead — §4) | **No** |
| `admin-api-route` | `/api/admin` | `admin-backend:10001` | `true` | Yes |

`admin-api` is new — it was added when the admin app gained real production compose services, so an earlier finding that `kong.yml` had no admin route is no longer true ([INFRASTRUCTURE.md](INFRASTRUCTURE.md) §4). It mirrors `core-api` exactly: `jwt` + the `pre-function` `X-User-Id` injection (§3) and the same 90/min–2000/hr bucket (§5).

The two "public" calendar routes deliberately have no `jwt` plugin attached (the `jwt` plugin block for `calendar-api` is explicitly scoped with `route: calendar-api-route`, so it doesn't apply to the other two routes under the same service) — they exist for calendar-be's own OAuth callback and Google webhook-intake endpoints, both of which must be reachable without a session by their nature (an OAuth provider's redirect, and Google's push-notification service — see [GOOGLE-OAUTH.md](calendar/GOOGLE-OAUTH.md) and [WEBHOOKS.md](calendar/WEBHOOKS.md) §3).

---

## 2. JWT validation happens at Kong itself

**Correction to an easy assumption**: Kong does not just pass tokens through — it fully validates them. The `jwt` plugin, scoped to `core-api-route` and `calendar-api-route`, verifies the token's signature against a registered consumer secret and checks the `exp` claim (`claims_to_verify: [exp]`) before the request ever reaches the backend.

**Consumer setup**: a single consumer, `arm-platform`, with one JWT credential — `key: arm-platform` (matched against the token's `iss` claim, via `key_claim_name: iss`), `secret: (secret removed) `algorithm: HS256`.

The secret is a **template placeholder**, substituted from `JWT_ACCESS_SECRET` when `make generate-kong-config` renders `kong.yml.tmpl` to `generated/kong.yml`. It therefore cannot drift from what `core-be` signs with, which is why this is no longer phrased as a constraint you must remember to honour.

> **This was broken until 2026-08-19, and instructively so.** The file shipped as `kong/kong.yml` with `secret: (secret removed) and **Kong does not interpolate anything in declarative config** — not `${VAR}`, and not `{vault://env/...}` either (`jwt_secrets.secret` is not a referenceable field in DB-less mode; both verified against `kong:3.7-ubuntu`). So that placeholder text *was* the HMAC key: every genuine `core-be` token was rejected 401, while a token signed with the literal string — public, committed to the repo — was accepted. The rename to `.tmpl` is deliberate; a file called `kong.yml` that looked like live config is how this survived. See [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §4.1.

There is only **one** consumer for the entire platform — every valid caller authenticates as the same `arm-platform` identity at the Kong layer. This is directly why the rate-limiting config (§5) is a platform-wide bucket, not a per-caller one — the file's own comment on this is explicit and worth repeating here.

---

## 3. `X-User-Id` injection — the concrete mechanism behind `TRUST_PROXY_HEADERS`

[AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) §3 documents that every backend guard trusts an `X-User-Id` header when `TRUST_PROXY_HEADERS=true`, "because Kong verified the JWT at the edge" — this document is where that header actually gets set. Both protected routes carry an identical `pre-function` plugin (custom Lua, `access` phase) that:

1. Reads the `Authorization: Bearer <token>` header.
2. Splits the JWT into its three dot-separated parts.
3. Base64url-decodes the payload (part 2) and JSON-parses it.
4. If `payload.sub` exists, sets `X-User-Id` to it via `kong.service.request.set_header`.

> **This Lua did not run at all until 2026-08-19.** Kong executes `pre-function` under `untrusted_lua = sandbox`, which forbids `require` — so `require("cjson")` below threw on every request that carried an `Authorization` header, and Kong returned **500** rather than proxying. Requests with no token still 401'd, because of the `if auth then` guard. Fixed with `KONG_UNTRUSTED_LUA_SANDBOX_REQUIRES: cjson` on Kong's environment in both compose files. The reasoning below about *why the unverified decode is safe* was independently confirmed once the Lua could run: with a tampered signature the upstream received nothing at all. See [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §4.1.

**This Lua script does no signature verification of its own** — it trusts the payload blindly. That's safe despite (not because of) plugin ordering: `pre-function`'s priority (1,000,000) actually runs *before* `jwt`'s (1,450) in Kong's fixed per-plugin-type priority order, so this Lua genuinely executes first, on every request, staging `X-User-Id` from a completely unverified payload. What actually prevents a bypass is that the `jwt` plugin, scoped to the same route, calls `kong.response.exit(401, ...)` on a bad/missing signature — halting the access phase immediately and never proxying to the upstream — so the staged header is discarded along with the rejected request, not because it was never set. **This is still a fragile pattern to extend carelessly**: if a future route ever added this same pre-function without also attaching the `jwt` plugin to it, it would inject an `X-User-Id` header from a completely unverified token — a direct (secret removed) bypass, just constructed differently. Anyone adding a new protected route to this config must pair both plugins together explicitly; nothing in Kong's config format enforces that pairing automatically.

---

## 4. Path rewriting on the two public routes

Since `strip_path: false` on both public routes, a custom `pre-function` handles the path rewrite instead of Kong's built-in stripping:

- `calendar-auth-public-route`: strips the literal prefix `/api/calendar/v1` from the incoming path via a Lua `gsub`, forwarding whatever remains (e.g. `/api/calendar/v1/auth/google/callback` → `/auth/google/callback` reaching `calendar-backend:4001`).
- `calendar-webhook-route`: hardcodes the outgoing path to exactly `/v1/events/notifications` regardless of the incoming path.

**Verified correction to [WEBHOOKS.md](calendar/WEBHOOKS.md)**: that document previously described the webhook intake route as `POST /events/notifications`, omitting calendar-be's global `v1` prefix (`app.setGlobalPrefix('v1')` in `main.ts`) — the real, live route is `POST /v1/events/notifications`, exactly matching what this Kong config hardcodes. `WEBHOOKS.md` has been corrected to match. This also surfaced a related, narrower bug worth knowing about: calendar-be's own `common/config/env.ts` hardcodes `NOTIFICATION_WEBHOOK_URL`'s *application-level fallback default* as `http://localhost:4001/events/notifications` — **also missing the `/v1/` prefix**, inconsistent with the real route. In every compose file this env var is actually either explicitly set with the correct `/v1/` path (`docker-compose.local.dev.yml`) or required with no fallback at all (`docker-compose.dev.yml`/`docker-compose.yml`, which would fail `env.ts`'s own startup validation if left unset) — so this buggy default is not exercised in any of the three shipped compose configurations today. It would only bite calendar-be run standalone outside all three compose files with the env var completely unset, registering Google webhooks against a URL that 404s. Narrow, but a real latent bug — worth a one-line fix in `env.ts` (`http://localhost:4001/v1/events/notifications`) whenever that file is next touched.

---

## 5. Rate limiting

| Route | `minute` | `hour` |
|---|---|---|
| `core-api-route` | 90 | 2000 |
| `calendar-api-route` | 90 | 2000 |
| `calendar-auth-public-route` | 20 | 300 |
| `calendar-webhook-route` | 30 | 500 |

`policy: local` (in-memory per-Kong-node counting, not shared across multiple Kong instances — fine for this platform's current single-node Kong deployment, would need `policy: redis` to stay accurate if Kong is ever horizontally scaled) and `fault_tolerant: true` (rate-limiting failures don't block requests) on every route. As the config's own comment states plainly: because there is only one consumer (`arm-platform`, §2) for the whole platform, **these buckets are a platform-wide ceiling shared by every caller, not per-tenant limits** — true per-tenant rate limiting would require core-be to issue per-tenant JWT claims and Kong to register one consumer per tenant, which is explicitly out of scope for the current config.

---

## 6. Prometheus plugin — already live, not deferred

A global `plugins: [{ name: prometheus }]` block at the end of the config exposes Kong's own metrics, and each environment's Prometheus config has a matching `kong` scrape job — both halves confirmed present and wired together.

**The port has since moved, and the reason matters.** Kong originally served metrics from its Admin API port (`8001`), which meant Prometheus could only scrape it if the admin API was reachable over the Docker network — and Kong's DB-less admin API has no authentication of its own (RBAC is Kong Enterprise-only), so a `POST /config` from any container on that network could replace Kong's entire routing and plugin configuration. All three environments now run a dedicated status listener instead:

```yaml
KONG_ADMIN_LISTEN: "127.0.0.1:8001"   # loopback only, never host-published
KONG_STATUS_LISTEN: "0.0.0.0:8100"    # /status and /metrics only
```

Prometheus scrapes Kong on `:8100`, which exposes only `/status` and `/metrics` and never the admin API. Any reference to scraping Kong on `8001` predates this split. **This corrects `deploy/arm-deploy-make/CLAUDE.md`'s Observability section**, which still describes "Kong's `prometheus` plugin" as explicitly deferred to "Phase B" — a fourth stale claim in that section, alongside the three [MONITORING.md](MONITORING.md) already corrected (Tempo, Grafana alerting, pino `redact`). The plan document that proposed this work (`docs/superpowers/plans/2026-08-08-tempo-tracing-kong-metrics.md`) has, in fact, already been fully carried out for its Kong-metrics half.

---

## 7. Config reload — no dedicated tooling exists yet

Kong runs `KONG_DECLARATIVE_CONFIG: /etc/kong/kong.yml`, DB-less, with `generated/kong.yml` mounted read-only into the container. **There is currently no Make target or script that reloads Kong's config specifically** — `make gateway-reload` (`deploy/arm-deploy-make/CLAUDE.md`) targets the separate, unrelated `arm-gateway` nginx container (`nginx -s reload`), which per [INFRASTRUCTURE.md](INFRASTRUCTURE.md)'s own known-pitfalls list is largely dead/unreferenced config predating the session service and admin app — don't confuse the two "gateway" concepts. Applying a config change today means **re-rendering, then restarting** the Kong container: `make generate-kong-config && docker compose restart kong` against whichever compose file is in play. Editing `generated/kong.yml` directly is pointless — the next render overwrites it; edit `kong/kong.yml.tmpl`. **The Admin API `POST /config` alternative is no longer available from another container** — the admin API is now bound to `127.0.0.1:8001` in every environment (§6), so that path requires a shell on the host itself. This is exactly the gap **RB-08 (Kong Configuration Reload)** is meant to close — still unwritten as of this document.

---

## Related documents

- [SESSION-FLOW.md](SESSION-FLOW.md) — the browser's actual request path, which does not go through Kong at all
- [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) §3 — the `TRUST_PROXY_HEADERS`/`X-User-Id` guard pattern this document's §3 supplies the concrete Kong-side mechanism for
- [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §3, §4 — edge routing by environment (Caddy terminates TLS in front of Kong on the server), and Kong's admin/status listener split
- [MONITORING.md](MONITORING.md) — the other 3 corrections to `deploy/arm-deploy-make/CLAUDE.md`'s Observability section, alongside this document's 4th
- [calendar/WEBHOOKS.md](calendar/WEBHOOKS.md) §3 — the webhook intake route this document's §4 corrected the exact path for
- [SECURITY.md](SECURITY.md) — broader threat-model context for the JWT/consumer model in §2
