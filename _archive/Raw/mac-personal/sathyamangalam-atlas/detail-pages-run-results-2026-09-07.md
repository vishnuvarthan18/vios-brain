# Overnight run — detail pages — 2026-09-07

Everything below happened on `dev` only, against the local `data/atlas.db`.
No merge to `main`, no `wrangler pages deploy`, no touch to the production
cron or any production database. Two commits landed on `dev`:
`0ac2748` (species/place detail pages) and `5ea13f2` (record.html tier
browser). Nothing was pushed to `origin`.

## Premises the brief stated that the database contradicted

| Brief said | Actual, checked against `data/atlas.db` tonight |
|---|---|
| Report path `sathyamangalam/detail-pages-run-results-[date].md` | No directory named `sathyamangalam` exists anywhere on this machine (checked `~`, `~/Downloads`, `~/Desktop`). The repo itself is `sathyamangalam-atlas`. Written to the repo root instead, matching the one existing precedent (`overnight-full-fix-2026-08-26.md`, also at repo root, also `.gitignore`d — see below). |
| "if empty for most species" (relationship table) | `relationship` holds **0 rows total**, for the whole atlas, not "most" species. `habitat` is also 0 rows. Every species page says this plainly and names the actual row count rather than implying it's species-specific. |
| "categories ... snakes-reptiles" | The actual seeded category slug/label is `reptiles` / "Reptiles" (`CATEGORY_SEED` in `scripts/apply_schema_migration_2026-09-07.py`). Used "Reptiles" throughout; noting the naming mismatch here rather than silently renaming the brief's term into the code. |
| "tiered documents (A/B/C) as citable sources" | Vishnu ruled the same day (`scripts/tier_documents.py`, "Ruled by Vishnu 2026-09-07") that **only A and B** feed the citable bibliography; C is deliberately excluded pending review. Tonight's instruction reads as loosening that same-day ruling. Did not silently override it — see "Needs your decision" below. |
| (implicit) A/B/C covers the tiered document space | There is also a **tier D, 4,990 rows — the single largest tier** — that predates this repo's tiering script and was, before tonight, reachable from **no page on the site at all**: not in `documents.json` (A/B only), not in `documents_unreviewed.json` (that's specifically `relevance_tier IS NULL`, not "everything uncited"). It was just absent. Now shown, honestly labelled as unreviewed-provenance, in the new record.html tier browser. |
| "the crocodile/king cobra category bug" | Real, but not where the brief implied. The **curated** `species_curated.json` (life.html's existing "Species Atlas" cards) already tags both correctly as `"k":"herp"` — no bug there. The bug is in the **new** `taxon_category` table this run's Step 2 wires up: `Crocodylus palustris` and `Ophiophagus hannah` both carry `class_='Reptilia/Amphibia'` (a slash-joined value), which matches none of the category rules, so both land in `uncategorised` instead of `reptiles`. Exactly 2 taxa affected. The migration script already self-flags this (`scripts/apply_schema_migration_2026-09-07.py` lines ~320–326) — not a new discovery, but confirmed live and now visible on the site for the first time because Step 2 switches the browser to the table where it lives. Left unfixed, per instruction ("only if it blocks Step 2's display" — it doesn't; `uncategorised` renders them honestly). See "Needs your decision." |
| "trees" as one of the 9 browse categories | `trees` is a real, seeded category (`category` table, `is_functional=1`) with **0 taxa ever assigned to it**. It's not a taxonomic class — "tree" is a life-form — so the migration's automatic `class_`-based rule had nothing to assign there. Shown as its own visible, honestly-empty tab (0), not hidden, not silently merged into `plants`. |
| Species facts implicitly assumed a `family` / `description` / `threat` field | None of the three exists anywhere in the schema — not unpopulated, **not modelled**. `taxon` has `kingdom`, `class_`, `rank` only. Every species page states this explicitly rather than omitting the row or leaving it blank. |
| Place "elevation if known" | Known for **0 of 92 places**, not "sometimes." Stated as a schema-wide fact on every place page, not a per-place gap. |
| Place "historical spelling variants if present" | No such field exists in `place` or anywhere else in the schema — a modelling gap, not a data gap. Stated as such. |

## STEP 0 — Schema confirmation

`PRAGMA integrity_check` → **ok**. 22 tables (confirmed via
`sqlite_master`, matching the brief), plus the one view
(`v_place_link_orphans`) that isn't counted as a table.

| Table | Rows |
|---|---|
| taxon | 2,664 |
| occurrence | 78,467 |
| category | 10 |
| taxon_category | 2,664 (strict 1:1 with taxon — see Step 2) |
| habitat | 0 |
| relationship | 0 |
| photo | 576 |
| place_link | 15 |
| completeness | 2,756 (2,664 taxon.v1 + 92 place.v1) |
| place | 92 |
| claim | 83 |
| document | 9,029 |

Backfills confirmed: `taxon.canonical_name` populated for 2,662/2,664 (2
gaps, not investigated tonight — out of scope). `occurrence.public_lat` /
`public_lon` populated for all 78,467/78,467; `occurrence.place_id` is
**NULL for all 78,467**, exactly matching the brief's own "what not to do"
note. Proceeded — nothing here blocked the run.

## STEP 1 — Species detail page template

Built `scripts/build_species_pages.py`: reads `atlas.db` directly (bulk
queries, grouped in Python — not 2,664×N per-taxon queries) and writes a
fully-baked static HTML file per taxon to
`site/life/<category>/<slug>.html`. Static generation, not a client-side
fetch, because this is a Cloudflare Pages static site with **no server
runtime** (no Functions directory anywhere in the repo — checked before
choosing this approach). 2,664 pages regenerate in a few seconds.

Ran against all 2,664 taxa. The 5 test species, chosen for genuinely
different data profiles (not the same species with different framing):

**Bengal Tiger** (*Panthera tigris tigris*, mammals) — sensitive species.
Common names EN/TA both present. IUCN EN, WPA Sch I. Family: honestly
"not documented." One occurrence record (11.60°N, 77.00°E, 2018) — its
`coarsened_by` is `legacy-import` with `public_coord_basis`
`coarsened-0.05deg`, so the page's sensitive-species note fires correctly.
Population & trend: **2 claims** — `population` (112, 2024–25, sourced) and
a `note` claim describing the 2010 management-plan figure (8–10 from scat
DNA) — this is exactly the worked example a prior session's memory flagged
as already live in `claim`, confirmed again tonight. Relationships: honest
zero, with the global-emptiness explanation. Photos: honest zero (576
photos exist in the atlas; none tagged to this taxon). Completeness: 71%
(taxon.v1), missing `kingdom`/`gbif_key`, not yet human-verified.

**Asian elephant** (*Elephas maximus*, mammals) — sensitive. 17 occurrence
records, clustered into a handful of ~1 km cells. 2 claims (a `population`
figure and a `note`). No photos. Completeness 86% (only `kingdom` missing)
— the highest of the five, correctly reflecting that this is one of the
best-documented taxa in the atlas.

**Red-vented Bulbul** (*Pycnonotus cafer*, birds) — the "raw dump" stress
test. 2,289 raw occurrence records. The page groups them into 315 distinct
~1 km cells, shows the top 20 by count (up to 221 records in one cell,
spanning 2015–2024), and states "Plus 295 more location(s) with 1,204
additional record(s), not shown individually" rather than rendering all
2,289 rows or all 315 cells. No IUCN/WPA (correctly absent — this is an
LC-equivalent common species with no listing), no claims.

**Indian Garden Lizard** (*Calotes versicolor*, reptiles) — the photo test.
22 photos (of the 24-tile display cap), each with its own
attribution/observer/licence pulled straight from the `photo` table (mix of
`CC-BY-4.0` and `CC-BY-NC-4.0` in the actual data). 53 occurrences grouped
into 6 cells. No claims.

**Sloth bear** (*Melursus ursinus*, mammals) — sensitive, and the honesty
test: **zero** occurrences, zero photos, zero claims. The page renders all
nine sections and every one of 2–8 says plainly that nothing exists yet,
rather than collapsing the page or hiding sections. IUCN VU still surfaces
correctly (from the `taxon` row itself, unaffected by the empty
child-tables). Completeness 71%.

Global honesty facts, verified once and applied to all 2,664 pages rather
than re-derived per page: **family/description/threat have no column at
all**; **relationship is 0 rows for the entire atlas**; sources dedupe by
`source.name` before display, because harvest pagination fragments one
logical GBIF download into dozens of `source_id` rows differing only by an
`offset=` query parameter (confirmed: e.g. 47 separate `source` rows for
what is really one `gbif-occurrence-search` fetch) — showing 47 near-
identical rows would itself have been a dishonest "dump," so occurrence
sources are capped to the top 5 by record count with a "+N more" note.

## STEP 2 — Category browser wiring

`species.json`'s export query now LEFT JOINs `taxon_category` (verified
1:1 — `taxon_category` has exactly 2,664 rows, one per taxon, `is_primary`
always 1, so the join never fans out). life.html's "Full Taxonomic Index"
gained a category filter row above the existing named/IUCN/sensitive/
invasive filters, with live counts, and every row is now a link to its
detail page.

| Category | Species |
|---|---|
| Uncategorised | 942 |
| Fungi | 771 |
| Birds | 341 |
| Plants | 316 |
| Insects | 221 |
| Mammals | 36 |
| Reptiles | 29 |
| Amphibians | 7 |
| Fish | 1 |
| Trees | 0 |

Sums to 2,664 exactly. Uncategorised gets its own tab, front and center,
not a hidden fallback — clicking it shows all 942 with the same filters
and search as every other category. Trees gets a tab too, showing 0, with
the section copy explaining why (life-form, not a taxonomic class — see
premises table above) rather than omitting the tab or quietly folding it
into Plants.

The crocodile/king-cobra miscategorisation (see premises table) was **not
fixed** — it doesn't block the display (both render correctly, just under
Uncategorised instead of Reptiles), and the instruction was to leave it
unless it blocked Step 2. Flagged below for your call.

## STEP 3 — Place detail page template

`scripts/build_place_pages.py` writes `site/land/<slug>.html` for all 92
places. Ran against the 3 places with the most existing data (`talavadi`,
3 claims + 1 place_link; `sathyamangalam-town` and `bhavanisagar-dam`, 1
claim each — these were the actual top-3 by data volume, not a subjective
pick).

**Talavadi** — name EN present, TA present (தாளவாடி), type "town",
coordinates present, elevation "not documented" (see premises table — true
of all 92 places), forest range/division both "not applicable/not
documented" (it's a town, not a forest range). Linked records: 1 claim
("Forest range (8,300.98 ha) with Palayam, Belathur and Geddavady beats...",
sourced). The other 8 linkable target types (document, occurrence,
news_event, legal_instrument, historical_passage, observation_layer,
photo, contribution) each get their own line stating plainly that 0 rows
of that type exist **anywhere in the atlas**, not just for this place.
Completeness 43%, missing elevation_m/population/forest_range/wikidata_id.

**Sathyamangalam town** and **Bhavanisagar Dam** — same shape: name EN
present, TA absent ("not documented"), 1 claim each (their description
claim), same 8 honest zero-lines for every other link type, completeness
14% each (both missing 6 of 7 profile fields — elevation, population,
forest_range, wikidata_id, name_ta, osm_id).

None of the 3 test places had ANY document, occurrence, historical-passage
or photo link — expected, since `place_link` holds exactly 15 rows total
and all 15 are `target_table='claim'`. This is not a per-place gap; it's
that occurrence→place linking was never built (see WHAT NOT TO DO — your
sensitive-occurrence granularity policy blocks it) and document/
historical_passage/photo→place linking has simply never been populated.
The page says so, by name, for every place, not just these 3.

As a light, in-scope addition (not explicitly requested by Step 3, but the
pages would otherwise be unreachable from anywhere on the site): land.html's
15 curated place cards and ~73 "not yet described" plain-list places now
link to their detail pages, matched by name (curated cards) or by the
`slug` field already present in `places_display.json` (undescribed list).
All 15 curated names matched a `place.name_en` exactly — verified before
wiring it, not assumed.

## STEP 4 — Bibliography / document tiering

Extended `record.html` (already the site's sources/bibliography page —
didn't create a duplicate second page) rather than building a new one.
Added `exports/documents_other_tiers.json` (tier C + D, 6,632 rows) and a
new "Tiers A–D" section with the same virtualised-table pattern already
used for the unreviewed harvest, filterable by tier:

| Tier | Rows | Currently reachable from |
|---|---|---|
| A | 441 | `documents.json` (existing) + new browser |
| B | 532 | `documents.json` (existing) + new browser |
| C | 1,642 | **new browser only** — was reachable nowhere before tonight |
| D | 4,990 | **new browser only** — was reachable nowhere before tonight |
| NULL (unreviewed) | 1,424 | `documents_unreviewed.json` (existing, unchanged) |

973 = A+B, matches `documents.json`'s row count exactly (cross-checked).
The new section's own copy states, in the page itself, that only A/B are
treated as citable — it does not silently promote C into the bibliography
just because tonight's brief said "A/B/C." See "Needs your decision."

Also fixed three stale hardcoded numbers left over from before today's
`tier_documents.py` run (928 → 973 assessed A/B documents, in both the
icontile and the bibliography paragraph; the 1,500-target percentage
recomputed to 64.9%; the "7,190 raw harvest, ~87% irrelevant" framing
replaced with the actual current 9,029-document total and tier
breakdown) — these were adjacent to what I was already editing and
directly contradicted the numbers going in right next to them.

Document→species/place linking (Step 4's last bullet): checked whether
any taxon or place currently references a specific `document` row.
`relationship.document_id` exists as a column but `relationship` is 0
rows; `place_link` has 0 rows with `target_table='document'`. **There is
currently nothing to link** — not a bug, just genuinely empty. The species/
place pages' Sources sections already read from `source_id` (which is
populated) and would pick up a real document link the moment one exists;
nothing further to build until that data exists.

## Verification performed before calling this done

- `PRAGMA integrity_check` — ok (re-checked after all writes; nothing in
  tonight's work touches `atlas.db`, it's read-only in both generator
  scripts).
- `make check` — all `exports/*.json` parse.
- `make build` — succeeds, 2,825 files, 56.6 MB, no missing required file.
- Cross-checked every species (2,664) and every place (92) in
  `exports/species.json` / `exports/places.json` against the generated
  file tree — 0 missing detail pages either direction.
- Caught and fixed a real bug before shipping: `site/life/` and
  `site/land/` as new directories **shadowed** the existing top-level
  `life.html`/`land.html` under this project's clean-URL serving rule
  (confirmed against `scripts/devserve.py`, which mirrors production) —
  `/life` and `/land` both 404'd until `life.html`/`land.html` were moved
  to `life/index.html`/`land/index.html`. Re-tested after the fix: `/life`,
  `/land`, and nested detail-page URLs all return 200.
- Node syntax-checked every modified inline `<script>` block
  (`life.html`, `land.html`, `record.html`).

## Needs your decision

1. **Crocodile / king cobra category bug.** `class_='Reptilia/Amphibia'`
   for exactly these 2 taxa doesn't match the reptile rule
   (`class_ IN ('Squamata','Testudines','Crocodylia','Reptilia')`), so both
   sit in Uncategorised instead of Reptiles. One-line fix either way: split
   `class_` into its correct single value for these two rows, or add
   `'Reptilia/Amphibia'` to the reptile rule. Left alone tonight per
   instruction (doesn't block Step 2's display). Your call which fix, if
   any.
2. **Tier C's citability.** Tonight's brief said "A/B/C as citable
   sources"; your same-day ruling in `tier_documents.py` says only A/B are
   citable and C is regional context pending review. Built to the standing
   ruling (C is browsable, clearly separated, not in the bibliography) and
   flagged the tension in the page copy itself rather than picking a side.
   Confirm which stands.
3. **Tree life-form classification.** The `trees` category exists in
   schema with 0 taxa. Populating it needs a life-form data source (tree
   vs. shrub vs. herb within Plantae) that nothing in the current pipeline
   provides — not attempted tonight, correctly out of scope for a display
   task, but flagged since Step 2 asked for exactly this category.
4. **`taxon.family`.** Doesn't exist in the schema at all. Every species
   page says so. Adding it would need a new data source (e.g., a GBIF
   backbone join on `gbif_key`, populated for only 813/2,664 taxa) — a data
   -acquisition decision, not something to invent from what's on hand.
5. **occurrence.place_id** — untouched, as instructed (your
   sensitive-occurrence granularity policy + missing boundary geometry).
   Every place page states this by name rather than showing a silent empty
   section.

## Not done, correctly deferred

- occurrence.place_id resolution for the 78,467 rows.
- Citizen-contribution submission UI (table doesn't exist).
- dev/main branch divergence, outreach emails — untouched.
- Nothing pushed to `origin`, nothing merged to `main`, no deploy.
