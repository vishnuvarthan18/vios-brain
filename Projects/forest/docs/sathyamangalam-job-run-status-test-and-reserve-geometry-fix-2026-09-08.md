# job-run-status test + missing reserve_geometry migration — fixed, 2026-09-08

Both items surfaced at the end of Phase 1 (harvest-engine overnight results) are fixed and verified locally. Nothing committed or deployed — working tree changes only, on branch `deploy/sensitive-species-fix-verify` in `~/sathyamangalam-atlas` (per the repo's documented friction with git writes from an automated shell, commits should be made from Vishnu's own Terminal).

## 1. `test/job-run-status.test.js` — was failing to load, now 9/9 passing

Root cause: the test file (already present, untracked) called `deriveJobRunStatus`, `JOB_RUN_STATUSES`, and a `withJobRun(db, ctx, (jobRunId, progress) => ...)` two-argument callback shape that `src/lib/job-run.js` didn't implement — none of it was exported, and `withJobRun` only ever called `fn(jobRunId)`.

Rewrote `harvest-engine/src/lib/job-run.js`:
- Exported `JOB_RUN_STATUSES` (mirrors migration 0004's CHECK constraint: running/success/partial/failed/skipped_duplicate).
- Added `deriveJobRunStatus({rowsWritten, rowsSkippedDuplicate, errors})` — pure function, counts-only. This is the fix for the actual bug class the audit exists to catch: a run that wrote 0 rows but skipped N duplicates is `skipped_duplicate`, never `success`, regardless of what the stream itself claims.
- `withJobRun` now: trusts a non-`'success'` self-reported status when there are no errors to weigh against it (needed by `gee-geometry-check`, whose result is a PASS/FAIL verdict counts can't reconstruct), but overrides any `'success'` claim with the counts-derived status. Also now passes a mutable `progress` accumulator (`{rowsWritten, rowsSkippedDuplicate, notes}`) as a second argument to the stream function, so a throw partway through a run no longer erases rows already written — this was a real bug found live (a gee run wrote 7 rows then threw and recorded `failed, rows_written=0`).
- `errors_json` now stores both real errors and `notes` (tagged `kind: "note"`) in one array, so partial-failure detail and informational notes share the column but stay distinguishable.
- Backward compatible: all 33 existing `withJobRun` call sites across `src/streams/*.js` only ever destructured `jobRunId` from the callback args, so the new second `progress` argument is inert for them until they're updated to use it.

Old behavior removed: `job-run.js` used to silently downgrade a true 0-written/0-skipped/no-error run from `success` to `partial` with a canned warning. That's gone — a genuine "nothing to do" run (e.g. an RSS feed with no new items) is a real `success` again. Distinguishing "nothing happened, nothing to do" from "nothing happened, something's broken" is `stream_health`'s job now (the table shipped in Phase 1), watching `rows_written`/timestamps over time — not `job-run.js`, which only ever sees one run in isolation.

Full suite: 39/44 passing. The other 5 failures are pre-existing, in unrelated files (`gee-request-hash.test.js`, `layer-review.test.js`, `place-identity.test.js`) — not touched, not caused by this change, not investigated this session.

## 2. Missing `reserve_geometry` migration — created

`migrations/0005_geometry_scope_annulus.sql` rebuilds a `reserve_geometry` table (rename→copy→drop→rename) as though it already existed, but no migration file in the repo ever created it — confirmed via `grep`, and already flagged (not fixed) in `test/fixtures/harness.js`'s own comments.

Added `migrations/0004_reserve_geometry_init.sql` (filed as 0004 — sorts before 0005, which depends on it — even though `0004_job_run_skipped_status.sql` already claims that number; the repo already has two `0003` files, so a shared numeric prefix disambiguated by filename is the established convention, not a new one). Creates `reserve_geometry` with the shape 0005's rebuild expects: `id, reserve_id, scope (CHECK bbox/core_wls/tiger_reserve/buffer), geojson, area_km2, measured_area_km2, source_note, updated_at, UNIQUE(reserve_id, scope)`.

No rows seeded — no stream currently writes to this table, and fabricating the bbox/core_wls area figures mentioned in 0005's comments without a verified source would misrepresent them as measured. 0005 populates its own `bbox_minus_core` row on top, unchanged.

**Verified end-to-end** with a throwaway Node script (`node:sqlite`, not the CLI — no `sqlite3` binary on this Mac): applied 0001 → 0002 → 0003_coverage_snapshot → 0003_geometry_scope → 0004_job_run_skipped_status → 0004_reserve_geometry_init in sequence, all clean, then applied 0005 on top — previously impossible, now succeeds and inserts its `bbox_minus_core` row (measured_area_km2 = 1804.46) correctly.

Updated the stale comment in `test/fixtures/harness.js` that used to flag this gap — it now points at the fix instead of describing an open gap.

## Not done / still open (unchanged from Phase 1 handoff)

- Nothing was committed or pushed — these are working-tree changes on top of already-uncommitted Phase 1 work.
- `historical-text`'s "0 rows ever" needs a Phase 3 explanation (registry files exist, contradicting the original audit's "missing config" note) — untouched.
- Whether/when to apply `0004_reserve_geometry_init.sql` and `0005`/`0006` to the real D1 database is a deploy decision, not made here.
