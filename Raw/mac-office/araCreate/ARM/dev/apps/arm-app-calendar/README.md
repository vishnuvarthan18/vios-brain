# arm-app-calendar

Calendar app for the ARM platform — syncs a user's Google (and Microsoft) calendars into "blocker" events on a target calendar, so busy time from one account shows up on another. One repo containing two independently-built apps: a NestJS backend and a React/Vite frontend.

- Backend (`src/backend/`) manages Google/Microsoft OAuth, calendar sync configs, and the sync engine (initial sync, webhook-driven delta updates, poll fallback)
- Frontend (`src/frontend/`) is a Module Federation **remote** — loaded by the `arm-core-fe` shell at runtime, not accessed directly
- Real-time sync status pushed to the frontend over SSE

## Stack

| Layer | Tool |
| --- | --- |
| Backend framework | NestJS |
| Frontend framework | React 19 + Vite, Module Federation remote |
| Database | MongoDB (Mongoose) |
| Queue | BullMQ (Redis) |
| Calendar APIs | Google Calendar API (`googleapis`), Microsoft Graph |
| Language | TypeScript |
| Package manager | pnpm |
| Tests | Jest (unit, integration via Testcontainers, e2e via Supertest) |

## Architecture

```text
arm-core-fe (shell)
        │  loads CalendarModule remote
        ▼
src/frontend (Vite/Module Federation) ──/bff/proxy/calendar/v1──▶ arm-bff ──▶ src/backend (NestJS)
                                                                                    │
                                                        ┌───────────────────────────┼───────────────────────────┐
                                                        ▼                           ▼                           ▼
                                                  MongoDB (sync state,        BullMQ/Redis            Google/Microsoft Calendar APIs
                                                  accounts, events)           (sync jobs, webhook       (OAuth, events, push
                                                                               processing)               notification channels)
```

- `src/backend/src/account/` — Google/Microsoft account connection per user
- `src/backend/src/auth/` — JWT auth guard/strategy
- `src/backend/src/calendar/`, `calendar-management/` — Calendar listing, watch, CRUD
- `src/backend/src/events/` — the sync engine (lifecycle, orchestrator, webhook processor, query, blocker, DLQ)
- `src/backend/src/google/`, `microsoft/` — provider API clients, mirrored OAuth flows
- `src/backend/src/webhook/` — Google push notification handler + renewal cron
- `src/frontend/src/features/calendar/` — sync UI (account page, sync view/detail, hooks for SSE and active syncs)

See the workspace-level [CLAUDE.md](../../CLAUDE.md) for how this app fits into the platform (BFF, core-be), and this repo's own [CLAUDE.md](./CLAUDE.md) for calendar-specific architecture detail (MongoDB collections, sync flow, webhook lifecycle).

## Commands

Each package has its own scripts today (`src/backend/package.json`, `src/frontend/package.json`) — a root `Makefile` with the standard targets is tracked as TPL-003 in `TASK_SHEET.md`.

In the full ARM workspace, this app normally runs via `make local-dev` from `deploy/arm-deploy-make/` — see the workspace [CLAUDE.md](../../CLAUDE.md) for the standard startup flow.

## Conventions

Repo conventions (file headers, naming, versioning, Makefile targets) follow `aracreate-template-codebase`. Template-conformance work for this repo is tracked in `TASK_SHEET.md`.

## License

Proprietary — see [LICENSE](./LICENSE), Copyright (C) 2026, araCreate Group.
