# arm-app-calendar (calendar-be) — API Reference

Every HTTP endpoint calendar-be exposes. Base path is `http://localhost:4001/v1` in local dev (global prefix `v1`, `main.ts`), normally reached through the session service proxy (`/session/proxy/calendar/v1/...`), not called directly by a browser — see [SESSION-FLOW.md](../SESSION-FLOW.md).

**No OpenAPI/Swagger spec exists for this service** — confirmed, no `@nestjs/swagger` dependency (Risk 10 in [`meta/documentation-inventory.md`](../meta/documentation-inventory.md), still open for this repo). This document is hand-verified against every controller in `src/` (`account`, `app`, `auth`, `calendar`, `calendar-management`, `events`, `google`, `health`, `webhook` — 2026-08-12) and is the closest thing to a spec this service has.

Endpoints already fully documented elsewhere are **linked, not repeated** here — this doc's job is the complete surface map plus what nothing else covers (calendar/event CRUD, calendar-management, sync-history/active-syncs, webhook renewal admin route, health, and the platform-wide error-shape inconsistency in §9).

---

## 1. Conventions

- **Auth**: every controller except `webhook.controller.ts`'s intake route (§6) and `app.controller.ts`'s root route requires `AuthGuard` — dual path, same pattern as every other backend: trusts Kong's `X-User-Id` header only when `TRUST_PROXY_HEADERS=true`, otherwise verifies the JWT directly (with a one-shot refresh-and-retry via `POST {CORE_API_URL}/v1/auth/refresh` on an expired token). `JWT_ACCESS_SECRET` must match `arm-core-be`'s.
- **Validation**: global `ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true })` — an unrecognized body field is rejected outright (`403`... actually `400 Bad Request`, "property X should not exist"), not silently dropped.
- **Idempotency**: `IdempotencyInterceptor`, applied per-route via `@UseInterceptors`, not globally — caches a response for 24h keyed on the `Idempotency-Key` header (when present) so a client retry replays the cached result instead of re-running the handler. Applied to `POST /account/connect` and `POST /events/:targetCalendarId/sync`. It has to specifically detect and skip caching "logical failures" — see §9, this is directly caused by the error-shape inconsistency documented there.
- **CORS**: `FRONTEND_URL`/`CORE_FRONTEND_URL`/`(secret removed)`/`(secret removed)`, credentials enabled.
- **`/metrics`** is restricted to internal/private IP ranges at the Express layer, ahead of any NestJS guard — a Prometheus-only route, not part of the API surface below.

---

## 2. Account endpoints (`/v1/account/*`)

Fully documented in [`LINKED-ACCOUNTS.md`](LINKED-ACCOUNTS.md) §4 — the complete 10-route table, request/response shapes, `toPublicAccount()` field stripping, and which routes the current UI actually calls vs. which are live-but-unreached. Not repeated here.

---

## 3. Calendar endpoints (`/v1/calendar/*`)

Direct CRUD against the resolved provider's calendar API (Google or Microsoft, per the account). All routes: `AuthGuard`.

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/calendar/events` | List events. Query: `calendarId` (default `primary`), `timeMin`, `timeMax`, `maxResults` (default 2500) |
| `GET` | `/calendar/events/:eventId` | Get one event. Query: `calendarId` |
| `POST` | `/calendar/events` | Create an event. Body: `CreateEventDto`. Query: `calendarId` |
| `PUT` | `/calendar/events/:eventId` | Update an event. Body: `UpdateEventDto`. Query: `calendarId` |
| `DELETE` | `/calendar/events/:eventId` | Delete an event. Query: `calendarId` |
| `GET` | `/calendar/calendars` | List every calendar across all of the caller's accounts |
| `GET` | `/calendar/calendars/by-account/:accountId` | List calendars for one specific account |

Every `events/*` route resolves which account's provider client to use via `calendarService.resolveAccountIdForUser(userId, calendarId)` — `calendarId` is optional on reads but effectively required for `resolveAccountIdForUser` to pick the right account when a user has more than one connected; omitting it falls back to whatever the service's own default-account resolution does.

---

## 4. Calendar management endpoints

**No `@Controller()` prefix on this controller** — routes hang directly off the global `v1` prefix, not `/v1/calendar-management/*` as the doc's original outline assumed. `AuthGuard` on both routes.

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/account/all-with-calendars` | Every one of the caller's accounts, each with its calendars attached — the route the current frontend actually uses |
| `GET` | `/v1/:accountId` | **Deprecated** — grouped `{ target, source }` calendars for one account. The controller's own comment: *"Kept for backward compatibility until all consumers migrate. Frontend has fully migrated — safe to remove once confirmed no external callers remain."* |

**Worth knowing**: `GET /v1/:accountId` is a bare top-level catch-all — any single-segment path under `/v1/` that doesn't match a more specific controller's route falls through to this handler, since NestJS matches routes in module-registration order and this one has no distinguishing prefix. It hasn't caused a collision (verified: no other controller registers an unprefixed single-segment GET), but it's a landmine for whoever adds one next, worth removing this deprecated route sooner rather than later per its own comment rather than leaving it as a compatibility trap.

---

## 5. Event / sync endpoints (`/v1/events/*`)

The sync engine's REST surface — mechanics (what a sync actually does, blocker events, BullMQ jobs, delta processing) are [`SYNC-FLOW.md`](SYNC-FLOW.md)'s subject; this is the endpoint inventory. `AuthGuard` on every route except the webhook intake route (§6, same controller).

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/events/:targetCalendarId/sync` | Start an initial sync. Body: `EventSyncDto { source: Record<string, string[]>, syncFromDate: string, name?: string, showTitle?: boolean, showDescription?: boolean, showLocation?: boolean }` — the last four fields are optional and persist to `SyncGroupMeta` (all toggles default `true` if omitted). Idempotency-protected. Returns `202` + `jobId` |
| `POST` | `/events/:targetCalendarId/sync/update` | Update an existing sync's source-calendar mapping, and/or its `name`/toggles (same `EventSyncDto` body) — persists the name/toggle fields even when the calendar mapping itself is unchanged |
| `POST` | `/events/:targetCalendarId/sync/resync` | Re-run initial sync against this target's already-configured sources, no body. Diffs and updates every already-synced blocker's content, not just new events — see [`SYNC-FLOW.md`](SYNC-FLOW.md) §10. Returns `202` + `jobIds` (one per distinct `syncFromDate` among the target's sources); `404` if no active sync exists, `403` if the caller doesn't own the target account, `409` if a sync is already running for this target |
| `POST` | `/events/:accountId/sync/stop` | Stop sync for an account, audit-logged (`sync.stop`) |
| `DELETE` | `/events/sync/:targetCalendarId` | Unsync a whole target calendar — queues background blocker cleanup, returns `202` + `jobId` |
| `DELETE` | `/events/source/:eventId` | Delete one source event and its corresponding blocker. Query: `calendarId` (required) |
| `GET` | `/events/preview/calendar/:accountId` | Preview `{ target, sources }` calendars before starting a sync |
| `GET` | `/events/active-syncs` | List the caller's currently-active syncs |
| `GET` | `/events/active-syncs-with-history` | Same, each with its sync history attached |
| `GET` | `/events/sync-history/:targetCalendarId` | Full sync-history list for one target calendar |
| `GET` | `/events/sync/status/:jobId` | BullMQ job status by ID — `@SkipThrottle()`, since the unsync flow's frontend polls this every 2s for up to 5 minutes during teardown, a rate that would otherwise exceed the platform's default throttle bucket (documented in-code as `BUG-007`) |
| `GET` | `/events/:accountId` | Paginated sync config / events for an account. Query: `page`, `limit` |
| `SSE` | `/events/stream` | Server-Sent Events stream — `CALENDAR_UPDATED_EVENT` pushed after a successful sync, consumed by `core-fe`/`calendar-fe`'s `use-calendar-sse.ts` |
| `POST` | `/events/notifications` | Google webhook push-notification intake — **no `AuthGuard`**, `@SkipThrottle()`, enqueues raw headers to a BullMQ queue and returns `200` immediately, real validation happens async. Full detail: [`WEBHOOKS.md`](WEBHOOKS.md) §2–§4 |

---

## 6. Webhook admin endpoint (`/v1/webhook/*`)

Distinct from the intake route above (different controller, different file: `webhook.controller.ts` vs. `events.controller.ts`).

| Method | Path | Guard | Purpose |
|---|---|---|---|
| `POST` | `/webhook/renew-all` | `InternalSecretGuard` | Force-renews every webhook channel platform-wide — `WebhookRenewalService.renewAllWebhooks()`, the same logic the daily cron in [`WEBHOOKS.md`](WEBHOOKS.md) runs automatically |

This route renews every channel for every tenant, not just the caller's own data, so an ordinary session guard is no longer used here. `InternalSecretGuard` requires the caller to present the `x-session-secret` header matching `SESSION_INTERNAL_SECRET`; a request without it (or with the wrong value) gets a bare `403 Internal endpoint`, whatever the caller's own session looks like. This also means a browser session cannot reach the route at all — the session service proxy strips inbound `x-session-secret` headers, so calling it requires a direct, non-proxied request with the secret attached (e.g. `curl`, or another backend service).

---

## 7. Auth / OAuth endpoints (`/v1/auth/*`, `/v1/google/*`)

Fully documented in [`GOOGLE-OAUTH.md`](GOOGLE-OAUTH.md) — not repeated here. Quick map of which controller owns which piece:

| Flow | Controller | Doc section |
|---|---|---|
| Add Account (Google + Microsoft) | `auth.controller.ts` — `/auth/google/add-account`, `/auth/google/add-account/callback`, `/auth/microsoft/add-account`, `/auth/microsoft/add-account/callback` | GOOGLE-OAUTH.md §2–§7, §10 (Microsoft twin) |
| Reconnect (Google) | `google.controller.ts` — `/google/update/account/:email`, `/google/update/account/callback` | GOOGLE-OAUTH.md §8 |
| Legacy auth-URL helper | `auth.controller.ts` — `GET /auth/google/url` (returns `{ authUrl: '{CORE_API_URL}/v1/auth/google' }`, a pointer to core-be's own OAuth entry point, not a calendar-be-issued URL itself) | Not separately documented — a thin redirect-URL builder, no state of its own |

---

## 8. Health endpoints (`/v1/health/*`)

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/health/live` | Process-alive only, no dependency checks — by design, so a Mongo/Redis outage never causes the orchestrator to kill and restart an otherwise-healthy process |
| `GET` | `/health/ready` | Pings MongoDB (`db.admin().ping()`, not just `readyState`, since that can lag mid-reconnect) and Redis (`PING` → `PONG`) in parallel via `Promise.allSettled`. Either failing throws `503` with `{ status: 'down', details: { mongo, redis } }` |
| `GET` | `/` | `app.controller.ts` — bare `getHello()` scaffold string, not a real health signal (unauthenticated, unguarded) |

This is the platform's cleanest live/ready split of the 5 backends — see [MONITORING.md](../MONITORING.md) §8 for the cross-service comparison (`core-be`'s equivalent has no genuine ready-vs-live separation).

---

## 9. Error response shapes — genuinely inconsistent, verified

**There is no global exception filter in this service** (unlike `core-be`'s `AllExceptionsFilter` — see [`core-be/API.md`](../core-be/API.md) §6). Two entirely different error patterns coexist, and which one a given route uses is not predictable from the outside without reading its source:

1. **Thrown exceptions** (most `calendar.controller.ts` and `events.controller.ts` routes, anything that doesn't wrap its body in a try/catch) — NestJS's default exception handling takes over: a real, correct HTTP status code (`404`, `400`, `500`, etc.) with NestJS's own default body shape `{ statusCode, message, error }`.
2. **Caught-and-returned** (most of `account.controller.ts`, some of `events.controller.ts`) — the handler catches its own error and `return`s a plain object like `{ statusCode: HttpStatus.BAD_REQUEST, success: false, message }`. **The real HTTP response status is whatever NestJS's default is for that route's verb and any `@HttpCode()` override (200 for a plain `@Get`/`@Delete`, 201 for a plain `@Post` unless overridden) — not the `statusCode` field in the body.** A client trusting `response.status` (the real HTTP code) instead of `response.data.statusCode` (the JSON field) will see a "successful" `200`/`201` response carrying a failure payload. Concrete example: `AccountController.connectAccount` (`POST /account/connect`, no `@HttpCode` override) returns `{ statusCode: HttpStatus.BAD_REQUEST, success: false, message }` from its catch block on failure — the actual response NestJS sends is `201 Created`.

**This isn't a guess** — `common/idempotency/idempotency.interceptor.ts`'s own code comment names this exact pattern and had to build a dedicated `isLogicalFailure()` check (`success === false` or `statusCode >= 400` in the *body*) specifically so a caught-and-returned failure isn't mistaken for a cacheable success and replayed for 24 hours on retry. The workaround is real and already shipped; the underlying inconsistency it works around is not.

**Practical guidance for any new API consumer**: check `response.data.success`/`response.data.statusCode` on every calendar-be call, don't rely on the real HTTP status code alone — that's the one true platform-wide rule this service's own inconsistency forces on every caller.

---

## Related documents

- [`LINKED-ACCOUNTS.md`](LINKED-ACCOUNTS.md) — full account-endpoint detail (§2 above)
- [`docs/arm-docs/docs/calendar/GOOGLE-OAUTH.md`](GOOGLE-OAUTH.md) — full OAuth-endpoint detail (§7 above)
- [`docs/arm-docs/docs/calendar/WEBHOOKS.md`](WEBHOOKS.md) — webhook intake route and renewal cron mechanics (§5, §6 above)
- [`docs/arm-docs/docs/calendar/SYNC-FLOW.md`](SYNC-FLOW.md) — what a sync job actually does once `/events/:targetCalendarId/sync` returns
- [`docs/arm-docs/docs/calendar/DELETE-CASCADE.md`](DELETE-CASCADE.md) — `DELETE /account/:accountId`'s full cascade
- [`docs/arm-docs/docs/core-be/API.md`](../core-be/API.md) — the sibling reference for `core-be`, including its own (different) error-shape convention
- [`docs/arm-docs/docs/MONITORING.md`](../MONITORING.md) §8 — health-endpoint comparison across all 5 backends
- [`docs/arm-docs/docs/meta/documentation-inventory.md`](../meta/documentation-inventory.md) — Risk 10, no OpenAPI spec for this service
