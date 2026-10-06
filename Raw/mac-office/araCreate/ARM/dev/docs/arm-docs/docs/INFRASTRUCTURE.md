# ARM Platform — Infrastructure & Deployment Guide

> What runs where, how it connects, and how to stand up or tear down each of the two environments: local and server. All infrastructure orchestration lives in `deploy/arm-deploy-make/` — this doc is the reference map through it.
>
> [ARCHITECTURE.md](ARCHITECTURE.md) §3 already resolved the Caddy-vs-Kong local-dev question — this document goes deeper: every compose file's actual service list and port mappings, Kong's real config, why `nginx.dev.conf` is dead code, the Redis Sentinel HA topology, and what backup tooling already exists versus what the inventory assumed was missing.
>
> **Read [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md) first if you are about to edit a service's compose definition.** No app service block is hand-written any more — every one is generated from its repo's own descriptor into `generated/` and `generated/prod/`, both gitignored and overwritten on every run.
>
> Verified against every `docker-compose*.yml` in `deploy/arm-deploy-make/`, `kong/kong.yml.tmpl`, `Caddyfile.prod`, `orchestrator/templates/**`, `generated/**`, `observability/`, `make/{backup,infra,ci,descriptor}.mk`, `scripts/backup-*.sh`, and `deploy/arm-deploy-make/CLAUDE.md` (**re-verified 2026-08-19**; the previous pass was 2026-08-10 and several of its findings have since been closed — noted inline where that happened).

---

## 1. Infrastructure component inventory

| Component | Role | Where it runs |
|---|---|---|
| **PostgreSQL 16** | `arm_core` (core-be), `arm_admin` (admin-be) | Every environment — one Postgres instance, two databases |
| **MongoDB 7** | `arm-calendar` (calendar-be) | Every environment |
| **Redis 7** | sessions, calendar-be BullMQ queues | Single instance (dev/local), **Sentinel HA** available for prod (§5) |
| **Kafka** (KRaft, no ZooKeeper) | core-be → notification-service event bus | 1 broker (dev/local), 3-broker cluster (`docker-compose.infra.prod.yml`) |
| **Kong 3.7** | API-gateway edge: JWT validation + `X-User-Id` injection, rate limiting, routing to core-be/calendar-be/admin-be | Server only (§4) |
| **Caddy** | Browser-facing reverse proxy, TLS termination, CSP headers | Server only (`Caddyfile.prod`). Not in local dev (§3) |
| **Authelia** | `forward_auth` gate in front of Grafana | Server only |
| **Prometheus / Grafana / Loki / Alloy / Tempo / cAdvisor / node-exporter / postgres-exporter / mongodb-exporter / redis-exporter** | Observability stack (10 containers) | Server only — `observability/prometheus.prod.yml` and `observability/alloy/config.prod.alloy` (§6) |

**Not part of this stack, despite appearing in generic infra-doc templates:** there is no MinIO anywhere in this workspace. **Verdaccio was removed** — `make/local-dev.mk`'s header comment documents the removal explicitly (a stale `.npmrc` pointing at `localhost:4873` is now auto-cleaned by `clone-repos`/`update-repos`), and `docs/guides/npm-publish.md` confirms `@aracreate/test-arm-ui` now ships via plain `npm publish --access public`, not an internal registry. If you're looking at this doc because `DOC-P25` (Verdaccio Internal Registry Setup) is still listed as a gap in `docs/meta/documentation-inventory.md`, that entry is stale — there is no internal registry to document anymore.

---

## 2. Docker Compose files — what each one is actually for

| File | Used for | App service blocks | Other services |
|---|---|---|---|
| `docker-compose.infra.yml` | Postgres, MongoDB, Redis, Kafka — single-instance, always-persistent. Started by `infra-up` / `make arm`. | n/a | postgres, mongodb, redis, kafka |
| `docker-compose.local.dev.yml` | **The everyday dev file.** All 8 app services with hot reload, no Kong/Caddy. Started by `make local-dev` / `make arm-run`. | **Generated** — an `include:` of `generated/docker-compose.<name>.yml` ×8 | none (networks only) |
| `docker-compose.yml` | **The server stack.** Caddy edge → Kong → backends, none of which are host-exposed (§7). Includes the admin app. | **Generated** — an `include:` of `generated/prod/docker-compose.<name>.yml` ×8 | kong, caddy, authelia, and the same 10 observability containers |
| `docker-compose.infra.prod.yml` | Production infra: 3-broker Kafka KRaft cluster, Postgres/Mongo/Redis with no host ports (internal-network only). Used by `infra-up-prod` / `ci-deploy-prod`. | n/a | postgres, mongodb, redis, kafka1, kafka2, kafka3 |
| `docker-compose.infra.redis-sentinel.yml` | Prod Redis HA — **replaces**, does not supplement, the single `redis` service in `docker-compose.infra.prod.yml`. Selected with `REDIS_HA=yes` (§5). | n/a | redis-master, redis-replica-1/2, redis-sentinel-1/2/3 |

**The "App service blocks" column is the one to read before editing anything.** Both environments get their 8 service definitions from each repo's own descriptor via `orchestrator generate-compose`; the files under `generated/` are gitignored and rewritten on every `make local-dev` / `make ci-deploy-prod`, so an edit there is lost on the next run. To change a generated service, edit that repo's own `descriptor:` Makefile target — see [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md).

**Two files used to sit in this table and no longer exist:** `docker-compose.dev.yml` (the shared CI/dev host) and `docker-compose.staging.yml`, both hand-written, both deleted on 2026-08-19. They were the only environments the descriptor engine did not generate. See [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md) §2.

The original inventory outline for this document didn't list `docker-compose.local.dev.yml` at all — it's the one every engineer actually runs day-to-day (`make arm-run` → `make local-dev`), so it's the most important row in this table despite being the newest addition to the list.

---

## 3. Caddy / Nginx configuration

**`Caddyfile.prod` is the server's browser edge** — the only Caddy config left. `Caddyfile.dev` (the shared CI/dev host's edge, which bypassed Kong entirely) was deleted with that environment on 2026-08-19. `nginx.dev.conf` remains in the repo as dead code, referenced by no compose file since the Caddy migration.

`Caddyfile.prod` can also be exercised locally: `make edge-up` mounts this same file (with
`SITE_DOMAIN=localhost`, so Caddy issues an internal-CA cert instead of going to Let's Encrypt) in front
of the running local stack. Its `/api/*` and `/session/*` routes work; the frontend catch-all and the
`grafana.` vhost 502, because the local frontends are Vite dev servers rather than nginx-on-80 and the
observability stack isn't part of that overlay. See [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md) §2b.

It is mounted by `docker-compose.yml` and uses Compose service names (`kong`, `arm-session`, `core-frontend`, `calendar-frontend`, `admin-frontend`), which resolve on the server's private network.

| Route | Destination |
|---|---|
| `/api/*` | `kong:8000` — Kong does JWT validation, `X-User-Id` injection, rate limiting, and its own `strip_path` (§4) |
| `/calendar/remoteEntry.js`, `/calendar/calendar-assets/*` | `calendar-frontend:80` — MF bundle and built assets only; navigation falls through to the catch-all |
| `/admin/remoteEntry.js`, `/admin/admin-assets/*` | `admin-frontend:80` — same pattern |
| `/session/*` | `arm-session:5000` |
| `/faro-collector/*` | `alloy:12347` |
| everything else | `core-frontend:80` (catch-all) |

A `grafana.{$SITE_DOMAIN}` vhost gates Grafana behind Authelia's `forward_auth`. There is no separate auth subdomain — Authelia's login portal is served under `/authelia/*` on the same domain (`authelia/configuration.prod.yml`).

**`SITE_DOMAIN` is a hard prerequisite, not a convenience.** It must be a real domain whose DNS A/AAAA records already point at the host before Caddy starts, because certificates are provisioned through Let's Encrypt's ACME HTTP-01 challenge — port 80 has to be reachable from the internet. `grafana.$SITE_DOMAIN` needs its own record pointed at the same host.

So the full picture: **Kong is the real API edge on the server, but it is not the *browser* edge in either** — Caddy terminates TLS in front of it and is the only container publishing host ports (§8).

**`nginx.dev.conf` is dead code — verified.** It is not referenced by any `docker-compose*.yml`, `Makefile`, or `make/*.mk` in the repo (confirmed by grep across all of them). It predates `Caddyfile.dev` (older file mtime) and its route table shows it: no `/session/*` route and no `/admin/*` route at all — it was written before the session service and admin app existed. Don't use it as a reference for current routing; if it isn't deleted at some point, that's a housekeeping item, not a functioning config.

---

## 4. Kong configuration (`kong/kong.yml.tmpl` → `generated/kong.yml`)

Kong runs in **DB-less declarative mode** — the entire config is one file, no admin API writes needed at runtime. That file is **generated**: `kong/kong.yml.tmpl` is rendered to `generated/kong.yml` by `make generate-kong-config`, and the compose files mount the rendered output. `ci-deploy-prod`, `up`, and `edge-up` all run the render first, so no path starts Kong without it.

It has to be rendered because **Kong interpolates nothing in declarative config** — neither `${VAR}` nor `{vault://env/...}` (`jwt_secrets.secret` is not a referenceable field in DB-less mode; both verified against `kong:3.7-ubuntu` on 2026-08-19). See §4.1 for what that cost before it was fixed.

| Service | Upstream | Routes | Plugins |
|---|---|---|---|
| `core-api` | `http://core-backend:3000` | `/api/core` (strip prefix) | `jwt` (validates `iss` claim, `exp`), `pre-function` (decodes the JWT payload and sets `X-User-Id` from `payload.sub`), `rate-limiting` (90/min, 2000/hr, shared bucket) |
| `calendar-api` | `http://calendar-backend:4001` | `/api/calendar` (strip), `/api/calendar/v1/auth` (public, no strip — separate looser rate limit for OAuth), `/api/calendar/events/notifications` (Google webhook path, no strip, own rate limit) | Same `jwt` + `X-User-Id` pattern on the main route; the auth and webhook routes get their own `pre-function` path rewrites and rate limits (20/min auth, 30/min webhook) |
| `admin-api` | `http://admin-backend:10001` | `/api/admin` (strip prefix) | Same `jwt` + `pre-function` `X-User-Id` pattern as `core-api`, and the same 90/min–2000/hr rate-limit bucket |

Global plugin: `prometheus` (metrics scraping, matches the "Kong metrics" plan referenced in the documentation inventory's DOC-P15/P13 status notes — that plan is what wired this). **Those metrics are no longer served from the admin API port** — see the listener split below.

**`admin-api` now exists — this reverses an earlier finding in this document.** The route was added alongside the descriptor-driven production rollout, at the same time the admin app gained real production compose services. The earlier finding in §7 that the admin app had no deployed path is fully closed.

**JWT consumer secret contract:** the single `arm-platform` consumer's `jwt_secrets[0].secret` is the template placeholder `{{KONG_JWT_SECRET}}`, substituted at render time from `JWT_ACCESS_SECRET`. Because the substitution is mechanical, Kong's key **cannot** drift from what `core-be` signs with — the constraint root `CLAUDE.md` documents between `core-be` and `calendar-be` is enforced for Kong rather than merely stated. The render fails closed if `JWT_ACCESS_SECRET` is empty, and fails if any placeholder is left unsubstituted.

### 4.1 Two bugs that meant Kong's auth never actually worked

Both were found on 2026-08-19 by `make edge-up` — the first time Kong ran anywhere but the server — and both are fixed. They are recorded here because the failure modes are instructive, and because they explain why nobody noticed: as this document's own §4 note and [KONG-CONFIG.md](KONG-CONFIG.md) both observe, no code in the workspace calls Kong's `/api/*` paths. The browser goes through the session service, which Caddy routes past Kong entirely.

**Kong's Lua sandbox blocked the `X-User-Id` injection.** The `pre-function` plugin calls `require("cjson")`, and Kong runs untrusted Lua with `untrusted_lua = sandbox` by default, which forbids `require`. Every request to `/api/*` carrying an `Authorization` header returned **500 from Kong itself** (`require 'cjson' not allowed within sandbox`); requests with no token still 401'd correctly, because the Lua is guarded by `if auth then`. Fixed with `KONG_UNTRUSTED_LUA_SANDBOX_REQUIRES: cjson` — the sandbox stays on and exactly one module is allowlisted. Rewriting the Lua to pattern-match `sub` out of the raw JSON was rejected: extracting an *identity* claim with string patterns means a crafted payload like `{"x":{"sub":"admin"},"sub":"real"}` can yield the wrong match.

**Kong's JWT secret was the literal string `${KONG_JWT_SECRET}`.** Since declarative config isn't interpolated, that placeholder *was* the HMAC key. Every genuine `core-be` token was rejected 401, while a token signed with the placeholder — a value committed to the repo — was accepted and its `sub` injected as `X-User-Id`. It was not exploitable end to end, because `TRUST_PROXY_HEADERS` is set nowhere and the backends therefore re-verify the JWT themselves (see [SECURITY.md](SECURITY.md) §3) — but it would have become full impersonation the moment that flag was turned on, which is exactly what that flag is for. Fixed by rendering the config, per §4.

The two interact: while the sandbox bug stood, everything 500'd, which accidentally fail-closed the secret bug. Fixing either alone would have been worse than fixing neither.

**A rate-limiting caveat already called out in the config's own comment:** the 90/min–2000/hr bucket on `core-api` is keyed to the single shared `arm-platform` JWT consumer — it's a platform-wide ceiling, not per-tenant. True per-tenant limiting would need one Kong consumer per tenant, which requires `arm-core-be` to issue per-tenant JWT claims — noted in `kong.yml` as tracked separately, not solved by this config.

**Admin API exposure — previously a dev-only hardening gap, now closed in all three environments.** Kong's DB-less admin API has no authentication of its own (RBAC is Kong Enterprise-only), and `POST /config` against it replaces Kong's entire routing and plugin configuration. Binding it to `0.0.0.0` therefore exposes a full config takeover to every other container on the shared network, not merely to the host's other interfaces.

All three compose files now bind:

```yaml
KONG_ADMIN_LISTEN: "127.0.0.1:8001"    # loopback only, never host-published
KONG_STATUS_LISTEN: "0.0.0.0:8100"     # /status and /metrics only
```

The second listener is why the first could be locked down: Prometheus scrapes `arm-kong-dev:8100`, which serves only `/status` and `/metrics` and never exposes the admin API. `observability/prometheus.yml`'s `kong` scrape job targets `8100` accordingly — if you find a reference anywhere to scraping Kong on `8001`, it predates this split.

---

## 5. Redis Sentinel (prod HA)

`docker-compose.infra.redis-sentinel.yml` runs *instead of*, not alongside, the single `redis` service in `docker-compose.infra.prod.yml`. It expects `arm-infra-network` to already exist (`external: true`), so it's meant to layer onto an infra stack already brought up another way. This is wired into the normal deploy flow — `make ci-deploy-prod REDIS_HA=yes` (or `infra-up-prod REDIS_HA=yes`) switches to it automatically, stopping whichever topology isn't selected first so both never run at once. Previously this file had to be run manually with no flag anywhere pointing an operator at it.

**Topology:** 1 master + 2 replicas + 3 sentinels.

| Container | Role | Host port |
|---|---|---|
| `arm-redis-master` | Primary — `--requirepass`/`--masterauth` set, AOF persistence (`appendonly yes`, `everysec`) | none (internal only) |
| `arm-redis-replica-1`, `arm-redis-replica-2` | `--replicaof arm-redis-master 6379`, same persistence settings, `depends_on: redis-master (healthy)` | none (internal only) |
| `arm-redis-sentinel-1/2/3` | Each writes its own `sentinel.conf` at container start (`sentinel resolve-hostnames yes` then `sentinel monitor arm-master arm-redis-master 6379 2` — quorum of 2) | `26379`, `26380`, `26381` |

**`sentinel resolve-hostnames yes` is required and easy to omit** — it defaults to `no` in Redis Sentinel, in which case `sentinel monitor`'s hostname argument is parsed as a literal IP and fails immediately with `Can't resolve instance hostname`, regardless of whether DNS itself resolves the name correctly (verified directly: this looks exactly like a DNS-readiness race but isn't one — the whole file used to crash-loop on every start before this was set).

**Failover parameters:** `down-after-milliseconds 5000`, `failover-timeout 60000`, `parallel-syncs 1`. Applications connect through Sentinel (port 26379+), not directly to the master, so a failover is transparent to Redis clients that support Sentinel-aware connection strings — `session`/`admin-be` do not currently use Sentinel-aware connection strings, so a failover still drops every session even with this topology running (unchanged, cross-repo, app-level gap).

Only the three Sentinel ports are host-exposed — master and replicas are reachable only from other containers on `arm-infra-network`, which is the correct posture for a database tier.

---

## 6. Why the observability stack lives in the app compose file, not `.infra.yml`

The stack's 10 containers are defined in `docker-compose.yml` alongside Caddy and Authelia. **Local dev has none of it** — the stack exists on the server only.

The placement decision is an infrastructure one rather than an application one: Grafana has to share the app network with the Caddy that fronts it, or the `forward_auth` gate can't reach it at all. Splitting the observability containers into their own compose file (as the stack's original proposal, `docs/architecture/observability-plan.html`, suggested) would put Grafana on a different Docker network than its own edge — solved for no benefit.

Configuration lives in `observability/prometheus.prod.yml` and `observability/alloy/config.prod.alloy`, with the Grafana vhost in `Caddyfile.prod` (`grafana.$SITE_DOMAIN`). **Until 2026-08-19 there were three parallel copies of each config** — one per deployed environment — because scrape targets are container names and those names were environment-suffixed (`arm-core-backend-dev` vs. the unsuffixed server names). A new scrape target had to be added to all three or it drifted silently. Removing the dev and staging environments left one copy of each.

**Verified drift against `CLAUDE.md`'s own text:** the CLAUDE.md's Observability section lists Tempo/tracing as "Out of scope, deferred, not overlooked... explicitly Phase B in the source doc." That is no longer accurate — `docker-compose.yml` **has a running `tempo` service** (`grafana/tempo:2.6.1`, config at `observability/tempo.yml`, persistent volume `arm_tempo_prod_data`). Tempo has shipped since that paragraph was written; the doc just hasn't been updated to say so. This matches the documentation inventory's own note under DOC-P13/P15 that a `2026-08-08-tempo-tracing-kong-metrics.md` plan exists — that plan is what this compose change implements.

Access convention for every container in this stack: **nothing gets a public path except Grafana**, and Grafana gets one only through its Caddy vhost behind Authelia, never a host port. Confirmed by grep — no `prometheus`, `grafana`, `loki`, `alloy`, or `tempo` entry appears in any compose file's host-port mappings. Everything else is reachable only on the server's app network plus `arm-infra-network`, mirroring the no-public-port convention already applied to each backend's own `/metrics` endpoint. `caddy` is the only container in `docker-compose.yml` itself with a `ports:` entry — the session service and the three frontends still publish host ports, but they do so from their generated include files, not from any hand-written block (§8).

---

## 7. Network topology per environment

Both environment stacks share one thing: **`arm-infra-network`**, always declared `external: true` in the app-stack compose file, meaning it must already exist (created by whichever infra compose file — `.infra.yml` or `.infra.prod.yml` — is brought up first). Each environment then gets its own private bridge network for app-to-app traffic:

| Environment | App network | Infra brought up by |
|---|---|---|
| Local | `arm-local-network` | `docker-compose.infra.yml` |
| Server | `arm-prod-network` | `docker-compose.infra.prod.yml` (optionally with `docker-compose.infra.redis-sentinel.yml` layered in, §5) |

```text
                     ┌─ arm-infra-network (external, shared) ─┐
                     │  postgres · mongodb · redis · kafka    │
                     └───────────────┬─────────────────────────┘
                                     │
                    ┌────────────────┴────────────────┐
                    │                                 │
             arm-local-network                 arm-prod-network
             (local app services)              (server app services +
                                                Kong/Caddy/Authelia/
                                                observability)
```

**Verified — the admin app is fully wired on the server, and the last inconsistency is now closed.** `docker-compose.yml` includes `generated/prod/docker-compose.admin-be.yml` and `.admin-fe.yml`, `kong.yml` has an `admin-api` route (§4), and `deploy-production.yml`'s verify job checks `arm-admin-backend`/`arm-admin-frontend` — all three halves are consistent. Staging was the one environment that never got an admin service, so `Caddyfile.prod`'s `/admin/*` routes 502'd there while Kong still advertised the route; deleting that environment on 2026-08-19 closed the gap.

**No backend is host-exposed on the server.** An earlier revision of staging published `core-backend` (`4000:3000`) and `calendar-backend` (`4001:4001`) directly, bypassing Kong for anyone who could reach the host; those entries were removed before that environment was deleted. On the server today no backend has a `ports:` entry — traffic reaches them only through Kong or from other containers on `arm-prod-network`. The session service and the three frontends do publish host ports (`arm-session` `5000`, `core-frontend` `3000:80`, `calendar-frontend` `3001:80`, `admin-frontend` `10000:80`), each declared as `PROD_HOST_PORT` in that service's own descriptor (§8).

---

## 8. Port reference

### Local dev (`docker-compose.local.dev.yml` — the everyday stack)

| Service | Host port | Container port |
|---|---|---|
| core-fe | `3000` | `3000` |
| calendar-fe | `3001` | `3001` |
| core-be | `4000` | `3000` |
| session | `5001` | `5000` |
| calendar-be | `4001` | `4001` |
| admin-fe | `10000` | `10000` |
| admin-be | `10001` | `10001` |
| notification | — | — (Kafka consumer, no HTTP surface) |
| postgres | `127.0.0.1:5432` | `5432` |
| mongodb | `127.0.0.1:27017` | `27017` |
| redis | `127.0.0.1:6379` | `6379` |
| kafka | `127.0.0.1:9092` | `9092` |

Every app-service row above comes from that service's own descriptor (`HOST_PORT`:`PORT`), not from a hand-written compose block — see [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md).

**The four infra ports are loopback-bound, and that is deliberate.** They were previously published as bare `5432`/`27017`/`6379`/`9092`, which on Docker means `0.0.0.0` — every database and the Kafka broker reachable from anything that could route to the developer's machine, including other devices on a café or hotel network, with only the DB password in the way. `127.0.0.1:`-prefixing them keeps `psql`/`mongosh`/`redis-cli` working from the host exactly as before while removing the off-host path entirely.

### CI/dev host (`docker-compose.dev.yml`)

| Service | Host port | Notes |
|---|---|---|
| Caddy | `80`, `443` | The real browser edge (§3) |
| Kong | — | **No host ports at all any more.** Present and healthy but not on the dev browser path (§3–§4); the admin API is loopback-only, and metrics are on the internal-only `8100` status listener that Prometheus scrapes over the Docker network |
| session | `5000` | Different host port than local dev's `5001` — this stack is reached via Caddy/DNS, not `localhost` directly |
| core-fe | `3000` | Container port `80` (built static assets, not Vite dev server) |
| calendar-fe | `3001` | Container port `80` |
| admin-fe | `10000` | Container port `80` |
| Grafana | — | No host port — reached only via `grafana.arametrics.app` through Caddy + Authelia |
| Prometheus, Loki, Alloy, Tempo, cAdvisor, exporters | — | No host ports — internal-network only |

### Server

| Service | Host port |
|---|---|
| Caddy | `80`, `443` — the only public path (§3) |
| Kong (proxy) | not host-exposed — reached at `kong:8000` from Caddy |
| Kong (admin) | `127.0.0.1:8001`, unpublished |
| Kong (status/metrics) | `8100`, internal-network only |
| session | `5000` |
| core-fe | `3000` → container `80` |
| calendar-fe | `3001` → container `80` |
| admin-fe | `10000` → container `80` |
| core-be | not host-exposed — Kong/internal only |
| calendar-be | not host-exposed — Kong/internal only |
| admin-be | not host-exposed — Kong/internal only |
| notification | — (Kafka consumer) |
| Grafana | no host port — `grafana.$SITE_DOMAIN` via Caddy + Authelia |
| Prometheus, Loki, Alloy, Tempo, cAdvisor, exporters | no host ports |

Every app-service row comes from that service's own descriptor `PROD_HOST_PORT` (§2).

### Production infra (`docker-compose.infra.prod.yml`)

No service in this file publishes a host port at all — Postgres, MongoDB, Redis, and all 3 Kafka brokers are internal-network-only, reachable only from `arm-prod-network`/`arm-infra-network` containers. Redis Sentinel (§5), if layered in, exposes only the 3 sentinel ports (`26379`–`26381`).

---

## 9. Volumes and persistence strategy

Every stateful service uses a named Docker volume (never an anonymous or bind-mounted data directory), scoped per environment where it matters:

| Volume | Backs | Environment |
|---|---|---|
| `arm_postgres_data`, `arm_mongodb_data`, `arm_redis_data`, `arm_kafka_data` | Postgres, Mongo, Redis, Kafka | Local/dev (`docker-compose.infra.yml`) |
| `arm_redis_master_data`, `arm_redis_replica1_data`, `arm_redis_replica2_data` | Redis Sentinel topology | Prod (opt-in, §5) |
| `arm_caddy_data`, `arm_caddy_config`, `arm_authelia_data`, `arm_prometheus_data`, `arm_loki_data`, `arm_tempo_data`, `arm_grafana_data` | Edge + observability stack state | CI/dev only |

The infra compose file's own comment (`docker-compose.infra.yml`) documents a one-time migration gotcha worth preserving here: the file used to have no top-level `name:`, so it ran under Docker Compose's directory-derived default project name. If infra was ever started before that change, `make infra-up` fails on `container_name` collisions under the new `arm-infra-dev` project name — the fix is a one-time `docker compose -p arm-deploy-make -f docker-compose.infra.yml down` (which does **not** remove the explicitly-named volumes — data survives), then a normal `make infra-up`.

### Backup and restore — tooling already implemented, runbook now written too

The underlying tooling was already real and working before this platform's documentation inventory caught up to that fact — `make/backup.mk`:

| Target | What it does |
|---|---|
| `make backup-postgres` | Runs `scripts/backup-postgres.sh` — `pg_dump`s **both** `arm_core` and `arm_admin` from the running `arm-postgres-prod` container, gzips each to `backups/postgres/<db>-<UTC timestamp>.sql.gz` |
| `make backup-mongodb` | Runs `scripts/backup-mongodb.sh` — `mongodump --archive --gzip` of the `arm-calendar` database from `arm-mongodb-prod` to `backups/mongodb/arm-calendar-<UTC timestamp>.archive` |
| `make backup-sync` | Runs `scripts/backup-sync-s3.sh` — `s3 sync`s `backups/` to S3-compatible object storage (Hetzner by default) through the official `amazon/aws-cli` Docker image, so there's no host-level AWS CLI dependency. **Skips with a warning and exits 0** if `BACKUP_S3_BUCKET` is unset, so it can't break `backup-all` on a host that hasn't opted in |
| `make backup-prune` | Deletes local backup files older than `BACKUP_RETENTION_DAYS` (default 14) |
| `make backup-all` | `backup-postgres` → `backup-mongodb` → `backup-sync` → `backup-prune` |
| `make restore-postgres-drill` | Restores the **most recent** `arm_core` backup into a throwaway `arm_core_restore_drill` database on the **same running instance**, counts restored tables, then drops the throwaway database. Never touches the live `arm_core` database. |
| `make restore-mongodb-drill` | The MongoDB equivalent — `mongorestore --nsFrom/--nsTo` remaps the latest archive into a throwaway `arm-calendar_restore_drill` database on the same running instance, counts collections, drops it. The live `arm-calendar` database is never written to. |
| `make backup-cron-install` / `-remove` / `-status` | Install, remove, or inspect a daily 02:00 crontab entry running `backup-all`, logging to `backups/cron.log`. Install is idempotent — it greps before adding, which is why `deploy-production.yml` can safely run it on every deploy |

**Three of the four gaps this section previously listed are now closed** (off-host storage, scheduling, and a MongoDB restore drill). What remains:

- **No automated drill for `arm_admin`**, despite `backup-postgres` dumping both databases — only `arm_core` has a tested restore path.
- **Nothing schedules the drills themselves.** The cron job runs `backup-all` (taking, syncing, and pruning backups), not the two `restore-*-drill` targets that prove those backups are usable. Running them stays a manual, on-call responsibility.
- **Retention is local-only.** `backup-prune` prunes local disk; the S3 bucket accumulates every synced backup forever unless a lifecycle rule is set on the bucket itself — deliberately left to the bucket, not this repo.
- **These are logical dumps, not PITR.** No WAL archiving (pgBackRest/wal-g), so the RPO is "since the last nightly dump" — a much larger undertaking than what closed here, and still open.
- **Both scripts hardcode the container names `arm-postgres-prod` / `arm-mongodb-prod`** — they work against the server compose stack's containers only, not the local container names.

**The runbook itself is now written**: [`runbooks/db-backup-restore.md`](runbooks/db-backup-restore.md) — including the first documented restore procedure for MongoDB and for `arm_admin`, neither of which had one before (only `arm_core`'s Postgres restore is drill-tested).

---

## 10. Known pitfalls / gaps (summary)

| Gap | Where | Status |
|---|---|---|
| `nginx.dev.conf` is unreferenced by every compose file and Makefile — dead config, predates the session service and admin app | §3 | Verified, harmless unless someone mistakes it for the current routing source of truth |
| `deploy/arm-deploy-make/CLAUDE.md`'s Observability section still says Tempo, Kong's `prometheus` plugin, and Grafana alerting are deferred to "Phase B" — all three have shipped | §6 | Verified drift, re-confirmed 2026-08-19: that doc's text is stale relative to its own compose files |
| No automated restore drill for `arm_admin`; nothing schedules the drills themselves; no PITR/WAL archiving | §9 | Verified — what's left after off-host sync, cron scheduling, and the MongoDB drill closed |
| DOC-P25 (Verdaccio Internal Registry Setup) in the documentation inventory documents infrastructure that has been removed | §1 | Verified — Verdaccio is gone, `npm publish --access public` is the current path |

**Closed since this document's previous pass (2026-08-10)**, kept here so a reader coming from an older copy can tell what changed rather than assuming the finding was wrong:

| Former gap | Closed by |
|---|---|
| Kong admin API open on all interfaces in dev | Loopback-only, with a separate `8100` metrics listener (§4) |
| Admin app absent from production | Descriptor-driven prod compose + `admin-api` Kong route (§4, §7) |
| Staging backends host-exposed, bypassing Kong | `ports:` entries removed, then the environment itself deleted (§7) |
| Infra databases published on `0.0.0.0` in local dev | `127.0.0.1:`-bound (§8) |
| Backups local-disk-only, unscheduled, no MongoDB drill | `backup-sync`, `backup-cron-install`, `restore-mongodb-drill` (§9) |
| Observability and TLS available only on the CI/dev host | Caddy + Authelia + the full stack now on the server (§1, §3, §6) |
| CI/dev and staging compose files hand-written while local and prod were descriptor-generated | Both environments deleted 2026-08-19 — every remaining service block is generated (§2) |
| Admin app had no staging deployment, so `Caddyfile.prod`'s `/admin/*` routed 502 there | Staging deleted; the server has admin wired end to end (§7) |

---

## 11. Related documents

| Doc | Role |
|---|---|
| [ARCHITECTURE.md](ARCHITECTURE.md) §3, §7 | Edge routing by environment, network boundaries at the platform level |
| [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md) | Where local dev's and production's service definitions come from — descriptors, the `services.conf` manifest, shared templates, `generated/` |
| [ENV-VARS.md](ENV-VARS.md) | Every env var referenced by these compose files, secret vs. non-secret |
| [CLAUDE.md](../../../CLAUDE.md) | Day-to-day `make` commands, local dev port map |
| [`deploy/arm-deploy-make/CLAUDE.md`](../../../deploy/arm-deploy-make/CLAUDE.md) | Makefile module architecture, `make arm` setup wizard internals, Makefile coding conventions |
| [`docs/guides/local-dev-setup.md`](guides/local-dev-setup.md) | Step-by-step first-time setup |
| [`deploy/arm-deploy-make/kong/readme.md`](../../../deploy/arm-deploy-make/kong/readme.md) | One-line Kong scope statement (edge-only, server, not local dev) |
| [`runbooks/restart-services.md`](runbooks/restart-services.md) | Restart commands per environment, healthcheck strength per service, what a restart does/doesn't affect |
| [KAFKA-ARCHITECTURE.md](KAFKA-ARCHITECTURE.md) | KRaft cluster config (dev vs. prod), topic replication factors, and the missing prod healthcheck stanza for `notification-service` |
| [KONG-CONFIG.md](KONG-CONFIG.md) | Every route/plugin in Kong's config, and the corrected Kong-prometheus-is-live finding |
| [ADR-007](adr/007-verdaccio-registry.md), [ADR-008](adr/008-redis-sentinel-ha.md) | The Verdaccio removal and Sentinel HA topology this document's §1/§5 describe, as decision records |
