# Overnight run — History and People sections (plan for 8→9 Sep 2026)

Paste each step to the coding agent in order. Same standing rules as every prior run.

**Standing rules — no exceptions:**
- Everything happens on `dev` and the staging/test database only. Nothing merges to `main`. Nothing deploys anywhere. Vishnu reviews in the morning.
- If a step is blocked by a decision only Vishnu can make, stop that step, write down what's blocked, and move to the next independent one.

**Why this is next:** the locked scope is 4 connected sections — Land, Life, History, People. Land and Life now have real detail pages (built 7→8 Sep). History and People don't exist yet as browsable sections at all. This is the natural next piece, and it's independent of the 5 items still waiting on Vishnu's decision from last night's report.

---

## STEP 0 — Confirm state and inventory what History/People content already exists

**Why:** before building pages, find out what's actually sitting in the database that could populate them — documents, historical passages, legal instruments, news events, any community/people-related records — so the build targets real data, not an empty shell.

**Instructions to paste:**
```
Confirm data/atlas.db on dev is still healthy (PRAGMA integrity_check).
Then inventory what exists that could feed History and People sections:
- document, historical_passage, legal_instrument, news_event tables (or
  whatever holds this content) — row counts, and a sample of 5 real rows
  from each so we know what fields are actually populated.
- Anything resembling "people" or "community" content — check the schema
  and existing tables for anything related to communities, organizations,
  or named contributors (Keystone Foundation material, tribal community
  records, etc. mentioned in earlier project docs). Report what exists vs.
  what would need to be modeled fresh.
Report back before building anything — if People has essentially zero
existing data, say so plainly; that changes what Step 2 below should do.
```

---

## STEP 1 — History detail pages

**Why:** same "full record, not a count" standard as species/place pages. History content already exists (documents, historical passages) — this is mostly wiring, not new data modeling.

**Instructions to paste:**
```
Build a history detail page template (e.g. /history/[slug]) using the same
generator pattern as the species/place pages (site/life/, site/land/).
Render, per historical record:
1. What it is: title/summary, date or date range, type (document, passage,
   legal instrument, news event, etc.)
2. Full source text or excerpt, with proper attribution.
3. Links to places and species this record mentions or connects to
   (use place_link and whatever taxon references exist).
4. Sources, and a completeness label following the same pattern as species/
   place pages.
Test against 5 real records with good data. Build an index page at
/history listing them, grouped sensibly (by era, type, or whatever the
data supports) rather than one flat list.
```

---

## STEP 2 — People/Community section (scope depends on Step 0's findings)

**Why:** the locked scope's 4th section. Per atlas-scope-locked-2026-09-06.md, this section is explicitly NOT about new fieldwork or interviews — it only uses cultural/community facts already documented in existing sources (management plan, Keystone Foundation comparison material already gathered, etc.).

**Instructions to paste:**
```
Based on Step 0's inventory: if there's real existing data (named
communities, villages with documented agreements, organizations like
Keystone Foundation already referenced in project docs), build a People
detail page template (e.g. /people/[slug]) following the same pattern as
History. If Step 0 found this is essentially empty, do NOT invent content
or fabricate placeholder pages — instead build a single honest /people
index page stating plainly what this section will eventually cover and
that it's not populated yet, consistent with the atlas's honesty-about-
gaps design rule. Report which path you took and why.
```

---

## STEP 3 — Cross-link what's already linkable

**Why:** the core differentiator is everything connecting to everything. Species and place pages exist now; history pages will exist after Step 1. Wire the connections that existing data actually supports, without inventing links that don't exist.

**Instructions to paste:**
```
Where a species page, place page, or history page has a real, existing
reference to another one of these (a document that names a specific
place, a claim that names a specific species, a place_link row), add a
visible cross-link between them. Do not infer or guess links that aren't
explicitly in the data. Report how many real cross-links you were able to
add, broken down by type (species<->place, species<->history,
place<->history).
```

---

## STEP 4 — Site navigation update

**Why:** History and People need to be reachable from the site's main navigation, matching the Land/Life/History/People structure that's already the site's stated design.

**Instructions to paste:**
```
Update the site's main navigation/index to properly link to the new
/history and /people sections alongside the existing /land and /life.
Confirm all four sections are reachable from the homepage. Do not touch
anything else on the homepage.
```

---

## What NOT to attempt tonight

- No merge to `main`, no deploy anywhere, no cron changes.
- Don't fabricate People content if the data doesn't exist — report honestly instead, per Step 2's instructions.
- Don't touch the 5 items still flagged from last night's report (crocodile/king cobra fix, tier-C citability tension, tree life-form classification, missing family field, stale report path) unless one blocks a step above — if so, fix only the narrow blocking piece and note it.
- Don't touch dev/main divergence or outreach emails.

## In the morning, report back

For each step: what real History/People content looks like on the new pages (actual examples), what's honestly still missing and why, and what needs Vishnu's decision next. Write this to `sathyamangalam/history-people-run-results-[date].md` in the project.
