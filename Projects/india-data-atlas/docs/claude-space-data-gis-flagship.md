# Flagship GIS project: satellite data (researched 2026-09-21)

Visual page: https://claude.ai/artifact/6EzvxSyNTVpawGAGpfoAdB

**Project:** India Protected Area Health Index. One score for each of the 602 protected areas, built from 8 satellite signals plus the platform's ground data. It updates on its own through a new "Space" engine (GEE zonal stats → Core API → PostGIS → web app map).

**Datasets (all free for research):** Landsat 8/9; Hansen GFC v1.13 (2000–2025); GEDI lidar; FIRMS (MODIS/VIIRS); Black Marble night lights; GRACE/GRACE-FO; GPM IMERG; SRTM/NASADEM; NISAR L-band (public via ASF) and S-band (ISRO Bhoonidhi); Sentinel-1/2 (Copernicus Data Space); ESA WorldCover (already ingested); ISRO Bhoonidhi (Resourcesat/Cartosat); JAXA ALOS-2 PALSAR-2.

**Why it's novel:** published studies are single-park or single-region (Kaziranga, Western Ghats, Bannerghatta). A health score for all Indian protected areas, updated automatically and citing its sources, is much rarer.

**Learning path (user is new to GIS):** NASA ARSET fundamentals → IIRS e-learning → sign up for GEE (noncommercial) → Hansen forest loss for 5 protected areas → scale to all 602.

**Risks:** GRACE is ~300 km, so regional only. Monsoon cloud breaks optical data, so use SAR. GEE has quotas. Keep only per-area results on the 4 GB VPS. OSM protected-area shapes should be cross-checked with WDPA.
