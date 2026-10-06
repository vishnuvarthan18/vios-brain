# araMetrics Calendar — App Spec (single source of truth)

**Status:** Approved 2026-07-13. Supersedes `CALENDAR-BUILD-PLAN.md`.
**Scope:** `/calendar` app + Settings → Integrations. Mock UI only — no real API.

---

## Locked decisions (user sign-off)

| # | Topic | Decision |
|---|-------|----------|
| 1 | Doc cleanup | Delete only `CALENDAR-BUILD-PLAN.md`; keep RADIX + CLEANUP plans |
| 2 | Create sync UX | Single page — name + target + sources together |
| 3 | Hide clones | Match CalendarBridge/OneCal — hide sync-generated copies on targets; native sources always show |
| 4 | Merged view | Sources only — targets don't feed the grid unless also a source |
| 5 | OAuth return | Return to where Connect was clicked (Integrations or sync editor) |
| 6 | Account remove | Drop sync entirely if it loses target or all sources |
| 7 | Sync status | `synced` \| `syncing` \| `error` — account reauth → sync `error` |
| 8 | Privacy note | Plural: *"On your target calendars, others see only Busy…"* |
| 9 | Initial state | Empty on fresh login |
| 10 | API layer | Pure context state |
| 11 | Sidebar | **Integrations** in Settings; in-card heading **Connected accounts** |

---

## A · Product definition

### A1. What the app is

A calendar merger inside araMetrics. Connect Google + Microsoft accounts once (platform level); create named syncs that mirror source events into a target calendar as **Busy** blocks; view everything merged in one calendar. UI only.

### A2. Locked product decisions

| ID | Decision |
|----|----------|
| D1 | Connected accounts → Settings → Integrations (platform asset) |
| D2 | Multiple named syncs (Model B) |
| D3 | Calendar-level granularity (target + sources = individual calendars) |
| D4 | One-way; copies read-only |
| D5 | Always Busy to others; full detail only in ARM |
| D6 | Sync rows: status pill + Edit / Resync / Remove |
| D7 | Merged view = union of all sync sources, colored by account, legend toggles + hide clones |
| D8 | Create sync = name + target + sources on one page |
| D9 | Radix/amber DS; competitors inform behavior only |

---

## B · Screen-by-screen spec

### B1. Settings → Integrations `/settings/integrations`

Platform account management. Settings-card shell; in-card label **Connected accounts**; account grid (`AccountCard` + provider badge + kebab); **Connect account** top-right.

**States:** empty · populated · reauth banner · all providers exhausted (hide Connect).

**Actions:** Connect → provider chooser → OAuth → return here · kebab Pause/Resume/Remove · Reconnect · card click = no-op.

**Copy:** Remove confirm per plan; Paused badge; reauth banner `{email} needs to be reconnected.`

### B2. Calendar `/calendar`

Merged calendar home. Week/Month tabs · **Manage syncs** link · merged grid · per-account legend toggles · hide-clones · privacy note.

**States:** no accounts → empty state CTA `/settings/integrations` · accounts/no syncs → create first sync CTA `/calendar/syncs/new` · healthy merged view · reauth banner · all hidden → empty message.

**Copy:** Privacy note plural; legend account + dot + provider badge; hide-clones checkbox.

### B3. Syncs list `/calendar/syncs`

Header **Syncs** + New sync · `SyncRow` list with status pills.

**Actions:** New · Edit · Resync · Remove (confirm).

### B4. Sync editor `/calendar/syncs/new` & `/calendar/syncs/:id`

Single page: name · target calendar (radio, grouped by account) · source calendars (checkbox tree) · preview `MonthGrid` · Cancel/Save.

Target auto-excluded from sources. Reauth accounts' calendars disabled.

### B5. OAuth `/calendar-oauth`

Unchanged flow; `return` query param for post-OAuth landing.

---

## C · Other ARM screens

- **Home:** granted-app card → `/calendar`; glance from merged union; no syncs → link `/calendar/syncs/new`
- **Admin Calendar Ops:** placeholder copy updated to new model
- **Search:** Calendar subtitle + Syncs result → `/calendar/syncs`

---

## D · Architecture

### D-arch. Layers

- **Platform** — `IntegrationsContext` @ Settings → Integrations
- **App** — `CalendarContext` @ Calendar (syncs + merged view)

### D-data

```ts
// integrations-context
{ accounts, pausedAccountIds, add/remove/reconnect/pause/resume, isPaused, availableProviders }

// calendar-context
type SyncStatus = "synced" | "syncing" | "error"
Sync { id, name, targetCalendarId, sourceCalendarIds[], status, lastSyncedLabel? }
{ syncs, view, weekOffset, hiddenCalendarIds, hideClones,
  mergedCalendarIds, visibleCalendarIds, getVisibleBlocksInRange(start,end),
  CRUD + toggleCalendarVisible + setHideClones }
// reads accounts via useIntegrations()
```

Visibility is **per calendar** (`hiddenCalendarIds`), not per account, so the
merged-view legend toggles individual source calendars. `getVisibleBlocksInRange`
returns date-anchored occurrences already filtered by calendar visibility +
hide-clones — the single source for the week grid, month dots, and Home glance.

### D-data — dated occurrences

`AVAILABILITY_BLOCKS` are week-relative *templates*. `DATED_AVAILABILITY_BLOCKS`
expands them into concrete dated occurrences across a window around `MOCK_TODAY`
(−5 … +9 weeks). The current week is always fully populated; other weeks vary
deterministically (stable hash) so week/month navigation actually changes what
you see. `getDatedBlocksInRange(start, end)` does the range lookup.

### D-react

`useMemo` (derived, not effect) prunes syncs when accounts change: strip missing
calendarIds; drop syncs with no target or no sources.

### D-routes

| Route | Page |
|-------|------|
| `/settings/integrations` | IntegrationsSettingsPage |
| `/calendar` | CalendarPage |
| `/calendar/syncs` | SyncsListPage |
| `/calendar/syncs/new`, `/calendar/syncs/:id` | SyncEditorPage |
| `/calendar-oauth` | CalendarOAuthPage |

**Sidebar:** Settings → Account, Linked, **Integrations** · Calendar → Calendar (end), **Syncs**

**Providers:** `<IntegrationsProvider>` outside `<CalendarProvider>`, whole tree.

---

## E · Files

**New:** `integrations-context.tsx`, `calendar-helpers.ts`, `IntegrationsSettingsPage.tsx`, `SyncsListPage.tsx`, `SyncEditorPage.tsx`, `SyncRow.tsx`, `SyncEditor.tsx`

**Reworked:** `calendar-context.tsx`, `AvailabilityView.tsx`, `SyncEditor.tsx`, `AccountManager.tsx`, `useAddAccountFlow.tsx`, `CalendarPage.tsx`, `CalendarEmpty.tsx`, `CalendarOAuthPage.tsx`, `MonthGrid.tsx` (per-day `dayDots`), `calendar-glance.ts` (dated), `App.tsx`, `Sidebar.tsx`, `HomePage.tsx`, `mock.ts` (dated occurrences), `status-colors.ts`, `stories/app/_mocks.tsx`

**Removed:** `CalendarAccountsPage.tsx`, `CalendarSetupPage.tsx`, `SyncStatus.tsx` (replaced by sync list), `SyncSetup.tsx` (→ `SyncEditor.tsx`)

---

## F · Build order

1. integrations-context + App wrap + useAddAccountFlow
2. IntegrationsSettingsPage + route + sidebar
3. calendar-context syncs + D-react + derived union
4. SyncsListPage + SyncRow + routes
5. SyncEditorPage + target picker
6. AvailabilityView union + legend + hide-clones
7. CalendarPage branching + HomePage + search
8. Remove dead routes; stories; Admin copy
9. Verify + delete `CALENDAR-BUILD-PLAN.md`; update `AGENTS.md`

---

## G · Verification

**Automated**

```bash
npm run lint && npm run typecheck && npm run build
```

**Live drive** (`npm run dev` — default http://localhost:5173; Vite picks the next free port if busy)

Login: `/login` → `user@arametrics.io` → 4-digit code from the Dev hint callout.

| # | Route / action | What to confirm |
|---|----------------|-----------------|
| 1 | `/calendar` (no accounts) | "Connect a calendar" opens provider chooser **on this page** — no hop to Integrations (F6) |
| 2 | `/settings/integrations` | Connect Google + Microsoft; reauth banner + named Reconnect buttons (U5) |
| 3 | `/calendar/syncs/new` | Pick target + sources; preview month dots update as sources are checked (F5); "Add account" mid-edit preserves draft after OAuth return (F3); reauth account cannot be target (F7) |
| 4 | `/calendar/syncs` | Two syncs listed; status pills incl. error row for reauth account |
| 5 | `/calendar` (week) | ‹ / › changes events, not a frozen loop (F1); empty week shows "Nothing scheduled this week" (F2) |
| 6 | `/calendar` (month) | Month ‹ / › works (F4); dots are per-account colored, max 3/day (U1) |
| 7 | `/calendar` legend | Toggles are **per calendar** (U2); paused accounts absent (U4); Hide clones suppresses target copies only |
| 8 | `/` Home | "Calendar at a glance" shows upcoming dated events |
| 9 | Remove an account | Syncs self-prune; no crash |
| 10 | Deep-link | Every route above loads directly |

Screenshots for review: Integrations, Syncs list (+error row), Sync editor, merged week view, Home glance.

**Regression baseline (pre-fixes):** connect accounts → create syncs → merged union → legend/hide-clones → account remove prunes syncs → deep-link all routes.

---

## H · Deferred

Real API, localStorage persistence, Day/Agenda, keyboard nav, two-way sync, per-sync privacy, sync start-date wiring.

---

## I · Post-build fixes (flow + UI)

Addressed after the first live drive surfaced flow/UI bugs vs. competitor patterns:

- **F1 · Real week navigation.** Events are now date-anchored (`DATED_AVAILABILITY_BLOCKS`); the week grid filters by the visible week's actual dates, so ‹ / › changes what's shown instead of looping the same week.
- **F2 · Empty-week state.** "Nothing scheduled this week" now shows whenever the visible week truly has no events (any offset), not just as dead code.
- **F3 · Draft survives "Add account".** The sync editor snapshots its form to `sessionStorage` (keyed by route) across the OAuth round trip; cleared on Save/Cancel/Back.
- **F4 · Real month navigation.** Month view has its own cursor; ‹ / › pages months and repaints dots for that month.
- **F5 · Live editor preview.** The editor's preview month grid renders colored dots for the currently selected source calendars.
- **F6 · One-hop connect.** The empty `/calendar` state opens the provider chooser directly (and handles the OAuth return) instead of bouncing to Integrations.
- **F7 · Reauth target locked.** Accounts needing reconnect are disabled as sync **targets** too (previously only sources).
- **U1 · Colored month dots.** `MonthGrid` renders up to 3 per-day dots in source-account colors (via `dayDots`), replacing the single generic amber dot.
- **U2 · Per-calendar legend.** Legend toggles individual source calendars (`hiddenCalendarIds` / `toggleCalendarVisible`) rather than whole accounts.
- **U4 · Paused hidden from legend.** Paused accounts' calendars no longer appear in the legend.
- **U5 · Named reconnect.** Reconnect buttons read "Reconnect {name}" so it's clear which account they act on.
