# Session 2026-09-23 — all engine jobs daily, geo-peaks retry fix

_Continues `open-items-next-session.md` (item 7, geo-peaks, is now fixed)._

## Decision
Owner chose "all 16 jobs daily". One exception made after evidence: `geo-overpass-peaks-passes` stays **weekly (Sat 04:00 UTC)**. On the catch-up run, overpass-api.de returned 504 and 429 on most grid cells; a daily grid walk would keep hitting the public server's throttle.

## What changed (all pushed to GitHub, deployed on VPS)
- Timers now daily and staggered: geo 04:00 / 04:40 / 05:30, species-gbif 05:00, species-powo 06:00, forest-fsi 06:20, census-st 06:35, fra-jk 06:50, worldcover 07:10, wetlands 07:30, desertification 07:45, culture-glot 08:00. pa/water/usgs/laws were already daily.
- `india-data-core/ops/scripts/make-all-daily.sh` writes systemd drop-ins (`/etc/systemd/system/<unit>.timer.d/daily.conf`) for the 12 non-daily timers. **It still lists the Overpass timer as daily** — re-running it would undo the weekly decision. Remove that line before any re-run.
- `india-geo-engine`: new `geo_engine/http_retry.py` (3 retries, 30/60/120s, only on 429/500/502/503/504/timeouts/connection errors; Retry-After honoured up to 300s). Used by `harvest_wikidata_peaks.py` and `harvest_wikidata_passes_ranges.py`. Peaks SPARQL now runs India/elevation/coord patterns first (`hint:Query hint:optimizer "None"`), subclass walk last. `geo-passes-ranges.service` TimeoutStartSec 600 -> 3600.
- Migration `india-ops-console/db/migrations/0022_all_jobs_daily.sql`: prefix match on heartbeat job_key (`^(geo|forest|forests|culture|species|water|laws|pa|extinct)-`) sets cadence daily. Applied live (17 rows). Also set by hand: `powo-names` daily, `geo-overpass-peaks-passes` weekly, `pa-geometry-backfill` / `forests-coverage-gaps` / `laws-egazette-probe` manual (one-off scripts, no timer).

## Verified
- All 12 changed timers fired immediately on `daemon-reload` (Persistent=true catch-up) — expected once. 9 finished clean; gbif and powo finished success; overpass ran long with 504/429s.
- geo-peaks: real run returned 454 peaks / 1,249 facts after 3 failed attempts (502, 504, 504) then success on attempt 4. Retry logic works; Wikidata is still slow on this query.
- D-71 fix confirmed present on the VPS after pull (`status in ("success","partial")`).

## Facts worth keeping
- **Heartbeat job_keys differ from systemd unit names** (e.g. `geo-wikidata-peaks` vs `geo-peaks`, `powo-names` vs `species-powo`). Migration 0019's exact-name lists missed several jobs for this reason. Match by prefix or check `SELECT job_key, cadence FROM heartbeat`.
- **VPS repo folders drop the `india-` prefix** and sit directly in `~`: `~/geo-engine`, `~/forest-engine`, `~/culture-engine`, `~/species-engine`, `~/water-engine`, `~/laws-engine`, `~/india-ops-console`, `~/india-data-platform`, and `~/core-infra` (= GitHub `india-data-core`).
- DB access: `docker exec -i core-postgres psql -U india -d india_data`.
- `systemctl start <oneshot>.service` blocks until the job ends; Ctrl+C only stops waiting. Oneshots in progress show `activating`, not `running`.
- Running a harvest by hand in a shell fails with "CORE_API_KEY is not set"; use the systemd service, which loads the env.
- Commits/pushes are run from the Mac Terminal, not the VPS.
- The `Claude outputs/` folder in india-data-core was accidentally committed once and removed; now in .gitignore.

## Still open
- geo-peaks.service `TimeoutStartSec` on the VPS not confirmed >= 1200s (retry cycle can take ~15 min). It completed this time, but check.
- Watch first full daily cycle at https://ops.vidivu.in (after ~06:00 UTC 2026-09-24) for red alerts; watch disk (was 24%) from daily raw archives.
- Monthly sources (Census, FRA, FSI, wetlands, desertification, glottolog) now re-download identical data daily — harmless (idempotent) but wasteful; revisit if disk or noise grows.
- Remaining items 6, 8-16 in `open-items-next-session.md` unchanged.
