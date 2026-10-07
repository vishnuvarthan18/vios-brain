# India Environmental Data Source Atlas

**A source survey for an India-wide data project across 8 categories. Verified 6 September 2026.**

Every API, bulk file and scrape target found for the eight categories — each one actually fetched, with the failures kept in. Sources that could not be reached are listed as failures rather than dropped, because knowing a well-known portal is broken is worth as much as knowing a working one exists.

| | |
|---|---|
| Sources checked | **136** |
| Live APIs | **30** |
| Bulk / scrape targets | **47** |
| Dead or blocked | **33** |
| No digital source at all | **26** |

*Method: live HTTP fetch of a specific data URL, not a homepage. Environment: cloud sandbox with a robots-respecting fetcher behind a filtering proxy.*

---

## Read this before you build anything

### 01 — data.gov.in has a working API, and it covers six of your eight categories

Every agent hit a robots wall on the `data.gov.in` website and wrote it off. The API behind it is fine. `https://api.data.gov.in/lists?api-key=(secret removed) returns 589 matching datasets; `filters[title]=wildlife` returns 68. `https://api.data.gov.in/resource/124b8f6e-0006-4685-9157-bb2996071068?api-key=(secret removed) returns 37 rows of state-wise recorded forest area and forest cover from ISFR 2015, as clean JSON. Register for your own free key rather than using the public sample one. Caveat: most of these tables are small parliamentary-answer extracts, not primary geodata — good for tabular state-level facts, useless for boundaries.

### 02 — GBIF's India total is 92% birds. Plan around that, not around the headline number

65,579,568 India occurrence records sounds like total coverage. 60,613,324 of them are the eBird Observation Dataset and 1,622,493 are iNaturalist — leaving roughly 3.3 million records for every plant, insect, fish, reptile, amphibian, fungus and mammal combined. All three counts were pulled from the GBIF API in this session. Two consequences: ingesting eBird or iNaturalist separately would duplicate 95% of what GBIF already gives you, and any "species coverage" metric you compute from GBIF will be wildly bird-skewed unless you exclude those two datasets.

### 03 — "Blocked here" is not "dead" — 15 sources need a re-test from your own crawler

The fetcher used for this survey respects `robots.txt` and sits behind a filtering proxy. iNaturalist, eBird, ChecklistBank, Xeno-canto, NCBI, Overpass, Wikidata, India-WRIS, CWC flood forecasting and CPCB were all refused at that layer, not by the source. They are marked Blocked, kept separate from Dead, and should be re-verified from your own infrastructure before you conclude anything about them. Note that respecting robots.txt is your own call to make for each one.

### 04 — Two blocks are real policy, not accident

The Paleobiology Database's own `robots.txt` explicitly disallows `/data1.2/` — the entire data API. The Supreme Court and NGT judgment searches are CAPTCHA-gated by design. TKDL restricts its traditional-medicine corpus to patent offices under agreement, deliberately, to block biopiracy. These are not obstacles to route around; treat them as the answer.

### 05 — India's own taxonomic and protected-area authorities are the least machine-accessible sources here

ZSI's Fauna of India portal shows "Website Under Construction". BSI sells Flora of India as print fascicles. WII ENVIS — the actual upstream supplier of India's protected-area records to the global WDPA — failed DNS resolution on every attempt. NBSS&LUP, India's soil-survey authority, publishes no data at all. The pattern is consistent: for Indian biodiversity and land data, taxonomic authority and digital accessibility are inversely correlated, and the usable sources are almost always international aggregators holding Indian data second-hand.

---

## Status legend

| Status | Meaning |
|---|---|
| `API` | Live, queryable, no key needed. Verified with a real query. |
| `API + key` | Live, free registration required. |
| `Bulk` | Downloadable files — shapefile, CSV, raster, PDF. |
| `Scrape` | HTML or PDF. **+JS** means it needs a headless browser. |
| `Blocked` | Unverifiable from this sandbox. Re-test yourself. |
| `Dead` | Genuinely broken, walled or abandoned right now. |
| `No source` | Exists only offline — print, field survey, or nowhere. |

---

## 01. Protected Areas

> Good coverage for the headline categories, thin everywhere else. Tiger reserves, elephant reserves, Ramsar sites and World Heritage sites each have an authoritative, reachable list. Conservation reserves, community reserves and eco-sensitive zone boundaries effectively have none. India's own protected-area database (WII ENVIS) is unreachable, so the practical route to boundary geometry is the international WDPA, which holds India's data second-hand.

| Source | Endpoint | Status | Key | Updates | What it actually gives you |
|---|---|---|---|---|---|
| **Protected Planet / WDPA (UNEP-WCMC)** | `api.protectedplanet.net/v3` | API + key | Free, self-serve | Monthly | The most complete PA inventory for India — every NP, sanctuary, conservation and community reserve with IUCN category, governance type, and polygon geometry via `with_geometry=true`. Returned a correct 401 without a token, confirming it's live. **License is restrictive: no commercial use, no redistribution.** |
| **NTCA — Tiger Reserves** | `ntca.gov.in/tiger-reserves/` | Scrape | No | On notification | One clean 58-row HTML table: reserve name, state, PA and TR notification years, core / buffer / total area in km², each row linking a gazette PDF. Best source for the core-buffer split, which WDPA does not carry. No geometry. |
| **MoEFCC — Elephant Reserves Atlas** | `moef.gov.in/uploads/2023/11/PE-Elephant-Reserve-of-India-an-atlas.pdf` | Bulk PDF | No | Rare (2023 ed.) | All 33 elephant reserves — name, state, area, year notified, 80,777 km² total. Elephant reserves are not an IUCN category, so this is the **only** authoritative source; WDPA will not have them as a clean layer. The Project Elephant HTML landing pages 404. |
| **Ramsar RSIS — India annotated summary** | `rsis.ramsar.org/sites/default/files/rsiswp_search/exports/Ramsar-Sites-annotated-summary-India.pdf` | Bulk PDF | No | Irregular | All 75 Indian Ramsar sites: site number, state, area in ha, coordinates, designation date, ecological description. Needs PDF table parsing. Use this because the search UI is down — see next row. |
| **Ramsar RSIS — country search interface** | `rsis.ramsar.org/ris-search?f[0]=regionCountry_en:India` | Dead | No | — | **502 Bad Gateway** on two independent attempts across two URL patterns. Individual per-site RIS pages (`/ris/463`) work, but the country-filtered search that would enumerate them does not. Re-check periodically. |
| **UNESCO World Heritage — India** | `whc.unesco.org/en/statesparties/in` | Scrape | No | Annual (Committee) | 45 sites — 37 cultural, 7 natural, 1 mixed — as linked names with inscription years. Prose, not a table. The machine-readable XML list endpoint is robots-disallowed. Adds only the heritage designation; the underlying parks are already in WDPA. |
| **data.gov.in — PA datasets** | `api.data.gov.in/lists?filters[title]=wildlife` | API + key | Free key | Irregular | 68 wildlife datasets including "Number and Area of National Parks and Wildlife Sanctuaries of India" and state-wise wildlife crime cases. Verified working. Small tabular extracts, not geometry. |
| **Data Basin — India PAs (2013)** | `databasin.org/datasets/e0722740353f4a90856af467a758006d/` | Bulk | No | Frozen since 2013 | 520+ **point** locations (not polygons) for parks, sanctuaries and biosphere reserves, CC-BY 3.0, Western-Ghats biased. Thirteen years stale — a historical cross-check only. |
| **WII ENVIS — Protected Area Database** | `wiienvis.nic.in` | Dead | — | — | **DNS resolution failure** on five separate paths, over both HTTP and HTTPS. This is India's designated national PA database and WDPA's own upstream supplier. Its absence is the single biggest gap in this category. |
| **PARIVESH — eco-sensitive zone notifications** | `parivesh.nic.in` | Scrape + JS | — | Continuous | Pure React shell to a static fetch; every guessed backend API path returned 404. The statutory ESZ notifications are real and important but need a headless browser and endpoint discovery via devtools. |
| **BirdLife DataZone — Indian IBAs** | `datazone.birdlife.org` | Walled | Data agreement | — | Country factsheet loads but every data table renders "1-0 of 0 / Loading…" — JS-injected, not server-side. BirdLife's stated route is a manual "request our data" agreement, not self-serve. |
| **OSM Overpass — boundary=protected_area** | `overpass-api.de/api/interpreter` | Blocked | No | Continuous | Four mirrors tried: the canonical instance is robots-disallowed, two timed out, and `overpass.osm.ch` returned valid JSON but zero elements for bounding boxes that definitely contain features — a partial mirror database. Re-test `overpass-api.de` yourself; it is the only complementary source of PA polygons besides WDPA. |
| **Conservation & community reserves** | — no national dataset found | No source | — | — | The two thinnest PA categories. No national table with area and notification year exists anywhere reachable. Lives in individual state gazette notifications and ISFR appendices. |
| **PA management plans & boundary history** | — print only | No source | — | — | Working plans and boundary revision records exist only as printed state forest department documents. Would need per-state RTI requests or field collection. |

**Gaps**

- **ESZ boundaries** — only as individual scanned Gazette SO notifications, one per protected area, plus PARIVESH's JS database. No consolidated machine-readable set.
- **Per-state PA shapefiles** — a long tail of ~28 separate forest-department portals, none uniform, none surveyed here.
- **Ramsar monitoring data** beyond the designation-time RIS — not digital anywhere.

**Which one to build against**

- **Geometry and IUCN category:** WDPA. It's the international standard, monthly, and the only reachable source of polygons.
- **Tiger reserve attributes:** NTCA, not WDPA — the core/buffer split exists nowhere else.
- **Elephant reserves:** the MoEFCC atlas, exclusively.
- **Skip:** Dataful's paid sanctuary dataset — it's a repackaging of WII ENVIS. Data Basin — thirteen years stale.

---

## 02. Forests & Land

> Forest cover and mangroves are well served. Wetlands are partly served. Grasslands and coral reefs are close to empty, and India's own official land datasets are mostly locked in image-based PDFs or behind portals that don't answer scripted requests. The reliable machine-readable sources here are all international.

| Source | Endpoint | Status | Key | Updates | What it actually gives you |
|---|---|---|---|---|---|
| **FSI — India State of Forest Report** | `fsi.nic.in/isfr-volumes → fsi.nic.in/isfr-2021/chapter-2.pdf` | Scrape PDF | No | Biennial | The legally authoritative forest-cover figures: state and district cover by density class, recorded forest area, growing stock, carbon. Chapter PDFs are all reachable — but **chapter 2, the one with the state tables, yielded no extractable text.** It's image-based; budget for OCR, not text parsing. |
| **FSI Van Agni — forest fire alerts** | `fsiforestfire.gov.in` | Scrape | No | Every 15 min | Live MODIS + VIIRS hotspot dashboard with fire-point search, large-fire monitoring and danger ratings; 337,501 SMS subscribers. No API and no bulk download found — dashboard scraping only. |
| **NASA FIRMS — fire detections** | `firms.modaps.eosdis.nasa.gov/api/area/csv/{KEY}/VIIRS_SNPP_NRT/{bbox}/{days}` | API + key | Free, by email | Near real-time | Point fire detections with lat/lon, confidence, brightness, acquisition time. **Documented limit: 5,000 transactions per 10-minute interval.** A cleaner programmatic substitute for scraping Van Agni, though FSI's version is India-tuned and feedback-verified. |
| **Global Forest Watch Data API** | `data-api.globalforestwatch.org` | API + key | Free signup | Annual | Catalog endpoint verified live, returning real dataset names (`gadm__tcl__iso_summary`, GLAD alert tables by admin level). The SQL query endpoint returned 403 twice — it needs an `x-api-key`. This is the practical access route to Hansen/UMD tree-cover-loss data for India. |
| **ESA WorldCover 10 m (via Planetary Computer STAC)** | `planetarycomputer.microsoft.com/api/stac/v1/collections/esa-worldcover` | API | No | 2020 & 2021 only | 10 m land cover, 11 LCCS classes, Cloud-Optimized GeoTIFF, **CC-BY-4.0**. Full India coverage. Warning: the two years use different algorithm versions, so year-on-year differencing is not valid change detection. |
| **Copernicus CDS — satellite land cover** | `cds.climate.copernicus.eu/datasets/satellite-land-cover` | API + key | Free CDS account | Annual, 1992– | 300 m annual land cover, 22 classes, NetCDF, three decades deep. Coarse but the only long time series. Access via the `cdsapi` Python client. |
| **Global Mangrove Watch** | `data.unep-wcmc.org/datasets/45` | Bulk | No | Annual since 2018 | Mangrove extent polygons for 1996, 2007–2010, 2015–2020 as per-year shapefiles plus Zenodo rasters. Full Indian coastline. **The only usable mangrove source** — no working Indian government equivalent was found. |
| **ISRIC SoilGrids** | `rest.isric.org/soilgrids/v2.0/properties/query?lon=79.0&lat=21.0&property=soc` | API | No | Static (v2.0) | 250 m gridded pH, organic carbon, clay, sand, silt, bulk density, CEC, nitrogen at six depths. Returned real values for central India (SOC 13.7 g/kg at 0–5 cm); some pixels return null. The only working soil source for India, official or otherwise. |
| **data.gov.in — forest datasets** | `api.data.gov.in/lists?filters[title]=forest` | API + key | Free key | Irregular | 589 forest datasets. Verified pull: 37 rows of state-wise recorded forest area vs forest cover from ISFR 2015, clean JSON. **The easiest route to ISFR state tables without OCRing the PDFs** — though only for the years someone happened to upload. |
| **DoLR Wasteland Atlas of India** | `dolr.gov.in/documents/wasteland-atlas` | Bulk PDF | No | ~5-yearly | State-wise wasteland area by category — gullied land, degraded pasture, salt-affected. Front matter downloaded successfully; the full atlas sits one link deeper. A genuine land-degradation proxy for India. |
| **VEDAS (SAC/ISRO)** | `vedas.sac.gov.in` | Scrape | API centre needs account | Multi-year | Describes the National Wetland Inventory (3.58 million wetlands), a desertification and land-degradation atlas, and forest biomass apps, plus an API centre for registered keys. **Real and promising, but no concrete download or endpoint could be pinned down** — guessed atlas paths 404. Needs manual navigation with an account. |
| **Bhuvan / NRSC LULC** | `bhuvan-app1.nrsc.gov.in/api/` | Scrape | Likely, free | Per epoch | The API root lists named endpoints — "LULC 50K Statistics", "LULC 250K AOI Wise Statistics" — but the documented paths 404, the wiki page 404s, and WMS GetCapabilities timed out. **India's own official LULC series, and no value could be extracted from it.** |
| **FAOSTAT land-use API** | `fenixservices.fao.org/faostat/api/v1/en/data/RL` | Blocked | No | Annual | Three attempts, three failures (robots timeout, two 400s); the bulk download server returned 403. Normally a genuinely open API — re-test from your own crawler. |
| **NBSS&LUP — Indian soil survey** | `nbsslup.in` | No source | — | — | India's official soil-survey authority publishes a list of PDF regional erosion reports and nothing else. No database, no map download, no API. SoilGrids is your only option. |
| **Grassland & rangeland extent** | — no official dataset | No source | — | — | No government portal, API or bulk dataset exists. What exists: an academic mapping of semi-arid open natural ecosystems (Rathore et al., *J. Biogeography* 2021) and a global ML product as a Google Earth Engine asset. India's first official open-ecosystems guide was only being produced in 2026. |
| **Coral reefs of India** | — nothing India-specific found | No source | — | — | ICRI's document library holds exactly one India artifact — a 2024 member-country report PDF. NCSCM's site has no data portal. Allen Coral Atlas was not reached at a correct URL and needs re-testing at `allencoralatlas.org`. Effectively a gap. |
| **Sacred groves (as land)** | `sacredearthtrust.in` | No source | — | — | The best directory documents ~14,000 of an estimated 150,000+ groves, from secondary literature, with no coordinates and no standardised fields. See category 07 for the ethics of publishing these at all. |

**Gaps**

- **Grasslands** and **coral reefs** — no usable India source at all. These need either a research partnership or derivation from WorldCover / global products.
- **State land-degradation time series** — the NRSC/VEDAS atlas is referenced everywhere and downloadable nowhere.
- **Bhuvan LULC values** — the portal exists, the data does not come out. Likely solvable with a registered account.
- **Mangrove species** (as opposed to extent) — GMW mentions species layers; not verified.

**Which one to build against**

- **Official forest area:** FSI ISFR, via data.gov.in's JSON extracts where the year exists, OCR where it doesn't. It is the legally recognised figure.
- **Forest change over time:** GFW/Hansen. But its "tree cover" is a spectral definition that includes plantations and orchards — it is **not** comparable to FSI's forest definition. Don't mix them in one metric.
- **Independent cross-check:** ESA WorldCover, 10 m, one snapshot.
- **Mangroves:** Global Mangrove Watch, uncontested.
- **Note:** the DoLR Wasteland Atlas and the VEDAS Desertification Atlas are two different agencies measuring related-but-different things. Reconcile before combining.

---

## 03. Mountains & Geography

> Terrain rasters are excellent and free. Named features are a problem. Survey of India — the only authority for official Indian spot heights — paywalls everything except toposheet PDFs, so peak elevations have to come from DEM sampling or crowd-sourced gazetteers, and those differ from official surveyed heights by tens of metres. Caves, sub-range boundaries and local place names have no source at all.

| Source | Endpoint | Status | Key | Updates | What it actually gives you |
|---|---|---|---|---|---|
| **Copernicus DEM 30 m (AWS Open Data)** | `copernicus-dem-30m.s3.amazonaws.com` | Bulk | No | Static (2010–15) | Public S3 bucket, valid listing returned, real tiles (~3.7 MB each) with auxiliary water-body and editing masks. 30 m, no key, no auth. **The strongest open substitute for Survey of India's restricted elevation data.** Attribution required per the per-tile EULA. |
| **OpenTopoData (SRTM 90 m)** | `api.opentopodata.org/v1/srtm90m?locations=27.9881,86.9250` | API | No | Static | Point elevation anywhere. Returned 8,722 m for Everest's coordinates — against the surveyed 8,848.86 m, which tells you exactly how much DEM sampling can differ from an official height. Public instance is soft-limited to about 1 request/second. |
| **Open-Elevation** | `api.open-elevation.com/api/v1/lookup?locations=27.9881,86.9250` | API | No | Static | Same class of data (returned 8,771 m for the same point), self-hostable. Useful as a fallback when OpenTopoData rate-limits you. |
| **OpenTopography global DEM API** | `portal.opentopography.org/API/globaldem` | API + key | Free registration | Static | Returned a correct 401, confirming it's live and enforcing auth. Clips SRTM/Copernicus/LiDAR rasters to any bounding box on demand — better than downloading global tiles if you want India by district. |
| **GeoNames gazetteer** | `api.geonames.org/searchJSON?q=Kangchenjunga&username={user}` | API + key | Free username | Continuous | Verified live — the shared demo account returned a real quota-exhausted JSON response, confirming schema and endpoint. Named peaks, passes, valleys with coordinates and feature codes. **CC-BY 4.0.** Free tier has a daily credit quota. |
| **USGS earthquake catalog (FDSNWS)** | `earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&minlatitude=6&maxlatitude=37&minlongitude=68&maxlongitude=97` | API | No | Minutes | Returned 43 real events for the South Asia box in one month, e.g. M5.1 115 km NE of Joshimath. **Public domain.** The practical open substitute for India's NCS, which sells its catalog. |
| **GADM administrative boundaries** | `geodata.ucdavis.edu/gadm/gadm4.1/shp/gadm41_IND_shp.zip` | Bulk | No | Rare (v4.1) | State, district and sub-district polygons. Real zip payload confirmed. **Licensed for non-commercial/academic use only** — check this before any commercial reuse. |
| **Natural Earth admin-1** | `naturalearthdata.com/downloads/10m-cultural-vectors/10m-admin-1-states-provinces/` | Bulk | No | Periodic | **Explicitly public domain** — the cleanest licence of any boundary source here. 1:10m scale: good for national maps, too coarse for district work. |
| **Local Government Directory (LGD)** | `lgdirectory.gov.in` | Bulk + API | NAPIX API likely | Continuous | 677,508 villages, 784 districts, 7,092 sub-districts with the canonical LGD codes that every other government system cross-references. **The best administrative gazetteer for India.** Directory reports download openly; the API sits on NAPIX. |
| **Census of India — NADA catalog** | `censusindia.gov.in/nada/index.php/catalog` | Bulk | No | Decadal | 40,184 datasets including the 2011 Location Code Directory and A-01 village/town/household/area tables. Note this fetched successfully here but the same host failed for the tribal-affairs agent — availability is inconsistent. |
| **Survey of India — Online Maps Portal** | `onlinemaps.surveyofindia.gov.in` | Scrape / paid | Account + payment | Legacy | Open Series Map PDFs are free. The Village Boundary Database, Digital Vector Database, Geo-Referenced Raster and Administrative Boundary Database all require registration and **payment**. This is the single biggest blocker in the category. |
| **GSI Bhukosh — geology & landslides** | `bhukosh.gsi.gov.in` | Dead | — | — | **Connection timeout.** The parent GSI portal page loads and confirms Bhukosh covers geology, geophysics, seismotectonics, meteorites and landslide hazard — but the map viewer itself never responded, and no bulk or API route was found. India's only geology data source, unreachable. |
| **National Center for Seismology** | `seismo.gov.in` | Paid | Commercial | — | Site states plainly that NCS provides earthquake data "to various user agencies… **on payment basis**". 160+ stations, no public API, no bulk catalog. Use USGS instead. |
| **Wikidata SPARQL** | `query.wikidata.org/sparql` | Blocked | No | Continuous | All three Wikidata endpoints returned "domain is cache-only and cannot be fetched" — a fetcher-level block. Normally the best no-key, CC0 source of structured peak data (name, elevation, coordinates, parent range). **Re-test this first** — it would fill much of this category's named-feature gap. |
| **DBpedia (per-page)** | `dbpedia.org/page/Kangchenjunga` | Scrape | No | Dump releases | Individual peak pages work — returned Kangchenjunga at 8,586 m. The SPARQL endpoint was proxy-rejected twice, so bulk querying is unverified. Page-by-page only, for now. |
| **Peakbagger / Nominatim** | `peakbagger.com · nominatim.openstreetmap.org` | Blocked | No | Continuous | Both robots-disallowed here. Peakbagger is reputationally rich for Himalayan peak lists; unverified. |
| **Himalayan Database** | `himalayandatabase.com` | Bulk | No | Twice yearly | 500+ peaks, 11,800+ expeditions, 95,000+ member records, free since v2. **Nepal-centric** — useful only for shared border massifs like Kangchenjunga, not an India source. |
| **Cave / karst inventory** | — none found | No source | — | — | No candidate URL even surfaced. A genuine absence, not a fetch failure. Speleological society records and journal literature only. |
| **Local & tribal place names** | — none found | No source | — | — | LGD and Census cover only standardised administrative names. Non-standard local toponyms for hills and passes — especially in the Northeast and Andamans — exist in no gazetteer. |
| **Mountain range & sub-range polygons** | — none found | No source | — | — | GADM and Natural Earth give administrative polygons, not physiographic ones. The exact extent of a named sub-range within the Western Ghats is not available anywhere. Same for the IS 1893 seismic zone map as a geodataset. |

**Gaps**

- **Official peak elevations.** DEM-derived heights are your only open option and they differ measurably from surveyed values. Publish which method you used per record.
- **Geology** is entirely blocked on GSI Bhukosh being reachable.
- **Caves, sub-ranges, local toponyms, seismic zones** — four genuine no-source areas.

**Which one to build against**

- **Terrain:** Copernicus DEM on S3 — no key, 30 m, whole-country. OpenTopography if you want per-district clips via API.
- **Point elevations:** OpenTopoData, with Open-Elevation as failover.
- **Administrative gazetteer:** LGD. It is canonical and everything else cross-references its codes.
- **Boundaries:** Natural Earth if licence cleanliness matters more than detail; GADM if detail matters more (non-commercial only).
- **Earthquakes:** USGS, decisively — free and real-time against NCS's paid catalog.
- **Named peaks:** re-test Wikidata and Overpass yourself. Both would resolve this category; neither could be confirmed here.

---

## 04. Water Systems

> The best surprise in this survey. India's National Water Data Portal exposes a full open CKAN API with no key — 70 contributing agencies, six-hourly groundwater telemetry, reservoir layers, glacial lakes. It is the successor to India-WRIS and it works. Against that: springs, waterfalls, estuaries and small water bodies returned literally zero results, and real-time discharge for transboundary rivers remains restricted.

| Source | Endpoint | Status | Key | Updates | What it actually gives you |
|---|---|---|---|---|---|
| **National Water Data Portal (NWIC)** | `nwdp.nwic.gov.in/api/3/action/datastore_search?resource_id=…` | API | No | 6-hourly → static | **The single most valuable source in this whole survey.** Open CKAN, no key. One groundwater telemetry resource alone returned 112,074 records with station, agency, LGD state/district codes, river, basin, tributary, lat/long and water level, timestamped to the current day. Catalog counts: 291 river datasets, 309 water-quality, 473 flood, 64 dam/reservoir, 1 glacial lakes layer (GeoJSON/SHP/KML). 70 agencies including CWC, CGWB, CPCB, IMD, NMCG, SAC, NRSC. Licence tagging is inconsistent per dataset — check each one. |
| **HydroRIVERS (HydroSHEDS)** | `data.hydrosheds.org/file/HydroRIVERS/HydroRIVERS_v10_as.gdb.zip` | Bulk | No | Static (v10) | Vectorised river network for all Asia (91–103 MB), reaches ≥10 km² catchment, with length, order and discharge estimates. Free for scientific, educational **and commercial** use with attribution. |
| **HydroLAKES** | `data.hydrosheds.org/file/hydrolakes/` | Bulk | No | Static (v10) | 1.4 million lakes and reservoirs ≥10 ha globally, with surface area, shoreline length and volume estimates. **CC-BY 4.0** — a cleaner licence than HydroRIVERS. |
| **JRC Global Surface Water Explorer** | `global-surface-water.appspot.com/download` | Bulk | No | ~Annual (v1.5) | Forty years (1984–2024) of water occurrence, seasonality, recurrence and transitions as 10°×10° GeoTIFFs, plus Earth Engine collections and WMTS. **"Free of charge, without restriction of use."** Excellent for reservoir and lake extent history. |
| **Randolph Glacier Inventory v7 (NSIDC)** | `daacdata.apps.nsidc.org/pub/DATASETS/nsidc0770_rgi_v7/` | Bulk | No | ~Decadal | Individual glacier outlines and attributes covering High Mountain Asia. Direct HTTPS listing, no registration. Based on imagery circa 2000 — a snapshot, not current state. |
| **GLIMS glacier database** | `glims.org/maps/glims` | Scrape | No | Ongoing | Map viewer with "Download data in current view" and "Download all". Multi-temporal outlines, which complements RGI's single epoch — better for change detection. Interactive, so scripted extraction needs work. |
| **NASA GRACE — groundwater storage** | `grace.jpl.nasa.gov/data/get-data/land-water-content/` | Bulk | No | Monthly | ASCII and NetCDF monthly water storage anomaly. **~300 km footprint** — valid for whole-basin trends (Ganga, Indus) and meaningless locally. Does not separate groundwater from soil moisture without post-processing. |
| **Water Bodies Census 2023 (Jal Shakti)** | `pib.gov.in/PressReleaseIframePage.aspx?PRID=1919482` | Bulk PDF | No | One-off | India's first-ever water bodies census: **2,424,540 water bodies** across 33 states/UTs, 97.1% rural. The only inventory that reaches ponds and tanks below the 10 ha cutoff of HydroLAKES and JRC. Two PDF report volumes, no structured data, no successor cycle announced. |
| **CGWB groundwater monitoring** | `cgwb.gov.in/en/ground-water-level-monitoring` | Scrape | WIMS is restricted | 6-hourly public | Documents the chain: ~25,000 stations and 5,260 digital recorders feed WIMS, which is **restricted to authorised users**, and only a public subset is released via India-WRIS/NWDP every six hours. Confirms NWDP is the correct public access point, not this site. |
| **India-WRIS portal** | `indiawris.gov.in` | Blocked | — | — | Robots-disallowed here, including its Swagger API catalog which search results confirm exists. Largely moot — NWDP is the operational successor and it's open. |
| **National Register of Large Dams** | `dsoindia.org.in · cwc.gov.in/sites/default/files/nrld-2019.pdf` | Dead | — | — | **DNS resolution failure** on the dam safety site; the NRLD 2019 PDF returned **401** from CWC's own server. India's dam register is currently unreachable by both routes. Use NWDP's reservoir layer as the stand-in. |
| **INGRES groundwater estimation** | `ingres.iith.ac.in/gecdataonline/gis/INDIA` | Scrape + JS | — | Annual | Loads as an empty JS shell titled "GecDashboard". Block-level groundwater categorisation (safe/semi-critical/over-exploited) is rendered client-side only. Needs a headless browser. |
| **CWC Flood Forecasting · CPCB water quality · GloFAS · Open-Meteo Flood** | `ffs.india-water.gov.in · cpcb.nic.in/real-time-water-quality · flood-api.open-meteo.com` | Blocked | Varies | Daily | All robots-blocked or JS-shelled here. Open-Meteo's flood API in particular is genuinely open and should work from your own crawler. NWDP already aggregates much of the CPCB water-quality series (309 datasets). |
| **Springs** | — zero results | No source | — | — | Searching NWDP for "spring" returned **zero datasets**. Spring inventories exist in NIH Roorkee field surveys, state groundwater department records and NITI Aayog's Himalayan spring-shed programme — none digitised or published. |
| **Waterfalls · estuaries** | — zero results | No source | — | — | Both returned zero NWDP results. Waterfall data exists only in tourism listings. Estuarine data sits in CWC and National Institute of Oceanography technical reports. |
| **Transboundary river discharge** | — restricted | No source | — | — | Real-time gauge discharge for Ganga/Brahmaputra reaches near Bangladesh and Indus reaches near Pakistan is treated as sensitive and gated. NWDP's river-level datasets are agency-contributed and patchy, not a complete national gauge network. |

**Gaps**

- **Springs, waterfalls, estuaries** — three confirmed zero-result searches on India's own national water catalog.
- **Small and traditional water bodies** — 2.4 million of them counted once, in 2023, in PDF form only.
- **GLOF hazard attribution** — NWDP's glacial lakes layer is geometry only, no outburst risk. The ISRO atlas exists by name with no access route.

**Which one to build against**

- **Everything India-specific:** NWDP. Start here, it is open and it works.
- **River topology and modelling:** HydroRIVERS for globally consistent topology and discharge estimates; NWDP for Indian agency river identity; OSM (once you can reach it) for small named tributaries neither has.
- **Lakes:** HydroLAKES for named polygons ≥10 ha, JRC for how their extent changed over 40 years. Complementary, not duplicative.
- **Groundwater:** NWDP telemetry is the only broadly public near-real-time source. GRACE only for basin-scale trend cross-checks.
- **Glaciers:** RGI v7 for a clean single-epoch inventory, GLIMS if you need change over time.

---

## 05. Living Species

> Superficially the best-served category and actually the most misleading one. GBIF gives you 65.6 million India records through one open API — but 92% are eBird and another 2.5% are iNaturalist, so non-bird coverage is roughly 3.3 million records for every other kingdom combined. India's own authorities, ZSI and BSI, publish nothing machine-readable at all. Fungi, lichens, freshwater invertebrates and vernacular names in Indian languages are effectively absent.

| Source | Endpoint | Status | Key | Updates | What it actually gives you |
|---|---|---|---|---|---|
| **GBIF Occurrence API** | `api.gbif.org/v1/occurrence/search?country=IN` | API | No (read) | Continuous | **65,579,568 India records.** Basis-of-record split: 62.4M human observation, 438,880 preserved specimens, 95,066 material samples, 5,354 fossils. IUCN category attached per record (CR 52,312 / EN 134,444 / VU 398,714 / EX 41). **Licence is per-record**, mixed CC0 / CC-BY / CC-BY-NC-SA — you must carry it through, not assume one blanket licence. |
| **GBIF Backbone / Species API** | `api.gbif.org/v1/species/search?q=Panthera+tigris` | API | No | Continuous | Canonical taxonomy and synonymy — the spine everything else keys against. Full hierarchy, authorship, vernacular name, IUCN status. Note: the backbone is global and **cannot be filtered to an India checklist**; derive that from occurrence facets instead. |
| **GBIF Dataset Registry** | `api.gbif.org/v1/dataset/search?publishingCountry=IN` | API | No | Continuous | 195 datasets from India-based publishers — BNHS reptile and amphibian collections, WII's Keoladeo avifauna (31,936 records), NCF SeasonWatch, the India Biodiversity Portal publication-grade dataset. **Crucially: `type=CHECKLIST` for India returns zero** — no national checklist is registered anywhere on GBIF. |
| **IPNI — plant nomenclature** | `ipni.org/api/1/search?q=…` | API | No | Continuous | Who named a plant, when, and in which publication, cross-linked to World Flora Online IDs. Nomenclature only — no distribution, no status, no occurrences. |
| **POWO — Plants of the World Online (Kew)** | `powo.science.kew.org/api/2/search?q=Ficus+religiosa` | API | No | Continuous | Accepted names, synonymy, common names, herbarium images for vascular plants. Note the old `plantsoftheworldonline.org/api/2` path 302-redirects here. Kew asks that heavy users go through the `pykew` client rather than raw scraping. |
| **Encyclopedia of Life** | `eol.org/api/search/1.0.json?q=tiger` | API | Optional key | Varies | Cross-aggregated species pages with text, images and traits. A downstream aggregator of GBIF and iNaturalist — good as a display layer, **never count it as independent coverage**. (One agent hit a timeout here, another got results; treat as flaky.) |
| **IUCN Red List API v4** | `api.iucnredlist.org` | API + key | Free, restricted | 1–2× yearly | Docs confirm live and state the terms explicitly: token by registration, **"commercial use is strictly forbidden"**, and "misusing your token (e.g. scraping) may result in it being revoked." No India data could be pulled without a key, so no record counts are claimed here. **The old v3 API at `apiv3.iucnredlist.org` is dead — HTTP 525.** |
| **Avibase — India checklist** | `avibase.bsc-eoc.org/checklist.jsp?region=IN` | Scrape | No | Taxonomy updates | **1,396 species, 84 endemics, 94 globally threatened** for India — rendered HTML with PDF export, no API. A checklist, not occurrences: complementary to eBird/GBIF rather than duplicative. |
| **Reptile Database** | `reptile-database.org` | Scrape + bulk | No | Ongoing | Search form plus an explicit `/data/` bulk-download section. No API. The standard global reptile taxonomy reference. |
| **Moths of India** | `mothsofindia.org` | Scrape | No | Continuous | **~3,440 species pages, 55,381 images, 43,587 observations from 3,599 contributors.** Browsable HTML, no API. One of very few genuinely India-specific invertebrate resources. Sister site Butterflies of India (ifoundbutterflies.org) timed out here — retry it. |
| **ENVIS FRLHT — Indian medicinal plants** | `envis.frlht.org` | Scrape | No | Ongoing | Nomenclature, traded medicinal plants, digital herbarium and atlas databases, actively maintained (last update Nov 2024). No API, and the botanical search endpoint returned a **500 error** during testing — flaky. |
| **iNaturalist API** | `api.inaturalist.org/v1` | Blocked | No | Continuous | Robots-blocked here. **Don't bother for occurrences anyway** — 1,622,493 iNaturalist India records are already inside GBIF, verified. Go direct only if you need photos, comments or identification workflow data GBIF doesn't relay. |
| **eBird API 2.0** | `api.ebird.org/v2` | Blocked | Free key | Continuous | Robots-blocked here. Same caution: **60,613,324 eBird India records are already in GBIF.** Use the eBird API only for what GBIF lacks — recent-nearby queries, hotspot metadata, checklist-level effort data. |
| **ChecklistBank · Xeno-canto · NCBI · AmphibiaWeb · Open Tree of Life** | `api.checklistbank.org · xeno-canto.org/api · eutils.ncbi.nlm.nih.gov · amphibiaweb.org/api` | Blocked | Varies | Varies | Robots-disallowed, WAF-blocked (Xeno-canto's BotStopper), timed out, or POST-only (Open Tree, which a GET-only fetcher can't test). All plausibly fine from your own infrastructure. `download.checklistbank.org` exists and serves a `/col` release path but returned 403 to the directory listing. |
| **India Biodiversity Portal** | `indiabiodiversity.org` | Dead / walled | — | — | **403 on the homepage and on `/api/v1/observation/list`; `api.indiabiodiversity.org` does not resolve.** Its data is reachable another way: the portal publishes its publication-grade dataset into GBIF, verified in the dataset registry. Go via GBIF. |
| **FishBase · World Flora Online (list server)** | `fishbase.ropensci.org · list.worldfloraonline.org` | Dead | — | — | Both failed with **self-signed / invalid TLS certificates** — a real server misconfiguration, not a robots policy. Neither is usable as-is. |
| **ZSI — Fauna of India** | `zsi.gov.in` | No source | — | — | The site loads and says **"Website Under Construction."** The Checklist of Indian Animals and ZSI-IMS collection system are named but have no public API; `/faunaofindia/index` 404s. India's official faunal authority publishes priced print volumes and annual "Animal Discoveries" PDFs. |
| **BSI — Flora of India** | `bsi.gov.in` | No source | — | — | Flora of India, fascicles, State Flora and District Flora are listed as **purchasable publications**. No e-Flora database, no API. India's official floral authority is entirely offline. |
| **Fungi, lichens, freshwater invertebrates** | — no national database | No source | — | — | No API and no browsable national database found for any of the three. Lives in journal taxonomic revisions (*Indian Journal of Lichenology*, *Kavaka*), BSI cryptogam herbarium records, and ZSI's Mollusca and Crustacea volumes. Would need OCR and manual compilation. |
| **Vernacular names in Indian languages** | — effectively unavailable | No source | — | — | GBIF and EOL vernacular fields returned English only in every sample. India Biodiversity Portal reportedly carries multilingual names — and its API is the one that 403s. **The single biggest access gap in this category.** |

**Gaps**

- **Non-bird occurrence density.** ~3.3M records for all non-avian taxa in a country of this size is thin. Insect and fungal coverage in particular will look like absence where it is under-recording.
- **No national checklist exists on GBIF** — confirmed by a zero-result query, not assumed.
- **Taxonomic authority is offline.** ZSI and BSI are legally authoritative and digitally absent. Anything you build inherits the taxonomy of international aggregators instead.

**Which one to build against**

- **Ingest GBIF once and only once.** It already contains iNaturalist (1.62M) and eBird (60.6M) India records. Pulling those APIs separately duplicates 95% of your corpus.
- **Exclude eBird and iNaturalist dataset keys** when computing any taxon-coverage statistic, or every chart you make will be a bird chart.
- **Plants:** POWO for accepted names and images, IPNI only for authorship citations. They share WFO identifiers.
- **Conservation status:** IUCN v4 with your own token. GBIF's attached categories are IUCN-derived, so they are not an independent check.
- **Don't ingest separately:** BNHS, WII and India Biodiversity Portal — all three publish through GBIF.

---

## 06. Extinct Species

> The thinnest category by a wide margin, and worth saying plainly: **a list of species extirpated from India does not exist as a queryable source anywhere.** Global extinction status exists (IUCN, behind a token). Occurrence-level evidence exists (GBIF). The India-specific layer between them — what was lost here, and when — has to be compiled by hand. The fossil record is worse: the obvious database forbids API access in its own robots.txt, and India's geological survey portal times out.

| Source | Endpoint | Status | Key | Updates | What it actually gives you |
|---|---|---|---|---|---|
| **GBIF — occurrence evidence for lost taxa** | `api.gbif.org/v1/occurrence/search?country=IN&taxonKey=2498189` | API | No | Continuous | The best available proxy for last-sighting data. Pink-headed duck in India returned **46 real records** — NHMUK 1936.1.16.3, USNM 309005 (1924), Yale specimens from Baghauni, Darbhanga, Bihar (1923), an eBird observation from 1935. Asiatic cheetah returned 14, correctly separating an 1833 museum skull from the 2025 reintroduced Kuno animals. **Per-species manual querying — there is no bulk "last record per extinct Indian species" endpoint.** |
| **GBIF — global extinct flag** | `api.gbif.org/v1/species/search?isExtinct=true` | API | No | Continuous | 1,189,283 taxa globally flagged extinct — but dominated by fossil taxa from source checklists, and **the species endpoint has no country filter**, so it cannot be sliced to India. Per-species lookups work well: *Rhodonessa caryophyllacea* returns `"extinct": true` with the distribution note "Formerly India and Myanmar. Extinct; last reported 1935." |
| **iDigBio — museum specimens** | `search.idigbio.org/v2/search/records/?rq={"country":"india"}` | API | No | Continuous | **529,744 India records.** Different institutional coverage from GBIF — strong on NHMUK fossil holotypes. Sample: *Sonarina tamilensis* and *Dionella coromandalis*, Late Cretaceous bryozoans from the Kallankurichchi Formation, Tamil Nadu. Not filterable by "extinct" — only by taxon, locality or geological age text. |
| **Neotoma Paleoecology Database** | `api.neotomadb.org/v2.0/data/sites` | API | No | Ongoing | API confirmed functional. But Neotoma's Quaternary pollen and sediment-core coverage concentrates in North America and Europe; Indian site density looked thin and was not quantified. Verify with a subcontinent bounding-box query before committing to it. |
| **IUCN Red List — EX / EW / CR(PE)** | `api.iucnredlist.org/api/v4` | API + key | Free, non-commercial | 1–2× yearly | The authority for extinction categories. Returned 403 without a token; the v3 API is dead (525) and the website is robots-blocked. **No India extinct-species count is claimed here because none could be retrieved.** Get a token and verify before relying on it. |
| **Wikipedia — Asian Holocene extinctions** | `en.wikipedia.org/wiki/List_of_extinct_animals_of_India` | Scrape | No | Continuous | A finding in itself: that title **redirects to "List of Asian species extinct in the Holocene"**. There is no India-specific extinct-species article. Content is continent-scoped, many entries lack extinction dates, and it would need manual India-filtering and cross-checking. CC-BY-SA. |
| **EDGE of Existence (ZSL)** | `edgeofexistence.org/species/pygmy-hog/` | Scrape | No | Stale | EDGE and ED scores, population estimates, threat summaries for a curated priority subset. The pygmy hog page cites **IUCN Red List Version 2017.1** — a derivative layer several years behind its own source. Narrow and dated. |
| **Paleobiology Database** | `paleobiodb.org/data1.2/occs/list.json?cc=IN` | Forbidden | — | — | **PBDB's own robots.txt explicitly disallows `/data1.2/`, `/data1.1/`, `/paleodb/data/` and `/public/data/`** — the entire data API. This was confirmed by fetching the robots.txt directly, so it is a deliberate policy, not a sandbox artifact. No India fossil occurrence count can be reported. The Fossilworks mirror returned 403. |
| **GSI Bhukosh — palaeontology layer** | `bhukosh.gsi.gov.in/Bhukosh/MapViewer.aspx` | Dead | — | — | Connection timeout. India's own official fossil and geology map layer could not be loaded at all. Likely needs a live browser session; may have no public REST API regardless. |
| **ZSI Red Data Book** | `zsi.gov.in` | Dead | — | — | Site shows "Website Under Construction". The institutionally correct authority for India-specific extinction status is **non-functional as a digital source right now**. Not a sandbox limitation. |
| **Species+ / CITES Checklist** | `api.speciesplus.net/api/v1/taxon_concepts` | Walled | Registration | Per CoP | Returned 401 — token required, no anonymous endpoint found. The CITES Checklist API path could not be located without further docs access. |
| **Re:wild — Search for Lost Species** | `rewild.org/lost-species` | Scrape | No | Occasional | A narrative landing page with headline figures ("4,300+ lost species"), **no structured data and no Indian species named on it**. Content sits on per-taxon sub-pages. Would need page-by-page scraping for marginal return. |
| **Species extirpated from India** | — must be compiled manually | No source | — | — | Cheetah, pink-headed duck, Sunderbans wild buffalo, rhino species lost from Indian range — none of these are flagged as a category in any database checked. Note GBIF's backbone even **resolves *Bubalus arnee* as a synonym of the domestic *Bubalus bubalis***, conflating the extirpated wild population with livestock. Compilation means cross-referencing IUCN regional assessments, ZSI monographs and species-specific literature by hand. |
| **Indian fossil localities (systematic)** | — GSI publications only | No source | — | — | A systematic Siwaliks / Deccan / Gondwana faunal inventory exists only in GSI Memoirs and the *Palaeontologia Indica* series — printed monographs, not structured data. iDigBio and GBIF give scattered museum type-specimens, not a survey. |

**Gaps — read this one carefully**

- This category is **mostly a manual compilation project**, not a scraping project. Budget researcher time, not spider time.
- The one automatable piece is **per-species GBIF occurrence pulls** to establish last-record dates, once you have a hand-built species list to iterate over.
- **Deep time is closed**: PBDB forbids it, GSI times out. Nothing to build against.

**Which one to build against**

- **Status:** IUCN, with your own token — it is the only authority for EX/EW/CR(PE).
- **Evidence:** GBIF occurrences. But note GBIF's extinct flag is *inherited from IUCN's own checklist*, so the two are not independent confirmation of each other.
- **Fossils:** iDigBio is the only reachable option, and it is a museum-holdings aggregator, not a fossil-occurrence survey.
- **Treat Wikipedia and EDGE as cross-checks only** — both are derived, and the EDGE page tested was citing 2017 data.

---

## 07. Tribal & Culture

> Almost entirely PDFs, and heavily blocked. The Ministry of Tribal Affairs publishes monthly Forest Rights Act progress reports going back to 2018 — real, structured, and every individual PDF was unreachable from this sandbox. Glottolog is the one clean, well-licensed, machine-readable source in the whole category. And Census 2021 never happened: the newest granular tribal demographic data available anywhere is still Census 2011, with the next enumeration now scheduled for 2027.

| Source | Endpoint | Status | Key | Updates | What it actually gives you |
|---|---|---|---|---|---|
| **Glottolog** | `glottolog.org/resource/languoid/id/gond1266` | API + bulk | No | ~Annual (v5.3) | **The best-behaved source in this category.** Machine-readable exports in JSON, RDF, Newick and PhyloXML, source data on GitHub, and an explicit **CC-BY 4.0** licence in the footer. Classification and bibliography for India's tribal and Adivasi languages — but no speaker counts (that's Census C-16's job). |
| **MoTA — FRA monthly progress reports** | `tribal.nic.in/fra.aspx` | Scrape | No | **Monthly** | The index page loads and confirms a real monthly archive: **12 reports per year, 2018 through 2025, with 2026 in progress.** Also ten guideline categories (community forest resource rights, PVTG-specific, PA-related, minor forest produce). The individual MPR PDFs were **all blocked** here, so their exact columns are unconfirmed — but the monthly cadence is solid. |
| **MoTA — Statistics index** | `tribal.nic.in/Statistics.aspx` | Scrape | No | Per Census round | Indexes the ST Statistical Profile, "ST in India as Revealed in Census 2011" (12.25 MB), the state-wise PVTG list, state-wise ST lists and Annual Reports 2004–2024. Filenames and existence confirmed; **the PDFs themselves were blocked**, so contents are unverified. |
| **Ministry of Panchayati Raj — Fifth Schedule Areas** | `panchayat.gov.in/en/state-wise-details-of-notified-fifth-schedule-areas/` | Scrape | No | Very rare | A real embedded table for 10 states — Andhra Pradesh, Chhattisgarh, Gujarat, Himachal Pradesh, Jharkhand, MP, Maharashtra, Odisha, Rajasthan, Telangana — split into fully vs partially covered districts (Surguja, Bastar, Dantewada fully; East Godavari partially). Last updated Feb 2023. Not downloadable; no notification numbers on-page. |
| **Census of India — main portal** | `censusindia.gov.in/census.website/data/census-tables` | Scrape | No | **Decadal — now 2027** | Census Tables for 2011, 2001 and 1991 as Excel collections, plus a village-level Population Finder and Digital Library. The table index is a client-side search widget and test queries returned no matches, so the specific ST-14 / C-16 / C-17 table codes could not be confirmed reachable through this route. |
| **Wikipedia — List of Scheduled Tribes in India** | `en.wikipedia.org/wiki/List_of_Scheduled_Tribes_in_India` | Scrape | No | Continuous | State-by-state tribe tables plus national population, growth rate, sex ratio — sourced to MoTA and the Constitution (ST) Order, and reflecting the **2022 Himachal and 2024 J&K amendments**. In practice **more current than MoTA's own 2013 PDF**. CC-BY-SA. Not citable as law, useful as a change log. |
| **Census NADA microdata catalog** | `censusindia.gov.in/nada/catalog/10191` | Blocked | — | Decadal | Every attempt failed on robots. Search results confirm the structure exists — per-state ST-14 studies, national and state C-16 mother-tongue studies — but download formats, registration and licence terms are unverified. **Note this same host succeeded for the geography agent**, so availability is inconsistent rather than absolute. |
| **Census C-16 mother tongue tables** | `language.census.gov.in/…/C-16_2011.pdf` | Blocked | — | Decadal | Robots/timeout failure. Existence confirmed only through search-result titles. This is the authoritative speaker-count source for every Indian tribal language — worth a determined retry. |
| **forestrights.nic.in** | `forestrights.nic.in` | Dead | — | — | Connection timeout. MoTA's FRA page points here for "live" claims data. Status unconfirmed — may be down or may just be blocked. |
| **Bhuvan FRA Atlas** | `bhuvan-app1.nrsc.gov.in/fra/` | Dead | — | — | 404. The Bhuvan Forest Sector wiki lists nine real forest applications and **none of them is an FRA or claims layer**. The "Bhuvan FRA Atlas" appears in academic references but no live public endpoint could be found. |
| **data.gov.in — tribal category** | `data.gov.in/catalog/data-pertaining-scheduled-tribes-india` | Empty | Free key | Stale | The catalog page exists and describes ST literacy, enrolment and health-infrastructure coverage — and the result panel returned **"No Result Found"**. A metadata shell with no file behind it. (The API does work for other categories; this particular catalog appears empty.) |
| **TKDL — traditional knowledge** | `tkdl.res.in` | Restricted by design | Patent offices only | — | States plainly: "Access to the full database is available to **Patent Offices only** under TKDL Access Agreement." Ayurveda, Unani, Siddha and Sowa-Rigpa formulations, CSIR + Ministry of AYUSH. **This restriction is the point** — TKDL exists as defensive publication against biopiracy. It is not a scraping target. |
| **Sacred groves — national inventory** | `sacredearthtrust.in · kerenvis.nic.in/Database/SacredGroves_1433.aspx` | No source | — | — | Kerala's ENVIS "database" page is a prose article, not records. The Sacred Earth Trust directory is the most honest source found — it states outright that it covers **~14,000 of an estimated 150,000+ groves**, from secondary literature, unevenly (975 in Andhra Pradesh vs 40 in Assam), crowdsourced, with no download. No authoritative coordinate-level national inventory exists. |
| **Anthropological Survey of India** | `ansi.gov.in/profile/` | No source | — | — | Lists Books, Journals, Memoirs, Reports and "Digital Archives" **by name only — no links, no search, no confirmation that the People of India volumes are digitised.** India's ethnographic record is a print archive. |
| **People's Linguistic Survey of India** | `bhasharesearch.org/plsi` | No source | — | — | ~50 planned print volumes via Orient BlackSwan. No online database, no downloads, nothing machine-readable. State tribal research institutes are similar: Odisha's SCSTRTI has some monographs on archive.org, uploaded by third parties, not an institutional portal — and every other state would need separate checking. |

**Gaps**

- **Village-level FRA data does not exist openly.** MoTA publishes state-level monthly aggregates. Claimant-level and village-level records are not public, and shouldn't be.
- **Census 2021 never happened.** Enumeration is now set for 2027, so every granular ST figure you can source is fifteen years old. Treat any claim of newer granular ST population data with suspicion unless it's a survey like NFHS.
- **Sixth Schedule areas** (Assam, Meghalaya, Tripura, Mizoram autonomous councils) — no authoritative digital list was found or confirmed either way.
- **Ethnography and oral history** — print only, across AnSI volumes, state TRI monographs and the PLSI series.

**Which one to build against — and what not to**

- **Legal ST list:** only the Constitution (ST) Order and its amendment Acts. MoTA and NCST mirror it; Wikipedia tracks amendments faster than MoTA's own PDF but is not citable as law.
- **Population:** Census of India, exclusively. MoTA's Statistical Profile and data.gov.in's ST catalog are both repackagings of the same 2011 numbers — none adds independent enumeration.
- **Languages:** Glottolog for classification, Census C-16 for speaker counts. They answer different questions.
- **Do not publish, even where technically reachable:** precise sacred-site coordinates (aggregate to district level — pinpointing invites looting and unwanted tourism at places communities may deliberately keep undisclosed); traditional medicinal knowledge (building an open scraped equivalent would undo exactly what TKDL's restricted-access model protects); and any individual or village-level FRA claimant data. State and district aggregates are the fair open-data line here.

---

## 08. Laws & Land Management

> Two structural problems. India Code — the only authoritative full-text repository for central *and* state Acts — is currently broken on both its old and new domains, with the old one redirecting to a new one that fails TLS verification. And the court and land-record sources are deliberately hardened: the Supreme Court and NGT judgment searches are CAPTCHA-gated, state land-record portals are JS shells, and CPCB robots-blocks its own document server. Read the terms note at the bottom of this section before pointing a spider at any of it.

| Source | Endpoint | Status | Key | Updates | What it actually gives you |
|---|---|---|---|---|---|
| **e-Gazette** | `egazette.gov.in` | Scrape | No (read) | **Daily** | Server-rendered, working. Keyword categories (Acts, Bills, Elections), "Gazettes on Demand" including Land Acquisition, central and state gazette sections, ministry-wise recent listings, date-organised tables. **The primary-source scan of every notification — upstream of India Code and MoEFCC alike.** Registration is only for publishing, not reading. |
| **MoEFCC notifications & rules** | `moef.gov.in/wildlife-notification` | Scrape | No | Irregular | Real navigation tree with distinct working URLs for Environment Protection Rules, Forest Conservation Rules, ESZ/ESA notifications, Wildlife Division, GEAC approvals. **Each notification is its own static page with full inline text** — verified on S.O.1092(E), the 2003 National Board for Wildlife Rules. No index, no RSS, so you'll need periodic re-crawls. |
| **PRS Legislative Research — Bill Track** | `prsindia.org/billtrack` | Scrape | No | Per session | Server-rendered bill list with status labels. Confirmed entries: Forest (Conservation) Amendment Bill 2023 — Passed; Water (Prevention and Control of Pollution) Amendment Bill 2024 — Passed; Coastal Aquaculture Authority (Amendment) Bill 2023. **Bill text and legislative history, not enacted Act text.** |
| **Environment Clearance portal** | `environmentclearance.nic.in` | Scrape | No (read) | **Continuous** | Live aggregate statistics returned: **34,772 total proposals**, broken down by fresh EC / validity extension / amendment / corrigendum, and by status (Stage-II, under process, rejected, withdrawn). **The most scrapeable entry point for EC data right now** — more so than PARIVESH itself. One sub-page hit an SSL error; retry. |
| **e-Green Watch — CAMPA & forest diversion** | `egreenwatch.nic.in` | Scrape | No | Periodic | Real numbers returned: **42,225 FCA-1980 projects, 427,177 ha of forest land diverted, 39,383 compensatory-afforestation parcels covering 676,964 ha** across 34,603 work sites, plus 115,607 non-CA plantation sites and year-wise GoI fund releases to states. **The only public CAMPA fund and compensatory-afforestation dataset found.** State breakdowns sit behind a selector. |
| **Indian Kanoon — judgment search** | `indiankanoon.org/search/?formInput=…` | Scrape | Paid API exists | Near-daily | Real results — "1–10 of 3083" for an NGT forest query — with case title, court, date, authoring judge, citation counts and snippets, paginated via `&pagenum=`. **No CAPTCHA on the free web interface**, which makes it the most practical cross-court judgment source. It is a re-publisher, not the court of record. |
| **NGT — legacy judgment table** | `greentribunal.in/judgement.php` | Scrape | No | Uncertain | Plain server-rendered HTML, **no CAPTCHA**: columns for appeal number, parties, date, judgment PDF, paginated across 13 pages via `?Form_Page=`. But the content seen was **2012–13 era** — likely a frozen legacy mirror. Verify freshness before trusting it for current judgments. |
| **NGT — official search** | `greentribunal.gov.in/judgementOrder/case-advance-search` | CAPTCHA | — | Weekly | Homepage is server-rendered with full navigation, but both the advance search and the zonal-bench judgment search are **CAPTCHA-gated with no pre-search list view**. Deliberate anti-bulk-scraping control. |
| **Supreme Court of India — judgment search** | `sci.gov.in/judgements-judgement-date/` | CAPTCHA | — | Daily | Server-rendered form with date range, diary number, case number, judge and free-text tabs — and **CAPTCHA verification required before any result**. Note `main.sci.gov.in` no longer resolves; the domain has moved to `sci.gov.in`. |
| **India Code** | `indiacode.nic.in → indiacode.gov.in` | Dead | — | — | **The biggest single gap in this category.** The old domain now serves only a migration notice with a JS redirect; all old `/handle/` and `/bitstream/` paths 404. The new domain fails repeatedly with **SSL hostname-mismatch and missing-issuer errors**, and the PDF subdomain times out. Both routes to India's authoritative Act repository are down. |
| **PARIVESH 2.0** | `parivesh.nic.in` | Scrape + JS | — | Continuous | Returns only `"You need to enable JavaScript to run this app"` — a pure React stub with no data or API references in static HTML. The legacy `cpc.parivesh.nic.in` wildlife-clearance search is server-rendered ASP.NET but showed only the dashboard shell unauthenticated. |
| **Forests Clearance portal** | `forestsclearance.nic.in` | Dead | — | — | Failed over both HTTP and HTTPS with **certificate-issuer errors**. Forest clearance data has no confirmed working route — unlike environment clearance, which does. |
| **DILRMP · UP Bhulekh · Bhu-Naksha** | `dilrmp.gov.in · upbhulekh.gov.in · bhunaksha.nic.in` | Scrape + JS | — | Varies | DILRMP's dashboard rendered every KPI as **"0" with "Loading… Please Wait…"**. UP Bhulekh returned an **empty shell with no body content and no form**. Bhu-Naksha serves a landing page describing a tool for authorised officials, with no state selector; `bhunaksha.gov.in` doesn't resolve at all. Land records will need a headless browser per state, and many add CAPTCHAs. |
| **National Judicial Data Grid** | `njdg.ecourts.gov.in` | Scrape + JS | No | Near real-time | A JS app, but this fetch caught rendered aggregates: **1,12,94,679 civil and 4,07,41,214 criminal cases pending, 5,20,35,893 total**, plus monthly institution/disposal and age-wise pendency across a 36 state/UT selector. Drill-downs need live JS. Statistics, not judgment text. Note `/njdgv3/` is a dead path. |
| **data.gov.in — clearance & legal datasets** | `api.data.gov.in/resource/{id}?api-key=(secret removed) | API + key | Free key | Irregular | Confirmed working. Includes an "Environmental Clearance granted (PARIVESH)" catalog entry and wildlife-clearance meeting records. **The `/resource/` web paths are robots-disallowed to crawlers — the intended access route is the registered API key.** |
| **OpenNyAI legal NLP datasets (HuggingFace)** | `huggingface.co/datasets?search=opennyai` | Bulk | No | Static | Five real datasets: `InLegalNER` (16.6k samples), `InRhetoricalRoles` (327), `InJudgements_dataset` (12k), `aalap_instruction_dataset` (22.3k), `aibe_dataset` (1.16k). Pre-labelled training data derived from Indian judgments — useful for processing what you collect, not a source of new judgments. Last updated 2023–24. |
| **CPCB** | `cpcb.nic.in` | Scrape (limited) | No | Varies | Site is up, but its **document-serving endpoints (`openpdffile.php`, `displaypdf.php`) are robots-disallowed**, and no consent-management or enforcement search page could be verified. State pollution control boards were not individually surveyed and will vary widely. |
| **Consolidated state forest & land rules** | — scattered across state law departments | No source | — | — | No single source exists. India Code covers state Acts when it's up; state forest rules and land revenue codes live on dozens of individual state department sites, none surveyed here. |
| **Amendment history / diffs** | — no structured source | No source | — | — | Nothing provides a machine-readable amendment graph — "Rule X amended by notification Y on date Z, superseding version W". e-Gazette is the raw material; building the graph means parsing gazette PDFs across years. |
| **Protected-area enforcement data** | — no central source | No source | — | — | Patrol effort, encroachment and enforcement at the level of an individual protected area has no central public API. NTCA and Chief Wildlife Warden orders are scattered. |

**Gaps**

- **India Code being down** is the load-bearing failure here. MoEFCC's per-notification pages plus PRS partially cover central environmental law, but nothing substitutes for consolidated as-amended text across all states.
- **Forest clearance** has no working route; **environment clearance** does. Don't assume symmetry.
- **Land records** are per-state, JS-rendered, often CAPTCHA'd. Confirmed on two of them; assume the pattern holds.

**Which one to build against**

- **Primary legal text:** e-Gazette. It is upstream of everything, including India Code's own consolidated versions, and it is the first place a new notification appears.
- **Consolidated as-amended Acts:** India Code, once it comes back. Nothing else does this job.
- **Judgments:** Indian Kanoon for practical coverage — the court sites are the record but are CAPTCHA'd. SCC Online and Manupatra are paywalled commercial re-publishers, not tested here.
- **Clearances:** `environmentclearance.nic.in` over PARIVESH for EC; e-Green Watch for CAMPA and forest diversion.

**Terms of use — read before writing spiders**

Nothing tested here surfaced an explicit written "no scraping" clause, but the controls in place amount to one. The Supreme Court and NGT gate judgment search behind CAPTCHAs, which is a deliberate anti-bulk-collection measure rather than incidental UI. CPCB robots-disallows its own document-serving endpoints. data.gov.in robots-disallows its `/resource/` web paths while operating a free registered API — meaning the API is the intended route and scraping the UI is not. PBDB, in category 06, disallows its entire data API in robots.txt. For all of these, the honest options are the official API, a data-sharing agreement, or authorised manual access — not an unattended spider. Where a source offers a key, take the key.

---

## Method note

Every source above was fetched live on 6 September 2026 from a cloud sandbox whose HTTP client respects `robots.txt` and sits behind a filtering proxy. Sources marked **Blocked** were refused at that layer, not by the source, and should be re-verified from your own infrastructure before you act on them. Record counts, HTTP status codes and quoted licence terms are what the responses actually returned; where a key was required and unavailable, no counts are claimed. Nothing here is included on reputation alone.
