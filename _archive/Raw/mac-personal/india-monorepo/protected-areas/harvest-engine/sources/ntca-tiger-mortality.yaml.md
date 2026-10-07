---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/ntca-tiger-mortality.yaml
---

name: ntca-tiger-mortality
description: >
  NTCA's live Tiger Mortality page — per-incident records (date, state,
  sex, age, location, tiger reserve, inside/outside reserve, whether a
  wildlife-crime seizure was made) covering 2021 through the present.
  Verified live 2026-08-27: 8 <table> elements sharing an identical header
  row (Sl No/Date/State/Sex/Age/Location/Tiger Reserve/Inside or Outside
  Tiger Reserve/Whether seizure) — tables 0/1/2 cover 2021/2022/2023
  uniquely; table 3 (433 rows) is a combined 2024-through-present table
  that already contains everything in tables 4/5/6/7 (which are exact
  per-year/duplicate subsets of table 3's own date range — verified by
  comparing first/last dates per table). The parser (see
  harvest_engine/spiders/seed_list.py's parse_ntca_tiger_mortality)
  therefore only extracts tables 0, 1, 2, and 3, and further dedupes by
  (date, state, tiger_reserve, location) since table 3 alone spans
  2024-2026 and could itself contain incidental duplicate rows.

  Confirmed present for all 3 reference reserves (Bandipur, Mudumalai,
  Sathyamangalam) across multiple years, including at least one row with
  "Whether seizure" = Yes for each — see scripts/normalize_threats.py for
  why only seizure=Yes rows become a `threat` row (the table records
  mortality + seizure status, not cause of death — a seizure=No mortality
  is NOT necessarily poaching and storing it as one would be fabricating a
  claim the source doesn't make).
kind: html
spider: seed_list
parser: ntca_tiger_mortality
base_url: https://ntca.gov.in
license: "Government of India content — no blanket reuse license published on the page; treat as All Rights Reserved pending explicit confirmation, same posture as other .gov.in sources here."
terms_url: https://ntca.gov.in
rate_limit_ms: 2000
auth: none
robots_txt_verified: "2026-08-27: verified live at ntca.gov.in/robots.txt — only disallows /wp-admin/ (this page is not under that path). Same domain/posture as sources/ntca-tiger-reserves.yaml."
user_agent_override: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
notes: >
  Same bare-"Mozilla/5.0" block observed as ntca-tiger-reserves.yaml
  (confirmed live 2026-08-27: HTTP 406 without the override) — required,
  not optional. Tables are identified by header-row signature (first
  row's first cell text == "Sl No"), not by CSS class (no distinguishing
  class exists on these tables in the live DOM) or table position/index,
  so the parser survives new tables being added/removed on the page.
  Date format is inconsistent across rows (seen: "06.01.2024",
  "09-12-2021", "05-11-23") — the parser tries several formats rather
  than assuming one.

requests:
  - name: ntca-tiger-mortality-index
    url: https://ntca.gov.in/tiger-mortality/
    method: GET
