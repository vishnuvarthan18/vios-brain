**Vishnu** (2026-09-06T02:17): Build the sync pipeline that moves data from the harvest-engine's D1 database into atlas.db (the database the website actually reads). This does not exist yet — per harvest-engine/README.md it was planned but never built.

Work on the `dev` branch only. Do NOT merge to main, do NOT deploy anywhere, do NOT run wrangler pages deploy or wrangler deploy — stop and report back once the code is written and tested locally, and wait for explicit go-ahead before any deploy or merge.

1. Write a sync script/stage that reads from D1 tables (place, taxon, occurrence, document, claim, source) and writes/updates the corresponding rows in atlas.db, matching whatever schema atlas.db already uses for those entity types. Check both schemas first and report any mismatches before writing the sync logic — don't guess field mappings.

2. Handle this as an idempotent upsert (match on a stable key like source_url + retrieved_at, or whatever unique identifier exists) so re-running the sync later doesn't create duplicates — we're planning to re-run harvest-engine after fixing its known bugs, and will re-sync again then.

3. Add provenance: every row synced into atlas.db should be traceable back to which D1 source/job_run it came from, so we can tell "real collected data" apart from anything already in atlas.db from other sources.

4. Run it once locally against a LOCAL COPY of atlas.db (not the live one) to test, and report exactly how many rows moved per table, and any rows skipped/rejected with reasons.

5. Report back: the sync script's location, the local test run's row counts, any schema mismatches found, and confirm nothing was deployed or merged.

Known caveat to flag in your report, not fix now: harvest-engine's data was frozen by a dedup bug for over a week, so what syncs now is a stale/incomplete snapshot, not a full 15-day run. That's expected — we'll re-run harvest-engine properly later and re-sync.

**Claude** (2026-09-06T02:17): I'll start by understanding the current state of both systems before writing any sync logic.

**Claude** (2026-09-06T02:17): _[tool: Bash]_

**Claude** (2026-09-06T02:17): _[tool: Bash]_

**Claude** (2026-09-06T02:17): _[tool: Bash]_

**Claude** (2026-09-06T02:17): _[tool: Bash]_

**Claude** (2026-09-06T02:17): Now let me read the D1 schema and find the local D1 data.

**Claude** (2026-09-06T02:17): _[tool: Bash]_

**Claude** (2026-09-06T02:17): _[tool: Bash]_

**Claude** (2026-09-06T02:17): _[tool: Bash]_

**Claude** (2026-09-06T02:17): _[tool: Bash]_

**Claude** (2026-09-06T02:17): _[tool: Bash]_

**Claude** (2026-09-06T02:17): _[tool: Bash]_

**Claude** (2026-09-06T02:18): _[tool: Bash]_

**Claude** (2026-09-06T02:18): _[tool: Bash]_

**Claude** (2026-09-06T02:18): Both local D1 copies are stale dev leftovers (Aug 24–25). The real data lives in remote D1. Let me get onto `dev` first and check wrangler auth.

**Claude** (2026-09-06T02:18): _[tool: Bash]_

**Claude** (2026-09-06T02:18): On `dev` now. The dev README is more current — let me read the rest and check dev's migrations.

**Claude** (2026-09-06T02:18): _[tool: Bash]_

**Claude** (2026-09-06T02:18): _[tool: Bash]_

**Claude** (2026-09-06T02:18): _[tool: Bash]_

**Claude** (2026-09-06T02:18): _[tool: Bash]_

**Claude** (2026-09-06T02:18): Wrangler is authenticated. Let me read the **live** remote D1 schema and row counts (read-only queries, no deploy).

**Claude** (2026-09-06T02:18): _[tool: Bash]_

**Claude** (2026-09-06T02:19): _[tool: Bash]_

**Claude** (2026-09-06T02:19): _[tool: Bash]_

**Claude** (2026-09-06T02:19): Live D1 schema matches the migrations. Now the row counts — my JSON parse was wrong, let me look at the actual shape.

**Claude** (2026-09-06T02:19): _[tool: Bash]_

**Claude** (2026-09-06T02:19): _[tool: Bash]_

**Claude** (2026-09-06T02:19): _[tool: Bash]_

**Claude** (2026-09-06T02:19): _[tool: Bash]_

**Claude** (2026-09-06T02:19): Live D1 has real data. Let me set up a reusable query helper rather than fighting with output parsing.

**Claude** (2026-09-06T02:19): _[tool: Bash]_

**Claude** (2026-09-06T02:19): _[tool: Bash]_

**Claude** (2026-09-06T02:19): _[tool: Bash]_

**Claude** (2026-09-06T02:20): I now have both schemas. Before mapping fields, let me check the actual data conventions in atlas.db and what identifiers the streams actually store.

**Claude** (2026-09-06T02:20): _[tool: Bash]_

**Claude** (2026-09-06T02:20): _[tool: Bash]_

**Claude** (2026-09-06T02:21): atlas.db already holds 74,180 GBIF + 720 iNaturalist occurrences — so something already loaded this kind of data. I need to find that path before I risk duplicating it.

**Claude** (2026-09-06T02:21): _[tool: Bash]_

**Claude** (2026-09-06T02:21): _[tool: Bash]_

**Claude** (2026-09-06T02:21): _[tool: Bash]_

**Claude** (2026-09-06T02:21): _[tool: Bash]_

**Claude** (2026-09-06T02:21): No importer exists in the repo — atlas.db is a checked-in artifact built out-of-repo. Now the decisive question: does D1's occurrence retain any external identifier?

**Claude** (2026-09-06T02:21): _[tool: Bash]_

**Claude** (2026-09-06T02:21): Critical finding confirmed. Let me verify the data implications on live D1.

**Claude** (2026-09-06T02:21): _[tool: Bash]_

**Claude** (2026-09-06T02:21): _[tool: Bash]_

**Claude** (2026-09-06T02:22): _[tool: Bash]_

**Claude** (2026-09-06T02:22): The registry gives an authoritative source→stream map. Now let me quantify the collision risks for each table.

**Claude** (2026-09-06T02:22): _[tool: Bash]_

**Claude** (2026-09-06T02:22): _[tool: Bash]_

**Claude** (2026-09-06T02:23): Data is clean. Let me check the R2 key format and the overlap with existing atlas.db rows.

**Claude** (2026-09-06T02:23): _[tool: Bash]_

**Claude** (2026-09-06T02:23): R2 key's last segment is the content SHA-256 — that maps to `atlas.source.sha256`. Let me check data freshness and test extraction paging.

**Claude** (2026-09-06T02:23): _[tool: Bash]_

**Claude** (2026-09-06T02:23): _[tool: Bash]_

**Claude** (2026-09-06T02:24): Freshness confirms the dedup-bug signature exactly: `source` advanced to 2026-09-05 but `occurrence`/`taxon` froze at 2026-08-29. Let me build the script, starting with extraction.

**Claude** (2026-09-06T02:24): _[tool: Bash]_

**Claude** (2026-09-06T02:24): The registry covers the mapping. Now let me write the sync script.

**Claude** (2026-09-06T02:27): _[tool: Write]_

**Claude** (2026-09-06T02:27): Now let me extract the D1 snapshot and set up a local test copy of atlas.db.

**Claude** (2026-09-06T02:27): _[tool: Bash]_

**Claude** (2026-09-06T02:27): Extraction succeeded (8,166 entity rows). Now let me make a **local copy** of atlas.db — using sqlite's backup so the WAL is included — and verify it's a true copy.

**Claude** (2026-09-06T02:28): _[tool: Bash]_

**Claude** (2026-09-06T02:28): Copy verified identical. Now a dry run first (rolls back) before any write.

**Claude** (2026-09-06T02:28): _[tool: Bash]_

**Claude** (2026-09-06T02:28): Dry run is clean — zero skips. Before committing, let me check two things that look suspicious: what `place` adopted, and why 3 documents were "updated" mid-run.

**Claude** (2026-09-06T02:28): _[tool: Bash]_

**Claude** (2026-09-06T02:28): _[tool: Bash]_

**Claude** (2026-09-06T02:28): Place adoption is legitimate — 42 match on both slug *and* osm_id consistently. The 3 document "updates" must come from case-normalisation in the fingerprint. Let me run live and inspect the actual provenance records.

**Claude** (2026-09-06T02:28): _[tool: Bash]_

**Claude** (2026-09-06T02:29): Now let me verify the result — especially the sensitive-species coordinates and those 3 document collisions.

**Claude** (2026-09-06T02:29): _[tool: Bash]_

**Claude** (2026-09-06T02:29): _[tool: Bash]_

**Claude** (2026-09-06T02:29): Safety guarantee holds — all 5 sensitive occurrences carry coarsened coordinates, verified against the raw values. Now the 3 document merges:

**Claude** (2026-09-06T02:29): _[tool: Bash]_

**Claude** (2026-09-06T02:29): These 3 merges are correct — same paper, same DOI in different case (DOIs are case-insensitive by spec). But that exposes a latent bug: my *natural key* lookup on `doi` is case-sensitive in SQLite. Let me check the scale.

**Claude** (2026-09-06T02:29): _[tool: Bash]_

**Claude** (2026-09-06T02:29): Real issue confirmed: atlas.db already contains 2 case-colliding DOI pairs, and D1 has 21 uppercase DOIs. Case-sensitive matching would create duplicates. Let me fix the lookup to be case-insensitive.

**Claude** (2026-09-06T02:29): _[tool: Edit]_

**Claude** (2026-09-06T02:30): Now the critical test: rebuild a fresh copy and run the sync **twice** to prove idempotency.

**Claude** (2026-09-06T02:30): _[tool: Bash]_

**Claude** (2026-09-06T02:30): Idempotency proven — run 2 inserted **zero** rows. Let me verify the exact deltas and that the live db is untouched.

**Claude** (2026-09-06T02:30): _[tool: Bash]_

**Claude** (2026-09-06T02:30): Live atlas.db is byte-identical and has no provenance tables. Now let me verify the provenance requirement actually works end-to-end.

**Claude** (2026-09-06T02:30): _[tool: Bash]_

**Claude** (2026-09-06T02:30): _[tool: Bash]_

**Claude** (2026-09-06T02:31): Full traceability works. Let me verify the site's own export script still runs against the synced database — that would catch any schema breakage.

**Claude** (2026-09-06T02:31): _[tool: Bash]_

**Claude** (2026-09-06T02:31): _[tool: Bash]_

**Claude** (2026-09-06T02:31): `SELECT *` is used, so my choice to add only new *tables* (never columns) matters. Let me prove it in a sandbox that can't touch the repo.

**Claude** (2026-09-06T02:31): _[tool: Bash]_

**Claude** (2026-09-06T02:31): Export pipeline runs clean against the synced DB. One more thing to verify before wrapping up — what the sync *overwrote* on the 1,282 adopted (pre-existing) rows.

**Claude** (2026-09-06T02:31): _[tool: Bash]_

**Claude** (2026-09-06T02:31): This is important — the sync is overwriting identity columns on pre-existing rows. Let me look at the actual cases.

**Claude** (2026-09-06T02:31): _[tool: Bash]_

**Claude** (2026-09-06T02:32): _[tool: Bash]_

**Claude** (2026-09-06T02:32): Clear finding, and it changes the design. atlas.db's slugs preserve diacritics (`micropsalliota-höhn`); harvest-engine's slugifier mangles them (`micropsalliota-h-hn`). Overwriting curated atlas values with raw machine output is a regression. Let me fix the merge policy.

**Claude** (2026-09-06T02:32): _[tool: Edit]_

**Claude** (2026-09-06T02:32): _[tool: Edit]_

**Claude** (2026-09-06T02:32): Now update `record_provenance` to persist the origin:

**Claude** (2026-09-06T02:33): _[tool: Edit]_

**Claude** (2026-09-06T02:33): _[tool: Edit]_

**Claude** (2026-09-06T02:33): _[tool: Edit]_

**Claude** (2026-09-06T02:33): Now re-test from a fresh copy, twice, to confirm both the fix and idempotency:

**Claude** (2026-09-06T02:33): _[tool: Bash]_

**Claude** (2026-09-06T02:33): Now let me verify no curated values were clobbered this time:

**Claude** (2026-09-06T02:33): _[tool: Bash]_

**Claude** (2026-09-06T02:34): The fix works — 634 genuine enrichments (205 abstracts, 130 OA links, 204 common names), zero overwrites. Final verification batch:

**Claude** (2026-09-06T02:34): _[tool: Bash]_

**Claude** (2026-09-06T02:34): _[tool: Bash]_

**Claude** (2026-09-06T02:34): That re-check surfaced a serious problem. Elephant, leopard and white-rumped vulture are now showing **full-precision** coordinates. Let me investigate.

**Claude** (2026-09-06T02:34): _[tool: Bash]_

**Claude** (2026-09-06T02:34): _[tool: Bash]_

**Claude** (2026-09-06T02:34): Found a real safety defect. harvest-engine's registry lists `Elephas maximus` / `Panthera pardus` as binomials, but GBIF returns **trinomials** (`Elephas maximus indicus`), which never match — so the coarsening trigger never fires for elephant and leopard. Let me quantify.

**Claude** (2026-09-06T02:34): _[tool: Bash]_

**Claude** (2026-09-06T02:35): Confirmed: 8 occurrences (5 elephant, 2 leopard, 1 vulture) would land on the public site at **exact** coordinates. The sync must not trust D1's flag alone. Let me add defence-in-depth.

**Claude** (2026-09-06T02:35): _[tool: Edit]_

**Claude** (2026-09-06T02:35): Now wire the oracle into the occurrence and taxon sync:

**Claude** (2026-09-06T02:35): _[tool: Edit]_

**Claude** (2026-09-06T02:35): _[tool: Edit]_

**Claude** (2026-09-06T02:35): Now the taxon side — a newly-inserted taxon must not land with `sensitive=0` when the registry says otherwise:

**Claude** (2026-09-06T02:35): _[tool: Edit]_

**Claude** (2026-09-06T02:35): _[tool: Bash]_

**Claude** (2026-09-06T02:35): Now let me add the coarsening report and re-run the full test:

**Claude** (2026-09-06T02:36): _[tool: Edit]_

**Claude** (2026-09-06T02:36): _[tool: Bash]_

**Claude** (2026-09-06T02:36): Now the definitive safety check — no sensitive species may retain an exact coordinate:

**Claude** (2026-09-06T02:36): _[tool: Bash]_

**Claude** (2026-09-06T02:36): All 13 sensitive records are coarsened. Final idempotency re-check and export validation:

**Claude** (2026-09-06T02:36): _[tool: Bash]_

**Claude** (2026-09-06T02:36): _[tool: Bash]_

**Claude** (2026-09-06T02:36): All green. Let me finalise: gitignore the snapshot dir, add the provenance schema as a documented SQL file, and commit on `dev`.

**Claude** (2026-09-06T02:36): _[tool: Bash]_

**Claude** (2026-09-06T02:36): _[tool: Bash]_

**Claude** (2026-09-06T02:37): _[tool: Write]_

**Claude** (2026-09-06T02:37): _[tool: Bash]_

**Claude** (2026-09-06T02:37): _[tool: Bash]_

**Claude** (2026-09-06T02:37): Let me fix two stale docstring references before committing:

**Claude** (2026-09-06T02:37): _[tool: Bash]_

**Claude** (2026-09-06T02:37): Now let me commit on `dev` (local only — no push, no merge, no deploy):

**Claude** (2026-09-06T02:37): _[tool: Bash]_

**Claude** (2026-09-06T02:38): _[tool: Bash]_

**Claude** (2026-09-06T02:38): _[tool: Bash]_

**Claude** (2026-09-06T02:38): Done. Everything is written, tested locally, and committed on `dev` only.

## Location

- **[scripts/sync_d1_to_atlas.py](scripts/sync_d1_to_atlas.py)** — the sync pipeline (two phases: `--extract` pulls D1 to a JSON snapshot; the load phase transforms and upserts)
- **[data/migrations/0001_harvest_provenance.sql](data/migrations/0001_harvest_provenance.sql)** — provenance schema (also auto-applied by the script)
- Commit `fae5c9b` on `dev`, **local only — not pushed**

```bash
python3 scripts/sync_d1_to_atlas.py --atlas <copy>.db --extract [--dry-run]
```

## Local test run (copy of atlas.db, live file never opened for write)

| table | inserted | adopted | blanks filled | skipped |
|---|---|---|---|---|
| source | 557 | 1 | 2 | 0 |
| place | 4 | 42 | 1 | 0 |
| taxon | 771 | 549 | 204 | 0 |
| document | 1839 | 691 | 427 | 0 |
| occurrence | 3567 | 0 | 0 | 0 |
| claim | 0 | 0 | 0 | 0 |

**6,738 inserted, 1,283 adopted, 0 skipped/rejected.** Row counts 53→610 source, 89→93 place, 1893→2664 taxon, 7190→9029 document, 74900→78467 occurrence. A second run over the same snapshot inserted **0** rows — idempotency verified, not assumed. `scripts/export_from_db.py` runs clean against the result.

## Schema mismatches found

Nothing was 1:1. The ones that would have caused silent damage:

- **`occurrence` has no external identifier in D1.** atlas requires `UNIQUE(dataset, external_id)`. The GBIF stream *reads* `rec.key` but never stores it — its comment claims it dedupes on the GBIF key while the code actually dedupes on `(taxon, date, lat, lon)`. That's almost certainly your dedup bug. I did not fabricate GBIF keys; IDs are namespaced `harvest-engine:<stream>` / `d1:<id>` so they can't collide with atlas's existing 74,180 `gbif` rows — meaning D1 occurrences that duplicate existing atlas ones **won't be merged**. Your call whether to reconcile them; it can't be done reliably without the upstream key.
- **`public_precision_m` (metres) vs `public_precision_deg` (degrees)** — not a unit cast. A non-sensitive row can carry 500,000 m uncertainty while being uncoarsened (grid 0).
- **`place.precision`** is TEXT (`'village'`) vs `precision_m` INTEGER. Left NULL rather than inventing a distance; the label is kept in provenance. **This is an open editorial call for you.**
- `retrieved_at` ISO-8601 TEXT → REAL unix epoch; `license`→`licence`; `base_url`→`url` (700 D1 rows collapse to 558, since D1 is per-fetch and atlas is per-URL); `claim.subject_id` INTEGER → `subject_key` TEXT slug; `name_local`→`name_ta`; D1 free-text `type` → atlas's 2-value `kind`. `endemic_flag`, `place_id`, `kind`, `terms_url`, `http_status` have no home in atlas — all preserved in `notes_json`.

## Two safety problems this surfaced

**1. atlas.db has no `public_lat`/`public_lon` — its `lat`/`lon` *is* the public coordinate.** The sync writes D1's coarsened columns and never its raw ones.

**2. D1's `is_sensitive` is wrong for the highest-risk species.** `sensitive-species.json` lists binomials (`Elephas maximus`), but GBIF returns trinomials (`Elephas maximus indicus`), which never match — so the coarsening trigger never fires and **D1's own `public_lat` holds the exact coordinate** for 5 elephant, 2 leopard and 1 white-rumped vulture records. The sync now re-decides sensitivity from the union of D1 + atlas.db + the registry (prefix-matched) and coarsens itself; all 13 sensitive records verified coarsened. **This is an upstream harvest-engine bug that still needs fixing** — worth doing before the re-run.

Also: adopted rows are fill-blanks-only. My first version overwrote curated values — it replaced atlas's correct `micropsalliota-höhn` with harvest-engine's mangled `micropsalliota-h-hn` and clobbered 68 titles. Now 634 blanks were filled (205 abstracts, 130 OA links, 204 common names) and **zero existing values overwritten**, verified by diff.

## Your caveat, confirmed with evidence

The freeze is visible in the data: `source.retrieved_at` runs to 2026-09-05, but `occurrence` and `taxon` stop dead at **2026-08-29** — fetches kept happening, occurrence writes stopped. So this is an 8-day-stale partial snapshot, as you said.

## Nothing deployed or merged

Live `data/atlas.db` is byte-identical (md5 `dcbee4e3…`) and has no provenance tables. `exports/` untouched. Remote D1 unchanged — every query was a `SELECT`. No `wrangler deploy` or `pages deploy` ran. Still on `dev`, ahead of `origin/dev` by 1, unpushed.

One housekeeping note: your uncommitted `harvest-engine/wrangler.toml` edit on `main` blocked the branch switch, so I stashed it — `stash@{0}`, recoverable with `git stash pop` once you're back on `main`. Both branches already have `crons = []`; only the comment differs.

**Vishnu** (2026-09-06T02:51): Apply the D1 → atlas.db sync (scripts/sync_d1_to_atlas.py) to the real dev atlas.db now — not a copy this time. Stay on the `dev` branch. Do NOT merge to main, do NOT deploy, do NOT run wrangler pages deploy or wrangler deploy.

Before running: take a backup copy of the current dev atlas.db first (timestamped filename), so we can roll back if anything looks wrong after.

Coordinate handling: do NOT force-coarsen sensitive-species coordinates for this dev database. Exact coordinates are fine in dev/staging — the project has forest department permission for precise location data outside the public production environment. Only production needs coarsening, and that's a separate gate to build later, not something to apply here. Write D1's raw (uncoarsened) coordinates for sensitive species into dev's atlas.db, not the masked ones.

Run it for real (not --dry-run) against the actual dev atlas.db. Report:
1. The backup file location.
2. Final row counts per table (source, place, taxon, document, occurrence, claim), same format as the test run.
3. Confirm idempotency again by running it a second time and showing 0 new inserts.
4. Confirm nothing was deployed, merged, or pushed.

Known caveat already understood: this is an 8-day-stale partial snapshot due to the dedup-freeze bug — expected, not a new problem.

**Claude** (2026-09-06T02:52): Understood — you've made the call on coordinates, and the reasoning (dev/staging under forest department permission, production coarsening as a separate gate) is clear. I'll write raw coordinates.

First let me verify the current state of the target file.

**Claude** (2026-09-06T02:52): _[tool: Bash]_

**Claude** (2026-09-06T02:52): Target is clean and unchanged. Now let me add an explicit `--raw-coordinates` flag — safe default stays "coarsen", so this can only happen when asked for by name.

**Claude** (2026-09-06T02:52): _[tool: Bash]_

**Claude** (2026-09-06T02:52): _[tool: Edit]_

**Claude** (2026-09-06T02:52): _[tool: Edit]_

**Claude** (2026-09-06T02:52): Now wire the flag through the Syncer, driver, CLI and report:

**Claude** (2026-09-06T02:53): _[tool: Bash]_

**Claude** (2026-09-06T02:53): Let me update the module docstring to reflect the new mode, then verify the flag behaves correctly on a copy *before* touching the real file.

**Claude** (2026-09-06T02:53): _[tool: Bash]_

**Claude** (2026-09-06T02:54): Now let me refresh the D1 snapshot and validate the new flag on a scratch copy first.

**Claude** (2026-09-06T02:54): _[tool: Bash]_

**Claude** (2026-09-06T02:55): _[tool: Bash]_

**Claude** (2026-09-06T02:55): Let me verify the coordinates written really are D1's raw values before touching the real file.

**Claude** (2026-09-06T02:55): _[tool: Bash]_

**Claude** (2026-09-06T02:55): All 3,567 occurrences carry D1's raw values, and the mode is recorded in the database. Now the backup, then the real run.

**Claude** (2026-09-06T02:56): _[tool: Bash]_

**Claude** (2026-09-06T02:56): Backup verified (integrity `ok`, counts match). Let me gitignore the backup directory so a 26 MB binary can't get committed by accident, then run for real.

**Claude** (2026-09-06T02:56): _[tool: Bash]_

**Claude** (2026-09-06T02:56): Now the real run against `data/atlas.db`:

**Claude** (2026-09-06T02:56): _[tool: Bash]_

**Claude** (2026-09-06T02:56): Now the second run to confirm idempotency on the real database:

**Claude** (2026-09-06T02:56): _[tool: Bash]_

**Claude** (2026-09-06T02:57): Zero inserts on the second run. Let me verify the final state of the real database.

**Claude** (2026-09-06T02:57): _[tool: Bash]_

**Claude** (2026-09-06T02:57): _[tool: Bash]_

**Claude** (2026-09-06T02:57): _[tool: Bash]_

**Claude** (2026-09-06T02:58): _[tool: Bash]_

**Claude** (2026-09-06T02:58): The `main` ambiguity is that stray 0-byte `main` file (pre-existing, not mine). Let me disambiguate:

**Claude** (2026-09-06T02:58): _[tool: Bash]_

**Claude** (2026-09-06T02:58): Done. The sync ran for real against the dev `data/atlas.db`.

## 1. Backup location

```
data/db-backups/atlas-20260906-082616-pre-d1-sync.db
```

26,415,104 bytes, `PRAGMA integrity_check` → `ok`, row counts verified identical to the pre-sync database. Taken with `sqlite3 .backup` so the WAL is folded in. I gitignored `data/db-backups/` so the rollback copy can't be committed by accident.

Rollback is either `cp` that file back over `data/atlas.db`, or — since `data/atlas.db` is git-tracked and I did **not** commit it — simply `git checkout -- data/atlas.db`.

## 2. Final row counts

| table | before | after | delta |
|---|---|---|---|
| source | 53 | 610 | +557 |
| place | 89 | 93 | +4 |
| taxon | 1893 | 2664 | +771 |
| occurrence | 74900 | 78467 | +3567 |
| document | 7190 | 9029 | +1839 |
| claim | 83 | 83 | +0 |

**6,738 inserted, 1,283 adopted, 634 blanks filled, 0 skipped/rejected.** `integrity_check` → `ok`. 8,166 provenance rows written. The 74,900 pre-existing occurrences are untouched and remain distinguishable (no `harvest_provenance` entry).

## 3. Idempotency

Second run against the same real database:

```
TOTAL                0     8166        0        0              0
             inserted  updated  adopted  skipped  blanks filled
```

**0 inserts, 0 adoptions, 0 blanks filled** — all 8,166 rows matched their existing provenance entries and updated in place. Row counts unchanged.

## 4. Nothing deployed, merged, or pushed

- Branch: `dev`. `main` still at `025be91`, unchanged since session start; `dev` is not merged into it.
- Two commits on `dev`, both local — `git status` shows `[ahead 2]`, nothing pushed to `origin`.
- No `wrangler deploy` or `wrangler pages deploy`. `exports/` and `dist/` untouched, so nothing was even built for deploy.
- Remote D1 read-only throughout and unchanged (700/1320/2533/3567).

## Coordinates

Written raw as you asked. All 3,567 synced occurrences carry D1's exact `lat`/`lon` — verified value-by-value against the snapshot, zero differences. That includes 13 rows across 6 sensitive taxa (5 Asian elephant, 2 leopard, 2 Indian vulture, 2 sandalwood, 1 white-rumped vulture, 1 red-headed vulture).

I added a `--raw-coordinates` flag rather than editing the coarsening out, so the default stays "coarsen" and a database can only hold exact sensitive coordinates deliberately. Two things I kept deliberately, since they make the later production gate cheap:

- **`taxon.sensitive` is still set correctly**, from the union of D1 + atlas.db + the registry with prefix matching — so the trinomials (`Elephas maximus indicus`) that harvest-engine itself misses are still flagged. The rows needing masking stay findable: `SELECT o.* FROM occurrence o JOIN taxon t ON t.id=o.taxon_id WHERE t.sensitive=1`.
- **`harvest_sync_run.mode`** records `live+raw-coordinates`, so how this database was populated is auditable rather than inferred.

## Two things left for you

**I did not commit `data/atlas.db`.** It's modified in the working tree (26 MB → 37.8 MB). Committing it would write exact sensitive-species coordinates into git history permanently on a branch that may later merge to `main` — that's your call to make explicitly, not a side effect of a sync. Only the script and `.gitignore` are committed.

**I did not regenerate `exports/`.** The site reads those JSON files, not atlas.db directly, so the new rows aren't visible to the site yet — `exports/` and `data/atlas.db` are now out of sync, which `export_from_db.py`'s own header warns against. When you want that: `make export`. I left it alone because it rewrites tracked files that feed the deploy path, and you asked me to stop before anything deploy-adjacent.

**Vishnu** (2026-09-06T03:07): Regenerate exports/ from the current dev atlas.db (post-sync, with the +6,738 rows). Run whatever the repo's standard command is (`make export`, or `export_from_db.py` directly if that's the actual entrypoint).

Stay on `dev`. Do NOT commit, merge, push, or deploy anything — just regenerate the JSON files and stop.

Report back:
1. Which files under exports/ changed and their new sizes/row counts.
2. Whether export_from_db.py's own staleness check (the one that warned atlas.db and exports/ were out of sync) now passes.
3. Confirm the dev site, when run locally, actually reflects the new data (e.g. spin up the local dev server and check one page that pulls from the newly-synced tables — occurrence or document counts somewhere visible).
4. Confirm nothing was committed, pushed, merged, or deployed.

**Claude** (2026-09-06T03:07): Let me capture the current state of `exports/` first, and check what staleness machinery actually exists in the script.

**Claude** (2026-09-06T03:07): _[tool: Bash]_

**Claude** (2026-09-06T03:07): _[tool: Bash]_

**Claude** (2026-09-06T03:07): No programmatic staleness check exists — line 15 is prose in the docstring and line 81's warning is about a missing curated file. Let me see what `manifest.json` actually records, then run the export.

**Claude** (2026-09-06T03:07): _[tool: Bash]_

**Claude** (2026-09-06T03:07): `manifest.json` records counts — that's the real drift indicator (it currently shows the pre-sync 89/1893/74900/53). Let me check the documents filter, then run the export.

**Claude** (2026-09-06T03:07): _[tool: Bash]_

**Claude** (2026-09-06T03:07): `documents.json` filters on `relevance_tier IN ('A','B')` — that explains the 928. Let me check how the new rows land, and verify the claim columns the export needs.

**Claude** (2026-09-06T03:08): _[tool: Bash]_

**Claude** (2026-09-06T03:08): `status`/`superseded_by_claim_id` do exist, so the export will run. Noting one thing already: 1,841 harvested documents carry a NULL `relevance_tier`, and `documents.json` only includes A/B. Let me run the export.

**Claude** (2026-09-06T03:08): _[tool: Bash]_

**Claude** (2026-09-06T03:08): `documents.json` still shows 928 — but the database now has 929 A/B documents. Let me verify that directly rather than assume.

**Claude** (2026-09-06T03:08): _[tool: Bash]_

**Claude** (2026-09-06T03:08): The row count is unchanged but the content changed. Let me check what the pre-sync database actually held, since that determines whether exports/ was ever in sync.

**Claude** (2026-09-06T03:08): _[tool: Bash]_

**Claude** (2026-09-06T03:09): Now it's consistent. Let me confirm the tier distribution of the *newly inserted* documents specifically.

**Claude** (2026-09-06T03:09): _[tool: Bash]_

**Claude** (2026-09-06T03:09): All 1,839 new documents have NULL tier. Let me confirm that came from D1, then compile the full file-level diff.

**Claude** (2026-09-06T03:09): _[tool: Bash]_

**Claude** (2026-09-06T03:09): _[tool: Bash]_

**Claude** (2026-09-06T03:09): Important: **all 2,533 D1 documents have NULL `relevance_tier`** — harvest-engine never assigns one. Let me get the row counts and catch `places.geojson` (my glob missed it).

**Claude** (2026-09-06T03:09): _[tool: Bash]_

**Claude** (2026-09-06T03:09): Manifest now reflects post-sync counts. Let me verify it against the database, then start the local server.

**Claude** (2026-09-06T03:09): _[tool: Bash]_

**Claude** (2026-09-06T03:09): Manifest agrees with the database on every entity. Now let me start the local dev server and check a page that renders this data.

**Claude** (2026-09-06T03:10): _[tool: Bash]_

**Claude** (2026-09-06T03:10): _[tool: Bash]_

**Claude** (2026-09-06T03:10): Server serving the fresh data. Now let me find which page visibly renders these counts.

**Claude** (2026-09-06T03:10): _[tool: Bash]_

**Claude** (2026-09-06T03:10): _[tool: Bash]_

**Claude** (2026-09-06T03:10): `about.html` is the page — it fetches manifest, species, places.geojson, claims and coverage. Let me see how it renders them.

**Claude** (2026-09-06T03:10): _[tool: Bash]_

**Claude** (2026-09-06T03:10): _[tool: Bash]_

**Claude** (2026-09-06T03:10): `land.html` is the right page — it renders a gazetteer from `places.json` (89→93). Let me try to actually render it rather than just assert.

**Claude** (2026-09-06T03:10): _[tool: Bash]_

**Claude** (2026-09-06T03:10): Chrome is available. Let me render `land.html` headlessly and read the actual DOM the browser produces.

**Claude** (2026-09-06T03:11): _[tool: Bash]_

**Claude** (2026-09-06T03:11): `#placesHave` rendered as **93** — the post-sync value (was 89). Let me count the gazetteer entries properly and check a second page.

**Claude** (2026-09-06T03:11): _[tool: Bash]_

**Claude** (2026-09-06T03:11): _[tool: Bash]_

**Claude** (2026-09-06T03:11): `about.html` renders **32.5%** (was 31.6%). Now the strongest test — are the 4 newly-synced places actually visible in the rendered page?

**Claude** (2026-09-06T03:11): _[tool: Bash]_

**Claude** (2026-09-06T03:12): _[tool: Bash]_

**Claude** (2026-09-06T03:12): All 4 new places render. Let me stop the server and do the final integrity checks.

**Claude** (2026-09-06T03:12): _[tool: Bash]_

**Claude** (2026-09-06T03:12): Server stopped (exit 144 is my own `pkill`, expected). Final integrity checks:

**Claude** (2026-09-06T03:12): _[tool: Bash]_

**Claude** (2026-09-06T03:12): Exports regenerated with `make export` (which runs `python3 scripts/export_from_db.py` — that's the actual entrypoint).

## 1. Files changed under `exports/`

| file | before | after | Δ bytes | rows before → after |
|---|---|---|---|---|
| `species.json` | 512,575 | 724,360 | +211,785 | 1,893 → **2,664** |
| `documents.json` | 1,236,299 | 1,244,221 | +7,922 | 928 → 928 |
| `places.json` | 25,150 | 26,383 | +1,233 | 89 → **93** |
| `places_display.json` | 12,356 | 12,838 | +482 | 85 → **89** |
| `places.geojson` | 30,236 | 30,998 | +762 | 89 features |
| `coverage.json` | 1,757 | 1,758 | +1 | obj (31.6% → **32.5%**) |
| `manifest.json` | 343 | 343 | 0 | obj (counts updated) |

Unchanged: `claims.json`, `legal.json`, `news.json`, `passages.json`, and the three hand-curated files (`species_curated`, `places_curated`, `bibliography_curated` — straight copies from `data/curated/`).

**`documents.json` gained 0 rows, and that's the finding worth your attention.** It filters `WHERE relevance_tier IN ('A','B')`, and **all 2,533 documents in D1 have `relevance_tier` NULL** — harvest-engine never assigns one. So all 1,839 newly inserted documents are in atlas.db but invisible to the site. The file still changed because 690 pre-existing documents got enriched (205 abstracts, 130 OA links, 92 licences filled into previously blank fields). The 113 A + 29 B harvested documents that *do* have tiers are adopted rows that already had them from atlas. Assigning tiers to the new 1,839 is a separate job — either upstream in harvest-engine or as a triage pass.

## 2. The staleness check

**There isn't one — I need to correct the premise, which my own wording last turn probably created.** I said "`export_from_db.py`'s own header warns against" drift; that's prose in the module docstring (line 15: "so the two never drift apart"), not code. Grepping for `stale|drift|mtime|assert|sys.exit` finds nothing programmatic; the only `WARNING` in the script fires when a curated source file is missing.

The closest real drift indicator is `manifest.json`, which records row counts at build time. That now agrees with the database on every entity:

| entity | manifest | atlas.db | match |
|---|---|---|---|
| place | 93 | 93 | yes |
| taxon | 2,664 | 2,664 | yes |
| occurrence | 78,467 | 78,467 | yes |
| document (A/B) | 928 | 928 | yes |
| claim | 83 | 83 | yes |
| source | 610 | 610 | yes |

Before regeneration it read 89 / 1,893 / 74,900 / 53 with `built_at_iso` of 2026-08-26 — stale by exactly the sync delta. Now `2026-09-06T03:08:19Z`.

## 3. Local site verified

Ran `scripts/devserve.py` on port 8765 and rendered pages in headless Chrome (`--dump-dom`), so these are values the browser actually computed from the fetched JSON, not assertions about the files:

- **`/land`** — `#placesHave` rendered **93** (was 89). All four newly-synced places appear in the rendered gazetteer DOM: Sathyamangalam, Kembainaickenpalayam, Kolathur, Periyakodiveri — with their Tamil names.
- **`/about`** — `#aboutCovPct` rendered **32.5%** (was 31.6%).
- `/exports/species.json` served HTTP 200 at 724,360 bytes (the new size).

Server stopped afterwards. The background task's "exit code 144" is my own `pkill`, not a crash.

## 4. Nothing committed, pushed, merged, or deployed

- Still on `dev`. All 8 modified files (`data/atlas.db` + 7 exports) are **uncommitted** in the working tree.
- Commits still `345e078` / `fae5c9b` from last turn — no new commits.
- `[ahead 2]` — nothing pushed to `origin`.
- `main` still at `025be91`, unchanged; `dev` not merged into it.
- No `wrangler deploy` / `pages deploy`. `dist/` untouched — still dated Aug 26, so nothing was even built for deploy.

One thing to flag for whenever you do commit: `data/atlas.db` is a tracked binary that grew 26 MB → 37.8 MB, and it holds exact sensitive-species coordinates. Committing it writes those into git history permanently on a branch that may later reach `main`. Still your call, not something I'll do without you saying so.

**Vishnu** (2026-09-06T03:41): Commit the current working-tree changes to dev — nothing else.

Files to commit: data/atlas.db (the post-sync database, now with exact sensitive-species coordinates per the standing dev/staging policy) and the 7 regenerated exports/ files (species.json, documents.json, places.json, places_display.json, places.geojson, coverage.json, manifest.json).

Commit message should note: D1→atlas.db sync applied to dev, exports regenerated, coordinates left raw per the sensitive-species-coordinate-policy (dev/staging exempt, production still needs coarsening).

Stay on dev. Do NOT merge to main, do NOT push to origin, do NOT deploy anything.

Confirm after: git status clean on dev, no new commits pushed, main untouched.

**Claude** (2026-09-06T03:41): You've made the call on the coordinates explicitly — proceeding. Let me verify the exact file set first.

**Claude** (2026-09-06T03:41): _[tool: Bash]_

**Claude** (2026-09-06T03:41): Exactly the 8 files you named. Staging them explicitly by path so nothing else can be swept in:

**Claude** (2026-09-06T03:41): _[tool: Bash]_

**Claude** (2026-09-06T03:42): _[tool: Bash]_

**Claude** (2026-09-06T03:42): Committed. Now the confirmations:

**Claude** (2026-09-06T03:43): _[tool: Bash]_

**Claude** (2026-09-06T03:43): _[tool: Bash]_

**Claude** (2026-09-06T03:43): Committed as `fbb4584` on `dev`.

**Files** — exactly the 8 you named, staged by explicit path so nothing else could be swept in:

```
data/atlas.db               Bin 26,415,104 -> 37,801,984 bytes
exports/coverage.json
exports/documents.json
exports/manifest.json
exports/places.geojson
exports/places.json
exports/places_display.json
exports/species.json
```

The commit message records the sync (row deltas, 6,738 inserted / 1,283 adopted / 0 skipped), the exports regeneration, and the coordinate policy — raw per the dev/staging exemption, with the note that production still needs a coarsening gate that doesn't exist yet, plus the `taxon.sensitive = 1` query for finding the 13 affected rows later.

## Confirmations

**1. `git status` clean on dev** — no tracked modifications remain. Seven untracked paths still show (`docs/comparables/`, `docs/project/`, `docs/share/`, `main`, and three `scripts/` files) — those predate my work, aren't mine, and I left them alone.

**2. Nothing pushed** — `dev` is `[ahead 3]` of `origin/dev`, which is still at `178d8d1`. All three commits (`fbb4584`, `345e078`, `fae5c9b`) are local only.

**3. `main` untouched** — still `025be91`, unchanged since session start. `dev` is not merged into it, and `git branch --contains fbb4584` returns `dev` alone.

**4. Nothing deployed** — no `wrangler deploy` or `pages deploy`. `dist/` is still dated Aug 26, untouched.

One consequence worth naming now that it's real: `data/atlas.db` is in git history at 37.8 MB with exact sensitive-species coordinates. That's what you authorised, and it's fine on `dev` — but it means a later `dev → main` merge carries both the file size and those coordinates into `main`'s history, where removing them would need a history rewrite rather than a revert. The coarsening gate is the thing that needs to land before that merge, not after it.

**Vishnu** (2026-09-06T10:55): Start the local dev server on dev (scripts/devserve.py, same one used to verify earlier) on port 8765, and leave it running in the foreground/background so I can open it in my own browser — don't run it as a one-shot headless check this time.

Tell me the exact local URL to open (e.g. http://localhost:8765), and which pages are worth looking at to see the new data (/land for the gazetteer with the 4 new places, /about for the coverage % change).

Leave it running until I say to stop it.

**Claude** (2026-09-06T10:55): _[tool: Bash]_

**Claude** (2026-09-06T10:55): It binds to `127.0.0.1`. Starting it in the background so it persists:

**Claude** (2026-09-06T10:55): _[tool: Bash]_

**Claude** (2026-09-06T10:55): _[tool: Bash]_

**Claude** (2026-09-06T10:56): Server is up and running in the background. It'll stay up until you tell me to stop it.

## Open this

**http://localhost:8765**

(Bound to `127.0.0.1` only, so it's local to your machine — not reachable from the network.)

## Pages worth looking at

**http://localhost:8765/land** — the strongest place to see the sync. The gazetteer now lists **93 places, up from 89**. The four new ones from harvest-engine are in the undescribed list near the bottom, with their Tamil names:

- Sathyamangalam — சத்தியமங்கலம்
- Kembainaickenpalayam — கேம்பைநாயக்கன்பாளையம்
- Kolathur — கொளத்தூர்
- Periyakodiveri — பெரியகோடி ஏரி

The "places" coverage figure on that page reads **93**. They have coordinates, so they should appear on the map layer too.

**http://localhost:8765/about** — coverage now reads **32.5%** (was 31.6%). This page also links the raw JSON, so you can click straight through to `manifest.json` and see the new counts (place 93, taxon 2,664, occurrence 78,467, source 610).

**http://localhost:8765/exports/manifest.json** — the fastest single check that the exports match the database.

## Two pages that will look unchanged, so you're not hunting for a bug

- **`/life`** reads `species_curated.json`, a hand-maintained editorial file that the sync doesn't touch — so it won't reflect the +771 taxa. The machine-readable `species.json` did grow (512 KB → 724 KB), but the life page doesn't read it.
- **`/history`** reads `documents.json`, which filters `relevance_tier IN ('A','B')`. All 2,533 D1 documents have a NULL tier, so none of the 1,839 new documents surface there. They're in the database, just not reachable by the site yet.

## Still holding

Nothing pushed, nothing merged, nothing deployed — `dev` is `[ahead 3]` of `origin/dev`, `main` still at `025be91`. Say the word and I'll stop the server.

**Vishnu** (2026-09-06T10:57): I opened the local dev server but don't see any new data on the site. Verify end-to-end what's actually different and why I might not be seeing it. Check, in order:

1. Is the dev server actually serving from the current exports/ directory, or from a cached/stale build (dist/, a CDN cache, browser cache)? Check file timestamps of what's actually being served vs. exports/manifest.json's built_at_iso (should be 2026-09-06T03:08:19Z).

2. Which pages actually display the new data? Earlier verification only checked /land (places count 89→93) and /about (coverage 31.6%→32.5%) — list every page/component that reads from species.json, documents.json, places.json etc., and confirm which ones a person would need to visit to see any difference. If most of the new data (2,664 taxa, 78,467 occurrences) doesn't have a page/view that surfaces it, say so plainly — don't assume all synced data is visible somewhere.

3. Give me exact instructions: which URL, which page, and what specific number or item to look for that wasn't there before, so I can verify with my own eyes.

4. If the browser is just caching the old page, tell me exactly how to hard-refresh or clear it for this local server.

Confirm server is still running and give me the URL again.

**Claude** (2026-09-06T10:58): Let me verify this empirically rather than assume. Starting with what the server is actually serving.

**Claude** (2026-09-06T10:58): _[tool: Bash]_

**Claude** (2026-09-06T10:58): Server is serving the correct fresh manifest. Now let me test the actual cache mechanism and rule out `dist/`.

**Claude** (2026-09-06T10:58): _[tool: Bash]_

**Claude** (2026-09-06T10:58): Two things ruled out: `dist/` isn't served at all (404), and there's no CDN on localhost. But the server **does** return `304` on conditional requests, so browser cache is a live possibility.

Now the important question — what visibly differs. Let me build a true before/after by serving the pre-sync exports side by side.

**Claude** (2026-09-06T10:58): _[tool: Bash]_

**Claude** (2026-09-06T10:58): _[tool: Bash]_

**Claude** (2026-09-06T10:58): _[tool: Bash]_

**Claude** (2026-09-06T10:59): Both servers up with different data. Now let me render every page on both and diff the visible text.

**Claude** (2026-09-06T10:59): _[tool: Bash]_

**Claude** (2026-09-06T11:00): Definitive answer. Let me get the exact labels around those numbers so I can give you precise things to look for.

**Claude** (2026-09-06T11:00): _[tool: Bash]_

**Claude** (2026-09-06T11:00): _[tool: Bash]_

**Claude** (2026-09-06T11:00): Two things in that output need checking — the `/record` page still shows "1,893 taxa … 94.7%" *after* the sync, and text claiming coordinates are coarsened. Let me look at the source.

**Claude** (2026-09-06T11:00): _[tool: Bash]_

**Claude** (2026-09-06T11:01): _[tool: Bash]_

**Claude** (2026-09-06T11:01): _[tool: Bash]_

**Claude** (2026-09-06T11:01): Let me shut down the temporary comparison server so you can't accidentally open it and see old data.

**Claude** (2026-09-06T11:01): _[tool: Bash]_

**Claude** (2026-09-06T11:02): Server on **8765 is still running** and serving the fresh data. Here's what I found.

## 1. Not a stale build or cache on the server side

- The server is serving `built_at_iso: 2026-09-06T03:08:19Z` — the correct post-sync build. Verified over HTTP, not from disk.
- **`dist/` is not served at all** — `/dist/about` returns 404. `devserve.py` serves `site/` as web root and `exports/` from the repo root; `dist/` is never touched. (For the record it *is* stale — `dist/exports/manifest.json` still says `2026-08-26T16:11:58Z` — but it's irrelevant to what you're looking at.)
- No CDN — it's localhost.
- **But the server does return `304 Not Modified`** on conditional requests. It sends `Last-Modified` with no `Cache-Control`/`ETag`, so your browser can legitimately reuse cached JSON. That's the one plausible cause on your side.

## 2. What actually changes — measured, not assumed

I served the pre-sync exports on a second port and diffed the rendered text of every page in a real browser. Verdict:

| page | changes? | what differs |
|---|---|---|
| `/record` | **yes — richest** | Places 89→**93**, Taxa 1,893→**2,664**, Occurrences 74,900→**78,467**, coverage 31.6%→**32.5%** |
| `/land` | **yes** | 4 new settlements; 89→**93**; 7.4%→**7.8%**; type-bar 13→**17** |
| `/about` | yes | coverage only: 31.6% → **32.5%** |
| `/` | yes | coverage only: 31.6% → **32.5%** |
| `/places` | yes | it's a redirect to `/land#gazetteer` |
| `/life` | **no** | identical |
| `/history` | **no** | identical |
| `/govern`, `/people`, `/visit` | **no** | identical |

**Plainly: most of the synced data has no view that surfaces it.** The 771 new taxa and 3,567 new occurrences appear *only as aggregate counts* on `/record`. There is no page listing them:

- `/life` reads `species_curated.json`, a hand-maintained editorial file of 76 species. It never reads `species.json`, so the 2,664 taxa are invisible there.
- `/history` reads `documents.json`, filtered to `relevance_tier IN ('A','B')`. All 2,533 D1 documents have NULL tier, so **none** of the 1,839 new documents appear.
- No page reads occurrence rows individually — only the count via `coverage.json`.

So the four new *places* are the only individual new records a person can see anywhere on the site.

## 3. Exact things to look for

**Best single check — http://localhost:8765/record**

Scroll to the coverage table. You should see:

```
Places        93     1,200    7.8%
Taxa       2,664     2,000     100%
Occurrences 78,467  30,000     100%
```

and below it: `Overall coverage: 32.5%`. Before, those read 89 / 1,893 / 74,900 / 31.6%.

**http://localhost:8765/land** — scroll to the gazetteer. Four names that did not exist before:

- **Kolathur** — Settlement
- **Kembainaickenpalayam** — Settlement
- **Periyakodiveri** — Settlement
- **Sathyamangalam** — Settlement

and the sentence "the atlas backend currently holds **93** named places … about **7.8%** complete."

## ⚠️ Two stale hardcoded claims on `/record` you should know about

While verifying I found `site/record.html` has numbers baked into the HTML that the exports don't drive, so the page now contradicts itself:

- **Line 85**: "1,893 taxa recorded — 94.7% of the 2,000-taxon target" — sits a few centimetres from the table saying 2,664.
- **Line 80**: "74,180 georeferenced GBIF records, one citable download, **sensitive taxa coarsened to a 0.05° grid**."

That second one matters more than the number drift: **the site now states coordinates are coarsened, and for this database that is false** — we deliberately wrote raw coordinates. (It also says 0.05°; the real grid is 0.045°.) Contained on dev, but it's a factual claim the page makes about the data, and it must not survive into production alongside the coarsening gate. I haven't touched it — it's editorial copy and your call.

## 4. Clearing the browser cache for this server

- **macOS Chrome/Edge**: **⌘⇧R** — or open DevTools (**⌘⌥I**), then right-click the reload button → *Empty Cache and Hard Reload*.
- **Safari**: **⌘⌥E** (empty caches), then **⌘R**. Needs *Develop* menu enabled in Settings → Advanced.
- **Firefox**: **⌘⇧R**.
- **Most reliable** — open DevTools, Network tab, tick **Disable cache**, and leave DevTools open while you browse.
- **Zero-cache sanity check** — a private/incognito window at http://localhost:8765/record.
- If you want to bypass it entirely without touching settings, add a query string: **http://localhost:8765/record?v=2**.

Quickest way to tell whether you're on fresh data at all: open **http://localhost:8765/exports/manifest.json** — it should read `"built_at_iso": "2026-09-06T03:08:19Z"` and `"taxon": 2664`. If it says `2026-08-26` you're on cache.

## Server

**http://localhost:8765** — still running, bound to `127.0.0.1` only. I stopped the temporary comparison server on port 8766 so you can't land on it by accident and see the old numbers; its "exit code 144" notification is that shutdown, not a crash.

Still on `dev`, nothing pushed, `main` untouched at `025be91`.

**Vishnu** (2026-09-06T11:06): The user is unsatisfied — only a summary number changed (place count 89→93), no actual content is visibly new or different on the site. Find out exactly what a visitor sees for the 4 new places and confirm whether this is a real content gap or just a rendering gap.

1. Pull the full record for the 4 newly-synced places (Sathyamangalam, Kembainaickenpalayam, Kolathur, Periyakodiveri) from places.json — every field, not just name. Compare field-by-field against an existing, pre-sync place record (e.g. one of the original 89) to see what's actually populated vs blank.

2. Does each new place have its own detail page/URL, or do they only appear in a list/count with no page to click into? If there's no per-place page, say that plainly.

3. For the new taxa (771 of them) and new occurrences (3,567): is there ANY page on the dev site where a visitor can browse or search individual species/occurrence records, or does the site currently only show aggregate counts and pre-curated content (the species_curated.json / places_curated.json files mentioned earlier)? Check whether the site's actual pages read from the raw synced data at all, or only from the hand-curated files.

4. If the honest answer is "the synced data has no page that displays it yet, only the total counts changed" — say that directly. Don't reframe a real content gap as a caching or rendering issue if it isn't one.

5. If there IS a display gap (data exists, is correct, but the frontend was never built to show individual records — only curated/summary content), describe exactly what page/component would need to be built to actually surface this data, and how much work that is.

Give me a plain answer: is the new data genuinely visible anywhere, or did we just move numbers into a database with no way for a person to see the actual content yet?

**Claude** (2026-09-06T11:06): Let me get the actual data rather than characterise it.

**Claude** (2026-09-06T11:06): _[tool: Bash]_

**Claude** (2026-09-06T11:06): _[tool: Bash]_

**Claude** (2026-09-06T11:06): _[tool: Bash]_

**Claude** (2026-09-06T11:07): Found something important. Let me check that `Sathyamangalam` / `Satyamangalam` pair — they share a Wikidata ID.

**Claude** (2026-09-06T11:07): _[tool: Bash]_

**Claude** (2026-09-06T11:07): One of the four is a duplicate. Now the key question — which pages actually **fetch** these files versus merely link to them as downloads.

**Claude** (2026-09-06T11:07): _[tool: Bash]_

**Claude** (2026-09-06T11:07): `species.json` is never fetched — only linked as a download. Let me confirm the occurrence situation and whether gazetteer entries are clickable.

**Claude** (2026-09-06T11:07): _[tool: Bash]_

**Claude** (2026-09-06T11:08): _[tool: Bash]_

**Claude** (2026-09-06T11:08): Let me verify that by slug — `places.json` has 93 rows but the map layer has only 89 features, so something doesn't add up.

**Claude** (2026-09-06T11:08): _[tool: Bash]_

**Claude** (2026-09-06T11:08): _[tool: Bash]_

**Claude** (2026-09-06T11:08): _[tool: Bash]_

**Claude** (2026-09-06T11:09): There's a site-wide search palette. Let me check what it actually indexes — this could matter a lot.

**Claude** (2026-09-06T11:09): _[tool: Bash]_

**Claude** (2026-09-06T11:09): _[tool: Bash]_

**Claude** (2026-09-06T11:09): _[tool: Bash]_

**Claude** (2026-09-06T11:09): The palette lives in `site/shared.js`. Let me check what corpus it searches.

**Claude** (2026-09-06T11:09): _[tool: Bash]_

**Claude** (2026-09-06T11:09): _[tool: Bash]_

**Claude** (2026-09-06T11:09): _[tool: Bash]_

**Claude** (2026-09-06T11:09): _[tool: Bash]_

**Claude** (2026-09-06T11:10): Let me get the concrete details needed to scope the work.

**Claude** (2026-09-06T11:10): _[tool: Bash]_

**Claude** (2026-09-06T11:10): Server still running at **http://localhost:8765**. Here's the plain answer.

## The plain answer

**It's a real content/display gap, not caching and not a rendering bug.** The data landed correctly and is verifiably in the database. But the frontend was never built to display individual synced records. Apart from four place names in one list, a visitor sees only totals.

## 1. The 4 new places — full records

| field | the 4 new | 89 originals |
|---|---|---|
| slug, name, type, tier, source_id | 4/4 | 89/89 |
| lat, lon | 4/4 | 85/89 |
| **name_ta** (Tamil) | **4/4** | 14/89 |
| **wikidata** | **4/4** | 8/89 |
| osm_id | **0/4** | 72/89 |
| range, division | 0/4 | 4/89 |
| population | 0/4 | 1/89 |
| elevation_m, census_code | 0/4 | **0/89** |

Example: `{"slug":"kolathur","name":"Kolathur","name_ta":"கொளத்தூர்","type":"settlement","lat":11.51,"lon":77.45,"tier":"core","wikidata":"Q2443959","source_id":556}` — with `range`, `division`, `elevation_m`, `population`, `osm_id`, `census_code` all null.

They're not impoverished versions of the originals — they're *better* on Tamil names and Wikidata IDs, worse on OSM IDs. The decisive missing field isn't in this table at all: **`description`**. That only exists in `places_curated.json` (15 hand-written entries). No description → the place falls into the "undescribed" bucket, which renders as inert text.

**⚠️ And one of the four is a duplicate.** New `Sathyamangalam` and pre-existing `Satyamangalam` both carry Wikidata **Q242563** — the same town, differently transliterated. My natural key matched on `slug OR osm_id`; the new row had no `osm_id` and a different slug spelling, so it inserted rather than matched. **`wikidata_id` should be in the match key.** So it's **3 genuinely new places, not 4.**

## 2. Per-place pages: none exist

Not for the new places, not for any place. The site is 13 static HTML files with no dynamic routing and no per-record template. What a visitor literally gets for Kolathur:

```html
<span class="gaz-plain-item">Kolathur <i>Settlement</i></span>
```

No `<a href>`, no `onclick` — confirmed in the rendered DOM. The 15 curated places do render as clickable cards, but clicking only flies the map; they have no pages either.

The one thing that *is* genuinely new and visual: all 4 are in `places.geojson`, so they appear as **map pins** on `/land` (85 → 89 features).

## 3. Do any pages read the raw synced data?

I separated actual `fetch()` calls from mere download links:

| page | fetches | reads raw synced data? |
|---|---|---|
| `/land` | places.json, places_display.json, places_curated.json, coverage.json | **yes** — the 4 places |
| `/history` | documents.json | yes, but **tier-filtered → 0 new rows** |
| `/life` | **species_curated.json only** | **no** — 76 hand-curated species |
| `/record` | bibliography_curated.json, claims.json, coverage.json | no — aggregates only |
| `/about` | coverage.json | no |
| `/` | claims.json, coverage.json | no |

- **`species.json` is never fetched by any page.** It's a download `<a href>` on `/about`. The 771 new taxa (2,664 total) have no view.
- **There is no occurrences export at all.** No page reads occurrence records. The 3,567 new occurrences exist only inside `coverage.json`'s count.
- **The search palette doesn't index them either** — it's 11 page links plus 15 curated places, 76 curated species, 44 bibliography entries.

## 4. Direct statement

The sync moved 6,738 rows into a database with, for practical purposes, no way for a person to see the content:

- **3,567 occurrences** — no export, no page, count only.
- **771 taxa** — exported to `species.json`, fetched by nothing, count only.
- **1,839 documents** — in the DB, excluded from `documents.json` by the A/B tier filter (all D1 docs have NULL tier), count only.
- **3 places** (+1 duplicate) — the only new individual records visible anywhere: name + type as inert text, plus a map pin.

Earlier I told you `/record` was the "richest" page for seeing the sync. That was accurate about numbers changing and misleading about substance — those are five aggregate figures, not content. I should have led with the gap.

## 5. What would need building

**a. Species browser — smallest real win.** `species.json` already exists (2,664 rows, 782 KB) with `scientific_name, authority, rank, kingdom, class, common_en, common_ta, iucn, wpa, gbif_key, invasive, sensitive`. `/life` already has a search input (`#spSearch`) wired to the curated 76. Point a second section at `species.json`, virtualise the list. **Half a day.** Caveat: **1,851 of 2,664 taxa have no common name** — mostly fungi and invertebrates from GBIF — so it reads as a scientific-name index, not a field guide. Worth deciding whether that belongs on a public page or behind a "full taxonomic index" link.

**b. Documents.** Cheapest of all: the 1,839 rows are excluded solely by `relevance_tier IN ('A','B')` with all D1 tiers NULL. Either assign tiers (upstream in harvest-engine, or a triage pass) or add a tier-less section. **An hour** for the filter change; the tiering itself is the real work and is editorial.

**c. Occurrences — the big one.** Needs a new export in `export_from_db.py` first. 78,467 rows won't ship as one JSON, so it needs aggregation (per-taxon counts, per-grid-cell density) or per-taxon slicing, then a view. **Two to three days**, and it's the one where the production coarsening gate becomes load-bearing, since this is the data holding exact sensitive coordinates.

**d. Per-record detail pages.** Currently impossible without either a static-generation step (emit one HTML per place/taxon at build time) or client-side routing. **A day or two** for a generator, plus template design.

**Also worth fixing regardless:** add `wikidata_id` to the place match key so re-syncs stop creating transliteration duplicates, and de-dupe the existing Sathyamangalam/Satyamangalam pair. That's a ~20-line change to the sync plus a one-row merge — I'd suggest doing it before the post-bugfix re-harvest, or the duplicate class grows.

I haven't changed any of this — say which of a–d you want and I'll build it.

**Vishnu** (2026-09-06T11:12): On dev, in this order:

1. Fix the place sync's matching key to include wikidata_id, not just slug/osm_id, so re-syncs don't create transliteration duplicates. Then manually merge the existing Sathyamangalam/Satyamangalam duplicate (same Wikidata Q242563) into one record — keep whichever has more complete fields, note the merge in a commit message.

2. Build a basic species browser on /life: a second section reading species.json (2,664 rows), reusing the existing #spSearch search pattern already wired to the curated 76. Virtualize the list for performance. It's fine that ~1,851 of these have no common name — label that section clearly as a "full taxonomic index" separate from the curated species cards, so it doesn't read as a broken field guide.

3. Change documents.json's export filter so documents with a NULL relevance_tier are included in a separate "unreviewed" section, distinct from the curated A/B tier documents — don't just remove the filter and mix them together.

Test locally, confirm what's now visible with specifics (not just row counts — actual species names, actual document titles appearing on the rendered page). Stay on dev, no merge/deploy without asking.

**Claude** (2026-09-06T11:12): Starting with task 1. Let me look at the current place matching key.

**Claude** (2026-09-06T11:12): _[tool: Bash]_

**Claude** (2026-09-06T11:12): _[tool: Edit]_

**Claude** (2026-09-06T11:12): Before testing — I need to verify D1's place rows actually carry `wikidata_id`, or the fix does nothing.

**Claude** (2026-09-06T11:12): _[tool: Bash]_

**Claude** (2026-09-06T11:12): D1 carries `wikidata_id` for all four. Let me verify the fix against a pre-sync copy — the duplicate should no longer appear.

**Claude** (2026-09-06T11:12): _[tool: Bash]_

**Claude** (2026-09-06T11:13): Fix confirmed: 89 → 92 (3 new + 1 adopted onto the existing row), no duplicates. Now the live database still has the duplicate. Let me examine both rows and anything referencing them.

**Claude** (2026-09-06T11:13): _[tool: Bash]_

**Claude** (2026-09-06T11:13): _[tool: Bash]_

**Claude** (2026-09-06T11:13): Let me check the D1 side, then add a guard for provenance pointing at deleted rows before I do any merging.

**Claude** (2026-09-06T11:13): _[tool: Bash]_

**Claude** (2026-09-06T11:14): D1 itself contains the duplicate — both rows carry Q242563 from different streams. My fix collapses them. Now let me add a guard so provenance pointing at a manually-deleted row can't silently no-op.

**Claude** (2026-09-06T11:14): _[tool: Bash]_

**Claude** (2026-09-06T11:14): _[tool: Edit]_

**Claude** (2026-09-06T11:14): _[tool: Bash]_

**Claude** (2026-09-06T11:14): Now let me back up, do the merge, and re-sync to re-derive the mapping through the tested code path.

**Claude** (2026-09-06T11:14): _[tool: Bash]_

**Claude** (2026-09-06T11:14): Keeper is a strict superset. Executing the merge:

**Claude** (2026-09-06T11:14): _[tool: Bash]_

**Claude** (2026-09-06T11:15): _[tool: Bash]_

**Claude** (2026-09-06T11:15): Task 1 done. Let me regenerate exports and commit it before moving on.

**Claude** (2026-09-06T11:15): _[tool: Bash]_

**Claude** (2026-09-06T11:15): _[tool: Bash]_

**Claude** (2026-09-06T11:15): Task 1 committed. Now task 2 — the species browser. Let me study the existing `/life` structure.

**Claude** (2026-09-06T11:15): _[tool: Bash]_

**Claude** (2026-09-06T11:16): _[tool: Bash]_

**Claude** (2026-09-06T11:16): _[tool: Bash]_

**Claude** (2026-09-06T11:16): Let me check for a style block and whether anything references section numbers.

**Claude** (2026-09-06T11:16): _[tool: Bash]_

**Claude** (2026-09-06T11:16): _[tool: Bash]_

**Claude** (2026-09-06T11:17): Let me check the actual value shapes in species.json before writing the renderer.

**Claude** (2026-09-06T11:17): _[tool: Bash]_

**Claude** (2026-09-06T11:17): Now let me add the CSS:

**Claude** (2026-09-06T11:17): _[tool: Bash]_

**Claude** (2026-09-06T11:17): Now the HTML section, inserted after the Species Atlas:

**Claude** (2026-09-06T11:17): _[tool: Bash]_

**Claude** (2026-09-06T11:17): Now the virtualized renderer:

**Claude** (2026-09-06T11:18): _[tool: Bash]_

**Claude** (2026-09-06T11:18): Let me test it in the browser before moving on.

**Claude** (2026-09-06T11:18): _[tool: Bash]_

**Claude** (2026-09-06T11:18): Virtualization confirmed — 19 DOM nodes for 2,664 rows. Let me verify actual species names render and test the filters.

**Claude** (2026-09-06T11:18): _[tool: Bash]_

**Claude** (2026-09-06T11:18): Real names render. But I spot an upstream data problem — let me quantify it.

**Claude** (2026-09-06T11:19): _[tool: Bash]_

**Claude** (2026-09-06T11:19): _[tool: Bash]_

**Claude** (2026-09-06T11:19): Filters and Tamil search work. Let me confirm where those 420 duplicates come from.

**Claude** (2026-09-06T11:19): _[tool: Bash]_

**Claude** (2026-09-06T11:19): Significant: **411 of the 771 new taxa are authority-suffixed duplicates** (`Acridotheres tristis (Linnaeus, 1766)` vs `Acridotheres tristis`). Since users will see these while scrolling, the page should say so.

**Claude** (2026-09-06T11:19): _[tool: Bash]_

**Claude** (2026-09-06T11:19): Task 2 done. Now task 3 — the documents export filter.

**Claude** (2026-09-06T11:19): _[tool: Bash]_

**Claude** (2026-09-06T11:20): `history.html` consumes `documents.json` as a flat array, so reshaping it would break the decade chart. Let me add a separate export instead — same intent, no breakage.

**Claude** (2026-09-06T11:20): _[tool: Bash]_

**Claude** (2026-09-06T11:20): _[tool: Bash]_

**Claude** (2026-09-06T11:20): This vindicates keeping them separate — "Customer Satisfaction in SK Traders" and "Financial Performance of Bannari Amman Spinning Mills" matched on local place names, not subject. Now let me surface them on `/record`.

**Claude** (2026-09-06T11:20): _[tool: Bash]_

**Claude** (2026-09-06T11:21): _[tool: Bash]_

**Claude** (2026-09-06T11:21): _[tool: Bash]_

**Claude** (2026-09-06T11:21): Now the renderer for the unreviewed list:

**Claude** (2026-09-06T11:21): _[tool: Bash]_

**Claude** (2026-09-06T11:21): Now let me test both pages in the browser.

**Claude** (2026-09-06T11:21): _[tool: Bash]_

**Claude** (2026-09-06T11:22): Both render. One display issue — some titles carry embedded markup from Crossref. Let me quantify.

**Claude** (2026-09-06T11:22): _[tool: Bash]_

**Claude** (2026-09-06T11:22): Worth fixing at the display layer. Let me add title cleanup:

**Claude** (2026-09-06T11:22): _[tool: Bash]_

**Claude** (2026-09-06T11:22): Titles clean. Let me verify nothing broke across the whole site.

**Claude** (2026-09-06T11:22): _[tool: Bash]_

**Claude** (2026-09-06T11:23): _[tool: Bash]_

**Claude** (2026-09-06T11:23): All render. The emptynote flags are likely matching the JS source strings rather than actual error states — let me confirm.

**Claude** (2026-09-06T11:23): _[tool: Bash]_

**Claude** (2026-09-06T11:24): Zero real error states. Let me confirm `/history` still works off the unchanged `documents.json`, then commit.

**Claude** (2026-09-06T11:24): _[tool: Bash]_

**Claude** (2026-09-06T11:24): `/history` unaffected (921 dated docs, 11 decade bars). Committing tasks 2 and 3:

**Claude** (2026-09-06T11:24): _[tool: Bash]_

**Claude** (2026-09-06T11:24): _[tool: Bash]_

**Claude** (2026-09-06T11:25): All three done, tested in a real browser, committed on `dev`. Server still at **http://localhost:8765**.

## 1. Place matching fixed, duplicate merged — `e9490d8`

Match key is now **`wikidata_id` → `osm_id` → `slug`** (was `slug OR osm_id`). The fingerprint follows the same precedence so a D1 rebuild still re-identifies rows.

Verified against a pre-sync copy: place goes **89 → 92** instead of 93, and no `wikidata_id` appears on more than one row.

**Merge:** kept place 50 `satyamangalam`, deleted place 90 `sathyamangalam`. 50 is a strict superset — `osm_id node/4612606679`, `population 10000`, `type 'town'` not `'settlement'`, 7-decimal coordinates, same Tamil name — so nothing needed carrying over. Both D1 rows now map to atlas place 50. **place: 93 → 92.**

The duplicate is upstream too: D1 itself holds place 1 (`sathyamangalam`, no osm_id) and place 36 (`satyamangalam`, with osm_id), both `Q242563`, written by two different gazetteer streams. My fix collapses them; harvest-engine should also dedupe on `wikidata_id`.

I also added a guard this merge would otherwise have needed: if a provenance row points at a deleted atlas row, the mapping is dropped and the row re-identified. Previously the UPDATE would hit a missing id, succeed as a no-op, and the row would silently vanish on every later sync.

## 2. Species browser on `/life` — section 03, "Full Taxonomic Index"

Go to **http://localhost:8765/life#taxindex**. Rendered values confirmed in headless Chrome:

- **2,664** taxa, **1,851** flagged as having no common name
- **19 `.tx-row` nodes in the DOM** for 2,664 rows — virtualization works; sizer height 117,216px
- The curated 76 cards in section 02 are untouched (`#spCount` still 76)

Actual names on screen, alphabetical: *Abisara bifasciata* (Double-banded Judy), *Abrus precatorius* (rosary pea), *Absidia anomala*, *Absidia cuneospora*, *Abutilon hirtum* (Florida Keys Indian mallow).

Filters tested against the real data: All 2,664 · Has common name 813 · IUCN listed 82 · Sensitive 20 · Invasive 6. Search: `panthera` → 4 (*P. tigris tigris* EN, *P. pardus fusca* NT, *P. pardus*, *P. tigris* EN), `aves` → 348, **`புலி` → 2** (Tamil works).

**One thing you should see and know about:** ~420 of these entries are the same organism listed twice — *Acridotheres tristis* and *Acridotheres tristis (Linnaeus, 1766)*. **411 of the 771 taxa this sync inserted are such duplicates.** The harvester stores GBIF's `scientificName`, which sometimes carries authorship, instead of the canonical name. I put this in the section note so the 2,664 figure isn't read as a species count, but I did **not** fix it — reliable authorship-stripping is a harvest-engine change, not a display one. Your call whether to fix upstream before the re-harvest.

## 3. Unreviewed documents — section 03 on `/record`

**http://localhost:8765/record#unreviewed** — **1,839** documents, `#dxCount` 1,839, 19 DOM nodes.

Actual titles rendering: *"Angiospermic Diversity of Gobichettipalayam Taluk"*, *"A Study on Ecological and Socio-Economic Impacts of Neltuma Juliflora Invasion"*, *"A comprehensive assessment of fuel reformulation strategies in Delonix regia biodiesel"* — and, plainly, *"A Study on Customer Satisfaction in SK Traders, Gobichettipalayam"* and *"Financial Performance Analysis of Bannari Amman Spinning Mills"*. Those last two are the harvester matching on place names, and they're the argument for the separation you asked for. The section says so on the page.

**On the export shape:** I did *not* reshape `documents.json`. `history.html` reads it as a flat array for its by-decade chart, so `{reviewed, unreviewed}` would have broken it. Instead `documents.json` keeps its exact shape and A/B filter (928 rows), and the NULL-tier rows go to a new **`exports/documents_unreviewed.json`** (691 KB) — same separation, no risk to consumers. If you'd rather have the nested shape, it's a small change plus a fix to `history.html`. `abstract` is omitted from the unreviewed export; including it roughly triples the payload for a list meant for scanning titles.

Verified `/history` unaffected: `#docsHave` 921, 11 decade bars.

## Site health

All 11 pages render; **zero real error states** (the earlier "emptynote" hits were matching JS source strings, not rendered elements — confirmed by stripping `<script>` blocks first).

## Status

`dev`, working tree clean, **`[ahead 5]` — nothing pushed**. `main` untouched at `025be91`. No deploy.

Two stale claims I found but left alone, since they're editorial copy and now factually wrong given the raw-coordinate decision:
- `record.html:80` — "74,180 georeferenced GBIF records… sensitive taxa coarsened to a 0.05° grid"
- `life.html` section 02 — "1,839 taxa… 92% of the 2,000-taxon target… all 385 sensitive records verified coarsened"

Both assert coarsening that this database deliberately doesn't have. Worth correcting before anything reaches production alongside the masking gate — tell me and I'll do it.

**Vishnu** (2026-09-06T11:27): save all stop server

**Claude** (2026-09-06T11:27): _[tool: Bash]_

**Claude** (2026-09-06T11:27): _[tool: Bash]_

**Claude** (2026-09-06T11:28): Both ports are down, but 2 processes still match. Let me check what they are.

**Claude** (2026-09-06T11:28): _[tool: Bash]_

**Claude** (2026-09-06T11:28): _[tool: Bash]_

**Claude** (2026-09-06T11:28): Both done.

## Saved

Nothing needed saving — everything was already committed. Working tree is clean on `dev`, five commits ahead of `origin/dev`:

```
ece82ab  feat(site): surface the synced data — taxonomic index and unreviewed harvest
e9490d8  fix(sync): match places on wikidata_id, and merge the duplicate it let through
fbb4584  data: apply the D1 -> atlas.db sync to dev, regenerate exports
345e078  feat(sync): add --raw-coordinates for dev/staging databases
fae5c9b  feat(sync): build the D1 -> atlas.db sync stage that never existed
```

## Server stopped

Ports 8765 and 8766 both closed, no `devserve` processes remain, nothing listening. The earlier "exit code 144" notification is my own `pkill`, not a fault. (My first check reported "count: 2" — that was the counting pipeline matching itself; `pgrep` and `lsof` both confirm nothing is left.)

## Where things stand

- On `dev`. Nothing pushed, nothing merged — `main` still at `025be91`. Nothing deployed.
- Two rollback backups in `data/db-backups/` (gitignored): `atlas-20260906-082616-pre-d1-sync.db` and `atlas-20260906-164448-pre-place-merge.db`.
- To bring the site back up: `python3 scripts/devserve.py 8765`.

Open items from this session, none of them blocking, all still yours to decide:

1. **Production coarsening gate** doesn't exist. `data/atlas.db` holds exact sensitive-species coordinates and is now in git history on `dev` — that needs the gate before any merge to `main`.
2. **Two stale copy claims** asserting coarsening that this database doesn't have — `record.html:80` and `life.html` section 02.
3. **411 duplicate taxa** from harvest-engine storing GBIF's `scientificName` with authorship instead of the canonical name — disclosed on the page, unfixed upstream.
4. **1,839 documents have no relevance tier** — now browsable on `/record`, but untriaged.
5. **harvest-engine's dedup bug** and its binomial-vs-trinomial sensitive-species gap, both still unfixed; the re-harvest and re-sync wait on those.