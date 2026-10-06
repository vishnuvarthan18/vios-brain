# Session handoff — 25 Aug 2026

Pick up here. Companion docs: `credentials-and-wdpa-finding-2026-08-24.md` (the WDPA ruling and credential state), `harvest-engine-session-handoff-2026-08-21.md` (stage ledger).

## Published artifacts from this session

- **Technical blueprint** (for external review — architecture, all 29 sources, cleaning mechanisms, the five questions to ask advisors): https://claude.ai/code/artifact/61986951-2396-4bb5-9291-311061b992ed
- **Repository audit** (where the build actually stands): https://claude.ai/code/artifact/364a1941-620d-4c29-98c6-745a8061e2f4

Both are light-theme only, deliberately. Update by republishing to the same URL.

## Credentials — all resolved, do not re-hunt

Five keys obtained and verified this session. **Values are deliberately not in this doc** — they live in `harvest-engine/.dev.vars` and as Cloudflare production secrets. `wrangler secret list` confirms what is set.

| Secret | State |
|---|---|
| `FIRMS_MAP_KEY` | Was already issued 21 Aug, recovered from Gmail. 5000 txn / 10 min |
| `EBIRD_API_KEY` | Verified live — real records returned from inside the reserve |
| `IUCN_API_KEY` | Verified live — v4 API, `Panthera tigris` sis_id 15955. **Stream still not written** |
| `BHL_API_KEY` | Issued. Use API **v3**; v2 retires 31 Dec 2026 |
| `DATA_GOV_IN_API_KEY` | Via the "Generate API Key" button on the resource page. Cap verified lifted |
| `GEE_SERVICE_ACCOUNT_JSON` | Cloudflare production secret + local file (see below) |
| `WDPA_API_KEY` | ⛔ Ruled out — India withholds ~900 PAs. Recorded in `sources/wdpa.json` |

**LGD resource_id:** `c967fe8f-69c4-42df-8afc-8a2c98057437` — Ministry of Panchayati Raj, 18,893 Tamil Nadu villages, updated 23 Aug 2026. State filter is `filters[stateNameEnglish]`.

⚠️ **The data.gov.in silent-truncation trap.** The public sample key returns HTTP 200 with `status:"ok"` and exactly 10 rows regardless of the `limit` requested. Measured: asked 100, server forced 10, of 18,893. `lgd.js` must assert `count == total` or paginate to exhaustion.

## Two machines — resolved

There are two clones on `mac-2-lan`:

- `~/Downloads/sathyamangalam-atlas-clean` — **old**, last touched 23 Aug, `wrangler.toml` still `REPLACE_WITH_D1_DATABASE_ID`
- `~/sathyamangalam-atlas` — **the agent's working copy**, real D1 id `2a72d827-df49-4b79-9262-5e4f6a27dce2`

Both files the agent needed were in the *old* copy and have been copied across, checksums verified, both correctly gitignored:

- `harvest-engine/secrets/gee-service-account.json` (2,393 bytes, project `sathyamangalam-record`, account `harvest-engine-gee@…`)
- `harvest-engine/db-backups/harvest-engine-d1-local-2026-08-23.sql` (7,581,381 bytes, sha256 `a6b4f9c4cab6aa1d…`)

Dump verified before copying: 5,390 occurrence · 4,104 document · 1,654 taxon · 1,182 historical_passage · 335 extracted_table · 58 place · 20 legal_instrument · 4 observation_layer · 1 news_event · 0 claim.

## 🔴 The structural finding — two databases, diverged

**The public site and the harvest engine are separate systems with different data.**

The site reads `data/atlas.db` (old Python harvester, built 18 Aug). The engine writes to D1. They are not subsets of one another.

| table | site · atlas.db | engine · D1 |
|---|---|---|
| historical_passage | **1** | **1,182** |
| document | 928 (tier A+B) | 4,104 |
| legal_instrument | 2 | 20 |
| observation_layer | 0 | 4 (now 6) |
| occurrence | 74,900 | 5,390 |
| taxon | 1,893 | 1,654 |
| place | 89 | 58 |
| claim | 82 | 0 |

Counts move in **both directions**, so "sync D1 to the site" is a reconciliation, not plumbing. Occurrences would drop 93% if D1 simply overwrote the site. **This has never been decided and everything downstream depends on it.**

Consequence: the published coverage figure of **31.6% is computed from the wrong database** and understates the historical domain by three orders of magnitude.

## 🔴 Seven of eleven site pages are placeholders

`STUB`: /record · /life · /land · /people · /history · /govern · /visit
`LIVE`: /index · /about · /contact · /places (map)

Outreach went out 18 Aug to the Field Director, the PCCF & Chief Wildlife Warden, three DDs and six NGOs, with links to this site.

### The launch-gate decision, unresolved

Vishnu's rule: each of the 7 content categories needs 50% data before going live. Tested against actuals, **4 of 7 already pass**:

| Page | Have / target | % | |
|---|---|---|---|
| History | 1,182 / 600 | 197% | ✅ |
| Life | 1,893 / 2,000 | 95% | ✅ |
| Record | coverage stats | n/a | ✅ |
| Visit | hand-written | n/a | ✅ |
| Land | 89 / 1,200 places | 7% | ❌ |
| Governance | 20 / 120 | 17% | ❌ |
| People | 0 / 100 | 0% | ❌ |

**People can never pass by harvesting** — it is consented oral history and ethnobotany, requiring fieldwork that itself depends on having something credible to show. If People gates the launch, the launch never happens. Vishnu paused on this; do not push it again unprompted.

## Repository audit — gaps found

**Stage 4 (Government) is the weakest stage and none of it is credential-blocked.** Registries exist with no stream file behind them:

- `indiankanoon` — free API, Madras HC judgments on invasive removal, grazing, FRA, night traffic. Highest-value unbuilt source in the repo.
- `tamilnaduarchives` — holds GOMS No.122 (2008) and the 2013 notification, **the only source for the buffer geometry**
- `egazette`, `sathytiger`, `gdelt` — registry, no code
- `moef` / Parivesh — no registry at all

Also unbuilt: IUCN (key live since 21 Aug, nothing reads it), India Biodiversity Portal, Xeno-canto, Dinamani sitemap parser.

**Stale record:** `sources/wdpa.json` carries the correct ruled-out decision, but its `description` field above it still says "not yet obtained, no WDPA_API_KEY in .dev.vars". A reader hits the stale text first.

## Agent progress this session

Commits: `3d775c7` polygon in production via OSM API + geometry_scope enum · `ad5c6e7` GEE geometry gate · `4486166` NDVI and elevation core-scoped.

- **Polygon resolved.** OSM relation `4192204`, 1 outer + 15 inner rings, **791.62 km²**. GEE independently confirmed the same figure with `geodesic=true, evenOdd=true`; the outer-ring control returned 830.80 gross, proving the holes subtract.
- **It is the Wildlife Sanctuary, not the Tiger Reserve.** 791.62 matches the core/WLS figure of 793.49 (0.24%), not the 1,408.40 TR. `tiger_reserve` and `buffer` are legal scopes with **no geometry from any source** — recorded as rows in a new `reserve_geometry` table so the gap is queryable.
- **The 521 was retracted.** Overpass is reachable; it was a ~30-minute outage misread as a permanent block. Endpoint order is now OSM API first, Overpass second.
- **NDVI went DOWN**, 0.4072 bbox → 0.3663 core (−10%). Prediction was wrong. Geometry confirmed fine by SRTM: elevation 787.1 → 875.2 m as predicted, and terrain has no seasonal confound. Working hypothesis: the irrigated command area below Bhavanisagar out-greens dry-deciduous forest in late dry season. **Unverified.**
- **🔴 The `request_hash` bug — the most important find.** The hash excluded geometry, so migrating bbox → polygon didn't change it, `findExistingSource` matched the bbox-era row, and the recompute was skipped as a duplicate. `elevation` hit this exactly: `job_run` reported a **clean run for a computation that never executed**. Every historical `skipped_duplicate` is now suspect and needs auditing.

## Open decisions

1. **Which database is canonical** — D1 or atlas.db. Blocks everything downstream.
2. **When to launch** — see the gate analysis above.
3. **Item 6 ordering** — advised: `pixelArea()` on bbox FIRST (predicts 8.17 × 0.974 ≈ 7.96 km², a falsifiable check against the known −2.64% mechanism), then geometry as a separate commit.
4. **NDVI hypothesis test** — advised the annulus test (bbox MINUS core, same window) over waiting for the northeast monsoon. One calculation, tests the mechanism directly.
5. **The instrumentation / status board** — full prompt was written and **never sent to the agent**. The `request_hash` bug is the argument for it.

## Standing policies — do not relitigate

- Conflicting figures: **no winner, no `superseded_by`**. Both values, both sources, both methods. `superseded_by` only when the *same issuing authority* republishes a correction.
- 335 pending `extracted_table` rows: deprioritised, not reviewed. Sample 25 and report the error rate instead.
- Google News, Vikatan: ruled out. Dinamalar: deferred. Dinamani: build the parser.
- Cron: news + FIRMS only first — both accumulate forward only.

## Deadlines

- **1 Nov 2026** — NASA retires Suomi NPP; FIRMS must move to NOAA-21/20
- **31 Dec 2026** — BHL API v2 retires

## Unrelated but live

GitHub secret-scanning alert on `vishnuvarthan18/w2d-admin`, opened 19 Aug, still unread: "Possible valid secrets detected."
