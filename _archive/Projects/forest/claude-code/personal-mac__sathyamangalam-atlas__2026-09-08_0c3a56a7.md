**Vishnu** (2026-09-08T18:34): In ~/sathyamangalam-atlas/harvest-engine, on the current branch. This is
Phase 2, item 5 of harvest-engine-master-build-plan-2026-09-09.md — the
fix that actually closes the original 8-day dedup freeze (see
harvest-engine-dedup-freeze-and-pause-2026-09-05.md for the incident).

THE BUG: findExistingSource() in src/lib/hash.js matches any source row
ever written for a given request hash, with no age check. A stream that
fetched once on day 1 reports "success" forever without ever calling the
API again, because it always finds that old row and skips.

THE FIX:

1. In src/lib/hash.js, change findExistingSource's signature to accept an
   optional maxAgeDays:

   export async function findExistingSource(db, reserveId, requestHash, { maxAgeDays } = {}) {
     if (maxAgeDays == null) {
       // unchanged behavior — infinite window, for static one-shot sources
       const row = await db
         .prepare("SELECT * FROM source WHERE reserve_id = ? AND request_hash = ?")
         .bind(reserveId, requestHash)
         .first();
       return row ?? null;
     }
     const cutoff = new Date(Date.now() - maxAgeDays * 24 * 60 * 60 * 1000).toISOString();
     const row = await db
       .prepare("SELECT * FROM source WHERE reserve_id = ? AND request_hash = ? AND retrieved_at >= ?")
       .bind(reserveId, requestHash, cutoff)
       .first();
     return row ?? null;
   }

   Adjust exact syntax/style to match the file's existing conventions, but
   keep the behavior: omitting maxAgeDays must be identical to today's
   behavior (this is a backward-compatible additive change — every other
   stream that doesn't pass maxAgeDays is unaffected).

2. Add "max_age_days": 7 to the top-level of the source object (not the
   file) in these five files only:
     sources/crossref.json
     sources/europepmc.json
     sources/gbif.json
     sources/inaturalist.json
     sources/ia-scholar.json
   Follow whatever shape each file already uses for its source entry — if
   a file has multiple entries under "sources", add it to each one; check
   first rather than assuming.

3. Do NOT add max_age_days to sources/wikidata.json, sources/census.json,
   or sources/wdpa.json — they stay on the infinite window, unchanged, per
   the 5 Sep editorial decision recorded in
   harvest-engine-dedup-freeze-and-pause-2026-09-05.md.

4. Update the five stream files (src/streams/crossref.js, europepmc.js,
   gbif.js, inaturalist.js, ia-scholar.js) to read max_age_days from their
   already-imported source config and pass it through:
     const existingSource = await findExistingSource(db, reserve.id, reqHash, { maxAgeDays: sourceConfig.max_age_days });
   Match each file's actual variable name for its loaded source config
   (grep each file first — don't assume it's called sourceConfig
   everywhere, crossref.js's is but verify the others).

5. Every OTHER stream calling findExistingSource (unpaywall, gee, gbif's
   sibling calls if any, core, shodhganga, bhuvan, lgd, overpass, wdpa,
   census, bhl, openalex, firms, wikidata, ebird, semanticscholar,
   overpass-boundary, rss.js, crawler.js, dummy.js) must be left
   completely untouched — don't pass a fourth argument, don't add
   max_age_days to their config files. This fix is scoped to exactly the
   five polling sources named above.

VERIFY before committing:
- npm test — report the full pass/fail count. It should be unchanged from
  the current 39/5 baseline (this fix has no existing test coverage yet,
  so it shouldn't move that number in either direction) — if it does,
  stop and report rather than proceeding.
- Grep to confirm exactly five sources/*.json files gained max_age_days
  and no others did:
    grep -rl "max_age_days" sources/
- Grep to confirm exactly five stream files pass maxAgeDays and no others
  changed:
    git diff --stat src/streams/ | cat

COMMIT, scoped to exactly: src/lib/hash.js, the five sources/*.json files,
and the five src/streams/*.js files listed above. Nothing else.

  git commit -m "fix(harvest-engine): max-age refetch window for polling sources

findExistingSource() matched any source row ever written, with no age
check — a stream that fetched once kept reporting success forever
without recontacting the API. This is the fix that closes the 8-day
dedup freeze of 5 Sep 2026 (harvest-engine-dedup-freeze-and-pause-2026-09-05.md).

findExistingSource() now takes an optional maxAgeDays; omitting it keeps
today's infinite-window behavior unchanged (every stream not touched by
this commit is unaffected). Five polling sources — crossref, europepmc,
gbif, inaturalist, ia-scholar — get a 7-day window via a new
max_age_days field in their sources/*.json entry, matching the 3x/day
cron cadence with margin. Static one-shot sources (wikidata, census,
wdpa) are untouched and keep the infinite window, per the 5 Sep
editorial decision.

Phase 2, item 5 of the master build plan."

Do NOT deploy. Do NOT touch max_batch_size (already committed separately
in 3f4b176) or anything from the DLQ/stream_health commit (eb0bda1).
Report back: commit hash, final test count, and confirmation the two
greps above came back with exactly five files each.

**Claude** (2026-09-08T18:34): I'll start by inspecting the current state of the files.

**Claude** (2026-09-08T18:34): _[tool: Bash]_

**Claude** (2026-09-08T18:34): _[tool: Bash]_

**Claude** (2026-09-08T18:34): _[tool: Bash]_

**Claude** (2026-09-08T18:34): All five use `sourceConfig`, and each of the five source files has exactly one entry. Running the baseline test while I make edits.

**Claude** (2026-09-08T18:34): _[tool: Bash]_

**Claude** (2026-09-08T18:34): _[tool: Bash]_

**Claude** (2026-09-08T18:35): _[tool: Bash]_

**Claude** (2026-09-08T18:35): Baseline confirmed: 39 pass / 5 fail, matching the brief. Now applying the fix.

**Claude** (2026-09-08T18:35): _[tool: Bash]_

**Claude** (2026-09-08T18:35): _[tool: Bash]_

**Claude** (2026-09-08T18:35): All checks pass. Committing the ten scoped files.

**Claude** (2026-09-08T18:35): _[tool: Bash]_

**Claude** (2026-09-08T18:35): _[tool: Bash]_

**Claude** (2026-09-08T18:35): _[tool: Bash]_

**Claude** (2026-09-08T18:35): Done.

**Commit:** `8b91909` on `deploy/sensitive-species-fix-verify`

**Tests:** 44 total, **39 pass / 5 fail** — unchanged from baseline. I ran the suite *before* editing to confirm the 39/5 baseline was real rather than trusting the brief's number; it matched, and the 5 failures are the pre-existing `place-identity.test.js` ones, untouched by this fix.

**Greps — both exactly five:**
- `grep -rl "max_age_days" sources/` → crossref, europepmc, gbif, inaturalist, ia-scholar. wikidata, census, wdpa clean.
- `git diff --stat src/streams/` → the same five `.js` files, 1 line each.

**Two things I verified rather than assumed, per your instruction:**
1. All five streams do name their config `sourceConfig` (not just crossref), so the pass-through line is identical in each.
2. Each of the five JSON files has exactly **one** entry under `sources`, so `max_age_days: 7` went on one object per file — no multi-entry case to handle. I placed it next to `rate_limit_ms` in each, and re-parsed all five as JSON to confirm validity and that the field reads back as `7`.

One note worth flagging: the age filter compares `retrieved_at >= cutoff` as a **string**. That's correct here because `retrieved_at` is `TEXT` and rows are written as ISO-8601 UTC, which sorts lexicographically — but it silently breaks if any writer ever stores a non-ISO or non-UTC timestamp. Not in scope to change, and no existing test covers the new window at all. Worth an item for a follow-up test.

Not deployed. `max_batch_size` and the DLQ/stream_health work untouched.

**Vishnu** (2026-09-09T02:21): In ~/sathyamangalam-atlas/harvest-engine, on the current branch. Phase 3,
Cohort A — but scoped to what's actually true, not the master plan's
assumptions (verify as you go, don't take the plan's claims at face value,
per what's already been found this session):

PART 1 — lgd (the real mystery in this cohort)

sources/lgd.json confirms villageNameEnglish is the correct field
(verified live 2026-09-02, 129 matching records), lgd.js already reads it
correctly, and DATA_GOV_IN_API_KEY is already a Cloudflare Worker secret.
So the "zero rows written" bug here is NOT a field-name mismatch — it's
somewhere else in the pipeline (api_filters application, the
findOrCreatePlace write path in src/lib/place.js, request-hash dedup
before Phase 2's fix, or something else).

Investigate with a REAL run against the REAL data.gov.in API but WITHOUT
touching production D1:
  wrangler dev --local
(this runs the Worker locally against a local, throwaway D1 instance —
confirm this in the wrangler docs for the installed version before
relying on it; the point is: real network call, fake database, zero
production risk). Trigger the lgd stream against it, inspect what
actually happens to the 129 candidate rows step by step — do they reach
findOrCreatePlace at all? Does it reject them? Fix whatever the real
cause turns out to be. If `wrangler dev --local` doesn't work as
expected in this repo's wrangler version, stop and report rather than
improvising a workaround that touches the real database.

PART 2 — bhl: do NOT attempt a code fix

No BHL_API_KEY exists yet (confirmed in sources/bhl.json — this is a
Cohort B blocker, not Cohort A). Do not guess at or "fix" the
FullTitle/TitleName field names without a real authenticated response to
check them against — that would be exactly the kind of unverified change
this whole audit exists to prevent. Leave bhl.js untouched. Just note in
your report that bhl should move to Cohort B in tracking.

PART 3 — ntca and mongabay-india: one retry each, honestly

Both were blocked when fixtures were captured (ntca.gov.in returned 406
from its own WAF; india.mongabay.com/feed/ returned a Cloudflare bot
challenge instead of RSS). Try ONE fetch each again, using the same
honest, identified User-Agent this project already uses elsewhere (check
src/lib/robots.js's buildUserAgent) — not spoofing a browser, not
rotating IPs, nothing that would violate the project's own scraping
posture. If still blocked, leave both exactly as documented (blocked,
not fixable by retrying) — do not chase around the block. If NOT
blocked this time, capture the real response as a new fixture replacing
the representative one, and note whether the existing parser handles it
correctly.

PART 4 — gbif, wikidata, wii, historical-text, toi-coimbatore, thehindu-tn: no code changes

Their fixture contract tests already pass against real captured
responses (test/fixtures/silent-streams.test.js). Do not touch their
parsers. Their production zero-rows problem was most likely Phase 2's
now-fixed dedup freeze, not a parsing bug — this can only be confirmed by
an actual deploy-and-observe cycle, which is out of scope for this
prompt (no staging environment currently exists for harvest-engine — it
was deleted in an earlier cleanup as unused, and recreating one isn't
your call to make). Just note "provisionally resolved pending deploy" for
these six in your report.

Do NOT deploy anything, anywhere. Do NOT create or recreate a staging
environment. Do NOT touch main. If PART 1 produces a real, verified fix,
commit it alone with a message explaining exactly what the real bug was
(not what the plan assumed it was) and how you verified it. Report back:
what you found for lgd, and confirmation nothing else was touched.

**Claude** (2026-09-09T02:21): Starting with Part 1 — reading the actual code before assuming anything about the cause.

**Claude** (2026-09-09T02:21): _[tool: Bash]_

**Claude** (2026-09-09T02:21): _[tool: Bash]_

**Claude** (2026-09-09T02:22): Two plausible causes are already visible in the code. Let me check the local-dev preconditions before running, since the API key is a *production* secret and secrets can't be read back.

**Claude** (2026-09-09T02:22): _[tool: Bash]_

**Claude** (2026-09-09T02:22): _[tool: Bash]_

**Claude** (2026-09-09T02:22): Wrangler 4.125.0 confirms the brief's premise: `dev` is local-by-default, `--local` disables remote bindings, D1 persists to `.wrangler/state`, and `fetch()` still hits the real network. But `DATA_GOV_IN_API_KEY` is **not** in `.dev.vars`, and `wrangler.toml`'s own comment says it was never obtained — which contradicts the brief. Verifying against the live account:

**Claude** (2026-09-09T02:22): _[tool: Bash]_

**Claude** (2026-09-09T02:22): _[tool: Bash]_

**Claude** (2026-09-09T02:22): The secret exists in production (the brief is right; `wrangler.toml`'s comment is stale). But secret values can't be read back, so a local run has no key. Let me locate `buildUserAgent` and check whether the repo has any other route to the key.

**Claude** (2026-09-09T02:22): _[tool: Bash]_

**Claude** (2026-09-09T02:22): _[tool: Bash]_

**Claude** (2026-09-09T02:23): `buildUserAgent` produces an honest identified UA. The key exists only as an unreadable production secret, so before deciding Part 1 is blocked let me test the real API contract directly with data.gov.in's *published demo key* (the same one `sources/lgd.json` records using) — real network, no database at all.

**Claude** (2026-09-09T02:23): _[tool: Bash]_

**Claude** (2026-09-09T02:23): Real API responds 200, filter works (`total: 486`), and `villageNameEnglish` is confirmed present. But note: `limit=100` returned only **10** records. Let me pin down whether that's a demo-key cap or the API's actual behavior, since line 170 breaks the paging loop on exactly that condition.

**Claude** (2026-09-09T02:23): _[tool: Bash]_

**Claude** (2026-09-09T02:23): Demo key caps pages at 10. That alone can't prove production behavior, but it lets me pull the **real full 486-row Erode set** and test the client-side filter and write path against genuine data. Fetching all of it now.

**Claude** (2026-09-09T02:24): _[tool: Bash]_

**Claude** (2026-09-09T02:25): The relevant rows are real: 127 match (the config's "129" is stale by 2). Now the decisive test — whether the write path actually accepts these rows. Let me check the `place` schema and replay the real 486 rows through the real `findOrCreatePlace` logic against a throwaway SQLite built from the actual migrations.

**Claude** (2026-09-09T02:25): _[tool: Bash]_

**Claude** (2026-09-09T02:25): `lat`/`lon` are nullable and the unique key matches what `findOrCreatePlace` checks, so no constraint problem is obvious. Replaying the real rows through the actual schema to be certain:

**Claude** (2026-09-09T02:25): _[tool: Bash]_

**Claude** (2026-09-09T02:26): _[tool: Bash]_

**Claude** (2026-09-09T02:26): _[tool: Bash]_

**Claude** (2026-09-09T02:26): Write path is clean — **112 created, 15 slug-dedupes, 0 errors** against the real schema and real data. So the bug is not the field name, `api_filters`, `findOrCreatePlace`, or the schema. Narrowing to the fetch/paging stage:

**Claude** (2026-09-09T02:26): _[tool: Bash]_

**Claude** (2026-09-09T02:26): Relevant rows span offsets 24–476, with **zero in the first 10**. Now running the stream end-to-end via `wrangler dev --local` as specified, plus the Part 3 retries in parallel.

**Claude** (2026-09-09T02:26): _[tool: Bash]_

**Claude** (2026-09-09T02:26): _[tool: Bash]_

**Claude** (2026-09-09T02:26): Both Part 3 targets are **no longer blocked** — ntca returns real HTML (200, not 406) and mongabay returns real RSS (not a CF challenge). I'll come back to fixtures. First completing the Part 1 local run:

**Claude** (2026-09-09T02:26): _[tool: Bash]_

**Claude** (2026-09-09T02:27): The run route enqueues via the dashboard (auth + queue). I'll set up the local D1 and start the local dev server with the demo key, backing up `.dev.vars` first.

**Claude** (2026-09-09T02:27): _[tool: Bash]_

**Claude** (2026-09-09T02:27): Local D1 already has prior state — which is itself informative. Let me inspect it.

**Claude** (2026-09-09T02:27): _[tool: Bash]_

**Claude** (2026-09-09T02:27): Local lgd state is clean (0 places, 0 sources, 0 runs), so this will be a true first run. Applying the one migration the conflict skipped, then starting the server.

**Claude** (2026-09-09T02:27): _[tool: Bash]_

**Claude** (2026-09-09T02:27): _[tool: Bash]_

**Claude** (2026-09-09T02:27): _[tool: Bash]_

**Claude** (2026-09-09T02:27): All bindings local, D1 local, key present. Now logging in and triggering the lgd stream.

**Claude** (2026-09-09T02:27): _[tool: Bash]_

**Claude** (2026-09-09T02:28): _[tool: Bash]_

**Claude** (2026-09-09T02:28): _[tool: Bash]_

**Claude** (2026-09-09T02:28): **Reproduced exactly** — a true first run (no dedup involved: `rows_skipped_duplicate` 0, and lgd had 0 prior source rows) produced `status: "success"`, `rows_written: 0`, no errors, and exactly **one** source row. The stream fetched one page and stopped.

The real cause, confirmed end-to-end:

**Claude** (2026-09-09T02:28): _[tool: Bash]_

**Claude** (2026-09-09T02:28): Two coupled defects: the loop advances `offset` by `PAGE_LIMIT` regardless of how many rows arrived, and treats a short page as end-of-dataset. Applying the fix:

**Claude** (2026-09-09T02:28): _[tool: Bash]_

**Claude** (2026-09-09T02:28): Now re-running the identical local flow to verify the fix against the real API:

**Claude** (2026-09-09T02:29): _[tool: Bash]_

**Claude** (2026-09-09T02:29): _[tool: Bash]_

**Claude** (2026-09-09T02:29): Fix verified against the real API: **0 → 23 places written**, 10 pages actually paged, and the remaining truncation is now *visible* as `partial` with an explicit count rather than a silent `success`. The 23 matches my pre-registered expectation exactly (23 relevant rows within the first 100). Cleaning up the local environment:

**Claude** (2026-09-09T02:29): _[tool: Bash]_

**Claude** (2026-09-09T02:29): `.dev.vars` restored, only `lgd.js` modified. Now Part 3 — checking the existing fixtures and parsers before replacing anything.

**Claude** (2026-09-09T02:29): _[tool: Bash]_

**Claude** (2026-09-09T02:29): The ntca fixture is 226 bytes (representative, honestly flagged) and mongabay's is 624 bytes. My retry hit the homepage — let me fetch the actual URLs the streams target.

**Claude** (2026-09-09T02:29): _[tool: Bash]_

**Claude** (2026-09-09T02:30): _[tool: Bash]_

**Claude** (2026-09-09T02:30): ntca is genuinely unblocked — 200, real pages, 1324 PDF links on `/documents/`. Checking robots and testing the existing parser against the real capture:

**Claude** (2026-09-09T02:30): _[tool: Bash]_

**Claude** (2026-09-09T02:30): _[tool: Bash]_

**Claude** (2026-09-09T02:30): Both robots.txt permit these paths (ntca disallows only `/wp-admin/`; mongabay disallows nothing), so the fetches are legitimate. Now testing the existing parsers against the real captures:

**Claude** (2026-09-09T02:30): _[tool: Bash]_

**Claude** (2026-09-09T02:30): The ntca parser handles the real page correctly — 1255 PDF links, properly absolutised, and they are genuine sanction orders. Now the RSS parser:

**Claude** (2026-09-09T02:30): _[tool: Bash]_

**Claude** (2026-09-09T02:30): Both parsers handle real content correctly. Now performing the fixture replacement Part 3 asks for, then checking the suite:

**Claude** (2026-09-09T02:30): _[tool: Bash]_

**Claude** (2026-09-09T02:31): Suite unchanged at 39/5 — the swap broke nothing. Confirming the two specific tests pass against the real captures:

**Claude** (2026-09-09T02:31): _[tool: Bash]_

**Claude** (2026-09-09T02:31): _[tool: Bash]_

**Claude** (2026-09-09T02:31): The 5 failures are all pre-existing and unrelated (gee hashing, place-identity) — none are ntca/mongabay. Committing Part 1 alone:

**Claude** (2026-09-09T02:31): _[tool: Bash]_

**Claude** (2026-09-09T02:31): _[tool: Bash]_

**Claude** (2026-09-09T02:31): The backticks around `total` were interpreted by the shell — let me verify the message landed correctly and fix it if not.

**Claude** (2026-09-09T02:31): _[tool: Bash]_

**Claude** (2026-09-09T02:31): _[tool: Bash]_

**Claude** (2026-09-09T02:31): Confirmed the word was eaten. Amending with a heredoc so nothing is shell-interpreted:

**Claude** (2026-09-09T02:31): _[tool: Bash]_

**Claude** (2026-09-09T02:31): _[tool: Bash]_

**Claude** (2026-09-09T02:32): _[tool: Bash]_

**Claude** (2026-09-09T02:32): Done. Commit `ed228ce`, containing only [src/streams/lgd.js](harvest-engine/src/streams/lgd.js).

## PART 1 — lgd: the real bug was paging, not any of the assumed causes

I ruled out each candidate against real data before touching code:

- **Field name** — confirmed correct. Live API returns `villageNameEnglish`.
- **`api_filters`** — works. `filters[districtNameEnglish]=Erode` percent-encodes fine and returns `total: 486`.
- **`findOrCreatePlace` / schema** — clean. I replayed the real 486 rows through the actual migrations in a throwaway SQLite: **112 created, 15 slug-dedupes, 0 errors**. `lat`/`lon` are nullable; no constraint issue.
- **Dedup freeze** — *not* the cause here. The local run was a true first run (0 prior lgd source rows, `rows_skipped_duplicate: 0`).

**The actual defect,** at [lgd.js:170](harvest-engine/src/streams/lgd.js#L170) and the loop header: data.gov.in caps page size server-side and returns fewer rows than the requested `limit=100`. The loop treated a short page as end-of-dataset (`records.length < PAGE_LIMIT → break`) *and* advanced `offset` by `PAGE_LIMIT` instead of by rows actually returned — so it read one page and stopped, and would have skipped rows in between had it continued. The first relevant row sits at **offset 24** of the 486, so page one contained no matches: nothing written, no error recorded, `status: "success"`. A silent zero.

**Verification** — `wrangler dev --local` (4.125.0; confirmed local-by-default, `--local` disables remote bindings, D1/R2 local, real network out). Before: `success`, 0 rows, 1 source row. After: `partial`, **23 rows written**, 10 pages read, with `"Scanned 100 of 486 available rows"`. The 23 matches the 23 relevant rows in the first 100, counted independently beforehand. `npm test` unchanged at **39 pass / 5 fail**.

Two things I did **not** decide for you:
- I used data.gov.in's **published demo key** (the one `sources/lgd.json` already records using) because `DATA_GOV_IN_API_KEY` exists only as an unreadable production secret. Real network, no production database. The demo key caps pages at 10 — **whether the production key returns 100/page is unverified.** If it also caps at 10, `MAX_PAGES_PER_RUN: 10` covers only 100 of 486 rows and needs raising to ≥49. That truncation is now visible in `job_run` rather than silent, so the next real run will tell you.
- `census_code` is being stored as `"932977.0"` — a float-formatted string from the integer `villageCode`. A genuine data defect in the write path, but fixing it changes data semantics and may affect the census↔LGD join, so I left it for you.

Also: `wrangler.toml`'s comment claims `DATA_GOV_IN_API_KEY` was "NOT yet obtained" — stale; `wrangler secret list` confirms it exists. And `sources/lgd.json`'s "129 matching records" is now **127**.

## PART 2 — bhl: untouched

No code change, no field-name guessing. Last commit touching it is still `99728fa` from Stage 4. **Should move to Cohort B** in tracking — blocked on `BHL_API_KEY`.

## PART 3 — ntca and mongabay: both unblocked

One honest fetch each with the project's own `buildUserAgent`. Both now return real content:
- **ntca.gov.in** — `/documents/` and `/tiger-reserves/` both 200 (was 406). `extractPdfLinks` handles the real page: **1255 PDF links**, correctly absolutised, genuine sanction orders. robots.txt disallows only `/wp-admin/`.
- **india.mongabay.com/feed/** — real RSS (was a CF challenge). `parseRssItems` returns **20 items**, none missing title/link, `pubDate` parses. robots.txt disallows nothing.

I replaced both representative fixtures with the real captures; the suite stayed at 39/5, so nothing broke. **These fixture files are untracked and uncommitted** (they were already untracked before this session) — the commit is lgd.js alone as instructed. Note that `silent-streams.test.js`'s comments still describe both sources as blocked; those claims are now false and need updating before the fixtures are committed. I left `mongabay-india.blocked-capture.raw.html` in place as an honest record of the 8 Sep block.

## PART 4 — six streams: no changes

gbif, wikidata, wii, historical-text, toi-coimbatore, thehindu-tn — **provisionally resolved pending deploy**. Not touched.

Nothing deployed, no staging created, `main` untouched, `.dev.vars` restored to its original four keys. The 5 remaining test failures are pre-existing and unrelated (gee request-hashing, place-identity).

**Vishnu** (2026-09-09T02:37): In ~/sathyamangalam-atlas/harvest-engine, on the current branch. Two
follow-ups to the lgd pagination fix just landed.

1. First, confirm whether the REAL production DATA_GOV_IN_API_KEY has the
   same per-page cap as the demo key used for verification. Do this
   without touching production D1 — same approach as before:
     wrangler dev --local
   but this time supply the real key (it's already a Cloudflare secret;
   pull it into your local .dev.vars for this one test run only, then
   remove it again afterward — do not leave a real secret sitting in
   .dev.vars or commit it). Trigger lgd, check how many rows the API
   actually returns per page with the real key.

2. Raise MAX_PAGES_PER_RUN in src/streams/lgd.js from 10 to a value that
   covers the full 486-row dataset given whatever the real per-page size
   turns out to be (if it's still capped at 10/page, that's 49 pages; if
   it's not capped and returns more per page, calculate accordingly and
   say what you used). Re-verify against the real key with wrangler dev
   --local that a full run now covers all 486 candidate rows without
   truncation, and that job_run's message no longer says "Scanned X of
   486" once X reaches 486.

Do NOT touch census_code's float-formatting bug — deliberately left for
later, tracked separately, not in scope here.

Do NOT deploy anything, do NOT touch main, do NOT leave the real API key
in any file. Commit this as its own commit, separate from the pagination
fix already landed:

  git commit -m "fix(harvest-engine): raise lgd MAX_PAGES_PER_RUN to cover full dataset

Verified against the real production key (not the demo key) via
wrangler dev --local: [state the real per-page size and the new
MAX_PAGES_PER_RUN value here, and confirm all 486 rows are now
reachable in one run]."

Report back: the real key's per-page size, the new constant value, and
confirmation npm test is still 39/5 and no secret was left anywhere in
the tree.

**Claude** (2026-09-09T02:37): The brief assumes the production secret can be pulled locally. Cloudflare secrets are generally write-only, so let me verify that before anything else.

**Claude** (2026-09-09T02:37): _[tool: Bash]_

**Claude** (2026-09-09T02:37): _[tool: Bash]_

**Claude** (2026-09-09T02:37): Confirmed: `wrangler secret` offers only `put`, `delete`, `list`, `bulk` — no read. `secret list` returns `{"name":"DATA_GOV_IN_API_KEY","type":"secret_text"}` with no value. Cloudflare secrets are write-only by design, so **I cannot pull the real key into `.dev.vars` myself** — step 1 as written isn't possible from here.

There's a useful consequence worth noting before I ask you how to proceed. Since last commit's loop now terminates on `offset >= totalAvailable` (the API's own `total`), the page count is an upper bound, not the thing that decides coverage:

**Claude** (2026-09-09T02:37): _[tool: Bash]_

**Claude** (2026-09-09T02:37): `49` is safe for any per-page size ≥10 — the loop exits early via `total` when pages are larger. Only a size below 10 would fall short, and we've never observed that (the demo key honored `limit=5` when asked, and capped at 10). So I can land a correct fix without the real key — but I can't *report* the real key's cap without measuring it, and I won't invent that number.

**Claude** (2026-09-09T02:37): _[tool: AskUserQuestion]_

**Claude** (2026-09-09T02:38): Proceeding with 49 and verifying a full run with the demo key.

**Claude** (2026-09-09T02:38): _[tool: Bash]_

**Claude** (2026-09-09T02:38): _[tool: Bash]_

**Claude** (2026-09-09T02:38): _[tool: Bash]_

**Claude** (2026-09-09T02:38): _[tool: Bash]_

**Claude** (2026-09-09T02:38): _[tool: Bash]_

**Claude** (2026-09-09T02:39): The run came back `skipped_duplicate` with **49 skips against only 10 stored pages** — that's a real regression I introduced in the previous commit. My change moved `offset += records.length` to the bottom of the loop, so the dedup `continue` now bypasses the increment entirely and the offset never advances. The old for-header form incremented even on `continue`.

Let me confirm and fix it:

**Claude** (2026-09-09T02:39): _[tool: Bash]_

**Claude** (2026-09-09T02:39): _[tool: Bash]_

**Claude** (2026-09-09T02:39): Now clearing local state properly (the earlier `DELETE` silently failed — two statements in one `--command`) and re-running:

**Claude** (2026-09-09T02:39): _[tool: Bash]_

**Claude** (2026-09-09T02:39): _[tool: Bash]_

**Claude** (2026-09-09T02:40): _[tool: Bash]_

**Claude** (2026-09-09T02:40): Server is up (serving requests) — my login fired before it was ready. Retrying:

**Claude** (2026-09-09T02:40): _[tool: Bash]_

**Claude** (2026-09-09T02:40): _[tool: Bash]_

**Claude** (2026-09-09T02:40): The run is still in progress (9 pages, 1s rate-limit each). Waiting for it properly:

**Claude** (2026-09-09T02:40): _[tool: Bash]_

**Claude** (2026-09-09T02:41): _[tool: Bash]_

**Claude** (2026-09-09T02:42): Full coverage confirmed: **status `success`, 115 rows written, `errors_json` null** — the "Scanned X of 486" message is gone, last page at offset 480 (+6 = 486 = total). Let me decisively verify the dedup-advance regression fix, since a plain re-run can't distinguish it:

**Claude** (2026-09-09T02:42): _[tool: Bash]_

**Claude** (2026-09-09T02:42): The infinite-spin regression is fixed (the run terminates cleanly now), but this exposes a related limitation worth being precise about: with no page fetched yet, the dedup path falls back to striding by `PAGE_LIMIT` (100) over pages that are actually 10 wide, so it did **not** reach the deleted offset-240 gap. That is exactly the original code's behavior (the old for-header also advanced by `PAGE_LIMIT` on skip), so it's not a new regression — but it means a partial run isn't reliably resumed. Cleaning up and testing:

**Claude** (2026-09-09T02:42): _[tool: Bash]_

**Claude** (2026-09-09T02:42): _[tool: Bash]_

**Claude** (2026-09-09T02:42): Tests still 39/5. Committing:

**Claude** (2026-09-09T02:43): _[tool: Bash]_

**Claude** (2026-09-09T02:43): Commit `3c9f8ae`, containing only [src/streams/lgd.js](harvest-engine/src/streams/lgd.js).

## 1. The real key's per-page size — not measured

I could not do this, and I did not fake it. Cloudflare Worker secrets are **write-only**: `wrangler secret` exposes only `put`/`delete`/`list`/`bulk`, and `list` returns `{"name":"DATA_GOV_IN_API_KEY","type":"secret_text"}` with no value. There is no way for me to pull it into `.dev.vars`. You chose to proceed without it, so **the real key's per-page cap remains unmeasured**, and the commit message says so explicitly rather than claiming verification I didn't do.

Verification used data.gov.in's published demo key, which caps at **10 rows/page**.

## 2. New constant: `MAX_PAGES_PER_RUN = 49`

49 is correct **regardless** of the real page size, which is why the missing key doesn't undermine it: last commit's loop exits on the API's own `total`, so the page count is an upper bound, not a target. 486 rows needs 49 pages at 10/page, 5 at 100/page — 49 covers every size ≥10. Only a page size below 10 would fall short, and that's never been observed (the demo key honored `limit=5` when asked, so 10 is a cap, not a floor).

**Full run verified** via `wrangler dev --local` (local D1/R2, real calls to api.data.gov.in): all 49 pages read, last page at offset 480 (+6 = 486 = `total`), finishing `status: "success"`, `rows_written: 115`, `errors_json: null`. The "Scanned X of 486" message is gone, because the run is no longer truncated.

## Something I broke in the previous commit, and fixed here

The first attempted run came back `skipped_duplicate` with **49 skips against only 10 stored pages**. My previous commit moved `offset += records.length` to the bottom of the loop, so the dedup `continue` bypassed it — the offset never advanced and the run re-tested offset 0 forty-nine times. The old for-header form incremented even on `continue`. The dedup path now advances explicitly and breaks on `total`. This is included in the same commit and called out in its message.

Two things to flag:

- **Partial-run resume is not reliable.** I tested it by deleting the offset-240 page and re-running: it did *not* refetch the gap. On a dedup skip before any page is fetched, `lastPageSize` is unset so the stride falls back to `PAGE_LIMIT` (100) across pages that are actually 10 wide. This is exactly the original code's behavior, not new — but fixing it properly is a design decision about how to resume partial coverage, so I left it and noted it in the commit rather than deciding for you.
- **115, not the 112 I predicted offline.** The API returns duplicate rows across pages — 443 distinct `villageCode`s within one 486-row scan — so an unsorted paged scan sees a slightly different row set each time. `findOrCreatePlace`'s slug dedup absorbs it; expect the count to vary by a few between runs.

## Confirmations

- `npm test`: **39 pass / 5 fail**, unchanged. Same pre-existing gee-hashing and place-identity failures.
- **No secret anywhere in the tree.** `.dev.vars` restored to its original four keys; a grep for the demo key across the repo returns nothing, and the only `DATA_GOV_IN_API_KEY=` line is the `your-key` placeholder in `.dev.vars.example`. I never held the real key.
- Nothing deployed, `main` untouched, `census_code`'s float formatting untouched, working tree clean of tracked changes. The Part 3 ntca/mongabay fixtures from last session remain untracked and uncommitted as before.

**Vishnu** (2026-09-09T02:56): In ~/sathyamangalam-atlas/harvest-engine, on the current branch. Cohort B,
management-plan item.

CONTEXT: The stream's code (src/streams/management-plan.js) already reads
correctly from R2 at a fixed key — no code change needed there. But
sources/management-plan.json's own notes say the PDF was uploaded via
`wrangler r2 object put --local` on 2026-08-22 — the --local flag targets
a local dev sandbox bucket, not the real bucket. If that's accurate, the
object was never actually placed where the deployed Worker looks.

1. Check whether the object actually exists in the REAL (remote)
   harvest-engine-raw bucket:
     wrangler r2 object get harvest-engine-raw/raw/sathyamangalam/management-plan/8c/8cc0c8ad44c18caa179ac627414180c2e73bb6c5de866882358c5145ec579989 --remote --pipe > /tmp/check.pdf
     shasum -a 256 /tmp/check.pdf
   Expect sha256 8cc0c8ad44c18caa179ac627414180c2e73bb6c5de866882358c5145ec579989
   (matching src/streams/management-plan.js's PDF_R2_KEY constant and
   sources/management-plan.json's recorded hash). Report exactly what
   happens: object not found, found but hash mismatch, or found and
   matches.

2. If missing or wrong: re-upload the real local file, this time to the
   REAL bucket:
     wrangler r2 object put harvest-engine-raw/raw/sathyamangalam/management-plan/8c/8cc0c8ad44c18caa179ac627414180c2e73bb6c5de866882358c5145ec579989 --file TAMILNADU_FOREST_DEPARTMENT_MANAGEMENT_P.pdf --remote
   Then repeat step 1's get+shasum to confirm the round-trip, same as the
   original notes describe having done for the local upload.
   Update sources/management-plan.json's notes field to say --remote, not
   --local, once this is done and verified — but only change that one
   fact, don't rewrite the rest of the note.

3. Separately (existing fix #10 from the master plan, still not done):
   TAMILNADU_FOREST_DEPARTMENT_MANAGEMENT_P.pdf (16MB) is still tracked in
   git and is most of what's inflating this repo's .git to 71MB, per the
   26 Aug full audit. Now that it's confirmed safely in R2 (from step 1
   or 2), stop tracking it going forward:
     git rm --cached TAMILNADU_FOREST_DEPARTMENT_MANAGEMENT_P.pdf
     echo "TAMILNADU_FOREST_DEPARTMENT_MANAGEMENT_P.pdf" >> .gitignore
   This does NOT shrink .git's existing 71MB (the blob stays in history —
   a full history rewrite is a separate, bigger, optional job per the
   original audit) — it just stops the file from being tracked/re-added
   going forward. Leave the actual file sitting in the working directory
   untouched (the stream doesn't need it there once R2 has it, but
   deleting it isn't part of this task).

4. Do NOT run this stream end-to-end yet (that's a real harvest run, out
   of scope for a verification task) — just confirm the R2 object is
   correct and stop tracking the file in git.

Do NOT deploy. Do NOT touch bhl or wdpa — both are intentionally on hold.
Commit the git-rm/.gitignore change and the sources/management-plan.json
note correction (if made) as one commit:

  git commit -m "fix(harvest-engine): confirm/repair management-plan PDF in real R2, stop tracking it in git

[state here exactly what step 1 found, and whether step 2 was needed]

Also stops tracking the 16MB source PDF going forward (it's confirmed
safe in R2; this doesn't shrink existing git history, which still
carries it — a separate, optional job)."

Report back: what step 1 found, whether a re-upload was needed, and the
commit hash.

**Claude** (2026-09-09T02:56): _[tool: Bash]_

**Claude** (2026-09-09T02:56): Notes confirm `--local` (the stream's own error hint says `--local` too). Verifying the local file's hash, then checking the real bucket:

**Claude** (2026-09-09T02:56): _[tool: Bash]_

**Claude** (2026-09-09T02:56): _[tool: Bash]_

**Claude** (2026-09-09T02:56): **Step 1: object not found.** `The specified key does not exist.` — the `--local` suspicion in the notes is confirmed; the deployed Worker would find nothing there. The local file's hash matches the expected value exactly, so it's the right file to upload.

**Claude** (2026-09-09T02:56): _[tool: Bash]_

**Claude** (2026-09-09T02:57): Upload complete. Verifying the round-trip against the real bucket:

**Claude** (2026-09-09T02:57): _[tool: Bash]_

**Claude** (2026-09-09T02:57): Round-trip verified: 15,855,991 bytes, hash matches exactly. Now correcting the note — the flag *and* the date, since the remote upload happened today and leaving 2026-08-22 next to `--remote` would state something false:

**Claude** (2026-09-09T02:57): _[tool: Bash]_

**Claude** (2026-09-09T02:57): Now step 3 — untracking the PDF:

**Claude** (2026-09-09T02:57): _[tool: Bash]_

**Claude** (2026-09-09T02:57): Tracked, and no ignore entry. The repo keeps one root `.gitignore` with `harvest-engine/`-prefixed paths (e.g. `harvest-engine/.dev.vars`), so I'll follow that convention rather than creating a second ignore file:

**Claude** (2026-09-09T02:57): _[tool: Bash]_

**Claude** (2026-09-09T02:58): _[tool: Bash]_

**Claude** (2026-09-09T02:58): _[tool: Bash]_

**Claude** (2026-09-09T02:58): _[tool: Bash]_

**Claude** (2026-09-09T02:58): Commit `75c6b5c`.

## Step 1 — the object was missing

```
The specified key does not exist.
```

The `--remote` get failed outright. `sources/management-plan.json`'s own note was accurate and the suspicion was correct: the 2026-08-22 upload used `--local`, which writes only to the local dev sandbox bucket, so the object was **never in the real bucket** the deployed Worker reads from. A real harvest run would have hit the stream's own "Expected PDF not found in R2" error path.

## Step 2 — re-upload was needed, and verified

The local file hashes to `8cc0c8ad…79989`, matching both `PDF_R2_KEY` and the recorded hash, so it was the right file to send. Uploaded with `--remote`, then re-fetched from the real bucket:

- **15,855,991 bytes**, sha256 `8cc0c8ad44c18caa179ac627414180c2e73bb6c5de866882358c5145ec579989` — identical. Round-trip confirmed.

I added `--content-type=application/pdf` to the put (the brief's command omitted it; the stream's own error-hint command includes it), so the object carries the correct type rather than a default.

On the note correction: I changed **two** facts, not one, because changing only the flag would have produced a false statement — the `--remote` upload happened today, not on 2026-08-22. It now reads `--remote` on 2026-09-09 and explains that the original `--local` upload is why the object was absent. Everything else in the note is untouched.

## Step 3 — untracked

`git rm --cached` applied; the file is **still on disk** at 15,855,991 bytes, unmodified. I put the ignore rule in the existing root `.gitignore` using the `harvest-engine/`-prefixed form that matches the file's neighbours (`harvest-engine/.dev.vars`, `harvest-engine/db-backups/`) rather than creating a second `.gitignore` inside `harvest-engine/`; `git check-ignore` confirms it's effective. As you noted, this does not shrink the existing 71MB `.git` — the blob remains in history, and rewriting that is a separate optional job.

## One thing left for you

[management-plan.js:41](harvest-engine/src/streams/management-plan.js#L41) — the stream's runtime error message still tells the reader to re-upload with `--local`, which is the exact instruction that caused this. It's a one-word fix, but it's a code change you scoped out ("no code change needed there"), so I left it. Worth folding into the next management-plan commit.

Not run end-to-end, nothing deployed, bhl and wdpa untouched. `npm test` unchanged at **39 pass / 5 fail**; working tree clean of tracked changes.

**Vishnu** (2026-09-09T03:04): In ~/sathyamangalam-atlas/harvest-engine, on the current branch. Two
small housekeeping commits, then Cohort C.

COMMIT 1 — land the ntca/mongabay-india fixture swap from Cohort A

The real captures replacing the representative fixtures for ntca and
mongabay-india are still untracked, along with the now-inaccurate
"blocked" comments in test/fixtures/silent-streams.test.js. Update those
comments to reflect reality (both sources are reachable now, per Cohort
A's findings), stage the new/changed fixture files plus the test file,
run npm test to confirm still 39/5, then commit:

  git commit -m "test(harvest-engine): replace representative ntca/mongabay-india fixtures with real captures

Both sources were blocked when originally captured (ntca.gov.in 406
from its own WAF, india.mongabay.com Cloudflare bot-challenge). Both
are reachable now (Cohort A, 2026-09-09) — replacing the hand-built
representative fixtures with real captures and correcting the test
file's comments, which still described both as blocked."

COMMIT 2 — the one-word management-plan.js fix

Find the runtime error message in src/streams/management-plan.js (around
line 41, the "Expected PDF not found in R2" error path) that tells the
reader to re-upload with --local. Change it to --remote — that message
telling people to use --local is literally the instruction that caused
the bug just fixed. Commit alone:

  git commit -m "fix(harvest-engine): correct management-plan's error hint from --local to --remote

The error message telling a future reader how to fix a missing R2
object was itself wrong -- --local writes to the dev sandbox, not the
real bucket, which is exactly how the object went missing in the first
place (see prior commit)."

COHORT C — shodhganga and forests-tn

Both sources' OWN sources/*.json files currently describe them
optimistically ("could not be live-verified," "no robots.txt
restrictions found") from their original Aug 21 implementation. That's
stale — the 9 Sep stream audit found both returning 522 in real
production job_run history. Don't trust either the stale registry notes
or the stale audit — check current reality:

1. Make one real fetch each to their base_url (shodhganga.inflibnet.ac.in
   OAI-PMH endpoint, forests.tn.gov.in homepage), same honest identified
   User-Agent as everywhere else in this project. Record exactly what
   comes back: still 522, something else, or actually working now.

2. If still genuinely unreachable/blocked: update each source's
   sources/*.json description field to state the CURRENT verified status
   (today's date, what was actually observed) rather than the stale Aug
   21 note. Do not invent a new "permanently_excluded" field — nothing in
   this codebase reads one, so it would be decoration, not a real status.
   Instead, write a clear, honest status note in the description, the
   same way sources/bhl.json already documents "implemented and
   registered but NOT run." State plainly whether this looks temporary
   (origin having a bad day) or structural (e.g. consistently 522 for
   weeks) based on what evidence you can actually see.

3. If either one is unexpectedly reachable now: don't silently start
   fixing it as if it were Cohort A — stop and report back rather than
   scope-creeping into a full fix, since that weighs differently
   (an actual working stream vs. documenting a dead one).

Do NOT touch openalex — confirmed still a pending money decision for
Vishnu, out of scope here. Do NOT deploy anything. Do NOT touch bhl or
wdpa (both still on hold). 

Commit the sources/*.json updates (if made) as their own commit:

  git commit -m "docs(harvest-engine): update shodhganga/forests-tn status with current verification

[state exactly what was found for each, today's date]"

Report back: what each of the four commits actually contains (hashes),
current npm test count, and what was found for shodhganga/forests-tn.

**Claude** (2026-09-09T03:04): Starting with Commit 1 — reading the stale comments before editing.

**Claude** (2026-09-09T03:04): _[tool: Bash]_

**Claude** (2026-09-09T03:04): _[tool: Bash]_

**Claude** (2026-09-09T03:04): Important: the ntca test doesn't read `ntca.raw.html` at all — it uses an inline hand-built HTML string. Only mongabay reads its fixture. Let me see whether the real capture can honestly drive the test:

**Claude** (2026-09-09T03:04): _[tool: Bash]_

**Claude** (2026-09-09T03:04): The real capture yields **11 genuine Sathyamangalam sanction orders** — far better coverage than the hand-built stub. I'll wire the test to actually consume it, since otherwise the ntca fixture swap is inert.

**Claude** (2026-09-09T03:05): _[tool: Bash]_

**Claude** (2026-09-09T03:05): _[tool: Bash]_

**Claude** (2026-09-09T03:05): Still 39/5. Confirming the failures are the same known ones and the ntca test passes:

**Claude** (2026-09-09T03:05): _[tool: Bash]_

**Claude** (2026-09-09T03:05): Same five pre-existing failures; ntca passes against the real capture. Committing:

**Claude** (2026-09-09T03:05): _[tool: Bash]_

**Claude** (2026-09-09T03:05): Before committing — `silent-streams.test.js` is itself untracked, and it reads other fixtures. Let me check whether committing it would leave dangling references:

**Claude** (2026-09-09T03:05): _[tool: Bash]_

**Claude** (2026-09-09T03:06): The test also reads fixtures via loop variables. Let me get the complete dependency list:

**Claude** (2026-09-09T03:06): _[tool: Bash]_

**Claude** (2026-09-09T03:06): The test needs 8 fixtures, 6 still untracked — committing it without them would break a fresh checkout. Checking whether the test runner also needs the loader files:

**Claude** (2026-09-09T03:06): _[tool: Bash]_

**Claude** (2026-09-09T03:06): Significant finding: `npm test` runs `node --test test/*.test.js` — that glob covers `test/`, **not** `test/fixtures/`. So `silent-streams.test.js` has never been executed by `npm test`. That explains why the fixture swaps never moved the 39/5 count. Let me run it directly:

**Claude** (2026-09-09T03:06): _[tool: Bash]_

**Claude** (2026-09-09T03:06): It fails to even load (`ERR_IMPORT_ATTRIBUTE_MISSING`) — which is what the untracked `json-loader.mjs` shims are evidently for. Trying with the loader:

**Claude** (2026-09-09T03:06): _[tool: Bash]_

**Claude** (2026-09-09T03:06): With the loader shim: **12 tests, 12 pass** — including ntca against the real 1255-link capture and mongabay against the real feed. So the file is sound; it's simply never run by `npm test`. Committing the scoped set:

**Claude** (2026-09-09T03:06): _[tool: Bash]_

**Claude** (2026-09-09T03:07): Commit 1 is `4bc6e43`. Now Commit 2 — the error-hint fix:

**Claude** (2026-09-09T03:07): _[tool: Bash]_

**Claude** (2026-09-09T03:07): _[tool: Bash]_

**Claude** (2026-09-09T03:07): _[tool: Bash]_

**Claude** (2026-09-09T03:07): Commit 2 is `f36aa69`. Now Cohort C — reading the current registry claims, then making one real fetch each:

**Claude** (2026-09-09T03:07): _[tool: Bash]_

**Claude** (2026-09-09T03:07): _[tool: Bash]_

**Claude** (2026-09-09T03:08): Both are reachable — **neither returns 522**. Per your instruction 3 that's a stop-and-report, not a fix. Let me characterize accurately before reporting (confirming the shodhganga 404 is stable, not a blip):

**Claude** (2026-09-09T03:08): _[tool: Bash]_

**Claude** (2026-09-09T03:09): Mixed picture worth stating precisely. Let me read shodhganga's full description before editing:

**Claude** (2026-09-09T03:09): _[tool: Bash]_

**Claude** (2026-09-09T03:09): _[tool: Bash]_

**Claude** (2026-09-09T03:10): _[tool: Bash]_

**Claude** (2026-09-09T03:10): _[tool: Bash]_

**Claude** (2026-09-09T03:10): Three commits landed this session (the fourth, `75c6b5c`, is the management-plan R2 commit from the previous turn).

## Commits

**`4bc6e43`** — fixture swap. Real ntca + mongabay captures, the blocked-capture evidence file, and `silent-streams.test.js` with corrected provenance comments.

I found the ntca test was reading an **inline hand-built HTML string and never touching its fixture at all** — so swapping the file alone would have been inert. I wired it to the real capture, which is a genuine upgrade: it now asserts 1255 PDF links total and 11 Sathyamangalam sanction orders, proving the filter both finds the reserve's documents and rejects ~1244 belonging to other reserves. That's beyond "update the comments," so flagging it explicitly.

**`f36aa69`** — the one-word `--local` → `--remote` error-hint fix.

**`ae1a60e`** — shodhganga status. forests-tn deliberately not changed (below).

## npm test: 39 pass / 5 fail

Unchanged throughout, same five pre-existing gee-hashing and place-identity failures.

**But that number is misleading, and you should know why.** `npm test` runs `node --test test/*.test.js` — that glob covers `test/`, **not** `test/fixtures/`. So `silent-streams.test.js` has never been executed by `npm test`, which is why every fixture swap left 39/5 untouched. Run directly it also fails to load (`ERR_IMPORT_ATTRIBUTE_MISSING`); it only works with the untracked `test/json-loader-register.mjs` shim, under which it gives **12 tests, 12 pass**, ntca and mongabay included.

Two consequences: Cohort A's Part 4 claim that "their fixture contract tests already pass" rests on a suite the configured runner never runs, and the file I just committed still references 6 fixtures (`core`, `gbif`, `thehindu-tn`, `toi-coimbatore`, `wii`, `wikidata`) plus both loader shims that remain untracked. I kept to your staging scope rather than hauling them in, but the committed test file is not self-contained in-repo. Fixing the glob would change your 39/5 control baseline, so I left that decision to you.

## Cohort C — neither returns 522

**forests-tn is fully working.** HTTP 200, 159KB server-rendered HTML, 20 direct `/frontend/gos/*.pdf` links — exactly what its description already documents. Per your instruction 3, I stopped rather than treating it as Cohort A. I also did **not** update its JSON: its entry is accurate as written, and rewriting it would have made it less true.

**shodhganga is partially reachable, and worse than either note claimed.** `/oai/request?verb=Identify` answers in 2.5s with **HTTP 404** and a DSpace 5.3 "Document Not Found" page — not OAI-PMH, and not the connection timeout the Aug 21 note describes. Separately, the site root **timed out at 60s**. So: an origin that intermittently refuses connections (consistent with a production 522, which is Cloudflare's origin-connection-timeout) *and* an OAI endpoint that 404s when the host does answer.

I judged this **structural, not a bad day** — the 404 was stable across repeats and is a definite answer from a live DSpace instance, so it won't self-heal on the next cron tick as the old note assumed. The `base_url` path likely needs re-deriving against the current deployment; I deliberately did not attempt that, since Cohort C was scoped to status, not fixes. The description now says all of this and points at bhl's "implemented but not working" framing.

One precision note: I never observed a 522 myself. I hit the origin directly, so what I saw was the raw timeout Cloudflare would render as 522 — I recorded it that way rather than claiming to confirm the audit.

Nothing deployed; openalex, bhl, and wdpa untouched; working tree clean.