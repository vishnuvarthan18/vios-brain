# Overnight coding-agent run — Sathyamangalam Atlas (plan for 6→7 Sep 2026)

Paste each step to the coding agent in order. Do not skip ahead. Each step says how to confirm it worked before moving on.

**Standing rules that apply to this entire run, no exceptions:**
- Everything happens on the `dev` branch and/or the staging database. Nothing merges to `main`. Nothing deploys anywhere (dashboard, harvest-engine, public site). This run is meant to produce work ready for Vishnu to review in the morning, not to go live overnight.
- Production coordinate-coarsening rule for sensitive species stays in force — dev/staging may hold exact coordinates, production may not (see `sensitive-species-coordinate-policy.md`).
- No new community fieldwork or cultural-archive work.
- If any step's confirmation check fails, or a step depends on a decision Vishnu hasn't made, STOP that step, write down exactly what's blocked and why, and move to the next independent step instead of guessing.

---

## STEP 0 — Check the status of the last unfinished prompt

**Why:** before this session was frozen, a prompt was sent asking for: (1) fix the place-matching duplicate bug, (2) build a basic species browser on `/life`, (3) separate out tier-less documents into their own section. Status was unknown at freeze time.

**Instructions to paste:**
```
Check the current state of the dev branch for three things I asked for earlier:
1. A fix to the place-matching key so it includes wikidata_id (to prevent the
   Sathyamangalam/Satyamangalam duplicate).
2. A basic species browser page at /life.
3. Tier-less documents (relevance_tier IS NULL) separated into their own section
   instead of being invisible.
Report which of these are (a) fully done and merged into dev, (b) partially done,
or (c) not started at all. Show me the relevant commits or file diffs as proof,
don't just say "done." Do this before starting anything else tonight.
```

**How to confirm:** the agent shows actual commits/diffs, not just a claim. If something is already done, skip the matching step below instead of redoing it.

---

## STEP 1 — Database schema design pass (the main deliverable of tonight)

**Why:** the atlas's scope is now locked as one connected database (Land/Life/History/People, everything cross-linked, full per-record detail — see `atlas-scope-locked-2026-09-06.md`). The current schema can't hold most of this. This has to be designed properly before any more collection or display code is written — this is explicitly the blocking next step from the last session.

**Instructions to paste:**
```
Read sathyamangalam/atlas-scope-locked-2026-09-06.md and
sathyamangalam/session-close-2026-09-06-full-freeze.md from the project docs
before starting this.

Design (as a written schema proposal first — do NOT run any migration yet)
additions/changes to data/atlas.db's schema to support:

1. Species split by category: birds, mammals, insects, snakes/reptiles, trees,
   plants, fungi, fish, amphibians — not one flat table. Decide whether this is
   a `category` column on the existing taxon table with indexes, or genuinely
   separate tables, and write down the tradeoff you're choosing and why.

2. An `occurrences` table (or confirm/fix the existing one) that makes the
   78,467 existing occurrence rows individually browsable: species, place,
   date, source, coordinates (respecting the coordinate-coarsening rule for
   sensitive species in production).

3. A `population_history` table for tracking a number over time per species
   with a source per data point (e.g. tigers 8-10 in 2009 -> 112 in 2024).
   Look at the existing empty `claim` table first — this should extend that
   pattern, not duplicate it. Write down whether you're extending `claim` or
   adding a new table and why.

4. A `relationships` table for species-to-species (eats/eaten-by) and
   species-to-habitat links.

5. A `photos` table/pipeline linking iNaturalist CC-licensed images (already
   harvested) to species records.

6. Places-as-hubs: every place needs to show every species/document/history/
   community record linked to it. Design the join structure that makes this
   possible without n+1 query explosions on a place page.

7. A `completeness` field or table (last verified date, missing-field flags)
   attachable to every record type — species, place, occurrence.

8. A `contributions` table for future citizen input (sightings, photos,
   corrections) with a moderation_state field (pending/approved/rejected) and
   a reference to who submitted it and what record it's proposing to change.
   This does NOT need a UI or submission flow built tonight — just the schema
   so it's ready.

Write the full proposal to sathyamangalam/schema-proposal-[today's date].md
in the project docs, including: table definitions, foreign keys, indexes,
and a short migration plan (what changes to existing tables, if any, and in
what order to avoid breaking the current site). Do NOT apply any migration
yet — this step is proposal-only. Stop here and wait for review.
```

**How to confirm:** the doc exists, has real table definitions (not vague descriptions), and explicitly states any breaking changes to the current schema.

---

## STEP 2 — Gazetteer fix (if Step 0 shows it wasn't already done)

**Why:** highest-value fix flagged repeatedly — only 93 of 800–2,000 target places found, likely an Overpass query bug.

**Instructions to paste:**
```
Confirm the current place count from the gazetteer stream (should be running
against harvest-engine-staging, not production). If a wider Overpass query
fix was already applied (check Step 0's findings), report the current count.
If not yet fixed, propose the fix as a diff (do not deploy or run against
production) and test it against the staging database only. Report before
and after place counts.
```

**How to confirm:** a real before/after count from the staging database, not an estimate.

---

## STEP 3 — Dedup-freeze bug (harvest-engine cron is paused because of this)

**Why:** the harvest cron has been paused since 5 Sep because the dedup logic checks "ever fetched" instead of "fetched recently" — meaning re-runs skip everything. See `harvest-engine-dedup-freeze-and-pause-2026-09-05.md` for the 9-item fix list, none shipped yet.

**Instructions to paste:**
```
Read sathyamangalam/harvest-engine-dedup-freeze-and-pause-2026-09-05.md in
full. Work through its 9-item fix list on the dev branch against the staging
database only. Do not re-enable the production cron. For each of the 9 items,
report status (fixed/skipped and why) with a test result, not just a claim.
```

**How to confirm:** 9 individual pass/fail results with evidence, run against staging.

---

## STEP 4 — Sensitivity matcher root-cause fix

**Why:** the binomial-vs-trinomial species name matching bug caused the sensitive-species coordinate leak that's currently only patched at sync time, not fixed at the source. See `sensitive-species-coordinate-policy.md`.

**Instructions to paste:**
```
Read sathyamangalam/sensitive-species-coordinate-policy.md. Find the sensitivity
matcher in the harvest-engine source that currently only matches binomial
species names (genus + species) and fails on trinomial names (genus + species
+ subspecies). Fix it to correctly match both. Test against a known trinomial
sensitive-species record from the staging database and confirm it's now
correctly flagged as sensitive. Do not touch the sync-time coordinate-coarsening
patch — that stays as defense in depth even after this root-cause fix.
```

**How to confirm:** a specific trinomial test case that was previously missed, now caught.

---

## STEP 5 — relevance_tier NULL documents

**Why:** 1,839 harvested documents are invisible on the site because `relevance_tier` is NULL and the site only shows tier A/B. No tiering mechanism exists yet.

**Instructions to paste:**
```
On the dev branch: design and implement a basic tiering pass for the 1,839
documents with relevance_tier IS NULL. This does not need to be a perfect
classifier — propose a simple rule-based first pass (e.g. based on source
type, keyword match against species/place names already in the database, or
similar) that assigns a reasonable tier, and flag anything it can't confidently
tier as a new tier "unclassified" that gets its own visible section on the
site (per the atlas's "no invisible data" honesty principle) rather than
staying hidden. Test against the staging database. Do not deploy this
display change anywhere.
```

**How to confirm:** a report of how many of the 1,839 got tiered vs how many landed in "unclassified," with a few sample records shown.

---

## What NOT to attempt tonight

- No merge of `dev` into `main`, under any circumstances, without Vishnu's explicit go-ahead in that exact conversation.
- No deploy of the dashboard, harvest-engine, or public site anywhere.
- No re-enabling of the production harvest cron.
- The `dev`/`main` branch divergence (~3.5k lines) — leave alone, it's its own reconciliation job, not an overnight task.
- No new fieldwork, community outreach, or cultural-archive work.
- Do not touch the outreach emails (Tier 2 draft) — that's blocked on this schema/data work being done first, not something to advance in parallel.

## In the morning, report back

For each step: what was done, what was skipped and why, and what needs Vishnu's decision before it can go further. Put this as a new `sathyamangalam/overnight-run-results-[date].md` doc in the project.
