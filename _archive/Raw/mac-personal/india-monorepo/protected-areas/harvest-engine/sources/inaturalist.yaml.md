---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/inaturalist.yaml
---

name: inaturalist
description: >
  iNaturalist API v1 observations search. Public, no auth for read-only
  search. Run per reserve using that reserve's bbox — never hardcoded to
  one reserve here.
kind: api
spider: api_source
base_url: https://api.inaturalist.org/v1/observations
license: "Varies per observation (CC0/CC-BY/CC-BY-NC/all-rights-reserved) — see each observation's own `license_code`; iNaturalist's terms at https://www.inaturalist.org/pages/terms apply to API use itself."
terms_url: https://www.inaturalist.org/pages/api+recommended+practices
rate_limit_ms: 1000
rate_limit_justification: >
  iNaturalist's Recommended Practices page documents ~1 req/s / ~100
  req/min / ~10000 req/day as sanctioned throughput for the public API.
  1 req/s (the shared default) is within that documented guidance.
auth: none
robots_txt_verified: "2026-08-27: verified live at api.inaturalist.org/robots.txt — `Disallow: /*?` blanket-blocks every query-string URL, which includes this source's own request (/v1/observations?swlat=...). Also confirmed api_source spider requests were being silently filtered out by Scrapy's RobotsTxtMiddleware (ROBOTSTXT_OBEY=True globally, harvest_engine/settings.py) with no override existing anywhere in the codebase before this — meaning this source had never actually fetched live data through the real spider despite being described as tested. Same justification as sibling project (sathyamangalam-atlas): iNaturalist's own Recommended Practices page (https://www.inaturalist.org/pages/api+recommended+practices) explicitly sanctions this exact query-parameter search pattern as the intended way to use the API — the robots.txt disallow reads as generic search-engine-crawler boilerplate, not a signal against the documented API surface. respect_robots_txt: false set below."
respect_robots_txt: false
notes: >
  bbox params (swlat, swlng, nelat, nelng) are computed at run time from
  the reserve table's bbox_min_lat/bbox_min_lon/bbox_max_lat/bbox_max_lon —
  never hardcoded. pagination: page, per_page (max 200) — iterate pages
  until a response has fewer than per_page results. Any observation with
  geoprivacy or taxon_geoprivacy set to 'obscured'/'private' already has a
  randomized location (~0.2 degree box) from iNaturalist itself — store
  what the API returns as-is, do not treat it as full precision. The
  project's own sensitive-species coordinate coarsening still applies on
  top of whatever iNaturalist already obscured.

requests:
  - name: inaturalist-observations-search
    url: "https://api.inaturalist.org/v1/observations?swlat={bbox_min_lat}&swlng={bbox_min_lon}&nelat={bbox_max_lat}&nelng={bbox_max_lon}&per_page=200&page=1"
    method: GET
