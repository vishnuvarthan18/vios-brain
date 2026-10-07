# Welcome to ARM

This is a day-1 guide: what ARM is, how the workspace is laid out, what access you'll need, how to get a working local environment, and how to make your first change. It's deliberately non-technical where it can be — for the technical deep dives, it links out rather than repeating them.

---

## What ARM is

**ARM (Ara Resource Manager)** is a multi-tenant SaaS platform for resource and calendar management. "Multi-tenant" means multiple organizations share the platform, each with their own isolated data — users, roles, and calendar sync all scope to an organization.

The platform is being built as a set of independently deployable services rather than one large application: a shared login and session layer, a core service for users/roles/orgs, a calendar app that syncs and manages Google/Microsoft calendars, an admin portal for platform operators, a notification service, and a shared UI component library — all stitched together in the browser via Module Federation so it looks and feels like one product.

A fuller business/product picture (target customers, roadmap, success metrics) belongs in a Business Requirements Document and a Product Roadmap — neither exists yet as a written doc (tracked as `DOC-P20`/`DOC-P21` in the [documentation inventory](meta/documentation-inventory.md)). Ask whoever owns product direction if you need that context sooner than those get written.

---

## Repository structure tour

ARM is a **multi-repo workspace**, not a monorepo — ten independent git repositories checked out under one directory. `make clone-repos` fetches the six application repos listed in `deploy/arm-deploy-make/repos.conf`; the other four are checked out alongside them.

```text
arm/                                    ← workspace root (you are here)
├── core/
│   ├── arm-session/                    ← Session service: session layer, OAuth, proxy to all backends
│   ├── arm-core-be/                    ← Core backend: users, roles, orgs (PostgreSQL)
│   └── arm-core-fe/                    ← Core frontend: shell app (Module Federation host)
├── apps/
│   ├── arm-app-calendar/               ← Calendar: backend + frontend (one repo)
│   └── arm-admin/                      ← Admin: backend + frontend (one repo)
├── services/
│   └── arm-service-notification/       ← Notification service: email via Kafka + SMTP
├── library/
│   └── arm-ui-library/                 ← Shared React component library (published to npm)
├── deploy/
│   └── arm-deploy-make/                ← ALL make commands run from here
├── docs/
│   └── arm-docs/                       ← Every platform doc, including the one you're reading
├── arm-tool-cli/                       ← Scaffolder for new ARM repos (not in repos.conf — clone by hand)
└── aracreate-template-codebase/        ← The araCreate repo template (reference, not an ARM service)
```

Each repo has its own `git` history, its own `CLAUDE.md` (a dense, technical reference — written primarily for AI coding agents, but genuinely useful for a human skimming a new repo fast), and usually its own `README.md`, `TASK_SHEET.md` (pending work), and `CHANGELOG.md`.

**Four directory names don't match their GitHub repo names**, which will bite you the first time you clone or dispatch a deploy by hand: `core/arm-session` is `arm-core-bff` on GitHub (the rename never reached the remote), `apps/arm-admin` is `arm-app-admin`, `library/arm-ui-library` is `arm-library-ui-components`, and `deploy/arm-deploy-make` is `arm-devops-makefile`. The full mapping is in [ARCHITECTURE.md](ARCHITECTURE.md) §1.1.

Two `app` repos each hold a backend and a frontend under `src/`, so the eight *runnable* services don't map one-to-one onto repos — `deploy/arm-deploy-make/services.conf` is the manifest of what actually starts.

**Where to look for what:**

| If you're touching... | Look in |
| --- | --- |
| Login, sessions, "why am I logged out" | `core/arm-session`, `core/arm-core-be` |
| The main app shell, navigation, routing | `core/arm-core-fe` |
| Calendar sync, Google/Microsoft integration | `apps/arm-app-calendar` |
| User management, audit log, admin portal | `apps/arm-admin` |
| Email sending | `services/arm-service-notification` |
| Shared buttons/inputs/design tokens | `library/arm-ui-library` |
| Docker, deploy, CI, environment variables | `deploy/arm-deploy-make` |
| Any platform documentation | `docs/arm-docs` |
| Scaffolding a brand-new ARM service repo | `arm-tool-cli` |

---

## Access you'll need

Exactly who grants each of these and how isn't written down anywhere in the repos yet — ask whoever set up your onboarding for the request process. What you'll need access *to*:

- **GitHub** — the `aracreate-group` org, and read/write on the repos in `deploy/arm-deploy-make/repos.conf`
- **npm** — `@aracreate/test-arm-ui` (the shared UI library) publishes to the public npm registry, so installing it needs no special access; publishing a new version does. There is **no** internal registry — Verdaccio was removed, see [ADR-007](adr/007-verdaccio-registry.md)
- **Server access** — credentials/VPN if you'll be testing or debugging outside local dev
- **Observability dashboards** — Grafana, once the stack described in `docs/architecture/observability-plan.html` is live; ask if you're not sure it's ready yet
- **Google Cloud Console** — only if you're working on the Google OAuth / Calendar API integration specifically, for test credentials and quota visibility

---

## Local development setup

Full step-by-step instructions — prerequisites, `.env` generation, cloning the repos, starting infrastructure, building images, day-to-day commands, and a troubleshooting section — already exist: **[docs/guides/local-dev-setup.md](guides/local-dev-setup.md)**. Follow that end to end before writing any code; don't duplicate it here.

The short version, once you've read that guide: `make env-init` to generate secrets, fill in the handful of values that can't be auto-generated (Google OAuth credentials, SMTP), then `make arm` for first-time setup and `make arm-run` every day after.

---

## Architecture primer

Read the root [CLAUDE.md](../../../CLAUDE.md) next — it's short, and it covers the constraints that aren't obvious from reading code: why there's exactly one session service (not one per app), why Kong isn't in the local-dev path, how the frontends are stitched together with Module Federation, and which environment variables have cross-repo consistency requirements that will bite you if you get them wrong.

Then read [ARCHITECTURE.md](ARCHITECTURE.md) — the consolidated platform overview: the repo inventory, the service/port table, edge routing per environment, database ownership, Module Federation topology, and the four data-flow walkthroughs. Its §10 is the index into every other document here, so you don't have to guess which one answers your question.

The two to read after that, in order:

- [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) — how login, sessions, and admin access control work end to end
- [SESSION-FLOW.md](SESSION-FLOW.md) — the session/proxy layer everything else sits behind (implementation notes in [core/arm-session/README.md](../../../core/arm-session/README.md))

If you'd rather browse than grep, `make dev` in `docs/arm-docs` serves this whole document set as a searchable site at `http://localhost:3300`.

---

## Making your first change

**Branching:** every repo works off a `dev` branch day-to-day, with `main` reserved for the server. `deploy/arm-deploy-make`'s CI deploys `main` → server, and that is the only automated deploy path — pushing to `dev` runs CI but deploys nothing (confirmed against its own GitHub Actions workflows). CI setup varies by repo right now (some have a full `ci.yml`, some don't have one yet — don't assume every repo enforces the same checks). Feature branch naming is loosely `feat/<name>` or `fix/<name>` in practice, not a strictly enforced convention.

**Commits** follow [TEMPLATE-CONFORMANCE.md](TEMPLATE-CONFORMANCE.md)'s conventions:

- Conventional commit prefixes (`feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, ...)
- No `Co-Authored-By` trailers
- No task/ticket IDs, emails, or usernames in the commit message itself — that detail belongs in the repo's own `TASK_SHEET.md`, not the commit log

**PR template:** none exists yet in any repo as of this writing — there's no `.github/PULL_REQUEST_TEMPLATE.md` to fill out. Write a clear description of what changed and why until one exists.

**Before you open a PR:**

1. Check the repo's `TASK_SHEET.md` — is what you're doing already tracked, in progress, or explicitly decided against?
2. Run that repo's tests and lint locally (`make test`, `make lint` from the repo root, or via the workspace's `make local-dev-*` targets)
3. If your change touches `.env` variables, run `make env-check` from `deploy/arm-deploy-make` — several vars (JWT secrets, Mongo password) have cross-repo consistency requirements this catches

---

## Team contacts and escalation path

Not yet formalized in writing — every file header across every repo currently attributes to a single author (`Kishor <kishor@aracreate.group>`), which reflects the platform's current team size rather than an org chart worth writing down permanently. Whoever ran your onboarding should tell you directly who to ask for what; update this section once there's a team large enough to need it.

---

*If a section here goes stale — paths change, a doc gets written, a process gets formalized — update it. This is meant to be the thing a new engineer actually reads first, not a snapshot frozen at whatever date it was written.*
