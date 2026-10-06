# DOCS

The actual content of this repo — every ARM platform reference document, cross-cutting and per-service alike. This repo has no `src/`: documentation is the product, so `docs/` fills the role `src/` plays in an application repo.

| Subfolder | Holds |
|---|---|
| (root of `docs/`) | Platform architecture, auth, session flow, module federation, security, database, Redis, Kafka, monitoring, CI/CD, infrastructure, deploy orchestration, Kong, env vars, coding standards, testing strategy, onboarding, template conformance, production resilience notes — plus the dated audits (`PRODUCTION-AUDIT-*`, `FAILURE-ANALYSIS-*`), the risk register, and the roadmap/migration plan entry points |
| `adr/` | Architecture Decision Records — short context/decision/consequences write-ups per significant choice, cross-linked into the fuller reference docs |
| `architecture/` | Standalone architecture proposals and design specs not folded into the main reference docs |
| `roadmap/` | The seven-phase remediation roadmap — one file per phase, indexed by `IMPLEMENTATION-ROADMAP.md` |
| `plan/infra/` | The multi-server migration work breakdown (12 numbered parts + index), the execution detail behind `MULTI-SERVER-MIGRATION.md` |
| `guides/` | Step-by-step operational guides (local dev setup, GitHub secrets setup, npm publish) |
| `runbooks/` | Operational runbooks — restart services, deploy rollback, backup/restore, database server setup |
| `troubleshooting/` | Symptom-first diagnostic guides — start from what a user/on-call engineer observes, not the architecture |
| `meta/` | The documentation inventory itself (what's done, what's pending) and diagram-planning notes |
| `core-be/` | arm-core-be's `API.md` — REST API reference |
| `calendar/` | arm-app-calendar's `ARCHITECTURE.md`, `SYNC-FLOW.md`, `WEBHOOKS.md`, `GOOGLE-OAUTH.md`, `DELETE-CASCADE.md`, `DATA-MODEL.md`, `API.md`, and `LINKED-ACCOUNTS.md` |

The `BFF-RENAME-*.md` files at the root are a historical record of the `arm-core-bff` → `arm-session` rename (see [ADR-009](adr/009-rename-bff-to-session.md)), kept for archaeology rather than as current reference.

Each service keeps only its own `README.md` (project overview, quick start) and `CLAUDE.md` (Claude Code auto-loads these by directory) in its own repo — every other doc, whether platform-wide or service-specific, lives here.

`index.html` and `_sidebar.md` in this folder turn it into a browsable [Docsify](https://docsify.js.org) site (nav sidebar + search) — run `make dev` from the repo root and open `http://localhost:3300`. This is content, not code: they don't change what's in the markdown, just how it's browsed. `readme.md` (this file) is the site's homepage.
