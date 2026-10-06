# Session checkpoint — 2026-08-24/25

Read this first when picking the project back up. Supersedes
`atlas-session-checkpoint-2026-08-14.md` for anything they disagree on.

## What is live right now

| | URL | Notes |
|---|---|---|
| Public site | https://sathyamangalam.online | Cloudflare **Pages**, project `sathyamangalam-atlas`. **Direct upload, NOT git-connected** — pushing to GitHub deploys nothing. |
| Harvest engine | https://engine.sathyamangalam.online | Cloudflare **Worker**, password-gated. Separate deploy target from the site. |

Deploy the site with `make deploy`. Deploy the engine with
`cd harvest-engine && npx wrangler deploy`.

Cloudflare account: `(removed)` (`aa523b5d2ceed84e54997db0dc6cbaec`).

### Engine resources (created 2026-08-24)

- D1 `harvest-engine-db` — `2a72d827-df49-4b79-9262-5e4f6a27dce2`
- R2 `harvest-engine-raw`
- Queues `harvest-engine-jobs` + `-dlq`
- Secrets set: `DASHBOARD_PASSWORD`, `SESSION_SECRET`, `CONTACT_EMAIL`,
  `EBIRD_API_KEY`, `FIRMS_MAP_KEY`, `IUCN_API_KEY`, `BHL_API_KEY`,
  `DATA_GOV_IN_API_KEY`, `GEE_SERVICE_ACCOUNT_JSON`
- **Cron is OFF** (`crons = []`). Turning it on starts an unattended daily
  crawl of all 30 streams including `.gov.in` hosts. Deliberate.

The dashboard password is NOT in this repo. It was generated during the
2026-08-24 session and handed over in chat; if it is lost, rotate it with
`npx wrangler secret put DASHBOARD_PASSWORD`.

## Three bugs fixed on the public site

1. **`exports/` was never deployed.** Pages uploaded `site/` as the web root,
   but every page fetches `../exports/*.json`, which from the web root resolves
   to `/exports/*.json`. All of it 404'd. The live home page rendered
   `(load failed, serve over http)` where the coverage figure belongs, to every
   visitor. `scripts/build_dist.py` now assembles `dist/` = `site/` + `exports/`
   and hard-fails if `coverage.json` is missing.
2. **Dead scripts on the Under Construction stubs.** Commit `3aeacc5` replaced
   seven page bodies but left four pages' data scripts running against removed
   elements — `land`/`places` threw and fetched 4 JSON exports to render
   nothing; `life`/`record` threw. Removed.
3. **`python3 -m http.server` cannot serve this site.** Since `93cc85f` the
   pages use clean extensionless URLs with `site/` as the web root, so every
   nav link 404s locally and `href="/"` exposes a directory listing of the repo
   root including `.git`. Use `scripts/devserve.py` (or `make serve`).

## The reserve polygon — the big one

`reserve.boundary_geojson` had been NULL since Stage 0, so every spatial figure
was bbox-scoped. Measured, that bbox is **2,594 km² against a real core of
791.62 km² — 3.28x too large**, and it is not even a strict superset: the
boundary reaches 11.8344N while the bbox stops at 11.82N, so bbox queries were
diluted AND clipping the northern edge.

**WDPA is ruled out permanently.** India withholds ~900 protected areas from
public publication on Protected Planet; searches return nothing for
Sathyamangalam, nothing for the `Satyamangalam` spelling, nothing for Mudumalai
as a control. Publication-level restriction, not an access tier — a key
authenticates to the same public subset. Do not register. See
`sources/wdpa.json`.

**OSM relation 4192204** supplies it: 1 outer + 15 inner rings, **791.62 km²**.

> **This is the CORE / wildlife sanctuary boundary, NOT the tiger reserve.**
> It matches the core figure (793.49 km²) to 0.24% and not the 1,408.40 km²
> Tiger Reserve. Tags agree: `wikipedia=en:Sathyamangalam Wildlife Sanctuary`,
> `protect_class=4`. The 614.91 km² buffer is not mapped in OSM.
> **Never label anything "reserve-scoped".**

Confirmed by three independent fetches (Overpass from a shell, Overpass from a
Worker, OSM API from a Worker — byte-identical geometry) and two independent
implementations of the assembler.

### geometry_scope is enforced, not just documented

Migration `0003_geometry_scope.sql` adds a `geometry_scope` column to
`observation_layer` with the enum enforced by trigger:

    bbox | core_wls | tiger_reserve | buffer

It rejects `reserve_scoped` outright. `tiger_reserve` and `buffer` are legal
values with **no geometry behind them**, so the gap is nameable. The new
`reserve_geometry` table makes that queryable:

| scope | official | measured | geometry |
|---|---|---|---|
| core_wls | 793.49 | 791.62 | present |
| tiger_reserve | 1,408.40 | — | **NONE EXISTS** |
| buffer | 614.91 | — | **NONE EXISTS** |

## GEE geometry gate — passed (job_run id=9)

Before recomputing anything, Earth Engine was asked for the polygon's own area.

| variant | GEE |
|---|---|
| `geodesic=true, evenOdd=true` | **791.62 km²** (0.000% off) |
| `geodesic=false, evenOdd=true` | 791.62 km² |
| outer ring only (control) | 830.80 km² = exactly the gross area |

Record this setting on every core-scoped row:
`ee.Geometry.Polygon(geodesic=true, evenOdd=true)`,
`Geometry.area(maxError=ErrorMargin(1, meters))`.

Holes-dropped, winding-order and geodesic-vs-planar are all ruled out by
measurement. 674 vertices accepted whole — **no simplification applied**.

Gotchas found here, both live: `getGeeAccessToken` returns
`{accessToken, expiresIn, projectId}` — using the object as a bearer token
gives `Bearer [object Object]` and a 401 that reads exactly like a stale
credential. And `Geometry.area`'s `maxError` is an `ErrorMargin`, not a number.

## NDVI went DOWN, and that is real

Pre-registered: cutting to the core should RAISE NDVI (bbox contains the
Bhavanisagar reservoir, farmland, three towns). It fell.

| | scope | NDVI | pixels | images |
|---|---|---|---|---|
| control | bbox | 0.4072 | 890,527 | 32 |
| recompute | core_wls | **0.3663** | 264,567 | 24 |

The paired control reproduces the remembered 0.407 **in the same run and same
window**, so the seasonal confound is eliminated. The geometry is verified
correct by an independent check with no seasonal confound — SRTM elevation
**787.1 → 875.2 m (+88.1)**, slope +1.73°, floor 187 → 251 m, exactly as
predicted.

Likely explanation, NOT verified: ~70% of this reserve's rain comes from the
**northeast** monsoon (Sep–Nov), so a late-July/August window is late dry season
for the dry-thorn/dry-deciduous core, while the irrigated command area below
Bhavanisagar (sugarcane, banana, paddy) stays green year-round. **Test:**
recompute the same pair in a post-northeast-monsoon window; the sign should
flip.

## A silent bug worth remembering

`request_hash` did not include the geometry. Migrating a computation from bbox
to polygon therefore did not change its hash, `findExistingSource` matched the
bbox-era row, and the recompute was **silently skipped as a duplicate** while
`job_run` reported a clean run. `elevation` hit this exactly (no rolling date
window). NDVI only escaped because its window rolls daily. Fixed —
`geometry_scope`, `vertex_count`, `ring_count` are now part of the hash.

Class of bug to watch for: anything that makes a run look successful while
quietly doing nothing.

## Overpass availability — corrected

An earlier note in this repo claimed Overpass is unreachable from Cloudflare
Workers. **That was wrong.** Two runs returned HTTP 521 and their byte-identical
bodies were misread as proof of a permanent block; a 521 error page is
byte-identical every time. A run 27 minutes later succeeded. It was a ~30-minute
outage.

Endpoint order is now `api.openstreetmap.org` first (right tool for a known
relation id), Overpass second. **This does not rescue the gazetteer** — the
~1,200-place gap needs "every place node in this bbox", which only Overpass
answers. The gazetteer must tolerate outages via retries and rescheduling.

Mirrors tested: `kumi.systems` and `private.coffee` return 500;
`overpass.osm.ch` returns **HTTP 200 with an empty result set** — a silent-zero
source, do not use.

## Open — needs Vishnu

1. **D1 dump** `harvest-engine/db-backups/harvest-engine-d1-local-2026-08-23.sql`
   — gitignored, not on this Mac. Holds Stage 5's 1,182 historical passages and
   Stage 8's 335 extracted tables. Load rather than re-harvest.
2. **GEE service account JSON** → `harvest-engine/secrets/` (gitignored, absent).
   Production has the secret so remote runs work; local runs cannot.
3. **LGD trap**: data.gov.in publishes a sample key that caps at 10 records
   while returning HTTP 200 `status:"ok"`. `lgd.js` must assert
   `count == total` or paginate to exhaustion. Resource id
   `c967fe8f-69c4-42df-8afc-8a2c98057437`, 18,893 Tamil Nadu villages.
   `villageCensus2011Code` is a direct join key to Census 2011.
4. **BHL** retires API v2 on 2026-12-31 — use `/api3`. BHL is also NOT fully
   credential-gated: OAI-PMH and the bulk TSV exports need no key.
5. **NASA** retires Suomi NPP on 2026-11-01 — move FIRMS to NOAA-20/21.
6. **IUCN** — the key was never the blocker; the stream was never written.

## Queue as it stands

Done: polygon in production · OSM API path verified · geometry_scope enum +
backfill · GEE gate · NDVI and elevation core-scoped.

Next:
1. `pixelArea()` on the all-time 8.17 km² forest-loss figure — **its own
   commit**. Note forest-loss is still bbox-scoped, so moving geometry and
   changing area maths are two effects on one number; sequence them.
2. Move forest-loss / rainfall / temperature / Landsat to the polygon, one at a
   time, each with a paired control.
3. Instrumentation / status board — a permanently failing source must show as
   failed, not as "implemented".
4. Buffer geometry from the 2013 TR notification and GOMS No.122 (2008), both
   already in the Stage 8 corpus. **If the boundary is described textually
   (village/survey numbers) rather than as coordinates, say so and stop — do
   not synthesise a polygon.**
5. Load the D1 dump, verify row counts on all 13 tables, then re-run only the
   geo-dependent streams.
6. The Hindu TN relevance filter (bare tiger/elephant terms missing; dropped two
   genuine stories) → then cron ON for **news + FIRMS only** (they can only
   accumulate forward: Stage 7 RSS has no date parameter, FIRMS NRT caps at 10
   days). The other 28 stay manual until each has one clean hand-run.
7. `/record` page restored from `a367949`, then `/life`. Outreach went to the
   Field Director, the PCCF and six NGOs all linking to placeholder text. Port
   content forward onto current branding and clean URLs — do **not** merge the
   old branch.

## Standing policies

- **Conflicting figures**: no winner, no `superseded_by`. Keep both values, both
  sources, both dates, both methods; surface the disagreement as content.
  `superseded_by` is only for the same issuing authority republishing a
  correction. Officers were emailed directly about the leopard 111-vs-20 and
  tiger 112-vs-8-10 splits; resolution comes from them. The 791.62 km²
  measurement is recorded as a third independent data point on the core-area
  figure — it does **not** resolve it.
- **335 pending `extracted_table` rows**: do not review wholesale. Pull a
  stratified sample of 25 across table types and report the extraction error
  rate. Same for un-annotated Stage 5 passages. `pending` stays the default;
  nothing auto-publishes.
- **Google News and Vikatan**: ruled out on robots/terms grounds. Not
  revisiting, not contacting publishers. **Dinamani**: build the
  `news_sitemap.xml` parser (it has no RSS).
- **Pages 349–353** (JPEG2000, `unpdf` cannot decode): deferred, logged as a
  known gap. Correct not to guess at image content. When worth doing, it is not
  a Workers job — `pdfimages -j2k` then `opj_decompress`, outside the Worker.
  The 25 never-extracted appendices are worth more.
- **Verification playbook**: `wrangler d1 execute --remote --file` returns only
  an aggregate summary and swallows per-statement rows. Do not specify a
  single-transaction verification script that needs to report values — split it
  into discrete `--command` calls.
