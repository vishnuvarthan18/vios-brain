# Phase 4: Sync Engine Hardening

> Fix data integrity bugs in the calendar sync engine. These bugs cause silent data loss, orphaned events, and retry storms under load.

**Priority:** P2 — Do before external users run real calendar syncs
**Effort:** ~10 hours total
**Prerequisite:** Phase 2 complete (security bugs fixed first)
**Source:** Production Audit — Sync Engine + Account/Google/Microsoft sections

---

## Critical Bugs Addressed

| Bug ID | Issue | Task |
|---|---|---|
| C-13 | Race condition on teardown stoppingAt gate | T-4.1 |
| C-14 | Stale blocker sweep incomplete on large calendars | T-4.2 |
| C-15 | Orphaned blockers created during cancellation window | T-4.3 |

## High Bugs Addressed

| Bug ID | Issue | Task |
|---|---|---|
| H-06 | Missing stoppingAt filter on findBySourceCalendarIds | T-4.4 |
| H-07 | Creates blockers for targets mid-teardown | T-4.4 |
| H-08 | Redis lock TTL 600s — crash = 10 min poll starvation | T-4.5 |
| H-17 | Null tokenExpiryAt blocks refresh | T-4.6 |
| H-18 | SESSION_INTERNAL_SECRET falls back to empty string | T-4.7 |
| H-19 | Google orphan sweep searches inert BLOCKER title | T-4.8 |
| H-20 | Microsoft orphan sweep searches inert BLOCKER title | T-4.8 |
| H-23 | Partial deletion throws — BullMQ retry storm | T-4.9 |
| H-24 | No exponential backoff on Google 429 | T-4.10 |
| H-25 | Blindly recreates blockers for all syncs on target | T-4.11 |
| H-26 | No notification timestamp validation | T-4.12 |
| H-27 | Empty filtered list deletes ALL calendars | T-4.13 |

## Medium Bugs Addressed

| Bug ID | Issue | Task |
|---|---|---|
| M-14 | No backoff on duplicate sync detection | T-4.14 |
| M-15 | Full resync on syncToken invalid loads all events | T-4.14 |
| M-16 | Mongo delete BEFORE Google delete — orphans | T-4.15 |
| M-17 | $or query without composite index | T-4.14 |
| M-18 | Unbounded event pagination in fullyResyncCalendar | T-4.14 |
| M-19 | JSON parse error swallowed — frontend never notified | T-4.16 |
| M-20 | SSE publish failure swallowed | T-4.16 |
| M-21 | Auto-removed jobs bypass DLQ | T-4.16 |

---

## Tasks

### T-4.1: Fix race condition on teardown stoppingAt gate

**Source:** C-13
**File:** `apps/arm-app-calendar/src/backend/src/events/sync-lifecycle.service.ts:498`
**Function:** `executeTeardown()`
**Bug:** Checks `stoppingAt` is set but does not acquire a lock. Concurrent teardowns cause duplicate deletion attempts.

- [ ] Add a Redis distributed lock keyed by syncConfigId before starting teardown
- [ ] Use a short TTL (e.g., 60s) so a crash doesn't block teardown forever
- [ ] If lock not acquired, skip (another instance is already tearing down)
- [ ] Release lock on completion (or let TTL expire on crash)
- [ ] Add test: two concurrent teardown calls — only one executes

**Effort:** 45 minutes

---

### T-4.2: Fix stale blocker sweep to handle pagination

**Source:** C-14
**File:** `apps/arm-app-calendar/src/backend/src/events/webhook-processor.service.ts:275`
**Function:** `sweepStaleBlockers()`
**Bug:** Assumes `currentEventIds` is complete, but doesn't verify pagination was fully consumed. >2500 events = incomplete sweep.

- [ ] Add pagination loop to fully consume all pages of events before comparing IDs
- [ ] Or: use the stored syncToken/deltaLink to fetch only changes (already handles pagination correctly)
- [ ] Add a safeguard: if total events exceed a threshold (e.g., 10000), log a warning and skip sweep (fallback to incremental reconciliation)
- [ ] Add test: calendar with >2500 events correctly sweeps stale blockers

**Effort:** 45 minutes

---

### T-4.3: Reduce orphaned blocker window during cancellation

**Source:** C-15
**File:** `apps/arm-app-calendar/src/backend/src/events/sync-orchestrator.service.ts:385`
**Function:** `performInitialSync()`
**Bug:** Cancellation checked every 10 events. With slow blocker creation (5s each), 50 seconds of orphans.

- [ ] Check cancellation every 1 event (not every 10)
- [ ] Add a flag in BullMQ job data that the job can check between operations
- [ ] On cancellation detected: clean up any blockers created in this batch before returning
- [ ] Add test: cancellation during sync cleans up partial blockers

**Effort:** 45 minutes

---

### T-4.4: Add stoppingAt filter to findBySourceCalendarIds

**Source:** H-06, H-07
**File:** `apps/arm-app-calendar/src/backend/src/events/sync-config.repository.ts:64`
**Function:** `findBySourceCalendarIds()`
**Bug:** Missing `stoppingAt` filter. Webhooks are processed for syncs being torn down, creating blockers on targets mid-teardown.

- [ ] Add `stoppingAt: { $exists: false }` (or `stoppingAt: null`) to the query filter
- [ ] Match the same filter pattern used by other repository methods
- [ ] Add test: sync with stoppingAt set is excluded from webhook processing

**Effort:** 30 minutes

---

### T-4.5: Reduce Redis lock TTL on poll processor

**Source:** H-08
**File:** `apps/arm-app-calendar/src/backend/src/events/sync-poll-processor.ts:48`
**Function:** `process()`
**Bug:** Redis lock TTL is 600s (10 min). If the process crashes, no other instance can poll for 10 minutes.

- [ ] Reduce TTL to 120s (2 minutes) — polls should complete well within this
- [ ] Add lock extension for long-running polls (touch the lock every 30s while still processing)
- [ ] Add test: crashed poll doesn't block subsequent polls for more than 2 minutes

**Effort:** 30 minutes

---

### T-4.6: Handle null tokenExpiryAt in Google refresh

**Source:** H-17
**File:** `apps/arm-app-calendar/src/backend/src/google/google-client.factory.ts:177`
**Function:** `refreshTokenIfExpiredForAccount()`
**Bug:** Null `tokenExpiryAt` blocks refresh — the self-healing path is broken.

- [ ] If `tokenExpiryAt` is null or undefined, treat the token as expired (force refresh)
- [ ] `if (!tokenExpiryAt || tokenExpiryAt < new Date()) { /* refresh */ }`
- [ ] Add test: account with null tokenExpiryAt triggers refresh

**Effort:** 15 minutes

---

### T-4.7: Fix SESSION_INTERNAL_SECRET empty string fallback

**Source:** H-18
**File:** `apps/arm-app-calendar/src/backend/src/common/core-auth/core-auth.service.ts:41`
**Bug:** `SESSION_INTERNAL_SECRET` falls back to empty string when not set. Empty string passes comparison.

- [ ] Throw at startup if `SESSION_INTERNAL_SECRET` is empty or undefined
- [ ] Add to env validation in `common/config/env.ts`
- [ ] Add test: empty SESSION_INTERNAL_SECRET throws on module init

**Effort:** 15 minutes

---

### T-4.8: Fix orphan sweep to identify by extended property, not title

**Source:** H-19, H-20 (references CLAUDE.md "Blocker identification — a live trap")
**Files:**
- `apps/arm-app-calendar/src/backend/src/calendar/google-calendar-api.service.ts:465` (`listBlockerEvents`)
- `apps/arm-app-calendar/src/backend/src/calendar/microsoft-calendar-api.service.ts:291` (`listBlockerEvents`)
**Bug:** Both search for `[ BLOCKER ]` title. Nothing writes that title anymore. Orphan sweep is permanently inert.

- [ ] **Google:** Use `privateExtendedProperty=isSource=true` parameter on `events.list`
- [ ] **Microsoft:** Use `$filter` on `singleValueExtendedProperties/Any(ep: ep/id eq '...' and ep/value eq 'true')` or equivalent
- [ ] Remove the title-based search entirely
- [ ] Do NOT reinstate a title prefix — the title is user-facing and controlled by toggles
- [ ] Add test: blocker events with `Busy` title and `isSource=true` property are found

**Effort:** 1 hour

---

### T-4.9: Fix partial deletion retry storm in executeTeardown

**Source:** H-23
**File:** `apps/arm-app-calendar/src/backend/src/events/sync-lifecycle.service.ts:533`
**Function:** `executeTeardown()`
**Bug:** Partial deletion throws. BullMQ retries the entire job, re-deleting already-deleted blockers = storm of 404s.

- [ ] Track which blockers have been successfully deleted (e.g., mark them in Mongo)
- [ ] On retry, skip already-deleted blockers
- [ ] Or: use `Promise.allSettled()` and only retry the failures
- [ ] Add test: partial deletion failure retries only the failed blockers

**Effort:** 45 minutes

---

### T-4.10: Add exponential backoff on Google API 429

**Source:** H-24, Failure Analysis Scenario 6
**File:** `apps/arm-app-calendar/src/backend/src/events/sync-orchestrator.service.ts:395`
**Function:** `performInitialSync()` — event listing loop
**Risk addressed:** 20 users with 5 calendars = 10K+ API calls in burst. Google returns 429. No backoff = quota exhausted for 24 hours.

- [ ] Add exponential backoff when Google returns 429:
  ```typescript
  // Start at 1s, double each retry, max 5 retries (1s, 2s, 4s, 8s, 16s)
  const delay = Math.min(1000 * Math.pow(2, retryCount), 16000);
  await new Promise(resolve => setTimeout(resolve, delay));
  ```
- [ ] Apply to all Google Calendar API calls (events.list, events.insert, events.update, events.delete)
- [ ] Also apply to Microsoft Graph API calls (same pattern)
- [ ] Add test: 429 response triggers backoff, 3 retries then fails

**Effort:** 1 hour

---

### T-4.11: Fix blind blocker recreation on target webhook

**Source:** H-25
**File:** `apps/arm-app-calendar/src/backend/src/events/webhook-processor.service.ts:650`
**Function:** `handleTargetWebhook()`
**Bug:** Blindly recreates blockers for ALL syncs targeting this calendar. Should only recreate the blocker that was tampered with.

- [ ] Check the specific event that changed (from the webhook notification)
- [ ] Only recreate if the changed event matches a known blocker (by `uniqueId` or extended property)
- [ ] Skip recreation for events that aren't our blockers
- [ ] Add test: non-blocker event change on target doesn't trigger recreation

**Effort:** 45 minutes

---

### T-4.12: Validate notification timestamp in handleWebhook

**Source:** H-26
**File:** `apps/arm-app-calendar/src/backend/src/events/webhook-processor.service.ts:160`
**Function:** `handleWebhook()`
**Bug:** No notification timestamp validation. Stale/replayed webhooks apply outdated events.

- [ ] Check the notification timestamp against the last processed timestamp for that channel
- [ ] Skip if the notification is older than the last processed one
- [ ] Store last processed timestamp per channel in the Calendar document
- [ ] Add test: stale notification with old timestamp is skipped

**Effort:** 30 minutes

---

### T-4.13: Guard against empty filtered list deleting ALL calendars

**Source:** H-27
**File:** `apps/arm-app-calendar/src/backend/src/calendar/google-calendar-api.service.ts:105`
**Function:** `listCalendarsByAccountId()`
**Bug:** Empty filtered list triggers `deleteStaleForAccount` with empty `returnedIds` — deletes ALL calendars for that account.

- [ ] Add guard: if `returnedIds` is empty and the API returned calendars, something is wrong — do NOT delete
- [ ] Only call `deleteStaleForAccount` if `returnedIds` is non-empty
- [ ] Or: if all calendars were filtered out, log a warning and skip stale cleanup
- [ ] Add test: empty returnedIds does NOT trigger deleteStaleForAccount

**Effort:** 30 minutes

---

### T-4.14: Fix remaining Medium sync engine bugs (batch)

**Source:** M-14, M-15, M-17, M-18

These are lower-priority but should be addressed together since they're in the same area:

- [ ] **M-14:** Add backoff on duplicate sync detection in `queueInitialSync()` — return 409 with a retry-after header
- [ ] **M-15:** Add pagination/streaming to `pollCalendar()` when syncToken is invalid — don't load all events at once
- [ ] **M-17:** Add composite index for `$or` query in `findBlockersBySourceEvent()` — `{ sourceCalendarId: 1, sourceEventId: 1 }`
- [ ] **M-18:** Add event count limit to `fullyResyncCalendar()` pagination — same pattern as C-11 fix

**Effort:** 1.5 hours

---

### T-4.15: Fix Mongo-before-Google delete order in executeTeardown

**Source:** M-16
**File:** `apps/arm-app-calendar/src/backend/src/events/sync-lifecycle.service.ts:511`
**Function:** `executeTeardown()`
**Bug:** Mongo delete BEFORE Google delete. If Google delete fails, blocker events stay on Google forever.

- [ ] Reverse the order: delete from Google/Microsoft FIRST, then delete from Mongo
- [ ] If provider delete fails, keep the Mongo record (so retry can find the blocker again)
- [ ] Mark Mongo records as "provider-deleted" after successful provider deletion
- [ ] Add test: Google delete failure leaves Mongo record intact for retry

**Effort:** 45 minutes

---

### T-4.16: Fix SSE and DLQ notification swallowing (batch)

**Source:** M-19, M-20, M-21

- [ ] **M-19:** In `onSyncCompleted()` — wrap JSON.parse in try/catch with a fallback that logs the raw data AND still publishes an SSE event (so frontend isn't stale)
- [ ] **M-20:** In `onTeardownCompleted()` — same: catch SSE publish failure, retry once, log if both fail
- [ ] **M-21:** In BullMQ configuration — ensure `removeOnComplete` jobs that fail to move to DLQ are logged. Add a `failed` event listener that checks if the job was captured by DLQ

**Effort:** 45 minutes

---

## Completion Checklist

- [ ] All 16 tasks done
- [ ] Teardown race condition fixed (C-13)
- [ ] Stale blocker sweep handles pagination (C-14)
- [ ] Cancellation window minimized (C-15)
- [ ] Webhook processing skips syncs being torn down (H-06, H-07)
- [ ] Orphan sweep identifies by extended property, not title (H-19, H-20)
- [ ] Retry storm on partial deletion prevented (H-23)
- [ ] Google API 429 backoff in place (H-24)
- [ ] Target webhook doesn't blindly recreate all blockers (H-25)
- [ ] Stale webhooks rejected (H-26)
- [ ] Empty calendar list doesn't delete all calendars (H-27)
- [ ] All test suites pass
