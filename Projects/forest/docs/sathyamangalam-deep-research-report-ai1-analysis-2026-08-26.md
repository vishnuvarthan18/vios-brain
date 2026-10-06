# Deep analysis — AI #1's full deep-research report (Tasks 1–4)

**Source report:** `Sathyamangalam_Atlas_Research_Report.md`, compiled 25–26 Aug 2026 (the "first AI"). Full report — Task 1 (worldwide inventory, 250+ projects), Task 2 (how the category works), Task 3 (how these projects win/earn), Task 4 (disputed-facts precedents, Darwin Core, timelines, validation/contradiction).

**Status:** This is AI #1's report only, analysed standalone. A second AI's independent research on the same brief has been separately initiated and has not yet returned. Do not merge or reconcile against the bug list until both are in hand — this doc is the first half of a planned comparison.

---

## Overall verdict

Trustworthy enough to act on. It self-undermines in the right places — Task 2 opens by admitting only ~24 of 250+ projects disclose team size or budget, flags its own survivorship bias, and labels every inferred number as inferred. It validates its central thesis against Sathyamangalam's own contradictory published numbers rather than staying theoretical (§4.4.1). That combination is a good trust signal.

## The four findings that actually change the plan

1. **The disputed-figures feature is diagnosed, not speculative.** §4.4.1 pulls Sathyamangalam's own public numbers: at least 4 conflicting area figures, and 12 different tiger-population figures across 5 methods for the same reserve — including a 3.8× swing in 2010 (12 vs 46) and a 2024–25 official census (112) that is *lower* than a 2022 official verbal estimate (~120). This is now the strongest pitch line: not proposing a feature, but documenting an existing defect using the reserve's own data as Exhibit A.

2. **Money doesn't predict survival; institutional ownership does.** INBio (Costa Rica, 3.7M specimens, international funding) is dead at both its original and successor URL. `vncreatures.net` (one person, a Gmail address, zero funding) added entries three weeks before this report was compiled. Grant-funded academic builds die on schedule at year 4–6; solo/small-team builds cluster at 15–24 years (3–4× longer). The project's current shape — solo aspiring to small-lab/community-governed ("E aspiring to D/F" in the report's archetype table) — is one of the two most durable positions in the whole corpus.

3. **A concrete, implementable disputed-figures schema exists (§4.1.6).** Six primitives, synthesized from Wikidata's claim/qualifier/rank model, OWID's side-by-side comparison convention, and legal citators (KeyCite/Shepard's):
   - the assertion is the record, not the fact (never a single `area_sq_km` column)
   - rank, don't delete (preferred / normal / deprecated-but-retained)
   - qualify with method (mandatory `determinationMethod`)
   - carry explicit certainty (`certain` / `less-certain` / `uncertain`, plus a genuinely-unknown value)
   - keep superseded versions live and addressable
   - version the whole resource so any figure is citable "as of vX"

   This is close to a final data model, not just inspiration.

4. **Darwin Core is solved and cheap for one domain; the real standards gap is the other four.** Four required fields for occurrence data; the GBIF IPT auto-generates the archive. DwC has zero coverage for gazetteer, legal instruments, historical documents, or literature. Realistic standards per domain: Linked Places Format (gazetteer), no standard exists (legal — closest patterns are Flanders' Inventaris Onroerend Erfgoed and Terras Indígenas' seven legal phases), Dublin Core/Omeka (documents), plain bibliographic formats (literature). The Humboldt Extension (51 terms) is flagged as the one extension actually worth learning — it's the only standard that encodes "looked and didn't find" vs. "wasn't looking," which is what makes heterogeneous historical/reserve surveys comparable.

## Where to be skeptical or verify before acting

- **India's GBIF status is a real landmine (§4.2.5).** GBIF's own country page calls India an "Observer Country," not a Participant — which should block the free Hosted Portal route — yet India Biodiversity Portal has published to GBIF since 2019 under an endorsing node. The report correctly flags the contradiction rather than resolving it. Action: email `hostedportals@gbif.org` before any plan assumes a free GBIF-hosted front end.
- **Keystone Foundation Archives running on Omeka, and the Assam/Western Ghats Biodiversity Informatics Platform sub-portals being live**, are both explicitly unverified (403-blocked to automated fetch). Both sit on the Task 4 unknowns checklist (§4.4.4). Verify manually before treating Keystone as a confirmed technical precedent or plausible partner.
- **Commercial double-edge, stated but unresolved (§3.4.2 / §4.4.3.8):** the same disputed-figures feature that builds credibility also helps a clearance consultant find contestable numbers to exploit. The report names this and moves on — needs an explicit project position, not just awareness.
- **Tribal-knowledge layer is a live legal matter, not a data problem.** 9 settlements inside the reserve (7 core, 2 buffer — Soliga and Oorali/Irula communities); Tamil Nadu has resolved only ~25% of FRA claims (8,594 of 34,837) vs. Kerala's ~60%. Every precedent found (Sarawak Biodiversity Centre, Inuit Heritage Trust, TKDL) says "document but do not publish." This needs an explicit governance decision early, not a technical solution.

## Sequencing the report commits to (§4.3.3)

1. Scope statement + gap audit — 6–8 weeks, publishable immediately, no build required.
2. Gazetteer skeleton in Linked Places Format — 4–8 months — sourced from Erode/Coimbatore/Salem district gazetteers, Survey of India sheets, revenue records, Imperial Gazetteer via DSAL. Highest-value first deliverable because no structured version of these gazetteers exists anywhere.
3. First citable artefact — months 6–12 — short data paper + DOI deposit (Zenodo/Dryad).
4. Legal-instrument layer — year 1–2 — PARIVESH 2.0 bulk CSV, sanctuary/tiger-reserve GOs, ESZ notifications, FRA claim statistics, modelled as entities distinct from the places they cover.
5. Disputed-figures layer — year 1–2, in parallel — implement the six primitives above over the area/tiger-count figures already gathered in §4.4.1.
6. Historical documents — year 2–4+ — ZSI Conservation Area Series, colonial forest working plans, DSAL page images, with certainty ratings per interpreted feature.
7. Occurrence — year 2–5, mostly by aggregation — eBird/iNaturalist/GBIF aggregation, not primary collection. Export via DwC/IPT.

Realistic timeline overall: 10–16 years for a full multi-domain atlas, no counterexample in the 250+-project corpus. A publishable artefact is available in 6–8 weeks; a DOI'd gazetteer dataset within 12 months.

## What this analysis deliberately does not do yet

- Does not reconcile against `reviewer-findings-and-bug-list-2026-08-25.md`.
- Does not cross-check against the second AI's independent research (not yet returned).
- Does not re-verify any of the report's own flagged unknowns (§4.4.4 checklist: GBIF hosted-portal eligibility, Assam/Western Ghats portal liveness, Keystone's platform, GeoNature 2026 deployment cost, Nunaliit maintenance status, FES's orphaned bibliography, WII's "National Wildlife Database," TIGERNET record counts/cadence, existence of an Indian EIA desk-study market, FLAME University Gazetteer Project contact).

**Next step (on hold):** once the second AI's report is in hand, run a joint deep analysis — agreement, contradiction, and unique findings each report has that the other missed — before folding anything into the bug list.
