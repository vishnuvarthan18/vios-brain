# GIS Master Plan — learn and build together, zero spend (2026-09-22)

Read this first for anything GIS. Related docs: space-data-gis-flagship.md, webapp-gis-first-plan.md, academic-project-subjects.md.
Visual pages: subjects https://claude.ai/artifact/K1w2So9CW4x12GYUzi68vc · space data https://claude.ai/artifact/6EzvxSyNTVpawGAGpfoAdB · platform https://claude.ai/artifact/Gigs4dZRp1RknS73mdJDQZ

## Decisions
- **Zero spend.** Only free courses, free tools and free data. No paid Geoversity/ENVI courses.
- **GIS first.** The web app starts as a map.
- **No public API for now.** The web app reads Postgres directly through a read-only role, hosted separately from the 4 GB VPS.
- **Flagship project:** India Protected Area Health Index. One score for each of the 602 protected areas, from satellite data plus our own ground data.
- **Learn by doing.** Every topic learned is applied to our own parks straight away.

## Free tools
QGIS (desktop) · Google Earth Engine (noncommercial) · Python (GeoPandas, Rasterio) · ESA SNAP (radar) · EnMAP-Box (hyperspectral) · NetLogo (agent-based modelling) · OpenDroneMap (drones) · PostGIS (have it) · Martin + MapLibre (web map)

## Free satellite data
Landsat 8/9, Hansen GFC v1.13 (2000–2025), GEDI, FIRMS, Black Marble, GRACE-FO, GPM IMERG, SRTM/NASADEM, NISAR (ASF + Bhoonidhi), Sentinel-1/2 (Copernicus Data Space), ESA WorldCover (have it), ISRO Bhoonidhi, JAXA ALOS-2 PALSAR-2

## Geoversity topics → free route
| Topic | Free route |
|---|---|
| EO for nature | Geoversity EO for Ecosystem Conservation I & II |
| Free satellite data | Geoversity Copernicus/Sentinel, ClimateSERV, Collect Earth Online |
| SAR / radar | NASA ARSET SAR trainings + ESA SNAP |
| Spectral RS | NASA ARSET fundamentals + QGIS/Python |
| Hyperspectral | EnMAP-Box + EnMAP data |
| ENVI | Replaced by QGIS + Python + GEE |
| Web GIS | Geoversity Geo Web App (free) |
| Disaster / risk | Geoversity Holistic Decision Making (free) + FIRMS + GPM |
| Agent-based modelling | Geoversity Intro to ABM (free, NetLogo) |
| Ethics | GeoTechE, Do No Harm, Auditing Public Funds (free) |
| Drones | Geoversity UAVs in Precision Agriculture (free) + OpenDroneMap |
| Leadership | Skipped |

## The plan — 6 phases

**Phase 0 — Setup (week 1)**
Install QGIS. Sign up for GEE (noncommercial), NASA Earthdata, Copernicus Data Space and Bhoonidhi. All free.

**Phase 1 — Basics (weeks 2–4)**
Learn: NASA ARSET Fundamentals of Remote Sensing; Geoversity EO for Ecosystem Conservation I.
Build: load our 602 protected areas into QGIS and make the first map.

**Phase 2 — First real result (weeks 5–8)**
Learn: spatialthoughts GEE course; Geoversity EO Conservation II; Geoversity Copernicus course.
Build: forest loss 2000–2025 (Hansen) and NDVI (Sentinel-2) for 5 test parks.

**Phase 3 — All 8 signals (weeks 9–16)**
Learn: ARSET SAR + SNAP; ClimateSERV; Collect Earth Online (for accuracy checks).
Build: add GEDI height, FIRMS fire, Black Marble lights, GPM rain, GRACE, Sentinel-1/NISAR radar. Check against Collect Earth Online samples.

**Phase 4 — Health Index for all 602 parks (weeks 17–22)**
Build: combine the signals with AHP weights to get one score per park. Build a new "Space" engine: a monthly job runs GEE zonal stats → Core API → PostGIS. Store only the results per park, never raw imagery.

**Phase 5 — Web app map (weeks 23–30)**
Learn: Geoversity Geo Web App course.
Build: MapLibre + Martin map showing the Health Index and all layers (peaks, water, species, tribes, languages). Keep the district-precision rule for sensitive data.

**Phase 6 — Extra depth (ongoing)**
Ethics courses (tribal and sacred-site data). OSM vs WDPA comparison (needs the free WDPA key). Springs prediction. Agent-based model of tourist pressure. Hyperspectral with EnMAP.

## Still needed from the user (unchanged)
Register the 5 API keys (WDPA, IUCN, GeoNames, OpenTopography, GFW), rotate DATA_GOV_IN_API_KEY, deploy the ops console.

## Risks
GRACE is ~300 km, so regional only. Monsoon cloud breaks optical data, so use radar. GEE has quotas. The 4 GB VPS must never hold raw images. OSM shapes need a WDPA cross-check before any world-class claim.
