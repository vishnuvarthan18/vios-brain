# Overnight run — build the detail pages (plan for 7→8 Sep 2026)

Paste each step to the coding agent in order. Same standing rules as every prior run.

**Standing rules — no exceptions:**
- Everything happens on `dev` and the staging/test database. Nothing merges to `main`. Nothing deploys anywhere. This is for Vishnu to review in the morning, not to go live overnight.
- Production coordinate-coarsening rule stays in force for sensitive species.
- If a step is blocked by a decision only Vishnu can make, stop that step, write down what's blocked, and move to the next independent one.

---

## STEP 0 — Confirm the migrated schema is what this build targets

**Why:** last night's migration (commit `dcc5cc5` on dev) added the tables this work depends on: category, taxon_category, habitat, relationship, photo, place_link, completeness, and the backfilled taxon/occurrence/claim columns. Confirm it's still there and nothing regressed before building on top of it.

**Instructions to paste:**
```
Confirm data/atlas.db on dev still has the schema from commit dcc5cc5 (22
tables, the category/taxon_category/habitat/relationship/photo/place_link/
completeness tables, backfilled taxon.canonical_name and
occurrence.public_lat/public_lon). Run PRAGMA integrity_check and confirm
"ok". Report table count and row counts for taxon, occurrence, category.
Do not proceed to Step 1 if this doesn't check out — report what's wrong
instead.
```

---

## STEP 1 — One species detail page template, built against the real data

**Why:** this is the actual atlas standard from atlas-scope-locked-2026-09-06.md — a full record per species, not a flat list. Build ONE template that works for every category (birds, mammals, insects, etc.), tested against a handful of real, well-documented species first.

**Instructions to paste:**
```
Build a species detail page template (e.g. /life/[category]/[slug]) that
renders, per species:
1. Basic facts: common name (English + Tamil if available), scientific
   name, family, IUCN status, WPA schedule, description.
2. Where it's been recorded: pull from the occurrence table (public_lat/
   public_lon, respecting the coordinate-coarsening rule for sensitive
   species), grouped or listed, not a raw dump of all rows if a species has
   thousands.
3. Population/trend over time: pull from the claim table where it holds
   data points for this species, each with its source, shown as a simple
   timeline/list (not a chart library yet unless one's trivial to add).
4. Relationships: pull from the new relationship table for this taxon
   (predator/prey/habitat links). If empty for most species right now,
   show an honest "no relationships documented yet" rather than hiding
   the section.
5. Photos: pull from the photo table, show attribution/licence per image.
6. Threats: use whatever threat/description field already exists; if
   none, say so honestly rather than leaving a blank space.
7. Local/tribal names: show if present in existing data, otherwise state
   plainly "no local name documented yet" (per the honesty design rule).
8. Sources: list sources for the facts shown.
9. Completeness label: pull from the new completeness table if populated;
   otherwise compute a simple version live (e.g. "last verified [date]",
   "no photo yet") and note that a proper completeness pass hasn't run yet.

Test this template against 5 real species you pick that have good data
(one bird, one mammal, one with population history in claim, one with
photos, one with almost nothing — to prove the honesty-about-gaps rule
actually works, not just the happy path). Screenshot or describe each of
the 5 rendered pages in your report.
```

---

## STEP 2 — Wire the category browsers to the new schema

**Why:** the existing /life species browser is a flat list from before the schema migration. Point it at the new category table so species split by type (birds, mammals, insects, snakes/reptiles, trees, plants, fungi, fish, amphibians) as the locked scope requires.

**Instructions to paste:**
```
Update the /life species browser to use the new category/taxon_category
tables instead of one flat list. Each category should be its own
browsable section. Link each species row to its new detail page from
Step 1. Report the count of species shown per category, and flag the 942
"uncategorised" species from last night's migration — these need their own
visible section too (not hidden), per the atlas's honesty rule.
```

---

## STEP 3 — One place detail page template

**Why:** places-as-hubs is the core structural idea locked in atlas-scope-locked-2026-09-06.md — every place should show every species/document/history record linked to it.

**Instructions to paste:**
```
Build a place detail page template (e.g. /land/[slug]) that renders, per
place:
1. Basic facts: name (English + Tamil + historical spelling variants if
   present), type, coordinates, elevation if known.
2. Everything linked to it via the new place_link table: species sighted
   here, documents mentioning it, historical passages, anything else
   linked. If place_link is mostly empty right now (expected — occurrence.
   place_id was left NULL in last night's migration pending a policy
   decision), say so honestly on the page rather than showing an empty
   section with no explanation.
3. Administrative details, sources, completeness label — same pattern as
   species pages in Step 1.

Test against 3 real places with the most existing data. Report what's
actually populated vs. what's honestly empty because the underlying links
don't exist yet.
```

---

## STEP 4 — Document tiering and bibliography display

**Why:** last week's tiering run promoted 45 documents into the citable bibliography and left 1,424 as unclassified (in relevance_tier_auto). Nothing currently displays this on the site.

**Instructions to paste:**
```
Add a bibliography/sources page showing the tiered documents (A/B/C) as
citable sources, and a separate "unreviewed documents" section showing the
1,424 unclassified ones — visible, not hidden, per the honesty rule. Link
individual documents from species/place pages where they're already
referenced as sources.
```

---

## What NOT to attempt tonight

- No merge to `main`, no deploy anywhere, no cron changes.
- Don't attempt to resolve occurrence.place_id for all 78,467 rows — that's blocked on Vishnu's sensitive-occurrence granularity policy decision and the missing boundary geometry. Build the templates to handle "not yet linked" gracefully instead.
- Don't build the citizen-contribution submission UI yet — the contributions table itself wasn't even created in last night's migration (correctly deferred). Not in scope tonight.
- Don't touch the crocodile/king cobra category bug unless it blocks Step 2's category display — if it does, fix just that narrow bug and note it, don't scope-creep into a broader rewrite.
- Don't touch the dev/main divergence or outreach emails.

## In the morning, report back

For each step: what real species/places look like on the new pages (with actual examples, not just "done"), what's honestly still empty and why, and what needs Vishnu's decision next. Write this to `sathyamangalam/detail-pages-run-results-[date].md` in the project.
