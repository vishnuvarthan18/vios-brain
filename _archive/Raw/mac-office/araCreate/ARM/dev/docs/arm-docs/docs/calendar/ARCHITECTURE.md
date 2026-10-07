# Calendar App — Architecture Overview

`apps/arm-app-calendar` is one repo containing two independently-built apps: a NestJS backend (`src/backend/`, port `4001`) and a React/Vite frontend (`src/frontend/`, port `3001`, a Module Federation remote loaded by the shell). The backend syncs events between Google and Microsoft calendars by creating a "blocker" event in a target calendar for every event on a source calendar, keeps them in sync via webhook (Google) or poll (Microsoft, and Google as a fallback), and exposes the whole thing to the frontend through the session service proxy. A blocker's title/description/location are resolved per-sync from field-visibility toggles (default: real content) — not the fixed `'[ BLOCKER ]'` placeholder older versions of this doc described; see [SYNC-FLOW.md](SYNC-FLOW.md) §10.

This document covers the backend's module structure and the sync/webhook/token mechanics that tie them together. It does not re-document what already has a dedicated doc — see [Related documents](#related-documents) for OAuth flow, delete-cascade, schema field tables, MF wiring, and auth guard detail.

Every fact below was verified directly against `src/backend/src/` and `src/frontend/src/` source, not taken from `CLAUDE.md` at face value — §11 lists the drift found along the way.

---

## 1. Backend module map

| Module | Responsibility |
|---|---|
| `account/` | Account lifecycle. `AccountConnectionService` (431 lines) handles `syncFromCoreAuth()` (pull a token core-be already holds from SSO login) and full account deletion (cascades to calendars/events/sync-config/sync-history — see [DELETE-CASCADE.md](DELETE-CASCADE.md)). `AccountTokenService` and `AccountService` (facade) round it out. |
| `audit/` | `AuditLogService` — thin, fire-and-forget append-only log for exactly two privileged actions (`account.delete`, `sync.stop`), a closed TS union, not free text. Failures are logged and swallowed, never thrown. |
| `auth/` | `AuthGuard` — the dual `X-User-Id`/JWT pattern shared with every other ARM backend; see [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §3 for the cross-service comparison. Also `PerUserThrottlerGuard` and the Google/Microsoft add-account OAuth routes (`AuthController`). |
| `calendar/` | Provider-specific REST wrappers — `google-calendar-api.service.ts` (490 lines), `microsoft-calendar-api.service.ts` (451 lines), `microsoft-recurrence-mapper.ts` (218 lines, rrule↔Graph recurrence translation). `CalendarService` is a thin facade that resolves a `CalendarProvider` via the registry (§2) and delegates. |
| `calendar-management/` | `CalendarManagementService` merges a user's accounts + calendars into `{target, source}` groupings for the frontend's account picker, falling back to cached DB calendars and flipping `isAuthenticated` on failure. |
| `common/circuit-breaker/` | Generic closed→open→half-open state machine. Not a NestJS-injected singleton — each consumer instantiates its own (§9). |
| `common/config/` | `env.ts` — fail-fast typed env accessor; `requireWebhookUrl` validates `NOTIFICATION_WEBHOOK_URL` is HTTPS or a known-internal host. |
| `common/core-auth/` | `CoreAuthService` — fetches/revokes Google tokens from arm-core-be over `fetch()`, wrapped in a `core-be`-named circuit breaker. |
| `common/crypto/` | `TokenEncryptionService` — (secret removed) token-at-rest encryption (§5). |
| `common/idempotency/` | `IdempotencyInterceptor` — Redis-cached response replay keyed on an `Idempotency-Key` header, 24h TTL. Opt-in, used only on the sync-creation endpoint. |
| `common/observability/` | Event-loop lag monitoring, graceful OTel shutdown hook. |
| `common/redis/` | `SseRedisService` — per-user SSE pub/sub over a dedicated Redis connection; shared `REDIS_CLIENT` token used across BullMQ, idempotency, health, and SSE. |
| `events/` | The sync engine. See §3–§4. |
| `google/` | `GoogleClientFactory` (token refresh + circuit breaker), `GoogleOAuthService` (add-account/reconnect OAuth — see [GOOGLE-OAUTH.md](GOOGLE-OAUTH.md)), `GoogleService` (facade). |
| `health/` | Liveness vs. readiness split — see §8. |
| `microsoft/` | `MicrosoftClientFactory` (token refresh + circuit breaker), `MicrosoftOAuthService` (add-account code exchange, two circuit breakers), `microsoft.service.ts`. |
| `migrations/` | `StartupMigrationService` — two idempotent backfills run once at boot: `Account.provider` default, and `Event.accountId` (joined via `Calendar`). |
| `providers/` | `CalendarProvider` interface + `CalendarProviderRegistry`. See §2. |
| `webhook/` | **Only** the Google webhook-channel renewal cron/worker and an admin force-renew controller (§4). The inbound push-notification *handler* lives in `events/`, not here — see §11.2. |

---

## 2. The `CalendarProvider` abstraction

`providers/calendar-provider.interface.ts` defines `CalendarProvider`, implemented identically by `GoogleCalendarProvider` and `MicrosoftCalendarProvider` (81 lines each, thin 1:1 delegation to the corresponding `*-calendar-api.service.ts`):

```typescript
interface CalendarProvider {
  readonly name: string;
  listCalendarsByAccountId(accountId): Promise<Calendar[]>;
  listEvents(...): Promise<Event[]>;
  listEventsIncremental(...): Promise<...>;
  getEvent(...) / createEvent(...) / updateEvent(...) / deleteEvent(...);
  listBlockerEvents(...): Promise<Event[]>;
  watchEvents(...): Promise<...>;   // register a push-notification channel
  stopChannel(...): Promise<void>;
}
```

A separate `CalendarWebhookProvider` interface (`parseNotification`/`createSubscription`/`stopSubscription`) is implemented **only by Google** (`providers/google/google-calendar-webhook.provider.ts`) — there is no Microsoft equivalent file at all, because Microsoft Graph subscriptions aren't wired in (below).

`CalendarProviderRegistry.forAccount(accountId)` looks up `Account.provider` in Mongo, caches the resolved **provider name** (not the instance) in an in-memory `Map` with a 5-minute TTL, defaults to `'google'` if unset, then returns the matching singleton provider.

**Microsoft is CRUD-only — by design, not by gap.** `MicrosoftCalendarApiService.watchEvents()`/`stopChannel()` (`calendar/microsoft-calendar-api.service.ts:393-406`) unconditionally `throw new ForbiddenException('Microsoft push notifications are not yet supported')`. `SyncOrchestratorService` catches exactly this (`error instanceof ForbiddenException || message.includes('403')`) at two call sites — source-calendar initial sync (line ~528) and target-calendar tamper-restoration setup (line ~599) — and registers a 5-minute poll job (`pollQueue.upsertJobScheduler`) instead. Since `watchEvents` always throws for Microsoft, **every** Microsoft-connected calendar takes this poll path by construction; Google takes it only when a real webhook registration fails. A source calendar that already has an active webhook channel (`Calendar.channelId` set) skips this whole registration attempt entirely, Microsoft or Google — added specifically so re-running initial sync via Resync ([SYNC-FLOW.md](SYNC-FLOW.md) §10) doesn't leak a duplicate channel.

---

## 3. Sync engine (`events/`)

`EventsService` (`events/events.service.ts`) is 114 lines and does nothing but delegate — no business logic of its own — to five focused services, plus two small standalone helpers:

| Service | Lines | Responsibility |
|---|---|---|
| `webhook-processor.service.ts` | 921 | Largest file in the module by a wide margin. See §4. |
| `sync-lifecycle.service.ts` | 608 | `updateSyncConfig()` (cross-target migration), `stopSync()`, `enqueueTeardown()`/`executeTeardown()` — retryable, exponential-backoff, batched blocker deletion. |
| `sync-orchestrator.service.ts` | 611 | Initial sync **and** Resync — see naming caveat and §10 cross-reference below. |
| `sync-query.service.ts` | 299 | Read-side queries backing the frontend's sync list/detail views. |
| `blocker.service.ts` | 256 | Blocker event creation/deletion helpers shared by orchestrator, webhook processor, and lifecycle teardown; also owns `createOrUpdateBlocker()`, the shared create-or-diff-and-update logic both `sync-orchestrator.service.ts` and `webhook-processor.service.ts` call into — see [SYNC-FLOW.md](SYNC-FLOW.md) §10. |
| `blocker-content.util.ts` | 45 | Pure function, no DI — resolves a blocker's title/description/location from a source event plus a target's field-visibility toggles. Not a service; consumed only by `blocker.service.ts`. |
| `sync-group-meta.repository.ts` + `.schema.ts` | 64 + 35 | CRUD for the per-target `name`/toggle document (§6, [SYNC-FLOW.md](SYNC-FLOW.md) §10). |

**`SyncOrchestratorService`'s name is closer to accurate than it used to be, but still overstates its scope** — it now handles both initial sync and Resync (added 2026-08-13, [SYNC-FLOW.md](SYNC-FLOW.md) §10), but still not the other two trigger paths below. `queueInitialSync()` validates ownership (SEC-006), writes `SyncConfig` + a `SyncHistory(PENDING)` row synchronously (so `getActiveSyncs` is consistent immediately) and persists this target's `SyncGroupMeta` (name/toggles), then enqueues a BullMQ job. `performInitialSync()` — the worker, run by `sync.processor.ts` on the `SYNC_QUEUE` — pages through source events, creates-or-updates a blocker event in the target for each one (content resolved per this target's toggles, not a fixed string — §10), sweeps stale blockers, then registers a webhook or 5-minute poll per source calendar and separately for the target calendar (tamper-restoration, NR-01). `queueResync()` re-enqueues the same worker job against a target's already-configured sources, with no new user input.

The other two sync triggers do **not** go through `SyncOrchestratorService` at all:

- **Webhook-triggered**: `events.controller.ts`'s `POST /v1/events/notifications` → `WEBHOOK_PROCESSING_QUEUE` → `webhook-processing.processor.ts` → `EventsService.handleWebhook()` → `WebhookProcessorService`.
- **Poll-triggered**: `sync-poll.processor.ts` (BullMQ worker on `SYNC_POLL_QUEUE`), using a Redis `SET NX EX` lock (`poll-lock:{calendarId}`, 600s TTL) to skip overlapping runs, then calling into `WebhookProcessorService` the same way the webhook path does.

So in practice there are three independent trigger paths converging on one shared processing service (`WebhookProcessorService`), not one orchestrator governing all three.

---

## 4. Webhook & renewal architecture

`WebhookProcessorService` (995 lines, `events/webhook-processor.service.ts`) is where webhook- and poll-triggered work actually lands: `handleWebhook()` (parses/validates Google's push via `GoogleCalendarWebhookProvider`, verifies `x-goog-channel-token` with `crypto.timingSafeEqual`, has a loop-guard against target-only calendars re-triggering endlessly), `sweepStaleBlockers()`, `pollTargetCalendar()`/`handleTargetWebhook()` (NR-01 tamper restoration), `pollCalendar()`, `fullyResyncCalendar()`, `applyDeltaEvents()`.

Two independent, differently-shaped renewal mechanisms keep sync alive long-term, because Google and Microsoft's underlying APIs work differently:

**Google — daily webhook-channel renewal.** `webhook/webhook-renewal.service.ts` runs `@Cron('0 0 * * *')` (midnight daily): finds calendars with a channel expiring within 48h, enqueues a `renew` job per calendar (`WEBHOOK_RENEWAL_QUEUE`, 3 attempts, exponential backoff). `webhook/webhook-renewal.processor.ts` does the actual work — stop the old channel, refresh the token if needed, register a new channel via `GoogleCalendarWebhookProvider.createSubscription()`. `webhook/webhook-renewal.service.ts` also exposes `renewAllWebhooks()` as a platform-wide force-renew-all, via `POST /webhook/renew-all` — used after changing `NOTIFICATION_WEBHOOK_URL`. This is not admin-only in the RBAC sense: the route is gated by `InternalSecretGuard`, which requires an `x-session-secret` header matching `SESSION_INTERNAL_SECRET`, not an admin role or even an ordinary session. A browser can't reach it at all — the session service proxy strips that header — so calling it means a direct, non-proxied request with the secret attached.

**Microsoft — monthly full resync.** `events/microsoft-delta-renewal.service.ts` runs `@Cron('0 0 1 * *')` (midnight on the 1st): every Microsoft-provider calendar with an active delta sync token gets a full `fullyResyncCalendar()` call. This exists because Microsoft Graph delta chains expire outright and don't self-extend the way Google sync tokens do — there's no equivalent "renew the channel" operation, so the only reliable recovery is a periodic full resync. This is a materially different reliability mechanism from Google's, and worth knowing about specifically if Microsoft sync ever silently stops working after ~weeks of uptime.

---

## 5. Token & credential flow

Two account-provisioning paths populate `Account` docs differently, and it matters for token refresh — see [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §5 for the full comparison and [GOOGLE-OAUTH.md](GOOGLE-OAUTH.md) for the Add-Account/Reconnect UI flow.

- **`syncFromCoreAuth()` accounts** (from SSO login) frequently have no local `refreshToken`. `google/google-client.factory.ts`'s `refreshTokenIfExpiredForAccount()` handles this: if `refreshToken` is absent, it calls `refreshFromCoreAuth()` — fetches a fresh token from core-be — instead of failing outright. Only when a local `refreshToken` *is* present does it use `(secret removed)()`, wrapped in the `google-token-refresh` circuit breaker.
- **Microsoft has no such fallback.** `microsoft/microsoft-client.factory.ts`'s `refreshIfExpired()` marks `isAuthenticated = false` immediately if `refreshToken` is missing — acceptable because Microsoft accounts are only ever created via the native Add-Account OAuth flow, which always populates it, never via `syncFromCoreAuth()`.
- **Encryption**: `common/crypto/token-encryption.service.ts` uses (secret removed) (12-byte IV), stored as `enc:<iv_hex>:<ciphertext_hex>:<authTag_hex>`. See §11.4 for a caveat on key-length enforcement.
- **`isAuthenticated` self-heal is 401-only, not 401/403** — see §11.1.

---

## 6. Data model

Backend state lives in MongoDB (`arm-calendar`) across 7 collections — full field-level tables, indexes, and the `Calendar.accountId` vs. `Event.accountId` ID-type inconsistency are documented in [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5, not repeated here. Summary:

| Collection | Purpose |
|---|---|
| `accounts` | One doc per user's Google/Microsoft connection — encrypted tokens, `provider`, `isAuthenticated`. |
| `calendars` | One doc per calendar — Google webhook channel fields, `syncToken`, `accessRole`, `webhookSupported`. |
| `events` | Synced/blocker events — `extendedProperties.private.isSource` marks blocker rows. |
| `syncconfigs` | Source calendar → target calendar mapping, one active config per pair. |
| `syncgroupmetas` | One doc per target calendar — the sync's user-facing `name` plus `showTitle`/`showDescription`/`showLocation` toggles, all default `true`. Added 2026-08-13; see [SYNC-FLOW.md](SYNC-FLOW.md) §10. |
| `synchistories` | Per-run audit trail — `PENDING` uniqueness enforced only among pending rows. |
| `auditlogs` | Privileged-action log — `account.delete`/`sync.stop` only, closed union. |

Schemas are colocated per-feature (`account/account.schema.ts`, `events/event.schema.ts`, etc.), not gathered in a shared `schemas/` directory.

---

## 7. Circuit breakers & idempotency

`common/circuit-breaker/circuit-breaker.ts` is a generic closed→open→half-open state machine, instantiated directly (not DI-managed) at exactly 5 call sites:

| Site | Breaker name |
|---|---|
| `common/core-auth/core-auth.service.ts` | `core-be` |
| `google/google-client.factory.ts` | `google-token-refresh` |
| `microsoft/microsoft-client.factory.ts` | `microsoft-token-refresh` |
| `microsoft/microsoft-oauth.service.ts` (token exchange) | `microsoft-token-exchange` |
| `microsoft/microsoft-oauth.service.ts` (profile fetch) | `microsoft-graph-profile` |

**Direct Calendar/Graph data calls are not protected.** `calendar/google-calendar-api.service.ts` and `calendar/microsoft-calendar-api.service.ts` — the actual event CRUD and listing calls the sync engine makes constantly — have no circuit breaker at all. Only the token-refresh and OAuth-exchange paths are covered.

`common/idempotency/idempotency.interceptor.ts` is a Redis-backed **response-replay cache**, not a distributed lock — keyed on `idempotency:{method}:{route}:{userId}:{key}`, 24h TTL, opt-in via `@UseInterceptors`, used only on the sync-creation endpoint. It skips caching "logical failures" (a `{success:false}` or `statusCode>=400` body) even when the HTTP status itself was 200.

---

## 8. Health checks

`health/health.controller.ts` deliberately splits liveness from readiness:

- `GET /health/live` — process-alive only, **no dependency checks**, so an orchestrator won't kill a healthy process over a transient dependency blip.
- `GET /health/ready` — `Promise.allSettled([checkMongo(), checkRedis()])`. Mongo check does a `readyState` guard *plus* a real `db.admin().ping()` round-trip (readyState alone can lag reality mid-reconnect). Redis check verifies the literal `PONG` reply. Either failing returns `503` with `{status, details: {mongo, redis}}`.

---

## 9. Frontend

React 19 + Vite, Module Federation **remote** (federation name `calendar`, `remoteEntry.js` on port `3001`), loaded by `core/arm-core-fe` at runtime — see [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) for host/remote wiring, shared-dependency negotiation, and the CSS strategy, none of which is repeated here.

**Exposes** (`vite.config.ts`) — five separate entries, not one combined module:

| Export | Component | Shell route | Purpose |
|---|---|---|---|
| `./Page-1` | `calendar-account-page.tsx` | `/calendar/syncs` | Linked-accounts / calendar picker |
| `./SyncDetail` | `sync-detail-page.tsx` | `/calendar/syncs/new`, `/calendar/syncs/:id` | Single sync's detail view |
| `./MergedView` | `calendar-merged-page.tsx` | `/calendar/view` | Merged grid across every sync's source calendars |
| `./SyncView` | `sync-view-page.tsx` | `/calendar/view/:targetCalendarId` | Single sync's calendar overview |
| `./AuthCallback` | `auth-callback.tsx` | `/calendar/account`, `/calendar/auth/callback` | OAuth popup return page |

Both grid views render the design system's `BigCalendarView` (react-big-calendar). FullCalendar was removed from this app — it survives only as a stale `package-lock.json` entry, and the `.calendar-scope` wrapper it required is gone from both the shell and this app's own `App.tsx`.

`src/features/calendar/` holds all feature code: `components/` (the exposed pages plus modal/dialog UI), `hooks/` (`use-calendar-sync`, `use-active-syncs`, `use-calendar-sse`, `use-sync-detail`, `use-sync-detail-view`, `use-theme`), `services/api.ts` (every session service-proxied fetch call), `types/`, `utils/`, `constants/`.

**Standalone-dev stubbing**: `vite.config.ts`'s `stubMfRemotesPlugin()` intercepts `core/theme-store` (a host-provided MF import) during standalone `pnpm dev` and swaps in a no-op stub, since the host isn't present to provide it. Add a stub here for any new host-provided import.

**Live updates**: `use-calendar-sse.ts` opens `EventSource` against `/events/stream` — backed server-side by `common/redis/SseRedisService`'s per-user pub/sub channel — so sync progress and blocker creation appear in the UI without polling.

---

## 10. Environment variables (calendar-be specific)

See [ENV-VARS.md](../ENV-VARS.md) for the full platform reference; calendar-be-specific constraints:

- `GOOGLE_TOKEN_ENCRYPTION_KEY` — intended to be exactly 64 hex chars (`openssl rand -hex 32`); see §11.4 for what actually happens if it isn't.
- `JWT_ACCESS_SECRET` — must be identical to `arm-core-be`. Calendar-be does **not** read `JWT_REFRESH_SECRET` at all.
- `MONGODB_URI` — must embed the same password as `MONGO_ROOT_PASSWORD`.
- `NOTIFICATION_WEBHOOK_URL` — must be HTTPS or a known-internal host (`common/config/env.ts`'s `requireWebhookUrl` enforces this at startup). This is the URL Google's push service calls directly — see §11.3, it does **not** go through the session service proxy.
- `TRUST_PROXY_HEADERS` — must stay unset/`false` anywhere without a real Kong gateway in front; see [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §3 for why (SEC-001).

---

## 11. Verified drift from `CLAUDE.md`

Found while cross-checking `apps/arm-app-calendar/CLAUDE.md` against source for this document. None are blocking; all are worth fixing in `CLAUDE.md` since it's the file Claude Code auto-loads for this repo.

1. **401/403 self-heal claim is half-true.** `CLAUDE.md` says both Google and Microsoft API services "flip `isAuthenticated` to `false` when a call returns 401/403." In fact `isAuthError()` in both `calendar/google-calendar-api.service.ts` and `calendar/microsoft-calendar-api.service.ts` checks **only 401**. Google 403s go through a separate `isQuotaError()` path that retries with backoff, not `markAccountUnauthenticated()`; Microsoft has no 403-specific handling at all.
2. **`webhook/` doesn't contain "the Google push notification handler."** `CLAUDE.md`'s directory-tree comment labels it that way, but the directory holds only the renewal cron/processor/admin-controller (§4). The actual inbound handler for Google's push notifications is `POST /v1/events/notifications` in `events/events.controller.ts`, processed entirely within `events/` (§3–§4).
3. **The webhook intake route doesn't go through the session service proxy**, contrary to `CLAUDE.md`'s example path (`/session/proxy/calendar/v1/webhooks/...`). The real route is `POST /v1/events/notifications`, called directly via `NOTIFICATION_WEBHOOK_URL` — it has to be, since Google's push service can't carry a browser session cookie or hit an authenticated session service route. The route has no `AuthGuard` (correct — Google calls it, not a logged-in user) and is `@SkipThrottle()`.
4. **`GOOGLE_TOKEN_ENCRYPTION_KEY`'s "must be exactly 64 hex characters" is not enforced.** `token-encryption.service.ts` silently pads/truncates a malformed key to 32 bytes rather than throwing at startup — a misconfigured key produces a usable but insecure result instead of failing fast, contrary to what the docs imply.
5. **`auditlogs` is missing from `CLAUDE.md`'s MongoDB Collections table** even though `CLAUDE.md` itself references `audit-log.schema.ts` one line above that table. [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5.6 already documents it correctly — treat that as the source of truth over `CLAUDE.md`'s table.
6. **Microsoft's monthly delta-renewal cron is undocumented as an architectural mechanism** — `CLAUDE.md` mentions it only as three words in a directory-tree comment. It's a materially different reliability strategy from Google's daily channel renewal (§4) and deserves to be described as such, not just named.
7. **`SyncOrchestratorService` isn't really "the orchestrator"** of all three sync triggers — only initial sync. Webhook- and poll-triggered syncs route through `WebhookProcessorService`/`sync-poll.processor.ts` independently and never touch it (§3). Worth naming this precisely in `CLAUDE.md` so readers don't assume one class governs every trigger path.

---

## Related documents

- [SYNC-FLOW.md](SYNC-FLOW.md) — algorithm-level walkthrough of §3–§4's sync/webhook engine: initial-sync steps, BullMQ retry config, delta processing, teardown
- [WEBHOOKS.md](WEBHOOKS.md) — channel registration, intake validation, renewal, and the precise scope of the open renewal-failure gap (`TASK-045`)
- [GOOGLE-OAUTH.md](GOOGLE-OAUTH.md) — Add Account / Reconnect OAuth flow (popup, session service routes, BroadcastChannel)
- [DELETE-CASCADE.md](DELETE-CASCADE.md) — Account unlink cascade, ordered cleanup, ID contract
- [API.md](API.md) — every calendar-be HTTP endpoint, the complete surface map
- [LINKED-ACCOUNTS.md](LINKED-ACCOUNTS.md) — the Linked Accounts feature end to end (core-fe UI + calendar-be endpoints)
- [DATA-MODEL.md](DATA-MODEL.md) — calendar-scoped index into the MongoDB schema reference
- [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5 — full MongoDB schema field tables and indexes
- [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) — guard pattern comparison across all 4 backends, deny-list gap
- [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) — MF host/remote wiring, CSS isolation, shared-dependency negotiation
- [SECURITY.md](../SECURITY.md) — OWASP coverage, SEC-00x fix history referenced throughout this doc
