# Open items — flagged for future sessions

_Last updated: 2026-09-21. **The ops console is deployed and live.** The observation pause is closed, D-71 is fixed and pushed, D-70 is fixed. `india-ops-console` pushed to GitHub (`main` branch). This is the single "what's left" list — check here first before starting new work._

Plain-language article covering the whole platform: https://claude.ai/code/artifact/b6ba74f3-a47a-45b0-9cc6-798d179bf957

## Ops console — DEPLOYED (2026-09-21)

**Live at `https://ops.vidivu.in`.** The one thing blocking everything else is now done.

- Migrations 0019, 0020, 0021 applied to the live database.
- `ops_console` DB role created (read-only, password set, stored in VPS `.env`).
- Console container built and running (`docker compose up -d`), healthy, connected to Postgres.
- Metrics collector installed and running (`ops-metrics.timer`).
- **Public access set up:** subdomain `ops.vidivu.in` added in Cloudflare DNS (proxied), SSL mode set to Full (Strict), Cloudflare Origin Certificate installed on the VPS at `/etc/nginx/cloudflare/`. Nginx reverse-proxies `443` → `127.0.0.1:8010`, with **HTTP Basic Auth** (`/etc/nginx/cloudflare/.htpasswd`, user `admin`) as a second login layer in front of the console's own Argon2id login. Firewall opened on 80/443.
- **Known gap:** `CONTROL_TOKEN` was left empty on purpose. The console runs **read-only** — "trigger a run" / "rotate key" buttons are present but disabled. Wiring up `ops-control` (the host-side allowlisted action service) is a follow-up, not urgent, since read-only was the safer first deploy.
- Not yet done: pin the base image (`./scripts/pin-base-image.sh`), external review of `control/control_service.py`, deploy history (section 8 second half).
- The private local-only path (`ssh -L 8010:127.0.0.1:8010 ...`) still works as a fallback if the public route ever needs to be pulled.

## geo-peaks job — known flaky (Wikidata-side)

`geo-peaks.service` failed twice in a row on 2026-09-20/21 — once with a Wikidata `500`, once with a 180s read timeout. Root cause: the SPARQL query walks `wdt:P31/wdt:P279*` (full subclass tree) before filtering to India, which is expensive on Wikidata's end and inconsistent day to day — not a bug in our code. Not retried a third time back-to-back on purpose (avoid hammering their endpoint).

**Recommended fix, not yet done:** simplify `harvest_wikidata_peaks.py`'s query (filter to India first, or drop the `P279*` walk for a direct-type check) and add retry-with-backoff (3 attempts, 30/60/120s, only on 500/502/503/504/429). Needs `~/india-platform` file access to implement.

## Data snapshot (2026-09-21, from live DB)

20,552 entities, 68,597 facts, 14,572 taxa / 36,288 taxon-entity links (species is by far the largest contributor), 11,478 raw archive refs, 118 harvest runs, 1,672 rejections (~1.4%). `occurrence`, `timeseries_point`, `timeseries_series`, `entity_relation` all still 0 rows — features built but unused (matches item 13 below, metrics trend graphs).

**Discussed use cases for this data**, ranked by fit: peak + species + protected-area overlay (strongest — cross-engine, well-populated, matches the platform's own ecotourism focus), tribal-rights/land-use overlay (FRA J&K + census ST + protected areas), conservation risk dashboard (forest desertification + wetlands + PA boundaries), regulatory change tracking (e-Gazette feed). None built yet — this was research only.

## The VPS is a second author — fetch before you work

**The single most useful operational finding of 2026-09-17.** The Mac was silently behind on some repos before, because earlier sessions pushed from the VPS to the same branches.

`git fetch` on the Mac before starting local work in any of these repos.

## Local repo layout (as of 2026-09-17)

All 10 repos live side by side under `~/india-platform/`:

```
~/india-platform/
  india-data-core/        india-data-platform/    india-ops-console/
  india-culture-engine/   india-extinct-engine/   india-forest-engine/
  india-geo-engine/       india-laws-engine/      india-species-engine/
  india-water-engine/
```

**Note for future sessions working through the device bridge:** the shell on the Mac has no git identity and no GitHub credentials — it cannot commit, fetch or push. Read-only inspection only, and use `git --no-optional-locks status`. Commits and pushes must be run by the user in Terminal.

## Needs the user directly — nobody else can do these

1. **Register 5 pending API keys** — WDPA, IUCN v4, GeoNames, OpenTopography, GFW. Blocks the most future engine work.
2. **Rotate `DATA_GOV_IN_API_KEY`** — covers 6 of 8 engines, so treat a leak as platform-wide.
3. **OSM vs WDPA decision** for Protected Areas geometry — parked on OSM, unresolved.
4. **India Code / Indian Kanoon build-or-not call** — blocks `laws-engine` beyond its one working e-Gazette job.
5. **Capacity call on remaining bulk datasets**, plus a retest of India-WRIS/MoTA from an actual Indian egress point (the VPS is in Oregon).

These are also rows in the console's own open-items checklist — now genuinely visible at https://ops.vidivu.in.

## Real dev work still to do

6. **Audit other engine repos for the D-71 `else 2` bug.** `ssh ubuntu@40.160.137.239 "grep -rn 'else 2' ~/*/scripts/*.py"`. Only india-culture-engine has been checked.
7. **Fix `geo-peaks` Wikidata query + retry logic** — see above, newly flagged 2026-09-21.
8. **Pin the console's base image.**
9. **Wire up `ops-control`** (`CONTROL_TOKEN`) so the console's action buttons work — currently read-only by design.
10. **External review of `control/control_service.py`** — ~300 lines, runs as root, reviewed only by its author.
11. **Build the public-facing website / API layer** using the data — research done 2026-09-21 (see Data snapshot above), nothing built.
12. **`india-extinct-engine`** — repo exists, never deployed, 0 jobs. Build-or-drop decision.
13. **`laws-engine`** — only e-Gazette running; the rest blocked on item 4.
14. **Deploy history** — the unbuilt half of plan section 8.
15. **Metrics trend graphs** — `ops_console.host_metric` exists but nothing writes to it.
16. **GitHub Organization move** — cosmetic, not urgent.

## D-70 and D-71 — both fixed

**D-71 (exit codes), fixed, committed to `india-culture-engine` and pushed.** `harvest_fra_jk.py` returned exit code 2 on a `partial` run. Partial now exits 0.

**Operational lesson worth keeping:** editing a script on the VPS is not enough. `docker compose build` is required after any hot-fix.

**D-70 (false stale alerts), fixed in console migration 0019.** Per-job cadence replaces the flat 36-hour window.

## Live job inventory (confirmed 2026-09-21)

16 engine jobs across 6 of 8 engines. All dailies ran within 24h; weeklies on schedule except geo-peaks (see above). `core-api-heartbeat` every 5 minutes; `core-pg-backup.timer` ran on schedule, 29G free disk (24% used).

| Engine | Jobs | Cadence |
|---|---|---|
| Mountains & Geography | geo-usgs, geo-overpass-peaks-passes, geo-passes-ranges, geo-peaks | daily + 3 weekly |
| Forests & Land | forest-desertification, forest-fsi, forest-worldcover, forest-wetlands | all monthly |
| Tribal & Culture | culture-glot, culture-census-st, culture-fra-jk | all monthly |
| Protected Areas | pa-harvest | daily |
| Water Systems | water-nwdp | daily |
| Living Species | species-gbif, species-powo | weekly |
| Laws & Management | laws-egazette | daily |
| Extinct Species | none | — |

## What's confirmed done (do not re-do)

- VPS access root-caused: login is `ssh ubuntu@40.160.137.239`, not root.
- Main repo renamed `ecotourism` → `india-data-platform`.
- Migrations 0012–0021 applied live.
- **All 16 jobs verified to actually run cleanly** except geo-peaks flakiness (Wikidata-side, see above).
- Postgres daily backups confirmed live.
- Ops console built, redesigned, hardened, tested (184 tests) — **and now deployed and publicly reachable at `https://ops.vidivu.in`.**
- Local repo cleanup and consolidation (2026-09-17), all 10 repos under `~/india-platform/`.
- All 9 GitHub-backed repos verified in sync with origin (2026-09-17); `india-ops-console` pushed as the 10th (2026-09-21).
