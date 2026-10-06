# Harvest Engine — Phase 1 (trust layer) results, 2026-09-08

Scope actually executed: **Phase 1 only**, per your explicit instruction not to start Phase 2 or 3 without checking back. Everything below is code-only, on this branch (`deploy/sensitive-species-fix-verify`), unmerged, undeployed. No `wrangler deploy`, no `--remote` D1/queue command, no email actually sent — see "Why code-only" below.

## Why code-only, not a live dev/staging run

`wrangler.toml` defines exactly one environment — one D1 database, one queue, one Worker, all labeled `production` — and this session's `wrangler` CLI is authenticated with real, write-capable Cloudflare credentials. There is no separate dev/staging environment in this repo to safely "stay on." I flagged this before writing any code and you chose **code-only, no remote commands**: everything below was built and verified locally (`node --test` against an in-memory SQLite database migrated with the real production schema), nothing was deployed or run against the live account. Confirming this hasn't touched production, or setting up a real staging environment, is the first thing to decide before any of this goes further.

## Plan-vs-reality: what didn't match before I built anything

Per the plan document's own instruction to verify every claim before acting on it:

- **Phase 1 item 3 (dashboard rebuild fixing the hardcoded status bug) was already done.** Commits `5e4409d` and `367d1f3`, already on this branch, built the `STATUS_META` single-source-of-truth fix and a stream-health panel. I did not rebuild it — I added the one piece it was still missing (DLQ depth, see below).
- **"Declared cadence from `sources/*.json`" doesn't exist.** No source file has a cadence field; `dashboard/render.js`'s own comments already document this. `stream_health` uses the same fleet-wide cadence constant (8h × 3 = 24h) the dashboard already uses, not a per-stream value — the plan's premise there was wrong.
- **A "separate scheduled Worker" for stream_health, literally, would mean a second Worker with its own wrangler.toml/bindings for one function call** — disproportionate. I computed it independently of `job_run.status` (the substantive requirement — verified by test, see below) inside the same Worker's existing `scheduled()`, not as a second deployable.
- **The stream audit's list of 11 silently-broken streams checked out**: gbif, core, bhl, wikidata, lgd, ntca, wii, historical-text, mongabay-india, toi-coimbatore, thehindu-tn — confirmed against the audit doc, matches the plan.

## What shipped

### 1. DLQ consumer + alert
- `wrangler.toml`: added a second `[[queues.consumers]]` block for `harvest-engine-jobs-dlq` (confirmed as the real DLQ name), and a `[[send_email]]` binding (`DLQ_ALERT_EMAIL`).
- `src/queue-consumer.js`: `handleDlqBatch()` — every DLQ message is recorded to a new `dlq_alert` table first (so the alert exists even if email fails), then an email send is attempted via `src/lib/dlq-alert.js`.
- `src/index.js`: `queue()` now branches on `batch.queue` to route DLQ messages to the new handler.
- **Not verified end-to-end — cannot be, without deploying.** `sendDlqAlertEmail()` uses Cloudflare's native `cloudflare:email` binding, which needs a destination address verified in the Cloudflare dashboard (Email Routing) — a manual step outside any code change, and not done. Until that happens, the function throws by design and the failure is recorded in `dlq_alert.email_error` rather than silently lost. **Open item for you**: verify a destination address in the dashboard, fill in the two `REPLACE_WITH_...` placeholders in `wrangler.toml`, then this can actually deliver.

### 2. `stream_health` table, computed independently of `job_run.status`
- `migrations/0006_dlq_and_stream_health.sql`: new `dlq_alert` and `stream_health` tables.
- `src/lib/stream-health.js`: `computeStreamHealth()` — verdict (`healthy` / `stale` / `never_written` / `no_runs`) derived only from `rows_written` and timestamps, never from the self-reported `status` column. Wired into the existing `scheduled()` cron fire.
- **Verified by test, including the required exit proof**: `test/stream-health.test.js` reproduces the exact audit pattern (5 consecutive `job_run` rows, `status='success'`, `rows_written=0`) and asserts verdict `never_written`, not `healthy`. **I then deliberately broke the detection** (made the `never_written` branch unreachable, simulating "verdict trusts status"), re-ran the test, watched it fail red (`actual: 'healthy', expected: 'never_written'`), then reverted and re-ran green. That's the Phase 1 exit criterion, done locally since there's no staging system to break something in live.

### 3. Dashboard: DLQ depth (the one piece of the rebuild that was still missing)
- `dashboard/render.js`: added a DLQ panel reading the new `dlq_alert` table (guarded with try/catch for a DB that hasn't run migration 0006 yet, same pattern the existing coverage-snapshot code already uses).

### 4. Frozen-fixture contract tests for the 11 silently-broken streams
- `test/fixtures/harness.js`: a real D1-shaped SQLite adapter (Node 24's built-in `node:sqlite`) migrated with the actual production schema (migrations 0001–0004, 0006), not a hand-rolled stub.
- `test/fixtures/silent-streams.test.js`: 12 tests, one (two for historical-text) per stream, all passing. Each feeds a captured or representative API response through the stream's real parsing logic and the real shared find-or-create helper, asserting a genuinely new record is written and an exact repeat is deduped — the precise contract that broke in production.
- **Fixture provenance, stated plainly, not disguised as uniform**:
  - **Real, fetched live 2026-09-08**: gbif, core, wikidata, wii, toi-coimbatore, thehindu-tn.
  - **Representative, hand-built, clearly labeled in the test file** — not live captures: **bhl** and **lgd** (need `BHL_API_KEY` / `DATA_GOV_IN_API_KEY`, not available this session), **ntca** (ntca.gov.in's own ModSecurity WAF returned "406 Not Acceptable" to a scripted request — preserved at `test/fixtures/ntca.raw.html` as evidence, itself worth a look in Phase 3), **mongabay-india** (a live fetch of `india.mongabay.com/feed/` returned a Cloudflare bot-challenge page instead of RSS — preserved at `test/fixtures/mongabay-india.blocked-capture.raw.html`; also worth Phase 3 attention, since the same WAF/CDN posture could be why the stream itself never writes).
  - Swap these three/four for real captures (once keys arrive, and once a real-browser capture of ntca.gov.in is available) before treating this as full production parity.

## Two things the tests themselves surfaced, unprompted

- **`historical-text`'s registry files exist and are real**, contrary to the audit's note that it "does not exist" — it was looking for the wrong filename. `sources/historical-volumes.json` (6 real Internet Archive volume identifiers) and `sources/historical-spelling-variants.json` are both present and non-placeholder. My test confirms `findOrCreateDocument` correctly creates a document from the real registry data — meaning the audit's "0 rows written ever" for this stream is **not** explained by a missing registry, and needs a different Phase 3 explanation (possibly the same request-hash freeze pattern as gbif/bhl/wikidata, but not verified — flagging, not diagnosing).
- **Migration 0005 references a `reserve_geometry` table that no migration file in this repo creates.** Same class of gap as the already-known "0004 was reconstructed because its file went missing" — there's a second missing migration. Not fixed here (out of Phase 1 scope), but real and worth tracking down before it causes a rebuild-from-scratch to diverge from production again.

## Pre-existing gaps found, not caused by this work, not fixed here

Verified with `git stash` (nothing to stash — these files are untracked from before this session) that all of the below existed identically before I touched anything:

- **`test/job-run-status.test.js` cannot even load.** It imports `deriveJobRunStatus` and `JOB_RUN_STATUSES` from `src/lib/job-run.js`, but that file only exports `startJobRun`, `finishJobRun`, and `withJobRun` — no `deriveJobRunStatus`, no `JOB_RUN_STATUSES`. This is the test file for the exact "counts are evidence, not the claimed status" logic that fixed the original 2026-08-25 dedup freeze — and it has apparently never run successfully via the project's own documented `npm test`. This is the most important thing in this report to look at: **the regression test for the root historical bug has been silently broken**, which is precisely the kind of blindness Phase 1 exists to close. I did not fix it — it's adjacent to but outside the four things you scoped for tonight, and I didn't want to touch core status-derivation logic without your go-ahead.
- 5 other pre-existing test failures in `test/gee-request-hash.test.js` and `test/place-identity.test.js` (place merge-by-osm_id logic, GEE request hashing) — real, unrelated to Phase 1, not investigated further.
- Running `npm test` as documented (`node --test test/*.test.js`) also fails outright on any file that transitively imports a `.json` file, because Node 24's ESM loader requires an explicit `type: "json"` import attribute that this codebase's ~30 JSON imports don't have (wrangler's esbuild bundler tolerates the omission; plain Node doesn't). I added `test/json-loader-register.mjs` + `test/json-loader.mjs` as a test-only loader hook so my new tests could run without editing 30 source files — but this means `npm test`, as currently documented in `package.json`, silently fails for the JSON-importing test file above too. Worth deciding whether to update `package.json`'s `test` script to use this loader, or fix the imports directly.

## Test results

```
node --import ./test/json-loader-register.mjs --test test/*.test.js test/fixtures/*.test.js
tests 48
pass  42
fail  6   (all 6 pre-existing — see above; 0 regressions from this session's changes)
```

The 16 new tests (4 stream-health + 12 fixture-contract) all pass, including the deliberate break/revert exit proof.

## What's ready for your review vs. what still needs work

**Ready for review (code complete, locally verified, nothing deployed):**
- `migrations/0006_dlq_and_stream_health.sql`
- `src/lib/stream-health.js`, `src/lib/dlq-alert.js`
- `src/queue-consumer.js`, `src/index.js` (DLQ routing + scheduled health check)
- `dashboard/render.js` (DLQ panel)
- `test/fixtures/harness.js`, `test/fixtures/silent-streams.test.js`, `test/stream-health.test.js`

**Needs your input before this can go further:**
1. Is there a real dev/staging environment to set up, or should Phase 1+ keep targeting production directly once you're ready to deploy?
2. A destination email address needs verifying in the Cloudflare dashboard (Email Routing) before DLQ alerts can actually send — I can't do this from code.
3. `wrangler.toml`'s two `REPLACE_WITH_...` placeholders need real values.
4. The broken `job-run-status.test.js` import and the 5 other pre-existing test failures — want these fixed before or separately from Phase 2?
5. Given all of the above, Phase 2 (max-age refetch window, batch size) and Phase 3 have **not been started**, per your instruction to check back first.
