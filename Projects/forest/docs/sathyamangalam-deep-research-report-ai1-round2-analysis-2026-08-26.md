# Deep analysis — AI #1's Round 2 follow-up (Angles A–H)

**Source:** `Sathyamangalam_Atlas_Research_Report_1.md` (the consolidated master doc — Task 1–4 plus Round 2 merged in) and its three component files (`Round2_AngleAB_Language_Literature.md`, `Round2_AngleCD_Code_Funders.md`, `Round2_AngleF_India.md`). **This is still AI #1** — a self-initiated second pass by the same research AI, re-searching in other languages, other surfaces (registries, code repos, funder databases), and India-specific verticals Round 1 never touched. **This is not yet the second, independently-initiated AI's report** — that comparison is still pending.

**Headline of Round 2 as a whole:** it found real new material and it self-corrected twice in public (a rare and good sign) — first overcorrecting on Costa Rica's CRBio, then correcting its own overcorrection three sections later. Read that exchange (§AB.2 then §CD.0) as a demonstration of how the report treats its own errors: visibly, not quietly.

---

## What actually changed vs. the Round 1 analysis already on file

### 1. Costa Rica's CRBio — corrected twice, now settled
Round 1 said CRBio's mitigation (moving specimens to mandated institutions after INBio wound down) "failed" because both the original and successor portal were dead. Round 2's Angle A+B initially struck that down too hard, claiming CRBio was "alive and active." Angle C+D caught its own error: the Living Atlases participants index lists CRBio as **Offline**; a successor (CONAGEBIO's BiodataCR, on a `.go.cr` government domain, running Atlas of Living Australia software) exists but has **blank counters and no news since October 2019**. Verdict: this is Round 1's "silent stall" failure mode with working infrastructure underneath — not a rehoming success, not a total loss either. Use this corrected version, not either of the two earlier claims.

### 2. Several of Round 1's "white spaces" were English-search artefacts, not real gaps
Six of six non-English queries found something Round 1 had missed or marked unverified: Kazakhstan (`biokadastr.kz`, a live government portal), Türkiye (Nuh'un Gemisi, government national database), Indonesia (SIBIS), a francophone West Africa print atlas, a Dominican Republic print atlas, and full Catalogue of Life China figures (162,717 total for 2025, +6,857 species over 2024 — a clean working example of "version the whole resource" from §4.1.6). **Lesson for how to read Round 1's Task 1 going forward:** "not found" in that document means "not found in English," not "does not exist." Myanmar, the Gulf states, Mongolia and most of the Caribbean are still untested in their own languages.

### 3. Registries beat search — a new standing discovery method
The Living Atlases participant list (39 entries, status-labelled: 22 live, 4 offline, 9 in discussion) alone surfaced 7 projects new to the whole corpus in one page-read, plus a real finding: **all four Caribbean Living Atlases instances (Caribbean, Barbados, Suriname, Trinidad & Tobago) are offline together**, sharing one contact (Dimitri Ouboter) and one host domain, with zero documentation ever captured and no explanation on record — the clearest case anywhere in either round of named-person dependency at regional scale. GBIF's own Hosted Portals list also resolved two of Round 1's dead ends (Argentina's SNDB, Colombia's SiB) via their hosted-portal mirrors instead.

### 4. GitHub/code search does not work for this sector — confirmed negative finding
Angle C searched GitHub topics expecting to find a hidden layer of small atlas tools (reasoning from `vncreatures`-style invisibility). It didn't find one. The sector's actual software (Symbiota, Living Atlases, GeoNature, Nunaliit, Biodiversity Informatics Platform) is discovered through institutional registries, not code-topic browsing. One useful exception: a one-person Python+Streamlit project auditing 26M GBIF records against IUCN criteria — a real scale marker for what aggregation-only work looks like for a solo builder.

### 5. Rufford's project database — a 662-entry India collaborator directory
Not just a funding-tier reference (already known from Round 1) but an actual browsable list of 662 Indian Rufford grantees, several already working in the Western Ghats. Each produces a final report — grey literature, unindexed elsewhere, exactly what the atlas's literature layer needs. Unverified: how many are Tamil Nadu-specific or documentation-type rather than field ecology.

### 6. BISS/TDWG conference proceedings — a new discovery surface, 1,580 abstracts deep
Round 1 cited this once. It's actually a structured, peer-reviewed directory of active projects and tools, 2017–2025, better than open web search for finding projects "of exactly the Sathyamangalam type." Paired finding: publishing a data paper buys **permanence and citability, not readership** — sampled citation counts across *Scientific Data*/*Biodiversity Data Journal* are low (most articles under 5 citations). Keep publishing one, but for the right reason.

### 7. India-specific source layers Round 1 never searched (Angle F) — the single most consequential addition
This is the strongest new material in Round 2, and it's now reflected in the master doc's "ten findings" (up from eight):

- **267,608 People's Biodiversity Registers exist nationwide under the Biological Diversity Act 2002; only 11,951 (4.5%) are verified; none are digitised.** Every panchayat bordering Sathyamangalam, including the 9 tribal settlements, is legally required to hold one. This is statutory, village-indexed, already-written, and covers exactly the atlas's subject matter (species + traditional knowledge + landscape). It also carries a documented consent problem — knowledge collected from communities in at least one documented case (Maduranthakam) with no benefit-sharing. **Action, not research: ask the Tamil Nadu Biodiversity Board for the PBR status of panchayats bordering STR.**
- **Three new live figure-disputes in Sathyamangalam's own landscape**, adding to the area/tiger-count contradictions already logged: elephant corridors (88 → 101 → 150, with a published academic critique specifically naming the Nilgiri Biosphere Reserve as under-validated), the Western Ghats Eco-Sensitive Area boundary (129,037 km² Gadgil vs. 59,940 km² Kasturirangan vs. 56,825 km² current draft — a 15-year unresolved dispute), and sacred groves (13,270 documented vs. 100,000–150,000 estimated, roughly a 10× gap, both figures from the same author). Plus a **fifth area figure for the reserve itself** (1,455 km², from a peer-reviewed faunal survey — the outlier is in the literature, not a random website).
- **The gazetteer layer already has a direct, contactable precedent**: the French Institute of Pondicherry's Historical Atlas of South India (inscriptions-derived historical GIS for Tamil Nadu and Kerala, pilot in Pudukkottai). Status likely dormant (last activity 2014) but the institution and its inscription corpus are real and reachable.
- **A named person connects both of the atlas's hardest layers**: Muthusankar Gowrappan is on the team of both the Historical Atlas of South India *and* the Western Ghats Portal (confirmed in the E+G+H angle, below). One institute, one person, both hard problems, already attempted once for this exact region.
- **Tamil-language document infrastructure exists and works**: Noolaham Foundation (Sri Lankan Tamil digital library, volunteer-run, MediaWiki + Islandora stack, chapters in Canada/UK/Norway, zero state funding) is a proven technology answer for the historical-document layer.
- **No sacred-grove or tank/irrigation-heritage inventory exists for Tamil Nadu at all** — two more standalone, bounded, publishable datasets in the same shape as the gazetteer opportunity.
- **LGD (Local Government Directory) codes, not village boundary shapefiles, are the correct gazetteer spine** — Tamil Nadu village boundaries are absent from the open DataMeet dataset, so LGD codes are the join key to use instead.

### 8. The Western Ghats Portal question — now fully resolved (Angle H)
Round 1 flagged this as a priority unknown (HTTP 403, activity unverified). It's now answered directly from IFP's own project page: the Western Ghats Portal was absorbed into the India Biodiversity Portal, holds roughly **10,000 observations for a 129,037 km² landscape across six states** — thin, confirming Round 1's suspected negative finding that the Western Ghats is not already adequately documented. Same team as the Historical Atlas of South India (see above). **Recommended action: contact IFP before any build decision.**

### 9. A purpose-built answer to the tribal-knowledge access question (Angle E)
**Mukurtu CMS** — open-source, built since 2007 specifically for this problem (not adapted from a general system like Round 1's four precedents were). Communities hold "cultural protocols," not the atlas; protocol stewards (not site admins) control membership; one item can carry multiple protocols with different visibility to different communities simultaneously; Traditional Knowledge Labels let a community assert terms over material it doesn't legally own. This directly answers the design question Round 1 raised but didn't solve for the PBR/FRA-claim data specifically: a PBR-derived record can exist in the system, be indexed and counted, and stay invisible to the public (or to a clearance consultant) until the originating community decides otherwise.

### 10. Link rot — now quantified, not inferred
Round 1's lifespan numbers (15–24 years for solo survivors, 4–6 years for grant-funded deaths) were built from its own corpus alone. Independent measurement literature confirms the same range from outside: ~14-year half-life for scholarly links, 9.3-year median webpage lifespan, only 62% of cited pages ever archived, 49% of links in US Supreme Court opinions now dead. Practical consequence for the atlas's legal layer specifically: **store the government notification document itself (page image + transcription), never just a link to it** — on this evidence a link-based legal-instrument register will be roughly half broken within a decade.

---

## Net effect on the plan

Nothing in Round 2 overturns Round 1's core recommendations (sequencing, build-on-a-stack, disputed-figures schema). It does three things:

1. **Adds two more demonstrable figure-disputes** (elephant corridors, Western Ghats ESA, sacred groves) on top of the area/tiger-count pair already logged — five live disputes in the reserve's own landscape now, enough to build and demonstrate the feature without new fieldwork.
2. **Adds one concrete, high-value, non-technical action**: contact the French Institute of Pondicherry (Muthusankar Gowrappan) before any build decision — the two hardest layers have already been attempted once, for this region, by a reachable team.
3. **Adds one governance-technology answer that was missing**: Mukurtu CMS for the tribal-knowledge layer, which is a better fit than any of Round 1's four access-tier precedents and is free, funded, and maintained.

The PBR finding (267,608 registers, 4.5% verified, none digitised) is the single most consequential new fact — it's now the #8 finding in the master doc's top-ten list and deserves to be treated as an immediate records request, not a research task.

---

## Still pending

This document analyses AI #1's Round 1 + Round 2 output only. The second, independently-initiated AI's report has not yet been supplied. Once it arrives, run the planned joint comparison (agreement / contradiction / unique findings each side missed) before touching `reviewer-findings-and-bug-list-2026-08-25.md`.
