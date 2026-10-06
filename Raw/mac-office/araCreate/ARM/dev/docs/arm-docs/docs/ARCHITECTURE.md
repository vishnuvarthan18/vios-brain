# ARM Platform — Architecture Overview

> Canonical read-first document for how ARM services relate to each other. For deeper slices, follow the links in each section rather than duplicating them here.
>
> Verified against `deploy/arm-deploy-make` compose + Make targets, `repos.conf` / `services.conf`, root/`deploy` CLAUDE notes, and the live session service / MF configs (structure re-verified 2026-08-27). Where older docs disagree with the compose files that `make local-dev` actually runs, **this document follows the compose files**.

---

## 1. What ARM is

**ARM (Ara Resource Manager)** is a multi-tenant SaaS platform for resource and calendar management. It is assembled as a **multi-repo workspace** (not a monorepo): independently deployable git repositories checked out under one directory. Six of them — the application repos — are cloned via `make clone-repos` from `deploy/arm-deploy-make/repos.conf`; the rest are checked out alongside them.

`deploy/arm-deploy-make` is orchestration only — Make targets, Compose files, env init/check, edge configs, CI deploy. It owns no application business logic. A root `Makefile` forwards into that repo.

### 1.1 Workspace repository inventory

```text
arm/                                   ← workspace root
├── core/
│   ├── arm-core-be/                   ← users, roles, orgs (PostgreSQL arm_core)
│   ├── arm-core-fe/                   ← shell app, Module Federation host
│   └── arm-session/                   ← session layer, OAuth, proxy to all backends
├── apps/
│   ├── arm-app-calendar/              ← src/backend/ + src/frontend/ in one repo
│   └── arm-admin/                     ← src/backend/ + src/frontend/ in one repo
├── services/
│   └── arm-service-notification/      ← Kafka consumer → SMTP
├── library/
│   └── arm-ui-library/                ← shared React component library
├── deploy/
│   └── arm-deploy-make/               ← ALL make commands run from here
├── docs/
│   └── arm-docs/                      ← this document set
├── arm-tool-cli/                      ← repo scaffolder / platform CLI
└── aracreate-template-codebase/       ← the araCreate repo template (reference, not an ARM service)
```

| Directory | GitHub repo | In `repos.conf` | Branch |
|---|---|---|---|
| `core/arm-core-be` | `arm-core-be` | yes | `dev` |
| `core/arm-core-fe` | `arm-core-fe` | yes | `dev` |
| `core/arm-session` | **`arm-core-bff`** | yes | `dev` |
| `apps/arm-app-calendar` | `arm-app-calendar` | yes | `dev` |
| `apps/arm-admin` | `arm-app-admin` | yes | `dev` |
| `services/arm-service-notification` | `arm-service-notification` | yes | `dev` |
| `library/arm-ui-library` | `arm-library-ui-components` | no | `dev` |
| `deploy/arm-deploy-make` | `arm-devops-makefile` | no | `main` |
| `docs/arm-docs` | `arm-docs` | no | `main` |
| `arm-tool-cli` | `arm-tool-cli` | no | `dev` |

**Four name mismatches are live and load-bearing** — the directory name is not the repo name for `arm-session` (remote is still `arm-core-bff.git`: the [rename](adr/009-rename-bff-to-session.md) changed the working tree and every identifier, but the GitHub repository itself was never renamed), `arm-admin` (`arm-app-admin`), `arm-ui-library` (`arm-library-ui-components`), and `arm-deploy-make` (`arm-devops-makefile`). Anything that resolves a repo by directory name — a deploy dispatch, a CI checkout, a clone script — has to use the right side of that table.

**`arm-tool-cli` sits at the workspace root, not under a category directory, and is not in `repos.conf`** — so `make clone-repos` does not fetch it and a fresh workspace does not have it. [MULTI-SERVER-MIGRATION.md](MULTI-SERVER-MIGRATION.md) Part 3 plans to register it at `tools/arm-tool-cli`; until that lands, it must be cloned by hand. What it does — scaffolding new ARM repos, including the `make descriptor` target that plugs a service into compose generation — is covered in [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md).

Which services actually run, and how each is installed and started, is a separate manifest: `deploy/arm-deploy-make/services.conf` lists **eight runnable units** (`core-be`, `core-fe`, `session`, `calendar-be`, `calendar-fe`, `admin-be`, `admin-fe`, `notification`) — the two `app` repos each contribute two. See [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md).

Terminology used across the platform:

| Term | Meaning |
|---|---|
| **Platform** | The full ARM system |
| **Core** | `arm-core-*` (BE + FE + session service) |
| **App** | A vertical product slice (Calendar, Admin) |
| **Service** | Standalone infra-facing service (Notification) |
| **Module / Feature / Flow** | UI section → bounded capability → multi-step journey |

---

## 2. High-level system shape

```text
                      ┌──────────────────────────────────────┐
                      │              Browser                 │
                      │  shell :3000 · remotes :3001/:10000  │
                      └──────────────────┬───────────────────┘
                                         │
                 ┌───────────────────────┼───────────────────────┐
                 │                       │                       │
                 ▼                       ▼                       ▼
          core-fe (host)          arm-session :5001        MF remoteEntry
          Module Federation       SESSION_ID cookie        calendar / admin
                 │                       │
                 │                       │  east-west DIRECT
                 │                       │  (never via Kong)
                 │            ┌──────────┼──────────┐
                 │            ▼          ▼          ▼
                 │        core-be   calendar-be  admin-be
                 │        PG          Mongo       PG
                 │        arm_core    arm-calendar arm_admin
                 │                       │
                 │                       ▼
                 │                 Redis (BullMQ + sessions)
                 │                 Kafka → notification → SMTP
                 └───────────────────────────────────────────────
```

**Hard rules (do not work around):**

1. **One shared session service** (`arm-session`) for the whole platform — never a per-app session service.
2. **Tokens never reach the browser** — only the opaque httpOnly `SESSION_ID` cookie.
3. **session service → backends are east-west direct** — no Kong hop on that path in any environment.
4. **Frontends call backends only through** `/session/proxy/<service>/v1/...`.

Decision rationale for the standalone session service: [`docs/architecture/session-architecture.md`](architecture/session-architecture.md).

---

## 3. Edge routing by environment (read this before drawing Kong)

Local and deployed edges are **not the same**. Baking “Browser → Kong → session service” into every diagram reintroduces [Risk 12](meta/documentation-inventory.md).

| Environment | Edge | How the browser reaches the session service / APIs |
|---|---|---|
| **Local** (`make local-dev` / `make arm-run`) | **Neither Caddy nor Kong** | Direct host ports (`:3000` shell, `:5001` session service, `:3001` / `:10000` remotes). Authoritative: `make/local-dev.mk`, `docker-compose.local.dev.yml` (“No Caddy. No Kong”). |
| **Server** (`docker-compose.yml` + `Caddyfile.prod`) | **Caddy in front of Kong** | Browser → Caddy `:443` (TLS) → `/api/*` → `kong:8000` → backends. `/session/*` → session service directly. Kong publishes no host port; Caddy is the only container publishing `:80`/`:443`. |

**There are exactly two environments.** The hand-written `docker-compose.dev.yml` (shared CI/dev host, `Caddyfile.dev`) and `docker-compose.staging.yml` were deleted on 2026-08-19 — see [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md).

**The "Kong is the edge" shorthand is half true**: Kong is the *API* edge (JWT validation, `X-User-Id` injection, rate limiting), but it is not the *browser* edge — Caddy terminates TLS in front of it. See [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §3.

Auth guards everywhere still have a dual path: trust `X-User-Id` only when `TRUST_PROXY_HEADERS=true` (real Kong); otherwise verify JWT. **Local uses the JWT path.** See [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) §3.

> Older text in root `CLAUDE.md` / some deploy READMEs that says “local uses Caddy” is **stale** relative to `docker-compose.local.dev.yml`. Prefer this section.

---

## 4. Service inventory

| Service | Host port (local) | Container port | Stack | Store | Path |
|---|---|---|---|---|---|
| **arm-core-fe** (shell) | `3000` | `3000` | React 19 + Vite + MF + UnoCSS | — | `core/arm-core-fe` |
| **arm-core-be** | `4000` | `3000` | NestJS + TypeORM + Kafka producer | PostgreSQL `arm_core` | `core/arm-core-be` |
| **arm-session** | `5001` | `5000` | NestJS + Redis session + axios proxy | Redis sessions | `core/arm-session` |
| **calendar-fe** | `3001` | `3001` | React + Vite MF remote (`base: /calendar/`) | — | `apps/arm-app-calendar/src/frontend` |
| **calendar-be** | `4001` | `4001` | NestJS + Mongoose + BullMQ | MongoDB `arm-calendar` | `apps/arm-app-calendar/src/backend` |
| **admin-fe** | `10000` | `10000` | React + Vite MF remote (`base: /admin/`) | — | `apps/arm-admin/src/frontend` |
| **admin-be** | `10001` | `10001` | NestJS + TypeORM | PostgreSQL `arm_admin` (+ reads `arm_core`) | `apps/arm-admin/src/backend` |
| **notification** | *(none published)* | `4001` in-container | NestJS Kafka consumer + SMTP | — (Kafka in, SMTP out) | `services/arm-service-notification` |
| postgres | `5432` | `5432` | Postgres 16 | — | infra compose |
| mongo | `27017` | `27017` | Mongo 7 | — | infra compose |
| redis | `6379` | `6379` | Redis 7 | — | infra compose |
| kafka | `9092` | `9092` | Kafka 3.8 KRaft | — | infra compose |

**Notes**

- Browser entry is always **`localhost:3000`**. Host `:4000` / `:4001` / `:10001` are for debugging; product FE traffic goes through the session service.
- Notification is **not** on host `:4001` — that port is calendar-be. Notification has no host `ports:` mapping in local compose.
- The shared UI package lives at `library/arm-ui-library` (GitHub repo `arm-library-ui-components`, published to npm as `@aracreate/test-arm-ui`). It is not a running service and has no compose entry — the three frontends consume it as a dependency and share it as a Module Federation singleton.
- `arm-tool-cli` and `aracreate-template-codebase` are developer tooling, not deployed units — neither appears in `services.conf` or any compose file.

---

## 5. Database & queue ownership

| Owner | Engine | What lives there |
|---|---|---|
| arm-core-be | PostgreSQL `arm_core` | Users, orgs, roles, platform auth |
| calendar-be | MongoDB `arm-calendar` | Accounts, calendars, events, sync state, Google channel metadata |
| admin-be | PostgreSQL `arm_admin` | Admin users, audit log, deny-list rows (also reads `arm_core`) |
| arm-session | Redis | Sessions / token cache (`SESSION_ID` → Redis) |
| calendar-be | Redis + BullMQ | Sync jobs, webhook renewal, poll schedulers |
| core-be → notification | Kafka | OTP / email messages (notification has **no** Redis/BullMQ) |

---

## 6. Module Federation

| Role | Federation name | Local port | Exposes | Shell routes |
|---|---|---|---|---|
| **Host** | `core` | `3000` | `./toast`, `./theme-store` | owns router, navbar, auth state |
| **Remote** | `calendar` | `3001` | `./Page-1`, `./SyncDetail`, `./SyncView`, `./MergedView`, `./AuthCallback` | `/calendar/*` |
| **Remote** | `admin` | `10000` | `./AdminApp` | `/admin/*` |

Shell route map for the calendar remote (`core-fe/src/routes/app.router.tsx`):

| Route | Exposed module |
|---|---|
| `/calendar` | redirects to `/calendar/syncs` |
| `/calendar/syncs` | `calendar/Page-1` |
| `/calendar/syncs/new`, `/calendar/syncs/:id` | `calendar/SyncDetail` |
| `/calendar/view` | `calendar/MergedView` |
| `/calendar/view/:targetCalendarId` | `calendar/SyncView` |
| `/calendar/account`, `/calendar/auth/callback` | `calendar/AuthCallback` (outside the protected layout, so the OAuth popup redirect lands) |

Local remoteEntry URLs (Compose):  
`http://localhost:3001/calendar/remoteEntry.js`,  
`http://localhost:10000/admin/remoteEntry.js`.

**CSS isolation — the `.calendar-scope` wrapper is gone.** It existed only because FullCalendar reset CSS globally. FullCalendar has been removed from the calendar app entirely (the grid is now the design system's `BigCalendarView`, built on react-big-calendar), so the wrapper was dropped from `app.router.tsx` and from `calendar-fe`'s own `App.tsx`. Do not reintroduce it. What remains true: `calendar-styles.css` is imported only by the standalone-dev `App.tsx` and therefore never loads on the federated path.

Full deep dive — annotated `vite.config.ts`/`uno.config.ts` for all three apps, shared-dependency version drift, the reverse remote→host consumption pattern, and verified CSS/UnoCSS config gaps — is in [MODULE-FEDERATION.md](MODULE-FEDERATION.md).

---

## 7. Network boundaries

```text
┌─ Public (browser) ───────────────────────────────────────────┐
│  core-fe · calendar-fe remoteEntry · admin-fe remoteEntry    │
│  arm-session (/session/*) — sets/reads SESSION_ID            │
└────────────────────────────┬─────────────────────────────────┘
                             │ session cookie / proxy
┌─ Platform internal (east-west) ──────────────────────────────┐
│  arm-session ──direct──► core-be · calendar-be · admin-be    │
│  core-be / calendar-be / admin-be                            │
│       ◄── X-Session-Secret ──► arm-session                   │
│       & each other for internal token/session calls          │
│  core-be ──Kafka──► notification ──SMTP──► mail provider     │
│  calendar-be ──BullMQ──► Redis                               │
└──────────────────────────────────────────────────────────────┘
┌─ Infra ──────────────────────────────────────────────────────┐
│  PostgreSQL · MongoDB · Redis · Kafka                        │
└──────────────────────────────────────────────────────────────┘
```

### Inter-service auth

| Path | Auth |
|---|---|
| Browser → session service | `SESSION_ID` httpOnly cookie |
| session service → backend (proxy) | Injects `Authorization: Bearer <accessToken>` from Redis session |
| Service → session service `/session/internal/*` or calendar → core token fetch | `X-Session-Secret: (secret removed) (timing-safe compare) |

`X-Session-Secret` is **not** “Kong → session service” for the normal proxy path — it is platform S2S. Env reference: [ENV-VARS.md](ENV-VARS.md).

---

## 8. Data-flow walkthroughs

### 8.1 Login (SSO / email OTP)

Password login does not exist. Both SSO and email OTP converge on:

```text
core-be issues JWT access/refresh
  → POST /session/internal/create-session (X-Session-Secret)
  → session service stores tokens in Redis, returns signed session handoff
  → browser lands on /session/oauth/callback → SESSION_ID cookie only
```

Full detail: [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md). session lifecycle: [SESSION-FLOW.md](SESSION-FLOW.md) and [`core/arm-session/README.md`](../../../core/arm-session/README.md).

### 8.2 Authenticated API call

```text
MFE → ${VITE_SESSION_URL}/session/proxy/{core|calendar|admin}/v1/...
  → SessionGuard (SESSION_ID → Redis)
  → silent refresh if access token near expiry
  → axios to CORE_BE_URL / CALENDAR_BE_URL / ADMIN_BE_URL (direct)
  → backend AuthGuard / KongJwtGuard verifies JWT
```

Proxy map: `SERVICE_ENV_MAP` in `core/arm-session` (`core` / `calendar` / `admin`). Full proxy/refresh behavior: [SESSION-FLOW.md](SESSION-FLOW.md).

### 8.3 Calendar sync (high level)

```text
Initial sync  → BullMQ job → performInitialSync (source events → target blockers)
Incremental   → Google push webhook → calendar-be webhook handler → delta apply
Fallback      → poll schedulers (also the Microsoft path — no push)
UI refresh    → SSE /session/proxy/calendar/v1/events/stream
```

Detail: `apps/arm-app-calendar/CLAUDE.md` (“Sync Architecture”). Unlink teardown: [`DELETE-CASCADE.md`](calendar/DELETE-CASCADE.md). Add Account OAuth: [`GOOGLE-OAUTH.md`](calendar/GOOGLE-OAUTH.md).

### 8.4 Webhook renewal (high level)

Google watch channels expire (~7 days). A renewal scheduler (daily / 48h window in calendar-be) enqueues BullMQ jobs to renew before expiry. If renewal stops silently, incremental sync dies without a user-visible error — tracked as a Critical doc gap (`WEBHOOKS.md` / DOC-A07, still partial via `failure-modes.md`).

---

## 9. Technology choices (short)

Every significant choice now has a written ADR under [`adr/`](adr/readme.md) — the one-liners below are the summary, the ADR is the context/decision/consequences record.

| Choice | Why (evidenced) | ADR |
|---|---|---|
| Standalone shared `arm-session` | A session layer inside core-be cannot serve every MFE without becoming a god proxy — [`session-architecture.md`](architecture/session-architecture.md) | [001](adr/001-standalone-bff.md) |
| Tokens only in Redis session | Browser-based OAuth backend-for-frontend pattern (IETF / OWASP guidance cited in that doc) | [001](adr/001-standalone-bff.md) |
| Module Federation shell + remotes | Independently deployable FE apps, one UX | [002](adr/002-module-federation.md) |
| MongoDB for calendar | Document model for events / sync / provider state | [003](adr/003-mongodb-for-calendar.md) |
| BullMQ for calendar jobs | Sync + webhook renewal on Redis; **not** used by notification | [004](adr/004-bullmq-vs-kafka.md) |
| Kafka for notification | core-be produces; notification consumes → SMTP | [004](adr/004-bullmq-vs-kafka.md) |
| Google OAuth in a popup, not a redirect | Keeps the shell's SPA state alive across the consent round-trip | [005](adr/005-google-oauth-popup.md) |
| Per-app UnoCSS instances + host cross-scan | Each MFE ships its own CSS; the shell scans calendar source for utilities | [006](adr/006-unocss-mfe-scoping.md) |
| No internal npm registry | Verdaccio was adopted then removed — `@aracreate/test-arm-ui` publishes to public npm | [007](adr/007-verdaccio-registry.md) (superseded) |
| Redis Sentinel for HA | Failover without an app-side rewrite — but `session`/`admin-be` are not Sentinel-aware yet | [008](adr/008-redis-sentinel-ha.md) |
| `bff` → `session` naming | The service is the session owner, not a generic backend-for-frontend | [009](adr/009-rename-bff-to-session.md) |
| East-west direct from `arm-session` | Kong (where present) is edge-only — never re-entered for session→backend | — |

---

## 10. Related documents

### Platform reference

| Doc | Role |
|---|---|
| [ONBOARDING.md](ONBOARDING.md) | Day-1 engineer guide — workspace tour, access, first change |
| [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) | Login, JWT, per-service guards, admin RBAC, the deny-list gap |
| [SESSION-FLOW.md](SESSION-FLOW.md) | Session lifecycle, proxy rewrite, silent refresh, Redis schema |
| [MODULE-FEDERATION.md](MODULE-FEDERATION.md) | MF host/remote deep dive, shared-dependency drift, UnoCSS scanning |
| [DATABASE-DESIGN.md](DATABASE-DESIGN.md) | Field-level schema for `arm_core`, `arm_admin`, `arm-calendar`; migration inventories |
| [REDIS-ARCHITECTURE.md](REDIS-ARCHITECTURE.md) | Every Redis key namespace, BullMQ queue inventory, the Sentinel-awareness gap |
| [KAFKA-ARCHITECTURE.md](KAFKA-ARCHITECTURE.md) | Producer/consumer wiring, topics, DLQ, KRaft cluster config |
| [SECURITY.md](SECURITY.md) | Threat model, guard pattern across all 4 backends, OWASP coverage, open issues |
| [ENV-VARS.md](ENV-VARS.md) | Every env var per service, plus the cross-repo sync constraints |
| [CODING-STANDARDS.md](CODING-STANDARDS.md) | NestJS structure, guards, facades, error handling |
| [TESTING-STRATEGY.md](TESTING-STRATEGY.md) | Test layers, per-repo coverage, the memory-safe `test:agent` runner |
| [TEMPLATE-CONFORMANCE.md](TEMPLATE-CONFORMANCE.md) | araCreate template checklist applied to this workspace |

### Infrastructure and delivery

| Doc | Role |
|---|---|
| [INFRASTRUCTURE.md](INFRASTRUCTURE.md) | Every compose file's real purpose, Kong/Caddy routing, Redis Sentinel, port map per environment |
| [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md) | `services.conf`, per-repo `make descriptor`, shared compose templates, `generated/` — **read before editing any compose block** |
| [KONG-CONFIG.md](KONG-CONFIG.md) | Routes, JWT plugin, `X-User-Id` injection, the rendered-not-authored `kong.yml` rule |
| [CICD.md](CICD.md) | Every GitHub Actions workflow across the repos, `main` → server deploy path, verified pipeline bugs |
| [RELEASE-PROCESS.md](RELEASE-PROCESS.md) | Release vs. deploy, per-repo automation, the `arm-ui-library` publish gap |
| [MONITORING.md](MONITORING.md) | Logs, metrics, tracing, alerting, dashboards; partial tracing/Faro rollout |
| [PRODUCTION-RESILIENCE-NOTES.md](PRODUCTION-RESILIENCE-NOTES.md) | General production-engineering reference (not ARM-specific) |

### Audits, risks, and plans

| Doc | Role |
|---|---|
| [PRODUCTION-AUDIT-2026-08-21.md](PRODUCTION-AUDIT-2026-08-21.md) | Dated deep production audit |
| [FAILURE-ANALYSIS-2026-08-21.md](FAILURE-ANALYSIS-2026-08-21.md) | Dated failure analysis and infrastructure risk report |
| [INFRASTRUCTURE-RISKS.md](INFRASTRUCTURE-RISKS.md) | Standing infrastructure risk register |
| [IMPLEMENTATION-ROADMAP.md](IMPLEMENTATION-ROADMAP.md) | Index over [Phase 1](roadmap/PHASE-1-EMERGENCY.md)–[Phase 7](roadmap/PHASE-7-UX-BUGS.md) |
| [MULTI-SERVER-MIGRATION.md](MULTI-SERVER-MIGRATION.md) | Single-host → S1–S4 migration plan; work breakdown in [`plan/infra/`](plan/infra/README.md) |

### Per-service and operational

| Doc | Role |
|---|---|
| [core-be/API.md](core-be/API.md) | Every arm-core-be endpoint, guards, error format |
| [calendar/ARCHITECTURE.md](calendar/ARCHITECTURE.md) | Calendar app backend + frontend, sync/webhook engine |
| [calendar/SYNC-FLOW.md](calendar/SYNC-FLOW.md), [calendar/WEBHOOKS.md](calendar/WEBHOOKS.md) | Sync algorithm; Google watch-channel registration and renewal |
| [calendar/API.md](calendar/API.md), [calendar/DATA-MODEL.md](calendar/DATA-MODEL.md) | calendar-be endpoints; Calendar-scoped schema index |
| [calendar/GOOGLE-OAUTH.md](calendar/GOOGLE-OAUTH.md), [calendar/DELETE-CASCADE.md](calendar/DELETE-CASCADE.md), [calendar/LINKED-ACCOUNTS.md](calendar/LINKED-ACCOUNTS.md) | Add Account / Reconnect flow; unlink teardown; the Linked Accounts feature |
| [`guides/local-dev-setup.md`](guides/local-dev-setup.md) | Setup steps, prerequisites, common fixes |
| [`runbooks/`](runbooks/restart-services.md) | Restart services, deploy rollback, backup/restore, DB server setup |
| [`troubleshooting/`](troubleshooting/mfe-not-loading.md) | Symptom-first triage: MFE not loading, auth failures, sync not working |

### Workspace and meta

| Doc | Role |
|---|---|
| [CLAUDE.md](../../../CLAUDE.md) | Workspace constraints + day-to-day commands |
| [`architecture/session-architecture.md`](architecture/session-architecture.md) | Why the session service exists |
| [`core/arm-session/README.md`](../../../core/arm-session/README.md) | Session service implementation |
| [`adr/readme.md`](adr/readme.md) | ADR index (001–009) |
| [`meta/documentation-inventory.md`](meta/documentation-inventory.md) | What's written, what's pending, and every verified drift found while writing each doc |
| [`meta/miro-diagrams.md`](meta/miro-diagrams.md) | The diagram backlog, prioritized |
