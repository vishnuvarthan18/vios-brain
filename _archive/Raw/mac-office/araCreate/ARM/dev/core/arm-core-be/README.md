# ARM Core Backend

Core backend for the ARM platform. Owns user identity/authentication, JWT issuance, and Google/Microsoft OAuth token storage. All auth routes are accessed through the platform's single BFF (`arm-bff`) — this service is never called directly from the browser.

- Email-OTP login and Google/Microsoft SSO, both auto-provisioning accounts on first login (password login, signup, and reset were removed entirely)
- Issues and rotates JWT access/refresh token pairs; refresh tokens are stored as hashes, grouped by rotation family (stolen-token defense)
- Stores encrypted Google/Microsoft OAuth tokens on behalf of `arm-app-calendar` (calendar-be fetches them via internal, `X-BFF-Secret`-guarded endpoints)
- Publishes login-OTP and welcome-email events to Kafka, consumed by `arm-service-notification`
- `registry` module — service/MFE self-registration for Module Federation remote-URL discovery (write path unused today — see [REGISTRY.md](./REGISTRY.md))

## Stack

| Layer | Tool |
| --- | --- |
| Framework | NestJS |
| ORM | TypeORM |
| Database | PostgreSQL |
| Cache | Redis (OTP storage, rate-limit counters) |
| Messaging | Kafka (`@nestjs/microservices`, publish-only) |
| Auth | Passport (JWT strategy, Google OAuth2, Microsoft OAuth2) |
| Language | TypeScript |
| Package manager | pnpm |
| Tests | Jest |

## Architecture

```text
Browser → arm-bff (session, JWT never reaches the browser) → arm-core-be (:4000 host / :3000 container)
                                                                 │
                                                                 ├─ PostgreSQL — users, refresh_tokens, google_tokens, microsoft_tokens
                                                                 ├─ Redis — OTP codes + attempt counters
                                                                 └─ Kafka → arm-service-notification (login-OTP, welcome emails)
```

- `src/auth/` — email-OTP + Google/Microsoft SSO login, JWT issuance, `KongJwtGuard` (Kong header in prod, Passport JWT fallback in local dev)
- `src/users/` — user CRUD (`create`/`getAll`/`remove` are permanently disabled on the controller)
- `src/registry/` — service/MFE registry for Module Federation remote-URL discovery (see [REGISTRY.md](./REGISTRY.md))
- `src/database/migrations/` — TypeORM migrations, run at startup
- `src/libs/` — Kafka producer, Redis service, (secret removed) token encryption

See this repo's own [CLAUDE.md](./CLAUDE.md) for the full auth-flow breakdown, guard behavior, and API endpoint table, and the workspace-level [CLAUDE.md](../../CLAUDE.md) for how this service fits into the platform (BFF, calendar-be).

## Commands

No `Makefile` yet (tracked as `TPL-004` in `TASK_SHEET.md`) — today this repo is driven directly via its `package.json` scripts:

```sh
pnpm install      # install dependencies
pnpm start:dev    # run locally with hot reload
pnpm build        # compile to dist/
pnpm test         # unit tests
pnpm test:e2e     # e2e tests
pnpm lint         # eslint (check only)
pnpm lint:fix     # eslint --fix
```

In the full ARM workspace, this service normally runs via `make local-dev` from `deploy/arm-deploy-make/` — see the workspace [CLAUDE.md](../../CLAUDE.md) for the standard startup flow.

## Conventions

Repo conventions (file headers, naming, versioning, Makefile targets) follow [aracreate-template-codebase](../../aracreate-template-codebase). Template-conformance work for this repo is tracked in `TASK_SHEET.md`.

## License

Proprietary — see [LICENSE](./LICENSE), Copyright (C) 2026, araCreate Group.
