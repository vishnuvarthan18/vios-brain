# SOURCE

The backend's own source — a NestJS service, no defined layout in the araCreate template (Node/NestJS isn't one of the covered project types), so this follows Nest's own module conventions instead:

```
src/
├── main.ts                  # Nest app bootstrap — global prefix, helmet, validation pipe, CORS
├── app.module.ts             # Root module
├── account/                  # Google/Microsoft account connection per user
├── auth/                     # JWT auth guard + strategy
├── calendar/                  # Google Calendar list, watch, incremental list
├── calendar-management/       # Calendar CRUD exposed via REST
├── common/                    # Cross-cutting: env config, core-auth client, token crypto, circuit breaker, Redis
├── events/                    # The sync engine — lifecycle, orchestrator, webhook processor, query, blocker, DLQ, processors, repositories
├── google/                    # googleapis client wrapper, token refresh
├── microsoft/                 # Microsoft Calendar OAuth + sync, mirrors the Google path
├── migrations/                 # Startup migrations, run once via StartupMigrationService
├── providers/                  # Calendar-provider abstraction (interface + registry) shared by Google/Microsoft
├── schemas/                    # Mongoose schemas
└── webhook/                    # Google push notification handler + renewal cron
```

No third-party code is vendored here. See this app's own [CLAUDE.md](../../../CLAUDE.md) for the full architecture writeup.
