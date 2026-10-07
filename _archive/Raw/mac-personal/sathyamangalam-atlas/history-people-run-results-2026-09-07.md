# Overnight run — History & People — 2026-09-07

Everything below happened on `dev` only, against the local `data/atlas.db`
(never written to — every generator here is read-only, same trust root as
last night's species/place pages). No merge to `main`, no `wrangler pages
deploy`, no touch to the production cron or any production database.
Nothing pushed to `origin`. The five items flagged in
`detail-pages-run-results-2026-09-07.md` (crocodile/king cobra category,
tier-C citability, tree life-form, missing `family` field, stale report
path) were **not touched** — none of them blocked a step below.

## Premises the brief stated that the database contradicted

| Brief said | Actual, checked against `data/atlas.db` tonight |
|---|---|
| "document, historical_passage, legal_instrument, news_event tables ... hold this content" | `historical_passage` holds **1 row**, `legal_instrument` **2 rows**, `news_event` **0 rows** — not enough to be a section. The real seed is 80 rows inside `document` tagged `kind='reference'`: the actual primary-source catalog (Nicholson's 1887 Manual, the Madras Forest Act 1882, Buchanan's 1807 survey, Tamil Nadu Gazette notifications, decades of dated Bhavanisagar assembly records) that `history.html`'s hand-written timeline already draws from — distinct from the 8,949 `kind='article'` rows, which are the general ecology bibliography `record.html` already serves. See "Scope decision" below. |
| "Report path `sathyamangalam/history-people-run-results-[date].md`" | No directory named `sathyamangalam` exists anywhere on this machine (checked again tonight, same result as last night). Written to the repo root instead, matching both existing precedents. |
| "test against 5 real records with good data" | Once the real population (83 rows: 80 reference documents + 1 passage + 2 legal instruments) is used instead of the near-empty three named tables, 83 clears "5" easily — built and verified against all 83, not a cherry-picked 5. The report below narrates 5 of them for range, per the brief's spirit. |
| "Anything resembling people/community content ... Keystone Foundation material, tribal community records" | **Zero** structured rows anywhere in the 22-table schema — no `community`, `organization`, or `contributor` table exists at all. Every fact about Soliga/Irula/Kurumba/Malasar or Keystone Foundation/ATREE/Kalpavriksh on `people.html` is hand-written narrative text, sourced but not modelled as rows. This is the "essentially zero" case the brief anticipated — see Step 2. |
| (implicit) People and History are new sections needing nav wiring | Both `history.html` and `people.html` **already existed** as full, well-sourced narrative pages before tonight, and `/history` + `/people` were **already** in the header nav, footer nav, homepage tiles, homepage explore-modal, breadcrumbs, sitemap.xml, the search palette, and `site_common.py`'s shared chrome (used by every generated species/place page). What was actually missing was **per-record detail pages**, the same gap species/place had before last night. See Step 4 — there was nothing to wire. |

## STEP 0 — Confirm state and inventory

`PRAGMA integrity_check` → **ok** (checked before and after — nothing tonight writes to `atlas.db`).

| Table | Rows | What's actually in it |
|---|---|---|
| `document` (kind='reference') | 80 | The real primary-source catalog: gazetteers, acts, surveys, Tamil Nadu Gazette notifications, ~20 dated Bhavanisagar-specific legislative-assembly records (1954–2023), plus a handful of Wikipedia/Wikidata/GBIF citation stubs |
| `document` (kind='article') | 8,949 | The general ecology/biodiversity academic bibliography — out of scope for History, already served by `record.html` |
| `historical_passage` | 1 | The 1927 ASI transcription of the Kaveripuram inscription (ARE No. 193 of 1927) — genuine transcribed text, sourced to *South Indian Inscriptions, Vol. 19* |
| `legal_instrument` | 2 | 2019 NTCA Management Effectiveness Evaluation (award), 2020–2022 CA\|TS accreditation — **not** government notifications, despite the schema's `kind`/`number`/`dated`/`issuing_body` columns clearly being built for those. The real notifications (2011, 2013, 2018, 2025 Tamil Nadu Gazette entries) ended up filed under `document(kind='reference')` instead — a real modelling mismatch, not touched tonight since it's a schema/ETL decision, not a display one. |
| `news_event` | 0 | Nothing |
| Community/organisation/contributor tables | **none** | Not one of the 22 tables in `atlas.db` models this. `people.html`'s content is entirely hand-authored. |
| `photo.observer` | 576 photos / **48 distinct named people** | Real, structured, sourced — but citizen-science photo attribution (iNaturalist usernames, public profile URLs, open licences), not community/organisation representation. See Step 2. |

Also checked and ruled out before designing to them: `claim.subject_taxon_id` and `claim.subject_place_id` are never both set on the same row (0 rows) — so there is no explicit structured species↔place link anywhere in the data, confirming last night's finding still holds. `place_link`'s `target_table` CHECK constraint already permits `'document'`, `'historical_passage'`, `'legal_instrument'`, `'news_event'` — the schema anticipated exactly tonight's cross-linking need — but has **0 rows** of any of those types (all 15 existing rows are `target_table='claim'`), so nothing was already linked; everything in Step 3 below is newly surfaced.

**Scope decision, stated as a rule:** "historical record" for tonight's build = `document` rows with `kind='reference'` (80) + all of `historical_passage` (1) + `legal_instrument` (2) + `news_event` (0) = **83 records**. The 8,949 `kind='article'` rows are excluded — they're the general bibliography, and folding them into History would duplicate `record.html` and dilute what "History" means on this site, not add to it. This is a boundary I'm naming explicitly rather than deciding silently; if it's wrong, the fix is a one-line change to `site_common.fetch_history_records()`.

## STEP 1 — History detail pages

Built `scripts/build_history_pages.py`, following the same read-only, bulk-query, static-generation pattern as `build_species_pages.py` / `build_place_pages.py`. Writes `site/history/<slug>.html` for all 83 records; also patches a new, machine-regenerated "Primary-Source & Legal Record Catalog" section into `site/history/index.html` between HTML comment markers (`HISTORY_CATALOG:START`/`:END`) so re-running the script never clobbers the hand-written narrative around it.

**The one-time migration:** `site/history.html` was moved to `site/history/index.html` (same shadow-fix last night applied to `life.html`/`land.html` — the clean URL `/history` needs a directory now that `/history/<slug>` exists). Every internal relative link (`href="record"`, `href="people"`, `href="land#gazetteer"`, `href="credits"`, `href="govern#legal"`, plus every asset path) was rewritten to root-relative form (`/record`, `/people`, ...) — mechanically, via a script that only touches attribute values not already starting with `/`, `http`, `#` or `mailto:`, then diffed line-by-line before applying. All the hand-written timeline content, the pull-quote, the four featured-paper cards, and the decade bar chart are untouched.

Each detail page renders the brief's four points, adapted per record type (document / historical passage / legal instrument / news event have different real fields):
1. **What it is** — type badge, date (or "not documented" — 27 of 80 reference documents have no year), and a description. For `document` rows, the `authors` column is doing double duty as a short curatorial note rather than a byline (e.g. "The single richest document on this landscape — ranges, beats, checklists, climate, working circles") — the page says so rather than presenting it as an author credit.
2. **Source text** — the historical_passage row's `text` field is genuine transcribed source text; everything else has no stored full text, so the page says that plainly and links to the external URL where one is recorded (67 of 83 records have a URL) rather than fabricating an excerpt.
3. **Cross-links** — see Step 3.
4. **Completeness** — stated as **not scored**, not blank-by-omission: `completeness_profile` only registers `taxon.v1` and `place.v1`, nothing for history records, so no score is invented.

**Five records, chosen for range (not the first five in the export):**

- **Battle of Sittimungulum** (doc id 1, 1790, tier A) — the earliest record and the whole site's origin story. No stored full text; links out to the Wikipedia article recorded as its source.
- **Manual of the Coimbatore District** (doc id 5, Nicholson, 1887, **tier D**) — the single most load-bearing source for the entire History narrative, and it sits in the lowest citability tier from last night's automated tiering pass. A concrete instance of "a right total can sit on a wrong description" from the earlier tiering run: tier alone would suggest this is marginal; it's the opposite.
- **TN Forest Dept Management Plan 2010–2020** (doc id 17, tier A, `full_text_ok=1`) — the richest single document in the atlas per its own `authors`-as-description field; flagged as full-text accessible at its academia.edu URL (no local copy stored).
- **Desilting of Bhavanisagar Dam** (doc id 7491, 1993, tier A) — one of three Bhavanisagar-Dam-specific records that now carry a real, mechanically-detected cross-link to the `Bhavanisagar Dam` place page (see Step 3).
- **ARE No. 193 of 1927** (the only `historical_passage` row) — genuine transcribed source text ("Kongu-era Vikrama Chola land-endowment inscription... Kaveripuram was later submerged by the Stanley Reservoir"), the richest "Source Text" section of any page built tonight precisely because it's the one row where the schema's intended use (a transcribed passage) actually happened.

The two `legal_instrument` rows (2019 MEE evaluation, 2020–2022 CA\|TS accreditation) are both built and both real, but neither is a legal notification despite the table's name — see the Step 0 table above.

**Verification:** cross-checked all 83 records in `atlas.db` against the file tree in `site/history/` — 0 missing, 0 extra, both directions. `python3 scripts/devserve.py` + manual requests confirm `/history`, `/history/<real-slug>` (checked 3), `/land`, `/life` all return 200, and a fabricated slug (`/history/doc-nonexistent-slug`) correctly 404s. `make check` (exports parse) and `make build` (2,908 files, 58.4 MB, up from last night's 2,825/56.6 MB by exactly 83) both succeed.

## STEP 2 — People/Community section

**Zero structured data. Went the honest-index route, but the index already existed.**

Unlike History, there is no table anywhere in the schema that models communities, organisations, or tribal records — not unpopulated, not modelled at all. `people.html`'s existing content (Soliga/Irula/Kurumba/Malasar narrative, the FRA claims record, the consent protocol) is entirely hand-authored prose, sourced the same way the rest of the atlas is, but not queryable and not backed by any per-community or per-organisation row.

Per instruction, this ruled out building `/people/<slug>` detail pages — there is nothing to generate them from. It also meant Step 2's instruction to "build a single honest /people index page" needed adjusting to the actual premise: `/people` already exists, already reads as an honest, well-sourced page (it already states its own record is "incomplete precisely where it matters most"), and duplicating it with a second flat page would be worse than extending it. So `people.html` stayed a flat page (no directory conversion — there's nothing to shadow-fix, since no `/people/<slug>` exists), and gained one new section:

> **03 — What This Page Is, and Isn't, Built On.** States plainly, by name, that no `community`/`organisation`/`contributor` table exists in `atlas.db` (checked against all 22 tables before writing this), that every fact on the page is hand-written narrative rather than database-backed the way Life/Land/History now are, and that this page is the whole of the People section right now, not an index into a larger one.

**A real, differently-shaped dataset exists and is flagged, not acted on:** `photo.observer` holds **48 distinct named individuals** behind the atlas's 576 species photographs — real iNaturalist usernames with public profile URLs and open licences (`parithi`, 170 photos, 84 species; `pjeganathan`, 65 photos; down to 27 people with a single contribution each). This is genuine "people" data in the sense of named, sourced individuals, but it is citizen-science photo attribution, categorically different from the tribal-community/organisation framing the brief's own examples pointed at. Building `/people/<observer>` contributor pages from it is entirely feasible (the data is real and complete: username, count, species covered, licence, profile link) — but folding photographers in under the same "People" banner as forest-dwelling communities fighting active FRA claims is an editorial call, not a mechanical one, and isn't mine to make silently. **Flagging for your decision** (see "Needs your decision" below); nothing was built from this data tonight.

## STEP 3 — Cross-link what's already linkable

Implemented in `site_common.compute_history_crosslinks()` (one shared function, used by all three generators) as a mechanical, auditable rule: **whole-word, whole-string, case-insensitive matches** between a history record's title/summary/text and (a) a real place's full `name_en`, or (b) a real taxon's `scientific_name`/`canonical_name` — **never** `common_en`. That exclusion is deliberate and is the one judgement call in this step, so it's stated as a rule rather than applied ad hoc: matching tried `common_en` too, and the single common name "Tiger" alone matched **13 of the 83 records** — mostly through "Sathyamangalam **Tiger** Reserve", "**Tiger** Conservation Plan", "Conservation Assured \| **Tiger** Standards" (institutional names, not statements about the animal), mixed in with a few that plausibly are about the animal itself ("Old Veerappan lair set to be a **tiger** den", "**Tiger** presence in a hitherto unsurveyed jungle"). Disambiguating those 13 needs human judgement per item; none were auto-linked, and all 13 are listed below rather than silently dropped.

**Real cross-links applied: 12 edges, across 9 distinct history records and 8 distinct place/species pages.**

| Type | Count | Detail |
|---|---|---|
| Species ↔ place | **0** | Confirmed again tonight: `occurrence.place_id` is NULL for all 78,467 rows, `place_link` has 0 rows targeting `document`/`occurrence`, and no `claim` row has both `subject_taxon_id` and `subject_place_id` set. Nothing to link — not a new finding, but re-verified rather than assumed from last night's report. |
| Species ↔ history | **5 edges / 3 documents / 4 species** | *Native mammals disperse Senna spectabilis* (2021) → `Senna spectabilis` and `Senna` (genus); *The Effective Management of Prosopis juliflora...* (2023–24) → `Prosopis juliflora`; *Tree husbandry practices in Tamilnadu* → `Prosopis juliflora` and `Casuarina equisetifolia`. All four matched taxa are invasive-tree species, which tracks — these are exactly the documents actually about a named plant, not general reserve reports. |
| Place ↔ history | **7 edges / 6 documents / 4 places** | Three Bhavanisagar-Dam-specific records (*Bhavanisagar Dam as Tourist Centre* 1996, *Desilting of Bhavanisagar Dam* 1993, *Release of water from Bhavanisagar Dam* 1990) → `Bhavanisagar Dam`; *New record of ... from Moyar River* (1997) → `Moyar`; *Lifting of vehicular traffic ban on Bannari-Thimbam road* (2022) → `Bannari`; *Handing over land ... to Arulmiru Sri Bannari Amman Temple* (2023) → both `Bannari` and `Bannari Amman Temple`. |

Both directions render on real pages: e.g. `site/land/bhavanisagar-dam.html` now has a "History & Legal Records Naming This Place" section listing those three documents, and each of those three document pages under `/history/` has a "Places & Species This Record Names" section pointing back to Bhavanisagar Dam. Every rendered cross-link carries an explicit note that it's a mechanical whole-name match, not a manually curated one, with a link to the rule.

**A near-miss, surfaced rather than silently bridged:** the phrase "Sathyamangalam Tiger Reserve" appears verbatim in 4 of these records, but the place row's actual `name_en` is `"Sathyamangalam WLS/Tiger Reserve"` — not an identical string, so the strict rule correctly does *not* link it. This is a real naming inconsistency between how the reserve is referred to in sourced text versus how it's spelled in the `place` table, not a bug in the matching code. Fixable either by adding an alias to the place row or by relaxing the matching rule for this one entity — both are edits to source-of-truth data or to the matching rule itself, so raised here rather than patched.

**The 13 excluded "Tiger" candidates**, for your call on any you'd want promoted individually (documents unless noted): *Tiger Conservation Plan & MEE — STR*; *Sathyamangalam Tiger Reserve official portal*; *Tiger presence in a hitherto unsurveyed jungle of India*; *A contemporary assessment of tree species in Sathyamangalam Tiger Reserve*; *Floristic Structure of Sathyamangalam Tiger Reserve...*; *Old Veerappan lair set to be a tiger den*; *Celebrating tiger numbers while tribal residents await their rights*; *TN night ban on road via tiger reserve...*; *Sathyamangalam Tiger Reserve* (Wikipedia ref); *Wikidata Q2226064 — Sathyamangalam Tiger Reserve*; the 2019 NTCA MEE evaluation and the CA\|TS accreditation (both `legal_instrument` rows).

## STEP 4 — Site navigation update

**Nothing to build — verified, not changed.** `/history` and `/people` were already present, identically to `/land` and `/life`, in: the header nav (`site_common.page_header()`, already used by every generated page), the footer nav, the homepage hero tiles (`index.html` lines 113–138), the homepage "Explore" modal (lines 634–643), the breadcrumb chrome, `sitemap.xml`, and the search palette in `shared.js`. Confirmed live via `devserve.py`: `/`, `/land`, `/life`, `/people`, `/history` all return 200. Homepage was not touched, per instruction, beyond nothing — no edit was needed.

**One adjacent, unfixed observation** (not touched, since the instruction was explicitly hands-off on the homepage): the homepage's Life tile still reads "1,893 Taxa Recorded" against a live count of 2,664 — stale, but outside tonight's scope (nav *linking*, not tile *copy*), so left alone and flagged here rather than fixed as a drive-by edit.

## Verification performed before calling this done

- `PRAGMA integrity_check` — ok, before and after (nothing tonight writes to `atlas.db`).
- Cross-checked all 83 history records against `site/history/*.html` — 0 missing, 0 extra, both directions.
- `make check` — all `exports/*.json` parse.
- `make build` — 2,908 files, 58.4 MB (up from 2,825/56.6 MB by exactly the 83 new history pages + `index.html`).
- `devserve.py` + direct requests: `/`, `/land`, `/life`, `/people`, `/history`, three real `/history/<slug>` pages, `/land/bhavanisagar-dam`, `/life/uncategorised/senna-spectabilis` all 200; a fabricated `/history/<slug>` 404s correctly.
- Diffed the full `site/history.html` → `site/history/index.html` link-rewrite line-by-line before applying — confirmed only relative internal hrefs/srcs changed, external URLs and in-page anchors untouched.
- `git status` reviewed in full: every changed file is one of `Makefile`, the 3 modified generator scripts + `site_common.py`, `people.html`, the deleted `site/history.html`, or a regenerated file under `site/land/`/`site/life/`/`site/history/`. Nothing under `docs/`, `harvest-engine/`, `data/`, or the 5 items flagged from last night's report was touched. The pre-existing untracked `main` file at the repo root was not created or modified by tonight's work — it was already untracked when this run started.

## Needs your decision

1. **Photo-contributor "People" content.** 48 real, named, sourced individuals exist in `photo.observer` — genuine data, distinct in kind from the tribal-community framing `people.html` uses. Build `/people/<observer>` contributor pages from it, build it as a separate section entirely (e.g. under `/credits` or a new path), or leave it as inline photo attribution only (status quo)? See Step 2.
2. **`legal_instrument` vs. the actual notifications.** The table's own schema (`kind`, `number`, `dated`, `issuing_body`) is clearly built for government orders and gazette notifications, but its 2 real rows are an award and an accreditation — the actual notifications live under `document(kind='reference')` instead. Worth an ETL fix to move/tag them properly, or leave as-is since tonight's `/history` catalog surfaces both regardless of which table they're in?
3. **"Sathyamangalam Tiger Reserve" naming mismatch.** 4 real historical/legal records name the reserve exactly as "Sathyamangalam Tiger Reserve"; the `place` row is `"Sathyamangalam WLS/Tiger Reserve"`. Add an alias, rename the place row, or leave the strict no-alias matching rule as the permanent policy?
4. **The 13 excluded "Tiger" candidates** (Step 3) — any of these worth a manual, individually-reviewed link despite the generic-common-name exclusion rule?
5. Everything already flagged in `detail-pages-run-results-2026-09-07.md` (crocodile/king cobra, tier-C citability, tree life-form, `family` field, stale report path) — untouched tonight, still open.

## Not done, correctly deferred

- `/people/<slug>` detail pages — no structured data exists to generate them from (Step 2).
- Species ↔ place cross-links — genuinely zero explicit structured data anywhere (Step 3), unchanged from last night's finding.
- The 13 "Tiger" candidate cross-links — need per-item human judgement, not a mechanical rule (Step 3).
- `legal_instrument` re-population with the real gazette notifications — an ETL/schema decision, not a display one.
- dev/main branch divergence, outreach emails — untouched, per instruction.
- Nothing pushed to `origin`, nothing merged to `main`, no deploy.
