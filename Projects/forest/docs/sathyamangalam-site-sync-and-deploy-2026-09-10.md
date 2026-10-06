# Site sync + deploy — 10 Sep 2026 — DONE

Closed the loop Vishnu asked about: does fixing the harvest engine actually change the live website? Answer was no by default — found and fixed the real gap.

## The pipeline (documented for future reference)
```
Harvest engine (Cloudflare Worker, runs 3x/day)
    -> writes to harvest-engine-db (D1)
    -> scripts/sync_d1_to_atlas.py (MANUAL, not scheduled)
    -> data/atlas.db (single source of truth for the site)
    -> make export (scripts/export_from_db.py -> exports/*.json)
    -> make build (scripts/build_dist.py -> dist/)
    -> make deploy (wrangler pages deploy -> sathyamangalam.online, Cloudflare Pages, direct upload, NOT git-connected)
```
No step from D1 onward is automated. Someone has to run this manually after every harvest-engine change that should reach the live site.

## What was done
1. Ran `sync_d1_to_atlas.py --extract` then load (live) — pulled all current D1 data into atlas.db. First run: 6,744 inserted, 167 updated, 1,283 adopted (existing rows only had blanks filled, nothing curated overwritten), 0 skipped/rejected.
2. **Found a real bug**: sync script's ENTITY_TABLES never included `news_event` — meaning the mongabay-india/toi-coimbatore/thehindu-tn streams fixed this week could collect articles into D1 forever and they'd NEVER reach the site. Fixed: added `sync_news_events()` function (same upsert/provenance pattern as `sync_documents`), added to ENTITY_TABLES, wired into run_load, added to the printed report. Field mapping: D1 `date/headline/outlet/url/summary_25w` -> atlas `dated/headline/outlet/url/summary`, natural key is `url` (UNIQUE in atlas.news_event). Syntax-checked (`py_compile`) before running.
3. Re-ran sync with the fix: 2 news_event rows now flow through correctly (small number — streams were only fixed hours earlier).
4. **Found a second real issue, not yet fixed**: 13 occurrence rows (5 elephant, 4 vulture, 2 sloth bear, 2 leopard) arrived from D1 with EXACT coordinates for sensitive species instead of coarsened ones. The sync script's own safety net (SensitivityOracle) caught and blurred them before writing to atlas.db, so nothing leaked publicly — but this means harvest-engine's own coordinate-coarsening has a live gap for these specific species right now, not just historically. Needs investigation: why did D1's own trigger not coarsen these before the sync script had to intervene as backup.
5. `make export`, `make check` (all JSON valid), `make build` (2,911 files, 58.3 MB, coverage 32.5%).
6. `make deploy` — live, verified via `curl https://sathyamangalam.online/exports/coverage.json` showing the fresh coverage stats and `news_event: 2`.

## Site coverage snapshot, 10 Sep 2026
Overall 32.5%. Strong: taxa (100%, 2,664 species), occurrences (100%, 78,467 records). Weak: places (7.7%), historical (0.2%), legal (1.7%), news (0.2%, expected — just started), layers/media/community (0%, not yet built).

## New issue found, not yet fixed: 3 data conflicts live on the site
`coverage.json` surfaces contradicting values for the same fact, both currently "on file":
- tigers: 112 vs 8-10
- elephants: 350-450 vs 651
- leopards: 111 vs ~20

These are shown to visitors as unresolved. Not caused by today's work — pre-existing, just newly visible in the refreshed coverage data. Needs Vishnu's editorial judgment on which source is correct, or a note explaining the discrepancy.

## Also carried forward, unfixed
The 13-occurrence coordinate-coarsening gap (item 4 above) — this is upstream in harvest-engine itself (likely in `src/lib/` sensitivity logic or one of the streams writing occurrences directly rather than through the shared sensitivity check), separate from the species-name-matching fix already shipped this week for GBIF's binomial/trinomial shapes.

## Next
- Decide on the 3 data conflicts (tiger/elephant/leopard counts).
- Investigate the coordinate-coarsening gap for the 13 sensitive occurrences.
- Continue as planned: register free OpenAlex API key, build the new CORE stream.
- Remember: this whole sync->export->build->deploy sequence needs to be re-run manually any time you want new harvest-engine data to reach the live site — it does not happen on its own.
