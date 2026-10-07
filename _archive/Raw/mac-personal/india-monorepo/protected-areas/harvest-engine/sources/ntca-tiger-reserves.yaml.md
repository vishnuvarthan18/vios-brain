---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/ntca-tiger-reserves.yaml
---

name: ntca-tiger-reserves
description: >
  National Tiger Conservation Authority — live list of all Tiger Reserves
  in India with notification years, core/buffer/total area, and links to
  each reserve's brief note + gazette notification PDF. Verified live
  2026-08-27: 58 tiger reserves (61 table rows including a few un-numbered
  satellite/buffer-unit rows and one trailing totals row, which the parser
  skips).
kind: html
spider: seed_list
parser: ntca_tiger_reserves
base_url: https://ntca.gov.in
license: "Government of India content — no blanket reuse license published on the page; treat as All Rights Reserved pending explicit confirmation, same posture as other .gov.in sources here."
terms_url: https://ntca.gov.in
rate_limit_ms: 2000
auth: none
robots_txt_verified: "2026-08-27: not yet independently checked at ntca.gov.in/robots.txt — verify before the first scheduled (non-manual) run and update this field with the result."
user_agent_override: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
notes: >
  IMPORTANT: this site rejects requests carrying only the bare "Mozilla/5.0"
  user agent (returns an empty/blocked response, confirmed 2026-08-27) —
  user_agent_override above is required, not optional, for this source.
  The page also embeds ~19 unrelated <table class="eco-reserve-table">
  sub-tables (per-state breakouts elsewhere on the page); the parser
  targets the distinctly-classed <table class="sanctions-table ..."> for
  the authoritative national list, not a table index, so it survives the
  eco-reserve-table count changing. This source only covers Tiger
  Reserves — National Parks/Wildlife Sanctuaries/Conservation Reserves/
  Community Reserves/Biosphere Reserves come from
  wii-gazette-notifications.yaml.

requests:
  - name: ntca-tiger-reserves-index
    url: https://ntca.gov.in/tiger-reserves/
    method: GET
