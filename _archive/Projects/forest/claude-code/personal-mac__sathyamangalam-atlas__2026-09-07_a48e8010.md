**Vishnu** (2026-09-07T16:00): Work through these steps IN ORDER overnight. Same standing rules as every
prior run: everything stays on the dev branch and staging/test database
only. Do not merge dev into main, do not deploy anywhere, do not touch the
production cron or production database. If a step is blocked by a decision
only Vishnu can make, stop that step, write down exactly what's blocked,
and move to the next independent one instead of guessing.

STEP 0 — Confirm the migrated schema
Confirm data/atlas.db on dev still has the schema from commit dcc5cc5 (22
tables, the category/taxon_category/habitat/relationship/photo/place_link/
completeness tables, backfilled taxon.canonical_name and
occurrence.public_lat/public_lon). Run PRAGMA integrity_check and confirm
"ok". Report table count and row counts for taxon, occurrence, category.
Do not proceed if this doesn't check out — report what's wrong instead.

STEP 1 — One species detail page template
Build a species detail page template (e.g. /life/[category]/[slug]) that
renders, per species:
1. Basic facts: common name (English + Tamil if available), scientific
   name, family, IUCN status, WPA schedule, description.
2. Where it's been recorded: pull from the occurrence table (public_lat/
   public_lon, respecting the coordinate-coarsening rule for sensitive
   species), grouped or listed sensibly, not a raw dump of thousands of rows.
3. Population/trend over time: pull from the claim table where it holds
   data points for this species, each with its source, shown as a simple
   timeline/list.
4. Relationships: pull from the relationship table for this taxon. If
   empty for most species, show an honest "no relationships documented
   yet" rather than hiding the section.
5. Photos: pull from the photo table, show attribution/licence per image.
6. Threats: use whatever threat/description field already exists; if
   none, say so honestly.
7. Local/tribal names: show if present, otherwise state plainly "no local
   name documented yet."
8. Sources: list sources for the facts shown.
9. Completeness label: pull from the completeness table if populated,
   otherwise compute a simple version live (e.g. "last verified [date]",
   "no photo yet") and note a proper completeness pass hasn't run yet.

Test against 5 real species you pick with varying data quality (one bird,
one mammal, one with population history in claim, one with photos, one
with almost nothing — to prove the honesty-about-gaps rule works, not just
the happy path). Describe each of the 5 rendered pages in your report.

STEP 2 — Wire the category browsers to the new schema
Update the /life species browser to use the category/taxon_category tables
instead of one flat list, split into birds/mammals/insects/snakes-reptiles/
trees/plants/fungi/fish/amphibians. Link each species row to its new
detail page from Step 1. Report species count per category, and give the
942 "uncategorised" species from the migration their own visible section
too (never hidden).

STEP 3 — One place detail page template
Build a place detail page template (e.g. /land/[slug]) that renders, per
place:
1. Basic facts: name (English + Tamil + historical spelling variants if
   present), type, coordinates, elevation if known.
2. Everything linked via the place_link table: species sighted here,
   documents mentioning it, historical passages. If place_link is mostly
   empty (expected — occurrence.place_id was left NULL pending a policy
   decision), say so honestly on the page rather than showing an
   unexplained empty section.
3. Administrative details, sources, completeness label — same pattern as
   species pages.

Test against 3 real places with the most existing data. Report what's
actually populated vs. honestly empty because the underlying links don't
exist yet.

STEP 4 — Document tiering and bibliography display
Add a bibliography/sources page showing the tiered documents (A/B/C) as
citable sources, and a separate "unreviewed documents" section showing the
1,424 unclassified ones — visible, not hidden. Link individual documents
from species/place pages where already referenced as sources.

WHAT NOT TO DO TONIGHT
Don't resolve occurrence.place_id for all 78,467 rows — blocked on
Vishnu's sensitive-occurrence granularity policy and missing boundary
geometry; build templates to handle "not yet linked" gracefully instead.
Don't build the citizen-contribution submission UI — that table wasn't
even created yet, correctly deferred. Don't fix the crocodile/king cobra
category bug unless it blocks Step 2's display, and if so, fix only that
narrow bug. Don't touch the dev/main divergence or outreach emails.

MORNING REPORT
For each step: what real species/places look like on the new pages (with
actual examples, not just "done"), what's honestly still empty and why,
and what needs Vishnu's decision next. Write this to
sathyamangalam/detail-pages-run-results-[date].md.

**Claude** (2026-09-07T16:00): I'll start by reviewing the standing memory notes in full, then verify the actual repo/db state before touching anything — this project's memory explicitly warns that briefs here often have stale premises.

**Claude** (2026-09-07T16:00): _[tool: Bash]_

**Claude** (2026-09-07T16:00): _[tool: Bash]_

**Claude** (2026-09-07T16:00): _[tool: Bash]_

**Claude** (2026-09-07T16:01): _[tool: Bash]_

**Claude** (2026-09-07T16:01): _[tool: Bash]_

**Claude** (2026-09-07T16:01): Let me verify Step 0 first — confirming the actual state of the database before designing anything against it.

**Claude** (2026-09-07T16:01): _[tool: Bash]_

**Claude** (2026-09-07T16:01): _[tool: Bash]_

**Claude** (2026-09-07T16:01): _[tool: Bash]_

**Claude** (2026-09-07T16:01): Table count checks out: 22 tables (plus 1 view). Now row counts and the specific backfill claims from the brief.

**Claude** (2026-09-07T16:01): _[tool: Bash]_

**Claude** (2026-09-07T16:01): _[tool: Bash]_

**Claude** (2026-09-07T16:01): _[tool: Bash]_

**Claude** (2026-09-07T16:02): _[tool: Bash]_

**Claude** (2026-09-07T16:02): _[tool: Bash]_

**Claude** (2026-09-07T16:02): _[tool: Bash]_

**Claude** (2026-09-07T16:03): _[tool: Bash]_

**Claude** (2026-09-07T16:03): _[tool: Bash]_

**Claude** (2026-09-07T16:03): _[tool: Read]_

**Claude** (2026-09-07T16:03): Key premise gaps found already: `taxon` has no `family`, `description`, or `threat` column; `relationship` and `habitat` tables are 100% empty (not just "mostly"); document tiers include an unexplained **D** band (4,990 rows) the brief never mentions. Let me now map the existing site's build system before writing anything.

**Claude** (2026-09-07T16:03): _[tool: Agent]_

**Claude** (2026-09-07T16:03): While that runs, let me continue verifying the remaining DB premises I still need for choosing test species and understanding place/document linkage.

**Claude** (2026-09-07T16:03): _[tool: Bash]_

**Claude** (2026-09-07T16:04): _[tool: Bash]_

**Claude** (2026-09-07T16:04): _[tool: Bash]_

**Claude** (2026-09-07T16:04): _[tool: Bash]_

**Claude** (2026-09-07T16:04): Good — final 5 test species locked in with real variety: Red-vented Bulbul (bird, huge occurrence set), Asian elephant (mammal, sensitive, claims), Bengal Tiger (population trend flagship), Indian Garden Lizard (22 photos), Sloth bear (sensitive, but zero occurrences/photos/claims — the honesty test). Now let me check on the Explore agent and also look at places data while waiting.

**Claude** (2026-09-07T16:04): _[tool: Bash]_

**Claude** (2026-09-07T16:04): _[tool: Bash]_

**Claude** (2026-09-07T16:05): _[tool: Bash]_

**Claude** (2026-09-07T16:05): _[tool: Bash]_

**Claude** (2026-09-07T16:05): _[tool: Bash]_

**Claude** (2026-09-07T16:05): _[tool: Bash]_

**Claude** (2026-09-07T16:05): _[tool: Bash]_

**Claude** (2026-09-07T16:05): _[tool: Bash]_

**Claude** (2026-09-07T16:05): _[tool: Bash]_

**Claude** (2026-09-07T16:05): _[tool: Bash]_

**Claude** (2026-09-07T16:06): _[tool: Bash]_

**Claude** (2026-09-07T16:09): This is a large but tractable task, and I now have a solid empirical picture. Before generating anything, let me check one more premise — the report's target path — and pull the exact HTML/CSS conventions I'll need to match.

**Claude** (2026-09-07T16:09): _[tool: Bash]_

**Claude** (2026-09-07T16:09): Confirmed: no bare `sathyamangalam` directory exists anywhere on this machine — that path in the brief is stale. I'll write the report to the repo root, matching the existing precedent (`overnight-full-fix-2026-08-26.md` already lives there). Now let me pull the exact page conventions I need to match.

**Claude** (2026-09-07T16:09): _[tool: Read]_

**Claude** (2026-09-07T16:09): _[tool: Bash]_

**Claude** (2026-09-07T16:09): _[tool: Bash]_

**Claude** (2026-09-07T16:09): _[tool: Read]_

**Claude** (2026-09-07T16:09): _[tool: Read]_

**Claude** (2026-09-07T16:10): _[tool: Bash]_

**Claude** (2026-09-07T16:10): _[tool: Bash]_

**Claude** (2026-09-07T16:10): _[tool: Read]_

**Claude** (2026-09-07T16:11): _[tool: Read]_

**Claude** (2026-09-07T16:11): _[tool: Bash]_

**Claude** (2026-09-07T16:12): Now I have everything needed. Let me build the shared page scaffolding and the two generator scripts.

**Claude** (2026-09-07T16:12): _[tool: Write]_

**Claude** (2026-09-07T16:12): _[tool: Bash]_

**Claude** (2026-09-07T16:13): _[tool: Write]_

**Claude** (2026-09-07T16:14): Let me fix a syntax risk (backslash inside an f-string expression, which fails pre-3.12) and remove dead code.

**Claude** (2026-09-07T16:14): _[tool: Bash]_

**Claude** (2026-09-07T16:14): _[tool: Edit]_

**Claude** (2026-09-07T16:14): _[tool: Edit]_

**Claude** (2026-09-07T16:14): Now the place page generator.

**Claude** (2026-09-07T16:14): _[tool: Write]_

**Claude** (2026-09-07T16:15): Now let's run both generators and check for errors.

**Claude** (2026-09-07T16:15): _[tool: Bash]_

**Claude** (2026-09-07T16:15): _[tool: Edit]_

**Claude** (2026-09-07T16:15): _[tool: Edit]_

**Claude** (2026-09-07T16:15): _[tool: Bash]_

**Claude** (2026-09-07T16:15): _[tool: Bash]_

**Claude** (2026-09-07T16:15): Both generators ran cleanly. Let me spot-check the 5 chosen species pages and 3 place pages for correctness.

**Claude** (2026-09-07T16:15): _[tool: Bash]_

**Claude** (2026-09-07T16:15): Let me look at the tiger page and sloth bear page content in full to verify the honesty-about-gaps rendering is correct.

**Claude** (2026-09-07T16:15): _[tool: Read]_

**Claude** (2026-09-07T16:15): _[tool: Read]_

**Claude** (2026-09-07T16:15): This is exactly the honesty-about-gaps proof case the brief asked for. Let me check the photos page (garden lizard) and a place page.

**Claude** (2026-09-07T16:15): _[tool: Bash]_

**Claude** (2026-09-07T16:16): Excellent — all rendering correctly and honestly. Now let's wire Step 2: update the export script and `life.html`.

**Claude** (2026-09-07T16:16): _[tool: Bash]_

**Claude** (2026-09-07T16:16): _[tool: Read]_

**Claude** (2026-09-07T16:16): Now let me update the species.json query to add category, and add the new tier C/D export.

**Claude** (2026-09-07T16:16): _[tool: Edit]_

**Claude** (2026-09-07T16:16): Now let me find where `documents_unreviewed.json` is built, to add the new tier C/D export right after it.

**Claude** (2026-09-07T16:16): _[tool: Read]_

**Claude** (2026-09-07T16:16): _[tool: Edit]_

**Claude** (2026-09-07T16:16): _[tool: Bash]_

**Claude** (2026-09-07T16:16): All exports regenerated cleanly, confirming the exact counts. Now let's update `life.html` to add category browsing and detail-page links.

**Claude** (2026-09-07T16:16): _[tool: Edit]_

**Claude** (2026-09-07T16:16): _[tool: Edit]_

**Claude** (2026-09-07T16:17): _[tool: Edit]_

**Claude** (2026-09-07T16:17): Now update `txDraw` to render each row as a link to the detail page:

**Claude** (2026-09-07T16:17): _[tool: Edit]_

**Claude** (2026-09-07T16:17): _[tool: Bash]_

**Claude** (2026-09-07T16:17): `.tx-row{display:grid;position:absolute;...}` — that applies regardless of tag, but I should add link reset styling. Let me add a small CSS rule scoped to `a.tx-row`.

**Claude** (2026-09-07T16:17): _[tool: Edit]_

**Claude** (2026-09-07T16:17): Now wire `renderCatFilters()` into `bootTaxIndex()`:

**Claude** (2026-09-07T16:17): _[tool: Edit]_

**Claude** (2026-09-07T16:17): _[tool: Read]_

**Claude** (2026-09-07T16:17): _[tool: Edit]_

**Claude** (2026-09-07T16:18): _[tool: Bash]_

**Claude** (2026-09-07T16:18): _[tool: Bash]_

**Claude** (2026-09-07T16:18): Syntax checks out. Now let's use `devserve.py` to actually serve the site and verify with a headless check via `curl` + basic assertions.

**Claude** (2026-09-07T16:18): _[tool: Bash]_

**Claude** (2026-09-07T16:18): This exactly mirrors clean-URL production behavior. Let me start it and verify pages load correctly, including the new nested routes.

**Claude** (2026-09-07T16:18): _[tool: Bash]_

**Claude** (2026-09-07T16:18): I need to verify against the local dev server before declaring this correct — and I just found a real bug: my new `site/life/` and `site/land/` directories now shadow the existing `life.html`/`land.html` top-level pages.

**Claude** (2026-09-07T16:18): _[tool: Bash]_

**Claude** (2026-09-07T16:19): _[tool: Bash]_

**Claude** (2026-09-07T16:19): _[tool: Bash]_

**Claude** (2026-09-07T16:19): Good, those use clean URLs already — no changes needed there. Let me fix the generator scripts to preserve `index.html`, then physically relocate the two pages.

**Claude** (2026-09-07T16:19): _[tool: Edit]_

**Claude** (2026-09-07T16:19): _[tool: Edit]_

**Claude** (2026-09-07T16:19): _[tool: Edit]_

**Claude** (2026-09-07T16:20): Now let's physically relocate the two hand-authored pages into their new index.html positions.

**Claude** (2026-09-07T16:20): _[tool: Bash]_

**Claude** (2026-09-07T16:20): _[tool: Bash]_

**Claude** (2026-09-07T16:20): _[tool: Bash]_

**Claude** (2026-09-07T16:20): Good — remaining matches are all legitimate absolute URLs. Now let's re-run both generators (to confirm `index.html` survives) and restart the dev server to verify.

**Claude** (2026-09-07T16:20): _[tool: Bash]_

**Claude** (2026-09-07T16:20): _[tool: Bash]_

**Claude** (2026-09-07T16:21): All 2664 species pages and 92 place pages exist and link correctly. Now let's wire up `land.html`'s curated cards and undescribed list to the new place pages.

**Claude** (2026-09-07T16:21): _[tool: Read]_

**Claude** (2026-09-07T16:21): _[tool: Edit]_

**Claude** (2026-09-07T16:21): _[tool: Edit]_

**Claude** (2026-09-07T16:21): _[tool: Bash]_

**Claude** (2026-09-07T16:21): _[tool: Bash]_

**Claude** (2026-09-07T16:21): Let me add link-reset styling for the anchor variant, consistent with the `a.tx-row` fix earlier.

**Claude** (2026-09-07T16:21): _[tool: Edit]_

**Claude** (2026-09-07T16:21): Now build `SLUG_BY_NAME` in `boot()`:

**Claude** (2026-09-07T16:21): _[tool: Read]_

**Claude** (2026-09-07T16:21): _[tool: Edit]_

**Claude** (2026-09-07T16:21): _[tool: Bash]_

**Claude** (2026-09-07T16:22): Now let's update the "70 undescribed places" heading text since the number is stale (README said ~70/77, but current export shows 88 total display places).

**Claude** (2026-09-07T16:22): _[tool: Bash]_

**Claude** (2026-09-07T16:22): _[tool: Edit]_

**Claude** (2026-09-07T16:22): _[tool: Edit]_

**Claude** (2026-09-07T16:22): _[tool: Edit]_

**Claude** (2026-09-07T16:23): _[tool: Bash]_

**Claude** (2026-09-07T16:23): Now let's update `record.html`: fix stale numbers, insert the new tier section, renumber subsequent sections.

**Claude** (2026-09-07T16:23): _[tool: Edit]_

**Claude** (2026-09-07T16:23): _[tool: Edit]_

**Claude** (2026-09-07T16:23): _[tool: Bash]_

**Claude** (2026-09-07T16:23): _[tool: Edit]_

**Claude** (2026-09-07T16:23): Now let's insert the new "Tiers C & D" section right after the Unreviewed Harvest section, and renumber the following sections.

**Claude** (2026-09-07T16:23): _[tool: Read]_

**Claude** (2026-09-07T16:23): _[tool: Bash]_

**Claude** (2026-09-07T16:23): Let me renumber 04→05, 05→06, 06→07, 07→08 first (in reverse order to avoid collisions), then insert the new section 04.

**Claude** (2026-09-07T16:23): _[tool: Bash]_

**Claude** (2026-09-07T16:24): Now insert the new section 04 before "Archives Worth Visiting in Person":

**Claude** (2026-09-07T16:24): _[tool: Edit]_

**Claude** (2026-09-07T16:24): _[tool: Read]_

**Claude** (2026-09-07T16:24): _[tool: Read]_

**Claude** (2026-09-07T16:24): Now let's insert the new `ec` (every-corpus tier browser) script block right after `bootUnreviewed()`:

**Claude** (2026-09-07T16:24): _[tool: Edit]_

**Claude** (2026-09-07T16:24): Now let's syntax-check and functionally test the updated record.html.

**Claude** (2026-09-07T16:24): _[tool: Bash]_

**Claude** (2026-09-07T16:24): Now let's regenerate all exports/pages, rebuild dist, run `make check`, and functionally test record.html in the dev server.

**Claude** (2026-09-07T16:24): _[tool: Bash]_

**Claude** (2026-09-07T16:25): Now let's wire the Makefile so these generators run automatically as part of the build pipeline, then do a full `make build` dry run.

**Claude** (2026-09-07T16:25): _[tool: Edit]_

**Claude** (2026-09-07T16:25): _[tool: Bash]_

**Claude** (2026-09-07T16:25): _[tool: Bash]_

**Claude** (2026-09-07T16:25): _[tool: Bash]_

**Claude** (2026-09-07T16:25): _[tool: Bash]_

**Claude** (2026-09-07T16:25): _[tool: Bash]_

**Claude** (2026-09-07T16:25): _[tool: Bash]_

**Claude** (2026-09-07T16:26): _[tool: Bash]_

**Claude** (2026-09-07T16:26): _[tool: Bash]_

**Claude** (2026-09-07T16:26): _[tool: Bash]_

**Claude** (2026-09-07T16:27): _[tool: Bash]_

**Claude** (2026-09-07T16:27): _[tool: Bash]_

**Claude** (2026-09-07T16:27): Two clean, separable hunks. Let me stage just the first one for commit 1.

**Claude** (2026-09-07T16:27): _[tool: Bash]_

**Claude** (2026-09-07T16:27): _[tool: Bash]_

**Claude** (2026-09-07T16:27): Exactly the split I wanted. Also stage `exports/species.json` and `exports/manifest.json` for commit 1 (life.html and detail pages need them), then commit.

**Claude** (2026-09-07T16:27): _[tool: Bash]_

**Claude** (2026-09-07T16:27): Good — `site/record.html` and `exports/documents_other_tiers.json` remain unstaged for commit 2. Let's commit now.

**Claude** (2026-09-07T16:27): _[tool: Bash]_

**Claude** (2026-09-07T16:27): Commit 1 succeeded. Now commit 2 for the record.html tier browser and the remaining export change.

**Claude** (2026-09-07T16:27): _[tool: Bash]_

**Claude** (2026-09-07T16:28): _[tool: Bash]_

**Claude** (2026-09-07T16:28): _[tool: Bash]_

**Claude** (2026-09-07T16:28): _[tool: Bash]_

**Claude** (2026-09-07T16:28): Good, still clean. Now let me pull the remaining details I need for the other 3 species and 3 place page write-ups (I've already inspected tiger, elephant, bulbul in detail; need the lizard's occurrence data and the two remaining places).

**Claude** (2026-09-07T16:28): _[tool: Bash]_

**Claude** (2026-09-07T16:28): Now let's get the other two place pages (sathyamangalam-town, bhavanisagar-dam) content.

**Claude** (2026-09-07T16:28): _[tool: Bash]_

**Claude** (2026-09-07T16:28): _[tool: Bash]_

**Claude** (2026-09-07T16:29): Now let me write the comprehensive morning report.

**Claude** (2026-09-07T16:30): _[tool: Write]_

**Claude** (2026-09-07T16:30): Now let me add the matching `.gitignore` entry, following the exact precedent set for the prior overnight report.

**Claude** (2026-09-07T16:30): _[tool: Read]_

**Claude** (2026-09-07T16:30): _[tool: Edit]_

**Claude** (2026-09-07T16:30): _[tool: Bash]_

**Claude** (2026-09-07T16:30): Good — the report file itself is now correctly untracked (ignored), matching the established convention. Let me commit the `.gitignore` update.

**Claude** (2026-09-07T16:30): _[tool: Bash]_

**Claude** (2026-09-07T16:30): _[tool: Bash]_

**Claude** (2026-09-07T16:31): Everything is clean: on `dev`, 17 commits ahead of `origin/dev` (nothing pushed), no dangling processes, working tree clean aside from the pre-existing unrelated stray `main` file I left untouched.

## Summary

All four steps are done, committed to `dev` (3 commits: `0ac2748`, `5ea13f2`, `ac9e37b`), nothing pushed or deployed. Full writeup is in `detail-pages-run-results-2026-09-07.md` at the repo root (gitignored, matching this project's existing convention for these notes).

Highlights:
- **Step 0**: schema confirmed clean (`integrity_check` ok, 22 tables, row counts match).
- **Steps 1 & 3**: built two generator scripts that bake a real static HTML page per species (2,664, at `/life/<category>/<slug>`) and per place (92, at `/land/<slug>`) straight from `atlas.db` — this site has no server runtime, so pages are fully baked at generation time. Every section is honest about gaps that turned out to be larger than the brief assumed: `relationship`/`habitat` are 0 rows for the *entire* atlas (not "most" species), `family`/`description`/`threat` don't exist as columns at all, and elevation is undocumented for *all* 92 places.
- **Step 2**: wired `life.html`'s index to the new category tables — caught that "Trees" is a seeded category with zero species in it (life-form ≠ taxonomic class), and confirmed the crocodile/king-cobra bug is real but narrowly scoped (2 taxa, a malformed `class_` value) and correctly left unfixed per your instructions.
- **Step 4**: extended `record.html` rather than duplicating it, and surfaced tier D — 4,990 documents, the single largest tier, that was reachable from *no page on the site* before tonight.
- Caught and fixed one real bug pre-ship: the new `site/life/`/`site/land/` directories shadowed the existing top-level pages under this site's clean-URL rules, 404ing both — fixed by moving them to directory-index files, verified against the dev server that mirrors production.

Five items are flagged for your decision in the report (crocodile/king cobra fix, tier-C citability tension with your same-day ruling, tree life-form classification, the missing `family` field, and the report's own stale target path).

**Vishnu** (2026-09-07T16:33): Pick one well-documented species (the one with the most data, that you
used to test Step 1) and one well-documented place, and give me a way to
actually see them:
1. If the dev server is running locally, give me the exact URL to open in
   my browser for each (e.g. http://localhost:xxxx/life/mammals/tiger and
   http://localhost:xxxx/land/some-place).
2. If that's not convenient, take a screenshot of each rendered page and
   save it somewhere I can open, and tell me the file path.

**Claude** (2026-09-07T16:34): Good picks to verify with. By raw data volume, the **Red-vented Bulbul** (2,289 occurrence records — by far the most of the 5 test species) and **Talavadi** (3 claims + 1 place_link — the most of the 3 test places) are the clear "most documented" picks. Since this is running on your local machine, I'll start the dev server and hand you direct links.

**Claude** (2026-09-07T16:34): _[tool: Bash]_

**Claude** (2026-09-07T16:34): _[tool: Bash]_

**Claude** (2026-09-07T16:34): Server's up and both pages confirmed live (200). Here are your links:

**Red-vented Bulbul** (most-documented species — 2,289 occurrence records, the largest dataset of the 5 I tested):
**http://localhost:8791/life/birds/pycnonotus-cafer**

**Talavadi** (most-documented place — 3 claims + 1 place_link, more than the other two test places):
**http://localhost:8791/land/talavadi**

A couple of notes so what you see makes sense:
- The bulbul page's highlight is Section 02 ("Where It's Been Recorded") — it clusters those 2,289 raw records into 315 ~1km cells and shows only the top 20 by count, with a "Plus 295 more locations..." note, rather than dumping all of them.
- It has no IUCN/WPA badges and no population claims — that's correct, not a gap: it's a common species with no listing and no claim-table entries, unlike the tiger/elephant pages.
- Talavadi's Section 02 ("Linked Records") shows its one real claim, then explicitly lists all 8 other link types (documents, occurrences, photos, etc.) as zero *for the whole atlas*, not just for this place.

The server (`devserve.py`, PID 10969) will keep running in the background until you stop it or restart your machine — it's not tied to this conversation.