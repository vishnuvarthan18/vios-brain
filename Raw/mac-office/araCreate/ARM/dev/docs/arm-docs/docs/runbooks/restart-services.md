# Runbook — Restart Services

How to restart ARM platform services, in every environment `deploy/arm-deploy-make` manages, and how to confirm a restart actually succeeded rather than just "the container is running again." Verified against `deploy/arm-deploy-make`'s `make/*.mk` files and `docker-compose.local.dev.yml` as of 2026-08-11.

---

## 1. Quick reference

| Environment | Restart everything | Restart one service | Compose file |
|---|---|---|---|
| Local dev (hot reload) | `make local-dev-restart` | `make local-dev-rebuild SERVICE=<name>` | `docker-compose.local.dev.yml` |
| Server | `make restart` | `make core-be` / `core-fe` / `session` / `calendar-be` / `calendar-fe` / `notification` (up-only, not a stop+start) | `docker-compose.yml` |
| Infra (Postgres/Mongo/Redis/Kafka) | `make infra-restart` | — | `docker-compose.infra.yml` |

Run every command from the workspace root (`arm/`) or from `deploy/arm-deploy-make/` — the root `Makefile` is a pure delegation layer, both locations are equivalent.

**Production's per-service targets (`core-be`, `session`, etc.) only run `docker compose up -d <service>`, not a stop-first restart** — if the container is already running, this is a no-op; it does not pick up code/config changes the way `local-dev-rebuild` does (which explicitly stops, rebuilds, and restarts). **There is no per-service target for `admin-be`/`admin-fe` in production** (`make/apps.mk`'s individual-service section lists only `core-be`, `core-fe`, `session`, `calendar-be`, `calendar-fe`, `notification`) — restarting just the admin service in production means falling back to `docker compose -f docker-compose.yml restart <admin-service-name>` directly, or a full `make restart`.

---

## 2. Local dev — day-to-day case

**Restart everything:**
```sh
make local-dev-restart   # = local-dev-down + local-dev
```
Tears down and re-creates all 8 app containers. Waits up to 180s, polling health every 10s, printing a per-service status line (`core-be:healthy session:running cal-be:healthy ...`).

**Restart one service** (the usual case — e.g. after a `package.json` change that hot reload can't pick up):
```sh
make local-dev-rebuild SERVICE=calendar-be
```
Valid values: `core-be`, `core-fe`, `session`, `calendar-be`, `calendar-fe`, `notification`, `admin-be`, `admin-fe`. Sequence: stop the named container → rebuild its image → start it → poll health for up to 60s. **All other services keep running** — this is the right tool for "I changed one service's dependencies," not for a routine code-only restart (hot reload / Vite HMR already handles that without any `make` command).

**Startup order is enforced by the health-check dependency cascade**, not by the restart command itself:
```
core-be (healthy)
  ├─→ session (healthy) ──┐
  ├─→ calendar-be (healthy) → calendar-fe (healthy) ──┤
  └─→ admin-be (healthy) → admin-fe (healthy) ─────────┤
                                                         ↓
                                                    core-fe
```
`notification` has no dependencies and starts independently. `core-fe` won't start until `session`, `calendar-fe`, and `admin-fe` all report healthy. Full readiness takes 60–90s from a cold restart.

---

## 3. Verifying a restart actually succeeded

`docker inspect`'s health status (what `local-dev-rebuild`/`local-dev` poll) is only as good as each container's own `healthcheck:` definition. These are not uniform in strength — know which signal you're actually getting:

| Service | Healthcheck hits | Strength |
|---|---|---|
| core-be | `http://localhost:3000/v1/health` | Strong — full check: Postgres ping + Redis ping + service info |
| calendar-be | `http://localhost:4001/v1/health/ready` | Strong — Mongo `db.admin().ping()` + Redis `PONG` (see [ARCHITECTURE.md](../calendar/ARCHITECTURE.md) §8) |
| admin-be | `http://localhost:10001/v1/health` | Strong — checks both Postgres connections (`arm_core` + `arm_admin`) + Redis |
| notification | `http://localhost:4001/v1/health/ready` (own container, same default port as calendar-be — not a conflict, separate containers) | Strong — real Kafka broker round-trip (`KafkaHealthService`) |
| session | Raw TCP connect on port `5000`, no HTTP request at all | **Weak** — confirms the process is listening, not that it booted correctly or that Redis is reachable. A session service that's up but can't reach Redis will still report "healthy." |
| admin-fe | `http://localhost:10000/` | Weak — confirms Vite is serving *something*, not that the app is functional |
| calendar-fe | `http://localhost:3001/calendar/` | Weak — same caveat |
| core-fe | **No healthcheck defined at all** | None — `docker inspect` falls back to container `Status` (`running`), so `core-fe` is reported "up" the instant the process starts, regardless of whether Vite has finished booting |

**Practical implication**: if `session` or `core-fe` show "healthy"/"running" but the app doesn't actually work in the browser, don't trust the healthcheck — go straight to `make local-dev-logs-fe` / `-be` or `docker logs arm-session-local`. The healthcheck told you the port opened, not that the service works.

**Manual verification** (works in any environment, adjust host/port):
```sh
curl -sf http://localhost:4000/v1/health      # core-be — prod/dev port; local dev uses :4000 too
curl -sf http://localhost:5001/session/health     # session service — local dev host port (5001, not 5000 — see port map below)
curl -sf http://localhost:4001/v1/health/ready # calendar-be
curl -sf http://localhost:10001/v1/health     # admin-be
```
`make health` (prod/dev-oriented, not local-dev) runs a similar sweep but against production-style ports (`session` on `:5000`, not `:5001`) and checks calendar-be with a bare connectivity probe (`curl -sf http://localhost:4001` — no path), which is weaker than the `/v1/health/ready` check local-dev's own container healthcheck uses. Prefer `make local-dev-ps` (shows Docker's own health status per container) over `make health` when working locally.

---

## 4. Server

```sh
make restart           # down + up — the server stack (Kong, Caddy, Authelia, observability)
```

This is a **full-stack** restart — every service in `docker-compose.yml` stops, then starts. There's no dependency-ordered health cascade like local dev's (no `local-dev-rebuild`-equivalent exists for the server) — `docker compose up -d` starts everything roughly in parallel, ordered only by each service's own `depends_on`.

`reset-apps` (server only) is a variant that also rebuilds images (`down` + `up -d --build`) with infra left untouched — use this instead of `make restart` when a restart alone won't pick up an image change.

**`ci-deploy-prod` intentionally omits `--remove-orphans`** from its `compose down` calls — adding it would stop infra containers sharing the same network. Don't add it if writing a new deploy/restart target by hand.

---

## 5. What restarting does — and doesn't — affect

- **Sessions survive a session service restart.** Session state lives in Redis (`session:data:{uuid}`), not in the session service process — restarting `session` doesn't log anyone out, mid-request reconnects notwithstanding. See [SESSION-FLOW.md](../SESSION-FLOW.md).
- **Sync/event data survives a calendar-be restart** — it's in MongoDB, not process memory. But **in-flight BullMQ jobs behave differently depending on which queue they're on**: `initial-sync` jobs have no retry/backoff configured at all (see [SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §2) — a job that was actively running when the container stopped is simply lost, not resumed, unless BullMQ's own stalled-job recovery (`lockDuration`) happens to catch it before the container comes back. `sync-teardown`/`webhook-processing`/`webhook-renewal` jobs do have retry policies and will be picked up again by the worker once it restarts.
- **Google webhook channels are external state — a calendar-be restart doesn't touch them.** Channels registered with Google survive the container being restarted; only an in-progress *renewal* job gets interrupted (and per [WEBHOOKS.md](../calendar/WEBHOOKS.md) §4, an interrupted/failed renewal is the one genuinely fragile path — worth checking `dead-letter` queue contents after restarting `calendar-be` if the restart happened to land mid-renewal-window).
- **Postgres/MongoDB/Redis/Kafka are not restarted by any app-level restart command** — `up`/`down`/`restart`/`local-dev-restart` all target only app-service compose files, never `docker-compose.infra.yml`. Restarting infra requires the explicit `make infra-restart`, and doing so drops every service's active DB/Redis/Kafka connections simultaneously — expect a wave of reconnect errors across every app service's logs, not just downtime for the infra containers themselves.

---

## 6. Gateway reload — not the same as a restart

```sh
make gateway-reload
```
Runs `nginx -s reload` inside the `arm-gateway` container — reloads nginx config without dropping active connections. Use this after an nginx config change; use a full service restart for an application code/env-var change. Only applies where the nginx gateway container is actually part of the stack (not local dev, which uses neither Caddy nor Kong/nginx — see root [CLAUDE.md](../../../../CLAUDE.md)'s Kong/edge routing section).

---

## 7. What NOT to do

- **Don't restart infra containers casually.** Every app service loses its DB/Redis/Kafka connection simultaneously; some reconnect automatically (bounded retries — see `apps/arm-app-calendar/docs/failure-modes.md` for calendar-be's specific reconnect/timeout behavior per dependency), others (Mongo, per that doc) crash-loop if the dependency isn't back within the retry window at *boot* time specifically.
- **Don't use `make -j` (parallel mode)** with any of these targets — Make targets in this repo have strict sequential ordering; `arm-deploy-make/CLAUDE.md` explicitly calls this out as a hard rule.
- **Don't skip `infra-wait` after an infra restart** — Kafka topics (`send_welcome_email`, `send_login_otp_email`, `notification.dlq`) are created by `infra-wait`, not automatically by Kafka itself. Restarting infra without following up with a wait step leaves those topics missing until something creates them.
- **Don't assume a "healthy" container means a working service** — see §3's strength table. `session` and the three frontends give weaker signal than the three backends with real dependency-check health endpoints.

---

## Related documents

- [deploy-rollback.md](deploy-rollback.md) — the related but distinct case of rolling back to a *previous* deploy, not just restarting the current one
- [INFRASTRUCTURE.md](../INFRASTRUCTURE.md) — full compose-file inventory, Kong/Caddy routing per environment
- [calendar/ARCHITECTURE.md](../calendar/ARCHITECTURE.md) §8 — calendar-be's liveness/readiness split in detail
- [calendar/SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §2 — BullMQ queue retry configuration referenced in §5 above
- [calendar/WEBHOOKS.md](../calendar/WEBHOOKS.md) §4 — the renewal-failure gap referenced in §5 above
- [REDIS-ARCHITECTURE.md](../REDIS-ARCHITECTURE.md) — what's stored in Redis and what a restart does/doesn't lose
- `apps/arm-app-calendar/docs/failure-modes.md` — per-dependency reconnect/timeout behavior on outage, relevant to what a restart looks like from the affected service's side
