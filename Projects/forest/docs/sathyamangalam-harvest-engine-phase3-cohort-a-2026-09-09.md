# Phase 3, Cohort A — results, 2026-09-09

Branch: `deploy/sensitive-species-fix-verify`. Nothing deployed, no staging created, main untouched.

## lgd — real bug found and fixed, then a second bug found and fixed

**Bug 1 (first commit):** NOT a field-name mismatch (the master plan's assumption was wrong — villageNameEnglish was already correct, verified live). The real bug: data.gov.in caps page size server-side below the requested `limit=100`; the pagination loop in lgd.js treated a short page as end-of-dataset and stopped after page 1 instead of advancing by rows actually returned. First matching row was at offset 24 — page one had zero matches, so the stream silently wrote 0 rows and reported `success`.

**Bug 2 (second commit, self-caught):** raising MAX_PAGES_PER_RUN to 49 (see below) surfaced a second defect the first commit introduced: moving `offset += records.length` to the bottom of the loop meant the dedup `continue` bypassed it, so a run that hit an existing page never advanced its offset — 49 attempts all re-tested offset 0. Fixed in the same commit: the dedup path now advances the offset explicitly before continuing.

**MAX_PAGES_PER_RUN: 10 → 49.** Chosen as an upper bound (486 rows ÷ 10/page = 49; the loop exits on the API's own `total` regardless of actual page size, so 49 is safe for any page size ≥10). The real production key's actual per-page size could NOT be measured — Cloudflare Worker secrets are write-only (`wrangler secret list` returns name/type only, never the value), so there was no way to pull the real key into a local test without exposing it, which was correctly refused. Verification used the public demo key (10 rows/page) via `wrangler dev --local`: full run now reads all 49 pages, reaches offset 480 (+6 = 486 = total), finishes `status: "success"`, `rows_written: 115`, no truncation message.

Row count varies run to run (115 vs. an offline-predicted 112) because the API returns overlapping/duplicate rows across an unsorted paged scan — `findOrCreatePlace`'s slug dedup absorbs it, expect a few rows of variance between runs.

Also: `sources/lgd.json`'s "129 matching records" note is now stale — real count is 127. `wrangler.toml`'s comment claiming DATA_GOV_IN_API_KEY was "not yet obtained" is also stale — confirmed present via `wrangler secret list`.

## bhl — untouched, moved to Cohort B

No BHL_API_KEY exists. No code touched. Belongs in Cohort B (blocked on key), not Cohort A — the master plan had this wrong too.

## ntca and mongabay-india — both unblocked on retry

- `ntca.gov.in` — now 200 (was 406 from its own WAF). 1255 real PDF links extracted correctly.
- `india.mongabay.com/feed/` — now real RSS (was a Cloudflare bot-challenge page). 20 items parse cleanly.

Both representative fixtures replaced with real captures; suite held at 39/5. Fixtures are untracked/uncommitted still. `test/fixtures/silent-streams.test.js`'s comments still describe both as blocked — now false, needs a wording update before those fixtures get committed.

## gbif, wikidata, wii, historical-text, toi-coimbatore, thehindu-tn — untouched

Provisionally resolved pending an actual deploy-and-observe cycle (no staging environment exists to test this against right now).

## Open items, not yet decided

1. **census_code stored as `"932977.0"`** (float-formatted string). Deliberately left for later per Vishnu's decision 2026-09-09 — changing it affects census↔LGD join semantics.
2. **Partial-run resume is unreliable.** Tested by deleting a mid-range page and re-running: the gap was not refetched. On a dedup skip before any page is fetched, `lastPageSize` is unset so the offset stride falls back to PAGE_LIMIT (100) across pages that are actually 10 wide — same behavior as the original code, not newly introduced, but not fixed either. Fixing it properly is a design decision about how partial coverage should resume, not a mechanical patch — flagged, not actioned.
3. **The 4 replaced fixtures (ntca, mongabay-india) and their stale "blocked" comments in silent-streams.test.js are still uncommitted.** Worth landing as their own commit with the comment wording corrected.
