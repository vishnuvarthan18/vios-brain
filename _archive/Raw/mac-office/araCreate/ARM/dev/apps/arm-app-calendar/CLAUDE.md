# Calendar App — Claude Context

> Service-level context for `apps/arm-app-calendar/`. Read the workspace-level [CLAUDE.md](../../CLAUDE.md) first for platform-wide rules.

---

## What this repo is

One repo containing two independently-built apps: a NestJS backend (`apps/backend/`) and a React/Vite frontend (`apps/frontend/`). The backend manages Google Calendar sync and event blocking. The frontend is a Module Federation **remote** loaded by the core shell at runtime.

---

## Internal Structure

```
apps/arm-app-calendar/
├── apps/
│   ├── backend/src/          ← NestJS backend
│   │   ├── account/          ← Google account per user (isAuthenticated, token storage)
│   │   ├── auth/             ← JWT auth guard, JWT strategy
│   │   ├── calendar/         ← Google Calendar list, watch, incremental list
│   │   ├── calendar-management/ ← Calendar CRUD exposed via REST
│   │   ├── config/           ← env.ts — plain module (not injectable, see TASK-028)
│   │   ├── core-auth/        ← Fetches Google tokens from arm-core-be via BFF
│   │   ├── crypto/           ← AES-256 token encryption (GOOGLE_TOKEN_ENCRYPTION_KEY)
│   │   ├── events/           ← Main sync engine: initial sync, webhook, poll, blockers
│   │   ├── google/           ← Wraps googleapis client
│   │   ├── libs/             ← circuit-breaker.ts (exists but underused — see below)
│   │   ├── migrations/       ← Runs once on startup via StartupMigrationService
│   │   ├── schemas/          ← Mongoose schemas (5 collections)
│   │   └── webhook/          ← Google push notification handler + renewal cron
│   └── frontend/src/
│       ├── config/           ← API base URL and env vars
│       ├── features/
│       │   └── calendar/
│       │       ├── components/   ← calendar-account-page, sync-view-page, sync-detail-page
│       │       ├── components/ui/← Dialog/modal components
│       │       ├── hooks/        ← use-calendar-sync, use-active-syncs, use-calendar-sse
│       │       ├── pages/        ← auth-callback.tsx (Google OAuth popup return page)
│       │       ├── services/     ← api.ts — all fetch calls to the BFF proxy
│       │       └── types/        ← TypeScript types for calendar domain
│       └── services/             ← core-auth.service.ts — reads BFF session
```

---

## MongoDB Collections

| Collection | Schema file | Purpose |
|---|---|---|
| `accounts` | `account.schema.ts` | One doc per user's Google connection. Stores encrypted tokens, `isAuthenticated` flag, `tokenExpiryAt`. |
| `calendars` | `calendar.schema.ts` | Google Calendar entries per account. Stores `channelId`, `resourceId`, `expiration` for webhook tracking. |
| `events` | `event.schema.ts` | Synced/blocker events. `start`/`end` are `any` typed — see TASK-006. |
| `syncconfigs` | `sync-config.schema.ts` | Maps source calendars → target calendar for a user. The core data structure for a sync. |
| `synchistories` | `sync-history.schema.ts` | Audit log of sync runs. `SyncStatus.PENDING` is defined but never written before a job starts — see TASK-044. |

---

## How Google Integration Works

```
User authenticates with Google in arm-core-be (OAuth)
        ↓
arm-core-be stores encrypted Google tokens

Calendar-be needs to call Google:
        ↓
CoreAuthService.getGoogleToken(userId)
→ calls core-be directly: GET {CORE_API_URL}/v1/auth/google/token/{userId}
→ includes x-bff-secret header (BFF_INTERNAL_SECRET) for inter-service auth
→ returns { accessToken, expiresAt, scope }
        ↓
TokenEncryptionService decrypts/encrypts for local storage in Account
        ↓
GoogleService.getClient(account) creates authenticated googleapis client
        ↓
CalendarService / EventsService makes the actual Google API calls
```

**Calendar-be has two separate token paths for its own `Account` documents.** The diagram above (`CoreAuthService.getGoogleToken`) is used only when reading Account data provisioned via `syncFromCoreAuth()` — that path's `Account.refreshToken` is almost never populated (TASK-037). The "Add Account" flow (both frontends' "+ Add Account" button) is a separate, calendar-be-native OAuth flow (`GoogleOAuthService`/`MicrosoftOAuthService`, wired through `auth.controller.ts`'s `/auth/{google,microsoft}/add-account` routes) — it does its own consent redirect and code exchange, and does populate `refreshToken` directly from the token response. Do not confuse the two paths; TASK-037 does not affect accounts created via "Add Account".

---

## Sync Architecture

### What a sync does
A **SyncConfig** maps one user's source calendars (Google Calendars with real events) to a target calendar (where "blocker" events are created). Blocker events have `[ BLOCKER ]` as a title prefix and appear in the target calendar to show the user as busy.

### Sync flow
1. **Initial sync** — `queueInitialSync()` enqueues a BullMQ job. Worker runs `performInitialSync()`: lists all Google events → creates blocker events in target → marks sync complete.
2. **Webhook notifications** — Google sends a push to `/bff/proxy/calendar/v1/webhooks/...` when any event changes. `EventsService.handleWebhook()` → `applyDeltaEvents()` processes the delta.
3. **Poll fallback** — `PollService.pollCalendar()` runs on a schedule when webhook is unavailable.
4. **SSE updates** — After a successful sync, `CALENDAR_UPDATED_EVENT` is emitted via SSE to the frontend (`/bff/proxy/calendar/v1/events/stream`). Frontend hook `use-calendar-sse.ts` listens and triggers a refresh.

### Google webhook channels
- Expire every ~7 days. Renewed by a daily cron in `webhook-renewal.service.ts` (checks 48hr window).
- If renewal fails, the dead channel stays in the DB with no polling fallback — TASK-045.
- `expiration` field is stored as `string` but compared numerically in renewal query — TASK-048.

---

## Frontend API Pattern

All API calls go through the BFF proxy. Base URL is `/bff/proxy/calendar/v1`.

```typescript
// frontend/src/config/ — base URL
const API_BASE = '/bff/proxy/calendar/v1';

// example call
fetch(`${API_BASE}/accounts`)          // → BFF → calendar-be /v1/accounts
fetch(`${API_BASE}/events/stream`)     // → SSE stream
```

**Auth** — the frontend reads its session from `/bff/me` on mount. It does not have its own auth — it reuses the session managed by the core shell and arm-bff.

---

## Module Federation

- **Exposes:** `CalendarModule` — the entire calendar feature as one lazy-loaded entry
- **Host:** `arm-core-fe` loads this remote at runtime
- **CSS rule:** When `SyncView` is embedded in the core shell, it must be inside a `.calendar-scope` div. FullCalendar's CSS resets break the shell layout without it.
- **Port:** Runs on `:3001` in local dev. Not accessed directly — the shell loads it.

---

## Key Architecture Constraints

### `circuit-breaker.ts` is underused
`libs/circuit-breaker.ts` is implemented and wired to `CoreAuthService`. It is **not wired to `GoogleService`**. All Google API calls in `events.service.ts` (the 1,598-line loop-heavy sync engine) are unprotected — TASK-013.

### `EventsService` is a monolith
`backend/src/events/events.service.ts` is 1,598 lines handling initial sync, webhook processing, poll-based sync, blocker creation/deletion, and lifecycle management. TASK-010 plans to split it into four focused services. **Fix Phase 1 bugs before touching this file.**

### Token refresh is broken for most accounts
Accounts created via `syncFromCoreAuth()` never have a `refreshToken` stored. When the access token expires (~1 hour), there is no auto-refresh — all subsequent Google API calls fail permanently until the user manually re-authenticates. TASK-037.

### `isAuthenticated` is not self-healing
When a Google API call fails with 401 or 403, `isAuthenticated` is NOT set to `false`. The account stays marked as healthy and continues making failing API calls. TASK-038.

---

## Known Critical Bugs (fix before new features)

All in Phase 1 of `TASK_SHEET.md`. Summary:

| Task | Location | Issue |
|---|---|---|
| ~~TASK-001~~ | ~~`crypto/token-encryption.service.ts:40`~~ | ~~Decryption failure returns plaintext silently~~ — ✅ Fixed |
| ~~TASK-002~~ | ~~`events/events.service.ts:271`~~ | ~~Sync cancellation checked too infrequently — orphaned blockers~~ — ✅ Fixed |
| ~~TASK-003~~ | ~~`events/events.service.ts:625`~~ | ~~SyncConfig deleted before Google cleanup — permanent orphaned events~~ — ✅ Fixed |
| ~~TASK-004~~ | ~~`events/events.service.ts:1319`~~ | ~~`Promise.allSettled` swallows sync failures — silent data loss~~ — ✅ Fixed |
| ~~TASK-035~~ | ~~`core-auth/core-auth.service.ts:26`~~ | ~~Google OAuth scope never validated — calendar-less accounts appear authenticated~~ — ✅ Fixed |
| ~~TASK-036~~ | ~~`google/google.service.ts:75`~~ | ~~Token expiry never checked — stale tokens sent to Google~~ — ✅ Fixed |
| ~~TASK-037~~ | ~~`account/account.service.ts:171`~~ | ~~Refresh token never stored in syncFromCoreAuth path~~ — ✅ Fixed |
| ~~TASK-038~~ | ~~`calendar/calendar.service.ts:349`~~ | ~~401/403 from Google doesn't flip `isAuthenticated` to false~~ — ✅ Fixed |
| ~~TASK-039~~ | ~~`account/account.service.ts:136`~~ | ~~Token fetch failure wrongly sets `isAuthenticated: true`~~ — ✅ Fixed |
| ~~TASK-040~~ | ~~`events/events.service.ts:1487`~~ | ~~SSE only fires for first target account in batch~~ — ✅ Fixed |

---

## Environment Variables (calendar-be specific)

- `GOOGLE_TOKEN_ENCRYPTION_KEY` — AES-256 key, **must be exactly 64 hex characters**. `openssl rand -hex 32` gives exactly this.
- `JWT_ACCESS_SECRET` / `JWT_REFRESH_SECRET` — must be **identical** to `arm-core-be`. Shared token format.
- `MONGODB_URI` — must embed the same password as `MONGO_ROOT_PASSWORD` in `.env`.

---

## What NOT to Do

- Do not add a *third* calendar-account OAuth path. Two exist already: the calendar-be-native "Add Account" flow (`GoogleOAuthService`/`MicrosoftOAuthService`) and the older `CoreAuthService`/`syncFromCoreAuth()` proxy path. New providers should follow the "Add Account" pattern (see `google-oauth.service.ts` and `microsoft-oauth.service.ts`), not invent a third mechanism.
- Do not expose tokens to the frontend in any API response.
- Do not add new features to `EventsService` until Phase 1 bugs are fixed — the file is fragile and has known data-loss paths.
- Do not route calendar-be → core-be calls through the BFF proxy. The BFF proxy requires a browser `SESSION_ID` cookie. East-west calls go direct with `x-bff-secret` header.
- Do not use `console.error` — use the NestJS `Logger` (TASK-014).
- Do not add custom HTML/divs for UI that the `@aracreate/test-arm-ui` library already covers (TASK-049 through TASK-057).
