# ARM Platform — Monitoring & Logging Architecture

> Log format and aggregation, every Prometheus scrape target, distributed tracing (Tempo + OpenTelemetry), frontend error tracking (Faro), alerting rules, health-check endpoints, and the 3 Grafana dashboards — consolidated from `docs/architecture/observability-plan.html` and 4 implementation plans under `docs/superpowers/plans/` into one settled reference reflecting what's actually running today.
>
> [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §6 covers *why* the observability stack lives in the server's app compose file rather than `.infra.yml` (§1). This document covers what the stack actually does, verified against every config file and every service's own instrumentation source — not the planning docs, which describe intent at a point in time that source has since moved past in three separate places (§5, §6).
>
> Verified against `observability/prometheus.prod.yml`, `observability/{loki-config,tempo}.yml`, `observability/alloy/config.prod.alloy`, `observability/grafana/provisioning/alerting/*.yaml`, all 3 `observability/grafana/dashboards/*.json`, every backend's `LoggerModule.forRoot()`/pino config, `apps/arm-app-calendar/src/backend/src/tracing.ts`, `core/arm-core-fe/src/lib/faro.ts`, and every service's health controller (2026-08-10; **stack placement and the Kong scrape port re-verified 2026-08-19** — see §1).

---

## 1. Stack overview

Prometheus, Grafana, Loki, Alloy, Tempo, cAdvisor, node-exporter, and 3 DB exporters (postgres/mongodb/redis) — 10 containers, defined in the server's `docker-compose.yml`. **Local dev runs none of it.** See [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §6 for why the containers live in the app compose file rather than `.infra.yml`.

Prometheus reads `observability/prometheus.prod.yml` and Alloy reads `observability/alloy/config.prod.alloy`. Until 2026-08-19 there were three parallel copies of each (dev / staging / prod) that had to be edited in lockstep and drifted silently when they weren't; deleting the dev and staging environments left one copy of each. The `.prod` filenames were kept — see [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md) §2 on why the identifiers stay `prod`.

Nothing but Grafana gets a public path — reached only via Caddy → Authelia's `forward_auth` gate at `grafana.$SITE_DOMAIN`.

**Because the stack exists only on the server, any change to it is first exercised there.** There is no pre-production environment in which a broken scrape config or dashboard would surface first.

**Four separate corrections to `deploy/arm-deploy-make/CLAUDE.md`'s Observability section have now surfaced** — the same section [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §6 already caught once (Tempo). All four are things that section describes as "deferred" or "not done" that are, in fact, live:

1. **Tempo tracing** — already documented as a correction in [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §6.
2. **Grafana alerting** — the CLAUDE.md text says alerting/contact points are "not part of Phase A's own scope." They're fully implemented: 7 real alert rules, a real email contact point, wired into Grafana's provisioning (§7).
3. **Pino `redact`** — the CLAUDE.md text says "none of the 5 backends configure pino `redact`." **All 5 do**, with identical `redact.paths` (§2). This also corrects a claim [SECURITY.md](SECURITY.md) §12/§13 made trusting that same CLAUDE.md text — see the note in §2 below.
4. **Kong's `prometheus` plugin** — the CLAUDE.md text lists this as explicitly deferred to "Phase B" alongside Tempo. It's live: a global `prometheus` plugin in `kong/kong.yml.tmpl` plus a matching `kong` scrape job in each environment's Prometheus config (the `kong` row in the table below) — see [KONG-CONFIG.md](KONG-CONFIG.md) §6 for the full verification.

Treat that CLAUDE.md section as describing the stack's *original* scope, not its current state — four of its "not done" claims are now wrong in the same direction (things shipped without the doc being updated), which is a pattern worth someone revisiting that file directly.

---

## 2. Log format and redaction

Every one of the 5 backends (`core-be`, `session`, `calendar-be`, `admin-be`, `notification`) configures `nestjs-pino`'s `LoggerModule.forRoot()` identically:

```ts
pinoHttp: {
  level: ['production','staging'].includes(NODE_ENV) ? 'info' : 'debug',
  transport: !isProd ? { target: 'pino-pretty', options: { colorize: true, singleLine: true } } : undefined,
  genReqId: (req) => req.headers['x-correlation-id'] || req.headers['x-request-id'] || crypto.randomUUID(),
  customProps: () => ({ service: '<service-name>' }),
  redact: {
    paths: ['req.headers.authorization', 'req.headers.cookie', '*.password', '*.accessToken', '*.refreshToken', '*.token', '*.secret'],
    censor: '[Redacted]',
  },
  serializers: {
    req: (req) => ({ method: req.method, url: req.url, correlationId: req.id }),
    res: (res) => ({ statusCode: res.statusCode }),
  },
}
```

- **Structured JSON in staging/production**, human-readable `pino-pretty` (colorized, single-line) in every other `NODE_ENV` — including local dev.
- **Correlation ID propagation:** every request gets a `correlationId`, either inherited from an incoming `x-correlation-id`/`x-request-id` header or freshly generated. This is the mechanism that would let you trace one request across services in Loki — *if* every service actually forwarded the header to the next hop. Not verified in this pass whether the session service proxy or calendar-be's outbound calls to core-be actually propagate `x-correlation-id` onward; the generation and per-service logging exist, but cross-service propagation wasn't confirmed end-to-end.
- **`redact` is real and platform-wide, verified in all 5 backends' source** (`core-be`, `calendar-be`, `session`, `admin-be`, `notification` — each read directly, not sampled). Both the identical `redact.paths` list and the identical explanatory comment ("Defense-in-depth: the req/res serializers already strip headers/bodies... this catches sensitive fields in any ad-hoc `logger.log(obj)` call") appear verbatim across services, indicating this was rolled out as one platform-wide pattern. **This closes the gap [SECURITY.md](SECURITY.md) listed under OWASP A09 and its Known Issues table (§12–13)** — that finding trusted `deploy/arm-deploy-make/CLAUDE.md`'s claim without re-verifying against each backend's actual `LoggerModule` config, which is exactly the kind of thing this pass caught. `SECURITY.md` should be corrected to reflect this; not done as part of this document, flagged here for whoever picks it up next.
- **Serializers strip the full request/response objects** down to method/URL/correlationId and status code — no headers, no body, logged by default from the HTTP access-log layer itself. `redact` is the second layer, catching anything logged outside that access-log path.

---

## 3. Log aggregation (Loki + Alloy)

- **Alloy** (`grafana/alloy`, River-syntax config at `observability/alloy/config.alloy`) discovers every running Docker container (`discovery.docker`) and tails **all of their stdout/stderr** into Loki via `loki.source.docker` — this is container-level log collection, not an application integration each service opts into. Any container that logs to stdout is automatically captured; the pino `redact` config (§2) is what keeps sensitive fields out of what gets shipped, since Alloy itself does no redaction.
- **Loki** (`observability/loki-config.yml`): filesystem storage, single replica, TSDB index (schema v13), **720h (30 day) retention** (`limits_config.retention_period`), with a compactor handling retention enforcement. `reject_old_samples_max_age: 168h` (7 days) — a log line older than a week arriving out of order gets rejected, not silently accepted.
- Reachable only through Grafana's Explore view (or the `service-health`/`capacity`/`dependency-map` dashboards, §8) behind the same Authelia gate as everything else in this stack.

---

## 4. Application metrics (Prometheus)

Full scrape config (`observability/prometheus.yml`), 15s scrape/eval interval:

| Job | Target | Path |
|---|---|---|
| `core-be` | `arm-core-backend-dev:3000` | `/v1/metrics` |
| `session` | `arm-session-dev:5000` | `/metrics` |
| `calendar-be` | `arm-calendar-backend-dev:4001` | `/v1/metrics` |
| `admin-be` | `arm-admin-backend-dev:10001` | `/v1/metrics` |
| `notification` | `arm-notification-service-dev:4001` | `/metrics` |
| `kong` | `arm-kong-dev:8100` | (Kong's own `prometheus` plugin, served from the dedicated `KONG_STATUS_LISTEN` port — **not** the admin API port, which is now loopback-only) |

Path differs by whether the service has a global `v1` prefix (`core-be`/`calendar-be`/`admin-be` do; `session`/`notification` don't). All 5 backends use `@willsoto/nestjs-prometheus`, confirmed via source in every one.

**Per-service metrics beyond the shared `http_requests_total`/`http_request_duration_seconds` pair** (all 5 have these via a shared `createHttpMetricsMiddleware` pattern, registered as Express middleware specifically so it wraps Nest's entire guard/interceptor/filter pipeline and can see guard-rejected requests like 429s):

| Service | Extra metrics |
|---|---|
| `session` | `session_miss_total`, `session_proxy_duration_seconds` |
| `calendar-be` | Event-loop-lag gauge (`common/observability/event-loop-lag.service.ts`) — backs the "Calendar-be Event Loop Lag" dashboard panel (§8) |
| `notification` | Kafka message metrics by status (`common/metrics/kafka-metrics.providers.ts`) — backs the "Notification Kafka Messages (by status)" dashboard panel |

Every backend also restricts its own `/metrics` (or `/v1/metrics`) endpoint to internal-network callers only — the same private-IP-range check (`127.0.0.1`, `::1`, `10.*`, `172.*`, `192.168.*`) implemented identically in each `main.ts`, confirmed while writing [SECURITY.md](SECURITY.md).

---

## 5. Infrastructure metrics

| Exporter | Scrapes | Job name |
|---|---|---|
| `node-exporter` | Host CPU/memory/disk | `node` |
| `cadvisor` | Per-container resource usage | `cadvisor` |
| `postgres-exporter` | Postgres (dedicated `arm_monitoring` role, not the app's own credentials — [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §6) | `postgres` |
| `mongodb-exporter` | MongoDB (dedicated `arm_monitoring` user) | `mongodb` |
| `redis-exporter` | Redis (memory, clients, commands/sec — see [REDIS-ARCHITECTURE.md](REDIS-ARCHITECTURE.md) §10 for the gap in visualizing this) | `redis` |

---

## 6. Distributed tracing — infra is fully built, instrumentation is not

**Verified: only one of five backends is actually instrumented for tracing.** `apps/arm-app-calendar/src/backend/src/tracing.ts` sets up a full OpenTelemetry `NodeSDK` with auto-instrumentation (`getNodeAutoInstrumentations()` — patches `http`, MongoDB, ioredis, and other modules at `require()` time, which is why the file must be the very first thing `main.ts` imports) and exports to `OTEL_EXPORTER_OTLP_ENDPOINT`, falling back to console output if that env var is unset. **None of `core-be`, `session`, `admin-be`, or `notification` has an `opentelemetry` dependency at all** — confirmed by checking every `package.json`.

The receiving infrastructure is fully built regardless of that gap:

```
calendar-be (only instrumented service)
    │ OTLP gRPC/HTTP
    ▼
Alloy (otelcol.receiver.otlp → otelcol.processor.batch → otelcol.exporter.otlp)
    │ OTLP gRPC :4317
    ▼
Tempo (720h / 30-day retention, local filesystem storage)
    │
    ▼
Grafana (Tempo datasource, not yet a dedicated tracing dashboard — see §8)
```

**Practical consequence:** a trace exists end-to-end only for spans generated inside `calendar-be` itself (its own HTTP handlers, its Mongo queries, its Redis calls). A request that flows browser → Caddy → session service → calendar-be → core-be (e.g. calendar-be fetching a Google token from core-be) produces a trace with a gap — the session service hop and the core-be hop emit no spans, so they're invisible in Tempo even though calendar-be's own spans for that same request exist. Distributed tracing, in the sense of following one request across service boundaries, isn't achievable yet — only single-service tracing for calendar-be specifically. Whether the other four services were deliberately left out of this pass (a phased rollout) or the work simply stopped after calendar-be wasn't determined from source; either way, it's the current state.

---

## 7. Alerting — fully implemented, contrary to CLAUDE.md's claim

`observability/grafana/provisioning/alerting/` (mounted read-only into Grafana via `./observability/grafana/provisioning:/etc/grafana/provisioning:ro` — the same mount in all three environments' compose files, and the one observability config that *is* shared rather than split per environment) contains real, non-stub configuration:

**7 alert rules** (`rules.yaml`, folder `ARM`, 60s evaluation interval):

| Rule | Condition | Severity |
|---|---|---|
| ARM service is down | `up{job=~"core-be\|session\|calendar-be\|admin-be\|notification"} < 1` for 2m | critical |
| Postgres is down | `pg_up < 1` for 2m | critical |
| MongoDB is down | `mongodb_up < 1` for 2m | critical |
| Redis is down | `redis_up < 1` for 2m | critical |
| High HTTP 5xx error ratio | 5xx rate / total rate `> 5%` over 5m — **only `job=~"core-be\|admin-be"`** | warning |
| Host disk space low | free/total filesystem bytes `< 10%` for 5m | warning |

**Verified gap in the 5xx rule specifically:** it only covers `core-be` and `admin-be` — `calendar-be`, `session`, and `notification` can have an elevated 5xx rate with no alert firing. This matches the "Service Health" dashboard's own panel scoping (§8), so it's a consistent choice across both the dashboard and the alert rule, not an oversight in one but not the other — but it's still a real coverage gap worth knowing about.

**Notification and delivery, real not placeholder:**
- One contact point (`contact-points.yaml`): `email-alerts`, type `email`, address from the `$GRAFANA_ALERT_EMAIL` env var (interpolated by Grafana's own provisioning engine — note `$VAR` not `${VAR}` syntax, called out in the compose file's own comment as intentional).
- One notification policy (`notification-policies.yaml`): groups by `alertname`, 30s group-wait, 5m group-interval, **4h repeat-interval** (a firing alert re-notifies at most every 4 hours, not on every evaluation).
- Grafana's own SMTP is configured on the `grafana` service in each environment's compose file (`GF_SMTP_*` env vars, reusing the platform's `SMTP_HOST`/`SMTP_USER`/`SMTP_PASS`) — the same mail credentials the notification service uses for user-facing email.
- `GRAFANA_ALERT_EMAIL` is already a required secret in `deploy-dev.yml`'s GitHub Actions secret check ([CICD.md](CICD.md) §4) — confirming this was provisioned as part of the same rollout, not a dangling reference to an unset variable.

---

## 8. Health check endpoints

| Service | Endpoint(s) | Checks |
|---|---|---|
| `core-be` | `GET /health` (aggregated), `GET /health/live` (bare) | Postgres (`TypeOrmHealthIndicator`, 2s timeout), Redis (`RedisService.isHealthy`) |
| `calendar-be` | `GET /health/live` (process-only), `GET /health/ready` (dependency check) | **Deliberately split** — `live` never touches a dependency (so an orchestrator doesn't kill a healthy process just waiting on a DB reconnect); `ready` pings MongoDB (`db.admin().ping()`, not just `readyState`) and Redis (`PING`/`PONG`) via `Promise.allSettled`, returning 503 with per-dependency detail if either fails |
| `admin-be` | `GET /v1/health` (public) | Both Postgres connections (`arm_core`, `arm_admin`) + Redis, per its own `CLAUDE.md` |
| `session` | `GET /session/health` | Liveness only — no dependency checks (`session.controller.ts`, not a separate health module) |
| `notification` | `GET /v1/health/live` (process-only), `GET /v1/health/ready` (Kafka broker round-trip) | **Fixed 2026-08-10** — was a hardcoded `{ status: 'ok' }` stub (see §11). Now split the same way as `calendar-be`: `live` never touches Kafka; `ready` opens a fresh, throwaway `kafkajs` admin client (`KafkaHealthService`, 3s connect/request timeout, no retries) and calls `listTopics()` — deliberately not reusing the app's own long-lived producer/consumer connections, since those had already proven they can silently die without anything noticing. Empirically verified: `ready` returns `200 {kafka:"up"}` with Kafka reachable, `503 {kafka:"down"}` within seconds of the broker going away, and back to `200` once it recovers — tested against a real broker, not just unit-mocked. |

**`calendar-be`'s live/ready split was, until this fix, the only one in the platform following the orchestrator-safety pattern properly** (liveness ≠ readiness, dependency checks only on the readiness path). `notification` now matches it. `core-be` and `admin-be` still check dependencies on their only health endpoint (risking the restart-loop-on-transient-DB-blip failure mode `calendar-be`'s own code comment explicitly warns against), and `session` still checks nothing at all — both remain open gaps, not addressed by this fix.

---

## 9. Grafana dashboards

Three dashboards, versioned as JSON under `observability/grafana/dashboards/` — edited as code, not through the Grafana UI (the provisioner reloads from these files):

| Dashboard | Panels |
|---|---|
| **ARM Service Health** | Service Up, HTTP Request Rate (core-be, admin-be), HTTP p95 Latency (core-be, admin-be), HTTP 5xx Error Rate, Session Proxy Duration p95, Notification Kafka Messages (by status), Calendar-be Event Loop Lag |
| **ARM Capacity** | Host CPU Usage %, Host Memory Available, Host Filesystem Free (root), Container CPU Usage (top 10), Container Memory Usage (top 10) |
| **ARM Dependency Map** | Datastore Up, Postgres Active Connections, Postgres Transaction Rate, MongoDB Connections, Redis Connected Clients / Memory |

**Same coverage gap as §7's 5xx alert:** "HTTP Request Rate" and "HTTP p95 Latency" panels are scoped to `core-be, admin-be` only — `calendar-be`, `session`, and `notification` aren't broken out in Service Health beyond the shared "Service Up" panel. No dedicated Redis dashboard exists despite `redis-exporter` metrics being scraped (restated from [REDIS-ARCHITECTURE.md](REDIS-ARCHITECTURE.md) §10 — Redis only appears as a few panels inside Dependency Map, not its own board). No tracing-specific dashboard exists either, consistent with §6's finding that tracing itself is only partially rolled out.

---

## 10. Frontend error tracking (Faro)

Alloy runs a Faro receiver (`faro.receiver`, port 12347) accepting browser telemetry — reached via `Caddyfile.dev`'s `/faro-collector/*` route (`arm-alloy-dev:12347`), CORS-restricted to `https://dev.arametrics.app`. Faro-received data is forwarded into Loki alongside container logs.

**Verified: only `core-fe` (the shell) has the Faro SDK integrated** — `@grafana/faro-react`'s `initializeFaro()` with `getWebInstrumentations()` (errors, web vitals, resource timing, session tracking), tagged `app.name: "arm-core-fe"`. **Neither `calendar-fe` nor `admin-fe` has the Faro SDK as a dependency at all.**

**Practical nuance, not a total blind spot:** since all three MFEs render inside one browser tab/JS runtime once federated into the shell, `core-fe`'s automatic instrumentation (unhandled errors, unhandled promise rejections) likely still captures a crash that originates inside `calendar-fe`'s or `admin-fe`'s own component code — JS errors aren't scoped to the module that threw them. What's genuinely missing is **attribution**: every captured event is tagged `app.name: "arm-core-fe"` regardless of which MFE the error actually came from, and neither remote can push a deliberate custom Faro event (`faro.api.pushError`, a business-logic-level "this flow failed" signal) since neither has the SDK to call. Root-causing "is this a shell bug or a calendar-fe bug" from Faro data alone isn't possible today.

`initFaro()` skips silently (no throw) if `VITE_FARO_COLLECTOR_URL` isn't set — local dev without the observability stack running keeps working, by design.

---

## 11. Known pitfalls (summary)

| Pitfall | Where | Status |
|---|---|---|
| Only `calendar-be` has OpenTelemetry tracing — the other 4 backends have zero `opentelemetry` dependencies | §6 | Verified, infra fully built and unused by 4/5 services |
| Only `core-fe` has the Faro SDK — `calendar-fe`/`admin-fe` errors aren't attributable to their own MFE | §10 | Verified, automatic error capture likely still works platform-wide, attribution doesn't |
| 5xx alert rule and Service Health dashboard panels both scope to `core-be`/`admin-be` only | §7, §9 | Verified, consistent gap across both — `calendar-be`/`session`/`notification` uncovered |
| `notification`'s `/v1/health` was a hardcoded stub with no Kafka connectivity check | §8 | **Fixed 2026-08-10** — split into `/v1/health/live` + `/v1/health/ready` (real broker check), both `docker-compose.local.dev.yml` and `docker-compose.dev.yml` healthchecks updated to use `/ready`, verified against a real broker (up → down → recovered) |
| Only `calendar-be` implements the liveness/readiness split correctly; the rest either check dependencies on their only endpoint or check nothing | §8 | Verified |
| `deploy/arm-deploy-make/CLAUDE.md`'s Observability section is stale in 3 separate places (Tempo, alerting, pino redact) — all describe unimplemented work that has since shipped | §1 | Verified across this document and [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §6 |
| [SECURITY.md](SECURITY.md)'s pino-`redact` gap (OWASP A09, §13) is stale — redact is implemented in all 5 backends | §2 | Verified here; `SECURITY.md` not yet corrected |
| Correlation-ID cross-service propagation (`x-correlation-id` forwarded by session-service proxy / inter-service calls) generated and logged per-hop, but not confirmed to actually propagate end-to-end | §2 | Not verified either way in this pass |

---

## 12. Related documents

| Doc | Role |
|---|---|
| [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §6 | Why the stack lives in each environment's app compose file, the per-environment config split, and the first Tempo correction |
| [SECURITY.md](SECURITY.md) §12–13 | The pino-`redact` finding this document corrects (OWASP A09) |
| [REDIS-ARCHITECTURE.md](REDIS-ARCHITECTURE.md) §10 | The Redis-dashboard gap restated here in §9 |
| [CICD.md](CICD.md) §4 | `GRAFANA_ALERT_EMAIL` as a required deploy secret, confirming §7's alerting is provisioned, not dangling |
| `docs/architecture/observability-plan.html` | The original stack proposal — source of the 30-day retention range this document confirms was implemented at its top end |
| [KAFKA-ARCHITECTURE.md](KAFKA-ARCHITECTURE.md) §8 | The `notification-service` Kafka health-check fix this document references, and the prod healthcheck-stanza gap that undercuts it |
