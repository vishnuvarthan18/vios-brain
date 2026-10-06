# Troubleshooting — Calendar Sync Not Working

Symptom-first triage for a stuck, failed, or silently broken calendar sync. This doc is a decision tree pointing at the specific mechanism most likely responsible — [`calendar/SYNC-FLOW.md`](../calendar/SYNC-FLOW.md), [`calendar/WEBHOOKS.md`](../calendar/WEBHOOKS.md), and [`calendar/ARCHITECTURE.md`](../calendar/ARCHITECTURE.md) already document the mechanics in full; `apps/arm-app-calendar/docs/failure-modes.md` already documents dependency-outage behavior (Mongo/Redis/core-be/Google down) in depth. Don't re-derive any of that here — this doc routes you to the right section.

**Read this first**: `SyncHistory` is **not** a complete activity log — successful incremental (webhook/poll) updates never write to it, only initial-sync runs and delta-path failures do ([SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §4). If you're checking "why hasn't `SyncHistory` updated in hours" for an otherwise-healthy ongoing sync, that's expected, not a symptom.

---

## 1. Quick triage table

| Symptom | Most likely cause | Jump to |
|---|---|---|
| Sync created, nothing ever appears in target calendar | `initial-sync` job failed with zero retries — check dead-letter, not just `SyncHistory` | §2 |
| Sync was working, then new source events stopped appearing | Webhook channel dead (renewal failure) or account token expired | §3 |
| Deleted source events still show as blockers in target | Orphaned blocker — stale-sweep hasn't run yet | §4 |
| Sync works on Google, not on the Microsoft side of the same setup (or vice versa) | Provider asymmetry — not a bug, a real capability gap | §5 |
| Account shows "connected" but sync is silently dead | `isAuthenticated` self-heal only triggers on 401, not every failure mode | §6 |
| A dependency (Mongo/Redis/core-be/Google) is down | Covered exhaustively elsewhere — don't duplicate the diagnosis here | §7 |
| Toggled a sync's title/description/location visibility, but already-synced blockers still show the old content | Expected — a toggle edit only takes effect on future webhook deltas unless you also trigger a Resync | §8 |

---

## 2. Sync created, nothing ever appears

Check the `initial-sync` BullMQ queue's dead-letter contents first, not just `SyncHistory` — per [SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §2, `initial-sync` jobs have **no retry/backoff configured at all**. A whole-job-level exception (thrown before the per-calendar/per-event try/catch even engages — e.g. a malformed request) goes straight to `dead-letter` on the very first failure. If nothing shows in `SyncHistory` for the sync at all (not even a `pending`/`failed` row), the job likely never got past its own top-level setup — check `dead-letter`, then application logs for that job ID.

If `SyncHistory` **does** show a row: `pending` stuck with no progression means the worker never picked up the job (check the `sync.processor.ts` worker is actually running, concurrency limits per [SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §2). A `failed`/`partial` row has `failureReasons` populated directly — read those first, they're written per-calendar/per-event and usually name the actual Google/Microsoft API error.

**Diagnostic endpoints** (calendar-be, via `events.controller.ts`): `GET /v1/events/active-syncs-with-history` and `GET /v1/events/sync-history/:targetCalendarId` — check these before assuming you need database access.

---

## 3. Sync was working, then stopped

This is almost always one of two things, and they're easy to conflate:

- **Webhook channel died and didn't renew.** Per [WEBHOOKS.md](../calendar/WEBHOOKS.md) §4, this is a confirmed, still-open gap (`TASK-045`): if a renewal fails after the old channel was already stopped, the DB is left with a stale, dead channel record and **no automatic poll fallback** — silent, until someone notices. Distinguish this from a channel that was never registered in the first place (which *does* get a poll fallback automatically, §1 of the same doc) — only a channel that succeeded once and then failed to *renew* falls into this gap. Check `Calendar.expiration` against the current time, and check the `webhook-renewal` queue's dead-letter contents for that calendar.
- **The linked account's token needs reconnecting.** This is a completely different clock than the platform login session — see [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §6 and [`troubleshooting/auth-failures.md`](auth-failures.md) §9. Check `Account.isAuthenticated` — but read §6 below before trusting that flag alone.

**No dashboard or alert exists for either of these today** — per [WEBHOOKS.md](../calendar/WEBHOOKS.md) §7, this is genuinely a "someone has to notice and go looking" situation, not something that pages anyone.

---

## 4. Deleted source events still appear as blockers

Two separate sweep mechanisms exist and neither runs on every sync tick — per [SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §6:

- **F-21** runs on every `initial-sync` pass (re-running the sync manually, per §8 below, forces this).
- **NR-06** runs only as a side effect of a full re-fetch (a Google `410 Gone` on the stored sync token, or Microsoft's monthly delta-chain renewal).

**A normal incremental delta cycle that hits neither of these never sweeps** — deletions there rely entirely on the provider sending an explicit cancellation event. If a source event vanished from the provider without an explicit cancellation (this does happen), the orphaned blocker persists until one of the two sweeps above runs. This is a known, bounded window, not an unbounded bug — but it can look like "sync is broken" if someone expects instant deletion propagation.

---

## 5. Works on one provider, not the other — check if this is actually a bug

Several Google/Microsoft asymmetries are **by design**, not gaps — confirm against this list before treating it as broken:

- Microsoft has no push notifications at all — every Microsoft-connected calendar is poll-only (5-minute interval) by construction, not a fallback-from-failure. Don't expect near-real-time propagation on the Microsoft side. ([`calendar/ARCHITECTURE.md`](../calendar/ARCHITECTURE.md) §2)
- Microsoft has no quota/rate-limit detection at all — a Microsoft `429` is treated as a plain non-retryable failure (one attempt, no backoff), while the equivalent Google failure gets 3-attempt exponential backoff. If Microsoft-side blocker deletes are failing under load where Google ones aren't, this asymmetry is why. ([SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §5)
- Microsoft's renewal mechanism is a **monthly full resync**, not a channel renewal — a materially different reliability mechanism from Google's daily renewal. If a Microsoft-side sync has been silently drifting for weeks, this monthly cadence (not a bug) is why the correction takes as long as it does. ([`calendar/ARCHITECTURE.md`](../calendar/ARCHITECTURE.md) §4)

---

## 6. Account shows "connected" but sync is dead

`isAuthenticated` only self-heals to `false` on a real `401` — confirmed as a verified drift from what older docs claimed ([`calendar/ARCHITECTURE.md`](../calendar/ARCHITECTURE.md) §11.1). A `403` (Google) goes through a separate quota-retry path instead of flipping this flag; Microsoft has no equivalent handling for its own `403`s at all. **This means `isAuthenticated: true` does not guarantee the account is actually working** — it only guarantees the last relevant call didn't fail with a plain 401. If the account looks connected but nothing is syncing, don't stop at the `isAuthenticated` check; look at actual recent API call outcomes in logs.

Also check whether this is a **login-provisioned** account (`syncFromCoreAuth()`) vs. an **Add-Account**-provisioned one — the two have different refresh-token availability, and only the CoreAuth path has a fallback for a missing local refresh token (Google only, not Microsoft) — see [`calendar/ARCHITECTURE.md`](../calendar/ARCHITECTURE.md) §5 and [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) §5.

---

## 7. A dependency is down (Mongo, Redis, core-be, Google/Microsoft APIs)

**Don't re-diagnose this here.** `apps/arm-app-calendar/docs/failure-modes.md` is a source-verified, per-dependency runbook covering exactly this — what breaks, what an on-call engineer sees, and what to do, for Mongo, Redis, core-be, Google OAuth token-refresh, and the Google Calendar API. Go there directly. Its Microsoft Graph API section is stale (predates the Microsoft calendar-sync integration — see [WEBHOOKS.md](../calendar/WEBHOOKS.md) §8) — for Microsoft-specific issues, use §5 above instead.

---

## 8. Forcing a resync

**As of 2026-08-13, a real endpoint exists for this** — `POST /v1/events/:targetCalendarId/sync/resync`, no body required ([`calendar/API.md`](../calendar/API.md) §5, [SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §10). It re-runs the full `performInitialSync` algorithm ([SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §1) against this target's already-configured sources, grouped by each source's own stored `syncFromDate` — no need to know or re-supply the original source/target mapping. It also runs the F-21 stale-blocker sweep (§4 above) as a side effect, same as any initial sync.

**This is a genuine content refresh, not just a rescan for new events** — initial sync's blocker step now diffs an existing blocker's stored content and time against freshly-resolved values and updates on a mismatch, instead of only creating blockers for events that don't have one yet. So resyncing after changing a sync's `showTitle`/`showDescription`/`showLocation` toggles (via `POST /v1/events/:targetCalendarId/sync/update`) is the way to make that change actually visible on events synced before the toggle was flipped — those blockers otherwise sit unchanged until their source event happens to receive a webhook delta.

Guarded against misuse: `404` if no active (non-stopping) sync exists for the target, `403` if the caller doesn't own the target account, `409` if a sync is already `pending`/`in-progress` for this target (refuses to enqueue an overlapping run rather than racing one).

**RB-03 (Manual Calendar Re-sync)** in `meta/documentation-inventory.md` predates this endpoint and describes the old workaround (re-POSTing the sync-creation route) as the only option — that framing is now stale; a dedicated runbook documenting the real endpoint above still doesn't exist as a separate file, but the underlying capability gap it was tracking is closed.

---

## Related documents

- [`calendar/SYNC-FLOW.md`](../calendar/SYNC-FLOW.md) — the sync engine at the algorithm level, referenced throughout this doc
- [`calendar/WEBHOOKS.md`](../calendar/WEBHOOKS.md) — webhook registration/renewal, the `TASK-045` gap referenced in §3
- [`calendar/ARCHITECTURE.md`](../calendar/ARCHITECTURE.md) — module map, provider capability differences referenced in §5–§6
- `apps/arm-app-calendar/docs/failure-modes.md` — per-dependency outage behavior, referenced in §7
- [`troubleshooting/auth-failures.md`](auth-failures.md) §9 — the separate calendar-account token clock referenced in §3, §6
