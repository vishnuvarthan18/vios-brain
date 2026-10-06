---
tags: chat
date: 2026-08-27
source: Claude personal account
uuid: fac314c8-3b46-4fc7-92be-a067d6bf215d
---
# Flora and fauna normalization pipeline with species deduplication

## Summary
**Conversation Overview**

This conversation is part of an ongoing software build project centered on a wildlife reserve data harvesting engine called `harvest-engine`, targeting Cloudflare D1 (SQLite) and R2 storage. The person completed and verified Branch 1 (flora/fauna normalization) and immediately moved to begin Branch 2 (threats data). The branch 1 work included two new files — `0003_reserve_bbox_and_normalize_tracking.sql` (a migration fixing a pre-existing gap where bbox columns were referenced but never created) and `normalize_flora_fauna.py` (a normalization script reading raw GBIF/eBird/iNaturalist JSON from R2 and writing typed rows to D1) — plus workflow and README updates. Key verified behaviors included cross-run species deduplication, overlapping bbox multi-reserve linking, CC license filtering with content-hashed R2 storage, and fail-one-continue-all error handling. Testing used a local sqlite3/file stand-in against real GBIF and iNaturalist responses for Sathyamangalam, Bandipur, and Mudumalai tiger reserves, with a documented-shape eBird fixture due to no available API key.

For Branch 2, Claude identified that the intended threats scope (poaching, encroachment, human-wildlife conflict across all reserve types) lacked sufficient structured sources. The one confirmed usable source was NTCA's tiger mortality page (ntca.gov.in/tiger-mortality/), which provides real per-event structured data from 2012–2025 for tiger reserves only. Claude presented two options: narrow the branch to tiger-reserve poaching/seizure data from NTCA only, or skip threats and reorder to Branch 3 (hydrology) or Branch 4 (people/tribe/food) where sources may be more available. The conversation ended awaiting the person's decision.

A standing project constraint — explicit and repeatedly observed — is that no AI-based data collection or cleaning is used anywhere; all field mapping must be plain and inspectable. Real D1/R2 infrastructure is not yet provisioned; the workflow remains `workflow_dispatch`-only pending a real-infra dry run.

## Chat

**Vishnu:** Everything is exactly as expected — no stray scratch files leaked into the repo, and the changeset is scoped to what this branch of work required. This branch of work is complete and verified end-to-end. Summary of what was built and tested:
New files:

* `harvest-engine/db/migrations/0003_reserve_bbox_and_normalize_tracking.sql` — adds `reserve.bbox_min_lon/lat/max_lon/lat` (a pre-existing gap: `run_api_spider.py` and the GBIF/eBird/iNaturalist YAMLs already depended on these columns, but no migration had ever created them, so the fetch step itself couldn't run against real D1) and `source.normalized_at` for idempotent normalization tracking.
* `harvest-engine/scripts/normalize_flora_fauna.py` — reads raw GBIF/eBird/iNaturalist JSON from R2, writes typed `species`/`species_reserve`/`occurrence`/`media` rows to D1, with:
   * Species deduplicated by `scientific_name` across reserves and across separate runs (verified: tiger appeared in 2 GBIF fetches across 3 reserves → one species row, three `species_reserve` links).
   * Occurrences linked to every reserve whose bbox contains the coordinates (overlapping buffer zones correctly produce multiple links).
   * Media rows only for images with an unambiguous CC license (CC0/BY/BY-SA/BY-NC/BY-NC-SA — no ND, no missing/all-rights-reserved); images are downloaded and stored into R2 under a content-hashed key, never a bare external URL.
   * Fail-one-continue-all at the record, response, and source level.
   * Full provenance (`source_url`/`retrieved_at`/`license`/`confidence`) on every row.
   * Plain, inspectable field-mapping code — no AI-based collection or cleaning anywhere.

Modified: `.github/workflows/harvest.yml` (wires in the normalize step + migration apply), top-level `README.md`, `harvest-engine/sources/README.md` (status updates).
Verified end-to-end against a local D1/R2 stand-in (real sqlite3 + local file store mimicking the actual HTTP APIs), using real fetched GBIF and iNaturalist responses for 3 real tiger reserves (Sathyamangalam, Bandipur, Mudumalai — the latter two with a deliberately overlapping bbox), plus a documented-shape eBird fixture (no API key available in this environment). Confirmed idempotency, cross-run species dedup, correct multi-reserve linking, and correct license filtering.
Not done, by design: no real D1/R2 (not provisioned yet), and the workflow stays `workflow_dispatch`-only until a real-infra dry run happens.

**Claude:** Branch 1-flora/fauna normalization is done. Next up per your build order is 2-threats.

Before you move on — two things worth flagging now rather than after they compound:

- **Real D1/R2 dry run.** Everything above is verified against a sqlite3/local-file stand-in. That's good enough to trust the logic, but bbox math, R2 key collisions, and D1 write limits/timeouts behave differently for real. Worth doing one real dry run against actual Cloudflare resources before stacking three more branches on top of an unverified foundation.
- **eBird fixture.** You said "documented-shape" — meaning you wrote it from API docs, not a real response. If eBird's actual JSON differs in any field name or nesting, normalize_flora_fauna.py will silently fail-one-continue-all right past it and you won't notice until you query for bird occurrences and find none. Get an eBird API key and one real fetch before trusting that path.

Want to start threats now, or pause for the real-infra dry run first?

**Vishnu:** lets duild

**Claude:** Building 2-threats.

Quick scope confirmation before I write anything — the data hierarchy says THREATS covers poaching/encroachment/human-wildlife conflict, tagged with source_url/retrieved_at/license/confidence like everything else. But I don't have a confirmed source for this yet (unlike flora/fauna which had GBIF/eBird/iNaturalist already lined up).

Do you have a specific source for threats data, or do we need to find one first?

**Vishnu:** find one first

**Claude:** I'll look for real, structured sources for poaching/encroachment/human-wildlife conflict data tied to Indian reserves.

_[tool: web_search]_

None of these are usable. They're academic papers with sample-survey data (specific studies, not per-reserve structured datasets), a news-mining project, or PDF reports — nothing with a bulk/structured feed we could scrape into `threat` rows with real `source_url`/`confidence` per record.

Checking for official government sources next — MoEFCC, NCRB (crime), forest department poaching/seizure records.

_[tool: web_search]_

Found it. NTCA's tiger mortality page (ntca.gov.in/tiger-mortality/) is exactly the kind of source we used for the reserve table in branch 0 — real, structured, reserve-wise, government-published, 2012–2025 coverage, with cause classification (natural/poaching/unnatural/seizure). Let me check the actual page structure before designing the spider.

_[tool: web_fetch]_

Real, structured, government source confirmed — but this only covers tiger reserves, and only one threat type (mortality). Let me check what it doesn't cover before deciding scope.

**What NTCA gives us:**
- Real per-event rows: date, state, sex, age, location text, tiger reserve name, inside/outside, seizure flag
- 2012–2025, single HTML page, tables embedded per year
- Covers only **tiger reserves** (58 of your ~700+ total reserves) and only **tiger mortality/poaching**, not encroachment or general human-wildlife conflict (elephants, leopards, crop raids etc.)

**What's missing entirely** — no structured, bulk-scrapable source exists for:
- Encroachment (nothing found; likely buried in unstructured PDF annual reports per state)
- Human-wildlife conflict beyond tigers (the PNAS/academic sources are survey samples of ~11 reserves, not usable as ongoing structured data)
- Non-tiger reserves entirely (WLS, NP, biosphere reserves have no equivalent mortality tracker)

So threats-as-designed (poaching + encroachment + HWC, across all reserve types) can't be built from real sources right now. Only a **tiger-poaching-only, tiger-reserve-only** slice is buildable today.

Two honest paths:

- **Narrow the branch**: build `threat` rows only for tiger reserves, only cause=poaching/seizure, sourced from NTCA. Real data, real provenance, but covers ~8% of your reserve set and one threat type.
- **Skip threats, reorder**: move to 3-hydrology or 4-people/tribe/food where sources are more likely to exist, and leave threats as a stub until a source turns up (WCCB, state forest reports).

Given "AI use is paused, plain inspectable code only, no guessing" — building a schema for a mostly-empty table feels premature. Which do you want?
