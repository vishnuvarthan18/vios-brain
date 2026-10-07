# arm-make

*Cloned into this workspace as `deploy/arm-deploy-make/` — the directory name differs from the repo name. Formerly `arm-devops-makefile`.*

> **Start here.** `deploy/arm-deploy-make/` is the entry point for the whole ARM platform. Clone this repo first, then run `make arm` from inside it — that clones every other repo into the workspace around it, generates `.env`, brings up infra, and builds the images. A root-level delegating `Makefile` also forwards `make <target>` from the workspace root (`arm/`) to here; both are equivalent.
>
> ```sh
> git clone git@github.com:aracreate-group/arm-make.git arm/deploy/arm-deploy-make
> cd arm/deploy/arm-deploy-make
> make env-init   # generate secrets into .env, then fill GOOGLE_* and SMTP_* by hand
> make arm        # setup wizard: clone-repos -> env check -> infra up -> build images
> make arm-run    # day-to-day start (idempotent)
> ```
>
> Then open **http://localhost:3000** — the core-fe shell is the only URL the browser needs.

Deployment and development orchestration for the ARM platform. Owns the root `Makefile`, every Docker Compose file (local dev, server, infra), environment variable templates/validation, repo clone/update logic, the interactive setup wizard, CI/CD pipeline targets, and the Kong/Caddy edge configs. It owns nothing about application logic — it only orchestrates the platform's other independent repos, listed below.

- Clones and tracks all platform repos from `repos.conf`
- Generates and validates `.env` (secrets, cross-service constraints like matching JWT secrets / Mongo password)
- Brings up infra (Postgres, MongoDB, Redis, Kafka) and the full app stack, in local and server shapes
- Drives CI/CD builds, image tagging, deploys, and rollbacks

## Platform Repositories

All repos live under the [`aracreate-group`](https://github.com/aracreate-group) GitHub org
(all private except where noted). You clone *this* repo by hand; `make clone-repos` then reads
[`repos.conf`](./repos.conf) and clones the six application repos marked ✓ below around it. The
library, CLI and docs repos are **not** in `repos.conf` — clone them by hand into the paths shown
if you need them locally.

**Directory names do not always match repo names, and several repos were renamed** — the
mapping in this table is the current one, verified against GitHub. Old names still redirect,
so existing clones keep working, but new references should use the current name.

| Component | `repos.conf` | Workspace path | Repository | Renamed from |
| --- | :---: | --- | --- | --- |
| Deploy / orchestration (**this repo, start here**) | — | `deploy/arm-deploy-make/` | [arm-make](https://github.com/aracreate-group/arm-make) | `arm-devops-makefile` |
| Core backend — users, roles, orgs (PostgreSQL) | ✓ | `core/arm-core-be/` | [arm-core-be](https://github.com/aracreate-group/arm-core-be) | — |
| Core frontend — shell, Module Federation host | ✓ | `core/arm-core-fe/` | [arm-core-fe](https://github.com/aracreate-group/arm-core-fe) | — |
| Session service — sessions, OAuth, backend proxy | ✓ | `core/arm-session/` | [arm-session](https://github.com/aracreate-group/arm-session) | `arm-core-bff` |
| Calendar app — backend + frontend | ✓ | `apps/arm-app-calendar/` | [arm-app-calendar](https://github.com/aracreate-group/arm-app-calendar) | — |
| Admin app — backend + frontend | ✓ | `apps/arm-admin/` | [arm-app-admin](https://github.com/aracreate-group/arm-app-admin) | — |
| Notification service — Kafka in, SMTP out | ✓ | `services/arm-service-notification/` | [arm-service-notification](https://github.com/aracreate-group/arm-service-notification) | — |
| Shared React UI component library | — | `library/arm-ui-library/` | [arm-ui-library](https://github.com/aracreate-group/arm-ui-library) | `arm-library-ui-components` |
| CLI — scaffolds core modules and apps | — | `arm-tool-cli/` | [arm-cli](https://github.com/aracreate-group/arm-cli) | `arm-tool-cli` |
| Platform documentation | — | `docs/arm-docs/` | [arm-docs](https://github.com/aracreate-group/arm-docs) | — |

Not cloned by `clone-repos`, but referenced by this workspace:

| Purpose | Workspace path | Repository | Renamed from |
| --- | --- | --- | --- |
| araCreate repo conventions (public) | `aracreate-template-codebase/` | [aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions) | `aracreate-template-codebase` |
| Reference — UI design system for core + calendar | `ref/arm-design-ui/` | [arm-ref-ui](https://github.com/aracreate-group/arm-ref-ui) | `arm-design-ui` |
| Reference — design components + full design system | — | [arm-ref-ds](https://github.com/aracreate-group/arm-ref-ds) | — |

## Deployed Environments

There are exactly two environments: **local** and **server**. Server values come from the
`production` GitHub environment on [arm-make](https://github.com/aracreate-group/arm-make)
(`PROD_*` variables), which `ci-deploy-prod` renders into the server's `.env`.

| Environment | URL | Edge |
| --- | --- | --- |
| Local dev | http://localhost:3000 | none — services on published host ports |
| Server (`SITE_DOMAIN`) | **https://dev.arametrics.app** | Caddy (browser) + Kong (`/api/*`) |
| Server dashboards (`GRAFANA_DOMAIN`) | https://grafana.arametrics.app | Caddy; Grafana's own login |

Server routes off `https://dev.arametrics.app`:

| Route | Serves |
| --- | --- |
| `/` | core-fe shell (MF host) |
| `/calendar` | calendar-fe remote |
| `/admin` | admin-fe remote |
| `/session/*` | session service (OAuth callbacks, backend proxy) |
| `/api/core/v1` | core-be, via Kong |
| `/api/calendar/v1` | calendar-be, via Kong |

## Stack

| Layer | Tool |
| --- | --- |
| Orchestration | GNU Make (modular `make/*.mk` includes) |
| Containers | Docker Compose (infra, local-dev, server variants) |
| Scripting | Bash |
| Edge (local dev) | none — services are reached on published host ports |
| Edge (server) | Caddy (browser) + Kong (`/api/*`) |
| Observability | Prometheus, Grafana, Loki, Alloy, Tempo |

## Architecture

The root `Makefile` is include-only; logic lives in `make/*.mk`, one file per concern:

| File | Responsibility |
| --- | --- |
| `make/config.mk` | Paths, compose file variables, colours, `REQUIRED_VARS` tiers |
| `make/env.mk` | `env-init` (secret generation), `env-check` (validation) |
| `make/arm.mk` | `arm` (setup wizard), `arm-run` (daily start), sentinel management |
| `make/repos.mk` | `clone-repos`, `update-repos`, `repos-status` |
| `make/infra.mk` | `infra-up`, `infra-down`, `infra-wait`, `infra-status`, and the split-estate pair `infra-up-app` / `infra-up-db` |
| `make/install.mk` | `install`, `install-session` |
| `make/apps.mk` | `up`/`down`/`restart` — the server stack, build targets |
| `make/dev.mk` | `dev-core-be`, `dev-session`, etc. — pnpm hot-reload locally |
| `make/local-dev.mk` | `local-dev`, `local-dev-build`, `local-dev-rebuild`, health polling |
| `make/descriptor.mk` | `descriptor-check`, `generate-compose`, `generate-compose-prod` |
| `make/kong.mk` | `generate-kong-config` — renders `kong/kong.yml.tmpl` into `generated/kong.yml` |
| `make/tune.mk` | `tune-infra` — sizes infra containers to the host |
| `make/test.mk` | `test`, `test-all`, `smoke-test`, per-service test targets |
| `make/ci.mk` | `ci-setup`, `ci-build`, `ci-push`, `ci-deploy-prod`, `rollback` |
| `make/utils.mk` | `health`, `ps`, `logs`, `gateway-reload`, `clean*` |
| `make/backup.mk` | `backup-postgres`, `backup-mongodb`, `backup-all`, restore drill |

See [CLAUDE.md](./CLAUDE.md) for the full setup-wizard flow, environment variable constraints, and Makefile coding conventions (`$$` escaping, single-`@`-block rule, no `make -j`).

## Commands

```sh
make arm        # first-time setup wizard (repos → env → infra → images)
make arm-run    # daily start (idempotent)
make install    # install dependencies for all services
make dev        # start dev stack in Docker
make build      # build production images
make test       # run all services' test suites
make test-self  # self-tests for this repo's own Make logic (no Docker, no network)
make clean      # remove build artefacts
```

`make help` (default) prints the full target list — dozens of targets beyond the standard set above, grouped by category (repos, infra, backups, full stack, local dev, CI/CD). This Makefile is the standalone orchestrator for the whole platform, not a single-repo entry point — there's no separate per-repo Makefile to fall back to.

### Databases on their own host

The server can run as one machine or two. The single-host path is `infra-up-prod`
(all four engines together). The split path puts PostgreSQL and MongoDB on a
dedicated database host and leaves Redis and Kafka with the applications —
Redis because the session service reads it on every request, Kafka because it
advertises its brokers by container hostname, so an off-host client cannot
connect to what the broker hands back.

```sh
# on the database host, by hand over the VPN
make infra-up-db          # needs DB_BIND_IP (its own private IP) in .env
make infra-wait-db        # waits, then creates arm_admin + the monitoring roles
./scripts/db-firewall.sh apply 10.0.0.2   # allow only the app host (arm-master); 10.0.0.3 is the VPN gateway, not the app host

# on the application host — both are prerequisites of ci-deploy-prod
make check-db-reachable   # TCP-probes the DB host before anything is built
make infra-up-app         # Redis + Kafka only
```

The app services find the databases through `DB_HOST`, `ARM_CORE_DB_HOST`,
`ARM_ADMIN_DB_HOST` and `MONGO_HOST`. All four are required on the server
(`env-check-server`): left blank, every compose default falls back to a
container name on the app host, and the platform would quietly serve a local
database instead of the real one. `MONGODB_URI` carries its own copy of the
host and is checked against `MONGO_HOST` for drift.

Both halves refuse to do the silent-wrong-thing: `infra-up-db` will not create
empty databases without `CONFIRM_FRESH_INFRA=yes`, and the backup and
restore-drill targets refuse to run on a host where the database container
they dump from is not present.

## Conventions

Repo conventions (file headers, naming, versioning) follow [aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions) (formerly `aracreate-template-codebase`, still checked out locally under that name at [`../../aracreate-template-codebase`](../../aracreate-template-codebase)). Template-conformance work for this repo is tracked in `TASK_SHEET.md`.

## License

Proprietary — see [LICENSE](./LICENSE), Copyright (C) 2026, araCreate Group.
