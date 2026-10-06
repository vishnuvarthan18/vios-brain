# Overnight run results — 2026-09-07

Branch: `dev` (not merged, not pushed). Nothing was deployed. The production
cron was not re-enabled — it is `crons = []` in `wrangler.toml` and I did not
touch it, and prod's last `job_run` is 2026-09-05T11:03, consistent with it
already being paused.

**Changes are UNCOMMITTED in the working tree.** My standing instruction is to
commit only when asked, and the brief did not ask. Inventory in §6 so you can
review the diff and commit it yourself, or tell me to.

---

## 0. First, the thing that blocks the most: four briefing documents do not exist

I was told to read these four before starting:

1. `sathyamangalam/session-close-2026-09-06-full-freeze.md`
2. `sathyamangalam/atlas-scope-locked-2026-09-06.md`
3. `sathyamangalam/sensitive-species-coordinate-policy.md`
4. `sathyamangalam/harvest-engine-dedup-freeze-and-pause-2026-09-05.md`

**None of them exists anywhere.** Verified four ways: the working tree; `git
ls-tree` over all seven local branches plus `origin/main` and `origin/dev`;
`git log --all --diff-filter=A` over every path ever added in this repo's
history; and a full-text grep for their distinguishing phrases (`dedup freeze`,
`scope lock`, `coarsen`, `full freeze`). There is no `sathyamangalam/`
directory at the repo root either — the convention here is
`docs/claude-project/sathyamangalam/`, whose newest file before tonight was
dated 2026-08-26. I have written tonight's two documents there.

I did not guess at their contents. Effect per document:

| Doc | Effect on tonight |
|---|---|
| (3) coordinate policy | **Recovered from the live system instead** — `coarsen.js`, the `trg_occurrence_coarsen_*` triggers, `sensitive-species.json`, and the sync's `SensitivityOracle` agree on one rule, and I designed and tested against that. If the written policy differs, the code is what I built to and the difference is yours to arbitrate. |
| (2) scope lock | Could not be recovered. Design choices that are plausibly scope-limited are flagged as open questions, not decided. |
| (1) session close / full freeze | Could not be recovered. I treated the brief's own prohibitions as the freeze boundary. |
| (4) dedup-freeze 9-item list | **STEP 3 is blocked on this.** See §3 — I did not invent nine items. |

---

## STEP 0 — status of the three earlier asks: all three DONE

Verified against commits *and* the live artefacts, not just commit messages.

**1. Place-matching key includes `wikidata_id` — DONE downstream, and I found the
upstream half still open.**

`e9490d8` changed `scripts/sync_d1_to_atlas.py` from matching places on
`slug OR osm_id` to `wikidata_id → osm_id → slug`, and merged the duplicate it
had already let through (atlas place 90 `sathyamangalam` deleted, place 50
`satyamangalam` kept; 93 → 92 places). The diff is real and the reasoning is in
the commit body.

But that commit explicitly said it *"does not fix harvest-engine, which should
dedupe on `wikidata_id` too."* I checked: `harvest-engine/src/lib/place.js`
`findOrCreatePlace()` still matched on `osm_id` then `slug` only. **I fixed that
upstream half tonight** — see §2.

**2. Species browser at `/life` — DONE.**
`ece82ab` added section 03 "Full Taxonomic Index" to `site/life.html`
(line 97), reading `exports/species.json` (2,664 rows), virtualised, separate
from the 76 curated cards in section 02. `exports/species.json` exists (724 KB).

**3. Tier-less documents in their own visible section — DONE.**
`ece82ab` added section 03 "The Unreviewed Harvest" to `site/record.html`
(line 148), reading a new `exports/documents_unreviewed.json`. I confirmed that
file exists and holds **exactly 1,839 rows** — the `relevance_tier IS NULL`
count. `documents.json` kept its shape deliberately, because `history.html` and
`record.html` both read it as a flat array.

Per the brief I skipped later steps these made redundant. Nothing was redundant:
STEP 2 (gazetteer) and STEP 5 (tiering) are about different things than item 3.

**Incidental, and larger than it looks:** local `dev` is **5 commits ahead** of
`origin/dev` (which sits at `178d8d1`, a commit local `dev` does not contain).
More importantly `dev` and `main` have **diverged in both directions**: 10
commits on `main` are absent from `dev` (the `feat/stream-health` merge, the lgd
field-mapping fix, coverage snapshots, the 3×-daily cron change) and 20+ on `dev`
are absent from `main` (the whole D1→atlas sync, the site work). They now hold
contradictory `job-run.js` logic — see STEP 3 (c). Not resolved: merging is a
promotion decision, and this one is not mechanical.

---

## STEP 1 — schema proposal: DELIVERED

**`docs/claude-project/sathyamangalam/schema-proposal-2026-09-07.md`** (872 lines).
All eight items designed with DDL, foreign keys, indexes, and a 13-step
migration plan. **No migration was written or run.** Waiting on your review.

Three of the brief's premises turned out to be wrong, and the corrections
changed the design:

- **`occurrence` already exists with exactly 78,467 rows.** Item 2 is not a new
  table; it is 4 missing columns and 5 missing indexes.
- **`claim` is not empty — it holds 83 rows, and already contains the brief's own
  worked example**: claim 11 (`tigers` = `8-10`, as_of 2009) and claim 10
  (`tigers` = `112`, as_of `2024-25`), each with its own `source_id`. That
  settled item 3 in favour of extending `claim`.
- **"93 places"** was atlas.db *before* the `e9490d8` merge. Live counts:
  atlas.db 92, production D1 46, staging D1 59.

Headline design decisions and the measurements behind them:

- **Categories: one `taxon` table + an M:N `taxon_category` join, not separate
  tables.** The nine categories sit at four different taxonomic ranks, and
  **"trees" is a growth habit, not a clade — every tree is also a plant**, which
  a single-valued column cannot express and separate tables would have to
  duplicate. Also 825 of 2,664 taxa have *both* `kingdom` and `class_` NULL, so
  an honest `uncategorised` bucket is required.
- **The most important finding in the proposal: `occurrence.lat/lon` means
  different things in different copies of atlas.db** depending on whether the
  sync ran with `--raw-coordinates`, and nothing in the row records which. I
  propose mirroring D1's two-pair model (`public_lat`/`public_lon` separate from
  raw `lat`/`lon`) so the site can only read the public pair. D1's `occurrence`
  already has both, plus a `place_id` that atlas.db drops — so this aligns atlas
  with the upstream model rather than inventing one.
- **Nothing in atlas.db references `place(id)` — not one table.** So
  places-as-hubs is not an n+1 optimisation problem yet; the edges do not exist.
  I propose one polymorphic `place_link` table plus a materialised
  `place_hub_count`, and deliberately keep the 78,467 occurrences *out* of the
  link table (they get a direct `place_id` instead) so hub counts cannot inflate
  by 78k.
- **`PRAGMA foreign_keys` is currently `0`** — every existing FK is declarative
  only. I ran `PRAGMA foreign_key_check`: **zero violations**, so enforcement can
  be turned on safely, and doing it before adding tables is much cheaper.
- **Photos: the corpus is 576 rows, not the whole iNaturalist harvest.** Only 576
  of 78,467 occurrences carry a `media_url`, all from the pre-harvest-engine
  import. The 791 `harvest-engine:inaturalist` rows have no media and their
  `licence` column holds a *prose paragraph* ("Varies per observation…"), so they
  cannot legally enter a photo table until per-record licence is captured
  upstream. **43 of the 576 are `cc-by-nc-nd`** — thumbnailing those is a licence
  breach, so "derivatives allowed" is a column, not a footnote.

Eight decisions I need from you are listed in §10 of that document.

---

## STEP 2 — gazetteer: measured on staging, and the target is not reachable

### The target

The brief asks for 500–2,000+ places. **That is not achievable from Overpass,
and the ceiling is upstream, not in the query.** Measured live tonight against
the reserve bbox (`11.48,76.83,11.82,77.46`):

| Query | Named features |
|---|---|
| settlements only (the query production is actually running) | **43** |
| the widened node-only query already in the repo, undeployed | **60** |
| **my further-widened `nwr` query (tonight)** | **81** |
| every named OSM element of any tag — the hard ceiling | **258** |

258 is the absolute maximum any Overpass query can return here, and most of the
gap between 81 and 258 is not gazetteer material: 33 highways/routes, 14
administrative boundaries, 19 hospitals, 8 residential landuse polygons, plus
shops, banks, a restaurant and a greengrocer. Under a defensible "named
geographic feature" definition the ceiling is ~82, and I reach 81 of them.

Two internal cross-checks agree that hundreds is the wrong order of magnitude:
`claim` 17 records **136 villages in the zone of influence** and `claim` 22
records **184 inhabited villages (1887)**.

**The route to 500–2,000 is a different SOURCE, not a wider query** — the LGD or
Census village directories. `src/streams/lgd.js` and `census.js` already exist,
and per `wrangler.toml` both are **blocked on `DATA_GOV_IN_API_KEY`, which has
not been obtained**, and `sources/lgd.json`'s `resource_id` is still unfilled.
**That is a decision only you can make** (free/instant signup at data.gov.in).

### What production is actually running — and it is not what the repo says

Production D1 holds **46 places**. Its only Overpass source row is 541
(2026-09-01). I fetched that run's archived raw blob from R2
(`raw/sathyamangalam/overpass/ec/ec0421…`) and it contains **45 elements, 42
named, all nodes, all `place=*`** — i.e. the **old settlements-only query**. The
widened query committed on 2026-08-26 has never run in production.

### Before/after on staging (the test the brief asked for)

Staging D1, `harvest-engine-staging`, applied through a faithful mirror of
`findOrCreatePlace` (including tonight's new `wikidata_id` precedence):

| Source | Query | Places | Types |
|---|---|---|---|
| 1 | original settlements-only (same `request_hash` as prod's source 541) | 42 | hamlet, village, town |
| 2 | widened node-only (repo, undeployed) | +17 | temple, farm, river, waterbody, dam, stream |
| 3 | **my `nwr` query, tonight** | **+19** | cliff, village, forest, protected_area, viewpoint, wood, temple, waterbody, river |
| | **total** | **78** | |

**59 → 78 on staging. Production untouched at 46.**

The +19 are dominated by one class of miss: the old query asked only for
`node`s, so **villages mapped as *areas* were invisible** (`way place=village`,
5 of them), along with 4 temples mapped as buildings and 5 protected-area
boundaries. A village drawn as a polygon is not a rarer village.

Diff is in `harvest-engine/src/streams/overpass.js` (query + `placeType`, so a
protected-area boundary is not filed as a village).

---

## STEP 3 — dedup-freeze: BLOCKED on the missing 9-item list

The fix list lives only in
`harvest-engine-dedup-freeze-and-pause-2026-09-05.md`, which does not exist
(§0). **I did not invent nine items, and I am not reporting pass/fail against a
list I cannot read.**

What I did instead: since "dedup freeze" names a mechanism, I audited that
mechanism empirically. Four findings, each independently verified. I am *not*
claiming these are the nine items.

**(a) The Overpass stream has been frozen since 2026-09-01, reporting success.**
Every run from 2026-09-01 to the 2026-09-05 pause — 12+ consecutive, on an
8-hourly cron — recorded `status='success', rows_written=0,
rows_skipped_duplicate=1`. Mechanism: `overpass.js` computes `request_hash` from
the query text, and `findExistingSource(db, reserveId, requestHash)` matches on
`reserve_id + request_hash` alone, **with no regard for whether that run
succeeded**. A single-shot query whose text never changes therefore
short-circuits forever after its first success, so OSM edits can never be
picked up. (I first suspected error-body poisoning; I checked, and source 541
legitimately wrote 40 places, so that was wrong — it is the static-hash freeze,
not a poisoned row.) *Incidentally, deploying the widened query would change the
hash and unfreeze it once — then it freezes again.*

**(b) Many streams have written nothing for their entire history.**
Across 920 production `job_run` rows, streams recording `success` with zero rows
written: `wikidata` 29/30 runs (**0 rows written, ever**), `gbif` 23/38 (**0 rows
ever**), `historical-text` 28, `bhl` 28, `erode-nic` 28, `wii` 28, `census` 28,
`ntca` 26, `mongabay-india` 19, `toi-coimbatore` 20. Separately, six streams have
**never once succeeded**: `gee` 0/34, `management-plan` 0/30, `shodhganga` 0/30,
`forests-tn` 0/29, `wdpa` 0/29, `openalex` 0/25.

Whether each of these is a freeze or a genuinely exhausted source needs
per-stream triage, which is what the missing 9-item list presumably contains. A
GBIF search that legitimately re-finds only known occurrences would look
identical from `job_run` alone. I am flagging the shape, not diagnosing 33
streams.

**(c) `main` and `dev` encode CONTRADICTORY policies for exactly this case.**
I initially wrote that the repo's status-deriver fix was undeployed. **That is
wrong, and the truth is more useful.** The two branches disagree:

- **`main`** (what production deploys from) has `classifyOutcome()`, which
  downgrades `success` → `partial` **only** when a run wrote nothing *and*
  skipped nothing, and *deliberately* leaves the skipped-duplicate case alone:
  *"A run that skipped duplicates did do work and is left alone."* So the frozen
  Overpass runs reporting `success` are **main behaving as designed**, not a bug.
- **`dev`** has `deriveJobRunStatus()`, which maps
  `rowsWritten=0, rowsSkippedDuplicate>0, no errors` → **`skipped_duplicate`**,
  with a test asserting it, and `withJobRun` overruling a stream's self-reported
  `success`.

Consistent with main's policy, **`skipped_duplicate` has never been recorded in
production**: the only statuses ever written are `success` 584, `failed` 249,
`running` 64, `partial` 23. The schema is ready for it — migration
`0004_job_run_skipped_status.sql` was applied to prod on 2026-08-25 — so merging
`dev` would silently change what "success" means on the dashboard for a dozen
streams.

**Decision needed:** which policy is right? I lean to dev's — a run that fetched
one cached query and wrote nothing is not a success — but `main`'s
`classifyOutcome` catches a real failure mode dev's does not (main's comment
records it: the `lgd` stream *"sat green over an empty place table … it read
`villagename` from a payload whose key is `villageNameEnglish`"*). **The two are
complementary, not alternatives, and the merge should keep both checks.**

**(d) Migration drift between repo and production.**
Production `d1_migrations` lists `0003_coverage_snapshot.sql`, which **does not
exist in the repo**. The repo has `0005_geometry_scope_annulus.sql`, which is
**not applied in production**. And two different migrations are both numbered
`0003` (`0003_geometry_scope.sql`, `0003_coverage_snapshot.sql`) — the same
collision `data/migrations/` has with its two `0001` files.

**Also: 64 `job_run` rows are stuck in `running`** (2026-08-27 … 2026-09-01) —
runs that never finished, with no timeout reaper to close them.

**What I need from you:** the 9-item list, or permission to treat (a)–(d) plus a
fresh audit as the list.

---

## STEP 4 — sensitivity matcher root cause: FIXED and verified

### Root cause

`harvest-engine/src/lib/taxon.js` decided sensitivity by exact set membership on
the lower-cased name:

```js
SENSITIVE_NAMES.has(String(scientificName).toLowerCase())
```

`sources/sensitive-species.json` lists **8 binomials**. GBIF and iNaturalist
return other shapes, none of which matched, so `is_sensitive` stayed `0`,
`trg_occurrence_coarsen_insert` never fired, and
`trg_occurrence_passthrough_insert` copied the **exact** coordinate into
`public_lat`/`public_lon`.

### The fix

New pure module **`harvest-engine/src/lib/species-name.js`**. It parses names
using the one rule that needs no list of author names: **binomial nomenclature
capitalises the genus and lower-cases every epithet, while author citations are
capitalised surnames, initials or years** — so the epithets are the run of
lower-case tokens after the genus, and the name ends at the first token that
cannot be one. That holds even for this corpus's worst fungal names
(`Cutaneotrichosporon smithiae (Middelhoven, Scorzetti, Sugita & Fell) Xin Zhan
Liu, F.Y.Bai, M.Groenew. & Boekhout` → two words). A registry entry then matches
when it is a **nomenclatural prefix** of the observed name, one-way, matching the
sync oracle's semantics.

`taxon.js` now calls it. The sync-time `SensitivityOracle` is **left in place as
defence in depth**, as instructed.

Two bugs in my own first implementation, both caught by the rule-based tests
rather than by inspection: I lower-cased each token *before* testing whether it
was lower-case (which made `Letcher` parse as an epithet), and I rejected
all-lower-case input outright. Both fixed; the caseless path is documented.

### Verified against staging

`harvest-engine/scripts/verify-trinomial-sensitivity.sql`, run on
`harvest-engine-staging` with pre-registered expectations and paired controls,
sentinel coordinates in the Indian Ocean, every row deleted afterwards:

| Name | `is_sensitive` | public coords | verdict |
|---|---|---|---|
| Elephas maximus indicus | 1 | −12.33, 98.775 | **PASS coarsened** |
| Panthera pardus fusca | 1 | −12.33, 98.775 | **PASS coarsened** |
| Panthera tigris tigris | 1 | −12.33, 98.775 | **PASS coarsened** |
| Acridotheres tristis tristis (control) | 0 | −12.345678, 98.765432 | PASS passthrough |
| Gyps bengalensis (Gmelin, 1788) | 0 | exact | passthrough — see below |
| Melursus ursinus ursinus | 0 | exact | passthrough — see below |

Staging returned to baseline afterwards: 59 places, 0 taxa, 0 occurrences, 0
sentinels, and only the 2 pre-existing source rows. Unit tests: **39/39 pass**
(the matcher test alone is 8 registry entries × 12 name shapes).

### What the fix does NOT reach — and this needs your decision

**A second, independent leak that no code change can close.** While measuring, I
found uncoarsened occurrences whose taxon is flagged `sensitive=0`, which makes
them **invisible to any audit query that filters on `sensitive=1`**:

- `taxon` 2485 `Gyps bengalensis (Gmelin, 1788)` — **3 occurrences, uncoarsened**,
  while `taxon` 30 `Gyps bengalensis` is `sensitive=1`. Same bird, authorship
  suffix.
- `taxon` 2051 `Melursus ursinus ursinus` — **2 occurrences, uncoarsened**.

Both stay unprotected after my fix, because **9 of the 20 species atlas.db flags
sensitive are absent from `sensitive-species.json` altogether**: *Crocodylus
palustris, Gyps bengalensis, Lutrogale perspicillata, Melursus ursinus, Neophron
percnopterus, Ophiophagus hannah, Prionailurus rubiginosus, Python molurus,
Tetracerus quadricornis*. The registry is meant to be the source of truth and it
is missing half the list — including three Critically Endangered vultures and the
king cobra. Adding them only ever *increases* coarsening, so it is safe, but it
changes what production redacts, so **I did not edit the registry.** A test
(`registry content gap`) documents the nine names and will fail deliberately once
you extend it.

**One thing I got wrong and corrected mid-run:** I first read the 13 uncoarsened
sensitive occurrences in atlas.db as a trinomial leak. They are not — all 13 are
`harvest-engine:*` rows and all 13 harvest-engine sensitive rows are raw, i.e.
the documented `--raw-coordinates` dev flag behaving correctly, while all 385
legacy `gbif`/`inaturalist` rows are coarsened. The trinomial correlation was a
coincidence of which taxa the newer harvest covered. The real leak is the 5 rows
above. Also worth knowing: those iNaturalist rows carry
`coord_uncertainty_m = 22000` (iNaturalist obscures at source) and the eBird ones
share a single hotspot centroid — so their ten decimal places are false
precision, not exact locations.

### Bonus, same defect class: the upstream place-duplicate fix

`findOrCreatePlace` now matches `wikidata_id → osm_id → slug`, closing the half
`e9490d8` left open. New regression test `test/place-identity.test.js` drives the
**real** `findOrCreatePlace` against a fake D1 and reproduces the exact bug:
old code → **2 rows** for Q242563 (`sathyamangalam`, `satyamangalam`); new code →
**1 row**, with the second stream's `osm_id` merged onto the survivor. Tested in
both stream orders, plus a guard that NULL `wikidata_id` never matches NULL.

**A claim I made and then disproved:** I initially said 6 of the staging dedupe
hits landed on `wikidata_id` and would have been duplicates under the old key. I
ran the paired control: **both precedences insert exactly 19** on this dataset.
The wikidata key only changed which key got credit. The duplicate is a
*cross-stream* phenomenon — within one Overpass run every element has a unique
`osm_id` — which is why the regression test simulates two streams.

---

## STEP 5 — relevance_tier NULL: rule recovered, implemented, dry-run only

### The rule was not invented — it was recovered and validated

No tier-assignment code exists anywhere in this repo, so I recovered the rule
from the 2,200 already-labelled documents that still carry `relevance_terms`
(populated for tiers A/B/C only; empty for D and NULL). It is:

> a **38-term vocabulary partitioned into three mutually exclusive bands**;
> a document's tier is the **highest-precedence band** any matched term falls in
> (A > B > C).

- **band A** (12) the reserve and places inside it — `sathyamangalam`,
  `bhavanisagar`, `moyar`, `bargur`, `hasanur`, `thengumarahada`, `talamalai`, …
- **band B** (12) adjoining protected areas and the district — `nilgiri
  biosphere`, `mudumalai`, `bandipur`, `erode district`, `biligiri`, `br hills`, …
- **band C** (14) regional context, species and communities — `tamil nadu`,
  `western ghats`, `lantana camara`, `senna spectabilis`, `soliga`, `irula`, …

**Validation: 2,200/2,200, 100%, and zero terms appear in more than one band.**
Replayed independently from `title + abstract`: **2,199/2,200**, 0 with no term
found. I checked title-only as a control — it reproduces just 1,415/2,200 — so
`title + abstract` is confirmed as the surface the original process matched on.
The single mismatch is document 604, where the rule is arguably *more* correct
than the stored label (a habitat-suitability paper filed C on `tamil nadu` whose
abstract names Sathyamangalam and Mudumalai).

### Implementation

**`scripts/tier_documents.py`.** The instrument gates itself: it re-derives the
bands, replays them against all 2,200 labelled rows, and **refuses to run** if it
cannot reproduce every stored label, or if any term is ambiguous across bands, or
if the text replay drops below 95%. Word-boundary guards keep `bhavani` from
firing inside `bhavanisagar` and `irula` from firing inside `irular` — real
separate terms in different bands.

### Result on the 1,839 untiered documents (dry run)

| | n | share |
|---|---:|---:|
| A | 34 | 1.8% |
| B | 11 | 0.6% |
| C | 370 | 20.1% |
| **unclassified** | **1,424** | **77.4%** |

An honest limitation: **only 718 of the 1,839 have an abstract at all** (39%,
against 83% for tier A), so the rule has far less text to work with on exactly
the corpus it is applied to. That accounts for much of the 77%.

### What I deliberately did NOT do — this is your call

**The script never writes `relevance_tier`, and I ran it dry-run only.** Tiers A
and B are the filter feeding `documents.json` — the "safe to cite" bibliography
`record.html` and `history.html` read. Writing these results would promote **45
never-human-reviewed documents into the bibliography on a term match**, and
`ece82ab` already documented that this corpus is visibly full of false positives
(customer satisfaction at a trading firm in Gobichettipalayam, a spinning mill
named after Bannari Amman). That is an editorial decision, not a scripted one.

So the first pass is designed to land in a **separate** column
(`relevance_tier_auto`, written only under `--apply-auto`, which I did not run),
leaving `relevance_tier` NULL until you rule. The "unclassified" requirement —
never invisible — is already satisfied by `documents_unreviewed.json` and
`/record` section 03 from `ece82ab`, which surface all 1,839 today; splitting
that section by `relevance_tier_auto` is a small export change once you decide.

**Decision needed:** do auto-tiers feed the bibliography, or only a review queue?
My recommendation is a review queue.

---

## 6. Change inventory (uncommitted, on `dev`)

Modified:

| File | Change |
|---|---|
| `harvest-engine/src/lib/taxon.js` | uses the new matcher; exports `checkSensitive()` |
| `harvest-engine/src/lib/place.js` | identity precedence `wikidata_id → osm_id → slug` |
| `harvest-engine/src/streams/overpass.js` | widened `nwr` query (60 → 81 named); `placeType` extended |
| `harvest-engine/package.json` | `test` script fixed — `node --test test/` fails on Node 24, so **the suite has been unrunnable via `npm test`**; verified pre-existing by removing my own test file. Now `node --test test/*.test.js`, 39/39 pass. |

New:

| File | Purpose |
|---|---|
| `harvest-engine/src/lib/species-name.js` | name parsing + sensitive matcher (the STEP 4 root-cause fix) |
| `harvest-engine/test/species-name.test.js` | 9 rule-based tests |
| `harvest-engine/test/place-identity.test.js` | 5 tests reproducing the `e9490d8` duplicate |
| `harvest-engine/scripts/verify-trinomial-sensitivity.sql` | staging end-to-end coarsening proof |
| `scripts/tier_documents.py` | self-gating document tierer (dry-run default) |
| `docs/.../schema-proposal-2026-09-07.md` | STEP 1 deliverable |
| `docs/.../overnight-run-results-2026-09-07.md` | this file |

**Databases:** `data/atlas.db` **unmodified** (verify with `git status`).
`harvest-engine-staging` D1 gained 19 places + 1 source row (STEP 2's before/after,
59 → 78) and was returned to baseline after the STEP 4 sentinel test.
**Production D1 was read-only throughout** — the only writes anywhere were to
staging.

---

## 7. What needs your decision

Ranked by what blocks most.

1. **Do the four missing documents exist somewhere?** (§0) Blocks STEP 3 entirely
   and puts §2a/§5 of the schema proposal on reconstructed rather than stated
   policy.
2. **Extend `sensitive-species.json` with the 9 missing species?** (STEP 4) Three
   Critically Endangered vultures and the king cobra are currently unprotected
   upstream, and no code fix reaches them. Safe in one direction only — it can
   only increase coarsening.
3. **The 9-item dedup-freeze list**, or permission to treat my (a)–(d) as the list.
3b. **Which `job-run.js` status policy survives the `dev`/`main` merge?** They
   currently contradict each other and each catches a failure the other misses;
   my recommendation is to keep both checks rather than pick one.
4. **`DATA_GOV_IN_API_KEY`** (STEP 2) — the only route to hundreds of places;
   free/instant signup. Overpass cannot get there.
5. **Do auto-tiers feed the bibliography or a review queue?** (STEP 5)
   Recommendation: review queue.
6. **The eight schema questions** in §10 of the proposal — most load-bearing are
   the `occurrence.lat/lon` two-pair split, and whether `contribution` belongs in
   a git-committed database (recommendation: no).
7. **Commit tonight's diff on `dev`?** And separately, `dev` is 5 commits ahead of
   `origin/dev`, which holds one commit local `dev` lacks.
