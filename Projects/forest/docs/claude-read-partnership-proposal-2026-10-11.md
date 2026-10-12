---
tags: project-doc
project: forest
updated: 2026-10-11
---
# For future agent
Partnership proposal + talking points prepped for Vishnu's meeting with the READ (readsouthindia.in) owner, to pitch a Keystone-Foundation-style joint org for Sathyamangalam.

# Proposal: A Joint Conservation Org for Sathyamangalam

**Prepared by Vishnu (araCreate Group) for discussion with READ**

## 1. The Opportunity — Sathyamangalam Tiger Reserve
- 1,408 km² where the Western Ghats and Eastern Ghats meet — a rare genetic corridor for tigers and elephants.
- Tiger recovery: 8–10 tigers (2010) → ~112 today.
- Elephants ~350–450 → 651; leopards ~20 → 111.
- 225 bird species, striped hyena, four-horned antelope, blackbuck, gaur.
- 184 inhabited villages, 9 tribal settlements inside the reserve, 136 villages in the zone of influence.
- Aligned with UN SDG 15 (Life on Land) and the Global Tiger Forum's TX2 doubling standard.
- 54 anti-poaching camps.

## 2. What We've Built — The Sathyamangalam Record (Atlas)
- Open, cited, source-verified data platform: 92 places, 2,664 species, 78,000+ occurrence records, 2,000+ documents.
- ~33% toward full target coverage, continuously growing.
- Sensitive species locations (tiger, elephant, leopard) automatically coordinate-coarsened before going public.
- Disputed official figures (tiger/elephant/leopard counts across years) shown transparently side by side, never silently resolved.
- Geo-mapping: 88 real places (villages, forest ranges, beats) plotted on a live interactive map — turns "651 elephants somewhere in 1,408 km²" into "elephants pass right next to this specific village."
  - Example: Thengumarahada village (Bhavanisagar forest range) sits directly in a known elephant corridor — visible as pins on the map, not just a number in a table.
- Gap: deep data/digital infrastructure, no ground presence, no community programs, no legal/institutional form yet.

## 3. The Model to Follow — Keystone Foundation
- Nilgiris-based org, mission: "enhance quality of life and the environment."
- Integrated programs under one roof: biodiversity conservation, community wellbeing, climate, water/sanitation, community archives.
- Proves conservation + grassroots community work can live in one org, strengthening both.

## 4. What READ Already Brings
- 25 years of ground work in Tamil Nadu, already active inside Sathyamangalam Tiger Reserve: tribal education, healthcare, livelihoods, government welfare access.
- Deep community trust and local legitimacy.
- No data/tech layer today.

## 5. The Fit
| | READ | Us (Atlas) |
|---|---|---|
| Ground trust & legitimacy | yes | — |
| Community programs | yes | — |
| Data / digital infrastructure | — | yes |
| Conservation data & documentation | — | yes |

## 6. Use & Motive (in case asked)
**Use:**
1. Human-wildlife conflict planning — mapped occurrence data shows where conflict risk concentrates near specific villages.
2. Funding & grant evidence — sourced tiger-recovery numbers etc. are fundable proof of impact.
3. Research partnerships — existing contact with ATREE (Western Ghats/CEPF funding) and WII (Wildlife Institute of India) gives institutional credibility.
4. Land & rights documentation — documented village/settlement presence supports tribal land/customary rights claims.
5. Cultural preservation — archive for oral history, local place names, community memory.
6. Government accountability — transparent disputed figures as leverage with TN Forest Dept / TX2 funding talks.
7. Education / eco-tourism base — visitor info, school programs, interpretation content.

**Motive:**
- Independent — not run by or answerable to the Forest Department or any government body, so it can tell the full truth.
- Not a product for sale — contributed to a shared mission, not monetized.
- Personal: care about this specific place, built the hard infrastructure alone; the ask is for an institutional home that can act on the ground with it.

## 7. The Ask
- Explore forming a joint entity (trust/society) for Sathyamangalam, modeled on Keystone's integrated structure.
- READ brings legal standing, ground operations, community relationships.
- We bring the Atlas as a standing program arm.
- First step: a pilot or MOU, not a full merger — e.g. a shared community-documented history/archive project, or a joint data-and-outreach pilot in one village cluster.

## 8. What to Show in the Room
- Live demo of the Atlas (local copy ready, full data, geo-mapping, no "under construction" pages).
- This doc as the leave-behind outline.

---

# Session notes (2026-10-11)
- Cloned `sathyamangalam-atlas` repo fresh on a new (office) Mac via `gh` (already authenticated).
- Confirmed live `dev.sathyamangalam.online` shows "under construction" stub pages (main branch) — the full redesigned pages live on git branch `dev` (not the same thing as the `dev.` subdomain).
- Checked out `origin/dev` into worktree `~/code/sathyamangalam-demo`, ran locally with `scripts/devserve.py` — full site loads correctly (land, species, places all real data).
- Ran `scripts/sync_d1_to_atlas.py` (dry-run then live) against production harvest-engine D1 — pulled 535 new rows (502 documents, 33 sources), refreshed ~8,400 fields. Old atlas.db backed up to `data/atlas_pre_sync_backup.db` in the worktree.
- Committed the sync locally to a new `dev` branch in the worktree (commit `139a281`) — **not pushed**. Per this repo's own README rule ("office Mac: pull/clone only, never push from office infra"), Vishnu will bring this commit to his personal Mac to push.
- Local dev server stopped at end of session.
- Added Keystone Foundation's knowledge base/archive link: https://keystone-archives.org/archive/s/keystone-foundation-archives/page/welcome (saved to `Companies/Keystone Foundation.md`).
