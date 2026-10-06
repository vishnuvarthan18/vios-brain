# Harvest Engine — master build plan for the dev agent, 9 Sep 2026

This is the single consolidated plan. Combines: the trust-layer plan (world-class-harvest-engine-plan-2026-09-09.md), the per-stream audit findings (harvest-engine-stream-audit-2026-09-09.md), and the coverage gaps (coverage-improvement-plan-2026-09-09.md). Hand phases to the dev agent ONE AT A TIME, in this order. Do not skip ahead.

## Ground rules for every phase
- Never merge or deploy to `main`/production without Vishnu's explicit go-ahead in that turn, same as every prior run.
- Build and verify on `dev`/staging first.
- Cron stays off (`crons = []`) until Phase 4 explicitly says to turn it back on.
- Report back after each phase before starting the next.

---

## PHASE 1 — The trust layer (build this first, touches nothing broken yet)

**Goal:** make sure the engine can never again report "success" for 8 days while doing nothing.

1. **DLQ alert.** `harvest-engine-jobs-dlq` currently has no consumer and nothing watches its depth. Add a consumer Worker (or scheduled check) that fires a Cloudflare Email Routing notification to Vishnu the moment any message lands in it. A non-zero DLQ should always be visible, same-day.
2. **`stream_health` table.** New table: one row per stream, tracking `last_real_write_at`, a declared `expected_cadence` (from `sources/*.json`), and a computed `is_stale` flag. Computed by a scheduled Worker that is INDEPENDENT of the streams' own code — it must not trust `job_run.status`, only real row timestamps in the target tables, the same method used to catch the original dedup freeze.
3. **Dashboard rebuild.** Put staleness + DLQ depth at the top of the dashboard, above the pass/fail counts. Fix `dashboard/render.js`'s hardcoded 4-status assumption (existing fix #9) as part of this, so a future `blocked` status doesn't get silently miscounted.
4. **Frozen-fixture contract tests.** For each of the 11 known-silent streams first: save one real captured API response as a test fixture, write one test asserting the parser still extracts the same fields from it today. This is what would have caught BHL's `Title` vs `FullTitle` mismatch immediately instead of after 100% data loss. Backfill fixtures for the other 18 streams over time, not all at once.

**Verify before moving on:** trigger a deliberate test failure (temporarily point one fixture test at a wrong field) and confirm the alert/dashboard actually flags it. Report results.

---

## PHASE 2 — Fix the two root-cause bugs from the 5 Sep freeze incident

5. **Max-age refetch window** in `findExistingSource` / `hash.js`, per-stream from `sources/*.json`. 7-day default for polling sources (crossref, europepmc, gbif, inaturalist, ia-advancedsearch); static one-shot sources (wikidata, census, wdpa) keep infinite window. This is the fix that closes the original freeze.
6. **`max_batch_size` 5 → 1** in `wrangler.toml`. Config-only. Shrinks blast radius of any future redelivery bug to one stream instead of five.

**Verify:** confirm via Phase 1's `stream_health` table that a test stream's staleness clears after a real fetch, not just a status flag.

---

## PHASE 3 — Fix the 16 broken streams, by cohort, each verified against Phase 1's fixture harness before merging

**Cohort A — pure parser/config bugs (fast, mechanical):**
- `bhl` — parses `FullTitle`/`TitleName`; API returns `Title`. Confirmed 100% data loss.
- Audit `gbif`, `wikidata`, `ntca`, `wii`, `historical-text`, `mongabay-india`, `toi-coimbatore`, `thehindu-tn` with the same rigor used to find the BHL bug — each is currently silently writing zero rows and none have a confirmed root cause yet. Do not assume they're the same bug — check each independently.
- `lgd` — source config documents 129 confirmed-matching upstream records never written. Root-cause this one specifically; it's the highest-value fix in this cohort given the gazetteer is the project's single biggest coverage gap.

**Cohort B — blocked on missing keys/config (needs Vishnu's inputs from Phase 0, see below):**
- `wdpa` — WDPA/Protected Planet API token (Vishnu has requested it, pending approval by email).
- `management-plan` — missing prerequisite PDF in R2; move the existing 16MB PDF there (existing fix #3) and update the stream to read from R2 instead of the repo.

**Cohort C — genuinely dead sources, decide explicit status rather than endless silent retry:**
- `shodhganga`, `forests-tn` — both return 522/robots.txt blocks from the source itself. Mirror the existing `wii`-style precedent: either an explicit, justified `robots_txt_override` if legally defensible, or mark `permanently_excluded` in `sources/*.json` so the dashboard stops implying these are recoverable.
- `openalex` — paused on a paid-quota wall (HTTP 429). Do not re-enable without Vishnu confirming whether any current site content depends on OpenAlex-sourced data (per the 5 Sep decision to hold this as a money question).

**Cohort D — retry/status plumbing that makes the above fixes actually stick:**
- Fix #7: real `blocked` status for missing-secret/missing-asset gates (currently 59 of 245 recorded failures are mis-classified as ordinary failures).
- Fix #8: `fetchWithRetry` never retries a 5xx HTTP response, only thrown errors — every other retry-related fix in this plan is inert until this lands.
- Fix #6: stale-run reaper for the 64 tombstoned jobs plus a wall-clock budget in the queue batch handler.

---

## PHASE 0 (parallel, not blocking) — Vishnu's two signups, status as of this writing

- **data.gov.in API key:** obtained. `DATA_GOV_IN_API_KEY` — Vishnu will hand this to whoever runs `wrangler secret put` directly; do not write the raw key into any file or doc.
- **WDPA/Protected Planet API token:** requested, pending approval by email (arrives at vishnu88varthan@gmail.com, may take a day or two). Cohort B above is blocked on this arriving.

When both are confirmed in hand, run:
```
wrangler secret put DATA_GOV_IN_API_KEY
wrangler secret put WDPA_API_KEY
```
against the `harvest-engine` Worker (staging first if a staging env exists, else confirm with Vishnu before touching production secrets).

---

## PHASE 4 — Resume cron, one stream at a time

Do NOT flip all 29 streams back on at once — that's exactly the blast radius Phase 2's `max_batch_size` change and Phase 1's staleness monitoring exist to prevent.

1. Re-enable cron for one verified-healthy stream only (e.g. `unpaywall`, already confirmed working).
2. Watch its `stream_health` row for a real cadence match over 48-72 hours.
3. Add the next stream. Repeat.
4. Only after every stream in Phases 2-3 has been individually verified healthy, restore the full cron schedule (`0 3 * * *`, `0 11 * * *`, `0 19 * * *` per the 5 Sep pause note).

---

## What this build plan deliberately does NOT include

- No rebuild from scratch — `unpaywall`, `gee`, `firms`, `census`, `bhuvan`, the source registry, and the R2-raw-first design are sound and stay as-is.
- No heavyweight orchestrator (Airflow/Dagster/Temporal) — wrong scale for 29 streams at this data volume.
- No chasing exactly-once delivery — idempotent writes on the existing at-least-once queue already handle this.
- The gazetteer's other join sources (Census 2011, Wikidata) are not blocked on new keys and can be worked on any time — they were never the bottleneck; LGD and WDPA were.

## Status of this plan
Written and saved. Ready to hand Phase 1 to the dev agent as the next concrete step — nothing in Phase 1 touches a broken stream or requires the new API keys, so it can start immediately, in parallel with waiting on the WDPA approval email.
