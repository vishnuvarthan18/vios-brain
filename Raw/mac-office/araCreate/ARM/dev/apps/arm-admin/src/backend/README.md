# ARM Admin — Backend

The admin backend for the ARM platform (`arm-admin-be`). NestJS + TypeORM + PostgreSQL. Newest service in the workspace — not present when the platform's [documentation inventory](../../../../docs/arm-docs/docs/meta/documentation-inventory.md) was first generated.

- Admin user management — list, view, update, disable/enable, promote/demote, and hard-delete platform users
- Deny-list — writes a Redis key intended to let other services reject a disabled user's active session immediately (see the caveat in [Auth & Access Control](#auth--access-control) below — this isn't fully wired up yet)
- Audit log — immutable record of every admin write operation
- Admin portal access control — the `admin_users` table gates who can use the portal at all

This is the only ARM service that talks to **two** PostgreSQL databases at once: `arm_core` (core-be's database — read-write, no migrations run here) and `arm_admin` (its own database, migrations run on startup).

Runs on port `10001` (host). The admin frontend (`apps/arm-admin/src/frontend`) is its only consumer, reached through the BFF proxy at `/bff/proxy/admin/v1/...`.

## Stack

| Layer | Tool |
| --- | --- |
| Framework | NestJS 11 |
| Databases | PostgreSQL (`arm_core` + `arm_admin`) via TypeORM, two named connections |
| Session/cache | Redis (`ioredis`) — deny-list keys + admin-role cache |
| Logging | `nestjs-pino` |
| Metrics | `@willsoto/nestjs-prometheus` |
| Rate limiting | `@nestjs/throttler` |
| Language | TypeScript |
| Package manager | pnpm |

## Architecture

Sits behind `arm-bff` like every other ARM backend — the frontend never calls it directly. Auth is two guards in sequence (`KongJwtGuard` → `AdminRoleGuard`): the first establishes *who* the caller is from a normal platform JWT, the second checks whether that user is *also* an admin. There's no separate admin login — any platform user who exists in `admin_users` gets elevated access on their regular session.

```text
src/
├── app.module.ts                          ← Root module — wires everything
├── main.ts                                ← Bootstrap: v1 prefix, CORS, pipes, TypeORM error filter, metrics
├── config/
│   └── env.validation.ts                  ← Joi schema for all env vars
├── database/
│   ├── database.module.ts                 ← Two named TypeORM connections + seeders
│   ├── entities/
│   │   ├── core/                          ← Read-only mirrors of arm_core tables (users, refresh_tokens, google_tokens)
│   │   └── admin/                         ← arm_admin-owned tables (admin_users, audit_logs, disabled_users)
│   ├── migrations/                        ← Only ever run against arm_admin
│   └── seeders/
│       ├── admin-user.seeder.ts           ← Seeds the first admin from ADMIN_SEED_USER_ID
│       └── deny-list.seeder.ts            ← Re-populates the Redis deny-list from disabled_users on startup
├── auth/
│   ├── admin.controller.ts                ← GET /v1/admin/me
│   ├── strategies/jwt.strategy.ts         ← Passport JWT; validate() returns { userId }
│   ├── guards/
│   │   ├── kong-jwt.guard.ts              ← Dual-path: trusted X-User-Id header, or JWT verification
│   │   └── admin-role.guard.ts            ← Checks admin_users; caches role in Redis (5 min)
│   └── decorators/current-admin.decorator.ts
├── redis/
│   └── deny-list.service.ts               ← disabled:{userId} keys, 900s TTL
├── users/
│   ├── users.repository.ts                ← All queries against arm_core
│   ├── users.service.ts                   ← list, get, update, disable, enable, delete
│   └── users.controller.ts                ← GET/PATCH/POST/DELETE /v1/users
├── audit/
│   ├── audit.service.ts                   ← log() never throws; findAll() for the controller
│   └── audit.controller.ts                ← GET /v1/audit
└── health/
    └── health.controller.ts               ← GET /v1/health (public, no auth)
```

### Two TypeORM connections

Every repository/DataSource injection must name its connection explicitly — there is no default:

```typescript
@InjectRepository(CoreUser, 'arm_core')
@InjectRepository(AdminUser, 'arm_admin')
```

Omitting the name doesn't fall back to anything — NestJS throws at startup, since no unnamed default connection exists. The `arm_core` connection is `synchronize: false` with no migrations configured here; this service reads and writes `arm_core` data (e.g. revoking refresh tokens) but never alters its schema. Any schema change to `arm_core` goes through `arm-core-be`.

### Startup sequence

1. TypeORM `arm_core` connection established (no migrations)
2. TypeORM `arm_admin` connection established → pending migrations run (`migrationsRun: true`)
3. `onApplicationBootstrap`: `AdminUserSeeder` seeds the first admin from `ADMIN_SEED_USER_ID` if `admin_users` is empty, then `DenyListSeeder` re-populates the Redis deny-list from the `disabled_users` table
4. NestJS begins accepting requests

Migrations always run before seeders; both seeders are idempotent, safe to run on every restart.

## Auth & Access Control

Every protected endpoint runs `KongJwtGuard` then `AdminRoleGuard`:

- **`KongJwtGuard`** — same dual-path pattern as `arm-core-be` and calendar-be: trusts Kong's `X-User-Id` header only when `TRUST_PROXY_HEADERS=true` (production/staging behind Kong); otherwise falls through to verifying the JWT signature via Passport. `TRUST_PROXY_HEADERS` must stay `false` anywhere this service isn't genuinely deployed behind Kong — this gate exists specifically because trusting that header unconditionally was once a real auth-bypass bug (tracked elsewhere as SEC-001). In local dev, Kong is not in the path at all; the BFF forwards the JWT directly.
- **`AdminRoleGuard`** — runs second, reads `request.user.userId`, and looks it up in `admin_users`. Not found → `403`. Found → sets `request.admin = { userId, role }` and caches `admin_role:{userId}` in Redis for 5 minutes (falls back to a DB query and a warning log if Redis is unreachable — never fails closed on Redis being down). After revoking or changing someone's admin role, their cached role can stay live for up to 5 minutes; there's no active cache invalidation yet.
- **`@CurrentAdmin()`** — controller-param decorator that extracts the resolved `request.admin`.

### Deny-list — disable/enable a user

Disabling a user is a two-layer write, both driven from `UsersService.disableUser`:

1. Revoke all their refresh tokens in `arm_core.refresh_tokens` (`revoked = true`)
2. Insert a row into `arm_admin.disabled_users` (inside a transaction)
3. Write `disabled:{userId}` to Redis (900s TTL) — only *after* the DB row commits, so a deny-list key can never exist without a backing row
4. Write an audit log entry (`USER_DISABLED`)

Enabling reverses steps 1 and 3 (delete the Redis key, delete the row, audit-log `USER_ENABLED`). `DenyListSeeder` re-writes every `disabled_users` row to Redis on every service startup, so the deny-list survives a Redis restart. `disableUser` is idempotent — disabling an already-disabled user is a no-op, which avoids a unique-constraint error on `disabled_users.userId`.

**This mechanism is currently incomplete.** The 900-second TTL is chosen to match the platform's default access-token lifetime (`JWT_ACCESS_TOKEN_TTL` in core-be), on the assumption that core-be's and calendar-be's auth guards check this Redis key before honoring a request — that's what would make disabling someone take effect almost immediately instead of waiting for their token to expire naturally. **Neither guard currently does this check** (verified by searching both repos' `src/auth` directories — the `disabled:` key pattern appears nowhere outside this service). Until it's wired in on at least one side, disabling a user revokes their ability to get a *new* access token but does not invalidate a *currently held* one — a disabled user can keep working against core-be and calendar-be for up to the access token's remaining lifetime (default 15 minutes). See the platform-level [`AUTH-ARCHITECTURE.md`](../../../../docs/arm-docs/docs/AUTH-ARCHITECTURE.md) (§4) and the documentation inventory's Risk 13 for the full writeup — this needs a product decision (wire in the check, shorten the TTL, or explicitly accept the window), not just a comment fix.

## Modules

### `users`

CRUD plus lifecycle operations over `arm_core.users`, all guarded by `KongJwtGuard` + `AdminRoleGuard`:

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/v1/users` | Paginated list, search/filter |
| `GET` | `/v1/users/:id` | Single user detail, including disabled/Google-connection status |
| `PATCH` | `/v1/users/:id` | Update name, email, picture |
| `POST` | `/v1/users/:id/disable` | Disable — see the deny-list mechanism above |
| `POST` | `/v1/users/:id/enable` | Re-enable — clears the deny-list |
| `POST` | `/v1/users/:id/grant-admin` | Grants `AdminRole.ADMIN`; no-op if already an admin |
| `POST` | `/v1/users/:id/revoke-admin` | Revokes admin access; `400` if you try to revoke your own |
| `DELETE` | `/v1/users/:id` | Hard delete from `arm_core` plus cleanup |

`arm_admin` users are not a separate identity from `arm_core` users — every admin operation reads and mutates the same `arm_core.users` row core-be owns, via the read-write `arm_core` TypeORM connection described above. Being "an admin" is purely a row in `arm_admin.admin_users` keyed by that same user ID.

### `audit`

`AuditService.log()` writes to the immutable `audit_logs` table and **never throws** — a failure to write an audit entry is logged, not re-thrown, so it can never roll back the admin action that triggered it. `GET /v1/audit` returns a paginated, filterable view of the log. Every mutating admin action (disable, enable, grant-admin, revoke-admin, delete, user update) writes one.

### `health`

`GET /v1/health` is the only public (unauthenticated) route — checks both database connections and Redis.

## Environment Variables

| Variable | Required | Notes |
| --- | --- | --- |
| `PORT` | No | Default `10001` |
| `NODE_ENV` | No | Default `development` |
| `ARM_CORE_DB_HOST` / `PORT` / `NAME` / `USER` / `PASSWORD` | Yes | core-be's PostgreSQL — no migrations run against it here |
| `ARM_ADMIN_DB_HOST` / `PORT` / `NAME` / `USER` / `PASSWORD` | Yes | This service's own PostgreSQL — migrations run on startup |
| `JWT_ACCESS_SECRET` | Yes | Must be identical to `arm-core-be` and `arm-app-calendar` — this service verifies tokens core-be issued |
| `TRUST_PROXY_HEADERS` | No | Default `false`. Only set `true` when genuinely deployed behind Kong (see SEC-001 above). Leave unset in local dev |
| `REDIS_HOST` | Yes | Shared Redis instance |
| `REDIS_PORT` | No | Default `6379` |
| `REDIS_PASSWORD` | No | Set in the local dev compose; empty is allowed |
| `CORS_ORIGIN` | Yes | Admin frontend origin, e.g. `http://localhost:10000` — no trailing slash |
| `ADMIN_SEED_USER_ID` | No | A user ID from `arm_core.users` to seed as the first admin; ignored once `admin_users` is non-empty |
| `CORE_BE_INTERNAL_URL` | Yes | Internal URL of `arm-core-be`, e.g. `http://localhost:4000` |
| `BFF_INTERNAL_SECRET` | Yes | Shared secret for internal service-to-service calls |

## Known Constraints

- **`google_tokens` uses snake_case columns**, unlike every other `arm_core` table (camelCase). The `CoreGoogleToken` entity maps every column explicitly — don't remove those `name:` options, TypeORM would look for camelCase columns that don't exist.
- **`admin_role_enum` is a native PostgreSQL enum**, created in a migration. Adding a new role means a migration using `ALTER TYPE ... ADD VALUE` — TypeORM's `synchronize` cannot alter native enums.
- **The deny-list TTL (900s) is meant to track `JWT_ACCESS_TOKEN_TTL`** — if that changes in core-be, `DISABLED_TTL_SECONDS` in `deny-list.service.ts` should change with it (though see the incomplete-wiring caveat above: right now neither TTL actually gates anything in core-be or calendar-be).
- **Admin role cache (5 min) has no active invalidation.** A role change is eventually consistent, not immediate.

## Commands

```sh
pnpm install
pnpm run start:dev     # hot reload
pnpm run build
pnpm run test
pnpm run test:e2e
pnpm run lint
```

## What NOT to Do

- Don't write migrations against `arm_core`. This service reads and writes `arm_core` data but never alters its schema — all `arm_core` schema changes go through `arm-core-be`.
- Don't omit the connection name in `@InjectRepository()` — there is no default connection; every injection must say `'arm_core'` or `'arm_admin'` explicitly.
- Don't add business logic to `AuditService.log()` — it must stay fire-and-forget. Throwing from it would roll back the admin action that called it.
- Don't bypass `AdminRoleGuard`. Every data-mutating endpoint needs both `KongJwtGuard` and `AdminRoleGuard`; the health endpoint is the one intentional exception.
- Don't add a global Express namespace augmentation (`declare global { namespace Express { ... } }`) — `module: nodenext` doesn't pick those up. Define typed request interfaces locally in each guard/decorator file instead.
- Don't build a second BFF or a separate login here. Auth flows go through `arm-bff`; this service verifies JWTs, it doesn't issue or manage sessions.

For AI-agent-specific conventions and constraints (kept in sync with this guide), see [CLAUDE.md](./CLAUDE.md).
