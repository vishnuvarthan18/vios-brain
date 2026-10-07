# Calendar App — Delete Cascade Specification

> Authoritative description of what happens when a linked calendar account is unlinked (`DELETE /v1/account/:accountId`).
>
> This documents **what the code does today** in `AccountConnectionService.deleteAccount`, not an aspirational wishlist. Known gaps are called out explicitly in §7.
>
> Verified against `apps/arm-app-calendar/src/backend/src/account/account-connection.service.ts` and callers (2026-08-10; step 8b and the §7 Google-search gap added 2026-08-13 alongside per-sync field-visibility toggles; §1 gained three frontend entry points on 2026-08-28 when an account with no imported calendars stopped surviving — the cascade itself is unchanged; §1's Settings caller gained a pre-flight impact disclosure on 2026-08-29, again with no cascade change).

---

## 1. Entry point

| Item | Value |
|---|---|
| Route | `DELETE /v1/account/:accountId` |
| Guard | `AuthGuard` (JWT from session service; or `X-User-Id` only when `TRUST_PROXY_HEADERS=true`) |
| Controller | `AccountController.delete` → `AccountService.deleteAccount` → `AccountConnectionService.deleteAccount` |
| Proxy path (FE) | `DELETE /session/proxy/calendar/v1/account/{id}` |
| Audit | `auditLogService.record({ action: 'account.delete', targetId, outcome })` on success and failure |

### Path param contract

Despite the name `:accountId`, the value **must be the Mongo `_id`** of the `accounts` document.

| FE field | Meaning | Use for DELETE? |
|---|---|---|
| `account.id` | Mongo `_id` | **Yes** — both UIs send this |
| `account.accountId` | Provider id (Google email / MS id) | **No** — used inside cascade as `googleAccountId` for Calendar ownership + Google client |

Lookup: `accountRepo.findByIdAndUserId(id, userId)` → `{ _id: id, userId }`.

Two different outcomes here, and they are easy to conflate (HTTP status is 200 from Nest either way — the body carries the real one):

| Id | Path through the controller | Response body |
|---|---|---|
| Well-formed ObjectId, no such document | lookup returns `null` → `Error('Account not found')` | `statusCode: 410`, `'Account does not exist'` |
| Not a valid ObjectId (e.g. a provider account id) | Mongoose `CastError` before the lookup can report absence | `statusCode: 400`, `'Account deletion failed'` |

Verified 2026-08-18 against `tests/account.e2e-spec.ts`. An earlier revision of this section claimed 410 for both; sending a provider id gets you the 400 row, not the 410 one.

### UI callers

| Surface | Sends | Trigger |
|---|---|---|
| Calendar account page (`handleUnlink`) | `account.id` | Explicit Disconnect, confirmed |
| Settings → Linked Accounts (`SettingsService.deleteLinkedAccount`) | `account.id` | Explicit Disconnect, confirmed — confirm dialog names the affected calendars first (see below) |
| Calendar account page (`discardUnimportedAccount`) | `account.id` | Import picker dismissed with nothing imported — **no confirm** |
| Calendar account page (`handleRemoveCalendar`) | `account.id` | Removing an account's **last** imported calendar, confirmed |
| Calendar account page (load sweep) | `account.id` | Mount finds an account with nothing imported — **no confirm, no user gesture** |

All five hit the same calendar-be cascade via the session service proxy.

### Pre-flight impact disclosure (Settings only)

Since 2026-08-29 the Settings caller tells the user what the cascade will take
with it *before* they confirm, rather than after. It is disclosure only — the
unlink is never blocked, and nothing about the cascade or the endpoint changed.

`useLinkedAccounts` fetches `GET /events/active-syncs` alongside the account
list, and `deriveUnlinkImpact` (`core/arm-core-fe/src/utils/unlink-impact.ts`)
splits the account's syncs by role:

| Role in the sync | What unlinking does | Disclosed as |
|---|---|---|
| **Target** (`targetAccountMongoId` matches) | Removes a calendar the user is already deleting | Counted in `syncCount` only |
| **Source** (any `sources[].sourceAccountMongoId` matches) | Strips blockers from a calendar belonging to a **different** account, which the user never selected — steps 5 and the AC-3 sweep below | Named in `otherCalendars`, calendar + owning email |

`syncCount` counts distinct target calendars, not source→target edges, so a
target fed by three sources reads as one sync — matching the calendar app's own
sync grouping. The same derivation drives an at-rest "In N syncs" line on each
account row, so the dialog is not the first place this is visible.

Two limits worth knowing:

- **`getActiveSyncs` does not filter `stoppingAt`** (it calls
  `syncConfigRepo.findByUserId`, unlike `findBySourceCalendarIds` and
  `countBySourceCalendarId`, which both filter `stoppingAt: null`). A sync
  mid-teardown is still counted, for the length of that teardown.
- A calendar the backend reports as `[deleted]` is dropped from `otherCalendars`
  — there is no name worth showing — but still counted in `syncCount`.

The four calendar-app callers have no equivalent disclosure.

The last three come from "an account with no imported calendars does not survive"
(see the calendar app's `CLAUDE.md`). Two things follow that are worth stating
plainly, because this cascade is destructive and two of its callers are now
unattended:

- **The sweep runs on mount only**, never on the refetch that follows an action.
  The connect flow refetches before its import picker opens, so a sweep wired to
  that refetch would cascade away the account the user just connected.
- **The sweep is scoped by `Account.createdAt`** against the client's
  `IMPORT_RULES_LIVE_FROM`. Accounts predating the rule are grandfathered and are
  never swept, so this cascade cannot reach accounts a user has been living with.

Neither unattended caller can reach an account with a live sync: a sync requires
an imported calendar, and both fire only when there are none.

---

## 2. ID cheat-sheet

| Symbol in code | Source | Used for |
|---|---|---|
| `accountId` (arg) | Mongo `_id` | Authz, SyncConfig/History keys, final Account delete |
| `googleAccountId` = `account.accountId` | Provider id | `Calendar.accountId`, Google Calendar client |
| `accountCalendarIds` | `Calendar.calendarId` list | Event deletes, BullMQ job filter, orphan sweep |
| SyncConfig `sourceAccountMongoId` / `targetAccountMongoId` | Mongo `_id` | SyncConfig/History delete + source-role blocker path |

Event deletion is **not** `deleteMany({ accountId })` even though Event has an `accountId` field — it keys off calendar ids and `extendedProperties.private.sourceCalendarId`.

---

## 3. Cascade order (as coded)

All steps run inside `AccountConnectionService.deleteAccount(accountId, userId)`.

```text
0.  findByIdAndUserId — abort if missing / wrong user
0b. refreshFromCoreAuth (best-effort) — so Google API calls can succeed when refreshToken is empty
1.  Prefetch calendars by provider accountId
2.  Remove BullMQ `initial-sync` jobs whose targetCalendarId ∈ owned calendars (best-effort)
3.  Stop Google webhook channels when channelId + resourceId present (best-effort)
4.  Account as SYNC TARGET: Google-native blocker sweep (`q: '[ BLOCKER ]'`) + eventRepo.deleteByCalendarId
5.  Account as SYNC SOURCE: find SyncConfigs → delete blockers on target calendars (Google + DB)
6.  deleteByCalendarIds(owned calendars)
6b. AC-3: deleteOrphansBySourceCalendarIds (Events elsewhere sourced from this account)
7.  syncConfigRepo.deleteByAccountId (both roles)
8.  syncHistoryRepo.deleteByAccountId (F-08 order: after Google + SyncConfig)
8b. syncGroupMetaRepo.deleteByTargetAccountId — added 2026-08-13, target role only (safe even when this account was only ever a source elsewhere, since the query only matches rows keyed by this account as target)
9.  calendarRepo.deleteByAccountId(provider id)
10. accountRepo.deleteByIdAndUserId — hard delete; throw if deletedCount === 0
```

### Step detail

| # | External / queue | Mongo | Failure handling |
|---|---|---|---|
| 0 / 0b | Core-auth token refresh | Load Account | Missing account throws; refresh errors swallowed |
| 2 | BullMQ queue `initial-sync` | — | Per-job / query warn; continue |
| 3 | `channels.stop` via `GoogleService.getCalendarClient` | — | Warn; continue |
| 4 | `events.list` / `events.delete` on owned calendars | `deleteByCalendarId` **always** after Google attempt | Google warn; DB still cleaned |
| 5 | `events.delete` on **target** account’s calendar | `deleteById` per blocker | Google warn; DB delete still attempted |
| 6 / 6b | — | Events by calendar id + orphan `sourceCalendarId` | Uncaught → aborts cascade |
| 7–9 | — | SyncConfig → SyncHistory → SyncGroupMeta (8b) → Calendars | Uncaught → aborts |
| 10 | — | Account hard delete | Throw `'Account deletion failed'` if `deletedCount === 0` |

Hard-delete throughout — no soft-delete / `deletedAt` on Account.

---

## 4. Authorization & scoping

- Every ownership check and the final delete require **`userId` match**.
- User A cannot delete user B’s account by guessing a Mongo `_id` → treated as not found (410 body).
- Google cleanup for **source-role** blockers uses the **target** account’s credentials (looked up via target calendar → account). That is intentional: blockers live on someone else’s calendar.

---

## 5. Atomicity

- **No Mongo multi-document transaction.**
- Best-effort sequential: Google / queue errors are logged and skipped; later Mongo steps still run.
- If a **non-caught** Mongo error throws mid-way, earlier Google deletions may already have happened while Account / SyncConfig / etc. remain.

| Fail after… | Typical residue |
|---|---|
| Steps 3–5 Google OK, Mongo throws | Google channels/blockers gone; Mongo still has rows |
| Steps 6–9 done, Account delete fails | Account row left; calendars/events/sync rows may already be gone |
| Step 2 queue query fails | Stale `initial-sync` jobs may still run |
| Poll schedulers never cleared (§7) | `poll:*` / `poll-target:*` keep firing after unlink |

Concurrent `stopSync` is considered in comments: SyncConfig may already be gone, which is why step 4 queries Google directly for `[ BLOCKER ]` instead of trusting SyncConfig alone. `SyncLifecycleService` keeps SyncConfig discoverable via `markAsStopping*` so unlink and stopSync can race without silently skipping Google cleanup.

---

## 6. What this is not

Unlink is **not** the same as `stopSync`:

| Concern | `deleteAccount` (unlink) | `stopSync` / `stopSyncByCalendar` |
|---|---|---|
| Removes Account document | Yes | No |
| Clears poll job schedulers (`poll:` / `poll-target:`) | **No** | Yes |
| Soft-marks SyncConfig stopping | No — hard deletes at step 7 | Yes, then teardown |
| Token revoke at Google | **No** | N/A |

---

## 7. Known gaps (still open)

These are product/code gaps relative to a fully safe unlink — keep them visible until closed:

| Gap | Impact |
|---|---|
| **Poll schedulers not cleared** | `SYNC_POLL_QUEUE` schedulers removed in `stopSync`, not here — can keep firing after unlink |
| **Only `initial-sync` BullMQ jobs removed** | Other queues (teardown, etc.) are not swept by target calendar id |
| **No OAuth token revoke** | `revokeGoogleAuth` / core-be revoke exist but are not called from `deleteAccount` — Google grant remains until user revokes in Google account settings |
| **Microsoft subscriptions** | No Graph subscription delete; `MicrosoftCalendarApiService.stopChannel` throws “not yet supported”; Google-oriented steps warn/skip for MS accounts |
| **Provider-agnostic API** | Channel stop uses `GoogleService.getCalendarClient` directly, not a provider registry |
| **Event.`accountId` unused** | Deletes key off calendar ids; fine today if calendars were always prefetched, but easy to misread from schema alone |
| **E2E tests use wrong id** | Some e2e specs DELETE with `account.accountId` (provider id) instead of `account.id` — does not match production FE or `findByIdAndUserId` |
| **Step 4's Google-native blocker search is a literal-string match that's now unreliable** | Added 2026-08-13: since a blocker's title/description are resolved per-sync from `SyncGroupMeta` toggles ([`calendar/SYNC-FLOW.md`](SYNC-FLOW.md) §10) rather than a fixed string, step 4's `events.list({ q: '[ BLOCKER ]' })` full-text search will not match a blocker whose `showTitle` toggle is on and whose title is therefore the real source event's summary. It still matches blockers created before this shipped (until self-healed by a later sync) and any blocker whose title toggle is off. Practical effect: an account deleted after this date may leave behind un-swept Google-side blocker events on calendars it was the sync **target** for, silently — the DB-side `eventRepo.deleteByCalendarId` at the same step still runs regardless and cleans the local record either way, so this is a Google-side residue gap, not a DB-consistency one. Not fixed as part of shipping the toggle feature — the correct fix is switching this query to `extendedProperties.private.isSource = 'true'`, the same signal every other blocker lookup in this codebase already treats as authoritative. |

Unit coverage (`account-connection.service.spec.ts`, TEST-U-004) locks the target-calendar blocker sweep and error resilience. It does **not** assert BullMQ removal, `channels.stop`, source-role path, SyncConfig/History/Calendar deletes, AC-3, or cross-user authz.

---

## 8. Compared to the original inventory wishlist

Original DOC-A08 ideal order:

1. Stop BullMQ sync jobs  
2. Stop Google webhook channels  
3. Delete Events where `accountId = X`  
4. Delete Calendars  
5. Delete SyncConfig  
6. Delete SyncHistory  
7. Delete Account  

**Actual code is richer and differently ordered:**

- Adds refreshFromCoreAuth, Google-native target blocker sweep, source-role remote blocker cleanup, AC-3 orphan sweep.
- Deletes SyncConfig → SyncHistory → Calendars → Account (calendars **after** sync rows, not before).
- Event delete filters differ from `accountId = X`.
- Still missing poll-scheduler clear, full queue sweep, token revoke, Microsoft cleanup.

Historical `P1-3` / ephemeral `task.md` references in older inventory notes are not present as open TASK_SHEET items; related BUG/TEST/RACE items around stopSync orphans and Google-native sweep are marked Done. Treat §7 as the live gap list.

---

## 9. Failure modes (operator view)

| Symptom | Check |
|---|---|
| Unlink returns “does not exist” | Client sent provider `accountId`/email instead of Mongo `id` |
| Account gone but Google still has blockers | Google API failed mid-step 4/5 (warnings in calendar-be logs); re-run manual cleanup or re-auth was already impossible |
| Sync still polling after unlink | Poll schedulers not cleared (§7) — remove `poll:*` / `poll-target:*` via stopSync path or Redis/BullMQ admin |
| Channels still open in Google | Step 3 warn — channel fields missing or `channels.stop` failed |
| Decrypt / Google 401 during unlink | refreshFromCoreAuth failed and account had no usable refresh token |

Google Calendar API has no circuit breaker in this path (see `docs/failure-modes.md`) — unlink Google calls can hang or fail under quota independently of other breakers.

---

## Related docs

| Doc | Why |
|---|---|
| [`GOOGLE-OAUTH.md`](GOOGLE-OAUTH.md) | How the Account row was created before unlink |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | Module map, `SyncLifecycleService` teardown mechanics, data model |
| [`SYNC-FLOW.md`](SYNC-FLOW.md) §8, §10 | Full teardown algorithm — `enqueueTeardown`/`executeTeardown`, retry layers, `stopSync`'s non-queued equivalent; §10 covers the `SyncGroupMeta` cleanup rules step 8b follows |
| [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5.7 | `SyncGroupMeta` schema — what step 8b actually deletes |
| [`docs/failure-modes.md`](../../../../apps/arm-app-calendar/docs/failure-modes.md) | Dependency outages affecting Google calls during cascade |
| `SyncLifecycleService.stopSync` | Poll-scheduler cleanup that unlink does **not** do |
| Future `WEBHOOKS.md` (DOC-A07) | Channel registration / renewal that step 3 tears down |
| [ENV-VARS.md](../ENV-VARS.md) | `TRUST_PROXY_HEADERS`, Google credentials used during cleanup |
