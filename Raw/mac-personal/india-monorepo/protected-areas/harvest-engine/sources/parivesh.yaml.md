---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/parivesh.yaml
---

name: parivesh
description: >
  data.gov.in dataset "State/UT-wise Details of Wildlife Sanctuaries and
  National Parks in the Country" (sourced from a MoEFCC/National Wildlife
  Database Centre reply to an Unstarred Question, 10 August 2023) — used
  to cross-check/backfill state-wise counts against the seed list built
  from ntca-tiger-reserves.yaml + wii-gazette-notifications.yaml.

  Renamed from an earlier "parivesh" placeholder: PARIVESH (parivesh.nic.in)
  itself is a project-clearance workflow tool, not a protected-area
  registry, and does not publish a bulk open-data export of protected
  areas — verified 2026-08-27, no such export found on the PARIVESH portal
  itself. This dataset on data.gov.in is the closest actually-existing
  open-data source in that spirit; kept the filename `parivesh.yaml` only
  because the build spec named the PARIVESH source — see notes below for
  the correction.
kind: api
spider: api_source
base_url: https://api.data.gov.in
license: "Government Open Data License – India (GODL)"
terms_url: https://data.gov.in/government-open-data-license-india
rate_limit_ms: 1000
auth: (secret removed)
url_params_from_env:
  api_key: (secret removed)
robots_txt_verified: "n/a — REST API endpoint, not a scraped page"
notes: >
  Uses DATA_GOV_IN_API_KEY (registered, present in
  ~/core-infra/harvest-secrets.env). Passed as the api-key query param; the
  {api_key} placeholder is filled from env, never hardcoded. The core API
  redacts it from stored provenance URLs (D-17).

  RESOURCE ID CORRECTED 2026-09-07 (D-39). The id this descriptor queried,
  ee69ffd5-5bab-4b16-8f55-30ec0b3e4918, does not exist: with a valid key it
  returns {"message":"Meta not found","status":"error","total":0,
  "records":[]}. Neither does the "older, staler" alternative recorded below,
  d8c07c72-ca17-448c-a4ab-8cbeadd5c677, nor the catalog endpoint for
  139f7251-... ("Invalid Catalog id").

  The live resource id IS 139f7251-81dc-4132-bc82-bdda0915bbbf — the value
  this file previously recorded as the CATALOG uuid. The two were transposed.
  Verified live 2026-09-07: HTTP 200, status "ok", 37 rows, title
  "State/UT-wise Details of Wildlife Sanctuaries and National Parks in the
  Country (in Reply to Unstarred Question on 10 August, 2023)" — the exact
  dataset this descriptor's description names.

  WHY THE OLD VERIFICATION WAS WRONG: this file recorded the resource as
  "verified live" because querying it without a key returned 403 "Key not
  authorised" rather than 404. data.gov.in authenticates BEFORE it looks up
  the resource, so a 403 is returned for any resource id, real or invented,
  and proves only that the key was missing. A resource id can only be
  verified with a working key.

  COLUMN NAMES ARE NOW VERIFIED (they were explicitly unverified before):
  sl__no_, state__ut, number_of_national_parks, number_of_wildlife_sanctuaries.

  TWO THINGS THE NORMALIZER MUST HANDLE:
  1. The last row is a TOTAL row (sl__no_ = "Total", state__ut = "Total",
     106 national parks / 570 wildlife sanctuaries). It is not a state and
     must be skipped, not ingested as one.
  2. Counts arrive inconsistently typed — some as JSON numbers, some as
     strings ("0"). Coerce; do not assume int.
  Internal consistency checked: the 36 real rows sum to exactly the totals
  the Total row states, and those totals agree with a second live dataset
  (260619cf-01dd-43e3-9ba9-25d29221c0d1, year-wise PA counts, which gives
  106/570 for 2022).

  ORG IS "Rajya Sabha", NOT MoEFCC. Parliamentary-answer datasets are filed
  on data.gov.in under the answering House, not the ministry that supplied
  the figures. An org-scoped search for MoEFCC does not find this dataset —
  worth knowing before hunting for any other Unstarred-Question source.

  HOW TO SEARCH data.gov.in FOR A REPLACEMENT, since this cost real effort:
  /lists ignores q= entirely (the total comes back unchanged at 287,810), but
  filters[field] maps onto an Elasticsearch TERM query and `title` is an
  analysed field — so filters[title]=<single lowercase token> is a working
  full-text title search. `filters[title]=sanctuaries` returns 11 rows and
  finds this dataset. That beats paging the whole 181k-resource active
  catalogue, which is ~725 MB of responses and dies on IncompleteRead.

  A RICHER SOURCE EXISTS, not used here: resource
  6420c35b-9ccf-4796-8f7c-e8d345551f07, "State/UT-wise List of Wildlife
  Sanctuaries and Area As on December, 2019", carries 553 rows of INDIVIDUALLY
  NAMED protected areas with per-area figures, rather than per-state counts.
  That is per-reserve data this engine could cross-match by name, not just an
  aggregate cross-check. It is older (Dec 2019) and its own source dataset,
  so it belongs in its own descriptor rather than swapped in here.

  PARIVESH (parivesh.nic.in) itself remains, as recorded before, a
  project-clearance workflow tool with no bulk protected-area export — this
  data.gov.in dataset is the nearest actually-existing open-data source in
  that spirit, and the filename is kept only for continuity.

requests:
  - name: parivesh-adjacent-wls-np-statewise
    url: "https://api.data.gov.in/resource/139f7251-81dc-4132-bc82-bdda0915bbbf?api-key=(secret removed)
    method: GET
