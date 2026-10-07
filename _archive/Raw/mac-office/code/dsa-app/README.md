# DreamSpace Academy — Platform Monorepo

Platform for a Sri Lankan social enterprise running makerspace hubs for
underserved youth. Phase 1 = the Makerspace module. See `CLAUDE.md` for the full
architecture and rules, and `docs/` for sprint plans and references.

## Workspaces (pnpm)

| Package   | Stack                     | Status (Sprint 1)    |
| --------- | ------------------------- | -------------------- |
| `api/`    | Express + Prisma + TS     | **Built & verified** |
| `web/`    | Vite + React + TS         | Scaffold             |
| `mobile/` | Capacitor (iOS + Android) | Scaffold             |
| `public/` | Next.js                   | Scaffold             |

## Quick start

```bash
corepack enable && corepack prepare pnpm@10 --activate   # if pnpm is missing
pnpm install

# --- API ---
cp api/.env.example api/.env        # local-first defaults work as-is (AUTH_MODE=mock)
pnpm --filter api exec prisma generate
# Point DATABASE_URL at any Postgres, then:
pnpm --filter api exec prisma migrate dev --name init
#   run once after first migrate:
#   psql "$DATABASE_URL" -f api/prisma/sql/sequences.sql
#   psql "$DATABASE_URL" -f api/prisma/sql/hardening.sql   (see file header)
pnpm --filter api run db:seed
pnpm --filter api dev               # http://localhost:4000  (GET /health)
```

## Verify the backend (no external services / no Docker needed)

```bash
pnpm --filter api run typecheck                         # 0 errors
pnpm --filter api exec tsx --test src/permissions.test.ts   # 9/9 permission tests
pnpm --filter api exec tsx scripts/smoke.ts             # full e2e (embedded Postgres)
```

The smoke harness boots a real throwaway Postgres, pushes the schema, seeds, and
drives the API (auth, GRANT/DENY overrides, hub scope, soft delete, validation,
audit insert-only) — printing `CHECK <id> PASS/FAIL`. Last run: **15/15 PASS**.

## Sprint 1 — what the API does

Auth (Auth0 staff, provider-swappable mock for local dev; parent phone OTP),
users + roles, hubs, and the central permission engine
(`api/src/permissions.ts`) with three-layer resolution **DENY > GRANT > role
baseline**, hub-scope enforcement, Zod validation, and an INSERT-only audit log.
Routes follow `docs/API_ROUTES.md` under `/api/v1`.
