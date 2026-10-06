# india-water-engine

Water Systems engine — the platform's second engine (`PLAN.md` Phase F).

Built second on purpose. The National Water Data Portal was rated the best
source in the Atlas's entire 136-source survey, and its data shape —
time-series telemetry rather than static polygons — is a genuine stress test
of a core API that had only ever seen protected-area boundaries.

## Sources

| Source | Status | Note |
|---|---|---|
| NWDP CKAN (`nwdp.nwic.gov.in`) | **working** | 558 datasets, no API key. Verified from the VPS 2026-09-06. |
| HydroRIVERS / HydroLAKES | registered | Static bulk downloads; see "Bulk sources" below. |
| JRC Global Surface Water | registered | 40-year surface-water extent history. |
| India-WRIS | **blocked** | Genuinely unreachable from this VPS — DNS resolves, TCP times out. Likely geo-filtered. |
| CWC flood forecasting | untested | |
| INGRES groundwater | untested | JS-only; one of the four sources where Playwright is sanctioned. |

## Licensing is the hard part

`PLAN.md` §F.1 says to check every NWDP dataset's licence tag individually
because they are inconsistent. That is not a formality — until the inventory
exists, nothing built on NWDP can honestly state what it may do with any
given record. So `scripts/harvest_nwdp_catalog.py` treats the catalogue as a
first-class deliverable: every dataset becomes an entity carrying **its own**
licence, and a dataset with no licence tag is recorded as `UNSTATED`, never
quietly upgraded to "open".

## Verifying the Atlas rather than trusting it

The catalogue pass re-runs the Atlas's own per-keyword counts against the
live portal on every run. As of 2026-09-06 it confirmed `river`=291,
`flood`=473, `glacial`=1, and confirmed `spring` and `waterfall` as genuine
zero-result absences — but found `estuary`=2, where the Atlas records zero.
See `DECISIONS.md` D-13.

Confirmed absences are written as `coverage_gap` entities with
`is_permanent_gap`, so they are recorded once rather than re-investigated.

## Bulk sources

HydroRIVERS, HydroLAKES and JRC Global Surface Water are large static
downloads (the global HydroRIVERS geodatabase alone is around 2 GB). On a
40 GB / 4 GB-RAM VPS shared with Postgres and MinIO, ingesting those is a
capacity decision, not just a code one — they are registered with their
licences recorded, and the ingest wants the regional (Asia) extracts and a
streaming reader rather than a naive full download.

## Running it

```bash
cd ~/water-engine
set -a; . ~/core-infra/engine-keys.env; set +a
docker compose run --rm harvest python scripts/harvest_nwdp_catalog.py --dry-run --limit 20
docker compose run --rm harvest python scripts/harvest_nwdp_catalog.py
```
