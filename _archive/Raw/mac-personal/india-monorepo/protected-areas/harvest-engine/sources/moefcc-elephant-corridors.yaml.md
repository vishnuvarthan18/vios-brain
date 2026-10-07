---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/moefcc-elephant-corridors.yaml
---

name: moefcc-elephant-corridors
description: >
  MoEFCC / Project Elephant's "Elephant Corridors of India 2023" report —
  a real, text-based (not scanned) 183-page PDF with a consistent
  per-corridor structured block (Connectivity, State, Indicative
  length/width, Geo coordinates, Forest ranges, Ecological importance,
  Elephant movement status, etc.) for every named corridor in the country.
  Verified live 2026-08-27: HTTP 200, 9MB, confirmed real text layer via
  pdfplumber (264KB extracted text across 183 pages — consistent with a
  genuine text PDF, not a scan).

  projectelephant.gov.in itself is dead (DNS failure, confirmed
  2026-08-27) — this PDF is hosted on moef.gov.in instead.

  IMPORTANT — what this source does and does NOT let us claim, per the
  spec's corridor_reserve join table: each corridor's "Connectivity" field
  is free-text prose naming forest ranges/reserve forests (e.g. "connects
  Kaniyanpura Reserve Forest with Moyar Reserve Forest of Bandipur Tiger
  Reserve"), not a clean two-tiger-reserve pair. scripts/normalize_corridors.py
  only creates a corridor_reserve row for a reserve EXPLICITLY named in
  that text (matched against reserve.name in D1) — it does not infer a
  second/other endpoint reserve from geographic proximity or landscape
  descriptions ("Mudumalai – Bandipur – Wayanad – Sathyamangalam complex"
  language appears for several corridors as ecological context, not a
  literal two-reserve connectivity claim, and is deliberately NOT used to
  create a corridor_reserve link). A corridor with only one matched
  reserve is stored with only that one corridor_reserve row — its second
  endpoint is a real gap for a human to confirm by reading the source PDF
  directly, not something this script guesses at.
kind: pdf
spider: seed_list
base_url: https://moef.gov.in
license: "Government of India / Ministry of Environment, Forest and Climate Change content — no blanket reuse license published; treat as All Rights Reserved pending explicit confirmation, same posture as other .gov.in sources here."
terms_url: https://moef.gov.in
rate_limit_ms: 2000
auth: none
robots_txt_verified: "2026-08-27: not yet independently checked at moef.gov.in/robots.txt — verify before the first scheduled (non-manual) run and update this field with the result."
notes: >
  Single fetch (not per-reserve) — the whole national PDF is fetched once
  and parsed for every corridor entry it contains, not filtered to a
  single reserve at fetch time. species_corridor rows: this report is
  elephant-specific by definition (a Project Elephant publication), so
  every corridor it produces is tagged to the "Elephant" species entry
  (Elephas maximus) in the species table if one exists there already
  (from the FLORA/FAUNA branch's GBIF/eBird/iNaturalist harvesting) —
  skipped (not fabricated) if no matching species row exists yet.

requests:
  - name: moefcc-elephant-corridors-2023-pdf
    url: https://moef.gov.in/uploads/2023/11/PE-Elephant-Corridor-of-India-2023.pdf
    method: GET
