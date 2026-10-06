---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/moefcc-elephant-reserves.yaml
---

name: moefcc-elephant-reserves
description: >
  MoEFCC / Wildlife Institute of India's "Elephant Reserves of India: An
  Atlas" (Version 2/2023) — a real, text-based (not scanned) 78-page PDF.
  Verified live 2026-09-07: HTTP 200, 4.9MB, confirmed real text layer via
  pypdf (not a scan, despite one tool's own wrong first guess to the
  contrary — checked directly rather than trusted).

  Elephant Reserves are a Project Elephant / MoEFCC designation, NOT an
  IUCN protected-area category, so neither WDPA nor OSM carries them. This
  Atlas is the only source found for a keyless, national, per-reserve
  listing.

  WHAT THIS SOURCE GIVES, AND THE ONE TRAP IN IT: Table 1 (page 8,
  "Elephant Reserves in the country") lists all 33 reserves with State,
  Area (km2) and P.A. in ER (km2, the protected-area overlap within the
  reserve — "-" where none is stated). Table 5 (page 10, "List of Elephant
  Reserve mapped using gazette notification") separately gives each
  reserve's Notified year. Table 5's OWN Area column has a verified swap
  bug for two Assam reserves: it reports Chirang-Ripu's area as 1420.0 and
  Sonitpur's as 2600.0 — exactly swapped relative to Table 1 (2600.0 /
  1420.0), which itself is cross-confirmed by an independent source (a PIB
  Rajya Sabha press release, PRID=1986211, giving the same figures as Table
  1). scripts/normalize_elephant_reserves.py therefore reads area and
  P.A.-overlap ONLY from Table 1, and Notified year ONLY from Table 5,
  matched by a normalised reserve name (case/hyphen/space-insensitive,
  since the two tables spell a few names slightly differently, e.g.
  "Sarguja-Jashpur" vs "Sarguja Jashpur") — never Table 5's area column.
kind: pdf
spider: seed_list
base_url: https://moef.gov.in
license: "Government of India / Ministry of Environment, Forest and Climate Change (Project Elephant Division, Wildlife Institute of India) content — no blanket reuse license published; treated as All Rights Reserved pending explicit confirmation, same posture as moefcc-elephant-corridors and the other .gov.in sources here."
terms_url: https://moef.gov.in
rate_limit_ms: 2000
auth: none
robots_txt_verified: "2026-09-07: not yet independently checked at moef.gov.in/robots.txt — same open item this platform already carries for moefcc-elephant-corridors; verify before the first scheduled (non-manual) run."
notes: >
  Single fetch (not per-reserve) — the whole national PDF is fetched once
  and parsed for every reserve entry it contains. A dated one-off Atlas
  edition (Version 2/2023), not a feed; the annual schedule_tier exists
  only to notice a successor edition, not because this content changes
  more often than that.

requests:
  - name: moefcc-elephant-reserves-atlas-v2-2023-pdf
    url: https://moef.gov.in/uploads/2023/11/PE-Elephant-Reserve-of-India-an-atlas.pdf
    method: GET
