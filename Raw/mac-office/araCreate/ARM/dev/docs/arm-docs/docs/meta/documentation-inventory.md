# ARM Platform — Documentation Inventory

> **Generated:** 2026-05-26
> **Last refreshed:** 2026-08-27 — see [Since Last Refresh (2026-08-27)](#since-last-refresh-2026-08-27-structure-pass) for what changed; earlier refreshes are kept below for history
> **Scope:** Full platform — all services, MFEs, infrastructure, and operational documentation
> **Architecture baseline:** arm-core-fe (shell) · arm-core-be · arm-session · arm-app-calendar (fe + be) · arm-admin (fe + be) · arm-service-notification · arm-ui-library · arm-deploy-make · arm-docs · arm-tool-cli — see [ARCHITECTURE.md](../ARCHITECTURE.md) §1.1 for the directory ⇄ GitHub-repo mapping, which is **not** one-to-one
>
> **Relocated 2026-08-11:** every cross-cutting platform doc referenced by bare filename in this inventory (`ARCHITECTURE.md`, `SECURITY.md`, `DATABASE-DESIGN.md`, etc.) now lives in this repo, at `docs/` (i.e. `docs/arm-docs/docs/<file>.md` from the assembled workspace root) — not at the workspace root anymore. Per-repo docs (`API.md`, `GOOGLE-OAUTH.md`, `DELETE-CASCADE.md`, every service's own `README.md`/`CLAUDE.md`) stay in their own repos, unmoved. File-name references throughout this table are otherwise unchanged.

---

## Since Last Refresh (2026-08-27, structure pass)

A pass over the **structure** of the platform and of this document set — repo inventory, service inventory, and the navigation/index files — rather than over any one subsystem's behaviour. Verified against `repos.conf`, `services.conf`, every workspace `.git/config` remote, the three frontends' `vite.config.ts` / `uno.config.ts` / `package.json`, `core-fe/src/routes/app.router.tsx`, and `calendar-fe/src/App.tsx`.

### The repo inventory was wrong everywhere it appeared

`README.md`, `ARCHITECTURE.md` §1, and `ONBOARDING.md` each stated a different repo count ("8 repos + this one is the 9th", "seven independently deployable git repositories", "seven independent git repositories"). None matched. The workspace holds **ten** ARM git repos plus `aracreate-template-codebase` (a reference template, not an ARM service); `repos.conf` lists only **six** of them, and `services.conf` declares **eight** runnable units, because the two `app` repos each contribute a backend and a frontend.

Recorded as a new [ARCHITECTURE.md §1.1](../ARCHITECTURE.md) with the full directory tree and a directory ⇄ GitHub-repo table, because the two names differ for four repos:

| Directory | Actual GitHub repo |
|---|---|
| `core/arm-session` | `arm-core-bff` |
| `apps/arm-admin` | `arm-app-admin` |
| `library/arm-ui-library` | `arm-library-ui-components` |
| `deploy/arm-deploy-make` | `arm-devops-makefile` |

**`arm-session`'s remote is still `arm-core-bff.git`.** The [rename](../adr/009-rename-bff-to-session.md) changed the working tree, every identifier, and every doc — but not the GitHub repository itself. Anything resolving a repo by directory name (a deploy dispatch, a CI checkout) needs the right-hand column.

**`arm-tool-cli` sits at the workspace root and is in no manifest** — not `repos.conf`, not `services.conf`. `make clone-repos` does not fetch it, so a fresh workspace does not have it. [MULTI-SERVER-MIGRATION.md](../MULTI-SERVER-MIGRATION.md) Part 3 plans to register it at `tools/arm-tool-cli`; until then it is a manual clone, and that was written down nowhere.

### The `.calendar-scope` / FullCalendar rule is obsolete — corrected in five places

`.calendar-scope` existed for exactly one reason: FullCalendar injected reset CSS at global scope. **FullCalendar is gone from `apps/arm-app-calendar/src/` entirely** (the grid is now the design system's `BigCalendarView` on react-big-calendar); it survives only as a stale `package-lock.json` entry. Nothing in either repo references `.calendar-scope` any more — the wrapper was removed from both `core-fe/src/routes/app.router.tsx` and calendar-fe's own `App.tsx`.

This had propagated into five live documents, each instructing a reader to add or preserve a wrapper that no longer has a cause: [ARCHITECTURE.md](../ARCHITECTURE.md) §6, [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) (§5 rewritten — its "brand theming never applies when federated" gap is **closed by deletion**, not by a fix), [calendar/ARCHITECTURE.md](../calendar/ARCHITECTURE.md) §9, [troubleshooting/mfe-not-loading.md](../troubleshooting/mfe-not-loading.md) §1/§4, and [miro-diagrams.md](miro-diagrams.md)'s FE-03 spec — which would have had someone *draw* the obsolete rule into a permanent diagram.

**The DOC-P02 and DOC-A17 suggested-sections lists below still name `.calendar-scope` and FullCalendar** (this file, lines ~199 and ~1503–1507, and the DOC-P02 gap note at ~1637). Those are preserved as the historical record of what was suggested at the time — **do not write to them.** MODULE-FEDERATION.md §5 is the current answer.

### Structural drift found while re-verifying Module Federation

| Finding | Was documented as | Actually |
|---|---|---|
| `calendar` exposes **five** modules | four (`./MergedView` missing) | `./Page-1`, `./SyncDetail`, `./SyncView`, `./MergedView`, `./AuthCallback` |
| remoteEntry URLs carry the remote's `base` | `http://localhost:3001/remoteEntry.js` | `http://localhost:3001/calendar/remoteEntry.js`, `http://localhost:10000/admin/remoteEntry.js` |
| Shell calendar routes | `/calendar`, `/calendar/sync/view/:id` | `/calendar/syncs*` and `/calendar/view*` are canonical; `/calendar/sync*` persists only in calendar-repo call sites |
| `react-router-dom` shared pin | `^7.8.2` | `^7.18.2` in all three |
| `@aracreate/test-arm-ui` shared pin | `2.0.4` / `2.0.4` / not shared | **`3.1.4` / `3.1.5` / `3.1.4`** — admin *does* share it now |
| `@radix-ui/themes` | not mentioned | shared singleton `^3.3.0` in all three |
| Remote imports wrapped in `timedLazy()` (15s timeout) | not mentioned | `core-fe/src/utils/load-remote-module.ts` |

**New gap, recorded not fixed:** `@aracreate/test-arm-ui` is pinned to **exact** versions that disagree — `calendar-fe` on `3.1.5`, host and admin on `3.1.4`. As a federation singleton the host's `3.1.4` wins for every app in the session, so a calendar component written against a `3.1.5`-only export typechecks, passes standalone dev on `:3001`, and fails only once federated behind the shell. The React `19.2.0` vs `19.1.1` drift already recorded is the same mechanism with a benign outcome; this one is not necessarily benign. See [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §4 and §9.

Also newly documented rather than newly broken: calendar-fe has **four** stylesheets, not one, and only `calendar-styles.css` was ever described here. `theme-overrides.css` is a deliberate verbatim copy of core-fe's (divergence = two different yellows), and `calendar-ds.css` patches in classes the shared library's own components render but ship no styles for — the real fix for which is upstream in the library's `tokens.css`. MODULE-FEDERATION.md §5.3 is new.

### Navigation was missing a third of the document set

`_sidebar.md` listed 63 of the 100 markdown files in `docs/`. Entirely absent: the seven-phase [roadmap](../IMPLEMENTATION-ROADMAP.md), the 13-file [`plan/infra/`](../plan/infra/README.md) breakdown, [MULTI-SERVER-MIGRATION.md](../MULTI-SERVER-MIGRATION.md), both dated audits, [INFRASTRUCTURE-RISKS.md](../INFRASTRUCTURE-RISKS.md), [TESTING-STRATEGY.md](../TESTING-STRATEGY.md), [DEPLOY-ORCHESTRATION.md](../DEPLOY-ORCHESTRATION.md), [ADR-009](../adr/009-rename-bff-to-session.md), the three `BFF-RENAME-*` records, and two `architecture/` design specs. Docsify's sidebar is the only nav this site has, so an unlisted file was effectively unpublished.

Rewritten with all 100 files, regrouped (Platform Reference · Infrastructure & Delivery · Audits & Risk Registers · Roadmap & Plans · Multi-Server Migration · ADRs · Proposals · Guides · Runbooks · Troubleshooting · Per-Service · History · Meta), and every link verified to resolve. `README.md`'s layout tree and `docs/readme.md`'s subfolder table were stale in the same way and now match. Three sidebar labels mangled by the bff→session sed pass ("session service Session Flow", "ADR-001: Standalone session service") were fixed.

### Stale "doesn't exist yet" claims removed

Three documents still told readers a document didn't exist that has existed for weeks:

- `ONBOARDING.md` — "A single consolidated platform architecture document doesn't exist yet (`DOC-P01`)". [ARCHITECTURE.md](../ARCHITECTURE.md) has existed since 2026-08-10. Replaced with a read-this-next ordering.
- `ARCHITECTURE.md` §9 — "No formal ADR files exist yet (DOC-P16)". Nine ADRs exist. The decision table now links each row to its ADR.
- `ARCHITECTURE.md` §10 — "Still thin / not written as dedicated docs: `MODULE-FEDERATION.md`, `INFRASTRUCTURE.md`, ADR collection". All three exist. §10 is now a four-table index over the whole document set.

`ONBOARDING.md`'s access list also still required **Verdaccio** access to install `@aracreate/*`. Verdaccio was removed from the stack — see [ADR-007](../adr/007-verdaccio-registry.md); the package publishes to public npm, so installing needs no special access at all. A new engineer would have blocked on requesting access to a registry that does not exist.

### Not fixed in this pass

- **Root `CLAUDE.md` says the notification service sends "email via BullMQ + SMTP".** It is Kafka — `kafkajs` + `@nestjs/microservices`, no BullMQ or ioredis dependency at all. The docs here have been correct on this since 2026-08-10; the workspace `CLAUDE.md` is the file that is wrong, and it is outside this repo.
- **Root `CLAUDE.md` says "7 independent git repositories".** Same correction as §1.1 above, same reason it is out of scope here.
- **`core-fe/vite.config.ts`'s UnoCSS cross-scan glob still resolves to a nonexistent directory** (`../calendar/src/**` → `core/calendar/src`). Unchanged since first recorded; `uno.config.ts` has the correct path. Still MODULE-FEDERATION.md §6 / §9.

---

## Since Last Refresh (2026-08-19, Kong edge fixes)

**`make edge-up` was added** — Kong and Caddy in front of the running local stack, using the same config the server mounts (see [DEPLOY-ORCHESTRATION.md](../DEPLOY-ORCHESTRATION.md) §2b). It closes the edge half of the gap the environment collapse opened: with dev and staging gone, Kong/Caddy/Authelia and the observability stack exist only on the server. The observability half is still open.

**It found two live Kong bugs on its first run**, both now fixed and both added to [SECURITY.md](../SECURITY.md) §11:

| ID | Bug | Why it went unnoticed |
|---|---|---|
| `SEC-006` | `pre-function` Lua called `require 'cjson'`, which Kong's sandbox forbids — every `/api/*` request carrying a token returned 500 *from Kong*, so `X-User-Id` injection had never once run | Nothing calls Kong's `/api/*` paths; the browser goes through the session service, which Caddy routes past Kong |
| `SEC-007` | Kong's JWT consumer secret was the literal, repo-committed string `${KONG_JWT_SECRET}` — declarative config is not interpolated, so that placeholder *was* the HMAC key. Genuine tokens 401'd; tokens signed with the public placeholder were accepted | Same reason, plus `SEC-006`'s 500 masked it entirely — everything failed closed |

Numbered 006/007 to continue the platform sequence; `SEC-001`–`SEC-005` were already taken, and the `SEC-001` referenced in SECURITY.md §3 is the unrelated 2026-07-02 `X-User-Id` trust bug.

**Structural change:** `kong/kong.yml.tmpl` became `kong/kong.yml.tmpl`, rendered to `generated/kong.yml` by `make generate-kong-config`. Any doc referring to `kong/kong.yml.tmpl` as a live config file is now wrong — a file that looked authored but held an unsubstituted placeholder is precisely how `SEC-007` survived.

**Docs updated in this pass:** `INFRASTRUCTURE.md` (§3, §4 + new §4.1), `KONG-CONFIG.md` (header, consumer setup, the `pre-function` section, §7 reload procedure), `SECURITY.md` (§3, §7, §11 table, §13 risk list), `DEPLOY-ORCHESTRATION.md` (new §2a, §2b), this file.

**Still open, unchanged:** the observability stack has no local counterpart, and `RB-08` (Kong config reload runbook) is still unwritten — though the procedure has changed and is now recorded in `KONG-CONFIG.md` §7: re-render, then restart.

---

## Since Last Refresh (2026-08-19, environment collapse)

**The platform went from four environments to two: local and server.** `docker-compose.dev.yml` (the shared CI/dev host, 758 lines) and `docker-compose.staging.yml` (640 lines) were deleted, along with `Caddyfile.dev`, the dev/staging Prometheus + Alloy + Authelia configs, the `dev`/`staging`/`build-dev`/`build-staging`/`ci-deploy-dev`/`ci-deploy-staging` make targets, and the `deploy-dev.yml` / `deploy-staging.yml` workflows. Net effect on the deploy repo: 28 files changed, −2,557 lines.

The two deleted environments were the only ones the descriptor engine did not generate, so removing them took generation coverage to 100% and reduced the observability stack from three near-identical copies to one. Identifiers stay `prod` in code (`COMPOSE_PROD`, `generated/prod/`, `--env-scope prod`, `PROD_*` descriptor keys) because those keys are declared in each service's own repo; "server" is the word used in prose.

**Findings closed by the deletion** — these had been recorded as open in the pass below, and are closed by removal rather than by fixing:

| Finding | Recorded in | Now |
|---|---|---|
| Admin app has no path to staging — no service in `docker-compose.staging.yml` while Kong routed `/api/admin` to it | DOC-P05, DOC-P06 | **Closed** — staging deleted; the server has admin wired end to end |
| Staging's `verify` job checks admin containers its own compose file never defines | DOC-P06 | **Closed** — the workflow was deleted |
| CI/dev and staging compose files hand-written while local and prod are descriptor-generated | DEPLOY-ORCHESTRATION §2 | **Closed** — no hand-written app service block remains |
| Dev/staging have no image tagging and no rollback job | DOC-P06, RB-07 | **Closed** — the server is the only deploy target and has the full tag → verify → auto-rollback loop |
| Dev/staging `verify` checks container status only, not health | DOC-P06 | **Closed** — the server polls both |
| Observability config triplicated across three environments, drifting silently when only one is edited | DOC-P13 | **Closed** — one copy of each config |

**New gap surfaced by the collapse, recorded not fixed:** Kong, Caddy, Authelia, and the entire observability stack now exist **only** on the server, so a change to any of them gets no pre-server exercise — local dev runs none of them. The same applies to schema migrations: with no pre-production environment, a migration's first real run is on the server, and `make rollback` is images-only by design. Tracked in `deploy/arm-deploy-make/TASK_SHEET.md`; the mitigation is `make backup-all` + `make restore-postgres-drill` as a gate on any migration deploy.

**Docs updated in this pass:** `INFRASTRUCTURE.md` (§1–§3, §6–§8, §10), `CICD.md` (§2–§4, §6, §8), `DEPLOY-ORCHESTRATION.md` §2, `ARCHITECTURE.md` §3, `MONITORING.md` §1, `SECURITY.md`, `ENV-VARS.md`, `AUTH-ARCHITECTURE.md`, `KONG-CONFIG.md`, `ONBOARDING.md`, `RELEASE-PROCESS.md`, `PRODUCTION-RESILIENCE-NOTES.md`, `guides/github-setup.md` (collapsed to one environment), `runbooks/{deploy-rollback,restart-services,db-backup-restore}.md`, `troubleshooting/auth-failures.md`, `architecture/sso-otp-auth-tasks.md`, `meta/miro-diagrams.md`.

**Mentions of "staging" deliberately left in place:** application-code predicates that still test `NODE_ENV` against `'staging'` (`SESSION-FLOW.md`, `AUTH-ARCHITECTURE.md` §2, `core-be/API.md`) — that code is unchanged, so rewriting the docs would make them wrong; `arm-core-be`'s own `deploy.yaml`, which still declares a `staging` branch trigger and was already broken for unrelated reasons; and the historical refresh entries below.

---

## Since Last Refresh (2026-08-19)

A re-verification pass against `deploy/arm-deploy-make` after the descriptor-driven deploy rollout and the 2026-08-17/18 infrastructure work. **Several findings recorded below as open gaps have since been closed** — they are listed here rather than deleted in place, so a reader who has seen an older copy of a doc can tell what changed and why.

**New document:**

- **DEPLOY-ORCHESTRATION.md** — ✅ Done (2026-08-19). Net-new, no prior inventory entry: the `services.conf` manifest, the per-repo service descriptor contract, the `orchestrator` engine, the 5 shared KIND+FE_BE template categories, and `generated/` + `generated/prod/`. Written because **local dev's and production's app service blocks are no longer hand-written** and nothing in this doc set said so — an engineer following `INFRASTRUCTURE.md` §2 would have edited a generated file that the next `make local-dev` overwrites. Also recorded what the rollout did *not* cover at the time: `docker-compose.dev.yml` and `docker-compose.staging.yml` were still hand-written, so descriptor changes reached two of four environments. Both were deleted later the same day — see the section above.

**Findings closed since the 2026-08-10 pass** (each corrected in place in the doc that recorded it):

| Finding | Recorded in | Now |
|---|---|---|
| Admin app has no path to staging or production | DOC-P05, DOC-P06 | **Production closed** — real compose services, a Kong `admin-api` route, and matching verify checks. **Staging still open** |
| Kong's admin API open on all interfaces in dev | DOC-P05 | Closed — `127.0.0.1:8001` in all three environments, with a separate `8100` status listener for metrics |
| Backups local-disk-only, unscheduled, no MongoDB restore drill | DOC-P05, RB-05 | Closed — `backup-sync` (S3), `backup-cron-install` (daily 02:00), `restore-mongodb-drill`. `arm_admin` drill and drill scheduling remain open |
| `rollback` coded but unreachable from any workflow | DOC-P06, RB-07 | Closed for production — per-deploy `IMAGE_TAG`, health-gated `.last-good-tag`, automatic `rollback-on-failure`. Dev/staging unchanged |
| Observability and TLS only on the CI/dev host | DOC-P13, DOC-P05 | Closed — Caddy, Authelia, and all 10 observability containers now run in staging and production, with per-environment Prometheus/Alloy configs |
| Staging backends host-exposed, bypassing Kong | DOC-P05 | Closed — `ports:` entries removed |
| Prod `.env` written via `printf '${{ secrets.X }}'` chains | DOC-P06 | Closed — `env:`-bound heredoc + `scp`, no secret in command text |

**Findings that survived re-verification, unchanged:** `repos.conf` pins every app repo to `dev` for all three environments; `arm-library-ui-components`'s clone path doesn't match `library/arm-ui-library`; `arm-admin` has zero CI (now *more* consequential — its code reaches production while skipping staging entirely); `core-be`'s broken Kubernetes `deploy.yaml`; `session`/`admin-be` are not Sentinel-aware (and `REDIS_HA=yes` now makes turning Sentinel on a one-flag decision, so this matters more, not less); `nginx.dev.conf` is dead code; `deploy/arm-deploy-make/CLAUDE.md`'s Observability section still lists four shipped things as deferred.

**Docs updated in this pass:** `INFRASTRUCTURE.md` (§1–§4, §6–§10), `CICD.md` (§3, §4, §6, §8), `runbooks/deploy-rollback.md` (rewritten around the working production rollback), `ARCHITECTURE.md` §3 (Caddy now fronts Kong in staging/prod), `MONITORING.md` §1, `KONG-CONFIG.md` §1/§6/§7, `ENV-VARS.md` (7 previously-undocumented vars: `SITE_DOMAIN`, `ADMIN_FRONTEND_URL`, 5 × `(secret removed)`), `REDIS-ARCHITECTURE.md` §1.

**New gap surfaced, not yet actioned:** `SITE_DOMAIN` is deploy-blocking for the server (Caddy cannot obtain a certificate without it) but is in neither `REQUIRED_CORE_VARS` nor `REQUIRED_OPTIONAL_VARS`, so `env-check` passes without it — see `ENV-VARS.md` §2.6.

---

## Since Last Refresh (2026-08-10)

A codebase pass turned up the following since this inventory was generated. Individual entries below are tagged `Status:` where something changed; this section is the summary.

**Major finding — every repo now has a current, substantial `CLAUDE.md`:** All seven repos' `CLAUDE.md` files were refreshed together on 2026-08-07/08 (root `CLAUDE.md` is current too) and are far more comprehensive than typical AI-context files — source structure, API endpoint maps, auth flows, MF wiring, env vars, and explicit "what not to do" constraints, per repo. They are not a substitute for the onboarding-friendly docs this inventory scopes (dense, terse, written for an agent, no diagrams, no "why" narrative), but they mean the *technical content* behind a large slice of the "Critical/Important, Not Started" items already exists and mostly just needs to be adapted/expanded into the target doc rather than researched from scratch. Every App entry below now carries a `Status` note pointing at the relevant `CLAUDE.md` section(s); several Platform entries do too (DOC-P01–P03, P05, P08, P09).

**Written or substantially covered since generation:**

- **DOC-P07 (Local Dev Setup)** — ✅ Done. `docs/guides/local-dev-setup.md` now exists.
- **DOC-P13 (Monitoring & Logging)** — 🟡 In progress. `docs/architecture/observability-plan.html` plus four implementation plans in `docs/superpowers/plans/` (Grafana observability stack, alerting, Faro frontend errors, Tempo tracing + Kong metrics) cover most of the intended content but live as plans, not a settled reference doc.
- **DOC-P15 (Kong Config)** — 🟡 In progress. `docs/superpowers/plans/2026-08-08-tempo-tracing-kong-metrics.md` covers the Prometheus plugin; access-control plans (`2026-08-07-authelia-caddy-grafana-access.md`, `2026-08-07-caddy-mtls-grafana-access.md`) cover edge access to Grafana specifically, not the full route/plugin inventory.
- **DOC-P04 (Auth Architecture)** — ✅ Done (2026-08-10), written as `AUTH-ARCHITECTURE.md`. **Correction to this section's earlier read:** the SSO/email-OTP migration is not "in flight" — it is done. `core/arm-core-be/CLAUDE.md` (2026-08-07) states password login/signup/reset "were removed entirely"; `core/arm-core-fe/CLAUDE.md`'s route map and `core-be/README.md` (2026-07-28) confirm the same. `docs/architecture/sso-otp-auth-tasks.md` (2026-07-14, all tasks shown 🔲 Pending) is a **stale snapshot from mid-migration**, not the current status — don't trust its task table at face value. Writing the doc also surfaced two factual corrections against source: `core-be/CLAUDE.md`'s JWT-secret cross-repo claim was wrong (only `JWT_ACCESS_SECRET` needs to match `arm-app-calendar`, not `JWT_REFRESH_SECRET` too), and `apps/arm-admin/src/backend/CLAUDE.md`'s deny-list "hot-path check in core-be and calendar-be guards" doesn't exist in either guard — see new **Risk 13**.
- **DOC-P08 (Environment Variables Reference)** — ✅ Done (2026-08-10), written as `ENV-VARS.md`. Consolidated from `deploy/arm-deploy-make/.env.example`, every service `.env.example`, `REQUIRED_*_VARS` in `config.mk`, and per-repo CLAUDE notes. Also records known drifts (github-setup `hex 16` vs `hex 32`, calendar-fe/`core-be` `:5000` session service URL examples, missing `SESSION_INTERNAL_SECRET`/`REDIS_URL` in calendar-be `.env.example`).
- **DOC-A05 (Calendar Google OAuth)** — ✅ Done (2026-08-10), written as `apps/arm-app-calendar/GOOGLE-OAUTH.md`. Source-verified against session service controller + calendar google/auth services + both FE entry points. Corrections vs the original inventory outline: Add Account success uses **BroadcastChannel** (`google_auth_done` / `GOOGLE_AUTH_SUCCESS`), not primary `postMessage`; Reconnect is a separate Update Account path (`GOOGLE_REDIRECT_URI`, encrypted state) not a re-run of Add Account session service routes; local-dev BroadcastChannel cross-origin (`:3000` vs `:5001`) and Settings `useOAuthPopup` listener gap documented.
- **DOC-A08 (Delete Cascade Spec)** — ✅ Done (2026-08-10), written as `apps/arm-app-calendar/DELETE-CASCADE.md`. Source-verified against `AccountConnectionService.deleteAccount`. Documents actual order (richer than the old wishlist: Google-native blocker sweep, source-role cleanup, AC-3 orphans), Mongo `_id` path-param contract, best-effort non-transactional semantics, and live gaps (poll schedulers not cleared, no token revoke, no Microsoft subscription cleanup, e2e using wrong id type).
- **DOC-P01 (Platform Architecture Overview)** — ✅ Done (2026-08-10), written as `ARCHITECTURE.md`. Resolves **Risk 12** for the architecture doc: local = neither Caddy nor Kong (compose-authoritative); CI/dev = Caddy edge; prod-oriented compose = Kong. Also corrects stale root CLAUDE claims (CalendarModule expose name; notification as Redis/BullMQ; notification on host `:4001`).
- **DOC-P03 (session service Session & Proxy Flow)** — ✅ Done (2026-08-10), written as `SESSION-FLOW.md`. Source-verified lifecycle (OTP vs SSO cookie paths), Redis schema, silent refresh, proxy rewrite, dedicated OAuth pointer. Corrections: inventory “parseUrl bug” was actually in `forward()` and is **fixed**; OTP does **not** use `/session/internal/create-session` (AUTH-ARCHITECTURE §1 updated); `X-Session-Secret` is S2S not Kong→session service.
- **DOC-A07 (Webhooks)** — ✅ Done, see `calendar/WEBHOOKS.md`. **TS-01 (Calendar Sync Not Working)** — still 🟡 Partial. `apps/arm-app-calendar/docs/failure-modes.md` is a source-verified failure-mode runbook (Mongo down, Redis down, core-be down, Google OAuth/Calendar API down, notification service down — note its Microsoft Graph section is now stale, see `WEBHOOKS.md` §8) — closer to a troubleshooting doc than TS-01's original scope but covers real adjacent gaps. `WEBHOOKS.md` and `SYNC-FLOW.md` now add the architecture-level content failure-modes.md doesn't cover; TS-01 itself (a dedicated troubleshooting doc synthesizing both) is still not written.
- **DOC-P06 (CI/CD) / DOC-P18 (Release) / DOC-P25 (Verdaccio)** — 🟡 Partial. `docs/guides/github-setup.md` (secrets/variables) and `docs/guides/npm-publish.md` (package release workflow) each cover one slice; no doc ties CI/CD, release, and Verdaccio publish together end-to-end yet.

**New gap — arm-admin app, closed:** `apps/arm-admin/` (backend + frontend, restructured mid-2026-08 to `src/backend/` + `src/frontend/`) did not exist at the original inventory date. Both `src/backend/README.md` and `src/frontend/README.md` were unedited scaffolding placeholders (`ac-superapp-backend-nest-template` / `ac-superapp-frontend-template`) — **both now rewritten** (DOC-A18, DOC-A19), ported from their respective `CLAUDE.md` files and re-verified against source. See **Risk 11** (now resolved) below.

**Gap closed 2026-08-12 — multi-provider calendar accounts:** `failure-modes.md` and `apps/arm-app-calendar/CLAUDE.md` both document live Microsoft Graph API handling — "Google fully featured, Microsoft is CRUD-only (no push notifications)", cross-provider syncs supported — not just a placeholder `provider` field as DOC-A10 originally assumed. DOC-A10 (Linked Accounts) is now written and covers the Microsoft provider as a real integration throughout, not future groundwork — see its own entry for what else the write-up surfaced (a UI-unreachable legacy account-creation path, and an implemented-but-unexposed pause/resume control).

**Architecture doc drift to verify before writing DOC-P05/DOC-P15:** `deploy/arm-deploy-make/docker-compose.local.dev.yml` states outright "No Caddy. No Kong" for local dev, while `CLAUDE.md` documents Caddy as the local-dev reverse proxy. Reconcile which is current before either doc is finalized.

---

## How to read this document

Documents are grouped first by **priority** (Critical → Important → Optional), then by **category** (Platform vs. App). Each entry includes:

- **Purpose** — what it explains
- **Category** — Platform (cross-cutting) or App (service/MFE-scoped)
- **Type** — Technical or Non-technical
- **When** — Now (write immediately) · Before prod · Later (low urgency)
- **Suggested sections** — recommended internal structure
- **Gap note** — why it is missing or risky (where applicable)

---

## Table of Contents

- [Since Last Refresh (2026-08-10)](#since-last-refresh-2026-08-10)
- [CRITICAL Priority](#critical-priority)
  - [Platform — Critical](#platform--critical)
  - [App — Critical](#app--critical)
  - [Runbooks — Critical](#runbooks--critical)
  - [Troubleshooting — Critical](#troubleshooting--critical)
- [IMPORTANT Priority](#important-priority)
  - [Platform — Important](#platform--important)
  - [App — Important](#app--important)
  - [Runbooks — Important](#runbooks--important)
  - [Troubleshooting — Important](#troubleshooting--important)
- [OPTIONAL Priority](#optional-priority)
  - [App — Optional](#app--optional)
- [Final Risk Report](#final-risk-report)
- [Master Summary Table](#master-summary-table)

---

# CRITICAL Priority

> These documents must exist before the platform handles production traffic or onboards new engineers. Their absence directly causes incidents, data loss, or onboarding failure.

---

## Platform — Critical

---

### DOC-P01 — Platform Architecture Overview

| Field | Value |
|---|---|
| **File** | `ARCHITECTURE.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `ARCHITECTURE.md`. Covers service inventory, edge-by-environment (§3 resolves Risk 12), MF exposes, DB/queue ownership, and login/proxy/sync/webhook walkthroughs with links out |

**Purpose:** Single source of truth for how all ARM services relate to each other. The canonical read-first document before touching any individual service.

**Suggested sections:**
1. High-level system diagram (Kong → session service → backends → infra)
2. Service inventory table (name, port, language, database, repo path)
3. Module Federation host/remote relationship
4. Data flow walkthrough: login, authenticated API call, calendar sync, webhook renewal
5. Network boundary diagram (public / internal / east-west)
6. Technology choices rationale (brief — deep dives go in ADRs)

**Gap note:** Written 2026-08-10. `docs/architecture/session-architecture.md` remains the session service *decision* deep dive; this overview is the cross-platform map.

---

### DOC-P02 — Module Federation Architecture

| Field | Value |
|---|---|
| **File** | `MODULE-FEDERATION.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `MODULE-FEDERATION.md`, linked from `ARCHITECTURE.md` §6/§10 and root `CLAUDE.md`'s Key Documents table. Verified against all three apps' `vite.config.ts`/`uno.config.ts` and `package.json` rather than trusting the per-repo `CLAUDE.md` files at face value; found and documented four real drifts: (1) `core-fe/vite.config.ts`'s UnoCSS cross-scan path for calendar resolves to a nonexistent `core/calendar/src` directory (`uno.config.ts` has the correct path), (2) `calendar-styles.css`'s `.calendar-scope .fc` FullCalendar theme overrides never load in the federated path — only in the standalone-only `App.tsx` entry — so brand theming silently doesn't apply when embedded in the shell, (3) host/remote `react`/`react-dom` `shared` version pins disagree (`19.2.0` vs `19.1.1`, singleton negotiation silently picks the host's), (4) `admin-fe` doesn't use the shared `@aracreate/test-arm-ui` UnoCSS preset the other two apps use. Also confirmed the reverse remote→host consumption pattern (`core/toast`, `core/theme-store` dynamically imported with no static `remotes` declaration) and that `@module-federation/vite` (not `@originjs`) is the plugin in use everywhere |

**Purpose:** Explains how the shell loads remotes, how shared dependencies are negotiated, how UnoCSS scanning works across MFE boundaries, and how federation behaves differently in dev vs. production builds.

**Suggested sections:**
1. Host/remote registration (`vite.config.ts` exposes and consumes — annotated examples)
2. Shared dependency negotiation and version pinning strategy
3. UnoCSS cross-MFE scanning (path configuration in `vite.config.ts` and `uno.config.ts`)
4. CSS isolation strategy (`.calendar-scope` wrapper pattern, `virtualExposes` vs. `App.tsx` entry point difference)
5. Local dev workflow (running shell and remotes simultaneously)
6. Known pitfalls (calendar-styles.css only loaded in App.tsx — not in federation path)
7. Adding a new MFE remote — step-by-step checklist

---

### DOC-P03 — session service Architecture & Session Flow

| Field | Value |
|---|---|
| **File** | `SESSION-FLOW.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `SESSION-FLOW.md`. Covers login→logout lifecycle (OTP vs SSO), cookie/Redis schema, proxy/`SERVICE_ENV_MAP`, silent refresh, internal auth; notes parseUrl/forward query bug as fixed and `session/activate` as unwired |

**Purpose:** Runtime behavior of arm-session — session lifecycle, proxy mechanics, token injection, and Redis session store. Distinct from `session service-ARCHITECTURE.md` which covers only the architectural decision.

**Suggested sections:**
1. Session lifecycle: login → Redis write → cookie issue → request → token inject → logout
2. SESSION_ID cookie attributes (httpOnly, SameSite, Secure, domain scope)
3. Redis session schema (fields stored, TTL, key format)
4. Proxy route pattern (`ALL /session/proxy/{service}/{rest}`) and `SERVICE_ENV_MAP`
5. Known bug: query param duplication in `parseUrl()` — workaround and fix status
6. OAuth redirect chains — why 302 cannot be proxied generically → dedicated `/session/calendar/add-account`
7. Silent token refresh flow
8. X-Session-Secret header (internal authentication — platform S2S, not Kong→session service for proxy)

**Gap note:** Written 2026-08-10. Historical query-param duplication is fixed (`buildTargetUrl`); open limitations remain multipart buffering and cookie maxAge vs env TTL drift.

---

### DOC-P04 — Authentication & Authorization Architecture

| Field | Value |
|---|---|
| **File** | `AUTH-ARCHITECTURE.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `AUTH-ARCHITECTURE.md`, linked from root `CLAUDE.md`'s Key Documents table. Assembled from the 3 `CLAUDE.md` files plus `apps/arm-admin/src/backend/CLAUDE.md`; two claims corrected against source in the process (JWT_REFRESH_SECRET cross-repo match claim was wrong; the disable-user deny-list is not actually checked by core-be/calendar-be guards — see Risk 13) |

**Purpose:** All auth concerns across the platform: JWT strategy, session-based auth via session service, Google OAuth for calendar accounts, and role/guard strategy in NestJS services.

**Suggested sections:**
1. Platform auth flow: credentials → core-be → session service → SESSION_ID cookie
2. JWT structure (claims, signing algorithm, expiry)
3. Google OAuth flow for calendar accounts (popup → postMessage → session service → calendar-be callback)
4. Auth guard implementation per service (how each NestJS service validates the Bearer token from session service)
5. Role-based access control (registry, admin routes)
6. Multi-account model (one ARM user → multiple Google accounts)
7. Token expiry and reconnect flow
8. Security constraints (CORS, SameSite, Secure flag requirements)

---

### DOC-P05 — Infrastructure & Deployment Guide

| Field | Value |
|---|---|
| **File** | `INFRASTRUCTURE.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10), **re-verified and corrected 2026-08-19** — findings (3), (4), and (5) below have since been closed or narrowed; see [Since Last Refresh (2026-08-19)](#since-last-refresh-2026-08-19). Written at `INFRASTRUCTURE.md`, linked from `ARCHITECTURE.md` §6/§10 and root `CLAUDE.md`'s Key Documents table. Verified against every `docker-compose*.yml`, `kong/kong.yml.tmpl`, `Caddyfile.dev`, `nginx.dev.conf`, and `make/backup.mk` rather than trusting `deploy/arm-deploy-make/CLAUDE.md` at face value. Found: (1) `nginx.dev.conf` is dead code, referenced nowhere; (2) `CLAUDE.md`'s claim that Tempo is deferred/Phase B is stale — Tempo already runs in `docker-compose.dev.yml`; (3) the admin app has no path to staging or production — absent from both compose files and from `kong.yml`; (4) Kong's admin API is open on all interfaces in dev, loopback-only in staging/prod; (5) RB-05's backup/restore tooling already exists and works (`make backup-postgres`/`backup-mongodb`/`restore-postgres-drill`) — local-disk-only, Postgres-only restore-drill, no MongoDB or `arm_admin` drill; (6) DOC-P25 (Verdaccio) documents infrastructure that has been removed from the stack entirely — `npm publish --access public` replaced it |

**Purpose:** Complete reference for all infrastructure components — what runs where, how they connect, and how to stand up or tear down each environment.

**Suggested sections:**
1. Infrastructure component inventory (Kong, Redis, PostgreSQL, MongoDB, Kafka, Verdaccio, MinIO)
2. Docker Compose files reference:
   - `docker-compose.yml` — production
   - `docker-compose.infra.yml` — infrastructure only
   - `docker-compose.infra.prod.yml` — production infrastructure
   - `docker-compose.infra.redis-sentinel.yml` — HA Redis
   - `docker-compose.dev.yml` — local development
3. Caddy / Nginx configuration (dev routing, `nginx.dev.conf`, `Caddyfile.dev`)
4. Kong configuration (`kong/kong.yml.tmpl` → `generated/kong.yml` — routes, plugins, upstream definitions)
5. Redis Sentinel setup (HA config, failover behavior)
6. Network topology per environment (local / server)
7. Port reference table (all services and their ports)
8. Volume and persistence strategy

---

### DOC-P06 — CI/CD Pipeline Documentation

| Field | Value |
|---|---|
| **File** | `CICD.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10), **re-verified and corrected 2026-08-19** — findings (4) and (6) below have since been closed or narrowed to staging; see [Since Last Refresh (2026-08-19)](#since-last-refresh-2026-08-19). Written at `CICD.md`, linked from root `CLAUDE.md`'s Key Documents table. Verified against every `.github/workflows/*.yml` in all 8 repos plus `make/ci.mk`, `repos.conf`, and git history — not just `deploy/arm-deploy-make/CLAUDE.md`'s summary. Found six real gaps: (1) `repos.conf` pins every app repo to `dev` for all three deploy environments — staging/prod deploy `dev` HEAD, not a promoted branch; (2) `arm-library-ui-components`'s configured clone path doesn't match the actual `library/arm-ui-library` directory; (3) `arm-admin` has zero `.github/workflows/` — no CI at all; (4) staging/prod `verify` jobs check for admin containers those environments' compose files never define (dated to commit `2f4ec207`, 2026-06-30, not yet exercised since `main` predates it); (5) `core-be`'s `deploy.yaml` is a live, currently-broken Kubernetes deploy workflow (references a nonexistent `k8s/` directory) with no relation to the platform's real Docker Compose + SSH model — fails on every push to `dev`/`staging`/`main` in that repo; (6) the `rollback` Makefile target requires a manual `ci-build`+`ci-push` that no automated workflow performs, so it's coded correctly but currently unreachable from any real deploy |

**Purpose:** Documents all GitHub Actions workflows, the branch-to-environment mapping, and the Makefile deploy targets used in `arm-deploy-make`.

**Suggested sections:**
1. Branch strategy (`main` → server; `dev` runs CI only)
2. Workflow inventory (`ci.yml`, `deploy-production.yml`)
3. Makefile targets (`ci-deploy-prod`, `rollback`) and what each does
4. GitHub Secrets and Variables reference (link to `GITHUB_SETUP.md` — do not duplicate)
5. `.env` file generation flow: secrets → workflow writes `.env` → Makefile reads it
6. Build artifacts and Docker image strategy
7. Rollback procedure

---

### DOC-P07 — Local Developer Setup Guide

| Field | Value |
|---|---|
| **File** | `LOCAL-DEV-SETUP.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now — write this first |
| **Status** | ✅ Done — see `docs/guides/local-dev-setup.md` |

**Purpose:** Step-by-step guide to get a new developer from zero to a fully running ARM stack locally. Currently no single document covers this end-to-end.

**Suggested sections:**
1. Prerequisites (Node version, pnpm version, Docker, make)
2. Cloning the monorepo and workspace setup
3. Verdaccio — starting the local registry and installing `@aracreate/test-arm-ui`
4. Infrastructure startup (`docker-compose.infra.yml`)
5. Environment variables per service (`.env.example` for each — or link to `ENV-VARS.md`)
6. Service startup order: session service → core-be → calendar-be → notification → core-fe → calendar-fe
7. Running the full stack in one command (Makefile target if it exists)
8. Verifying the stack is healthy (health check URLs, expected console output)
9. Common first-run problems and fixes

**Gap note:** `GITHUB_SETUP.md` covers CI secrets only. Nothing covers local dev. With 6+ services, 2 databases, Redis, Kafka, and Verdaccio, onboarding without this guide takes days of trial and error. **Highest-priority missing document.**

---

### DOC-P08 — Environment Variables Reference

| Field | Value |
|---|---|
| **File** | `ENV-VARS.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `ENV-VARS.md`, linked from root `CLAUDE.md`'s Key Documents table. Assembled from platform `.env.example`, per-service `.env.example` files, `REQUIRED_CORE_VARS`/`REQUIRED_OPTIONAL_VARS`, and CLAUDE critical-constraint notes; §10 lists known example/guide drifts rather than silently copying them |

**Purpose:** Canonical list of every environment variable consumed by every service — type, example value, which environments it applies to, and whether it is a secret or a public config value.

**Suggested sections:**
1. arm-session variables (`SERVICE_ENV_MAP` entries, Redis URL, session secret, CORS origin)
2. arm-core-be variables (PostgreSQL, JWT secret, mail config)
3. arm-core-fe variables (session service URL, remote registry URL)
4. calendar-be variables (MongoDB URI, Google OAuth credentials, BullMQ config)
5. calendar-fe variables (session service URL)
6. arm-service-notification variables (Kafka broker, SMTP config)
7. Shared variables (JWT secret if shared, Redis URL pattern)
8. Secret vs. non-secret classification
9. Google OAuth variables (CLIENT_ID, CLIENT_SECRET, redirect URIs per environment)

**Gap note:** Previously scattered across per-repo `CLAUDE.md` files and `GITHUB_SETUP.md` (CI only). Consolidated 2026-08-10 into `ENV-VARS.md`.

---

### DOC-P09 — Security Architecture

| Field | Value |
|---|---|
| **File** | `SECURITY.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `SECURITY.md`, linked from root `CLAUDE.md`'s Key Documents table. Verified against all four backends' `main.ts`/`app.module.ts` and auth guard source files directly, not just their `CLAUDE.md` summaries. Found: (1) `apps/arm-app-calendar/TASK_SHEET.md`'s SEC-001 cross-reference claiming an "identical gap" in admin-be is stale — admin-be's `kong-jwt.guard.ts` already has the fix; (2) `admin-be` has no `helmet()` at all — the only backend without default security headers, on the most privilege-sensitive service; (3) `ValidationPipe` strictness (`forbidNonWhitelisted`) is inconsistent across the four backends; (4) SEC-24 (token in OAuth URL) was already resolved in `SESSION-FLOW.md`, just not linked from a security-specific doc; (5) no dependency/CVE scanning exists in any of the 8 repos' CI. Includes a full OWASP Top 10 (2021) coverage map and restates Risk 13 (deny-list gap) in the security-control frame rather than the auth-doc frame |

**Purpose:** Security controls, threat model, and known risks across the entire platform.

**Suggested sections:**
1. Threat model summary (attack surfaces, trust boundaries)
2. Token storage strategy (tokens never reach the browser — httpOnly cookie only)
3. CORS policy per service and per environment
4. Rate limiting (Kong-level and service-level)
5. Input validation strategy (class-validator DTOs in NestJS)
6. Secrets management (GitHub Secrets → server `.env` — no secrets in code)
7. X-Session-Secret — what it is, where it is validated
8. SEC-24: token in OAuth URL — current status and mitigation
9. OWASP Top 10 coverage map
10. Known open security issues with risk rating

---

### DOC-P10 — Database Design

| Field | Value |
|---|---|
| **File** | `DATABASE-DESIGN.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `DATABASE-DESIGN.md`, linked from root `CLAUDE.md`'s Key Documents table and `ARCHITECTURE.md` §10. Built from scratch by reading every TypeORM entity/Mongoose schema/migration file directly, not from any service's `CLAUDE.md`. Surfaced a severe, likely-currently-live bug (§4): `admin-be`'s read-only `arm_core` mirror entities (`CoreUser`, `CoreRefreshToken`, `CoreGoogleToken`) were never updated after `core-be`'s `RenameColumnsToParamCase` migration renamed every `arm_core` column to kebab-case — `CoreUser`/`CoreRefreshToken` have no `name:` overrides at all (default to the wrong camelCase), `CoreGoogleToken`'s overrides are the old snake_case. `UsersRepository.findAll()` unconditionally orders by `user.createdAt`, which should fail on every call given the mismatch — high-confidence from source, not confirmed by reproducing against a live database. Also found: the `registry` Postgres table/API is dead code (no live consumer in core-fe), `Calendar.accountId` vs. `Event.accountId` store different ID types (provider ID vs. Mongo `_id`) confirming a gap `DELETE-CASCADE.md` §7 already flagged from the test side, and `users.isActive` (core-be) vs. admin-be's disable/deny-list are two disconnected mechanisms |

**Purpose:** Schema reference for all databases — PostgreSQL (arm-core-be) and MongoDB (calendar-be) — in one place.

**Suggested sections:**
1. PostgreSQL — ERD for arm-core-be (User, Auth entities, Registry)
2. MongoDB — Collection schemas with all fields:
   - `Account` (including `provider` field added in P1-1)
   - `Calendar`
   - `Event` (including `accountId` field added in P1-2)
   - `SyncConfig`
   - `SyncHistory`
3. Field-level descriptions and data types
4. Index definitions per collection and why each index exists
5. Cross-collection relationships (Account → Calendar → Event, Account → SyncConfig, Calendar → SyncHistory)
6. Migration strategy (PostgreSQL TypeORM migrations, MongoDB one-time migration scripts)
7. Migration script inventory (with run-once guarantees and idempotency)
8. Data retention and soft-delete vs. hard-delete policy per collection

---

### DOC-P11 — Onboarding Guide (New Engineer)

| Field | Value |
|---|---|
| **File** | `ONBOARDING.md` (repo root) |
| **Category** | Platform |
| **Type** | Non-technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `ONBOARDING.md`, linked from root `CLAUDE.md`'s Key Documents table. Two sections (exact access-request process, team contacts) are left as explicit placeholders rather than fabricated — neither is derivable from the codebase, every file's author line is a single person, and there's no written org chart or access-approval process anywhere in the 7 repos. Whoever owns onboarding should fill those in |

**Purpose:** Day-1 guide for new engineers — project context, access requests, environment setup pointer, and first contribution workflow.

**Suggested sections:**
1. What ARM is and who uses it (2–3 paragraphs, non-technical)
2. Repository structure tour (what lives in `core/`, `apps/`, `services/`, `library/`, `deploy/`)
3. Access required (GitHub, server access, secrets, monitoring dashboard)
4. Local dev setup (link to `LOCAL-DEV-SETUP.md`)
5. Architecture primer (link to `ARCHITECTURE.md`)
6. First contribution guide (branch naming convention, PR template, review process)
7. Team contacts and escalation path

---

## App — Critical

---

### DOC-A01 — arm-core-fe: Shell App Guide

| Field | Value |
|---|---|
| **File** | `core/arm-core-fe/README.md` (expanded) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — `core/arm-core-fe/README.md` expanded in place (matching the DOC-A03/A18/A19 approach — README carries the practical content, `CLAUDE.md` stays the exhaustive reference). Added: how to register a new MF remote (step-by-step, cross-linked to `ENV-VARS.md`/root `CLAUDE.md`), shared-singleton version drift (restated from `MODULE-FEDERATION.md` §4), the UnoCSS cross-scan path bug (restated from `MODULE-FEDERATION.md` §6), and a from-scratch Settings-page section (two-tab structure, lazy-mounted Linked Accounts tab, Add/Reconnect/Unlink wiring cross-linked to `GOOGLE-OAUTH.md`/`DELETE-CASCADE.md`) — none of which existed in any prior doc. Also documented `pnpm preview`/`electron:dev`/`electron:build`, which exist in `package.json` but aren't wrapped by any `make` target |

**Purpose:** Developer reference for the Module Federation shell — the app that hosts all MFE remotes.

**Suggested sections:**
1. Role of the shell (Module Federation host — what it does and does not own)
2. Remote registration in `vite.config.ts` (how to add/update a remote)
3. Route structure and lazy loading strategy
4. UnoCSS scanning configuration (why paths to remote source files must be listed)
5. Auth flow (session cookie via session service — no token stored in browser)
6. Settings page architecture (tab structure, Linked Accounts tab integration)
7. Build, dev, and preview commands

---

### DOC-A02 — arm-core-be: Core API Reference

| Field | Value |
|---|---|
| **File** | `core/arm-core-be/API.md` or OpenAPI spec (`openapi.yaml`) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-11) — written at `core/arm-core-be/API.md`, linked from root `CLAUDE.md`'s Key Documents table. **Corrects Risk 10 below** — a machine-readable spec already exists (`@nestjs/swagger`, live at `GET /api-docs`, dev-only), it just wasn't discovered before now. Verified Swagger coverage is real but partial by counting `@Api*` decorators per controller: `auth.controller.ts` fully annotated (39 decorators), `users.controller.ts` bare `@ApiTags` only (routes inherited from `GenericController`, which has zero decoration), `registry.controller.ts`/`health.controller.ts` none at all. Fixed the Swagger `DocumentBuilder`'s generic, never-customized title/description (`'My API'` / `'API documentation for my NestJS app'`) as a small drive-by correction — not yet committed, left for review alongside the doc. Also surfaced, previously undocumented anywhere in this workspace: `JwtStrategy` supports secret rotation via `JWT_ACCESS_SECRET_PREVIOUS` (tries current secret, falls back to previous on signature mismatch, both compared with `timingSafeEqual`) — confirms what `ENV-VARS.md` already listed as a fallback var without knowing the code path behind it |

**Purpose:** Complete reference for all endpoints exposed by arm-core-be.

**Suggested sections:**
1. Auth endpoints: `POST /v1/auth/login`, `POST /v1/auth/logout`, `POST /v1/auth/refresh`
2. User endpoints: `GET /v1/users/me`, `PATCH /v1/users/:id`, etc.
3. Registry endpoints: `GET /v1/registry` (MFE remote URL registry)
4. Health endpoint: `GET /health`
5. JWT validation strategy (guard implementation, token format expected)
6. Request/response shapes with TypeScript interfaces
7. Error response format (HTTP status codes, error body shape)
8. PostgreSQL entity reference (User, Auth)

---

### DOC-A03 — arm-session: Implementation Guide

| Field | Value |
|---|---|
| **File** | `core/arm-session/README.md` (expanded) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — `core/arm-session/README.md` expanded with endpoint map, session lifecycle, both login flows, add-account OAuth pattern, proxy/`SERVICE_ENV_MAP`, internal auth, CORS/cookie behavior verified against `main.ts`/`session.guard.ts`/`session.controller.ts` source, and the Redis scaling path |

**Purpose:** Developer reference for arm-session — the single session service for all ARM frontend apps.

**Suggested sections:**
1. Service responsibilities (session management + proxy — nothing else)
2. Adding a new backend service to the proxy (adding to `SERVICE_ENV_MAP`, updating `.env`)
3. Dedicated endpoint pattern (for OAuth redirects — why the generic proxy cannot handle 302 chains)
4. session service guard (`session.guard.ts`) — how it validates incoming requests
5. Redis session read/write API (session service methods)
6. Local dev vs. production CORS differences
7. Known bugs and their workaround status (query param duplication, SEC-24 OAuth URL token)
8. Environment variables required

---

### DOC-A04 — Calendar App: Architecture Overview

| Field | Value |
|---|---|
| **File** | [`docs/calendar/ARCHITECTURE.md`](../calendar/ARCHITECTURE.md) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Overall architecture of the calendar app — how the frontend remote and backend API relate to each other and to the platform.

**Written 2026-08-11**, verified against `src/backend/src/` and `src/frontend/src/` directly (not `CLAUDE.md` alone). Covers: the 18-module backend map, the `CalendarProvider` abstraction (Google full-featured / Microsoft CRUD-only by design, `ForbiddenException` → 5-min poll fallback), the sync engine's real trigger topology (initial-sync via `SyncOrchestratorService`, webhook- and poll-triggered syncs both converging on `WebhookProcessorService` independently — `SyncOrchestratorService` does *not* govern all three despite its name), Google's daily webhook-renewal cron vs. Microsoft's monthly full-resync cron (materially different reliability mechanisms, previously undocumented as such), token/credential flow (CoreAuth refresh fallback for Google-only, (secret removed) encryption), the 6 MongoDB collections (cross-linked to `DATABASE-DESIGN.md` §5 rather than re-tabulated), the 5 circuit-breaker instantiation sites vs. the unprotected direct Calendar/Graph API calls, health-check split, and the MF frontend's 4 exposed components. Surfaced 7 verified drift items against `CLAUDE.md` — see the doc's §11 — most notably: the 401/403 self-heal claim is actually 401-only; `webhook/` does not contain the inbound push-notification handler (that's in `events/`); the webhook intake route does not go through the session service proxy (Google calls it directly, `NOTIFICATION_WEBHOOK_URL`); `GOOGLE_TOKEN_ENCRYPTION_KEY`'s "must be 64 hex chars" is not enforced (silently padded/truncated instead); `auditlogs` is missing from `CLAUDE.md`'s collection table (already correct in `DATABASE-DESIGN.md`); Microsoft's monthly delta-renewal cron was undocumented as an architectural mechanism.

---

### DOC-A05 — Calendar App: Google OAuth Integration

| Field | Value |
|---|---|
| **File** | `apps/arm-app-calendar/GOOGLE-OAUTH.md` |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `apps/arm-app-calendar/GOOGLE-OAUTH.md`, linked from root `CLAUDE.md` and `AUTH-ARCHITECTURE.md`. Covers Add Account (popup → dedicated session service routes → BroadcastChannel) and Reconnect/Update Account; documents local-dev BroadcastChannel origin gotcha and Settings `useOAuthPopup` listener gap |

**Purpose:** Complete documentation of the popup-based Google OAuth flow for adding and reconnecting calendar accounts — one of the most complex flows in the system with multiple non-obvious constraints.

**Suggested sections:**
1. Full flow diagram: Add Account → popup → `/session/calendar/add-account` → Google → callback → `postMessage` → popup closes
2. Why the popup pattern (session cookie cannot be sent cross-origin; popup inherits parent session)
3. session service endpoint `GET /session/calendar/add-account` — what it does and why it is a dedicated route (not the generic proxy)
4. calendar-be OAuth callback handler — how it processes the Google authorization code
5. `postMessage` contract: event type (`GOOGLE_AUTH_SUCCESS`), origin validation, payload shape
6. Reconnect flow (re-auth for an expired or revoked account)
7. Security considerations (origin validation, OAuth state parameter anti-CSRF)
8. postMessage origin bug (P1-6 in task.md) — current status and fix
9. Environment variables required (GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, redirect URIs per environment)

**Gap note:** Written 2026-08-10. Primary completion signal is BroadcastChannel (not postMessage); P1-6/`postMessage('*')` remains only on the legacy `auth-callback.tsx` error/reconnect path.

---

### DOC-A06 — Calendar App: Sync Flow

| Field | Value |
|---|---|
| **File** | [`docs/calendar/SYNC-FLOW.md`](../calendar/SYNC-FLOW.md) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Documents how calendar events are pulled from Google/Microsoft and stored in MongoDB — initial sync, incremental sync via webhooks/poll, cross-target migration, and teardown, at the algorithm level (distinct from `ARCHITECTURE.md`'s module/service-ownership view).

**Written 2026-08-11**, verified line-by-line against `sync-orchestrator.service.ts`, `webhook-processor.service.ts`, `sync-lifecycle.service.ts`, `blocker.service.ts`, and `migrations/startup-migration.service.ts`. Notable findings: **`initial-sync` BullMQ jobs have zero retry/backoff configured** (unlike every other queue in the module) — a whole-job-level exception goes straight to the dead-letter queue; **`SyncHistory` is not a complete activity log** — successful incremental webhook/poll updates never write to it, only initial-sync runs and delta-path *failures* do; **rate-limit resilience is asymmetric between providers** — Google blocker deletes get 3-attempt exponential backoff on quota errors (`isQuotaError`, 403→429 normalization), Microsoft has no quota-error detection at all, so an equivalent Graph 429 is a single-shot non-retryable failure; **cross-target migration ("F-32") is delete-and-recreate, not a data move** — every blocker on the new target is a fresh `createEvent`; **two independent stale-blocker sweep mechanisms exist** with different triggers (every initial-sync run, vs. only on a full re-fetch after sync-token invalidation) — a source event deleted without an explicit provider cancellation can persist as an orphaned blocker between sweeps; recurring events are never expanded into instances anywhere in the sync engine (RRULE passed through as-is, both create and update paths).

---

### DOC-A07 — Calendar App: Webhook Architecture

| Field | Value |
|---|---|
| **File** | [`docs/calendar/WEBHOOKS.md`](../calendar/WEBHOOKS.md) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Documents Google Calendar push notification webhooks — registration, renewal, and processing. This is a stateful external dependency that silently breaks when not maintained.

**Written 2026-08-11**, verified against `providers/google/google-calendar-webhook.provider.ts`, `calendar/google-calendar-api.service.ts`'s `watchEvents`/`stopChannel`, `events/events.controller.ts`'s intake route, `events/webhook-processor.service.ts`'s validation sequence, and `webhook/webhook-renewal.processor.ts`. Confirms and precisely scopes the risk this doc exists to close: **`TASK-045` is still open** — a renewal that fails after the old channel was already stopped leaves a stale, dead `Calendar.channelId`/`channelToken`/`expiration` record in the DB with no automatic recovery and no poll fallback, until 3 exhausted BullMQ attempts land it in `dead-letter`. Critically, **this gap is narrower than `CLAUDE.md`'s phrasing suggests** — it's specific to *renewal* failures only; *initial* webhook registration already has a working poll fallback on failure (confirmed at both source- and target-calendar registration sites). Also newly documented: the intake controller does **zero validation** — it enqueues raw headers and returns `200 OK` before any check runs, with all real validation (channel lookup, `crypto.timingSafeEqual` token check, the `x-goog-resource-state: sync` handshake no-op) happening asynchronously; the channel expiration TTL (~7 days) is a hardcoded literal (`604_600_000`ms), not Google's own default; and no dashboard/alert exists for webhook-renewal failures today (confirmed against `MONITORING.md`/`REDIS-ARCHITECTURE.md`) — only pino logs and an unwatched dead-letter queue. Also flagged: `apps/arm-app-calendar/docs/failure-modes.md` §6 (Microsoft Graph) is now stale, predating the Microsoft calendar-sync integration this doc describes.

---

### DOC-A08 — Calendar App: Delete Cascade Specification

| Field | Value |
|---|---|
| **File** | `apps/arm-app-calendar/DELETE-CASCADE.md` |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `apps/arm-app-calendar/DELETE-CASCADE.md`. Spec of *current* `deleteAccount` behavior (not aspirational); §7 lists live gaps (poll schedulers, token revoke, Microsoft, e2e id mismatch) |

**Purpose:** Authoritative specification of the full delete cascade when a linked Google account is removed — preventing orphaned data and Google API resource leaks.

**Suggested sections:**
1. Required cascade order:
   - Stop active BullMQ sync jobs for the account
   - Stop Google webhook channels (`stopChannel` API call)
   - Delete all Events where `accountId = X`
   - Delete all Calendars where `accountId = X`
   - Delete all SyncConfig records referencing the account
   - Delete all SyncHistory records referencing the account
   - Delete the Account document
2. Scoping requirement: entire operation must be scoped to the authenticated `userId`
3. Atomicity strategy (what happens if a step fails mid-cascade)
4. Test coverage requirement (unit/integration test covering the full cascade)
5. Known gaps (status from P1-3 audit)

**Gap note:** Written 2026-08-10 against source. Code is richer than the original wishlist (blocker sweeps, AC-3) but still omits poll-scheduler clear, OAuth revoke, and Microsoft subscription cleanup — see the doc's §7.

---

## Runbooks — Critical

---

### RB-01 — Restart Services

| Field | Value |
|---|---|
| **File** | [`docs/runbooks/restart-services.md`](../runbooks/restart-services.md) |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** How to restart each service safely without causing data loss or session disruption.

**Written 2026-08-11**, verified against `deploy/arm-deploy-make`'s `make/local-dev.mk`, `make/apps.mk`, `make/utils.mk`, and `docker-compose.local.dev.yml`'s actual healthcheck definitions. Findings not previously documented anywhere: **production has no per-service restart target for `admin-be`/`admin-fe`** — `make/apps.mk`'s individual-service section only covers `core-be`/`core-fe`/`session`/`calendar-be`/`calendar-fe`/`notification`, an omission from when admin was added later; **the session service's Docker healthcheck is a bare TCP connect on port 5000, not an HTTP request to `/session/health`** — meaningfully weaker than every other backend's healthcheck (all of which hit a real dependency-checking endpoint), so a "healthy" session service container only proves the process is listening, not that it worked; **`core-fe` has no healthcheck defined at all** — Docker reports it "up" the instant the container starts, `local-dev-rebuild`'s health-polling loop treats bare `running` status as success for this one container specifically. Also documents which BullMQ queues survive a mid-job container restart and which don't (cross-referencing `SYNC-FLOW.md`'s queue-by-queue retry table), and that Google's webhook channels are external state unaffected by a `calendar-be` restart — only an in-progress *renewal* can be interrupted by one, tying back to `WEBHOOKS.md`'s `TASK-045` gap.

---

### RB-05 — Database Backup and Restore

| Field | Value |
|---|---|
| **File** | [`docs/runbooks/db-backup-restore.md`](../runbooks/db-backup-restore.md) |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Critical |
| **When** | Before production |
| **Status** | ✅ Done — **updated 2026-08-17/19.** Three of the four gaps this entry originally recorded are closed: off-host S3 sync (`make backup-sync`), daily cron scheduling (`make backup-cron-install`, run on every production deploy), and a MongoDB restore drill (`make restore-mongodb-drill`). Still open: no `arm_admin` drill, nothing schedules the drills themselves, no PITR/WAL archiving, S3 retention left to a bucket lifecycle rule |

**Purpose:** Step-by-step backup and restore procedures for PostgreSQL and MongoDB.

**Written 2026-08-11**, verified directly against `deploy/arm-deploy-make/make/backup.mk` and both backup scripts. As [INFRASTRUCTURE.md](../INFRASTRUCTURE.md) §9 already found, the underlying tooling (`make backup-postgres`/`backup-mongodb`/`backup-all`/`restore-postgres-drill`) was already real and working — this closes the actual gap, the runbook document itself. New in this pass: **the first documented restore procedure for MongoDB and for `arm_admin`** — neither had an automated drill or even a manual command written down anywhere before this (only `arm_core`'s Postgres restore is drill-tested); both are now given exact, verified `mongorestore`/`psql` commands adapted from the existing tooling's own patterns. Also confirmed and restated plainly: no off-host backup storage exists (local-disk-only, a single-host-loss risk), no retention/cleanup policy (backups accumulate unbounded), no scheduling (fully manual, `make backup-all` has to be run by a human on a cadence nothing enforces), and both scripts hardcode production container names only.

---

### RB-07 — Deploy Rollback

| Field | Value |
|---|---|
| **File** | [`docs/runbooks/deploy-rollback.md`](../runbooks/deploy-rollback.md) |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Critical |
| **When** | Before production |
| **Status** | ✅ Done — **rewritten 2026-08-19.** The original doc's central finding (image-tag rollback coded but unreachable, git revert the only real path) no longer holds for production, which now tags every deploy and rolls back automatically on a failed health check. Dev and staging are unchanged, so the git-revert procedure is still the path there |

**Purpose:** Step-by-step rollback procedure to the previous working deployment.

**Written 2026-08-11.** Leads with the operationally important fact, cross-verified against `CICD.md`'s independent finding: `make rollback TAG=<sha>` is fully coded (including a well-designed `CONFIRM_NO_SCHEMA_ROLLBACK` safety gate) but **not reachable in practice** — no deploy workflow ever runs `ci-build`/`ci-push`, so no tagged image exists anywhere for it to roll back to. The doc's real contribution is documenting what actually works instead: because every `ci-deploy-<env>` rebuilds fresh from each application repo's git branch HEAD (`repos.conf`-pinned to `dev` for **all three** environments — the same gap `CICD.md` §2 found independently), a `git revert` + re-triggered deploy workflow reproduces the effect of a rollback without needing the disconnected image-tag mechanism at all. Also documents a real asymmetry in schema-rollback readiness: `arm-core-be` has a working `migration:revert` script, `apps/arm-admin`'s backend has none despite also running TypeORM migrations on startup, and `calendar-be`'s MongoDB backfills don't need reverting since they're additive/idempotent by design.

---

## Troubleshooting — Critical

---

### TS-01 — Calendar Sync Not Working

| Field | Value |
|---|---|
| **File** | [`docs/troubleshooting/calendar-sync-not-working.md`](../troubleshooting/calendar-sync-not-working.md) |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Diagnostic decision tree for stuck, failed, or silently broken calendar syncs.

**Written 2026-08-11**, exactly the fast-follow this inventory expected — assembled almost entirely from already-verified facts in `SYNC-FLOW.md`, `WEBHOOKS.md`, `calendar/ARCHITECTURE.md`, and `failure-modes.md` rather than new research, routing each symptom to the specific mechanism already documented instead of re-deriving. Leads with the single most important framing point: `SyncHistory` is not a complete activity log (only initial-sync runs and delta-failures write to it), so "no new `SyncHistory` rows" for an otherwise-healthy ongoing sync is expected, not a symptom. Covers: `initial-sync`'s zero-retry dead-letter behavior, the `TASK-045` webhook-renewal gap, the two-mechanism stale-blocker sweep window, provider asymmetries that are by-design (Microsoft poll-only, no Microsoft quota detection, monthly vs. daily renewal cadence) rather than bugs, and the 401-only `isAuthenticated` self-heal caveat. Explicitly defers to `failure-modes.md` for dependency-outage diagnosis rather than duplicating it. Confirms **RB-03 (Manual Calendar Re-sync) is a real, still-open gap** — no dedicated force-resync endpoint exists; the closest equivalent is re-calling the sync-creation endpoint.

---

### TS-02 — Authentication Failures

| Field | Value |
|---|---|
| **File** | [`docs/troubleshooting/auth-failures.md`](../troubleshooting/auth-failures.md) |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Diagnostic guide for all auth-related failures — missing session, expired JWT, Google token revoked.

**Written 2026-08-11** as a symptom-first triage document built on top of `AUTH-ARCHITECTURE.md` and `SESSION-FLOW.md` (both already thorough — this doc maps symptoms to mechanism rather than re-explaining architecture). Structured around a quick-triage table covering 8 distinct symptom classes: missing `SESSION_ID` cookie, post-login 401s (traced to the session service's soft-fail silent-refresh design — a failed refresh forwards a stale token rather than blocking, so the 401 the caller sees is one layer downstream of the real failure), OTP delivery (a Kafka/notification dependency chain, not an auth-guard issue at all), SSO redirect failures (CSRF `state` vs. `X-Session-Secret` mismatch — two distinct causes easily conflated), the deny-list gap explicitly called out as **expected current behavior, not a bug to debug** (escalate as a feature gap, not an incident), cross-service JWT-secret/`TRUST_PROXY_HEADERS` drift, and — the most consequential structural point — that **local dev's Kong-absent guard fallback path can mask a real Kong misconfiguration in staging/prod**, since the same guard code "passes" via its JWT-verification branch in both a working Kong setup and a completely absent one. Also distinguishes platform-login auth from the separate Google/Microsoft calendar-account token clock, which fails independently and shouldn't be diagnosed with the same playbook.

---

### TS-03 — MFE Not Loading

| Field | Value |
|---|---|
| **File** | [`docs/troubleshooting/mfe-not-loading.md`](../troubleshooting/mfe-not-loading.md) |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Critical |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Diagnostic guide for Module Federation failures — remote not loading, CSS missing, federation errors in console.

**Written 2026-08-11**, built primarily on `MODULE-FEDERATION.md`'s existing §9 (5 verified pitfalls) rather than new research — symptom-first triage, not architecture re-explanation. **Corrects the inventory's own original §1 assumption**: there is no runtime registry the shell queries for a remote's URL. `core-be`'s `GET /v1/registry` and `core-fe`'s own `registry-service.ts` client both exist, but per `DATABASE-DESIGN.md` §2.4 nothing in `core-fe`'s actual code calls that service — remote discovery is purely static `VITE_CALENDAR_REMOTE_URL`/`VITE_ADMIN_REMOTE_URL` env vars baked in at build time. Anyone debugging "the shell doesn't know where the remote is" by checking the registry endpoint is looking in the wrong place. The rest of the doc routes each symptom (blank remote, version-mismatch console errors, unstyled FullCalendar, missing UnoCSS classes, silently-no-op toast/theme calls, standalone-vs-embedded behavior differences) directly at `MODULE-FEDERATION.md`'s already-documented, already-verified explanations rather than re-deriving them.

---

---

# IMPORTANT Priority

> These documents are necessary for a well-operated platform. Their absence increases risk and slows down development but is not immediately incident-causing.

---

## Platform — Important

---

### DOC-P12 — Redis Architecture

| Field | Value |
|---|---|
| **File** | `REDIS-ARCHITECTURE.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `REDIS-ARCHITECTURE.md`, linked from root `CLAUDE.md`'s Key Documents table and `ARCHITECTURE.md` §10. Built by reading every service's actual Redis client construction and every `BullModule.registerQueue()` call, not from any `CLAUDE.md` summary. Found the most significant gap: `core-be` and `calendar-be` are Sentinel-aware (branch on `REDIS_SENTINELS`), but `session` and `admin-be` connect with a plain fixed `host`/`port` — a production Sentinel failover would break every active session (`session` owns `session:data:*`) and the admin disable/enable flow, with no automatic reconnect. Also found: `initial-sync` BullMQ jobs have no retry/backoff at all (unlike every other queue with retry semantics); `redis_exporter` metrics are scraped by Prometheus but no dedicated Grafana dashboard surfaces them; and restated Risk 13 (deny-list write/read gap) from the Redis-key angle |

**Purpose:** Documents all Redis use cases — sessions, BullMQ queues, and caching — and the key namespace conventions to prevent teams from colliding.

**Suggested sections:**
1. Redis instances (single instance vs. Sentinel HA — when each is used)
2. Key namespace conventions (session keys, BullMQ queue names, cache keys)
3. Session store: key format, TTL, eviction policy
4. BullMQ queues: queue names, job shapes, retry strategies (calendar sync, webhook renewal)
5. Sentinel configuration (`docker-compose.infra.redis-sentinel.yml` — failover behavior)
6. Monitoring: memory usage, evictions, connection count thresholds

---

### DOC-P13 — Monitoring & Logging Architecture

| Field | Value |
|---|---|
| **File** | `MONITORING.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — written at `MONITORING.md`, linked from root `CLAUDE.md`'s Key Documents table and `ARCHITECTURE.md` §10. Consolidated from `observability-plan.html` + the 4 plan docs, but verified against actual live config (`observability/*.yml`, `alloy/config.alloy`, alerting provisioning, every backend's pino/tracing/metrics source) rather than trusting the plans' stated scope. Found three separate corrections to `deploy/arm-deploy-make/CLAUDE.md`'s Observability section (Tempo already found stale by `INFRASTRUCTURE.md`; **alerting is fully implemented** — 7 real rules + working email contact point — contrary to its "not Phase A scope" claim; **pino `redact` is configured in all 5 backends**, contrary to its "none configure it" claim — this also corrected a stale finding in `SECURITY.md`'s OWASP A09 row, fixed in place). Also found: only `calendar-be` has OpenTelemetry tracing wired in (4/5 backends have zero `opentelemetry` dependency, despite Tempo/Alloy's full receiving pipeline existing); only `core-fe` has the Faro SDK (`calendar-fe`/`admin-fe` don't); the 5xx alert rule and two dashboard panels both scope to `core-be`/`admin-be` only; `notification`'s health endpoint is a hardcoded stub with no Kafka check |

**Purpose:** Describes log formats, log aggregation, metrics collection, alerting, and dashboards across all services.

**Suggested sections:**
1. Log format (structured JSON, fields: timestamp, level, service, traceId, userId)
2. Log aggregation stack (confirm: Loki/Grafana, ELK, or other)
3. Application metrics exposed per service
4. Infrastructure metrics (Docker/host-level)
5. Alerting rules and thresholds (sync failures, BullMQ queue depth, Redis memory)
6. Health check endpoints per service (`GET /health`)
7. Distributed tracing strategy (if planned — traceId propagation)

**Gap note:** Written 2026-08-10. Now a documented starting point for triage exists; the real remaining gaps are partial tracing/Faro instrumentation rollout (§6, §10) and uneven alert/dashboard coverage across services (§7, §9), not the absence of the doc itself.

---

### DOC-P14 — Kafka / Message Queue Architecture

| Field | Value |
|---|---|
| **File** | [`docs/KAFKA-ARCHITECTURE.md`](../KAFKA-ARCHITECTURE.md) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Documents Kafka usage in arm-service-notification — topic definitions, consumer group strategy, and DLQ handling.

**Written 2026-08-11**, verified against `core/arm-core-be/src/libs/kafka/` (producer) and `services/arm-service-notification/src/` (consumer + DLQ) directly. Confirms Kafka's role is narrowly scoped — email notifications only, nothing else on the platform produces/consumes it — and explicitly distinguishes it from calendar-be's unrelated BullMQ-based DLQ, which shares a name pattern but no infrastructure. Key findings: **the producer's `emit()` calls are fire-and-forget by design** — every triggering HTTP request (OTP request, new-user welcome) succeeds regardless of whether the Kafka publish itself actually succeeded, so a publish failure is silently invisible to the caller with only a log line as evidence; **the DLQ is pure application code**, not a KafkaJS feature, and its own consumer does nothing but log — no replay, no persistence, no alerting; **retry policy is fail-once-then-DLQ**, no per-message retry anywhere in the pipeline; topic replication factor is correctly 1 in dev and 3 in prod (matching the 3-broker KRaft cluster), but **partition count stays at 1 everywhere**, capping per-topic consumer parallelism regardless of cluster size. **Most actionable finding: production's own `docker-compose.yml` has no `healthcheck:` stanza at all for `notification-service`** — dev and local-dev both correctly poll `/v1/health/ready` (the live/ready split built earlier this documentation pass specifically to detect a silently-dead Kafka consumer connection), but prod has nothing consuming that endpoint, so the one environment with the largest blast radius has no automated restart trigger for the exact failure mode the health check exists to catch. This is a real, fixable gap in `deploy/arm-deploy-make`'s compose file, not just a documentation note — flagged to the user for a decision on fixing it directly.

---

### DOC-P15 — Kong API Gateway Configuration

| Field | Value |
|---|---|
| **File** | [`docs/KONG-CONFIG.md`](../KONG-CONFIG.md) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Explains every route, plugin, and upstream in `kong.yml` and the rationale for each — so the configuration is not just a black box.

**Written 2026-08-11**, verified against the full 170-line `deploy/arm-deploy-make/kong/kong.yml` directly. **Corrects the inventory's own original §4 assumption** ("What Kong explicitly does NOT handle — JWT validation") — Kong *does* fully validate JWTs (signature + `exp`) via its `jwt` plugin before a request ever reaches a backend; this is in fact the concrete mechanism behind `AUTH-ARCHITECTURE.md`'s abstract `TRUST_PROXY_HEADERS`/`X-User-Id` description — a custom Lua pre-function decodes the already-verified token's `sub` claim into that header, safe only because of Kong's plugin-priority execution order (jwt runs before pre-function, not by declaration order). Also found: **Kong's `/api/core`/`/api/calendar` routes have no known consumer anywhere in this workspace** — a repo-wide search found nothing calling them; the browser's real path is exclusively the session service proxy. **Corrects a 4th stale claim in `deploy/arm-deploy-make/CLAUDE.md`'s Observability section** — Kong's `prometheus` plugin, listed there as deferred to "Phase B" alongside Tempo, is actually already live (confirmed in both `kong.yml` and `prometheus.yml`) — `MONITORING.md` updated to reflect 4 corrections, not 3. **Corrected a route-path error this same documentation pass had itself introduced**: `ARCHITECTURE.md` and `WEBHOOKS.md` both previously documented the webhook intake route as `POST /events/notifications`, omitting calendar-be's global `v1` prefix — the real route is `POST /v1/events/notifications`, confirmed by Kong's own hardcoded path-rewrite target; both docs fixed. That same investigation surfaced a genuine, narrow latent bug: calendar-be's `env.ts` hardcodes its `NOTIFICATION_WEBHOOK_URL` fallback default without the `/v1/` prefix — not currently exercised by any shipped compose file, but a real one-line fix waiting whenever that file is next touched. Also documented: no Make target or script reloads Kong's config specifically (`make gateway-reload` targets an unrelated, largely-dead nginx container) — exactly the gap RB-08 exists to close.

---

### DOC-P16 — ADR Collection (Architecture Decision Records)

| Field | Value |
|---|---|
| **File** | [`docs/adr/`](../adr/readme.md) — 8 numbered files + index |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now (write retroactively for past decisions) |
| **Status** | ✅ Done |

**Purpose:** Records the context, decision, and consequences of significant architectural choices so future engineers understand the "why," not just the "what."

**Written 2026-08-11**, all 8 originally-planned ADRs completed, drawing on already-verified facts from this documentation pass rather than new research for most of them:

| # | Title | Status |
|---|---|---|
| [ADR-001](../adr/001-standalone-bff.md) | Standalone arm-session vs. embedding session service in arm-core-be | Decided, implemented — sourced from a preserved original proposal doc (`docs/architecture/session-architecture.md`) |
| [ADR-002](../adr/002-module-federation.md) | Module Federation over alternative MFE approaches | Decided, implemented |
| [ADR-003](../adr/003-mongodb-for-calendar.md) | MongoDB for calendar data vs. PostgreSQL | Decided, implemented — **no original rationale document exists**; written as inferred reasoning from the schema shape, explicitly flagged as such rather than presented as verified history |
| [ADR-004](../adr/004-bullmq-vs-kafka.md) | BullMQ for calendar sync jobs, Kafka reserved for notifications | Decided, implemented |
| [ADR-005](../adr/005-google-oauth-popup.md) | Google OAuth popup + BroadcastChannel pattern | Decided, implemented — **title corrected** from the inventory's original "postMessage pattern" once research showed BroadcastChannel is the primary mechanism and postMessage is a secondary/legacy path (`GOOGLE-OAUTH.md` §6) |
| [ADR-006](../adr/006-unocss-mfe-scoping.md) | UnoCSS scoping strategy across MFE boundaries | Decided, implemented — with a verified-broken cross-scan path recorded as a direct consequence |
| [ADR-007](../adr/007-verdaccio-registry.md) | Verdaccio for internal package registry | **Superseded** — adopted, then removed entirely (already known from `INFRASTRUCTURE.md`'s DOC-P25-is-obsolete finding); written to preserve the fact an internal registry ever existed, since nothing else in the codebase records that |
| [ADR-008](../adr/008-redis-sentinel-ha.md) | Redis Sentinel for HA vs. Redis Cluster | Decided, **partially implemented** — 2 of 4 Redis-using services aren't actually Sentinel-aware (`REDIS-ARCHITECTURE.md` §8's highest-severity finding), recorded as an incomplete rollout rather than a clean decision |

**Notable pattern across the collection**: 3 of 8 ADRs (003, 007, 008) don't end at a tidy "decided and done" — one has no preserved rationale, one was later reversed, one is a partially-executed rollout with a real production gap. ADRs exist to preserve exactly this kind of messiness rather than presenting architecture as a series of clean, completed decisions; the index (`docs/adr/readme.md`) says this explicitly rather than leaving it implicit.

**ADR template sections used**: Context, Decision, (Alternatives considered, where source material supported it), Consequences, Related documents.

---

### DOC-P17 — Coding Standards & Conventions

| Field | Value |
|---|---|
| **File** | [`docs/CODING-STANDARDS.md`](../CODING-STANDARDS.md) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Team-wide rules for code style, naming, file structure, testing, and review — reduces PR friction and cognitive overhead.

**Written 2026-08-11** as a description of patterns actually observed across the codebase, not an aspirational style guide — deliberately scoped as the in-code complement to `TEMPLATE-CONFORMANCE.md` (repo-level structure/naming), not a duplicate of it. Verified: the exact curated `tsconfig.json` strict-flag subset shared identically across all 3 TypeORM backends (not blanket `strict: true` — the NestJS CLI's own default); the facade pattern's repeated, intentional use (`EventsService`, `CalendarService`, `AccountService`); that every backend sets a global `v1` route prefix, which is exactly the fact whose absence from earlier docs caused the `/events/notifications` vs. `/v1/events/notifications` error `KONG-CONFIG.md` caught and fixed. Two things explicitly **not** invented despite the original suggested-sections outline asking for them: no platform-wide test coverage threshold exists anywhere, and pre-commit enforcement is **not** platform-wide — only `arm-service-notification` has a husky/lint-staged hook, confirmed by a repo-wide search; every other backend and frontend lints/tests at CI time only. Both stated as genuinely unanswered/inconsistent rather than papered over with an assumed standard.

---

### DOC-P18 — Release Process

| Field | Value |
|---|---|
| **File** | [`docs/RELEASE-PROCESS.md`](../RELEASE-PROCESS.md) |
| **Category** | Platform |
| **Type** | Technical / Non-technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Step-by-step checklist for getting code from a working branch onto the server.

**Written 2026-08-11.** Leads with the same structural fact `runbooks/deploy-rollback.md` found from the rollback angle: there is no dev → staging → production **code-promotion** pipeline for application repos — `repos.conf` pins every one to `dev` for all three environments, so the originally-suggested "branch promotion checklist" section doesn't describe anything that exists. Restructured around what's actually true instead: "release" (semantic-release: versioning/changelog/GitHub release) and "deploy" (arm-deploy-make's SSH-triggered fresh clone-and-build) are two unrelated mechanisms that don't reference each other. Confirmed only 3 of 7 application repos (`arm-session`, `arm-service-notification`, `arm-ui-library`) have semantic-release wired up at all — `arm-admin` has zero CI/CD of any kind. **New finding, not previously documented anywhere**: `arm-ui-library`'s own `.releaserc.json` sets `@semantic-release/npm`'s `npmPublish: false` — the CI-automated release workflow never actually runs `npm publish`; the real publish step is manual, per `docs/guides/npm-publish.md`, and that guide's own `npm version` step appears to duplicate semantic-release's automatic version-bump-and-commit behavior with no documented reconciliation between the two. Flagged as unclear rather than asserted as a definite bug.
5. Rollback procedure (link to `RUNBOOKS/deploy-rollback.md`)
6. Hotfix process (bypassing normal branch flow for critical fixes)
7. arm-ui-library release process (version bump + Verdaccio publish)

---

### DOC-P19 — Testing Strategy

| Field | Value |
|---|---|
| **File** | [`docs/TESTING-STRATEGY.md`](../TESTING-STRATEGY.md) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Defines the test pyramid, what each test layer covers in this stack, and expectations per service.

**Written 2026-08-12.** The single highest-yield finding: **CI does not run most of the test infrastructure that exists.** `calendar-be` has the platform's only real integration/E2E layer (Testcontainers-backed MongoDB + Redis, both a `test:integration` and `test:e2e` script) — its own `ci.yml` never invokes either, only `pnpm test` (unit). `core-fe`'s `ci.yml` runs lint + `tsc --noEmit` but never `pnpm test`, so its 8 Vitest unit specs never run in CI at all (only the separate Playwright `e2e.yml` does). Also found, and verified by checking `app.module.ts`'s `controllers:` array directly: `arm-session` and `admin-be` both still carry the **unmodified NestJS CLI scaffold** `tests/app.e2e-spec.ts` (`GET / → 'Hello World!'`) against an `AppModule` that registers no `AppController` in either repo — this test would fail if it ever ran, and it never does, since neither repo's CI invokes `test:e2e`. Confirmed `admin-be` has zero automated testing at any layer in any environment (no CI per [CICD.md](../CICD.md), and not part of `deploy/arm-deploy-make/make/test.mk`'s `test` target either — that target only wires up `test-core`/`test-session`/`test-calendar`/`test-notification`, `arm-admin` and `arm-ui-library` were never added to it). Also confirmed `calendar-fe` and `admin-fe` have **zero test tooling of any kind** — no Vitest, no Testing Library, not even a `test` script — by dependency inspection, consistent across both Module Federation remotes, not a one-off gap. Restated (not re-derived) from [CODING-STANDARDS.md](../CODING-STANDARDS.md) §6: no coverage threshold exists anywhere on the platform, confirmed again directly against every Jest/Vitest config's absence of a `coverageThreshold`/`thresholds` block.

---

### DOC-P20 — Business Requirements Document

| Field | Value |
|---|---|
| **File** | `BRD.md` (repo root) or Notion/Confluence |
| **Category** | Platform |
| **Type** | Non-technical |
| **Priority** | Important |
| **When** | Now (may already exist externally — link here if so) |

**Purpose:** Documents the business goals, user personas, and success metrics for the ARM platform.

**Suggested sections:**
1. Problem statement (what ARM solves for end users)
2. Target user personas
3. Business goals and KPIs
4. Scope and explicitly out-of-scope items
5. Regulatory / compliance requirements
6. Stakeholder map

**Explicitly deferred, checked 2026-08-12.** Confirmed no source material exists anywhere in the workspace for the core sections (personas, KPIs, stakeholder map, compliance requirements) — grepped every doc for business/persona/stakeholder/KPI content and found nothing beyond [ONBOARDING.md](../ONBOARDING.md)'s own line 13, which already flags this exact gap ("neither exists yet as a written doc ... ask whoever owns product direction"). Unlike every other Important-tier doc in this pass, this one can't be assembled from code/config/git evidence — it needs real product-owner input. Asked whether to scaffold with explicit placeholders (matching DOC-P11's pattern for its two unknown sections) or wait for real content; decided to wait rather than publish a mostly-placeholder BRD. Revisit once product direction is available.

---

### DOC-P21 — Product Roadmap

| Field | Value |
|---|---|
| **File** | `ROADMAP.md` (repo root) or Linear/Notion |
| **Category** | Platform |
| **Type** | Non-technical |
| **Priority** | Important |
| **When** | Now |

**Purpose:** Living document of planned features, current phase, and priorities — visible to both technical and non-technical stakeholders.

**Suggested sections:**
1. Current sprint / phase summary (e.g. Phase 0–3: Linked Accounts feature)
2. Upcoming milestones (Linked Accounts, Projects MFE, Outlook provider, etc.)
3. Backlog items
4. Deferred / won't-do decisions (with brief rationale)

---

### DOC-P22 — Incident Response Runbook

| Field | Value |
|---|---|
| **File** | `RUNBOOKS/INCIDENT-RESPONSE.md` |
| **Category** | Platform |
| **Type** | Non-technical (process) |
| **Priority** | Important |
| **When** | Before production traffic |

**Purpose:** Defines how the team responds to production incidents — roles, communication, escalation, and post-mortem process.

**Suggested sections:**
1. Severity levels (P0–P3) and response SLAs per level
2. On-call rotation and escalation path
3. Communication channels (who to notify, when, and in what format)
4. Triage checklist (where to look first: Kong logs, session service logs, Redis health, BullMQ queue depth)
5. Common incident types and first-response steps (link to Troubleshooting guides)
6. Post-mortem template (timeline, root cause, action items, blameless culture statement)

---

### DOC-P23 — Operations & Support Guide

| Field | Value |
|---|---|
| **File** | `OPERATIONS.md` (repo root) |
| **Category** | Platform |
| **Type** | Non-technical |
| **Priority** | Important |
| **When** | Before production traffic |

**Purpose:** Reference for non-engineer operations or support staff.

**Suggested sections:**
1. How to check system health (health URLs, expected uptime metrics)
2. Common user-reported issues and how to resolve them without engineering
3. How to initiate a manual calendar re-sync for a user (link to `RB-03`)
4. How to revoke a user's Google account access (link to `RB-06`)
5. Escalation path to engineering (who to contact and when)

---

### DOC-P24 — arm-ui-library: Component Catalogue

| Field | Value |
|---|---|
| **File** | `library/arm-ui-library/UI-LIBRARY.md` + Storybook (if applicable) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |

**Purpose:** Reference for all shared UI components so teams do not rebuild components that already exist in `@aracreate/test-arm-ui`.

**Suggested sections:**
1. Component inventory with props, slots, default values, and usage examples
2. Design tokens (colors, spacing, typography — from `uno.config.ts`)
3. Version history (link to existing `CHANGELOG.md`)
4. Publishing to Verdaccio — how to bump the version and publish
5. How to use in a consuming app (install, import, UnoCSS scanning path requirement)

---

### DOC-P25 — Verdaccio Internal Registry Setup

| Field | Value |
|---|---|
| **File** | `VERDACCIO-SETUP.md` (repo root) |
| **Category** | Platform |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | 🟡 Partial — `docs/guides/npm-publish.md` covers the publish side; registry setup/consumption side still undocumented |

**Purpose:** Documents how the internal npm registry works, how to publish packages, and how consuming apps resolve `@aracreate/*` packages.

**Suggested sections:**
1. Starting Verdaccio locally (Docker Compose target, default port)
2. Registry URL and `.npmrc` / pnpm configuration for consuming apps
3. Publishing a new version of `arm-ui-library` (bump → build → publish)
4. Adding a new internal package
5. Upstream fallback to npmjs.com (what happens for packages not in Verdaccio)

---

## App — Important

---

### DOC-A09 — arm-core-be: MFE Registry

| Field | Value |
|---|---|
| **File** | `core/arm-core-be/REGISTRY.md` |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** The `registry` module manages the runtime URLs of MFE remotes — a non-obvious system that controls which version of each remote the shell loads per environment.

**Suggested sections:**
1. What the registry stores (remote name → URL mapping per environment)
2. How core-fe fetches the registry at startup
3. How to add a new remote to the registry (database entry or config — clarify which)
4. Environment-specific URL overrides (local points to localhost, the server to deployed URLs)
5. What happens if the registry is unreachable at startup

**Written 2026-08-12 — this doc found and fixed a genuine, multi-file factual error already in the workspace, not just a documentation gap.** `DATABASE-DESIGN.md` §2.4, `core-be/API.md` §4, and `troubleshooting/mfe-not-loading.md` all previously stated the registry API is never called by `core-fe` ("dead code," "unused for its documented purpose"), each citing a `MODULE-FEDERATION.md` §2.4 that didn't actually exist in that document. Traced directly against source instead: `core-fe/src/lib/remote-registry.ts`'s `initRemoteRegistry()` **does** call `GET /v1/registry/calendar`, and it's invoked from `ProtectedRoute` (`core-fe/src/components/auth/common/protected-route.tsx:53`) on every authenticated session — not dead code, not disconnected. What's actually true, and more precise than the old claim: nothing anywhere in the workspace ever calls `POST /v1/registry/register` (grepped every repo), so the table is permanently empty and the read always comes back with nothing — which is why the *observed effect* ("build-time env var wins") was always correctly documented even though the *mechanism* ("nothing calls this API") wasn't. Also found this repo's own `README.md` mischaracterized `src/registry/` as "placeholder for future org/role management," contradicted by both the entity schema (`name`/`type: 'backend'|'mfe'`/`url`/`remoteUrl`) and `git log -- src/registry`, which shows only MFE-registry-related commits. All four documents (this one plus the three that cited the false claim) were corrected in place, with `MODULE-FEDERATION.md` gaining the §2.4 it was missing since at least three other docs already pointed to it.

---

### DOC-A10 — Calendar App: Linked Accounts Feature

| Field | Value |
|---|---|
| **File** | [`docs/calendar/LINKED-ACCOUNTS.md`](../calendar/LINKED-ACCOUNTS.md) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now (feature in active development) |
| **Status** | ✅ Done |

**Purpose:** Feature-level documentation for the Linked Accounts system — translates the task.md implementation plan into a permanent architectural reference.

**Suggested sections:**
1. Feature overview (one ARM user → multiple Google accounts → multiple calendars)
2. Account data model (`Account` schema fields including the new `provider` field)
3. Sync status model (`SyncStatus` — lifecycle states and what each means)
4. API endpoints:
   - `GET /v1/account` — list all accounts for authenticated user
   - `DELETE /v1/account/:accountId` — unlink with full cascade
   - `GET /v1/account/:accountId/sync-status` — latest sync status
5. Frontend components: LinkedAccountCard (props, badge states), LinkedAccountsTab (data fetching, empty state)
6. Provider abstraction groundwork (`provider` field — commented future values for Outlook, Apple)
7. Reconnect flow (how to re-auth an expired account from the Settings page)
8. Known constraints (one Google OAuth app, redirect URIs must match registered URIs)

**Written 2026-08-12, confirming the correction flagged above**: Microsoft is documented throughout as a live, working provider, not future groundwork — §2's `provider` field description and §7's constraints both reflect that. Also a genuine two-repo feature, not calendar-app-only: the actual UI (`LinkedAccountsTab`/`LinkedAccountCard`/`useLinkedAccounts`) lives in `core-fe`, not `calendar-fe` — the doc says so up front rather than pretending the feature is contained in one repo just because that's where the target filename lives. **New finding, verified by grep across both frontends**: `GET /v1/account/sync-from-core`, `/account/available`, `/account/connect`, and `calendar/AuthCallback`'s `token`/`userId` query-param branch — the older `syncFromCoreAuth()` account-creation path `apps/arm-app-calendar/CLAUDE.md` describes as one of "two" existing OAuth paths — have no reachable UI trigger anywhere in the current app; both frontends only ever open the modern `/session/calendar/add-account(/microsoft)` popup, which completes through a different branch entirely. Live code, not formally dead, but unreachable from anything a user can actually click today. Also found the `onSync` pause/resume field and its `PUT /v1/account/:accountId` endpoint are fully implemented backend-side with no corresponding UI control in `LinkedAccountCard`/`LinkedAccountsTab` — an account can only be unlinked, never paused, from the current UI.

---

### DOC-A11 — Calendar App: API Reference

| Field | Value |
|---|---|
| **File** | [`docs/calendar/API.md`](../calendar/API.md) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Complete reference for all endpoints exposed by calendar-be, consumed via the arm-session proxy.

**Suggested sections:**
1. Account endpoints (`/v1/account/*` — list, delete, sync-status, reconnect)
2. Calendar endpoints (`/v1/calendar/*` — list, get, enable/disable)
3. Calendar management endpoints (`/v1/calendar-management/*`)
4. Event endpoints (`/v1/event/*` — list, get, filter by calendar/account)
5. Google OAuth endpoints (`/v1/google/*` — auth initiation, callback)
6. Webhook endpoint (Google-facing — validation, delta processing)
7. Auth requirements (all endpoints require JWT from session service — document expected header)
8. Error response shapes

**Written 2026-08-12.** Verified against all 9 controllers directly, not just DTOs. Two corrections to the original outline: calendar-management has **no `@Controller()` prefix at all** — its routes hang off the bare `v1` prefix, including a deprecated `GET /v1/:accountId` catch-all the controller's own comment says is safe to remove once confirmed unused (still present); account endpoints are cross-linked to `LINKED-ACCOUNTS.md` §4 rather than duplicated, since that doc already covers all 10 routes in full. **Biggest finding, §9**: unlike `core-be` (which has a global `AllExceptionsFilter`), calendar-be has none — two incompatible error patterns coexist across controllers. Thrown exceptions get a real, correct HTTP status; but several handlers (most of `account.controller.ts`, some of `events.controller.ts`) catch their own errors and `return` a plain `{ statusCode: 400, success: false, ... }` object with no `@HttpCode()` override, so the *real* HTTP response is whatever NestJS defaults to for that verb (200/201) — a client trusting the real status code over the JSON body's own `statusCode` field will see "success" on a logical failure. Not a guess: `common/idempotency/idempotency.interceptor.ts`'s own code comment names this exact pattern and had to add an `isLogicalFailure()` check so these responses aren't cached as if they were real successes — the workaround is shipped, the underlying inconsistency it works around isn't documented anywhere until now. Also flagged, not fixed: `POST /webhook/renew-all` (force-renew every webhook channel platform-wide) uses the same plain `AuthGuard` as every ordinary user-scoped route — no elevated-privilege check restricts it to an account owner or admin. Whether that's intentional is a product/security call, noted here the same way Risk 13 is, not silently resolved.

---

### DOC-A12 — Calendar App: Data Model Reference

| Field | Value |
|---|---|
| **File** | [`docs/calendar/DATA-MODEL.md`](../calendar/DATA-MODEL.md) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Detailed field-level schema reference for all MongoDB collections, including post-task.md additions and inter-collection relationships.

**Written 2026-08-12 — deliberately not a duplicate.** Every item in the original suggested-sections list (field-level schema per collection, index rationale, relationship diagram, migration inventory with idempotency guarantees, soft-delete policy) was found already fully covered in `DATABASE-DESIGN.md` §5-§6, written earlier this pass from direct source (every `*.schema.ts` file). Writing a second full schema reference would only create two documents to keep in sync as the schema changes. Instead this is a short, calendar-scoped index — a "where to find what" table pointing into `DATABASE-DESIGN.md`'s exact sections, plus the one thing that document doesn't cover: a one-line operational summary of what each collection is *for*, from calendar-be's own point of view (e.g., that `events` holds both real synced-event copies and `[ BLOCKER ]` events in the same collection, distinguished only by `extendedProperties.private.isSource`, and that `synchistories` vs. `auditlogs` are easy to conflate by name but track entirely different things).

---

### DOC-A13 — arm-service-notification: Service Guide

| Field | Value |
|---|---|
| **File** | `services/arm-service-notification/README.md` (expanded) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Developer reference for the notification service — what it listens to, what it sends, and how to test it locally.

**Suggested sections:**
1. Service responsibilities (Kafka consumer → email sender — nothing else)
2. Kafka topic subscriptions and message schema per topic
3. Email templates (what templates exist in `template/`, rendering engine used)
4. Mail provider configuration (SMTP settings, which provider is used per environment)
5. DLQ design (`dlq.service.ts`) — what triggers DLQ entry, inspection and drain procedure
6. Local dev (how to test email sending without a live Kafka broker)
7. Environment variables required

**Written 2026-08-12.** Deliberately thin on Kafka/DLQ mechanics — `KAFKA-ARCHITECTURE.md` (DOC-P14) already covers topology, retry philosophy, and consumer-crash/recovery in full, so this README cross-links rather than repeats, staying focused on what's genuinely new here: the exact 3-topic/handler/payload table, the Handlebars `strict: true` constraint on new templates (a missing context variable throws, doesn't render blank), and the `SMTP_PORT` string-vs-number coercion gotcha in `buildMailerConfig`. One structural finding worth keeping: `DlqConsumerController` isn't a separate service or process — `main.ts` runs one hybrid HTTP+Kafka app, and the DLQ consumer is wired into the *same* module and *same* `notification-group` consumer group as the two send-email handlers, confirmed directly from `app.module.ts`'s single `controllers` array and `main.ts`'s single `connectMicroservice()` call. Also confirmed no local SMTP catcher (Mailhog/Maildev) exists anywhere in the workspace — the practical no-live-infra dev path is this service's own test suite, which (per `TESTING-STRATEGY.md` §1) is one of the few "e2e"-named suites on the platform that's genuinely rewritten rather than left as NestJS scaffold boilerplate.

---

### DOC-A14 — arm-ui-library: Developer Guide

| Field | Value |
|---|---|
| **File** | `library/arm-ui-library/DEVELOPER.md` |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done |

**Purpose:** Guide for engineers contributing to the shared UI library.

**Suggested sections:**
1. Repo structure (`src/`, `dist/`, `scripts/`)
2. Build process (`tsup.config.ts` — what it produces and why)
3. Testing (Vitest — `vitest.components.config.ts` for component tests vs. `vitest.config.ts` for unit)
4. Adding a new component (checklist: implement → test → export from `index.ts` → update CHANGELOG → publish)
5. UnoCSS token system (`uno.config.ts` — how tokens are defined and consumed)
6. Publishing to Verdaccio (version bump → `pnpm build` → `pnpm publish --registry <url>`)
7. Consuming app UnoCSS scanning path (must include `node_modules/@aracreate/test-arm-ui/dist/**`)

**Written 2026-08-12.** Corrected item 6 before writing it — Verdaccio was fully removed (ADR-007), the real publish flow is public-npm and entirely manual per `guides/npm-publish.md`, which DEVELOPER.md cross-links rather than duplicates. **Found and fixed a genuinely wrong fact in `README.md` while researching this**: every install/import example in the README referenced `@kishor-aracreate/ac-ui-library-test`/`@kishor-aracreate/test-arm-ui` — neither of which is the real package. `package.json`'s own `name` field, and every one of the three frontends' `package.json` dependencies, confirm the actual published/consumed name is `@aracreate/test-arm-ui`. Fixed as a drive-by (17 occurrences plus the npm-version badge URL), same treatment as DOC-A02's Swagger-title fix — a small, unambiguous, high-value correction, not a judgment call. **Flagged, not fixed**: that same README's "Available Components" table lists 14 components; `src/components/` actually contains ~50 folders (accordion, combobox, dialog, drawer, form, navigation-menu, tabs, and many more never mentioned). Rewriting that catalogue is real content work beyond this doc's contributor-guide scope, left as an open gap rather than attempted here. Also found, from `preset.ts`'s own code comment: two color-token definitions exist (the live one in `preset.ts`'s preflights, and a duplicate in `src/styles/radix-brand-tokens.css`) and only the first actually reaches consumers — `dist/index.css` (built from the second) has no `exports` map entry and is never self-imported, so it's dead weight, not a redundant safety net.

---

### DOC-A18 — arm-admin: Backend Guide (Users, Audit, RBAC)

| Field | Value |
|---|---|
| **File** | `apps/arm-admin/src/backend/README.md` (expanded — written in place of a separate `GUIDE.md`, matching the DOC-A03 approach) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — README rewritten from the stale scaffolding placeholder; ported and verified `CLAUDE.md`'s source structure, TypeORM connections, auth guard stack, and deny-list content against actual source (repo was mid-restructure into `src/backend/` during this pass — paths below and in `AUTH-ARCHITECTURE.md` updated accordingly). The deny-list section documents the same Risk 13 gap found while writing DOC-P04 |

**Purpose:** Developer reference for the admin backend — the newest app in the workspace (not present at the original inventory date). Owns user management and audit logging against the `arm_admin` PostgreSQL database, a data surface with real sensitivity (audit trail integrity, admin access control).

**Suggested sections:**
1. Service responsibilities and relationship to arm-core-be (separate PostgreSQL database `arm_admin` — clarify why user mgmt is split from core-be's own user table)
2. `users` module — endpoints, entities, and how admin user records relate to (or differ from) arm-core-be users
3. `audit` module — what actions are logged, entity/DTO shape, retention policy
4. Auth strategy (`src/auth/` — `KongJwtGuard` + `AdminRoleGuard`, deny-list mechanism — already documented in `CLAUDE.md`, needs porting to prose)
5. Migrations and seeders (`src/database/migrations`, `src/database/seeders`)
6. Environment variables required

**Gap note:** The technical content already exists in `CLAUDE.md`; the gap is that it's written for an AI agent (terse, imperative, no "why") and lives in a file most new engineers won't think to open. Largely a rewrite/porting job, not new research — see corrected Risk 11 below.

---

### DOC-A19 — arm-admin: Frontend Guide

| Field | Value |
|---|---|
| **File** | `apps/arm-admin/src/frontend/README.md` (expanded, not a separate `GUIDE.md` — matches the DOC-A03/DOC-A18 approach) |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Important |
| **When** | Now |
| **Status** | ✅ Done (2026-08-10) — README rewritten from the stale scaffolding placeholder; ported and verified against source, including a second interceptor-level 403/401 redirect layer (`api/client.ts`) not documented in `CLAUDE.md`'s summary and the actual `@module-federation/vite` config (not the `@originjs` plugin `CLAUDE.md`'s wording could imply) |

**Purpose:** Developer reference for the admin MFE remote — mounted at `/admin/*` in the shell per `CLAUDE.md`, exposing `AdminApp`.

**Suggested sections:**
1. Module Federation registration (`AdminApp` expose, mount point `/admin/*` in arm-core-fe — already documented in `CLAUDE.md`)
2. Route structure and which admin capabilities exist today (users, audit log — confirm against `src/` module inventory)
3. API base URL (`/session/proxy/admin/v1` per `CLAUDE.md` — confirm session service `SERVICE_ENV_MAP` entry exists)
4. Auth/access restriction — `CLAUDE.md`'s "Auth — 403 Fallback" section already covers this, needs porting
5. Build, dev, and preview commands (Vite HMR on port `10000`)

---

## Runbooks — Important

---

### RB-02 — Redis Session Flush

| Field | Value |
|---|---|
| **File** | `RUNBOOKS/redis-flush.md` |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Important |
| **When** | Now |

**Purpose:** How to safely flush all active sessions (forcing all users to re-login) — required after a security incident or session schema change.

**Suggested sections:**
1. When to flush (security incident, SESSION_ID compromise, Redis schema migration)
2. Command to flush session keys (namespace-safe — flush only session keys, not BullMQ queues)
3. User impact (all users logged out simultaneously)
4. Verification (confirm sessions are gone, confirm queues are unaffected)
5. Post-flush monitoring (spike in login traffic expected)

---

### RB-03 — Manual Calendar Re-sync

| Field | Value |
|---|---|
| **File** | `RUNBOOKS/calendar-sync-manual.md` |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Important |
| **When** | Now |

**Purpose:** How to manually trigger a full calendar re-sync for a specific user or account.

**Suggested sections:**
1. When to use (user reports missing events, sync status stuck in `failed`)
2. API call to trigger re-sync (endpoint, auth, body)
3. How to monitor progress (SyncHistory record, BullMQ job status)
4. Expected duration and event volume
5. What to check if re-sync also fails

---

### RB-04 — Manual Webhook Renewal

| Field | Value |
|---|---|
| **File** | `RUNBOOKS/webhook-renew-manual.md` |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Important |
| **When** | Now |

**Purpose:** How to manually renew a Google webhook channel that has expired or is about to expire.

**Suggested sections:**
1. How to check channel expiry (what field to query in MongoDB or Redis)
2. API or admin script to force-renew a channel
3. Validation (confirm new channelId and expiry are stored)
4. What to do if Google returns an error during renewal
5. How to verify events are flowing again after renewal

---

### RB-06 — Google Account Revoke

| Field | Value |
|---|---|
| **File** | `RUNBOOKS/google-account-revoke.md` |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Important |
| **When** | Now |

**Purpose:** How to revoke a user's Google OAuth tokens and clean up the associated data.

**Suggested sections:**
1. When to revoke (user request, security incident, account deletion)
2. Steps: revoke token via Google API → trigger delete cascade → confirm cleanup
3. Link to delete cascade specification (`DELETE-CASCADE.md`)
4. Confirming the user's events are removed from MongoDB

---

### RB-08 — Kong Configuration Reload

| Field | Value |
|---|---|
| **File** | `RUNBOOKS/kong-reload.md` |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Important |
| **When** | Now |

**Purpose:** How to update `kong.yml` and reload Kong without service interruption.

**Suggested sections:**
1. Editing `kong/kong.yml.tmpl`, then re-rendering (what each section controls)
2. Validating the config before applying (`deck validate` or equivalent)
3. Applying the config (deck sync or Admin API call)
4. Verifying routes are live
5. Rolling back a bad Kong config change

---

### RB-09 — DLQ Drain

| Field | Value |
|---|---|
| **File** | `RUNBOOKS/queue-dlq-drain.md` |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Important |
| **When** | Now |

**Purpose:** How to inspect and retry jobs in the Dead Letter Queue for both BullMQ (calendar sync) and Kafka (notifications).

**Suggested sections:**
1. BullMQ DLQ — how to list failed jobs, inspect payloads, retry or discard
2. Kafka DLQ — how to read the DLQ topic, replay messages, inspect `dlq.service.ts`
3. When to retry vs. discard (transient errors vs. permanent failures)
4. Alerting threshold (how many DLQ entries trigger an alert)

---

### RB-10 — Session Debug

| Field | Value |
|---|---|
| **File** | `RUNBOOKS/session-debug.md` |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Important |
| **When** | Now |

**Purpose:** How to inspect a user's Redis session for debugging auth or proxy issues.

**Suggested sections:**
1. Finding a user's session key in Redis (key pattern, how to look up by userId)
2. Reading the session payload (fields, token presence, TTL)
3. Checking if the session token has expired
4. Manually invalidating a single session (without flushing all)
5. What to look for when the session service is injecting a wrong or stale token

---

## Troubleshooting — Important

---

### TS-04 — Webhooks Silently Stopped

| Field | Value |
|---|---|
| **File** | `TROUBLESHOOTING/webhook-silent-stop.md` |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Important |
| **When** | Now |

**Purpose:** How to detect and recover when Google webhook channels have expired without the renewal processor running.

**Suggested sections:**
1. Symptoms (sync history shows no new records, events not updating despite user changes in Google)
2. Check: is the webhook renewal BullMQ processor running?
3. Check: what is the `expiry` on the stored webhook channel record?
4. Manual renewal procedure (link to `RB-04`)
5. Root cause analysis (why did the processor stop — Redis down, job deleted, crash loop?)

---

### TS-05 — Session Service Proxy Errors

| Field | Value |
|---|---|
| **File** | `TROUBLESHOOTING/session-proxy-errors.md` |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Important |
| **When** | Now |

**Purpose:** Diagnosing 502/503 errors from the session service proxy and the known query param duplication bug.

**Suggested sections:**
1. Confirming the error is from the session service vs. from the upstream backend (log comparison)
2. Query param duplication bug in `parseUrl()` — symptoms, how to identify it, workaround
3. Backend not reachable from session service (service discovery, internal DNS, Docker network)
4. SESSION_ID present but session missing from Redis (eviction, Redis restart)
5. CORS preflight failures at the session service layer

---

### TS-06 — Local Dev Issues

| Field | Value |
|---|---|
| **File** | `TROUBLESHOOTING/local-dev-issues.md` |
| **Category** | Platform |
| **Type** | Operational |
| **Priority** | Important |
| **When** | Now |

**Purpose:** Common local development problems and their fixes — reduces onboarding friction.

**Suggested sections:**
1. Verdaccio not resolving `@aracreate/test-arm-ui` (registry not running, `.npmrc` not configured)
2. CORS errors in dev (shell and remote on different ports — Caddy/Nginx config check)
3. Calendar-fe hot reload not working (federation path vs. standalone path conflict)
4. UnoCSS classes missing in Core (scanning path not updated after adding a new MFE)
5. MongoDB connection refused (Docker not running, wrong URI)
6. Redis connection refused (Docker not running, wrong port)
7. Google OAuth not working locally (redirect URI not registered in Google Console)

---

---

# OPTIONAL Priority

> These documents improve developer experience and maintainability but are not blocking for production or onboarding. Write these after Critical and Important documents are complete.

---

## App — Optional

---

### DOC-A15 — Calendar App: Frontend Component Guide

| Field | Value |
|---|---|
| **File** | `apps/arm-app-calendar/apps/frontend/COMPONENTS.md` |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Optional |
| **When** | Later |

**Purpose:** Developer reference for calendar-fe components — useful once the component set stabilizes.

**Suggested sections:**
1. Exposed Module Federation components and their props (`./SyncView`, `./SyncDetail`, `./Page-1`, `./AuthCallback`)
2. SyncView page layout (left/right split layout, FullCalendar integration)
3. CSS scope requirement (`.calendar-scope` wrapper — must be applied by the Core shell)
4. Services layer (`services/` — API call methods, base URL from session service)
5. State management approach (local vs. shared store)
6. FullCalendar customization (theme variables, UnoCSS overrides in `calendar-styles.css`)

---

### DOC-A16 — arm-core-be: Auth Deep Dive

| Field | Value |
|---|---|
| **File** | `core/arm-core-be/AUTH-INTERNALS.md` |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Optional |
| **When** | Later |

**Purpose:** Internal implementation details of the auth module in core-be — strategies, guards, refresh token rotation, and the tasks scheduler for token cleanup.

**Suggested sections:**
1. JWT strategy implementation (`strategies/`)
2. Refresh token rotation (how refresh tokens are stored and rotated)
3. Auth guards (`guards/`) — local, JWT, and any role guards
4. Scheduled tasks (`tasks/`) — token expiry cleanup, session audit
5. Auth repository (what is stored in PostgreSQL vs. what is only in Redis via session service)

---

### DOC-A17 — arm-core-fe: State Management Guide

| Field | Value |
|---|---|
| **File** | `core/arm-core-fe/STATE-MANAGEMENT.md` |
| **Category** | App |
| **Type** | Technical |
| **Priority** | Optional |
| **When** | Later |

**Purpose:** Documents how global state is managed in the Core shell (`store/`) and what state is local vs. shared.

**Suggested sections:**
1. Store structure (`store/` directory — what slices exist)
2. What state is global (auth state, user profile, registry)
3. What state is local (page-level, component-level)
4. How MFE remotes share state with the shell (if applicable — props, context, or custom events)
5. State hydration on app startup

---

---

# Final Risk Report

> High-risk undocumented areas that represent active risk to the project right now.

---

## Risk 1 — No Local Dev Setup Guide `[RESOLVED 2026-08-10]`

**Impact:** A new engineer joining the team has no document telling them how to start the full stack. With 6+ services, 2 databases, Redis, Kafka, and Verdaccio, onboarding without this guide will take multiple days of trial and error and verbal knowledge transfer that won't scale.

**Affected document:** `LOCAL-DEV-SETUP.md` (DOC-P07)

**Status:** ✅ Written — `docs/guides/local-dev-setup.md` exists and is referenced from the root `CLAUDE.md`.

---

## Risk 2 — Google OAuth Popup Flow is Undocumented `[RESOLVED 2026-08-10]`

**Impact:** The popup → session service → calendar-be OAuth flow has multiple non-obvious constraints: why a dedicated session service route is needed (302 redirect cannot be proxied generically), cookie behavior inside popups, and how the parent learns the flow finished. Every engineer who touches this flow without documentation will rediscover the same pitfalls.

**Affected document:** `GOOGLE-OAUTH.md` (DOC-A05)

**Status:** ✅ Written — `apps/arm-app-calendar/GOOGLE-OAUTH.md`. Documents BroadcastChannel completion (not primary postMessage), separate Reconnect/Update Account path, and the local-dev `:3000`/`:5001` BroadcastChannel origin gotcha. Remaining product debt (Settings `useOAuthPopup` missing close-poll; legacy `postMessage('*')` in `auth-callback.tsx`) is called out in the doc rather than left tribal.

---

## Risk 3 — Webhook Renewal is Undocumented `[RESOLVED (doc) 2026-08-11, gap remains open in code]`

**Impact:** Google webhook channels expire. The renewal processor runs as a BullMQ background job. If this processor fails silently — Redis restart, job deleted, crash loop — sync stops entirely without any user-visible error. There is no runbook for detecting or recovering from this failure. This is a production silent-failure risk.

**Affected documents:** `WEBHOOKS.md` (DOC-A07), `RUNBOOKS/webhook-renew-manual.md` (RB-04), `TROUBLESHOOTING/webhook-silent-stop.md` (TS-04)

**Status:** ✅ Documented — [`calendar/WEBHOOKS.md`](../calendar/WEBHOOKS.md) §4 pins the gap down precisely: it's `TASK-045`, still open, and specific to *renewal* failures only (initial registration already has a working poll fallback). §7 confirms no dashboard/alert exists for this today — only pino logs and an unwatched dead-letter queue. **The underlying code gap itself is not fixed** — this closes the documentation risk (engineers can now find and understand the gap), not the production risk (the gap still exists and still needs an engineering fix: either give renewal a poll fallback on exhausted retries, or add alerting on `webhook-renewal` job failures / stale `Calendar.expiration`). RB-04 and TS-04 are still not written.

**Action:** Engineering fix (poll fallback on renewal failure, or alerting) still needed before first production sync traffic — this is now a tracked, understood gap rather than an undocumented one. RB-04 (manual renewal runbook) and TS-04 (troubleshooting) remain open.

---

## Risk 4 — Delete Cascade Has No Authoritative Spec `[RESOLVED 2026-08-10]`

**Impact:** Without a permanent cascade spec, engineers cannot tell which side effects unlink must perform (BullMQ jobs, webhook channels, Events, SyncConfig, SyncHistory, Account) versus what `stopSync` already covers. Partial cleanup leaves orphaned MongoDB documents and active Google API resource channels.

**Affected document:** `DELETE-CASCADE.md` (DOC-A08)

**Status:** ✅ Written — `apps/arm-app-calendar/DELETE-CASCADE.md` documents the live `AccountConnectionService.deleteAccount` order and explicitly lists remaining gaps (poll schedulers not cleared on unlink, no token revoke, no Microsoft subscription cleanup, e2e using provider id instead of Mongo `_id`). Closing those gaps is engineering work, not a missing-doc problem.

---

## Risk 5 — No Environment Variable Reference `[RESOLVED 2026-08-10]`

**Impact:** There is no single document listing all environment variables across all services with types, example values, and secret vs. non-secret classification. `GITHUB_SETUP.md` covers only CI secrets. Misconfigured env vars (wrong `CALENDAR_BE_URL`, missing `GOOGLE_CLIENT_SECRET`, wrong `SESSION_COOKIE_SECRET`) cause runtime failures that are hard to diagnose without a reference. Every new environment (staging, production) that is stood up relies on tribal knowledge.

**Affected document:** `ENV-VARS.md` (DOC-P08)

**Status:** ✅ Written — `ENV-VARS.md` exists and is referenced from the root `CLAUDE.md`. Keep it current alongside `.env.example` / `REQUIRED_*_VARS` changes; lingering example drifts are listed in the doc's §10 rather than left as tribal knowledge.

---

## Risk 6 — No Monitoring or Alerting Documentation `[HIGH → IN PROGRESS]`

**Impact:** No documented health check URLs, log locations, alerting thresholds, or dashboards. During a production incident, the team has no documented starting point for triage. There is also no documented alert for BullMQ queue depth or webhook channel expiry — two failure modes that do not produce visible errors until data is significantly stale.

**Affected documents:** `MONITORING.md` (DOC-P13), `RUNBOOKS/INCIDENT-RESPONSE.md` (DOC-P22)

**Status:** 🟡 Substantial groundwork exists as implementation plans (`docs/architecture/observability-plan.html`, plus Grafana stack/alerting/Faro/Tempo plans under `docs/superpowers/plans/`) but nothing has been consolidated into a settled `MONITORING.md`, and `RUNBOOKS/INCIDENT-RESPONSE.md` still does not exist.

**Action:** Once the observability stack plans are implemented, consolidate them into `MONITORING.md`. `INCIDENT-RESPONSE.md` still needs to be written from scratch.

---

## Risk 7 — No ADRs for Past Decisions `[MEDIUM]`

**Impact:** Key decisions (standalone session service, MongoDB for calendar data, BullMQ vs. Kafka for sync, Module Federation approach) were made without recorded ADRs. New engineers will question or attempt to reverse these decisions without understanding the constraints that drove them. The session service decision was already misunderstood once (a separate `calendar-session` was planned before the current architecture was settled).

**Affected document:** ADR Collection (DOC-P16)

**Action:** Write ADR-001 through ADR-008 retroactively — 2–4 hours of work, high long-term value.

---

## Risk 8 — MFE CSS Isolation Pattern is Tribal Knowledge `[MEDIUM]`

**Impact:** The `.calendar-scope` wrapper requirement, the `virtualExposes` vs. `App.tsx` entry point difference, and the UnoCSS cross-MFE scanning path issue are documented only in task.md as bug fixes — not in any permanent reference. Any engineer building a new MFE or modifying the calendar MFE without reading task.md will repeat these bugs, which took non-trivial time to diagnose.

**Affected document:** `MODULE-FEDERATION.md` (DOC-P02)

**Action:** Include a dedicated section on CSS isolation pitfalls and the `.calendar-scope` pattern.

---

## Risk 9 — No Backup Policy or Tested Restore Procedure `[MEDIUM → LARGELY RESOLVED]`

**Impact:** MongoDB and PostgreSQL have no documented backup schedule, retention policy, or tested restore procedure. Data loss from a host failure or accidental deletion has no recovery path until this is documented and tested.

**Affected document:** `RUNBOOKS/db-backup-restore.md` (RB-05)

**Status 2026-08-19:** the schedule (daily 02:00 cron, installed on every production deploy), the retention policy (`BACKUP_RETENTION_DAYS`, default 14, local disk), off-host copies (S3 sync), and tested restores for `arm_core` **and** `arm-calendar` all now exist and are documented. What's left of this risk: no drill for `arm_admin`, nothing schedules the drills themselves, and no PITR — so the RPO is "since last night's dump."

**Action:** Close the `arm_admin` drill and schedule the drills; decide separately whether the nightly-dump RPO is acceptable for production.

---

## Risk 10 — No Machine-Readable API Specification for Any Service `[MEDIUM → PARTIALLY RESOLVED]`

**Impact:** Frontend-backend integration is harder to verify without an OpenAPI spec, and the session service proxy risks drifting out of contract with its backends silently.

**Correction (2026-08-11, while writing DOC-A02):** the original claim that all three backends lack a spec was wrong for `arm-core-be` — it already has `@nestjs/swagger` wired up (`GET /api-docs`, dev-only), verified in `main.ts`. Coverage is uneven (`auth.controller.ts` fully annotated; `users`/`registry`/`health` controllers thin-to-undecorated — see `core/arm-core-be/API.md`'s own intro), but a real, live spec exists. **Verified still genuinely absent** in `arm-session`, `calendar-be`, and `arm-admin`'s backend — no `@nestjs/swagger` dependency in any of the three.

**Affected documents:** `core/arm-core-be/API.md` (DOC-A02) — ✅ done. `apps/arm-app-calendar/API.md` (DOC-A11) — ✅ done (2026-08-12), hand-verified against all 9 controllers since calendar-be still has no Swagger to lean on. That write-up surfaced its own finding, arguably more actionable than the missing spec itself: calendar-be has no global exception filter, and several handlers catch their own errors and return a body claiming a 4xx `statusCode` while the real HTTP response stays 200/201 — see DOC-A11's own entry.

**Action:** For `session`/`calendar-be`/`admin-be`: use NestJS Swagger module (`@nestjs/swagger`) to auto-generate OpenAPI specs from existing DTOs and decorators — low-effort, high-value, same pattern `core-be` already proves out. For `core-be` itself: fill in the thin `@ApiOperation`/`@ApiResponse` coverage on `users`/`registry`/`health`, matching what `auth.controller.ts` already has.

---

## Risk 11 — arm-admin's Onboarding-Facing Docs Are Stale Scaffolding (Technical Content Exists Elsewhere) `[RESOLVED 2026-08-10]`

**Impact:** `apps/arm-admin/` did not exist at the original inventory date. It now owns a `users` module and an `audit` module against its own `arm_admin` PostgreSQL database — an audit-log and admin-user-management surface. Both `src/backend/README.md` and `src/frontend/README.md` were the unedited generic scaffolding-template readmes (`ac-superapp-*-template`, last touched 2026-06-23) — a new engineer reading the README got zero real information and possibly a wrong first impression of the stack.

**Affected documents:** `apps/arm-admin/src/backend/README.md` (DOC-A18), `apps/arm-admin/src/frontend/README.md` (DOC-A19)

**Status:** ✅ Both rewritten — ported from their respective `CLAUDE.md` files and re-verified against source rather than trusted at face value. That verification pass also surfaced Risk 13 (deny-list gap) on the backend side and, on the frontend side, corrected two details `CLAUDE.md`'s summary didn't capture: a second interceptor-level 401/403 redirect layer in `api/client.ts`, and the actual MF plugin in use (`@module-federation/vite`).

---

## Risk 12 — Local-Dev Reverse Proxy Story Is Inconsistent Across Docs `[RESOLVED 2026-08-10]`

**Impact:** The root `CLAUDE.md` formerly stated local dev uses Caddy in place of Kong, while `deploy/arm-deploy-make/docker-compose.local.dev.yml` states "No Caddy. No Kong." Writing architecture/infra docs from the wrong claim would bake a stale edge story into canonical docs.

**Affected documents:** `ARCHITECTURE.md` (DOC-P01), `INFRASTRUCTURE.md` (DOC-P05), `KONG-CONFIG.md` (DOC-P15), root `CLAUDE.md`, `AUTH-ARCHITECTURE.md`

**Status:** ✅ Reconciled — compose + `make/local-dev.mk` are authoritative for local (neither edge). `ARCHITECTURE.md` §3 documents local / CI-dev (Caddy) / prod-oriented (Kong) separately. Root `CLAUDE.md` and `AUTH-ARCHITECTURE.md` §3 updated to match. Residual staleness may remain in `deploy/arm-deploy-make/README.md` / `kong/readme.md` until those are edited; do not copy them over `ARCHITECTURE.md` §3.

---

## Risk 13 — Disabling a User Doesn't Actually Revoke Their Active Session `[HIGH]` — found while writing DOC-P04, verified against source

**Impact:** `apps/arm-admin/src/backend/CLAUDE.md` documents a deny-list: disabling a user revokes their refresh tokens, writes a row to `arm_admin.disabled_users`, and writes `disabled:{userId}` to Redis (15 min TTL), described as a "hot-path check in core-be and calendar-be guards." A repo-wide search for the `disabled:` key pattern and for `DenyList`/`deny-list` found it written and read **only** inside `apps/arm-admin/src/backend` — `core-core-be`'s `KongJwtGuard`, calendar-be's `AuthGuard`, and `arm-session` never reference it. In practice: disabling a user blocks them from getting a *new* access token (refresh tokens are revoked), but any access token they already hold keeps working against core-be and calendar-be for up to 15 minutes (the default `JWT_ACCESS_TOKEN_TTL`) after being disabled — not the near-immediate cutoff the deny-list's own TTL and the documentation both imply. Written up in full in [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md)'s §4.

**Affected documents/code:** `apps/arm-admin/src/backend/CLAUDE.md` (claim needs correcting or the code needs to catch up to it), `core/arm-core-be/src/auth/guards/kong-jwt.guard.ts`, `apps/arm-app-calendar/apps/backend/src/auth/auth.guard.ts` (deny-list check is missing from both)

**Action:** This is a product decision as much as a doc fix — either wire the `disabled:{userId}` Redis check into both guards (closes the gap, matches the documented intent), or shorten the access-token TTL, or explicitly accept and document the up-to-15-minute window. Whichever is chosen, update `apps/arm-admin/src/backend/CLAUDE.md` so it stops claiming a check that doesn't exist.

---

---

# Master Summary Table

| ID | Document | File | Category | Type | Priority | When | Status |
|---|---|---|---|---|---|---|---|
| DOC-P01 | Platform Architecture Overview | `ARCHITECTURE.md` | Platform | Technical | **Critical** | Now | ✅ Done |
| DOC-P02 | Module Federation Architecture | `MODULE-FEDERATION.md` | Platform | Technical | **Critical** | Now | ✅ Done |
| DOC-P03 | session service Architecture & Session Flow | `SESSION-FLOW.md` | Platform | Technical | **Critical** | Now | ✅ Done |
| DOC-P04 | Authentication & Authorization Architecture | `AUTH-ARCHITECTURE.md` | Platform | Technical | **Critical** | Now | ✅ Done |
| DOC-P05 | Infrastructure & Deployment Guide | `INFRASTRUCTURE.md` | Platform | Technical | **Critical** | Now | ✅ Done |
| DOC-P06 | CI/CD Pipeline Documentation | `CICD.md` | Platform | Technical | **Critical** | Now | ✅ Done |
| **DOC-P07** | **Local Developer Setup Guide** | `LOCAL-DEV-SETUP.md` | Platform | Technical | **Critical** | **Now — write first** | ✅ Done |
| DOC-P08 | Environment Variables Reference | `ENV-VARS.md` | Platform | Technical | **Critical** | Now | ✅ Done |
| DOC-P09 | Security Architecture | `SECURITY.md` | Platform | Technical | **Critical** | Now | ✅ Done |
| DOC-P10 | Database Design | `DATABASE-DESIGN.md` | Platform | Technical | **Critical** | Now | ✅ Done |
| DOC-P11 | Onboarding Guide | `ONBOARDING.md` | Platform | Non-technical | **Critical** | Now | ✅ Done |
| DOC-A01 | arm-core-fe: Shell App Guide | `core/arm-core-fe/README.md` | App | Technical | **Critical** | Now | ✅ Done |
| DOC-A02 | arm-core-be: Core API Reference | `core/arm-core-be/API.md` | App | Technical | **Critical** | Now | ✅ Done |
| DOC-A03 | arm-session: Implementation Guide | `core/arm-session/README.md` | App | Technical | **Critical** | Now | ✅ Done |
| DOC-A04 | Calendar App: Architecture Overview | `docs/calendar/ARCHITECTURE.md` | App | Technical | **Critical** | Now | ✅ Done |
| **DOC-A05** | **Calendar App: Google OAuth Integration** | `apps/arm-app-calendar/GOOGLE-OAUTH.md` | App | Technical | **Critical** | **Now — high risk** | ✅ Done |
| **DOC-A06** | **Calendar App: Sync Flow** | `docs/calendar/SYNC-FLOW.md` | App | Technical | **Critical** | **Now** | ✅ Done |
| **DOC-A07** | **Calendar App: Webhook Architecture** | `docs/calendar/WEBHOOKS.md` | App | Technical | **Critical** | **Now — high risk** | ✅ Done |
| **DOC-A08** | **Calendar App: Delete Cascade Spec** | `apps/arm-app-calendar/DELETE-CASCADE.md` | App | Technical | **Critical** | **Now** | ✅ Done |
| **DOC-P26** | **Deploy Orchestration (descriptors, manifest, compose generation)** | `docs/DEPLOY-ORCHESTRATION.md` | Platform | Technical | **Critical** | **Now** | ✅ Done (2026-08-19) |
| RB-01 | Restart Services | `docs/runbooks/restart-services.md` | Platform | Operational | **Critical** | Now | ✅ Done |
| RB-05 | Database Backup and Restore | `docs/runbooks/db-backup-restore.md` | Platform | Operational | **Critical** | Before prod | ✅ Done |
| RB-07 | Deploy Rollback | `docs/runbooks/deploy-rollback.md` | Platform | Operational | **Critical** | Before prod | ✅ Done |
| TS-01 | Calendar Sync Not Working | `docs/troubleshooting/calendar-sync-not-working.md` | Platform | Operational | **Critical** | Now | ✅ Done |
| TS-02 | Authentication Failures | `docs/troubleshooting/auth-failures.md` | Platform | Operational | **Critical** | Now | ✅ Done |
| TS-03 | MFE Not Loading | `docs/troubleshooting/mfe-not-loading.md` | Platform | Operational | **Critical** | Now | ✅ Done |
| DOC-P12 | Redis Architecture | `REDIS-ARCHITECTURE.md` | Platform | Technical | Important | Now | ✅ Done |
| DOC-P13 | Monitoring & Logging Architecture | `MONITORING.md` | Platform | Technical | Important | Now | ✅ Done |
| DOC-P14 | Kafka / Message Queue Architecture | `docs/KAFKA-ARCHITECTURE.md` | Platform | Technical | Important | Now | ✅ Done |
| DOC-P15 | Kong API Gateway Configuration | `docs/KONG-CONFIG.md` | Platform | Technical | Important | Now | ✅ Done |
| DOC-P16 | ADR Collection | [`docs/adr/`](../adr/readme.md) | Platform | Technical | Important | Now (retroactive) | ✅ Done |
| DOC-P17 | Coding Standards & Conventions | `docs/CODING-STANDARDS.md` | Platform | Technical | Important | Now | ✅ Done |
| DOC-P18 | Release Process | `docs/RELEASE-PROCESS.md` | Platform | Technical / Non-technical | Important | Now | ✅ Done |
| DOC-P19 | Testing Strategy | `docs/TESTING-STRATEGY.md` | Platform | Technical | Important | Now | ✅ Done |
| DOC-P20 | Business Requirements Document | `BRD.md` | Platform | Non-technical | Important | Now | 🔲 Not started |
| DOC-P21 | Product Roadmap | `ROADMAP.md` | Platform | Non-technical | Important | Now | 🔲 Not started |
| DOC-P22 | Incident Response Runbook | `RUNBOOKS/INCIDENT-RESPONSE.md` | Platform | Non-technical | Important | Before prod | 🔲 Not started |
| DOC-P23 | Operations & Support Guide | `OPERATIONS.md` | Platform | Non-technical | Important | Before prod | 🔲 Not started |
| DOC-P24 | arm-ui-library: Component Catalogue | `library/arm-ui-library/UI-LIBRARY.md` | Platform | Technical | Important | Now | 🔲 Not started |
| DOC-P25 | Verdaccio Internal Registry Setup | `VERDACCIO-SETUP.md` | Platform | Technical | Important | Now | ⚠️ Obsolete — verified 2026-08-10 while writing `INFRASTRUCTURE.md`: Verdaccio was removed from the stack entirely (`make/local-dev.mk` documents the removal; `npm-publish.md` now publishes via plain `npm publish --access public`). No internal registry exists to document — this entry should be closed, not written |
| DOC-A09 | arm-core-be: MFE Registry | `core/arm-core-be/REGISTRY.md` | App | Technical | Important | Now | ✅ Done |
| DOC-A10 | Calendar App: Linked Accounts Feature | `docs/calendar/LINKED-ACCOUNTS.md` | App | Technical | Important | Now | ✅ Done |
| DOC-A11 | Calendar App: API Reference | `docs/calendar/API.md` | App | Technical | Important | Now | ✅ Done |
| DOC-A12 | Calendar App: Data Model Reference | `docs/calendar/DATA-MODEL.md` | App | Technical | Important | Now | ✅ Done |
| DOC-A13 | arm-service-notification: Service Guide | `services/arm-service-notification/README.md` | App | Technical | Important | Now | ✅ Done |
| DOC-A14 | arm-ui-library: Developer Guide | `library/arm-ui-library/DEVELOPER.md` | App | Technical | Important | Now | ✅ Done |
| DOC-A18 | arm-admin: Backend Guide | `apps/arm-admin/src/backend/README.md` | App | Technical | Important | Now | ✅ Done |
| DOC-A19 | arm-admin: Frontend Guide | `apps/arm-admin/src/frontend/README.md` | App | Technical | Important | Now | ✅ Done |
| RB-02 | Redis Session Flush | `RUNBOOKS/redis-flush.md` | Platform | Operational | Important | Now | 🔲 Not started |
| RB-03 | Manual Calendar Re-sync | `RUNBOOKS/calendar-sync-manual.md` | Platform | Operational | Important | Now | 🔲 Not started |
| RB-04 | Manual Webhook Renewal | `RUNBOOKS/webhook-renew-manual.md` | Platform | Operational | Important | Now | 🔲 Not started |
| RB-06 | Google Account Revoke | `RUNBOOKS/google-account-revoke.md` | Platform | Operational | Important | Now | 🔲 Not started |
| RB-08 | Kong Configuration Reload | `RUNBOOKS/kong-reload.md` | Platform | Operational | Important | Now | 🔲 Not started |
| RB-09 | DLQ Drain | `RUNBOOKS/queue-dlq-drain.md` | Platform | Operational | Important | Now | 🔲 Not started |
| RB-10 | Session Debug | `RUNBOOKS/session-debug.md` | Platform | Operational | Important | Now | 🔲 Not started |
| TS-04 | Webhooks Silently Stopped | `TROUBLESHOOTING/webhook-silent-stop.md` | Platform | Operational | Important | Now | 🔲 Not started |
| TS-05 | Session Service Proxy Errors | `TROUBLESHOOTING/session-proxy-errors.md` | Platform | Operational | Important | Now | 🔲 Not started |
| TS-06 | Local Dev Issues | `TROUBLESHOOTING/local-dev-issues.md` | Platform | Operational | Important | Now | 🔲 Not started |
| DOC-A15 | Calendar App: Frontend Component Guide | `apps/arm-app-calendar/apps/frontend/COMPONENTS.md` | App | Technical | Optional | Later | 🔲 Not started |
| DOC-A16 | arm-core-be: Auth Deep Dive | `core/arm-core-be/AUTH-INTERNALS.md` | App | Technical | Optional | Later | 🔲 Not started |
| DOC-A17 | arm-core-fe: State Management Guide | `core/arm-core-fe/STATE-MANAGEMENT.md` | App | Technical | Optional | Later | 🔲 Not started |

---

**Total: 60 documents** (58 original — the "57" in the prior version undercounted by one — + DOC-A18, DOC-A19 for arm-admin)

| Priority | Count |
|---|---|
| Critical | 25 |
| Important | 32 |
| Optional | 3 |

| Status | Count |
|---|---|
| ✅ Done | 39 |
| 🟡 Partial / In progress | 0 |
| 🔲 Not started | 20 |
| ⚠️ Obsolete | 1 |

**Update (2026-08-10 continued):** `DATABASE-DESIGN.md`'s admin-be `arm_core` mirror bug (§4) was confirmed live (empirically reproduced against a fresh migrated Postgres instance) and then **fixed** — `apps/arm-admin`'s three `core/*.entity.ts` files were corrected (kebab-case column mappings, `password` column removed, `microsoftId` added), typecheck and full Jest suite (39/39) pass, and the fix was committed locally (`apps/arm-admin` repo, `dev` branch, commit `0029484`). `DATABASE-DESIGN.md` §4 has been updated to reflect "fixed," not "currently-live." Separately, `SECURITY.md`'s pino-`redact` finding (OWASP A09) was corrected in place after `MONITORING.md` proved it wrong — see below.

**Recommended write order:** Phase 1 (pure porting from existing `CLAUDE.md` content) is complete. DOC-P11 (onboarding) is also done — the first item with no existing source material, assembled from root `CLAUDE.md`, `docs/guides/local-dev-setup.md`, `TEMPLATE-CONFORMANCE.md`, and empirical git/CI evidence (branch names, actual workflow triggers) rather than any single source doc; two sections (access-request process, team contacts) are left as explicit placeholders since neither is written down anywhere or derivable from code. Done so far: DOC-P07, DOC-P04, DOC-P11, DOC-P08, DOC-P01, DOC-P03, DOC-P02, DOC-P05, DOC-P06, DOC-P09, DOC-P10, DOC-P12, DOC-P13, DOC-A01, DOC-A02, DOC-A05, DOC-A08, DOC-A03, DOC-A18, DOC-A19, DOC-P14, DOC-P15, DOC-P16, RB-05, TS-01, DOC-P17, DOC-P18, DOC-P19, DOC-A09, DOC-A10, DOC-A11, DOC-A12, DOC-A13, DOC-A14. DOC-P02 (Module Federation) surfaced four verified config drifts along the way (stale UnoCSS cross-scan path, FullCalendar theme overrides silently dead in the federated path, host/remote React version mismatch, admin's UnoCSS preset divergence) — none blocking, all noted in the doc itself. DOC-P05 (Infrastructure) surfaced six more: `nginx.dev.conf` is dead code, `CLAUDE.md`'s Tempo-deferred claim is stale (Tempo already ships), the admin app has no path to staging/production, Kong's admin API exposure differs dev vs. staging/prod, RB-05's backup/restore tooling already exists (revised that row from Not-started to Partial), and DOC-P25 (Verdaccio) documents infrastructure that's been removed entirely (marked Obsolete, not Partial). DOC-P06 (CI/CD) surfaced the biggest findings yet: `repos.conf` pins every app repo to `dev` for all three deploy environments (staging/prod don't actually deploy promoted code), `arm-admin` has zero CI, staging/prod deploy-verify jobs check for containers that don't exist in those compose files (dated to a 2026-06-30 commit, not yet triggered), `core-be` has a live broken Kubernetes deploy workflow unrelated to the platform's real deploy model, and `rollback` is coded but unreachable from any automated workflow. DOC-P09 (Security) surfaced five more, all verified against guard/bootstrap source directly: `apps/arm-app-calendar/TASK_SHEET.md`'s SEC-001 cross-reference into admin-be is stale (admin-be is already fixed), `admin-be` has no `helmet()` at all, `ValidationPipe` strictness is inconsistent across the four backends, no dependency/CVE scanning exists anywhere in CI, and Risk 13 (deny-list gap) is restated in security-control terms with an OWASP A01 tag. **DOC-P10 (Database Design) surfaced the most severe finding of this whole pass, since fixed** (see Update above). Also found the `registry` Postgres table is dead code, and `Calendar.accountId`/`Event.accountId` store different ID types. **DOC-P12 (Redis Architecture) surfaced another high-severity, previously-undocumented gap:** `session` and `admin-be` are not Sentinel-aware (unlike `core-be`/`calendar-be`) — a production Sentinel failover would break every active session platform-wide with no automatic recovery. Also found `initial-sync` BullMQ jobs have no retry/backoff at all, and Redis metrics are scraped but have no dedicated Grafana dashboard. **DOC-P13 (Monitoring) turned up three separate stale "not implemented" claims in `deploy/arm-deploy-make/CLAUDE.md`'s Observability section** (Tempo, Grafana alerting, pino `redact` — all three actually shipped without that doc being updated), and corrected `SECURITY.md`'s OWASP A09 finding in place as a result. Also found only `calendar-be` (of 5 backends) has OpenTelemetry tracing wired in, and only `core-fe` (of 3 frontends) has the Faro error-tracking SDK — the receiving infrastructure for both is fully built and mostly unused. **DOC-P13's `notification` health-check finding was then fixed, not just documented:** verified live (started the real service against a real Kafka broker, killed the broker, confirmed `/v1/health` stayed `200 ok` throughout — including after the consumer had fully crashed), then fixed with a real `live`/`ready` split (`KafkaHealthService`, a throwaway broker round-trip check) mirroring `calendar-be`'s pattern, both compose healthchecks repointed at `/v1/health/ready`, verified again end-to-end (up → down → recovered), committed in `services/arm-service-notification` (`fe15b6b`) and `deploy/arm-deploy-make` (`565ce0f`), both on `dev`. `MONITORING.md` §8/§11 updated to "Fixed," not "verified stub." **DOC-A01 (arm-core-fe README)** added net-new content no prior doc had — the Settings/Linked Accounts page architecture — plus restated the UnoCSS path bug and shared-singleton version drift from `MODULE-FEDERATION.md` as practical README content rather than leaving them doc-only. **DOC-A02 (core-be API Reference) corrected Risk 10** — `core-be` already has a live Swagger spec (`/api-docs`), contrary to the risk's original claim; coverage is uneven across its 4 controllers (verified by counting decorators), and `session`/`calendar-be`/`admin-be` genuinely still have none. Fixed the Swagger config's generic never-customized title/description as a small drive-by (uncommitted, left for review). Also surfaced JWT secret-rotation support (`JWT_ACCESS_SECRET_PREVIOUS`) that no prior doc had traced to its actual code path. None of these newer findings block the docs themselves, all noted with evidence in their respective files. **DOC-A04 (Calendar App: Architecture Overview)** surfaced seven more verified drift items against `apps/arm-app-calendar/CLAUDE.md`, the most significant being: the 401/403 self-heal claim only actually covers 401 (403s take a separate quota-retry path, or are unhandled on Microsoft); `webhook/` does not contain the inbound push-notification handler despite its directory-tree comment saying so (the real handler lives in `events/`, reached via `POST /events/notifications`); that same route does **not** go through the session service proxy as `CLAUDE.md`'s example implies — Google calls `NOTIFICATION_WEBHOOK_URL` directly, since a push service can't carry a browser session cookie; `GOOGLE_TOKEN_ENCRYPTION_KEY`'s "must be exactly 64 hex characters" isn't enforced (a malformed key is silently padded/truncated instead of failing fast); and `SyncOrchestratorService` only actually handles initial sync, not webhook- or poll-triggered syncs (both of those converge on `WebhookProcessorService`/`sync-poll.processor.ts` independently) despite a name implying it coordinates all three. Also confirmed Microsoft's monthly full-resync cron (`microsoft-delta-renewal.service.ts`) is a materially different reliability mechanism from Google's daily webhook-channel renewal — previously undocumented as such beyond a three-word tree comment. **DOC-A06 (Calendar App: Sync Flow)** went to the algorithm level and found several things not visible at the module-map altitude of DOC-A04: `initial-sync` BullMQ jobs have zero retry/backoff configured (a whole-job exception goes straight to dead-letter, unlike every other queue in the module); `SyncHistory` is not a complete activity log — successful incremental webhook/poll updates never write to it, only initial-sync runs and delta-path failures do; rate-limit resilience is asymmetric between providers (Google blocker deletes get 3-attempt exponential backoff on quota errors, Microsoft has no quota-error detection at all — an equivalent Graph 429 is a single-shot non-retryable failure); cross-target migration ("F-32") is delete-and-recreate, never a data move; two independent stale-blocker sweep mechanisms exist with different triggers (every initial-sync run vs. only after a full re-fetch from sync-token invalidation), leaving a window where a source event deleted without an explicit provider cancellation persists as an orphaned blocker; and recurring events are never expanded into instances anywhere in the sync engine, on either the create or update path. **DOC-A07 (Calendar App: Webhook Architecture) closed Risk 3** — pinned the previously-vague "renewal can fail silently" concern down to a specific, still-open code gap (`TASK-045`): a renewal failure leaves a stale dead channel record in the DB with no poll fallback, specific to *renewal* (initial registration already has a working fallback), and confirmed no dashboard/alert exists for it. Also found the webhook intake controller does zero validation itself — it enqueues raw headers and returns `200 OK` before any check runs, with real validation (including `crypto.timingSafeEqual` token comparison) happening asynchronously — and that the ~7-day channel expiration is a hardcoded literal, not Google's default. This is a documentation fix, not an engineering fix — the underlying gap in `webhook-renewal.processor.ts` is still open and still needs either a poll fallback on exhausted retries or dedicated alerting. **RB-01 (Restart Services)** surfaced three previously-undocumented gaps in `deploy/arm-deploy-make` while verifying restart mechanics against actual `make/*.mk` and compose-file source: production has no per-service restart target for `admin-be`/`admin-fe` (an omission from when admin was added after `make/apps.mk` was written); the session service's Docker healthcheck is a bare TCP port-connect, not an HTTP request to its own `/session/health` endpoint, meaningfully weaker than every other backend's; and `core-fe` has no healthcheck defined at all, so it's reported "up" the instant the container starts. None of these are blocking, but they mean local-dev's health-polling loop can report a stack as fully healthy when the session service or core-fe haven't actually finished booting correctly — worth knowing when a "restart succeeded" doesn't match observed behavior. **RB-07 (Deploy Rollback)** confirms, from the operational side, the same gap `CICD.md` found from the pipeline-architecture side: `make rollback TAG=<sha>` is coded correctly but unreachable, since no deploy workflow ever produces a pullable tagged image. The doc's real value is documenting the actual working alternative — a `git revert` + redeploy, which works precisely because every deploy rebuilds fresh from a branch HEAD rather than pulling a promoted artifact — and a newly-surfaced asymmetry in schema-rollback readiness: `arm-core-be` has a working `migration:revert` script, `apps/arm-admin`'s backend (also TypeORM, also migrating on startup) has none. **TS-02 (Authentication Failures)** is a symptom-first triage doc built on `AUTH-ARCHITECTURE.md`/`SESSION-FLOW.md` rather than new research — its main contribution is organizational: it explicitly separates the deny-list gap (§4, "expected current behavior, escalate as a feature gap, not an incident") from genuine bugs, and flags the platform's single most consequential environment-parity trap — local dev's Kong-absent guard fallback "passing" via the same code path a real Kong misconfiguration in staging/prod would also silently pass through, meaning a broken Kong setup doesn't fail loudly, it just behaves like local dev unexpectedly. **TS-03 (MFE Not Loading) closes out every Critical-priority item in this inventory.** It corrected its own originally-suggested §1 ("check the registry endpoint") against `DATABASE-DESIGN.md`'s independent finding that `core-be`'s `/v1/registry` and `core-fe`'s own unused `registry-service.ts` client are dead code — real remote discovery is static env vars, not a runtime lookup — a good example of a doc's own suggested-sections outline being written before the underlying reality was verified, and getting corrected once it was. All Critical items (DOC-P01–P11, DOC-A01–A08, RB-01, RB-07, TS-02, TS-03) are now done or explicitly and accurately scoped as partial (TS-01). Next up: Important tier (14 items: Kafka architecture, Kong config, ADR collection, coding standards, testing strategy, BRD, roadmap, incident response, ops guide, arm-ui-library docs, calendar API/data-model/linked-accounts, notification service guide, remaining runbooks/troubleshooting) → Optional. TS-01 (sync troubleshooting) remains the easiest fast-follow, formalizing `failure-modes.md` alongside DOC-A04's §4, DOC-A06's §6/§9, and DOC-A07's §4/§7. **DOC-P14 (Kafka Architecture)**, first item of the Important tier, surfaced a genuine actionable production gap, not just documentation, and it's now **fixed**: `docker-compose.yml` (production) had no `healthcheck:` stanza at all for `notification-service`, meaning the live/ready health-check split built earlier this pass specifically to detect a silently-dead Kafka consumer connection (`KafkaJSNumberOfRetriesExceeded` crashing the consumer with nothing else noticing) had nothing polling it in the one environment with the largest blast radius — dev and local-dev both correctly had this wired up, prod did not. Fixed 2026-08-11 in `deploy/arm-deploy-make`: added the same `/v1/health/ready` healthcheck dev/local-dev already use, verified with `docker compose config` and confirmed no other service `depends_on` `notification-service` (purely additive, no ordering side effects). `KAFKA-ARCHITECTURE.md` §8 updated to "Fixed," not left as an open gap. Also confirmed the producer side is fire-and-forget by design (a Kafka publish failure is invisible to the HTTP caller, silently dropping an OTP/welcome email with only a log line as evidence) and that the DLQ's own consumer does nothing but log — no replay path exists today, and that gap remains open (a bigger lift than a one-line compose fix). **DOC-P16 (ADR Collection)** is up next per priority order, but **DOC-P15 (Kong Configuration)** turned out to be one of the highest-yield docs of the Important tier**: it corrected its own suggested-sections assumption (Kong does validate JWTs, contrary to the original outline's §4), found Kong's `/api/core`/`/api/calendar` routes have no consumer anywhere in this workspace, corrected a 4th stale `deploy/arm-deploy-make/CLAUDE.md` claim (Kong's prometheus plugin is live, not Phase-B-deferred — `MONITORING.md` updated), and — most notably — caught and fixed a route-path error **this same documentation pass had itself introduced**: `ARCHITECTURE.md` and `WEBHOOKS.md` both previously said the webhook intake route was `POST /events/notifications`, missing calendar-be's global `v1` prefix; both are now corrected to `POST /v1/events/notifications`. That same check surfaced a real, narrow latent bug in `env.ts`'s own fallback default for `NOTIFICATION_WEBHOOK_URL` (also missing `/v1/`, not currently exercised by any shipped compose file) — a good reminder that even docs written carefully in this same pass can carry a small error forward until something else cross-checks them. **RB-05 (Database Backup and Restore) closes the last remaining Critical-tier item.** The underlying tooling was already known-working ([INFRASTRUCTURE.md](../INFRASTRUCTURE.md) §9 found this earlier), so this was primarily a documentation gap — but the doc-writing pass itself produced something genuinely new: **the first documented restore procedure for MongoDB and for `arm_admin`**, neither of which had an automated drill or even a written-down manual command before now (only `arm_core`'s Postgres restore is drill-tested via `make restore-postgres-drill`). Both are now given exact commands adapted from the existing tooling's own patterns. Restated plainly in one place: no off-host backup storage, no retention policy, no scheduling — this remains a fully manual, on-call responsibility until those three are addressed. **DOC-P16 (ADR Collection)** completed all 8 originally-planned ADRs, mostly by re-organizing already-verified facts from this pass into decision-record form rather than fresh research — but 3 of the 8 (ADR-003 MongoDB, ADR-007 Verdaccio, ADR-008 Redis Sentinel) didn't resolve into a clean "decided, done" story: ADR-003 has no preserved original rationale anywhere in the workspace (written as explicitly-flagged inference from schema shape instead); ADR-007 records a decision that was later fully reversed (Verdaccio adopted, then removed — nothing else in the codebase would tell a future reader this ever existed); ADR-008 is a rollout still only half-finished in production (2 of 4 Redis-using services aren't Sentinel-aware, `REDIS-ARCHITECTURE.md`'s highest-severity finding). Also renamed ADR-005 from its originally-planned "postMessage pattern" title to "BroadcastChannel pattern" once `GOOGLE-OAUTH.md`'s research showed BroadcastChannel is the primary mechanism, postMessage a secondary/legacy one — a small but accuracy-driven correction to the inventory's own original ADR list. **TS-01 (Calendar Sync Not Working)** was the last remaining partial item and, as expected, the fastest write of the whole pass — almost entirely assembled from already-verified facts in `SYNC-FLOW.md`, `WEBHOOKS.md`, `calendar/ARCHITECTURE.md`, and `failure-modes.md`. Its main value is organizational, same pattern as TS-02/TS-03: routing each symptom to the right already-documented mechanism, explicitly calling out which provider asymmetries are by-design rather than bugs (Microsoft poll-only, no Microsoft quota detection, monthly vs. daily renewal), and confirming RB-03 (Manual Calendar Re-sync) is a genuinely open gap — no dedicated force-resync endpoint exists today. **DOC-P17 (Coding Standards)** was scoped deliberately narrow — the in-code complement to `TEMPLATE-CONFORMANCE.md`'s repo-level conventions, not a duplicate of it — and stuck to patterns actually confirmed across the codebase (the shared `tsconfig.json` strict-flag subset, the repeated intentional facade pattern, the global `v1` prefix on every backend) rather than inventing standards the original suggested-sections outline assumed existed. Two explicit non-findings worth keeping: no platform-wide test coverage threshold exists anywhere, and pre-commit enforcement is not platform-wide — only `arm-service-notification` has a husky/lint-staged hook; everything else lints/tests at CI time only. **DOC-P18 (Release Process)** found the same `repos.conf` branch-pinning gap `runbooks/deploy-rollback.md` found from the rollback angle — application repos have no real dev→staging→prod promotion pipeline, so the doc was restructured around what's actually true (release and deploy are two unrelated mechanisms) rather than the originally-suggested promotion-checklist shape. New, previously-undocumented finding: `arm-ui-library`'s `.releaserc.json` sets `npmPublish: false`, so the CI-automated semantic-release workflow never actually publishes to npm — the real publish step is the manual one in `npm-publish.md`, and how that manual `npm version` step is meant to coexist with semantic-release's own automatic version-bump-and-commit isn't documented anywhere, flagged as unclear rather than asserted as a bug. Open from writing DOC-P04: **Risk 13** (disable-user deny-list not actually enforced) needs a product decision, not just a doc fix — flag to whoever owns admin/security before it's forgotten. Open from writing DOC-A08: poll-scheduler clear / token revoke / Microsoft subscription cleanup on unlink are engineering follow-ups listed in `DELETE-CASCADE.md` §7. Also note: `apps/arm-admin` was mid-restructure (`backend/`/`frontend/` moved under `src/`) during this pass — re-check paths in this inventory if they look stale again later. **DOC-P19 (Testing Strategy) went one level deeper than DOC-P17 and found the platform's testing story is weaker than its test *files* suggest**: `calendar-be` has the only real integration/E2E layer on the platform (Testcontainers-backed MongoDB + Redis) and its own CI never runs either suite, only unit; `core-fe`'s `ci.yml` runs lint + typecheck but never `pnpm test`, so its 8 Vitest specs never execute in CI, only the separate Playwright workflow does; `arm-session` and `admin-be` both still carry the unmodified NestJS scaffold `app.e2e-spec.ts` against an `AppModule` that registers no `AppController` in either repo — a test that would fail if it ever ran, and never does, since neither CI invokes `test:e2e`. Confirmed `admin-be` has zero automated testing at any layer (no CI at all, and not part of `deploy/arm-deploy-make`'s `make test` target either — that target was never updated to include `arm-admin` or `arm-ui-library`). Also confirmed `calendar-fe` and `admin-fe` have no test tooling whatsoever — no Vitest, no Testing Library, not even a `test` script — by direct dependency inspection. **DOC-A09 (arm-core-be: MFE Registry) corrects a factual error this pass itself had propagated across three files** — DOC-P10's "the `registry` Postgres table is dead code" line above, `DATABASE-DESIGN.md` §2.4, `core-be/API.md` §4, and `troubleshooting/mfe-not-loading.md` all stated `core-fe` never calls the registry API, each citing a `MODULE-FEDERATION.md` §2.4 that didn't exist. Verified against source instead: `core-fe/src/lib/remote-registry.ts`'s `initRemoteRegistry()` is called from `ProtectedRoute` on every authenticated session and does call `GET /v1/registry/calendar` — live, not dead. The corrected, more precise fact: the *write* path (`POST /registry/register`) is never called by anything in the workspace, so the table is permanently empty and the read never returns anything useful — the practical guidance in `mfe-not-loading.md` ("check the env var, not the registry") survives unchanged, but the reasoning behind it was wrong until now. All three previously-affected docs, plus `arm-core-be/README.md`'s separate, unrelated "placeholder org/role management" mischaracterization, were corrected in place; `MODULE-FEDERATION.md` gained the §2.4 section it was missing.
 **DOC-A10 (Linked Accounts) and DOC-A11 (Calendar API Reference)** confirmed Microsoft is documented as a live provider throughout, not future groundwork, and found the older `syncFromCoreAuth()` account-creation path has no reachable UI trigger anywhere in either frontend today (live code, not dead, just unreachable by anything a user can click) — plus that `calendar-management`'s controller has no `@Controller()` prefix at all (routes hang off the bare `v1` prefix) and that calendar-be has two incompatible error-response patterns coexisting across controllers, one of which returns HTTP 200 on a logical failure, a gap the idempotency interceptor's own code comment had to work around without ever being documented until now. **DOC-A12 (Calendar Data Model)** was written deliberately thin — every item its suggested-sections list asked for was already fully covered in `DATABASE-DESIGN.md` §5-§6, so this became a short calendar-scoped index into that document rather than a second, duplicate schema reference, plus a one-line operational summary DATABASE-DESIGN.md doesn't provide (what each collection is actually *for*). **This closes the last remaining partial item in the inventory — every item is now either Done or Not Started, zero Partial.** Also corrected in this pass: DOC-A10/DOC-A11 had been written at their originally-suggested locations (`apps/arm-app-calendar/LINKED-ACCOUNTS.md`, `apps/arm-app-calendar/API.md`) rather than moved into `docs/arm-docs/docs/calendar/` per the workspace convention established earlier this session (every doc except each repo's own `README.md`/`CLAUDE.md` lives in `arm-docs`) — moved into place alongside the other calendar docs, internal/external relative links recomputed and verified. Open from writing DOC-P04: **Risk 13** (disable-user deny-list not actually enforced) needs a product decision, not just a doc fix — flag to whoever owns admin/security before it's forgotten. Open from writing DOC-A08: poll-scheduler clear / token revoke / Microsoft subscription cleanup on unlink are engineering follow-ups listed in `DELETE-CASCADE.md` §7.
