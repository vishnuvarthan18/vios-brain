---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/overpass.yaml
---

name: overpass
description: >
  Overpass API (OpenStreetMap) — queries water features (rivers, streams,
  lakes/ponds/reservoirs, wetlands, springs) within each reserve's bbox.
  Public, no auth. Run per reserve using scripts/run_api_spider.py, same
  pattern as gbif/ebird/inaturalist — the {bbox_min_lon}/etc placeholders
  below are filled in at run time from that reserve's bbox_min_lon/
  bbox_min_lat/bbox_max_lon/bbox_max_lat columns, never hardcoded to one
  reserve here.

  Query verified live 2026-08-27 against Bandipur's real bbox
  (11.5945587,76.2040472,11.9795887,76.8880758, sourced from Nominatim —
  see tests/fixtures.py): 895 water-feature elements returned, including
  named rivers (Moyar, Nugu, Gundlu) and reservoirs/ponds/lakes/streams.
  The `data=` query param is percent-encoded Overpass QL — the
  {bbox_min_lon}/etc placeholders survive percent-encoding as literal text
  (`%7Bbbox_min_lon%7D`) and are substituted by api_source.py's normal
  `url.format(**bbox)` call before the request is sent, same mechanism
  every other api_source.py source already uses for its own placeholders.
kind: api
spider: api_source
base_url: https://overpass-api.de/api/interpreter
license: "OpenStreetMap contributors, Open Database License (ODbL) — https://www.openstreetmap.org/copyright"
terms_url: https://wiki.openstreetmap.org/wiki/Overpass_API
rate_limit_ms: 5000
rate_limit_justification: >
  Overpass API's own wiki (wiki.openstreetmap.org/wiki/Overpass_API)
  states "a rate limit is in place" without a documented fixed number, and
  asks large-scale users to notify the project in advance. 5s/request is
  deliberately more conservative than this project's other api_source
  defaults (GBIF/iNaturalist/eBird all use 1000ms) — live testing during
  this build hit an HTTP 504 "server is probably too busy" response after
  a burst of manual test queries in short succession, direct evidence this
  shared public instance is more load-sensitive than the others.
auth: none
robots_txt_verified: "2026-08-27: verified live at overpass-api.de/robots.txt — `Disallow: /api/` blanket-blocks this source's own request path (/api/interpreter). This is Overpass's own documented, sanctioned query endpoint (wiki.openstreetmap.org/wiki/Overpass_API describes /api/interpreter as the API surface, with a rate limit, not a page meant for exclusion) — the disallow reads as generic search-engine-crawler boilerplate, same pattern as this project's eBird/iNaturalist sources. respect_robots_txt: false set below."
respect_robots_txt: false
notes: >
  bbox params are computed at run time from the reserve table's
  bbox_min_lat/bbox_min_lon/bbox_max_lat/bbox_max_lon — never hardcoded.
  [bbox:south,west,north,east] is Overpass QL's own bbox-filter form (note
  the lat,lon order differs from GeoJSON's lon,lat — verified against the
  live query above). Query covers waterway=river/stream (ways),
  natural=water (way+node, covers OSM's water=lake/pond/reservoir
  sub-tagging), natural=wetland (way+node), natural=spring (node) — chosen
  to be a single combined query per reserve rather than one query per
  feature type, to minimize request count against a rate-limited shared
  public instance. `out center tags` returns each way's centroid (not full
  geometry) plus all its tags — sufficient for a point-representable
  water_body row per db/migrations/0002_full_hierarchy.sql's schema
  (lat/lon columns, geojson TEXT is optional/nullable); full way geometry
  (`out geom`) was deliberately not requested, to keep response size and
  server load down, since the schema doesn't require it.

  OSM completeness varies significantly by area — verified live that
  Sathyamangalam is well-mapped as a `boundary=protected_area` relation in
  OSM, but Bandipur/Mudumalai (adjacent, similarly major tiger reserves)
  are NOT tagged with any protected-area boundary relation under that name
  in OSM (confirmed via a scoped Overpass query, not a lookup mistake) —
  their locations were instead resolved via Nominatim's own `way` match
  (see tests/fixtures.py). This means OSM's water-feature density inside a
  reserve's bbox is itself uneven and a function of how much local mapping
  attention that specific park has received, not the actual number of
  water features on the ground — treat a reserve with few results as
  possibly under-mapped, not necessarily water-scarce.

requests:
  - name: overpass-water-features
    url: "https://overpass-api.de/api/interpreter?data=%5Bout%3Ajson%5D%5Btimeout%3A60%5D%5Bbbox%3A{bbox_min_lat}%2C{bbox_min_lon}%2C{bbox_max_lat}%2C{bbox_max_lon}%5D%3B%28way%5B%22waterway%22%3D%22river%22%5D%3Bway%5B%22waterway%22%3D%22stream%22%5D%3Bway%5B%22natural%22%3D%22water%22%5D%3Bnode%5B%22natural%22%3D%22water%22%5D%3Bway%5B%22natural%22%3D%22wetland%22%5D%3Bnode%5B%22natural%22%3D%22wetland%22%5D%3Bnode%5B%22natural%22%3D%22spring%22%5D%3B%29%3Bout%20center%20tags%3B"
    method: GET
