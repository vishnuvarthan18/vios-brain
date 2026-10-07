# arm-admin

Admin portal for the ARM platform — user management and audit log. One repo containing two independently-built apps: a NestJS backend and a React/Vite frontend.

- Backend (`src/backend/`) manages admin-only user actions (disable/enable) and records an audit trail; reads from both the `arm_core` and `arm_admin` PostgreSQL databases (two TypeORM datasources)
- Frontend (`src/frontend/`) is a Module Federation **remote** — loaded by the `arm-core-fe` shell at `/admin/*`, not accessed directly
- Admin-only access, enforced via `AdminRoleGuard`

## Stack

| Layer | Tool |
| --- | --- |
| Backend framework | NestJS |
| Frontend framework | React 19 + Vite, Module Federation remote |
| Database | PostgreSQL (TypeORM) — `arm_core` (read) + `arm_admin` (own tables) |
| Cache | Redis (deny-list) |
| Language | TypeScript |
| Package manager | pnpm |
| Tests | Jest (backend) |

## Architecture

```text
arm-core-fe (shell)
        │  loads AdminApp remote, mounted at /admin/*
        ▼
frontend (Vite/Module Federation) ──/bff/proxy/admin/v1──▶ arm-bff ──▶ backend (NestJS)
                                                                            │
                                                    ┌───────────────────────┼───────────────────────┐
                                                    ▼                       ▼                       ▼
                                              arm_core (Postgres,     arm_admin (Postgres,      Redis
                                              read: users, tokens)    own: admin_users,          (deny-list)
                                                                       audit_log, disabled_users)
```

- `src/backend/src/users/` — admin user list/disable/enable endpoints
- `src/backend/src/audit/` — audit log write + query endpoints
- `src/backend/src/auth/` — `AdminRoleGuard`, `KongJwtGuard`, current-admin decorator
- `src/backend/src/database/entities/{admin,core}/` — TypeORM entities for both datasources
- `src/backend/src/redis/` — deny-list service (disabled users blocked at the gateway)
- `src/frontend/src/pages/{users,audit}/` — user management and audit log UI

See the workspace-level [CLAUDE.md](../../CLAUDE.md) for how this app fits into the platform (BFF, core-be), and this repo's own `src/backend/CLAUDE.md` / `src/frontend/CLAUDE.md` for app-specific detail.

## Commands

```sh
make install       # install dependencies for both packages
make setup         # create .env from .env.example in both packages if missing
make dev           # run backend + frontend together with hot reload
make dev-backend   # run only the backend
make dev-frontend  # run only the frontend
make build         # build both packages
make test          # run backend unit tests
make test-e2e      # run backend e2e tests
make lint          # eslint (check only) on both packages
make lint-fix      # eslint --fix on both packages
make clean         # remove build artefacts from both packages
```

`make help` (default) prints the full target list.

In the full ARM workspace, this app normally runs via `make local-dev` from `deploy/arm-deploy-make/` — see the workspace [CLAUDE.md](../../CLAUDE.md) for the standard startup flow.

## Conventions

Repo conventions (file headers, naming, versioning, Makefile targets) follow [aracreate-template-codebase](../../aracreate-template-codebase). Template-conformance work for this repo is tracked in `TASK_SHEET.md`.

## License

Proprietary — see [LICENSE](./LICENSE), Copyright (C) 2026, araCreate Group.
