**Vishnu** (2026-09-07T03:52): Read these project docs first, in this order:
1. sathyamangalam/session-close-2026-09-06-full-freeze.md
2. sathyamangalam/atlas-scope-locked-2026-09-06.md
3. sathyamangalam/sensitive-species-coordinate-policy.md
4. sathyamangalam/harvest-engine-dedup-freeze-and-pause-2026-09-05.md

Then work through these steps IN ORDER overnight. Do not skip ahead — do not
merge dev into main, do not deploy anything anywhere (dashboard, harvest-engine,
public site), do not re-enable the production cron. Everything happens on the
dev branch and/or staging database only. If any step is blocked by a decision
only Vishnu can make, stop that step, write down exactly what's blocked, and
move to the next one instead of guessing.

STEP 0 — Check status of last unfinished prompt
Check the current state of the dev branch for three things asked for earlier:
1. A fix to the place-matching key so it includes wikidata_id (prevents the
   Sathyamangalam/Satyamangalam duplicate).
2. A basic species browser page at /life.
3. Tier-less documents (relevance_tier IS NULL) separated into their own
   visible section instead of being hidden.
Report which are (a) fully done and merged into dev, (b) partially done, or
(c) not started. Show actual commits/diffs as proof, not just a claim. Skip
any later step below that this shows is already done.

STEP 1 — Database schema design pass (the main deliverable tonight)
Design (as a written proposal — do NOT run any migration yet) additions to
data/atlas.db's schema to support:
1. Species split by category (birds, mammals, insects, snakes/reptiles, trees,
   plants, fungi, fish, amphibians) — decide category-column vs separate
   tables and justify it.
2. An occurrences table making the 78,467 existing rows individually
   browsable: species, place, date, source, coordinates (respecting the
   coordinate-coarsening rule for sensitive species in production).
3. A population_history table tracking a number over time per species with a
   source per data point (e.g. tigers 8-10 in 2009 -> 112 in 2024) — extend
   the existing empty claim table rather than duplicating it; say which you
   chose and why.
4. A relationships table for species-to-species (eats/eaten-by) and
   species-to-habitat links.
5. A photos table linking already-harvested iNaturalist CC-licensed images to
   species records.
6. Places-as-hubs: every place shows every species/document/history/community
   record linked to it — design the join structure avoiding n+1 explosions.
7. A completeness field/table (last verified date, missing-field flags)
   attachable to species, place, and occurrence records.
8. A contributions table for future citizen input (sightings, photos,
   corrections) with a moderation_state field (pending/approved/rejected) and
   a reference to submitter + target record. No UI needed tonight, just the
   schema.
Write the full proposal to sathyamangalam/schema-proposal-[today's date].md
with table definitions, foreign keys, indexes, and a migration plan (order of
changes, what could break). Do NOT apply any migration. Stop and wait for
review.

STEP 2 — Gazetteer fix (only if Step 0 shows it's not already done)
Confirm current place count from the gazetteer stream against staging. If not
already fixed, propose a wider Overpass query as a diff, test against staging
only, report before/after place counts (target 500-2,000+, current is 93).

STEP 3 — Dedup-freeze bug
Work through the 9-item fix list in
harvest-engine-dedup-freeze-and-pause-2026-09-05.md on dev/staging only. Do
not re-enable the production cron. Report pass/fail per item with test
evidence.

STEP 4 — Sensitivity matcher root-cause fix
Fix the binomial-vs-trinomial species name matching bug in the harvest-engine
sensitivity matcher (root cause behind the coordinate-leak patch). Test
against a known trinomial sensitive-species record from staging. Leave the
existing sync-time coordinate-coarsening patch in place as defense in depth.

STEP 5 — relevance_tier NULL documents
Design and implement a rule-based first-pass tiering for the 1,839 documents
with relevance_tier IS NULL. Anything not confidently tierable goes into a
new visible "unclassified" section (never stays invisible). Test against
staging. Do not deploy.

MORNING REPORT
For every step: what was done, what was skipped and why, what needs Vishnu's
decision. Write this to sathyamangalam/overnight-run-results-[date].md.

**Claude** (2026-09-07T03:52): I'll start by reading the four project docs in the order given.

**Claude** (2026-09-07T03:52): _[tool: Bash]_

**Claude** (2026-09-07T03:52): There's no top-level `sathyamangalam/` directory. Let me find the docs.

**Claude** (2026-09-07T03:52): _[tool: Bash]_

**Claude** (2026-09-07T03:52): _[tool: Bash]_

**Claude** (2026-09-07T03:52): None of the four docs exist at those paths. Let me check whether they exist anywhere in git history or other branches before concluding.

**Claude** (2026-09-07T03:53): _[tool: Bash]_

**Claude** (2026-09-07T03:53): The four docs don't exist anywhere in the repo, any branch, or git history. Let me search by content in case they live under different names, and check the untracked directories.

**Claude** (2026-09-07T03:53): _[tool: Bash]_

**Claude** (2026-09-07T03:53): Confirmed: all four docs are absent — not in the working tree, any branch, or git history. That's a real blocker I'll record rather than guess around. But the substance they'd describe exists in code (`harvest-engine/src/lib/coarsen.js`, `sensitive-species.json`, `sync_d1_to_atlas.py`), so I'll recover the effective rules from the live system. Starting STEP 0.

**Claude** (2026-09-07T03:53): _[tool: Bash]_

**Claude** (2026-09-07T03:53): Item 1 is done and well-documented. Now items 2 and 3.

**Claude** (2026-09-07T03:53): _[tool: Bash]_

**Claude** (2026-09-07T03:53): _[tool: Bash]_

**Claude** (2026-09-07T03:53): All three STEP 0 items are verified done. Noting one thing: `origin/dev` is one commit ahead of local `dev`. Let me confirm, then move to the main deliverable — the schema.

**Claude** (2026-09-07T03:53): _[tool: Bash]_

**Claude** (2026-09-07T03:54): Two corrections to the brief already: `occurrence` already exists with exactly 78,467 rows, and the `claim` table is **not empty** — it has 83 rows. Let me get the full schema.

**Claude** (2026-09-07T03:54): _[tool: Bash]_

**Claude** (2026-09-07T03:54): Now I have the real schema. Let me ground the design decisions in the actual data distributions rather than assumptions.

**Claude** (2026-09-07T03:54): _[tool: Bash]_

**Claude** (2026-09-07T03:54): `class_` is messier than it looks — 825 taxa have both `kingdom` and `class_` NULL, and `class_` sometimes holds a kingdom value. Also note `rank` has case-duplicated values (`SPECIES`/`species`, `subspecies`/`SUBSPECIES`) — directly relevant to STEP 4. Let me check the rest.

**Claude** (2026-09-07T03:54): _[tool: Bash]_

**Claude** (2026-09-07T03:54): This is decisive for item 3: the `claim` table **already contains** the exact example from the brief — claim 10 (`tigers`=112, as_of 2024-25) and claim 11 (`tigers`=8-10, as_of 2009). Let me check referential integrity and the occurrence/photo data.

**Claude** (2026-09-07T03:55): _[tool: Bash]_

**Claude** (2026-09-07T03:55): Important: `claim` rows for `subject_type='taxon'` store **scientific names, not slugs** — 0 of 37 resolve against `taxon.slug`. And two of them are trinomials (`Panthera tigris tigris`, `Panthera pardus fusca`) — the exact shape of the STEP 4 bug. Let me verify.

**Claude** (2026-09-07T03:55): _[tool: Bash]_

**Claude** (2026-09-07T03:55): Critical: the 791 `harvest-engine:inaturalist` rows have **zero** media, and their `licence` field holds prose ("Varies per observation…") rather than a licence code. Let me quantify what item 5 actually has to work with, and check the sensitive-species coordinate exposure.

**Claude** (2026-09-07T03:55): _[tool: Bash]_

**Claude** (2026-09-07T03:55): **Live coordinate leak found.** 13 occurrences of sensitive taxa are uncoarsened, and the affected taxa are disproportionately trinomials (`Elephas maximus indicus` 5/11 uncoarsened, `Panthera pardus fusca` 2/3). Let me identify the mechanism before asserting a cause.

**Claude** (2026-09-07T03:56): _[tool: Bash]_

**Claude** (2026-09-07T03:56): Correction to what I said a moment ago: this is **not** the trinomial bug. The split is purely by pipeline — all 13 uncoarsened rows are `harvest-engine:*`, and all 13 harvest-engine sensitive rows are uncoarsened, while all 385 legacy `gbif`/`inaturalist` rows are coarsened to 0.05°. That's the documented `--raw-coordinates` dev flag behaving as designed. The trinomial correlation was a coincidence of which taxa the newer harvest happened to cover.

Also worth noting: those iNaturalist rows carry `coord_uncertainty_m=22000` (iNaturalist's own obscuration), and the three eBird rows share one identical hotspot centroid — so they're generalized at source too. Let me read the actual coarsening and matcher code.

**Claude** (2026-09-07T03:56): _[tool: Bash]_

**Claude** (2026-09-07T03:56): Now the STEP 4 bug is concretely visible: the config lists **8 binomials**, but the taxon table holds trinomials (`Panthera tigris tigris`, `Panthera pardus fusca`, `Elephas maximus indicus`) that an exact-string match would miss. Let me find the matcher itself.

**Claude** (2026-09-07T03:56): _[tool: Bash]_

**Claude** (2026-09-07T03:56): Root cause confirmed at [taxon.js:12-14](harvest-engine/src/lib/taxon.js#L12-L14) — exact `Set` membership on lowercased full name. `"elephas maximus indicus"` is not in the set, so `is_sensitive=0` and the trigger never fires. Let me read the trigger that does the enforcing.

**Claude** (2026-09-07T03:56): _[tool: Bash]_

**Claude** (2026-09-07T03:56): Now the full chain is clear. The D1 trigger is correct but gated entirely on `taxon.is_sensitive` — so a trinomial gets `is_sensitive=0`, the *passthrough* trigger fires instead, and exact coordinates land in `public_lat/public_lon`. Let me read the sync-time patch that's meant to catch this.

**Claude** (2026-09-07T03:56): _[tool: Bash]_

**Claude** (2026-09-07T03:57): The sync already implements a three-way `SensitivityOracle` with **binomial prefix matching** — that's the defense-in-depth layer, and it's why atlas.db correctly flags the trinomials. Let me read it as the reference implementation to port upstream.

**Claude** (2026-09-07T03:57): _[tool: Bash]_

**Claude** (2026-09-07T03:57): The docstring even records the expected counts (5 elephant, 2 leopard, 1 vulture), matching what I measured. Now let me check what actually links to `place` today — central to item 6.

**Claude** (2026-09-07T03:57): _[tool: Bash]_

**Claude** (2026-09-07T03:57): **Structural finding that reframes item 6: nothing references `place(id)` at all.** Zero foreign keys point to it — `place` is an isolated island that only points outward at `source`. The single path in is the soft string join `claim.subject_key = place.slug`. So places-as-hubs isn't an n+1 optimization problem yet; the links don't exist. Let me check the gazetteer and staging setup.

**Claude** (2026-09-07T03:57): _[tool: Bash]_

**Claude** (2026-09-07T03:57): Production cron is already `crons = []` (off) — I'll leave it. There's a real staging D1. Let me check whether I can actually reach it, since STEPs 2/3/5 all require staging tests.

**Claude** (2026-09-07T03:57): _[tool: Bash]_

**Claude** (2026-09-07T03:57): _[tool: Bash]_

**Claude** (2026-09-07T03:57): Authenticated with D1 write access. Let me query staging read-only to establish the STEP 2 baseline.

**Claude** (2026-09-07T03:57): _[tool: Bash]_

**Claude** (2026-09-07T03:58): Staging is a thin sandbox: 59 places, 0 taxa/occurrences/documents. Let me read the Overpass stream and check the production baseline.

**Claude** (2026-09-07T03:58): _[tool: Bash]_

**Claude** (2026-09-07T03:58): Key finding for STEP 2: the Overpass query has **already been widened** (2026-08-26), with a measured ceiling recorded in the code — 258 named OSM elements of *any* tag within the reserve bbox. That puts the STEP 2 target of "500–2,000+" out of reach from Overpass alone. Let me verify that ceiling empirically rather than trust the comment.

**Claude** (2026-09-07T03:58): _[tool: Bash]_

**Claude** (2026-09-07T03:58): Note an internal cross-check already available: claim 17 records `villages_zone_of_influence = 136` and claim 22 records `inhabited_villages = 184` (1887). Let me measure Overpass live, pre-registering expectations.

**Claude** (2026-09-07T03:58): _[tool: Bash]_

**Claude** (2026-09-07T03:59): Reproduces the recorded measurements (43 vs comment's 42; 60 exactly). Now the ceiling and a candidate wider query.

**Claude** (2026-09-07T03:59): _[tool: Bash]_

**Claude** (2026-09-07T03:59): Confirmed exactly 258 — the comment is precise. But the composition matters: the ceiling is mostly roads (33), boundaries (14), hospitals (19), even shops and banks. Let me compute precisely what a defensibly wider query could capture, and what the current one misses.

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): Precise result: a defensibly wider query captures **82 vs the current 60** — the biggest misses are villages mapped as *areas* (`way place=village`, invisible to `node["place"]`) and temples as buildings. Let me validate the actual query syntax live.

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): That's a transient Overpass dispatcher failure returning **HTTP 200 with an error body** — which reveals a real bug. Let me check `findExistingSource`, because the stream inserts the `source` row (and its `request_hash`) *before* parsing.

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:01): Confirmed the mechanism. `findExistingSource` keys on `reserve_id + request_hash` alone, with no regard for whether that run succeeded. So a 200-with-error-body inserts the source row, the parse then fails, and every later run early-returns **"success"** while writing nothing — permanently. Let me check whether this has already happened.

**Claude** (2026-09-07T04:01): _[tool: Bash]_

**Claude** (2026-09-07T04:01): **The dedup freeze is live and confirmed in production.** Every `overpass` run from 2026-09-01 to 2026-09-05 reports `success` with `rows_written: 0, rows_skipped_duplicate: 1` — 12+ consecutive no-ops reported as success. Let me establish whether source 541 is the poisoned row.

**Claude** (2026-09-07T04:01): _[tool: Bash]_

**Claude** (2026-09-07T04:02): Correction: source 541 is **not** poisoned — it legitimately wrote 40 places. So the mechanism is different from my hypothesis: a static query's `request_hash` never changes, so once it succeeds the stream short-circuits forever. Let me settle which query actually ran by reading the archived raw blob.

**Claude** (2026-09-07T04:02): _[tool: Bash]_

**Claude** (2026-09-07T04:02): Definitive: the archived blob holds 45 elements, 42 named, **all `place=*` nodes** — the *old* settlements-only query. So the widened query committed on 2026-08-26 has never run in production. Let me retry my proposed query's syntax.

**Claude** (2026-09-07T04:02): _[tool: Bash]_

**Claude** (2026-09-07T04:03): 81 named features, all with usable coordinates. Let me gather the last facts I need for the schema proposal.

**Claude** (2026-09-07T04:03): _[tool: Bash]_

**Claude** (2026-09-07T04:03): Key STEP 5 finding: `relevance_terms` is populated **only** for tiers A/B/C (2,200 docs) and empty for D (4,990) and NULL (1,839) — so the existing tiering is term-driven and the NULL rows were never scored. Let me find that existing logic so I reuse its vocabulary instead of inventing one.

**Claude** (2026-09-07T04:03): _[tool: Bash]_

**Claude** (2026-09-07T04:03): _[tool: Bash]_

**Claude** (2026-09-07T04:03): No tier-assignment code exists anywhere in the repo — so the A/B/C/D labels came from a process outside it. That's actually useful: I can derive the rule vocabulary empirically from the 2,200 already-labelled docs rather than inventing one. Let me look at `relevance_terms`.

**Claude** (2026-09-07T04:03): _[tool: Bash]_

**Claude** (2026-09-07T04:04): The tiering rule is fully recoverable — the tiers are **precedence bands over a 38-term vocabulary** (`;`-separated). Let me validate that hypothesis against all 2,200 labelled docs before relying on it. Pre-registered expectation: if the rule is "A if any A-term, else B, else C", it should reproduce 100% of labels.

**Claude** (2026-09-07T04:04): _[tool: Bash]_

**Claude** (2026-09-07T04:04): **Rule recovered exactly: 2,200/2,200, 100% accuracy, zero ambiguous terms.** A 38-term vocabulary partitioned into three mutually exclusive bands, tier = highest band matched (A>B>C). Now let me dry-run it against the 1,839 unclassified docs.

**Claude** (2026-09-07T04:04): _[tool: Bash]_

**Claude** (2026-09-07T04:06): 100% control agreement (2,199/2,200) confirms title+abstract is the right matching surface. Let me gather final facts for the migration risk section — notably whether FK enforcement is even on.

**Claude** (2026-09-07T04:06): _[tool: Bash]_

**Claude** (2026-09-07T04:06): All grounding is in place. FK enforcement is currently **off** (`PRAGMA foreign_keys=0`) but there are **zero** existing violations — so it can be safely enabled. Now let me write the main deliverable.

**Claude** (2026-09-07T04:06): _[tool: Bash]_

**Claude** (2026-09-07T04:10): _[tool: Bash]_

**Claude** (2026-09-07T04:10): STEP 1 is written (872 lines). Now STEP 4 — the root-cause fix. Let me check the existing test setup first.

**Claude** (2026-09-07T04:10): _[tool: Bash]_

**Claude** (2026-09-07T04:10): _[tool: Bash]_

**Claude** (2026-09-07T04:10): Good — the tests here assert *rules*, not examples. Note `taxon.js` imports JSON without an import attribute, so it can't be imported from a plain Node test. I'll extract the pure logic into its own lib module. First let me design the normalizer against real names rather than imagined ones.

**Claude** (2026-09-07T04:10): _[tool: Bash]_

**Claude** (2026-09-07T04:11): **A second, independent leak.** `taxon` 2485 = `Gyps bengalensis (Gmelin, 1788)` has `sensitive=0`, while `taxon` 30 = `Gyps bengalensis` has `sensitive=1` — same bird, authorship suffix bypasses the matcher. Let me check its occurrences.

**Claude** (2026-09-07T04:11): _[tool: Bash]_

**Claude** (2026-09-07T04:11): More leaks, and these are **not** explained by `--raw-coordinates`: `Melursus ursinus ursinus` (sloth bear, trinomial, 2 rows) and `Gyps bengalensis (Gmelin, 1788)` (3 rows) both have `sensitive=0`, so they were invisible to my earlier `sensitive=1` query. Let me confirm the full list.

**Claude** (2026-09-07T04:11): _[tool: Bash]_

**Claude** (2026-09-07T04:12): Two distinct defects now, and only one is a code bug:

1. **Name-shape matching** (trinomial + authorship) — fixable in code, this is STEP 4.
2. **Registry incompleteness** — 9 of the 20 species atlas.db flags sensitive are absent from `sensitive-species.json` entirely (`Crocodylus palustris`, `Melursus ursinus`, `Python molurus`, `Ophiophagus hannah`…). No code fix reaches those.

Let me build the fix as a pure, testable module.

**Claude** (2026-09-07T04:13): _[tool: Bash]_

**Claude** (2026-09-07T04:13): Now wire it into `taxon.js`.

**Claude** (2026-09-07T04:13): _[tool: Bash]_

**Claude** (2026-09-07T04:13): `parseScientificName` is now unused in that import — let me drop it, then write the test.

**Claude** (2026-09-07T04:14): _[tool: Bash]_

**Claude** (2026-09-07T04:14): Both failures are real bugs in my implementation, correctly caught. Bug 1: I lowercase the token *before* testing whether it's lowercase, which destroys the very signal that distinguishes an epithet from an author surname (`Letcher` → `letcher` → accepted). Bug 2: an all-lowercase input can't be parsed at all. Let me fix both properly.

**Claude** (2026-09-07T04:15): _[tool: Bash]_

**Claude** (2026-09-07T04:15): All 9 pass — the first test alone is 8 registry entries × 12 name shapes. Let me confirm I broke nothing else, then run the end-to-end test against staging as STEP 4 requires.

**Claude** (2026-09-07T04:15): _[tool: Bash]_

**Claude** (2026-09-07T04:15): Staging has all three triggers. But the full suite reports a failure — let me find out whether that's pre-existing or mine.

**Claude** (2026-09-07T04:15): _[tool: Bash]_

**Claude** (2026-09-07T04:15): All 34 tests pass individually across 4 files. The `npm test` failure looks like a runner-invocation issue, not a test failure. Let me confirm it's pre-existing rather than caused by my file.

**Claude** (2026-09-07T04:15): _[tool: Bash]_

**Claude** (2026-09-07T04:15): Confirmed **pre-existing**: `npm test` fails identically with my file removed — Node 24 no longer accepts a bare directory there. The glob form passes 34/34. That means the suite has been unrunnable via `npm test`, so I'll fix the one-line script since it blocks verifying my own work.

**Claude** (2026-09-07T04:15): _[tool: Bash]_

**Claude** (2026-09-07T04:16): Now the staging end-to-end test. Let me get the new matcher's verdicts and the staging table shape, then pre-register expectations.

**Claude** (2026-09-07T04:16): _[tool: Bash]_

**Claude** (2026-09-07T04:16): D1's `occurrence` already has `place_id` — so my §2b proposal aligns atlas.db with D1's existing model rather than inventing one. I'll note that. Now the staging test, with pre-registered expectations.

**Claude** (2026-09-07T04:17): _[tool: Bash]_

**Claude** (2026-09-07T04:17): `source.r2_key` is NOT NULL. Let me check the full constraint set rather than guess again.

**Claude** (2026-09-07T04:17): _[tool: Bash]_

**Claude** (2026-09-07T04:17): _[tool: Bash]_

**Claude** (2026-09-07T04:17): _[tool: Bash]_

**Claude** (2026-09-07T04:18): The script executed and cleaned up, but `--json` collapsed the SELECT output. Let me rerun to capture the proof table.

**Claude** (2026-09-07T04:18): _[tool: Bash]_

**Claude** (2026-09-07T04:18): D1's `--file` mode returns only a summary. Let me split into setup / proof / cleanup so the proof is visible.

**Claude** (2026-09-07T04:18): _[tool: Bash]_

**Claude** (2026-09-07T04:18): Exactly as pre-registered. And the test makes the registry gap visible: `Melursus ursinus ursinus` and `Gyps bengalensis (Gmelin, 1788)` pass *through* uncoarsened, because those species aren't in the registry at all. Now cleanup.

**Claude** (2026-09-07T04:18): _[tool: Bash]_

**Claude** (2026-09-07T04:19): _[tool: Bash]_

**Claude** (2026-09-07T04:19): Staging is clean — STEP 4 verified. And staging has **two** overpass sources: one sharing prod's exact hash (the old query) and one different. That's the before/after STEP 2 asks for. Let me quantify.

**Claude** (2026-09-07T04:19): _[tool: Bash]_

**Claude** (2026-09-07T04:19): Clean before/after already on staging: **42 places (old query) → +17 (widened query) = 59**, and the widened query has never reached prod (still 46). Let me read `findOrCreatePlace` to see whether I can faithfully test my further-widened query against staging without deploying.

**Claude** (2026-09-07T04:19): _[tool: Bash]_

**Claude** (2026-09-07T04:19): `findOrCreatePlace` dedups on `osm_id` then `slug` — **not** `wikidata_id`. That's the upstream half of STEP 0 item 1, still unfixed (the commit `e9490d8` explicitly flagged it). Let me fix both the query and the dedup key.

**Claude** (2026-09-07T04:20): _[tool: Bash]_

**Claude** (2026-09-07T04:20): Now the upstream `wikidata_id` dedup fix — the other half of STEP 0 item 1.

**Claude** (2026-09-07T04:20): _[tool: Bash]_

**Claude** (2026-09-07T04:20): Now let me test the widened query against staging — using the *actual* shipped function to build the query, so the test can't drift from the code.

**Claude** (2026-09-07T04:20): _[tool: Bash]_

**Claude** (2026-09-07T04:21): 81 named elements from the shipped function. Now let me apply them to staging with a faithful mirror of `findOrCreatePlace`, dry-run first.

**Claude** (2026-09-07T04:21): _[tool: Bash]_

**Claude** (2026-09-07T04:21): 6 dedupe hits landed on `wikidata_id` — under the old precedence those would have become duplicate rows. Let me verify that with a paired control before applying.

**Claude** (2026-09-07T04:21): _[tool: Bash]_

**Claude** (2026-09-07T04:22): Correction: my claim about the 6 wikidata hits was wrong. The paired control shows **both precedences insert exactly 19** — the wikidata key only changed which key got *credit*, not the outcome. Those 6 would also have matched on `osm_id`/`slug`.

The real bug is cross-stream, not within one Overpass run. Let me reproduce it faithfully.

**Claude** (2026-09-07T04:22): _[tool: Bash]_

**Claude** (2026-09-07T04:22): The fix reproduces and prevents the exact `e9490d8` duplicate. Let me make that a permanent regression test.

**Claude** (2026-09-07T04:23): _[tool: Bash]_

**Claude** (2026-09-07T04:23): 39/39 pass. Now let me apply the widened query's results to staging for the before/after count.

**Claude** (2026-09-07T04:23): _[tool: Bash]_

**Claude** (2026-09-07T04:23): STEP 2 is measured end-to-end on staging: **42 → 59 → 78** places (prod untouched at 46).

Now STEP 5. One constraint first: staging D1 holds **0 documents**, so the 1,839 NULL-tier rows exist only in atlas.db and prod D1 — there's nothing to test against in staging. Let me build the tierer so it validates itself before touching anything.

**Claude** (2026-09-07T04:24): _[tool: Bash]_

**Claude** (2026-09-07T04:24): The instrument gates itself (2200/2200 from terms, 2199/2200 from text). Let me identify that single mismatch rather than leave it unexplained.

**Claude** (2026-09-07T04:24): _[tool: Bash]_

**Claude** (2026-09-07T04:25): That "mismatch" is the rule being *more* correct than the stored label — doc 604's abstract names Sathyamangalam and Mudumalai, but it was filed C on `tamil nadu` alone. That implies the original process matched **titles only**. Let me test that.

**Claude** (2026-09-07T04:25): _[tool: Bash]_

**Claude** (2026-09-07T04:25): Correction: title-only reproduces just **1415/2200** (674 find no term), so the original surface was definitively title+abstract — my inference was wrong, and doc 604 is a genuine one-off anomaly. Worth noting: only 718/1839 NULL-tier docs have an abstract (39% vs 83% for tier A), which explains much of the 77% unclassified rate.

Now STEP 3. The 9-item list doesn't exist, but I found one live freeze — let me audit that whole class.

**Claude** (2026-09-07T04:25): _[tool: Bash]_

**Claude** (2026-09-07T04:25): The pattern is systemic, not just overpass: `wikidata` (0 rows in 30 runs), `gbif` (0 in 38), `historical-text`, `erode-nic`, `wii`, `ntca` — all reporting success while writing nothing. But the repo's own test asserts these should be `skipped_duplicate`, and `skipped` is 0 everywhere. Let me check the deriver.

**Claude** (2026-09-07T04:25): _[tool: Bash]_

**Claude** (2026-09-07T04:26): _[tool: Bash]_

**Claude** (2026-09-07T04:26): _[tool: Bash]_

**Claude** (2026-09-07T04:26): Two more findings: `skipped_duplicate` has **never** been recorded in prod (so the deriver is undeployed), and **64 runs are stuck in `running`**. Let me check whether the prod schema even permits the new status — that's a deployment hazard.

**Claude** (2026-09-07T04:26): _[tool: Bash]_

**Claude** (2026-09-07T04:26): Corrections worth noting: the prod schema **does** allow `skipped_duplicate` (migration 0004 applied 2026-08-25), so there's no deploy hazard there — but no run has ever used it, confirming the deployed code lacks the deriver. And there's migration drift: prod has `0003_coverage_snapshot.sql` which isn't in the repo, while the repo's `0005_geometry_scope_annulus.sql` isn't applied.

Let me collect the final change inventory and write the morning report.

**Claude** (2026-09-07T04:26): _[tool: Bash]_

**Claude** (2026-09-07T04:29): _[tool: Bash]_

**Claude** (2026-09-07T04:29): Both documents written. Final verification pass.

**Claude** (2026-09-07T04:29): _[tool: Bash]_

**Claude** (2026-09-07T04:29): _[tool: Bash]_

**Claude** (2026-09-07T04:29): One thing worth checking: `main` HEAD mentions a "job_run status display fix" that `dev` doesn't have. That could change my STEP 3 finding (c).

**Claude** (2026-09-07T04:29): _[tool: Bash]_

**Claude** (2026-09-07T04:29): `dev` and `main` have diverged substantially both ways, and both have a `job-run.js` deriver — but in *different* forms. Let me check main's version, since prod deploys from main.

**Claude** (2026-09-07T04:29): _[tool: Bash]_

**Claude** (2026-09-07T04:30): Important correction: `main` (what production deploys from) has `classifyOutcome`, not `deriveJobRunStatus`, and it **deliberately** leaves a skipped-duplicate run as `success` — "a run that skipped duplicates did do work and is left alone." `dev` has the opposite policy. My finding (c) was wrong. Let me fix the report.

**Claude** (2026-09-07T04:30): _[tool: Bash]_

**Claude** (2026-09-07T04:30): Let me check whether this pattern is already captured in memory before deciding to add anything.

**Claude** (2026-09-07T04:30): _[tool: Bash]_

**Claude** (2026-09-07T04:31): The three existing memories cover method well. One genuinely new and durable pattern emerged tonight that isn't in them or the repo: the *brief's own premises* were stale, not just my claims.

**Claude** (2026-09-07T04:31): _[tool: Bash]_

**Claude** (2026-09-07T04:31): Done. Both documents are written and everything is verified.

## The blocker first

**All four briefing documents you told me to read don't exist** — not in the tree, any of 7 local branches, `origin/*`, or `git log --all --diff-filter=A`. I recovered the coordinate policy from the code that implements it (`coarsen.js`, the triggers, `sensitive-species.json`, the sync's `SensitivityOracle`) and designed against that. The **dedup-freeze 9-item list is unrecoverable, so STEP 3 is blocked** — I didn't invent nine items.

## Per step

**STEP 0** — All three earlier asks are done, verified against live artefacts not just commit messages. But `e9490d8` fixed only the *downstream* half of the place-matching key; `findOrCreatePlace` still matched `osm_id → slug`. I fixed that upstream half.

**STEP 1** (main deliverable) — `schema-proposal-2026-09-07.md`, 872 lines, all 8 items with DDL/FKs/indexes and a 13-step migration plan. **No migration written or run.** Three of the brief's premises were wrong and changed the design: `occurrence` already exists with exactly 78,467 rows; **`claim` isn't empty — 83 rows, already containing your tiger example**; "93 places" was pre-merge. Biggest finding: `occurrence.lat/lon` means different things depending on whether `--raw-coordinates` ran, and nothing records which. Also **nothing in atlas.db references `place(id)`** — so places-as-hubs has no edges to optimise yet.

**STEP 2** — Measured on staging: **42 → 59 → 78** places. But **500–2,000 is unreachable**: the hard ceiling is 258 named OSM elements, mostly roads, hospitals and a greengrocer. I proved production still runs the *old* query by fetching its archived R2 blob (42 named `place` nodes). The route to hundreds is LGD/Census — blocked on `DATA_GOV_IN_API_KEY`.

**STEP 4** — Root cause fixed in a new pure module and **verified end-to-end on staging** with pre-registered expectations and paired controls; staging returned to baseline. 39/39 tests pass. Found a **second leak no code fix reaches**: 9 of the 20 species atlas.db flags sensitive are absent from `sensitive-species.json` — including three Critically Endangered vultures and the king cobra. I didn't edit the registry; that changes what production redacts.

**STEP 5** — Recovered the tiering rule rather than inventing one: a 38-term vocabulary in three precedence bands, **validated 2200/2200**. Result: 34 A, 11 B, 370 C, **1,424 unclassified**. **Dry-run only** — writing it would promote 45 unreviewed docs into the citable bibliography, which is your call, not a script's.

## Three things I got wrong and corrected

The 13 uncoarsened sensitive occurrences are the documented `--raw-coordinates` dev flag, not a trinomial leak. The 6 wikidata dedupe hits would *not* have been duplicates — the paired control showed both precedences insert exactly 19. And `main`'s `classifyOutcome` **deliberately** reports skipped-duplicate runs as success, so my "undeployed fix" claim was wrong — the real finding is that `dev` and `main` now hold **contradictory** job-run policies, each catching a failure the other misses.

**Changes are uncommitted on `dev`** (my standing instruction is to commit only when asked). `data/atlas.db` is untouched, production D1 was read-only throughout, cron still `crons = []`. Say the word and I'll commit. The `dev`/`main` divergence is 10 commits one way and 20+ the other, and that merge won't be mechanical.