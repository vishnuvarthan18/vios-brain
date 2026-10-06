---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/ebird.yaml
---

name: ebird
description: >
  eBird API 2.0 (Cornell Lab of Ornithology) recent-observations-by-geo
  endpoint — the spec's specified additional source for bird occurrences.
  Requires an API key (env.EBIRD_API_KEY, requested at
  ebird.org/api/keygen — never hardcoded). Run per reserve, using the
  reserve's bbox centroid + a covering radius, computed at run time — never
  hardcoded to one reserve.
kind: api
spider: api_source
base_url: https://api.ebird.org/v2/data/obs/geo/recent
license: "eBird Terms of Use (https://www.birds.cornell.edu/home/ebird-data-access-terms-of-use/) — data used per those terms, not redistributed publicly beyond this project's own D1/site."
terms_url: https://www.birds.cornell.edu/home/ebird-data-access-terms-of-use/
rate_limit_ms: 1000
rate_limit_justification: >
  eBird does not publish an exact per-second number; community-reported
  soft daily ceiling (~1000 req/day, unofficial) applies rather than a
  documented burst rate. Using 1 req/s as a conservative floor.
auth: (secret removed) header, value from env.EBIRD_API_KEY — request a key at ebird.org/api/keygen before first run"
auth_header:
  name: X-eBirdApiToken
  value: env.EBIRD_API_KEY
robots_txt_verified: "2026-08-27: verified live at api.ebird.org/robots.txt — blanket `Disallow: /` for the entire host. Also confirmed api_source spider requests were being silently filtered out by Scrapy's RobotsTxtMiddleware (ROBOTSTXT_OBEY=True globally, harvest_engine/settings.py) with no override existing anywhere in the codebase before this — meaning this source had never actually fetched live data through the real spider. Same justification as sibling project (sathyamangalam-atlas): this is an authenticated, key-gated API (X-eBirdApiToken, obtained by explicit registration at ebird.org/api/keygen), not a public crawlable site — a blanket robots.txt disallow on an entire API host that requires per-developer key registration reads as boilerplate aimed at web crawlers, not a restriction on the registered, sanctioned API usage the key itself grants. respect_robots_txt: false set below."
respect_robots_txt: false
notes: >
  eBird has no native bbox endpoint — params.geo below (lat, lng, dist) are
  computed at run time from each reserve's bbox centroid + a radius that
  covers the reserve (max dist = 50km per the API). back = days back
  (max 30) — this endpoint only returns RECENT observations, not a full
  historical archive; a full-archive pull would need eBird Basic Dataset
  access (out of scope until specifically requested). If EBIRD_API_KEY is
  missing/revoked at run time, the run must fail loudly for this source,
  not silently skip it — fail-one-continue-all applies to OTHER sources
  continuing, not to fabricating or silently omitting this one.

requests:
  - name: ebird-obs-geo-recent
    url: "https://api.ebird.org/v2/data/obs/geo/recent?lat={reserve_centroid_lat}&lng={reserve_centroid_lon}&dist=25&back=30&maxResults=10000"
    method: GET
