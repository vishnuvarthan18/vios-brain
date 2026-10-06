---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/gbif.yaml
---

name: gbif
description: >
  GBIF (Global Biodiversity Information Facility) Occurrence Search API.
  Public, no auth. Run once per reserve — the {bbox_*} placeholders below
  are filled from that reserve's bbox_min_lon/bbox_min_lat/bbox_max_lon/
  bbox_max_lat columns at run time by scripts/run_api_spider.py, never
  hardcoded to a single reserve here. India-wide: iterate over every
  reserve row, not one bounding box.
kind: api
spider: api_source
base_url: https://api.gbif.org/v1/occurrence/search
license: "Varies per record (CC0, CC-BY, CC-BY-NC) — see each occurrence's own `license` field returned by the API; GBIF's aggregated-dataset reuse policy at https://www.gbif.org/citation-guidelines also applies."
terms_url: https://www.gbif.org/terms
rate_limit_ms: 1000
rate_limit_justification: >
  GBIF does not publish a fixed numeric rate limit for /occurrence/search
  (only an internal, undocumented Varnish gauge is exposed — no
  Retry-After/X-RateLimit-* contract). 1 req/s is a conservative, safe
  default, not a documented allowance.
auth: none
robots_txt_verified: "2026-08-27: verified live at api.gbif.org/robots.txt — only disallows /v1/image/unsafe (unrelated to this source's /v1/occurrence/search endpoint). No override needed."
notes: >
  Prefer geometry (WKT polygon) over decimalLatitude/decimalLongitude
  ranges when the reserve has a boundary_geojson, to avoid pulling
  occurrences from outside the reserve boundary. offset+limit is capped at
  100000 for /occurrence/search — use the async Downloads API beyond that,
  not implemented here yet. Sensitive-species coordinate handling: GBIF
  itself may already generalize coordinates for some taxa; do not treat a
  returned coordinate as necessarily full-precision.

requests:
  - name: gbif-occurrence-search
    url: "https://api.gbif.org/v1/occurrence/search?geometry=POLYGON(({bbox_min_lon}%20{bbox_min_lat},{bbox_max_lon}%20{bbox_min_lat},{bbox_max_lon}%20{bbox_max_lat},{bbox_min_lon}%20{bbox_max_lat},{bbox_min_lon}%20{bbox_min_lat}))&limit=300"
    method: GET
