# Calendar App — Linked Accounts Feature

The "connect a Google/Microsoft calendar and sync it" feature end-to-end. **This is a two-repo feature, not a calendar-app-only one**: the UI (`LinkedAccountsTab`, `LinkedAccountCard`, the whole Settings entry point) lives in `core-fe`, the API and data model live here in calendar-be. This document is the single reference tying both halves together — it does not duplicate the OAuth mechanics ([`GOOGLE-OAUTH.md`](GOOGLE-OAUTH.md) already covers those in full) or the unlink cascade ([`DELETE-CASCADE.md`](DELETE-CASCADE.md) already covers that), it cross-links both and covers what neither does: the data model, the sync-status model, the full API surface, and the frontend components.

**Correcting the original scope of this doc**: it was originally planned around a `provider` field as future groundwork for Outlook/Apple. That's stale — Microsoft is already a fully live second provider (see [`calendar/ARCHITECTURE.md`](ARCHITECTURE.md) and `apps/arm-app-calendar/CLAUDE.md`'s "Calendar providers: Google fully featured, Microsoft is CRUD-only" section). This document covers Microsoft as a real, working integration throughout, not a placeholder.

---

## 1. Feature overview

One ARM user → multiple connected Google/Microsoft accounts → each account's calendars can be a sync source and/or target. The user manages this from **Settings → Linked Accounts** (`core-fe`, `/settings`), which lists every connected account, shows live sync status per account, and offers Add/Reconnect/Unlink actions. Connecting an account is a popup-based OAuth flow into calendar-be; syncing itself (what actually happens once an account is linked) is [`SYNC-FLOW.md`](SYNC-FLOW.md)'s subject, not this one.

```text
core-fe (Settings page)                    calendar-be
─────────────────────                      ───────────
LinkedAccountsTab
  ├─ useLinkedAccounts() ──── GET /v1/account/accounts ──────► AccountController.listCalendarsAccounts
  │                     ──── GET /v1/account/:id/sync-status ► AccountController.getSyncStatus
  ├─ LinkedAccountCard × N (one per account, badge + actions)
  ├─ "Add Account" dropdown ── popup: /session/calendar/add-account(/microsoft) ─► see GOOGLE-OAUTH.md
  ├─ "Reconnect" (per expired card) ── GET reconnect URL ────► see GOOGLE-OAUTH.md §8
  └─ UnlinkConfirmDialog ──── DELETE /v1/account/:id ─────────► see DELETE-CASCADE.md
```

All calls go through the session service proxy (`/session/proxy/calendar/v1/...`), never calendar-be directly — same rule as every other MFE-to-backend call on this platform.

---

## 2. Account data model

Mongo collection `accounts` (`account.schema.ts`), one document per connected Google/Microsoft account:

| Field | Notes |
|---|---|
| `userId` | ARM platform user ID (indexed, not unique — one user can have many accounts) |
| `accountId` | Unique, indexed — the value the frontend and every other API keys on |
| `email`, `displayName`, `organization`, `picture` | Profile fields from the provider's userinfo/profile call |
| `provider` | `'google'` \| `'microsoft'` (default `'google'`) — **live**, drives which `CalendarProvider` adapter `CalendarProviderRegistry` resolves for this account, not a placeholder. See `calendar/ARCHITECTURE.md`. |
| `accessToken` / `refreshToken` | Encrypted ((secret removed)), never returned to the frontend — see `toPublicAccount()` in §4 |
| `tokenExpiryAt` | Checked before every provider API call |
| `syncDate` | Last successful sync timestamp — distinct from `SyncHistory` (§3), this is a denormalized convenience field on the account itself |
| `onSync` | Default `true` — whether this account participates in scheduled syncs; toggled via `PUT /v1/account/:accountId` (pause/resume), not exposed anywhere in the current UI (see §4) |
| `isAuthenticated` | Default `true`, flips to `false` on a 401/403 from the provider (`markAccountUnauthenticated`, self-healing per `apps/arm-app-calendar/CLAUDE.md`) — this is what drives the "Reconnect required" badge in §5 |

**An account can be created two different ways, and they don't populate the same fields** (per `apps/arm-app-calendar/CLAUDE.md`):

1. **"Add Account" (calendar-be-native, the only path the current UI exercises)** — `GoogleOAuthService`/`MicrosoftOAuthService`, does its own consent redirect and code exchange, populates `refreshToken` directly from the token response. Full flow: [`GOOGLE-OAUTH.md`](GOOGLE-OAUTH.md) §2.
2. **`syncFromCoreAuth()` (older, proxies core-be's own Google token)** — `refreshToken` is almost never populated this way (a known gap, `apps/arm-app-calendar/CLAUDE.md` calls it "TASK-037", since fixed to fall back to `refreshFromCoreAuth()` at refresh time rather than fail). **Verified for this document: nothing in the current frontend ever triggers this path.** `GET /v1/account/sync-from-core` (`AccountController.syncFromCore` → `AccountService.syncFromCoreAuth`) has no caller anywhere in `core-fe` or `calendar-fe` — grepped both. Same for the `token`/`userId` query-param branch of `calendar/AuthCallback` (`apps/arm-app-calendar/src/frontend/src/features/calendar/pages/auth-callback.tsx`'s `fetchAvailableAccounts()`/`handleAccountSelect()`, which calls `GET /v1/account/available` and `POST /v1/account/connect`): both core-fe's `linked-accounts-tab.tsx` and calendar-fe's `calendar-account-page.tsx` only ever open the modern `/session/calendar/add-account(/microsoft)` popup, which completes via the `action=account_added` branch (BroadcastChannel, per `GOOGLE-OAUTH.md` §6), never the `token`+`userId` branch. This whole path — `syncFromCoreAuth`, `/account/sync-from-core`, `/account/available`, `/account/connect`, and the corresponding `auth-callback.tsx` branch — is live, working code with no reachable UI trigger today, not formally dead but not exercised by anything a user can actually do in the app.

---

## 3. Sync status model

Per-account sync status is **not** stored on the `Account` document itself — `GET /v1/account/:accountId/sync-status` (`AccountTokenService.getSyncStatus`) reads the most recent `SyncHistory` record for that account instead (see [`SYNC-FLOW.md`](SYNC-FLOW.md) for how those records get written):

| `SyncStatus` value (`sync-history.schema.ts`) | Meaning | `LinkedAccountCard` badge |
|---|---|---|
| *(no `SyncHistory` record exists yet)* | Never synced | `idle` — neutral "Idle" |
| `pending` | `SyncOrchestratorService` wrote the record, job not yet started | `pending` — warning "Pending" |
| `in-progress` | Sync actively running | `in-progress` — primary "Syncing" with a spinner |
| `completed` | Last run succeeded | `completed` — success "Synced" |
| `partial` | Last run partially succeeded | *(no dedicated badge — falls through to the `idle` default in `STATUS_BADGE`, since `partial` isn't a key in that map)* |
| `failed` | Last run failed | `failed` — error "Sync failed", expandable error-details accordion showing `errorMessage` |

**`isAuthenticated: false` overrides all of the above** — `LinkedAccountCard` computes its status as `!account.isAuthenticated ? "expired" : (syncStatus?.status ?? "idle")`, so an expired token always shows "Reconnect required" (warning badge + orange card border + banner), regardless of what the last sync's status actually was.

`useLinkedAccounts()` polls every 15s (`POLL_INTERVAL_MS`), but only while at least one account's status is `pending` or `in-progress` — otherwise no polling happens at all, so a `completed`/`failed`/`idle`/`expired` account list is fetched once and left alone until the user reopens the tab or triggers an action.

---

## 4. API endpoints (`/v1/account/*`, calendar-be)

All guarded by `AuthGuard` (JWT). Fuller than the doc's original outline assumed — 10 routes, not 3:

| Method | Path | Purpose | Called from the current UI? |
|---|---|---|---|
| `POST` | `/account` | Generic create/update by arbitrary body | Not observed in either frontend |
| `GET` | `/account` | List accounts for the caller, paginated | Not observed — superseded by `/account/accounts` below |
| `GET` | `/account/accounts` | List accounts for the caller, paginated — **this is the one `SettingsService.getLinkedAccounts()` actually calls** | Yes — `useLinkedAccounts()` |
| `GET` | `/account/available` | List the caller's already-core-authenticated Google accounts not yet linked | No — see §2, part of the unreached legacy path |
| `POST` | `/account/connect` | Finalize linking one of the `available` accounts (idempotency-guarded) | No — see §2 |
| `GET` | `/account/sync-from-core` | Sync a Google account from core-be's own token store | No — see §2 |
| `GET` | `/account/:accountId/sync-status` | Latest `SyncHistory` status for one account (§3) | Yes — `useLinkedAccounts()` |
| `DELETE` | `/account/:accountId` | Unlink, full cascade | Yes — `UnlinkConfirmDialog` → `SettingsService.deleteLinkedAccount()`. Full behavior: [`DELETE-CASCADE.md`](DELETE-CASCADE.md) |
| `PUT` | `/account/:email/tokens` | Update stored tokens for an account, keyed by email | Not observed in either frontend (internal/service use) |
| `PUT` | `/account/:accountId` | Toggle `onSync` (pause/resume scheduled syncing) | Not observed — no pause/resume control exists in `LinkedAccountCard` or `LinkedAccountsTab` today, despite the endpoint and the `onSync` field both being fully implemented |

**Response shape never includes tokens.** Every route that returns an account runs it through `AccountService.toPublicAccount()` first: `{ id, accountId, email, displayName, organization, picture, provider, isAuthenticated, onSync, syncDate }` — `accessToken`/`refreshToken`/`tokenExpiryAt` are stripped before the response leaves calendar-be.

---

## 5. Frontend components (`core-fe`)

| Component | File | Role |
|---|---|---|
| `SettingsPage` → Linked Accounts tab | `core/arm-core-fe/src/pages/home/settings-page.tsx` | Lazily mounts `LinkedAccountsTab` only when the tab is opened — `useLinkedAccounts()`'s fetch doesn't fire on every Settings page load |
| `LinkedAccountsTab` | `core/arm-core-fe/src/components/home/linked-accounts/linked-accounts-tab.tsx` | Header, "Add Account" dropdown (Google/Microsoft), account list, empty state, wires up `useOAuthPopup` and `UnlinkConfirmDialog` |
| `LinkedAccountCard` | `.../linked-accounts/linked-account-card.tsx` | One account row: avatar + provider icon, identity, "Last synced" relative time, status badge (§3), Reconnect/Unlink buttons, expandable error details when `failed` |
| `UnlinkConfirmDialog` | `.../linked-accounts/unlink-confirm-dialog.tsx` | Explicit confirmation before unlink — copy states plainly that synced events will be deleted and it can't be undone |
| `useLinkedAccounts()` | `core/arm-core-fe/src/hooks/use-linked-accounts.ts` | Fetch/poll accounts + sync statuses, unlink state machine (`unlinkingId`, `confirmDialog`) |
| `SettingsService` | `core/arm-core-fe/src/api/services/settings-service.ts` | The 4 API calls this feature actually uses: `getLinkedAccounts`, `deleteLinkedAccount`, `getAccountSyncStatus`, `getReconnectUrl` |

Types: `LinkedAccount`/`SyncStatus` in `core/arm-core-fe/src/types/linked-account.ts` — the file's own comment already flags `provider` as free-form ("stored/returned as-is, never used server-side to construct commands") and lists `'outlook' | 'apple'` as documented future values, consistent with `provider` being live for Google/Microsoft today and genuinely open-ended, not hardcoded to two values anywhere in the type system.

---

## 6. Add Account / Reconnect flows — pointers, not duplicated here

Both are fully documented in [`GOOGLE-OAUTH.md`](GOOGLE-OAUTH.md) (Microsoft's twin flow: §10) — this section only maps UI action to doc section:

- **Add Account** (`LinkedAccountsTab`'s dropdown → `useOAuthPopup` → `/session/calendar/add-account(/microsoft)`) → GOOGLE-OAUTH.md §2–§7
- **Reconnect** (`LinkedAccountCard`'s "Reconnect" button, shown when `isAuthenticated: false` → `SettingsService.getReconnectUrl(email)` → same popup mechanism) → GOOGLE-OAUTH.md §8
- **Unlink** (`UnlinkConfirmDialog` → `DELETE /v1/account/:accountId`) → [`DELETE-CASCADE.md`](DELETE-CASCADE.md), including its §7 known gaps (poll schedulers not cleared, no token revoke, no Microsoft subscription cleanup)

---

## 7. Known constraints

- **One OAuth app per provider, platform-wide** — `GOOGLE_CLIENT_ID`/`MICROSOFT_CLIENT_ID` are single env vars, not per-tenant config. Every ARM deployment shares one Google Cloud OAuth consent screen and one Azure AD app registration.
- **Redirect URIs must be pre-registered exactly** — `OAUTH_CALLBACK_URL` (Add Account) and `GOOGLE_REDIRECT_URI`/`GOOGLE_CALENDAR_CALLBACK_URL` (Reconnect) are **different URIs for the same provider** and both must be registered in Google Cloud Console; a mismatch fails the OAuth exchange with a provider-side error, not an ARM one. Full detail and known `.env.example` drift: GOOGLE-OAUTH.md §9.
- **Pause/resume (`onSync`) has no UI today** — the field and its `PUT /v1/account/:accountId` endpoint are fully implemented backend-side (§2, §4) but nothing in `LinkedAccountCard`/`LinkedAccountsTab` exposes a toggle for it. An account can only be fully unlinked, never temporarily paused, from the current UI.
- **The legacy `available`/`connect`/`sync-from-core` path (§2) is unreachable, not removed** — if you're debugging why a Google account doesn't appear after some flow, confirm which of the two account-creation paths was actually exercised before assuming the modern Add Account flow (GOOGLE-OAUTH.md) is at fault.

---

## Related documents

- [`calendar/GOOGLE-OAUTH.md`](GOOGLE-OAUTH.md) — Add Account / Reconnect OAuth mechanics in full, both providers
- [`calendar/DELETE-CASCADE.md`](DELETE-CASCADE.md) — unlink cascade order and known gaps
- [`calendar/ARCHITECTURE.md`](ARCHITECTURE.md) — `CalendarProviderRegistry`, why Microsoft is CRUD-only, cross-provider sync support
- [`calendar/SYNC-FLOW.md`](SYNC-FLOW.md) — what actually happens once an account is linked and syncing starts, `SyncHistory` write points
- [`core/arm-core-fe/README.md`](../../../../core/arm-core-fe/README.md) — Settings page tab structure, where Linked Accounts sits alongside Account Settings
