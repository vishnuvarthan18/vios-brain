# ARM Docs

Every reference document for the ARM platform, in one place — both platform-wide docs and per-service reference docs. ARM (Ara Resource Manager) is a multi-tenant SaaS platform assembled from nine independent git repositories — this repo is the tenth: all documentation except each service's own `README.md` and `CLAUDE.md` lives here, so it doesn't get scattered across (or lost inside) any single service's repo.

- Full platform architecture, security posture, database schema, Redis usage, monitoring/observability, CI/CD pipelines, and infrastructure reference — all verified against live source, not just design intent
- Onboarding guide for new engineers, plus the araCreate template-conformance checklist used to bring every repo in the workspace in line with [aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions) (checked out locally as [`aracreate-template-codebase`](../../aracreate-template-codebase))
- Per-service reference docs — e.g. `core-be/API.md`, `calendar/GOOGLE-OAUTH.md`, `calendar/DELETE-CASCADE.md` — grouped under a subfolder per service
- The documentation inventory itself (`docs/meta/documentation-inventory.md`) — what's written, what's pending, and every verified drift/bug found while writing each doc
- What's **not** here: each service's own `README.md` (project overview, quick start) and `CLAUDE.md` (Claude Code auto-loads these by directory, so they must stay put) — everything else moves here

## Stack

| Layer | Tool |
| --- | --- |
| Content | Markdown (+ one HTML file, `docs/architecture/observability-plan.html`) |
| Site | [Docsify](https://docsify.js.org) — renders the markdown into a browsable, searchable site at runtime, no build step |
| Task runner | make |
| Release automation | semantic-release (not yet wired in — see `CHANGELOG.md`) |

`make dev` serves `docs/` as a site (sidebar nav + search) at `http://localhost:3300` via `docsify-cli` (runs through `npx`, no local dependency to install). There's still no build step — Docsify fetches and renders the `.md` files client-side, so the files themselves stay the source of truth and remain just as readable directly on GitHub or in an editor.

## Layout

```text
docs/
├── ARCHITECTURE.md              ← platform architecture overview
├── AUTH-ARCHITECTURE.md         ← login flow, JWT, guards, RBAC
├── SESSION-FLOW.md              ← session lifecycle, proxy, Redis schema
├── MODULE-FEDERATION.md         ← MF host/remote wiring, shared deps, CSS strategy
├── DATABASE-DESIGN.md           ← Postgres + MongoDB schema reference
├── REDIS-ARCHITECTURE.md        ← Redis key namespaces, BullMQ queues, Sentinel HA
├── KAFKA-ARCHITECTURE.md        ← Kafka producer/consumer, topics, DLQ, KRaft cluster config
├── SECURITY.md                  ← threat model, OWASP coverage, auth guard verification
├── ENV-VARS.md                  ← every env var, per service
├── CODING-STANDARDS.md          ← in-code conventions: NestJS structure, guards, facades, error handling
├── TESTING-STRATEGY.md          ← test layers, per-repo coverage, the safe `test:agent` runner
├── TEMPLATE-CONFORMANCE.md      ← araCreate template checklist for this workspace
├── API-SURFACE-REVIEW.md        ← review of every public endpoint across the platform
├── ONBOARDING.md                ← day-1 guide for new engineers
│
├── INFRASTRUCTURE.md            ← compose files, Kong/Caddy, network topology
├── DEPLOY-ORCHESTRATION.md      ← services.conf, per-repo descriptors, generated/ compose blocks
├── KONG-CONFIG.md               ← Kong routes, JWT plugin, X-User-Id injection, rate limiting
├── CICD.md                      ← every GitHub Actions workflow, deploy pipeline
├── RELEASE-PROCESS.md           ← release vs. deploy, per-repo automation, arm-ui-library's npm publish gap
├── MONITORING.md                ← logs, metrics, tracing, alerting, dashboards
├── PRODUCTION-RESILIENCE-NOTES.md ← general production-engineering reference (not ARM-specific)
│
├── PRODUCTION-AUDIT-2026-08-21.md   ← dated deep production audit
├── FAILURE-ANALYSIS-2026-08-21.md   ← dated failure analysis + infrastructure risk report
├── INFRASTRUCTURE-RISKS.md          ← standing infrastructure risk register
├── IMPLEMENTATION-ROADMAP.md        ← index over the seven remediation phases
├── MULTI-SERVER-MIGRATION.md        ← single-host → multi-server (S1–S4) migration plan
├── BFF-RENAME-{INVENTORY,TASKS,P0-RECORD}.md ← historical record of the arm-core-bff → arm-session rename
│
├── adr/                         ← Architecture Decision Records (001–009)
├── architecture/                ← standalone architecture proposals and design specs
├── roadmap/                     ← PHASE-1 … PHASE-7 of the remediation roadmap
├── plan/infra/                  ← the 12-part multi-server migration work breakdown
├── guides/                      ← step-by-step operational guides (local dev, GitHub setup, npm publish)
├── runbooks/                    ← operational runbooks (restart services, deploy rollback, backup/restore, DB server setup)
├── troubleshooting/             ← symptom-first diagnostic guides
├── meta/                        ← the documentation inventory + diagram planning notes
├── core-be/
│   └── API.md                   ← arm-core-be REST API reference
└── calendar/
    ├── ARCHITECTURE.md          ← Calendar app backend/frontend architecture, sync/webhook engine
    ├── SYNC-FLOW.md             ← Sync engine at the algorithm level — initial sync, delta processing, teardown
    ├── WEBHOOKS.md              ← Google webhook registration, intake validation, renewal, the renewal-visibility gap
    ├── GOOGLE-OAUTH.md          ← Calendar Add Account / Reconnect Google OAuth flow
    ├── DELETE-CASCADE.md        ← Account unlink cascade — ordered cleanup, ID contract
    ├── DATA-MODEL.md            ← Calendar-scoped index into DATABASE-DESIGN.md's schema reference
    ├── API.md                   ← Every calendar-be HTTP endpoint
    └── LINKED-ACCOUNTS.md       ← The Linked Accounts feature (core-fe UI + calendar-be endpoints)
```

Every document cross-links the others (and each service's own `README.md`/`CLAUDE.md` that stayed in its repo) via relative paths — that only resolves correctly when this repo is checked out inside the assembled ARM workspace at `docs/arm-docs/`, alongside the other nine repos, not cloned standalone.

## Commands

```sh
make install    # nothing to install yet — docsify-cli runs via npx, no local deps
make setup      # nothing to set up yet
make dev        # serve docs/ as a browsable site (Docsify) at http://localhost:3300
make build      # nothing to build — Docsify renders markdown at runtime, no build step
make test       # nothing to test yet — add a markdown linter / link-checker here when one exists
make release    # cut a semantic release
make clean      # nothing to clean yet
```

`make help` (default) prints the full target list.

## Conventions

Repo structure, file headers, naming, and git conventions follow [aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions) — formerly `aracreate-template-codebase` and still checked out locally under that name at [`../../aracreate-template-codebase`](../../aracreate-template-codebase) — see that repo for the full rationale, and [`docs/TEMPLATE-CONFORMANCE.md`](docs/TEMPLATE-CONFORMANCE.md) for the distilled checklist used across this workspace. This repo has no `src/` — `docs/` fills that role, since documentation is what this repo ships, not application code.

## License

Proprietary — see [LICENSE](./LICENSE), Copyright (C) 2026, araCreate Group.
