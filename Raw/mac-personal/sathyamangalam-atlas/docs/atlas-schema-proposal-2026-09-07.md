# atlas.db schema proposal — 2026-09-07

**Status: PROPOSAL. No migration has been applied. No DDL in this document has been run
against any database.** Nothing was deployed; `dev` was not merged into `main`; the
production cron was not touched.

Author: overnight run, 2026-09-07. Written against `data/atlas.db` as committed at
`ece82ab` (dev), 36 MB, 12 tables.

---

## 0. Read this before the schema: four briefing documents do not exist

The run was instructed to read, in order:

1. `sathyamangalam/session-close-2026-09-06-full-freeze.md`
2. `sathyamangalam/atlas-scope-locked-2026-09-06.md`
3. `sathyamangalam/sensitive-species-coordinate-policy.md`
4. `sathyamangalam/harvest-engine-dedup-freeze-and-pause-2026-09-05.md`

**None of the four exists.** Verified by: working tree search; `git ls-tree` across all
seven local branches plus `origin/main` and `origin/dev`; `git log --all --diff-filter=A`
over every path ever added in history; and full-text grep for their distinguishing
phrases (`dedup freeze`, `scope lock`, `coarsen`). There is also no `sathyamangalam/`
directory at the repository root — the established location for documents of this kind is
`docs/claude-project/sathyamangalam/`, where the newest file is dated 2026-08-26.

This matters differently per document, and I did not paper over it:

- **The coordinate policy (3)** I was able to reconstruct from the live system rather than
  guess: `harvest-engine/src/lib/coarsen.js`, the `trg_occurrence_coarsen_*` triggers in
  `harvest-engine/migrations/0001_init.sql`, `harvest-engine/sources/sensitive-species.json`,
  and the `SensitivityOracle` in `scripts/sync_d1_to_atlas.py`. The rule those four agree
  on is stated in §2 and §5 and is what this proposal is designed against. If the written
  policy differs from the code, **the code is what I designed to, and the difference is
  yours to arbitrate.**
- **The scope lock (2)** I could not reconstruct. Where a design choice below is plausibly
  scope-limited, it is flagged as an open question rather than decided.
- **The dedup-freeze fix list (4)** is a 9-item list that exists nowhere. I did not invent
  nine items. See the run report for what I did instead.

**Everything in §1–§8 below is grounded in measurements against the live database, quoted
inline. Where a figure in the brief disagreed with the database, the database is quoted and
the disagreement is flagged.**

---

## 0.1 Three premises in the brief that the database contradicts

| Brief says | Database says | Consequence |
|---|---|---|
| "an occurrences table making the 78,467 existing rows individually browsable" | `occurrence` **already exists** with **exactly 78,467** rows | Item 2 is not a new table. It is four missing columns and five missing indexes. §2. |
| "extend the existing **empty** claim table" | `claim` holds **83 rows** — and already contains the brief's own worked example | Strengthens the case for extending `claim`. It is not a blank canvas; it is a working store. §3. |
| "current is 93" places | atlas.db **92**, production D1 **46**, staging D1 **59** | 93 was atlas.db *before* the duplicate merge in `e9490d8` (93 → 92). §2 of the run report. |

The claim-table example is worth quoting, because the brief proposed
"tigers 8-10 in 2009 -> 112 in 2024" as the thing a *new* table should support:

```
id  subject_type  subject_key     field   value  unit         as_of    source_id
10  reserve       sathyamangalam  tigers  112    individuals  2024-25  2
11  reserve       sathyamangalam  tigers  8-10   individuals  2009     4
```

That series is already in `claim`, with per-point sources. This is decisive for item 3.

---

## 1. Species by category — one table, a category column, and an M:N join

### Decision

**Keep the single `taxon` table. Do not create per-category tables.** Add a controlled
`category` vocabulary table and a **many-to-many** `taxon_category` join, plus one
`is_primary` category per taxon for display.

### Why not separate tables

The nine requested categories — birds, mammals, insects, snakes/reptiles, trees, plants,
fungi, fish, amphibians — are **not a partition of anything**. Measured against the live
`taxon` table:

- They sit at **four different taxonomic ranks**: `Aves`/`Mammalia`/`Insecta`/`Amphibia`
  are classes; `Squamata` (27 rows) and `Testudines` (2) are orders; `Plantae`/`Fungi` are
  kingdoms; "fish" is not a clade at all (`Actinopterygii` — 1 row here).
- **"trees" is not taxonomic.** It is a growth habit spanning `Magnoliopsida` (272 rows)
  and `Liliopsida` (33). **Every tree is also a plant.** A single-valued category column
  cannot say that, and separate `tree` and `plant` tables would have to store
  *Santalum album* twice. This alone rules out both a plain enum and separate tables.
- **31% of the corpus cannot be categorised at all today:** 825 of 2,664 taxa have *both*
  `kingdom` and `class_` NULL; 899 have no `kingdom`; 771 have no `rank`.
- **The existing columns are dirty.** `class_` sometimes holds a kingdom (13 rows have
  `class_='Plantae'`), and `rank` is both case-inconsistent (`SPECIES` 1353 / `species`
  115; `SUBSPECIES` 8 / `subspecies` 11) and **wrong in the highest-risk rows**:
  `Panthera tigris` (a binomial) is stored as `rank='SUBSPECIES'` while
  `Panthera tigris tigris` (a trinomial) is stored as `rank='species'`.

Separate tables would additionally break three things that work: the global
`UNIQUE(scientific_name)` dedupe, `occurrence.taxon_id` (78,467 rows pointing at one
table), and every future FK in §4–§8, each of which would need a polymorphic nine-way
target.

### DDL

```sql
CREATE TABLE category (
  slug          TEXT PRIMARY KEY,
  label_en      TEXT NOT NULL,
  label_ta      TEXT,
  sort_order    INTEGER NOT NULL,
  -- 1 for categories that are a growth habit or folk grouping rather than a
  -- clade ('trees', 'fish'). The site must not present these as taxonomy.
  is_functional INTEGER NOT NULL DEFAULT 0,
  -- The stated, auditable basis for membership, e.g. "class_ = 'Aves'".
  definition    TEXT NOT NULL
);

CREATE TABLE taxon_category (
  taxon_id    INTEGER NOT NULL REFERENCES taxon(id) ON DELETE CASCADE,
  category    TEXT    NOT NULL REFERENCES category(slug),
  -- Exactly one row per taxon may carry is_primary=1 (enforced below). This is
  -- the bucket the taxon appears under when it must appear exactly once.
  is_primary  INTEGER NOT NULL DEFAULT 0,
  -- 'class_=Aves' | 'kingdom=Plantae' | 'gbif-backbone' | 'curated' | 'habit-list'
  basis       TEXT    NOT NULL,
  confidence  REAL    NOT NULL DEFAULT 1.0,
  assigned_at TEXT    NOT NULL,
  source_id   INTEGER REFERENCES source(id),
  PRIMARY KEY (taxon_id, category)
);

CREATE INDEX idx_taxcat_category ON taxon_category(category, taxon_id);
-- One primary category per taxon. SQLite supports partial indexes.
CREATE UNIQUE INDEX idx_taxcat_one_primary ON taxon_category(taxon_id) WHERE is_primary = 1;
```

Seed rows carry their own definition, so the rule is data and stays auditable:

| slug | is_functional | definition |
|---|---|---|
| `birds` | 0 | `class_='Aves'` |
| `mammals` | 0 | `class_='Mammalia'` |
| `insects` | 0 | `class_='Insecta'` |
| `reptiles` | 0 | `class_ IN ('Squamata','Testudines','Crocodylia','Reptilia')` |
| `amphibians` | 0 | `class_='Amphibia'` |
| `fish` | 1 | `class_ IN ('Actinopterygii','Chondrichthyes','Teleostei','Actinistia')` |
| `plants` | 0 | `kingdom='Plantae' OR class_ IN ('Magnoliopsida','Liliopsida',…)` |
| `trees` | 1 | curated habit list, **subset of `plants`** |
| `fungi` | 0 | `kingdom='Fungi'` |
| `uncategorised` | 1 | matched no rule — **visible, never hidden** |

`uncategorised` is deliberately a real category with a real page, not a NULL. On today's
data it would hold on the order of 800+ taxa, and per the standing rule that a gap is
disclosed rather than concealed, that number belongs on the page.

### Companion change this depends on (recommended, not optional)

Categories are only as good as the names they key off. **539 of 2,664 `scientific_name`
values embed authorship** (`'Acridotheres tristis (Linnaeus, 1766)'`), and word-count
runs to 20 words. `ece82ab` already documented that ~420 taxa are the same organism twice,
bare and authored, and that 411 of the 771 taxa the sync inserted are such duplicates.

Add to `taxon`:

```sql
ALTER TABLE taxon ADD COLUMN canonical_name TEXT;   -- authorship stripped
ALTER TABLE taxon ADD COLUMN name_words     INTEGER; -- 1=uninomial 2=binomial 3=trinomial
ALTER TABLE taxon ADD COLUMN binomial       TEXT;   -- first two words of canonical_name
CREATE INDEX idx_taxon_canonical ON taxon(canonical_name);
CREATE INDEX idx_taxon_binomial  ON taxon(binomial);
```

`binomial` is not cosmetic — §5 of the run report uses it to close the sensitivity-matcher
bug at its root, and it is what lets a trinomial inherit its species' category and
sensitivity by index lookup instead of a `LIKE` scan.

**Open question for you:** merging the ~411 authored duplicates onto their canonical twins
is a *data* decision with the same shape as the place merge in `e9490d8`, and it is not
proposed here. `UNIQUE(scientific_name)` currently permits both spellings; `canonical_name`
is deliberately **not** made unique, so the duplicates stay visible and countable until you
rule on them.

---

## 2. Occurrences — the table exists; it is missing a place, a usable date, and a public coordinate

`occurrence` holds **78,467** rows, all with `taxon_id` and coordinates, 78,420 with a
date. The brief's four browse axes are in three different states:

| Axis | State |
|---|---|
| species | **works** — `taxon_id` FK + `idx_occ_taxon` |
| source | **works** — `source_id` FK, plus `dataset` (`gbif` 74,180 / `harvest-engine:gbif` 2,573 / `harvest-engine:inaturalist` 791 / `inaturalist` 720 / `harvest-engine:ebird` 203) |
| date | **stored but not browsable** — mixed formats, no index |
| **place** | **does not exist** — there is no `place_id`, and see §6 |
| coordinates | **stored ambiguously** — see the hazard below |

### 2a. The coordinate hazard (the most important finding in this section)

D1 stores **two** coordinate pairs per occurrence: raw `lat`/`lon` and the public
`public_lat`/`public_lon` written by `trg_occurrence_coarsen_insert`. **atlas.db stores only
one pair**, and which one it is depends on a command-line flag:

- default sync → `lat`/`lon` hold D1's **coarsened** `public_lat`/`public_lon`
- `--raw-coordinates` (added in `345e078`, for dev/staging) → they hold the **raw** values

So `occurrence.lat` means different things in different copies of the file, and nothing in
the row records which. Measured on this dev copy: of 398 occurrences of sensitive taxa,
**385 are coarsened to the 0.045° grid and 13 are not** — and the 13 are exactly the
`harvest-engine:*` rows (10 iNaturalist, 3 eBird), i.e. the rows this dev database
deliberately holds raw. That is correct *for dev*. It is one mistaken flag away from being
a production coordinate leak, and no constraint would catch it.

**Proposal: mirror D1's two-pair model so the public value is a different column from the
private one, and the site can only read the public one.**

```sql
ALTER TABLE occurrence ADD COLUMN public_lat  REAL;
ALTER TABLE occurrence ADD COLUMN public_lon  REAL;
-- 'exact' | 'coarsened-0.045deg' | 'source-obscured' | 'withheld'
ALTER TABLE occurrence ADD COLUMN public_coord_basis TEXT;
-- Which rule produced public_*: 'd1-trigger' | 'sync-fallback' | 'passthrough'
ALTER TABLE occurrence ADD COLUMN coarsened_by TEXT;
```

Every export and every page reads `public_lat`/`public_lon` and never `lat`/`lon`. The
existing `public_precision_deg` (0.0 on 78,082 rows, 0.045 on 385) stays as the grid size.

Two facts worth recording in `public_coord_basis` rather than losing: the 10
`harvest-engine:inaturalist` sensitive rows carry `coord_uncertainty_m = 22000`, i.e.
iNaturalist has **already** obscured them at source to ~22 km, and their ten-decimal
`lat`/`lon` is a random point in that box, not a precise location. The three
`harvest-engine:ebird` sensitive rows all share one identical coordinate
(`11.984685, 77.2527211`) — a hotspot centroid, generalised at source. Ten decimal places
on a 22 km uncertainty is exactly the kind of false precision a `public_coord_basis` value
prevents someone from misreading later.

### 2b. Place, date, indexes

```sql
ALTER TABLE occurrence ADD COLUMN place_id INTEGER REFERENCES place(id);
-- 'boundary-contains' | 'nearest-within-5km' | 'bbox' | 'curated'; NULL place_id
-- is a legitimate outcome and must stay legitimate — do not default it.
ALTER TABLE occurrence ADD COLUMN place_basis      TEXT;
ALTER TABLE occurrence ADD COLUMN place_distance_m INTEGER;

-- event_date is mixed: '1884-01-01T00:00' and '2026-08-26 17:59' both occur,
-- across 44 distinct years. Normalise rather than parse at query time.
ALTER TABLE occurrence ADD COLUMN event_date_norm TEXT;    -- 'YYYY-MM-DD'
ALTER TABLE occurrence ADD COLUMN event_year      INTEGER;

CREATE INDEX idx_occ_place        ON occurrence(place_id);
CREATE INDEX idx_occ_year         ON occurrence(event_year);
CREATE INDEX idx_occ_taxon_year   ON occurrence(taxon_id, event_year);
CREATE INDEX idx_occ_dataset      ON occurrence(dataset);
CREATE INDEX idx_occ_place_taxon  ON occurrence(place_id, taxon_id);
```

`idx_occ_taxon_year` and `idx_occ_place_taxon` are the two composites that keep a species
page and a place page from scanning 78k rows.

**Open question for you:** assigning a `place_id` to an occurrence of a *sensitive* taxon
re-identifies it. A coarsened coordinate that still says "Thengumarahada" has leaked the
location by name. My recommendation is that sensitive occurrences get `place_id` at
range/division granularity only, never village — but that is a policy call, it is exactly
what the missing coordinate-policy document would have settled, and I have not decided it.

---

## 3. Population history — extend `claim`; do not add `population_history`

### Decision

**Extend `claim`.** Four additive columns and two indexes, no new table.

### Why

The brief asked for a table "tracking a number over time per species with a source per
data point (e.g. tigers 8-10 in 2009 -> 112 in 2024)". `claim` **already does this**, and
the brief's example is literally claim rows 10 and 11 (quoted in §0.1). `claim` also
already has what a defensible time series needs and a fresh table would have to re-grow:
`as_of`, per-point `source_id`, `confidence`, `unit`, and a supersede chain
(`status`, `superseded_by_claim_id`, `idx_claim_status`).

A `population_history` table would duplicate all of that, create two places to write the
same fact, need a conflict rule between them, and require a second export consumer
alongside the `claims.json` the site already reads. The premise that `claim` is empty was
the main argument for a new table, and it is false — 83 rows, 23 of them the reserve's own
`reserve/sathyamangalam` metrics.

### What `claim` genuinely lacks

`claim` stores a *quoted assertion*, not a *number*. Four gaps, all measured:

1. **`value` is TEXT holding ranges.** Real values include `8-10`, `~20`, `350-450`,
   `800–900` (en-dash, not hyphen), and `Thengumarahada; Talamalai (pt); …`. Nothing can
   plot that.
2. **`as_of` is TEXT in three shapes** — `1887`, `2013`, `2024-25` (4 rows). Sorting is
   accidental and `2024-25` has no defined start.
3. **`subject_key` does not resolve.** Measured: `subject_type='place'` resolves 15/15
   against `place.slug`; **`'taxon'` resolves 0 of 37** (it holds scientific names —
   `'Panthera tigris tigris'` — not slugs); **`'reserve'` resolves 0 of 23**;
   `'range'` 2 of 8. So a species page cannot find its own population claims today, and
   the reserve's 23 metrics are not reachable from `place` 88.
4. **No method field.** Comparing `8-10` (2009) with `112` (2024-25) across unstated
   methods is the classic population-trend trap. The schema should force the method to be
   stated next to the number.

### DDL

```sql
-- Parsed numerics. `value` stays untouched and authoritative as the quoted string.
ALTER TABLE claim ADD COLUMN value_min   REAL;
ALTER TABLE claim ADD COLUMN value_max   REAL;   -- = value_min for a point estimate
ALTER TABLE claim ADD COLUMN value_is_approx INTEGER NOT NULL DEFAULT 0;  -- '~20' -> 1

-- Normalised time. Both ISO-8601 dates; a bare year becomes YYYY-01-01..YYYY-12-31.
ALTER TABLE claim ADD COLUMN as_of_start TEXT;
ALTER TABLE claim ADD COLUMN as_of_end   TEXT;

-- Resolved subjects, kept ALONGSIDE subject_key (which stays as provenance).
ALTER TABLE claim ADD COLUMN subject_taxon_id INTEGER REFERENCES taxon(id);
ALTER TABLE claim ADD COLUMN subject_place_id INTEGER REFERENCES place(id);

-- How the number was produced: 'camera-trap-SECR' | 'dung-count' | 'line-transect'
-- | 'waterhole-census' | 'expert-estimate' | 'unstated'. NOT NULL so it cannot be
-- silently omitted; 'unstated' is an honest value, a blank is not.
ALTER TABLE claim ADD COLUMN method TEXT NOT NULL DEFAULT 'unstated';
-- Groups points into one comparable series.
ALTER TABLE claim ADD COLUMN series_key TEXT;

CREATE INDEX idx_claim_series ON claim(series_key, as_of_start);
CREATE INDEX idx_claim_taxon  ON claim(subject_taxon_id, field, as_of_start);
CREATE INDEX idx_claim_place  ON claim(subject_place_id, field, as_of_start);
```

**Open questions for you — I have not decided these:**

- **`2024-25` → what dates?** Indian financial year (2024-04-01 … 2025-03-31) is the
  likely intent for a forest-department figure, but "the 2024-25 tiger estimate" may
  denote a survey window instead. Four rows depend on it. Editorial call.
- **The 6 `taxon`/`population` claims have `source_id = 1` and only one has an `as_of`**
  (`Panthera tigris tigris`, `2024-25`). Five undated population numbers cannot join a
  time series. Whether to date them from their source or drop them from the series is
  yours.
- **`reserve` vs `place`.** `subject_type='reserve'`, `subject_key='sathyamangalam'` is not
  a `place.slug` — and note `e9490d8` *deleted* the row whose slug was `sathyamangalam`.
  The reserve is `place` 88 (`sathyamangalam-wls-tiger-reserve`). Pointing those 23 claims
  at place 88 is a one-line backfill but it is an assertion about what they describe, so it
  needs your sign-off, not mine.

---

## 4. Relationships — one table, a predicate vocabulary, inverses derived not stored

### Decision

**One `relationship` table** covering both species↔species and species↔habitat, with a
controlled predicate vocabulary and a new `habitat` table. Store each edge **once**, in a
canonical direction, and derive the inverse.

### Why one table and one direction

Two tables (`trophic` + `habitat_association`) would duplicate the provenance columns every
assertion needs and force the UI to query both for "everything about this species". The
predicate vocabulary is the thing that varies, and a vocabulary belongs in rows.

Storing both `A eats B` and `B eaten-by A` is the real trap: the two can drift out of sync,
and nothing in SQLite would notice. Store `eats` only; get `eaten-by` from
`predicate.inverse_slug` at read time.

Every edge here is an **assertion**, not an observation — "leopard eats chital in this
reserve" is a claim someone made in a document. So `source_id` is `NOT NULL` and
`confidence` is mandatory. Without that this table becomes folklore with a schema.

### DDL

```sql
CREATE TABLE habitat (
  id         INTEGER PRIMARY KEY,
  slug       TEXT UNIQUE NOT NULL,
  name_en    TEXT NOT NULL,
  name_ta    TEXT,
  -- 'southern-dry-deciduous', 'riparian', 'scrub', 'montane-sholas', 'grassland'
  kind       TEXT,
  description TEXT,
  source_id  INTEGER REFERENCES source(id)
);

CREATE TABLE relationship_predicate (
  slug         TEXT PRIMARY KEY,      -- 'eats', 'pollinates', 'occurs_in', 'host_of'
  label_en     TEXT NOT NULL,
  inverse_slug TEXT REFERENCES relationship_predicate(slug),  -- 'eaten_by'
  -- 'taxon-taxon' | 'taxon-habitat' — constrains which object column is legal
  object_kind  TEXT NOT NULL,
  symmetric    INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE relationship (
  id                INTEGER PRIMARY KEY,
  subject_taxon_id  INTEGER NOT NULL REFERENCES taxon(id) ON DELETE CASCADE,
  predicate         TEXT    NOT NULL REFERENCES relationship_predicate(slug),
  object_taxon_id   INTEGER REFERENCES taxon(id)   ON DELETE CASCADE,
  object_habitat_id INTEGER REFERENCES habitat(id) ON DELETE CASCADE,
  -- Exactly one object. XOR, enforced.
  CHECK ((object_taxon_id IS NOT NULL) + (object_habitat_id IS NOT NULL) = 1),
  -- No self-predation rows.
  CHECK (object_taxon_id IS NULL OR object_taxon_id <> subject_taxon_id),
  confidence   REAL    NOT NULL DEFAULT 1.0,
  -- 'local-observation' | 'literature-general' | 'inferred-from-diet-study'
  evidence     TEXT    NOT NULL,
  -- TRUE only if the assertion was made about THIS reserve, not the species globally.
  local_to_reserve INTEGER NOT NULL DEFAULT 0,
  document_id  INTEGER REFERENCES document(id),
  source_id    INTEGER NOT NULL REFERENCES source(id),
  notes        TEXT,
  created_at   TEXT    NOT NULL,
  UNIQUE (subject_taxon_id, predicate, object_taxon_id, object_habitat_id, source_id)
);

CREATE INDEX idx_rel_subject ON relationship(subject_taxon_id, predicate);
CREATE INDEX idx_rel_obj_tax ON relationship(object_taxon_id, predicate);
CREATE INDEX idx_rel_obj_hab ON relationship(object_habitat_id, predicate);
```

`idx_rel_obj_tax` is what makes the derived inverse cheap: "what eats the chital" is an
index seek on `(object_taxon_id, predicate)`, not a scan.

**Note on scope:** `local_to_reserve` exists because most trophic facts available for these
species are global natural-history statements, not Sathyamangalam observations. Presenting
the two identically would overclaim. Whether the site shows global relationships at all is
plausibly a scope-lock question, and the missing document (2) may already answer it.

---

## 5. Photos — the corpus is 576 rows, not the whole iNaturalist harvest

### Measured starting point

The brief refers to "already-harvested iNaturalist CC-licensed images". What exists:

- **576 of 78,467 occurrences carry a `media_url`.** All 576 are `dataset='inaturalist'`
  (the pre-harvest-engine import).
- **The 791 `harvest-engine:inaturalist` rows have no media at all**, and their `licence`
  column holds a **prose paragraph**, not a code: *"Varies per observation (CC0/CC-BY/
  CC-BY-NC/all-rights-reserved) — see each observation's own `license_code`…"*. Per-record
  licence was never captured for them.
- Licences on the 576 that do have media: `cc-by` 356, `cc-by-nc` 173, **`cc-by-nc-nd` 43**,
  `cc-by-nc-sa` 4.

Two consequences the schema must encode rather than smooth over. **43 images are `-nd`
(no derivatives)** — generating a cropped thumbnail of those is a licence breach, so
"derivatives allowed" has to be a queryable column, not a footnote. And the 791
harvest-engine rows **cannot legally enter a photo table at all** until per-record licence
is captured upstream, because "varies" is not a licence.

### DDL

```sql
CREATE TABLE photo (
  id            INTEGER PRIMARY KEY,
  taxon_id      INTEGER REFERENCES taxon(id),
  occurrence_id INTEGER REFERENCES occurrence(id) ON DELETE SET NULL,
  place_id      INTEGER REFERENCES place(id),
  url           TEXT NOT NULL,
  thumb_url     TEXT,
  width         INTEGER,
  height        INTEGER,

  -- Normalised code ONLY. 'CC0-1.0','CC-BY-4.0','CC-BY-NC-4.0','CC-BY-NC-ND-4.0',
  -- 'CC-BY-NC-SA-4.0'. NOT NULL and CHECKed: a row whose licence is unknown must
  -- not exist, because its existence here is what authorises display.
  licence_code  TEXT NOT NULL CHECK (licence_code IN (
                  'CC0-1.0','CC-BY-4.0','CC-BY-SA-4.0','CC-BY-NC-4.0',
                  'CC-BY-NC-SA-4.0','CC-BY-NC-ND-4.0')),
  -- Required by every CC-BY variant. NOT NULL for the same reason.
  attribution_text TEXT NOT NULL,
  observer       TEXT,
  observer_url   TEXT,
  licence_url    TEXT NOT NULL,
  -- Derived from licence_code at write time so the renderer never re-parses a
  -- licence string. commercial_ok=0 for -NC; derivative_ok=0 for -ND.
  commercial_ok  INTEGER NOT NULL,
  derivative_ok  INTEGER NOT NULL,
  share_alike    INTEGER NOT NULL DEFAULT 0,

  external_id    TEXT,
  dataset        TEXT,
  licence_verified_at TEXT,
  source_id      INTEGER REFERENCES source(id),
  created_at     TEXT NOT NULL,
  UNIQUE (url)
);

CREATE INDEX idx_photo_taxon ON photo(taxon_id, derivative_ok);
CREATE INDEX idx_photo_place ON photo(place_id);
CREATE INDEX idx_photo_occ   ON photo(occurrence_id);
```

`idx_photo_taxon(taxon_id, derivative_ok)` is deliberate: the commonest query is "a
thumbnailable photo for this species", and `derivative_ok` in the index answers it without
a row visit.

**Backfill scope: 576 rows**, of which 533 are thumbnailable (`cc-by` + `cc-by-nc` +
`cc-by-nc-sa`) and 43 are display-only.

**Blocked, needs an upstream change not a schema change:** capturing
iNaturalist's per-observation `license_code` in the harvest-engine stream so the 791 rows
become usable. That is a `harvest-engine/src/streams/inaturalist.js` change, out of
tonight's no-deploy scope.

---

## 6. Places as hubs — the links do not exist yet, so n+1 is not the first problem

### The finding that reframes this item

**Nothing in the database references `place(id)`. Not one table.** Verified with
`PRAGMA foreign_key_list` on all 12 tables and a scan of every `CREATE TABLE` for
`REFERENCES place`: the result is empty. `place` has one FK of its own, outward to
`source`. It is an island.

The only path into a place today is the soft string join `claim.subject_key = place.slug`,
which resolves 15/15 for `subject_type='place'` and 0/23 for `'reserve'`.

So the brief's concern — "design the join structure avoiding n+1 explosions" — is real but
premature by one step. There is no join structure to optimise. Every edge has to be created
first, and the design question is what shape to create.

### Decision

**One polymorphic `place_link` table, plus a materialised `place_hub_count` summary.**
Not seven FK columns and not seven link tables.

Seven separate link tables would mean seven queries per hub page, seven migrations every
time a record type is added, and seven code paths. One link table with a covering index
gives one query per page. The cost is that `target_id` cannot be a real FK — accepted
deliberately, and mitigated below. This is also **the idiom the codebase already uses**:
`claim.subject_type`/`subject_key` is the same soft-polymorphic pattern, so this reuses a
convention rather than introducing a second one.

### DDL

```sql
CREATE TABLE place_link (
  place_id     INTEGER NOT NULL REFERENCES place(id) ON DELETE CASCADE,
  target_table TEXT    NOT NULL CHECK (target_table IN (
                 'document','claim','occurrence','news_event','legal_instrument',
                 'historical_passage','observation_layer','photo','contribution')),
  target_id    INTEGER NOT NULL,
  -- 'mentions' | 'about' | 'located_in' | 'jurisdiction_over' | 'observed_at'
  relation     TEXT    NOT NULL DEFAULT 'mentions',
  -- 'term-match' | 'coordinate-containment' | 'wikidata' | 'curated'
  basis        TEXT    NOT NULL,
  confidence   REAL    NOT NULL DEFAULT 1.0,
  source_id    INTEGER REFERENCES source(id),
  created_at   TEXT    NOT NULL,
  PRIMARY KEY (place_id, target_table, target_id, relation)
);

-- The reverse direction: "which places does this document belong to". Without
-- this index that question is a full scan, and it is the one a record page asks.
CREATE INDEX idx_place_link_target ON place_link(target_table, target_id);
CREATE INDEX idx_place_link_hub    ON place_link(place_id, target_table, relation);
```

### How n+1 is actually avoided

Two distinct pages, two distinct patterns:

**The place index (92 rows, each showing counts).** Naively this is 92 places × 9 target
types = 828 count queries. Instead, a summary table refreshed by
`scripts/export_from_db.py` in one pass:

```sql
CREATE TABLE place_hub_count (
  place_id     INTEGER NOT NULL REFERENCES place(id) ON DELETE CASCADE,
  target_table TEXT    NOT NULL,
  relation     TEXT    NOT NULL,
  n            INTEGER NOT NULL,
  computed_at  TEXT    NOT NULL,
  PRIMARY KEY (place_id, target_table, relation)
);
```

built by a single `INSERT … SELECT place_id, target_table, relation, COUNT(*) … GROUP BY 1,2,3`.
The index page then reads one table, and the static export means the site does no counting
at all.

**A single place page.** One query with a `UNION ALL` branch per section, each branch
`LIMIT`ed, so the page cost is bounded by what it displays rather than by what the place
has:

```sql
SELECT 'document' AS kind, d.id, d.title, d.year, pl.relation, pl.confidence
  FROM place_link pl JOIN document d ON d.id = pl.target_id
 WHERE pl.place_id = ?1 AND pl.target_table = 'document'
 ORDER BY d.year DESC LIMIT 25
UNION ALL
SELECT 'claim', c.id, c.field || ' = ' || c.value, CAST(strftime('%Y', c.as_of_start) AS INTEGER), pl.relation, pl.confidence
  FROM place_link pl JOIN claim c ON c.id = pl.target_id
 WHERE pl.place_id = ?1 AND pl.target_table = 'claim' LIMIT 25
-- … one branch per target_table
```

Occurrences are the exception and should **not** go through `place_link`: at 78,467 rows
they would double the link table for no gain. They get the direct
`occurrence.place_id` column from §2b and `idx_occ_place`, and the place page reads them
as an aggregate (`GROUP BY taxon_id`) plus a `LIMIT`ed sample, never a full list.

### The trade-off, stated plainly

`target_id` has no referential integrity. Deleting a document leaves a dangling
`place_link` row, and SQLite will not stop it. Mitigations, in order of preference:

1. `PRAGMA foreign_keys = ON` for `place_id` at least — see §9, this is currently **off**.
2. An orphan-detection view run in the export, failing the build if it returns rows:
   ```sql
   CREATE VIEW v_place_link_orphans AS
     SELECT * FROM place_link pl WHERE pl.target_table='document'
       AND NOT EXISTS (SELECT 1 FROM document d WHERE d.id = pl.target_id)
     UNION ALL /* … one branch per target_table … */;
   ```
3. `ON DELETE` triggers per target table if deletions ever become routine (they are not
   today — the only deletion in recent history is the manual place merge in `e9490d8`).

---

## 7. Completeness — one polymorphic table, and a hard split between *computed* and *verified*

### Decision

One `completeness` table keyed `(target_table, target_id, profile)`, plus a
`completeness_profile` table that defines which fields count. Consistent with §6's
polymorphic idiom.

### The distinction the brief conflates

"Last verified date" and "missing-field flags" are **not the same kind of fact**. A missing
field can be computed by a script in milliseconds. "Verified" means a person looked. If one
column holds both, a nightly recompute will silently overwrite human verification with a
machine timestamp, and the field becomes worthless. They are separate columns here, and
`last_verified_at` is never written by the recompute job.

### DDL

```sql
CREATE TABLE completeness_profile (
  profile      TEXT NOT NULL,           -- 'taxon.v1', 'place.v1', 'occurrence.v1'
  target_table TEXT NOT NULL,
  field        TEXT NOT NULL,
  weight       REAL NOT NULL DEFAULT 1.0,
  -- 1 = its absence means the record is unusable, not merely thin
  is_required  INTEGER NOT NULL DEFAULT 0,
  PRIMARY KEY (profile, field)
);

CREATE TABLE completeness (
  target_table     TEXT NOT NULL CHECK (target_table IN ('taxon','place','occurrence')),
  target_id        INTEGER NOT NULL,
  profile          TEXT NOT NULL,
  -- MACHINE: recomputed on every export. JSON array of absent field names.
  missing_fields   TEXT NOT NULL DEFAULT '[]',
  missing_count    INTEGER NOT NULL DEFAULT 0,
  score            REAL,                 -- 0.0..1.0, weighted by profile
  computed_at      TEXT NOT NULL,
  -- HUMAN: written only by a person. Never touched by the recompute job.
  last_verified_at TEXT,
  verified_by      TEXT,
  verified_note    TEXT,
  PRIMARY KEY (target_table, target_id, profile)
);

CREATE INDEX idx_completeness_score   ON completeness(target_table, score);
CREATE INDEX idx_completeness_stale   ON completeness(target_table, last_verified_at);
```

`idx_completeness_stale` answers the question the table exists for — "what has nobody
checked in longest" — as an index scan.

### What it would say on day one (measured)

`taxon` (n=2,664): no `common_en` **1,851** (69%) · no `common_ta` **2,604** (98%) ·
no `iucn` **2,582** (97%) · no `kingdom` **899** · no `class_` **825** · no `rank` **771** ·
no `gbif_key` **430**

`place` (n=92): no `elevation_m` **92 (100% — the column is entirely unused)** ·
no `population` **91** · no `forest_range` **88** · no `wikidata_id` **81** ·
no `name_ta` **75** · no `osm_id` **20** · no coordinates **4**

Two of those are findings in their own right: `place.elevation_m` has never been populated
for any row, and 4 of 92 places have no coordinates at all — which also means they can
never receive an `occurrence.place_id` by containment under §2b.

---

## 8. Contributions — schema only, with the safety rules in the constraints

### DDL

```sql
CREATE TABLE contributor (
  id            INTEGER PRIMARY KEY,
  display_name  TEXT NOT NULL,
  -- Store a hash, not the address. The plaintext contact, if ever needed, does
  -- not belong in a database that is committed to git (see §9).
  email_sha256  TEXT UNIQUE,
  affiliation   TEXT,
  is_trusted    INTEGER NOT NULL DEFAULT 0,
  created_at    TEXT NOT NULL
);

CREATE TABLE contribution (
  id             INTEGER PRIMARY KEY,
  contributor_id INTEGER REFERENCES contributor(id) ON DELETE SET NULL,
  kind           TEXT NOT NULL CHECK (kind IN ('sighting','photo','correction','comment')),

  -- NULLABLE on purpose: a new sighting has no existing record to point at.
  target_table   TEXT CHECK (target_table IN
                   ('taxon','place','occurrence','document','claim','photo')),
  target_id      INTEGER,
  -- A correction MUST say what it corrects; a sighting need not.
  CHECK (kind <> 'correction' OR (target_table IS NOT NULL AND target_id IS NOT NULL)),

  payload_json   TEXT NOT NULL,
  claimed_taxon_id INTEGER REFERENCES taxon(id),
  claimed_place_id INTEGER REFERENCES place(id),
  observed_on    TEXT,

  -- Submitted coordinates are PRIVATE until moderated, and a sighting of a
  -- sensitive taxon must be coarsened before it is ever readable. Same rule as
  -- §2a, applied at the point of entry.
  raw_lat        REAL,
  raw_lon        REAL,
  public_lat     REAL,
  public_lon     REAL,
  coords_withheld INTEGER NOT NULL DEFAULT 0,

  moderation_state TEXT NOT NULL DEFAULT 'pending'
                   CHECK (moderation_state IN ('pending','approved','rejected')),
  moderated_by   TEXT,
  moderated_at   TEXT,
  moderation_note TEXT,
  -- An approved contribution must never silently mutate a curated row. Record
  -- what it became, so the provenance chain stays followable in both directions.
  applied_table  TEXT,
  applied_id     INTEGER,
  CHECK (moderation_state <> 'approved' OR moderated_at IS NOT NULL),

  submitted_at   TEXT NOT NULL,
  submitter_ip_sha256 TEXT,
  user_agent     TEXT,
  created_at     TEXT NOT NULL
);

CREATE INDEX idx_contrib_queue  ON contribution(moderation_state, submitted_at);
CREATE INDEX idx_contrib_target ON contribution(target_table, target_id);
CREATE INDEX idx_contrib_taxon  ON contribution(claimed_taxon_id, moderation_state);
```

`idx_contrib_queue(moderation_state, submitted_at)` is the moderation inbox query, and it
is the only one that has to be fast.

Three constraints are doing real safety work and should not be relaxed for convenience:
the `kind <> 'correction' OR target…` check (a correction that names no target is
unactionable), the `approved` ⇒ `moderated_at` check (nothing becomes public without a
recorded moderation act), and the raw/public coordinate split (a citizen sighting of a
tiger is the single most sensitive record this project could ever hold).

**Open question for you:** whether contributions live in `atlas.db` at all. `atlas.db` is
**committed to git** (36 MB binary, §9). Public submissions — including PII and precise
coordinates of sensitive species — would then be in the repository's permanent history,
where deletion does not remove them. My recommendation is that `contribution` lives in D1
or a separate uncommitted store and only *approved, coarsened* rows ever reach `atlas.db`.
That is an architecture decision, so I am raising it rather than taking it.

---

## 9. Migration plan

### Constraints specific to this database

1. **`PRAGMA foreign_keys` is `0`** (off) — every existing FK is declarative only. I ran
   `PRAGMA foreign_key_check`: **zero violations**. So enforcement can be switched on
   safely today, and doing so *before* adding tables is much cheaper than after.
2. **`data/atlas.db` is a committed 36 MB binary.** Every migration that rewrites pages
   adds tens of MB to git history permanently. Batch the changes into as few commits as
   possible; `VACUUM` once at the end, not per step.
3. **SQLite `ALTER TABLE` limits.** Only `ADD COLUMN` (with a constant default),
   `RENAME`, and `DROP COLUMN` are available. **Every column above is nullable or has a
   constant default, so no table rebuild is required.** The three `NOT NULL DEFAULT`
   additions (`claim.method`, `value_is_approx`) are constant defaults and therefore legal.
4. **`claim`'s existing DDL is fragile.** Its `status`/`CHECK`/`superseded_by_claim_id`
   clauses were added by a rebuild and the formatting shows it (the `CHECK` binds across a
   line break). `ADD COLUMN` is safe; **do not rebuild `claim`** without dumping first.
5. **`scripts/sync_d1_to_atlas.py` writes these tables.** It builds column lists
   explicitly, so additive nullable columns are safe — but it must be re-read before any
   `NOT NULL` addition, and `harvest_provenance` has no mapping for the new tables, so the
   sync will ignore them until taught otherwise.
6. **`exports/*.json` shapes are load-bearing.** `documents.json` is consumed by
   `history.html` (by-decade chart) and `record.html` (bibliography) as a flat array — the
   reason `ece82ab` added a separate `documents_unreviewed.json` instead of reshaping it.
   No step below changes an existing export's shape.

### Order

Each step is independently revertible and leaves the database readable by the current site.

| # | Step | Risk | Why this order |
|---|---|---|---|
| 0 | `sqlite3 data/atlas.db ".backup"` to `data/db-backups/` | none | `data/db-backups/` already exists and is used |
| 1 | Enable `PRAGMA foreign_keys=ON` in `sync_d1_to_atlas.py` + `export_from_db.py`; re-run `foreign_key_check` in CI | low — 0 violations today | Cheapest now; every later FK depends on it meaning something |
| 2 | Additive columns on `taxon` (`canonical_name`, `binomial`, `name_words`) + backfill + indexes | low — new columns only | §1 and the §5 root-cause fix both need `binomial` |
| 3 | `category` + `taxon_category` + seed + derive | low — new tables | Depends on 2 for the ~800 uncategorisable rows to be countable |
| 4 | Additive columns on `occurrence` (`public_lat/lon`, `public_coord_basis`, `place_id`, `place_basis`, `event_date_norm`, `event_year`) | **medium** | Largest table (78,467). Backfill is a full rewrite — do it in one pass, not six |
| 5 | Repoint every export/page to `public_lat/public_lon`; **only then** backfill raw `lat/lon` | **highest in the plan** | Order is critical: if the read path is not moved first, step 4's backfill can expose raw coordinates. Do not reverse 4 and 5. |
| 6 | `occurrence` indexes (`place`, `year`, `taxon_year`, `dataset`, `place_taxon`) | low | After the columns exist and are backfilled, so each index is built once |
| 7 | Additive columns on `claim` + parse `value`/`as_of` + resolve subjects | low, but **needs your rulings** from §3 | Blocked on the `2024-25` and `reserve`→place 88 decisions |
| 8 | `habitat`, `relationship_predicate`, `relationship` | low — new tables, empty | No dependants |
| 9 | `photo` + backfill the 576 | low | Depends on 2 (`taxon_id` resolution) |
| 10 | `place_link` + `place_hub_count` + orphan view; populate from `claim` (15 resolvable) and `occurrence.place_id` | medium — the volume step | Must come after 4/6, since occurrence links come from `place_id` |
| 11 | `completeness_profile` + `completeness` + first compute | low | Wants every other column to exist first, or it measures a moving target |
| 12 | `contributor` + `contribution` | low — new, empty | **Gated on the §8 "does this belong in a committed database" question** |
| 13 | `VACUUM`, regenerate `exports/`, verify site | low | Once |

### What could break, concretely

- **Step 5 is the one that can leak data.** Repointing reads to `public_lat/public_lon`
  before those columns are populated shows an empty map; populating them while reads still
  point at raw `lat/lon` shows exact sensitive locations. The safe sequence is: add columns
  (4) → backfill `public_*` from current values → verify all 398 sensitive occurrences have
  `public_precision_deg = 0.045` → switch reads (5) → and only then treat `lat/lon` as
  private. On this dev copy 13 of those 398 are intentionally raw, so **the verification
  must be run against a production-mode sync, not this file.**
- **Step 4's backfill rewrites a 78,467-row table** in a 36 MB committed binary. Expect a
  large git delta. One commit.
- **Step 10 can double-count.** If occurrences are put into `place_link` as well as
  carrying `place_id`, the hub counts inflate by ~78k. The design deliberately excludes
  them; the orphan view should also assert `place_link` holds no `occurrence` rows.
- **Step 3's derived categories will be wrong for the 411 authored duplicates** until the
  §1 open question is settled: `Panthera tigris` and `Panthera tigris tigris` would both
  appear under `mammals` as separate taxa. That is *visible* wrongness, which is the
  intended failure mode — the alternative is hiding one of them.
- **Nothing above changes an existing column's meaning except `occurrence.lat/lon`**, and
  that change is the point of step 5.

### Not in this plan, on purpose

No migration file has been written and none has been run. `data/migrations/` currently
holds `0001_claim_supersede.sql` and `0001_harvest_provenance.sql` — note both are numbered
`0001`, so the next file should resolve that collision rather than add a third `0001`.

---

## 10. Decisions I need from you

Ranked by what blocks the most work.

1. **Do the four missing documents exist elsewhere?** (§0) If the coordinate policy says
   something other than what `coarsen.js` and the triggers do, §2a and §5 need revisiting.
2. **`occurrence.lat/lon` two-pair split** (§2a) — approve, and confirm the production
   verification in step 5 runs against a production-mode sync.
3. **`2024-25` → which date range** (§3), and whether the 5 undated `population` claims
   join the series.
4. **Point the 23 `reserve` claims at `place` 88** (§3) — a one-line backfill, but an
   assertion about what they describe.
5. **Merge the ~411 authorship duplicates?** (§1) Same shape as the `e9490d8` place merge.
6. **`place_id` granularity for sensitive occurrences** (§2b) — village-level naming
   re-identifies a coarsened coordinate.
7. **Does `contribution` belong in a git-committed database?** (§8) My recommendation is no.
8. **Whether global (non-local) relationships are in scope at all** (§4) — plausibly
   already answered by the missing scope-lock document.
