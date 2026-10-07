# Calendar App — Sync Flow

Algorithm-level reference for how a calendar sync actually runs, end to end — initial sync, ongoing incremental updates, cross-target migration, and teardown. [ARCHITECTURE.md](ARCHITECTURE.md) §3 covers *which service* owns each trigger path (`SyncOrchestratorService`, `WebhookProcessorService`, `sync-poll.processor.ts`); this document covers *what each one actually does*, step by step, verified against `src/backend/src/` directly.

All line numbers below refer to `src/backend/src/` files as of 2026-08-13 and will drift as the code changes — treat them as pointers, not guarantees. §10 was added 2026-08-13 to cover per-sync field-visibility toggles, the shared create-or-update blocker logic, and the Resync capability — everything else was re-verified against source at the same time, not just carried forward.

---

## 1. Initial sync — `sync-orchestrator.service.ts`, `performInitialSync()`

Runs as a `SYNC_QUEUE` (`'initial-sync'`) BullMQ job, enqueued by `queueInitialSync()` after it synchronously writes `SyncConfig` + a `PENDING` `SyncHistory` row (so `getActiveSyncs` is consistent immediately, before the worker even picks the job up).

Per source account → per source calendar:

1. **`PENDING` → `IN_PROGRESS`** on `SyncHistory` as soon as the worker starts that calendar.
2. **Load this target's field-visibility toggles** — one `SyncGroupMetaRepository.findByTargetCalendarId(targetCalendarId)` lookup per source calendar, defaulting `showTitle`/`showDescription`/`showLocation` to `true` when no `SyncGroupMeta` row exists (pre-migration syncs, or a target that's never had its name/toggles edited). See §10 for what these toggles actually control.
3. **Page through source events** — `do { … } while (pageToken)`, page size hardcoded to **2500** (Google `maxResults` / Microsoft `Prefer: (secret removed)`). The sync token from the *last* page is persisted to `Calendar.syncToken` — this is what the webhook/poll incremental path reads later. A listing error on any page fails that calendar (`SyncHistory` → `FAILED`) and moves to the next one; it does not abort the whole job.
4. **Cancellation re-check every 10 events** — the loop polls `syncConfigRepo.existsByPair` periodically so a mid-run `stopSync` call actually stops the run, not just future runs.
5. **Per-event filtering** — skip if: `status === 'cancelled'`; `extendedProperties.private.isSource === 'true'` or `summary === '[ BLOCKER ]'` (the loop-guard — prevents re-syncing a blocker that itself appears in the source listing; the summary check is an explicit fallback for blockers created before `extendedProperties` was enforced, and for blockers whose title toggle is off and so never carries real content in the first place); no `id`.
6. **Blocker create-or-update** — `findBlockerByEventId` locates any existing blocker, then both branches (new and already-synced) go through the same `BlockerService.createOrUpdateBlocker()` call — see §10 for exactly how title/description/location are resolved from this target's toggles. **This is a real behavior change from the original design**: initial sync used to only ever create missing blockers and silently skip ones that already existed; it now also diffs an existing blocker's stored content/time against freshly-resolved content and updates it if either drifted — which is what makes Resync (§10) and a toggle edit actually take effect on events synced before the change. `transparency: 'opaque'` and `extendedProperties.private: {isSource: 'true', sourceCalendarId}` are set on every create/update — that tag is the traceback link every `findBlockerByEventId` lookup depends on. **Recurring events are not expanded** — the source event's `recurrence` (RRULE strings) is passed through unmodified on both create and update, so one blocker event covers the whole series. Per-event failures increment `blockersFailed` and are recorded in `failureReasons`; they do not throw or abort the run. An unchanged blocker (content and time both already match) makes no Google/Graph API call at all and does not increment `eventsSynced`.
7. **F-21 stale-blocker sweep** — after the full page-through, any DB-tracked blocker for this `(target, source)` pair whose source `eventId` wasn't seen in this run's live set gets deleted (best-effort from Google, unconditionally from the DB). See §6 for how this differs from the incremental path's cleanup.
8. **`SyncHistory` terminal status** — `blockersFailed === 0 ? COMPLETED : (eventsSynced > 0 ? PARTIAL : FAILED)`.
9. **Webhook or poll registration, per source calendar** — skipped entirely (no re-registration attempt at all) if the source calendar's `Calendar` document already has a `channelId` — this guard exists specifically so Resync (§10), which re-runs this exact algorithm against already-configured sources, doesn't leak a duplicate webhook channel on every resync. Otherwise, a webhook is attempted only if `accessRole` is `owner`/`writer`; on `ForbiddenException`/403/"not supported" (the path Microsoft's `watchEvents` always throws through) it falls back to a 5-minute poll job scheduler instead.
10. **Target-calendar webhook (NR-01)** — after the whole source loop, if the target calendar has no webhook channel yet, one is registered on it too (or, on failure, a `poll-target:{targetCalendarId}` scheduler) — this is what detects and restores manually-tampered/deleted blockers on the target side.

---

## 2. BullMQ queue configuration

| Queue | Attempts | Backoff | Notes |
|---|---|---|---|
| `initial-sync` | **none — BullMQ default of 1** | none | See callout below. |
| `sync-poll` | none | none — repeatable every 5 min | Failure just means the next 5-minute tick retries implicitly. |
| `sync-teardown` | 10 | exponential, 60s base, jitter | See §8. |
| `webhook-processing` | 3 | exponential, 5s base, jitter | Inbound Google push notifications. |
| `webhook-renewal` | 3 | exponential, 30s base, jitter | Daily Google channel renewal. |
| `microsoft-delta-renewal` | 3 | exponential, 30s base, jitter | Monthly Microsoft full resync. |

**`initial-sync` jobs have no retry configuration at all.** BullMQ treats an unset `attempts` as 1 — so a job that throws at the top level is immediately moved to the dead-letter queue (`dlq.service.ts` reads `job.opts.attempts ?? 1` as the threshold), zero retries. This matters less than it sounds, because `performInitialSync` is written to swallow *partial* failure internally — per-calendar and per-event errors are caught and recorded in `SyncHistory`/`failureReasons` rather than thrown. A DLQ'd `initial-sync` job only happens on a whole-job-level exception thrown before those inner try/catches even engage (e.g. a malformed DTO). Still worth knowing if you're debugging a sync that silently never started: check the `dead-letter` queue, not just `SyncHistory`.

---

## 3. `Event.accountId` backfill migration

`StartupMigrationService.backfillEventAccountId()` runs on **every application boot**, not once-ever behind a flag. Its own completion check *is* its idempotency mechanism: it queries `Event.find({accountId: {$exists: false}})` first and returns immediately if that's empty — no separate migration-ledger collection or version marker exists. When there is work to do, it joins `Calendar.accountId` (the provider's own account ID string) → `Account.accountId` (same field) → `Account._id` (the Mongo ObjectId actually written to `Event.accountId`), one `updateMany` per distinct `calendarId`. A `Calendar` or `Account` doc that's gone missing for a given `calendarId` leaves those `Event` rows permanently un-backfilled — a real but narrow edge case, not a crash.

---

## 4. Sync status lifecycle

`SyncStatus`: `pending → in-progress → (completed | partial | failed)`. The schema enforces at most one `pending` row per `(sourceCalendarId, targetCalendarId)` pair via a **partial unique index** scoped to `syncStatus: 'pending'` — any number of historical non-pending rows for the same pair can coexist.

Write sites:

| Transition | Where |
|---|---|
| → `pending` | `queueInitialSync`, synchronously, before the BullMQ job is enqueued |
| → `in-progress` | `performInitialSync`, as the worker picks up each source calendar |
| → `failed` (early) | source account missing/not owned; `listEvents` threw |
| → `completed` / `partial` / `failed` (terminal) | end of `performInitialSync`, based on `blockersFailed`/`eventsSynced` |
| → `failed` (delta path) | `applyDeltaEvents`, only when a whole sync-target's processing rejected (not per-event errors) |

**Ongoing webhook/poll incremental updates do not create new `SyncHistory` rows on success.** `applyDeltaEvents` — the function shared by `handleWebhook`, `pollCalendar`, and `fullyResyncCalendar` — only writes to `SyncHistory` in its failure branch. A normal delta run (creates, updates, deletes, out-of-window removals, stale-blocker sweeps included) produces **zero** `SyncHistory` documents; the only trail is application logs and the mutated `Event` rows themselves. If you're building an "audit trail of sync activity" view, `SyncHistory` alone under-represents actual sync volume — it's an initial-sync log plus a delta-failure log, not a complete activity log.

---

## 5. Quota / rate-limit handling — an asymmetry between providers

**Google**: `isQuotaError()` in `calendar/google-calendar-api.service.ts` treats a `403` whose message contains `ratelimitexceeded` or `quota exceeded` as equivalent to a `429` (Google returns quota exhaustion as 403 with a reason string, not as a literal 429). This 403→429 normalization is wired into `deleteEvent` only, not `createEvent`/`updateEvent`/`listEvents` in that file — those fall through to a generic `BadRequestException` on any error, quota or not.

**The actual retry-with-backoff lives in `events/blocker.service.ts`, not in either API service.** `deleteBlockerWithRetry()`: up to 3 attempts, exponential delay **1s / 2s / 4s**, triggered only on an `HttpException` with status `429`; a `NotFoundException` is treated as already-deleted (success); any other error type gives up on the first attempt. `deleteBlockersBatched()` wraps this per blocker with a flat 300ms delay between items — this is the path `stopSync`, teardown, and cross-target migration all use for bulk blocker deletion.

**Microsoft has no quota-detection equivalent at all.** `calendar/microsoft-calendar-api.service.ts` has no `isQuotaError`-style function — a real Graph `429 Too Many Requests` falls straight into a generic `BadRequestException`, which never becomes the `HttpException(429)` shape `deleteBlockerWithRetry` looks for. In practice: a Microsoft rate-limit hit during blocker deletion is treated as a normal, non-retryable failure — one attempt, then give up — while the equivalent Google failure gets 3 attempts with backoff. Worth knowing before assuming both providers get equal resilience.

---

## 6. Stale blocker sweep ("F-21") vs. delta cancellations ("NR-06")

Two separate mechanisms handle blocker removal, and they trigger differently:

- **F-21 (`performInitialSync`, inline)** — runs **every initial-sync pass**, once per source calendar, unconditionally, right after the per-event blocker loop. "Stale" = a DB-tracked blocker whose source `eventId` wasn't in this run's freshly-fetched live set.
- **NR-06 (`WebhookProcessorService.sweepStaleBlockers`)** — the *same underlying query*, but only triggered as a side effect of a **full re-fetch**, not on every webhook/poll tick: when a stored `syncToken` is invalidated (Google `410 Gone`) and the service has to fall back to a complete re-listing, or when Microsoft's monthly delta-chain renewal forces one. The comment on this path explains why it's needed separately from delta processing: "on a full re-fetch, delete blockers for source events that silently disappeared — no cancellation signal was sent."

**A normal incremental delta cycle that doesn't hit a `410` never runs either sweep.** Deletions there rely entirely on the provider sending an explicit `cancelled`/`isCancelled` tombstone, handled inline in `applyDeltaEvents` (§9). If a source event disappears from a provider without ever sending a cancellation event — which does happen — the orphaned blocker survives until the next full re-fetch (a `410`) or the next initial-sync re-run picks it up via F-21.

---

## 7. Cross-target migration ("F-32") — `sync-lifecycle.service.ts`, `updateSyncConfig()`

Triggered when a `SyncConfig` update maps a source calendar that was previously synced to target A onto a new target B. **This is delete-and-recreate, not a move** — no blocker document or Google event is ever transferred between targets:

1. Refresh the old target account's token.
2. Delete every migrated-source blocker from Google (batched, 3-attempt/exponential-backoff-per-item — §5's `deleteBlockersBatched`).
3. **Orphan cleanup only runs if step 2 had zero failures** — if quota was still exhausted mid-delete, the remaining DB records are deliberately left as the recovery path for a future run, rather than risking a second pass compounding the failure.
4. Delete the confirmed-gone blockers from the local DB, stop the old target's webhook channels, delete `SyncHistory` for the old target/source pairs, delete the old `SyncConfig` rows — then, only if the old target has **zero** remaining `SyncConfig` rows after that delete (it may still have other, non-migrated sources), also delete its `SyncGroupMeta` row (§10). The new target's own name/toggles are untouched by this — a migration doesn't carry the old target's `SyncGroupMeta` over.
5. **Fall through into a normal `performInitialSync` call against the new target** — the "migration" is, mechanically, a teardown of the old target followed by a fresh initial sync against the new one, using the exact algorithm in §1.

---

## 8. Teardown flow — `sync-lifecycle.service.ts`

`enqueueTeardown()` first **closes the gate synchronously**: marks the `SyncConfig` row as "stopping" (webhook/poll handlers start skipping this calendar immediately) before enqueueing anything, then queues onto `sync-teardown` with 10 attempts / 60s-exponential backoff.

`executeTeardown()` (run per BullMQ attempt) is written to be **idempotent and retry-safe**:

1. Re-reads remaining `SyncConfig` rows fresh on every attempt — if a prior attempt already finished, this list is empty and the job no-ops immediately.
2. Deletes remaining tracked blockers from Google (batched delete, same as §5/§7).
3. Removes only the *confirmed-deleted* blockers from the DB — failed ones stay, so the next retry picks them up automatically.
4. **If any deletion failed, the whole job throws** — this is what drives the BullMQ-level retry (10 attempts, 60s exponential), on the theory that quota recovers given time.
5. Only after a clean pass: orphan cleanup (best-effort), webhook channel teardown, poll-scheduler cancellation (skipping sources still shared by another active sync), and finally `SyncConfig`/`SyncHistory`/`SyncGroupMeta` (§10) row deletion — deliberately last, "Google is clean at this point." Unlike F-32's guarded delete (§7), this one is unconditional — `executeTeardown` only ever runs against a target whose `SyncConfig` rows are all being removed, so there's no "other sources remain" case to guard against.

**Two nested retry layers apply**: per-blocker-delete (3 attempts, 1s/2s/4s, inside `deleteBlockerWithRetry`) nested inside per-teardown-attempt (10 attempts, 60s-exponential, at the BullMQ layer) — a genuinely resilient design for a operation that has to clean up an external API's state.

A separate, **synchronous, non-queued** path — `stopSync()` — exists for account-deletion and legacy flows. It runs the same Google-delete/webhook-stop/poll-cancel sequence inline, with only the per-item retry (no job-level retry), so it has no protection against a whole-operation-level failure the way `executeTeardown` does. It also unconditionally deletes every `SyncGroupMeta` row for the account being stopped (`deleteByTargetAccountId`), same as `executeTeardown`.

---

## 9. Delta processing (webhook- and poll-triggered) — `WebhookProcessorService.applyDeltaEvents()`

Shared by `handleWebhook`, `pollCalendar`, and `fullyResyncCalendar`. Fans each delta event out across every active `SyncConfig` for that source calendar (`Promise.allSettled`, so one target's error doesn't block another target sharing the same source), then per event per target:

1. **Loop-guard** — same `isSource`/`[ BLOCKER ]` filter as initial sync, so a blocker event appearing in its own source listing doesn't get re-processed.
2. **Cancelled event** → look up the matching blocker via `findBlockerByEventId(event.id, targetCalendarId, sourceCalendarId)` — this `(source eventId, target, source)` triple is the linking key used everywhere in this document — and delete it from both Google and the DB if found.
3. **Out-of-window reschedule** — a non-cancelled event whose new date falls before `SyncConfig.syncFromDate` is treated as removed from scope: existing blocker (if any) is deleted, no update is made.
4. **Create-or-update** — `findBlockerByEventId` locates any existing blocker, this target's toggles are loaded from `SyncGroupMeta` (same default-all-true-when-absent rule as §1 step 2), then the event is handed to the exact same `BlockerService.createOrUpdateBlocker()` initial sync uses (§1 step 6, §10) — this path used to duplicate the create/update/diff logic inline with its own separate description-comparison, it no longer does. Recurring events still pass `recurrence` through unmodified on both create and update.
5. **Failure surfacing** — only a rejected top-level `Promise.allSettled` entry (e.g. target account not found) writes a `SyncHistory` `failed` row (§4); per-event errors inside a target's processing are caught and logged, not thrown.
6. **SSE notification** — one `sseRedis.publish(userId, {sourceCalendarId})` per distinct affected user, deduped so multiple targets fed by the same source calendar only trigger one push to the browser.

---

## 10. Field-visibility toggles, blocker-content resolution, and Resync

A `SyncGroupMeta` document (one per `targetCalendarId`, `events/sync-group-meta.schema.ts`) stores a sync's user-facing `name` plus three booleans — `showTitle`, `showDescription`, `showLocation` — all defaulting to `true`. It is looked up (not written) every time a blocker is created or updated (§1 step 2, §9 step 4); a missing row (no edit has ever touched this target) behaves identically to a row with every toggle `true`.

**`resolveBlockerContent()`** (`events/blocker-content.util.ts`, a pure function with no DB access) is the single place title/description/location are derived from a source event plus this target's toggles:

- A source event whose provider-reported visibility is `private` or `confidential` (Google's `visibility` field; Microsoft's `sensitivity`, normalized onto the same vocabulary in `microsoft-calendar-api.service.ts`) **always** resolves to `summary: 'Busy'`, empty description, no location — **regardless of toggle state**. This overrides everything else and cannot be toggled off.
- Otherwise: `showTitle` on and the source event has a summary → the real summary; off, or the source event has none → `'Busy'`. `showDescription`/`showLocation` on → the real value (or empty/`undefined` if the source event has none); off → empty/`undefined` unconditionally.
- The old fixed `'[ BLOCKER ]'` title and 4-line `=== DO NOT EDIT ===` description are gone entirely — no code path writes either string to a new or updated blocker anymore. See the blast-radius note below for what that means for blockers created before this shipped.

**`BlockerService.createOrUpdateBlocker()`** (`events/blocker.service.ts`) is the shared function §1 step 6 and §9 step 4 both call — it resolves content via the function above, then either creates a new blocker or diffs the resolved content *and* start/end time against the existing blocker's stored values, updating only on a real difference (an unchanged blocker makes no external API call). This single function replaced two independently-hand-written create/update implementations that used to live in `sync-orchestrator.service.ts` and `webhook-processor.service.ts` separately.

**Resync** — `POST /events/:targetCalendarId/sync/resync` (`SyncOrchestratorService.queueResync`) — re-runs §1's exact algorithm against a target's already-configured sources, without asking the user for anything (the route takes no body): it groups the target's `SyncConfig` rows by each source's own stored `syncFromDate` (sources configured with different backfill dates get separate BullMQ jobs, since a single job takes one `syncFromDate`), then enqueues one `performInitialSync` job per group. Guards, checked before enqueueing anything:

| Condition | Result |
|---|---|
| No non-`stoppingAt` `SyncConfig` rows exist for this target | `404 NotFoundException` |
| Caller doesn't own the target account | `403 ForbiddenException` |
| A `SyncHistory` row for this target is already `pending`/`in-progress` | `409 ConflictException` — refuses to enqueue a second overlapping run |

Because §1 step 6 now diffs-and-updates rather than only creating missing blockers, a Resync genuinely refreshes every already-synced event's content — this is the mechanism that makes Resync (and a toggle/name edit via `POST /events/:targetCalendarId/sync/update`, which persists to `SyncGroupMeta` unconditionally even when the source-calendar set itself is unchanged) actually take visible effect, rather than only affecting events synced afterward.

**Blast-radius note, worth knowing before assuming "nothing changed here":** because default toggles are all-`true` and no `SyncGroupMeta` row is required for that default to apply, **every blocker created before this capability shipped** — which all carry the old `'[ BLOCKER ]'` title and 4-line description — will have that content silently rewritten to the real source event's title/description/location the next time it's touched: a webhook delta on its source event, or a manual Resync. This is intentional self-healing toward the new default behavior, not a bug, but it is a real one-time content change to already-synced data across every existing sync, not just new ones.

**A side effect worth flagging, not fixed as part of this change**: `AccountConnectionService.deleteAccount`'s target-role Google-native blocker sweep (see [DELETE-CASCADE.md](DELETE-CASCADE.md) §3 step 4) finds blockers to delete via a literal Google full-text search, `q: '[ BLOCKER ]'`. That query still matches every blocker created before this capability shipped (until self-healed per the note above) and any blocker whose `showTitle` toggle is off, but will **not** match a blocker whose real title no longer contains that literal string — see [DELETE-CASCADE.md](DELETE-CASCADE.md) §7 for the full gap description.

---

## 11. Notable findings

- **Initial-sync BullMQ jobs have zero retry/backoff** (§2) — a whole-job-level exception is a single-shot failure straight to the dead-letter queue, unlike every other queue in this service.
- **`SyncHistory` is not a complete activity log** (§4) — successful incremental (webhook/poll) updates never write to it; only initial-sync runs and delta-path *failures* do.
- **Rate-limit resilience is asymmetric between providers** (§5) — Google blocker deletes get 3-attempt exponential backoff on quota errors; Microsoft has no quota-error detection at all, so an equivalent Graph `429` is treated as a plain non-retryable failure.
- **Cross-target migration is delete-and-recreate** (§7), not a data move — every blocker on the new target is a fresh `createEvent`, never copied or referenced from the old target's blockers.
- **Two independent stale-blocker cleanup paths exist** (§6) with different trigger conditions (every initial sync, vs. only on a full re-fetch after token invalidation) — a source event deleted without an explicit provider cancellation event can persist as an orphaned blocker until one of those two paths runs.
- **Recurring events are never expanded into instances anywhere in the sync engine** — both initial sync and delta processing pass the source `recurrence` (RRULE) array straight through to the blocker's create/update call, for both create and update paths.
- **Blocker content is no longer hardcoded** (§10) — title/description/location are resolved per-target from `SyncGroupMeta` toggles, with private/confidential source events always forced to a bare `'Busy'` regardless of toggle state; the two independent inline create/update implementations that used to exist (initial sync, webhook delta) are now one shared function, `BlockerService.createOrUpdateBlocker()`.
- **A sync can now be manually refreshed** (§10) — `POST /events/:targetCalendarId/sync/resync` re-runs initial sync against already-configured sources, guarded against a concurrent teardown and against overlapping runs for the same target; this is a genuine content refresh, not just a re-scan for new events, because initial sync's create-only-if-missing behavior was replaced with create-or-update.

---

## Related documents

- [ARCHITECTURE.md](ARCHITECTURE.md) — module map, which service owns each trigger path, renewal cron mechanics
- [WEBHOOKS.md](WEBHOOKS.md) — channel registration/intake/renewal detail behind §5's 410 handling and §9's delta-processing entry point
- [GOOGLE-OAUTH.md](GOOGLE-OAUTH.md) — how an `Account` gets the token this whole sync engine depends on
- [DELETE-CASCADE.md](DELETE-CASCADE.md) — what happens to sync state on account unlink (a superset of the teardown flow in §8), and the §10-referenced Google-search gap
- [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5 — `SyncConfig`/`SyncHistory`/`Event`/`SyncGroupMeta` schema field tables
- [`troubleshooting/calendar-sync-not-working.md`](../troubleshooting/calendar-sync-not-working.md) — symptom-first triage built on this document, including how to trigger a Resync (§8)
