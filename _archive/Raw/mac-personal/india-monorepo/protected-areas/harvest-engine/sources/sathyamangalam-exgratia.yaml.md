---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/sources/sathyamangalam-exgratia.yaml
---

name: sathyamangalam-exgratia
description: >
  Sathyamangalam Tiger Reserve's own official site (Tamil Nadu Forest
  Department) — a live "Ex-Gratia" page reporting yearly human-wildlife
  conflict compensation data: human death cases, injury cases, crop
  damage, livestock loss, and property damage, each with case counts and
  settled compensation amounts (Rs. lakhs), plus total beneficiaries, one
  row per year (verified live 2026-08-27: 2022-2023 through 2025-2026,
  4 years).

  This is the ONLY structured, per-reserve, live human-wildlife-conflict
  source found for this project's 3 reference reserves as of 2026-08-27 —
  see scripts/normalize_threats.py and the top-level README for what else
  was checked and ruled out (NTCA's human-tiger-interactions page is
  policy text only; Karnataka's Bandipur has no equivalent public site —
  bandipurtigerreserve.com is an unregistered/parked domain; Mudumalai's
  own site, mudumalaitigerreserve.com, is a client-side-rendered Vue SPA
  whose real content isn't visible to a plain HTTP fetch, unlike
  Sathyamangalam's server-rendered page).

  Data is NOT a JSON API response — it's a JavaScript object literal
  (`const data = {...}`) embedded in a <script> block that drives an
  on-page Chart.js dashboard. The raw HTML is stored as-is (raw-first
  rule); scripts/normalize_threats.py extracts the known array fields by
  name via regex (years/humanDeathCases/humanDeathAmount/injuryCases/
  injuryAmount/cropDamageCases/cropDamageAmount/liveStockCases/
  liveStockAmount/propertyCases/propertyAmount/settledAmount/
  totalBeneficiaries) — not a general JS-literal parser or eval(), since
  this exact known structure is all that's needed and eval-ing arbitrary
  fetched content would be unsafe.
kind: html
spider: seed_list
parser: sathyamangalam_exgratia
base_url: https://sathytiger.tn.gov.in
license: "Government of Tamil Nadu / Tamil Nadu Forest Department content — no blanket reuse license published on the page; treat as All Rights Reserved pending explicit confirmation, same posture as other .gov.in/.tn.gov.in sources here."
terms_url: https://sathytiger.tn.gov.in
rate_limit_ms: 2000
auth: none
robots_txt_verified: "2026-08-27: verified live at sathytiger.tn.gov.in/robots.txt was not separately re-checked at time of writing — verify before the first scheduled (non-manual) run and update this field; the /exg page itself was fetched successfully with no server-side block observed (plain curl, no special user-agent needed, unlike the ntca.gov.in sources)."
notes: >
  Single-reserve source, deliberately not generalized to a
  {reserve_slug}-shaped URL template — Sathyamangalam is (as of this
  verification) the only Tamil Nadu tiger reserve site confirmed to expose
  this data via plain server-rendered HTML; Mudumalai's site (a Vue SPA)
  would need a different fetch approach (headless browser rendering) that
  this project does not currently have. Extending this to other reserves
  is a real gap, documented in the top-level README, not silently
  implied to be comprehensive by this file's existence.

requests:
  - name: sathyamangalam-exgratia-page
    url: https://sathytiger.tn.gov.in/exg
    method: GET
