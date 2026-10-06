---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/wii-gazette-notifications.yaml
---

name: wii-gazette-notifications
description: >
  WII's legacy (v1.wii.gov.in) Protected Area Gazette Notification
  Database — one page per state/UT, each a table of [S.No., Name of
  Protected Area, Gazette Notification PDF]. Replaces the earlier
  wii-protected-areas.yaml / moefcc-notifications.yaml placeholders, whose
  URLs (wii.gov.in/protected_area_network, wii.gov.in/protected-area-network,
  moef.gov.in/notifications) were verified 2026-08-27 to be dead or
  content-free department pages, not data sources.

  This is the primary source for National Parks, Wildlife Sanctuaries,
  Conservation Reserves, and Community Reserves (type parsed from a suffix
  in the name, e.g. "WLS"/"NP"/"CR" — normalize step, not this fetch).
  Tiger Reserves are better covered by ntca-tiger-reserves.yaml (richer
  fields: area, notification years) — cross-reference rather than
  duplicate when a name appears in both.

  IMPORTANT CAVEAT: wiienvis.nic.in — a domain that appears in search
  results and is even linked from WII's own current site — is confirmed
  DEAD (NXDOMAIN from multiple resolvers, 2026-08-27). Do not use it. This
  source deliberately uses v1.wii.gov.in instead, which is a still-live
  legacy mirror confirmed to serve real tabular data.
kind: html
spider: seed_list
parser: wii_gazette_state_table
base_url: https://v1.wii.gov.in
license: "Government of India / WII content — no blanket reuse license published; treat as All Rights Reserved pending explicit confirmation."
terms_url: https://v1.wii.gov.in
rate_limit_ms: 2000
auth: none
robots_txt_verified: "2026-08-27: not yet independently checked at v1.wii.gov.in/robots.txt — verify before the first scheduled (non-manual) run and update this field with the result. Note this is a legacy/archival subdomain, not the current wii.gov.in — check both if in doubt."
notes: >
  State-slug values below were extracted directly from the live
  <select id="select1" name="D2"> dropdown HTML at
  https://v1.wii.gov.in/protectedareagazettenotificationdatabase
  (verified 2026-08-27), not guessed — two are irregular and easy to get
  wrong if guessed: Karnataka's slug is "karnatka" (no
  "protectedareagazette_" prefix, and a misspelling of Karnataka, present
  in the live site's own markup) and Kerala's is "kerela" (same, and also
  a misspelling). All 35 URLs below were individually verified to return
  HTTP 200. Madhya Pradesh's page has historically consolidated into a
  single combined PDF for the whole state rather than per-PA rows —
  expect a single-row result there, not a parse failure. The gazette PDF
  link's path prefix is inconsistent across states ("protected_download/
  <state>/<slug>" vs "/images/<state>/<slug>.pdf" have both been observed)
  — the parser resolves via response.urljoin rather than assuming one
  pattern, so this doesn't need per-state config.

  Verified source data quality issue: Karnataka's page (slug "karnatka",
  confirmed 2026-08-27, 33 rows) has at least one row where the "Name of
  Protected Area" cell literally contains a filename
  ("AghansnhiniLTMCR.pdf") instead of a place name — a source-side HTML
  error, not a parser bug. Do not silently correct/drop rows like this in
  the parser; surface them as-is and let the normalize step (or a human)
  decide, since guessing the intended name would be fabricating data not
  in the source.

requests:
  - name: andhra-pradesh
    url: https://v1.wii.gov.in/protectedareagazette_andhrapradesh
    method: GET
    state_label: "Andhra Pradesh"
  - name: arunachal-pradesh
    url: https://v1.wii.gov.in/protectedareagazette_arunachalpradesh
    method: GET
    state_label: "Arunachal Pradesh"
  - name: assam
    url: https://v1.wii.gov.in/protectedareagazette_assam
    method: GET
    state_label: "Assam"
  - name: bihar
    url: https://v1.wii.gov.in/protectedareagazette_bihar
    method: GET
    state_label: "Bihar"
  - name: chhattisgarh
    url: https://v1.wii.gov.in/protectedareagazette_chhattisgarh
    method: GET
    state_label: "Chhattisgarh"
  - name: goa
    url: https://v1.wii.gov.in/protectedareagazette_goa
    method: GET
    state_label: "Goa"
  - name: gujarat
    url: https://v1.wii.gov.in/protectedareagazette_gujarat
    method: GET
    state_label: "Gujarat"
  - name: haryana
    url: https://v1.wii.gov.in/protectedareagazette_haryana
    method: GET
    state_label: "Haryana"
  - name: himachal-pradesh
    url: https://v1.wii.gov.in/protectedareagazette_himachalpradesh
    method: GET
    state_label: "Himachal Pradesh"
  - name: jammu-and-kashmir
    url: https://v1.wii.gov.in/protectedareagazette_jammu_kashmir
    method: GET
    state_label: "Jammu and Kashmir"
  - name: jharkhand
    url: https://v1.wii.gov.in/protectedareagazette_jharkhand
    method: GET
    state_label: "Jharkhand"
  - name: karnataka
    url: https://v1.wii.gov.in/karnatka
    method: GET
    state_label: "Karnataka"
  - name: kerala
    url: https://v1.wii.gov.in/kerela
    method: GET
    state_label: "Kerala"
  - name: madhya-pradesh
    url: https://v1.wii.gov.in/protectedareagazette_madhyapradesh
    method: GET
    state_label: "Madhya Pradesh"
  - name: maharashtra
    url: https://v1.wii.gov.in/protectedareagazette_maharashtra
    method: GET
    state_label: "Maharashtra"
  - name: manipur
    url: https://v1.wii.gov.in/protectedareagazette_manipur
    method: GET
    state_label: "Manipur"
  - name: meghalaya
    url: https://v1.wii.gov.in/protectedareagazette_meghalaya
    method: GET
    state_label: "Meghalaya"
  - name: mizoram
    url: https://v1.wii.gov.in/protectedareagazette_mizoram
    method: GET
    state_label: "Mizoram"
  - name: nagaland
    url: https://v1.wii.gov.in/protectedareagazette_nagaland
    method: GET
    state_label: "Nagaland"
  - name: odisha
    url: https://v1.wii.gov.in/protectedareagazette_orissa
    method: GET
    state_label: "Odisha"
  - name: punjab
    url: https://v1.wii.gov.in/protectedareagazette_punjab
    method: GET
    state_label: "Punjab"
  - name: rajasthan
    url: https://v1.wii.gov.in/protectedareagazette_rajasthan
    method: GET
    state_label: "Rajasthan"
  - name: sikkim
    url: https://v1.wii.gov.in/protectedareagazette_sikkim
    method: GET
    state_label: "Sikkim"
  - name: tamil-nadu
    url: https://v1.wii.gov.in/protectedareagazette_tamil_nadu
    method: GET
    state_label: "Tamil Nadu"
  - name: tripura
    url: https://v1.wii.gov.in/protectedareagazette_tripura
    method: GET
    state_label: "Tripura"
  - name: uttar-pradesh
    url: https://v1.wii.gov.in/protectedareagazette_uttar_pradesh
    method: GET
    state_label: "Uttar Pradesh"
  - name: uttarakhand
    url: https://v1.wii.gov.in/protectedareagazette_uttarakhand
    method: GET
    state_label: "Uttarakhand"
  - name: west-bengal
    url: https://v1.wii.gov.in/protectedareagazette_westbengal
    method: GET
    state_label: "West Bengal"
  - name: andaman-and-nicobar-islands
    url: https://v1.wii.gov.in/protectedareagazette_andaman_nicobar_islands
    method: GET
    state_label: "Andaman and Nicobar Islands"
  - name: chandigarh
    url: https://v1.wii.gov.in/protectedareagazette_chaandigarh
    method: GET
    state_label: "Chandigarh"
  - name: dadra-and-nagar-haveli
    url: https://v1.wii.gov.in/protectedareagazette_dadraandnagar_haveli
    method: GET
    state_label: "Dadra and Nagar Haveli"
  - name: daman-and-diu
    url: https://v1.wii.gov.in/protectedareagazette_daman_diu
    method: GET
    state_label: "Daman and Diu"
  - name: lakshadweep
    url: https://v1.wii.gov.in/protectedareagazette_lakshadweep
    method: GET
    state_label: "Lakshadweep"
  - name: delhi
    url: https://v1.wii.gov.in/protectedareagazette_delhi
    method: GET
    state_label: "Delhi"
  - name: puducherry
    url: https://v1.wii.gov.in/protectedareagazette_puducherry
    method: GET
    state_label: "Puducherry"
