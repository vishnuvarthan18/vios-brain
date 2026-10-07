# Sathyamangalam Conservation Atlas — Research Report

**Subject:** A landscape-scale conservation "atlas" for Sathyamangalam Tiger Reserve / Wildlife Sanctuary, Tamil Nadu, India — combining species occurrence data, a place-name gazetteer, legal and government instruments, historical document mining, and literature on one site, with a feature that surfaces disputed and conflicting figures side by side rather than silently selecting one.

**Prepared for:** Vishnuvarthan Venkatapathy
**Compiled:** 25–26 August 2026
**Contents:** Three rounds. **Round 1** is four sequential tasks — Task 1 is the evidence base; Tasks 2, 3 and 4 are built on it. **Round 2** re-searches in four registers Round 1 never entered, and corrects it in two places. **Round 3** closes the flagged unknowns and produces six sendable outreach documents; it is **complete**, and corrects Round 1 once more.

---

## How to read this report

This document is self-contained. It assumes no other context.

| Task | Question it answers | Method |
|---|---|---|
| **Task 1** | What comparable projects exist anywhere in the world, in any subject domain? | Six independent regional research passes, 250+ named projects individually checked and verified against live sites, 25 August 2026. Organised by continent → country → state/region. Each regional section carries its own comparison tables, dead/dormant list, unverified leads and sources |
| **Task 2** | How does this type of project actually work — funding, team, pipeline, maintainership, lifecycle? | Analysis of the Task 1 corpus, plus independent verification of funding-programme mechanics (grant tiers and envelope sizes) that no project in the corpus publishes |
| **Task 3** | What makes these projects win, and how do they earn? | Analysis of the Task 1 corpus, plus independent verification of real price points for the earned-revenue model |
| **Task 4** | Four follow-ups: disputed-facts precedents in any domain; Darwin Core feasibility; realistic small-team timelines; what validates or contradicts the current approach | Task 1 corpus plus independent verification of standards specifications, GBIF publishing requirements, non-biodiversity disputed-fact systems, and Sathyamangalam's own published figures |
| **R2 · Angle F** | What India-specific source layers exist that the atlas could ingest? | Statutory registers, epigraphy, land records, sacred sites, irrigation heritage, corridor atlases, Tamil-language archives |
| **R2 · Angles A+B** | What did English-only search miss? | Native-language keyword sweeps, and the research literature treated as a project directory |
| **R2 · Angles C+D** | What exists as infrastructure or funding record rather than as a website? | Participant registries, hosted-portal directories, code repositories, funder project databases |
| **R2 · Angles E+G+H** | What do adjacent domains solve, how fast do these resources actually die, and what could Round 1 not read? | Language-documentation archives, link-rot measurement literature, targeted re-checks |
| **R3 · P1** | What should actually be sent, to whom? | Six drafted documents: IFP outreach, TNBB RTI, GBIF eligibility email, plus Keystone, FES and Forest Department enquiries |
| **R3 · P2** | Are the shortlisted stacks alive, and is a free GBIF portal available? | Release histories, participant records, published terms |
| **R3 · P3–P4** | The twelve flagged unknowns from Rounds 1–2 | Seven resolved, two partial, three closed as tested negatives or confirmed environment blocks |

**Excluded by instruction throughout** (already known, not re-profiled): Atlas of Living Australia, NBN Atlas, GBIF, Minnesota Biodiversity Atlas, India Biodiversity Portal. These appear only where a listed project has a direct structural relationship to one of them.

**Corrections.** Round 2 corrects Round 1 in two places and corrects one of its own claims (**§CD.0** Costa Rica, **§EGH.1** Western Ghats Portal). Round 3 corrects Round 1 again (**§P3.9** — India Observatory/IBIS is active, not dormant) and draws the common lesson: **a stale copyright notice has been misread as project death three times across three rounds.** Where a Round 2 section supersedes an earlier one, the earlier section carries a warning pointing forward.

**Verification convention.** Every figure is either quoted from a fetched source or explicitly marked **unverified**. Where a project could not be reached, the reason is recorded (DNS failure, robots.txt disallow, TLS error, JavaScript-only rendering, proxy egress policy) and distinguished from evidence of death. Each task ends with its own sources list and an explicit list of what remains unverified.

---

## The twelve findings that matter most

1. **The disputed-figures feature is not hypothetical — it is a documented defect in Sathyamangalam's own public record.** The reserve publishes at least four different total-area figures and twelve tiger-population figures across five methods, including a 3.8× discrepancy within a single year (2010: 12 vs 46 tigers) and a 2024–25 official camera-trap census (112) that is *lower* than a 2022 official verbal estimate (about 120). See §4.4.1.

2. **Multi-domain place documentation is a genuine, stated gap in India.** Task 1's Asia pass concludes that India's entire small-project layer is single-taxon and occurrence-only, and that "nobody in this layer is doing multi-domain place documentation."

3. **Money does not predict survival; institutional ownership does.** INBio (Costa Rica, 3.7M specimens, international funding) wound down, its CRBio portal is offline, and its CONAGEBIO successor shows blank counters and no news since October 2019. `vncreatures.net` — one person, no institution, no visible funding — added entries on 17 August 2026. See §2.2.2, corrected at §CD.0.

4. **Solo and near-solo projects outlive grant-funded academic projects by three to four times.** Survivors cluster at 15–24 years; grant-funded builds die at years 4–6, on schedule. See §2.6.2.

5. **The presentation layer always dies before the data layer**, and the narrative/historical layer is the most fragile part of a multi-domain atlas. Bayernflora's structured database survived a November 2023 cyberattack; its wiki did not, and remains unrestored. See §2.5.1.

6. **Darwin Core costs almost nothing for the occurrence layer — four required fields — and is irrelevant to the other four domains.** The gazetteer answer is Linked Places Format; there is no standard for legal instruments. The extension worth learning is the Humboldt Extension (51 terms), because it encodes the difference between "looked and did not find" and "was not looking." See §4.2.

7. **A full multi-domain place atlas takes 10–16 years, with no counterexample in the corpus** — but a citable artefact is available in 6–8 weeks (a published scope statement and gap audit) and a DOI'd gazetteer dataset within 12 months. See §4.3.

8. **267,608 statutory village biodiversity registers exist in India, only 11,951 verified, none digitised.** Round 2's largest find: a village-indexed, legally mandated record of local species and traditional knowledge, covering the reserve's own settlements, already written and entirely offline. See §F.1.

9. **The atlas's two hardest layers have already been built once, for this region, by a contactable team.** Muthusankar Gowrappan at the French Institute of Pondicherry leads both the Historical Atlas of South India (an inscriptions-derived historical gazetteer) and the Western Ghats Portal. See §F.4 and §EGH.1.

10. **A national-scale disputed-figures case now exists, from a government research institute.** Two pages of the Wildlife Institute of India's own site describe the same National Wildlife Database with **1,014 vs 682 protected areas** and **175,169 vs 164,074 km²**, neither dated against its figures. See §P3.6.

11. **A stale copyright notice is not evidence of death — it has misled three times.** Costa Rica's CRBio, WII's National Wildlife Database and India Observatory/IBIS were each read as dormant from a dated footer; all three are live or partly live. India Observatory runs a "Code4Nature Challenge 2026" under a 2019 copyright. See §P3.9.

12. **The real prize is consequence, not adoption**: becoming the reference layer a regulatory process must consult. Sathyamangalam sits inside three live figure-generating legal machineries — PARIVESH clearances, Forest Rights Act claims, and NTCA/TIGERNET estimation. Becoming the reference layer for the disagreements *between* them is a position no project in the corpus currently occupies. See §3.5.

---

# Task 1 — Worldwide Classified Inventory: "Comprehensive Place/Population Documentation" Projects

**Prepared for:** the Sathyamangalam Tiger Reserve conservation atlas project
**Compiled:** 25 August 2026
**Method:** Six independent research passes (one per world region) combining web search and direct verification of live project sites (WebFetch) as of 25 August 2026, unless a different date is stated inline. Every entry is either drawn from a fetched/verified source or explicitly marked **unverified**, **partially unverified**, or listed under a region's "Leads I could not verify" subsection. No project name, URL, figure, or team detail below has been invented — where confidence was insufficient, that is stated in the table rather than smoothed over.

**Excluded by instruction** (already known to the requester, not re-profiled below): Atlas of Living Australia (ALA), NBN Atlas (UK), GBIF, Minnesota Biodiversity Atlas, India Biodiversity Portal. These five are referenced only in passing, where a listed project has a direct structural relationship to one of them (e.g. runs on the same software, is a national GBIF node with distinctive extra features, or is a state-level feed into ALA).

**Pattern being searched for:** any website/project — in any subject domain — that documents a place or a population comprehensively, from multiple source types (occurrence records, legal/government instruments, historical documents, literature, gazetteer/place-name data, community knowledge), on one site. This spans biodiversity/species atlases, forest and landscape documentation, single-species or single-reserve wildlife portals, bird atlases, indigenous/tribal documentation projects, marine and botanical atlases, and place-name/historical-document projects.

**Organisation:** by continent, then country, then state/region where the evidence base supports that level of granularity. Within each geography, projects are tabulated by scale, team size/type, scope (occurrence-only vs. multi-domain, with domains listed), subject, and active status with dated evidence. Each continent section ends with its own "Dead/dormant projects found," "Leads I could not verify," and "Sources" subsections — so each regional section is independently referenceable.

**A note on scale:** across the six passes, well over 250 distinct named projects were identified and individually checked. Coverage is necessarily uneven — dense in well-digitised regions (India, South Africa, Europe, North America, Australia/NZ) and thinner where either the underlying projects are genuinely scarcer or language/access barriers limited what a single research pass could verify (parts of Central Asia, Central Africa, francophone West Africa, conflict-affected states). Those gaps are named explicitly in each section's "Leads I could not verify," not silently omitted.

---

## Asia — comprehensive place/population documentation projects

Scope note: verified by opening project sites where possible (August 2026). Where a fact could not be confirmed on the live site or in a paper, it is marked **unverified** rather than estimated. Excluded per instruction: Atlas of Living Australia, NBN Atlas, GBIF, Minnesota Biodiversity Atlas, India Biodiversity Portal (IBP) — IBP is referenced only where another project runs on its software or is a sub-portal of it.

---

### India

#### National (India-wide)

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Biodiversity Atlas – India](https://www.bioatlasindia.org/) | National | Institution-anchored volunteer consortium. Chief Editor **Krushnamegh Kunte** (National Centre for Biological Sciences–TIFR, Bengaluru; Indian Foundation for Butterflies). 1,000+ participants | Multi-domain: occurrence + species pages + larval host plants + nectar plants + ID keys + checklists + 30-minute counts + local projects + publications | Butterflies, moths, Odonata, cicadas, amphibians, reptiles, birds, mammals | Active — launched 2017; project registry describes it as active with 1,000+ participants (verified Aug 2026) | Federated architecture: one platform, many taxon sites ([Butterflies of India](https://www.ifoundbutterflies.org/), Moths of India, Odonata of India, Mammals of India, Reptiles of India, Amphibians of India, Cicadas of India). Butterflies of India claims **>125,000 curated reference images**, "largest library" for the Indian region. Peer-reviewed/curated editorial model — the closest Indian analogue to a "disputed record adjudicated by named editors" workflow |
| [Bird Count India](https://birdcount.in/) | National umbrella | Consortium (Nature Conservation Foundation hosts; partners incl. BNHS, WWF-India, state forest departments); regional coordinators network | Occurrence (eBird-backed) + atlas protocols + events + training material | Bird | Active — homepage referenced eBird India **July 2026 Challenge** and WWF-India Annual Vulture Count **1–30 Sept 2026** (verified Aug 2026) | Hub for all Indian state/city bird atlases; hosts protocol documents (incl. a Nocturnal Bird Count protocol for the Western Ghats). *State of India's Birds 2023* assessed **942 species** from **~30 million observations** |
| [Zoological Survey of India digital archive](https://faunaofindia.nic.in/) | National | Government institution (ZSI, MoEFCC; HQ Kolkata, 16 regional centres) | Literature/document corpus, not occurrence: **200,000+ digitised pages**, searchable PDF + DjVu | Historical document mining / fauna | **Stale** — site copyright notice 2015; several journal series show last entries in the 1990s (verified Aug 2026) | Directly relevant to historical mining: 52 *Fauna of India* volumes, **335** Occasional Papers, **44** Conservation Area Series (several are single-reserve monographs), 23 Ecosystem Series, 48 pictorial handbooks. State fauna inventories for 20 states |
| [India Flora Online / Herbarium JCB](https://indiaflora-ces.iisc.ac.in/) | National (strongest in peninsular India) | Small academic team, Centre for Ecological Sciences, IISc Bengaluru; lead **K. Sankara Rao** | Multi-domain: herbarium specimens + field data + regional floras + landscape-scale floras | Botanical / herbarium | Active — regional flora modules present; no explicit last-updated date (verified Aug 2026) | **Contains a live internal contradiction useful as a design precedent:** the same site states the herbarium holds "more than 20,000 specimens" in one place and "14,000 specimens" in another. Sub-floras: Digital Flora of Karnataka, Digital Flora of Eastern Ghats, Flora of Peninsular India, plus landscape-scoped pages |
| [eFlora of India](https://efloraofindia.com/) | National | Volunteer community (Google Group origin), no single institution | Species pages, images, discussion threads, ID | Botanical | Active — long-running volunteer group with maintained site index (verified Aug 2026) | One of the oldest crowd botany efforts in India; useful as a model for volunteer identification threads as citable evidence |
| [Flowers of India](https://www.flowersofindia.net/) | National | **Solo-to-tiny**: founded 2005 by two laypeople; maintained by **"Tabish"** and **Thingnam Girija**; ~40+ named major contributors | Species pages, images, common names in many Indian languages, state flora lists | Botanical | Active — contributor credits run through 2023; site live (verified Aug 2026) | Best Indian example of a solo-built comprehensive resource that became a community "caravan". Multilingual vernacular-name coverage is directly relevant to gazetteer/name-variant work |
| [ENVIS Centre on Medicinal Plants (FRLHT / TDU)](https://envis.frlht.org/) | National | Institution: FRLHT–TDU Bengaluru + MoEFCC ENVIS; editors incl. **D. K. Ved** (late), **Suma Tagadur Sureshchandra**, **Vijay Barve** | Genuinely multi-domain: nomenclature database, traded-species database, digital herbarium, **digital atlas of geo-distribution**, raw drugs, statewise checklists, conservation-concern species | Botanical / ethnobotanical / trade | **Site states "Last updated: Monday, 18th November 2024"** (verified Aug 2026) — alive but slowing | Started 2002. The nomenclature database is explicitly a synonym/name-conflict resolution tool — the closest Indian precedent to a "disputed name/figure" reconciliation layer |
| [Indian Biodiversity Information System (IBIS) / India Observatory](https://www.indianbiodiversity.org/) · [tool page](https://www.indiaobservatory.org.in/tool/ibis) | National | Institution: Foundation for Ecological Security (FES) | Occurrence + taxonomy modules + museum collection databases + **bibliography of 500,000+ citations**; reptiles/amphibians "work in progress" | Multi-taxon | **Likely dormant** — tool page carries copyright **2006–2019**; reptile/amphibian modules still listed as in progress (verified Aug 2026) | India Observatory as a whole aims to fuse "social, ecological and economic parameters" — the multi-domain ambition closest to the user's atlas concept. The 500k-citation bibliography is the single largest India-focused literature index found |
| [TIGERNET](https://tigernet.nic.in/) | National | Government (National Tiger Conservation Authority) | Tiger mortality + wildlife crime records; state-wise statistics pages | Tiger / legal-enforcement | Active — NTCA maintains parallel mortality pages; site returned robots-blocked to automated fetch, so **last-update unverified** | Launched **January 2010** (announced via TRAFFIC). India's first tiger mortality + crime database. Structurally the closest existing thing to "one authoritative number that journalists then dispute" — press coverage regularly contrasts TIGERNET totals with independent tallies |
| [M-STrIPES](https://www.mstripes.in/) | National (all tiger reserves) | Government (NTCA) + Wildlife Institute of India | Patrol tracks, ecological monitoring, sign surveys, camera-trap workflow | Tiger / protected-area management | Active — login-gated production system (verified Aug 2026) | Login-walled; the *reason* a public reserve atlas has value. Data reaches the public only via the quadrennial All-India Tiger Estimation |
| [PARIVESH — environment & forest clearances](https://environmentclearance.nic.in/) · [forest](https://forestsclearance.nic.in/) · [OGD dataset](https://www.data.gov.in/catalog/environmental-clearance-granted-parivesh-20) | National | Government (MoEFCC) | Legal instruments: EIA notifications and amendments, clearance applications, conditions, compliance reports; bulk CSV via data.gov.in | Legal / environmental clearance | Active — PARIVESH 2.0 datasets published on the Open Government Data platform (verified Aug 2026) | The primary machine-readable source for Indian environmental legal instruments. PARIVESH 2.0 datasets on data.gov.in are the practical ingestion path for a legal-instrument layer |
| [India Environment Portal](https://www.indiaenvironmentportal.org.in/) | National | Institution: Centre for Science and Environment (with NIC) | **>400,000** curated, cross-tagged research reports and government documents | Legal / policy / historical document mining | Active — live, though no explicit last-updated timestamp exposed (verified Aug 2026) | The single largest cross-tagged Indian environmental document corpus. Its cross-tagging model (one document, many topical facets) is a directly transferable pattern |
| [Forest Rights Act resource portal (fra.org.in)](https://www.fra.org.in/) | National | Small NGO team — **Vasundhara**, Bhubaneswar, Odisha | Multi-domain: central and state government orders, guidelines, FAQs, **"evidence" resources (forest management plans, pre-1980 records)**, court cases, claims statistics, GIS maps, RTI resources | Legal / tribal / forest rights | Alive but **stale** — most recent dated notice references a 2022 document number; no site-wide update stamp (verified Aug 2026) | The "evidence" section is essentially historical-document mining for legal use — pre-1980 forest records marshalled as proof of occupation. Very close in spirit to the user's historical-mining + legal-instrument combination |
| [Land Conflict Watch](https://www.landconflictwatch.org/) | National | Small professional team: founder/director **Kumar Sambhav Shrivastava**; **9 core staff + 35+ researchers**; operated by Nut Graph LLP | Multi-domain per case: land area, people affected, investment at stake, legal status, government records, media sources | Legal / land / conflict | Active — public conflicts database live; page copyright 2023, most recent dated member notices June 2023 (verified Aug 2026) | **The single most relevant methodological precedent for the disputed-figures feature.** Four-step verification, with an explicit rule that each dispute and its facts must be verified from more than one source. Also demonstrates the *failure mode* to design against: the homepage simultaneously displays **1,084 ongoing conflicts / 14.1M people affected** and a second view showing **921 conflicts / 10.5M people** — an unlabelled internal conflict in the headline numbers |
| [Wetlands of India Portal](https://indianwetlands.in/) | National | Government (MoEFCC) | Interactive wetland map, National Wetland Inventory & Assessment, resources, e-learning | Wetland / marine-coastal | Active — resources pages live (verified Aug 2026); automated fetch blocked by TLS config | Companion to the **National Wetland Atlas** (SAC/ISRO, 2011) and NWIA atlas series on [VEDAS](https://vedas.sac.gov.in/vcms/en/National_Wetland_Inventory_and_Assessment_(NWIA)_Atlas.html). Precedent for reconciling a satellite-derived inventory against on-ground legal notification |
| [Citizen Science India](https://citsci-india.org/projects/) | National registry | Consortium/aggregator (unverified host) | Registry metadata: 39 projects with lead, platform, start year, status, participation numbers | Meta / all subjects | Active — registry enumerates active projects incl. 2022-start ones (verified Aug 2026) | The most efficient single discovery surface for small Indian projects. Each entry records coordinator name, host organisation, platform, start year and status — a useful schema to copy |
| [SeasonWatch](https://www.seasonwatch.in/) | National | Institution-hosted community (Nature Conservation Foundation) | Phenology observations on tagged trees; schools network | Botanical / phenology | Active — listed as active in the CitSci India registry (verified Aug 2026); site not independently opened, so **record counts unverified** | Long-running tree-phenology time series; relevant as a template for repeat-visit monitoring of named individual features (trees ≈ named places) |
| [Hornbill Watch](https://www.ncf-india.org/eastern-himalaya/hornbill-watch) | National, single family | Small institutional team (Nature Conservation Foundation; Aparajita Datta and colleagues) | Occurrence-focused, crowdsourced; published as a GBIF occurrence dataset | Hornbill (single taxon group) | Active as a project page; **live submission volume unverified** (Aug 2026) | Documented in a data paper in *Indian BIRDS* 14(3) — a good model for how a small single-taxon portal earns citability |
| [Harrier Watch, FireflyWatch, Tiger Beetle Watch, Croc Watch, Shieldtail Mapping Project, Wild Canids–India, Freshwater Turtles & Tortoises of India, Indian Turtle Conservation Action Network, Mapping Invasive Alien Plants, Reeflog](https://citsci-india.org/projects/) | National, single-taxon each | Small teams / individual coordinators (see registry) | Mostly occurrence-only | Bird, insect, reptile, marine, invasive plants | Listed active in registry (verified Aug 2026); individual sites not each opened — **per-project counts unverified** | Collectively the clearest evidence that India's small-project layer is single-taxon and occurrence-only. Nobody in this layer is doing multi-domain place documentation — that is the gap the user's atlas occupies |
| [Traditional Knowledge Digital Library (TKDL)](https://www.csir.res.in/en/documents/tkdl) | National | Government (CSIR + Ministry of Ayush) | Traditional-medicine formulations transcribed from classical texts into patent-examiner-readable form, multilingual | Tribal / traditional knowledge / historical texts | Active as a programme; **access is restricted to patent offices under access agreements** (verified Aug 2026) | Important cautionary precedent: comprehensive historical-text mining built deliberately as a *closed* database, and criticised for it (see Third World Quarterly analysis). Relevant if the atlas plans a traditional-knowledge layer |
| [Sahapedia](https://www.sahapedia.org/) | National | Not-for-profit society, est. **2011**; team size not published | Multi-domain by design: articles, photo essays, video, **oral-history interviews, maps, timelines**, museum partnerships, across **10 declared domains** including "Natural Environment" and "Built Spaces" | Gazetteer-adjacent / cultural encyclopedia | Active (verified Aug 2026); **article counts not published** | The closest Indian structural analogue to "comprehensive multi-source documentation of a place, on one site" outside biodiversity. Its 10-domain taxonomy plus interview/map/timeline object types is worth copying wholesale |
| [Imperial Gazetteer of India, Digital South Asia Library](https://dsal.uchicago.edu/reference/gazetteer/) | National (colonial-era) | Institution: University of Chicago / DSAL consortium | Full-text + page-image gazetteer, plus **[1909](https://dsal.uchicago.edu/reference/gaz_atlas_1909/) and [1931](https://dsal.uchicago.edu/reference/gaz_atlas_1931/) atlases** and [maps from vols 1–24](https://dsal.uchicago.edu/maps/gazetteer/index.html) | Gazetteer / place-names / historical | Active/stable archive (verified Aug 2026) | **The single most important gazetteer resource for this atlas.** Provides the colonial place-name baseline against which modern Tamil Nadu names can be reconciled. Page-image + transcription pairing is the right model for citable historical claims |
| [South Asia Open Archives (SAOA)](https://southasiaoa.org/) | National / regional | Library consortium; fiscal sponsor CLIR; delivered on JSTOR | Open-access digitised colonial and vernacular print: journals, reports, records | Historical document mining | Active — 2023 Annual Activities Report and a **FY2026–2027 transition plan** published (verified Aug 2026) | Passed **one million pages** (JSTOR Daily). Free-to-use complement to commercial *Gazetteers of British India, 1833–1962* (Brill, paywalled) |
| [The Gazetteer Project / The Districts Project, FLAME University](https://www.flame.edu.in/cka/the-gazetteer-project.php) | National, district-by-district | Small academic team, Centre for Knowledge Alternatives, FLAME University; concept note by **Yugank Goyal**, July 2021 | Multi-domain district documentation intended to replace/supersede colonial gazetteers | Gazetteer | **Status uncertain** — concept note dated July 2021; no published district volumes found (verified Aug 2026) | Explicitly framed as reviving the gazetteer form "minus the colonial lens" (see [The Print](https://theprint.in/ground-reports/india-is-reviving-gazetteers-minus-the-colonial-lens-moradabad-is-leading-the-way/2815525/) on the parallel Moradabad government effort). Directly overlapping ambition — worth contacting |
| [mahoot](https://mahoot.xyz/) | Transnational (Asian elephant range) | Appears **solo/anonymous** — no named team on site | Multi-domain: named individual elephants, camps/facilities, wild range maps and corridors, mahout-culture field notes, visit planning | Elephant (single species) | Active — records synced from elephant.se, **last synced 17 June 2026** (verified Aug 2026) | Interesting as a solo multi-domain build: **8,297 named elephants / 14,784 total records / 14,057 living / 1,864 camps**. But treat its numbers with suspicion — the site simultaneously advertises "14,000+ named individuals" against its own 8,297 figure, and claims coverage of "94 countries" across "13 Asian range states." A live example of exactly the unreconciled-figure problem the atlas is meant to solve |
| [Wild Atlas](https://www.wildatlas.in/) | National | **Undisclosed** — no team, institution or methodology published | Park pages, species pages, sighting data, safari booking redirect | Other (commercial aggregator) | Live (verified Aug 2026); no dates or counts published | Claims "100+ national parks, 500+ species." Included as an anti-pattern: an "atlas" with no provenance, no dates and a booking funnel. Useful contrast for the user's provenance-first positioning |

#### Tamil Nadu (user's own state — closest neighbours)

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Moths of Krishnagiri](https://citsci-india.org/projects/project/moths-of-krishnagiri/) | Single district | Small NGO — **Kenneth Anderson Nature Society (KANS)**, Hosur; coordinator **Surya Raghavendar**; 0–100 participants | Occurrence-only (iNaturalist-hosted) | Moth | **Active — started 2022**, registry status "Active" (verified Aug 2026) | **Geographically the nearest comparable effort to Sathyamangalam** — Krishnagiri/Hosur adjoins the STR landscape. Runs on iNaturalist rather than custom software; data immediately open |
| [Biodiversity of Krishnagiri](https://citsci-india.org/projects/project/biodiversity-of-krishnagiri/) | Single district | Same small-NGO cluster (KANS) — **team details unverified** | Occurrence-only | Multi-taxon | Listed active (verified Aug 2026) | Companion project; the district-scale multi-taxon framing is the pattern the STR atlas extends |
| [Coimbatore City Bird Atlas](https://birdcount.in/coimbatore-city-bird-atlas/) | Single city | Consortium: **SACON** (Salim Ali Centre for Ornithology and Natural History) + Bird Count India + Kerala birders (technical), Coimbatore birdwatchers (field) | Occurrence-only, structured grid | Bird | Surveys ran **2020–2022**, Feb and June rounds; **current status unverified** (verified Aug 2026) | **37 grid cells of 3.3 × 3.3 km, 333 sub-cells, 111 randomly selected sub-cells surveyed**; 4 × 15-minute travelling lists per sub-cell, 06:00–09:00. eBird account `kovaibirdatlas`. Methodologically inherited from Mysore and Kerala atlases. Finest-grain published protocol found in India |
| [Bird Gap Filling in Tamil Nadu](https://citsci-india.org/projects/project/bird-gap-filling-in-tamil-nadu/) | State | Partnership: Nature Conservation Foundation + **Tamil Birders Network** (Tirupur); coordinator **Vaazai Kumar P** | Occurrence-only (eBird + iNaturalist) | Bird | **Active — started 2022** (verified Aug 2026) | Explicitly a *coverage-gap* project — deliberately targets under-birded cells. Directly useful: Sathyamangalam sits in exactly the kind of gap this project targets |
| [Pongal Bird Count](https://citsci-india.org/projects/project/pongal-bird-count/) | State, annual event | Community/volunteer, Bird Count India-affiliated | Occurrence-only, time-boxed | Bird | Active annual event (verified Aug 2026); **organiser names unverified** | Culturally anchored survey window (Pongal) — a good precedent for tying survey effort to local calendars |
| [Biodiversity of Chennai](https://citsci-india.org/projects/project/biodiversity-of-chennai/) | Single city | Small/community — **unverified** | Occurrence-only | Multi-taxon | Listed active (verified Aug 2026) | — |
| [Keystone Foundation Archives](https://keystone-archives.org/archive/) | Single landscape (Nilgiri Biosphere Reserve, 3 states) | Small NGO — Keystone Foundation, Kotagiri, operating since **1993** | **Genuinely multi-domain**: **464 images, 53 audio files, 22 videos, 503 text documents, 13 other items**; themed by water, flora, wildlife, agriculture, biodiversity, indigenous cultures, land & livelihoods, food sovereignty, climate, ethnobotany; organised by NBR sub-regions | Forest / tribal / ethnobotany / archive | Live; **no update date published** (verified Aug 2026) | **The single closest existing project to what the user is building.** A landscape-scoped, multi-source, multi-format archive of the adjacent Nilgiri Biosphere Reserve, run by a small team. Appears to be built on **Omeka** (URL structure and UI; not stated explicitly). Includes oral/audio material — an object type most biodiversity atlases lack |
| [Keystone Foundation — biodiversity programme](https://keystone-foundation.org/biodiversity/) | Single landscape (NBR) | Small NGO; founded the **Nilgiri Natural History Society**; holds the chair of the IUCN/SSC Western Ghats Plant Specialist Group | Arboretum of **33 endangered Western Ghats tree species** with a tracking spreadsheet; species awareness materials | Botanical / forest | Active; project cycle "Building Lifetime Relationship with the Nilgiris Biosphere Reserve" funded **2020–2022** (verified Aug 2026) | Their species tracking currently lives in a **Google spreadsheet** — a concrete, adjacent dataset that a proper atlas could absorb |
| [Nilgiri Documentation Centre](https://nilgiridocumentation.com/) | Single landscape (Nilgiris) | Small NGO / arguably a personality-led centre; **staff names not published on site** | Collects and collates "Nilgiri-related material scattered in India and around the globe"; runs the **John Sullivan Memorial** and the **Nilgiri History Museum** | Gazetteer / historical / advocacy | Live but **thin and undated** — no collection counts, no update stamp; only a page-hit counter (verified Aug 2026) | Phase 1 as **Save Nilgiris Campaign 1985–2006**, Phase 2 as NDC **2006–present**. A 40-year documentation effort whose holdings are essentially *not online* — the strongest argument in this landscape for building a real digital atlas |
| [Nilgiri Archaeological Project](https://www.nilgiri.ugent.be/) | Single landscape (Nilgiris) | Academic team, **Ghent University**; **no lead named on site** | Four integrated datasets: megalithic tombs + pollen/phytolith sampling; **museum grave-goods collections (British Museum, Chennai Government Museum, Museum für Asiatische Kunst Berlin)**; **Old Kannada inscriptions and Old Tamil literature alongside contemporary oral histories**; **colonial herbaria and *Hortus Indicus Malabaricus* (1678–1693)** for indigenous ecological knowledge | Historical / archaeology / tribal / botanical | Site copyright **2026** (Ghent University), suggesting maintenance; **no project timeline or recent output dates published** (verified Aug 2026) | **The best methodological precedent in the region for the historical-document-mining pillar.** Covers 1 CE to ~1835 (Indo-Roman trade to British tea plantations) and explicitly triangulates inscriptions, colonial herbaria, museum collections and living oral history — exactly the multi-source-reconciliation problem the atlas faces |
| [Tamil Nadu District Gazetteers](https://en.wikipedia.org/wiki/Tamil_Nadu_District_Gazetteers) | State, district-by-district | Government series (Tamil Nadu Gazetteers Department) | Place-names, administration, history, natural features | Gazetteer | Series exists; **no state-run searchable digital portal found** (verified Aug 2026) | The Erode / Coimbatore / Salem district gazetteers are the primary local-name authority for the STR landscape and are **not** available as structured data — a clear build opportunity |

#### Kerala

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Kerala Bird Atlas](https://birdcount.in/kerala-bird-atlas/) | State | Large consortium: Bird Count India + **Kerala Forest Department** + **Kerala Agricultural University**; overall leads **P. O. Nameer** and **Ratish R. L.**; district coordinators in all 14 districts; hundreds of volunteers | Occurrence-only but exceptionally structured; district-level published atlases | Bird | **Active and in second cycle** — Atlas 1.0 surveys 2015 → **13 Sept 2020**, atlas released **25 Jan 2021**; **Atlas 2.0 inaugurated 15 July 2026** by Kerala Forest Minister Shibu Baby John, wet season 16 July–13 Sept 2026 (verified Aug 2026) | **India's first state-wide bird atlas.** **4,324 survey sub-cells across 38,863 km²**, 6.6 × 6.6 km grid. Six published district atlases (Alappuzha, Thrissur, Kannur, Kasaragod, Kottayam, Kozhikode). Peer-reviewed in *Current Science* ([122(3):298](https://www.currentscience.ac.in/Volumes/122/03/0298.pdf)) and *Forktail*. The gold standard for making a citizen-science atlas academically citable and government-adopted |
| [Kerala Forest Research Institute — web resources](https://www.kfri.res.in/about/kfri-websites.asp) | State | Government research institute (KSCSTE–KFRI, Peechi) | A cluster of separately-built databases; a Forest Management Information System division; forest statistics pages | Forest / botanical | Live index page (verified Aug 2026); **individual database freshness unverified** | Illustrates the common institutional failure mode: many small disconnected databases behind one "Our Websites" list rather than one integrated atlas |

#### Karnataka

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Mysuru City Bird Atlas](https://birdcount.in/mysore-bird-atlas/) · [data on Dryad](https://datadryad.org/dataset/doi:10.5061/dryad.8k3d81r) | Single city | Small volunteer team + Bird Count India support | Occurrence-only; **dataset archived with a DOI** | Bird | **Completed 2014–2016**; dataset and paper published (verified Aug 2026) | Published in *Indian BIRDS* 15(3). The **Dryad DOI deposit** is the key transferable practice: a small city atlas that made its raw data permanently citable. Methodological parent of the Coimbatore atlas |
| [Digital Flora of Karnataka (Herbarium JCB)](https://indiaflora-ces.iisc.ac.in/FloraKarnataka/welcome.php) | State | Small academic team, CES–IISc | Herbarium specimens + species pages + images | Botanical | Live (verified Aug 2026); **counts unverified** | Sub-flora of India Flora Online; a working example of state-scoped flora within a national platform |

#### Maharashtra

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Pune Bird Atlas](https://birdcount.in/pune-bird-atlas/) · [data portal](https://five.epicollect.net/project/pune-bird-atlas/data) | Single city | Volunteer collective + Bird Count India | Occurrence-only | Bird | Active (verified Aug 2026); **latest season dates unverified** | Notable tech choice: runs data collection on **Epicollect5** rather than eBird alone — a cheap off-the-shelf stack worth knowing about |
| [Marine Life of Mumbai](https://citsci-india.org/projects/project/marine-life-of-mumbai/) | Single city coastline | Small collective (Coastal Conservation Foundation and partners — **composition unverified**) | Occurrence + shore-walk outreach + species guides | Marine / coastal | Listed active (verified Aug 2026) | The best-known Indian urban intertidal documentation effort; strong public-engagement model |

#### Goa

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| Goa Bird Atlas (via [GBCN](https://www.gbcn.in/) and [Bird Count India](https://birdcount.in/)) | State | **Goa Bird Conservation Network** (registered society, founded **2010**) + Goa Forest Department + Bird Count India consortium | Occurrence-only; follows the Kerala protocol | Bird | **Published — India's second state bird atlas** (widely reported 2024–25; see [Drishti IAS](https://www.drishtiias.com/state-pcs-current-affairs/goa-becomes-second-indian-state-to-publish-a-bird-atlas)); GBCN active, monthly bird walks, Asian Waterbird Census since 2011 (verified Aug 2026) | GBCN also drove designation of **4 Important Bird Areas in 2011** and proposed 3 more in 2014 — a small society that has produced legally-referenced spatial designations. Note: GBCN's own About page does **not** mention the atlas, so attribution between GBCN and Bird Count India is **unresolved** |

#### Telangana / Andhra Pradesh

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Hyderabad Bird Atlas](https://birdcount.in/hyderabad-bird-atlas/) | Single city | Volunteer network (~200–220 volunteers per season) + Bird Count India | Occurrence-only, twice-yearly, minimum 3-year commitment | Bird | **Most demonstrably active atlas found in India.** Season 1 Feb 2025 (209 volunteers) → Season 4 Jul 2026 (200+ volunteers, 158 species, 75,000+ birds). Seasons 5 and 6 scheduled Feb 2027 and Jul 2027 (verified Aug 2026) | **45 grids of 6.6 × 6.6 km, ~180 sub-cells sampled per season. 248 unique species after four seasons** (Season 4 additions: Greater Flamingo, Yellow-footed Green-Pigeon, Brown-capped Pygmy Woodpecker). Publishes per-season volunteer counts, species counts and individual-bird counts — the best public "audit trail" of effort of any Indian atlas |
| Tirupati Bird Atlas (via [Bird Count India](https://birdcount.in/)) | City/region | Volunteer + Bird Count India; **coordinators unverified** | Occurrence-only, Kerala protocol | Bird | Described as ongoing in Bird Count India's [*Bird Atlases in India*](https://birdcount.in/wp-content/uploads/2024/07/Bird-Atlases-in-India-1.pdf) (2024); **current status unverified** | — |
| [Intertidal Biodiversity of Andhra Pradesh](https://citsci-india.org/projects/project/intertidal-biodiversity-of-andhra-pradesh/) | State coastline | Small team — **unverified** | Occurrence + education | Marine / coastal | Listed active (verified Aug 2026) | — |

#### Gujarat

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Charotar Crocodile Count](https://citsci-india.org/projects/project/charotar-crocodile-count/) | Single sub-region (Charotar) | Small NGO (Voluntary Nature Conservancy — **unverified**) | Population census, occurrence | Single species (mugger) | Listed active (verified Aug 2026) | A rare Indian example of a repeated *population census* at sub-district scale — closer to "document a population comprehensively" than most occurrence portals |
| [UPL Sarus Conservation Project](https://citsci-india.org/projects/project/upl-sarus-conservation-project/) | Regional | Corporate-funded (UPL) + volunteers | Population monitoring | Single species (Sarus crane) | Listed active (verified Aug 2026) | Corporate-sponsored single-species population documentation — a funding model worth noting |
| [Ahmedabad Bird Atlas](https://birdcount.in/ahmedabad-bird-atlas/) | Single city | Volunteer + Bird Count India | Occurrence-only | Bird | Listed on Bird Count India (verified Aug 2026); **season dates unverified** | — |

#### Bihar / eastern India

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Bhagalpur Bird Atlas](https://birdcount.in/bhagalpur-bird-atlas/) | District/city | Volunteer + Bird Count India | Occurrence-only | Bird | Listed as a systematic atlas in the 2024 Bird Count India review (verified Aug 2026); **current season unverified** | Notable as a rare structured atlas outside the south/west; overlaps the Vikramshila Gangetic Dolphin Sanctuary landscape |

#### Assam & Northeast India

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Assam Biodiversity Portal](https://assambiodiversity.indiabiodiversity.org/) | State | State-scoped instance of the Biodiversity Informatics Platform; institutional partners **unverified** | Multi-domain within the platform: observations, species pages, **document repository**, maps | Multi-taxon | Live; **observation counts and latest observation dates could not be retrieved (403 to automated fetch)** — activity level **unverified** (Aug 2026) | The clearest existing proof that a *state/region-scoped* portal can be spun up on the same open-source stack rather than built from scratch — the most directly reusable technical route for a Sathyamangalam atlas |
| [Bihu Bird Count](https://citsci-india.org/projects/project/bihu-bird-count/) | State, annual event | Assam Bird Monitoring Network | Occurrence-only, time-boxed | Bird | Listed active (verified Aug 2026) | Second example (with Pongal Bird Count) of anchoring survey effort to a regional festival |
| [Hornbill Watch](https://www.ncf-india.org/eastern-himalaya/hornbill-watch) | National, NE-focused origin | NCF Eastern Himalaya programme | Occurrence | Hornbill | See national table | — |

#### Andaman & Nicobar Islands / Lakshadweep

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Andaman Nicobar Environment Team (ANET)](https://anetindia.org/) | Single island group | Small NGO with a named team page | Deliberately interdisciplinary: marine, terrestrial, community-oriented, island sustainability; establishing **India's first Centre for Island Sustainability and Long-Term Ecological Observatory** | Marine / forest / tribal | Active — CIS described as being established (verified Aug 2026); **no named public database found; dataset names unverified** | Explicitly commits to "blurring of disciplinary boundaries" and to "cognitive justice" — i.e. treating indigenous knowledge as a co-equal source rather than an appendix. Philosophically the closest match to a biocultural atlas, but **their data is not published as a portal** |
| [Community-Based Fisheries Monitoring (CBFM), Lakshadweep](https://citsci-india.org/projects/project/community-based-fisheries-monitoring-cbfm-in-the-lakshadweep/) | Single island group | Small team (host organisation **unverified**) | Catch/fisheries data with community participation | Marine | Listed active (verified Aug 2026) | Community-held data governance model for a small island population |

#### Western Ghats (multi-state landscape)

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [The Western Ghats Portal](https://thewesternghats.indiabiodiversity.org/) | Transboundary landscape (6 states) | Consortium; **[French Institute of Pondicherry](https://www.ifpindia.org/projects/western-ghats-portal-india-biodiversity-portal/)** is a documented partner | Multi-domain within the platform: observations, species pages, documents, maps | Forest / multi-taxon | Live but **could not be read (403 to automated fetch)**; counts and recency **unverified** (Aug 2026) | **The most direct precedent for a landscape-scoped Indian atlas** — a named biogeographic region, not an administrative unit, given its own portal on shared infrastructure. Sathyamangalam sits inside this landscape, so this is both a comparator and a likely data partner |
| [Biodiversity of Western Ghats (iNaturalist project)](https://www.inaturalist.org/projects/biodiversity-of-western-ghats) | Landscape | Community, no institution | Occurrence-only | Multi-taxon | Live on iNaturalist (verified Aug 2026); **record counts unverified** | The zero-cost route: an umbrella iNat project rather than a built portal. Worth knowing as the baseline the atlas must beat |
| [BIOTIK](http://www.biotik.org/) | Landscape (Western Ghats trees) | Multi-institution EU-funded project incl. French Institute of Pondicherry | Computer-aided tree identification keys, species pages | Botanical / forest | **Dying** — TLS certificate is invalid for the host (hostname mismatch, Aug 2026), indicating unmaintained infrastructure | See dead/dormant section. A cautionary case: a well-funded Western Ghats identification tool now decaying |

---

### Sri Lanka

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Sri Lanka Biodiversity Clearing House Mechanism](https://lk.chm-cbd.net/) | National | Government (Ministry of Environment, Biodiversity Division), under the CBD Secretariat's CHM network | **Genuinely multi-domain**: 12 documented ecosystems, **national legislation/legal instruments**, NBSAP and national reports, **43 protected areas**, 25 projects, 44 news items, 12 events, 20 national targets mapped to Aichi targets, 8 videos, 16 galleries | Legal / protected area / policy | Active — news to **October 2025**, events listed through **June 2026** (verified Aug 2026) | The best example in South Asia of a portal that puts **species/ecosystem data and legal instruments side by side** in one navigation. Also hosts the [National Red List 2020 flora volume](https://lk.chm-cbd.net/sites/lk/files/2022-06/The%20National%20Red%20List%202020-The%20Conservation%20Status%20of%20the%20Flora%20of%20Sri%20Lanka.pdf) and the [Biodiversity Profile](https://lk.chm-cbd.net/sites/lk/files/2022-06/Biodiversity_ProfileSriLanka.pdf) — successive Red Lists (2007, 2012, 2020) give a real worked example of *the same species assessed differently over time*, i.e. exactly the disputed-figure case |
| [Dilmah Conservation — Butterflies of Sri Lanka / butterfly garden](https://www.dilmahconservation.org/butterfly-garden/) | National, single taxon | Corporate CSR programme (Dilmah Tea) | Species pages, e-publications, garden/site documentation | Butterfly | Active (verified Aug 2026); **counts unverified** | Corporate-funded single-taxon documentation with a free e-publication library — an unusual and durable funding model |
| [Biodiversity of Sri Lanka (blog)](https://biodiversityofsrilanka.blogspot.com/) | National | Apparently **solo/anonymous** | Species lists and notes | Multi-taxon | Live Blogspot; **last-post date unverified** (Aug 2026) | Representative of the large hidden layer of Asian solo documentation efforts living on free blogging platforms — high mortality, low citability |

Note: no independent national Sri Lankan species-occurrence portal comparable to MyBIS or TaiBIF was found. The 2007/2012/2020 Red Lists and the CHM are the national record. This is a genuine gap.

---

### Nepal

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Himalayan Nature](https://www.himalayannature.org/) | National | Small research institute, founded **2000**, registered charity No. 818/056/57; patron **Carol Inskipp**; **Dr. Hem Sagar Baral** and **Tulsi Subedi** publish under it | Multi-project rather than a single portal: **National Red List of Birds**, **Kosi Bird Observatory** (a permanent research station), Asian Waterbird Census | Bird | Active — Asian Waterbird Census 2024 listed; blog activity from 2020 onward (verified Aug 2026) | The [National Red List of Nepal's Birds](https://www.himalayannature.org/works/projects/national-red-list-of-nepals-birds/) pairs *global* and *national* conservation status for each species — a two-column status model that is itself a designed way of presenting two legitimate, conflicting assessments side by side. Strongly worth copying |
| [National Trust for Nature Conservation — species pages](https://ntnc.org.np/thematic-area/species) · [Biodiversity Conservation Center](https://ntncbcc.org.np/services/species/) | National / single-landscape (Chitwan) | Large national NGO | Species accounts, programme documentation | Multi-taxon / rhino / tiger | Live (verified Aug 2026); **update cadence unverified** | The BCC pages are effectively single-landscape (Chitwan) documentation embedded in an institutional site rather than a portal |

No Nepal Bird Atlas equivalent to the Kerala atlas was found; Nepal's bird knowledge base rests on Inskipp-lineage checklists and the national Red List. **This is a gap, not a finding of absence — searched but not exhaustively.**

---

### Bhutan

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Bhutan Biodiversity Portal](https://biodiversity.bt/) | National | Six-institution consortium: **National Biodiversity Centre; Ministry of Agriculture and Forests (ICS); College of Natural Resources, Royal University of Bhutan; Department of Forests and Parks Services; Ugyen Wangchuck Institute for Conservation and Environmental Research; WWF-Bhutan**. Tech partner **Strand Life Sciences** | Multi-domain: **8,160 species, 117,000 observations, 2,850 registered users, 314 documents in a literature repository, 45 maps** | Multi-taxon national | Active — live counters, recent observations displayed; also publishes to GBIF as a [dataset](https://www.gbif.org/dataset/926f3dd4-89d9-4cbd-8ed7-32fb6fcfa861) (verified Aug 2026) | **The most important single comparator for the user.** Runs on the **open-source Biodiversity Informatics Platform v5.0.5** (the same codebase family as the excluded India Biodiversity Portal), available on GitHub, with Mapbox for mapping. It proves that a small country/landscape can stand up a multi-domain portal — species + observations + **documents** + **maps** — on existing open-source software rather than building one. Has an installable web app on Google Play |
| [Biological Specimen Collections of Bhutan](http://bhutanbiodiversity.net/) | National | Academic, US Fulbright-supported; **institution/lead unverified** | Specimen records, checklists, distribution maps, identification keys | Herbarium / specimens | **Dormant** — site copyright **2017**, no later activity indicators (verified Aug 2026) | Built on **Symbiota**. Two Bhutanese portals on two different stacks with overlapping mandates — a real-world cautionary tale about duplicate portals, and a source of divergent species counts |

---

### Bangladesh

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Marine Biodiversity Portal of Bangladesh](https://marinebiodiversity.org.bd/) | National (marine) | **Single-lab project**: Prof. Dr. **Kazi Ahsan Habib**, Aquatic Bioresource Research Lab, Dept. of Fisheries Biology and Genetics, Sher-e-Bangla Agricultural University, Dhaka | Multi-domain: **>2,500 marine species across 12 groups** (incl. **765 fish**, **644 molluscs**, 7 seagrasses, 1 tunicate), **7 habitat classes** (offshore, sea bottom, coral reef, sandy shore, estuary, mangrove wetland, mud flat), plus a **DNA barcode library** | Marine | Published **2023**; live (verified Aug 2026); **update cadence unverified** | **The best "small team, comprehensive, one place" exemplar in South Asia.** One university lab produced a habitat-classified national marine encyclopedia with a genetic reference layer, and documented it as a [data paper](https://www.researchgate.net/publication/377965492_Marine_Biodiversity_Portal_of_Bangladesh_marinebiodiversityorgbd_A_smart_online_encyclopedia_of_marine_fauna_and_flora_of_the_country). The habitat-class facet is a directly transferable schema idea |
| [ENGAGE4Sundarbans — archival research](https://engage4sundarbans.org/archival-research/) | Single landscape (Sundarbans, transboundary) | Academic consortium; **membership unverified** | Archival/historical research strand within a wider project | Historical / forest / delta | Live (verified Aug 2026); **outputs and dates unverified** | One of very few explicitly *archival* strands in a South Asian landscape project |
| [Bangladesh Fisheries Information Share Home](https://en.wikipedia.org/wiki/Bangladesh_Fisheries_Information_Share_Home) | National | **Unverified** | Fisheries species information | Marine / fisheries | **Status unverified** — documented on Wikipedia; live site not confirmed (Aug 2026) | Listed as a lead, not a verified project |

Also relevant: the printed **[Sundarbans Atlas: Bangladesh Forest Compartment Maps and Gazetteer](https://www.nhbs.com/sundarbans-atlas-book)** — a rare example of a single-reserve atlas that pairs forest compartment maps with a **gazetteer**, i.e. exactly the map + place-name pairing the user needs. Print only; no digital edition found.

---

### Pakistan

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Flora of Pakistan (eFloras)](http://www.efloras.org/flora_page.aspx?flora_id=5) | National | Missouri Botanical Garden + Harvard University Herbaria hosting | Taxonomic treatments, district and grid maps, inventory of Pakistani plants | Botanical | **Explicitly dead** — the page itself states "this is a legacy site which is now out of date" and redirects users to tropicos.org (verified Aug 2026) | Rare case of a project that *labels its own obsolescence*. Good practice worth imitating: an honest staleness banner rather than a silently rotting page |
| Biodiversity of Pakistan: Database and Global Networking (BGN) | National | Academic; **institution and lead unverified** | Species database | Multi-taxon | **Unverified** — known only from a [2022 paper](https://www.researchgate.net/publication/366191883_BIODIVERSITY_OF_PAKISTAN_DATABASE_AND_GLOBAL_NETWORKING_BGN); no live URL located (Aug 2026) | Classic "paper describes a portal, portal cannot be found" case — see unverified leads |

---

### China (mainland)

China has the densest layer of small and solo taxon portals found anywhere in Asia. The most complete discovery source is a community-compiled list: [Useful websites about China's biodiversity](https://www.inaturalist.org/posts/109663-useful-websites-about-china-s-biodiversity) (iNaturalist).

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Chinese Virtual Herbarium (CVH)](https://www.cvh.ac.cn/) | National | Institution: Institute of Botany, Chinese Academy of Sciences; aggregates **16 major herbaria** + 4 specialised sub-platforms | Specimens: **8.6 million+ specimen records, 66,192 type specimens, 66,113 colour photographs** | Botanical / herbarium | Active — copyright span **2004–2025**; no explicit last-update stamp (verified Aug 2026) | Largest herbarium digitisation aggregation in Asia. Operates as the online sharing platform of the National Plant Specimen Resource Center |
| [National Specimen Information Infrastructure (NSII)](http://nsii.org.cn/2017/home.php) | National | Institution / national infrastructure programme | Cross-taxon specimen aggregation | Specimens, multi-taxon | Live; **could not be read (TLS hostname mismatch, Aug 2026)** — counts **unverified**. The broken certificate is itself weak evidence of thin maintenance | Also hosts a [Chinese University iPlant Association / CampusFlora](http://site.nsii.org.cn/CampusFlora.html) module — campus-scale floras, a nice small-area precedent |
| [Catalogue of Life China (Species 2000 China node)](http://sp2000.org.cn/) | National | Institution (CAS-affiliated) | Annual national checklist | Taxonomy | Live; **fetch blocked (TLS wrong-version error, Aug 2026)** — checklist year and totals **unverified** | Publishes a dated annual checklist edition — a good model for versioned authority lists that let you cite "the 2024 view" vs "the 2026 view" |
| [AmphibiaChina](https://www.amphibiachina.org/) | National, single class | Institution: Kunming Institute of Zoology, CAS; **named lead not published** | Species pages, taxonomy, image galleries, **annual taxonomic-change updates** | Amphibian | **Active — 759 species as of 23 July 2026**; launched **2014** (verified Aug 2026) | The **annual taxonomic-change log** is the single best feature found in Asia for the user's disputed-data problem: it explicitly records what changed, when, and why — turning taxonomic disagreement into a browsable history rather than a silent overwrite |
| [Plant Photo Bank of China](http://ppbc.iplant.cn/) | National | Institution + large volunteer photographer base | Images keyed to taxa | Botanical | Live (verified Aug 2026); **counts unverified** | — |
| [Flora Reipublicae Popularis Sinicae (FRPS)](http://www.iplant.cn/frps) · [Flora of China (eFloras)](http://www.efloras.org/flora_page.aspx?flora_id=2) | National | Institutional consortia (CAS; MBG/Harvard for the English edition) | Full taxonomic flora, digitised | Botanical | Live (verified Aug 2026) | Two editions of the same flora (Chinese and English) that **do not always agree** — a ready-made worked example of two authoritative sources in conflict |
| [Shanghai Digital Flora](https://shflora.ibiodiversity.net/) · [Hangzhou Online Flora](http://db.hzbg.cn/hzflora/) · [Flora of Jiangsu](http://www.jszwzw.com/HomePage/) | City / province | Small institutional teams (botanical gardens, provincial bodies) | Regional floras | Botanical | Live per community list (verified Aug 2026); **individual freshness unverified** | **Directly relevant: three separate examples of city- and province-scoped comprehensive floras** — the sub-national scale the user is working at |
| [Biotracks](http://www.biotracks.cn/) · [Biogrid](https://biogrid.scbg.ac.cn/app) | National | Biogrid hosted by South China Botanical Garden, CAS | Occurrence recording platforms (iNaturalist-like) | Multi-taxon | Live per community list; the list recommends Biogrid **over** Biotracks (verified Aug 2026) | Community preference documented in the source — informal evidence that Biotracks is losing ground |
| [China Bird Report Center](http://www.birdreport.cn/) | National | Birdwatcher community + institutional support | Bird records | Bird | Live per community list (verified Aug 2026); **counts unverified** | China's main national bird records repository |
| [Dongniao](https://dongniao.net/) | National / East Asia | Appears **small team or solo**; **unverified** | Bird species pages and images | Bird | Live; **could not be read (TLS hostname mismatch, Aug 2026)** — activity **unverified** | — |
| [Chinese Terrestrial Mollusc Database (shellsmap)](https://shellsmap.com/) · [GobiologyCN](https://gobiologycn.com/) · [Odonata Research (china-odonata.top)](https://www.china-odonata.top/) · [Ant Network](http://www.ants-china.com/index.html) · [Insecta Integration Specimen Database](https://insectaintegration.com/) · [Mycopedia](http://www.mycopedia.top) · [Liu Dayadan's photo bank](http://www.liudayadan.com) | National, single taxon each | **Mostly solo or hobbyist-collective** — the clearest cluster of solo taxon portals found in Asia | Occurrence + images + taxonomy, variable | Mollusc, goby, dragonfly, ant, insect, fungi, arthropod | Listed as live by the community compilation (Aug 2026); **none independently verified for recency** | Collectively the most important finding for the user's "look hard for small and solo projects" brief: **a whole hidden ecology of single-person taxon encyclopedias, discoverable only via a community-maintained link list, not via search engines.** Note `liudayadan.com` is named after an individual — an explicitly personal domain |
| [China Animal Scientific Database](http://zoology.especies.cn/) · [China Biological Chronicles Library](https://species.sciencereading.cn/biology/v/biologicalIndex/122.html) | National | Institutional | Faunal literature and data; the Chronicles Library is **paywalled and incomplete** per the community list | Multi-taxon / literature | Live per community list (Aug 2026) | Paywalled national fauna literature — a comparator for the openness question |

#### Hong Kong

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Hong Kong Biodiversity Information Hub (HKBIH)](https://bih.gov.hk/) | Single territory | Government: Agriculture, Fisheries and Conservation Department (AFCD) + collaborating partners | Multi-domain: **Species Database, Multimedia Database, Biodiversity Geographic Information System, education-programme platform, thematic pages** | Multi-taxon territorial | Active — launched **1 March 2022** ([government press release](https://www.info.gov.hk/gia/general/202203/01/P2022030100691.htm)); About page metadata dated **17 September 2025** (verified Aug 2026) | **The best single-territory multi-domain government atlas in Asia,** and an excellent size match: a small area documented properly. Built under the Hong Kong Biodiversity Strategy and Action Plan, and explicitly describes curation as "an evolving process where information is continuously amassed, updated and fine-tuned" — honest framing about provisionality. Its predecessor, [Hong Kong Biodiversity Online](https://www.afcd.gov.hk/english/conservation/hkbiodiversity/hkbiodiversity.html), still exists in parallel |

---

### Taiwan

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Taiwan Biodiversity Information Alliance (TBIA)](https://tbiadata.tw/en-us/) | National | **Consortium of 19+ members** — Academia Sinica research centres, National Museum of Natural Science, National Taiwan Museum, National Museum of Marine Biology, NTU School of Forestry, Taiwan Forestry Research Institute, plus Water Resources, Ocean Conservation and Rural Development agencies | Aggregated occurrence records, collections, datasets from Taiwanese institutions | Multi-taxon | Active (verified Aug 2026); **aggregate record counts unverified** | The most explicitly *federated* national model in Asia: a formal alliance with named institutional members rather than a single owner. Directly relevant if the atlas wants to aggregate from multiple Tamil Nadu institutions without owning their data |
| [Catalogue of Life in Taiwan (TaiCOL)](https://taicol.tw/) | National | Funded by the Forest Bureau; **maintained by TaiBIF** | Taxonomy with synonyms, **vernacular names**, and status facets: Taiwan-endemic, nationally protected, national red-list level, **and IUCN category as a separate field** | Taxonomy / conservation status | Active (verified Aug 2026); **taxon totals could not be read from the statistics page** | **Its status model is the key takeaway: national red-list level and IUCN category are stored as separate, simultaneously-displayed fields rather than reconciled into one.** That is precisely the "show disputed figures side by side" pattern the user wants — already in production. Published to GBIF as the [national checklist dataset](https://www.gbif.org/dataset/1ec61203-14fa-4fbd-8ee5-a4a80257b45a) |
| [TaiBIF](https://portal.taibif.tw/en/) | National | Institution (GBIF national node) | Occurrence data products, tools, training | Multi-taxon | Active (verified Aug 2026); **counts unverified** | Runs an [IPT instance](https://ipt.taibif.tw/) — a lightweight, standard route to making a small atlas' data citable and harvestable |
| [Lyudao (Green Island) LTSER](https://ltsertwlyudao.org/) | **Single site** | Small academic team; **names unverified** | **Multi-domain by design: ecological *and social* datasets** for one island — coral reef ecosystems plus sustainable-development data | Marine / social-ecological | Active (verified Aug 2026); **dataset counts unverified** | **The single best "one small place, many kinds of data" precedent in Asia.** A long-term social-ecological research site that deliberately holds ecological and social data in one place for one island — structurally the same ambition as a single-reserve atlas |

---

### Japan

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Japan Initiative for Biodiversity Information (JBIF)](https://gbif.jp/en/) | National | Consortium (GBIF national node) with [named collaborating organisations](https://gbif.jp/en/about/jbif/related/) | Occurrence aggregation + [data statistics](https://gbif.jp/en/datause/stat/) + IPT hosting | Multi-taxon | Active (verified Aug 2026) | Documented in a [BISS paper](https://biss.pensoft.net/article_preview.php?id=111893) on how Japan collects and publishes biodiversity information — a rare published account of national-node mechanics |
| [J-IBIS / Biodiversity Center of Japan](https://www.biodic.go.jp/) | National | Government (Ministry of the Environment) | Multi-domain: vegetation surveys, National Survey on the Natural Environment series, [Monitoring Sites 1000](https://www.biodic.go.jp/moni1000/), [CHM metadata search](https://www.biodic.go.jp/chm/) | Vegetation / multi-taxon / monitoring | Active (verified Aug 2026) | **Monitoring Sites 1000** is the most relevant component: ~1,000 fixed long-term monitoring sites nationwide — a national framework of *place-anchored* repeat observation. The 2nd/3rd National Survey vegetation data is [published to GBIF](https://www.gbif.org/dataset/d4cca499-31be-445d-a16a-b927a035767f), giving a long time-depth comparison against later surveys |
| [BISMaL](https://www.godac.jamstec.go.jp/bismal/j/) · [J-OBIS](https://www.godac.jamstec.go.jp/j-obis/j/) | National (marine) | Institution: JAMSTEC | Marine species occurrence, biogeography | Marine | Active (verified Aug 2026); **counts unverified** | Japan's marine equivalents of IndOBIS |
| [Fish Photo Database (fishpix)](https://fishpix.kahaku.go.jp/fishimage/) | National, single group | Institution: National Museum of Nature and Science | Specimen and ecological images keyed to taxa | Fish | Live (verified Aug 2026); **counts unverified** | Long-running museum-run image reference collection |
| [JaLTER](http://www.jalter.org/) · [JaLTER Metacat](https://db.cger.nies.go.jp/JaLTER/) | Network of single sites | Academic network | Site-based long-term ecological datasets with metadata catalogue | Ecosystem monitoring | Active (verified Aug 2026) | Site-scoped dataset catalogue model — relevant to publishing a single reserve's datasets discoverably |
| [Invasive Species Database (NIES)](https://www.nies.go.jp/biodiversity/invasive/) | National | Institution: National Institute for Environmental Studies | **~500 alien species** with profiles | Invasive species | Live (verified Aug 2026) | — |
| [Phytosociological Relevé Database (FFPRI)](https://www.ffpri.go.jp/labs/prdb/) · [Grassland Vegetation Facts DB (NARO)](https://www.naro.go.jp/laboratory/nilgs/vegetation/index.html) · [Database of Japanese semi-natural grassland flora](https://gbif.jp/ipt/resource?r=jsngf) | National | Institutional research labs | Vegetation plot / relevé data | Botanical / vegetation | Live (verified Aug 2026) | **Vegetation plot databases are the most under-replicated category in South Asia** — Japan has three and India has no public equivalent found. A relevé layer would be a genuine differentiator for a forest atlas |
| [Fukui Prefecture Green Data Bank](https://fncc.pref.fukui.lg.jp/fukuinature/greendatabank) | Single prefecture | Prefectural government | Distribution data for one prefecture | Botanical / multi-taxon | Live (verified Aug 2026) | A rare **sub-national government** biodiversity data bank — the administrative analogue of a state/district atlas |
| [ESABII data portal](https://www.esabii.biodic.go.jp/) | Transnational (East & Southeast Asia) | Intergovernmental initiative hosted by Japan's Biodiversity Center | Species pages across the East/SE Asian region | Multi-taxon | Live (verified Aug 2026); **update cadence unverified** | Regional aggregation layer above national nodes; useful for cross-border species like the Asian elephant |

---

### South Korea

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Species Korea / National Institute of Biological Resources](https://species.nibr.go.kr/) | National | Government institution (NIBR) | National species list, species pages | Multi-taxon | Live (verified Aug 2026); **counts and last-update unverified** | Underpins the [Korean National Species List](https://en.wikipedia.org/wiki/Korean_National_Species_List). A [2023 *Scientific Data* paper](https://www.nature.com/articles/s41597-022-01924-z) shows large national herpetofauna datasets being released from this ecosystem — evidence of active data flow |

No Korean equivalent of a single-reserve multi-domain atlas was located.

---

### Southeast Asia

The most efficient discovery surface for the region is the [ASEAN Biodiversity Knowledge Platform resource index](https://km.aseanbiodiversity.org/), which enumerates national systems country by country.

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Malaysia Biodiversity Information System (MyBIS)](https://mybis.gov.my/) | National | Government consortium: Ministry of Natural Resources and Environmental Sustainability (NRES) → **Malaysia Biodiversity Centre (MBC)** with **Forest Research Institute Malaysia (FRIM)** | **Strongly multi-domain**: species records, **specimen collections**, photo galleries, **protected-area information and maps**, **literature and references**, **expert directory**, interactive education, open datasets, **timeline analyses** | Multi-taxon national | Active — articles dated **2 October 2024**; portal live (verified Aug 2026); **record counts not published** | **Closest national-scale match to the user's multi-domain ambition.** Two features are unusual and directly worth copying: an **expert directory** (who to ask about what, per taxon/place) and **timeline analyses** (change over time as a first-class view rather than a chart). No conflicting-figures mechanism found |
| [Sarawak Biodiversity Centre](https://www.sbc.org.my/) | Sub-national (Sarawak) | Statutory authority; divisions include a dedicated **Traditional Knowledge Documentation** department | Regulatory (access to biological resources) + **[SORAS](https://www.sbc.org.my/) research-application system** + ethnobiological/indigenous knowledge documentation | Tribal / traditional knowledge / regulatory | Active (verified Aug 2026); **no public-facing TK database found; holdings unverified** | The only Asian body found with **traditional knowledge documentation as a named institutional department with a legal access-and-benefit-sharing mandate**. Critical precedent for the ethical architecture of a tribal-knowledge layer: documented, but deliberately **not** publicly browsable |
| [Biodiversity of Singapore](https://singapore.biodiversity.online/) | Single city-state | Small team, "A Digital Reference Collection for Singapore's Biodiversity"; associated with an "APS / Animals Plants Singapore" identity; **named individuals not published** | Species pages with images across insects, plants, vertebrates, crustaceans, molluscs, fungi and more | Multi-taxon | Active — **14,978 species** in the collection (verified Aug 2026); **no last-updated date published** | **Outstanding scale-to-area ratio: ~15,000 species documented for a 730 km² territory** by what appears to be a small team. The strongest available evidence that exhaustive small-area documentation is achievable |
| [Flora and Fauna Web (NParks)](https://www.nparks.gov.sg/florafaunaweb) | Single city-state | Government agency (National Parks Board) | Species pages spanning **wild and cultivated** plants plus animals; conservation status; field guides; comparative ID tools | Botanical / multi-taxon | **Active — page states "Last updated 12 August 2026"** (verified Aug 2026) | The clearest last-updated stamp of any Asian portal examined. Notable scope choice: cultivated and horticultural taxa are treated as first-class alongside wild ones — relevant where plantation and agroforestry species matter, as in the STR landscape |
| [Singapore Biodiversity Records (LKCNHM)](https://lkcnhm.nus.edu.sg/publications/sg-biodiversity-records/volumes/) | Single city-state | Institution: Lee Kong Chian Natural History Museum, NUS | A **citable serial publication** for individual sighting records, ISSN 2345-7597 (online) | Multi-taxon | Active serial (verified Aug 2026) | **The most important structural idea in this whole survey for a small atlas:** rather than a database of unattributed points, individual records are published as short, citable, dated notes in a numbered serial. Every observation becomes a permanent, attributable, disputable document. Enabled the [PNAS analysis of two centuries of Singapore biodiversity discovery and loss](https://www.pnas.org/doi/10.1073/pnas.2309034120) |
| [Thailand Biodiversity Information Facility (TH-BIF)](https://km.aseanbiodiversity.org/resources/thailand-biodiversity-information-facility-th-bif.49/) | National | Government (BEDO / national node) | Species occurrence and checklists | Multi-taxon | **Status unverified** — indexed by ASEAN; direct fetch failed on DNS resolution (Aug 2026). Possible URL change or outage | Flag for follow-up. GBIF-side Thai activity is live (e.g. [bee digitisation project](https://www.gbif.org/project/1tRUhXbB0fXAxKywxgIR4T/digitizing-and-databasing-of-bee-specimens-in-thailand)), so the country's data flow continues even if the portal is unreachable |
| [INDOBIOSYS](http://www.indobiosys.org/) | National (Indonesia) | Bilateral academic consortium (Indonesian–German; DAAD-supported) | Biodiversity discovery + information system; specimen/molecular pipeline | Multi-taxon | Live (verified Aug 2026); **recency and counts unverified** | Discovery-oriented rather than place-documentation oriented |
| [Co's Digital Flora of the Philippines](https://www.philippineplants.org/) | National | **Three named editors — a genuinely small team**: **P. B. Pelser, J. F. Barcelona & D. L. Nickrent**; continues the work of **Elmer D. Merrill** and **Leonardo L. Co** | **~10,200 plant species**, illustrated with **>146,000 photos covering ~59% of the flora**; checklist with synonyms, distribution, conservation status, literature references | Botanical | Active — "2011 onwards" per its own citation format (verified Aug 2026); **explicit last-update date not published** | **The premier small-team-comprehensive exemplar in Asia.** Three editors, a national flora, ~146,000 images, funded partly by the Rufford Foundation, documented in the [*Philippine Journal of Science*](https://philjournalsci.dost.gov.ph/cos-digital-flora-of-the-philippines-plant-identification-and-conservation-through-cybertaxonomy/) and adopted as a [World Flora Online regional flora](https://about.worldfloraonline.org/floras/cos-digital-flora-of-the-philippines). It is explicitly named for a deceased predecessor — a model for building on an individual's lifetime of prior documentation |
| Cebu Biodiversity Information System (CebuBioSys) · MABIDA (Marine Biodiversity Database) · CDFP (Checklist of native, naturalized and invasive vascular plants) — all indexed at [ASEAN BKP](https://km.aseanbiodiversity.org/) | Provincial (Cebu) / national | **Unverified** | Occurrence and checklist | Multi-taxon / marine / botanical | **URLs and status unverified** (Aug 2026) | **CebuBioSys is a high-priority follow-up**: a *province-scoped* biodiversity information system is the closest administrative analogue to a district/reserve atlas found in Southeast Asia |
| [Sinh vật rừng Việt Nam / Vietnam Forest Creatures](https://www.vncreatures.net/) | National | **Effectively solo/small, non-institutional** — public contact is an individual mailbox (`(removed)`) via Đồng Nai forest protection | Multi-domain: **>2,000 species** across insects, animals, plants and fungi; image galleries; **scientific reports and regulatory documents**; **national park pages** | Multi-taxon / forest / legal | **Active — new species entries dated 17 August 2026** (three conifer entries added); guidance articles dated 17 March 2025 (verified Aug 2026) | **The single most encouraging find for a solo builder.** One person has sustained a national multi-taxon encyclopedia for well over a decade that *also* carries regulatory documents and protected-area pages — the same species + legal + place combination the user wants. Caveat: the site serves `noindex, nofollow`, so it is invisible to search engines — a real lesson about discoverability |
| Flora of Myanmar Database · Myanmar Biodiversity Zone — indexed at [ASEAN BKP](https://km.aseanbiodiversity.org/) | National | **Unverified** | Flora checklist / biodiversity information | Botanical | **URLs and status unverified** (Aug 2026) | Myanmar is the largest verification gap in this survey |
| Brunei, Cambodia, Lao PDR national Clearing House Mechanisms — indexed at [ASEAN BKP](https://km.aseanbiodiversity.org/) | National | Government | Legal instruments, national reports, biodiversity facts | Legal / policy | Indexed; **individual status unverified** (Aug 2026) | CHM sites are the reliable minimum for legal-instrument coverage in every CBD party — worth using as the baseline legal source for any Asian country |
| [ASEAN Biodiversity Dashboard](https://dashboard.aseanbiodiversity.org/) | Transnational | Intergovernmental (ASEAN Centre for Biodiversity) | Indicators by country | Multi-taxon / indicators | Live (verified Aug 2026) | Indicator-level, not record-level; relevant only as a cross-country comparator |

---

### Central Asia, Mongolia and Russia-Asian

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Plantarium](https://www.plantarium.ru/lang/en.html) | Transnational (Russia and the former USSR, plus neighbours) | **Effectively solo-built with a large volunteer base**: designed and programmed by **Dmitry Gennadievich Oreshkin**; open to "botanists and photographers, professionals and amateurs" | Multi-domain: species atlas + illustrated identification guide + images + **regional organisation by both administrative and physiographic regions** + explicit taxonomic source authorities (Czerepanov for vascular plants, separate moss, liverwort/hornwort and lichen floras) | Botanical / lichen | Active — copyright span **2007–2026** (verified Aug 2026); **species and image totals not published on the English landing page** | **The most important solo-builder precedent in this entire survey.** One named programmer has run a continent-scale, multi-kingdom, community-contributed atlas for nearly two decades. Two design details worth stealing: (1) it names the *authority source* for each taxonomic group rather than presenting one merged tree, and (2) it lets users browse by physiographic region as well as administrative region — exactly the dual geography a tiger reserve needs (reserve boundary vs. watershed vs. district) |
| [Moscow Digital Herbarium](https://www.gbif.org/dataset/902c8fe7-8f38-45b0-854e-c324fed36303) | National | Institutional **consortium since 2019**, led from Moscow State University (MW herbarium) | Digitised specimen records | Herbarium | Active as a GBIF-publishing dataset (verified Aug 2026); **counts unverified** | Documented in a [2020 paper](https://www.researchgate.net/publication/341188646_Moscow_Digital_Herbarium_a_consortium_since_2019) — a clean case study of small herbaria federating rather than each building a portal |
| ["Flora of Russia" on iNaturalist](https://www.researchgate.net/publication/345983589_Flora_of_Russia_on_iNaturalist_a_dataset) | National | Community project on borrowed infrastructure | Occurrence-only | Botanical | Dataset published (verified via paper, Aug 2026); **current volume unverified** | The "don't build a portal, build a project on someone else's platform, then publish a data paper" strategy — the cheapest credible route for a small team |
| [Global Snow Leopard & Ecosystem Protection Program (GSLEP)](https://globalsnowleopard.org/) | Transnational (12 range countries) | Intergovernmental programme secretariat | Programme, policy and range-country reporting | Snow leopard | Live (verified Aug 2026); **no unified occurrence database found** | Despite two decades of effort, **no public pan-range snow leopard occurrence database was found** — range-country data stays national. Relevant context: [India's Project Snow Leopard document](https://moef.gov.in/uploads/2018/03/Project-Snow-Leopard-2008.pdf) (2008) and an [*Oryx* interview-based occupancy study](https://www.cambridge.org/core/journals/oryx/article/assessing-changes-in-distribution-of-the-endangered-snow-leopard-panthera-uncia-and-its-wild-prey-over-2-decades-in-the-indian-himalaya-through-interviewbased-occupancy-surveys/BC53A59280520EA73D20F6E2DCC61727) covering two decades in the Indian Himalaya |
| [Central Asian Mammals and Climate Adaptation (CAMCA)](https://camcaproject.org/) | Transnational (Central Asia) | Academic project consortium; **membership unverified** | Species accounts with climate-adaptation framing | Mammal | Live (verified Aug 2026); **recency unverified** | One of very few Central Asia-wide species resources located |

No national biodiversity occurrence portal was found for Kazakhstan, Kyrgyzstan, Uzbekistan, Tajikistan, Turkmenistan or Mongolia. Central Asian species knowledge appears to reach the public mainly through **journal-published checklists** — e.g. the [first checklist of alien vascular plants of Kyrgyzstan](https://bdj.pensoft.net/article/145624/) in *Biodiversity Data Journal* and an [aquatic plant inventory](https://www.sciencedirect.com/science/article/pii/S2287884X23001048) — and through the CEPF [Mountains of Central Asia hotspot](https://www.cepf.net/our-work/biodiversity-hotspots/mountains-central-asia/species) profile. **This is the largest true white space in Asia.**

---

### West Asia / Middle East

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [BioGIS — Israel Biodiversity Information System](https://biogis.huji.ac.il/) | National | Academic, Hebrew University of Jerusalem; **named team not published on the landing page** | Occurrence + **spatial analysis tools**: filter/analyse/calculate search results; analyse species diversity for pre-defined regions, uploaded shapefiles, or **custom user-drawn polygons** | Multi-taxon | Live (verified Aug 2026); **record counts, launch year and last-update not published on the landing page** — all **unverified**. Publishes to GBIF as a [publisher](https://www.gbif.org/publisher/67268f0e-3401-4be5-847b-3752cba6e71c) | **The best analytical UX found in Asia for a place-based atlas: draw an arbitrary polygon, get a diversity analysis.** For a tiger reserve with contested and shifting boundaries (core/buffer/ESZ), arbitrary-polygon querying is arguably the single highest-value feature to copy. Announced in *Science* as far back as [2001](https://www.science.org/doi/10.1126/science.291.5504.559d) — over two decades of continuity |
| [Israel Center for Citizen Science database](https://citizen-science.smnh.tau.ac.il/database/?lang=en) | National | Institution: Steinhardt Museum of Natural History, Tel Aviv University | Citizen-science observation database across projects | Multi-taxon | Live (verified Aug 2026); **counts unverified** | Museum-hosted citizen science with a single shared database across projects — a tidier architecture than India's project-per-site sprawl |
| TÜBİVES (Turkish Plants Data Service) | National | Academic; associated with **Emel Uslu** and colleagues per an ["Updates and improvements of TÜBİVES"](https://independent.academia.edu/EmelUslu) record | Flora of Turkey checklist and distribution data | Botanical | **Status unverified** — no reachable canonical URL confirmed (Aug 2026) | Turkey's alien-flora work is better documented ([checklist and ecological attributes](https://www.researchgate.net/publication/317318234_Alien_flora_of_Turkey_Checklist_taxonomic_composition_and_ecological_attributes); [first alien plants database](https://www.researchgate.net/publication/330349572_The_First_Alien_Plants_Database_of_Turkey)) than the general flora portal |

No national multi-domain biodiversity atlas was located for Iran, Iraq, Saudi Arabia, the UAE, Oman, Jordan, Lebanon or Yemen within the search budget. **Treat this as unsearched rather than absent** — see unverified leads.

---

#### Dead / dormant projects found

| Project | Evidence of death or dormancy | Why it matters |
|---|---|---|
| **[Biodiversity of India / Project Brahma](https://www.biodiversityofindia.org/)** | **Definitively dead.** Landing page footer reads *"This page was last modified on 8 October 2011, at 21:41."* Content frozen at **664 species** and **65 "Bio Notes"** articles; "Image of the Week" described in the past tense; divisions still labelled "(beta)". Coordinator named in bylines as **Gaurav Moghe**. Built on [Semantic MediaWiki](https://www.semantic-mediawiki.org/wiki/Site:Biodiversity_of_India) | **The most instructive failure in this survey.** An ambitious, well-designed, semantically-structured Indian biodiversity wiki that died within ~2 years and has been serving a stale page for **15 years**. The lesson is not about technology — Semantic MediaWiki was a good choice — it is that a volunteer wiki with no institutional host and no ongoing data inflow stops. Also note the site still ranks in search results, actively misinforming |
| **[Oriental Bird Images](https://www.orientalbirdclub.org/oriental-bird-images)** | **Deliberately closed and migrated.** Oriental Bird Club [announced the closure on 17 March 2021](https://www.orientalbirdclub.org/club-news/2021/3/17/oriental-bird-images-website-is-closing); the [Macaulay Library announced the new home on 22 October 2021](https://www.macaulaylibrary.org/2021/10/22/the-oriental-bird-club-image-database-has-a-new-home-at-the-macaulay-library/) | **The best-executed shutdown found.** A two-decade Asia-wide photographic archive wound down by transferring its collection to a larger institution with a stable identifier scheme, rather than going dark. This is the exit plan a solo-built atlas should design for from day one |
| **[Flora of Pakistan on eFloras](http://www.efloras.org/flora_page.aspx?flora_id=5)** | **Self-declared dead.** The page states it "is a legacy site which is now out of date" and points users to tropicos.org | Honest staleness labelling. Rare and worth imitating: a banner that tells the reader the data is stale is far better than a page that looks current |
| **[BIOTIK](http://www.biotik.org/)** | **Infrastructure decay.** TLS certificate is invalid for the host (hostname mismatch, verified Aug 2026) — the certificate has not been maintained. An EU-funded Western Ghats tree-identification project involving the French Institute of Pondicherry | Directly relevant to the user's own landscape. A funded Western Ghats identification resource whose hosting is now unmaintained; its content may only be recoverable via the Wayback Machine |
| **[Biological Specimen Collections of Bhutan](http://bhutanbiodiversity.net/)** | **Dormant.** Site copyright **2017**; no later activity indicators. Built on Symbiota, supported through the US Fulbright Program | Grant-cycle mortality: a portal built on a visiting-fellowship timescale, stranded when the fellowship ended. Compounded by the fact that Bhutan's *other* portal (biodiversity.bt) is thriving — the duplicate now generates conflicting species counts for one small country |
| **[India Observatory / IBIS](https://www.indiaobservatory.org.in/tool/ibis)** | **Likely dormant.** Copyright **2006–2019**; reptile and amphibian modules still described as "work in progress" seven years on | A large institutional project (FES) that stalled mid-build. Its **500,000+ citation bibliography** may be the most valuable orphaned asset in Indian conservation informatics — worth asking FES about |
| **[ZSI digital archive](https://faunaofindia.nic.in/)** | **Stale but valuable.** Site copyright **2015**; several journal series show final entries in the 1990s | Not dead as a corpus — 200,000+ digitised pages of Conservation Area Series and Fauna of India volumes remain the best historical faunal source for Indian reserves. But nobody is adding to it, so treat it as a fixed historical archive to mine, not a living feed |
| **[Thailand Biodiversity Information Facility (TH-BIF)](https://km.aseanbiodiversity.org/resources/thailand-biodiversity-information-facility-th-bif.49/)** | **Possible outage or URL change.** The `thbif.in.th` host failed DNS resolution (Aug 2026) while still being indexed as Thailand's national facility by the ASEAN Biodiversity Knowledge Platform | Textbook "the aggregator still lists it, the site no longer resolves" — the exact pattern to look for when hunting dead portals |
| **MigrantWatch (India)** | **Believed superseded** by eBird India, but **not verified.** It remains listed as a project in the [Citizen Science India registry](https://citsci-india.org/projects/project/migrantwatch/) while no independent live site was confirmed | Flag rather than a conclusion — but if confirmed, another case of an aggregator carrying a dead entry |
| **Biodiversity of Pakistan: Database and Global Networking (BGN)** | **Paper without a portal.** Described in a [2022 publication](https://www.researchgate.net/publication/366191883_BIODIVERSITY_OF_PAKISTAN_DATABASE_AND_GLOBAL_NETWORKING_BGN); no live URL found | The purest form of the pattern worth cataloguing: the literature asserts a portal exists; the web does not corroborate it |

---

#### Leads I could not verify

These appear to exist but could not be confirmed within this session's search budget. Each is a concrete follow-up, not a finding.

1. **Goa Bird Atlas attribution and publication data.** Multiple news sources report Goa as India's second state to publish a bird atlas, but the [GBCN "About Us"](https://www.gbcn.in/about-us) page does not mention the atlas, and no dedicated atlas site was found. Publication year, grid design, species total and lead names are all **unverified**.
2. **CebuBioSys (Cebu Biodiversity Information System, Philippines).** Indexed by the ASEAN Biodiversity Knowledge Platform; no URL resolved. **Highest-value single follow-up** — a province-scoped multi-taxon system is the nearest administrative analogue to a district/reserve atlas.
3. **Flora of Myanmar Database and Myanmar Biodiversity Zone.** Indexed by ASEAN; URLs and status unknown.
4. **TH-BIF (Thailand).** Whether the portal has moved, been rebranded, or been discontinued.
5. **Biome (Singapore Biodiversity and Environment Database System).** Listed by ASEAN as a Singapore national system; its relationship to Flora & Fauna Web and to Biodiversity of Singapore is unclear — possibly three overlapping Singapore systems.
6. **MABIDA and CDFP (Philippines).** Marine biodiversity database and vascular plant checklist; URLs unconfirmed.
7. **TÜBİVES (Turkey).** No canonical URL confirmed; only an academic record of "updates and improvements" associated with **Emel Uslu**.
8. **The entire Gulf / Levant / Iran region.** No searches completed for Iran, Iraq, Saudi Arabia, UAE, Oman, Qatar, Kuwait, Bahrain, Jordan, Lebanon, Syria or Yemen. **Absence here reflects my search budget, not the absence of projects.** Iran in particular has substantial published floristic work that very likely has a portal.
9. **Central Asia and Mongolia national portals.** Nothing found for Kazakhstan, Kyrgyzstan, Uzbekistan, Tajikistan, Turkmenistan or Mongolia, but coverage was thin. Note that **Kyrgyzstan does publish to GBIF** (e.g. a [GRIIS dataset](https://www.gbif.org/dataset/d893c53b-a6b3-4e43-829f-fcd0cbb9bad3)), implying institutional capacity that may have an unfound national front end.
10. **Nepal Bird Atlas.** No systematic grid-based Nepali bird atlas comparable to Kerala's was found. Given Nepal's strong ornithological tradition (Inskipp lineage, Himalayan Nature), one may exist or be in progress.
11. **Wildlife Institute of India "National Wildlife Database."** The [WII page](https://wii.gov.in/national_wildlife_database) that search engines associate with this name does not, when opened, describe a national wildlife database — only labs and repositories. Whether the NWD is defunct, renamed, or simply internal is **unresolved.** For a tiger-reserve atlas this is a significant unknown, since WII holds the All-India Tiger Estimation data.
12. **TIGERNET record counts and update cadence.** The site blocks automated fetching, so mortality-record totals and freshness are **unverified** despite the site being live.
13. **Kerala Bird Atlas raw volume.** Sub-cell count (4,324) and area (38,863 km²) are confirmed, but total checklists, total records and total species for Atlas 1.0 were **not** confirmed on the project page.
14. **Assam Biodiversity Portal and The Western Ghats Portal activity levels.** Both returned HTTP 403 to automated fetching. Both are live, but observation counts and most-recent-observation dates are **unverified** — which matters, because a state/landscape sub-portal that is live-but-empty would be an important negative finding.
15. **Record counts for Butterflies of India and the wider Biodiversity Atlas India network.** The site displays live counters for species pages, observations, images, contributors, checklists and host plants, but the values could not be read (the counters render client-side; `ifoundbutterflies.org` also returned robots-disallowed). Only the ">125,000 reference images" figure is confirmed.
16. **Sahapedia's actual scale.** Its "About" page publishes no article count, module count or team size — unusual for a project of its visibility.
17. **South Asia Open Archives item count.** JSTOR Daily reports it passed one million pages; the current figure is **unverified**.
18. **FLAME University Gazetteer / Districts Project outputs.** A July 2021 concept note exists; no published district volumes were found. May be dormant.
19. **Keystone Archives platform.** Strongly appears to be **Omeka** from URL structure and interface, but this is **not stated on the site** — inferred, not verified. Worth confirming directly, since it is the closest analogue project and its stack choice is informative.
20. **NSII (China) and Catalogue of Life China totals.** Both hosts have broken or misconfigured TLS, preventing verification of specimen counts, checklist year and taxon totals.
21. **The Chinese solo-taxon cluster.** shellsmap, GobiologyCN, china-odonata.top, ants-china.com, insectaintegration.com, mycopedia.top, liudayadan.com and Dongniao were located via a community link list but **none was independently verified for recency, authorship or scale.** This is the richest unexplored seam for small/solo project research in Asia and deserves a dedicated pass.
22. **Whether any Asian project actually implements side-by-side conflicting figures.** The closest verified instances are structural rather than explicit: **TaiCOL** storing national red-list level and IUCN category as separate simultaneous fields; **Himalayan Nature's** National Red List of Birds presenting global *and* national status per species; **AmphibiaChina's** annual taxonomic-change log; and **Land Conflict Watch's** two-source verification rule. **No Asian project was found that presents a disputed figure as a first-class object with competing sources attached.** On the evidence gathered, that feature would be novel in this region.

---

#### Sources

Aggregators and discovery surfaces
- [Citizen Science India — project registry](https://citsci-india.org/projects/)
- [Useful websites about China's biodiversity (iNaturalist)](https://www.inaturalist.org/posts/109663-useful-websites-about-china-s-biodiversity)
- [ASEAN Biodiversity Knowledge Platform — resources](https://km.aseanbiodiversity.org/)
- [JBIF — biodiversity-related sites list (Japan)](https://gbif.jp/services/useful/)
- [TBIA — resources and links (Taiwan)](https://tbiadata.tw/en-us/resources/link)
- [Bird Count India — homepage](https://birdcount.in/) · [Bird Atlases in India (PDF, 2024)](https://birdcount.in/wp-content/uploads/2024/07/Bird-Atlases-in-India-1.pdf)
- [ASEAN Biodiversity Dashboard](https://dashboard.aseanbiodiversity.org/)
- [ESABII data portal](https://www.esabii.biodic.go.jp/)

India — national
- [Biodiversity Atlas – India](https://www.bioatlasindia.org/) · [constituent websites](https://www.bioatlasindia.org/bai-websites) · [citizen science](https://www.bioatlasindia.org/citizen-science) · [registry entry](https://citsci-india.org/projects/project/biodiversity-atlas-india/)
- [Butterflies of India](https://www.ifoundbutterflies.org/)
- [Zoological Survey of India digital archive](https://faunaofindia.nic.in/)
- [India Flora Online / Herbarium JCB](https://indiaflora-ces.iisc.ac.in/) · [research team](https://indiaflora-ces.iisc.ac.in/people.php) · [Digital Flora of Karnataka](https://indiaflora-ces.iisc.ac.in/FloraKarnataka/welcome.php) · [IISc launch announcement](https://iisc.ac.in/events/iisc-launches-online-database-of-plants-in-peninsular-india/)
- [eFlora of India](https://efloraofindia.com/efi/india-flora-online/)
- [Flowers of India](https://www.flowersofindia.net/) · [credits](https://www.flowersofindia.net/misc/credits.html)
- [ENVIS Centre on Medicinal Plants (FRLHT/TDU)](https://envis.frlht.org/) · [about](https://envis.frlht.org/envis.htm) · [traded medicinal plants database](http://envis.frlht.org/traded-medicinal-plants-database.php)
- [Indian Biodiversity Information System](https://www.indianbiodiversity.org/) · [India Observatory IBIS tool page](https://www.indiaobservatory.org.in/tool/ibis)
- [TIGERNET](https://tigernet.nic.in/) · [NTCA tiger mortality](https://ntca.gov.in/tiger-mortality/) · [TRAFFIC launch report](http://www.traffic.org/home/2010/1/7/india-launches-tigernet.html)
- [M-STrIPES](https://www.mstripes.in/) · [NTCA M-STrIPES page](https://ntca.gov.in/a/m-stripes/)
- [Water Source Atlas of Tiger Reserves (NTCA, PDF)](https://ntca.gov.in/assets/uploads/Reports/Others/Water_source_atlas_small.pdf)
- [Elephant Reserves of India: An Atlas (MoEFCC, PDF)](https://moef.gov.in/uploads/2023/11/PE-Elephant-Reserve-of-India-an-atlas.pdf)
- [PARIVESH environment clearance](https://environmentclearance.nic.in/) · [EIA notifications](https://environmentclearance.nic.in/report/EIA_Notifications.aspx) · [forest clearances](https://forestsclearance.nic.in/homenew.aspx) · [PARIVESH 2.0 open data](https://www.data.gov.in/catalog/environmental-clearance-granted-parivesh-20) · [environment & forest sector datasets](https://www.data.gov.in/sector/environment-and-forest)
- [India Environment Portal](https://www.indiaenvironmentportal.org.in/)
- [Forest Rights Act portal (Vasundhara)](https://www.fra.org.in/) · [MoTA FRA page](https://tribal.nic.in/fra.aspx)
- [Land Conflict Watch](https://www.landconflictwatch.org/) · [about](https://www.landconflictwatch.org/about) · [conflicts database](https://www.landconflictwatch.org/all-conflicts) · [methodology essay, Datajournalism.com](https://datajournalism.com/read/handbook/two/assembling-data/documenting-land-conflicts-across-india) · [RRI profile](https://rightsandresources.org/blog/indias-land-conflict-watch/)
- [Wetlands of India portal](https://indianwetlands.in/) · [NWIA page](https://indianwetlands.in/resources-and-e-learning/national-wetland-inventory-assessment/) · [NWIA atlas on VEDAS](https://vedas.sac.gov.in/vcms/en/National_Wetland_Inventory_and_Assessment_(NWIA)_Atlas.html) · [Coral Reef Atlas on VEDAS](https://vedas.sac.gov.in/en/Coral_Reef_Atlas.html)
- [IndOBIS](https://indobis.in/) · [CMLRE IndOBIS page](https://www.cmlre.gov.in/mlr-programs/indobis) · [OBIS node spotlight, April 2026](https://obis.org/2026/04/29/node-spotlight-indobis/)
- [Hornbill Watch (NCF)](https://www.ncf-india.org/eastern-himalaya/hornbill-watch) · [GBIF dataset](https://www.gbif.org/dataset/de3da1ee-6946-4d54-97a0-779599721359) · [Indian BIRDS data paper (PDF)](https://indianbirds.in/pdfs/IB_14_3_DattaETAL_HornbillWatch.pdf)
- [Traditional Knowledge Digital Library (CSIR)](https://www.csir.res.in/en/documents/tkdl) · [WIPO article](https://www.wipo.int/en/web/wipo-magazine/articles/protecting-indias-traditional-knowledge-37721) · [critical analysis, Third World Quarterly](https://www.tandfonline.com/doi/full/10.1080/01436597.2021.2019009)
- [People's Biodiversity Register (NBA)](http://www.nbaindia.org/content/105/30/1/pbr.html) · [Mongabay India explainer](https://india.mongabay.com/2024/07/explainer-what-is-a-peoples-biodiversity-register/) · [Assam PBR](https://environmentandforest.assam.gov.in/frontimpotentdata/people%E2%80%99s-biodiversity-register-0) · [Meghalaya PBR](https://megbiodiversity.nic.in/peoples-biodiversity-register) · [Haryana PBR](https://sbb.haryanaforest.gov.in/project/peoples-biodiversity-register-pbr/)
- [Sahapedia](http://www.sahapedia.org/) · [about](http://www.sahapedia.org/about-us)
- [Wildlife Institute of India — national wildlife database page](https://wii.gov.in/national_wildlife_database)
- [mahoot — Asian Elephant Database](https://mahoot.xyz/)
- [Wild Atlas](https://www.wildatlas.in/)
- [Project Brahma / Biodiversity of India wiki](https://www.biodiversityofindia.org/index.php?title=Biodiversity_of_India:A_Wiki_Resource_For_Indian_Biodiversity) · [Semantic MediaWiki site record](https://www.semantic-mediawiki.org/wiki/Site:Biodiversity_of_India)
- [Mongabay India — digital biodiversity platforms](https://india.mongabay.com/2017/12/enthusiasts-and-experts-document-biodiversity-on-a-digital-platform/)

India — gazetteers and historical documents
- [Imperial Gazetteer of India, DSAL](https://dsal.uchicago.edu/reference/gazetteer/) · [table of contents](https://dsal.uchicago.edu/reference/gazetteer/toc.html) · [1909 atlas](https://dsal.uchicago.edu/reference/gaz_atlas_1909/) · [1931 atlas](https://dsal.uchicago.edu/reference/gaz_atlas_1931/) · [maps, vols 1–24](https://dsal.uchicago.edu/maps/gazetteer/index.html) · [DSAL reference resources](https://dsal.uchicago.edu/reference/)
- [South Asia Open Archives](https://southasiaoa.org/) · [on JSTOR](https://www.jstor.org/site/south-asia-open-archives/saoa/) · [JSTOR Daily on reaching a million pages](https://daily.jstor.org/south-asia-open-archives-hits-a-million)
- [FLAME University Gazetteer Project](https://www.flame.edu.in/cka/the-gazetteer-project.php) · [Districts Project concept note (PDF)](https://www.flame.edu.in/cka/pdfs/Concept-Note.pdf) · [The Print on gazetteer revival](https://theprint.in/ground-reports/india-is-reviving-gazetteers-minus-the-colonial-lens-moradabad-is-leading-the-way/2815525/)
- [Tamil Nadu District Gazetteers (Wikipedia)](https://en.wikipedia.org/wiki/Tamil_Nadu_District_Gazetteers) · [District Gazetteer (Wikipedia)](https://en.wikipedia.org/wiki/District_Gazetteer) · [FIBIwiki gazetteers guide](https://wiki.fibis.org/w/Gazetteers) · [Gazetteers of British India 1833–1962 (Brill)](https://brill.com/display/package/9789004197916?language=en)
- [IndiaOpenAtlas (GitHub)](https://github.com/justinelliotmeyers/IndiaOpenAtlas)

India — Tamil Nadu, Kerala, Karnataka, Western Ghats
- [Moths of Krishnagiri (KANS)](https://citsci-india.org/projects/project/moths-of-krishnagiri/) · [Biodiversity of Krishnagiri](https://citsci-india.org/projects/project/biodiversity-of-krishnagiri/)
- [Coimbatore City Bird Atlas](https://birdcount.in/coimbatore-city-bird-atlas/)
- [Bird Gap Filling in Tamil Nadu](https://citsci-india.org/projects/project/bird-gap-filling-in-tamil-nadu/) · [Pongal Bird Count](https://citsci-india.org/projects/project/pongal-bird-count/) · [Biodiversity of Chennai](https://citsci-india.org/projects/project/biodiversity-of-chennai/)
- [Keystone Foundation Archives](https://keystone-archives.org/archive/) · [Keystone Foundation](https://keystone-foundation.org/) · [biodiversity programme](https://keystone-foundation.org/biodiversity/) · [People & Nature Collectives](https://keystone-foundation.org/people-and-nature-collectives/) · [publications](https://keystone-foundation.org/publications/)
- [Nilgiri Documentation Centre](https://nilgiridocumentation.com/) · [Nilgiris page](https://nilgiridocumentation.com/nilgiris.html)
- [Nilgiri Archaeological Project, Ghent University](https://www.nilgiri.ugent.be/) · [project description](https://www.nilgiri.ugent.be/project/)
- [The Nilgiris Foundation](https://thenilgirisfoundation.org/)
- [Kerala Bird Atlas](https://birdcount.in/kerala-bird-atlas/) · [Current Science paper (PDF)](https://www.currentscience.ac.in/Volumes/122/03/0298.pdf) · [atlas PDF](https://birdcount.in/wp-content/uploads/2021/01/Kerala-Bird-Atlas-Final-Compressed.pdf) · [WWF India feature](https://www.wwfindia.org/wwf-india-updates/feature_stories/bird_atlas_kerala/) · [ResearchGate paper](https://www.researchgate.net/publication/371856566_Kerala_Bird_Atlas_2015-20_features_outcomes_and_implications_of_a_citizen-science_project)
- [KFRI websites index](https://www.kfri.res.in/about/kfri-websites.asp) · [Forest Management Information System division](https://www.kfri.res.in/divisions/forest-management-information-system.asp)
- [Mysore Bird Atlas](https://birdcount.in/mysore-bird-atlas/) · [Dryad dataset](https://datadryad.org/dataset/doi:10.5061/dryad.8k3d81r) · [Indian BIRDS paper (PDF)](https://indianbirds.in/pdfs/IB_15_3_ShivaprakashETAL_MysuruCityBirdAtlas.pdf)
- [The Western Ghats Portal](https://thewesternghats.indiabiodiversity.org/) · [French Institute of Pondicherry project page](https://www.ifpindia.org/projects/western-ghats-portal-india-biodiversity-portal/) · [ATREE Western Ghats](https://www.atree.org/focus-geography/western-ghats/) · [Biodiversity of Western Ghats on iNaturalist](https://www.inaturalist.org/projects/biodiversity-of-western-ghats) · [BIOTIK](http://www.biotik.org/) · [CEPF Western Ghats & Sri Lanka ecosystem profile (PDF)](https://d29l0tur8ol1gj.cloudfront.net/sites/default/files/western-ghats-ecosystem-profile-english.pdf)
- [Sathyamangalam Tiger Reserve (Wikipedia)](https://en.wikipedia.org/wiki/Sathyamangalam_Tiger_Reserve)

India — other states
- [Pune Bird Atlas](https://birdcount.in/pune-bird-atlas/) · [Epicollect5 data](https://five.epicollect.net/project/pune-bird-atlas/data) · [Indie Journal coverage](https://www.indiejournal.in/article/pune-bird-atlas-a-people-s-project-to-map-the-city-s-bird-diversity)
- [Hyderabad Bird Atlas](https://birdcount.in/hyderabad-bird-atlas/)
- [Ahmedabad Bird Atlas](https://birdcount.in/ahmedabad-bird-atlas/) · [Bhagalpur Bird Atlas](https://birdcount.in/bhagalpur-bird-atlas/)
- [Goa Bird Conservation Network](https://www.gbcn.in/) · [about](https://www.gbcn.in/about-us) · [Drishti IAS on Goa's atlas](https://www.drishtiias.com/state-pcs-current-affairs/goa-becomes-second-indian-state-to-publish-a-bird-atlas) · [GKToday](https://www.gktoday.in/goa-becomes-second-state-to-publish-bird-atlas/)
- [Assam Biodiversity Portal](https://assambiodiversity.indiabiodiversity.org/) · [document list](https://assambiodiversity.indiabiodiversity.org/document/list) · [Assam State Biodiversity Board](https://asbb.assam.gov.in/information-services/biodiversity-of-assam)
- [Bihu Bird Count](https://citsci-india.org/projects/project/bihu-bird-count/)
- [Marine Life of Mumbai](https://citsci-india.org/projects/project/marine-life-of-mumbai/) · [Charotar Crocodile Count](https://citsci-india.org/projects/project/charotar-crocodile-count/) · [UPL Sarus Conservation Project](https://citsci-india.org/projects/project/upl-sarus-conservation-project/) · [Intertidal Biodiversity of Andhra Pradesh](https://citsci-india.org/projects/project/intertidal-biodiversity-of-andhra-pradesh/) · [Reeflog](https://citsci-india.org/projects/project/reeflog/) · [Lakshadweep CBFM](https://citsci-india.org/projects/project/community-based-fisheries-monitoring-cbfm-in-the-lakshadweep/) · [SeasonWatch](https://citsci-india.org/projects/project/seasonwatch/) · [MigrantWatch](https://citsci-india.org/projects/project/migrantwatch/)
- [Andaman Nicobar Environment Team](https://anetindia.org/) · [research](https://anetindia.org/research/) · [team](https://www.anetindia.org/about/team/) · [the islands](https://anetindia.org/about/the-islands/)

Sri Lanka, Nepal, Bhutan, Bangladesh, Pakistan
- [Sri Lanka Biodiversity Clearing House](https://lk.chm-cbd.net/) · [National Red List 2020 flora (PDF)](https://lk.chm-cbd.net/sites/lk/files/2022-06/The%20National%20Red%20List%202020-The%20Conservation%20Status%20of%20the%20Flora%20of%20Sri%20Lanka.pdf) · [Biodiversity Profile of Sri Lanka (PDF)](https://lk.chm-cbd.net/sites/lk/files/2022-06/Biodiversity_ProfileSriLanka.pdf) · [2007 Red List (IUCN, PDF)](https://portals.iucn.org/library/sites/library/files/documents/RL-548.7-003.pdf)
- [Dilmah Conservation butterfly garden](https://www.dilmahconservation.org/butterfly-garden/) · [e-publications](https://www.dilmahconservation.org/butterfly-garden/e-publications.html) · [Biodiversity of Sri Lanka blog](https://biodiversityofsrilanka.blogspot.com/p/butterfly-diversity-of-sri-lanka.html)
- [Himalayan Nature](https://www.himalayannature.org/) · [National Red List of Nepal's Birds](https://www.himalayannature.org/works/projects/national-red-list-of-nepals-birds/) · [NTNC species](https://ntnc.org.np/thematic-area/species) · [NTNC Biodiversity Conservation Center](https://ntncbcc.org.np/services/species/)
- [Bhutan Biodiversity Portal](https://biodiversity.bt/) · [GBIF dataset](https://www.gbif.org/dataset/926f3dd4-89d9-4cbd-8ed7-32fb6fcfa861) · [Wikipedia](https://en.wikipedia.org/wiki/Bhutan_Biodiversity_Portal) · [Biological Specimen Collections of Bhutan](http://bhutanbiodiversity.net/) · [Android app](https://play.google.com/store/apps/details?id=bt.biodiversity.twa)
- [Marine Biodiversity Portal of Bangladesh](https://marinebiodiversity.org.bd/) · [data paper](https://www.researchgate.net/publication/377965492_Marine_Biodiversity_Portal_of_Bangladesh_marinebiodiversityorgbd_A_smart_online_encyclopedia_of_marine_fauna_and_flora_of_the_country) · [ENGAGE4Sundarbans archival research](https://engage4sundarbans.org/archival-research/) · [Sundarbans Atlas (NHBS)](https://www.nhbs.com/sundarbans-atlas-book) · [Bangladesh Fisheries Information Share Home](https://en.wikipedia.org/wiki/Bangladesh_Fisheries_Information_Share_Home)
- [Flora of Pakistan (eFloras legacy)](http://www.efloras.org/flora_page.aspx?flora_id=5) · [Biodiversity of Pakistan BGN paper](https://www.researchgate.net/publication/366191883_BIODIVERSITY_OF_PAKISTAN_DATABASE_AND_GLOBAL_NETWORKING_BGN)

China, Hong Kong, Taiwan
- [Chinese Virtual Herbarium](https://www.cvh.ac.cn/) · [NSII](http://nsii.org.cn/2017/home.php) · [CampusFlora](http://site.nsii.org.cn/CampusFlora.html) · [Catalogue of Life China](http://sp2000.org.cn/browse/browse_taxa) · [AmphibiaChina](https://www.amphibiachina.org/) · [Plant Photo Bank of China](http://ppbc.iplant.cn/) · [FRPS](http://www.iplant.cn/frps) · [Flora of China (eFloras)](http://www.efloras.org/flora_page.aspx?flora_id=2) · [Moss Flora of China](http://www.efloras.org/flora_page.aspx?flora_id=4) · [Shanghai Digital Flora](https://shflora.ibiodiversity.net/) · [Hangzhou Online Flora](http://db.hzbg.cn/hzflora/) · [Flora of Jiangsu](http://www.jszwzw.com/HomePage/) · [Biotracks](http://www.biotracks.cn/) · [Biogrid](https://biogrid.scbg.ac.cn/app) · [China Bird Report Center](http://www.birdreport.cn/) · [Dongniao](https://dongniao.net/) · [shellsmap](https://shellsmap.com/) · [GobiologyCN](https://gobiologycn.com/) · [Odonata Research](https://www.china-odonata.top/) · [Ant Network](http://www.ants-china.com/index.html) · [Insecta Integration](https://insectaintegration.com/) · [Mycopedia](http://www.mycopedia.top) · [Chinese Field Herbarium](https://www.cfh.ac.cn/) · [China Animal Scientific Database](http://zoology.especies.cn/) · [biodiversity science in China overview, National Science Review](https://academic.oup.com/nsr/article/8/7/nwab032/6147049)
- [HKBIH](https://bih.gov.hk/en/about-us/index.html) · [species database](https://bih.gov.hk/en/species-database/index.html) · [launch press release, 1 March 2022](https://www.info.gov.hk/gia/general/202203/01/P2022030100691.htm) · [AFCD HKBIH page](https://www.afcd.gov.hk/english/conservation/hkbiodiversity/hkbih/hkbih.html) · [Hong Kong Biodiversity Online](https://www.afcd.gov.hk/english/conservation/hkbiodiversity/hkbiodiversity.html) · [Digital Policy Office case study](https://www.digitalpolicy.gov.hk/en/our_work/success_stories/hkbin_and_bgis/)
- [TBIA](https://tbiadata.tw/en-us/) · [TaiCOL](https://taicol.tw/en) · [TaiCOL GBIF dataset](https://www.gbif.org/dataset/1ec61203-14fa-4fbd-8ee5-a4a80257b45a) · [TaiBIF data products](https://portal.taibif.tw/en/data-product) · [TaiBIF GBIF publisher page](https://www.gbif.org/publisher/12b1df00-3f75-11d8-aa2d-b8a03c50a862) · [TaiBIF on re3data](https://www.re3data.org/repository/r3d100011310) · [TaiBIF IPT](https://ipt.taibif.tw/resource?r=taibnet_com_all) · [Lyudao LTSER](https://ltsertwlyudao.org/)

Japan and Korea
- [JBIF](https://gbif.jp/en/) · [activities](https://gbif.jp/en/activities/) · [data statistics](https://gbif.jp/en/datause/stat/) · [about](https://gbif.jp/en/about/jbif/summary/) · [collaborating organisations](https://gbif.jp/en/about/jbif/related/) · [BISS paper on Japan's biodiversity information](https://biss.pensoft.net/article_preview.php?id=111893&skip_redirect=1)
- [J-IBIS / Biodiversity Center of Japan](https://www.biodic.go.jp/) · [Monitoring Sites 1000](https://www.biodic.go.jp/moni1000/) · [CHM metadata search](https://www.biodic.go.jp/chm/) · [National Survey vegetation dataset on GBIF](https://www.gbif.org/dataset/d4cca499-31be-445d-a16a-b927a035767f)
- [BISMaL](https://www.godac.jamstec.go.jp/bismal/j/) · [J-OBIS](https://www.godac.jamstec.go.jp/j-obis/j/) · [fishpix](https://fishpix.kahaku.go.jp/fishimage/) · [JaLTER](http://www.jalter.org/) · [JaLTER Metacat](https://db.cger.nies.go.jp/JaLTER/) · [NIES Invasive Species Database](https://www.nies.go.jp/biodiversity/invasive/) · [FFPRI relevé database](https://www.ffpri.go.jp/labs/prdb/) · [NARO grassland vegetation database](https://www.naro.go.jp/laboratory/nilgs/vegetation/index.html) · [semi-natural grassland flora dataset](https://gbif.jp/ipt/resource?r=jsngf) · [Fukui Green Data Bank](https://fncc.pref.fukui.lg.jp/fukuinature/greendatabank)
- [Species Korea (NIBR)](https://species.nibr.go.kr/) · [Korean National Species List](https://en.wikipedia.org/wiki/Korean_National_Species_List) · [Korean herpetofauna dataset, Scientific Data](https://www.nature.com/articles/s41597-022-01924-z)

Southeast Asia
- [MyBIS](https://mybis.gov.my/one/) · [ASEAN entry](https://km.aseanbiodiversity.org/resources/malaysia-biodiversity-information-system-mybis.61/)
- [Sarawak Biodiversity Centre](https://www.sbc.org.my/) · [Wikipedia](https://en.wikipedia.org/wiki/Sarawak_Biodiversity_Centre)
- [Biodiversity of Singapore](https://singapore.biodiversity.online/) · [NParks Flora and Fauna Web](https://www.nparks.gov.sg/florafaunaweb) · [Singapore Biodiversity Records volumes](https://lkcnhm.nus.edu.sg/publications/sg-biodiversity-records/volumes/) · [archives](https://lkcnhm.nus.edu.sg/publications/nature-in-singapore/singapore-biodiversity-records-archives/) · [ISSN record](https://portal.issn.org/resource/ISSN/2345-7597) · [LKCNHM online database announcement](https://lkcnhm.nus.edu.sg/online-database-captures-spores-rich-biodiversity/) · [PNAS: two centuries of biodiversity discovery and loss in Singapore](https://www.pnas.org/doi/10.1073/pnas.2309034120)
- [TH-BIF (ASEAN entry)](https://km.aseanbiodiversity.org/resources/thailand-biodiversity-information-facility-th-bif.49/) · [Thailand bee digitisation project on GBIF](https://www.gbif.org/project/1tRUhXbB0fXAxKywxgIR4T/digitizing-and-databasing-of-bee-specimens-in-thailand)
- [INDOBIOSYS](http://www.indobiosys.org/) · [DAAD project page](https://www.daad-indonesia.org/en/find-funding/indonesian-biodiversity-discovery-and-information-system-indobiosys/)
- [Co's Digital Flora of the Philippines](https://www.philippineplants.org/) · [Philippine Journal of Science paper](https://philjournalsci.dost.gov.ph/cos-digital-flora-of-the-philippines-plant-identification-and-conservation-through-cybertaxonomy/) · [Rufford Foundation grant](https://www.rufford.org/projects/pieter-pelser/cos-digital-flora-of-the-philippines-cybertaxonomy-to-the-rescue-of-conservation/) · [World Flora Online listing](https://about.worldfloraonline.org/floras/cos-digital-flora-of-the-philippines) · [Philippine KBAs and Red List assessments (ASEAN)](https://km.aseanbiodiversity.org/resources/philippine-key-biodiversity-areas-atlases-and-red-list-assessments.54/)
- [Sinh vật rừng Việt Nam / vncreatures](https://www.vncreatures.net/) · [introduction page](https://www.vncreatures.net/introduction.php) · [Vietnam biodiversity presentation to CBD (PDF)](https://www.cbd.int/doc/c/e644/9797/f8d779f6711667fad577ac29/chmws-2018-01-item-04-vn-en.pdf)
- [Mongabay: Bornean ecosystems and Indigenous lives survey](https://news.mongabay.com/2022/10/survey-captures-bornean-ecosystems-and-indigenous-lives-around-them/)

Central Asia, Russia, West Asia
- [Plantarium](https://www.plantarium.ru/lang/en.html) · [Moscow University Herbarium on GBIF](https://www.gbif.org/dataset/902c8fe7-8f38-45b0-854e-c324fed36303) · [Moscow Digital Herbarium paper](https://www.researchgate.net/publication/341188646_Moscow_Digital_Herbarium_a_consortium_since_2019) · ["Flora of Russia" on iNaturalist dataset paper](https://www.researchgate.net/publication/345983589_Flora_of_Russia_on_iNaturalist_a_dataset)
- [GSLEP](https://globalsnowleopard.org/) · [CAMCA project](https://camcaproject.org/species/snow-leopard/) · [Project Snow Leopard, MoEFCC (PDF)](https://moef.gov.in/uploads/2018/03/Project-Snow-Leopard-2008.pdf) · [Oryx snow leopard occupancy study](https://www.cambridge.org/core/journals/oryx/article/assessing-changes-in-distribution-of-the-endangered-snow-leopard-panthera-uncia-and-its-wild-prey-over-2-decades-in-the-indian-himalaya-through-interviewbased-occupancy-surveys/BC53A59280520EA73D20F6E2DCC61727)
- [CEPF Mountains of Central Asia species](https://www.cepf.net/our-work/biodiversity-hotspots/mountains-central-asia/species) · [ecosystem profile (PDF)](https://d29l0tur8ol1gj.cloudfront.net/sites/default/files/mountains-central-asia-ecosystem-profile-eng.pdf) · [Kyrgyzstan alien vascular plants checklist, BDJ](https://bdj.pensoft.net/article/145624/) · [Kyrgyzstan GRIIS dataset](https://www.gbif.org/dataset/d893c53b-a6b3-4e43-829f-fcd0cbb9bad3) · [Kyrgyzstan aquatic plant inventory](https://www.sciencedirect.com/science/article/pii/S2287884X23001048)
- [BioGIS Israel](https://biogis.huji.ac.il/) · [GBIF publisher page](https://www.gbif.org/publisher/67268f0e-3401-4be5-847b-3752cba6e71c) · [Science 2001 announcement](https://www.science.org/doi/10.1126/science.291.5504.559d) · [Israel Center for Citizen Science database](https://citizen-science.smnh.tau.ac.il/database/?lang=en)
- [TÜBİVES updates record](https://independent.academia.edu/EmelUslu) · [Alien flora of Turkey checklist](https://www.researchgate.net/publication/317318234_Alien_flora_of_Turkey_Checklist_taxonomic_composition_and_ecological_attributes) · [First alien plants database of Turkey](https://www.researchgate.net/publication/330349572_The_First_Alien_Plants_Database_of_Turkey)

Dead / migrated
- [Oriental Bird Images closure notice, 17 March 2021](https://www.orientalbirdclub.org/club-news/2021/3/17/oriental-bird-images-website-is-closing) · [Macaulay Library new home, 22 October 2021](https://www.macaulaylibrary.org/2021/10/22/the-oriental-bird-club-image-database-has-a-new-home-at-the-macaulay-library/) · [OBC image database landing page](https://www.orientalbirdclub.org/oriental-bird-images) · [Macaulay OBI collection](https://www.macaulaylibrary.org/oriental-bird-images/)

## Europe — Landscape/Place Documentation Projects Scan

**Verification date for all "active status" claims: 25 August 2026** unless a different date is stated. Method note: WebSearch quota for this session was exhausted partway through, so later verification relied on direct WebFetch against known/derived URLs. Several hosts returned proxy `403` (egress policy) rather than a site error — those are recorded as *unverified*, never as dead.

---

### Transnational / pan-European

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [EBBA2 — European Breeding Bird Atlas 2](https://ebba2.info/) | Transnational (Europe) | Consortium — European Bird Census Council; ~120,000 contributors stated on site | Occurrence + modelled abundance + change vs EBBA1 (1980s) | Bird | Active — site copyright 2026; interactive maps + data-request system live (25 Aug 2026) | 596 species; explicit **change maps** comparing two atlas periods — a built-in "two sources disagree" view. Data access is by formal request, not open download |
| [EBCC](https://www.ebcc.info/) | Transnational | Consortium of national bird-monitoring orgs | Programme hub, atlas + PECBMS indices | Bird | Active | Coordinating body for EBBA1/EBBA2 and national atlas alignment |
| [Euro+Med PlantBase](https://europlusmed.org/) | Transnational (Europe, Mediterranean, Caucasus) | Institution + distributed editor network; Secretariat at Botanic Garden Berlin (BGBM) | Nomenclature + area records (occurrence at country/region level) | Botanical | Active — migrated to EDIT Platform for Cybertaxonomy; edits "immediately visible on taxon pages"; site notes unresolved known issues | Regional-editor model: each area's record is attributable to a named editor. Whether competing taxonomic opinions are shown side by side: **unverified** |
| [EUNIS — European Nature Information System](https://eunis.eea.europa.eu/) | Transnational | Institution — European Environment Agency, Copenhagen | Multi-domain: species (esp. species named in legal texts), habitat types (Habitats Directive Annex I), designated sites | Legal + habitat + occurrence | Active | The clearest European model of **species ↔ habitat ↔ legal instrument ↔ site** cross-linking in one search. Record counts not published on landing page |
| [Natura 2000 data / N2K viewer (EEA)](https://www.eea.europa.eu/data-and-maps/data/natura-14) | Transnational | Institution — EEA / DG ENV, Member State submissions | Site boundaries + Standard Data Forms (species lists, habitat %, threats, management) | Legal | Active | SDFs are effectively **per-site multi-domain dossiers** submitted by governments — the closest EU analogue to a legal instrument register tied to a place |
| [Protected Planet / WDPA](https://www.protectedplanet.net/en) | Global (Europe well covered) | Institution — UNEP-WCMC + IUCN | Designations, governance, management-effectiveness assessments, OECMs | Legal | Active — statistics last updated **August 2026**; monthly update cycle | 312,943 protected areas (296,001 terrestrial/inland, 16,942 marine). Monthly versioned releases make it citable for disputed area figures |
| [ECOLEX](https://www.ecolex.org/) | Global | Consortium — FAO + IUCN + UNEP (partnership agreement 2001) | Treaties, national legislation, court decisions, literature | Legal | Active (maintained; no explicit "last updated" on landing page) | "Over hundred thousand references." Directly analogous to the legal-instrument layer the user wants; note it merges three organisations' catalogues, so duplicate/variant records occur |
| [EMODnet](https://emodnet.ec.europa.eu/en) | Transnational (European seas) | Institution/consortium — DG MARE, European Commission | Bathymetry, Biology, Chemistry, Geology, Human Activities, Physics, Seabed Habitats, Data Ingestion | Marine | Active — news items July 2026, events Oct–Nov 2026 | Seven-theme federated model with Map Viewer + Data Products Catalogue + ERDDAP. Good template for "many domains, one viewer" |
| [Marine Regions / Marine Gazetteer](https://www.marineregions.org/gazetteer.php) | Global, European core | Institution — VLIZ (Flanders Marine Institute) | Gazetteer: named marine features, boundaries, sampling stations | Gazetteer + marine | Active (in use by downstream apps; no version date on page) | **Most directly relevant gazetteer design in this list**: names stored as a separate entity from the geographic object, so one object carries many names/languages; typed relations (part-of, adjacent-to, flows-through). Explicitly built because different authorities define "the same" sea differently |
| [WoRMS — World Register of Marine Species](https://www.marinespecies.org/) | Global | Institution — VLIZ, with distributed taxonomic editors | Taxonomy + distribution + literature | Marine | Active | Accepted vs unaccepted names are both retained and linked — a working pattern for surfacing disagreement rather than silently overwriting it |
| [HELCOM Map and Data Service](https://helcom.fi/baltic-sea-trends/data-maps/) | Transnational (Baltic Sea) | Institution — HELCOM secretariat; Baltic Data Flows (EU-funded) | Geospatial: assessments, pressures, shipping density, habitats | Marine + legal | Active | ArcGIS REST + OGC WMS/WFS + metadata catalogue; per-dataset lineage |
| [Time Machine Organisation](https://www.timemachine.eu/) | Transnational | Consortium — originally EU H2020 grant 820323 | "Local Time Machines" — place-based historical data federations | Historic | Active as an organisation; per-LTM status varies and is **unverified** | Umbrella model for city/region-scale historical document mining; funding-opportunity brokering for local projects |
| [OldMapsOnline](https://www.oldmapsonline.org/) | Global, European-heavy | Small company — Klokan Technologies GmbH (Czechia) | Historical map discovery across library collections | Historic maps | Active (accounts, community sections live); total map/institution counts not on landing page | Georeferenced-extent search across many institutions; the same company built PastPlace's infrastructure |
| [Vici.org — Archaeological Atlas of Antiquity](https://vici.org/) | Transnational (Roman world) | **Solo** — René Voorburg | Sites + linked bibliography + linked open data cross-links | Historic/archaeology + gazetteer | Active — maintained; author publicly active (archaeo.social/@vici) | One-person, editable atlas that federates with Pleiades via linked data. Strong small-team precedent |
| [Pleiades](https://pleiades.stoa.org/) | Transnational (ancient Mediterranean/Europe) | Volunteer editorial community + institutions (Ancient World Mapping Center UNC; ISAW NYU; NEH funding since 2006) | Gazetteer: places, names, locations, connections | Gazetteer + historic | Active — "continuously published scholarly reference work" | 100% open source; CC-BY; every place has provenance and named contributors. US-hosted but its subject matter is European/Mediterranean — included for the gazetteer data model, which separates **place / name / location** into three linked entities |
| [Nuohtti](https://nuohtti.com/Content/about?lng=en-gb) | Transnational (Sápmi: NO/FI/SE) | Consortium — Sámi Archives of Norway, Finland and Sweden; built 2018–2021 with Univ. of Lapland (lead), Oulu, Umeå; EU Interreg Nord | Aggregated search across digitised archive material held in multiple European memory institutions | Indigenous/minority documentation + historic | Active and maintained | Purpose-built to reunite dispersed Indigenous heritage held *outside* the community's territory — closely analogous to colonial-archive recovery for a place like Sathyamangalam |
| [RomArchive](https://www.romarchive.eu/en/) | Transnational | Institution (now) — Documentation and Cultural Centre of German Sinti and Roma, Heidelberg; originally Kulturstiftung des Bundes-funded | Ten curated sections: Visual Arts, Dance, Film, Flamenco, Theatre & Drama, Literature, Music, Civil Rights Movement, Politics of Photography, Voices of the Victims | Indigenous/minority documentation | Live; custodianship transferred to the Documentation Centre. Recent-content evidence is thin (award 2020, publication 2020) — **likely in preservation rather than growth mode** | Curator-per-section, self-representation governance model; EN/DE/Romani |
| [GeoNature](https://geonature.fr/) | Transnational-capable, French-origin | Consortium of public bodies — Écrins, Cévennes, Vanoise, Mercantour National Parks + OFB | Full stack: Occtax (field data), TaxHub (taxonomy), GeoNature-atlas (public portal), GeoNature-citizen | Other (infrastructure) | Active — BAM ("Biodiversity Around Me") launched July 2026; national meeting Nov 2026; public roadmap; GitHub + demo.geonature.fr | **The most directly reusable tech stack for a single-site atlas.** Free/open-source, designed exactly for protected-area-scale multi-source occurrence + taxonomy + public atlas |
| Biolovision / `ornitho.*` network (operator: Biolovision Sàrl, Switzerland) | Transnational infrastructure | Small company powering ~20 national portals | Occurrence, with expert validation workflows | Bird (+ herps, odonata) | Active — powers ornitho.it, ornitho.de, ornitho.cat, faune-france, ornitho.pl etc. | Single codebase, per-country governance. Illustrates the "shared platform, local ownership" model |

---

### United Kingdom

#### England — national scale

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [BSBI Plant Atlas 2020](https://plantatlas2020.org/) | National (Britain & Ireland) | Institution partnership — BSBI + UKCEH + Biological Records Centre; thousands of volunteer recorders and vice-county recorders | Occurrence + trends + phenology + altitude + "apparency" frequency + conservation status | Botanical | Active (data release also on Zenodo, 2 May 2024) | **30+ million records; 3,495 species mapped (2,863 in the book).** Presents four time-slices (1930s, 1950s, 1990s, 2000–2019) and both long-term (1930–2019) and short-term (1987–2019) change indices on one page — an explicit apparatus for "which figure do you mean?" |
| [Rothamsted Insect Survey](https://www.rothamsted.ac.uk/insect-survey) | National | Institution — Rothamsted Research; ~80 light traps largely run by **volunteers** | Standardised long-term abundance time series + retained physical samples/bycatch | Other (insects) | Active — continuous since **1964** (60 years) | 16 suction traps (12 England, 4 Scotland), ~80 light traps UK+Ireland; >400 of ~600 UK aphid species, >1,500 moth species; daily records Apr–Nov. The exemplar for "one protocol, sustained for decades" |
| [Portable Antiquities Scheme database](https://finds.org.uk/database) | National (England & Wales) | Institution — British Museum-led + Finds Liaison Officers; public finders | Objects + findspot + parish + dating + images; ingested university datasets | Historic/archaeology | Active; records begin 1998, 1m objects by Sept 2014 | **1,531,145 objects in 982,775 records.** Tiered record visibility by user account (findspot precision withheld from public) — a real precedent for **sensitive-location redaction** in an open place atlas |
| [Heritage Gateway](https://www.heritagegateway.org.uk/) / [Historic Environment Records](https://historicengland.org.uk/advice/technical-advice/information-management/hers/) | National federation of local records | Institutions — **over 80 HERs**, maintained by county/unitary/district councils and national park authorities; Historic England coordination | Monuments; Events (excavations/surveys); Sources & archives; designations; historic landscape studies; PAS finds | Historic/archaeology | Active — ~two-thirds of HERs cross-searchable via Heritage Gateway | The single best European example of the user's target pattern at *administrative-area* scale: every place has monuments, the fieldwork events that produced knowledge about it, and citations to the archive. Coverage is deliberately uneven and documented as such |
| [Digital Survey of English Place-Names](https://epns.nottingham.ac.uk/) *(redirects to the [INS resource page](https://www.nottingham.ac.uk/research/groups/ins/Resources/Digital-Survey-of-English-Place-Names.aspx))* | National (England, by pre-1974 county) | Institution — Institute for Name-Studies, University of Nottingham; originated in a 2011–13 Jisc project with QUB and Edinburgh | Historical name attestations with document citations and dates, drawn from EPNS survey volumes 1925–2009 | Gazetteer + historic document mining | Active but explicitly **frozen in content**: it "currently reproduces the content of survey volumes without updating it, even where the etymologies suggested are now considered less likely than alternative explanations" | **The single most instructive precedent for the user's disputed-figures feature.** The project deliberately keeps the superseded interpretation *and* publishes a separate corrective layer (KEPN, below) rather than silently overwriting. Ongoing data-quality remediation by Nottingham DTS since 2013 |
| [Key to English Place-Names](https://kepn.nottingham.ac.uk/) | National (England) | Institution — Institute for Name-Studies, Nottingham | Name meaning + element breakdown + language of each element | Gazetteer | Live | The curated/consensus layer sitting over the raw Digital Survey. Uses pre-1974 counties — itself a versioned-geography problem worth noting |
| [British History Online](https://www.british-history.ac.uk/) | National | Institution — Institute of Historical Research, University of London (v6.0, © 2024) | ~1,300 volumes: Victoria County Histories, calendars, primary documents, maps, datasets | Historic document mining | Active — remaining VCH volumes being released over a two-year programme | ~40,000 images, ~10,000 historic map tiles; **hand-transcribed to stated 99% accuracy**. Mostly free; a small subscription tier for some Calendars of State Papers. VCH is the closest UK analogue to a parish-by-parish multi-source place history |
| [A Vision of Britain through Time](https://www.visionofbritain.org.uk/) | National | University research group — Univ. of Portsmouth (Humphrey Southall); boundary mapping since 1994 | Census statistics 1801→, vital registration/cause-of-death 1851–1910, administrative gazetteer, historic boundaries, travel writing, historical maps | Historic + gazetteer | Live | Explicitly built around **changing administrative geography** — an "Administrative Gazetteer... a definitive list of what units existed." Cited data volumes include 502,965 values from 1931 occupational tables and ~2m cause-of-death values plus ~800k derived values. The derived-vs-original distinction is exactly the disputed-figure problem |
| [Open Domesday](https://opendomesday.org/) | National (England, 1086) | **Solo** — Anna Powell-Smith; non-profit | Places, manors, people, folio images; searchable | Historic document mining | Live and still being written about (Adafruit feature, Apr 2026) | Solo front-end over an AHRC-funded 1990s dataset by Prof. J.J.N. Palmer's Hull team; data CC-BY-NC-SA hosted by Hull, folio images CC-BY-SA. **The model for "one person builds the interface, an institution holds the data"** |
| [Gatehouse Gazetteer](https://www.gatehouse-gazetteer.info/) | National (England & Wales) | **Solo** — Philip Davis | Site records with dense bibliographic citation to primary and secondary sources | Historic/archaeology + gazetteer | Site resolves but returned HTTP 403 to automated fetch on 25 Aug 2026; a full copy has been deposited at the [Internet Archive](https://archive.org/details/Gatehouse-Gazetteer) and it has a [Wikidata item](https://m.wikidata.org/wiki/Q59259501). Treat as **live-but-static / preservation-secured**; author's ongoing activity unverified | The exemplary solo gazetteer: its value is the *citation apparatus*, not the site list. Archive deposit as an explicit succession plan is worth copying |
| [Megalithic Portal](https://www.megalithic.co.uk/) | Global, UK & Ireland core | **Solo-led with community** — Andy Burnham (site), Pete Evans (layout); user-submitted content | Sites + photos + visit logs + forum + news | Historic/archaeology | Active — article submissions dated **August 2026** | 20+ years of continuous one-person stewardship with volunteer contribution; useful governance precedent for a small-team atlas |
| [ALERC (Association of Local Environmental Records Centres)](https://www.alerc.org.uk/) | National federation | Membership body; LERCs are county-scale not-for-profits | LERCs hold occurrence records, site data, habitat data, expert networks | Other (infrastructure) | Active — 2025 conference, job postings on site | LERCs are the county-scale bodies that both feed the NBN Atlas *and* keep richer local data (site dossiers, sensitive-species policies) that never reaches the national aggregator. The gap between local holding and national aggregate is exactly where conflicting figures arise |

#### England — county / local scale (the small-team and solo tier)

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [NatureSpot](https://www.naturespot.org/) | Regional — Leicestershire & Rutland | **Volunteer-run registered charity** (no. 1138852); partner-supported | Occurrence + species accounts + photos + local status; feeds the county records centre | Multi-taxon natural history | Active — species-of-the-month posts for June and July 2026; news items Aug 2026 | **8,495 species** on the site; running annual totals published openly. The clearest "two-county, volunteers only, still going after 15 years" model in the UK. Hosted on commodity hosting (Clook) |
| [Essex Field Club](https://www.essexfieldclub.org.uk/) | Regional — Essex (vc 18/19) | Small team — registered charity (no. 1217515), volunteer officers; physical base at Wat Tyler Country Park | Occurrence database + species accounts + **site (place) dossiers** + geology + digitised historical natural-history archives + paid ecological data-search service | Multi-domain natural history + historic | Active — records shown for **August 2026**; field meetings scheduled to Oct 2026 | **7,593,231 records for 22,218 species.** Runs a commercial "Datasearch" desk-study service off the same database — a viable funding model for a small place-atlas. Also actively digitising its own archive: occurrence + historical documents + site records in one site |
| [Sussex Botanical Recording Society](https://www.sussexflora.org.uk/) | Regional — vc 13 & 14 | Small team — volunteer society; BSBI vice-county recorders sit on the committee | Recording scheme feeding a printed county flora; downloadable documents | Botanical | Website live; **most recent dated content found was "The Stoneworts of Sussex (2020)"** and the *Flora of Sussex* (Feb 2018). Recording activity continues via BSBI but the *site* looks low-churn | A common failure mode worth naming: strong fieldwork, book-shaped output, thin web layer. No online atlas found |
| [Hampshire botany resource](https://www.hantsplants.uk/) | Regional — Hampshire & Isle of Wight | **Near-solo** — curated by a former BSBI South Hampshire recorder, with current recorders and the Hampshire Flora Group (supported by Hants & IoW Wildlife Trust) | News + resources for county botany | Botanical | Live; recency **unverified** (no dates found on the page fetched) | Illustrative of the personal-curation county resource — high expertise, single point of failure |
| [Nature of Dorset](https://www.natureofdorset.co.uk/) | Regional — Dorset | **Solo** — author states "This is my personal guide... it is my hobby, not my livelihood" | Species accounts + site guide + habitat accounts + own sightings database (species, site, date, observer) | Multi-taxon natural history | Live; recency **unverified** (no dates surfaced) | Notable for building a *personal* occurrence database with proper observer attribution, plus separate desktop and mobile builds |
| [Know Your Place](https://maps.bristol.gov.uk/kyp/) | Regional — Bristol, extended across West of England (incl. a Worcester edition at `?edition=wor`) | Institution + university + community — Bristol City Council; grew out of Univ. of Bristol's "Know Your Bristol" engagement project | Historic OS maps 1880s–1930s, 1946 and modern aerial imagery, an "info layer", and (per project literature) a community contribution layer | Historic + community heritage | Live and licensed under current OS terms; a community-layer feature is described in project write-ups but was not visible in the page fetched — **partly unverified** | Local-authority-hosted, multi-edition (each partner council gets its own edition off one codebase). Very close to what a district-scale Indian atlas would need |
| [Layers of London](https://layersoflondon.org/) | Regional — Greater London | Institution-led with public contribution — Institute of Historical Research / School of Advanced Study, National Lottery Heritage Fund; MOLA partner | Georeferenced historic map layers + crowd-contributed records pinned to place | Historic + community heritage | Live and navigable; record counts and recent-activity dates **not verifiable from the site** — the About/stats content did not render | Grant-funded crowd-sourcing over historic maps. Watch as a cautionary case: heavily promoted 2018–2020, current growth rate unclear |
| [Wytham Woods](https://www.wythamwoods.ox.ac.uk/) | **Single-site** (~1,000 acres, SSSI) | Institution — University of Oxford; Conservator's office within Oxford Green Estate | Long-term ecological datasets (some from the 1940s) + publications bibliography + iRecord species data + per-compartment gallery | Forest / single-site | Active | The canonical European single-site research forest. Its structure — *place → compartments → long-term datasets → publication list* — is the closest analogue to a single tiger-reserve atlas, though the datasets are not served as a unified public portal |

#### Scotland

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Canmore](https://canmore.org.uk/) | National | Institution — Historic Environment Scotland | Archaeology, buildings, industrial and maritime sites; photographs, drawings, manuscripts; monument-type and period thesauri | Historic/archaeology | Active (routine weekly maintenance windows advertised) | **>340,000 sites; >350,000 images.** Published controlled vocabularies (thesauri) are the reusable part — the Sathyamangalam atlas will need equivalent term lists |
| [PastMap](https://www.pastmap.org.uk/) | National | Institution — Historic Environment Scotland | Combined map search across archaeology, historic buildings and landscapes | Historic/legal | Active — maintenance notice dated 17–18 Aug; running Drupal 10 | **Explicitly warns it "should not be relied upon to confirm the presence of current designations"** and points to the HES Portal for live legal data. A candid, reusable pattern: separate the *discovery* map from the *authoritative legal register*, and say so on the page |
| [NLS Map Images — georeferenced explorer](https://maps.nls.uk/geo/explore/) | National + beyond | Institution — National Library of Scotland | Georeferenced historic map series; searchable by NGR, postcode, **historic place name**, county, parish | Historic maps + gazetteer | Live | **>420,000 zoomable map images** of Scotland, Ireland, England, Wales and beyond (figure cited on the ScotlandsPlaces closure page). Historic-place-name and parish search as first-class entry points |
| [Ainmean-Àite na h-Alba (Gaelic Place-Names of Scotland)](https://www.ainmean-aite.scot/) | National | Small institutional team — based at Sabhal Mòr Ostaig, Isle of Skye; commercial memberships | Searchable name map + downloadable name lists + academic papers + reading list | Gazetteer (minority language) | Active — news item dated **14 May 2026** | Minority-language authority with a **partial commercial model** (corporate memberships, co-produced maps). Directly relevant to Indian-language/tribal-name normalisation |

#### Wales

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [List of Historic Place Names of Wales](https://historicplacenames.rcahmw.gov.uk/) | National | Institution — RCAHMW; statutory list established under the Historic Environment (Wales) Act 2016, launched **2017** | "Hundreds of thousands" of historic name forms drawn from historical maps and other sources; glossary; blog on sources (incl. the EHC database) | Gazetteer + historic | Active — blog maintained; underpinned by statute | **A statutory historic-name register** — rare in Europe. Retains *multiple historical forms of the same place from different documents*, with source attribution: exactly the "conflicting names/spellings side by side" behaviour the user wants. Bilingual (Welsh/English) |
| [Archwilio](https://archwilio.org.uk/) | National (Wales HER) | Institution — Heneb: the Trust for Welsh Archaeology, on behalf of Welsh Ministers; consolidates the four former regional trusts (Clwyd-Powys, Dyfed, Glamorgan-Gwent, Gwynedd) | Historic environment records: sites and investigative events | Historic/archaeology | Active — site modified **August 2026**; records created after April 2016 searchable | Bilingual. Note the institutional consolidation of four independent trusts into one — a real-world lesson in what happens to per-region data models when organisations merge |

#### Northern Ireland

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Flora of Northern Ireland](http://www.habitas.org.uk/flora/) (habitas.org.uk) | National (NI) | Institution — National Museums NI (Ulster Museum) | Species list, plant groups, habitats, protected species | Botanical + legal status | Content live but **copyright line reads "© National Museums NI, 2010-2023"** — i.e. static/dormant. The related `www2.habitas.org.uk` host now serves a **parked cPanel holding page** (25 Aug 2026) | A useful decay case: the content survives, the surrounding infrastructure (which also carried CEDaR material) has partly evaporated. Note: CEDaR itself remains the NI records centre feeding the NBN Atlas |

---

### Ireland

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Biodiversity Maps](https://maps.biodiversityireland.ie/) / [National Biodiversity Data Centre](https://www.biodiversityireland.ie/) | National | Institution — an initiative of the Heritage Council, **operated under a service-level agreement by Compass Informatics** (a private contractor); partners incl. NPWS and DHLGH | Terrestrial + marine occurrence; taxon atlases (e.g. Ladybird Atlas 2025); national biodiversity indicators | Multi-taxon + marine | Active — **"Last Updated: 21 August 2026"** displayed on the portal | **9,261,812 records; 19,724 species; 211 datasets.** Ireland's GBIF node. The outsourced-operations model (state initiative, contractor-run platform) is an important governance option to weigh |
| [logainm.ie — Placenames Database of Ireland](https://www.logainm.ie/en/) | National | Institution partnership — Fiontar & Scoil na Gaeilge, Dublin City University + Dept. of Rural and Community Development and the Gaeltacht; technical platform by Gaois (© 2008–2026) | Official/recommended Irish and English forms, archival research records of the state Placenames Branch, [documented sources](https://www.logainm.ie/en/about/sources) | Gazetteer | Active — Gaois copyright to 2026; site relaunched **May 2022** | Describes itself as "a comprehensive management system for the placenames data, records and research of the State." Includes **Meitheal Logainm**, a crowdsourcing arm. Bilingual authority file with the state's own research files behind each name — the strongest model for an official-vs-vernacular name reconciliation layer |
| [dúchas.ie — National Folklore Collection](https://www.duchas.ie/en) | National, but recorded parish-by-parish | Institution + crowd — UCD (National Folklore Collection) and DCU; UCD Library administration since 2015; NFC itself is "a small team of specialist staff" | Main Manuscript Collection (**>3,500 volumes**), Schools' Collection, ~**90,000** photographs, audio (wax cylinders, acetate, tape, film 1940s–60s) | Historic document mining + folklore | Active; portal launched **December 2013**; **crowdsourced public transcription open since 2015**; UNESCO Memory of the World inscription Dec 2017 | The best European example of **crowdsourced transcription of place-linked oral tradition**. Schools' Collection material is inherently indexed by school → townland, making it a de facto local knowledge gazetteer. Directly transferable to Sathyamangalam's oral/tribal knowledge layer |
| [Wildflowers of Ireland](https://www.wildflowersofireland.net/) | National | **Solo** — Zoë Devlin | Species accounts with own photographs; **folklore, herbal use, historical and literary allusions**; search by English/Latin/**Irish** name or by colour | Botanical + folklore | Active — "© Zoë Devlin 2008 – 2026"; three books published, latest *Blooming Marvellous* | Eighteen years of continuous solo work. Notable for treating **folklore and vernacular naming as first-class data alongside botany** — closest single-person analogue to the multi-domain ambition |
| Down Survey of Ireland (Trinity College Dublin) | National, 1650s | Institution — TCD research project | Digitised 1650s Down Survey maps + Books of Survey and Distribution + Civil Survey, linked to modern geography | Historic maps + historical document mining | **Unverified** — `downsurvey.tcd.ie` refused automated fetch on 25 Aug 2026 (robots/connection failure); no closure evidence found | Listed as a lead: a landmark example of joining a 17th-century cadastral survey to modern coordinates |

---

### Nordics

#### Sweden

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Artportalen](https://www.slu.se/en/environment/statistics-and-environmental-data/environmental-data-catalogue/artportalen/) | National | Institution — SLU Swedish Species Information Centre (ArtDatabanken), funded by the Swedish EPA | Occurrence across **eleven** groups: algae, microorganisms, mammals, fish, birds, amphibians & reptiles, invertebrates, vascular plants, lichens, mosses, fungi | Multi-taxon | Active — **updates daily**; catalogue page dated June 2025 | **>100 million observations.** Explicit statement that much of the data is "quality assured through expert verification of both species identification **and place**" — place-verification as a named step is unusual and directly relevant |
| [SBDI / Bioatlas](https://biodiversitydata.se/) (bioatlas.se redirects here) | National | Consortium of **11 Swedish institutions** (Swedish Museum of Natural History, SLU, Stockholm, Uppsala, Lund, KTH and others) with an executive office | Occurrence + images + spatial layers + biologging + eDNA/ASV datasets | Multi-taxon | Active — "Data Fika" launching Sept 2026; events Sept–Oct 2026 | **259 datasets from 36 institutions; >177 million occurrence records; 340,844 species; 1.3m images; 74 spatial layers; 7 biologging datasets; 23 ASV datasets.** Built on the **Living Atlases** framework (same lineage as ALA/NBN) |
| [Ortnamnsregistret / Isof place-name archive](https://www.isof.se/namn/ortnamn) — legacy scans at [gamlaortnamnsregistret.isof.se](https://gamlaortnamnsregistret.isof.se/ortnamn.html) | National | Institution — Institutet för språk och folkminnen (Isof) | Scanned card-index of place names, arranged by county and parish; digitisation programme reported to UNGEGN | Gazetteer | Legacy scan interface live; the fetch of the modern Isof names page failed on 25 Aug 2026 (**partly unverified**) | The "old register" is browsable **by county → parish** — the archival card index preserved as-is rather than normalised, so original slips remain inspectable. That is a defensible answer to conflicting name evidence: publish the source card |

#### Norway

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Artsdatabanken / biodiversity.no](https://biodiversity.no/) | National | Institution — Norwegian Biodiversity Information Centre (Trondheim) | Species Map Service, Species Observations (Artsobservasjoner), Red List for Species, **Red List for Ecosystems and Habitat Types**, alien species, Nature in Norway (NiN) habitat typology, name database, Norwegian Taxonomy Initiative | Multi-taxon + habitat + legal-adjacent | Active — dataset version 1.364 published **24 Aug 2026**, daily update frequency | [The Species Observation Service dataset](https://ipt.artsdatabanken.no/resource?r=speciesobservationsservice2&request_locale=en) holds **38,270,228 occurrence records** plus 3.1m multimedia records. Distinctive for pairing species data with a formal **national habitat/ecosystem typology (NiN)** and its own Red List — a habitat classification the user would need an Indian equivalent of |
| [Stadnamnportalen / stadnamn.no](https://stadnamnportalen.uib.no/) | National | Institution — University of Bergen (Språksamlingane / language collections) | Federated place-name datasets, including the digitised **[Norske Gaardnavne](https://stadnamnportalen.uib.no/view/rygh)** (O. Rygh's farm-name survey) | Gazetteer + historic | Live (dataset list at `/datasets`); direct fetch returned 403 on 25 Aug 2026, so record counts **unverified** | Publishes a 19th-century scholarly name survey **as a queryable dataset alongside modern name registers** — i.e. historical and current authorities coexist rather than one superseding the other |

#### Finland

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [FinBIF / laji.fi](https://laji.fi/en) | National | Consortium — LUOMUS (Finnish Museum of Natural History, Univ. of Helsinki) with SYKE, LUKE, universities and agencies; CoreTrustSeal-certified | Occurrence + collections + [Red List (punainenkirja.laji.fi)](https://laji.fi/en) + invasive species (vieraslajit.fi) + iNaturalist Finland + Pinkka identification learning environment | Multi-taxon | Active — CoreTrustSeal application 2024; R package `finbif` on CRAN | Widely cited as a best-practice national biodiversity infrastructure ([Sci Data 2021](https://www.nature.com/articles/s41597-021-00919-6)). Fully API-first with an official R client. Homepage counters did not render on fetch, so live totals **unverified** |
| [Nimiarkisto — The Names Archive](https://en.kotus.fi/corpora-and-other-material/the-names-archive/) | National | Institution — Institute for the Languages of Finland (Kotus), part of the Finnish National Agency for Education; digital platform built with CSC – IT Center for Science | **~2.7 million place-name card files** (>2.3m for Finland), **466,500** personal-name records, **417,000** foreign geographical names, ~**40,000** maps | Gazetteer + indigenous/minority (Finnish **and Saami** names) | Active; toponymic collection **fully digitised**, project completed for Finland's 2017 centenary; live interface at nimiarkisto.fi (Finnish) | Covers ~**95% of Finland's traditional place names**. Digitisation added coordinates to card records, and the interface allows **enrichment of existing records** — a card-index-to-gazetteer conversion pattern that maps well onto Indian revenue/forest records |

#### Denmark

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Naturbasen](https://www.naturbasen.dk/) | National | Small independent operator (Denmark), self-described **25 years** of citizen science; operator entity not named on the page | Occurrence + species encyclopaedia + community ID help + **six specialised atlas projects** (butterflies, dragonflies, amphibians, etc.) | Multi-taxon | Active — recent-species feed live | **119,474 members; ~40,000 Danish species covered.** Free to users; sustained by **cooperation agreements with municipalities and national parks** rather than data sales. A genuinely viable small-operator funding model |
| [DOFbasen](https://dofbasen.dk/) | National | Institution — Dansk Ornitologisk Forening (BirdLife Denmark), Copenhagen; volunteer observers | Occurrence + photos + sound recordings, organised by **named locality** | Bird | Active — observations, photos and recordings from **August 2026** | **39,485,418 records across 20,782 named localities.** The locality dictionary (a controlled place gazetteer that observers must select from) is the reason the dataset is spatially usable — a directly copyable design decision |

#### Iceland

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Náttúrufræðistofnun / natt.is](https://www.natt.is/en) (formerly ni.is) | National | Institution — Icelandic Institute of Natural History | Red Lists, taxonomy, collections (geology, botany, zoology, mycology); **vegetation maps, geological maps, land cover, habitat-type mapping (terrestrial/freshwater/coastal), aerial photography, historical geospatial data, volcanic activity mapping**; map viewers + open-data download | Multi-domain (botanical, habitat, geological, marine-adjacent) | Active — Icelandic Red List of Birds **2025**; pollen monitoring and bird ringing ongoing | Unusually integrated: species + habitat typology + geology + **historical geospatial layers** under one institutional open-data portal. Domain migrated ni.is → natt.is (a live example of URL churn breaking citations) |

---

### Baltics

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [eElurikkus / eBiodiversity](https://elurikkus.ee/en) | National — Estonia | Institution + societies — University of Tartu Natural History Museum on the **PlutoF** platform; partners incl. Estonian Naturalists' Society, Estonian Ornithological Society | Occurrence + taxonomy + **three national atlases** + monitoring programmes (butterflies, pollinators, lepidoptera) + natural history collections + **protected-species lists by the three statutory protection categories** + **per-protected-area species lists (Soomaa, Matsalu, Lahemaa National Parks)** | Multi-domain: occurrence, legal, atlas, collections | Active — content referencing 2018–2026, 2025 activities listed | **45,397 species; >7 million occurrence records; 6 data resources, 6 data partners.** The **per-national-park species dossier** is the single closest existing feature to what the user is building for Sathyamangalam. Bundles: [Estonian Breeding Bird Atlas](https://elurikkus.ee/en/bird-atlas) (2018, Elts/Kuus/Leibak), [Atlas of Estonian Flora](https://elurikkus.ee/en/plant-atlas) (Kull & Kukk, orig. 2005), and an Estonian Mammals atlas as an ArcGIS StoryMap (from end-2019) |
| [loodusveeb.ee — databases & distribution atlases index](https://loodusveeb.ee/en/themes/nature-conservation/databases-distribution-atlases-and-other-data-sources) | National — Estonia | Institution — Estonian Environment Agency | A curated **register of registers**: atlases, observation databases, EELIS | Other (meta-index) | Live | Worth copying outright: a maintained public index of *which* database holds *what*, with operator named per entry. Also names EELIS (Estonian Nature Information System, `keskkonnaportaal.ee/et/eelis`) and the Nature Observations Database (`lva.keskkonnainfo.ee`) |
| [Dabasdati.lv](https://www.dabasdati.lv/en/) | National — Latvia | Small team / NGO (Latvian Fund for Nature lineage); community observers | Occurrence + forum + gallery + field guide; primarily birds | Multi-taxon, bird-dominant | Active — observations through **June 2026**; 249 active users at fetch | **2,325,630 observations.** Quadrilingual (LV/EN/RU/LT) — a small-country project that solved multilingual access early |
| Lithuania national biodiversity node | National | — | — | — | **Unverified** — no working node URL confirmed within remaining budget | Listed as a gap, not a negative finding |

---

### Western Europe

#### Netherlands

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [NDFF Verspreidingsatlas](https://www.verspreidingsatlas.nl/) ([about](https://www.verspreidingsatlas.nl/over.aspx)) | National | **Consortium of seven species organisations** — FLORON, BLWG, NMV, LIK, Stichting ANEMOON, RAVON, Zoogdiervereniging/Zodion — inside the Nationale Databank Flora en Fauna | Occurrence + distribution maps across vascular plants, mosses, lichens, fungi, charophytes, algae, mammals, reptiles, amphibians, fish, butterflies, dragonflies, molluscs, crustaceans, marine organisms | Multi-taxon | Active — **"maps are updated every night"** with the previous day's validated observations | **>200 million observations in the underlying database; ~20 million added annually.** Initiative began 2007; became integral to NDFF late 2015. The taxon-society federation model (each society owns and validates its own domain, one shared atlas front end) is the best European answer to "who is responsible for each domain's data quality" |
| [Sovon Vogelatlas](https://sovon.nl/tellen/telprojecten/vogelatlas) — previous edition preserved at [vogelatlas2015.sovon.nl](https://sovon.nl/tellen/telprojecten/vogelatlas) | National | Institution + volunteers — Sovon Vogelonderzoek Nederland | Breeding, wintering and migrant birds; systematic 55-minute counts in 1 km² squares within **1,685 atlas blocks** (5×5 km) | Bird | **Active and in a new cycle** — new fieldwork runs Dec 2026–2030; a new project website launches Sept 2026 | Repeats roughly every 15 years. Crucially, **the 2012–2015 atlas stays online at its own subdomain when the new one starts** — versioned atlases side by side rather than replacement. Directly relevant to conflicting-figures design |
| [Amsterdam Time Machine](https://amsterdamtimemachine.nl/) | Single-city | Institution — University of Amsterdam, with national heritage institutions | Linked Open Data federation of city + national heritage datasets; pilot projects incl. protest-location mapping and **historical address geocoding** | Historic document mining | Active — DH Benelux 2026 conference presentation; "Inside-Out Time Machines" collaboration with other Dutch cities | Honest about maturity: the data index is labelled "work in progress." **Geocoding historical addresses** is the core technical problem and is treated as such — the same problem as reconciling historical revenue-village names |

#### Belgium

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [waarnemingen.be](https://waarnemingen.be/) | National (Flanders-centred) | NGO + platform — Natuurpunt with Observation.org (webmaster listed as info@observation.org) | Multi-taxon occurrence with validation | Multi-taxon | Live but bot-protected: returned "Access Denied / Protected by BotStopper" to automated fetch on 25 Aug 2026 — **statistics unverified** | Part of the Observation.org family, which has also absorbed Spain's Biodiversidad Virtual uploads — consolidation of national citizen-science platforms into one international operator is a live European trend worth flagging |
| [Inventaris Onroerend Erfgoed](https://inventaris.onroerenderfgoed.be/) | Regional — Flanders | Institution — Agentschap Onroerend Erfgoed | **>90,000 heritage objects** across built heritage, archaeological heritage, **landscape heritage**, and maritime/floating heritage; plus "designation objects" carrying **juridical status** (protected/designated, UNESCO World Heritage, heritage landscapes, and zones with no expected archaeological remains) | Historic + legal | Active — recently-added items displayed; entries observed dated 2024 | **The best European example of separating the descriptive inventory record from the legal designation record**, with the legal object linked to but distinct from the thing described. That separation is precisely what lets you show "described as X, protected as Y, and the two disagree" |

#### France

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [INPN](https://inpn.mnhn.fr/) + [OpenObs](https://openobs.mnhn.fr/) | National | Institution — Muséum national d'Histoire naturelle, for the SINP programme (Ministry of Environment) | Occurrence, taxonomy (TAXREF), habitats, protected areas, statutory species status | Multi-domain | **DOWN.** OpenObs serves a service-interruption notice: "L'inventaire National du Patrimoine naturel et ses services sont indisponibles depuis la cyber-attaque subie cet été 2025." Confirmed still unavailable **25 Aug 2026** — over a year of outage. The `inpn.mnhn.fr` host answers only with a BunkerWeb WAF placeholder | The most consequential negative finding in this scan: France's single national aggregator has been offline for ~14 months after a cyberattack, while its **regional** platforms (e.g. SILENE, below) kept running. A powerful argument for federation over a single national point of failure |
| [SILENE](https://silene.eu/) | Regional — Provence-Alpes-Côte d'Azur | Institution — regional SINP platform; two audiences: SILENE Nature (public) and SILENE Expert (professional) | Fauna + flora occurrence | Multi-taxon | **Active — latest update 27 January 2026** (≈90,000 new observations); an October 2025 release added 331,666 records | Kept publishing through the national INPN outage. Also a good precedent for **two interfaces over one database, differentiated by audience** (public vs professional, with different precision) |
| [Faune-France](https://www.faune-france.org/) | National | NGO-led — LPO France with partner associations; Biolovision platform | Birds (garden birds, waterbirds, raptors), reptiles, insects (dragonflies, mantids), other fauna | Multi-taxon, bird-led | Active — CGU updated 28 Jan 2025; live observation feeds | Feeds EuroBirdPortal — a national platform designed from the start to export upward to a transnational one |
| [Tela Botanica](https://www.tela-botanica.org/) | National + francophone world | Association/network of French-speaking botanists | **eFlore** (descriptions, ecology, nomenclature), **Carnet en Ligne** (personal observation notebook with photos), **eVeg** (European phytosociological vegetation database), collaborative identification | Botanical | Active — ongoing participatory-science programmes and field journals | Notable for including a **vegetation-plot (phytosociological)** domain alongside occurrence and nomenclature. Member counts and observation totals not on the landing page — **unverified** |
| [Biodiv'Écrins](https://biodiversite.ecrins-parcnational.fr/) | **Single-site** — Parc national des Écrins | Institution — park staff; director Ludovic Schultz, scientific coordination Cédric Dentant | Occurrence by species, by commune, with photos; observations by park staff since the park's creation (1973) | Forest/protected-area single-site | Active — latest observations **24 August 2026**, continuously updated | **The single closest structural analogue to the user's project.** Built on **GeoNature-atlas** (open source, developed by the park itself). Critically, it publishes an explicit honesty caveat: the data come from "different scientific protocols" and are "neither exhaustive nor complete distribution data" — a model disclaimer for a reserve atlas assembled from heterogeneous sources |
| [Atlas de la Biodiversité Communale (ABC) programme](https://ofb.gouv.fr/les-atlas-de-la-biodiversite-communale) | National programme of **local** atlases | Institution-funded, locally executed — Office français de la biodiversité funds municipalities; each ABC is a small local team | Per-commune inventory + mapping + local action plan | Multi-taxon, place-based | Active — 2025 funding campaign run via regional agencies; [toolkit](https://ofb.gouv.fr/mettre-en-oeuvre-un-projet-atlas-de-la-biodiversite-communale-abc) published; funded-project data published on [data.gouv.fr](https://www.data.gouv.fr/datasets/atlas-de-la-biodiversite-communale-donnees-des-projets-finances) | **The most relevant funding/replication model found anywhere in Europe**: a national agency publishes a *methodology toolkit* plus grants so hundreds of small places each build their own atlas to a shared standard. OFB even publishes "L'Atlas de la biodiversité communale : et après ?" on what happens after the grant ends |

#### Germany

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [FloraWeb](https://www.floraweb.de/) | National | Institution — Bundesamt für Naturschutz (BfN), with Senckenberg, GBIF Deutschland, NetPhyD | **>4,000 taxa**: distribution maps, taxonomy/nomenclature, biology, ecology, Red List status, **legal protection status**, photos, herbarium specimens, phenology, butterfly host-plant relations, plant communities | Botanical + legal | Live (copyright 2026); running **GetSimple CMS**; no visible last-updated date | Distinctive for putting **legal protection status and ecological relations on the same species page as distribution**. Its dated CMS is itself a lesson in long-lived government portals |
| [NetPhyD](https://www.netphyd.de/) / [Deutschlandflora portal](https://dflora.netphyd.de/main) | National | Registered association (e.V.) — Netzwerk Phytodiversität Deutschland; volunteer field botanists; publishes with BfN | Occurrence and grid distribution behind the *Verbreitungsatlas der Farn- und Blütenpflanzen Deutschlands* (~3,000 maps, 2013/14) | Botanical | **Partially broken.** NetPhyD's own site is active (data preparation for the next German Red List running to Sept 2026; portal relocated March 2024; Deutschlandflora 3.0 mobile apps released). But **the portal URL the association itself gives, `dflora.netphyd.de/main`, returned HTTP 500 on 25 Aug 2026** | A volunteer association carrying a national atlas — impressive reach, fragile web infrastructure. Record counts not published |
| [Flora von Bayern / Bayernflora](https://www.bayernflora.de/) | Regional — Bavaria | Consortium of societies + agencies — Bayerische Botanische Gesellschaft, Bavarian State Office for the Environment, BUND Naturschutz, regional natural-history societies, university herbaria | **>4,000 species**: species profiles, distribution maps, checklist with photos, Red List, monitoring, forum, wiki | Botanical | **Partially dead.** The site states the **wiki infrastructure was hit by a cyberattack in November 2023 and remains under restoration**, with content only reachable via the Internet Archive and many links broken; the BIB database tools (profiles, maps, checklists, Red Lists) still work | Nearly three years of unrestored wiki loss. Note ~85% of German plant species occur in Bavaria, so this regional project has near-national coverage — and the wiki (the narrative/historical-literature layer) is exactly the part that was lost, while the structured database survived. A pointed warning about which layer to back up |
| [ArtenFinder Rheinland-Pfalz](https://www.artenfinder.rlp.de/) | Regional — Rhineland-Palatinate | Cooperation — Land Rheinland-Pfalz + Stiftung Natur und Umwelt Rheinland-Pfalz | Occurrence (animals, plants, fungi) with photo evidence and expert validation | Multi-taxon | Active — newsletter June 2025; excursion scheduled 23 May 2026; BANU training courses running | Notable privacy/consent design: **a user's records stay private until they explicitly release them for conservation use.** Relevant to tribal/sensitive-site data governance |
| [ornitho.de](https://www.ornitho.de/) | National (Germany + Luxembourg) | Institution + volunteers — Dachverband Deutscher Avifaunisten (DDA), with regional societies and conservation authorities | Occurrence + population surveys | Bird | Active — data displayed for **25 August 2026** | Feeds German atlas work (ADEBAR lineage); observation and observer totals not on the landing page |
| [ZOBODAT](https://www.zobodat.at/) | Transnational (Austria/Germany-centred) | Institution — Oberösterreichisches Landesmuseum (Biologiezentrum Linz) | Four linked domains: **literature/publications, biographies of naturalists, taxa, occurrence records** | Multi-domain: literature + biography + occurrence | Live; record counts and update dates not on the landing page — **unverified** | **The only project in this scan whose first-class entity is the naturalist and the publication, not the observation.** For historical-document mining that is the right shape: who recorded it, in what paper, and where. Highly relevant to reconstructing colonial-era records for Sathyamangalam |

#### Switzerland & Austria

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Info Flora](https://www.infoflora.ch/en/general/info-flora.html) (part of [InfoSpecies](https://www.gbif.org/publisher/64ee55c9-570a-42af-b7da-3f13c6b4e5a9)) | National | Foundation — national data and information centre for Swiss flora; one of several sister centres (incl. [SwissFungi](https://swissfungi.wsl.ch/en/distribution-data/), CSCF for fauna) under the InfoSpecies umbrella | Occurrence, Red List, [data services](https://www.infoflora.ch/en/data.html); published to GBIF as the Swiss National Databank of Vascular Plants | Botanical | Active — IPT resource maintained at [ipt.gbif.ch](https://ipt.gbif.ch/resource?r=infoflora-trac) | Switzerland's design choice is instructive: **one specialist centre per taxonomic domain**, federated under InfoSpecies, rather than one monolithic national portal |
| [Swiss Breeding Bird Atlas 2013–2016](https://www.vogelwarte.ch/en/atlas/) | National (CH + Liechtenstein) | Institution + volunteers — Swiss Ornithological Institute, Sempach; project manager and main author **Peter Knaus**; **>2,000 field volunteers** | Distribution, abundance and **altitudinal range** for ~200 breeding species | Bird | Fieldwork closed 2016; book published; **corrigendum published June 2020**; no interactive online atlas found on this page | Publishes its own effort accounting: ~3.9 working-years of fieldwork, 46,438 km travelled. And it **publishes a corrigendum** — a formal errata mechanism, which is the minimum viable version of the user's conflicting-data feature |
| [ortsnamen.ch](https://www.ortsnamen.ch/) | National | Institution/consortium (Swiss place-name research) | Place names with historical attestations and etymologies | Gazetteer | **Unverified** — the site returned an empty body to automated fetch (25 Aug 2026); no closure evidence | Listed as a lead |
| [Biodiversitäts-Atlas Österreich](https://biodiversityatlas.at/) ([about](https://biodiversityatlas.at/ueber-den-atlas/)) | National | Institution — funded by the Science & Research Department of **Lower Austria** (Land NÖ); part of the [Biodiversitäts-Hub Österreich](https://www.biodiversityaustria.at/biodiversitaets-hub/projekte/atlas/); Danube University Krems involved | Occurrence + [natural history collections registry](https://collectory.biodiversityatlas.at/) + [tools](https://biodiversityatlas.at/funktionen/) | Multi-taxon | Active — began 2017, portal online **end of 2019**, "continuously further developed" | Built on the **Atlas of Living Australia open-source infrastructure** (Living Atlases). Uses **GBIF Backbone Taxonomy version 2021** — a pinned taxonomic backbone version, which is exactly how you make conflicting names reproducible. Record counts not published on the about page |

---

### Southern Europe

#### Iberia

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Anthos](http://www.anthos.es/) | National — Spain | Institution — Real Jardín Botánico, CSIC ([GBIF Spain collection record](https://gbif.es/coleccion/csic-real-jardin-botanico-anthos-sistema-de-informacion-de-las-plantas-de-espana/)) | Occurrence + nomenclature for Spanish flora; linked to Flora iberica; mobile app released | Botanical | **Unverified** — the host returned proxy 403 (organisation egress policy), not a site error, on 25 Aug 2026. Dataset is [published to GBIF](https://www.gbif.org/dataset/4cf3eec1-b902-40c9-b15b-05c5fe5928b6) | Anthos is the information system built around the multi-decade *Flora iberica* project — a rare case of a printed critical flora with a matched live occurrence system |
| [Banc de Dades de Biodiversitat de Catalunya (BDBC)](https://mediambient.gencat.cat/ca/05_ambits_dactuacio/patrimoni_natural/sistemes_dinformacio/banc_de_dades_de_biodiversitat/) — legacy interfaces at [biodiver.bio.ub.es/bdbc](http://biodiver.bio.ub.es/bdbc/) and [/biocat](http://biodiver.bio.ub.es/biocat/) | Regional — Catalonia | Institution — Universitat de Barcelona with the Generalitat de Catalunya | Regional occurrence bank underpinning Catalan flora/fauna atlases | Multi-taxon | **Unverified** — university host returned proxy 403 (policy block) on 25 Aug 2026. The Generalitat landing page and third-party project records ([Observatori del Patrimoni Natural](https://observatorinatura.cat/projectes/info/37/), [Flora Catalana Milfulles](https://floracatalana.cat/drupal843/milfulles/numeros/num4/recursos1)) remain live | One of Europe's oldest regional biodiversity data banks. Note the split between a legacy university interface and a government portal — a common decay pattern |
| [Ornitho.cat](https://www.ornitho.cat/) | Regional — Catalonia | Institution — Institut Català d'Ornitologia (ICO); Biolovision platform | Birds, mammals, amphibians, reptiles, fish, insects, molluscs | Multi-taxon, bird-led | Active — "latest observations added 1 minute ago"; news through **August 2026** | Gateway to a family of ICO sub-portals: SIOC, Orenetes (swallow nests), Nius (nests) — domain-specific micro-atlases under one regional institution |
| [SIOC — Servidor d'Informació Ornitològica de Catalunya](https://www.sioc.cat/) | Regional — Catalonia | Institution — ICO, with the Catalan Biodiversity Data Bank (BDBC), regional and municipal authorities, and the Natural Sciences Museum of Barcelona; "hundreds of volunteers" | Synthesises **multiple separate monitoring projects** per species and per locality: SOCC, Projecte Sylvia, spring and autumn migration projects, PERNIS | Bird | Active — recommended citation reads "ICO.2026" | **Explicitly a synthesis layer over several independently-designed monitoring schemes** — structurally the same problem as reconciling figures from different surveys of one reserve. Whether it displays competing population estimates with intervals: **unverified** |
| [Biodiversidad Virtual](https://www.biodiversidadvirtual.org/) → now [Fotografía y Biodiversidad](https://www.fotografiaybiodiversidad.org/biodiversidad-virtual/) | National — Spain | Non-profit association "Fotografía y Biodiversidad" (managing since **2010**); volunteer-run; began as **Insectarium Virtual in 1995** | Photo-vouchered occurrence across multiple taxon galleries | Multi-taxon | **Active but hollowed out**: the association is active (monthly insect-group challenges posted **August 2026**) — but **the data-upload system has been migrated into observation.org**, with the association now a partner rather than platform operator | A 30-year volunteer project that outlived its own infrastructure and handed the platform layer to an international operator while keeping the community. An important end-game scenario for any small atlas |
| [Toponimia de Galicia](https://toponimia.xunta.gal/) | Regional — Galicia | Institution + crowd — Real Academia Galega and Xunta de Galicia; public contribution via the **Galicia Nomeada** app | Toponyms from municipality level down to **micro-toponyms** (individual fields, hills, streams) | Gazetteer (minority language) | Active — "Toponimízate" campaign events reported through 2026 | **>550,000 toponyms, with >478,000 registered covering 35% of the territory** — i.e. it publishes its own coverage gap as a headline figure. Explicit rationale: "Nobody knows Galicia better than Galicians themselves." Micro-toponymy at field/stream level is precisely the granularity a forest gazetteer needs |
| [Flora-On](https://flora-on.pt/) (+ [Azores edition](https://acores.flora-on.pt/)) | National — Portugal | **Small volunteer association** — Sociedade Portuguesa de Botânica; a handful of very active contributors | Occurrence + distribution maps + photos + [endemics lists](https://flora-on.pt/?q=endemismos) | Botanical | **Active, with the best update transparency in this whole scan**: the [update log](https://flora-on.pt/novidades-geo.php) publishes weekly counts — week of 17–23 Aug 2026: 2,060 new records, 299 new grid squares, 220 species; 10–16 Aug: 1,664 records; 3–9 Aug: 503 records | Publishes **weekly per-contributor credit by name** (Paulo Ventura Araújo, Josué Orfão, Francisco Clamote, Maria João Correia, João Domingues Almeida, João Lourenço, Duarte Frade, Guilherme Ramos, Udo Schwarzer, and others). A ~10-person core producing thousands of records a week. **The single best small-team model to imitate** |
| [Flora Digital de Portugal](https://jb.utad.pt/flora) | National — Portugal | Institution — Botanical Garden of UTAD (Univ. de Trás-os-Montes e Alto Douro), Vila Real; supported by the Council of Rectors of Portuguese Universities | Taxonomy, distribution, habitat, flowering periods, **common names**, photographs | Botanical | Active — page shows "last update **25/08/2026**"; launched **2004** | 22 years of continuous operation from a single university botanic garden. Self-described as "one of the largest floristic databases in Southern Europe." Note Portugal has **two** independent national flora sites (this and Flora-On) — an inherent conflicting-figures situation |

#### Italy & Greece

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Portale della Flora d'Italia / FlorItaly](https://dryades.units.it/floritaly/) | National | Institution — Department of Life Sciences, University of Trieste; Museo di Storia Naturale di Milano involved in the underlying inventory | **Aggregates the national checklists of native and alien vascular plants plus their published updates**, with nomenclatural and distributional data and links out to partner projects | Botanical | Active — **version 2025.2** | **Explicitly versioned (2023.1, 2025.2 …), which is the mechanism that makes disputed taxonomy tractable**: you can cite the version a figure came from. Aggregating several checklists plus their errata streams is exactly the user's problem. Whether disagreements are shown side by side: **unverified** |
| [Wikiplantbase](https://www.researchgate.net/publication/318440745_The_Wikiplantbase_project_the_role_of_amateur_botanists_in_building_up_large_online_floristic_databases) / [Acta Plantarum (IPFI)](https://escholarship.org/uc/item/1gm4k5x8) | Regional series + national | **Amateur-driven** — Wikiplantbase runs as regional instances (Toscana, Liguria, Sardegna…) built substantially by amateur botanists; Acta Plantarum grew out of a forum into a national floristic distribution database | Occurrence + regional floristic novelties | Botanical | Active as projects (Liguria novelties paper 2023; Acta Plantarum national database paper 2021); individual instance URLs **unverified** | The published literature explicitly argues the case for **amateur-built large floristic databases** — useful citable evidence for a small-team atlas's scientific legitimacy |
| [Ornitho.it](https://www.ornitho.it/) | National | Consortium of ornithological associations, partnered with CISO and LIPU; Biolovision platform | Birds + amphibians + reptiles + dragonflies, with expert validation for some taxa | Multi-taxon, bird-led | Active — observations dated **25 August 2026** | Explicitly positions itself as the data-collection layer **for distribution atlases**. Full maps require login — a deliberate access tier |
| [Network Nazionale della Biodiversità (NNB)](https://www.nnb.isprambiente.it/) | National | Institution — established by the Ministry for Ecological Transition (MASE), managed by **ISPRA**; federates national, regional and research institutions | Federated occurrence from many institutional nodes + citizen science initiatives | Multi-taxon | Active — current news feed, ongoing citizen-science initiatives | **~35,000 species; ~15 million observations.** A network-of-nodes architecture rather than a single database |
| [Filotis](https://filotis.itia.ntua.gr/) | National — Greece | **University research group** — ITIA, School of Civil Engineering, National Technical University of Athens | **Multi-domain by design**: Landscapes of Particular Natural Beauty (ΤΙΦΚ), NATURA biotopes, CORINE biotopes, other landscapes/biotopes — *plus* species in seven groups (plants, amphibians, invertebrates, reptiles, mammals, birds, fish) | Legal + habitat + occurrence + landscape | Live; serves **WMS/WFS** map services; "current edition" but no version date — recency **partly unverified** | **The closest European analogue to the user's combined legal + site + species + landscape structure**, and remarkably it is run by a civil-engineering hydrology group rather than a biodiversity agency. Landscape designations (ΤΙΦΚ) sit alongside EU designations, so the same place carries multiple overlapping legal identities |

---

### Central & Eastern Europe

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Pladias — Database of the Czech Flora and Vegetation](https://pladias.cz/en/) | National — Czechia | Consortium — Masaryk University (Brno), Institute of Botany of the Czech Academy of Sciences, University of South Bohemia; originally a Czech Science Foundation project **2014–2018** | **Occurrence + species traits + vegetation types + distribution maps**, with structured [downloads by theme](https://pladias.cz/en/download/phytogeography) | Botanical, multi-domain | Active — "continuously updated also after the end of the Pladias project"; no explicit last-update date on the site (**partly unverified**) | Documented in a full methods paper ([Preslia](https://www.preslia.cz/P211ChytryLo.pdf)) — the gold standard for a citable data infrastructure. Publishes an explicit **scope rule** (native and naturalised flora; most cultivated plants excluded but major crops and woody plants included) — the kind of stated inclusion criterion that prevents later disputes about counts |
| [Atlas roślin Polski (atlas-roslin.pl)](https://www.atlas-roslin.pl/mapy-wystepowania.htm) | National — Poland | **SOLO** — Marek Snowarski; "Copyright © 2002 – 2026 by Marek Snowarski" | Distribution maps at three zoom sizes with the **ATPOL grid** overlaid, plus species pages | Botanical | Active — page "generated 26 lipca 2026", content "last changed 27 grudnia 2024" | **The standout solo project in Europe: 24 years, one person, a national plant atlas.** Crucially it is transparent about provenance — the maps are derived from the published Zając & Zając ATPOL atlas (2001) and its 2019 update, not from unattributed data, and the site distinguishes *page generated* from *content last changed*. That two-timestamp discipline is directly worth copying |
| [ATPOL — Atlas rozmieszczenia roślin naczyniowych w Polsce](https://biblioteka.botany.pl/bib/9579) | National — Poland | Institution — Jagiellonian University / Institute of Botany PAN (Zając & Zając) | The authoritative grid-based distribution atlas (print) underlying atlas-roslin.pl | Botanical | Print atlas; 2019 supplement; no open web atlas of its own found | The classic case of an authoritative dataset whose only public web face is a volunteer's site |
| [ornitho.pl](https://ornitho.pl/) | National — Poland | Biolovision platform with Polish partners | Bird occurrence | Bird | **Uncertain — treat as a lead.** The fetch on 25 Aug 2026 returned a page whose latest observations were dated **26–27 August 2016** and whose newest news item was 6 May 2015. This may be a cached/stale response rather than a dead site; other `ornitho.*` instances were unambiguously live | Flagged rather than asserted. Needs a manual check |
| [Geoserwis GDOŚ](https://www.geoserwis.gdos.gov.pl/mapy/) | National — Poland | Institution — Generalna Dyrekcja Ochrony Środowiska | Protected areas, Natura 2000 sites, nature reserves as map layers | Legal | **Unverified** — host did not resolve for automated fetch (25 Aug 2026) | Poland's statutory protected-area viewer; listed as a lead |
| [MME / BirdLife Hungary](https://www.mme.hu/) | National — Hungary | NGO + volunteers — Magyar Madártani és Természetvédelmi Egyesület | Ringing and migration research, bird monitoring, species-specific surveys (owls, Mar–Jul), and a **"Térképes adatbázis / MAP"** map-based database | Bird | Active — a "MAP kihívás" (MAP challenge) ran May–July **2026** | Runs an engagement campaign *on the database itself* ("MAP challenge") to drive recording — a cheap and effective mechanism. Record counts **unverified** |
| [Bioportal](https://bioportal.hr/en/) | National — Croatia | Institution — Ministry of Environmental Protection and Green Transition (Zagreb), with the Institute for Environmental Protection and Nature | **Seven modules in one portal**: GIS viewer, BioAtlas (observations), Red List, **CroSpeleo** (cave register), IAS (invasive species), FCD (floristic collections), HR IPT (GBIF publishing) | Multi-domain: occurrence, legal, speleological, collections | Active — news items **May–August 2026**; live "Have you seen them?" citizen-science campaign | **26,325 species and 2,035,361 observations in BioAtlas; 3,270 valid Red List assessments; 4,559 objects in CroSpeleo.** Also runs as [GBIF Croatia](https://bioportal.hr/en/gbif-croatia/) and a [Living Atlases](https://living-atlases.gbif.org/participants/croatia/) participant. The **separate cave register** is a good example of adding a domain-specific sub-atlas rather than forcing everything into one schema |
| Naravovarstveni atlas (Slovenia) — [described in the literature](https://www.researchgate.net/publication/272496592_Naravovarstveni_atlas_-_the_Slovene_Information_System_for_Nature_Conservation) | National — Slovenia | Institution — Slovene Institute for Nature Conservation / ministry | Protected areas, Natura 2000, habitat types, species | Legal + occurrence | **Unverified** — `naravovarstveni-atlas.si` returned a redirect loop on 25 Aug 2026; the [Slovene Natura 2000 site](https://natura2000.gov.si/en/nature/) is live | Documented in the literature as "the Slovene Information System for Nature Conservation"; listed as a lead |
| [SmartBirds](https://smartbirds.org/) / [SmartBirds Pro](https://smartbirds.org/m/) + [Atlas of Breeding Birds in Bulgaria](https://atlas.bspb.org/en/) | National — Bulgaria | NGO — Bulgarian Society for the Protection of Birds (BSPB), with volunteer observers | Occurrence via structured **form types** per taxon group; breeding (15 Apr–15 Jul) and wintering (1 Dec–28 Feb) seasons on a UTM grid since **2016**; atlas layer | Bird (+ other groups) | Active — data collection ongoing via SmartBirdsPro; BSPB is a [major GBIF publisher](https://bspb.org/en/huge-contribution-of-bspb-to-the-global-biodiversity-information-facility/); the SmartBirds landing page rendered only metadata to automated fetch, so live counts are **unverified** | First atlas published **2007** (Yankov); a 2020 assessment covered **72 species with limited distribution in Bulgaria** from 2013–2020 data. **Continuous monitoring has replaced discrete atlas editions** — an explicit design choice that changes how you cite a figure. The original atlas is also archived as a [dataset](https://cloud.gbif.org/eca/resource?r=atlas_breeding_birds_bulgaria_2007) |
| [UkrBIN — Ukrainian Biodiversity Information Network](https://ukrbin.com/) | National — Ukraine | Small team / volunteer community | Photo-vouchered occurrence, taxonomy | Multi-taxon | **Currently failing** — the site returned a raw MySQL error, "Error 1040: Too many connections", on **25 Aug 2026**. No closure notice; likely resource exhaustion rather than abandonment | The single clearest illustration in this scan of the small-project failure mode: not a decision to stop, just an unpaid-for database tier under load, in a country at war |
| [Plantarium](https://www.plantarium.ru/) | National+ — Russia and neighbouring former-USSR states | **SOLO with community** — created and maintained by **Dmitry Gennadyevich Oreshkin, 2007–2026** | Open atlas + identification guide: hierarchical databases of vascular plants, mosses, liverworts, hornworts and **lichens**; photo galleries; **geographic points added by project participants** | Botanical | Active — 2026 date displayed; open call for photographers and botanists | **Nineteen years, one maintainer, transnational scope.** Clear rights model: text freely shareable with attribution, image rights retained by photographers — a rights split worth copying for a community photo atlas |
| [Russian Bird Conservation Union (RBCU)](https://www.rbcu.ru/) | National — European Russia | NGO — ~6,000 members across 66 Russian regions, with affiliates in neighbouring countries | Runs the **Atlas of Breeding Birds of European Russia** among its projects | Bird | Active — conferences and press releases dated **August 2026**; active forum | A national breeding-bird atlas sustained by a membership NGO. Whether an online interactive atlas exists: **unverified** |
| [Trakuş](https://www.trakus.org/) | National — Türkiye (incl. European Türkiye) | Small team / community — site built by Longplay Dijital Ajans, programming credited to Serkan Karaot; content from a photographer/observer community | Species pages + photos + videos + observations + forum | Bird | **Highly active** — content timestamped **25 August 2026**, daily uploads; species-page revisions through June 2026 | **513 species; 243,780 photographs; 10,303 observations; 2,295 videos.** A photo-first national bird atlas — note the photo:observation ratio (~24:1), which tells you what the community actually values and is a real design signal for a media-heavy atlas |
| Bizim Bitkiler (Turkish Plants Data Service) | National — Türkiye | Institution (Turkish universities/ANKARA lineage) | Nomenclature, distribution, herbarium and literature data for Turkish plants | Botanical | **Unverified** — `bizimbitkiler.org.tr` timed out on automated fetch (25 Aug 2026) | Listed as a lead |

---

#### Dead / dormant projects found

| Project | Evidence | Date checked | What was lost / what survives |
|---|---|---|---|
| **ScotlandsPlaces** ([scotlandsplaces.gov.uk](https://scotlandsplaces.gov.uk/)) | The site itself states: "ScotlandsPlaces **was** a collaborative resource from Historic Environment Scotland, the National Records of Scotland and the National Library of Scotland." **Closed 24 June 2025.** | 25 Aug 2026 | The *cross-institutional joined-up view* is gone; the underlying material (Ordnance Survey Name Books, Tax Rolls, 420,000+ NLS map images) survives, but only separately at each of the three partners. **The classic multi-partner-portal failure: the integration layer dies first, and it was the whole point.** Directly relevant to the user's aggregation design |
| **GB1900** (`gb1900.org`) | The **domain has lapsed and now serves an unrelated online-gambling site (branded "KOKO5000")** while still carrying a canonical URL referencing GB1900. | 25 Aug 2026 | GB1900 crowdsourced the transcription of place-name labels from Ordnance Survey County Series maps across all of Britain — the largest crowdsourced historical gazetteer in the UK. The transcription dataset was released openly (via the Vision of Britain / Portsmouth team and open-data mirrors) and survives; **the project's own web presence and brand are gone, and the domain now actively misleads anyone following a citation.** The single strongest argument in this report for depositing both data *and* documentation in an archive with a persistent identifier |
| **Cymru1900Wales** ([cymru1900wales.org](https://www.cymru1900wales.org/)) | The site's own text says the project "is being extended to cover the whole of Britain" as GB1900, and was available only "until GB1900 launched." Superseded — and its successor is itself now dead (above). | 25 Aug 2026 | A two-step loss: Welsh place-name transcription → GB1900 → lapsed domain. Operators and exact dates not stated on the page (**unverified**) |
| **New Forest Knowledge** (`newforestknowledge.org.uk`) | **DNS does not resolve** ("Name or service not known"). The Internet Archive could not be consulted — `web.archive.org` and the Wayback availability API are blocked by this session's egress policy. | 25 Aug 2026 | Was a Heritage-Lottery-era single-landscape knowledge base for the New Forest combining archaeology, ecology, historic documents and maps — precisely the pattern the user is building. Its disappearance is worth treating as a warning about grant-funded single-site portals with no institutional host. Because the URL could not be independently corroborated in this session, treat the identification as **probable, not certain** |
| **INPN / OpenObs, France** ([inpn.mnhn.fr](https://inpn.mnhn.fr/), [openobs.mnhn.fr](https://openobs.mnhn.fr/)) | OpenObs serves a notice: services "indisponibles depuis la cyber-attaque subie cet été 2025." `inpn.mnhn.fr` returns only a BunkerWeb firewall placeholder. **Still down ~14 months later.** | 25 Aug 2026 | France's entire national biodiversity aggregator — occurrence, TAXREF taxonomy, protected-area and statutory-status data. Regional platforms (SILENE) and NGO platforms (Faune-France, Tela Botanica) kept running throughout. **The strongest available evidence for federating rather than centralising** |
| **Bayernflora wiki** ([bayernflora.de](https://www.bayernflora.de/)) | Site states the wiki infrastructure was hit by a **cyberattack in November 2023**, remains under restoration, content is reachable only via the Internet Archive, and many links are dead. The BIB database tools still work. | 25 Aug 2026 | ~3 years unrestored. The **structured database survived; the wiki — the narrative, historical-literature and discussion layer — did not.** A precise warning about which layer of a multi-domain atlas is most fragile |
| **Deutschlandflora portal** ([dflora.netphyd.de/main](https://dflora.netphyd.de/main)) | **HTTP 500** server error, at the exact URL NetPhyD's own site gives as the current portal after its March 2024 relocation. | 25 Aug 2026 | Germany's national plant distribution portal, run by a volunteer association. The association is demonstrably active (Red List data prep to Sept 2026; new mobile apps) — the *web portal* is what is broken |
| **habitas.org.uk (Northern Ireland)** | `www2.habitas.org.uk` now serves a **parked cPanel/Zen hosting holding page**. The surviving [Flora of Northern Ireland](http://www.habitas.org.uk/flora/) content carries "© National Museums NI, **2010-2023**". | 25 Aug 2026 | Long-standing Ulster Museum natural-history web estate; content partly static, hosting partly evaporated |
| **UkrBIN, Ukraine** ([ukrbin.com](https://ukrbin.com/)) | Raw database error page: **"Error 1040: Too many connections."** | 25 Aug 2026 | Not abandoned — resource-starved. Included because this is the most common realistic death of a small atlas, and it is invisible from the outside (looks like a bug, behaves like an outage) |
| **Biodiversidad Virtual, Spain** (partial) | Association active (posts Aug 2026), but "the platform's data upload system is now integrated into observation.org," with the association reduced to partner status. | 25 Aug 2026 | A 30-year project (Insectarium Virtual, 1995 →) that survived by surrendering its platform. Community retained, infrastructure ceded |
| **Sussex Botanical Recording Society web presence** (soft dormancy) | Site live, but the newest dated content found was "The Stoneworts of Sussex (**2020**)" and the *Flora of Sussex* (Feb 2018). No online atlas. | 25 Aug 2026 | Fieldwork continues under BSBI; the web layer is low-churn. The commonest and least-reported failure mode: an active recording community with a fossilised website |
| **ornitho.pl** (uncertain) | Fetch returned observations dated **26–27 August 2016** and a newest news item of 6 May 2015, while sibling `ornitho.*` sites returned Aug 2026 data. | 25 Aug 2026 | Possibly a stale cache rather than a dead site. **Flagged, not asserted** — needs a manual browser check |

---

#### Leads I could not verify

**Blocked by this session's egress policy (proxy HTTP 403 — *not* evidence of site failure).** These must be rechecked manually:
- `anthos.es` — Anthos, Spanish plant information system (RJB-CSIC). Dataset confirmed live via GBIF.
- `biodiver.bio.ub.es` — legacy BDBC Catalonia interfaces (`/bdbc`, `/biocat`).
- `stadnamnportalen.uib.no` — Norwegian place-name portal (existence and datasets confirmed via search; record counts not obtained).
- `gatehouse-gazetteer.info` — Philip Davis's solo medieval fortifications gazetteer (site resolves; content and last-update date not obtained; Internet Archive copy exists).
- `waarnemingen.be` — bot-protection block; observation/species/observer totals not obtained.
- `web.archive.org` and the Wayback availability API — **entirely blocked**, which is why several dead-project entries rest on DNS/HTTP evidence rather than archived snapshots.
- `archaeologydataservice.ac.uk` — HTTP 403; the England **Historic Landscape Characterisation** archive (county-by-county HLC GIS deposits) could not be profiled. This is a significant gap: HLC is the single European practice closest to "characterise a whole landscape from many source types," and it deserves a dedicated follow-up.

**Rendered empty / metadata-only, needs manual check:** `ortsnamen.ch` (Swiss place names); `smartbirds.org` landing page (Bulgaria); `bizimbitkiler.org.tr` (Turkish Plants Data Service, connection timeout); `maps.nls.uk/geo/explore/` (map counts and recent additions); `layersoflondon.org` (record counts, contributor numbers, recent activity — the About content did not render); `insectsurvey.com` (the standalone Rothamsted Insect Survey site — figures were obtained instead from `rothamsted.ac.uk/insect-survey`).

**Hosts that did not resolve for guessed URLs (existence at those exact addresses unconfirmed — do not treat as dead):** `wiltshirebotany.org.uk`, `sedn.org.uk` (Shropshire Ecological Data Network), `kentbotany.org`, `gbif.lt`, `bioras.rs`, `geoserwis.gdos.gov.pl`, `biodiversitymaps.ie` (the working address is `maps.biodiversityireland.ie`), `downsurvey.tcd.ie`, `naravovarstveni-atlas.si` (redirect loop), `kartenforum.slub-dresden.de` (robots-disallowed), `app.bto.org/mapstore` (robots-disallowed — the BTO Bird Atlas 2007–11 online mapstore could not be profiled), `finds.org.uk` operator attribution.

**Quantities and team details that remain unverified across otherwise confirmed projects:** EBBA2 grid-square and country counts; Euro+Med taxon count and whether competing taxonomic opinions are displayed; EUNIS and Natura 2000 record counts; Marine Regions gazetteer entry count and version date; HELCOM dataset count; Time Machine's Local Time Machine count and per-LTM status; OldMapsOnline map and institution counts; Pleiades place count; RomArchive object count and launch year; laji.fi live totals; ZOBODAT record counts and update date; FloraWeb last-update date; ornitho.de/.it/.cat observation and observer totals; MME record counts; SmartBirds totals; Tela Botanica membership and observation counts; SIOC's handling of competing population estimates; NatureSpot's record total (species total 8,495 confirmed; record total not captured); Hampshire flora site and Nature of Dorset recency; Know Your Place community-layer confirmation; Historic Place Names of Wales exact name count.

**Method limitation to disclose:** the WebSearch quota for this session was exhausted after ~30 queries, before local-language searching could be completed for Denmark (Atlas III / DOF), Latvia, Lithuania, Greece (Hellenic Ornithological Society bird atlas), Türkiye, Serbia, Romania (SOR breeding bird atlas), Slovakia, and Nordic historical-map georeferencing projects. Those countries are consequently thinner here than the evidence base would allow, and Romania, Slovakia, Serbia and Bosnia-Herzegovina are effectively unrepresented.

---

#### Most transferable findings for the Sathyamangalam atlas

1. **The disputed-figures feature already has a working precedent, in place-names rather than biodiversity.** The Nottingham [Digital Survey of English Place-Names](https://www.nottingham.ac.uk/research/groups/ins/Resources/Digital-Survey-of-English-Place-Names.aspx) deliberately reproduces superseded etymologies unchanged — stating plainly that it does so "even where the etymologies suggested are now considered less likely than alternative explanations" — and publishes the corrective consensus as a *separate* resource, [KEPN](https://kepn.nottingham.ac.uk/). Two layers, both citable, neither overwriting the other.
2. **Version your whole atlas, not just your records.** [FlorItaly](https://dryades.units.it/floritaly/) ships as "2025.2"; [Biodiversitäts-Atlas Österreich](https://biodiversityatlas.at/ueber-den-atlas/) pins GBIF Backbone Taxonomy **v2021**; [Protected Planet](https://www.protectedplanet.net/en) releases monthly. Without a version, "the figure" is unciteable and every disagreement is unresolvable.
3. **Keep the superseded edition online.** Sovon keeps the 2012–2015 bird atlas at its own subdomain while the 2026–2030 fieldwork runs. Replacement destroys the comparison; coexistence is the feature.
4. **Separate the description from the legal designation.** Flanders' [Inventaris Onroerend Erfgoed](https://inventaris.onroerenderfgoed.be/) models "designation objects" as distinct entities from the heritage objects they cover. Greece's [Filotis](https://filotis.itia.ntua.gr/) lets one place carry several overlapping legal identities (national ΤΙΦΚ, NATURA, CORINE). This is what makes "protected as X, described as Y" expressible.
5. **Separate place, name, and location as three linked entities.** [Marine Regions](https://www.marineregions.org/gazetteer.php) and [Pleiades](https://pleiades.stoa.org/) both do this; it is why they can hold many names and many competing boundary definitions for one object without contradiction. [DOFbasen](https://dofbasen.dk/)'s 20,782 controlled localities show the cheap version of the same idea.
6. **Adopt an existing stack rather than building one.** [GeoNature/GeoNature-atlas](https://geonature.fr/) is French-park-built, open-source, actively released, and was designed for exactly this scale — see [Biodiv'Écrins](https://biodiversite.ecrins-parcnational.fr/) as the working single-site instance. The Living Atlases lineage (used by Sweden's [SBDI](https://biodiversitydata.se/), Austria's atlas, Croatia's BioAtlas) is the alternative.
7. **Publish coverage gaps and update cadence as headline numbers.** [Toponimia de Galicia](https://toponimia.xunta.gal/) leads with "478,000 toponyms across 35% of the territory." [Flora-On](https://flora-on.pt/novidades-geo.php) publishes weekly record counts with named contributors. Both convert incompleteness from an embarrassment into a credibility signal.
8. **State your inclusion rule and your protocol heterogeneity up front.** [Pladias](https://pladias.cz/en/) defines exactly which plants are in scope; [Biodiv'Écrins](https://biodiversite.ecrins-parcnational.fr/) warns its data come from "different scientific protocols" and are "neither exhaustive nor complete."
9. **Design record-level access tiers from day one.** The [Portable Antiquities Scheme](https://finds.org.uk/database) varies findspot precision by user account; [ArtenFinder RLP](https://www.artenfinder.rlp.de/) keeps a recorder's data private until they release it; [SILENE](https://silene.eu/) runs public and expert interfaces over one database. All three matter for sensitive species and for tribal knowledge.
10. **Plan the succession explicitly.** Solo and small projects here span 18–24 years ([atlas-roslin.pl](https://www.atlas-roslin.pl/mapy-wystepowania.htm) 2002–; [Plantarium](https://www.plantarium.ru/) 2007–; [Wildflowers of Ireland](https://www.wildflowersofireland.net/) 2008–) — but the failures are instructive: **GB1900's domain now hosts a gambling site**, ScotlandsPlaces' integration layer closed in 2025 while its sources survived, and France's national portal has been down for 14 months. Archive deposit with a persistent identifier (as Gatehouse did at the Internet Archive) is not optional.
11. **The most relevant funding model is France's [ABC programme](https://ofb.gouv.fr/les-atlas-de-la-biodiversite-communale)**: a national agency publishes a reusable methodology toolkit plus grants, so hundreds of small localities each build a comparable atlas — including published guidance on ["what happens after"](https://professionnels.ofb.fr/fr/node/375) the grant ends. And [Essex Field Club](https://www.essexfieldclub.org.uk/) demonstrates the self-funding variant: sell ecological desk studies off the same 7.6-million-record database that powers the free public atlas.
12. **The closest single existing feature to what you are building** is eElurikkus's **per-national-park species dossier** (Soomaa, Matsalu, Lahemaa) sitting inside a portal that also carries three national atlases, statutory protection categories, monitoring programmes and museum collections: [elurikkus.ee/en](https://elurikkus.ee/en).

---

#### Sources

- [EBBA2](https://ebba2.info/) · [EBCC](https://www.ebcc.info/) · [EBCC — EBBA2 page](https://www.ebcc.info/what-we-do/ebba2/) · [BTO on EBBA2](https://www.bto.org/our-science/projects/completed-projects/european-breeding-bird-atlas-2)
- [Euro+Med PlantBase](https://europlusmed.org/) · [EUNIS](https://eunis.eea.europa.eu/) · [EEA Natura 2000 data](https://www.eea.europa.eu/data-and-maps/data/natura-14) · [EEA Natura 2000 datahub item](https://www.eea.europa.eu/en/datahub/datahubitem-view/6fc8ad2d-195d-40f4-bdec-576e7d1268e4) · [Natura 2000 network (EEA)](https://www.eea.europa.eu/themes/biodiversity/natura-2000/the-natura-2000-protected-areas-network) · [Natura 2000 Slovenia](https://natura2000.gov.si/en/nature/)
- [Protected Planet](https://www.protectedplanet.net/en) · [ECOLEX](https://www.ecolex.org/) · [EMODnet](https://emodnet.ec.europa.eu/en) · [Marine Gazetteer (VLIZ)](https://www.marineregions.org/gazetteer.php) · [VLIZ on Marine Regions](https://www.vliz.be/en/marine-regions-gazetteer-un-ocean-decade-project) · [WoRMS](https://www.marinespecies.org/) · [HELCOM data & maps](https://helcom.fi/baltic-sea-trends/data-maps/)
- [Time Machine Organisation](https://www.timemachine.eu/) · [OldMapsOnline](https://www.oldmapsonline.org/) · [Klokan — PastPlace](https://www.klokantech.com/references/pastplace/) · [PastPlace (Portsmouth research portal)](https://researchportal.port.ac.uk/en/publications/pastplace-historical-gazetteer/)
- [Vici.org](https://vici.org/) · [Vici.org about](https://vici.org/about-vici.php) · [Pleiades](https://pleiades.stoa.org/) · [Pleiades on Vici.org](https://pleiades.stoa.org/news/blog/vici-org-via-the-pleiades-linked-data-sidebar)
- [Nuohtti](https://nuohtti.com/Content/about?lng=en-gb) · [Digital Access to Sámi Heritage Archives](https://digisamiarchives.org/news/) · [UArctic on Nuohtti](https://www.uarctic.org/news/2023/3/the-nuohtti-service-provides-digital-access-to-sami-material-in-european-archives/) · [RomArchive](https://www.romarchive.eu/en/) · [Documentation Centre takes over RomArchive](https://blog.romarchive.eu/documentation-and-cultural-centre-of-german-sinti-and-roma-takes-over-romarchive/)
- [GeoNature](https://geonature.fr/) · [Biodiv'Écrins](https://biodiversite.ecrins-parcnational.fr/)
- [Plant Atlas 2020](https://plantatlas2020.org/) · [BSBI Plant Atlas 2020 project page](https://bsbi.org/take-part/projects/plant-atlas-2020) · [BRC — Plant Atlas data released](https://www.brc.ac.uk/article/plant-atlas-2020-data-released) · [Plant Atlas 2020 data on Zenodo](https://zenodo.org/records/11103067)
- [Rothamsted Insect Survey](https://www.rothamsted.ac.uk/insect-survey) · [PAS database](https://finds.org.uk/database) · [Historic England — HERs](https://historicengland.org.uk/advice/technical-advice/information-management/hers/) · [Heritage Gateway](https://www.heritagegateway.org.uk/) · [Heritage Gateway HER list](https://www.heritagegateway.org.uk/gateway/chr/) · [Historic England research records](https://www.heritagegateway.org.uk/Gateway/Resource_Desc.aspx?resourceID=19191) · [ALGAO HER/SMR list](https://www.algao.org.uk/localgov/hersmr)
- [Digital Survey of English Place-Names (INS)](https://www.nottingham.ac.uk/research/groups/ins/Resources/Digital-Survey-of-English-Place-Names.aspx) · [epns.nottingham.ac.uk](https://epns.nottingham.ac.uk/) · [Key to English Place-Names](https://kepn.nottingham.ac.uk/)
- [British History Online](https://www.british-history.ac.uk/) · [A Vision of Britain through Time](https://www.visionofbritain.org.uk/) · [Vision of Britain sources](https://www.visionofbritain.org.uk/about/sources/1) · [Open Domesday](https://opendomesday.org/) · [Open Domesday about](https://opendomesday.org/about/) · [Gatehouse Gazetteer](https://www.gatehouse-gazetteer.info/) · [Gatehouse Gazetteer at the Internet Archive](https://archive.org/details/Gatehouse-Gazetteer) · [Megalithic Portal](https://www.megalithic.co.uk/)
- [Know Your Place](https://maps.bristol.gov.uk/kyp/) · [Know Your Place guide (SW Heritage)](https://swheritage.org.uk/historic-environment-service/local-heritage-list/know-your-place/) · [Layers of London](https://layersoflondon.org/) · [Layers of London (IHR)](https://www.history.ac.uk/resources/online-resources/layers-london) · [Layers of London (MOLA)](https://www.mola.org.uk/discoveries/research-projects/layers-london-mapping-citys-heritage)
- [NatureSpot](https://www.naturespot.org/) · [NatureSpot species totals](https://www.naturespot.org/species_totals) · [NatureSpot LRERC page](https://www.naturespot.org/content/leicestershire-rutland-records-centre) · [Essex Field Club](https://www.essexfieldclub.org.uk/) · [Sussex Botanical Recording Society](https://www.sussexflora.org.uk/) · [Flora of Sussex project](https://www.sussexflora.org.uk/projects/new-flora-of-sussex/) · [hantsplants.uk](https://www.hantsplants.uk/) · [Nature of Dorset](https://www.natureofdorset.co.uk/) · [ALERC](https://www.alerc.org.uk/) · [Wytham Woods](https://www.wythamwoods.ox.ac.uk/)
- [Canmore](https://canmore.org.uk/) · [PastMap](https://www.pastmap.org.uk/) · [ScotlandsPlaces (closure notice)](https://scotlandsplaces.gov.uk/) · [NLS georeferenced maps](https://maps.nls.uk/geo/explore/) · [Ainmean-Àite na h-Alba](https://www.ainmean-aite.scot/)
- [Historic Place Names of Wales](https://historicplacenames.rcahmw.gov.uk/) · [HPNW search](https://historicplacenames.rcahmw.gov.uk/placenames) · [HPNW data sources blog](https://historicplacenames.rcahmw.gov.uk/blog/our-data-sources-ehc-database) · [RCAHMW on the List](https://rcahmw.gov.uk/the-list-of-historic-place-names-of-wales/) · [Archwilio](https://archwilio.org.uk/) · [Cymru1900Wales](https://www.cymru1900wales.org/) · [GB1900 domain (now unrelated)](http://www.gb1900.org/) · [Welsh Place-Name Society links](https://www.cymdeithasenwaulleoedd.cymru/links/general/)
- [Flora of Northern Ireland (habitas)](http://www.habitas.org.uk/flora/) · [www2.habitas.org.uk (parked)](https://www2.habitas.org.uk/)
- [National Biodiversity Data Centre (IE)](https://www.biodiversityireland.ie/) · [Biodiversity Maps (IE)](https://maps.biodiversityireland.ie/) · [logainm.ie](https://www.logainm.ie/en/) · [logainm about](https://www.logainm.ie/en/about) · [logainm sources](https://www.logainm.ie/en/about/sources) · [dúchas.ie](https://www.duchas.ie/en) · [National Folklore Collection](https://www.duchas.ie/en/info/cbe) · [Dúchas project phases](https://www.duchas.ie/en/info/steps) · [Wildflowers of Ireland](https://www.wildflowersofireland.net/)
- [Artportalen (SLU)](https://www.slu.se/en/environment/statistics-and-environmental-data/environmental-data-catalogue/artportalen/) · [Artportalen (SLU, open data)](https://www.slu.se/en/environment/statistics-and-environmental-data/search-for-open-environmental-data/swedish-species-observation-system-artportalen/) · [SBDI / biodiversitydata.se](https://biodiversitydata.se/) · [SBDI collections](https://collections.biodiversitydata.se/public/show/dr5) · [GBIF Sweden IPT](https://www.gbif.se/ipt/resource?r=artdata&v=92.286) · [Isof — Ortnamn](https://www.isof.se/namn/ortnamn) · [Gamla Ortnamnsregistret](https://gamlaortnamnsregistret.isof.se/ortnamn.html) · [UNGEGN — Digitalisation of the Names Archive](https://unstats.un.org/unsd/geoinfo/UNGEGN/docs/29th-gegn-docs/WP/WP28_16_Digitalisation_of_the_Names_Archive.pdf)
- [Artsdatabanken / biodiversity.no](https://biodiversity.no/) · [Norwegian Species Observation Service (IPT)](https://ipt.artsdatabanken.no/resource?r=speciesobservationsservice2&request_locale=en) · [Norway's Species Map Service](https://artsdatabanken.no/Pages/135494/Norway_s_Species_Map_Service) · [Stadnamnportalen](https://stadnamnportalen.uib.no/) · [Stadnamnportalen datasets](https://stadnamnportalen.uib.no/datasets) · [Norske Gaardnavne](https://stadnamnportalen.uib.no/view/rygh)
- [laji.fi (FinBIF)](https://laji.fi/en) · [FinBIF portal info](https://laji.fi/en/about/897) · [FinBIF in a nutshell](https://info.laji.fi/en/frontpage/mission/) · [FinBIF in Scientific Data](https://www.nature.com/articles/s41597-021-00919-6) · [FinBIF CoreTrustSeal](https://cms.laji.fi/wp-content/uploads/2024/04/CoreTrustSeal_Application_FinBIF.pdf) · [Kotus — The Names Archive](https://en.kotus.fi/corpora-and-other-material/the-names-archive/)
- [Naturbasen (DK)](https://www.naturbasen.dk/) · [DOFbasen](https://dofbasen.dk/) · [Náttúrufræðistofnun (IS)](https://www.natt.is/en)
- [eElurikkus](https://elurikkus.ee/en) · [eElurikkus FAQ](https://elurikkus.ee/en/lvm/kkk) · [Estonian databases & atlases index (loodusveeb)](https://loodusveeb.ee/en/themes/nature-conservation/databases-distribution-atlases-and-other-data-sources) · [Dabasdati.lv](https://www.dabasdati.lv/en/)
- [NDFF Verspreidingsatlas](https://www.verspreidingsatlas.nl/) · [NDFF Verspreidingsatlas — about](https://www.verspreidingsatlas.nl/over.aspx) · [Sovon Vogelatlas](https://sovon.nl/tellen/telprojecten/vogelatlas) · [vogelatlas.nl](https://www.vogelatlas.nl/) · [Sovon atlas portal](https://portal.sovon.nl/atlas/index/) · [Amsterdam Time Machine](https://amsterdamtimemachine.nl/)
- [waarnemingen.be](https://waarnemingen.be/) · [Inventaris Onroerend Erfgoed](https://inventaris.onroerenderfgoed.be/)
- [INPN](https://inpn.mnhn.fr/) · [OpenObs](https://openobs.mnhn.fr/) · [OpenObs (Living Atlases)](https://living-atlases.gbif.org/participants/openobs/) · [SILENE](https://silene.eu/) · [Faune-France](https://www.faune-france.org/) · [Tela Botanica](https://www.tela-botanica.org/) · [OFB — Atlas de la biodiversité communale](https://ofb.gouv.fr/les-atlas-de-la-biodiversite-communale) · [OFB ABC toolkit](https://ofb.gouv.fr/mettre-en-oeuvre-un-projet-atlas-de-la-biodiversite-communale-abc) · [OFB — "et après ?"](https://professionnels.ofb.fr/fr/node/375) · [ABC funded-project data](https://www.data.gouv.fr/datasets/atlas-de-la-biodiversite-communale-donnees-des-projets-finances)
- [FloraWeb](https://www.floraweb.de/) · [FloraWeb data sources](https://www.floraweb.de/ueberfloraweb/datenquellen.html) · [NetPhyD](https://www.netphyd.de/) · [NetPhyD projects](https://www.netphyd.de/index.php/projekte/) · [NetPhyD — Deutschlandflora portal notice](https://netphyd.de/index.php/projektaktivitaeten/deutschlandatlas/31-wichtige-infos-fuer-nutzer-des-deutschlandflora-portals) · [dflora.netphyd.de/main (500 error)](https://dflora.netphyd.de/main) · [Bayernflora](https://www.bayernflora.de/) · [ArtenFinder RLP](https://www.artenfinder.rlp.de/) · [ornitho.de](https://www.ornitho.de/) · [ZOBODAT](https://www.zobodat.at/)
- [Info Flora](https://www.infoflora.ch/en/general/info-flora.html) · [Info Flora data](https://www.infoflora.ch/en/data.html) · [Info Flora IPT](https://ipt.gbif.ch/resource?r=infoflora-trac) · [InfoSpecies (GBIF publisher)](https://www.gbif.org/publisher/64ee55c9-570a-42af-b7da-3f13c6b4e5a9) · [SwissFungi distribution data](https://swissfungi.wsl.ch/en/distribution-data/) · [Swiss Breeding Bird Atlas](https://www.vogelwarte.ch/en/atlas/) · [ortsnamen.ch](https://www.ortsnamen.ch/)
- [Biodiversitäts-Atlas Österreich](https://biodiversityatlas.at/) · [Atlas — about](https://biodiversityatlas.at/ueber-den-atlas/) · [Atlas — tools](https://biodiversityatlas.at/funktionen/) · [Atlas collectory](https://collectory.biodiversityatlas.at/) · [Biodiversitäts-Hub Österreich](https://www.biodiversityaustria.at/biodiversitaets-hub/projekte/atlas/)
- [Anthos](http://www.anthos.es/) · [Anthos (GBIF Spain)](https://gbif.es/coleccion/csic-real-jardin-botanico-anthos-sistema-de-informacion-de-las-plantas-de-espana/) · [Anthos dataset (GBIF)](https://www.gbif.org/dataset/4cf3eec1-b902-40c9-b15b-05c5fe5928b6) · [BDBC (Generalitat)](https://mediambient.gencat.cat/ca/05_ambits_dactuacio/patrimoni_natural/sistemes_dinformacio/banc_de_dades_de_biodiversitat/) · [BDBC (UB legacy)](http://biodiver.bio.ub.es/bdbc/) · [BDBC (Observatori del Patrimoni Natural)](https://observatorinatura.cat/projectes/info/37/) · [BDBC (Flora Catalana Milfulles)](https://floracatalana.cat/drupal843/milfulles/numeros/num4/recursos1) · [Ornitho.cat](https://www.ornitho.cat/) · [SIOC](https://www.sioc.cat/) · [Biodiversidad Virtual → Fotografía y Biodiversidad](https://www.fotografiaybiodiversidad.org/biodiversidad-virtual/) · [Toponimia de Galicia](https://toponimia.xunta.gal/)
- [Flora-On](https://flora-on.pt/) · [Flora-On distribution updates log](https://flora-on.pt/novidades-geo.php) · [Flora-On endemics](https://flora-on.pt/?q=endemismos) · [Flora-On Açores](https://acores.flora-on.pt/) · [Flora Digital de Portugal (UTAD)](https://jb.utad.pt/flora)
- [FlorItaly / Portale della Flora d'Italia](https://dryades.units.it/floritaly/) · [FlorItaly index](https://dryades.units.it/floritaly/index.php) · [Ornitho.it](https://www.ornitho.it/) · [Network Nazionale della Biodiversità (ISPRA)](https://www.nnb.isprambiente.it/) · [Wikiplantbase project paper](https://www.researchgate.net/publication/318440745_The_Wikiplantbase_project_the_role_of_amateur_botanists_in_building_up_large_online_floristic_databases) · [Acta Plantarum paper](https://escholarship.org/uc/item/1gm4k5x8) · [Filotis](https://filotis.itia.ntua.gr/)
- [Pladias](https://pladias.cz/en/) · [Pladias phytogeography downloads](https://pladias.cz/en/download/phytogeography) · [Pladias traits downloads](https://pladias.cz/en/download/features) · [Pladias rules/info](https://pladias.cz/en/homepage/rules) · [Pladias methods paper (Preslia)](https://www.preslia.cz/P211ChytryLo.pdf)
- [atlas-roslin.pl distribution maps](https://www.atlas-roslin.pl/mapy-wystepowania.htm) · [ATPOL (IB PAN catalogue)](https://biblioteka.botany.pl/bib/9579) · [ornitho.pl](https://ornitho.pl/) · [Geoserwis GDOŚ](https://www.geoserwis.gdos.gov.pl/mapy/)
- [MME (BirdLife Hungary)](https://www.mme.hu/)
- [Bioportal (Croatia)](https://bioportal.hr/en/) · [Bioportal — GBIF Croatia](https://bioportal.hr/en/gbif-croatia/) · [BioAtlas (Living Atlases)](https://living-atlases.gbif.org/participants/croatia/) · [NIPP — Bioportal release](https://www.nipp.hr/default.aspx?id=673) · [Naravovarstveni atlas (paper)](https://www.researchgate.net/publication/272496592_Naravovarstveni_atlas_-_the_Slovene_Information_System_for_Nature_Conservation)
- [SmartBirds](https://smartbirds.org/) · [SmartBirds Pro](https://smartbirds.org/m/) · [BSPB on SmartBirds](https://bspb.org/en/about-birds/smartbirds/) · [Atlas of Breeding Birds in Bulgaria](https://atlas.bspb.org/en/) · [Atlas — take part](https://atlas.bspb.org/en/take-part/) · [2007 atlas dataset (GBIF ECA)](https://cloud.gbif.org/eca/resource?r=atlas_breeding_birds_bulgaria_2007) · [BSPB GBIF contribution](https://bspb.org/en/huge-contribution-of-bspb-to-the-global-biodiversity-information-facility/)
- [UkrBIN](https://ukrbin.com/) · [Plantarium](https://www.plantarium.ru/) · [Russian Bird Conservation Union](https://www.rbcu.ru/) · [Trakuş](https://www.trakus.org/)
- [Atlases of the flora and fauna of Great Britain and Ireland (Wikipedia)](https://en.wikipedia.org/wiki/Atlases_of_the_flora_and_fauna_of_Great_Britain_and_Ireland) · [NDFF Verspreidingsatlas (Wikipedia)](https://nl.wikipedia.org/wiki/NDFF_Verspreidingsatlas) · [Placenames Database of Ireland (Wikipedia)](https://en.wikipedia.org/wiki/Placenames_Database_of_Ireland) · [Swedish Institute for Language and Folklore (Wikipedia)](https://en.wikipedia.org/wiki/Swedish_Institute_for_Language_and_Folklore) · [Norske Gaardnavne (Wikipedia)](https://en.wikipedia.org/wiki/Norske_Gaardnavne) · [The Sámi Archives (Wikipedia)](https://en.wikipedia.org/wiki/The_S%C3%A1mi_Archives) · [Flora-On (Wikipedia)](https://en.wikipedia.org/wiki/Flora-On) · [Gatehouse Gazetteer (Wikidata)](https://m.wikidata.org/wiki/Q59259501)

## Africa & Middle East — "Document One Place/Population Comprehensively" Projects

Scope: North, West, Central, East, Southern Africa; Indian Ocean islands; Arabian Peninsula/Levant/Iran/Iraq. Excludes ALA, NBN Atlas, GBIF, Minnesota Biodiversity Atlas, India Biodiversity Portal (already catalogued elsewhere). All dates/counts are as found in searches/fetches on 2026-08-25; "unverified" flags where I could not independently confirm.

### Southern Africa

#### South Africa

| Project (URL) | Scale | Team size & type | Scope (domains) | Subject | Active status (evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [The Virtual Museum](https://vmus.adu.org.za/vm_projects.php) (ADU/BDI) | National, pan-African contributions | Small core team (Animal Demography Unit → Biodiversity & Development Institute, UCT-linked, Les Underhill et al.) + thousands of citizen "atlasers" | Multi-domain: 17 taxon-specific sub-atlases (birds, mammals, reptiles, frogs, fish, dung beetles, spiders, scorpions, orchids, weaver nests/PHOWN, etc.) | Multi-taxon occurrence records | Active — >2 million records reported (2023 paper); sub-projects show start dates from 2005 (ReptileMAP) to 2015 (DungBeetleMAP); page served Dec 2024 | Federated "MAP" architecture is the direct occurrence-data analogue the user wants; each MAP is its own mini single-subject atlas |
| [SABAP2 / BirdMap Africa](https://sabap2.birdmap.africa/) | Regional (S. Africa, Lesotho, Swaziland) → continental sister sites | FitzPatrick Institute of African Ornithology, UCT; core staff + volunteer "atlasers" | Occurrence + historical comparison (SABAP1 vs SABAP2) | Birds | Active since 2007, ongoing; "SABAP2 legacy" review paper (2022) documents long-running use | Direct precedent for historical-vs-current conflicting-range comparison (SABAP1 1987-91 vs SABAP2 2007-present) |
| [BODATSA / POSA](https://posa.sanbi.org/) (SANBI, BRAHMS Online) | National | SANBI institutional (National Herbarium Pretoria, Compton Herbarium, KZN Herbarium) | Occurrence + specimen/taxonomic backbone | Plants | Active — page states "31 March 2026" update | Explicitly disclaims completeness because of "specimen-based data collected over 200+ years" in different systems — a direct precedent for surfacing historical data-quality conflicts; verified-record flagging system |
| [Custodians of Rare and Endangered Wildflowers (CREW)](https://www.sanbi.org/biodiversity/building-knowledge/biodiversity-monitoring-assessment/custodians-of-rare-and-endangered-wildflowers-crew-programme/) | National | SANBI + Botanical Society of SA; regional citizen-science "custodian" groups | Occurrence + threat monitoring | Rare/threatened plants | Active (newsletters through ~2020 found; Facebook group active) | Citizen-science population monitoring feeding SANBI Red List |
| [Protea Atlas Project](https://www.proteaatlas.org.za/) | Regional (Cape Floristic Region) | SANBI/Kirstenbosch, led by Tony Rebelo; small team + volunteers | Occurrence + WORLDMAP conservation-planning analysis | Proteaceae | **Dormant since ~2001** (see Dead/dormant section) | Historic conflicting-range handling: notes "one-third of all proteas have had their distribution ranges extended" through atlassing |
| [BiodiversityGIS (BGIS)](https://bgis.sanbi.org/) / [Protected Areas Register](https://egis.environment.gov.za/protected_areas_register) / EIA Screening Tool | National | SANBI + Dept. of Forestry, Fisheries & Environment | Legal/governance + species-sensitivity layers | Protected areas, EIA screening, species maps | Active | Closest South African analogue to a legal-instrument/EIA registry the user wants for Sathyamangalam |
| [PRECIS / South African National Plant Checklist](https://www.gbif.org/dataset/f5dc22ca-0bb1-4692-8a97-cc7ac54d7ed9) | National | SANBI | Taxonomic backbone, 1970s digitisation of paper herbarium records | Plants | Active, continuously revised (2024 TAXON paper) | Early (1970s) computerisation of colonial-era herbarium records — historical document mining precedent |

#### Namibia

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [Namibian Biodiversity Database](https://www.biodiversity.org.na/) incl. [Tree Atlas of Namibia](https://treeatlas.biodiversity.org.na/) | National | Originated as a near-solo effort by **John Irish** (Namibia Scientific Society); now under Namibia's Environmental Information Service network | Occurrence, endemics, tree atlas | Multi-taxon, esp. endemics & trees | Unverified current status — site returned robots-disallowed on fetch; described in a library-catalogue title as "'Namibianising' biodiversity information" | Named solo/small-team origin fits the "solo project" pattern strongly |
| [NACSO — communal conservancies "State of Community Conservation" reports](https://nacso.org.na/) | National | Namibian Association of CBNRM Support Organisations (multi-NGO consortium) | Multi-domain: wildlife counts, community benefits/revenue, legal conservancy gazettement | Communal-conservancy wildlife & governance | Active — annual reports through 2022 report found on site | Combines legal-instrument data (conservancy gazettement) with wildlife occurrence and community-benefit data in one system |

#### Botswana / KAZA (Angola, Botswana, Namibia, Zambia, Zimbabwe)

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [KAZA Elephant Survey](https://www.kavangozambezi.org/2022/08/31/kaza-elephant-survey-begins/) | 5-country transfrontier | KAZA Secretariat + national wildlife departments + NGO partners | Aerial census, periodic (not continuous DB) | Elephants | Active — 2022 survey results published 2023 | Periodic large-N aerial survey, not a live database — useful contrast case |
| [TFCA Portal](https://tfcaportal.org/) (SADC) | 14 TFCAs, 11 SADC states | SADC TFCA Network secretariat | Multi-domain: legal/policy (SADC anti-poaching strategy), species/anti-poaching, community-benefit programmes, GIS | Transfrontier conservation areas | Active — news dated through Aug 2026, events through Nov 2026 | Closest Africa-wide analogue of a legal+species+community single-portal system |
| Okavango Delta species inventories (academic, no unified public portal found) | Site-level | Multiple academic groups (e.g. UCL, ANU-linked researchers) | Occurrence (freshwater fauna) | Freshwater biodiversity | Unverified as a standing public database — appears to be journal-paper-based, not a maintained portal | Lead only — see "leads I could not verify" |

#### Zimbabwe / Zambia / Mozambique / Botswana (cross-border)

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [Flora of Zimbabwe](https://www.zimbabweflora.co.zw/) / [Flora of Zambia](https://www.zambiaflora.com/) / Flora of Mozambique / Flora of Botswana (sister sites) | 4-country, expanding network | **Mark Hyde, Bart Wursten, Petra Ballings, Meg Coates Palgrave** — a small, largely volunteer/solo team | Species pages, photos, distribution, historical taxonomic notes | Plants | Active — copyright "2002–26"; software last modified 11 June 2025; images updated on rolling basis | Textbook example of a durable **solo/small-team** multi-decade site (24 years) — directly relevant precedent for the user's single-site model |
| [Zambian Carnivore Programme](https://www.zambiacarnivores.org/) | Multi-site (South Luangwa, Kafue, Liuwa Plain) | Small dedicated NGO team (Scott/Becker et al.) | Individual-ID monitoring, publications | Lion, leopard, hyena, wild dog | Active | Site-based long-term carnivore ID database, similar model to single-reserve documentation asked for |
| [SEOSAW (Socio-Ecological Observatory for Southern African Woodlands)](https://seosaw.github.io/) | Regional (Zambia, Mozambique, Tanzania, Zimbabwe, Malawi, Angola) | Academic consortium (Edinburgh-led) | Forest plot network, GitHub-hosted open data | Miombo woodland ecology | Active | Open-data/GitHub-based landscape documentation model for Miombo, akin to what a forest-landscape module in the atlas might use |

#### Mozambique

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [Gorongosa — E.O. Wilson Biodiversity Lab / "Biodiversity Exploration"](https://gorongosa.org/biodiversity/) | Single reserve | Gorongosa Restoration Project + E.O. Wilson Biodiversity Foundation, resident taxonomists & Mozambican para-ecologists | Multi-domain: species inventory, taxonomic training, human history/community, tourism | Whole-reserve biodiversity | Active — lab opened ~2014, ongoing "Science" pages | **Closest single-reserve model to what the user is building** — one site, many domains, active field taxonomy program |

### East Africa

#### Kenya

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [Kenya Bird Map](https://kenya.birdmap.africa/) | National | A Rocha Kenya + National Museums of Kenya + ADU/BDI technical support | Occurrence + historical atlas comparison | Birds | Active | Underlies Nussbaumer et al. 2025 (*Diversity and Distributions*) "Historical Bird Atlas and Contemporary Citizen Science Data Reveal Long-Term Changes" — comparing the 1970-84 printed *Bird Atlas of Kenya* to 2000-2020 citizen data; there is even a [GitHub digitisation project](https://github.com/Rafnuss/Digitization-of-A-Bird-Atlas-of-Kenya) re-digitising the old printed atlas's hand-drawn range maps — a direct historical-document-mining analogue |
| [Amboseli Elephant Research Project](https://amboseli.org/the-amboseli-elephant-research-project-a-legacy-of-science-protection-and-coexistence/) | Single ecosystem | Amboseli Trust for Elephants (founded by Cynthia Moss) | Individual-ID longitudinal database | Elephants | Active — running since 1972, called "50-year" project in multiple 2022-24 sources | Longest continuous individual-based population database in Africa; single-population precedent |
| [Mara Predator Conservation Programme](https://www.marapredatorconservation.org/) | Single ecosystem (Greater Mara) | Small NGO team | Individual-ID lion/cheetah/hyena catalogues, quarterly reports | Predators | Active — Q1 2025 technical report found | Public "Lion ID Catalogues" — individual-level occurrence data |
| [Save the Elephants — Samburu Elephant Project](https://savetheelephants.org/our-work/science/behaviour-society/samburu-elephant-project/) | Single ecosystem | Save the Elephants (Iain Douglas-Hamilton) | Individual-ID tracking database | Elephants | Active — since 1997, 2024 annual report found | GPS-tracking + individual ID single-population database |
| [Northern Rangelands Trust](https://www.nrt-kenya.org/) — Status of Wildlife Reports | Regional (39 community conservancies) | NRT + member conservancies | Multi-domain: wildlife counts, community/legal conservancy governance | Community-conservancy wildlife | Active — "Status of Wildlife Report 2005-2019" found | Indigenous/community-land conservation reporting parallel to the user's community/legal-instrument interest |
| [Chepkitale Indigenous Peoples' Development Project — Ogiek mapping](https://chepkitale.org/projects/mapping/) / ["Ogiek Peoples' Ancestral Territories Atlas"](https://www.iapad.org/wp-content/uploads/2016/09/HL07_Ogiek_Peoples_Ancestral_Territories_Atlas.pdf) | Single community/landscape (Mt Elgon) | Small community organisation + Forest Peoples Programme/IAPAD support | Participatory territorial mapping | Indigenous (Ogiek) land & resource documentation | Active project, atlas document itself may be static | Direct analogue for the "indigenous/community documentation" bullet — ancestral-territory atlas format |
| [National Museums of Kenya — Botany Dept / East African Herbarium](https://museums.or.ke/botany-department/) | National | National Museums of Kenya (with JRS Biodiversity Foundation grant, 2018) | Herbarium specimens, GBIF-published occurrence | Plants | Active (specimens on GBIF; JRS grant 2018) | State-museum-run herbarium digitisation, JRS-funded biodiversity informatics |
| [Mpala Research Centre](https://www.mpalalive.org/) | Single site (Laikipia) | Mpala Research Centre/Wildlife Foundation | Species checklists (birds, herps, dung beetles) built from decades of research papers | Multi-taxon | Species lists appear compiled from academic papers rather than a live occurrence database (old.mpala.org page found, "old." subdomain suggests site migration) | Lead — see unverifiable/partial entries; genuine single-site inventory but unclear current live database |

#### Tanzania

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [Serengeti Biodiversity Program](https://serengeti-forever.org/wp-content/uploads/2025/01/serengeti-biodiversity-report-web.pdf) | Single ecosystem | Serengeti research consortium (multi-institution) | Multi-domain research reporting | Whole-ecosystem biodiversity | Active — Annual Report 2023-2024 found | |
| [Snapshot Serengeti](https://www.nature.com/articles/sdata201526) | Single ecosystem | University of Minnesota Lion Project + Zooniverse citizen science | Camera-trap occurrence, 40 mammal species | Mammals | Data-paper published 2015; ongoing Zooniverse project (unverified current cadence) | Citizen-science-annotated camera-trap dataset — a distinct "occurrence data" generation method |
| [AfricanBioServices](https://africanbioservices.eu/) (Serengeti-Mara transboundary) | Transboundary ecosystem (Tanzania/Kenya) | EU Horizon2020 consortium (NTNU, U. Copenhagen, U. Groningen, ILRI etc.) | Multi-domain: biodiversity, ecosystem services, land use | Whole Serengeti-Mara ecosystem | Project period ended (H2020-funded, ~2015-2020); website still exists | Time-bound EU-funded landscape documentation project — good example of a "grant-funded, now static" project |
| Ngorongoro Conservation Area Authority — [Research & Monitoring](https://www.ncaa.go.tz/research-and-monitoring/) | Single site | NCAA (government authority) | Monitoring reports | Whole conservation area | Unverified as a public database (site is agency information page, not obviously a searchable DB) | Lead |

#### Ethiopia

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [Ethiopian Wolf Conservation Programme](https://www.ethiopianwolf.org/) | Single population (Bale Mountains + other Afroalpine sites) | WildCRU (Oxford) + Ethiopian Wildlife Conservation Authority | Individual-ID population monitoring, disease surveillance, community programme | Ethiopian wolf | Active — since 1988; 2022 annual report found | Long-running single-species/single-landscape monitoring, multi-domain (ecology + community + disease) |

#### Transboundary East/Central Africa

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [International Gorilla Conservation Programme — Mountain Gorilla Census](https://igcp.org/) | Transboundary (Virunga Massif: Rwanda/Uganda/DRC; Bwindi-Sarambwe: Uganda/DRC) | IGCP + partner governments/NGOs | Periodic censuses (not continuous DB) | Mountain gorilla | Active — censuses 2010, 2015-16, 2018, and Bwindi-Sarambwe census launched May 2025 | Periodic transboundary census model, similar structure to what user might need for a transboundary tiger landscape comparison |

### Central Africa

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| Congo Basin protected-area documentation (RAPAC, "Protected Areas in the Congo Basin" project) | Regional | RAPAC (Réseau des Aires Protégées d'Afrique Centrale) + academic partners | Governance + biodiversity reporting | Congo Basin PAs | Unverified current portal status — found only via secondary academic sources (rainforestparksandpeople.org), no live RAPAC data portal confirmed in this search | Lead — likely exists but I could not verify a live searchable database |

### West Africa

#### Nigeria

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [Nigerian Bird Atlas Project (NiBAP)](https://nibap.ng/) | National | A.P. Leventis Ornithological Research Institute (APLORI, Jos) + ADU/BDI technical support | Occurrence, pentad-grid atlassing | Birds | Active — started end of 2015; atlasing events documented through June 2022; covers 11,000+ pentads nationally | Strong solo-institution-led (APLORI) national atlas modeled directly on SABAP2 |

#### Ghana / Senegal-Gambia / Mali-Mauritania region

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| "West African Bird DataBase" (referenced via Birds4Africa) | Regional | Unverified host institution | Occurrence | Birds | Listed as "2010 – on-going" on Birds4Africa summary page; I could not locate/fetch the live database itself | Lead — likely exists under a different current URL; per Birds4Africa's tally, 23 of 54 African countries have completed atlases (Ghana, Senegal/Gambia, Mali/Mauritania, Liberia, Benin/Togo among "completed" historic projects, largely pre-digital/print-era) |

### North Africa

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| Egypt / Sinai flora & fauna documentation (St Katherine Protectorate) | Site-level (South Sinai) | Academic groups (VegEgypt/University collaborations) | Occurrence, vegetation | Sinai flora/fauna | Appears to be academic-paper-based (VegEgypt ecoinformatics paper, 2015) rather than a standing public portal | Lead — no live EEAA national biodiversity portal confirmed |

*(Libya, Tunisia, Algeria, Morocco: no site-level or population-level documentation projects surfaced in this pass beyond generic Wikipedia/CBD country-report pages — see "leads I could not verify.")*

### Indian Ocean Islands

#### Madagascar

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [REBIOMA (Réseau de la Biodiversité de Madagascar)](http://www.rebioma.net/) | National | Multi-institution network (WCS, Missouri Botanical Garden, Kew, others) | Multi-domain conservation-planning data portal | Whole-country biodiversity | **DEAD** — confirmed: `data.rebioma.net` now redirects (302) to a domain-resale page (dropcatch.com), and `rebioma.org` returns a 521 server error | Directly on point for the "abandonment in low-funding contexts" ask — a well-published (2018 BISS paper), multi-institution national portal now fully offline |

#### Seychelles

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [Seychelles Islands Foundation — Aldabra Atoll monitoring](https://www.sif.sc/aldabra) | Single atoll (UNESCO World Heritage) | SIF (small NGO, resident research station) | Long-term ecological monitoring (giant tortoise, green turtle nesting, vegetation/drought indicators) | Whole-atoll biodiversity | Active — recent scientific papers (2017-2019) built on continuous SIF monitoring data | Genuine single-site, long-term, multi-domain (species + climate) monitoring by a small dedicated foundation — close structural analogue to the user's model |
| Seychelles national biodiversity ("Species" pages, Ministry of Environment) | National | Government ministry (MACCE) | Species lists, endemics, red-list status | Multi-taxon | Appears to be static informational pages, not a searchable occurrence database | Lead |

#### Mauritius

No dedicated single biodiversity/occurrence database portal was found (National Parks and Conservation Service exists as an agency but no live public data portal was located). **Lead only — could not verify.**

### Arabian Peninsula / Levant / Iran / Iraq

#### Saudi Arabia

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [National Center for Wildlife — Open Data](https://www.ncw.gov.sa/en/open-data) | National | Saudi government agency (NCW) | Multi-domain: protected areas, endangered species, invasive species/plants, marine organisms | Whole-country wildlife | Active — datasets dated update June 2, 2025 and Aug 3, 2025 | Government open-data model, structurally close to a national species+PA legal registry |

#### United Arab Emirates

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [Abu Dhabi Species Portal](https://adnature.ead.ae/index.html) | Emirate-level | Environment Agency – Abu Dhabi (EAD) | Terrestrial + aquatic species records | Multi-taxon | Loads (site reachable); operator/species-count details not confirmable from fetch — **partially unverified** | Sub-national government species portal |

#### Oman

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| Oman Environment Authority biodiversity pages | National | Government (Environment Authority) | Biological diversity overview pages | Multi-taxon | Appears to be an informational agency page, not a searchable database — **unverified as a true database** | Lead |

#### Yemen (Socotra)

No standing digital biodiversity database/portal for Socotra was found — coverage is via Wikipedia/NGO articles and older UNEP material only. Given the ongoing conflict, this is a strong candidate for the "abandonment in low-funding/conflict contexts" pattern the task asked me to hunt for, but I found no specific project name/URL to confirm as ever having existed as a database (rather than print/paper checklists). **Lead only.**

#### Jordan

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| RSCN — Biodiversity Information Management System (BIMS) / "National Biodiversity Database" | National, with focal sites (Jerash, Dana Biosphere Reserve, Wadi Rum) | Royal Society for the Conservation of Nature (RSCN), built under a GEF/UNDP-funded project ("Integrating Biodiversity in the Tourism Sector") | Multi-domain: 2,482 plant + 736 fauna species, distribution maps, protected-area info | Whole-country biodiversity | **Dead/dormant application, live parent page**: RSCN's own page (`rscn.org.jo/national-biodiversity-database`) still describes it as active and links to `www.bims.rscn.org.jo`, but that URL returned a **404**, and the original IP-based URL (`80.90.161.188/bims`) timed out on connection | Strong case study in the exact "claimed-active-but-actually-dead" pattern; launched publicly per a June 2018 Jordan Times article |

#### Lebanon

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [SPNL — "Mapping Lebanon's Green Heart"](https://www.spnl.org/mapping-lebanons-green-heart-a-landmark-flora-and-biodiversity-map/) | National | Society for the Protection of Nature in Lebanon (SPNL) | Flora + biodiversity mapping | Plants/habitats | Active initiative per SPNL site (recency of the specific map project unverified — date not captured) | |

#### Israel

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [Israel Breeding Bird Atlas](https://ebird.org/atlasilps/about) | National | Israel Ornithological Center / hosted on eBird's atlas platform | Occurrence, breeding-atlas effort maps | Birds | Active | Built on eBird infrastructure rather than bespoke tech — a "hosted-platform" alternative to a custom-built atlas |

#### Iraq

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| Nature Iraq — Key Biodiversity Survey (Mesopotamian Marshes) | National/site (Southern Marshes) | Nature Iraq NGO | Site-level biodiversity survey reporting | Marsh biodiversity | **Likely dead**: `natureiraq.org` failed with an SSL handshake failure on fetch, consistent with an expired/unmaintained certificate or defunct hosting | Directly matches the "conflict/low-funding abandonment" pattern requested |

#### Iran

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [IranVeg — Vegetation Database of Iran](https://vcs.pensoft.net/article/114081/) | National | Iranian academic consortium (published via Pensoft/Zenodo, Dec 2024) | Vegetation plot database | Plant communities | Active/recent — publication Dec 2024/2025 | Recent (2024-25) national vegetation-plot database, a plant-community-scale analogue rather than species-occurrence |

### Pan-African / Multi-Country Single-Species Portals

| Project (URL) | Scale | Team | Scope | Subject | Status | Notable |
|---|---|---|---|---|---|---|
| [African Elephant Database / africanelephantdatabase.org](https://africanelephantdatabase.org/) | Continental | IUCN African Elephant Specialist Group (AfESG) | Periodic continental status reports (2002, 2007, 2013/16 data shown) | Elephants | Active — copyright through 2026; "African Elephant Range (2015)" map shown | Long documented in AfESG status reports for classifying survey estimates by reliability category (definite/probable/possible/speculative aerial vs. dung-count vs. informed-guess figures) — **this specific classification-scheme detail is widely cited in the literature but I did not get a direct on-page confirming quote in this session, so treat as high-confidence but technically unverified here** — this is precisely the "disputed/conflicting figures side by side" mechanism the user wants to build |
| [African Lion Database (ALD)](https://www.africanliondatabase.org/) | Continental | Originally launched 2018; now hosted by the IUCN SSC Cat Specialist Group's newly established (2025) Centre for Species Survival – Cats, funded by the Lion Recovery Fund & National Geographic | Population/distribution status by range state | Lion | Active — page modified Aug 12, 2026; 2025 population estimate (20,000-25,000) cited | Notable institutional migration: an earlier "African Lion Database Project" was separately hosted by ALERT/lionalert.org — i.e., the same concept/name has had **two distinct hosting lineages**, a useful case study in project succession/rebranding |
| [RhODIS (Rhino DNA Index System)](https://www.up.ac.za/research-matters/news/rhodis-rhino-dna-barcoding-rhino-dna-database-puts-poachers-cross-hair) | Continental (pan-African + international) | University of Pretoria (Cindy Harper's lab) + national wildlife/forensic agencies | Forensic DNA profile database linked to poaching case records | Rhino (black & white) | Active | Legal/forensic-evidence database tied to a population — different domain combination (genetics + law enforcement) than the user's current design |
| [GiraffeSpotter — Wildbook for Giraffe](https://giraffespotter.org/) | Continental | Giraffe Conservation Foundation + Wild Me (AI/tech partner) | Individual photo-ID via AI pattern-matching | Giraffe | Active | AI-based individual recognition — a tech-stack idea (automated pattern-matching ID) transferable to tiger-stripe ID in the Sathyamangalam context |
| [Range-Wide Conservation Program for Cheetah and African Wild Dogs (RWCP)](https://cheetahandwilddog.wordpress.com/) | Continental | ZSL + WCS, conceived by Sarah Durant & Rosie Woodroffe (2007) | Regional conservation strategies/action plans | Cheetah, African wild dog | **Website dormant**: WordPress blog's visible content tops out at 2016 newsletters; program itself appears to continue via a newer site, [Cheetah Conservation Initiative](https://cheetahconservationinitiative.com/) (2022 strategy document found) | Good example of a program migrating off a stale WordPress site to a newer domain, leaving the old one as a dormant husk |
| [CORDIO East Africa Data & Information Portal](https://cordio-data-portal-cordioea.hub.arcgis.com/) | Regional (WIO) | CORDIO East Africa NGO | ArcGIS-hub-based coral reef and coastal data | Coral reefs/marine | Active | Marine domain example, built on ArcGIS Hub (a replicable tech-stack option) |
| [WIOMSA — "Is there a WIO Coral Triangle?"](https://www.wiomsa.org/researchsupport/phase-iii-is-there-a-western-indian-ocean-coral-triangle/) | Regional (WIO) | Western Indian Ocean Marine Science Association | Coral biogeography research programme | Coral reefs | Project-phase based (Phase III referenced); continuously active as an association | |
| [Red Sea Atlas — Khaled bin Sultan Living Oceans Foundation](https://www.livingoceansfoundation.org/the-red-sea-atlas/) | Regional (Red Sea) | KSLOF (single foundation, expedition-based "Global Reef Expedition") | Satellite-derived reef habitat maps, published atlas (also as a book/Issuu PDF) | Coral reef habitats | Likely dormant as an active mapping programme (Global Reef Expedition concluded years ago; site now serves as a static publication archive) — **partially unverified**, flagging for confirmation | One-time expedition-based atlas rather than a continuously updated database — a distinct model type |

### Dead / dormant projects found

- **REBIOMA (Madagascar)** — `data.rebioma.net` 302-redirects to a domain-resale broker (dropcatch.com); `rebioma.org` returns a 521 server error. A 2018 peer-reviewed paper (BISS) documents it as a serious multi-institution (WCS, Missouri Botanical Garden, Kew) conservation-planning data portal; it is now fully offline. **Confirmed dead.**
- **Protea Atlas Project (South Africa)** — Ran 1991-2001 under SANBI/Tony Rebelo using WORLDMAP software; the project's own pages state the most recent distribution-map data version is from November 2001, with intent to update "every year" evidently unmet. iNaturalist community posts ("Time to start thinking about Protea Atlas Project II") treat it as a closed historical dataset needing a successor. The static site itself remains online as an archive. **Confirmed dormant/frozen since ~2001**, superseded informally by iNaturalist/CREW-style citizen science.
- **RSCN Jordan BIMS ("National Biodiversity Database")** — RSCN's institutional page still describes it as active and gives the URL `www.bims.rscn.org.jo`, but that address returns a 404, and the earlier IP-based URL (`80.90.161.188/bims/public/index.aspx`) timed out entirely. Launched publicly ~2018 per Jordan Times, covering 2,482 plant + 736 animal species. **Application appears dead despite the parent organisation's page not having been updated to reflect that.**
- **Nature Iraq (Mesopotamian Marshes key-biodiversity site)** — `natureiraq.org` failed with an SSL/TLS handshake failure, consistent with an expired certificate or abandoned server; the organisation's substantive published biodiversity-survey work dates to ~2010-2011. **Likely dead**, in a conflict/funding context matching the pattern the task asked to look for.
- **Range-Wide Conservation Program for Cheetah and African Wild Dogs — WordPress blog** — Visible content stalls at 2016 newsletters; the underlying conservation program continues, but this specific website property reads as an abandoned/superseded artifact (a newer domain, Cheetah Conservation Initiative, has since taken over the public-facing role). **Dormant website, live program elsewhere.**
- **Red Sea Atlas (Living Oceans Foundation)** — appears to be a closed, one-time expedition output (Global Reef Expedition) rather than an updated database; treat as dormant pending direct confirmation.

### Leads I could not verify

- **RAPAC / Congo Basin protected-area data portal** — referenced only via secondary academic sources; I could not locate or fetch a live, searchable RAPAC database in this session.
- **"West African Bird DataBase"** (Senegal/Gambia/Mali/Mauritania/Ghana/Benin/Togo atlases) — named on the Birds4Africa summary page as "2010 – on-going," but I could not find or fetch the actual live site/URL.
- **Mauritius National Parks and Conservation Service** — agency exists; no public occurrence/species database portal located.
- **Oman Environment Authority biodiversity database** — only informational agency pages found; no evidence of a searchable species database distinct from general text pages.
- **Socotra (Yemen) biodiversity database** — extensive endemic-species documentation exists in the literature/NGO articles, but no dedicated database/portal URL was found; plausible this never existed digitally given the conflict context, or exists but is unindexed.
- **Egypt EEAA national biodiversity portal** — Sinai/St Katherine flora-fauna documentation found only as academic papers (e.g., VegEgypt), not a standing public database.
- **Mpala Research Centre (Kenya) live species database** — species checklists found are compiled from individual academic papers and an "old.mpala.org" subdomain (suggesting a site migration); could not confirm a current live searchable database distinct from static checklists.
- **Colonial forest-department / hunting-record digitisation projects specifically tied to range reconstruction** — general scholarship on colonial archive digitisation in Africa was found (e.g., work on Pointe-Noire, Republic of Congo archives), but no single named project matching "historical hunting/forest records digitised specifically to reconstruct species ranges" was surfaced.
- **A dedicated Africa-wide historical place-name gazetteer** (parallel to what the user wants for Sathyamangalam) — only academic/critical-toponymy literature (colonial naming practices) was found, no standing gazetteer database/tool.
- **Abu Dhabi Species Portal (adnature.ead.ae) — operating details** — page loads but operator/species-count/update-recency could not be extracted from the fetch; treat the "active" status as provisional.
- **San/Bushmen and Maasai/Loita community-mapping projects** — real, well-documented participatory-mapping efforts exist (WIMSA-adjacent San initiatives, Loita Maasai land-use mapping via Ilkerin Loita Integral Development Programme materials, landDX Kenya-Tanzania borderlands dataset), but none of these appeared as a persistent, browsable "site" the way GBIF-style portals are — they read more as one-off cartographic/report outputs. Listing here rather than in the main table since I could not confirm a living, queryable web portal for any of them.

### Sources

- [The Virtual Museum: an African biodiversity database - two million records (iNaturalist journal)](https://www.inaturalist.org/projects/adu-virtual-museum-data/journal/113062-the-virtual-museum-an-african-biodiversity-database-two-million-records)
- [The Virtual Museum home](https://vmus.adu.org.za/vm_projects.php)
- [Southern African Bird Atlas Project (SABAP2) — FitzPatrick Institute](https://science.uct.ac.za/fitzpatrick/research-citizen-science/southern-african-bird-atlas-project-sabap2)
- [SABAP2 site](https://sabap2.birdmap.africa/)
- [The SABAP2 legacy review paper](https://scielo.org.za/scielo.php?script=sci_arttext&pid=S0038-23532022000100009)
- [African Bird Atlas Project / BirdMap Africa](https://www.birdmap.africa/)
- [Nigerian Bird Atlas Project (NiBAP)](https://nibap.ng/)
- [Kenya Bird Map](https://kenya.birdmap.africa/)
- [Digitization of A Bird Atlas of Kenya (GitHub)](https://github.com/Rafnuss/Digitization-of-A-Bird-Atlas-of-Kenya)
- [Historical Bird Atlas and Contemporary Citizen Science Data — Nussbaumer et al. 2025](https://onlinelibrary.wiley.com/doi/10.1111/ddi.13935)
- [Birds4Africa — African bird atlases summary](https://birds4africa.org/2020/03/20/african-bird-atlasses/)
- [Israel Breeding Bird Atlas (eBird)](https://ebird.org/atlasilps/about)
- [BODATSA / POSA (SANBI, BRAHMS Online)](https://posa.sanbi.org/sanbi)
- [SANBI BiodiversityGIS (BGIS)](https://bgis.sanbi.org/)
- [South African Protected Areas Register](https://egis.environment.gov.za/protected_areas_register)
- [CREW Programme — SANBI](https://www.sanbi.org/biodiversity/building-knowledge/biodiversity-monitoring-assessment/custodians-of-rare-and-endangered-wildflowers-crew-programme/)
- [Protea Atlas Project](https://www.proteaatlas.org.za/)
- [Protea Atlas Project II discussion (iNaturalist)](https://www.inaturalist.org/posts/72823-time-to-start-thinking-about-protea-atlas-project-ii)
- [Namibian Biodiversity Database](https://www.biodiversity.org.na/)
- [Tree Atlas of Namibia](https://treeatlas.biodiversity.org.na/)
- [NACSO — State of Community Conservation reports](https://nacso.org.na/)
- [TFCA Portal (SADC)](https://tfcaportal.org/)
- [KAZA Elephant Survey 2022 — technical report](https://www.researchgate.net/publication/373555995_KAZA_Elephant_Survey_2022_Volume1_Results_and_Technical_Report)
- [Flora of Zimbabwe](https://www.zimbabweflora.co.zw/)
- [Flora of Zambia](https://www.zambiaflora.com/)
- [Zambian Carnivore Programme](https://www.zambiacarnivores.org/)
- [SEOSAW plot network](https://seosaw.github.io/)
- [Gorongosa — Biodiversity Exploration](https://gorongosa.org/biodiversity/)
- [E.O. Wilson Biodiversity Foundation — Gorongosa](https://eowilsonfoundation.org/places-hef/gorongosa-national-park/)
- [Amboseli Elephant Research Project (Wikipedia)](https://en.wikipedia.org/wiki/Amboseli_Elephant_Research_Project)
- [Amboseli.org — 50-year conservation chronicle](https://amboseli.org/population-of-elephants-amboseli/)
- [Mara Predator Conservation Programme](https://www.marapredatorconservation.org/)
- [MPCP Q1 2025 lion & cheetah figures](https://www.marapredatorconservation.org/wp-content/uploads/2025/05/MPCP-Q1-2025-Technical-Report-web-15.05.2025.pdf)
- [Save the Elephants — Samburu Elephant Project](https://savetheelephants.org/our-work/science/behaviour-society/samburu-elephant-project/)
- [Northern Rangelands Trust — Status of Wildlife Report 2005-2019](https://www.nrt-kenya.org/s/NRT-Status-of-Wildlife-Report-2020.pdf)
- [Chepkitale Indigenous Peoples' Development Project — Mapping](https://chepkitale.org/projects/mapping/)
- [Ogiek Peoples' Ancestral Territories Atlas (PDF)](https://www.iapad.org/wp-content/uploads/2016/09/HL07_Ogiek_Peoples_Ancestral_Territories_Atlas.pdf)
- [National Museums of Kenya — Botany Department](https://museums.or.ke/botany-department/)
- [National Museums of Kenya JRS grant, 2018](https://jrsbiodiversity.org/grants/national-museums-of-kenya-2018/)
- [Mpala Research Centre — Flora and Fauna (old subdomain)](https://old.mpala.org/Flora_and_Fauna.php)
- [Serengeti Biodiversity Program Annual Report 2023-2024](https://serengeti-forever.org/wp-content/uploads/2025/01/serengeti-biodiversity-report-web.pdf)
- [Snapshot Serengeti data paper (Nature Scientific Data, 2015)](https://www.nature.com/articles/sdata201526)
- [AfricanBioServices project](https://africanbioservices.eu/)
- [Ethiopian Wolf Conservation Programme](https://www.ethiopianwolf.org/)
- [IGCP — Mountain Gorilla census history](https://igcp.org/about-us/history/)
- [Bwindi-Sarambwe mountain gorilla census launch, May 2025](https://greatervirunga.org/mountain-gorilla-census-launched-for-bwindi-and-sarambwe-ecosystem-on-the-6-may-2025/)
- [REBIOMA Data Portal — BISS paper](https://biss.pensoft.net/article/25864/)
- [REBIOMA main site](https://www.rebioma.org/index.php/en/biodiversite/portal)
- [Seychelles Islands Foundation — Aldabra](https://www.sif.sc/aldabra)
- [Aldabra tortoise habitat/drought paper](https://www.sciencedirect.com/science/article/abs/pii/S1470160X17302844)
- [National Center for Wildlife (Saudi Arabia) — Open Data](https://www.ncw.gov.sa/en/open-data)
- [Abu Dhabi Species Portal](https://adnature.ead.ae/index.html)
- [IranVeg — Vegetation Database of Iran](https://vcs.pensoft.net/article/114081/)
- [Nature Iraq — Key Biodiversity](http://www.natureiraq.org/key-biodiversity.html)
- [RSCN — National Biodiversity Database](https://www.rscn.org.jo/national-biodiversity-database?lang=en)
- [Jordan Times — Online database on Kingdom's fauna and flora, 2018](https://jordantimes.com/news/local/online-database-kingdom%E2%80%99s-fauna-and-flora-available-public)
- [SPNL — Mapping Lebanon's Green Heart](https://www.spnl.org/mapping-lebanons-green-heart-a-landmark-flora-and-biodiversity-map/)
- [African Lion Database Project (ALD)](https://www.africanliondatabase.org/)
- [African Elephant Database](https://africanelephantdatabase.org/)
- [RhODIS — University of Pretoria](https://www.up.ac.za/research-matters/news/rhodis-rhino-dna-barcoding-rhino-dna-database-puts-poachers-cross-hair)
- [GiraffeSpotter — Wildbook for Giraffe](https://giraffespotter.org/)
- [Range-Wide Conservation Program for Cheetah and African Wild Dogs](https://cheetahandwilddog.wordpress.com/)
- [Cheetah Conservation Initiative (successor site)](https://cheetahconservationinitiative.com/)
- [CORDIO East Africa Data Portal](https://cordio-data-portal-cordioea.hub.arcgis.com/)
- [WIOMSA — Coral Triangle project](https://wiomsa.org/researchsupport/phase-iii-is-there-a-western-indian-ocean-coral-triangle/)
- [Red Sea Atlas — Living Oceans Foundation](https://www.livingoceansfoundation.org/the-red-sea-atlas/)
## North American landscape/place documentation projects — survey for the Sathyamangalam atlas

Scope: USA, Canada, Mexico, Caribbean. All liveness checks performed **2026-08-25** via WebFetch unless a different date is given. `curl` was blocked by the environment's proxy policy, so "dead" claims rest on WebFetch/DNS failures and on-page staleness evidence, which I state explicitly. Where I could not confirm team size, funding, or counts I write **unverified** rather than guess.

Excluded as instructed: ALA, NBN Atlas, GBIF, Minnesota Biodiversity Atlas (mentioned only as a Symbiota sibling), India Biodiversity Portal.

---

### United States

#### National / multi-state (cross-cutting infrastructure most relevant to your build)

| Project (URL) | Scale | Team size & type | Scope (occurrence-only vs multi-domain — domains) | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [NPSpecies](https://irma.nps.gov/NPSpecies/) | Every NPS unit (~400+), park-by-park | Federal program, NPS Inventory & Monitoring; size unverified | Multi-domain: park species checklists + **evidence records** (vouchers, observations, reports), residency/abundance status, park-specific literature citations | All taxa per park | Live; part of IRMA portal, actively published to GBIF as "NPS - NPSpecies - Park Species Lists" | **The closest US analogue to a single-reserve atlas.** Each park×species row carries an *evidence type* and *park status* field, so contradictory claims (e.g. "present" from a 1970s report vs "unconfirmed") are stored as separate evidence with provenance rather than collapsed |
| [FWS ECOS / ECOSphere + IPaC](https://ecos.fws.gov/ecp/) | National, county-resolvable | Federal (USFWS) | Multi-domain: listed species, **critical habitat designations**, section 7 biological opinions, recovery plans, delisting/reclassification records, contaminants samples (>100,000) | Legally protected species & habitat | Live | The model for a **place-keyed legal-instrument register**: query a project polygon → official species list + the instruments that attach. Directly analogous to your legal/government instruments layer |
| [USGS GNIS](https://www.usgs.gov/us-board-on-geographic-names/download-gnis-data) | National gazetteer | US Board on Geographic Names + USGS | Gazetteer: official names, feature class, coordinates, variant names, decision history | Place names | Live; bulk downloads maintained | **Variant-name and decision handling** is the transferable pattern — GNIS records official vs variant/historical names and BGN decisions, i.e. an authority file that preserves the dispute |
| [Newberry Atlas of Historical County Boundaries](https://digital.newberry.org/ahcb/) | US counties 1629–2000; states/territories 1783–2000 | Newberry Library project; team size unverified | Multi-domain: GIS shapefiles + **chronologies compiled from statutes and court decisions** + metadata per state | Administrative/legal geography | Live and downloadable; project itself completed (no new boundary work observed) | **Legal-instrument mining done properly**: every boundary change is footnoted to its enabling statute. If you are reconciling notification boundaries for Sathyamangalam over time, this is the reference implementation |
| [Flora of North America](https://floranorthamerica.org/Main_Page) | Continental (N of Mexico) | Large distributed editorial network; exact size unverified | Multi-domain: taxonomic treatments, keys, descriptions, distribution, discussion of taxonomic disagreement | Vascular plants + bryophytes | Live wiki + [efloras.org](http://www.efloras.org/flora_page.aspx?flora_id=1) delivery; volume programme ongoing | Treatments include explicit "taxonomic dispute" discussion — a **prose-level conflicting-data convention** |
| [BONAP / North American Plant Atlas](http://www.bonap.org/) | Continental, county-level | Effectively a very small core (John T. Kartesz as director — **unverified from the site itself**) + volunteer contributors | Multi-domain: taxonomic data centre + county distribution maps + floristic tension-zone and density maps | Vascular plants | **Stale.** Page states the 2014 edition (dated 11/17/2014) is the most recent major release | Site explicitly states "not every datum is accurate" and solicits corrections — honest uncertainty framing. Near-solo continental atlas; a cautionary case in single-person dependency |
| [BAMONA](https://www.butterfliesandmoths.org/) | US + Canada + Mexico | Operated by Metalmark Web and Data (small commercial team) + volunteer **regional coordinators** | Occurrence-heavy + species profiles, images, checklists by state/county | Butterflies & moths | **Very active** — 1,004,053 verified sightings, 9,392 species profiles, 13,101 gallery images, 93,357 members; newest sightings dated 2026-08-24/25 | **Expert-verification gate before a record enters the database.** The verified/unverified split is exactly the mechanism your disputed-figures feature needs at record level |
| [OdonataCentral](https://www.odonatacentral.org/) | Western Hemisphere | Birds in the Hand, LLC; support from NSF and Alabama Museum of Natural History | Occurrence + checklists (North America checklist 2024, Canada checklist April 2024), identification | Odonata | Live; "Odolympics" scheduled 13–23 June 2026. Site copyright notice reads 2019 — copyright is stale, content is not | Regional checklists are versioned by year — a lightweight way to publish "the count as of date X" |
| [SEINet Portal Network](https://swbiodiversity.org/seinet/index.php) | Originally AZ/NM, now continental | Symbiota core team + hundreds of contributing collections | Multi-domain: specimens, images, checklists, interactive keys, taxonomic thesaurus | Plants, bryophytes, fungi, lichens, pteridophytes | Live — **24 million records from 456 collections** | Hub of the Symbiota federation; shared central database behind many regional sites below. Annotation/determination history preserves competing identifications |
| [NatureServe Explorer](https://explorer.natureserve.org/) / [Explorer Pro](https://explorer.natureserve.org/pro/) | US + Canada | NatureServe central staff + 60-program network | Multi-domain: conservation status ranks (global/national/subnational), distribution, ecosystem classification | Species + ecosystems of conservation concern | Live (Explorer 2.0). Explorer Pro launched for western US in July 2024 | **Three-tier rank system (G/N/S) is a structural way to hold disagreeing assessments side by side** — global rank can differ from a state rank for the same taxon |
| [LandScope America](http://www.landscope.org/) | National + Chesapeake | NatureServe + National Geographic Society | Multi-domain: maps, data, photos, narrative "stories" of places | Conservation lands & biodiversity | **Dormant.** Loads, but footer reads "Copyright © 2022 NatureServe"; LandScope Chesapeake still frames a 2025 target as future | Was the flagship "atlas of a place, many media types" attempt in the US NGO space. Instructive on maintenance failure modes |
| [iMapInvasives](https://www.imapinvasives.org/) | Multi-state | Hosted by NatureServe; state partners (Cornell IPM, NYSDEC, NYNHP named) | Occurrence + treatment/management records | Invasive species | Live; **only 5 participating jurisdictions listed at fetch (AZ, ME, NY, OR, PA)** — no Canadian provinces listed, despite earlier provincial participation (contraction, **unverified** cause) | A shared-instance model: one codebase, per-jurisdiction tenancy |
| [Whaling History](https://whalinghistory.org/) | Global voyages, US-hosted | Mystic Seaport Museum + New Bedford Whaling Museum + Nantucket Historical Association | **Strongly multi-domain**: 9 interlinked databases — voyages, logbooks (~1,000 scanned), crew lists, cargo, plus the Dennis Wood Abstracts (>6,700 American voyages) | Historical whaling (place/voyage-based) | Active — British Southern Whale Fishery datasets updated with 900 amended/new oil-cargo entries, January 2025 | **Best US exemplar of historical document mining feeding a structured database.** "Amended entries" as a first-class update type is worth copying |
| [World Historical Gazetteer](https://whgazetteer.org/) | Global, US-hosted | Institute for Spatial History Innovation, Univ. of Pittsburgh; led by Ruth Mostern | Gazetteer + reconciliation service: 2,236,719 indexed places; index of 47M places / 67M toponyms; API, workbench, teaching materials | Historical place names | **Very active** — v3.2; moved to ISHI Sept 2025; blog post dated 2026-08-21 | **The single most useful technical model for your gazetteer.** Linked Places format, attestation-with-date-and-source per toponym, and explicit endonym/exonym treatment mean the same place can carry contradictory names from different eras and languages without one overwriting another |
| [SlaveVoyages](https://www.slavevoyages.org/) | Atlantic world, US-hosted (Rice; earlier Emory) | Multi-institution scholarly consortium; size unverified | Multi-domain: voyage database (~35,000 transatlantic voyages, 1501–1867), intra-American, African Origins names | A population, from archival sources | Live (homepage is JS-only; counts above from the [Wikipedia article](https://en.wikipedia.org/wiki/Trans-Atlantic_Slave_Trade_Database)) | **The canonical "documented vs estimated" split**: the site publishes documented voyages *and* a separate Estimates interface (≈12.5M embarked / 10.7M arrived) with the imputation method exposed. This is the pattern your disputed-figures feature should emulate |
| [Enslaved.org](https://enslaved.org/) | Atlantic world | MSU Matrix + Harvard Hutchins Center + UC Riverside; funded by Mellon and NEH | Multi-domain LOD: **731,500 people, 474,012 events, 10,381 places, 3,940 sources** | A population, from archival sources | Live | Every assertion is source-attached; contributions arrive as **citable datasets via the *Journal of Slavery and Data Preservation***. A publishing model for contributor credit that a community atlas could reuse |

#### Symbiota regional/state consortia (US) — one codebase, many place-scoped portals

Existence of each verified from the [Symbiota portal directory](https://symbiota.org/symbiota-portals/) (fetched 2026-08-25). Record counts verified only where noted.

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Consortium of Pacific Northwest Herbaria](https://www.pnwherbaria.org/) | WA/OR/ID/MT/BC/AK | Managed by UW Herbarium, Burke Museum; NSF-supported | Specimens + images + label data | Plants, bryophytes, lichens, fungi, algae | Live — **3,328,306 records, 1,800,701 images, 55 herbaria**; footer "© 2007-2026" | Runs its own stack (not Symbiota), a useful counter-example on independence vs shared infrastructure |
| [SERNEC](https://sernecportal.org/portal/index.php) | 14 southeastern states | 230 participating herbaria; NSF Award 1410069 | Specimens, images, georeferencing | Plants | Live | Largest US regional herbarium network by institution count |
| [Consortium of Midwest Herbaria](https://midwestherbaria.org/) | Midwest states | Consortium | Specimens, images, checklists, inventory projects | Plants | Live (homepage blocks automated fetch via robots.txt; collections search reachable) | Hosts named "Inventory Projects" — place-scoped checklists inside a regional portal, a good pattern for a reserve inside a state atlas |
| [Consortium of California Herbaria (CCH2)](https://cch2.org/portal/index.php) | California | Consortium (Berkeley-anchored) | Specimens, images, annotations | Plants | Live | Annotation history retains superseded determinations |
| [Consortium of Northeastern Herbaria](https://neherbaria.org/) | NE states | Consortium | Specimens, images | Plants | Live | — |
| [Consortium of Southern Rocky Mountain Herbaria (SoRo)](https://www.soroherbaria.org/portal/) | Southern Rockies | Consortium | Specimens, images, checklists | Plants | Live | — |
| [Intermountain Herbaria (Intermountain Biota)](https://intermountainbiota.org/portal/collections/index.php) | Great Basin / Intermountain West | Consortium | Specimens, images | Plants | Live | — |
| [Mid-Atlantic Herbaria](https://midatlanticherbaria.org/portal/) | Mid-Atlantic | Consortium | Specimens, images | Plants | Live | — |
| [TORCH](https://portal.torcherbaria.org/portal/index.php) | Texas + Oklahoma | Consortium | Specimens, images | Plants | Live | — |
| [Northern Great Plains Herbaria](https://ngpherbaria.org/portal/) | Northern Great Plains | Consortium | Specimens, images | Plants | Live | — |
| [Four Corners Plants](https://fourcornersplants.org/portal/index.php) | Four Corners region | Consortium | Specimens, checklists | Plants | Live | Bioregional rather than political-boundary scoping — relevant if your atlas is landscape- not district-scoped |
| [North American Network of Small Herbaria](https://nansh.org/portal/index.php) | Continental | Network of small collections | Specimens, images | Plants | Live | **Explicitly built for institutions too small to run their own portal.** The most directly transferable governance model for small Indian herbaria/collections |
| [California Islands Biodiversity Information System (Cal-IBIS)](http://www.cal-ibis.org/) | Channel Islands | Consortium; size unverified | Specimens, checklists | Island biota | Listed as live in Symbiota directory; liveness not independently fetched | Single-archipelago all-taxa scoping |
| [Indian River Lagoon Species Inventory](https://irlspecies.org/index.php) | One 156-mile estuary | Curated by Smithsonian (Marine Station, Fort Pierce lineage) | Multi-domain: **11,150 species reports, 12,361 taxa, 633,656 occurrence records**, species accounts, imagery, habitat/threat/stewardship topics | All taxa of one place | Live | **The best single-place US analogue to your atlas**: narrative species accounts + occurrence records + habitat and stewardship sections for one named landscape |
| [BioGator](https://biogator.org/index.php) | Florida | Univ. of Florida collections | Specimens across taxa | Multi-taxon | Live per directory | Multi-taxon (not plants-only) Symbiota instance |
| [Flora of Wisconsin (WisFlora)](https://wisflora.herbarium.wisc.edu/index.php) | Wisconsin | UW-Madison Herbarium | Specimens, images, county maps | Plants | Live | — |
| [Illinois Natural History Survey Biocollections](https://biocoll.inhs.illinois.edu/portal/index.php) | Illinois | INHS / Univ. of Illinois | Specimens, multi-taxon | Multi-taxon | Live | — |
| [Kansas Biodiversity](https://ks.symbiota.org/portal/) | Kansas | Kansas institutions | Specimens, checklists | Multi-taxon | Live per directory | — |
| [Univ. of Colorado Museum Herbarium (COLO)](https://botanydb.colorado.edu/index.php) | Colorado | Single institution | Specimens | Plants | Live per directory | — |
| [OregonFlora](https://oregonflora.org/) | Oregon | OSU Herbarium team | Multi-domain: ~4,700 vascular species, specimens, images, checklists, ~200 native garden plant recommendations | Plants | Live; latest dated events on site were 2024 (wildflower shows) — **content freshness weaker than infrastructure** | Combines a scientific flora with a public horticulture layer — a "who else is this for" lesson |
| [Madrean Discovery / MABA](https://madreandiscovery.org/) | **Binational** Sky Islands, AZ/NM/Sonora/Chihuahua | Consortium; leads and staffing **unverified** (about-page blocked by robots.txt) | Flora + fauna databases, expedition-based observation records | Multi-taxon, one bioregion | Portal loads; counts unverified | Border-spanning bioregional atlas — relevant precedent for a reserve straddling state boundaries (Sathyamangalam/Karnataka edge) |

#### State-by-state (US)

##### Alaska

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Alaska Center for Conservation Science](https://accs.uaa.alaska.edu/) | Statewide + Arctic | UAA research centre; hosts the Alaska Natural Heritage Program | Multi-domain: five research programmes; portals for vegetation maps (akveg.org), non-native plants (AKEPIC), stream temperatures (AKTEMP), aquatic invasives, terrestrial animal ranges (Biotics) | Species, vegetation, invasives | Live; footer copyright 2025 | Deliberately **decomposed into separate thematic portals** rather than one monolith — a real architectural fork in the road for you |
| [Alaska Native Place Names Project](https://akplacenames.org/) | Statewide, multilingual | Small academic project; Gary Holton associated ([project page](https://gmholton.github.io/research/akplacenames)); staffing/funding unverified | Gazetteer: multilingual Indigenous place names + ecological knowledge linkage | Place names | **Dormant.** Site loads; most recent dated content on the landing page is a 2016 photo credit; no news/updates | Aim was "an authoritative statewide database of Indigenous place names" — an unfinished ambition worth studying before you commit to a gazetteer scope |
| [AOOS / Arctic Marine Biodiversity Observation Network](https://aoos.org/project/arctic-marine-biodiversity-observation-network/) | Chukchi/Beaufort shelf | NOAA/NOPP-funded consortium | Occurrence + oceanographic; archived at NCEI ([AMBON dataset](https://www.ncei.noaa.gov/access/metadata/landing-page/bin/iso?id=gov.noaa.nodc:NOPP-AMBON-US), 2015–2020) | Marine biodiversity | Project page live; the flagship data collection covers 2015–2020 | Regional marine BON with a formal archival endpoint — the "where does the data live after the grant" question answered |

##### Alabama

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Alabama Plant Atlas](https://alabama.plantatlas.usf.edu/) | Statewide | Alabama botanists + USF Water Institute hosting | Specimens, county maps, images, nomenclature | Plants | Site exists on the PlantAtlas.org platform; my fetch of `alabama.plantatlas.usf.edu` and `floraofalabama.org` both failed (DNS/robots and an SSL hostname mismatch on `www.floraofalabama.org`) — **liveness unverified** | Part of the **PlantAtlas.org shared-software family** (see Florida, New York) |

##### California

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Calflora](https://www.calflora.org/) | Statewide | Independent 501(c)(3), Berkeley; small staff (size unverified) | Occurrence-heavy + tools: **3.1M+ observations, 8,000+ plants**, 150+ curated viewing locations, mobile apps | Plants | Live; "over 20 years" of operation | Independent nonprofit sustaining a state-scale occurrence database outside any university — a viable governance model for a community atlas |
| [Jepson eFlora](https://ucjeps.berkeley.edu/eflora/) | Statewide | University & Jepson Herbaria; named editorial board (Baldwin convening ed., Rosatti scientific ed., Keil, Markos, Mishler, Patterson, Wilken) | Multi-domain: treatments, keys (KeyBase), maps, illustrations, index of accepted names **and synonyms** | Vascular plants | **Revision 11, posted 2022-12-23** — versioned, not continuously edited | **Numbered revisions with dates** is the cleanest citation model here: a reader can cite "as of Revision 11", which matters enormously when figures are disputed |
| [SFEI Historical Ecology programme](https://www.sfei.org/programs/rl/historical-ecology) | Watershed-by-watershed (Napa, Sacramento Valley, Hidden Nature SF, and more) | San Francisco Estuary Institute programme team | **Heavily multi-domain**: historical maps, land surveys, diaries, photographs, oral accounts → reconstructed baseline GIS layers; published as atlases and as a [GIS data compilation](https://www.sfei.org/data/historical-ecology-gis-data-compilation) | Historical landscape ecology | Live; long-running programme with many project pages | **The gold standard for archival→spatial baseline reconstruction.** Their published method (certainty ratings on interpreted features, source-by-source corroboration) is the single most directly reusable technique for your historical-document layer |
| [San Diego Natural History Museum atlas projects](https://www.sdnhm.org/science/atlas-projects/) | San Diego County + peninsular Baja | Museum-led, volunteer-heavy | Four separate atlases: Amphibian & Reptile Atlas of Peninsular California (online database), San Diego County Bird Atlas (book, 2004), [Plant Atlas](https://www.sdnhm.org/science/botany/projects/plant-atlas/) (from 2003, online database), Mammal Atlas | Multi-taxon, county-scale | Programme page live; publication states are mixed (bird atlas is a 2004 book with only partial online data; mammal atlas outputs described as book+CD) | **The clearest illustration of the book-vs-portal trap.** Plant Atlas reports concrete yield: 300+ new county records, 10 new state records, 2 new taxa — the kind of number that justifies a county atlas to funders |
| [KRIS Web](https://www.krisweb.com/) | 14 Northern California watersheds + Sheepscot (ME) + Kootenai (ID/BC) | Kier Associates with Institute for Fisheries Resources | **Multi-domain**: maps, data tables, charts, photographs, and per-watershed [bibliographies](https://www.krisweb.com/biblio/biblio_klamath.htm) | Salmonids, watershed condition | **Stale.** Loads, but footer copyright reads 2011; distribution model still mentions free CDs | Explicitly built as "one place-based information system per watershed," bibliography included. Architecturally the closest thing to your brief; also a warning about a decade of no updates |

##### Colorado

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Colorado Breeding Bird Atlas II](https://www.cobreedingbirdatlasii.org/) | Statewide | Colorado Bird Atlas Partnership + Colorado Parks and Wildlife; volunteer-based | Occurrence + breeding-status codes; queryable database | Breeding birds | Site live with working database queries; content frozen — book published Nov 2016 (744 pp., ed. Lynn E. Wickersham), footer copyright 2016 | Book + queryable site in parallel; typical atlas end-state |
| [Colorado Natural Heritage Program](https://cnhp.colostate.edu/aboutus/the-natureserve-network) | Statewide | Colorado State Univ.; NatureServe Sustaining Constituent Member | Element occurrences, status ranks, reports | Species & communities of concern | Live | — |

##### Connecticut

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Connecticut Bird Atlas](https://ctbirdatlas.org/) | Statewide | UConn-led with CT DEEP; volunteer network (**unverified** counts) | Breeding + winter season atlas | Birds | **Liveness unverified** — my fetch failed on a TLS/robots error, not a 404, so the host responded but could not be read | Notable design: a **combined breeding *and* non-breeding season atlas**, which forces explicit seasonality on every record |

##### Florida

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Atlas of Florida Plants](https://florida.plantatlas.usf.edu/) | Statewide | USF Institute for Systematic Botany (est. 1990) + USF Water Institute; named originators R. P. Wunderlin, B. F. Hansen, A. R. Franck, F. B. Essig; current contacts Chris Kiahtipes, Anita Crnjac, Alan Franck | Multi-domain: **4,920 species (3,320 native / 1,600 non-native)**, 243,240 digitized specimens, 19,025 photographs, county maps, nomenclature, literature citations | Plants | **Active** — redesigned site launched 2025-03-02 | Native/non-native counts are reported *separately and explicitly* — a small but important precedent for surfacing contested status rather than a single total |
| Indian River Lagoon Species Inventory | see Symbiota table above | | | | | |

##### Georgia

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Georgia Biodiversity Portal](https://georgiabiodiversity.org/) | Statewide | Georgia DNR Wildlife Resources Division | Multi-domain: species accounts across ~14 taxon groups, natural plant communities, protected-species and SWAP/SGCN status groupings, environmental review requests | Species + communities | Live; total counts not published on the landing page | **Combines the species atlas with the regulatory workflow** (environmental review request in the same site) — the same fusion your legal-instruments feature implies |

##### Hawaii

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Hawaii Biological Survey](https://www.bishopmuseum.org/hbs/) | Statewide (terrestrial, freshwater, marine) | Bishop Museum; **statutory mandate from the Hawaii State Legislature, 1992** | Multi-domain: collections (~4M specimens), checklists, and the annual *Records of the Hawaii Biological Survey* since 1994 | All native and non-native fauna and flora | Live; the *Records* series is the ongoing output mechanism | **~25,000 species total (19,000 terrestrial / 500 freshwater / 5,500 marine).** The legislated mandate is the standout feature: a legal instrument creating the atlas. Also the best model for **an annual, citable "new records" publication** as the update channel — every added or corrected record becomes a citable act |

##### Idaho

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Digital Atlas of Idaho](https://www.isu.edu/digitalatlas/) (also [digitalatlas.cose.isu.edu](https://digitalatlas.cose.isu.edu/aboutus/sref2.htm)) | Statewide | Idaho State Univ.; authorship, funding and dates **not stated on the site** | **Genuinely multi-domain**: Geology, Biology (butterflies, dragonflies, amphibians, reptiles, birds, mammals, plants), Archaeology, Geography, Hydrology, Climatology, Idaho Maps | Whole-state natural and cultural history | Live but **apparently frozen**; no dates or update notices anywhere I could reach. An [Internet Archive copy](https://archive.org/details/digital-atlas-of-idaho) exists, which is itself a preservation signal | **The nearest US structural match to your Sathyamangalam brief** — one place, seven domains, one site, including archaeology alongside biology. Also a warning: no visible provenance or update metadata makes it uncitable and unmaintainable |

##### Illinois

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Illinois Wildflowers](https://www.illinoiswildflowers.info/) | Statewide | **Solo** — Dr. John Hilty | Multi-domain: habitat-organised species pages plus dedicated databases of **plant-feeding insects, flower-visiting insects, and vertebrate–plant interactions**, all literature-derived | Plants + their faunal associations | **Frozen.** Copyright reads "© 2002-2020 by John Hilty" | The best US example of a **one-person site of genuine scholarly quality**. Its faunal-association databases are effectively a literature-mining product, and its stall in 2020 shows the succession risk |

##### Kansas

Covered in the Symbiota table ([Kansas Biodiversity](https://ks.symbiota.org/portal/)).

##### Massachusetts

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Boston Harbor Islands ATBI](https://www.nps.gov/boha/learn/nature/atbi.htm) ([Farrell Lab page](https://farrell.oeb.harvard.edu/boston-harbor-islands-all-taxa-biodiversity-inventory)) | One 34-island park | Harvard MCZ (Farrell Lab) + NPS + Boston Harbor Islands Partnership | Occurrence + specimen-backed inventory; bioblitz events 2005, 2006, 2008; published in *Northeastern Naturalist* 2018 | Terrestrial invertebrates | **Dormant.** NPS page "Last updated: October 25, 2019"; the project's own MantisWeb database host (`bhi.oeb.harvard.edu`) failed DNS resolution on my fetch | **A dead custom portal behind a live published paper** — precisely the failure mode you were asked to hunt. The science survives in print; the interface does not |
| [Massachusetts Butterfly Club](https://www.massbutterflies.org/) | Statewide | NABA chapter; volunteer; named Records Compiler Mark Fairbrother | Occurrence: **200,000+ sighting records**, flight-date charts, ID tools, MassLep listserver | Butterflies | **Active** — meetings scheduled 2026-04-11 and 2026-10-17 | Long-running volunteer records database with a single named compiler as the quality gate. (Sharon Stichter's historically-grounded Massachusetts species accounts are associated with this club; the specific species-accounts URL I tried returned 404, so **treat the historical-accounts component as unverified**) |
| Massachusetts NHESP | statewide | MassWildlife | element occurrences, regulatory habitat | rare species | not independently fetched — **unverified** | — |

##### Maryland

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Maryland Biodiversity Project](https://www.marylandbiodiversity.com/) | Statewide | 501(c)(3) nonprofit; volunteer-driven (founders **unverified from the site**, though widely credited to Bill Hubick and Jim Brighton) | Multi-domain: **22,466 species, 1,470,100 records, 1,583,299 photos**, county-level checklists, species accounts, [documented record standards](https://www.marylandbiodiversity.com/about/records.php) | All taxa, statewide | **Extremely active** — homepage announcement dated 2026-08-23 adding *Heliophanus kochii* as a new state spider record | **The single best model for your project.** All-taxa, volunteer-run, nonprofit, with a public "About Records" page defining what counts as a record. The daily new-species announcements make the growth of the atlas legible to its community |

##### Michigan

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Michigan Natural Features Inventory](https://mnfi.anr.msu.edu/) | Statewide | MSU Extension; NatureServe Leadership Constituent Member | Multi-domain: Natural Heritage Database, rare species and [natural community plant lists](https://mnfi.anr.msu.edu/communities/plant-lists), county element data, biological rarity index, rare species review service | Species + natural communities | Live; active volunteer programmes referenced | The **natural-community classification with per-community plant lists** is a layer your atlas may want: habitat as a first-class documented entity, not just a species attribute |
| [Michigan Flora Online](https://michiganflora.net/) | Statewide | Univ. of Michigan Herbarium | Taxa, county maps, images | Plants | Host responds but the site is a **JavaScript-only application** — no content readable without JS; counts and dates **unverified** | JS-only rendering makes a resource invisible to crawlers, archivers and text-based agents. Avoid this if you want your atlas cited and archived |

##### Minnesota

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Minnesota Wildflowers](https://www.minnesotawildflowers.info/) | Statewide | **Effectively solo** — Katy Chayka (former MN Master Naturalist volunteer) with contributors | Species pages, 18,000+ photographs, [plant search](https://minnesotawildflowers.info/page/search) | Plants — 1,800+ of the state's 2,100+ species | **Active** — copyright "2006-2026"; 2025 Annual Report downloadable; recent fern/moonwort additions listed | A near-solo project that publishes an **annual report** — unusually good accountability for a one-person site, and the reason it has survived 20 years |
| [Mapping Prejudice](https://mappingprejudice.umn.edu/) | Minneapolis, expanding nationally | Univ. of Minnesota Libraries + large volunteer transcription corps | **Historical document mining**: property deeds → racial covenants → maps; twice-monthly community mapping sessions | Legal instruments in the built environment | Live; February 2025 imagery of active sessions | **Crowdsourced legal-document mining at scale**, with the volunteer session as the engine. If your atlas needs to mine gazette notifications or land records, this is the workflow to copy |

##### Montana

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Montana Field Guide](https://fieldguide.mt.gov/) | Statewide | Montana Natural Heritage Program + NatureServe + Montana Fish, Wildlife & Parks; hosted by Montana State Library | **Multi-domain**: species accounts for animals, plants, fungi/lichens, **ecological communities**, and invasive/pest species; identification, distribution, status, ecology; photo galleries; downloadable custom PDF guides; Map Viewer and Species Snapshot tools | All taxa + communities | Live; footer "© 1997–2026 Montana State Library" | **Best-in-class US state portal.** Two features worth stealing: user-composed **custom PDF field guides** generated from the database, and treating ecological communities as peer entities to species |

##### North Carolina

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [NC Biodiversity Project](https://nc-biodiversity.com/) | Statewide, multi-taxon family of sites | Small mixed group — "professional biologists, science educators, conservationists, nature photographers, and amateur naturalists"; partnerships formalised 2016 with NC Division of Parks and Recreation, Southern Conservation Partners, NC Natural Heritage Program. **No named leads on the site** | Multi-domain family: per-taxon atlas websites (odonates, butterflies, moths, mammals, grasshoppers, millipedes) **plus a new [Habitats of North Carolina] site** | Multi-taxon, statewide | **Very active** — Habitats of NC website launched 2026-04-21; odonate publications Nov 2025; *Apheloria* millipede revision Oct 2025. Drupal-based | **The federated-sites model**: one umbrella project, one site per taxon, plus a separate habitats site. If Sathyamangalam's taxa have different expert communities, this decomposition is worth serious consideration. Note the 2016 public/private partnership as the funding/legitimacy mechanism |

##### New York

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [New York Flora Atlas](https://newyork.plantatlas.usf.edu/) | Statewide | New York Flora Association; led by David Werier and Kyle Webster, with Troy Weldy, Andrew Nelson and others; hosted by USF Water Institute | Multi-domain: **4,112 species, 169,815 herbarium records, 8,070 images**, habitat and ecological-community notes, taxonomy | Plants | **Active — "Data last modified: 7/5/2026"** | A displayed, machine-readable **data-last-modified date** — trivially cheap, and the thing most atlases omit. Same PlantAtlas.org software as Florida and Alabama |
| [New York Natural Heritage Program](https://www.nynhp.org/) | Statewide | SUNY ESF in partnership with NYSDEC | Multi-domain: rare species and ecological community databases, Conservation Guides, iMapInvasives, iNaturalist projects, GIS/status-list data portal, ~75,000 acres of mapped old-growth forest | Species + communities | Live; active projects (Dark Skies for Fireflies) | Conservation Guides = per-element narrative synthesis sitting on top of the occurrence database |
| [NY Breeding Bird Atlas legacy application](https://extapps.dec.ny.gov/cfmx/extapps/bba) ([DEC hub](https://dec.ny.gov/nature/animals-fish-plants/birds/breeding-bird-atlas), [past data](https://dec.ny.gov/nature/animals-fish-plants/birds/breeding-bird-atlas/past-data-maps)) | Statewide | NYSDEC | Occurrence + breeding codes; **Atlas 1 (1980–1985) and Atlas 2 (2000–2005) with side-by-side comparison tools** | Breeding birds | **Live but frozen** — page states "Data available on this website were finalized in March 2007". A third atlas is being run through eBird | **Directly relevant to your disputed-figures feature**: the interface's core function is showing two surveys of the same square disagreeing, without adjudicating. Also a live example of a legacy ColdFusion app outliving its data |

##### Ohio

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Ohio Odonata Society / Ohio Dragonfly Survey](https://www.ohioodonatasociety.org/) | Statewide | Volunteer society | Occurrence: distribution, flight season, state records — **data collection runs on iNaturalist**, the society site is the interpretive layer | Odonata | Site live; counts, years and leads not on the landing page (**unverified**) | **Delegating collection to iNaturalist and keeping only synthesis** is the cheapest possible atlas architecture. Worth costing against building your own capture layer |

##### Oregon

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Oregon Explorer](https://hub.oregonexplorer.info/) (redirects from oregonexplorer.info) | Statewide, topic portals | OSU Libraries / Institute for Natural Resources lineage (**host attribution unverified from the current hub page**) | Multi-domain: natural-resource data, maps, tools across land, water, climate and communities | Landscape & natural resources | Live; **recently re-platformed** (302 from the legacy domain to an ArcGIS-Hub-style hub); no update date exposed | A cautionary re-platforming: the old topic-portal structure (with reports and historical material) is not visible on the new hub. **Migrations silently drop the non-spatial content first** |
| OregonFlora | see Symbiota table | | | | | |

##### Pennsylvania

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Pennsylvania Natural Heritage Program](https://www.naturalheritage.state.pa.us/) | Statewide | Four-agency partnership: PA DCNR, PA Fish & Boat Commission, PA Game Commission, Western Pennsylvania Conservancy, with USFWS; NatureServe member | **Multi-domain**: County Natural Heritage Inventories, plant factsheets, fungi and vascular plant checklists, **Conservation Explorer / PNDI environmental review**, Wildlife Action Map, Climate Change Vulnerability Index, community prediction tool | Species, communities, regulatory review | Live; 2025 Annual Report and current *Wild Heritage News* | **County Natural Heritage Inventories are the closest US administrative analogue to a district/reserve atlas** — a formal, repeatable, county-scoped publication series backed by the state database. Also fuses regulatory screening into the same site |

##### South Dakota

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [South Dakota Breeding Bird Atlas II](https://gfp.sd.gov/breeding-bird-atlas/) | Statewide | SD Game, Fish & Parks + volunteers | Occurrence + breeding codes | Breeding birds | Agency-hosted page; dates/counts **unverified** | Second-round atlas hosted inside an agency site rather than standalone — the low-maintenance option |

##### Texas

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Fishes of Texas](https://www.fishesoftexas.org/) | Statewide, all drainages | UT Austin Biodiversity Center (Hendrickson lab) hosted at TACC; named contributors Tomislav Urban (creator), Dean A. Hendrickson, Adam E. Cohen | Multi-domain: **895,000+ records from museum specimens, agency reports and citizen-science platforms**; georeferencing; CC BY-NC-SA licensed | Freshwater fishes | **Active — version 3.10 released 2026-03-23**, described as a major website and data upgrade | **The best North American exemplar of explicit data-conflict handling.** The project's whole reason for existing is that aggregated records disagree: it applies verification and georeferencing protocols to "correct historical misidentifications and spatial outliers," and it **versions the dataset** so a correction is dated and citable. Funded by a coalition (UT, TPWD, TCEQ, DOI/Great Plains LCC) — a realistic multi-agency funding pattern |
| TORCH | see Symbiota table | | | | | |

##### Vermont

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Vermont Atlas of Life](https://val.vtecostudies.org/) | Statewide | Vermont Center for Ecostudies; partners GBIF, eBird, iNaturalist, UVM Natural History Museum, Espace pour la vie, VT Fish & Wildlife | Multi-domain umbrella: occurrence data plus a family of sub-atlases — [Bumble Bee Atlas](https://val.vtecostudies.org/projects/bumble-bee-atlas/), [Bees of Vermont](https://val.vtecostudies.org/projects/vtbees/about/), and [wildlife atlases](https://val.vtecostudies.org/wildlife-atlases/) for birds, butterflies, reptiles and amphibians; maps, photographs, primary data | All taxa, statewide | Live; About page's most recent dated artefact is a 2021 conference poster, so **the About page is stale even though projects are running** | **The closest US structural sibling to your brief**: one branded state atlas acting as the umbrella for many taxon sub-atlases, all feeding a common data layer, published under one identity. Described in a paper, "The Vermont Atlas of Life: Discovering and Sharing Biodiversity Knowledge" |

##### Virginia

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Virginia Breeding Bird Atlas (2nd)](https://vabirdatlas.org/) ([DWR hub](https://dwr.virginia.gov/wildlife/virginia-breeding-bird-atlas/)) | Statewide | Virginia DWR + Virginia Society of Ornithology + Virginia Tech Conservation Management Institute; **~1,500 trained volunteers** plus staff | Occurrence + breeding evidence: **6M+ field observations**, 200+ species, 2016–2020 fieldwork | Breeding birds | **Very active** — results site launched Oct 2025; [Phase Two launched 2026-08-05](https://dwr.virginia.gov/blog/phase-two-of-the-atlas-website-has-launched/) adding 10 pages of content | Publishes **explicit first-vs-second-atlas change findings** (8 species newly confirmed as breeders, incl. Anhinga, Trumpeter Swan, Mississippi Kite, Painted Bunting). Web results delivered *in phases* after fieldwork — a realistic staging plan |
| [Digital Atlas of the Virginia Flora](https://vaplantatlas.org/) | Statewide | **Virginia Botanical Associates** — a small independent group, not a university; historically rooted in Alton M. Harvill Jr.'s work; funded partly by [Virginia Native Plant Society appeals](https://vnps.org/z02-past-admin/digital-atlas-fund-2024/) | Multi-domain: taxa browsable by family, genus and **county**; lycophytes/pteridophytes, gymnosperms, monocots, dicots, bryophytes; plus a maintained **"Excluded Taxa" document** | Plants | Live; site copyright 2026; no data-modified date exposed | **The "Excluded Taxa" list is exactly your disputed-figures feature in its simplest form** — a published, reasoned register of records the compilers rejected, kept visible instead of silently deleted. Cheap to implement, enormously valuable for trust |
| Virginia Natural Heritage Program | statewide | VA DCR; NatureServe Leadership member | element occurrences, natural community classification | species & communities | listed as a NatureServe Leadership Constituent Member; site not independently fetched — **unverified** | — |

##### Washington

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [WA DNR Natural Heritage Program](https://dnr.wa.gov/natural-heritage-program) | Statewide | WA DNR | Element occurrences, natural heritage plan, ecosystem classification | Species & ecosystems | Live | — |
| Consortium of PNW Herbaria | see Symbiota table | | | | | |

##### Wyoming

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Wyoming Natural Diversity Database](https://www.uwyo.edu/wyndd/) | Statewide | Univ. of Wyoming, Berry Biodiversity Conservation Center | Multi-domain: **6M+ species observation records**, Data Explorer, Wyoming Field Guide, reports & publications, photo gallery; 128,229 data requests fulfilled; 2,296 partner organisations | Species & habitats of concern | Live; page timestamp 2026-07-30. **But the latest quarterly statistic shows "0" new species records added in the most recent quarter** — worth flagging as a possible data-intake stall | **Publishes its own throughput metrics** (records added per quarter, requests fulfilled, partner count). That transparency is why the stall is visible at all — a practice worth adopting even though it exposes bad quarters |

---

### Canada

#### National

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Canadensys](https://www.canadensys.net/) | National | Université de Montréal + Canadian Museum of Nature, GBIF-affiliated; based Montréal | Multi-domain infrastructure: **3M+ specimen occurrences (Feb 2025 milestone), ~10,000 datasets, ~1,000 publishers, ~1,000 species lists**, the **VASCAN** vascular plant name database, IPT publishing, coordinate/date parsing tools | Plants, animals, fungi | Active — GBIF Metabarcoding Data Toolkit launched May 2025 | **VASCAN is the transferable piece**: a national name authority with accepted names, synonyms and per-province status — the taxonomic backbone that makes conflicting occurrence claims comparable |
| [Birds Canada breeding bird atlases](https://www.birdscanada.org/bird-science/breeding-bird-atlases) | National programme, provincial atlases | Birds Canada coordinating; provincial volunteer corps in the thousands (per-atlas figures unverified) | Occurrence + breeding evidence codes, point counts, effort hours | Breeding birds | Live hub. Completed since 2000: **BC** ([birdatlas.bc.ca](https://www.birdatlas.bc.ca/what-is-an-atlas/)), **Manitoba** (birdatlas.mb.ca), **Maritimes** (mba-aom.ca), **Québec** (atlas-oiseaux.qc.ca). In progress: **Ontario Atlas-3** (from 2021-01-01, 5 years — [birdsontario.org](https://www.birdsontario.org/)), **Newfoundland** ([nf.birdatlas.ca](https://nf.birdatlas.ca/about-the-atlas/)), **Saskatchewan** (sk.birdatlas.ca) | **The most mature multi-jurisdiction atlas programme in the world**, all delivered on the shared [NatureCounts](https://naturecounts.ca/nc/onatlas/main.jsp) platform. One platform, per-jurisdiction branding, common data model — a strong argument for building Sathyamangalam on infrastructure that can host the next reserve too |
| [Native Land Digital](https://native-land.ca/) ([map](https://native-land.ca/maps/native-land)) | North America + beyond | Indigenous-led nonprofit; small team (size unverified) | Gazetteer/territory mapping: Indigenous territories, languages, treaties | Indigenous geography | Live; widely redistributed (e.g. [NY State GIS Gateway](https://opdgig.dos.ny.gov/datasets/indigenous-territories-native-land-digital/about), [Nat Geo MapMaker](https://education.nationalgeographic.org/resource/mapmaker-indigenous-territories/)) | Publishes prominent disclaimers that its boundaries are **contested, overlapping and not authoritative** — the most-copied example of a map that refuses to resolve conflicting claims. Exactly the posture your disputed-figures feature needs |
| [Indigenous Peoples Atlas of Canada](https://indigenouspeoplesatlasofcanada.ca/) | National | Published by Canadian Geographic with Indigenous organisations (partners not listed on the page fetched) | Multi-domain: reference maps, thematic articles (incl. [Place Names](https://indigenouspeoplesatlasofcanada.ca/article/place-names/) and [Traditional Land Use](https://indigenouspeoplesatlasofcanada.ca/article/traditional-land-use/)), historical and contemporary photography | Indigenous peoples, four sections (Truth and Reconciliation, First Nations, Inuit, Métis) | Live in EN and FR; **update cadence not stated — treat as a published edition rather than a living database** | Print-and-web edition model. Its Place Names article is a good short survey of Canadian Indigenous toponymy projects |
| [Nunaliit Atlas Framework](https://github.com/GCRC/nunaliit) | Software powering many atlases | Geomatics and Cartographic Research Centre (GCRC), Carleton University | Framework: user editing of documents *and* geometries, integrated multimedia, **document relations**, flexible schemas, self-replication, push updates, offline tablet editing and syncing | Atlas infrastructure | Repo public, 2,640 commits (latest commit date not exposed in my fetch) | **Read this before you choose a stack.** Purpose-built for exactly your problem: community-editable, multimedia, relational, offline-capable place atlases. Documented in [Mapping language and land with the Nunaliit Atlas Framework](https://www.elpublishing.org/docs/4/01/FEL-2018-09.pdf) |

#### British Columbia

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [E-Flora BC / E-Fauna BC (Biodiversity of BC)](https://linnet.geog.ubc.ca/biodiversity/) ([E-Fauna intro](https://linnet.geog.ubc.ca/biodiversity/efauna/introduction.html)) | Provincial | UBC Geography; **project coordinator Brian Klinkenberg**, co-coordinators for GIS/mapping and programming, plus **volunteer taxon editors** for plants, fungi, birds, insects, mammals | Multi-domain biogeographic atlases: species pages, distribution maps (incl. North American range maps), photos, essays, checklists, nomenclature editing, citizen science | All wild species of BC | **Dormant-to-fragile.** Atlases load. But the project [blog](https://biodiversitybc.blogspot.com/)'s most recent post is **2021-09-16** ("Birds of British Columbia Checklist 2021"), and an [April 2020 post](http://biodiversitybc.blogspot.com/2020/04/update-on-e-flora-bc-and-e-fauna-bc.html?m=0) documents recovering the site after maps broke | **Study this one closely.** It is the most ambitious volunteer-edited province-wide two-kingdom atlas in North America, run on a shoestring inside a university geography department, and its public record shows both a near-loss event (2020) and a communications stop (2021). The dependency on one named coordinator is the whole risk |
| BC Breeding Bird Atlas | provincial | Birds Canada + volunteers | occurrence, breeding codes ([codes reference](https://birdatlas.bc.ca/bcdata/codes.jsp?lang=en&pg=breeding)) | birds | complete; site live | — |

#### Alberta

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Alberta Biodiversity Monitoring Institute](https://www.abmi.ca/home.html) | Provincial | Institute with substantial permanent staff (count unverified); founding year not stated on site | **Strongly multi-domain**: **3,416 species monitored to date** across amphibians, aquatic invertebrates, birds, bryophytes, lichens, mammals, soil mites, vascular plants; plus land cover / land use / **human footprint mapping**; Open Data Portal, Biodiversity Browser, Mapping Portal, WildTrax sensor-data platform; Indigenous-led monitoring programmes | Species + habitat + human footprint | Live; footer ©2024; current work on algal blooms and caribou habitat | **The human-footprint layer alongside species is the standout.** For a tiger reserve facing land-use pressure, ABMI's model of documenting disturbance at the same resolution as biodiversity is directly applicable. Also the clearest example of separating raw-data portal, encyclopedia browser, and map viewer into distinct products |

#### Ontario

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Ontario Breeding Bird Atlas-3](https://www.birdsontario.org/) ([Ontario Nature page](https://ontarionature.org/programs/community-science/breeding-bird-atlas/), [story map](https://gis.birdscanada.org/portal/apps/storymaps/stories/6dca843321eb48929f839a5191108946)) | Provincial | Birds Canada + partners + thousands of volunteers | Occurrence + breeding codes + point counts | Breeding birds | **Active** — third atlas, 5-year term from 2021-01-01 | Third-generation atlas: a project with two prior baselines to disagree with |
| [Ontario Reptile and Amphibian Atlas](https://ontarionature.org/programs/community-science/reptile-amphibian-atlas/) | Provincial | Ontario Nature; **12,000+ volunteers** over the collection decade | Occurrence; book output | Herpetofauna | **Collection ended 2019**; book published 2023 (443 pp., 70+ maps, 300 photographs); ongoing observations now redirected to the "Herps of Ontario" iNaturalist project | Traces its lineage to the 1984 **Ontario Herpetofaunal Summary** — a 40-year documentary chain, with an explicit, graceful **handover from a bespoke atlas to iNaturalist**. If you expect your capture layer to eventually be absorbed by a global platform, this is how to plan the transition |

#### Québec

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Atlas des oiseaux nicheurs du Québec](https://www.atlas-oiseaux.qc.ca/index_en.jsp) | Provincial (southern + north of 50°30'N) | Birds Canada Québec + partners + volunteers; public **Top-10 contributor leaderboard** by records, hours, species, squares, point counts | Occurrence + breeding evidence; 100 km² squares | Breeding birds | Second atlas **published April 2019** (~10,000 copies over three printings, now out of print; **free PDF available**). The northern (>50°30'N) campaign has concluded with no book planned; data flow to NatureCounts | **Book out of print → free PDF** is the right end-state. Also notable that the northern data will exist *only* as a database, never as a book — a deliberate asymmetry between regions |

#### New Brunswick / Atlantic Canada

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Atlantic Canada Conservation Data Centre](https://accdc.com/) | NB, NS, PEI, NL | Offices in Sackville NB and Corner Brook NL; NatureServe network member | Multi-domain: **~2M geo-located species occurrence records** (~1/5 conservation-relevant), data requests, three iNaturalist rare-species projects (NB, NS, PEI), a new illustrated PEI botanical guide | Species of Atlantic Canada | Live; recent PEI flora guide and iNaturalist project launches | A **four-province shared CDC** — the "one data centre serving several jurisdictions" model, relevant if a Sathyamangalam atlas might extend across the Tamil Nadu/Karnataka/Kerala tri-junction |
| [BiodiverseNB](https://biodiverse-nb.ca/portal/index.php) | New Brunswick | Symbiota instance | Specimens, images | Multi-taxon | Live per Symbiota directory | Canada's provincial entry in the Symbiota federation |
| [Maritimes Breeding Bird Atlas](https://birdcanada.com/maritimes-breeding-bird-atlas/) (mba-aom.ca) | NB, NS, PEI | Birds Canada + volunteers | Occurrence + breeding codes | Birds | Complete; [reported as updated](https://canadiangeographic.ca/articles/maritime-breeding-bird-atlas-updated/) | Tri-province atlas |

#### Nunavut / NWT / Yukon — Indigenous place-name and knowledge atlases

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Gwich'in Atlas](https://atlas.gwichin.ca) | Gwich'in territories, northern Canada | GCRC + **Gwich'in Social and Cultural Institute**; 2012–2018 per the FEL paper | Multi-domain: interactive map where locations carry **audio, video, photos and commentary** | Place names, land use, oral knowledge | **Live** (Nunaliit/CouchDB shell responds; resource creation date 2011-06-01). JS-only, so counts unreadable | Community institution as the data owner, university as the technical host — the governance split your project may need for Indigenous/Adivasi knowledge |
| [Siku Atlas](https://sikuatlas.ca) | Cape Dorset, Clyde River, Igloolik, Pangnirtung (Baffin Island) | GCRC with Inuit communities; 2008–2018 | Sea-ice knowledge atlas: multimedia, terminology, place | Inuit sea-ice knowledge | **Live** (Nunaliit shell responds; JS-only) | Documents a *knowledge system* (sea ice terminology and use) rather than a species list |
| [Kitikmeot Place Name Atlas](https://atlas.kitikmeotheritage.ca/) | Cambridge Bay region, Nunavut | GCRC + **Kitikmeot Heritage Society**; 2009–2017; described in [The Kitikmeot Place Name Atlas](https://www.sciencedirect.com/science/article/abs/pii/B9780444627131000155) | Gazetteer + oral history: place names with audio/video/photo/commentary | Inuit place names | **Live** (Nunaliit shell responds; JS-only; creation date 2011-06-01) | Published as a book chapter *and* a live atlas — the citable-method-plus-live-resource pairing you should aim for |
| [Atlas of the Inuit Language in Canada / Inuktut Lexicon](https://inuktutlexicon.gcrc.carleton.ca) | 12 Inuit dialects and communities | GCRC; 2016–2018 per the FEL paper | Lexical atlas: dialect variation mapped | Language | Liveness **unverified** (blocked in this environment) | Maps *disagreement between dialects* as the primary content — a linguistic parallel to conflicting figures |
| [Inuit Heritage Trust Traditional Place Names](https://www.ihti.ca/traditional-place-names) | Nunavut-wide, 20+ communities (Ikpiarjuk, Iglulik, Mittimatalik, Kimmirut, Kinngait and more) | Inuit Heritage Trust; small staff (contact: lpeplinski@ihti.ca) | Gazetteer: place names integrated onto topographic maps; per-community **Google MyMaps**; the "Nunavut, Where We Live and Travel" map | Inuit place names | **Live but publicly stale** — most recent dated public artefact is the 2014 map update; the site instructs users to email for "our most up-to-date place names data" | **A real and instructive tension**: the authoritative dataset is deliberately *not* fully public, mediated by request, and tied to the Government of Nunavut's Geographic Names Policy — i.e. the gazetteer is a route to official recognition. Compare [Traditional Inuit Place Names in Nunavut](https://www.arcgis.com/home/item.html?id=07a987c2b0aa4d468c315b10298c4749) and [reporting on the programme](https://nunatsiaq.com/stories/article/inuit-heritage-trust-maps-traditional-names-before-theyre-lost/) |
| [Pan Inuit Trails Atlas](https://en.wikipedia.org/wiki/Pan_Inuit_Trails_Atlas) ([Climate Toolkit entry](https://toolkit.climate.gov/tool/pan-inuit-trails-atlas)) | Canadian Arctic | Academic project (Claudio Aporta and colleagues — **unverified from a primary source here**) | Historical document mining: **trails digitised from published accounts, expedition records, ethnographies and community maps**, assembled into a single network | Inuit mobility and occupancy | Documented and indexed; primary site liveness **unverified** | **The purest example in this report of building a place atlas out of scattered archival sources.** Coverage in [Sierra](https://www.sierraclub.org/sierra/these-inuit-maps-are-reimagining-arctic) and [RGS](https://www.rgs.org/our-collections/stories-from-our-collections/explore-our-collections/mapping-inuit-trails) |
| [Residential Schools Land Memory Atlas](https://carleton.ca/fass/2020/the-geomatics-and-cartographic-research-centre-gcrc-at-carleton-university-launches-the-residential-schools-land-memory-atlas-on-june-21-2020/) — delivered within the **Lake Huron Treaty Atlas**, [gcrc.carleton.ca](https://gcrc.carleton.ca/index.html?module=module.gcrcatlas_indigenousknowledge) ([project page](https://geomedialab.org/mapping_residential_schools.html)) | National, treaty-area anchored | GCRC (Carleton) + Concordia; partners Assembly of First Nations, Aboriginal Healing Foundation, National Research Centre on Residential Schools (Univ. of Manitoba), survivor groups, religious organisations, schools; **SSHRC-funded 2015–2020** | **Multi-domain**: archival and historical artefacts, oral histories, commemorative markers, mapped against land and buildings; residential schools map added to the atlas in 2012 | Institutional history tied to place | Atlas launched 2020-06-21; Nunaliit shell responds; JS-only so content not readable in my fetch. [Governor General's History Award](https://newsroom.carleton.ca/2020/carleton-geomatics-and-cartographic-research-centre-partner-wins-governor-generals-history-award-for-community-programming/) for community programming, 2020 | **A treaty atlas as the container for an institutional-history atlas** — legal instrument (treaty), place, archival record and testimony in one structure. The most sophisticated model I found for combining legal instruments with historical document mining and community authority |
| Arctic Bay Atlas (arcticbayatlas.ca) | Arctic Bay, Nunavut | GCRC + Nunavut Youth Consulting + Arctic College; 2009–2018 | was: community atlas, youth-and-elder produced | Inuit community knowledge | **DEAD — domain repurposed.** The URL now serves a commercial travel guide to Arctic Bay, not an atlas; the only atlas remnant is an outbound link to oldmapsonline.org | See dead-projects section |

---

### Mexico

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [SNIB — Sistema Nacional de Información sobre Biodiversidad](https://www.snib.mx/) | National | CONABIO, coordinating a national expert network; hosted on IBUNAM (UNAM) servers | **Very multi-domain**: **50,710,691 occurrence records / 118,652 species**; technical documentation for 3,700+ native and 1,300+ exotic species; **18,000+ thematic maps**; 620,000+ remote-sensing images; 155,000+ photographs and illustrations; **1,177 databases derived from 994 funded projects** | National biodiversity | **Active — last update 2026-02-24** (an earlier figure of 50.7M records is dated August 2024 on the same site) | **The most impressive national-scale multi-domain system in this report, and the best comparator for an Indian state-scale atlas.** Two things to steal: (1) the record count is *dated*, so a reader can see which snapshot they are quoting; (2) the data are organised as **1,177 named source databases from 994 funded projects**, so provenance is a first-class dimension — the natural place to hang conflicting figures |
| [EncicloVida](https://enciclovida.mx/) ([also via SNIB](https://www.snib.mx/snibgeoportal/enciclovida/?id=17527ANGIO)) | National | CONABIO; data compiled since 1992 | Multi-domain species encyclopedia: **118,000+ valid/accepted species with synonymy**; **common names in multiple Indigenous languages**; taxonomy; potential-distribution and endemism maps; **legal conservation status (NOM-059 at-risk, invasive, priority species) plus CITES and IUCN**; photos, video, audio; citizen-science records from Naturalista and AverAves; links to Macaulay Library, Missouri Botanical Garden, Biodiversity Heritage Library | Species of Mexico | Live; web plus iOS/Android apps ([launch coverage](https://www.gob.mx/conabio/prensa/conabio-lanza-enciclovida)) | **The closest existing model for your "many source types on one page."** A single species page carries taxonomy, synonymy, Indigenous-language names, occurrences, images, media, national legal status *and* two independent international assessments — i.e. it routinely displays **three different conservation verdicts side by side without reconciling them** |
| [Naturalista](https://www.naturalista.mx/) | National | CONABIO-operated iNaturalist node | Occurrence + community identification | All taxa | Site returned 403 to my fetch; **counts and liveness unverified** here, though it is referenced as a live data source by both EncicloVida and SNIB | A national iNaturalist node feeding the national system — the "adopt the global platform, keep the national identity" pattern |
| [Red de Herbarios Mexicanos](https://herbanwmex.net/portal/index.php) | National | Network of Mexican herbaria; Symbiota | Specimens, images | Plants | Live per Symbiota directory | Mexico's entry in the Symbiota federation |
| [DEMCA — Documenting Ethnobiology in Mexico and Central America](https://demca.mesolex.org/portal/) | Mexico + Central America | Academic (Mesolex); size unverified | **Ethnobiology**: specimens linked to Indigenous-language plant names and uses | Plants + Indigenous knowledge | Live per Symbiota directory | Symbiota bent to hold **ethnobiological/linguistic data alongside vouchers** — precedent for attaching local Tamil/Irula/Soliga plant names and uses to specimen records |
| Pa Ipai Astronomical Atlas (nunaliitworkshop.centrogeo.org.mx/paipai) | Santa Catarina and Héroes de la Independencia, Baja California | GCRC-trained team with Pa Ipai and Kumiai (Koal) peoples; 2018; arose from a [Nunaliit workshop in Mexico](https://research.carleton.ca/2018/08/geomatics-and-cartographic-research-center-delivers-workshops-in-mexico-on-the-innovative-nunaliit-digital-atlas-platform/) | Community atlas: astronomical/sky knowledge tied to place | Indigenous astronomy | Liveness **unverified** (workshop-subdomain URL, not independently fetched) | Evidence the Nunaliit model **transfers to the Global South via short training workshops** — the most encouraging capacity-building precedent for a project like yours |
| [Madrean Discovery / MABA](https://madreandiscovery.org/) | Binational Sky Islands | see US Symbiota table | | | | Binational, bioregional |

---

### Caribbean

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status | Notable |
|---|---|---|---|---|---|---|
| [Caribbean Marine Atlas (CMA2)](http://www.caribbeanmarineatlas.net/) | Wider Caribbean Region, IOCARIBE member states | Intergovernmental: CARICOM, UNESCO/IOC, FAO-WECAFC, CRFM, OSPESCA, CCAD, OECS, UN Environment/CEP | Multi-domain geospatial: **13 thematic fields across biotic, physical and anthropic categories** — habitats, protected areas, fisheries, natural hazards, oceanography — plus "related documents" | Marine environment and human societies | **Probably dormant.** Site loads, but the most recent framing on the page is the CLME+ Project period **2015–2020**; no update date, no post-2020 content visible | An intergovernmental multi-country atlas whose visible life ends with its funding project — the canonical grant-cycle death. **No comparably comprehensive live Caribbean biodiversity atlas surfaced in this survey**; treat this as a genuine regional gap rather than a search failure |

---

### Dead / dormant projects found

Ordered from most conclusively dead to merely stale. This is the section I would read first if I were you.

| Project | Evidence of death/dormancy | Date of evidence | Lesson |
|---|---|---|---|
| **BISON (Biodiversity Information Serving Our Nation)**, bison.usgs.gov — USGS national occurrence aggregator | **Domain no longer resolves**: my fetch failed with "robots.txt fetch failed: [Errno -5] No address associated with hostname". Retirement year **unverified** (my attempt at the USGS tool page returned 404); data are understood to have moved to GBIF-US infrastructure | 2026-08-25 | A federal-scale aggregator can be switched off entirely. Don't let your atlas be a portal with no independent archive |
| **NBII (National Biological Information Infrastructure)**, nbii.gov | **Domain no longer resolves** (DNS failure on fetch). Congressionally defunded; termination year **unverified** here | 2026-08-25 | The largest US biodiversity informatics programme of its era left no live trace at its own URL |
| **Arctic Bay Atlas**, arcticbayatlas.ca — Nunaliit community atlas, 2009–2018 | **Domain repurposed.** The URL now serves a commercial travel guide to Arctic Bay; the only atlas-like element is an external link to oldmapsonline.org. Named as an active atlas in the [2018 FEL paper](https://www.elpublishing.org/docs/4/01/FEL-2018-09.pdf) | 2026-08-25 | Worse than a 404: community-produced Indigenous content replaced by tourism copy at the same address. **Register domains long, and archive**|
| **Boston Harbor Islands ATBI database** (bhi.oeb.harvard.edu MantisWeb) | Custom database host **failed DNS resolution**; the [NPS project page](https://www.nps.gov/boha/learn/nature/atbi.htm) reads "Last updated: October 25, 2019" and describes bioblitzes only up to 2008. The science persists in [*Northeastern Naturalist* (2018)](https://bioone.org/journals/northeastern-naturalist/volume-25/issue-sp9/045.025.s903/Exploring-the-Microwilderness-of-Boston-Harbor-Islands-National-Recreation-Area/10.1656/045.025.s903.short) | 2026-08-25 | Bespoke portal dies, peer-reviewed paper survives. Publish the synthesis |
| **Electronic Cultural Atlas Initiative (ECAI)** — founded 1997 by Lewis Lancaster, UC Berkeley; the ancestor of today's historical gazetteers | Per [Wikipedia](https://en.wikipedia.org/wiki/Electronic_Cultural_Atlas_Initiative), meetings ran 1998–2009; its TimeMap software is [historical](https://en.wikipedia.org/wiki/TimeMap); surviving traces are a [CITRIS retrospective](https://citris-uc.org/the-electronic-cultural-atlas-initiative/) and a residual [CKAN data portal](https://ecaidata.org/dataset/ecaiclearinghouse-id-51) | fetched 2026-08-25 | ECAI's clearinghouse-of-distributed-datasets model failed; WHG's centralised-index-with-attestations model succeeded. **Choose the latter** |
| **Alaska Native Place Names Project**, akplacenames.org | Site loads; most recent dated content is a 2016 photo credit; **no leads, host, funding, place-name count or news anywhere on the site**. A companion [personal research page](https://gmholton.github.io/research/akplacenames) and [2015 press coverage](https://www.adn.com/culture/article/cultural-identity-climate-research-invigorated-push-native-place-names/2015/11/09/) exist | 2026-08-25 | The stated goal ("authoritative statewide database") was never publicly delivered. **Publish counts and dates or your project reads as abandoned even if it isn't** |
| **Caribbean Marine Atlas (CMA2)** | Loads; latest framing is the CLME+ project period 2015–2020; no post-2020 content or update date | 2026-08-25 | Grant-cycle dormancy at intergovernmental scale |
| **BONAP / North American Plant Atlas** | Page states the **2014 edition (11/17/2014)** is the most recent major release | 2026-08-25 | Twelve years since the last release, on a resource still widely cited as current |
| **KRIS Web** (krisweb.com) | Footer copyright **2011**; still offers distribution on **CD** | 2026-08-25 | Architecturally the closest match to your brief, and 15 years unmaintained |
| **Illinois Wildflowers** (illinoiswildflowers.info) | Copyright "© 2002-2020 by John Hilty" | 2026-08-25 | Single-person succession risk, realised |
| **LandScope America** (landscope.org) | Footer "Copyright © 2022 NatureServe"; still presents a 2025 target as forward-looking | 2026-08-25 | An NGO flagship quietly parked |
| **E-Flora BC / E-Fauna BC** — dormant, not dead | Project blog's last post **2021-09-16**; a [2020 post](http://biodiversitybc.blogspot.com/2020/04/update-on-e-flora-bc-and-e-fauna-bc.html?m=0) documents recovering from a mapping outage | fetched 2026-08-25 | Atlases still serve; the human layer went quiet first. **Blog silence is the leading indicator of atlas death** |
| **Digital Atlas of Idaho** | Live at [isu.edu/digitalatlas](https://www.isu.edu/digitalatlas/); **no authorship, funding, or date anywhere reachable**; an [Internet Archive item](https://archive.org/details/digital-atlas-of-idaho) exists; a 2004 [GSA abstract](https://gsa.confex.com/gsa/2004RM/finalprogram/abstract_72327.htm) and a [2011 blog post](http://spencerjardine.blogspot.com/2011/03/digital-atlas-of-idaho-idahos-natural.html?m=0) describe it in use | 2026-08-25 | Undated content is unciteable content |
| **NY Breeding Bird Atlas application** — frozen by design | "Data available on this website were finalized in **March 2007**" | 2026-08-25 | A frozen dataset with a stated freeze date is *healthy*. Contrast with Idaho |
| **San Diego County Bird Atlas** | 2004 book; museum page says only "some information can be found online as well" | 2026-08-25 | A landmark county atlas that never became a full portal |
| **Colorado Breeding Bird Atlas II site** | Database queries work; copyright 2016; book Nov 2016 | 2026-08-25 | Working legacy site, no ongoing programme |
| **OregonFlora content freshness** | Infrastructure current; latest dated events on site are 2024 | 2026-08-25 | Distinguish "platform maintained" from "content maintained" |
| **iMapInvasives jurisdictional contraction** | Only 5 states listed (AZ, ME, NY, OR, PA); no Canadian provinces, despite prior provincial participation (**cause unverified**) | 2026-08-25 | Multi-tenant networks shrink quietly |
| **WYNDD intake** | "0" new species records in the most recent quarterly statistic, on an otherwise live site (page timestamp 2026-07-30) | 2026-08-25 | Only visible because they publish throughput. Publish yours |
| **Oregon Explorer re-platforming** | oregonexplorer.info 302-redirects to a hub-style site; the previous topic-portal and report content is not evident on the new hub | 2026-08-25 | Migrations drop non-spatial content first: reports, bibliographies, historical documents. **Exactly your most valuable material** |

---

### Leads I could not verify

- **Plateau Peoples' Web Portal** (plateauportal.libraries.wsu.edu) and **Reciprocal Research Network** (rrncommunity.org) — both returned **HTTP 403** to my fetches. Both are highly relevant: the former is a WSU-hosted portal where tribal partners curate their own records of museum/archival items (Mukurtu-lineage, **unverified**), the latter a Northwest Coast multi-museum platform where communities add knowledge alongside institutional catalogue records. **Both let multiple parties describe the same object differently** — the single most relevant precedent for your conflicting-figures feature outside the biodiversity domain. Worth a manual visit.
- **SlaveVoyages** — homepage and `/about/about` both returned JS-only shells. Counts and the estimates methodology above come from [Wikipedia](https://en.wikipedia.org/wiki/Trans-Atlantic_Slave_Trade_Database), not the primary site. Current host, version and update date **unverified**.
- **SOFIA (South Florida Information Access, sofia.usgs.gov)** — returned HTTP 403. A USGS place-based system combining publications, data and historical documents for the Everglades; status **unverified**. Likely worth chasing given how closely a single-ecosystem USGS information system matches your brief.
- **Michigan Flora Online** and **Consortium of Midwest Herbaria** homepage — JS-only and robots-disallowed respectively; counts and dates **unverified**.
- **Alabama Plant Atlas** — both `alabama.plantatlas.usf.edu` (DNS/robots failure) and `www.floraofalabama.org` (**TLS certificate hostname mismatch**) failed. Existence confirmed only via the Symbiota/PlantAtlas ecosystem. Liveness **unverified**.
- **Connecticut Bird Atlas** (ctbirdatlas.org) — TLS/robots failure; host responded but content unreadable. Counts, leads, publication state **unverified**.
- **Hawaiian Ecosystems at Risk (HEAR)**, hear.org — fetch failed on robots parsing; could not determine whether live or dead. Long suspected dormant; **unverified**.
- **Gulf of Mexico Data Atlas** — [ncei.noaa.gov/maps/gulf-data-atlas](https://www.ncei.noaa.gov/maps/gulf-data-atlas/) returned a redirect loop and the legacy ncddc.noaa.gov host has a **TLS certificate hostname mismatch**. A [1985 edition](https://www.ncei.noaa.gov/maps/DataAtlas_1985/atlas.html) and [NOAA release note](https://www.noaa.gov/sites/default/files/2021-11/NOAAs-Gulf-of-Mexico-Data-Atlas-released.pdf) exist. Status **unverified**; the certificate failure on the legacy host is itself a decay signal. Note also the region's renaming to "Gulf of America" in current NOAA products ([ecowatch](https://ecowatch.noaa.gov/regions/gulf-of-america)) — **a live, government-level toponym dispute in your exact problem space**, and worth studying for how a gazetteer should represent it.
- **Naturalista (Mexico)** — HTTP 403; counts **unverified**.
- **Nunaliit latest commit date** — the GitHub page exposed "2,640 Commits" but no timestamp in my fetch. Whether the framework is actively developed in 2026 is **unverified**, and it matters if you are considering adopting it.
- **Inuktut Lexicon atlas, Lake Huron Treaty Atlas content, Pa Ipai Astronomical Atlas** — GCRC atlases are JavaScript-only, so I confirmed only that hosts respond. Place-name counts, community coverage and last-edit dates **unverified** for all Nunaliit atlases.
- **Team sizes and funding, nearly everywhere.** Only a handful of projects publish either: Virginia BBA (~1,500 volunteers), Ontario Reptile & Amphibian Atlas (12,000+ volunteers), SERNEC (230 herbaria, NSF 1410069), E-Flora BC (named coordinator + volunteer editors), Fishes of Texas (named leads + four funders), Enslaved.org (Mellon, NEH). For most state natural heritage programmes, herbarium consortia and solo sites, **staffing and funding are simply not published** — treat any figure I have not attributed as unverified.
- **Georgia Biodiversity Portal, NYNHP, MNFI, PNHP, NatureServe Explorer** — element/species counts not published on landing pages; **unverified**.
- **Alaska Native Place Names Project** — leads, host, funding and place-name count absent from the site; Gary Holton's association inferred from a personal research page, **unverified as current**.
- **Maryland Biodiversity Project founders** — not named on the site pages I fetched; commonly credited to Bill Hubick and Jim Brighton, **unverified from primary source**.
- **BONAP leadership** — John T. Kartesz's directorship is widely reported but **not verified from bonap.org** in this survey.
- **Massachusetts "Butterflies of Massachusetts" historical species accounts** (Sharon Stichter) — the species-accounts URL I tried returned 404. The historical-document-mining component of that project, which is the reason it interested me, is **unverified**.
- **Caribbean coverage generally.** Beyond the Caribbean Marine Atlas I found no comprehensive live island or national biodiversity atlas for Puerto Rico, Cuba, Hispaniola, Jamaica or the Bahamas. My WebSearch budget was exhausted mid-survey (200/200 calls, consumed session-wide), so **treat Caribbean coverage as under-searched rather than empty** — this is the largest known gap in this report.
- Similarly under-searched because of that budget exhaustion: US states with no entry above (AR, DE, IN, IA, KY, LA, ME, MS, MO, NE, NV, NH, NJ, NM, ND, OK, RI, SC, TN, UT, WV) — most have NatureServe-network heritage programmes and several have state plant atlases; absence here is **not** evidence of absence. Also under-searched: Manitoba, Saskatchewan, Nova Scotia, PEI, Newfoundland and the territories beyond the Nunaliit set; OBIS regional nodes (the [obis.org/institute](https://obis.org/institute/) path 404'd); tribal historic preservation office portals in the lower 48; and US state historical-gazetteer/HGIS projects.

---

#### Five things from this survey I would act on for Sathyamangalam

1. **Copy Fishes of Texas's versioning.** A dated, numbered dataset release (v3.10, 2026-03-23) makes every disputed figure citable as "the figure as of v3.10." Nothing else in this survey solves the conflicting-data problem so cleanly.
2. **Copy the Digital Atlas of the Virginia Flora's "Excluded Taxa" document.** A published register of rejected records, with reasons, is a one-page feature that buys enormous credibility — and it is the minimum viable version of your side-by-side feature.
3. **Copy World Historical Gazetteer's Linked Places model** for the place-name layer: attestation-per-source-per-date, with endonyms and exonyms coexisting. Do not build a single-preferred-name gazetteer; you will regret it the first time a village has three names in two scripts.
4. **Copy EncicloVida's species page.** One page showing NOM-059 status *and* CITES *and* IUCN without reconciling them is the exact interaction pattern you want for a species with a disputed status under Indian schedules vs IUCN.
5. **Copy SFEI's historical ecology method**, not just its outputs: certainty ratings on interpreted features, source-by-source corroboration, and a published GIS compilation separate from the narrative atlas. It is the only rigorous archival-to-spatial workflow in this survey with a documented method.

And two anti-patterns: **do not build a JavaScript-only atlas** (Michigan Flora, every Nunaliit atlas — invisible to crawlers, archivers, and any future automated reader), and **do not ship undated content** (Digital Atlas of Idaho is comprehensive, multi-domain, exactly your shape, and effectively unciteable because nothing on it carries a date).

---

#### Sources

Hubs, directories and framework documentation
- [Symbiota Portals directory](https://symbiota.org/symbiota-portals/) · [Symbiota portal-list docs](https://symbiota.org/docs/symbiota-introduction/portal-list/) · [SEINet on Symbiota](https://symbiota.org/seinet/)
- [Birds Canada — Breeding Bird Atlases](https://www.birdscanada.org/bird-science/breeding-bird-atlases) · [NatureCounts Ontario Atlas](https://naturecounts.ca/nc/onatlas/main.jsp) · [Wikipedia — Breeding bird atlas](https://en.wikipedia.org/wiki/Breeding_bird_atlas)
- [NatureServe Network](https://www.natureserve.org/natureserve-network) · [NatureServe western-US portal launch](https://www.natureserve.org/news-releases/natureserve-launches-conservation-data-portal-western-us) · [Esri on Explorer Pro](https://www.esri.com/about/newsroom/publications/wherenext/developing-natureserve-explorer-pro) · [CNHP on the NatureServe Network](https://cnhp.colostate.edu/aboutus/the-natureserve-network)
- [GCRC/nunaliit on GitHub](https://github.com/GCRC/nunaliit) · [About Nunaliit](https://svn.gcrc.carleton.ca/nunaliit/trunk/website/htdocs_src/about.html) · [Mapping language and land with the Nunaliit Atlas Framework (FEL 2018)](https://www.elpublishing.org/docs/4/01/FEL-2018-09.pdf) · [Nunaliit workshops in Mexico](https://research.carleton.ca/2018/08/geomatics-and-cartographic-research-center-delivers-workshops-in-mexico-on-the-innovative-nunaliit-digital-atlas-platform/) · [Nunaliit in relational lexicography](https://knowledgebase.arts.ubc.ca/nunaliit/)
- [Discover Life ATBI brochure](https://www.discoverlife.org/ATBI_brochure.html) · [Wikipedia — All-taxa biodiversity inventory](https://en.wikipedia.org/wiki/All-taxa_biodiversity_inventory) · [Wikipedia — Discover Life in America](https://en.wikipedia.org/wiki/Discover_Life_in_America)

United States
- [NPSpecies](https://irma.nps.gov/NPSpecies/) · [NPSpecies user guide](https://irma.nps.gov/content/npspecies/Help/docs/NPSpecies_User_Guide.pdf) · [NPSpecies GBIF resource](https://ipt.gbif.us/resource?r=nps-npspecies-park_species_lists) · [IRMA](https://irma.nps.gov/Portal/) · [NPS IRMA explainer](https://www.nps.gov/articles/irma.htm)
- [FWS ECOS](https://ecos.fws.gov/ecp/) · [USGS GNIS downloads](https://www.usgs.gov/us-board-on-geographic-names/download-gnis-data) · [GNIS tool page](https://www.usgs.gov/tools/geographic-names-information-system-gnis) · [Wikipedia — GNIS](https://en.wikipedia.org/wiki/Geographic_Names_Information_System)
- [Newberry AHCB](https://digital.newberry.org/ahcb/) · [Flora of North America](https://floranorthamerica.org/Main_Page) · [FNA at efloras](http://www.efloras.org/flora_page.aspx?flora_id=1) · [BONAP](http://www.bonap.org/) · [BAMONA](https://www.butterfliesandmoths.org/) · [OdonataCentral](https://www.odonatacentral.org/)
- [SEINet](https://swbiodiversity.org/seinet/index.php) · [SEINet AZ/NM node](https://swbiodiversity.org/) · [Consortium of Pacific Northwest Herbaria](https://www.pnwherbaria.org/) · [Consortium of Midwest Herbaria](https://midwestherbaria.org/) · [SERNEC](https://sernecportal.org/portal/index.php) · [CCH2](https://cch2.org/portal/index.php) · [CNH](https://neherbaria.org/) · [SoRo](https://www.soroherbaria.org/portal/) · [Intermountain Biota](https://intermountainbiota.org/portal/collections/index.php) · [Mid-Atlantic Herbaria](https://midatlanticherbaria.org/portal/) · [TORCH](https://portal.torcherbaria.org/portal/index.php) · [Northern Great Plains Herbaria](https://ngpherbaria.org/portal/) · [Four Corners Plants](https://fourcornersplants.org/portal/index.php) · [NANSH](https://nansh.org/portal/) · [Cal-IBIS](http://www.cal-ibis.org/) · [BioGator](https://biogator.org/index.php) · [Kansas Biodiversity](https://ks.symbiota.org/portal/) · [WisFlora](https://wisflora.herbarium.wisc.edu/index.php) · [INHS Biocollections](https://biocoll.inhs.illinois.edu/portal/index.php) · [COLO](https://botanydb.colorado.edu/index.php) · [Madrean Discovery](https://madreandiscovery.org/) · [Indian River Lagoon Species Inventory](https://irlspecies.org/index.php)
- [NatureServe Explorer](https://explorer.natureserve.org/) · [Explorer Pro](https://explorer.natureserve.org/pro/) · [LandScope America](http://www.landscope.org/) · [iMapInvasives](https://www.imapinvasives.org/)
- [Whaling History](https://whalinghistory.org/) · [World Historical Gazetteer](https://whgazetteer.org/) · [WHG blog](https://blog.whgazetteer.org/) · [WHG at ISHI](https://www.ishi.pitt.edu/world-historical-gazetteer) · [Pitt announcement](https://www.pittwire.pitt.edu/features-articles/2025/06/24/spatial-history-world-gazetteer) · [SlaveVoyages](https://www.slavevoyages.org/) · [Wikipedia — Trans-Atlantic Slave Trade Database](https://en.wikipedia.org/wiki/Trans-Atlantic_Slave_Trade_Database) · [Enslaved.org](https://enslaved.org/) · [Mapping Prejudice](https://mappingprejudice.umn.edu/) · [Densho](https://densho.org/)
- [ACCS](https://accs.uaa.alaska.edu/) · [Alaska Native Place Names Project](https://akplacenames.org/) · [ANPNP history](https://akplacenames.org/history/) · [Gary Holton research page](https://gmholton.github.io/research/akplacenames) · [Web Atlas of Alaska Native Traditional Place Names](https://storymaps.arcgis.com/stories/b31fc761a8ea4d7da349985d6932d58c) · [ADN coverage](https://www.adn.com/culture/article/cultural-identity-climate-research-invigorated-push-native-place-names/2015/11/09/) · [AOOS](https://aoos.org/) · [AMBON](https://aoos.org/project/arctic-marine-biodiversity-observation-network/) · [AMBON data at NCEI](https://www.ncei.noaa.gov/access/metadata/landing-page/bin/iso?id=gov.noaa.nodc:NOPP-AMBON-US) · [Other Arctic data portals](https://aoos.org/other-arctic-data-portals/)
- [Calflora](https://www.calflora.org/) · [Jepson eFlora](https://ucjeps.berkeley.edu/eflora/) · [SFEI Historical Ecology](https://www.sfei.org/programs/rl/historical-ecology) · [SFEI historical ecology GIS compilation](https://www.sfei.org/data/historical-ecology-gis-data-compilation) · [Napa Valley Historical Ecology Atlas](https://www.sfei.org/projects/napa-valley-historical-ecology-atlas) · [Sacramento Valley Historical Ecology](https://www.sfei.org/projects/sacramento-valley-historical-ecology) · [Hidden Nature SF](https://www.sfei.org/projects/hidden-nature-sf) · [SFEI Data Center](https://www.sfei.org/data-center) · [SDNHM Atlas Projects](https://www.sdnhm.org/science/atlas-projects/) · [San Diego Plant Atlas](https://www.sdnhm.org/science/botany/projects/plant-atlas/) · [KRIS Web](https://www.krisweb.com/) · [KRIS Klamath bibliography](https://www.krisweb.com/biblio/biblio_klamath.htm)
- [Colorado BBA II](https://www.cobreedingbirdatlasii.org/) · [Connecticut Bird Atlas](https://ctbirdatlas.org/) · [Atlas of Florida Plants](https://florida.plantatlas.usf.edu/) · [PlantAtlas.org](https://plantatlas.usf.edu/) · [Georgia Biodiversity Portal](https://georgiabiodiversity.org/) · [Hawaii Biological Survey](https://www.bishopmuseum.org/hbs/) · [Digital Atlas of Idaho (ISU)](https://www.isu.edu/digitalatlas/) · [Digital Atlas of Idaho sources page](https://digitalatlas.cose.isu.edu/aboutus/sref2.htm) · [Digital Atlas of Idaho at Internet Archive](https://archive.org/details/digital-atlas-of-idaho) · [Illinois Wildflowers](https://www.illinoiswildflowers.info/)
- [Boston Harbor Islands ATBI (NPS)](https://www.nps.gov/boha/learn/nature/atbi.htm) · [Farrell Lab ATBI](https://farrell.oeb.harvard.edu/boston-harbor-islands-all-taxa-biodiversity-inventory) · [BHI ATBI paper](https://bioone.org/journals/northeastern-naturalist/volume-25/issue-sp9/045.025.s903/Exploring-the-Microwilderness-of-Boston-Harbor-Islands-National-Recreation-Area/10.1656/045.025.s903.short) · [Massachusetts Butterfly Club](https://www.massbutterflies.org/)
- [Maryland Biodiversity Project](https://www.marylandbiodiversity.com/) · [MBP About Records](https://www.marylandbiodiversity.com/about/records.php) · [MBP contribute](https://www.marylandbiodiversity.com/about/contribute.php) · [MNFI](https://mnfi.anr.msu.edu/) · [MNFI natural community plant lists](https://mnfi.anr.msu.edu/communities/plant-lists) · [Michigan Flora Online](https://michiganflora.net/) · [Minnesota Wildflowers](https://www.minnesotawildflowers.info/) · [MN Wildflowers search](https://minnesotawildflowers.info/page/search)
- [Montana Field Guide](https://fieldguide.mt.gov/) · [NC Biodiversity Project](https://nc-biodiversity.com/) · [New York Flora Atlas](https://newyork.plantatlas.usf.edu/) · [NYNHP](https://www.nynhp.org/) · [NY BBA (DEC)](https://dec.ny.gov/nature/animals-fish-plants/birds/breeding-bird-atlas) · [NY BBA past data & maps](https://dec.ny.gov/nature/animals-fish-plants/birds/breeding-bird-atlas/past-data-maps) · [NY BBA legacy app](https://extapps.dec.ny.gov/cfmx/extapps/bba)
- [Ohio Odonata Society](https://www.ohioodonatasociety.org/) · [OregonFlora](https://oregonflora.org/) · [Oregon Explorer hub](https://hub.oregonexplorer.info/) · [PNHP](https://www.naturalheritage.state.pa.us/) · [South Dakota BBA II](https://gfp.sd.gov/breeding-bird-atlas/) · [Fishes of Texas](https://www.fishesoftexas.org/)
- [Vermont Atlas of Life](https://val.vtecostudies.org/) · [VAL About](https://val.vtecostudies.org/about/) · [VAL wildlife atlases](https://val.vtecostudies.org/wildlife-atlases/) · [VAL Bumble Bee Atlas](https://val.vtecostudies.org/projects/bumble-bee-atlas/about/) · [Bees of Vermont](https://val.vtecostudies.org/projects/vtbees/about/) · [VAL paper](https://www.researchgate.net/publication/357660402_The_Vermont_Atlas_of_Life_Discovering_and_Sharing_Biodiversity_Knowledge)
- [Virginia Bird Atlas](https://vabirdatlas.org/) · [Virginia DWR atlas](https://dwr.virginia.gov/wildlife/virginia-breeding-bird-atlas/) · [Atlas Phase Two launch](https://dwr.virginia.gov/blog/phase-two-of-the-atlas-website-has-launched/) · [Digital Atlas of the Virginia Flora](https://vaplantatlas.org/) · [VNPS Digital Atlas Fund](https://vnps.org/z02-past-admin/digital-atlas-fund-2024/) · [WA DNR Natural Heritage Program](https://dnr.wa.gov/natural-heritage-program) · [WYNDD](https://www.uwyo.edu/wyndd/)

Canada
- [Canadensys](https://www.canadensys.net/) · [ABMI](https://www.abmi.ca/home.html) · [AC CDC](https://accdc.com/) · [BiodiverseNB](https://biodiverse-nb.ca/portal/index.php)
- [Biodiversity of BC](https://linnet.geog.ubc.ca/biodiversity/) · [Biodiversity of BC intro](https://linnet.geog.ubc.ca/biodiversity/Biodiversityintroduction.html) · [E-Fauna BC intro](https://linnet.geog.ubc.ca/biodiversity/efauna/introduction.html) · [Biodiversity of BC blog](https://biodiversitybc.blogspot.com/) · [E-Flora/E-Fauna 2020 status post](http://biodiversitybc.blogspot.com/2020/04/update-on-e-flora-bc-and-e-fauna-bc.html?m=0) · [Klinkenberg publications](https://ibis.geog.ubc.ca/~brian/pubs.html)
- [BC Breeding Bird Atlas](https://www.birdatlas.bc.ca/what-is-an-atlas/) · [BC atlas breeding codes](https://birdatlas.bc.ca/bcdata/codes.jsp?lang=en&pg=breeding) · [Ontario Breeding Bird Atlas](https://www.birdsontario.org/) · [Ontario Nature BBA](https://ontarionature.org/programs/community-science/breeding-bird-atlas/) · [Atlas-3 story map](https://gis.birdscanada.org/portal/apps/storymaps/stories/6dca843321eb48929f839a5191108946) · [Ontario Reptile & Amphibian Atlas](https://ontarionature.org/programs/community-science/reptile-amphibian-atlas/) · [Maritimes BBA](https://birdcanada.com/maritimes-breeding-bird-atlas/) · [Maritimes atlas update](https://canadiangeographic.ca/articles/maritime-breeding-bird-atlas-updated/) · [Newfoundland BBA](https://nf.birdatlas.ca/about-the-atlas/) · [Atlas des oiseaux nicheurs du Québec](https://www.atlas-oiseaux.qc.ca/index_en.jsp)
- [Gwich'in Atlas](https://atlas.gwichin.ca) · [Siku Atlas](https://sikuatlas.ca) · [Kitikmeot Place Name Atlas](https://atlas.kitikmeotheritage.ca/) · [Kitikmeot atlas chapter](https://www.sciencedirect.com/science/article/abs/pii/B9780444627131000155) · [GCRC atlases](https://gcrc.carleton.ca/) · [Residential Schools Land Memory Atlas launch](https://carleton.ca/fass/2020/the-geomatics-and-cartographic-research-centre-gcrc-at-carleton-university-launches-the-residential-schools-land-memory-atlas-on-june-21-2020/) · [Carleton newsroom](https://newsroom.carleton.ca/2020/carletons-geomatics-and-cartographic-research-centre-to-launch-residential-schools-land-memory-atlas/) · [Residential Schools Land Memory Mapping Project](https://geomedialab.org/mapping_residential_schools.html) · [Governor General's History Award](https://newsroom.carleton.ca/2020/carleton-geomatics-and-cartographic-research-centre-partner-wins-governor-generals-history-award-for-community-programming/) · [Digital atlas returns knowledge to Inuit communities](https://research.carleton.ca/2016/04/new-digital-atlas-returns-early-traditional-knowledge-inuit-communities/)
- [Inuit Heritage Trust place names](https://www.ihti.ca/traditional-place-names) · [Nunatsiaq coverage](https://nunatsiaq.com/stories/article/inuit-heritage-trust-maps-traditional-names-before-theyre-lost/) · [Traditional Inuit Place Names in Nunavut (ArcGIS)](https://www.arcgis.com/home/item.html?id=07a987c2b0aa4d468c315b10298c4749) · [Pan Inuit Trails Atlas (Wikipedia)](https://en.wikipedia.org/wiki/Pan_Inuit_Trails_Atlas) · [Pan Inuit Trails (Climate Toolkit)](https://toolkit.climate.gov/tool/pan-inuit-trails-atlas) · [Sierra on Inuit maps](https://www.sierraclub.org/sierra/these-inuit-maps-are-reimagining-arctic) · [RGS on mapping Inuit trails](https://www.rgs.org/our-collections/stories-from-our-collections/explore-our-collections/mapping-inuit-trails) · [Inuit Land Use and Occupancy Project](https://en.wikipedia.org/wiki/Inuit_Land_Use_and_Occupation_Project)
- [Native-Land.ca](https://native-land.ca/) · [Native Land map](https://native-land.ca/maps/native-land) · [Wikipedia — Native Land Digital](https://en.wikipedia.org/wiki/Native_Land_Digital) · [Indigenous Peoples Atlas of Canada](https://indigenouspeoplesatlasofcanada.ca/) · [IPAC Place Names](https://indigenouspeoplesatlasofcanada.ca/article/place-names/) · [IPAC Traditional Land Use](https://indigenouspeoplesatlasofcanada.ca/article/traditional-land-use/) · [HillNotes on Indigenous mapping and place names](https://hillnotes.ca/2021/06/21/putting-indigenous-perspectives-on-the-map-indigenous-mapping-and-place-names/) · [Carleton Indigenous GIS guide](https://library.carleton.ca/guides/help/indigenous-studies-gis-resources) · [York University Indigenous maps guide](https://researchguides.library.yorku.ca/maps/indigenous) · [Land use and occupancy studies / counter-mapping (BC Studies)](https://ojs.library.ubc.ca/index.php/bcstudies/article/download/186217/185708/199384) · [GRASAC](https://grasac.artsci.utoronto.ca/)

Mexico & Caribbean
- [SNIB](https://www.snib.mx/) · [EncicloVida](https://enciclovida.mx/) · [EncicloVida via SNIB geoportal](https://www.snib.mx/snibgeoportal/enciclovida/?id=17527ANGIO) · [CONABIO EncicloVida app launch](https://www.gob.mx/conabio/prensa/conabio-lanza-la-aplicacion-movil-enciclovida) · [EncicloVida launch coverage](https://www.portalambiental.com.mx/ciencia-y-tecnologia/20191107/conabio-lanza-enciclovida-la-enciclopedia-de-la-naturaleza) · [CONABIO biodiversity data (SIMAR)](https://simar.conabio.gob.mx/sidmo-bioinfo/) · [Naturalista](https://www.naturalista.mx/) · [Red de Herbarios Mexicanos](https://herbanwmex.net/portal/index.php) · [DEMCA](https://demca.mesolex.org/portal/)
- [Caribbean Marine Atlas](http://www.caribbeanmarineatlas.net/)

Dead / dormant evidence
- [Electronic Cultural Atlas Initiative (Wikipedia)](https://en.wikipedia.org/wiki/Electronic_Cultural_Atlas_Initiative) · [CITRIS on ECAI](https://citris-uc.org/the-electronic-cultural-atlas-initiative/) · [ECAI data portal remnant](https://ecaidata.org/dataset/ecaiclearinghouse-id-51) · [TimeMap (Wikipedia)](https://en.wikipedia.org/wiki/TimeMap) · [Buckland on ECAI (CNI)](https://www.cni.org/wp-content/uploads/2013/04/H-Electronic-Buckland.doc)
- [Scout Archives — Digital Atlas of Idaho](https://archives.internetscout.org/r28432/the_digital_atlas_of_idaho) · [2004 GSA abstract on the Digital Atlas](https://gsa.confex.com/gsa/2004RM/finalprogram/abstract_72327.htm) · [2011 blog note on the Digital Atlas](http://spencerjardine.blogspot.com/2011/03/digital-atlas-of-idaho-idahos-natural.html?m=0)
- [Gulf of Mexico Data Atlas (NCEI)](https://www.ncei.noaa.gov/maps/gulf-data-atlas/) · [1985 Gulf Data Atlas](https://www.ncei.noaa.gov/maps/DataAtlas_1985/atlas.html) · [NOAA release note](https://www.noaa.gov/sites/default/files/2021-11/NOAAs-Gulf-of-Mexico-Data-Atlas-released.pdf) · [NCEI news](https://www.ncei.noaa.gov/news/discover-gulf-mexico-through-maps) · [NOAA "Gulf of America" ecosystem status](https://ecowatch.noaa.gov/regions/gulf-of-america)
- [GSMNP ATBI (DLIA)](https://dlia.org/about/atbi/) · [Atlas of the Smokies species mapper](https://species.atlasofthesmokies.org/) · [GRSM science & research](https://home.nps.gov/grsm/learn/scienceresearch.htm) · [1,000th new species coverage](https://www.nationalparkstraveler.org/2018/10/great-smoky-mountains-national-parks-biodiversity-inventory-reaches-1000-species-mark) · [Senate hearing record on the ATBI](https://www.govinfo.gov/content/pkg/CHRG-110shrg45337/html/CHRG-110shrg45337.htm)

## Latin America & the Caribbean — Comprehensive Place/Population Documentation Projects

**Method note:** WebSearch budget for this session was exhausted (200/200) early, so the bulk of verification here is **direct WebFetch against live URLs plus DNS/TCP probing**, all performed **2026-08-25**. This actually strengthened the dead-project hunt (DNS non-resolution is hard evidence) but limited discovery of projects whose URLs I could not construct. Every figure below is either quoted from a fetched page or explicitly marked unverified. Many Latin American portals are JavaScript-only single-page apps that return no server-side content — where that blocked figure extraction I say so rather than guessing.

---

### Regional / multi-country (Amazon basin & pan-Latin American)

#### Amazon basin (9 countries)

| Project (URL) | Scale | Team size & type | Scope (occurrence-only vs multi-domain) | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [RAISG — Amazonia Socioambiental](https://www.amazoniasocioambiental.org/en/) | Pan-Amazon, 9 countries | Network of 9 named CSO members: FAN (Bolivia), Imazon, ISA (Brazil), Gaia Amazonas (Colombia), EcoCiencia (Ecuador), IBC (Peru), Provita, Wataniba (Venezuela) | **Multi-domain**: indigenous territories, protected areas, hydroelectric dams, roads, oil & mining concessions, illegal mining, deforestation, fires/burned area, biomes, basins, wetlands | Amazon socio-environmental landscape | **Active.** Core layers (protected areas, indigenous territories, mining, roads) updated **2025**; publications June–July 2026 on wetlands, water vulnerability, climate. Fetched 2026-08-25 | **Best structural analogue in the region for the user's project.** Its whole reason for existing is reconciling 9 incompatible national datasets with differing definitions of "Amazonia." Note **uneven layer currency**: deforestation, fires and illegal-mining layers still dated **2020** while others are 2025 — a real, visible staleness-provenance problem worth studying |
| [MapBiomas](https://www.maapprogram.org/) → see note; MapBiomas at [mapbiomas.org](https://mapbiomas.org/) | Brazil + Amazonia + Chaco + Pampa + Bosque Atlántico + per-country editions (Peru, Bolivia, Colombia, Ecuador, Venezuela, Argentina, Uruguay, Indonesia) | Large distributed academic/NGO/tech network; **Peru edition coordinated by Instituto del Bien Común (IBC)** | Multi-domain remote sensing: land cover/use, deforestation alerts, fire, water, glaciers, mining, irrigation | Land-cover change, 1985–2025 | **Very active.** **MapBiomas Peru Collection 4 released 20 Aug 2026**, covering 1985–2025 (41 years). Fetched 2026-08-25 | Numbered, citable **"Collections"** — a versioning model that lets successive contradictory estimates coexist and be compared. Cross-cuts protected areas and indigenous territories as analysis units. Peru figures: 3.7 Mha natural vegetation lost (4%); mining area +2,510% nationally, ×190 in Amazonia; palm oil +1,075% |
| [MAAP — Monitoring of the Andean Amazon Project](https://www.maapprogram.org/) | Peru, Bolivia, Brazil, Colombia, Ecuador, Venezuela — "100% of the Amazon Basin" | Amazon Conservation initiative; small analytical team | Multi-domain monitoring: deforestation, fires, illegal gold mining, roads | Amazon deforestation hotspots | **Very active.** **249 numbered reports** published; most recent MAAP #249, "Gold Mining Deforestation in the Ecuadorian Amazon: Napo Province." Fetched 2026-08-25 | **Domain migration caught in the act**: `maaproject.org` now 302-redirects to `maapprogram.org` — a live example of link rot the user should plan for. Each report is a permanent numbered artefact, so superseded figures remain citable |

#### Pan-Latin American thematic

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Environmental Justice Atlas (EJAtlas)](https://ejatlas.org/) | Global, heavily Latin American | Academic (ICTA-UAB lineage); front end built by Geomatico | Multi-domain conflict cases: actors, claims, commodities, outcomes, sources | Environmental conflicts | **Live** (site served 2026-08-25) but **JavaScript-only** — no server-rendered content, so case counts, leadership and launch year **could not be verified** in this session | Purpose-built around **contested claims**: each case records rival accounts from companies, states and affected communities. Directly relevant to a disputed-figures feature, but treat all numbers as unverified here |
| [HGIS de las Indias](https://www.hgis-indias.net/) | Spanish America, **1701–1808** | Originally Univ. Graz, funded by Austrian FWF **2015–2019**; WebGIS support from EHESS; **now hosted by Yale University Economic Growth Center** | Multi-domain historical GIS: **gazetteer/encyclopedia of places and concepts**, administrative boundaries (dioceses, audiencias, intendencias, provinces), time-slider web-GIS, downloadable datasets via "Indias Dataverse" | Colonial Spanish American geography | **Live but post-funding / custodial.** Grant ended 2019; site now maintained by a different institution than the one that built it. Fetched 2026-08-25 | **The single closest analogue to the gazetteer + colonial-archive-mining component** of the Sathyamangalam atlas. Explicitly crowdsources correction: invites users to "locate places, correct boundaries, add knowledge." Institutional rehoming after grant end is a survival model worth copying. Exact place counts and named authors not stated on the landing page (**unverified**) |

---

### Mexico

#### Mexico

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [SNIB — Sistema Nacional de Información sobre Biodiversidad](https://conabio.snib.mx/) | National | CONABIO, government commission (est. 1992), large staff + expert networks | **Deeply multi-domain**: 50.7 M occurrence records; **3,700+ technical fact sheets for native species and 1,300+ for exotics**; **18,000+ thematic maps**; **620,000+ remote-sensing images**; **155,000+ photographs and illustrations**; 994 projects organised into 1,177 databases | Mexican biodiversity | **Active.** Page states **50,710,691 occurrence records of 118,652 species**, **last updated 24 February 2026**. Fetched 2026-08-25 | **The strongest overall model in Latin America for the user's atlas.** Three parallel access interfaces over one corpus — **geographic (Geoportal), taxonomic (EncicloVida), and project/provenance-based** — meaning every record is traceable to a funded project and database. That project-level provenance layer is exactly what a disputed-figures feature needs underneath it |
| [EncicloVida](https://enciclovida.mx/) | National | CONABIO | Multi-domain species pages: taxonomy + full synonymy, **common names in Spanish and indigenous languages**, potential-distribution maps, endemism, **national AND international risk categories**, priority and invasive species lists, photos/video/audio, citizen-science observations from Naturalista and AverAves, **uses and agrobiodiversity** | 118,000+ valid/accepted species | **Live**; no last-updated date exposed on the landing page (**unverified currency**, though SNIB behind it is Feb 2026). Fetched 2026-08-25 | **Displays national (NOM-059) and international (IUCN) conservation categories side by side** — a working precedent for the disputed-figures feature, where two authorities legitimately disagree about the same species. Also a rare production example of **indigenous-language name fields as first-class data**, relevant to a Tamil/Irula gazetteer |
| [Geoportal CONABIO](http://www.conabio.gob.mx/informacion/gis/) | National | CONABIO | Geospatial layer library (**17,518 layers indicated in the listing title**) | Mexican environmental geodata | **Live** (DNS + search listing). Layer count from listing metadata, not fetched page — **treat as indicative** | Companion GIS layer catalogue to SNIB |
| [Naturalista](https://www.naturalista.mx/) | National | CONABIO, national iNaturalist node | Occurrence-only (citizen science observations + photos) | All taxa | **Live but returned HTTP 403** to automated fetch 2026-08-25; existence corroborated as a data source cited by EncicloVida | National iNaturalist deployment feeding the national system — the federated-citizen-science pattern |
| [Atlas de los Pueblos Indígenas de México (INPI)](https://atlas.inpi.gob.mx/) | National | INPI (federal indigenous peoples' institute) | Intended multi-domain: peoples, territories, language, population, history, culture | Indigenous peoples of Mexico | **Host resolves and serves**, but robots.txt/TLS blocked automated content fetch on both http and https 2026-08-25 — **content, counts and last-update unverified** | Government-run indigenous documentation atlas; a governance model (state-run rather than NGO-run) contrasting with Brazil's ISA |
| [Biblioteca Digital de la Medicina Tradicional Mexicana (UNAM)](http://www.medicinatradicionalmexicana.unam.mx/) | National | UNAM (Instituto de Investigaciones Antropológicas) | Multi-domain traditional-knowledge corpus: medicinal plant atlas, encyclopedic dictionary, indigenous medical terminology | Traditional medicine & medicinal flora | **Degraded.** DNS OK, but fetch failed with **TLS hostname-mismatch on `www.` and HTTP 503 on the apex domain** (2026-08-25) — clear neglect signals on a university-hosted resource. Content **unverified** | Important precedent for **linking species records to traditional/ethnobotanical knowledge and indigenous-language terminology** — the same join the user needs. Its decay is a cautionary tale about university hosting |
| [Mariposas Mexicanas](https://mariposasmexicanas.com/) | National, single taxon | Small/volunteer (**unverified**) | Likely occurrence + images + checklists (**unverified**) | Mexican butterflies | **DNS resolves only**; no content verified 2026-08-25 | Lead for the small/solo category; needs manual follow-up |

---

### Central America

#### Costa Rica

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| **INBio + Atta + CRBio** — `inbio.ac.cr`, `atta.inbio.ac.cr`, `crbio.cr`, `especies.crbio.cr` | National, was Latin America's largest | INBio: large NGO/parastatal, founded 1989 | Was fully multi-domain: specimen records, species pages, bioprospecting, taxonomy | Costa Rican biodiversity | **DEAD — see Dead/dormant section. All four domains failed DNS resolution on 2026-08-25** | The definitive cautionary case; detailed below |
| [Janzen & Hallwachs ACG caterpillar/parasitoid inventory](http://janzen.sas.upenn.edu/) + [caterpillars.org](https://www.caterpillars.org/) | Single landscape: Área de Conservación Guanacaste | **Effectively a two-person project** (Daniel Janzen & Winnie Hallwachs) with ACG parataxonomists | Multi-domain: caterpillar rearing records, host plants, parasitoid associations, photographs, DNA barcodes, ACG documentary history | ACG Lepidoptera & parasitoids | **Hosts reachable (TCP 80/443 open) but content not verifiable**: `janzen.sas.upenn.edu` robots.txt fetch timed out; `caterpillars.org` failed TLS chain validation. 2026-08-25. **Counts, leads and currency unverified via automated access** | The archetypal **decades-long, tiny-team, single-landscape total inventory** — closest in spirit to a one-reserve atlas. Its fragile hosting (a personal directory on a university web server) is itself the lesson |
| [Organization for Tropical Studies](https://ots.ac.cr/) | La Selva, Palo Verde, Las Cruces stations | Research consortium | Station datasets, herbarium, bibliography (**unverified**) | Costa Rican field stations | **DNS resolves only**; content unverified 2026-08-25 | Long-running single-site data lineage; needs follow-up |
| [SINAC](https://www.sinac.go.cr/) / [Museo Nacional de Costa Rica](https://www.museocostarica.go.cr/) | National | Government | Protected-area legal registry; the **inheritor of INBio's 3.7 M specimens and Atta records** | Protected areas; national collections | **DNS resolves**; content unverified 2026-08-25 | Where the INBio data legally landed |

#### Panama

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [STRI research databases](https://stri.si.edu/research-computing/databases) | Panama + Neotropics + single sites | Smithsonian Tropical Research Institute, institutional | **Verified hosted set**: Shorefishes of the Eastern Pacific; Shorefishes of the Greater Caribbean; Panama Biota (Symbiota); Bryozoans of the Pacific; Círculo Herpetológico de Panamá; Euglossine (Orchid) Bees; ForestGEO; STRI GIS Portal; Physical Monitoring Program; **Galeta Oil Spill Project**; Pollen Morphological Database; Tropical Temperature Database | Panama & Neotropical biota, paleo, long-term monitoring | **Live.** Database index fetched 2026-08-25; **no record counts, dates or named leads exposed on the index page** | A **federated cabinet of many small databases under one institutional roof** rather than one monolith — a realistic architecture for a modest-budget atlas. The Galeta Oil Spill Project and Physical Monitoring Program show **long-term-monitoring and event-archive** material sitting beside taxonomic databases. `biogeodb.stri.si.edu` now 302s to this index (link-rot precedent) |
| [Panama Biota (Symbiota)](https://panamabiota.org/stri/) | National | STRI; self-described as **"the developing STRI Symbiota Portal"** | Specimen records + images across plants, insects, fish, reptiles, birds | Panamanian biota | **Live but explicitly still "developing"**; no record counts, collection list or dates on the landing page. Fetched 2026-08-25 | Notable as a **Symbiota** deployment — off-the-shelf collections software, a cheaper alternative to ALA-scale infrastructure |
| [ForestGEO (incl. Barro Colorado Island)](https://forestgeo.si.edu/) | Single 50-ha plots, global network incl. BCI, Yasuní | Smithsonian network | Long-term forest census: stems, mortality, growth, recensus over decades | Tropical forest dynamics | **DNS resolves**; content not fetched in this session (**unverified**) | The single-site, repeat-census model — successive censuses of the same plot inherently produce differing counts, handled by census versioning |

#### Mesoamerica-wide

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Flora Mesoamericana (Tropicos/MBG)](https://www.tropicos.org/project/fm) | Southern Mexico to Panama | Missouri Botanical Garden with UNAM and NHM London (**collaboration unverified in this session**) | Botanical treatments: descriptions, keys, synonymy, distribution | Mesoamerican vascular flora | **Robots.txt disallowed** automated fetch 2026-08-25; `tropicos.org` DNS OK. Volumes, counts and editors **unverified** | Long-running multinational floristic project on shared infrastructure (Tropicos) |
| [Healthy Reefs for Healthy People](https://www.healthyreefs.org/) | Mesoamerican Reef (Mexico, Belize, Guatemala, Honduras) | NGO-led partnership | Reef health indicators, report cards, site surveys | Mesoamerican Reef | **Live**; fetched 2026-08-25 but landing page carried **no site counts, indicator list, years or lead names** — those figures **unverified** | Periodic **"report card"** publication model — repeated scoring of the same sites over time, an alternative to a live database |

---

### Caribbean

#### Regional / island-wide

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Caribbean Herpetology](https://www.caribbeanherpetology.org/) | West Indies (Cuba, Jamaica, Hispaniola, Puerto Rico, Bahamas, Caymans, Lesser Antilles) | **Very small, named**: Editor-in-Chief & publisher **S. Blair Hedges** (Temple University); associate editors **Robert W. Henderson** (Milwaukee Public Museum), **Robert Powell** (Avila University), **R. Graham Reynolds** (UNC Asheville) | **Multi-domain and unusual**: a peer-reviewed open-access **journal fused with a species database and checklist** — ~750 species accounts, **~2,000 images**, distribution maps by island, selected **frog vocalizations** | Caribbean amphibians & reptiles | **Active, 16 continuous years.** Launched **August 2010** (Issue 1); most recent **Issue 100, June 2026**. ISSN 2333-2468. Fetched 2026-08-25 | **The standout small-project model for the user.** It solves the provenance problem by *publishing* new records as citable peer-reviewed notes that simultaneously update the database — so every datum in the atlas has a formal citation. Directly transferable to a tiger-reserve atlas needing defensible records. Sustained by 4 named academics with no large institution |
| [Digital Library of the Caribbean (dLOC)](https://www.dloc.com/) | Pan-Caribbean, multilingual | Consortium hosted on University of Florida digital-collections infrastructure | Multi-domain digitised archive: newspapers, colonial and government documents, maps, manuscripts (**material breakdown unverified**) | Caribbean documentary heritage | **Live** but the UF Digital Collections front end is **JavaScript-only**; item counts, partner count and launch year **unverified** 2026-08-25 | The regional model for **colonial-archive digitisation at consortium scale** — relevant to the historical-document-mining component. Runs on shared university repository infrastructure rather than bespoke code |
| [AGRRA — Atlantic & Gulf Rapid Reef Assessment](https://www.agrra.org/) | Wider Caribbean | Distributed survey network | Reef survey occurrence + condition data | Caribbean reefs | **DNS resolves only**; content unverified 2026-08-25 | Long-running standardised-protocol reef database; follow-up needed |

#### Cuba / Bermuda / Trinidad & Tobago

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Catálogo de la Flora de Cuba](https://catalogofloradecuba.wordpress.com/) | National | Small/solo (**unverified**) | Floristic checklist (**unverified**) | Cuban flora | **DNS resolves only** 2026-08-25; content unverified | Notable purely as a **WordPress.com-hosted national flora catalogue** — the minimum-viable-platform end of the spectrum. `floradecuba.com` and `avesdecuba.com` **do not resolve** |
| [Bermuda Department of Environment & Natural Resources](https://environment.bm/) | Single small island | Government | Species and habitat documentation (**unverified**) | Bermuda biota | **DNS resolves only** 2026-08-25 | Small-island total-documentation lead |
| Online Guide to the Animals of Trinidad & Tobago (UWI St. Augustine) | National | University course-based, student-authored species accounts (**unverified**) | Species accounts: description, distribution, ecology, references | T&T fauna | **Could not locate.** My constructed URL under `sta.uwi.edu` returned a **404** on 2026-08-25. Existence and status **unverified** | Interesting model if confirmed: species accounts written as graded student coursework — a low-cost content-generation pattern. Needs manual search |

---

### Brazil

#### Brazil

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [SiBBr](https://sibbr.gov.br/) | National | Federal government (MCTI/RNP lineage), institutional; **196 contributing institutions** | Multi-domain: occurrence records, **Spatial Portal** (integrated environmental-layer mapping), natural-collections catalogue, species lists by region and theme, iNaturalist citizen science, Biodiversity for Food & Nutrition module | Brazilian biodiversity | **Active.** **36,509,671 occurrence records**, 196 institutions (fetched 2026-08-25). GBIF publisher **since 20 November 2013**, 32 datasets published to GBIF | **Runs an Atlas of Living Australia–derived stack** — evidenced by `collectory.sibbr.gov.br` collection pages and an "Atlas SiBBr" section. Since the user excluded ALA/NBN themselves, SiBBr is the useful proof that **the ALA codebase is deployable by a middle-income national government** — a concrete tech-stack option. Species counter rendered as 0 (front-end bug observed 2026-08-25) |
| [speciesLink (CRIA)](https://specieslink.net/) | National + international contributors | **CRIA — a civil-society organisation, autonomously managed**, small technical team | Multi-domain: specimen records from biological collections, specimen images, environmental layers (land cover, bioclimatic), **taxonomic name-verification tools**, **geographic coordinate validation tools**, data filtering/qualification | Plants, animals, fungi, microorganisms | **Live.** Established **2002**; "millions of records." No last-updated date exposed. Fetched 2026-08-25 | **Directly relevant tooling**: it ships explicit **name-verification and coordinate-validation services** — i.e. machinery for detecting and flagging *bad or conflicting* records rather than silently ingesting them. Self-describes as **proprietary technology built for local needs** rather than adopting foreign platforms — the opposite architectural bet from SiBBr, and both survive. Notably, 24 years old and run by civil society, not the state |
| [Flora e Funga do Brasil](https://floradobrasil.jbrj.gov.br/) (data via [IPT](https://ipt.jbrj.gov.br/jbrj/resource?r=lista_especies_flora_brasil)) | National | Instituto de Pesquisas Jardim Botânico do Rio de Janeiro; **~1,000 Brazilian and foreign taxonomists** collaborating via an online submission platform | Multi-domain taxonomic monographs: nomenclature, synonymy, descriptions, keys, distribution, plus 6 extension tables | Plants, algae, fungi | **Active but check currency.** **151,267 core taxonomy records; 797,418 records across 6 extensions; version 393.397 published 1 January 2024.** Fetched 2026-08-25 | **A masterclass in versioning contested taxonomy**: a four-digit version number (393.397) on a continuously edited checklist, each version citable. Project lineage is itself documented — "Flora do Brasil 2020" (launched 2016) → renamed **"Flora e Funga do Brasil" in 2022** to adopt more inclusive biological terminology. The ~1,000-specialist distributed authorship model is how a small core team scales expert review. Main site is **robots.txt-disallowed** — data must be reached via the IPT |
| [Reflora Virtual Herbarium](https://reflora.jbrj.gov.br/reflora/herbarioVirtual/) | National, with repatriated foreign holdings | JBRJ; funded by CNPq, CAPES, FNDCT, **seven state research foundations** (FAPEAM, FAPESB, FAP-DF, FAPEMIG, FAPESC, FAPESP, FAPERJ), Fundação Araucária, and corporates **Natura** and **Vale S.A.** | Digitised specimen images with label transcription | Brazilian plant specimens | **Live.** **Over 2 million specimen images**; **8 foreign herbaria** repatriated digitally: **Kew, MNHN Paris, RBG Edinburgh, Missouri, New York, Swedish Museum of Natural History, Smithsonian, NHM Vienna**. Fetched 2026-08-25 (site itself robots-disallowed; verified via CNPq programme page) | **The flagship precedent for digitally repatriating colonial-era collections** — exactly the historical-document-mining problem for an Indian reserve whose type specimens sit in Kew and Paris. Also a template for **blended public-private funding** (federal + 7 state agencies + Natura + Vale) |
| [WikiAves](https://www.wikiaves.com.br/) | National | **Community-run**, no institutional host identified | Multi-domain: photo and sound records, species pages, distribution maps, **municipality-level record browsing**, active forum, curated "first record" sections | Brazilian birds | **Highly active.** **6,546,363 photo and sound records; 57,304 registered observers; 1,975 species.** Operating since **2008**; footer shows 2026; new submissions within the last 7 days. Fetched 2026-08-25 | **Municipality-level aggregation** is the key transferable idea — administrative-unit rollups, not just point maps, which is how local users actually ask questions. 18 years of purely community governance at 6.5 M records, with no state funding — the strongest counter-example to "you need a national institute" |
| [SALVE — ICMBio](https://salve.icmbio.gov.br/) | National | ICMBio (federal biodiversity institute) | Extinction-risk assessments: threat categories, taxa across mammals, birds, reptiles, amphibians, fish | Brazilian fauna red-listing | **Live**, version **v1.2.4** in page metadata; species counts, assessment-cycle detail and dates **not exposed server-side** (2026-08-25) | An assessment system built around **successive assessment cycles for the same species** — structurally a disputed-figures store, since a taxon's category legitimately differs between cycles. Worth a manual look for the user's conflicting-data feature |
| [Terras Indígenas no Brasil](https://www.terrasindigenas.org.br/) | National | **Instituto Socioambiental (ISA)**, NGO | **Multi-domain and legally granular**: 838 territories, each tracked through demarcation stages, plus photo archives, maps, news archive; filters by territory, FUNAI office, people, legal situation, biome, municipality | Indigenous lands of Brazil | **Live.** **838 indigenous territories**; indigenous lands = **14.8% of national territory**; **Drupal 9**; news archive entries to **2024** (so the news stream may be lagging). Fetched 2026-08-25 | **The best regional model for a legal/government-instrument register.** It does not store a single "status" but **seven distinct legal phases** — Em Identificação, Identificada, Declarada, Reservada, Homologada, Registrada, Encaminhada RI (15 lands) — so a territory's contested legal standing is the data model itself, not a footnote. **Drupal is a realistic, boring, maintainable stack** for a comparable atlas |
| [Povos Indígenas no Brasil](https://pib.socioambiental.org/) | National | Instituto Socioambiental (ISA) | Encyclopedia of peoples, languages, lands, rights, initiatives | Indigenous peoples of Brazil | **Live**; landing page yielded only section structure — **peoples count, language count, population figures and update frequency unverified** 2026-08-25 | Companion encyclopedia to Terras Indígenas. Its "Quantos são?" (How many are they?) sections are reported to compile **differing population estimates across censuses and sources** — a promising conflicting-figures precedent, but **I could not verify this** and flag it for manual checking |
| [Mapa de Conflitos: Injustiça Ambiental e Saúde](https://mapadeconflitos.ensp.fiocruz.br/) | National | **Fiocruz / Escola Nacional de Saúde Pública (ENSP)**, academic | Multi-domain conflict records: location, affected populations (**50+ categories** incl. indigenous peoples, quilombolas, fisherfolk, waste workers), conflict drivers (**40+ activity types**: mining, monocultures, dams, petrochemicals), health harms (**16 categories**), socio-environmental damages (**20+ categories**), per-case sources | Environmental-justice conflicts | **Live, recently maintained.** **694 documented conflicts** across all **27** states; **last update 7 November 2025.** Fetched 2026-08-25 | **Exemplary controlled vocabularies** — the 50/40/16/20-category taxonomies are the reason this dataset is queryable rather than a pile of prose. Explicitly frames its mission as amplifying marginalised accounts, i.e. **deliberately preserving the version of events that official figures omit** — the philosophical core of a disputed-data feature |
| [Instituto Mamirauá](https://www.mamiraua.org.br/) | Two reserves: Mamirauá & Amanã | Research institute (federal MCTI unit) | Multi-domain: **Biblioteca Henry Walter Bates**, "acervos e coleções" (archives and collections), real-time reserve monitoring systems, tracking systems, community co-management datasets | Central Amazon flooded forest | **Active.** Self-described as the **"first reserve in the world monitored in real time"**; pirarucu co-management flagship; **17th Managers' Meeting**; "Recomendações Seca 2026" (2026 drought recommendations). Fetched 2026-08-25 | **Single-protected-area total documentation** — the closest Brazilian analogue in *scale* to a one-tiger-reserve atlas. Combines a **named historical library (after naturalist H.W. Bates)** with live monitoring and community-based monitoring — the same span of source types the user wants. Specific dataset figures and named leads **unverified** |
| Catálogo Taxonômico da Fauna do Brasil ([fauna.jbrj.gov.br](http://fauna.jbrj.gov.br/fauna/)) | National | JBRJ platform, distributed taxonomists | Faunal taxonomic checklist | Brazilian fauna | **Host resolves but entered a redirect loop** on fetch 2026-08-25 — **counts, contributors and version unverified**; possible front-end fault | The zoological counterpart to Flora e Funga on the same institutional platform |
| [CNUC — Cadastro Nacional de Unidades de Conservação](https://cnuc.mma.gov.br/) | National | Ministério do Meio Ambiente | Protected-area legal registry | Brazilian protected areas | **DNS resolves only**; content unverified 2026-08-25 | Federal legal registry of protected areas — the instrument-database analogue |

---

### Andean South America

#### Colombia

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [SiB Colombia](https://biodiversidad.co/) | National | National node coordinated via Instituto Humboldt; large publisher network | Multi-domain: biological records, **Catálogo de la Biodiversidad**, species checklists, online collections catalogue, species datasheets | Colombian biodiversity | **Active.** News stream shows contributions **January–July 2026** from indigenous communities, universities, botanical gardens and NGOs (camera-trap monitoring, fungi, insects). Fetched 2026-08-25 | **Numbers deliberately withheld here.** My fetch returned obviously corrupted placeholder figures (repeated "100,000" across unrelated counters); the only plausible datum was **~280 publishing partners**, and even that is **unverified**. `biodiversidad.co/news/` returned a redirect loop. Notable that **indigenous community organisations appear as named data publishers** — a participatory-publishing model directly relevant to community-based monitoring |
| [Catálogo de la Biodiversidad de Colombia](https://catalogo.biodiversidad.co/) | National | Instituto Humboldt; SiB has publicly recruited front-end/back-end developers for it | Species datasheets: descriptions, distribution, uses, common names, threats (**unverified**) | Colombian species | **Live but a JavaScript-only SPA** returning no server-side content (2026-08-25) — all content claims **unverified** | Interesting governance detail: SiB **openly advertised for developers** for this product, indicating small in-house dev capacity — realistic for the user's scale |
| [Catálogo de Plantas y Líquenes de Colombia](https://catalogoplantasdecolombia.unal.edu.co/) | National | Instituto de Ciencias Naturales, Universidad Nacional, Bogotá; large distributed author/reviewer list (site has a "Lista de autores y revisores" page) | Taxonomic catalogue with filters on family, genus, species, **habit, origin, biogeographic region, elevation range, department, conservation status, endemism** | Colombian plants and lichens | **Live but stale: page states "Last update 09/02/2023"** — no update in ~3.5 years as of 2026-08-25 | The rich **facet set (habit × origin × elevation × department × endemism)** is a good model for a reserve-scale catalogue. Species/genus/family totals and named editors are on a "Presentación de la obra" page that **returned 404** when I fetched it — figures **unverified**. A live example of a valuable catalogue drifting into dormancy |
| [BioModelos](http://biomodelos.humboldt.org.co/) | National | Instituto Alexander von Humboldt + thematic expert groups (e.g. **Asociación Primatológica Colombiana**) | Expert-curated species distribution models plus curation of underlying biological records | Colombian species distributions | **Live**; most recent milestone described is the Primates thematic group completing **38 validated species models** feeding an "Atlas of Biodiversity of Colombia." Named leads and launch year **not stated** (2026-08-25) | **The most directly relevant conflicting-data mechanism I found.** Users **"qualify the geographic distribution hypotheses available for species"** — the system stores *multiple competing distribution hypotheses per species* and lets experts rate them, rather than forcing one authoritative map. That is precisely the "disputed figures side by side" pattern, implemented in production |
| [RUNAP](https://runap.parquesnacionales.gov.co/) | National | Parques Nacionales Naturales de Colombia | Protected-area legal registry (area, category, creating instrument) | Colombian protected areas | **Live but JavaScript-only**, no server-side content — area counts, hectares, categories and whether creating decrees are recorded all **unverified** 2026-08-25 | The national single-registry model for protected-area legal instruments |
| [SIAM — INVEMAR](https://siam.invemar.org.co/) | Marine & coastal | INVEMAR, marine research institute | Marine environmental information system (**unverified**) | Colombian marine biodiversity | **DNS resolves only**; content unverified 2026-08-25. Note **`sibm.invemar.org.co` does NOT resolve** — the marine biodiversity sub-portal I sought may have been retired or renamed | Marine counterpart to the terrestrial national system |

#### Ecuador

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [BioWeb Ecuador](https://bioweb.bio/) | National | **Pontificia Universidad Católica del Ecuador (PUCE)**; small per-portal author teams. ReptiliaWeb authors: **Omar Torres, Gustavo Pazmiño, David Salazar** | Multi-domain, federated into **FaunaWeb, FloraWeb, FungiWeb** sub-portals; per-portal: species lists, taxonomic index, photo galleries, downloadable **PDF field guides**, distribution mapping, statistics pages, organisation by **biogeographic and natural regions of Ecuador** | Ecuadorian fauna, flora, fungi | **Live.** Landing page and ReptiliaWeb fetched 2026-08-25. **Species counts not rendered** — the "Número Especies" statistics page returned navigation only; version/date **unverified** | **The best model in the region for a modest-team, university-hosted, taxon-federated atlas.** Each taxonomic portal has its own named 2–3 person author team under a shared brand and platform — a governance pattern that scales expertise without a big institution. Its organisation **by biogeographic region** rather than politically is directly applicable to a reserve landscape |
| [Galápagos Species Checklist / dataZone](https://datazone.darwinfoundation.org/en/checklist) | Single archipelago | **Charles Darwin Foundation**; source database "curated by researchers from around the world over decades" | Multi-domain dataZone: **Species Database**, **Climatology Database**, **Geoportal**, and **downloadable archives of previous published checklists**. Species filters by IUCN category (9 options) and by **origin: endemic, native, migrant, vagrant, introduced, cryptogenic** | Galápagos biota | **Live.** Fetched 2026-08-25 (via 302 from `darwinfoundation.org/en/datazone/checklist`). **Total taxon count, named compilers and update dates not exposed** on the checklist page | **Two features worth copying directly.** (1) It publishes **archived earlier checklist versions alongside the current one** — so superseded species totals stay retrievable and comparable, a ready-made disputed-figures mechanism. (2) The **"cryptogenic"** origin category is an explicit, first-class value for *"we genuinely do not know if this is native or introduced"* — institutionalised uncertainty rather than a forced guess. Backed by the **CDRS Natural History Collections** database, tying every claim to specimens |

#### Peru

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [SINIA](https://sinia.minam.gob.pe/) | National | Ministerio del Ambiente | National environmental information system (**unverified**) | Peruvian environment | **DNS resolves only**; content unverified 2026-08-25 | National environmental data hub |
| [SNIFFS / SERFOR geoportal](https://geo.serfor.gob.pe/) | National | SERFOR (forest & wildlife service) | Forest and wildlife information, incl. licensing/permits (**unverified**) | Peruvian forests & wildlife | **DNS resolves only** (`sniffs.serfor.gob.pe`, `geo.serfor.gob.pe`); `mapas.serfor.gob.pe` **does not resolve**. 2026-08-25 | Forest-sector legal/licensing register analogue |
| [SERNANP](https://www.sernanp.gob.pe/) | National | Protected areas service | Protected-area registry | Peruvian protected areas | **DNS resolves only**; unverified 2026-08-25 | Peru's protected-area authority |
| MapBiomas Peru — see Regional | National | **Instituto del Bien Común (IBC)** coordinating | Land cover/use 1985–2025 | Peruvian land change | **Collection 4 released 20 August 2026** | Freshest verified activity of any project in this report |

#### Bolivia

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Herbario Nacional de Bolivia (LPB)](https://herbariolpb.umsa.bo/) | National | UMSA, university herbarium | Herbarium collection data (**unverified**) | Bolivian flora | **DNS resolves only**; content unverified 2026-08-25 | Note `bolivia.fan-bo.org` (Fundación Amigos de la Naturaleza subdomain) **does not resolve** — possible retirement |

#### Venezuela

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Libro Rojo de la Fauna Venezolana](https://especiesamenazadas.org/) | National | **Provita** (NGO) in partnership with **Fundación Empresas Polar** | Species accounts browsable **by taxonomy and by threat category**; entries carry scientific names, descriptions, photographs with credits, links to detailed datasheets | Threatened Venezuelan fauna | **Live but almost certainly dormant.** First published **1995**; **fourth edition 2015**; site **copyright notice reads 2020** — no evidence of activity in ~6 years as of 2026-08-25 | A **corporate-philanthropy-funded red list** that successfully moved a printed book series online across four editions (1995→2015), then stalled. Its edition history is a natural conflicting-figures dataset (categories change between editions). Species totals and named editors **not on the landing page — unverified** |

---

### Southern Cone

#### Argentina

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [SIB — Sistema de Información de Biodiversidad, Administración de Parques Nacionales](https://www.sib.gob.ar/) | National protected-area system, **organised per protected area** | Administración de Parques Nacionales, government | Intended multi-domain: per-park species checklists, occurrence records, bibliography, maps (**unverified**) | Argentine national parks' biodiversity | **Live** — server returned the page title "SIB \| Sistema de Información de Biodiversidad, Parques Nacionales, Argentina" on 2026-08-25, but the body is client-rendered, so **record counts, park coverage, launch year and currency are unverified** | **Conceptually the closest government analogue to the user's project**: a national system whose primary organising unit is *the individual protected area*, aggregating species, records and literature per reserve. Worth manual investigation as a direct template for a single-reserve atlas |
| [EcoRegistros](https://www.ecoregistros.org/) | Argentina-centred, nominally global | **Essentially solo: founded and run by Jorge La Grotteria** | Multi-domain: photo, video and audio records, checklists, batch submissions, ID-request forums, species pages, **documentaries** section, **migration tracking**; organised by kingdom (animals, plants, fungi) | All taxa, Neotropical emphasis | **Active, 15 years.** Founded **2011**; **over 2.3 million records**; **adopted AviList global taxonomic standard in 2026**; published autumn-2026 Tierra del Fuego marine bird survey findings. Fetched 2026-08-25 | **The most impressive solo project found in this scope** — 2.3 M records from one person's platform. Its **2026 migration to AviList** shows a one-person project executing a full taxonomic-backbone swap, which is exactly the kind of upheaval a disputed-taxonomy feature must absorb. Strong evidence that solo multi-domain portals are viable at scale |
| [Flora Argentina](http://www.floraargentina.edu.ar/) / [search](https://buscador.floraargentina.edu.ar/) / [Instituto Darwinion](https://www.darwin.edu.ar/) | National | Instituto de Botánica Darwinion (CONICET–ANCEFN) | Vascular flora catalogue: synonymy, provincial distribution, habit, keys, images (**unverified**) | Argentine vascular plants | **Hosts resolve but content inaccessible to automated fetch**: `www.floraargentina.edu.ar` failed **TLS hostname verification**; `buscador.floraargentina.edu.ar` is **robots.txt-disallowed**. 2026-08-25. Counts, editors, currency **unverified** | The authoritative national flora; the TLS misconfiguration is a minor neglect signal |
| [SNDB — Sistema Nacional de Datos Biológicos](https://www.sndb.mincyt.gob.ar/) | National | MINCYT/CONICET, government | Occurrence records + collections catalogue (**unverified**) | Argentine biodiversity data | **Host resolves via `www.`, but all connection attempts failed** during fetch on 2026-08-25 — **possibly down or firewalled**. Bare `sndb.mincyt.gob.ar` also resolves. Status **uncertain, flagged for manual check** | Argentina's national aggregator; apparent reachability problems warrant follow-up |
| [Aves Argentinas](https://www.avesargentinas.org.ar/) | National | NGO (founded 1916) | Bird conservation; publisher of the **Atlas de las Aves Nidificantes de la Argentina** (**publication details unverified**) | Argentine birds | **DNS resolves only**; content unverified 2026-08-25 | The breeding-bird-atlas lineage the user asked about; needs manual verification of the atlas's digital availability |

#### Chile

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [SIMBIO](https://simbio.mma.gob.cl/) | National | Ministerio del Medio Ambiente; transitioning to **SBAP (Servicio de Biodiversidad y Áreas Protegidas)** | **Multi-domain**: species records, protected areas, terrestrial and marine ecosystems, wetlands inventory, **species recovery and conservation plans**, **ecological restoration projects**, watersheds (cuenca/subcuenca/subsubcuenca hierarchy); modules for geoportal, indicators, and regional views | Chilean biodiversity policy data | **Active and mid-transition.** Page states that **from 2 February 2026, SBAP becomes the body responsible for providing official data** under **Law 21.600**; SIMBIO continues as an interoperability/metadata layer, gradually synchronising with SBAP sources. Fetched 2026-08-25 | **A live, documented institutional handover of a national biodiversity portal** — invaluable precedent for the user on how to keep an atlas alive across a change of custodian: SIMBIO explicitly degrades into an *interoperability and metadata* service rather than shutting down. Including **conservation plans and restoration projects** as first-class datasets alongside species is an unusually policy-aware scope |
| [Clasificación de Especies según Estado de Conservación](https://clasificacionespecies.mma.gob.cl/) | National | Ministerio del Medio Ambiente, with public participation | Legal conservation classification: per-process species lists, regulatory history, downloadable consolidated list, public-consultation documents | Chilean species' legal conservation status | **Active.** **20 finalised classification processes** as of **June 2026**; **21st Process (2025) in public consultation** for preliminary classifications. Legal basis **Decreto N° 29 de 2011**, which replaced **Decreto N° 75 de 2004**. Fetched 2026-08-25 | **The single best legal-instrument model I found for the disputed-figures feature.** A species' official status is not one value but a **series of 20+ dated, decree-backed determinations**, each with a public evidence-submission and consultation record, plus a published *"historia de la clasificación de especies en Chile."* Contradictory statuses are preserved *by design* with the legal instrument attached to each. Whether individual species pages surface the decree and the cross-process change history was **not confirmed** — it may only exist in the downloadable spreadsheet |
| [Red de Observadores de Aves y Vida Silvestre de Chile (ROC)](https://www.redobservadores.cl/) | National | NGO / birder network | Bird records, atlas projects (**unverified**) | Chilean birds & wildlife | **DNS resolves only**; content unverified 2026-08-25. Note **`atlasaves.cl` does not resolve** | The Chilean breeding-bird-atlas lead; the non-resolving atlas domain suggests the atlas lives elsewhere or has moved |
| [Aves de Chile](https://www.aveschile.cl/) · [ChileFlora](https://www.chileflora.com/) · [Flora Chilena](https://florachilena.cl/) | National, single-kingdom | Small/solo (**unverified**) | Species accounts, images (**unverified**) | Chilean birds; Chilean flora | **DNS resolves only** for all three; content unverified 2026-08-25 | Three independent small-project leads in the small/solo category; ChileFlora is a long-standing commercial-ish flora site. All need manual assessment |

#### Paraguay

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [FaunaParaguay](https://www.faunaparaguay.com/) | National | **Solo/very small**, long associated with a single researcher (**identity not verified in this session**) | Reported multi-domain: species accounts, national checklists, photo galleries, **historical literature and bibliography archive**, expedition history | Paraguayan fauna | **Degrading.** DNS resolves and TCP 443 is open, but WebFetch failed on 2026-08-25 with **`CERTIFICATE_VERIFY_FAILED: certificate has expired`** from the upstream host. An expired TLS certificate means browsers now warn or block visitors — a strong indicator of lapsed maintenance. Content, counts and currency **unverified** | The most **at-risk solo project** found. Its reported combination of modern species accounts with a **historical literature archive** is close to what the user wants, making its decay worth studying — and worth archiving before it disappears. My own `openssl` probe returned the Anthropic egress gateway's re-signed certificate, so the expiry finding comes from the fetch proxy's upstream validation, not a direct read |
| [Guyra Paraguay](https://www.guyra.org.py/) | National | Conservation NGO | Bird and Chaco monitoring (**unverified**) | Paraguayan birds, Chaco | **DNS resolves only**; content unverified 2026-08-25 | Chaco deforestation-monitoring lead |

#### Uruguay

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Biodiversidata](https://biodiversidata.org/) | National | **Small academic consortium; named: Florencia Grattarola (developer/lead), Renata Polastri (brand design)** | Consolidated occurrence records plus a linked **NaturalistaUY** citizen-science stream; explicitly framed around filling **"serious geographic information gaps"** | Uruguayan biodiversity | **Live**, with active GitHub, Twitter and Flickr presence. Record counts, dataset counts, taxonomic coverage and publication years **not on the landing page — unverified**. Fetched 2026-08-25 | **The best small-team model for a data-poor region**, and methodologically the most relevant to a reserve atlas: it was built by explicitly *auditing and mapping the gaps* first, then assembling the country's first consolidated dataset from scattered sources. **Open by construction (public GitHub)** — a reproducible, low-cost pattern the user could copy directly. Note `uruguaybiodiversidad.uy` does not resolve |
| [Aves Uruguay](https://www.avesuruguay.org.uy/) · [Vida Silvestre Uruguay](https://www.vidasilvestre.org.uy/) | National | NGOs | Bird records; wildlife conservation (**unverified**) | Uruguayan birds & wildlife | **DNS resolves only** for both; content unverified 2026-08-25 | Aves Uruguay is the likely host of the Uruguayan nesting-bird atlas the user asked about; **unverified** |

---

#### Dead / dormant projects found

**1. INBio, the Atta database, and CRBio (Costa Rica) — completely dead, including the successor portal.**

This is the most important negative finding in the region, and it is worse than generally reported. On **2026-08-25 all of the following domains failed DNS resolution entirely** (not a 404 — the names no longer resolve): `inbio.ac.cr`, `www.inbio.ac.cr`, `atta.inbio.ac.cr`, `crbio.cr`, `www.crbio.cr`, `especies.crbio.cr`.

The institutional history, verified from a 2024 peer-reviewed retrospective in *Revista de Ciencias Ambientales* ([SciELO Costa Rica](https://www.scielo.sa.cr/scielo.php?script=sci_arttext&pid=S2215-38962024000200007)):

- Wind-down ran **2014–2016**; in 2014 a new government administration withdrew state support and collaboration.
- Causes given: dependence on international soft funding, the 2008 financial crisis, Costa Rica's reclassification as "middle-income" (which cut aid eligibility), and failed attempts to build an endowment or secure state funding.
- **3.7 million specimens** were dispersed: the main collection **plus the complete Atta database records** to the **Museo Nacional de Costa Rica**; the **Mollusca** collection (200,000+ specimens, 125,000 identified to species, 1,746 species described) to **Universidad de Costa Rica**; **Nematoda** (18,000 specimens) to **Universidad Nacional**.
- **CONAGEBIO** took over GBIF representation after **April 2015**.
- Costa Rica had been **Latin America's largest GBIF contributor by 2015** (3+ million records via INBio and partners).

**The lesson for the user is sharper than "INBio closed."** The retrospective presents **CRBio** as the successor national portal, integrating over 7 million occurrence records — and **CRBio's domain is now dead too**. So the mitigation failed as well: the specimens and the *tabular* data survived by being transferred into institutions with a mandate, while **both the original portal and its replacement web presence evaporated**. Anything that existed only as a website — species pages, images, interpretive content, URLs cited in the literature — is gone from the live web. Design accordingly: the data layer and the presentation layer need separate survival plans, and the presentation layer is the one that dies.

**2. Portal da Biodiversidade (ICMBio, Brazil) — explicitly deactivated.**

`portaldabiodiversidade.icmbio.gov.br` now 302-redirects to an [ICMBio notice page](https://www.gov.br/icmbio/pt-br/assuntos/programas-e-projetos/portal-da-biodiversidade) which states: *"Infelizmente, devido a aspectos tecnológicos, o Portal foi desativado"* ("Unfortunately, due to technological aspects, the Portal was deactivated"). Verified 2026-08-25.

- Launched **2020** (page dated 11/12/2020), built by MMA + ICMBio with technical support from Germany's **GIZ** and developers from **Escola Politécnica da USP**.
- Aggregated occurrence data from ICMBio's databases and the Rio de Janeiro Botanical Garden.
- Users are now redirected to **SiBBr**.

Notable that the stated cause was **technological**, not financial — a donor-funded portal built by an external academic dev team became unmaintainable once that team moved on, in only about five years. The graceful part worth copying: a **permanent explanatory notice with a redirect to a living alternative**, rather than a dead link. Contrast with INBio, which left nothing.

**3. Libro Rojo de la Fauna Venezolana (Provita) — live but apparently abandoned.** [especiesamenazadas.org](https://especiesamenazadas.org/) serves normally, but the fourth edition dates to **2015** and the site's copyright notice reads **2020**, indicating roughly six years without evident update (checked 2026-08-25).

**4. FaunaParaguay — decaying solo project.** Fetch on 2026-08-25 failed with an **expired TLS certificate** from the upstream host, which in practice means browser warnings for visitors. Host is otherwise reachable.

**5. Biblioteca Digital de la Medicina Tradicional Mexicana (UNAM) — degrading.** **TLS hostname mismatch** on `www.` and **HTTP 503** on the apex domain, 2026-08-25. A significant traditional-knowledge corpus in an unhealthy hosting state.

**6. Catálogo de Plantas y Líquenes de Colombia — dormant.** Site itself states **"Last update 09/02/2023."** Additionally its own "Presentación de la obra" page returned **404**, i.e. internal link rot within a live site.

**7. Retired or renamed hosts (link-rot evidence).** Domains that **failed DNS** on 2026-08-25 despite being plausible or formerly-used addresses: `sibm.invemar.org.co` (INVEMAR marine biodiversity sub-portal), `bolivia.fan-bo.org`, `mapas.serfor.gob.pe`, `atlasaves.cl`, `uruguaybiodiversidad.uy`, `floradecuba.com`, `avesdecuba.com`, `snap.gub.uy`, `flora-mesoamericana.org`, `herbarionacional.ucr.ac.cr`, `jamaicaclearinghouse.org`, `floraofpuertorico.org`. Some were my constructed guesses and prove nothing; but `sibm.invemar.org.co`, `snap.gub.uy` and `herbarionacional.ucr.ac.cr` correspond to institutions that certainly exist, suggesting genuine retirement or renaming.

**8. Domain migration in an active project.** `maaproject.org` → **`maapprogram.org`** (302, verified 2026-08-25). Even thriving projects move; 249 numbered reports' worth of citations now depend on a redirect.

---

#### Leads I could not verify

Grouped by why verification failed, so follow-up can be targeted.

**Search budget exhausted before discovery (never reached).** These were on my target list but I could not search for their URLs after the session's 200-search cap was hit, and I refuse to guess addresses: Guatemala's national biodiversity system (CONAP/SNIBgt); Belize Biodiversity Information System; national systems for Honduras, Nicaragua, El Salvador; Cuba's national biodiversity information system; Institute of Jamaica collections; Dominican Republic flora portals; CARICOMP (Caribbean Coastal Marine Productivity — suspected long-dead); Flora of the Guianas; Faune-Guyane / INPN French Guiana coverage; Falklands Conservation; jaguar-specific portals (Panthera's Jaguar Corridor Initiative databases, `jaguarnetwork.org` — both resolve but were not fetched); vicuña, Andean bear and macaw single-species portals; Atlantic Forest restoration atlases (SOS Mata Atlântica / INPE *Atlas dos Remanescentes Florestais*); Cerrado and Chaco documentation platforms; Uruguay's SNAP; Peru's Museo de Historia Natural UNMSM; colonial botanical-expedition digitisation (Real Expedición Botánica del Nuevo Reino de Granada / Mutis, "edition humboldt digital"); Archivo General de Indias/PARES; INEGI's Mexican geographic-names register (`snig.inegi.org.mx` did not resolve); INALI's catalogue of indigenous languages (`catalogo.inali.gob.mx` did not resolve); Atlas Sociolingüístico de Pueblos Indígenas en América Latina; Museu do Índio language-documentation programme; OCMAL mining-conflict observatory; ECOLEX/InforMEA regional coverage.

**Host reachable but content machine-inaccessible** (JavaScript-only SPA, robots.txt disallow, or TLS failure) — existence confirmed, figures not: EJAtlas case counts and leadership; dLOC holdings and partner counts; SiB Colombia's true record/species/dataset/partner figures (my fetch returned corrupted placeholder numbers — **do not reuse them**); Catálogo de la Biodiversidad de Colombia; RUNAP; Povos Indígenas no Brasil counts; Catálogo Taxonômico da Fauna do Brasil (redirect loop); SALVE species counts and assessment-cycle structure; Flora e Funga do Brasil main site (robots-disallowed; data reached via IPT instead); Reflora's own interface (robots-disallowed; verified via CNPq); Flora Argentina and its search interface; SIB Parques Nacionales Argentina (title only); Atlas de los Pueblos Indígenas de México; Biblioteca Digital de la Medicina Tradicional Mexicana; Janzen/Hallwachs ACG databases; Flora Mesoamericana on Tropicos; BioWeb Ecuador species totals (statistics page rendered client-side); Galápagos checklist taxon total; Naturalista México (HTTP 403).

**DNS-only confirmation** — the host exists and nothing more is known: Mariposas Mexicanas; OTS; SINAC; Museo Nacional de Costa Rica; AGRRA; Bermuda environment portals; Catálogo de la Flora de Cuba; CNUC; SIAM/INVEMAR; SINIA Peru; SNIFFS/SERFOR geoportal; SERNANP; Herbario Nacional de Bolivia (LPB); ROC Chile; Aves de Chile; ChileFlora; Flora Chilena; Guyra Paraguay; Aves Uruguay; Vida Silvestre Uruguay; Aves Argentinas; ForestGEO.

**Specific factual gaps worth one targeted check each:** whether SiBBr's platform is a confirmed Atlas of Living Australia fork (strongly implied by `collectory.sibbr.gov.br` and its "Atlas SiBBr" section, but I found no explicit statement); whether Chile's per-species classification pages expose the governing decree and the cross-process change history, or whether that lives only in the downloadable spreadsheet; whether the *Atlas de las Aves Nidificantes de la Argentina* and any Uruguayan/Chilean nesting-bird atlas exist as queryable databases rather than printed books; whether the UWI Trinidad & Tobago online animal guide still exists (my constructed URL 404'd); the current status of INBio's digital content in any web archive (Internet Archive was **not reachable** from this environment — `web.archive.org` is outside the network allowlist for both curl and the fetch proxy, so I could not date any of the deaths above via snapshots, only confirm them as of 2026-08-25).

---

#### Sources

- [SNIB — CONABIO, Mexico](https://conabio.snib.mx/)
- [EncicloVida — CONABIO](https://enciclovida.mx/)
- [SNIB Geoportal data-extraction documentation (PDF, Dec 2024)](https://www.snib.mx/ejemplares/docs/CONABIO-SNIB-DocumentacionExtraccionDatosGeoportal-202412.pdf)
- [Geoportal CONABIO](http://www.conabio.gob.mx/informacion/gis/)
- [Naturalista México](https://www.naturalista.mx/)
- [Atlas de los Pueblos Indígenas de México (INPI)](https://atlas.inpi.gob.mx/)
- [Biblioteca Digital de la Medicina Tradicional Mexicana (UNAM)](http://www.medicinatradicionalmexicana.unam.mx/)
- [Mariposas Mexicanas](https://mariposasmexicanas.com/)
- [El INBio: su labor innovadora… — Revista de Ciencias Ambientales, 2024 (SciELO)](https://www.scielo.sa.cr/scielo.php?script=sci_arttext&pid=S2215-38962024000200007)
- [Same article, PDF](https://www.scielo.sa.cr/pdf/rca/v58n2/2215-3896-rca-58-02-19990.pdf)
- [Museo Nacional recibe colección biológica custodiada por INBio — Presidencia de Costa Rica (now 404)](https://www.presidencia.go.cr/comunicados/2015/02/museo-nacional-recibe-coleccion-biologica-custodiada-por-inbio/)
- [INBio traspasa colecciones biológicas al Museo Nacional — La Nación](https://www.nacion.com/ciencia/medio-ambiente/inbio-traspasa-colecciones-biologicas-al-museo-nacional/2PGJVBYUAVCSPENVBBDTUPMN3I/story/)
- [Janzen & Hallwachs ACG site](http://janzen.sas.upenn.edu/)
- [caterpillars.org](https://www.caterpillars.org/)
- [Organization for Tropical Studies](https://ots.ac.cr/)
- [SINAC Costa Rica](https://www.sinac.go.cr/)
- [Museo Nacional de Costa Rica](https://www.museocostarica.go.cr/)
- [STRI research databases index](https://stri.si.edu/research-computing/databases)
- [Panama Biota (Symbiota)](https://panamabiota.org/stri/)
- [ForestGEO](https://forestgeo.si.edu/)
- [Flora Mesoamericana on Tropicos](https://www.tropicos.org/project/fm)
- [Healthy Reefs for Healthy People](https://www.healthyreefs.org/)
- [Caribbean Herpetology](https://www.caribbeanherpetology.org/)
- [Digital Library of the Caribbean](https://www.dloc.com/)
- [AGRRA](https://www.agrra.org/)
- [Catálogo de la Flora de Cuba](https://catalogofloradecuba.wordpress.com/)
- [Bermuda Department of Environment & Natural Resources](https://environment.bm/)
- [SiBBr](https://sibbr.gov.br/)
- [SiBBr as GBIF publisher](https://www.gbif.org/publisher/f5fd374b-89cb-4ab6-b3eb-794c65f232c3)
- [SiBBr collectory — ICMBio SISBio dataset (ALA-style collectory URL)](https://collectory.sibbr.gov.br/collectory/public/show/dr327)
- [SiBBr collectory — Flora e Funga do Brasil](https://collectory.sibbr.gov.br/collectory/public/show/dr66)
- [speciesLink (CRIA)](https://specieslink.net/)
- [Flora e Funga do Brasil — JBRJ IPT resource (v393.397)](https://ipt.jbrj.gov.br/jbrj/resource?r=lista_especies_flora_brasil&v=393.397&request_locale=pt)
- [Flora e Funga do Brasil site](https://floradobrasil.jbrj.gov.br/)
- [Flora e Funga do Brasil — GBIF dataset](https://www.gbif.org/dataset/aacd816d-662c-49d2-ad1a-97e66e2a2908)
- [REFLORA programme — CNPq](https://www.gov.br/cnpq/pt-br/acesso-a-informacao/acoes-e-programas/programas/reflora)
- [Reflora Virtual Herbarium](https://reflora.jbrj.gov.br/reflora/herbarioVirtual/)
- [Reflora digitisation manual (JBRJ, 2018, PDF)](https://dspace.jbrj.gov.br/jspui/bitstream/doc/103/1/Manual_Digitalizacao_exsicatas_2018.pdf)
- [Reflora FAPERJ–CNPq–SiBBr report 2011–2017 (PDF)](https://dspace.jbrj.gov.br/jspui/bitstream/doc/104/1/Relat%C3%B3rio%20REFLORA%20FAPERJ-CNPq-SiBBr%202011-2017.pdf)
- [SALVE — ICMBio](https://salve.icmbio.gov.br/)
- [Portal da Biodiversidade — ICMBio deactivation notice](https://www.gov.br/icmbio/pt-br/assuntos/programas-e-projetos/portal-da-biodiversidade)
- [Catálogo Taxonômico da Fauna do Brasil](http://fauna.jbrj.gov.br/fauna/)
- [CNUC — Cadastro Nacional de Unidades de Conservação](https://cnuc.mma.gov.br/)
- [WikiAves](https://www.wikiaves.com.br/)
- [MapBiomas](https://mapbiomas.org/)
- [Terras Indígenas no Brasil (ISA)](https://www.terrasindigenas.org.br/)
- [Povos Indígenas no Brasil (ISA)](https://pib.socioambiental.org/en/Main_Page)
- [Mapa de Conflitos envolvendo Injustiça Ambiental e Saúde (Fiocruz ENSP)](https://mapadeconflitos.ensp.fiocruz.br/)
- [Instituto de Desenvolvimento Sustentável Mamirauá](https://www.mamiraua.org.br/)
- [SiB Colombia](https://biodiversidad.co/)
- [SiB Colombia — tipos de datos](https://biodiversidad.co/compartir/tipos-de-datos/)
- [SiB Colombia — consulta de datos](https://biodiversidad.co/consultar)
- [SiB Colombia — developer recruitment for the Catálogo](https://sibcolombia.net/frontend-backend/)
- [SiB Colombia — catálogo de colecciones en línea](https://sibcolombia.net/proyectos/colombiabio/coleccionesenlinea/)
- [SiB Colombia — listas de especies](https://sibcolombia.net/proyectos/colombiabio/listas/)
- [SiB Colombia annual reports (Humboldt repository)](https://repository.humboldt.org.co/collections/3903d94f-77e3-40c4-b701-06dd81b67dae)
- [Catálogo de la Biodiversidad de Colombia](https://catalogo.biodiversidad.co/)
- [Biodiversidad en Cifras — SiB Colombia](https://cifras.biodiversidad.co/)
- [Catálogo de Plantas y Líquenes de Colombia](https://catalogoplantasdecolombia.unal.edu.co/)
- [BioModelos — Instituto Humboldt](http://biomodelos.humboldt.org.co/)
- [RUNAP](https://runap.parquesnacionales.gov.co/)
- [SIAM — INVEMAR](https://siam.invemar.org.co/)
- [Colombia, segundo país del mundo con más datos públicos sobre biodiversidad — MinAmbiente](https://www.minambiente.gov.co/colombia-segundo-pais-del-mundo-con-mas-datos-publicos-sobre-biodiversidad/)
- [BioWeb Ecuador](https://bioweb.bio/)
- [ReptiliaWeb Ecuador](https://bioweb.bio/faunaweb/reptiliaweb/)
- [Galápagos Species Checklist — CDF dataZone](https://datazone.darwinfoundation.org/en/checklist)
- [SINIA Peru](https://sinia.minam.gob.pe/)
- [SERFOR geoportal](https://geo.serfor.gob.pe/)
- [SERNANP](https://www.sernanp.gob.pe/)
- [MAAP](https://www.maapprogram.org/)
- [RAISG — Amazonia Socioambiental](https://www.amazoniasocioambiental.org/en/)
- [Herbario Nacional de Bolivia (LPB)](https://herbariolpb.umsa.bo/)
- [Libro Rojo de la Fauna Venezolana — Provita](https://especiesamenazadas.org/)
- [Provita](https://provita.org.ve/)
- [SIMBIO — Chile MMA](https://simbio.mma.gob.cl/)
- [Clasificación de Especies según Estado de Conservación — Chile MMA](https://clasificacionespecies.mma.gob.cl/)
- [Red de Observadores de Aves y Vida Silvestre de Chile](https://www.redobservadores.cl/)
- [Aves de Chile](https://www.aveschile.cl/)
- [ChileFlora](https://www.chileflora.com/)
- [Flora Chilena](https://florachilena.cl/)
- [SIB — Parques Nacionales Argentina](https://www.sib.gob.ar/)
- [SNDB Argentina](https://www.sndb.mincyt.gob.ar/)
- [EcoRegistros](https://www.ecoregistros.org/site/index.php)
- [Flora Argentina](http://www.floraargentina.edu.ar/)
- [Flora Argentina buscador](https://buscador.floraargentina.edu.ar/)
- [Instituto de Botánica Darwinion](https://www.darwin.edu.ar/)
- [Aves Argentinas](https://www.avesargentinas.org.ar/)
- [FaunaParaguay](https://www.faunaparaguay.com/)
- [Guyra Paraguay](https://www.guyra.org.py/)
- [Biodiversidata (Uruguay)](https://biodiversidata.org/)
- [Aves Uruguay](https://www.avesuruguay.org.uy/)
- [Vida Silvestre Uruguay](https://www.vidasilvestre.org.uy/)
- [Foro para la Conservación del Mar Patagónico](https://marpatagonico.org/)
- [HGIS de las Indias](https://www.hgis-indias.net/)
- [Environmental Justice Atlas](https://ejatlas.org/)

## Oceania: "Document a place comprehensively, from many source types, on one site"

**Scope:** Australia (by state/territory), New Zealand, Papua New Guinea, Pacific islands, Antarctica/Subantarctic. All liveness checks by direct fetch on **2026-08-25** unless a date is stated otherwise. Atlas of Living Australia, NBN, GBIF, Minnesota Biodiversity Atlas and India Biodiversity Portal are excluded per brief; ALA appears only as a relationship.

---

### Australia

#### National / cross-jurisdictional

| Project (URL) | Scale | Team size & type | Scope (occurrence-only vs multi-domain — domains) | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Birdata / Atlas of Australian Birds](https://birdata.birdlife.org.au/about-birdata) | National, continental | NGO (BirdLife Australia) + >10,000 volunteers historically; funded by Tony & Lisette Lewis Foundation, WildlifeLink | Occurrence-heavy but **multi-domain**: structured surveys, survey-effort metadata, monitoring programs (Birds in Backyards, Birds on Farms, Aussie Bird Count), threatened-species targeted surveys | Birds | **Active.** Site states ">30 million records", ">200,000 surveys submitted annually", data shared weekly with partners (fetched 2026-08-25) | Longest-running Australian digital bird database. **Deep historical layer**: Atlas 1 (1977–81) and Atlas 2 (1998–2002) — ~370,000 paper-form surveys, 7.5M sightings, retro-digitised. Good precedent for reconciling paper-era vs app-era effort |
| [DigiVol](https://volunteer.ala.org.au/) | National, plus international expeditions | Australian Museum in partnership with ALA; thousands of volunteer transcribers | **Multi-domain source-type mining**: specimen/collection labels, historical documents (field journals, ledgers, birth registrations, soil data cards 1920s–1960s), camera-trap image tagging ("Wildlife Spotter") | Any collection or archive | **Active.** Live expedition "Historic Soil Data Cards: Batch 9" at 2,293 tasks / 84% transcribed (2026-08-25) | **The single best Oceania model for historical document mining as a crowd workflow.** Multi-transcriber validation per task; leaderboards; forum for adjudicating unreadable/disputed readings. Directly relevant to Sathyamangalam's historical-document component |
| [Bush Blitz](https://bushblitz.org.au/) | National, expedition-based (single-reserve "all taxa" surveys) | Government–corporate–NGO partnership: Australian Government, BHP, Earthwatch Australia, Parks Australia | Multi-domain per site: species inventory, new-species descriptions, expedition reports, education outputs | All taxa, protected areas & Indigenous-owned land | **Active.** Pilliga NSW expedition 22 Sep–3 Oct 2025 (1,301+ species); King Island Tas Oct 2023 (683+ species); Wudjari Country WA Mar–Apr 2023 | Closest Australian analogue to a **single-site all-taxa** documentation project, repeated across reserves. 161 moth species new to science program-wide. Each expedition produces a bounded, citable site inventory |
| [FrogID](https://www.frogid.net.au/) | National | Australian Museum lead + 5 partner state museums (Qld, SA, WA, TMAG, MAGNT) | Occurrence + **acoustic evidence archive**; expert validation layer | Frogs | **Active.** Live counters: 930,680 calls submitted, 1,507,148 verified frogs, 232 species; site notes submission surge and validation backlog "since October 2025" (2026-08-25) | Publicly displays *submitted* vs *verified* counts as **two separate numbers** — an off-the-shelf pattern for showing raw vs adjudicated figures side by side |
| [Fungimap](https://fungimap.org.au/) | National | Small NGO, membership-based | Occurrence + target-species scheme (100 target taxa), advocacy, publications | Fungi | **Active.** "Great Aussie Fungi Hunt 2026" completed with 4,899 observers; news items July and August 2026 | Small-org model; 100-target-taxon design keeps a volunteer scheme tractable |
| [Australian Faunal Directory (ABRS)](https://biodiversity.org.au/afd/home) | National | Australian Biological Resources Study + external taxonomic specialists per group | **Multi-domain nomenclatural**: current names, full synonymy, nomenclatural history, type-specimen data, bibliography, distribution | All Australian fauna | **Slowing / partially dormant.** 127,933 species/subspecies in 4,468 families; "latest entries updated 11 April 2024" (fetched 2026-08-25) — no newer update shown | Synonymy + type data + literature per name is the classic **conflicting-taxonomy** substrate. Family-by-family update dates are exposed, so staleness is visible rather than hidden |
| [Australian National Species List / APNI / APC](https://biodiversity.org.au/nsl/) | National | Government (DCCEEW/ABRS + Council of Heads of Australasian Herbaria) | Nomenclature + **published taxon concepts**: APNI records every published *usage* of a name in a reference; APC records the single nationally accepted taxonomy | Vascular plants, algae, bryophytes, fungi, lichens | **Live** (fetched 2026-08-25); counts/version not shown on landing page — *unverified* | **Architecturally the most relevant model in this report for disputed data.** APNI/APC deliberately separates "what has been published" (many, conflicting) from "what we accept" (one), rather than collapsing them. Copy this split |
| [Flora of Australia online](https://profiles.ala.org.au/opus/foa) | National | ABRS/DCCEEW, delivered on the ALA profiles platform | Taxon profiles: descriptions, keys, images, distribution, linked occurrence records, document library | Australian flora | **Live** (fetched 2026-08-25). ISSN 2207-7820, CC-BY | Example of **reusing the ALA software stack** rather than building a portal — relevant if you want a hosted profile layer over your own occurrence data |
| [TERN](https://www.tern.org.au/) | Continental to plot scale | NCRIS-funded, hosted at University of Queensland | **Strongly multi-domain**: vegetation plots (AusPlots), flux towers, soil surveys & soil data, remote-sensing gridded products, phenocams, ecoacoustics (launched 2022), groundwater | Ecosystems | **Active** (fetched 2026-08-25). >100 observing sites | Standardised field-method protocols published alongside data — the "method provenance" habit worth copying for survey figures that later conflict |
| [AODN Portal](https://portal.aodn.org.au/) | National marine + Southern Ocean | IMOS (unincorporated joint venture led by University of Tasmania), NCRIS | Multi-domain marine: physical oceanography, biological, satellite, moorings, gliders | Marine & climate | **Active.** Portal footer shows **version 4.42.110, last updated 27 July 2026** | Explicit version + date stamping in the UI — cheap, high-trust provenance signal |
| [Seamap Australia](https://seamapaustralia.org/) | National marine (continental shelf) | IMAS, University of Tasmania; ~12+ partner institutions (CSIRO, Geoscience Australia, GBRMPA, NESP, Parks Australia, 5 universities) | Habitat classification + contextual layers; national benthic habitat classification scheme | Seafloor habitats | **Live** (fetched 2026-08-25). Launch year and layer count *unverified* | Its central contribution is a **shared classification vocabulary** that lets incompatible source surveys be compared — the marine equivalent of your gazetteer-reconciliation problem |
| [eAtlas](https://eatlas.org.au/content/about-e-atlas) | Regional (GBR + catchments, Wet Tropics, Torres Strait) | AIMS-hosted; funded through successive federal programs (MTSRF 2008–10, NERP TE, NESP TWQ 2008–21, NESP Marine & Coastal Hub 2022–present) | **Multi-domain**: interactive maps, imagery, research articles, scientific reports, reference datasets, GeoNetwork metadata catalogue | Tropical marine + terrestrial NE Australia | **Active** (fetched 2026-08-25). Began 2008 as "Reef Atlas", renamed eAtlas ~2010 | The closest structural match in Australia to what you're building: **one region, many source types (data + maps + reports + literature) on one site**, sustained across four funding regimes. Its own sister site nwatlas.org.au is now dead (below) |
| [EPBC Protected Matters Search Tool](https://www.environment.gov.au/epbc/protected-matters-search-tool) | National, point/area query | Australian Government (DCCEEW) | **Legal-instrument reporting**: listed threatened species & ecological communities, migratory species, Ramsar wetlands, World/National Heritage places, Commonwealth marine areas | Matters of National Environmental Significance | **Live** (fetched 2026-08-25); scheduled maintenance window published (Sun 23:55 – Mon 06:00 AEST) | The reference design for "**what legal instruments apply to this polygon**". Report carries an explicit "indicative only" caveat — the honest way to publish modelled distributions alongside hard records |
| [Composite Gazetteer of Australia / Place Names](https://placenames.fsdf.org.au/) | National + external territories | ICSM / Geoscience Australia, aggregating state & territory naming authorities | Gazetteer aggregation, downloadable | Place names | **Live** (fetched 2026-08-25); name count and contributing-agency list not on landing page — *unverified* | A **composite** gazetteer: same feature can carry names from multiple jurisdictional authorities. Structurally the right pattern for a multi-source toponym layer |
| [AUSTLANG (AIATSIS)](https://aiatsis.gov.au/austlang/about) | National | AIATSIS (Commonwealth institute) | Language varieties with alternative names/spellings, bibliographic sources, geographic extent, speaker information | Aboriginal & Torres Strait Islander languages | **Site responds but is JavaScript-rendered; content could not be read.** Entry count *unverified* | AUSTLANG's core job is holding **many competing spellings and names for one language variety**, each attributed to a source — near-exact analogue of a contested-toponym model |
| [Nyingarn](https://nyingarn.net/) | National | University of Melbourne / UWA consortium; ARC Linkage Infrastructure grant **LE200100006** | **Historical document mining**: locating early manuscript sources, OCR/HTR to searchable text, community-permission workflow, growing archive | Australian Indigenous language manuscripts | **Live** (fetched 2026-08-25). WordPress metadata 2023–2024; whether it is still ingesting in 2026 is *unverified* | Explicit **permissions gate before access** — a governance layer between "digitised" and "published" that a culturally sensitive atlas needs |
| [Gambay](https://gambay.com.au/) | National map | First Languages Australia (small NGO) + ABC content partnerships | Interactive language map, **dedicated place-names section**, videos (Word Up, Little Yarns), teacher notes | First Languages | **Live** (fetched 2026-08-25). ~200+ language groups listed alphabetically | Community-authored map where language groups control their own entry — an alternative to a single central editorial voice |
| [Keeping Culture KMS](https://keepingculture.com/) | Platform serving many communities | Small commercial/social-enterprise team | Configurable cultural-knowledge archive: media, protocols-based access control | Indigenous community archives | **Live.** Copyright 2026 on site (fetched 2026-08-25) | Commercial successor/sibling to Ara Irititja's software lineage; annual-licence cloud model. Relevant if you need **protocol-driven access restrictions** rather than open-by-default |
| [Ara Irititja](https://irititja.com/about-ara-irititja/who-are-we/) | Regional (APY Lands, far NW South Australia) | Founded 1994 by Anangu elders **Peter Nyaningu** and **Colin Tjapiya** with anthropologist **Ushma Scales** and archivist **John Dallwitz**; now under APY governance (from 2020), offices Adelaide + Alice Springs, South Australian Museum support | **Multi-domain social-history archive**: photographs, audio, film, documents, artefacts, oral stories | Anangu history & culture | **Active.** Governance transfer to APY Executive Board in 2020 documented on site (fetched 2026-08-25) | 30+ years continuous operation — the longest-lived Indigenous place-documentation project in Australia. Archive item count not published (*unverified*) |
| [NatureMapr](https://naturemapr.org/) | National network of regional nodes | Independent platform developed by **at3am**; ACT Government and Capital Ecology as official partners; volunteer moderator corps | Occurrence + expert moderation/verification, curated place-based maps (national parks, reserves, rural land), community forum, events, newsletters | All taxa | **Active.** naturemapr.org shows **844,303 sightings / 23,771 species / 16,082 members**; top moderator MichaelMulvaney with 60,600 verifications (2026-08-25) | **Documented figure conflict:** the same platform's Budawang Coast node reports **2,091,410 sightings / 18,688 species / 9,619 contributors** and 5,486 locations, while naturemapr.org and canberra.naturemapr.org both report 844,303/23,771/16,082. Different scoping of the same counters — a live example of the problem your feature addresses. A "flatter national structure" announcement URL (`/announcements/586`) now 404s |
| [Reef Life Survey](https://reeflifesurvey.com/) | Global, Australian-founded | Small science team; supported by The Ian Potter Foundation; RLS Australia + RLS International arms | Standardised diver transects (fish + invertebrates + habitat imagery), open data portal, publications | Shallow reef biodiversity | **Live** (fetched 2026-08-25). Survey/site/species totals not on homepage — *unverified* | Method-standardised volunteer science; global comparability from a single protocol |
| [DIISE — Database of Island Invasive Species Eradications](http://diise.islandconservation.org/) | Global, heavy Oceania/NZ content | Six-partner consortium: Island Conservation, TNC, UC Santa Cruz, **University of Auckland**, IUCN Invasive Species Specialist Group, **Manaaki Whenua Landcare Research** | Eradication-event records: island, target species, method, outcome; used as a global biodiversity indicator | Island invasive-vertebrate eradications | **Live**; page-modified timestamp **25 February 2025** (fetched 2026-08-25). Event/island counts not on landing page — *unverified* | Directly answers the brief's "offshore island restoration project database". Records *contested outcomes* (success/failure/unknown) per event |

#### New South Wales

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [BioNet Atlas](https://www.environment.nsw.gov.au/topics/animals-and-plants/biodiversity/nsw-bionet/about-bionet-atlas) | State | NSW DCCEEW, dedicated data team | **Explicitly multi-domain**: species sightings; systematic fauna survey; systematic flora survey; threatened biodiversity profiles; **vegetation classification and maps**; species names authority | All taxa NSW | **Active.** Page "Updated 28 May 2026"; site states data "constantly updated" and warns against using previously downloaded extracts | Aggregates from systematic surveys, consultants, researchers, **Australian Museum and Forests NSW**, *historical reports*, and public contributions. Publishes an unusually candid caveat that coverage is "extensive, but nevertheless patchy" and will not show full species distributions — the right register for an atlas that surfaces uncertainty. Feeds ALA |
| [PlantNET / NSW Flora Online](https://plantnet.rbgsyd.nsw.gov.au/) | State | Botanic Gardens of Sydney (National Herbarium of NSW) | Taxon profiles, traditional and interactive identification keys, spatial search, herbarium-backed distributions | NSW flora | **Live** (fetched 2026-08-25). Taxon count and last-update date not exposed on landing page — *unverified* | Has an explicit `limitofdata.html` page — publishing your own coverage limits as a first-class page |
| [SEED NSW](https://www.seed.nsw.gov.au/) | State | NSW Government, multi-agency (NSW Resources, EPA NSW, Water NSW, SES NSW) | Open-data catalogue + interactive mapping + topic hubs: air quality, water, natural hazards, **koala data**, future climate, natural capital accounting, imagery | Environment | **Active.** Featured content dated **2–3 July 2026** | Topic-hub model: a per-theme curated front door over a generic catalogue |
| [AHIMS — Aboriginal Heritage Information Management System](https://www.environment.nsw.gov.au/topics/heritage/search-heritage-databases/aboriginal-heritage-information-management-system) | State | Heritage NSW; register originally **established by the Australian Museum in the 1970s** | Sites and objects; declared Aboriginal Places; **archaeological reports**; **scanned original site cards back to the 1970s** | Aboriginal cultural heritage | **Active.** ">120,000 records"; system upgrade in **late 2023** added automated site-card delivery and digital payments | Explicitly notes that site-card formats and capture methods changed over 50 years, so **detail varies across records** — a documented, honest heterogeneity statement. Registration-gated; fee waivers for Aboriginal individuals and organisations; basic search free, extensive search A$60 |
| [NSW Geographical Names Board](https://www.nsw.gov.au/departments-and-agencies/geographical-names-board) | State | Statutory board under *Geographical Names Act 1966* | Determines **form, spelling, meaning and origin** of names; register; live public consultation on proposals | Place names | **Active.** Proposals under consultation as at November 2024 (e.g. Yanggaa Kelp Forest, Wululu Wetlands); proposals system at proposals.gnb.nsw.gov.au | Runs an **Aboriginal place naming and dual-naming program** tied to language revitalisation. The "meaning and origin" field plus a public proposal/objection process is a working model for contested toponyms |
| [Atlas of Life in the Coastal Wilderness](https://atlasoflife.org.au/) | Regional (Sapphire Coast / far SE NSW) | Community-run, volunteer-led | Occurrence + thematic sub-projects (bioluminescence, beach weeds, "Under the Wharf" at Tathra), photo competitions | All taxa | **Active.** Site records a 10-year anniversary; 2024 photo competition (fetched 2026-08-25) | **Small/community project** that solved sustainability by pushing records into iNaturalist rather than maintaining its own database — a realistic architecture for a modestly funded regional atlas |
| [NatureMapr Budawang Coast](https://budawangcoast.naturemapr.org/) | Regional (Budawang Coast, SE NSW) | Volunteer node of NatureMapr | Occurrence + moderation + locations | All taxa | **Active.** 2,091,410 sightings / 18,688 species / 5,486 locations / 9,619 contributors (2026-08-25) | Node-level counters conflict with network-level counters (see NatureMapr row) |

#### Victoria

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Victorian Biodiversity Atlas](https://www.environment.vic.gov.au/biodiversity/victorian-biodiversity-atlas/about-the-vba) | State | DEECA; support desk vba.help@deeca.vic.gov.au | Flora + fauna sightings, **survey effort**, expert-panel verification; search deliberately delegated to sibling tools NatureKit and CoastKit | All taxa Victoria | **Active.** About page "last updated 6 May 2026" | **The clearest conflicting-figures case I found.** Three published record counts, none reconciled: "more than **6 million**" and "over **seven million**" *in the same 2020 Connecting Country article*, and "more than **6.5 million**" on loveourlakes.net.au. Also unusual as a **contribution-first** atlas: the VBA is "primarily a tool for sharing your observations and survey effort", with discovery handled elsewhere |
| [VicFlora](https://rbgvictoria.github.io/vicflora-static-pages/about/) | State | National Herbarium of Victoria, Royal Botanic Gardens Victoria | Descriptions (from print *Flora of Victoria* 1994/1996/1999), **Census of Vascular Plants of Victoria** (8 print editions 1984–2007), keys, checklists, glossary, distribution maps from herbarium specimens **plus VBA records** | Victorian flora | **Active**, described as a continuously updated "living" product; software upgrade/relaunch documented July 2022 | Model for **absorbing a print flora and a print census into one live resource** while keeping the print lineage visible. Distribution maps draw on two different evidence classes (specimens vs field observations) — exactly the kind of mixed-provenance map that needs disclosure |
| [Victorian Places](https://www.victorianplaces.com.au/) | State, place-by-place | Monash University + University of Queensland collaboration | **Historical place documentation**: every town, city, suburb, village and settlement over 200 people; historical descriptions plus census series | Places (settlements) | **Likely dormant.** ">1,600 places"; copyright notice **2015**; forward-looking note about "a 2016 census update" that appears not to have been fully carried through; no visible recent update (fetched 2026-08-25) | Structurally almost identical to a gazetteer-plus-history layer: one page per place, longitudinal census figures, narrative history. Its stall is instructive — academic place-gazetteers decay when the grant ends |
| [Connecting Country](https://connectingcountry.org.au/) | Regional (Mount Alexander Region) | Small NGO: paid staff + volunteers | Monitoring programs (old trees, nest boxes, woodland birds, reptiles & frogs), data pushed into the VBA, public explainer content | Landscape restoration | **Active** organisation; the cited VBA explainer article is dated **16 January 2020**, author "Ivan" | Deliberately does **not** run its own database — a small group acting as a data *contributor* and *interpreter* for the state atlas |
| [Love our Lakes — Biodiversity Atlas page](https://loveourlakes.net.au/resources/biodiversity-atlas/) | Regional (lakes/wetlands) | Small community project | Signposts VBA; documents weeds and pest animals alongside indigenous species | Lakes biodiversity | **Live** (fetched 2026-08-25) | Source of one of the three conflicting VBA record counts ("more than 6.5 million") |

#### Queensland

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [WildNet](https://www.qld.gov.au/environment/plants-animals/species-information/wildnet) | State | Queensland DETSI, Science/Coastal/Biodiversity & Information division, with a dedicated curation team | **Multi-domain**: occurrence records with metadata; conservation status and nomenclature; habitat and distribution; **multimedia (documents, images, maps, sounds)**; pre-built species lists for reserves, LGAs and catchments; project/dataset source metadata | Native fauna (amphibians, birds, mammals, reptiles, some fish and invertebrates) + regulated plants | **Active.** Page "last modified 25 July 2025", "next review 27 May 2026". Public search now at wildnet.science-data.qld.gov.au (redirect from apps.des.qld.gov.au confirmed 2026-08-25) | Data spans **from the 1700s**, and the site itself advises filtering to "records since 1980" — an explicit, published temporal-reliability boundary. Curation team pre-screens for data quality, methodology and taxonomic relevance before ingest |
| [WetlandInfo](https://wetlandinfo.detsi.qld.gov.au/wetlands/) | State, tiled to 100 km map sheets and down to individual localities | Queensland DETSI | **Very multi-domain**: facts-and-maps by region/area type, ecology, **conceptual ecosystem models**, assessment/monitoring/inventory methods, management guidance, education; browsable by drainage division, NRM region and local government area | Wetlands & catchments | **Active.** Established **23 October 2017**; **last updated 16 January 2026** | The best Australian example of "**one page per place, assembled from many domains**". Its conceptual-model pages are an unusual and copyable device: a diagrammatic synthesis sitting between raw data and prose |

#### South Australia

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [BDBSA / NatureMaps](https://www.environment.sa.gov.au/topics/science/information-and-data/biological-databases-of-south-australia) · [NatureMaps viewer](https://data.environment.sa.gov.au/NatureMaps/pages/default.aspx) | State | SA Department for Environment and Water | Multi-domain: flora records, fauna records, **systematic vegetation surveys**, opportunistic observations, taxonomic systems; NatureMaps is the map front end | All taxa SA | **Live** (fetched 2026-08-25). Record counts not published on the landing page — *unverified*; a 2023 overview factsheet exists at data.environment.sa.gov.au | Separates the **database of record** (BDBSA) from the **public viewer** (NatureMaps) — the same split as Victoria's VBA/NatureKit. Feeds ALA and OBIS |
| [Electronic Flora of South Australia (FloraSA)](https://www.flora.sa.gov.au/) | State | State Herbarium of South Australia (Botanic Gardens and State Herbarium) | Taxon profiles (nomenclature, description, phenology, distribution, habitat, cultivation), **KeyBase-hosted interactive keys**, distribution maps by herbarium region, images, downloadable census PDFs, glossary, staff publications | Vascular plants, bryophytes, algae, fungi, lichens | **Live**; most recent image modification date seen **11 August 2025** (fetched 2026-08-25) | Unusually broad taxonomic reach for a state flora (includes marine algae). Keys are outsourced to shared national infrastructure (KeyBase) rather than rebuilt |

#### Western Australia

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [NatureMap](https://naturemap.dbca.wa.gov.au/) | State | DBCA (Department of Biodiversity, Conservation and Attractions), with WA Museum and WA Herbarium as key data sources | Occurrence aggregation across institutional datasets, area-based species lists | All taxa WA | **Live** (help page fetched 2026-08-25). A published account exists: *"NatureMap: Mapping Western Australia's Biodiversity"* (DBCA library item 080525) | State-level federated aggregator predating/paralleling ALA; publishes its own aggregation description |
| [FloraBase](https://florabase.dbca.wa.gov.au/) | State | Western Australian Herbarium, DBCA | Taxon profiles, images of flora **and fungi, slime moulds and seaweeds**, monthly featured plant, **hosts the journal *Nuytsia*** | WA flora and allied organisms | **Active** (fetched 2026-08-25). Taxon/specimen counts not on the landing page — *unverified* | Bundles a **peer-reviewed taxonomic journal into the atlas itself**, so new names are published and indexed in the same place they're browsed. Strong precedent for combining literature with occurrence data on one site |
| [NARvis](https://narvis.com.au/the-region/biodiversity/) | Regional — Northern Agricultural Region, WA (5 IBRA bioregions: Avon Wheatbelt, Jarrah Forest North, Swan Coastal Plain, Geraldton Sandplains, Yalgoo) | Northern Agricultural Catchments Council (NACC), small NRM body; Australian Government funded | **Eight-theme multi-domain narrative atlas**: biodiversity conservation, **Aboriginal custodianship**, climate change, coastal & marine, community capacity, invasive species, sustainable agriculture, water | Regional NRM | **Likely dormant.** Cited material spans **2008–2020**; no explicit update date (fetched 2026-08-25) | An **IBRA-based regional atlas** that explicitly synthesises from NatureMap, FloraBase, DBCA, DAWE, CSIRO and literature rather than holding primary records — a "synthesis atlas". Includes Aboriginal custodianship as a first-class theme alongside biophysical ones |
| [Ningaloo Atlas](https://ningaloo-atlas.org.au/) | Single region (greater Ningaloo) | Partnership: government, NGO, researchers, industry (AIMS, BHP Billiton named) | Multi-domain: articles, datasets, blog posts; framed around "**biodiversity, heritage, value, and way of life**" | Ningaloo coast & reef | **Live but activity uncertain.** Site describes itself as "a work in progress"; article listings present, but no news/blog index page resolved (`/blog` and `/latest-news` both 404 on 2026-08-25), so recent-activity dates are ***unverified*** | Explicitly a **single-place, multi-source-type atlas** including heritage and social values, not just species — the nearest Australian conceptual sibling to Sathyamangalam |

#### Tasmania

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Natural Values Atlas](https://nre.tas.gov.au/conservation/development-planning-conservation-assessment/planning-tools/natural-values-atlas) · [app](https://www.naturalvaluesatlas.tas.gov.au/) | State | NRE Tasmania, dedicated NVA team performing ongoing QA; support line (03) 6165 8839 | **The most multi-domain state atlas in Australia**: ~**4.7 million observations** of **>50,000 species** back to the 1800s; **TASVEG** statewide vegetation mapping; threatened native vegetation communities; vegetation condition assessments; **Tasmanian Geodiversity Database** (geology, geomorphology, soil conservation sites with management guidance); **>6,000 soil sites**; and "Locations & Activities" — monitoring plots, traps, nest boxes | Species + vegetation + geodiversity + soils + conservation infrastructure | **Active.** Page published **15 December 2025, 1:18 PM**; new data loaded **daily**, existing records regularly reviewed | The clearest existing proof that species + vegetation + **geodiversity** + soils + field-infrastructure can coexist in one atlas. Registration required. Publishes a document-attachment system for method instructions (e.g. TASVEG search and vegetation-condition-assessment guides) |
| [Threatened Species Link](https://www.threatenedspecieslink.tas.gov.au/) | State | NRE Tasmania | Per-species: listing statements, notesheets, recovery plans, **survey guidelines**, identification, habitat; area search tool; activity-specific advice | Tasmanian threatened species | **Partially dormant — self-declared.** Site carries the explicit disclaimer: *"The fauna data on the Threatened Species Link is currently not being maintained. As such, some of the information may be out of date."* Area-search tool launched September 2014; most recent dated update seen **1 December 2022** (fetched 2026-08-25) | **A model of honest decay.** Rather than quietly rotting, the site states which half of it is unmaintained. Adopt this: per-domain freshness declarations |

#### Northern Territory

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Fauna Atlas NT](https://nt.gov.au/environment/environment-data-maps/fauna-atlas) · [Flora Atlas NT](https://nt.gov.au/environment/environment-data-maps/flora-atlas) · [NTLIS metadata](http://www.ntlis.nt.gov.au/metadata/export_data?type=html&metadata_id=2DBCB771208B06B6E040CD9B0F274EFE) | Territory | NT Government (DEPWS); small team | Occurrence-focused, split into two sister systems (fauna, flora); herbarium-backed flora records | All taxa NT | **Exists and publishes to GBIF/OBIS and data.nt.gov.au; landing pages not fetched in this pass — record counts and update dates *unverified*.** Registered on the AGLDWG catalogue as "NT Fauna Atlas" | Unusual among the states in keeping fauna and flora as **two separate atlases** rather than one. Only Australian jurisdiction where the split persists |

#### Australian Capital Territory

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Canberra Nature Map](https://canberra.naturemapr.org/) | Territory + Southern Tablelands | Volunteer community + moderators; official partners ACT Government and Capital Ecology; platform by at3am | Occurrence + expert verification + **curated place maps by land type** (national parks, reserves, rural land) + forum + events + newsletters + mobile apps (Android, iOS) | All taxa | **Active.** 844,303 sightings / 23,771 species / 16,082 members; named top contributors (AlisonMilton 19K sightings, trevorpreston 13.8K) and top moderator (MichaelMulvaney, 60,600 verifications) as at 2026-08-25 | Originated the NatureMapr platform, which then spun out nationally. **Publicly named moderators with verification counts** — reputation as a visible provenance mechanism. Recommended by NSW Landcare Gateway (Kosciuszko to Coast group) |

---

### New Zealand

#### National

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [New Zealand Bird Atlas 2019–2024](https://ebird.org/atlasnz/home) · [scheme page](https://www.birdsnz.org.nz/schemes/nz-bird-atlas-scheme/) | National, 10×10 km grid | Birds New Zealand (OSNZ) volunteer society; coordination team led by **Dan Burgin**; delivered on Cornell's eBird platform | Occurrence + **effort-standardised grid coverage** + breeding codes | Birds | **Complete, closed.** Ran **1 June 2019 – 31 May 2024**; final data deadline end of July 2024; no news items after 2024 on the portal (fetched 2026-08-25) | Hard numbers from the [31 May 2024 media release](https://www.birdsnz.org.nz/wp-content/uploads/2024/05/MEDIA-RELEASE-NZ-Bird-Atlas-Project-Complete.pdf): **441,000 checklists**, **309 species**, **145,000 hours** of effort, **3,145 of 3,232 grid squares** with data = **97.3% coverage**. Most-recorded species by squares: pahirini/chaffinch 2,896; manu pango/blackbird 2,870; tauhou/silvereye 2,868. A **bounded, completed atlas with published coverage statistics** — the cleanest example in this report of an atlas that declares when it is finished |
| [iNaturalist NZ — Mātaki Taiao](https://inaturalist.nz/) | National | NZ Bio-Recording Network Trust community + iNaturalist platform (originally launched as NatureWatch NZ) | Occurrence + community identification + GBIF publication | All taxa | **Live** (fetched 2026-08-25). Totals page `/stats` is robots-blocked; observation/species/observer counts ***unverified*** | Bilingual naming (English + te reo Māori) baked into the brand. The renaming from NatureWatch NZ to iNaturalist NZ is itself a case study in a national project trading independence for platform sustainability |
| [National Vegetation Survey (NVS) Databank](https://nvs.landcareresearch.co.nz/) · [about](https://datastore.landcareresearch.co.nz/organization/about/national-vegetation-survey-nvs) | National, plot-based | Manaaki Whenua – Landcare Research; publicly funded | **Plot-based vegetation data**: plot records, species composition, structure, remeasurement time series; plus NVS Express tooling | Vegetation plots | **Live**; also published to GBIF as "NZ National Vegetation Survey occurrence data" (IPT resource at v1.82) and registered in re3data (r3d100011128) (fetched 2026-08-25) | Described by its host as world-leading for plot-based vegetation data. Documented in the peer-reviewed literature ([NZJE article 2124](https://newzealandecology.org/nzje/2124)) — an atlas that published its own architecture |
| [New Zealand Organisms Register (NZOR)](https://www.nzor.org.nz/) | National | Established by a **multi-agency steering group formed 2006** under the TFBIS programme; consolidation of previously scattered agency name lists | Taxonomic and nomenclatural names authority; provider attribution; data-quality statements | All organisms relevant to NZ | **Live but content appears stale.** Homepage text still reads prospectively ("will be electronically available through one or more portals") and cites a 2000 scoping document; ">100,000 organism names" quoted. No update date or news anywhere on the page (fetched 2026-08-25). Whether the underlying name index is still refreshed is ***unverified*** | **NZOR's whole purpose is reconciling conflicting names from multiple providers**, and it publishes dedicated [data-quality/use](https://www.nzor.org.nz/data-quality-and-use) and [what-data-is-provided](https://www.nzor.org.nz/what-data-is-provided) pages plus per-provider attribution. Conceptually the most directly reusable design in NZ for your disputed-figures feature — read it even though the site itself has aged |
| [Biota of New Zealand](https://biotanz.landcareresearch.co.nz/) | National | Manaaki Whenua – Landcare Research | Names + taxon information across fungi, land invertebrates and plants; consolidates the former separate Ngā Tipu Aotearoa, Ngā Harore o Aotearoa and Wētā/Ko te Aitanga Pepeke o Aotearoa databases | Fungi, invertebrates, plants | **Live** (fetched 2026-08-25); counts and update dates behind a JS interface — ***unverified***. Sibling **Wētā** inventory is described as covering **>38,000 names** of insects, mites, arthropods and nematodes | Bilingual naming across the whole database family; a **consolidation** of previously separate name databases into one portal |
| [Ngā Rauropi Whakaoranga — Māori plant use](https://maoriplantuse.landcareresearch.co.nz/) | National | Manaaki Whenua – Landcare Research | **Traditional-knowledge domain**: documented Māori traditional uses of native plants and other organisms, compiled from ethnobotanical and historical literature | Traditional use of biota | **Live** (fetched 2026-08-25). Record/use/source counts ***unverified*** | A government research institute running a **traditional-knowledge database as a peer of its taxonomic databases**. Directly analogous to an ethnobotany/local-use layer for Sathyamangalam |
| [Manaaki Whenua database suite](https://www.landcareresearch.co.nz/tools-and-resources/databases/) | National | Manaaki Whenua – Landcare Research (Crown Research Institute) | **20 named databases across three domains** — Plants/Animals/Fungi: Biota of New Zealand, Wētā, Ngā Tipu Aotearoa, Ngā Harore o Aotearoa, NVS Databank, Ngā Rauropi Whakaoranga, Systematics Collections Data. Land & Soil: LRIS, LRIS Portal, S-Map Online, Soils Portal, National Soil Data Repository, NZ Soil Classification, Land Resources Portal, Land Cover Explorer, **Antarctic Soils Explorer**, **Pacific Soils Portal** (Fiji, Kiribati, Samoa, Tonga, Tuvalu), **Whenua Māori Visualisation Tool**. Plus DataStore and the National Environmental Data Centre | Biota, land, soil, Māori land | **Live** (fetched 2026-08-25) | The single most efficient survey of NZ place-documentation infrastructure. Note the **Whenua Māori Visualisation Tool** (for Māori landowners to learn about their own land) and the **Pacific Soils Portal** — an NZ institution documenting other countries' places |
| [New Zealand Plant Conservation Network](https://www.nzpcn.org.nz/) | National | Non-profit membership network, founded **April 2003**, **>1,000 members worldwide** | Multi-domain: species database pages (vascular and non-vascular plants, lichens, fungi), **threat classifications**, national and regional plant lists, **hosts the NZ Botanical Society journal**, identification guides, ecosystem/habitat information, seedbank projects, training modules | NZ flora | **Very active.** Site announced 2026 Plant Conservation Awards (nominations to 31 August), July 2026 *Trilepidea* newsletter, and NZPCN 2026 Conference (fetched 2026-08-25) | 23 years of continuous volunteer-society operation, hosting a journal, a species database and a threat-classification layer on one site. The best NZ evidence that a **membership NGO** can sustain a multi-domain atlas without government hosting |
| [NZ Birds Online](https://nzbirdsonline.org.nz/) | National | Partnership: **Te Papa** (Museum of New Zealand) + **Birds New Zealand** | Per-species: text accounts, images, **sound recordings**, distribution, search by name/appearance/group/**location**/conservation status | NZ birds | **Live** (fetched 2026-08-25). Species count, launch year and update cadence not on the homepage; `/about` returns 404 — ***unverified*** | Museum + volunteer society co-production; "search by location" and "random species" as discovery devices |
| [New Zealand Freshwater Fish Database (NZFFD)](https://nzffdms.niwa.co.nz/) | National | NIWA (Crown Research Institute), with an administrative review gate on submissions | Occurrence + method + abundance + fish size + site description, linked to **River Environment Classification** environmental attributes | Freshwater fish | **Active.** ">50,000 freshwater fish sampling records"; application **version 4.3.9**, copyright 2026 (fetched 2026-08-25) | Contributor list is unusually broad and named: NIWA, DOC, regional councils, consultants, universities, fish and game councils, research institutes, **schools, iwi groups** and the public — with **administrative review before public release**. A clean two-tier submitted/approved model |
| [New Zealand Gazetteer of place names](https://www.linz.govt.nz/products-services/place-names/place-names-new-zealand) · [live gazetteer](https://gazetteer.linz.govt.nz/) | National + offshore islands + undersea features to the continental shelf + **Antarctica's Ross Sea region** | New Zealand Geographic Board Ngā Pou Taunaha o Aotearoa, within LINZ | **Official + unofficial names** with status, *New Zealand Gazette*/legislative references, feature type, coordinates, and optional **description, extent, "history, origin or meaning", and source data**; full CSV download via LINZ Data Service | Place names | **Active** (fetched 2026-08-25). Total entry count not on the page — ***unverified***. 824 Māori place names were made official in a single 2019 tranche | **The reference implementation for a contested gazetteer.** It records *unofficial* names that appear on official maps alongside official ones; carries original Māori place names where known; publishes a **separate CSV of Māori place names with macrons**; and states plainly that "not all Māori names have undergone spelling verification" — i.e. it flags its own uncertain records rather than presenting uniform confidence |
| [Māori GIS Mapping](https://maorigis.nz/docs/data/place-names) · [Matawhenua viewer](https://maps.maorigis.nz/) | National | **Solo project** — authored and run by **Duane Wilkins** (@pm4gis) | Guidance + tooling rather than data holdings: how to find, check, record and map **ingoa wāhi, official names, historical names and locally held names**; Matawhenua is a **browser-only** GIS with layers for Māori land blocks, marae, pā sites, parcels, awa, maunga, place names, DEM/DSM terrain | Māori place names & mapping practice | **Active — one of the most recently updated things in this report.** Documentation page "Last reviewed: **23 August 2026**" (fetched 2026-08-25) | **The standout small/solo project of this survey.** Its explicit four-way distinction between *official*, *historical* and *locally held* names, plus a documented "checking" method, is precisely the epistemics a disputed-toponym feature needs. Matawhenua runs entirely client-side with no cloud storage — nothing leaves the user's device unless exported, a genuine sovereignty-by-architecture choice |
| [ArchSite — NZ Archaeological Site Recording Scheme](https://www.archsite.org.nz/) | National | **New Zealand Archaeological Association** (volunteer society) in partnership with **Heritage New Zealand** and **DOC** | Site records with interactive map; request-based access to detail | Archaeological sites | **Live** (fetched 2026-08-25). Site count, per-site fields and scheme inception dates not on the landing page — ***unverified*** | Charges apply for some requests. A **learned-society-run national register** operating in formal partnership with two Crown agencies — an alternative governance model to a purely government register |
| [Te Papa Collections Online](https://collections.tepapa.govt.nz/) | National | Museum of New Zealand Te Papa Tongarewa; founding partners Wellington City Council and Ministry for Culture and Heritage | **Cross-domain museum collection**: Taonga Māori, Pacific Cultures, History, Photography, Art, Botany, Zoology | Objects, artworks, specimens | **Live.** ">1,000,000 artworks, objects and specimens"; ">400,000 images, with over 200,000 available for high resolution download" (fetched 2026-08-25). Update cadence not stated — ***unverified*** | One search across natural-history specimens **and** cultural taonga **and** archival photography — the integration your atlas needs, already in production at million-record scale |
| [Heritage New Zealand List / Rārangi Kōrero](https://www.heritage.org.nz/list) | National | Heritage New Zealand Pouhere Taonga (Crown entity) | Statutory heritage list with categories including historic places, historic areas, **wāhi tapu**, **wāhi tūpuna** | Heritage places | **Site responds but content is not readable by fetch (metadata only).** Entry counts and categories ***unverified*** as at 2026-08-25 | Notable for statutory recognition of **Māori-defined categories** (wāhi tapu, wāhi tūpuna) as list types, not annotations |
| [Department of Conservation biodiversity resources](https://www.doc.govt.nz/nature/biodiversity/biodiversity-new-zealand-resources/) | National | DOC (government department) | Policy framework (*Te Mana o te Taiao* — Aotearoa NZ Biodiversity Strategy 2020) + signposting | Biodiversity | **Live**, but thin: the only named data resource on the page is **iNaturalist NZ** (fetched 2026-08-25) | Instructive negative finding: **New Zealand has no single national occurrence atlas of its own.** DOC points the public to a third-party platform. This is the biggest structural difference from Australia's state-atlas landscape |

#### Te Waipounamu / South Island — iwi

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Kā Huru Manu — Ngāi Tahu Cultural Atlas](https://www.kahurumanu.co.nz/) | Te Waipounamu (South Island), iwi-wide | **Te Rūnanga o Ngāi Tahu** (iwi authority); design by NV Interactive; built on an embedded ArcGIS web app | **Multi-domain place documentation**: the Ngāi Tahu Atlas (interactive map of place names), Cultural Mapping Story (project history), **Kā Ara Tawhito** (traditional travel routes/trails), **Map Stories** (historical maps and individual narratives), Our People (contributors) | Māori place names and histories | **Active.** "**over 1,000 original Māori place names** of Te Waipounamu"; copyright 2026 (fetched 2026-08-25) | **The single most on-pattern project in the whole of Oceania for your brief.** It fuses a place-name gazetteer, oral histories, historical map scans, and traditional route networks into one atlas, governed by the community it documents — and it originated as an evidential by-product of **Te Kereme (the Ngāi Tahu Treaty claim)**, i.e. a legal instrument produced the documentation project. Study this one first |

---

### Papua New Guinea

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [PNGplants](http://www.pngplants.org/) | National (PNG) | **Solo — created by Barry Conn** (formerly National Herbarium of NSW) | **Multi-domain botanical**: internet-accessible **herbarium specimen database** with specimen images, the **PNGtrees** interactive tree identification guide, biographies of **plant collectors**, a **census of PNG vascular plant names**, and three published flora handbook volumes | PNG plants | **Live** (fetched 2026-08-25). No dates, specimen counts, or institutional host stated on the site — ***unverified*** | The clearest **solo-author national flora portal** found in this survey, and a warning: it is a one-person infrastructure for an entire country's botany. Its own PNGtrees domain has already gone (below) |
| ~~[PNGtrees](https://www.pngtrees.org/)~~ | National (PNG) | PNG Forest Research Institute / Australian collaborators (historically) | Interactive tree identification: keys, images, descriptions | PNG trees | **DEAD.** `pngtrees.org` fails DNS resolution — "Name or service not known" (2026-08-25) | Content apparently survives only as a sub-section of pngplants.org. Domain loss is the most common single point of failure in this landscape |
| ~~[Papuaweb](http://www.papuaweb.org/)~~ | Papua / West Papua (Indonesian New Guinea) | Academic consortium (historically ANU, UNCEN, UNIPA) | Multi-source place documentation: theses, bibliographies, maps, colonial-era documents, language materials | Papua region | **DEAD.** `papuaweb.org` fails DNS resolution — "No address associated with hostname" (2026-08-25) | Was one of the best examples anywhere of "**mine many document types about one region onto one site**". Its disappearance is the cautionary tale for this entire report |

---

### Pacific islands

#### New Caledonia

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Endemia.nc](https://endemia.nc/) | Territory-wide | Community/expert collaborative; site states quality "depends on the participation of all those willing to contribute their expertise" — organisational structure not stated on the page | **Multi-domain**: flora and fauna taxon pages, geographic localisation, links into **The Plant List** and the **IRD Herbarium (Nouméa) specimen database with collection maps**, external image search | Endemic and native flora and fauna of New Caledonia | **Active.** Site publishes a **visible per-record change log** with entries dated 25–26 May, including descriptor updates for *Eugenia rubiginosa*, *Tapeinosperma psaladense*, *Monolepta semiviolacea* and a *Sterna nereis* subspecies (fetched 2026-08-25) | Two things worth stealing: (1) a **public, per-taxon edit log** as the primary evidence of liveness; (2) documenting a hyper-endemic biota by **linking out to the herbarium of record** rather than duplicating specimen data. Endemia is also the recognised Red List authority workstream for New Caledonian flora — *unverified from the pages fetched* |

#### Regional (SPREP / Pacific-wide)

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Pacific Environment Portal](https://pacific-data.sprep.org/) + [Inform project](https://www.sprep.org/inform) and the 14 national portals (e.g. [Samoa](https://samoa-data.sprep.org/), [Vanuatu](https://vanuatu-data.sprep.org/)) | Regional + **14 national portals**: Cook Islands, FSM, Fiji, Kiribati, RMI, Nauru, Niue, Palau, PNG, Samoa, Solomon Islands, Tonga, Tuvalu, Vanuatu | SPREP (regional intergovernmental organisation) with member-country environment ministries; partners include BIOPAMA, PacWastePlus, UNESCO, UNDP | Multi-domain national environmental data: datasets, indicators, **State of Environment reporting** (soec.sprep.org), documentation site (docs.pacific-data.sprep.org), thematic databases including a **Pacific Seabird Colony Database** | National environments of 14 Pacific states | **Active.** Core project ran **2017–2021**; continues as **"Inform Plus"**. Samoa portal serving current content (Fourth National State of Environment Report); Vanuatu portal serving current content (Climate Watch App). Both fetched 2026-08-25 | The **most successful replicated-national-portal programme** in Oceania: one template, 14 countries, each nation owning its own instance. If Sathyamangalam is a template for other Indian reserves, this is the governance model to read. Dataset counts and tech stack per portal ***unverified*** |
| [Pacific Islands Protected Area Portal (PIPAP)](https://www.sprep.org/pipap) | Regional | SPREP + BIOPAMA + Protected Area Working Group (PAWG) | Protected-area registry with a "Pacific Dashboard" and decision-support tools | Pacific protected areas | **DORMANT.** Dashboard reads **"0 Protected Areas (0 Terrestrial, 0 Marine)"**; newest content is an "Action Plan 2014–2020" and a PAWG presentation from **Suva, July 2015** (fetched 2026-08-25) | A dashboard that renders zeroes is worse than no dashboard. Direct lesson for the disputed-figures feature: **a broken counter reads as an authoritative claim** |

#### Fiji

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [NatureFiji-MareqetiViti](https://www.naturefiji.org/) | National | "Fiji's first and only local biodiversity conservation NGO"; trustees, management council, life members, volunteers (headcount not published) | Programme-based, not database-based: threatened species, protected-area development, forest ecosystems, island ecosystems, awareness, education; describes "biodiversity information exchange" as part of its mission | Fijian biodiversity | **Active.** News items dated 30 June 2026 (19th anniversary), 20 May 2026, 2 April 2026; earlier item 8 July 2025 (Nanai Project volunteers) (fetched 2026-08-25) | Included as a **negative finding**: the only national conservation NGO in Fiji does *not* run a public species atlas. Fiji's occurrence data documentation runs through the SPREP national portal instead. 19 years old as of 2026 |

#### Vanuatu

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Vanuatu Cultural Centre / Vanuatu Kaljoral Senta](https://vanuatuculturalcentre.gov.vu/) | National | Government institution with named unit leads: **Ambong Thompson** (National Film, Sound and Photo unit), **Evelyne Bulegih** (Women's Culture & Field Worker Program coordinator), **Edson Willie** (Vanuatu National Heritage Register Manager) | **Multi-domain cultural documentation**: National Film, Sound and Photo Archive; the **fieldworker (filwoka) network**; **Vanuatu National Heritage Register**; National Arts Festival photo galleries (2019 onward) | Ni-Vanuatu culture, heritage and place | **Live** (fetched 2026-08-25). Digital database systems, item counts, and oral-tradition/place-name programme detail not on the site — ***unverified*** | The **filwoka model** — a national network of village-based volunteer cultural fieldworkers who document their own places — is the most transferable governance idea in the Pacific for community-sourced place documentation. Pairs a *register* (legal/heritage) with an *archive* (media) under one institution |

#### Hawai'i (framed as Pacific)

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [Papakilo Database](https://www.papakilodatabase.com/) | Statewide, archipelago | **Office of Hawaiian Affairs** (OHA); contractor DL Consulting; partnerships with many cultural and academic institutions | **The most multi-source place-documentation project found in this survey.** Self-described "**Database of Databases**": Hawaiian-language newspapers (1834–1937, plus 492 issues of *Ka Wai Ola* 1980–2023); **Māhele land records** (Land Commission Awards, Royal Patents, Native and Foreign Testimony); the **Kīpuka GIS place-names database**; Hula Preservation Society materials; Hawaiian ethnological notes; **ethnobotany records**; a genealogical Names Index (>110,000 records); historical photographs and artefacts (Louis R. Sullivan, John F.G. Stokes collections); 1800s missionary archives and correspondence | Native Hawaiian land, culture, history, genealogy, biota | **Active but slowing.** Launched **4 April 2011**; as at December 2020 held "**65 unique collections and over 1,000,000 records**"; most recent documented update **6 March 2024** (completion of the *Ka Wai Ola* run to 492 issues) (fetched 2026-08-25) | **Read this before designing your Sathyamangalam architecture.** It is exactly your brief — occurrence-adjacent data, a place-name gazetteer, legal/land instruments, mined historical documents, and literature, unified on one site — executed at million-record scale by an Indigenous governing body, and sustained for 15 years. Contact papakilodatabase@oha.org |
| [Kīpuka Database](https://www.kipukadatabase.com/) | Statewide | Office of Hawaiian Affairs (Honolulu; kipukainfo@oha.org) | GIS built on the **traditional Hawaiian land-division hierarchy**: mokupuni (islands) → moku (districts) → ahupuaʻa → ʻili → kuleana (individual plots); plus **inoa wahi (place names)**, historic records, burial councils, Native Hawaiian population statistics, land-tenure documentation | Native Hawaiian land, culture, history | **Live** (fetched 2026-08-25). Record counts and update dates ***unverified*** | **Uses the Indigenous spatial hierarchy as the primary index**, not colonial administrative units — the deepest structural lesson here for an Indian reserve atlas whose users think in terms of nadu/ur/kadu rather than survey numbers |
| [Ulukau — Hawaiian Electronic Library](https://www.ulukau.org/) | Statewide | Consortium (governing organisation not stated on the site) | Aggregator of Hawaiian-language and cultural collections: newspapers, **dictionaries**, historical documents, genealogies, photographs, audio/video, music resources, academic journals, and **Nā Inoa ʻĀina Hawaiʻi (Hawaiian place names) with maps** | Hawaiian language and culture | **Live** (fetched 2026-08-25). Founding year, operator and update dates not disclosed on the site — ***unverified*** | Fully **Hawaiian-language interface** with English secondary — the strongest example in the region of an atlas whose default language is the documented community's own |
| ~~[Hawaii Biodiversity & Mapping Program](http://hbmp.hawaii.edu/)~~ | Statewide | University of Hawai'i (historically) | Occurrence + rare-species mapping for the Hawaiian archipelago | Hawaiian biota | **DEAD.** `hbmp.hawaii.edu` fails DNS resolution — "Name or service not known" (2026-08-25) | Hawai'i's dedicated biodiversity atlas is gone while its **cultural/land** databases (Papakilo, Kīpuka, Ulukau) thrive. Institutional ownership, not subject matter, determines survival |
| ~~[PIER — Pacific Island Ecosystems at Risk](http://www.hear.org/pier/)~~ | Pacific-wide | US Forest Service, Institute of Pacific Islands Forestry; contact **Jim Space** | Invasive plant species profiles across Pacific islands | Pacific invasive plants | **DEAD — self-declared, with an explicit orphan notice.** Site states: "The last updates were made **15 May 2013** and were posted **1 June 2013**", and that the programme is "seeking an individual or organization that is interested in maintaining and possibly updating the PIER web site." Kept online as an archive (fetched 2026-08-25) | The most explicit **succession-failure notice** in this survey. Note also that its host domain `hear.org` (Hawaiian Ecosystems at Risk) was intermittently unreachable during this survey — connection timeout on robots.txt fetch (2026-08-25) |

---

### Antarctica and Subantarctic

| Project (URL) | Scale | Team size & type | Scope | Subject | Active status (+ evidence & date) | Notable |
|---|---|---|---|---|---|---|
| [SCAR Antarctic Biodiversity Portal — biodiversity.aq](https://biodiversity.aq/) | Continental + Southern Ocean | Multi-institution: Royal Belgian Institute of Natural Sciences, Belgian Biodiversity Platform, WoRMS, GBIF, OBIS, SCAR | Multi-domain: occurrence data across the SCAR and OBIS networks, **literature**, identification keys, CCAMLR ecosystem-monitoring data, a Southern Ocean fish diversity dashboard, links to external databases | Antarctic and Southern Ocean biodiversity | **Very active.** News items **6 August 2026** (SCAR OSC 2026, Oslo 10–14 August), **3 August 2026** (SYNCHRONY project launch), **15 June 2026** (call for photos "celebrating two decades of the portal" → founded ~2006) (fetched 2026-08-25) | Its homepage counters are deliberately structured as **"occurrences in the SCAR network", "occurrences in the OBIS network", and "occurrences in both"** — i.e. it displays the *overlap and disagreement between two source networks* on the front page rather than a single merged total. **This is the closest working implementation of your disputed-figures feature that I found anywhere in Oceania.** |
| [Biodiversity of Ice-free Antarctica Database](https://www.antarctica.gov.au/news/2025/antarctic-biodiversity-database-has-ice-free-areas-covered/) (via [AADC](https://data.aad.gov.au/)) | All ice-free Antarctic terrain | Australian Antarctic Division; led by **Dr Aleks Terauds**, Program Leader | Occurrence compilation drawn from the SCAR Antarctic Biodiversity Database **plus herbaria and field notes** — i.e. an explicit historical-document-mining component | Mosses, lichens, fungi, invertebrates, microbes, birds, seals | **Published 2025** in *Ecology*; **16 years** of compilation. **35,600+ records**, **1,890 species**, spanning early 1800s to 2019 (majority post-1950) | A bounded, citable, versioned dataset release rather than a perpetually shifting portal — and it names the 16-year effort and the sources (herbaria, field notes) that had to be mined to reach coverage |
| [SCAR Composite Gazetteer of Antarctica](https://placenames.aq/) · [SCAR page](https://scar.org/resources/place-names) | South of 60°S, terrestrial + undersea + under-ice | **Jointly managed by Italy and Australia since 2008** — Italy edits, Australia maintains database and website; coordinated by the SCAR Standing Committee on Antarctic Geographic Information (SCAGI); contact **Prof. Carlo Baroni** | Gazetteer aggregating names submitted by **national gazetteer authorities** | Antarctic place names | **Live.** Redirects from the legacy AADC URL to placenames.aq; app identifies as "SCAR CGA" (fetched 2026-08-25). The site is JavaScript-rendered so name counts could not be read — ***unverified*** | **The definitive model for a multi-authority contested gazetteer.** Its design: each recognised feature gets a **numerical Unique Identifier (UID)**, and the UID carries **a list of applicable place names** — deliberately not one canonical name. It also states outright that the CGA "is compiled purely for the convenience of the scientific community and **has no legal authority or standing**". Both the UID-plus-many-names data model and the explicit disclaimer of authority are directly transferable to Sathyamangalam toponymy |
| [Australian Antarctic Data Centre](https://data.aad.gov.au/) | Australian Antarctic Territory + Southern Ocean | Australian Antarctic Division (government) | Multi-domain: metadata catalogue, biodiversity collections, gazetteers, maps, species profiles, science datasets | Antarctic science | **Live** but **JavaScript-only** — the landing page returns "this app doesn't work properly without JavaScript enabled" to fetchers, so domain inventory and counts are ***unverified*** (2026-08-25) | Cautionary technical note: several of the most authoritative portals in this survey (AADC, placenames.aq, Natural Values Atlas, AUSTLANG, Biota of NZ, Trove) are **invisible to non-JavaScript clients** — which means invisible to archiving, citation-checking and much automated verification. Design against this |
| [Antarctic Soils Explorer](https://www.landcareresearch.co.nz/tools-and-resources/databases/) (Manaaki Whenua) | Antarctic (Ross Sea sector focus) | Manaaki Whenua – Landcare Research (NZ) | Soils research data plus **historical explorer narratives** | Antarctic soils | **Listed as live** on the Manaaki Whenua database index (fetched 2026-08-25); not independently fetched — ***unverified*** | An unusual pairing of physical-science data with historical exploration accounts about the same ground |
| [New Zealand Gazetteer — Ross Sea region](https://www.linz.govt.nz/products-services/place-names/place-names-new-zealand) | Ross Dependency | NZGB / LINZ | Official place names for Antarctica's Ross Sea region, held in the same gazetteer as NZ's | Antarctic place names | **Active** (fetched 2026-08-25) | A national gazetteer that extends into Antarctica — and whose names also feed the SCAR CGA, so the same feature can hold an NZGB name and other nations' names under one CGA UID |

---

#### Dead / dormant projects found

| Project | Status | Evidence (all checked 2026-08-25 unless dated) |
|---|---|---|
| **Papuaweb** (`papuaweb.org`) | **Dead — domain gone** | DNS resolution failure: "No address associated with hostname". Was a multi-source-type documentation site for Papua/West Papua (theses, bibliographies, maps, colonial documents) — i.e. the single closest dead analogue to the project being planned |
| **PNGtrees** (`pngtrees.org`) | **Dead — domain gone** | DNS resolution failure: "Name or service not known". Content partly survives inside pngplants.org |
| **NW Atlas** (`nwatlas.org.au`) | **Dead — domain gone** | DNS resolution failure: "Name or service not known" — despite still being named as a live partner site on the [eAtlas about page](https://eatlas.org.au/content/about-e-atlas) ("partner sites covering the Northwest coast and Ningaloo regions"). A live site advertising a dead sibling |
| **Hawaii Biodiversity & Mapping Program** (`hbmp.hawaii.edu`) | **Dead — domain gone** | DNS resolution failure: "Name or service not known" |
| **PIER — Pacific Island Ecosystems at Risk** | **Dead — orphaned, explicitly** | Site states: "The last updates were made **15 May 2013** and were posted **1 June 2013**", and the US Forest Service is "seeking an individual or organization that is interested in maintaining and possibly updating the PIER web site." Contact given as Jim Space. Host domain `hear.org` also intermittently unreachable (robots.txt ConnectTimeout) |
| **Dictionary of Sydney** | **Frozen — archived** | Homepage states plainly: "**This site was archived in 2021**." Content remains searchable and browsable; no new entries. Was a major multi-source single-place documentation project (entries, images, oral-history audio, walking-tour app) |
| **Pacific Islands Protected Area Portal (PIPAP)** | **Dormant** | Dashboard renders "**0 Protected Areas (0 Terrestrial, 0 Marine)**"; newest substantive content is an "Action Plan 2014–2020" and a PAWG presentation from Suva, **July 2015** |
| **Tasmanian Threatened Species Link — fauna component** | **Partially dormant, self-declared** | On-site disclaimer: "The fauna data on the Threatened Species Link is **currently not being maintained**. As such, some of the information may be out of date." Most recent dated update seen: **1 December 2022** |
| **Living Archive of Aboriginal Languages (CDU)** | **Wound down / migrated** | Homepage states materials have been "**moved to a new home: Territory Stories at Library & Archives NT**", directing enquiries to territorystories@nt.gov.au. Held ~**4,000 books in 50 languages from 40 communities**; launched 2012 |
| **Victorian Places (Monash + UQ)** | **Likely dormant** | Copyright notice **2015**; a forward-looking note about a 2016 census update that appears not to have been completed; no visible recent additions. >1,600 places documented |
| **Australian Faunal Directory** | **Slowing** | Latest per-family updates dated **11 April 2024** — nothing newer visible after 16 months. Holds 127,933 species/subspecies in 4,468 families |
| **NZOR (New Zealand Organisms Register)** | **Apparently frozen** | Homepage still describes the register in the *future tense* ("will be electronically available through one or more portals") and cites a **2000** scoping document and a **2006** steering group; no news, version or update date anywhere on the site. Underlying data refresh status ***unverified*** |
| **NARvis (Northern Agricultural Region, WA)** | **Likely dormant** | Cited material spans **2008–2020**; no update date published |
| **NatureMapr "flatter national structure" announcement** | **Content deleted** | `naturemapr.org/announcements/586` returns "This record no longer exists or has been deleted", though the announcement is still indexed by search engines under the title "NatureMapr moves to simpler, flatter national structure". The restructure appears real (naturemapr.org and canberra.naturemapr.org now serve identical counters) but its own announcement is gone |
| **BowerBird** (`bowerbird.org.au`) | **Uncertain — flag, do not assume** | The homepage renders and still solicits sign-ups ("Ready to contribute to the BowerBird community? Join BowerBird"), but **no dates or record counts appear anywhere on it**, and every sub-path I tried (`/observations`, `/sightings`) failed with robots.txt fetch errors. I could not obtain evidence of any activity, nor evidence of closure. Treat as *status unknown* |

---

#### Leads I could not verify

My WebSearch budget was exhausted early (200/200 calls, shared session limit), and direct `curl` egress is blocked by this session's proxy policy (403 on CONNECT for arbitrary hosts, confirmed against `archive.org` among others). Everything below was therefore either unreachable, JavaScript-gated, or reachable only as a name. **None of these should be cited without independent checking.**

- **Nadeaud Database (French Polynesia)** — the botanical database of the Délégation à la Recherche / Herbarium of French Polynesia. I could not locate or confirm a working URL. **The most significant gap in this report:** French Polynesia is the one major Pacific jurisdiction for which I found no verified documentation project at all.
- **Solomon Islands, Tonga, Kiribati, Palau, Nauru, Niue, Tuvalu, RMI, FSM, Cook Islands** — each has an Inform national environment portal named on the SPREP Inform page, but I fetched only the Samoa and Vanuatu instances. Dataset counts, tech stack (I believe CKAN-family, **unverified**) and update dates for all portals are unverified.
- **New Guinea Binatang Research Centre** (PNG community-based insect research and parataxonomist training) — not fetched.
- **Trove (National Library of Australia)** — `trove.nla.gov.au/about` returned **Access Denied**. Item counts, contributing-organisation count and digitised newspaper page counts unverified, despite Trove being the largest historical-document-mining platform in the region.
- **BHL Australia (Museums Victoria node of the Biodiversity Heritage Library)** — `biodiversitylibrary.org.au` does not resolve and my guessed Museums Victoria path 404'd. Existence is near-certain; URL, page counts and current status unverified.
- **AUSTLANG** entry count, per-entry fields, and update cadence — site is JavaScript-only.
- **SCAR Composite Gazetteer of Antarctica** — total place-name count, total UID/feature count, and the list of contributing national gazetteers. JavaScript-only.
- **Natural Values Atlas** application itself (`naturalvaluesatlas.tas.gov.au`) — JavaScript-only; all figures quoted come from the NRE Tas description page, not the app.
- **Australian Antarctic Data Centre** — full inventory of hosted databases and tools. JavaScript-only.
- **Biota of New Zealand** — taxon/name counts, last-updated date, and whether it exposes conflicting taxonomic treatments.
- **Heritage New Zealand List / Rārangi Kōrero** — entry counts and category breakdown (historic places / historic areas / wāhi tapu / wāhi tūpuna).
- **ArchSite** — recorded site count, per-site data fields, scheme inception date.
- **iNaturalist NZ** — observation, species and observer totals (`/stats` robots-blocked).
- **Ningaloo Atlas** — recent-activity dates; both `/blog` and `/latest-news` returned 404, so I cannot say whether it is being added to or merely still hosted.
- **NT Fauna Atlas / NT Flora Atlas** — record counts, update dates, and team; I saw only search-result metadata and third-party registry entries, never fetched the nt.gov.au pages.
- **Seamap Australia** — launch year, layer/dataset counts.
- **Reef Life Survey** — survey, site, species and country totals; founding year.
- **DIISE** — eradication-event count and island count.
- **Australian Heritage Database** (including whether the Register of the National Estate remains frozen) — the search CGI is robots-disallowed.
- **RIMReP / Reef 2050 Knowledge System (GBRMPA)** — `www2.gbrmpa.gov.au` robots.txt fetch timed out. Existence likely; status unverified.
- **PlantNET, FloraBase, FloraSA, VicFlora** — none publishes a taxon count or a database-level last-updated date on its landing page. All four counts unverified.
- **Endemia.nc** — the year of the change-log entries (the log showed "25–26 May" without a year), total taxa, organisational structure, and its Red List authority role.
- **Papakilo Database** — whether anything has been added since the March 2024 *Ka Wai Ola* completion.
- **Ulukau** — operator, founding year, collection counts.
- **Kīpuka Database** — record counts and update dates.
- **Vanuatu Cultural Centre** — whether the National Heritage Register or the film/sound/photo archive is digitally searchable, and any item counts.
- **BowerBird** — see the dead/dormant table; status genuinely unknown.
- **Composite Gazetteer of Australia** — total name count and contributing-agency list.
- **Regional/catchment atlases generally** — I found and verified four (Atlas of Life in the Coastal Wilderness, Connecting Country, Love our Lakes, NARvis) plus the NatureMapr nodes. There are certainly more NRM-body and catchment-authority atlases across all Australian states that this pass did not reach.
- **IBRA/IMCRA** current versions and bioregion/subregion counts — the DCCEEW page's robots.txt read timed out.

---

#### Sources

**Australia — national**
- [Birdata — About](https://birdata.birdlife.org.au/about-birdata) · [BirdLife Australia programs](https://birdlife.org.au/programs/birdata/) · [Atlas of Australian Birds (Wikipedia)](https://en.wikipedia.org/wiki/Atlas_of_Australian_Birds)
- [DigiVol](https://volunteer.ala.org.au/)
- [Bush Blitz](https://bushblitz.org.au/)
- [FrogID](https://www.frogid.net.au/)
- [Fungimap](https://fungimap.org.au/)
- [Australian Faunal Directory](https://biodiversity.org.au/afd/home) · [Australian National Species List (APNI/APC)](https://biodiversity.org.au/nsl/) · [Flora of Australia online](https://profiles.ala.org.au/opus/foa)
- [TERN](https://www.tern.org.au/) · [AODN Portal](https://portal.aodn.org.au/) · [Seamap Australia](https://seamapaustralia.org/) · [Reef Life Survey](https://reeflifesurvey.com/)
- [eAtlas — About](https://eatlas.org.au/content/about-e-atlas) · [eAtlas datasets](https://eatlas.org.au/datasets?page=6) · [eAtlas metadata catalogue record](https://catalogue.eatlas.org.au/geonetwork/srv/api/records/d2396b2c-68d4-4f4b-aab0-52f7bc4a81f5?language=eng)
- [EPBC Protected Matters Search Tool](https://www.environment.gov.au/epbc/protected-matters-search-tool)
- [Place Names (Composite Gazetteer of Australia)](https://placenames.fsdf.org.au/)
- [AIATSIS AustLang](https://aiatsis.gov.au/austlang/about) · [Nyingarn](https://nyingarn.net/) · [Gambay](https://gambay.com.au/) · [Living Archive of Aboriginal Languages](https://livingarchive.cdu.edu.au/) · [LAAL (Wikipedia)](https://en.wikipedia.org/wiki/Living_Archive_of_Aboriginal_Languages)
- [Aṟa Irititja — Who are we?](https://irititja.com/about-ara-irititja/who-are-we/) · [Aṟa Irititja](https://irititja.com/) · [Keeping Culture KMS](https://keepingculture.com/)
- [NatureMapr Australia](https://naturemapr.org/) · [NatureMapr — deleted announcement 586](https://naturemapr.org/announcements/586)
- [DIISE](http://diise.islandconservation.org/)
- [BowerBird](https://www.bowerbird.org.au/) · [Trove — About (access denied)](https://trove.nla.gov.au/about)

**Australia — NSW**
- [About BioNet Atlas](https://www.environment.nsw.gov.au/topics/animals-and-plants/biodiversity/nsw-bionet/about-bionet-atlas) · [NSW BioNet](https://www.environment.nsw.gov.au/topics/animals-and-plants/biodiversity/nsw-bionet) · [BioNet Species Sightings data](https://www.environment.nsw.gov.au/topics/animals-and-plants/biodiversity/nsw-bionet/about-bionet-atlas/species-sightings-data) · [BioNet on Data.NSW](https://data.nsw.gov.au/data/dataset/nsw-bionet-species-sightings-data-collection8a9c4) · [NSW BioNet Atlas on GBIF](https://www.gbif.org/dataset/0645ccdb-e001-4ab0-9729-51f1755e007e)
- [PlantNET](https://plantnet.rbgsyd.nsw.gov.au/) · [NSW Flora Online (Botanic Gardens of Sydney)](https://www.botanicgardens.org.au/our-science/our-services/nsw-flora-online-plantnet) · [PlantNET limits of data](https://plantnet.rbgsyd.nsw.gov.au/limitofdata.html)
- [SEED NSW](https://www.seed.nsw.gov.au/) · [AHIMS](https://www.environment.nsw.gov.au/topics/heritage/search-heritage-databases/aboriginal-heritage-information-management-system)
- [NSW Geographical Names Board](https://www.nsw.gov.au/departments-and-agencies/geographical-names-board)
- [Atlas of Life in the Coastal Wilderness](https://atlasoflife.org.au/) · [NatureMapr Budawang Coast](https://budawangcoast.naturemapr.org/) · [Canberra Nature Map via NSW Landcare Gateway](https://landcare.nsw.gov.au/groups/kosciuszko-to-coast/recommended-resources/canberra-nature-map/)
- [Dictionary of Sydney](https://dictionaryofsydney.org/)

**Australia — Victoria**
- [About the VBA](https://www.environment.vic.gov.au/biodiversity/victorian-biodiversity-atlas/about-the-vba) · [Victorian Biodiversity Atlas](https://www.environment.vic.gov.au/biodiversity/victorian-biodiversity-atlas) · [VBA on GBIF](https://www.gbif.org/dataset/37beb6ea-e591-41e3-b781-e384250dc42c) · [VBA marine records on OBIS](https://obis.org/dataset/0264be1a-9d3f-495d-afcb-ac22718a70ce) · [VBA flora records (DataVic)](https://discover.data.vic.gov.au/dataset/victorian-biodiversity-atlas-flora-records-unrestricted-for-sites-with-high-spatial-accuracy) · [VBA fauna records (DataVic)](https://discover.data.vic.gov.au/dataset/victorian-biodiversity-atlas-fauna-records-unrestricted-for-sites-with-high-spatial-accuracy)
- [VicFlora — About](https://rbgvictoria.github.io/vicflora-static-pages/about/) · [VicFlora software upgrade (2022)](https://vicflora.rbg.vic.gov.au/articles/2022/07/14/vicflora-software-upgrade) · [VicFlora relaunch media release](https://www.rbg.vic.gov.au/news-and-stories/vicflora-relaunch-media-release/)
- [Victorian Places](https://www.victorianplaces.com.au/)
- [Connecting Country — What is the VBA?](https://connectingcountry.org.au/what-is-the-victorian-biodiversity-atlas-vba-and-why-should-we-use-it/) · [Love our Lakes Biodiversity Atlas](https://loveourlakes.net.au/resources/biodiversity-atlas/)

**Australia — Queensland**
- [WildNet platform](https://www.qld.gov.au/environment/plants-animals/species-information/wildnet) · [WildNet species search](https://wildnet.science-data.qld.gov.au/) · [WildNet on GBIF](https://www.gbif.org/dataset/e4473544-f4cd-4429-b791-39d6e1fdb0a4) · [WildNet wildlife records (Qld Open Data)](https://www.data.qld.gov.au/dataset/wildnet-wildlife-records-published-queensland) · [Qld WildNet Data API](https://www.data.qld.gov.au/dataset/qld-wildlife-data-api)
- [WetlandInfo](https://wetlandinfo.detsi.qld.gov.au/wetlands/)

**Australia — SA / WA / Tas / NT**
- [BDBSA](https://www.environment.sa.gov.au/topics/science/information-and-data/biological-databases-of-south-australia) · [NatureMaps](https://data.environment.sa.gov.au/NatureMaps/pages/default.aspx) · [BDBSA overview factsheet (PDF, 2023)](https://data.environment.sa.gov.au/Content/Publications/bdbsa-overview-fact.pdf) · [SA Fauna on GBIF](https://www.gbif.org/dataset/40c0f670-ee87-4576-be9d-2725e0b47035) · [Electronic Flora of South Australia](https://www.flora.sa.gov.au/)
- [NatureMap WA help](https://naturemap.dbca.wa.gov.au/) · [NatureMap on GBIF](https://www.gbif.org/publisher/cd459985-539a-4dc6-bfab-48bd1222ca51) · [NatureMap: Mapping WA's Biodiversity (DBCA PDF)](https://library.dbca.wa.gov.au/Journals/080525/080525-26.pdf) · [FloraBase](https://florabase.dbca.wa.gov.au/) · [NARvis biodiversity](https://narvis.com.au/the-region/biodiversity/) · [Ningaloo Atlas](https://ningaloo-atlas.org.au/)
- [Natural Values Atlas (NRE Tas)](https://nre.tas.gov.au/conservation/development-planning-conservation-assessment/planning-tools/natural-values-atlas) · [NVA application](https://www.naturalvaluesatlas.tas.gov.au/) · [Tasmanian NVA on GBIF](https://www.gbif.org/dataset/2985efd1-45b1-46de-b6db-0465d2834a5a) · [NVA Newsletter 2021 (PDF)](https://nre.tas.gov.au/Documents/NVA%20Newsletter%202021_1.pdf) · [Threatened Species Link](https://www.threatenedspecieslink.tas.gov.au/)
- [Fauna Atlas NT](https://nt.gov.au/environment/environment-data-maps/fauna-atlas) · [Flora Atlas NT](https://nt.gov.au/environment/environment-data-maps/flora-atlas) · [Fauna Atlas NT metadata (NTLIS)](http://www.ntlis.nt.gov.au/metadata/export_data?type=html&metadata_id=2DBCB771208B06B6E040CD9B0F274EFE) · [NT Flora Atlas dataset](https://data.nt.gov.au/dataset/nt-flora-atlas) · [NT Fauna Atlas (AGLDWG)](https://catalogue.linked.data.gov.au/org/nt-atlas)
- [Canberra Nature Map](https://canberra.naturemapr.org/)

**New Zealand**
- [NZ Bird Atlas portal](https://ebird.org/atlasnz/home) · [Atlas news](https://ebird.org/atlasnz/news) · [Birds NZ Atlas Scheme](https://www.birdsnz.org.nz/schemes/nz-bird-atlas-scheme/) · [Media release, 31 May 2024 (PDF)](https://www.birdsnz.org.nz/wp-content/uploads/2024/05/MEDIA-RELEASE-NZ-Bird-Atlas-Project-Complete.pdf) · [eBird Portal Spotlight](https://ebird.org/news/portal-spotlight-new-zealand-bird-atlas)
- [iNaturalist NZ](https://inaturalist.nz/) · [DOC biodiversity resources](https://www.doc.govt.nz/nature/biodiversity/biodiversity-new-zealand-resources/)
- [NVS Databank](https://nvs.landcareresearch.co.nz/) · [NVS about (DataStore)](https://datastore.landcareresearch.co.nz/organization/about/national-vegetation-survey-nvs) · [NVS on re3data](https://www.re3data.org/repository/r3d100011128) · [NVS in NZJE](https://newzealandecology.org/nzje/2124)
- [NZOR](https://www.nzor.org.nz/) · [NZOR — what data is provided](https://www.nzor.org.nz/what-data-is-provided) · [NZOR — data quality and use](https://www.nzor.org.nz/data-quality-and-use) · [NZOR scope (TFBIS PDF)](https://www.nzor.org.nz/documents/nzor-scope.pdf) · [NZOR on GBIF](https://www.gbif.org/dataset/134eca5f-65ab-49a2-a229-3d0d35fcbefe)
- [Manaaki Whenua databases index](https://www.landcareresearch.co.nz/tools-and-resources/databases/) · [Biota of New Zealand](https://biotanz.landcareresearch.co.nz/) · [Ngā Rauropi Whakaoranga](https://maoriplantuse.landcareresearch.co.nz/)
- [NZPCN](https://www.nzpcn.org.nz/) · [NZPCN flora](https://www.nzpcn.org.nz/flora/) · [NZPCN plant lists](https://www.nzpcn.org.nz/publications/plant-lists/national-plant-lists/) · [NZPCN (Wikipedia)](https://en.wikipedia.org/wiki/New_Zealand_Plant_Conservation_Network)
- [NZ Birds Online](https://nzbirdsonline.org.nz/)
- [NZ Freshwater Fish Database](https://nzffdms.niwa.co.nz/)
- [Place names of New Zealand (LINZ)](https://www.linz.govt.nz/products-services/place-names/place-names-new-zealand) · [NZ Geographic Board](https://www.linz.govt.nz/our-work/new-zealand-geographic-board) · [Place naming guidance](https://www.linz.govt.nz/guidance/place-naming) · [824 Māori place names made official (Scoop, 2019)](https://www.scoop.co.nz/stories/CU1906/S00285/824-maori-place-names-made-official.htm) · [RNZ coverage](https://www.rnz.co.nz/news/national/392890/more-than-800-maori-place-names-officially-recognised)
- [Kā Huru Manu](https://www.kahurumanu.co.nz/) · [Kā Huru Manu Atlas](https://www.kahurumanu.co.nz/atlas)
- [Māori GIS — place names](https://maorigis.nz/docs/data/place-names) · [Matawhenua viewer](https://maps.maorigis.nz/)
- [ArchSite](https://www.archsite.org.nz/) · [Heritage NZ List](https://www.heritage.org.nz/list) · [Te Papa Collections Online](https://collections.tepapa.govt.nz/)

**Papua New Guinea**
- [PNGplants](http://www.pngplants.org/) · [PNGtrees (dead)](https://www.pngtrees.org/) · [Papuaweb (dead)](http://www.papuaweb.org/)

**Pacific islands**
- [Endemia.nc](https://endemia.nc/)
- [SPREP Inform](https://www.sprep.org/inform) · [Pacific Environment Portal](https://pacific-data.sprep.org/) · [Samoa portal](https://samoa-data.sprep.org/) · [Vanuatu portal](https://vanuatu-data.sprep.org/) · [PIPAP](https://www.sprep.org/pipap)
- [NatureFiji-MareqetiViti](https://www.naturefiji.org/) · [Vanuatu Cultural Centre](https://vanuatuculturalcentre.gov.vu/)
- [Papakilo Database](https://www.papakilodatabase.com/) · [Kīpuka Database](https://www.kipukadatabase.com/) · [Ulukau](https://www.ulukau.org/) · [PIER (dead)](http://www.hear.org/pier/) · [HEAR](https://www.hear.org/) · [Hawaii Biodiversity & Mapping Program (dead)](http://hbmp.hawaii.edu/)

**Antarctica / Subantarctic**
- [biodiversity.aq](https://biodiversity.aq/) · [SCAR biodiversity.aq page](https://scar.org/library-data/data/biodiversity) · [SCAR Data & Databases](https://scar.org/library-data/data) · [SCAR-AntOBIS on GBIF](https://www.gbif.org/publisher/104e9c96-791b-4f14-978c-f581cb214912)
- [Antarctic biodiversity database has ice-free areas covered (AAD news, 2025)](https://www.antarctica.gov.au/news/2025/antarctic-biodiversity-database-has-ice-free-areas-covered/) · [AADC](https://data.aad.gov.au/) · [AADC find data](https://data.aad.gov.au/find-data) · [AADC biodiversity collections](https://data.aad.gov.au/aadc/biodiversity/collections.cfm) · [AADC (Wikipedia)](https://en.wikipedia.org/wiki/Australian_Antarctic_Data_Centre)
- [SCAR Composite Gazetteer of Antarctica](https://placenames.aq/) · [SCAR place names page](https://scar.org/resources/place-names)

---

**Three things worth acting on immediately, in priority order:** (1) **[Kā Huru Manu](https://www.kahurumanu.co.nz/)** and (2) **[Papakilo](https://www.papakilodatabase.com/)** are the two closest existing implementations of exactly what you are building — one at iwi scale with 1,000+ place names and oral histories, one at million-record scale across newspapers, land instruments, place names and ethnobotany, both governed by the documented community. (3) For the disputed-figures feature specifically, three designs are already in production and worth copying rather than inventing: the **SCAR CGA's** UID-carrying-a-list-of-names model with an explicit disclaimer of legal authority; **APNI/APC's** separation of *every published usage* from *the one accepted concept*; and **biodiversity.aq's** front-page counters that report SCAR-network, OBIS-network, and both-networks occurrences separately instead of a single merged total.

---

# Task 2 — How This Type of Project Actually Works

**Prepared for:** the Sathyamangalam Tiger Reserve conservation atlas project
**Compiled:** 26 August 2026
**Evidence base:** `Task1_Worldwide_Inventory.md` (250+ named projects, six regional passes, verified 25 August 2026). Every project named below is drawn from that inventory; its URL, status evidence and verification date are recorded there and are not repeated in full here.
**Supplementary verification:** funding-programme mechanics (grant tiers, envelope sizes, co-funding rules) are the one thing the Task 1 corpus systematically does *not* record, because the projects themselves do not publish it. Those figures were verified independently on 26 August 2026 and are cited in this task's own Sources section.

---

## 2.0 A finding that has to come first: the disclosure problem

Before any analysis of funding or team size, the Task 1 corpus establishes something that constrains everything in this task:

> **Across 250+ projects individually checked, only a handful publish either their team size or their funding.**

Task 1 states this explicitly for North America ("For most state natural heritage programmes, herbarium consortia and solo sites, staffing and funding are simply not published") and the same gap recurs in every regional pass — Sahapedia publishes no team size or article count despite its visibility; the Digital Atlas of Idaho publishes no authorship, funding or date anywhere reachable; Wild Atlas (India) publishes no team, institution or methodology at all.

Three consequences for this task:

1. **Any generic claim about "typical team size" in this sector is unfounded.** What follows is built only from the projects that actually disclose, plus what can be inferred from published output rates. Where a number is inferred rather than stated, it says so.
2. **Non-disclosure is itself a maintenance signal.** Task 1's North America pass reaches this conclusion directly on the Alaska Native Place Names Project: "Publish counts and dates or your project reads as abandoned even if it isn't."
3. **There is severe survivorship bias in the corpus.** Live sites are easier to discover than dead ones; the dead projects Task 1 found are disproportionately those that left a trace (a paper, an aggregator entry, an institutional page). The true death rate in this sector is certainly higher than the corpus shows. Treat every lifespan figure below as an upper bound on health, not a central estimate.

---

## 2.1 Six archetypes — because funding, team and lifespan all follow from which one you are

The corpus does not contain one kind of project. It contains six, and they have different economics, different failure modes and different observed lifespans. Every later section in this task is organised against this typology.

| # | Archetype | Who runs it | Typical disclosed headcount | Money source | Characteristic death | Task 1 exemplars |
|---|---|---|---|---|---|---|
| **A** | **Statutory / agency infrastructure** | A government department with a legal mandate | Dedicated data team, size almost never published | Core appropriation | Almost none observed — but catastrophic when it happens (cyberattack, machinery-of-government change) | Natural Values Atlas (Tas), BioNet Atlas (NSW), WildNet (Qld), SNIB/CONABIO (Mexico), Artsdatabanken (Norway), MyBIS (Malaysia), HKBIH (Hong Kong), Hawaii Biological Survey (statutory 1992), Historic Place Names of Wales (statutory 2016) |
| **B** | **Multi-institution consortium / national node** | A formal alliance of named institutions | 11–230 member institutions; central staff small | Mixed: state programme funding + member in-kind | Integration layer dies while members survive | TBIA (19+ members, Taiwan), SBDI (11 institutions, Sweden), NDFF Verspreidingsatlas (7 taxon societies, NL), Bhutan Biodiversity Portal (6 institutions), SERNEC (230 herbaria), FinBIF (Finland), SiBBr (196 institutions, Brazil) |
| **C** | **Learned society / membership NGO** | A registered society or charity with members | Volunteer officers + 1 named quality gate + hundreds–thousands of recorders | Membership dues, small grants, earned service income | Fieldwork continues, the *website* fossilises | NZPCN (>1,000 members, 2003–), Essex Field Club (7,593,231 records), NatureSpot (8,495 species), Maryland Biodiversity Project, Calflora, Naturbasen (119,474 members, DK), BSPB (Bulgaria), MME (Hungary), Massachusetts Butterfly Club |
| **D** | **Small academic lab / university-hosted** | A named PI or research group | 1 lab to ~5 people | Competitive research grants | Grant ends; hosting rots on a university server | Marine Biodiversity Portal of Bangladesh (one lab), BioGIS (HUJI), Filotis (NTUA hydrology group), Lyudao LTSER (Taiwan), E-Flora/E-Fauna BC (UBC Geography), Biodiversidata (Uruguay), Janzen & Hallwachs ACG inventory (two people) |
| **E** | **Solo / near-solo** | One named person, sometimes with a volunteer periphery | 1–4 named | Effectively zero, or one small grant | Succession failure — the person stops | Plantarium (Oreshkin), atlas-roslin.pl (Snowarski), EcoRegistros (La Grotteria), vncreatures (Vietnam), PNGplants (Barry Conn), Illinois Wildflowers (John Hilty), Minnesota Wildflowers (Katy Chayka), Wildflowers of Ireland (Zoë Devlin), Māori GIS (Duane Wilkins), Open Domesday (Anna Powell-Smith), Gatehouse Gazetteer (Philip Davis), Vici.org (René Voorburg), Flora of Zimbabwe cluster (4 named), Co's Digital Flora of the Philippines (3 editors), Caribbean Herpetology (4 academics), FaunaParaguay |
| **F** | **Community-governed / Indigenous-authority atlas** | The documented community itself, usually with a technical partner | Community institution + university/contractor tech team | Treaty settlements, land-claim processes, cultural agencies, philanthropy | Domain/hosting loss after the partnership ends | Kā Huru Manu (Ngāi Tahu), Papakilo & Kīpuka (Office of Hawaiian Affairs), Gwich'in Atlas, Kitikmeot Place Name Atlas, Ara Irititja (APY), Terras Indígenas no Brasil (ISA), Ogiek Ancestral Territories Atlas, Vanuatu Cultural Centre |

**The Sathyamangalam project as currently described is an E aspiring to D/F.** That is not a marginal position in this corpus — it is one of the two most durable positions in it (see §2.6). But it is durable for reasons that are specific and copyable, not accidental.

---

## 2.2 Funding — what actually pays for these

### 2.2.1 The seven mechanisms, with verified figures

| Mechanism | Verified figures | Task 1 projects using it | Observed durability |
|---|---|---|---|
| **1. Statutory mandate + core appropriation** | No cash figures published by any project in the corpus | Hawaii Biological Survey (mandate from the Hawaii State Legislature, **1992**; *Records of the HBS* published annually since **1994**); List of Historic Place Names of Wales (statutory under the **Historic Environment (Wales) Act 2016**, launched 2017); NSW Geographical Names Board (*Geographical Names Act 1966*); Chile's species-classification register (**Decreto N° 29 de 2011**, replacing Decreto N° 75 de 2004); Chile SBAP under **Law 21.600** from 2 Feb 2026 | **Highest in the corpus.** No statutory project in Task 1 was found dead. The two failures are non-financial: France's INPN (cyberattack) and PIPAP (dashboard rendering zeroes) |
| **2. Competitive national research grants** | NSF awarded iDigBio **~US$20 million over five years** (2021) as the coordinating hub of the ADBC programme; the iDigBio portal holds **>128 million specimen records representing ~400 million specimens**, ~40% of US collection holdings, feeding **>2,000 studies**. *Total ADBC programme investment across all TCNs: **unverified*** | SERNEC (**NSF Award 1410069**); Pladias (Czech Science Foundation, **2014–2018**); HGIS de las Indias (Austrian FWF, **2015–2019**); Nyingarn (**ARC LE200100006**); Residential Schools Land Memory Atlas (**SSHRC 2015–2020**); AfricanBioServices (EU H2020, ~2015–2020); Nuohtti (EU Interreg Nord, 2018–2021); Time Machine (EU **H2020 grant 820323**) | **Lowest in the corpus.** Deaths cluster tightly at grant end — see §2.6, Stage 2 |
| **3. Small grants aimed at the Global South** | **Rufford Foundation** tiers (verified 26 Aug 2026): 1st Small Grant **£7,000**, 2nd **£8,000**, Booster **£12,000**, Completion **£18,000**; one application per 12 months, sequential, no further funding after a Completion Grant. **GBIF BID** phase 1: **€3.9 million** EU-funded programme; first round awarded **€972,156 across 23 projects** (3 regional, 10 national, 10 small) to **34 organisations in 20 African countries**, leveraging **€817,369** in co-funding — a mean of ~**€42,000** per project. **JRS Biodiversity Foundation**: grants range **US$7,000–US$525,000**, durations **12–48 months**, most 2025–26 awards **US$60,000–US$400,000**, focused on sub-Saharan Africa | Co's Digital Flora of the Philippines (**Rufford**-funded in part — exact tier unverified, but capped by the table at left); National Museums of Kenya East African Herbarium (**JRS**, 2018) | Mixed. Co's Digital Flora has run **2011–2026** on this scale of money. But the Rufford ladder terminates by design, so it cannot be a maintenance income |
| **4. National agency replication grants** (the most relevant model) | France's **Atlas de la Biodiversité Communale (ABC)** programme, verified figures: **2020 — €2.5M / 46 projects** (~€54k mean); **2021 — €5.0M / 98 projects** (~€51k); **2022 — €2.0M / 47 projects** (~€43k); **2023 — €7.6M / 100 projects** (~€76k). A **€15 million** envelope is available under the Stratégie Nationale Biodiversité. Open to communes, intercommunalities, mixed syndicates and national parks. *Per-project caps and co-financing rate: **not published** on either the OFB funding page or the Aides-territoires listing* | The ABC programme itself; regionally, a worked example is 14 communes in the PNR du Massif des Bauges sharing **€185,000** | Strong, and structurally distinctive: OFB publishes a **reusable methodology toolkit** plus a document titled *"L'Atlas de la biodiversité communale : et après ?"* on what happens after the grant. It is the only funder in the corpus that plans for its own grants ending |
| **5. Corporate / philanthropic** | Amounts almost never published | Bush Blitz (Australian Government + **BHP** + Earthwatch + Parks Australia); Reflora Virtual Herbarium (CNPq, CAPES, FNDCT, **seven state research foundations**, plus **Natura** and **Vale S.A.**); Enslaved.org (**Mellon** + **NEH**); African Lion Database (**Lion Recovery Fund** + **National Geographic**); Birdata (**Tony & Lisette Lewis Foundation**, WildlifeLink); Dilmah Conservation (Dilmah Tea, Sri Lanka); UPL Sarus Conservation Project (India); Libro Rojo de la Fauna Venezolana (Provita + **Fundación Empresas Polar**); Reef Life Survey (**Ian Potter Foundation**) | Variable. Reflora's blended federal + 7-state + 2-corporate stack is the most robust example found. Libro Rojo shows the failure mode: four editions 1995→2015, then a stall |
| **6. Earned revenue / service income** | Only two prices found in the entire corpus: **AHIMS** (NSW) extensive heritage search **A$60** (basic search free; fee waivers for Aboriginal individuals and organisations); **Symbiota** hosting is free for pre-2021 NSF-funded portals via iDigBio (up to **30 TB** image storage), otherwise fee-based contracts through a **KU Service Center** whose rates are quote-only and unpublished | **Essex Field Club** runs a commercial "Datasearch" desk-study service off the same **7,593,231-record / 22,218-species** database that powers its free public atlas — the clearest self-funding model in the corpus; **Naturbasen** (Denmark) is free to its 119,474 members and sustained by **cooperation agreements with municipalities and national parks** rather than data sales; **Ainmean-Àite na h-Alba** sells corporate memberships and co-produced maps; **ArchSite** (NZ) charges for some requests; **British History Online** runs a small subscription tier over mostly free content; **Keeping Culture KMS** is an annual-licence cloud product | Good where the buyer is a regulated industry (ecological consultants needing desk studies) or a public body. Essex Field Club and Naturbasen are both long-running |
| **7. Zero budget** | n/a | NatureSpot on commodity hosting (**Clook**); Biodiversity of Sri Lanka on **Blogspot**; Catálogo de la Flora de Cuba on **WordPress.com**; SEOSAW and Biodiversidata on **GitHub**; the entire Chinese solo-taxon cluster (shellsmap, GobiologyCN, china-odonata.top, ants-china.com, mycopedia.top, liudayadan.com), discoverable only through a community-maintained iNaturalist link list, not through search engines | The corpus's most surprising result — see immediately below |

### 2.2.2 The central finding on money

**Money is not what determines survival in this corpus, and the evidence is unusually clean.**

Set the two extremes side by side, both verified 25 August 2026:

- **INBio (Costa Rica)** — founded 1989, a large parastatal NGO, 3.7 million specimens, Latin America's largest GBIF contributor by 2015. Wound down 2014–2016 when a new government withdrew support, after failed attempts to build an endowment. **`inbio.ac.cr`, `atta.inbio.ac.cr` and the successor `crbio.cr` all fail DNS resolution today.** The mitigation failed as well as the original.
- **Sinh vật rừng Việt Nam (`vncreatures.net`)** — one person, contactable only at a personal Gmail address, no institution, no visible funding. Covers **>2,000 species** across insects, animals, plants and fungi, *plus* regulatory documents *plus* national-park pages — the same species + legal + place combination Sathyamangalam proposes. **Three new conifer entries dated 17 August 2026.**

The same contrast repeats within countries. Bhutan's **US Fulbright-supported** specimen portal is dormant at ©2017 while Bhutan's **six-institution** portal thrives. Hawai'i's dedicated **university-run** biodiversity atlas (`hbmp.hawaii.edu`) fails DNS resolution while its **Indigenous-authority-run** cultural and land databases (Papakilo, Kīpuka, Ulukau) all serve. Task 1's own conclusion on this is exact: *"Institutional ownership, not subject matter, determines survival."*

What money buys is **speed and scale**, not **duration**. What buys duration is covered in §2.5.

### 2.2.3 The specific Indian funding picture

The corpus records almost nothing about how Indian projects in this space are funded. Two mechanisms are visible:

- **Corporate sponsorship of single-species work** — the UPL Sarus Conservation Project is explicitly corporate-funded.
- **Institutional absorption** — Bird Count India operates as a Nature Conservation Foundation-hosted consortium with BNHS, WWF-India and **state forest departments** as partners; the Kerala Bird Atlas ran with the **Kerala Forest Department** and Kerala Agricultural University, and Atlas 2.0 was inaugurated on **15 July 2026 by the Kerala Forest Minister**. Goa's atlas involved the **Goa Forest Department**.

There is **no Indian equivalent of the OFB ABC programme** — no national agency publishing a methodology toolkit plus grants so that many localities each build a comparable atlas. This is a genuine structural gap, and it is the single most consequential funding fact for Sathyamangalam: the money route that most naturally fits a district-scale atlas does not exist in India, so the project must either be absorbed into an existing consortium (the Kerala/Goa route), earn its own income (the Essex route), or run on the zero-budget solo route (the vncreatures route).

---

## 2.3 Team structure and size

### 2.3.1 Every project in the corpus that actually publishes a headcount

This is the complete disclosed set. Its shortness is the point.

| Project | Disclosed team | What it produced |
|---|---|---|
| **EBBA2** (Europe) | ~120,000 contributors | 596 species, transnational change maps vs EBBA1 |
| **Ontario Reptile & Amphibian Atlas** | 12,000+ volunteers over the collection decade | Book, 443 pp., 70+ maps, 300 photographs (2023) |
| **Swiss Breeding Bird Atlas 2013–2016** | >2,000 field volunteers; **~3.9 working-years** of fieldwork, 46,438 km travelled | ~200 species with altitudinal range; **corrigendum published June 2020** |
| **Virginia Breeding Bird Atlas 2** | **~1,500 trained volunteers** plus staff | **6M+ field observations**, 200+ species, 2016–2020 fieldwork |
| **NZ Bird Atlas 2019–2024** | Volunteer society (Birds NZ), coordination led by Dan Burgin | **441,000 checklists, 309 species, 145,000 hours, 3,145 of 3,232 grid squares = 97.3% coverage** |
| **NZPCN** | >1,000 members worldwide | Species database + threat classifications + hosts the NZ Botanical Society journal |
| **Naturbasen** (Denmark) | 119,474 members | ~40,000 Danish species |
| **SERNEC** | 230 participating herbaria | Southeastern US specimen network |
| **Hyderabad Bird Atlas** | **200–220 volunteers per season** | 45 grids of 6.6 × 6.6 km; **248 unique species after four seasons**; Season 4 (Jul 2026): 200+ volunteers, 158 species, 75,000+ birds |
| **Land Conflict Watch** (India) | **9 core staff + 35+ researchers** | Public conflicts database with four-step, two-source verification |
| **SBDI** (Sweden) | 11 institutions + an executive office | 259 datasets, >177M records, 340,844 species |
| **TBIA** (Taiwan) | 19+ named member institutions | Federated national occurrence aggregation |
| **NDFF Verspreidingsatlas** (NL) | 7 species organisations | >200M observations, ~20M added annually, maps rebuilt nightly |
| **Bhutan Biodiversity Portal** | 6-institution consortium + Strand Life Sciences | 8,160 species, 117,000 observations, 314 documents, 45 maps |
| **Flora-On** (Portugal) | **~10-person active core**, credited weekly by name | Week of 17–23 Aug 2026 alone: **2,060 new records, 299 new grid squares, 220 species** |
| **Co's Digital Flora of the Philippines** | **3 named editors** (Pelser, Barcelona, Nickrent) | **~10,200 species, >146,000 photos covering ~59% of the flora**; adopted as a World Flora Online regional flora |
| **Caribbean Herpetology** | **4 named academics** (Hedges + 3 associate editors) | **~750 species accounts, ~2,000 images**, 100 issues over 16 continuous years, ISSN 2333-2468 |
| **Flora of Zimbabwe/Zambia/Mozambique/Botswana** | **4 named people** (Hyde, Wursten, Ballings, Coates Palgrave) | Four-country flora network, 2002–2026 |
| **Marine Biodiversity Portal of Bangladesh** | **One university lab** (Prof. Kazi Ahsan Habib, SAU Dhaka) | **>2,500 marine species across 12 groups**, 7 habitat classes, DNA barcode library |
| **BioWeb Ecuador** | **2–3 named authors per sub-portal** | FaunaWeb / FloraWeb / FungiWeb under one brand |
| **Biodiversidata** (Uruguay) | **2 named** (Grattarola, Polastri) | Uruguay's first consolidated national occurrence dataset |
| **Janzen & Hallwachs ACG inventory** | **2 people** + ACG parataxonomists | Decades of caterpillar–host plant–parasitoid rearing records |
| **Keystone Foundation Archives** (Nilgiris) | Small NGO, 1993– | **464 images, 53 audio, 22 videos, 503 text documents, 13 other items** |

### 2.3.2 The universal shape: a small editorial core plus a wide periphery

Every multi-contributor project in the corpus that works has the same structure — **a very small number of people who decide what is true, and a much larger number who supply material.** The corpus makes the ratio visible in several places:

- **NatureMapr / Canberra Nature Map**: 16,082 members, but the top moderator (MichaelMulvaney) alone has **60,600 verifications**.
- **Massachusetts Butterfly Club**: 200,000+ sighting records with **one named Records Compiler** (Mark Fairbrother) as the quality gate.
- **Flora-On**: ~10 named contributors producing 1,500–2,000 records per week.
- **BAMONA**: 93,357 members, 1,004,053 verified sightings, gated by a network of **regional coordinators** who verify before a record enters the database.
- **Euro+Med PlantBase**: a distributed **regional-editor** network where each area's record is attributable to a named editor.
- **Bird Count India / Kerala Bird Atlas**: hundreds of volunteers, **district coordinators in all 14 districts**, two overall leads (P. O. Nameer, Ratish R. L.).

**Implication for a 2–3 person team:** the corpus says the editorial core *is* the project, and that a core of 1–4 is not unusual or crippling. What the core cannot do is also be the entire capture layer. Every project above that reached scale either (a) recruited a periphery, or (b) delegated capture to an existing platform (§2.4.1).

### 2.3.3 Named-person dependency: the corpus's clearest structural risk

The solo/near-solo tier produces remarkable output and then stops when one person stops. The corpus contains both halves of this.

**Still running (as of 25 Aug 2026):**

| Project | Person | Span | Scale |
|---|---|---|---|
| atlas-roslin.pl | Marek Snowarski | **2002–2026 (24 yrs)** | National plant distribution atlas on the ATPOL grid |
| Flora of Zimbabwe cluster | Hyde, Wursten, Ballings, Coates Palgrave | **2002–2026 (24 yrs)** | 4-country flora; software last modified 11 Jun 2025 |
| speciesLink / CRIA | Civil-society org, small technical team | **2002–2026 (24 yrs)** | Millions of specimen records + name/coordinate validation tooling |
| Flora Digital de Portugal | UTAD Botanical Garden | **2004–2026 (22 yrs)** | Page shows "last update **25/08/2026**" |
| Flowers of India | "Tabish" + Thingnam Girija | **2005–2026 (21 yrs)** | Multilingual vernacular names, ~40+ major contributors |
| Minnesota Wildflowers | Katy Chayka | **2006–2026 (20 yrs)** | 1,800+ of the state's 2,100+ species, 18,000+ photos, **publishes an annual report** |
| Plantarium | Dmitry Oreshkin | **2007–2026 (19 yrs)** | Continent-scale, multi-kingdom, dual administrative/physiographic geography |
| WikiAves | Community, no institutional host | **2008–2026 (18 yrs)** | **6,546,363 photo and sound records, 57,304 observers, 1,975 species** |
| Wildflowers of Ireland | Zoë Devlin | **2008–2026 (18 yrs)** | Botany + folklore + herbal use + Irish-language names, 3 books |
| Caribbean Herpetology | 4 academics | **2010–2026 (16 yrs)** | Journal fused to database, Issue 100 in June 2026 |
| EcoRegistros | Jorge La Grotteria | **2011–2026 (15 yrs)** | **>2.3 million records**; executed a full **AviList taxonomic-backbone swap in 2026** |
| Co's Digital Flora | 3 editors | **2011–2026 (15 yrs)** | 10,200 species, 146,000 photos |
| Māori GIS / Matawhenua | Duane Wilkins | active | Docs page "Last reviewed: **23 August 2026**" — among the freshest items in the entire corpus |

**Stopped:**

| Project | Person | Froze at | What it had reached |
|---|---|---|---|
| Illinois Wildflowers | Dr. John Hilty | **© 2002–2020** | Habitat-organised species pages *plus* literature-derived databases of plant-feeding insects, flower-visiting insects and vertebrate–plant interactions |
| BONAP / North American Plant Atlas | John T. Kartesz (widely reported, unverified from the site) | **2014 edition, dated 11/17/2014** | Continental county-level plant atlas, still widely cited as current |
| E-Flora BC / E-Fauna BC | Brian Klinkenberg (coordinator) | Blog last post **16 Sep 2021**; a 2020 post documents recovering from a mapping outage | Most ambitious volunteer-edited province-wide two-kingdom atlas in North America |
| FaunaParaguay | Single researcher | **Expired TLS certificate** at fetch, 25 Aug 2026 | Species accounts + a historical literature and bibliography archive |
| PNGplants | Barry Conn | Live but undated | One person is the botanical web infrastructure for an entire country; its sibling `pngtrees.org` has already lost its domain |

The pattern is that solo projects do not decay gradually — they run at full quality for one to two decades and then stop cleanly, leaving a frozen but often still-valuable site. Task 1's North America pass reads this precisely: **"Blog silence is the leading indicator of atlas death."**

### 2.3.4 Governance splits that the corpus shows working

Three separations recur and all three are load-bearing:

1. **Community owns the data, an institution hosts the technology.** Gwich'in Social and Cultural Institute + Carleton's GCRC; Kitikmeot Heritage Society + GCRC. This is the model for Indigenous/Adivasi knowledge layers.
2. **The state initiates, a contractor operates.** Ireland's National Biodiversity Data Centre is an initiative of the Heritage Council **operated under a service-level agreement by Compass Informatics**, a private firm — with 9,261,812 records, 19,724 species and a displayed "Last Updated: 21 August 2026".
3. **A learned society runs the register, Crown agencies partner.** ArchSite is run by the **New Zealand Archaeological Association** in formal partnership with Heritage New Zealand and DOC.

And one that the corpus shows failing: **a portal built by an external academic dev team on donor funding.** Brazil's ICMBio Portal da Biodiversidade was built by MMA + ICMBio with technical support from Germany's **GIZ** and developers from **Escola Politécnica da USP**, launched 2020, and was deactivated within about five years — with the stated cause being **technological, not financial**. The developers moved on and no one could maintain it.

---

## 2.4 Data pipeline architecture

### 2.4.1 The four build strategies, with instance counts

| Strategy | What it means | Instances in Task 1 | Real cost | Real limit |
|---|---|---|---|---|
| **(a) Build nothing** — ride a global platform, keep only the synthesis layer | Capture happens on iNaturalist/eBird; your site is interpretation, checklists and narrative | Ohio Odonata Society (survey runs on iNaturalist, society site is "the interpretive layer"); Atlas of Life in the Coastal Wilderness; Moths of Krishnagiri (iNat-hosted, adjoins the STR landscape); Biodiversity of Western Ghats (iNat project); Israel Breeding Bird Atlas and **NZ Bird Atlas** (both on eBird); Kerala, Hyderabad, Coimbatore, Pune, Ahmedabad, Bhagalpur atlases (eBird, except Pune on **Epicollect5**); "Flora of Russia" on iNaturalist + a published data paper; Ontario Reptile & Amphibian Atlas handing over to a "Herps of Ontario" iNat project | Near zero | You do not control the data model, the taxonomy, or the survival of the platform. NatureWatch NZ → iNaturalist NZ is described in Task 1 as "a national project trading independence for platform sustainability" |
| **(b) Deploy an existing stack** | Install open-source atlas software someone else maintains | See the stack table below | Server + one competent admin | You inherit the stack's data model, including what it cannot express |
| **(c) Build custom** | Write your own | Consortium of Pacific Northwest Herbaria (own stack, not Symbiota: **3,328,306 records, 1,800,701 images, 55 herbaria**); **speciesLink/CRIA**, which self-describes as *proprietary technology built for local needs* rather than adopting foreign platforms — and has survived 24 years; BioGIS's arbitrary-polygon diversity analysis; Papakilo (OHA + contractor DL Consulting) | High and permanent | The ICMBio failure mode |
| **(d) Publish first, database second** | Every record is a citable, dated, attributable publication that also updates the database | **Singapore Biodiversity Records** (LKCNHM, ISSN 2345-7597) — individual sightings published as numbered notes, which enabled the PNAS analysis of two centuries of Singapore biodiversity; **Caribbean Herpetology** (ISSN 2333-2468) — peer-reviewed journal fused to the species database; **Records of the Hawaii Biological Survey**, annual since 1994; Mysuru City Bird Atlas → **Dryad DOI**; Hornbill Watch → data paper in *Indian BIRDS* 14(3); FloraBase hosting the journal *Nuytsia* inside the atlas | Editorial time | Slow. But it is the only strategy in the corpus that makes every single datum permanently citable and disputable |

### 2.4.2 The reusable stacks, ranked by relevance to a single-reserve atlas

| Stack | Deployments named in Task 1 | Built for | Notes |
|---|---|---|---|
| **GeoNature / GeoNature-atlas** | **Biodiv'Écrins** (Parc national des Écrins — the closest structural analogue in the corpus to a single-reserve atlas, latest observations **24 August 2026**) | Protected-area scale, exactly | Developed by a consortium of French national parks (Écrins, Cévennes, Vanoise, Mercantour) + OFB; free and open source; public roadmap; **"BAM" launched July 2026**; national user meeting **5–6 November 2026** in Paris. Maintenance is anchored in Écrins' information-systems team. Modules cover field capture (Occtax), taxonomy (TaxHub), public atlas (GeoNature-atlas), citizen science (GeoNature-citizen). *Number of deploying organisations and hosting cost: **unverified** — neither geonature.fr nor third-party guides publish either* |
| **Biodiversity Informatics Platform** (the India Biodiversity Portal codebase family) | **Bhutan Biodiversity Portal v5.0.5** (on GitHub, Mapbox mapping, Play Store app); **Assam Biodiversity Portal**; **The Western Ghats Portal** — the last of which is the landscape Sathyamangalam sits inside | Multi-domain national/regional portals | Proves a *state- or landscape-scoped* portal can be stood up on the same codebase rather than built from scratch. Both Indian sub-portals returned HTTP 403 to automated fetch, so their **observation counts and recency are unverified** — a live-but-empty sub-portal would be an important negative finding and should be checked manually |
| **Symbiota** | SEINet (**24 million records, 456 collections**), SERNEC (230 herbaria), CCH2, Consortium of Midwest/Northeastern/Mid-Atlantic Herbaria, TORCH, Intermountain Biota, SoRo, Four Corners Plants, **North American Network of Small Herbaria** (explicitly for institutions too small to run a portal), Indian River Lagoon Species Inventory (**11,150 species reports, 633,656 occurrence records for one estuary**), BioGator, Kansas Biodiversity, Panama Biota, **Red de Herbarios Mexicanos**, **DEMCA** (ethnobiology: specimens linked to Indigenous-language plant names and uses), BiodiverseNB, Bhutan specimen portal (dormant) | Natural history collections | Free hosting for pre-2021 NSF-funded portals via iDigBio (up to **30 TB** images); otherwise **fee-based contracts through a KU Service Center**, rates quote-only. DEMCA proves the schema bends to hold ethnobiological and linguistic data alongside vouchers |
| **Living Atlases** (ALA lineage) | **SBDI** Sweden, **Biodiversitäts-Atlas Österreich**, **Croatia BioAtlas**, **SiBBr** Brazil (36,509,671 records, 196 institutions) | National-scale occurrence infrastructure | Heavy for a single reserve. Austria's instance is notable for **pinning GBIF Backbone Taxonomy v2021** — a versioned backbone, which is how conflicting names become reproducible |
| **Nunaliit** (Carleton GCRC) | Gwich'in Atlas, Siku Atlas, Kitikmeot Place Name Atlas, Inuktut Lexicon, Lake Huron Treaty Atlas / Residential Schools Land Memory Atlas, **Pa Ipai Astronomical Atlas (Mexico)** | Community-editable multimedia place atlases with document relations, flexible schemas and offline tablet sync | Purpose-built for precisely the Sathyamangalam problem. Two cautions: **every Nunaliit atlas in the corpus is JavaScript-only** (see §2.4.4), and the framework's **latest commit date is unverified** — 2,640 commits, no timestamp obtained. The Pa Ipai deployment came out of a short **GCRC training workshop in Mexico in 2018**, which is the most encouraging capacity-transfer precedent in the corpus for a Global South team |
| **Biolovision / `ornitho.*`** | ~20 national portals (ornitho.de, .it, .cat, .pl, faune-france) | Bird occurrence with expert validation | One codebase, per-country governance. Small commercial operator |
| **PlantAtlas.org** | Atlas of Florida Plants (4,920 species, 243,240 specimens), New York Flora Atlas (**"Data last modified: 7/5/2026"**), Alabama Plant Atlas | State floras | New York's displayed data-last-modified date is the cheapest high-value feature in the corpus |
| **Boring general-purpose CMS** | **Terras Indígenas no Brasil on Drupal 9** (838 territories, seven distinct legal phases); NC Biodiversity Project on Drupal; FloraWeb on GetSimple CMS | Anything | Task 1's verdict: *"Drupal is a realistic, boring, maintainable stack."* Terras Indígenas expresses a genuinely sophisticated legal data model on entirely unremarkable software |

### 2.4.3 The ingest → validation → publication chain, as actually implemented

**Capture tiers.** Every credible project separates "submitted" from "accepted", and several display both:

- **FrogID** (Australia) shows **930,680 calls submitted** and **1,507,148 verified frogs** as two separate public numbers, and openly notes a validation backlog since October 2025.
- **BAMONA** gates on expert verification *before* a record enters the database.
- **NZ Freshwater Fish Database** applies **administrative review before public release**, with a named and unusually broad contributor list (NIWA, DOC, regional councils, consultants, universities, schools, **iwi groups**, public).
- **ArtenFinder Rheinland-Pfalz** keeps a recorder's data **private until they explicitly release it** — the model for sensitive-species and tribal-knowledge governance.
- **SILENE** (PACA, France) runs **two interfaces over one database**, public and professional, at different spatial precision.
- **Portable Antiquities Scheme** varies **findspot precision by user account** across 1,531,145 objects.

**Place validation as a named step.** Sweden's **Artportalen** (>100 million observations, updated daily) states that data is quality-assured through expert verification of both species identification **and place**. Denmark's **DOFbasen** holds 39,485,418 records across **20,782 named localities** that observers must select from — Task 1 identifies this controlled locality dictionary as the reason the dataset is spatially usable at all.

**Controlled vocabularies are frequently the actual product.** Seamap Australia's contribution is a shared benthic habitat classification that lets incompatible surveys be compared; Canmore publishes monument-type and period thesauri; Brazil's **Mapa de Conflitos** is queryable only because of its **50+ affected-population categories, 40+ activity types, 16 health-harm categories and 20+ damage categories**; Hawai'i's **Kīpuka** indexes on the Indigenous spatial hierarchy **mokupuni → moku → ahupuaʻa → ʻili → kuleana** rather than colonial administrative units.

**Taxonomic backbone as a versioned dependency.** Austria pins GBIF Backbone v2021. **EcoRegistros — a one-person project — executed a full migration to the AviList global standard in 2026.** Canada's **VASCAN** provides accepted names, synonyms and per-province status. Taiwan's **TaiCOL** stores national red-list level and IUCN category as **separate simultaneously-displayed fields**. New Zealand's **NZOR** exists specifically to reconcile conflicting names from multiple providers, and publishes per-provider attribution and data-quality pages — though the site itself now reads as frozen.

**The cheap publishing endpoint.** Multiple projects use a **GBIF IPT instance** as their machine-readable output: TaiBIF, JBIF, Croatia's HR IPT, `ipt.gbif.ch`, JBRJ's IPT (through which Flora e Funga do Brasil's data must be reached, since the main site is robots-disallowed), and Norway's Artsdatabanken, whose Species Observation Service dataset was at **version 1.364, published 24 August 2026**, holding 38,270,228 occurrence records with daily updates. This is the lowest-effort route from "a local database" to "harvestable, citable, and mirrored somewhere that outlives you."

### 2.4.4 The JavaScript-only anti-pattern, quantified

The corpus contains an unusually large number of authoritative resources that are **invisible to non-JavaScript clients** — which means invisible to crawlers, archivers, citation-checkers and automated verification:

Michigan Flora Online · every Nunaliit atlas (Gwich'in, Siku, Kitikmeot, Inuktut, Lake Huron Treaty) · Australian Antarctic Data Centre · SCAR Composite Gazetteer (`placenames.aq`) · AUSTLANG · Natural Values Atlas app · Biota of New Zealand · Trove · EJAtlas · RUNAP (Colombia) · Catálogo de la Biodiversidad de Colombia · SlaveVoyages · SIB Parques Nacionales (Argentina) · BioWeb Ecuador statistics page · Layers of London · Endemia.nc counters · SiB Colombia (which additionally **returned corrupted placeholder figures** — repeated "100,000" across unrelated counters).

Task 1's own verdict, reached independently in two regional passes: **do not build a JavaScript-only atlas.** For a project whose entire proposition is citability and provenance, this is not a cosmetic choice.

---

## 2.5 Long-term maintainership

### 2.5.1 The single most important structural finding in the corpus

> **The data layer and the presentation layer die at different rates, and the presentation layer always dies first. They need separate survival plans.**

The corpus proves this five separate times:

| Case | What survived | What died |
|---|---|---|
| **INBio → CRBio (Costa Rica)** | 3.7M specimens *and the complete Atta database records* transferred to the **Museo Nacional de Costa Rica**; Mollusca (200,000+ specimens) to **UCR**; Nematoda (18,000) to **UNA**; GBIF representation to **CONAGEBIO** from April 2015 | **Both** the original portal and its successor CRBio. Species pages, images, interpretive content and every URL cited in the literature are gone from the live web |
| **Boston Harbor Islands ATBI** | The peer-reviewed synthesis in *Northeastern Naturalist* (2018) | The custom MantisWeb database host (`bhi.oeb.harvard.edu`) — DNS failure |
| **GB1900** | The transcription dataset, released openly via the Vision of Britain / Portsmouth team and open-data mirrors | The project's web presence and brand. **The domain now serves an online-gambling site** while still carrying a canonical URL referencing GB1900 |
| **ScotlandsPlaces** | The underlying material at each of the three partners (OS Name Books, Tax Rolls, 420,000+ NLS map images) | **The cross-institutional integration layer, closed 24 June 2025** — which was the entire point of the project |
| **Bayernflora** | The BIB structured database tools (profiles, maps, checklists, Red Lists) | **The wiki** — the narrative, historical-literature and discussion layer — lost to a November 2023 cyberattack and still unrestored ~3 years later |

The Bayernflora case is the sharpest warning for Sathyamangalam specifically, because the layer that was lost is exactly the layer the project is most distinctive for: **narrative, historical literature, and discussion.** Structured occurrence tables are backed up as a matter of routine; historical document mining and interpretive prose are not.

### 2.5.2 Custodial handover — the full observed range

**Executed well:**

| Case | Mechanism |
|---|---|
| **Oriental Bird Images → Macaulay Library** | Closure announced **17 March 2021**; new home announced **22 October 2021**. Two decades of Asian photographic archive transferred to a larger institution with a stable identifier scheme. Task 1 calls it "the best-executed shutdown found" |
| **Chile SIMBIO → SBAP** | From **2 February 2026** under Law 21.600, SBAP becomes the official data body; **SIMBIO does not shut down — it degrades into an interoperability and metadata service** and gradually synchronises with SBAP sources |
| **ICMBio Portal da Biodiversidade → SiBBr** | Portal deactivated (stated cause: technological), but replaced with a **permanent explanatory notice plus a redirect to a living alternative** rather than a dead link |
| **HGIS de las Indias** | Built at Univ. Graz on Austrian FWF funding 2015–2019; **now hosted by Yale University's Economic Growth Center** — institutional rehoming after grant end |
| **Ara Irititja** | Governance transferred to the **APY Executive Board in 2020**, 26 years after its 1994 founding by Anangu elders |
| **African Lion Database** | Migrated to the IUCN SSC Cat Specialist Group's newly established (2025) Centre for Species Survival – Cats |
| **Ontario Reptile & Amphibian Atlas** | Collection ended 2019, book published 2023, ongoing observations **explicitly redirected to a "Herps of Ontario" iNaturalist project** — a graceful handover of the capture layer to a global platform |
| **Living Archive of Aboriginal Languages** | ~4,000 books in 50 languages from 40 communities **moved to Territory Stories at Library & Archives NT** |
| **Biodiversidad Virtual (Spain)** | A 30-year project (Insectarium Virtual, 1995 →) that **surrendered its platform to observation.org and kept its community**, becoming a partner rather than an operator |
| **Gatehouse Gazetteer** | A full copy **deposited at the Internet Archive** as an explicit succession plan, with a Wikidata item |

**Executed badly, or not at all:**

- **PIER (Pacific Island Ecosystems at Risk)** carries the corpus's most explicit succession-failure notice: last updates **15 May 2013**, posted 1 June 2013, and the US Forest Service is *"seeking an individual or organization that is interested in maintaining and possibly updating the PIER web site."* Nobody came.
- **Arctic Bay Atlas** — community-produced Inuit content; **the domain now serves a commercial travel guide**. Task 1: *"Worse than a 404."*
- **REBIOMA (Madagascar)** — `data.rebioma.net` now 302-redirects to a **domain-resale broker**.
- **RSCN Jordan BIMS** — the parent organisation's page still describes the National Biodiversity Database as active and links to a URL returning **404**.

### 2.5.3 Honest-decay labelling: a cheap, high-trust practice with many precedents

A surprising number of the healthiest projects in the corpus publish their own limitations. This is the least expensive credibility mechanism found anywhere in Task 1.

| Project | What it declares |
|---|---|
| **Tasmanian Threatened Species Link** | *"The fauna data on the Threatened Species Link is currently not being maintained. As such, some of the information may be out of date."* — **per-domain freshness declaration** |
| **Flora of Pakistan (eFloras)** | *"This is a legacy site which is now out of date"*, redirecting users to tropicos.org |
| **NY Breeding Bird Atlas** | *"Data available on this website were finalized in March 2007."* Task 1: a frozen dataset **with a stated freeze date** is healthy; contrast the undated Digital Atlas of Idaho |
| **Biodiv'Écrins** | Data come from *"different scientific protocols"* and are *"neither exhaustive nor complete distribution data"* |
| **BioNet Atlas (NSW)** | Coverage is *"extensive, but nevertheless patchy"* and will not show full species distributions |
| **AHIMS (NSW)** | Site-card formats and capture methods changed over 50 years, so **detail varies across records** |
| **WildNet (Qld)** | Data spans from the 1700s; the site itself advises filtering to **records since 1980** |
| **PastMap (Scotland)** | *"Should not be relied upon to confirm the presence of current designations"* — separating the **discovery map** from the **authoritative legal register** |
| **SCAR Composite Gazetteer** | Compiled *"purely for the convenience of the scientific community"* and **"has no legal authority or standing"** |
| **Native Land Digital** | Boundaries are **contested, overlapping and not authoritative** |
| **BONAP** | *"Not every datum is accurate"*, with an active solicitation for corrections |
| **HKBIH (Hong Kong)** | Curation is *"an evolving process where information is continuously amassed, updated and fine-tuned"* |
| **Digital Atlas of the Virginia Flora** | Maintains a published **"Excluded Taxa"** document — a reasoned register of rejected records kept visible instead of silently deleted |
| **New Zealand Gazetteer** | States plainly that *"not all Māori names have undergone spelling verification"* |

### 2.5.4 Transparency-as-maintenance

Projects that publish throughput metrics are visibly better maintained, and the metric itself is what makes decay detectable:

- **Flora-On** publishes **weekly** record counts with **named contributor credit**.
- **Wyoming Natural Diversity Database** publishes quarterly throughput — which is the only reason it is visible that **"0" new species records were added in the most recent quarter** on an otherwise live site timestamped 2026-07-30.
- **Hyderabad Bird Atlas** publishes per-season volunteer counts, species counts and individual-bird counts across four seasons — the best public audit trail of effort of any Indian atlas.
- **Toponimia de Galicia** leads with its own coverage gap: **>478,000 registered toponyms covering 35% of the territory**.
- **Minnesota Wildflowers** — a near-solo project — publishes an **annual report**.
- **Endemia.nc** publishes a **per-taxon public edit log** as its primary evidence of liveness.
- **AmphibiaChina** publishes an **annual taxonomic-change log** — 759 species as of 23 July 2026 — turning taxonomic disagreement into browsable history rather than a silent overwrite.
- **NZ Bird Atlas** closed by publishing **97.3% grid coverage** and 145,000 volunteer hours.

### 2.5.5 Decay signals, ranked by diagnostic value

From the corpus, in rough order of how conclusive each is:

1. **DNS failure** — conclusive. BISON, NBII, `papuaweb.org`, `pngtrees.org`, `nwatlas.org.au`, `hbmp.hawaii.edu`, `inbio.ac.cr`/`crbio.cr`, `bhi.oeb.harvard.edu`, `sibm.invemar.org.co`.
2. **Domain repurposed** — worse than death. GB1900 (gambling site), Arctic Bay Atlas (travel guide), REBIOMA (domain broker).
3. **Database resource exhaustion** — UkrBIN returning **"Error 1040: Too many connections"**. Task 1 calls this "the most common realistic death of a small atlas, and it is invisible from the outside."
4. **HTTP 500 at the project's own advertised URL** — Deutschlandflora at `dflora.netphyd.de/main`, the exact address NetPhyD itself publishes, while the association is demonstrably active.
5. **Expired or mismatched TLS certificate** — BIOTIK (a *Western Ghats* project, directly in Sathyamangalam's landscape), NSII China, Catalogue of Life China, `floraofalabama.org`, FaunaParaguay, Flora Argentina, Biblioteca Digital de la Medicina Tradicional Mexicana, `caterpillars.org`, `natureiraq.org`.
6. **Stale copyright with live infrastructure** — the commonest and least-reported failure: Illinois Wildflowers ©2020, BONAP 2014, Victorian Places ©2015, LandScope ©2022, Alaska Native Place Names 2016, Catálogo de Plantas de Colombia "last update 09/02/2023", Australian Faunal Directory 11 Apr 2024, ZSI digital archive ©2015, India Observatory/IBIS ©2006–2019.
7. **Blog or news silence while the data site still serves** — E-Flora BC, last post 16 Sep 2021.
8. **A counter rendering zero** — PIPAP's dashboard reads **"0 Protected Areas (0 Terrestrial, 0 Marine)"**. Task 1's lesson is exact and matters directly for a disputed-figures feature: **"a broken counter reads as an authoritative claim."**

### 2.5.6 Aggregator lag — why directories cannot be trusted as liveness evidence

Every aggregator in the corpus carries dead entries:

- The **ASEAN Biodiversity Knowledge Platform** still indexes **TH-BIF** as Thailand's national facility while `thbif.in.th` fails DNS resolution.
- **RSCN Jordan's** own page still links a BIMS URL returning 404.
- **eAtlas** still names `nwatlas.org.au` as a live partner site; the domain is gone.
- The **Citizen Science India registry** still lists **MigrantWatch**, believed superseded by eBird India but unverified.
- **NatureMapr's** own announcement of its restructure (`/announcements/586`) returns "This record no longer exists" while still being indexed by search engines.

---

## 2.6 Lifecycle: from launch to maturity or abandonment

### 2.6.1 The seven observed stages

**Stage 0 — Precursor (often decades, always print).**
Almost every mature atlas in the corpus has a paper ancestor, and the digitisation of that ancestor is a distinct project in its own right. Poland's `atlas-roslin.pl` derives its maps from the **published Zając & Zając ATPOL atlas (2001) and its 2019 supplement**, and says so. Kenya Bird Map rests on the printed **Bird Atlas of Kenya (1970–84)**, with a **dedicated GitHub project re-digitising its hand-drawn range maps**. Australia's Birdata retro-digitised **~370,000 paper-form surveys / 7.5 million sightings** from Atlas 1 (1977–81) and Atlas 2 (1998–2002). VicFlora absorbed the printed *Flora of Victoria* (1994/1996/1999) and **eight print editions of the Census of Vascular Plants (1984–2007)**. For Sathyamangalam the equivalent Stage 0 assets are already identified in Task 1: the **ZSI digital archive** (200,000+ pages, 52 *Fauna of India* volumes, 44 Conservation Area Series volumes — several of them single-reserve monographs), the **Imperial Gazetteer of India** via DSAL, and the **Erode / Coimbatore / Salem district gazetteers**, which Task 1 confirms exist as a series with **no state-run searchable digital portal**.

**Stage 1 — Launch (year 0–2): the highest-mortality window for volunteer wikis.**
The corpus's most instructive single failure is **Biodiversity of India / Project Brahma**: an ambitious, well-designed, semantically-structured Semantic MediaWiki that froze at **664 species and 65 "Bio Notes"**, footer reading *"This page was last modified on 8 October 2011"* — and has been serving that stale page **for fifteen years**, still ranking in search results. Task 1's diagnosis is not technological: *"a volunteer wiki with no institutional host and no ongoing data inflow stops."*

**Stage 2 — Build-out (year 2–6): where grant-funded projects die.**
This is the tightest cluster in the entire corpus. Every one of these ended within about a year of its funding ending:

| Project | Funding window | Outcome |
|---|---|---|
| AfricanBioServices | EU H2020, ~2015–2020 | Project period ended; website persists, static |
| Caribbean Marine Atlas (CMA2) | CLME+ Project, 2015–2020 | Loads; no post-2020 content |
| Biological Specimen Collections of Bhutan | US Fulbright | Dormant, ©2017 |
| ICMBio Portal da Biodiversidade | Launched 2020, GIZ + USP dev team | Deactivated ~5 years later, "technological" cause |
| New Forest Knowledge | Heritage-Lottery era | DNS does not resolve |
| Pladias | Czech Science Foundation 2014–2018 | **Survived** — "continuously updated also after the end of the Pladias project" |
| HGIS de las Indias | Austrian FWF 2015–2019 | **Survived by rehoming** to Yale |

The two survivors are the informative cases: both survived by acquiring a host institution that had an independent reason to keep them.

**Stage 3 — Maturity or plateau (year 6–20): institutionalise, or plateau at a sustainable level.**

*Institutionalised* — the project acquires a legal or governmental anchor:
Hawaii Biological Survey (legislature, 1992), Historic Place Names of Wales (statute, 2016), NSW Geographical Names Board (statute, 1966), **Kerala Bird Atlas** (Atlas 2.0 inaugurated by the Kerala Forest Minister on 15 July 2026; peer-reviewed in *Current Science* and *Forktail*; six published district atlases), Goa (state-published, India's second state bird atlas), Ireland's NBDC (Heritage Council initiative under an SLA).

*Plateaued but healthy* — the project finds a level it can sustain indefinitely:
Flowers of India, Plantarium, atlas-roslin.pl, Wildflowers of Ireland, Flora Digital de Portugal, NZPCN, Essex Field Club, Maryland Biodiversity Project, Calflora, WikiAves, speciesLink, Ara Irititja.

**Stage 4 — The second cycle: the credibility test, and the origin of the disputed-figures problem.**
A project only becomes scientifically valuable when it has surveyed the same place twice, and at that moment it acquires the exact problem Sathyamangalam is designing for. The corpus is rich here:

| Project | Cycles | How it handles the disagreement |
|---|---|---|
| **EBBA1 → EBBA2** | 1980s → 2010s | Explicit **change maps** comparing the two atlas periods |
| **SABAP1 → SABAP2** | 1987–91 → 2007– | Historical-vs-current range comparison as the core product |
| **NY Breeding Bird Atlas** | 1980–85, 2000–05, third via eBird | The interface's **core function is showing two surveys of the same square disagreeing, without adjudicating** |
| **Sovon (Netherlands)** | 2012–15, new fieldwork Dec 2026–2030 | **Keeps the previous atlas online at its own subdomain** while the new one runs. Coexistence, not replacement |
| **Kerala Bird Atlas** | 1.0 (2015–2020, released 25 Jan 2021), 2.0 (from 16 Jul 2026) | Second cycle underway across 4,324 sub-cells / 38,863 km² |
| **Ontario** | Atlas-3 from 2021 | Third generation — two prior baselines to disagree with |
| **Virginia BBA2** | 2016–2020 | Publishes **explicit first-vs-second-atlas change findings** (8 species newly confirmed as breeders) |
| **Bulgaria (BSPB)** | Atlas 2007 → continuous monitoring since 2016 | **Continuous monitoring has replaced discrete editions** — a design choice that changes how a figure is cited at all |
| **Galápagos Species Checklist** | Multiple published checklists | **Publishes archived earlier checklist versions alongside the current one** |
| **Chile species classification** | **20 finalised processes**, 21st in consultation (June 2026) | A species' status is **not one value but a series of 20+ dated, decree-backed determinations** |
| **Sri Lanka Red Lists** | 2007, 2012, 2020 | The same species assessed differently in three editions |

**Stage 5 — Decline: five distinct modes.**

1. **Silent stall.** Content freezes, infrastructure keeps serving. Illinois Wildflowers, BONAP, Victorian Places, LandScope America, Alaska Native Place Names, NARvis, Libro Rojo de la Fauna Venezolana, Catálogo de Plantas y Líquenes de Colombia, Australian Faunal Directory, Protea Atlas (frozen at November 2001 — **25 years**).
2. **Infrastructure exhaustion.** UkrBIN's "Error 1040", Deutschlandflora's HTTP 500, the whole TLS-expiry cohort. Not a decision; an unpaid bill or an unattended renewal.
3. **Custodial vacuum.** PIER's orphan notice, E-Flora BC's dependency on one named coordinator, PNGplants as one person's national infrastructure.
4. **Deliberate retirement.** ScotlandsPlaces (24 June 2025), ICMBio Portal, Oriental Bird Images, Dictionary of Sydney ("This site was archived in 2021"), Living Archive of Aboriginal Languages.
5. **Catastrophic external event.** France's **INPN/OpenObs has been offline since a cyberattack in summer 2025 — still down on 25 August 2026, roughly 14 months** — while its *regional* platform SILENE kept publishing throughout (latest update 27 January 2026, ~90,000 new observations). Bayernflora's wiki, November 2023, unrestored. Task 1's conclusion: this is the strongest available argument for **federating rather than centralising**.

**Stage 6 — Afterlife: what is actually left.**
Four possible end-states, in descending order of value: *data in a mandated institution* (INBio's specimens, Ontario's records in iNaturalist, OBI's images in Macaulay); *a peer-reviewed paper* (Boston Harbor Islands); *an archive deposit with a persistent identifier* (Gatehouse Gazetteer at the Internet Archive, Digital Atlas of Idaho at archive.org); or **nothing** (papuaweb, REBIOMA, CRBio, hbmp.hawaii.edu, NBII, BISON).

### 2.6.2 Observed lifespans, by archetype

Compiled only from projects where Task 1 records both a start and a most-recent-evidence date. **Read with the survivorship caveat in §2.0.**

| Archetype | Longest observed | Median-ish range for survivors | Typical death age |
|---|---|---|---|
| **A. Statutory / agency** | Rothamsted Insect Survey **1964–2026 (62 yrs, continuous)**; Amboseli Elephant Research **1972–2026 (54)**; Ethiopian Wolf Conservation Programme **1988–2026 (38)**; Hawaii Biological Survey **1992–2026 (34)** | 25–60 yrs | No natural death observed |
| **B. Consortium / national node** | speciesLink **2002–2026 (24)**; BioGIS announced in *Science* **2001**, still running (25) | 15–25 yrs | Integration-layer death (ScotlandsPlaces, ~2009–2025) |
| **C. Society / membership NGO** | Ara Irititja **1994–2026 (32)**; Keystone Foundation **1993–**; NZPCN **2003–2026 (23)**; Naturbasen ~25 yrs; Biodiversidad Virtual **1995–2026 (31, platform ceded)** | 18–32 yrs | Web-layer fossilisation, not organisational death (Sussex BRS) |
| **D. Small academic lab** | Janzen & Hallwachs ACG (decades); Lyudao LTSER | 5–20 yrs | **4–6 yrs — at grant end.** The most fragile archetype |
| **E. Solo / near-solo** | atlas-roslin.pl **2002–2026 (24)**; Flora of Zimbabwe **2002–2026 (24)**; Flora Digital de Portugal **2004–2026 (22)**; Flowers of India **2005–2026 (21)**; Minnesota Wildflowers **2006–2026 (20)**; Plantarium **2007–2026 (19)**; Wildflowers of Ireland **2008–2026 (18)**; WikiAves **2008–2026 (18)**; Caribbean Herpetology **2010–2026 (16)**; EcoRegistros **2011–2026 (15)** | **15–24 yrs** | Clean stop at 15–20 yrs (Illinois Wildflowers 18 yrs, BONAP ~12 yrs to last release) |
| **F. Community-governed** | Ara Irititja **1994–** (32); Papakilo **2011–2024** (13, slowing); Nunaliit atlases 2008–2018 build phases | 10–30 yrs | Domain/hosting loss after the technical partnership ends (Arctic Bay Atlas) |

**The headline number for Sathyamangalam:** the solo/near-solo archetype's survivors cluster at **15–24 years**, which is longer than the grant-funded academic archetype by a factor of three to four. This is counter-intuitive and it is the corpus's strongest single result. The reason is visible in the mechanism: a solo project has no funding cliff to fall off, so its only failure mode is the person stopping — whereas a grant-funded project has a scheduled, predictable death built into it from day one unless it acquires an institutional host before the money ends.

### 2.6.3 What the two-to-six year window actually requires

Combining Stage 2 and Stage 3, the corpus says a project must achieve **at least one** of the following before its initial energy or funding runs out, or it joins the Stage 5 silent-stall cohort:

1. **A legal or governmental anchor** (Hawaii, Wales, Kerala).
2. **An institutional host with an independent reason to care** (Pladias post-2018; HGIS de las Indias at Yale).
3. **Earned income from a distinct buyer** (Essex Field Club's Datasearch; Naturbasen's municipal agreements).
4. **A citable publication stream that makes the project part of the scientific record regardless of the website's fate** (Singapore Biodiversity Records; Caribbean Herpetology; Records of the Hawaii Biological Survey; Mysuru's Dryad DOI).
5. **A community large enough to survive the founder** (WikiAves at 57,304 observers; Flora-On's ~10-person credited core).

Route 4 is the only one of the five that a two-to-three person team can execute unilaterally, without permission, funding, or an institution agreeing to anything.

---

## 2.7 Summary of Task 2 findings

1. **Non-disclosure is endemic.** Of 250+ projects, only ~24 publish a team size and almost none publish a budget. Publishing your own counts, dates and coverage gaps is therefore a differentiator, not an overhead.
2. **Funding does not predict survival; institutional ownership does.** INBio (3.7M specimens, international funding) is dead at both the original and successor portal. vncreatures (one person, no funding) added entries on 17 August 2026.
3. **The dominant death is the grant cliff at years 4–6**, and it is entirely predictable from day one.
4. **The solo/small-team archetype outlives the grant-funded academic archetype by 3–4×**, with survivors clustering at 15–24 years.
5. **The presentation layer always dies before the data layer.** Five independent cases. The narrative/historical layer is the most fragile part of a multi-domain atlas (Bayernflora).
6. **A core of 1–4 editorial decision-makers is normal and sufficient** — provided the capture layer is either delegated to an existing platform or supplied by a recruited periphery.
7. **There is no Indian ABC-equivalent funding programme.** France's OFB spends €2–7.6M a year so that 46–100 localities each build a comparable atlas to a published standard. India has no analogue, which forecloses the most natural funding route for a district-scale atlas.
8. **Four viable build strategies exist and only one is expensive.** Riding a platform costs nothing; deploying GeoNature, the Biodiversity Informatics Platform, Symbiota or Nunaliit costs a server and an admin; publish-first costs editorial time; building custom costs permanently.
9. **The second survey cycle is when the disputed-figures problem becomes real**, and at least eleven projects in the corpus already handle it in production — none of them in India.
10. **JavaScript-only rendering is an archival decision, not a front-end one**, and roughly eighteen of the corpus's most authoritative resources have already made it wrongly.

---

## Sources — Task 2

**Primary evidence base**

- `Task1_Worldwide_Inventory.md` (this project, compiled 25 August 2026). All project names, URLs, counts, status evidence and verification dates above are drawn from it. Its own per-continent Sources sections carry the underlying citations for every project referenced here and are not duplicated.

**Funding-mechanism verification (independently checked 26 August 2026)**

- [Rufford Small Grants — general criteria and grants available](https://apply.ruffordsmallgrants.org/help/criteria) — grant tiers £7,000 / £8,000 / £12,000 / £18,000, sequential structure, eligibility, 12-month intervals
- [The Rufford Foundation](https://www.rufford.org/)
- [GBIF — First BID grants provide nearly €1 million in funding to 23 African projects](https://www.gbif.org/news/82784/first-bid-grants-provide-nearly-euro1-million-in-funding-to-23-african-projects) — €972,156 across 23 projects, €817,369 co-funding, 34 organisations in 20 countries, €3.9M EU programme budget
- [GBIF — BID: Biodiversity Information for Development programme page](https://www.gbif.org/programme/82243/bid-biodiversity-information-for-development) — regions covered (per-tier grant amounts *not published*)
- [JRS Biodiversity Foundation — Our Grants](https://jrsbiodiversity.org/our-grants/) — US$7,000–US$525,000 range, 12–48 month durations, sub-Saharan Africa focus, three programme areas
- [Office français de la biodiversité — Financement des Atlas de la biodiversité communale](https://ofb.gouv.fr/financements/financement-des-atlas-de-la-biodiversite-communale) — annual envelopes: €7.6M/100 projects (2023), €2.0M/47 (2022), €5.0M/98 (2021), €2.5M/46 (2020); eligibility
- [Aides-territoires — Mieux connaître et mobiliser pour agir pour la biodiversité (ABC)](https://aides-territoires.beta.gouv.fr/aides/61e1-mieux-connaitre-et-mobiliser-pour-agir-pour-l/) — €15M SNB envelope (per-project caps and co-financing rate *not published*)
- [PNR du Massif des Bauges — ABC: 14 communes / €185,000](https://parcdesbauges.com/abc/)
- [Florida Museum — iDigBio receives $20 million from NSF to sustain museum digitization](https://www.floridamuseum.ufl.edu/science/idigbio-receives-20-million-from-nsf-to-sustain-museum-digitization/) — ~US$20M over five years (2021), 128M+ specimen records representing ~400M specimens, ~40% of US collections, 2,000+ studies
- [NSF 15-576: Advancing Digitization of Biodiversity Collections (ADBC)](https://www.nsf.gov/publications/pub_summ.jsp?org=NSF&ods_key=nsf15576) — programme framing (total programme investment *not published*)
- [Symbiota — Sustaining Symbiota Services](https://symbiota.org/sustaining-symbiota-services/) — iDigBio-supported hosting for pre-2021 NSF portals up to 30 TB; fee-based contracts via a KU Service Center, rates quote-only
- [GeoNature](https://geonature.fr/) — maintaining parks, BAM launch July 2026, national meeting 5–6 November 2026, public roadmap (deploying-organisation count and hosting cost *not published*)
- [Natural Solutions — GeoNature: guide complet pour les collectivités](https://www.natural-solutions.eu/blog/geonature-guide-complet-pour-les-collectivits-et-la-communaut-francophone) — module structure and required roles (headcount and cost *not published*)
- [Virginia Bird Atlas — Acknowledgments](https://vabirdatlas.org/about/acknowledgments/) — funder split between VDWR (fieldwork 2016–2022) and VSO (publication 2023–2025); named coordinator roles (Sergio Harding, Ashley Peele, Austin Kane); partners VDWR / VSO / CMI at Virginia Tech (dollar amounts *not published*)

**Explicitly unverified in Task 2**

- Total NSF ADBC programme investment across all Thematic Collections Networks (only the ~US$20M iDigBio hub award is confirmed)
- Per-tier grant amounts within GBIF BID (regional / national / small)
- Per-project funding caps, co-financing rates and project durations for the OFB ABC programme
- Symbiota fee-based contract rates
- GeoNature deploying-organisation count, current release number, and hosting cost
- Which Rufford tier funded Co's Digital Flora of the Philippines
- Budget figures for every project in Task 1 other than those named above — none are published


---

# Task 3 — How These Projects Win and Earn

**Prepared for:** the Sathyamangalam Tiger Reserve conservation atlas project
**Compiled:** 26 August 2026
**Evidence base:** `Task1_Worldwide_Inventory.md` (verified 25 August 2026) and `Task2_How_It_Works.md`. Price points, membership structures and adoption metrics not recorded in Task 1 were verified independently on 26 August 2026 and are cited in this task's Sources section.

---

## 3.0 "Winning" is four different things, and the corpus shows them coming apart

Projects in this category are routinely described as successful without specifying which kind of success is meant. The Task 1 corpus makes the distinctions unavoidable, because it contains clear cases of each one **without** the others:

| Measure | Definition | A project that has it *without* the others |
|---|---|---|
| **Adoption** | People contribute to it and use it | **WikiAves** — 6,546,363 records, 57,304 observers, 18 years, no institutional host and no formal scientific standing |
| **Credibility** | It is cited, and treated as evidence | **Pladias** — a full methods paper in *Preslia*, an explicit published scope rule, and no mass participation at all |
| **Longevity** | It is still there | **Digital Atlas of Idaho** — comprehensive, seven domains, one place, still serving, and *effectively uncitable* because nothing on it carries a date or an author |
| **Consequence** | It changes a decision someone else has to make | **FWS ECOS / IPaC** — the query is legally consequential; nobody "contributes" to it and nobody praises its UX |

These four are separable, they are earned by different mechanisms, and only two of them generate money. Sections 3.1–3.4 take them in turn; §3.5 covers earning; §3.6 covers the measure that matters most for a tiger reserve.

---

## 3.1 Adoption — the mechanisms the corpus shows actually working

### 3.1.1 Anchor the survey to a calendar people already keep

The cheapest adoption mechanism in the corpus, and India has two independent instances of it:

| Project | Anchor |
|---|---|
| **Pongal Bird Count** (Tamil Nadu) | Pongal — a culturally anchored survey window |
| **Bihu Bird Count** (Assam) | Bihu, run by the Assam Bird Monitoring Network |
| **Odolympics** (OdonataCentral) | A named, dated event — 13–23 June 2026 |
| **Great Aussie Fungi Hunt** (Fungimap) | 2026 edition drew **4,899 observers** |
| **"MAP kihívás"** (MME, Hungary) | A challenge run *on the database itself*, May–July 2026 |
| **Toponimízate** (Toponimia de Galicia) | A recurring public campaign, events through 2026 |
| **eBird India July 2026 Challenge / WWF-India Annual Vulture Count** | 1–30 September 2026, promoted through Bird Count India |

### 3.1.2 Make the target list bounded and tractable

- **Fungimap** runs on **100 target taxa** — Task 1's read is that this "keeps a volunteer scheme tractable."
- **Bird Gap Filling in Tamil Nadu** deliberately targets *under-birded cells* rather than asking for records everywhere. Task 1 notes Sathyamangalam sits in exactly the kind of gap this project targets.
- **Keystone Foundation's** Western Ghats arboretum tracks **33 endangered tree species**, currently in a Google spreadsheet.

An unbounded "record anything" invitation produces less than a bounded one, and the bounded version also produces a defensible completeness claim.

### 3.1.3 Publish growth as news, and credit contributors by name

| Project | Mechanism |
|---|---|
| **Maryland Biodiversity Project** | Homepage announcement dated **23 August 2026** adding *Heliophanus kochii* as a **new state spider record**. Daily new-species announcements make the atlas's growth legible to its own community |
| **Flora-On** (Portugal) | Weekly update log with **per-contributor credit by name** — Paulo Ventura Araújo, Josué Orfão, Francisco Clamote, Maria João Correia and others; week of 17–23 Aug 2026: 2,060 records, 299 new grid squares, 220 species |
| **Canberra Nature Map / NatureMapr** | Publicly names top contributors (AlisonMilton 19K sightings, trevorpreston 13.8K) and the **top moderator with his verification count** (MichaelMulvaney, 60,600). Reputation as a visible provenance mechanism |
| **Atlas des oiseaux nicheurs du Québec** | Public **Top-10 contributor leaderboard** across five dimensions — records, hours, species, squares, point counts |
| **Hyderabad Bird Atlas** | Per-season volunteer counts, species counts and individual-bird counts across four seasons |
| **Endemia.nc** (New Caledonia) | A **public per-taxon edit log** as the primary evidence of liveness |

### 3.1.4 Put capture where the people already are

The corpus's cheapest architecture is also its best adoption mechanism. Ohio Odonata Society runs collection entirely on iNaturalist and keeps only the interpretive layer; Atlas of Life in the Coastal Wilderness solved sustainability the same way; Moths of Krishnagiri — the nearest comparable effort to Sathyamangalam geographically — is iNaturalist-hosted, so its data is immediately open; the Kerala, Hyderabad, Coimbatore, Ahmedabad and Bhagalpur atlases all run on eBird; Pune runs on **Epicollect5**.

### 3.1.5 Language and device

- **Dabasdati.lv** (Latvia, 2,325,630 observations) is **quadrilingual** (LV/EN/RU/LT) — a small-country project that solved multilingual access early.
- **Flowers of India** carries common names in many Indian languages; Task 1 identifies this as directly relevant to gazetteer/name-variant work.
- **EncicloVida** (Mexico) treats **common names in indigenous languages as first-class data fields**.
- **Ulukau** runs a **Hawaiian-language default interface** with English secondary.
- Mobile presence is now baseline: Bhutan Biodiversity Portal (Google Play), Naturalista, SmartBirds Pro, Canberra Nature Map (Android + iOS), Calflora, EncicloVida (iOS + Android), Deutschlandflora 3.0.

### 3.1.6 Roll up to the administrative unit people actually ask about

**WikiAves** offers **municipality-level record browsing** — Task 1 calls this "the key transferable idea: administrative-unit rollups, not just point maps, which is how local users actually ask questions." BONAP maps to county; Symbiota consortia host place-scoped "Inventory Projects"; **eElurikkus** publishes **per-national-park species dossiers** (Soomaa, Matsalu, Lahemaa).

### 3.1.7 The discoverability trap

Two failure modes worth naming because both are self-inflicted:

- **`vncreatures.net` serves `noindex, nofollow`** — the single most encouraging solo project in Asia is **invisible to search engines**.
- The **Chinese solo-taxon cluster** (shellsmap, GobiologyCN, china-odonata.top, ants-china.com, insectaintegration.com, mycopedia.top, liudayadan.com) is discoverable **only through a community-maintained iNaturalist link list**, not through search.

---

## 3.2 Credibility — the ladder from hobby site to evidence

The corpus supports a ranking. Each rung is achievable independently, and each is strictly cheaper than the one below it is valuable.

### Rung 1 — A peer-reviewed paper *about the project itself*

Not a paper using the data; a paper describing the resource, its method and its scope. This is the single most common credibility act in the corpus.

| Project | Publication |
|---|---|
| **Pladias** | Full methods paper in *Preslia* — Task 1 calls it "the gold standard for a citable data infrastructure" |
| **Kerala Bird Atlas** | *Current Science* **122(3):298** and *Forktail* |
| **Marine Biodiversity Portal of Bangladesh** | A data paper describing the portal, 2023 |
| **NVS Databank** (NZ) | *New Zealand Journal of Ecology* article 2124 — "an atlas that published its own architecture" |
| **Co's Digital Flora of the Philippines** | *Philippine Journal of Science*; subsequently adopted as a **World Flora Online regional flora** |
| **Vermont Atlas of Life** | "The Vermont Atlas of Life: Discovering and Sharing Biodiversity Knowledge" |
| **Moscow Digital Herbarium** | 2020 paper documenting the consortium model |
| **Wikiplantbase / Acta Plantarum** | Published literature explicitly arguing the case for **amateur-built large floristic databases** — directly citable evidence for a small team's scientific legitimacy |
| **Mysuru City Bird Atlas** | *Indian BIRDS* 15(3) |
| **Hornbill Watch** | Data paper in *Indian BIRDS* 14(3) |
| **JBIF** (Japan) | A BISS paper on national-node mechanics |

### Rung 2 — Publish records as citable, dated, attributable notes

Task 1 identifies this as "the most important structural idea in this whole survey for a small atlas."

| Project | Mechanism |
|---|---|
| **Singapore Biodiversity Records** (LKCNHM) | Individual sighting records published as short numbered notes, **ISSN 2345-7597**. Every observation becomes a permanent, attributable, disputable document — and this is what enabled the *PNAS* analysis of two centuries of Singapore biodiversity discovery and loss |
| **Caribbean Herpetology** | A peer-reviewed open-access **journal fused to the species database**, **ISSN 2333-2468** — new records are published as citable notes that simultaneously update the atlas. **Issue 100, June 2026**, from four named academics with no large institution |
| **Records of the Hawaii Biological Survey** | Annual since **1994** — "every added or corrected record becomes a citable act" |
| **FloraBase** (WA) | Hosts the peer-reviewed journal *Nuytsia* **inside the atlas**, so new names are published and indexed in the same place they are browsed |

### Rung 3 — Deposit the dataset with a persistent identifier

- **Mysuru City Bird Atlas** → **Dryad DOI 10.5061/dryad.8k3d81r**. Task 1: "the key transferable practice: a small city atlas that made its raw data permanently citable."
- **BSBI Plant Atlas 2020** → Zenodo release, 2 May 2024.
- **Bulgaria's 2007 breeding bird atlas** → archived as a GBIF dataset.
- **Gatehouse Gazetteer** → full copy deposited at the Internet Archive, with a Wikidata item, as an explicit succession plan.

### Rung 4 — Version the whole resource, not just the records

Without a version, "the figure" is uncitable and every disagreement is unresolvable. Nine production examples:

| Project | Versioning scheme |
|---|---|
| **Fishes of Texas** | **v3.10, released 2026-03-23** — Task 1: "the best North American exemplar of explicit data-conflict handling," because the project exists precisely because aggregated records disagree |
| **Flora e Funga do Brasil** | **v393.397**, published 1 January 2024 — a four-digit version on a continuously edited checklist |
| **FlorItaly** | **2025.2** (also 2023.1) |
| **Jepson eFlora** | **Revision 11, posted 2022-12-23** — a reader can cite "as of Revision 11" |
| **MapBiomas** | Numbered **Collections**; Peru Collection 4 released **20 August 2026**, covering 1985–2025 |
| **AODN Portal** | **v4.42.110, last updated 27 July 2026**, shown in the UI footer |
| **Artsdatabanken** | Dataset **version 1.364, published 24 August 2026**, daily cadence |
| **Biodiversitäts-Atlas Österreich** | Pins **GBIF Backbone Taxonomy v2021** |
| **Catalogue of Life China** | Dated annual checklist editions |
| **Protected Planet** | Monthly versioned releases; statistics last updated August 2026 |

### Rung 5 — Name who is responsible for each judgement

**Biodiversity Atlas – India** operates a peer-reviewed/curated editorial model under a named Chief Editor (Krushnamegh Kunte, NCBS-TIFR) — Task 1 calls it "the closest Indian analogue to a 'disputed record adjudicated by named editors' workflow." **Euro+Med PlantBase** makes each area's record attributable to a named regional editor. **Massachusetts Butterfly Club** runs 200,000+ records through one named Records Compiler.

### Rung 6 — Publish your inclusion rule and your rejections

- **Pladias** publishes an explicit **scope rule** (native and naturalised flora; most cultivated plants excluded but major crops and woody plants included) — "the kind of stated inclusion criterion that prevents later disputes about counts."
- **Digital Atlas of the Virginia Flora** maintains a published **"Excluded Taxa"** document — a reasoned register of rejected records kept visible instead of silently deleted. Task 1: "cheap to implement, enormously valuable for trust."
- **Land Conflict Watch** publishes a four-step verification method with an explicit rule that each dispute and its facts must be verified from **more than one source**.
- **Maryland Biodiversity Project** publishes a public "About Records" page **defining what counts as a record**.

### Rung 7 — Publish to GBIF via an IPT

The lowest-effort route from a local database to something harvestable, mirrored and citable: **Bhutan Biodiversity Portal** publishes as a GBIF dataset; **TaiCOL** publishes the national checklist dataset; **Hornbill Watch** publishes as a GBIF occurrence dataset; **BSPB** is documented as a major GBIF publisher; **NVS**, **TaiBIF**, **JBIF**, **Croatia HR IPT**, **ipt.gbif.ch** and **JBRJ's IPT** all run instances.

### Rung 8 — Government adoption (the terminal state)

| Case | What adoption looked like |
|---|---|
| **Kerala Bird Atlas** | Atlas 2.0 **inaugurated on 15 July 2026 by the Kerala Forest Minister**; ran with the Kerala Forest Department and Kerala Agricultural University |
| **Goa Bird Atlas** | India's second **state-published** bird atlas. Separately, **GBCN drove the designation of 4 Important Bird Areas in 2011** and proposed 3 more in 2014 — a small registered society producing legally-referenced spatial designations |
| **Hawaii Biological Survey** | A **statutory mandate from the Hawaii State Legislature, 1992** — a legal instrument creating the atlas |
| **List of Historic Place Names of Wales** | A **statutory** historic-name register under the Historic Environment (Wales) Act 2016 |
| **Inuit Heritage Trust place names** | The gazetteer is **a route to official recognition** under the Government of Nunavut's Geographic Names Policy — which is also why the authoritative dataset is deliberately not fully public |

### 3.2.9 The anti-credibility catalogue — and why it matters here specifically

The corpus contains an unusually rich set of projects that **display two contradictory figures without labelling them**. For a project whose central feature is surfacing disputed figures, these are not curiosities; they are the competitive case for the feature.

| Project | The unlabelled contradiction |
|---|---|
| **Land Conflict Watch** (India) | Homepage simultaneously shows **1,084 ongoing conflicts / 14.1M people affected** and a second view showing **921 conflicts / 10.5M people** — in the most methodologically rigorous disputed-facts project in India |
| **mahoot** (Asian elephants) | Advertises "14,000+ named individuals" against its own displayed **8,297** named elephants; also claims coverage of "94 countries" across "13 Asian range states" |
| **India Flora Online / Herbarium JCB** | States the herbarium holds "more than 20,000 specimens" in one place and **"14,000 specimens"** in another, on the same site |
| **Victorian Biodiversity Atlas** | Three published record counts, none reconciled: "more than **6 million**" and "over **seven million**" *in the same 2020 article*, plus "more than **6.5 million**" elsewhere |
| **NatureMapr** | The national site and Canberra node both report 844,303 sightings / 23,771 species / 16,082 members, while the Budawang Coast node reports **2,091,410 sightings / 18,688 species / 9,619 contributors** — different scoping of the same counters |
| **Bhutan** | Two national portals on two different stacks with overlapping mandates, generating divergent species counts for one small country |
| **Portugal** | Two independent national flora sites (Flora-On and Flora Digital de Portugal) — "an inherent conflicting-figures situation" |
| **Flora of China** | Chinese (FRPS) and English (Flora of China) editions of the same flora **do not always agree** |
| **SiB Colombia** | Returned **corrupted placeholder figures** — repeated "100,000" across unrelated counters |
| **PIPAP** (Pacific) | Dashboard renders **"0 Protected Areas (0 Terrestrial, 0 Marine)"**. Task 1: "a broken counter reads as an authoritative claim" |

And four ways to have no credibility at all:

- **Wild Atlas** (India) — claims "100+ national parks, 500+ species" with **no team, no institution, no methodology, no dates**, and a safari booking funnel. Task 1 includes it explicitly as an anti-pattern.
- **Digital Atlas of Idaho** — comprehensive, seven domains, exactly the Sathyamangalam shape, and "effectively uncitable because nothing on it carries a date."
- **M-STrIPES** — login-walled; Task 1: "the *reason* a public reserve atlas has value." Data reaches the public only via the quadrennial All-India Tiger Estimation.
- **TKDL** — comprehensive historical-text mining built deliberately as a **closed** database, access restricted to patent offices under access agreements, and criticised for it in *Third World Quarterly*.

---

## 3.3 Longevity — condensed from Task 2

Established in Task 2 and not repeated at length: institutional ownership beats funding; the presentation layer dies before the data layer; solo survivors run 15–24 years while grant-funded academic builds die at years 4–6. The five routes past the year-4–6 cliff were: statutory anchor, host institution, earned income, citable publication stream, or a community large enough to survive the founder. **Only the citable publication stream is executable unilaterally by a two-to-three person team.**

One addition specific to Task 3: **honest decay labelling is a longevity mechanism, not just an ethics one.** A project that declares which half of itself is unmaintained (Tasmania's Threatened Species Link), states its freeze date (NY Breeding Bird Atlas, "finalized in March 2007"), or publishes its own coverage gap (Toponimia de Galicia leading with "35% of the territory") keeps its credibility through decline. A project that goes stale silently loses credibility retroactively across everything it ever published — and the corpus's fifteen-year-stale **Biodiversity of India / Project Brahma** page still ranks in search results, actively misinforming.

---

## 3.4 Earning — the full observed revenue architecture

### 3.4.1 The seven revenue lines, with the real prices the corpus does not publish

| # | Revenue line | Verified price points (26 Aug 2026) | Who pays | Task 1 practitioners |
|---|---|---|---|---|
| **1** | **Ecological desk studies / data searches** | **Gloucestershire (GCER), from April 2026:** 1 km £155, 2 km £175, 3 km £225, 5 km £295; bats-only £90 / £95 / £120 / £135; extra subject £20–£40; **5 working days**; 25% volume discount; nil-return capped at £50+VAT. **Bucks & Milton Keynes ERC, from April 2023:** Package A (1 km) **from £150+VAT**; Package B (2 km) **from £208+VAT**; Package C (parish) **£58+VAT**; Package E (2 km bats) **£94+VAT**; extra species group **from £34+VAT**; nil-return admin fee £58+VAT; **non-commercial users receive up to 100% discount**, and a free 500 m notable-species search (Package D) | **Ecological consultancies doing planning and EIA work** | **Essex Field Club** — sells Datasearch off the same **7,593,231-record / 22,218-species** database that powers its free public atlas (prices on request). The wider **ALERC** network of county LERCs runs on this model |
| **2** | **Statutory search fees** | **AHIMS** (NSW Aboriginal heritage) extensive search **A$60**; basic search free; **fee waivers for Aboriginal individuals and organisations** | Developers, consultants | AHIMS; **ArchSite** (NZ) charges for some requests |
| **3** | **Membership dues** | Amounts not published by any corpus project | Individuals | **NZPCN** (>1,000 members worldwide, 2003–); **Fungimap** (membership-based); **Essex Field Club** (registered charity 1217515); **Russian Bird Conservation Union** (~6,000 members across 66 regions); **NatureSpot** (registered charity 1138852) |
| **4** | **Service contract to government** | Values not published | State agencies, municipalities | **Ireland's NBDC** — a Heritage Council initiative **operated under a service-level agreement by Compass Informatics**, a private contractor, delivering 9,261,812 records; **Naturbasen** (Denmark) is free to its 119,474 members and sustained by **cooperation agreements with municipalities and national parks** rather than data sales; **Keeping Culture KMS** sells an annual cloud licence to Indigenous community archives |
| **5** | **Corporate memberships and co-produced products** | Not published | Companies | **Ainmean-Àite na h-Alba** (Gaelic place names) sells corporate memberships and co-produced maps |
| **6** | **Publication sales, then free release** | **Atlas des oiseaux nicheurs du Québec**: ~**10,000 copies over three printings**, now out of print, **free PDF available** | Individuals, libraries | Québec; Ontario R&A Atlas (443 pp., 2023); Colorado BBA II (744 pp., Nov 2016); San Diego County Bird Atlas (2004) |
| **7** | **Subscription tier over free content** | Not published | Researchers | **British History Online** — mostly free, small subscription tier for some Calendars of State Papers |

### 3.4.2 The one earned-revenue model that transfers directly to India

Line 1 is the most important row in this task, and the reason is structural rather than financial.

The UK LERC model works because **a regulated process legally requires a desk study**, and the records centre is the cheapest credible source for it. The buyer is not a nature lover; it is an ecological consultant on a planning deadline who needs a defensible document.

India has the same structural requirement and Task 1 already identified the machinery:

- **PARIVESH** (environmentclearance.nic.in / forestsclearance.nic.in) is the national system for environmental and forest clearances, with **PARIVESH 2.0 datasets published as bulk CSV on data.gov.in** — Task 1 calls this "the practical ingestion path for a legal-instrument layer."
- Every clearance application in or near the Sathyamangalam landscape requires a baseline biodiversity statement.
- Nobody currently sells that baseline for this landscape, because **no structured source exists** — Task 1 confirms the Erode / Coimbatore / Salem district gazetteers are **not available as structured data**, and that India's small-project layer is uniformly single-taxon and occurrence-only, so "nobody in this layer is doing multi-domain place documentation."

The corollary is uncomfortable but worth stating plainly: **the same feature that makes the atlas ethically valuable — surfacing disputed figures rather than picking one — makes it commercially valuable to exactly the actors who might misuse it**, because a consultant preparing a clearance application benefits from knowing which figures are contestable. Task 1's precedents for handling this are the access-tier designs in §3.4.3.

### 3.4.3 Access tiers as both an ethics mechanism and a revenue mechanism

Four production designs, all in Task 1:

| Project | Design |
|---|---|
| **Portable Antiquities Scheme** | **Findspot precision varies by user account** across 1,531,145 objects — the reference implementation of sensitive-location redaction in an open atlas |
| **ArtenFinder Rheinland-Pfalz** | A recorder's records stay **private until they explicitly release them** for conservation use |
| **SILENE** (PACA, France) | **Two interfaces over one database** — SILENE Nature (public) and SILENE Expert (professional), at different precision |
| **Sarawak Biodiversity Centre** | A **Traditional Knowledge Documentation department** with a legal access-and-benefit-sharing mandate: documented, but deliberately **not publicly browsable** |

And the two ends of the spectrum as cautionary limits: **TKDL**, closed to everyone but patent examiners and criticised for it; **M-STrIPES**, login-walled, which is precisely why an open reserve atlas has value.

### 3.4.4 What does *not* earn

- **Advertising or booking funnels.** Wild Atlas's safari-booking redirect is the corpus's only instance, and it is catalogued as an anti-pattern that destroys provenance credibility.
- **Data sales to the public.** No project in the corpus sells occurrence data to individuals. Naturbasen explicitly *rejected* data sales in favour of municipal agreements.
- **Paywalling the core resource.** The **China Biological Chronicles Library** is paywalled and, per the community list, incomplete — noted in Task 1 as "a comparator for the openness question."

---

## 3.5 Consequence — the success measure that actually fits a tiger reserve

None of adoption, credibility or longevity is the real prize for Sathyamangalam. The prize is **becoming the thing a process has to consult.** The corpus contains a clear set of projects that achieved this, and they cluster into three mechanisms.

### Mechanism A — Fuse the atlas to a regulatory query

| Project | Fusion |
|---|---|
| **FWS ECOS / IPaC** (US) | Query a project polygon → **official species list plus the legal instruments that attach** (critical habitat designations, section 7 biological opinions, recovery plans). Task 1: "the model for a place-keyed legal-instrument register" |
| **EPBC Protected Matters Search Tool** (Australia) | "What legal instruments apply to this polygon" — listed threatened species and ecological communities, migratory species, Ramsar wetlands, World/National Heritage places. Report carries an explicit **"indicative only"** caveat |
| **Georgia Biodiversity Portal** (US) | Combines the species atlas with the **environmental review request workflow in the same site** |
| **Pennsylvania Natural Heritage Program** | **Conservation Explorer / PNDI environmental review** fused into the same site as the County Natural Heritage Inventories — which Task 1 identifies as "the closest US administrative analogue to a district/reserve atlas" |
| **BiodiversityGIS / EIA Screening Tool** (SANBI, South Africa) | Species-sensitivity layers wired into EIA screening — "the closest South African analogue to a legal-instrument/EIA registry" |
| **Sri Lanka Biodiversity CHM** | Puts species/ecosystem data and **national legislation side by side in one navigation** — the best South Asian example |

### Mechanism B — Let a legal process generate the documentation

This is the inverse of Mechanism A and is the more striking finding.

- **Kā Huru Manu** (Ngāi Tahu Cultural Atlas) — over 1,000 original Māori place names of Te Waipounamu, fused with oral histories, historical map scans and traditional travel routes — **originated as an evidential by-product of Te Kereme, the Ngāi Tahu Treaty claim.** A legal instrument produced the documentation project. Task 1 calls it "the single most on-pattern project in the whole of Oceania for your brief."
- **Forest Rights Act resource portal** (Vasundhara, Odisha) — its "evidence" section is essentially **historical-document mining for legal use**: forest management plans and **pre-1980 records marshalled as proof of occupation**. Task 1: "very close in spirit to the user's historical-mining + legal-instrument combination."
- **Terras Indígenas no Brasil** (ISA) — 838 territories, each tracked through **seven distinct legal phases** (Em Identificação, Identificada, Declarada, Reservada, Homologada, Registrada, Encaminhada RI). "A territory's contested legal standing is the data model itself, not a footnote."
- **Mapping Prejudice** (Minnesota) — **crowdsourced legal-document mining at scale**: property deeds → racial covenants → maps, driven by twice-monthly community mapping sessions. Task 1: "if your atlas needs to mine gazette notifications or land records, this is the workflow to copy."
- **Inuit Heritage Trust** — the place-name gazetteer functions as **a route to official name recognition** under Nunavut's Geographic Names Policy.

### Mechanism C — Produce a designation

- **Goa Bird Conservation Network** — a registered society founded 2010 that **drove the designation of 4 Important Bird Areas in 2011** and proposed 3 more in 2014.
- **Ogiek Peoples' Ancestral Territories Atlas** (Mt Elgon, Kenya) — participatory territorial mapping as the basis of an ancestral-territory claim.
- **NACSO** (Namibia) — combines **legal conservancy gazettement** data with wildlife counts and community-benefit data in one system.

### What this means for Sathyamangalam

The reserve already sits inside three live legal machineries that generate contested figures continuously: **PARIVESH** environmental and forest clearances, **Forest Rights Act** claims and their pre-1980 evidence base, and the **NTCA/TIGERNET** tiger-mortality and estimation apparatus, which Task 1 describes as "structurally the closest existing thing to 'one authoritative number that journalists then dispute'." An atlas that becomes the reference layer for any one of those has achieved consequence; one that becomes the reference layer for the *disagreements between* them has achieved something no project in the corpus has done.

---

## 3.6 What losing looks like

Six distinct ways to fail at winning, each with a corpus instance:

| Failure | Instance |
|---|---|
| **Book-shaped output, thin web layer** | **Sussex Botanical Recording Society** — strong fieldwork, a printed county flora (2018), newest dated web content "The Stoneworts of Sussex (2020)", **no online atlas**. Task 1: "the commonest and least-reported failure mode: an active recording community with a fossilised website" |
| **No provenance** | **Wild Atlas** — an "atlas" with no dates, no team, no methodology, and a booking funnel |
| **Invisible** | **vncreatures** serving `noindex, nofollow`; the JavaScript-only cohort (Michigan Flora, every Nunaliit atlas, AADC, placenames.aq, AUSTLANG, Trove, EJAtlas, RUNAP, SlaveVoyages) invisible to crawlers, archivers and citation-checkers |
| **Undated** | **Digital Atlas of Idaho** — "no authorship, funding, or date anywhere reachable" |
| **Closed** | **TKDL**; **M-STrIPES** |
| **Claimed alive by its own parent** | **RSCN Jordan BIMS** — the parent organisation's page still describes the National Biodiversity Database as active and links a **404**; a 2,482-plant / 736-fauna database launched publicly in 2018 |

---

## 3.7 Summary of Task 3 findings

1. **Adoption, credibility, longevity and consequence are four separate things**, earned by different mechanisms, and the corpus contains projects that have each one without the others.
2. **Adoption is bought cheaply**: anchor to an existing calendar, bound the target list, publish growth as news, credit contributors by name, delegate capture to eBird/iNaturalist, roll up to administrative units, and be multilingual. None of these costs money.
3. **Credibility is an eight-rung ladder** and the bottom rungs are unilateral: a method paper about your own project, records published as citable dated notes, a DOI'd deposit, and a version number. Government adoption sits at the top and cannot be self-awarded.
4. **India's small-project layer is uniformly single-taxon and occurrence-only.** Task 1 states this directly. Multi-domain place documentation is the gap Sathyamangalam occupies, and there is no domestic competitor.
5. **The corpus is full of unlabelled contradictory figures** — Land Conflict Watch's own homepage, mahoot, India Flora Online, the Victorian Biodiversity Atlas, NatureMapr, Bhutan's two portals, Portugal's two floras, China's two flora editions. This is the strongest available evidence that the disputed-figures feature addresses a real and widespread defect, including in the most rigorous projects.
6. **Only one earned-revenue model in the corpus transfers cleanly to India**: selling ecological desk studies to consultants working a regulated process. Real UK prices run **£58–£295 per search** with 5-working-day turnaround and free access for non-commercial users. India's equivalent regulated process is **PARIVESH**, and no structured source currently exists for the Sathyamangalam landscape.
7. **Access tiers are simultaneously the ethics mechanism and the revenue mechanism** — PAS's account-based findspot precision, ArtenFinder's private-until-released, SILENE's dual interfaces, Sarawak's documented-but-not-browsable traditional knowledge.
8. **The real prize is consequence, not adoption.** Three mechanisms achieve it: fuse the atlas to a regulatory query (ECOS/IPaC, EPBC PMST, Georgia, PNHP, SANBI), let a legal process generate the documentation (Kā Huru Manu from the Ngāi Tahu Treaty claim; the FRA portal's pre-1980 evidence; Terras Indígenas' seven legal phases; Mapping Prejudice), or produce a designation (GBCN's four IBAs in 2011).
9. **Sathyamangalam sits inside three live figure-generating legal machineries** — PARIVESH, the Forest Rights Act, and NTCA/TIGERNET. Becoming the reference layer for the disagreements between them is a position no project in the corpus currently occupies.

---

## Sources — Task 3

**Primary evidence base**

- `Task1_Worldwide_Inventory.md` (25 August 2026) — all project names, URLs, counts and status evidence
- `Task2_How_It_Works.md` (26 August 2026) — funding mechanisms, archetypes, lifespans

**Independently verified for this task (26 August 2026)**

- [Gloucestershire Centre for Environmental Records — Data Search](https://www.gcer.co.uk/datasearch.html) — search prices from April 2026 (1 km £155 / 2 km £175 / 3 km £225 / 5 km £295; bats-only tiers; extra subject £20–£40; 5-working-day turnaround; 25% volume discount; nil-return cap £50+VAT); customers identified as ecological consultancies
- [Buckinghamshire & Milton Keynes Environmental Records Centre — Charging Policy and Data Services](https://www.bucksmkerc.org.uk/charging-policy-and-data-services/) — Packages A/B/C/E prices from 1 April 2023, extra species-group fee, nil-return admin fee, and up to 100% discount plus a free Package D for non-commercial users
- [ALERC — Association of Local Environmental Records Centres](https://www.alerc.org.uk/) — the network these prices sit within
- [Essex Field Club](https://www.essexfieldclub.org.uk/) — confirms 7,593,231 records / 22,218 species and the Datasearch desk-study service; **prices are quote-only and not published**
- [Kerala Bird Atlas 2015–20: features, outcomes and implications of a citizen-science project, *Current Science* 122(3):298](https://www.currentscience.ac.in/Volumes/122/03/0298.pdf) — the peer-reviewed record of the atlas (*direct fetch blocked by robots.txt; citation confirmed via search result metadata and Task 1*)
- [An Atlas of the Birds of Kerala (Bird Count India, PDF)](https://birdcount.in/wp-content/uploads/2021/01/Kerala-Bird-Atlas-Final-Compressed.pdf) — the published atlas itself
- [Bird Count India — Kerala Bird Atlas](https://birdcount.in/kerala-bird-atlas/)

**Explicitly unverified in Task 3**

- Membership fee amounts for every membership-funded project in the corpus (NZPCN, Fungimap, Essex Field Club, NatureSpot, RBCU) — none publish them on the pages reachable in this session
- Contract values for Ireland's NBDC / Compass Informatics SLA, Naturbasen's municipal cooperation agreements, and Keeping Culture's annual licence
- Essex Field Club Datasearch prices (quote-only)
- Ainmean-Àite corporate membership rates
- Whether any Indian body currently sells biodiversity desk studies for EIA/clearance work — **no such service was found in Task 1 or in this task's searches, but absence of evidence here is weak evidence of absence and this deserves a targeted check**


---

# Task 4 — Four Follow-Up Angles

**Prepared for:** the Sathyamangalam Tiger Reserve conservation atlas project
**Compiled:** 26 August 2026
**Evidence base:** `Task1_Worldwide_Inventory.md`, `Task2_How_It_Works.md`, `Task3_Win_And_Earn.md`, plus independent verification of standards specifications, GBIF publishing requirements, non-biodiversity disputed-fact systems, and Sathyamangalam's own published figures — all checked 26 August 2026 and cited in this task's Sources section.

---

## 4.1 — Who has solved "disputed facts side-by-side" well, in any domain

### 4.1.1 The single best model: Wikidata's rank + qualifier + reference triple

Wikidata is the most complete production answer to this problem anywhere, and it is not a biodiversity system.

Its statement model has three layers stacked on every claim:

| Layer | What it does |
|---|---|
| **Claim** | A property–value pair. **Multiple claims for the same property coexist by default.** A place can carry four different area values simultaneously |
| **Qualifiers** | Scope each claim — *point in time*, *determination method*, *criterion used*, *stated as*. This is what turns "three contradictory numbers" into "three numbers measured differently" |
| **References** | Attach the source *to the individual claim*, not to the item. Two claims disagreeing carry their own separate provenance |
| **Rank** | **preferred / normal / deprecated** — three-valued, applied per claim |

The rank system is the part worth copying exactly:

- **Preferred** — the value to show by default (typically the most current or most reliable). It does **not** delete the others.
- **Normal** — a legitimate value that is not the default.
- **Deprecated** — a value **known to be wrong**, retained deliberately so the record shows that a source once asserted it. This is the piece almost every other system lacks: a place to keep a wrong number *because someone authoritative published it*.

For Sathyamangalam this maps directly. A tiger-count claim gets: value, `point in time`, `determination method` (camera trap / DNA pugmark / interview-based / informed guess), `stated by` (NTCA / TNFD / WWF / press), and a rank. Nothing is discarded, nothing is silently arbitrated, and the default display is still a single clean number.

### 4.1.2 Data journalism: Our World in Data's "no single best source" pattern

OWID's treatment of armed-conflict death counts is the clearest published example of refusing to reconcile.

- It presents **six major datasets side by side in comparison tables**, comparing them across conflict definition, death counting, time period and identification procedure.
- It makes the **definitional difference the headline**, not a footnote: UCDP counts conflicts with **at least 25 deaths in a year**, Correlates of War requires **at least 1,000**; some include civilian deaths, some do not; Project Mars covers only conventional wars with differentiated militaries and clear frontlines.
- Where uncertainty persists, sources publish **best / low / high estimates**, and OWID says which convention each uses (UCDP takes the lower figure as its best estimate when no primary source is more reliable).
- It closes with **use-case guidance rather than a verdict**: "there is no single 'best' approach" — UCDP for both large and small recent conflicts, Project Mars for long-term analysis.

The transferable move: **the reader leaves knowing which number to use for their question**, not which number is true.

### 4.1.3 Law: citators preserve superseded authority rather than deleting it

Legal case-law databases solved this problem decades ago and the vocabulary is mature.

- A superseded case **keeps its full original text and its citation value**. What changes is a *treatment layer* attached to it.
- **KeyCite (Westlaw)** flags: yellow (some negative treatment, still good law), red (overturned on at least one point), red stripe (partial reversal, other holdings valid), blue stripe (under appeal), orange caution (**relies on an overruled or otherwise invalid prior decision** — i.e. inherited invalidity), no flag.
- **Shepard's (Lexis)** signals: green plus (affirmed/followed), red stop (strong negative history), orange Q (validity questioned), yellow triangle (criticised by / distinguished by), blue A (analysed, neutral), blue I (cited without analysis).
- Treatment categories are a controlled vocabulary: *distinguished by*, *followed by*, *not followed by*, *declined to extend by*.

Two ideas worth stealing outright: **inherited invalidity** (a figure that was derived from a superseded figure should be flagged as such automatically), and **neutral treatment codes** — "cited without analysis" is a real and useful state that biodiversity systems have no word for.

### 4.1.4 Historical gazetteers: Linked Places Format

Task 1 identified the World Historical Gazetteer (2,236,719 indexed places; index of 47M places / 67M toponyms; v3.2, blog post 21 Aug 2026) as the most useful technical model for the place-name layer. The underlying specification is worth naming precisely because it is directly adoptable.

**Linked Places Format (LPF)** is JSON-LD that is simultaneously valid GeoJSON and valid RDF. Its relevant structures:

```
"names": [
  { "toponym": "…",
    "lang": "ta",
    "citations": [ { "label": "…", "year": 1908, "@id": "…" } ],
    "when": { "timespans": [ { "start": {"in":"1900"}, "end": {"in":"1947"} } ] } }
]
```

- **`names[]`** holds many toponyms per place, each with its own **`citations[]`** (label, year, URI) and its own **`when{}`** temporal scope. No preferred-name field is required to exist.
- **`geometry`** can be a **`GeometryCollection`** where each geometry carries its own `when` **and its own `certainty`** — so two competing boundary definitions for the same place coexist, each dated and each rated.
- **`certainty`** is a controlled three-value field: **`certain` / `less-certain` / `uncertain`**.
- **`when{}`** can scope an entire feature, *or a single name*, *or a single geometry*, *or a type*, *or a relation*.
- **`links[]`** carries `closeMatch` / `exactMatch` to external gazetteers.

Task 1's own conclusion stands and is now specific: **do not build a single-preferred-name gazetteer.** A reserve whose villages carry Tamil, transliterated-colonial and revenue-record names in two scripts will break one immediately.

Two supporting precedents from Task 1 reinforce the same design:

- **SCAR Composite Gazetteer of Antarctica** — each feature gets a numerical **UID, and the UID carries a list of applicable place names**, deliberately not one canonical name. It also states outright that it "has no legal authority or standing."
- **Nottingham's Digital Survey of English Place-Names** — deliberately reproduces superseded etymologies unchanged, stating it does so "even where the etymologies suggested are now considered less likely than alternative explanations", and publishes the corrective consensus as a *separate* resource, **KEPN**. Two layers, both citable, neither overwriting the other.

### 4.1.5 Biodiversity: eighteen production mechanisms already in the corpus

Task 1 concluded that **no Asian project presents a disputed figure as a first-class object with competing sources attached**, and nothing in this task changes that. But the component mechanisms all exist somewhere:

| Mechanism | Project | What it does |
|---|---|---|
| **Two authorities as two simultaneous fields** | **TaiCOL** (Taiwan) | Stores national red-list level **and** IUCN category as separate, simultaneously displayed fields |
| **Global and national status as two columns** | **Himalayan Nature** (Nepal) | National Red List of Nepal's Birds pairs global *and* national status per species |
| **Three-tier ranks** | **NatureServe Explorer** | G / N / S ranks — global rank can differ from a state rank for the same taxon, by design |
| **Three international + national verdicts on one page** | **EncicloVida** (Mexico) | A single species page carries **NOM-059 national status, CITES and IUCN** without reconciling them |
| **Published-vs-accepted split** | **APNI / APC** (Australia) | APNI records **every published usage** of a name in a reference (many, conflicting); APC records the single accepted taxonomy. Task 1: "Copy this split" |
| **Accepted and unaccepted both retained** | **WoRMS** | Unaccepted names kept and linked rather than overwritten |
| **Annual change log** | **AmphibiaChina** | Records what changed, when and why — 759 species as of 23 July 2026 |
| **Competing hypotheses, expert-rated** | **BioModelos** (Colombia) | Stores **multiple competing distribution hypotheses per species** and lets experts qualify them. Task 1: "the most directly relevant conflicting-data mechanism I found" |
| **Institutionalised "we don't know"** | **Galápagos Species Checklist** | **`cryptogenic`** as a first-class origin value meaning *we genuinely do not know if this is native or introduced*; also publishes archived earlier checklist versions alongside the current one |
| **Documented vs estimated, method exposed** | **SlaveVoyages** | Publishes documented voyages **and** a separate Estimates interface (≈12.5M embarked / 10.7M arrived) with the imputation method exposed |
| **Evidence type per assertion** | **NPSpecies** (US NPS) | Each park × species row carries an **evidence type** and **park status**, so "present per a 1970s report" and "unconfirmed" are stored as separate evidence with provenance |
| **Reliability categories on survey estimates** | **African Elephant Database** | Classifies estimates as **definite / probable / possible / speculative** by survey method. *Task 1 flags this specific detail as high-confidence but not directly confirmed on-page* |
| **Two networks' overlap on the front page** | **biodiversity.aq** (SCAR) | Homepage counters are structured as **"occurrences in the SCAR network", "in the OBIS network", and "in both"** — displaying the disagreement rather than a merged total |
| **Submitted vs verified as two public numbers** | **FrogID** (Australia) | 930,680 calls submitted / 1,507,148 verified frogs, plus an openly declared validation backlog |
| **Dated legal determinations as a series** | **Chile species classification** | A species' status is **20+ dated, decree-backed determinations**, each with its public consultation record |
| **Two surveys shown disagreeing, no adjudication** | **NY Breeding Bird Atlas** | The interface's core function is showing Atlas 1 (1980–85) and Atlas 2 (2000–05) disagree for the same square |
| **Previous edition kept live** | **Sovon** (Netherlands) | 2012–15 atlas stays at its own subdomain while 2026–2030 fieldwork runs |
| **Published register of rejections** | **Digital Atlas of the Virginia Flora** | A maintained **"Excluded Taxa"** document — rejected records kept visible with reasons |
| **Refusal to resolve, stated** | **Native Land Digital** | Prominent disclaimers that boundaries are contested, overlapping and not authoritative |

### 4.1.6 Synthesis: six primitives, and a concrete schema

Everything above reduces to six design primitives. A disputed-figures feature that implements all six is, on the Task 1 evidence, **novel anywhere in Asia and probably novel in conservation informatics generally**.

1. **The assertion is the record, not the fact.** Store `(subject, property, value, source, date, method, rank)` — never a single `area_sq_km` column. *From: Wikidata, LPF `names[]`.*
2. **Rank, don't delete.** Three states — preferred / normal / **deprecated-but-retained**. *From: Wikidata.*
3. **Qualify with method.** The reason two figures differ is almost always methodological. Make `determinationMethod` mandatory. *From: OWID, African Elephant Database, Wikidata qualifiers.*
4. **Carry an explicit certainty value.** `certain / less-certain / uncertain`, plus a first-class value for *genuinely unknown*. *From: LPF `certainty`, Galápagos `cryptogenic`.*
5. **Keep the superseded version live and addressable.** *From: Sovon, Galápagos archived checklists, Nottingham + KEPN, legal citators.*
6. **Version the whole resource so any figure is citable as "as of vX".** *From: Fishes of Texas v3.10, FlorItaly 2025.2, Jepson Revision 11, MapBiomas Collections.*

A minimal working schema:

```
Assertion
  ├── about         → entity URI (place | species | population | boundary)
  ├── property      → e.g. "totalArea" | "tigerCount" | "toponym"
  ├── value         + unit
  ├── temporalScope → when the value applies (not when it was published)
  ├── statedBy      → source URI  (+ document scan / page image)
  ├── publishedOn   → date
  ├── method        → controlled vocab (camera-trap | DNA | pugmark |
  │                    interview | aerial | notification | informed-guess)
  ├── certainty     → certain | less-certain | uncertain
  ├── rank          → preferred | normal | deprecated
  ├── treatment[]   → supersedes / superseded-by / contradicts /
  │                    derived-from / criticised-by   (legal-citator pattern)
  └── note          → free text, why this rank
```

---

## 4.2 — Darwin Core and the other standards: what adoption actually costs

### 4.2.1 The honest headline

**Publishing occurrence data to GBIF requires four mandatory fields.** Not four hundred, not forty.

| Required | Meaning |
|---|---|
| `occurrenceID` | Unique identifier, stable across dataset versions |
| `basisOfRecord` | Observation / physical specimen / fossil / etc. |
| `scientificName` | Full name to the lowest rank you can supply |
| `eventDate` | ISO 8601 date or interval |

**Strongly recommended:** `countryCode`, `taxonRank`, `kingdom`, `decimalLatitude`, `decimalLongitude`, `geodeticDatum`, `coordinateUncertaintyInMeters`, `individualCount` / `organismQuantity` / `organismQuantityType`.

That is the entire realistic barrier for the occurrence layer. Darwin Core is large — TDWG organises it into 20+ classes (Record-level, Occurrence, Event, Location, Taxon, Identification, MaterialEntity, MaterialSample, Organism, GeologicalContext, MeasurementOrFact, ResourceRelationship, Media, Provenance, and more) with hundreds of terms — but **almost all of it is optional**, and the "Simple Darwin Core" profile is a flat table you can produce from a spreadsheet.

A **Darwin Core Archive** is just a zip: a core CSV, optional extension CSVs, a `meta.xml` describing the columns, and an EML metadata file. **The GBIF IPT generates all of this for you** from an uploaded table.

### 4.2.2 The real migration cost is not the occurrence table

The genuine cost is that **Darwin Core has no home for four of your five domains**:

| Your domain | Darwin Core coverage | Realistic standard |
|---|---|---|
| Species occurrence | **Full.** This is what DwC is | Darwin Core, exported via IPT |
| Place-name gazetteer | **None.** `locality`, `verbatimLocality` and `higherGeography` are free-text strings on an occurrence record, not a place model | **Linked Places Format** (§4.1.4) |
| Legal / government instruments | **None** | No standard exists. The corpus's working patterns are Flanders' **Inventaris Onroerend Erfgoed** (designation objects modelled as entities distinct from the things they cover), **Terras Indígenas'** seven legal phases, and **ECOLEX**'s treaty/legislation/court-decision catalogue |
| Historical documents | **None** | Dublin Core / **Omeka** — which is very likely what the adjacent **Keystone Foundation Archives** already runs on (Task 1 infers this from URL structure but flags it as **unverified**) |
| Literature | **None** | Standard bibliographic formats; **ZOBODAT** is the corpus's only project whose first-class entities are *the naturalist and the publication* |

So the answer to "is Darwin Core realistic for a project this size" is: **yes for one-fifth of the project, and it is cheap. It is not a schema for the atlas.**

### 4.2.3 Extensions worth knowing about

| Standard | Size / status | Why it matters here |
|---|---|---|
| **Humboldt Extension to Darwin Core (`eco:`)** | **51 terms**, version 2024-02-28, TDWG | Purpose-built for **ecological inventories and survey completeness** — six categories: survey description, geospatial/temporal, target scope, methodology, **data completeness**, environmental context. Terms include `samplingEffortValue` / `samplingEffortUnit` / `samplingEffortProtocol`, `isSamplingEffortReported`, `isTaxonomicScopeFullyReported`, `hasNonTargetTaxa`. **This is the standard that lets you express "this survey looked for X and did not find it" versus "this survey was not looking for X"** — the exact distinction that makes heterogeneous reserve surveys comparable, and the thing Biodiv'Écrins warns about in prose but cannot encode |
| **Camtrap DP** | Four files: `datapackage.json`, `deployments.csv`, `media.csv`, `observations.csv`; built on Frictionless Data; TDWG Machine Observations Interest Group; R package `camtrapdp` | Directly relevant to a **tiger reserve**. Camera-trap data is the dominant evidence type for tiger counts, and this is the standard that makes it exchangeable |
| **Audubon Core** | TDWG | Media (images, audio) metadata — relevant given the corpus's evidence that photo:observation ratios run ~24:1 (Trakuş) |
| **Latimer Core** | TDWG | Describing collections as objects |
| **Frictionless Data Package** | `datapackage.json` + CSVs + column schemas | **The lightest credible option.** If Darwin Core feels heavy, this is a plain-CSV-plus-JSON-schema convention that costs almost nothing and is what Camtrap DP is built on |

### 4.2.4 The recommendation, and the corpus evidence for it

**Hold your own internal model; treat Darwin Core as an export format.** This is what the durable projects in Task 1 actually do — Bhutan, TaiCOL, Hornbill Watch, NVS, BSPB and the Croatian, Taiwanese, Japanese, Swiss and Brazilian nodes all maintain their own systems and publish DwC through an IPT. **speciesLink/CRIA** is the extreme case: 24 years old, self-describes as *proprietary technology built for local needs* rather than adopting foreign platforms, and ships its own name-verification and coordinate-validation tooling.

The cost of DwC export, honestly stated: **a few days of mapping work for the occurrence layer, once.** The benefit is that your data survives your website — which §2.5 established is the thing that actually matters.

### 4.2.5 A finding that changes the options: India is not a GBIF Participant

This was not in Task 1 and it is consequential.

- **GBIF's own country page for India states: "This country is not a participant of GBIF."** India is listed as **"A GBIF Observer Country from Asia."**
- **GBIF's free Hosted Portal service** — a branded, fully customisable website displaying a targeted subset of GBIF-mediated data, offered **free of charge** — is available to "representatives of GBIF Participants and data publishing institutions endorsed by a node", with priority to **Voting Participant** nodes, and **national portals restricted to GBIF Participants**.
- Yet the **India Biodiversity Portal has been a GBIF publisher since 17 April 2019, with "GBIF India" listed as its endorsing node.**

These two facts are in tension and **the tension must be resolved directly before any plan depends on it.** The practical reading is: *publishing data* to GBIF through an endorsed Indian publisher appears possible; *getting a free GBIF-hosted portal* probably is not. If it were available it would be the single cheapest credible route to a public front end for the occurrence layer. It costs one email to `hostedportals@gbif.org` to find out.

---

## 4.3 — Realistic timelines for a solo builder or 2–3 person team

### 4.3.1 Ground truth from the corpus — what things actually took

No vendor claims. Every row is a published span from Task 1 or verified here.

| Project | Team | Elapsed | Output |
|---|---|---|---|
| **Coimbatore City Bird Atlas** | SACON + Bird Count India + local birders | **2 years** (2020–2022) | 37 grid cells of 3.3 × 3.3 km, 333 sub-cells, 111 surveyed |
| **Mysuru City Bird Atlas** | Small volunteer team | **2 years** (2014–2016) | Dataset with a Dryad DOI + *Indian BIRDS* 15(3) paper |
| **Hyderabad Bird Atlas** | 200–220 volunteers/season | **18 months → 4 seasons** (Feb 2025 – Jul 2026); **minimum 3-year commitment** | 248 unique species after four seasons |
| **NZ Bird Atlas** | Volunteer society on eBird | **Exactly 5.0 years** fieldwork (1 Jun 2019 – 31 May 2024) | 441,000 checklists, 309 species, 145,000 hours, **97.3% grid coverage** |
| **Kerala Bird Atlas 1.0** | Large consortium: Forest Dept + KAU + 14 district coordinators + hundreds of volunteers | **~5.6 years** (2015 → 13 Sep 2020), released 25 Jan 2021 | 4,324 sub-cells across 38,863 km²; six district atlases; *Current Science* + *Forktail* |
| **Virginia BBA2** | ~1,500 trained volunteers + paid staff | **10 years start to full web delivery**: fieldwork 2016–2020, results site Oct 2025, Phase Two Aug 2026 | 6M+ observations, 200+ species |
| **Ontario Reptile & Amphibian Atlas** | 12,000+ volunteers | **4 years from data close to book** (collection ended 2019 → book 2023) | 443 pp., 70+ maps |
| **Swiss Breeding Bird Atlas** | >2,000 field volunteers | Fieldwork closed 2016; **corrigendum June 2020** | ~200 species; **3.9 working-years** of fieldwork effort, 46,438 km travelled |
| **Caribbean Herpetology** | **4 named academics** | **16 years, 100 issues** (Aug 2010 → Jun 2026) ≈ **6.25 issues/year** | ~750 species accounts, ~2,000 images |
| **Papakilo Database** | OHA + contractor | **~10 years** (launched 4 Apr 2011 → 65 collections / 1M+ records by Dec 2020) | "Database of databases" across newspapers, land records, gazetteer, ethnobotany |
| **Biodiversity of Ice-free Antarctica** | AAD-led | **16 years of compilation** → published 2025 | 35,600+ records, 1,890 species, sources incl. herbaria and field notes |
| **Nimiarkisto** (Finland) | Kotus + CSC | Digitisation **completed for the 2017 centenary** | ~2.7M place-name cards, ~95% of traditional place names |
| **Toponimia de Galicia** | RAG + Xunta + public app | Years, **still incomplete** | >478,000 of >550,000 toponyms = **35% of territory** |
| **HGIS de las Indias** | Univ. Graz | **4 years funded** (FWZ 2015–2019), then rehomed to Yale | Gazetteer + administrative boundaries + time-slider WebGIS, 1701–1808 |
| **Biodiversity of India / Project Brahma** | Volunteer wiki | **Froze within ~2 years** | 664 species, 65 Bio Notes — then 15 years stale |
| **Flora-On** throughput benchmark | **~10-person core** | Ongoing | **1,664–2,060 records/week** (Aug 2026) |

### 4.3.2 What this means for one to three people

Three conclusions the table forces:

1. **A bounded single-taxon survey of a small area takes 2 years with a volunteer corps.** Coimbatore and Mysuru both did it. Neither had 2–3 people; both had a supporting network.
2. **A full multi-domain place atlas takes 10–16 years.** Papakilo, the Antarctic compilation, Caribbean Herpetology, Toponimia de Galicia. There is no counterexample in the corpus.
3. **The gap between "fieldwork complete" and "published on the web" is routinely 4–6 years** — Virginia (5 years from fieldwork close to a full results site), Ontario (4 years to book), Kerala (4 months, because they planned publication into the project). Kerala is the outlier and the model.

### 4.3.3 A realistic phased plan, with a publishable artefact at every phase

Framed against the corpus's evidence, not against what a platform vendor would promise. "Publishable" here means *citable by someone else*, which §3.2 established as the credibility threshold.

| Phase | Duration | Work | Publishable artefact | Precedent |
|---|---|---|---|---|
| **0. Scope and gap audit** | **6–8 weeks** | Write the inclusion rule. Enumerate what exists and what does not for the STR landscape. Do not build anything | A **published scope statement + gap audit**. This alone is citable | **Biodiversidata** built Uruguay's first consolidated dataset by "explicitly auditing and mapping the gaps first"; **Pladias** publishes an explicit scope rule |
| **1. Gazetteer skeleton** | **4–8 months** | Place-name layer in Linked Places Format, sourced from the Erode / Coimbatore / Salem district gazetteers, Survey of India sheets, revenue records and the Imperial Gazetteer via DSAL. Multiple names per place, each with citation and date. No preferred-name field | **A gazetteer dataset with a DOI.** This is the highest-value first deliverable because Task 1 confirms **no structured source exists** for these names | **World Historical Gazetteer / LPF**; **Toponimia de Galicia**; **Nimiarkisto**'s card-index-to-gazetteer conversion |
| **2. First citable artefact** | **months 6–12** | A short data paper describing the gazetteer, its method and its coverage gap. Deposit the data with a DOI | **A peer-reviewed data paper + a Zenodo/Dryad DOI** | **Mysuru City Bird Atlas** → Dryad; **Marine Biodiversity Portal of Bangladesh** → data paper; *Biodiversity Data Journal* is the obvious venue |
| **3. Legal-instrument layer** | **year 1–2** | Ingest **PARIVESH 2.0 bulk CSV from data.gov.in**, the 2008 and 2011 sanctuary GOs, the 2013 tiger-reserve GO, ESZ notifications, and FRA claim statistics. Model designations as **entities distinct from the places they cover** | A queryable register: "what instruments apply to this polygon" | **FWS ECOS/IPaC**; **EPBC PMST**; **Inventaris Onroerend Erfgoed**; **Terras Indígenas'** seven phases |
| **4. Disputed-figures layer** | **year 1–2, in parallel** | Implement the six primitives from §4.1.6 over the area and population figures you already have (§4.4.1) | **The differentiating feature.** Novel in Asia on Task 1's evidence | Wikidata ranks; OWID; legal citators |
| **5. Historical documents** | **year 2–4+** | ZSI Conservation Area Series and *Fauna of India* volumes; colonial forest working plans; DSAL gazetteer page images. Certainty ratings on every interpreted feature | Incremental, per-document | **SFEI Historical Ecology** — "the only rigorous archival-to-spatial workflow in this survey with a documented method"; **Nilgiri Archaeological Project** triangulating inscriptions, colonial herbaria, museum collections and oral history |
| **6. Occurrence** | **year 2–5, mostly by aggregation** | Aggregate eBird / iNaturalist / GBIF rather than collecting. Export DwC via IPT | A DwC occurrence dataset | **Ohio Odonata Society** model — delegate capture, keep synthesis |

### 4.3.4 What will not happen, stated plainly

- **You will not produce a comprehensive species inventory of Sathyamangalam in three years with 2–3 people.** Bush Blitz — a government + BHP + Earthwatch + Parks Australia partnership with a full institutional taxonomic team — recorded **683 species on King Island** and **1,301+ in the Pilliga** per expedition. Singapore's ~15,000 species for 730 km² took a small team many years on a far better-studied territory.
- **You will not out-collect eBird in this landscape.** Bird Gap Filling in Tamil Nadu is already targeting exactly these under-birded cells, and the Krishnagiri projects adjoin the STR landscape.
- **You will not get the reserve's best occurrence data.** M-STrIPES holds the patrol tracks, sign surveys and camera-trap workflow behind a login, and public release runs on the quadrennial All-India Tiger Estimation cycle.

None of this is discouraging; it is the case for the gazetteer + legal + disputed-figures triad, which nobody else is doing and which does not require out-collecting anyone.

---

## 4.4 — What validates and what contradicts the current approach

### 4.4.1 Strong validation: the disputed-figures problem is demonstrable on Sathyamangalam's own numbers

This is the most useful result in this task. **The reserve's published figures already contradict each other, in both dimensions the project names.**

**Area:**

| Figure | Source | Note |
|---|---|---|
| **1,408.40 km²** (core 793.49 + buffer 614.91) | Official STR site, `sathytiger.tn.gov.in/about` | Internally consistent; matches the 2013 G.O. Ms. No. 45 figure of **1,40,840 ha** |
| **1,411.60 km²** (524.34 in 2008 + 887.26 in 2011) | Official STR site, same page | The **sanctuary** total — 3.20 km² larger than the reserve, on the same page, unexplained |
| **1,408.6 km²** | Wikipedia, citing the 2013 notification | A third value |
| **1,408 km²** | Mongabay India, 2022 | A fourth, rounded |
| **524.3494 km²** → **1,411.6 km²** | Wikipedia, giving the 2011 expansion as **September 2011** | The official site gives the same expansion as **11 August 2011, G.O. Ms. No. 93** — a **date conflict on a legal instrument** |

**Tiger population — five different figures for 2011 alone, by four methods:**

| Year | Figure | Method / source |
|---|---|---|
| 2009 | **12** | Government of Tamil Nadu wildlife survey |
| 2010 | **12** *and* **46** | Both attributed to Government of Tamil Nadu surveys — a **3.8× discrepancy in the same year** |
| 2010 | **18** | Official STR site |
| 2011 | **19–25** | Camera-trap studies |
| 2011 | **"at least 25"** | WWF study |
| 2011 | **~30** | DNA-based pugmark analysis (69 of 150 samples positive) |
| 2011 | **28** | Camera-trap study, December 2011 |
| 2012 | **25** | National wildlife survey |
| 2018 | **80** | Reserve population assessment (Wikipedia) |
| 2018 | **87** | Mongabay, describing the TX2 achievement "based on the 2018 estimate" |
| 2022 | **"about 120"** | DFO Devendra Kumar Meena, quoted by Mongabay |
| 2024–25 | **112** | Official STR site, camera-trap census |

Note the last two rows: an official 2024–25 census figure (**112**) that is **lower than a 2022 official-source verbal estimate (about 120)**. Under a single-number presentation, one of these silently disappears. Under the §4.1.6 model, both survive with method, date, speaker and rank attached — and the *shape of the disagreement* becomes the finding.

**This is no longer a hypothetical feature. It is a documented defect in the public record of the specific place the project covers.**

### 4.4.2 Other things that validate the approach

1. **The multi-domain gap in India is real and Task 1 states it outright.** "Collectively the clearest evidence that India's small-project layer is single-taxon and occurrence-only. Nobody in this layer is doing multi-domain place documentation — that is the gap the user's atlas occupies."
2. **The gazetteer instinct is correct and underserved.** The Tamil Nadu District Gazetteers series exists with **no state-run searchable digital portal**, and Task 1 calls the Erode / Coimbatore / Salem volumes "the primary local-name authority for the STR landscape… not available as structured data — a clear build opportunity."
3. **Landscape-scoped (rather than administrative) atlases have strong precedent.** The Western Ghats Portal (a named biogeographic region on shared infrastructure, containing Sathyamangalam), Biodiv'Écrins, eElurikkus's per-national-park dossiers, Indian River Lagoon Species Inventory (one 156-mile estuary, 11,150 species reports), Lyudao LTSER (one island, ecological *and* social data).
4. **Legal instruments beside species has multiple working precedents** — Sri Lanka's CHM, EUNIS, Greece's Filotis, Georgia Biodiversity Portal, Pennsylvania NHP, SANBI's BGIS.
5. **Historical-document mining has a methodological precedent in the adjacent landscape.** The **Nilgiri Archaeological Project** (Ghent University) explicitly triangulates Old Kannada inscriptions, Old Tamil literature, contemporary oral histories, colonial herbaria, *Hortus Indicus Malabaricus* (1678–1693) and museum grave-goods collections in London, Chennai and Berlin.
6. **The small-team archetype is durable**, at 15–24 years for survivors (§2.6.2).
7. **The nearest structural analogue is 100 km away and run by a small team.** **Keystone Foundation Archives** (Nilgiri Biosphere Reserve): 464 images, 53 audio files, 22 videos, 503 text documents, 13 other items, themed across water, flora, wildlife, agriculture, biodiversity, indigenous cultures, land and livelihoods, ethnobotany — the closest existing project to what is being built, and a plausible partner rather than a competitor.

### 4.4.3 What contradicts or complicates the approach

**1. "One site" is contradicted by the corpus's strongest structural evidence.**
France's **INPN/OpenObs — the national aggregator — has been offline since a cyberattack in summer 2025, still down ~14 months later**, while the *regional* SILENE platform kept publishing throughout (latest update 27 January 2026). **ScotlandsPlaces** closed on 24 June 2025 and it was precisely the cross-institutional **integration layer** that died while all three partners' underlying material survived. **Alaska Center for Conservation Science** deliberately decomposed into separate thematic portals rather than one monolith. Task 1's verdict: *"the strongest available evidence for federating rather than centralising."*
**This does not mean abandon the single site.** It means the atlas should be a *view* over separable, independently publishable datasets — each of which can be deposited, harvested and survive the site.

**2. Building your own portal is contradicted by four working stacks.**
GeoNature (built by French national parks for exactly protected-area scale; Biodiv'Écrins is the working single-site instance), the Biodiversity Informatics Platform (Bhutan v5.0.5, and — critically — **Assam and the Western Ghats already run sub-national instances**), Symbiota, and Nunaliit (purpose-built for community-editable multimedia place atlases, and demonstrably transferable to the Global South via a short workshop, per the Pa Ipai deployment in Mexico).

**3. India's GBIF status closes the cheapest front-end route.** See §4.2.5. Verify before planning around it.

**4. The layer that makes the project distinctive is the layer that dies first.**
**Bayernflora**: the structured BIB database tools survived the November 2023 cyberattack; **the wiki — the narrative, historical-literature and discussion layer — did not, and remains unrestored ~3 years later.** Historical document mining and interpretive prose are the least backed-up, least standardised, least recoverable part of a multi-domain atlas. Whatever else gets a backup regime, that does first.

**5. Occurrence data may be a trap.**
It is the domain where you are weakest (no institutional data access, no volunteer corps yet), most duplicated (eBird, iNaturalist, Bird Gap Filling TN, the Krishnagiri projects), and least differentiated. Meanwhile M-STrIPES holds the reserve's actual monitoring data behind a login. Aggregate it; do not compete on it.

**6. The tribal-knowledge layer has three precedents that all say *document but do not publish*.**
**Sarawak Biodiversity Centre** runs Traditional Knowledge Documentation as a named institutional department **with a legal access-and-benefit-sharing mandate and no public-facing database**. **Inuit Heritage Trust** deliberately mediates its place-name data by request, tied to an official naming policy. **TKDL** went fully closed and was criticised for it in *Third World Quarterly*. Against these: **ANET** (Andamans) commits explicitly to "cognitive justice" — treating indigenous knowledge as a co-equal source — but **publishes no portal at all**.
This is not abstract for Sathyamangalam. There are **nine tribal settlements inside the reserve — seven in the core, two in the buffer** — Soliga and Oorali (Irula) communities, and **Tamil Nadu has distributed only 8,594 titles against 34,837 Forest Rights Act claims (~25%)**, against Kerala's 26,924 of 44,575. A knowledge layer here is a live legal matter, not a documentation exercise. The access-tier designs in §3.4.3 (PAS, ArtenFinder, SILENE) are the technical answer; consent and governance are not a technical question at all.

**7. "Comprehensive" is a trap, and every credible project in the corpus refuses it.**
Toponimia de Galicia leads with **35% coverage**. NZ Bird Atlas closed by publishing **97.3%**. BioNet declares itself "extensive, but nevertheless patchy." Biodiv'Écrins warns its data are "neither exhaustive nor complete." Publishing the gap is the credibility move; claiming completeness is what Wild Atlas does.

**8. The disputed-figures feature is commercially double-edged.** As noted in §3.4.2, knowing which figures are contestable is valuable to a clearance consultant as well as to a conservationist. Worth deciding deliberately, early, rather than discovering it later.

### 4.4.4 Unknowns to resolve before committing

A concrete checklist, each item a single targeted action:

| # | Question | Action |
|---|---|---|
| 1 | Is a free GBIF Hosted Portal available to an Indian project? | Email `hostedportals@gbif.org`; resolve the Observer-country / "GBIF India endorsing node" contradiction |
| 2 | Are the **Assam** and **Western Ghats** Biodiversity Informatics Platform instances live-but-empty? | Manual browser check — both returned HTTP 403 to automated fetch. A live-but-empty sub-portal would be an important negative finding |
| 3 | What platform do the **Keystone Foundation Archives** run on? | Ask them. Task 1 infers Omeka from URL structure but flags it unverified. They are the closest analogue and a plausible partner |
| 4 | Is **GeoNature** actively released in 2026, and what does deployment cost? | The project publishes a roadmap and holds a national meeting 5–6 November 2026; deploying-organisation count and hosting cost are not published anywhere reachable |
| 5 | Is the **Nunaliit** framework still actively developed? | 2,640 commits, latest commit date not obtainable. Matters if you consider adopting it |
| 6 | What became of **FES's 500,000+ citation bibliography** (India Observatory / IBIS, copyright 2006–2019)? | Ask FES. Task 1 calls it "the most valuable orphaned asset in Indian conservation informatics" |
| 7 | Does the **Wildlife Institute of India** hold a "National Wildlife Database"? | The WII page search engines associate with this name does not describe one. Significant unknown for a tiger-reserve atlas, since WII holds the All-India Tiger Estimation data |
| 8 | Does anyone in India currently sell biodiversity desk studies for EIA/clearance work? | No such service surfaced in Task 1 or Task 3. If genuinely absent, §3.4.2 is a live opportunity |
| 9 | What are **TIGERNET's** actual record counts and update cadence? | Robots-blocked to automated fetch. Manual check |
| 10 | Can the **FLAME University Gazetteer / Districts Project** be contacted? | July 2021 concept note, no published district volumes found. Directly overlapping ambition — worth a conversation before duplicating it |

---

## 4.5 — Summary of Task 4 findings

1. **The disputed-facts problem is solved, repeatedly, outside biodiversity.** Wikidata's preferred/normal/**deprecated** rank system with per-claim qualifiers and references is the single best model; OWID's definitional side-by-side and legal citators' preserve-and-flag treatment layer are the other two.
2. **Eighteen partial mechanisms already exist inside the biodiversity corpus** — TaiCOL's dual status fields, APNI/APC's published-vs-accepted split, BioModelos' competing hypotheses, Galápagos' `cryptogenic`, biodiversity.aq's three-way overlap counters — but **no project assembles them into a first-class disputed-figure object.** That combination remains novel.
3. **Six design primitives** cover it: the assertion is the record; rank rather than delete; qualify with method; carry explicit certainty; keep superseded versions live; version the whole resource.
4. **Darwin Core adoption costs almost nothing for the occurrence layer** — **four required fields**, and the IPT builds the archive. It is also **irrelevant to four of your five domains**, for which the answers are Linked Places Format (gazetteer), no standard (legal), Dublin Core/Omeka (documents), and bibliographic formats (literature).
5. **The Humboldt Extension (51 terms) is the standard worth actually learning**, because it encodes survey completeness, sampling effort and non-detection — the distinction between "looked and did not find" and "was not looking", which is what makes heterogeneous reserve surveys comparable. **Camtrap DP** matters for a tiger reserve specifically.
6. **India is not a GBIF Participant**, which probably closes the free hosted-portal route. Resolve before planning around it.
7. **Realistic timelines**: 2 years for a bounded single-taxon survey with a volunteer corps; **10–16 years for a full multi-domain place atlas**, with no counterexample in the corpus; 4–6 years is the routine gap between fieldwork ending and web publication, and Kerala's 4 months is the outlier worth imitating.
8. **A publishable artefact is available in 6–8 weeks** — a scope statement plus gap audit, following Biodiversidata — and a DOI'd gazetteer dataset within 12 months.
9. **The approach is validated on its own subject matter.** Sathyamangalam publishes at least four different area figures and twelve tiger-population figures across five methods, including a 3.8× discrepancy within a single year (2010: 12 vs 46) and a 2024–25 official census (112) lower than a 2022 official verbal estimate (about 120).
10. **The main contradictions are architectural, not conceptual**: federate rather than centralise; adopt a stack rather than build one; protect the narrative/historical layer hardest because it dies first; aggregate occurrence rather than compete on it; and treat the tribal-knowledge layer as a live legal matter under active FRA claims, not a documentation exercise.

---

## Sources — Task 4

**Primary evidence base**

- `Task1_Worldwide_Inventory.md` (25 August 2026); `Task2_How_It_Works.md`; `Task3_Win_And_Earn.md`

**Disputed-facts mechanisms (verified 26 August 2026)**

- [Wikidata — Help:Ranking](https://www.wikidata.org/wiki/Help:Ranking) and [Wikidata:Data model](https://www.wikidata.org/wiki/Wikidata:Data_model) — preferred / normal / deprecated ranks, claims, qualifiers, references
- [Our World in Data — How major sources collect data on conflicts and conflict deaths, and when to use which one](https://ourworldindata.org/conflict-data-how-do-researchers-measure-armed-conflicts-and-their-deaths) — six datasets compared side by side; UCDP ≥25 vs Correlates of War ≥1,000 deaths; best/low/high estimates; "no single best approach"
- [Our World in Data — How we choose which topics to work on, and which metrics to provide](https://ourworldindata.org/choosing-our-topics-and-metrics)
- [Chicago-Kent College of Law — Citators guide](https://guides.kentlaw.iit.edu/case-law/citators) — KeyCite flag taxonomy (yellow / red / red stripe / blue stripe / orange caution) and Shepard's signals (green plus / red stop / orange Q / yellow triangle / blue A / blue I); treatment categories
- [Westlaw — Checking Cases with KeyCite](https://legal.thomsonreuters.com/blog/westlaw-tip-of-the-week-checking-cases-with-keycite/)
- [Linked Places Format — specification README](https://github.com/LinkedPasts/linked-places-format/blob/main/README.md) — `names[]` with `citations[]` and per-name `when{}`; `GeometryCollection` with per-geometry `when` and `certainty`; `certainty` values certain / less-certain / uncertain; required fields
- [Linked Places Format (kgeographer)](https://kgeographer.org/linked-places-format/)

**Standards (verified 26 August 2026)**

- [GBIF — Data quality requirements: Occurrence datasets](https://www.gbif.org/data-quality-requirements-occurrences) — the four required Darwin Core fields (`occurrenceID`, `basisOfRecord`, `scientificName`, `eventDate`) and the strongly recommended set
- [TDWG — Darwin Core terms](https://dwc.tdwg.org/terms/) — class structure (Record-level, Occurrence, Event, Location, Taxon, Identification, MaterialEntity, and others)
- [GBIF IPT User Manual — Darwin Core Archives How-to Guide](https://ipt.gbif.org/manual/en/ipt/latest/dwca-guide) — archive structure (core CSV, extensions, `meta.xml`, EML)
- [Humboldt Extension for Ecological Inventories — vocabulary list of terms, 2024-02-28](https://eco.tdwg.org/list/2024-02-28) — **51 terms**, six categories, completeness and sampling-effort terms
- [TDWG — Humboldt Extension](https://www.tdwg.org/community/osr/humboldt-extension/) and [Enabling ecological survey data integration with the Humboldt Extension to Darwin Core, *Ecography*](https://nsojournals.onlinelibrary.wiley.com/doi/10.1002/ecog.08223)
- [Camtrap DP](https://camtrap-dp.tdwg.org/) — four-file structure, Frictionless Data basis, TDWG Machine Observations Interest Group, `camtrapdp` R package
- [Camtrap DP: an open standard for the FAIR exchange and archiving of camera trap data, *Remote Sensing in Ecology and Conservation* (2024)](https://zslpublications.onlinelibrary.wiley.com/doi/10.1002/rse2.374)
- [Frictionless Data Package specification](https://specs.frictionlessdata.io/)

**GBIF participation and hosted portals (verified 26 August 2026)**

- [GBIF — India country page](https://www.gbif.org/country/IN/summary) and [India participation page](https://www.gbif.org/country/IN/participation) — **"A GBIF Observer Country from Asia"; "This country is not a participant of GBIF"**
- [GBIF — Hosted portals](https://www.gbif.org/hosted-portals) — service description, eligibility, 12 named portals in production
- [GBIF — Terms and processes for hosted portal applications](https://www.gbif.org/article/49Ulh5tfiJyqifDpCKvtPO/terms-and-processes-for-hosted-portal-applications) — **free of charge**; open to Participant nodes and node-endorsed publishing institutions; priority to Voting Participants; **national portals restricted to GBIF Participants**; ~two-week review; `hostedportals@gbif.org`
- [GBIF — India Biodiversity Portal publisher page](https://www.gbif.org/publisher/62431bdf-fa0c-452b-afce-be14884a47ff) — publisher since **17 April 2019**, endorsing node **GBIF India**, 2 published datasets

**Sathyamangalam's own figures (verified 26 August 2026)**

- [Sathyamangalam Tiger Reserve — About](https://sathytiger.tn.gov.in/about) — total 1,408.40 km²; core 793.49 km² (79,349.331 ha); buffer 614.91 km² (61,491.210 ha); sanctuary Phase I 524.34 km² (3 Nov 2008, G.O. Ms. No. 122); Phase II 887.26 km² (11 Aug 2011, G.O. Ms. No. 93); combined sanctuary 1,411.60 km²; tiger reserve declared 15 Mar 2013, G.O. Ms. No. 45 (1,40,840 ha); tigers 18 (2010), **112 (2024–25 camera-trap census)**
- [Wikipedia — Sathyamangalam Tiger Reserve](https://en.wikipedia.org/wiki/Sathyamangalam_Tiger_Reserve) — sanctuary 524.3494 km² (2008), 1,411.6 km² after a **September 2011** expansion; tiger reserve **1,408.6 km²** (2013); tiger counts 12 (2009), 12 *and* 46 (2010), 19–25 / "at least 25" / ~30 / 28 (2011), 25 (2012), 80 (2018)
- [Mongabay India — Celebrating tiger numbers in Sathyamangalam Tiger Reserve, while tribal residents await their rights (May 2022)](https://india.mongabay.com/2022/05/celebrating-tiger-numbers-in-sathyamangalam-tiger-reserve-while-tribal-residents-await-their-rights/) — 87 tigers (based on the 2018 estimate); "about 120" per DFO Devendra Kumar Meena; 1,408 km²; **nine tribal settlements — seven core, two buffer**; Soliga and Oorali (Irula); Tamil Nadu **8,594 titles distributed of 34,837 claims (~25%)** vs Kerala **26,924 of 44,575**
- [Mongabay India — From reserved forests to protected area: How tiger numbers increased in Sathyamangalam (March 2018)](https://india.mongabay.com/2018/03/from-reserved-forests-to-protected-area-how-tiger-numbers-increased-in-sathyamangalam/)
- [PIB — All India Tiger Estimation 2022: Release of the detailed Report](https://www.pib.gov.in/PressReleaseIframePage.aspx?PRID=1943922&reg=3&lang=2)

**Explicitly unverified in Task 4**

- Whether a GBIF Hosted Portal is in fact obtainable for an Indian project (the Observer-country status and the existence of a "GBIF India" endorsing node are in tension)
- GeoNature's current release number, deploying-organisation count and hosting cost
- Nunaliit's latest commit date and 2026 development status
- Whether the Keystone Foundation Archives run on Omeka
- Live record counts and recency for the Assam Biodiversity Portal and The Western Ghats Portal (both HTTP 403 to automated fetch)
- TIGERNET record counts and update cadence (robots-blocked)
- The African Elephant Database's definite/probable/possible/speculative classification — widely cited in the literature, not confirmed on-page in Task 1
- Whether any Indian body currently sells biodiversity desk studies for EIA/clearance work
- The precise reason for the 3.20 km² difference between the sanctuary total (1,411.60 km²) and the tiger reserve total (1,408.40 km²) on the official STR site


---

# Round 2 — New Angles, New Keywords

**Subject:** A second research pass on the Sathyamangalam Tiger Reserve conservation atlas, searching in registers Round 1 never entered.
**Prepared for:** Vishnuvarthan Venkatapathy
**Compiled:** 26 August 2026
**Relationship to Round 1:** Round 1 (`Sathyamangalam_Atlas_Research_Report.md`, Tasks 1–4) remains the evidence base. Round 2 does not replace it. It adds to it, and in two places **corrects it**.

---

## Why a second round

Round 1 searched in **one register only**: English keywords, typed into web search, for things that describe themselves as atlases or portals. It was thorough within that register — 250+ projects individually verified — but every one of its six regional passes **exhausted its search budget**, four of them at 200/200 queries. Its own method notes name the consequences: Myanmar as "the largest verification gap", Central Asia as "the largest true white space in Asia", the Caribbean as "the largest known gap in this report", the Gulf and Levant entirely unsearched, 21 US states with no entry, and local-language searching never started for eight European countries.

Round 2 enters four registers Round 1 never used.

| Angle | Register | Result |
|---|---|---|
| **F** | India-specific verticals — statutory registers, epigraphy, land records, sacred sites, irrigation heritage, corridor atlases, Tamil-language archives | **Ingestible source layers**, plus three new live figure disputes in the reserve's own landscape |
| **A + B** | Non-English keywords, and the research literature treated as a directory rather than a verifier | Filled four of Round 1's named geographic gaps; produced the first correction |
| **C + D** | Infrastructure registries, code repositories, and funder project databases | The strongest discovery surface found in either round; produced the second correction |
| **E + G + H** | Adjacent domains (language-documentation archives), deliberate death-hunting, and re-checks of what Round 1 could not read | A purpose-built answer to the tribal-knowledge problem; link rot quantified; the Western Ghats Portal resolved |

---

## The ten things that changed

1. **267,608 People's Biodiversity Registers exist in India, only 11,951 are verified, and none are digitised.** Statutory under the Biological Diversity Act 2002, village-indexed, covering the reserve's own settlements, and already written. Nothing found in either round combines those properties. (§F.1)

2. **The disputed-figures feature now has five demonstration cases in Sathyamangalam's own landscape**, three of them new and two unresolved as of August 2026: reserve area (**five** figures now, a 2019 peer-reviewed paper adds 1,455 km²), tiger population (twelve figures, five methods), elephant corridors (**88 → 101 → 150**, plus a published methodological critique), Western Ghats Eco-Sensitive Area (**129,037 / 59,940 / 56,825 km²**, deadline extended to 2027), and sacred groves (**13,270 / ~14,000 / 100,000–150,000**). (§F.2, §F.3)

3. **Muthusankar Gowrappan, at the French Institute of Pondicherry, leads both the Historical Atlas of South India and the Western Ghats Portal.** The atlas's two hardest layers — an inscriptions-derived historical gazetteer and a Western Ghats biodiversity portal — have already been built once, for this region, by a contactable team. **This is the single most consequential finding across both rounds.** (§F.4, §EGH.1)

4. **The Western Ghats is not already documented.** The Western Ghats Portal was absorbed into the India Biodiversity Portal and holds "over 10,000 observations" for a 129,037 km², six-state landscape. Round 1's suspected negative finding is confirmed. (§EGH.1)

5. **Round 1's geographic "white spaces" were partly an artefact of English search.** Six of six native-language queries returned projects Round 1 had missed — including a live Kazakh government portal, closing what Round 1 called "the largest true white space in Asia." (§AB.1, §AB.3)

6. **Two corrections to earlier findings, one of them to my own Round 2 claim.** Costa Rica's CRBio is *offline*; a successor exists (CONAGEBIO's BiodataCR, ALA software, government domain) but shows blank counters and no news since October 2019. Round 1 overstated in one direction; my first Round 2 pass overstated in the other. §CD.0 has the corrected reading, and §AB.2 carries a warning pointing to it.

7. **Registries beat search, decisively.** One page — the Living Atlases participant list — produced seven projects new to the corpus plus the finding that **four Caribbean Living Atlases instances are all offline, all with the same single named contact, all on one university domain.** That is the clearest case of named-person dependency at regional scale in either round. (§CD.1)

8. **Mukurtu CMS is a purpose-built answer to the tribal-knowledge access problem** — communities hold cultural protocols, protocol stewards control membership, multi-protocol assignment gives per-item per-community visibility, and TK Labels assert community terms over material the community does not legally own. Better than any of Round 1's four access-tier precedents, and free. (§EGH.2)

9. **Link rot is now quantified**, and it validates Round 1's inferred lifespans: a **~14-year half-life** for scholarly links, a **9.3-year median page lifespan**, **only 62% of cited pages archived**, **49% of links in US Supreme Court opinions dead**, and **23% of COVID-19 state dashboards gone within two years**. The operational conclusion: **store the document, not the link.** (§EGH.3)

10. **Rufford's project database lists 662 funded projects in India**, browsable by country, each with a named grantee and a final report — a collaborator directory and a grey-literature source for this exact landscape, not merely a funding record. (§CD.4)

---

## What Round 2 did not do

Stated plainly, so the gaps are not mistaken for absences:

- **Myanmar, the Gulf states, Mongolia, most of the Caribbean, and 21 US states** remain untested in their own languages. They should not be called empty until they are.
- **The Internet Archive / Wayback Machine remains unavailable in this environment.** No project death in either round has been dated from an archived snapshot. Anyone continuing this work from a normal browser should re-date Round 1's dead-project entries.
- **The Assam Biodiversity Portal and TIGERNET remain unread** after a second attempt.
- **BISS/TDWG's 1,580 conference abstracts were identified as the sector's best untapped directory but not systematically mined** — that is a round of work in itself. (§AB.5)

---



---

# Round 2 · Angle F — India-Specific Source Layers

**Prepared for:** the Sathyamangalam Tiger Reserve conservation atlas project
**Compiled:** 26 August 2026
**What this angle is:** Round 1 searched India's *biodiversity portal* layer and concluded it is uniformly single-taxon and occurrence-only. It never searched India's other place-documentation verticals — statutory registers, epigraphy, land records, sacred sites, irrigation heritage, corridor atlases, or Tamil-language document infrastructure. This angle does that.
**Why it matters:** the other Round 2 angles find *comparators*. This one finds **source layers the atlas can actually ingest**, and it found three new live figure disputes in the reserve's own landscape.

---

## F.1 — The headline finding: 267,608 statutory village registers exist, and none are digitised

Round 1 missed the single largest place-based documentation programme in India.

Under the **Biological Diversity Act 2002**, every local body must constitute a **Biodiversity Management Committee (BMC)** and prepare a **People's Biodiversity Register (PBR)** — a village-level record of local flora, fauna, traditional knowledge of medicinal plants, local livelihoods and landscape.

| Figure | Value | Source / date |
|---|---|---|
| PBRs created nationwide | **267,608** | As of May 2023 |
| PBRs **verified** | **11,951** | ≈ **4.5%** of those created |
| Deadline set by the National Green Tribunal | **2020** | Missed |
| Digitisation status | **None** — "no digitalisation currently exists" | Experts proposed it; blocked by funding shortage |
| BMC composition (mandated) | Chairperson + up to 6 members; **at least one-third women, 18% SC/ST** | Biological Diversity Act framework |

Documented quality criticisms: rushed preparation (some completed in under six months), nodal agencies lacking verification expertise, insufficient funding, low public awareness, irregular updates — Bengaluru's 2009 register **remains unupdated**.

**Why this is the most important find in Angle F:**

1. **Every panchayat in and around Sathyamangalam has one.** The nine tribal settlements inside the reserve (seven core, two buffer) sit in local bodies that are legally required to hold a PBR.
2. **It is exactly the atlas's subject matter** — species + traditional knowledge + landscape, indexed by village.
3. **It is a statutory instrument**, so it belongs in the legal layer *and* the knowledge layer simultaneously.
4. **Nobody has digitised it.** A 4.5% verification rate on 267,608 documents is itself a disputed-figures object: "a PBR exists for this village" and "a *verified* PBR exists for this village" are different claims with a 20:1 gap between them.
5. **It carries the consent problem in acute form.** These registers were compiled from community knowledge without, in the documented Maduranthakam case, any benefit flowing back — medicinal plant experts collected common nettle knowledge and residents received no compensation. Round 1's access-tier precedents (Sarawak, Inuit Heritage Trust, ArtenFinder) apply directly.

**Action:** ask the Tamil Nadu Biodiversity Board for the PBR status of the panchayats bordering STR. This is a records request, not a research project.

---

## F.2 — Three new live figure disputes, all in Sathyamangalam's landscape

Round 1 established that the disputed-figures feature had a real target. Angle F found three more, and two are **unresolved as of August 2026**.

### F.2.1 — Elephant corridors: 88 vs 101 vs 150, with a published critique

| Count | Source | Year |
|---|---|---|
| **88 corridors** | *Gajah* report | 2010 |
| **101 corridors** | *Right of Passage: Elephant Corridors of India*, 2nd edition (Wildlife Trust of India) — "listed and mapped by elephant experts in consultation with all state forest departments" | 2017 / republished 2022 |
| **150 corridors** across 15 range states | *Elephant Corridors of India 2023*, MoEFCC / Project Elephant | 2023 |

And a formal published dispute: **Puyravaud, Davidar & Cushman (2024)**, "A critique of the Right of Passage as a guide to elephant corridors in India", *Conservation Science and Practice*. Their allegations:

- Right of Passage provides **no working definition of "corridor"**, making differentiation criteria impossible.
- It is "at best **a compilation of expert opinions with little coordination**, together with a few corridors that were validated using movement data and dung pile counts."
- In the **Nilgiri Biosphere Reserve** — Sathyamangalam's own landscape — "**only half of the corridors identified by the ROP were supported by comprehensive and consistent analyses**."
- The methodology ignores hierarchical importance: some corridors matter more than others.
- The 2017 revision "identified 101 corridors with **no clarification or improvement in the methodology**."

They recommend least-cost path modelling, resistant kernel methods, circuit theory and landscape genetics instead.

**Sathyamangalam relevance:** the 2023 government document names the **Mudahalli–Talavadi corridor in the BR Hills and Sathyamangalam landscape** among restored corridors, and the **Segur corridor** in the adjoining Mudumalai landscape. So the reserve sits inside a corridor set whose count has moved 88 → 101 → 150 and whose method is under published attack.

### F.2.2 — Western Ghats Eco-Sensitive Area: an unresolved boundary dispute, 15 years running

| Proposal | Area | Share | Year |
|---|---|---|---|
| **Gadgil Committee** (Western Ghats Ecology Expert Panel) | **129,037 km²** with differentiated zones | Effectively the whole Ghats | 2011 |
| **Kasturirangan Report** (High Level Working Group) | **59,940 km²** | **37% of the natural landscape** | 2012 |
| **Current draft notification** | **56,825 km²** | — | **July 2026** |

Status as of August 2026: the expert committee's deadline has been **extended to 2027**. Kerala's notified 9,993.7 km² "was based on the state's own proposition"; Goa's ESA covers 108 villages. Kerala, Goa and Karnataka have hardened political positions; **Maharashtra, Gujarat and Tamil Nadu show broad acceptance**.

This is the clearest possible demonstration case for the atlas feature: **one place, three official boundary figures, a 2.3× spread, a fifteen-year dispute, and no resolution.**

### F.2.3 — Sacred groves: a tenfold national dispute

| Figure | Basis |
|---|---|
| **13,270** documented | Malhotra et al. 1998, reproduced by CPREEC's ecoheritage inventory |
| **~14,000** reported | Commonly cited "reported" total |
| **100,000–150,000** estimated | Malhotra 1998, expert estimate of the true number |
| **1,275** in Tamil Nadu | CPREEC inventory, attributed to Malhotra et al. 1998 |

The gap between "documented" and "estimated" is roughly **10×**, and both figures trace to the same author. Tamil Nadu's groves are locally called **kovil kadu**; documented examples span Theni (Allinagaram), Sivagangai (Kandanur), Kanyakumari and Salem districts, ranging "from a few trees to hundreds of hectares." The earliest documented reference is the **Census report of Travancore of 1891**, citing Ward and Conner (1827).

**No searchable sacred-grove database was found for Tamil Nadu.** CPREEC's page reproduces a table but does not publish its own survey counts, districts covered, or a search interface.

---

## F.3 — A fifth area figure for Sathyamangalam, and the species-count vacuum

Round 1 found four total-area figures for the reserve. Angle F found a fifth.

| Figure | Source |
|---|---|
| 1,408.40 km² (core 793.49 + buffer 614.91) | Official STR site; matches G.O. Ms. No. 45 (2013), 1,40,840 ha |
| 1,411.60 km² | Official STR site — the *sanctuary* total |
| 1,408.6 km² | Wikipedia, citing the 2013 notification |
| 1,408 km² | Mongabay India, 2022 |
| **1,455 km²** | **Baranidharan, Vijayabhama & Bhuvensh (2019)**, *Journal of Entomology and Zoology Studies* 7(5) — used repeatedly throughout a peer-reviewed faunal survey |

**Five figures, a 47 km² spread, and the outlier is in the peer-reviewed literature.**

The same paper illustrates the species-count vacuum. Its own field surveys (Sept 2013, Dec 2013, Mar 2014; 25 transect lines, 250 sampling plots, 14 dam sites, pitfall traps) recorded:

| Taxon | Recorded in STR by this study | Tamil Nadu state total (cited in the same paper) |
|---|---|---|
| Mammals | **21** (15 herbivores, 5 carnivores, 1 omnivore) | 187 |
| Reptiles | **13** | 177 |
| Amphibians | **4** | 76 |
| Birds | not surveyed | 454 |
| Freshwater fish | not surveyed | 165 |

No butterfly or plant/tree counts were produced. A companion paper, "A contemporary assessment of tree species in Sathyamangalam Tiger Reserve, Southern India" (2017), covers the tree layer separately.

**Reading:** there is no consolidated species inventory for Sathyamangalam. What exists is a handful of single-taxon papers with incompatible area denominators. That is the gap the atlas fills, and it can be filled by *aggregation and reconciliation* rather than new fieldwork — which §4.3 of Round 1 identified as the only realistic route for a small team.

Two more current population figures for context, both Tamil Nadu-wide:

- **Nilgiri tahr: 1,364** (synchronised survey, Tamil Nadu — the state animal, and a Western Ghats endemic)
- **Wild elephants: 3,170** (Tamil Nadu, 2025)

---

## F.4 — The gazetteer layer: a direct precedent that Round 1 missed entirely

### Historical Atlas of South India (French Institute of Pondicherry)

This is the closest existing project to the Sathyamangalam gazetteer, and it is 400 km away.

| Attribute | Detail |
|---|---|
| Leads | **Subbarayalu Yellava** and **Muthusankar Gowrappan** |
| Started | **August 2005**; phase 1 ran **2005–2008**; "updating ongoing" as of late 2014 |
| Temporal scope | **Prehistory to 1600 CE** |
| Geographic scope | Full coverage of **Tamilagam** (Tamil Nadu and Kerala); partial Karnataka and Andhra Pradesh. Pilot: **Pudukkottai** |
| Source base | "Historical and geographical information collected from a **large corpus of south Indian inscriptions**, besides archaeological data collected from a series of **field surveys**, supplemented with data from **archaeological reports of ASI**" |
| Technology | Web GIS with database querying; W3C-compliant Graphics / Open GIS |
| Content | Maps, photographs, illustrations, texts, GIS functions |
| Download availability | **Not stated — unverified** |
| Current status | **Unverified.** The last activity indicator is late 2014. Given Round 1's evidence on university-hosted project decay, treat as possibly dormant until checked |

Round 1 already flagged IFP as a documented partner of the Western Ghats Portal and a participant in BIOTIK (whose TLS certificate has since lapsed). **IFP is therefore the single most important institution to contact**: it holds the inscription corpus, the historical GIS, a Western Ghats connection, and a track record of building exactly this kind of resource in exactly this region.

### DASI — Digital Archive of South Indian Inscriptions

| Attribute | Detail |
|---|---|
| Consortium | **Uppsala University** (Sweden), **Cologne University** (Germany), and South Asian scholars |
| Named team | R. Nagaswamy, A. Veluppillai, P. Schalk, Gunilla Gren-Eklund, Thomas Malten, Sascha Ebeling |
| Launched | **2002**, targeting Tamil inscriptions |
| Corpus scale | **~25,000 inscriptions estimated to exist in Tamil Nadu**; only **~6,000 published** as of a 1973 assessment |
| Pilot subset | Buddhist-related Tamil inscriptions |
| Data model | **SGML** with three-letter tags (`<NUM>`, `<LOC>`, `<DES>`) capturing location, dynasty, kings, dates, language, text; diacritics encoded numerically |
| Languages | Tamil in script and transliteration, Grantha distinguished by tag, Sanskrit |
| Public access | **No public URL or search interface found — unverified whether one exists** |
| Funding | Margot och Rune Johanssons Stiftelse, Uppsala |

**The 25,000 vs 6,000 gap is itself a documentation-coverage statistic worth publishing**, and inscriptions are the primary source of historical place names for this region — which is precisely what the Historical Atlas of South India was built on.

### Tamil-language document infrastructure

Round 1 found no Tamil-language digitisation layer at all. It exists and it is substantial.

| Project | What it is | Scale / status |
|---|---|---|
| **Noolaham Foundation** | Volunteer-run digital library of Sri Lankan Tamil texts — ethnographic material, manuscripts, books, multimedia. Project Noolaham launched **2005**, incorporated as a company **2010** | Platform evolved HTML → Joomla → **MediaWiki + Islandora**. Organised into **seven sectors and forty-five processes**; registered chapters in **Canada, UK, Norway**; contributors in Australia, US, Switzerland. Total document count **not published — unverified** |
| **Project Madurai** | Free electronic Tamil texts, from **1998** | The oldest of the lineage |
| **Tamil Virtual Academy** | Government of Tamil Nadu digital Tamil education and text repository | Scale unverified |
| **Tamil Digital Library** (AISLS) | American Institute for Sri Lankan Studies digitisation | Scale unverified |
| **Government Oriental Manuscripts Library and Research Centre** (Chennai) | Palm-leaf and paper manuscript collection | Scale unverified |
| **Tamil Heritage Foundation** | Manuscript digitisation | Scale unverified |
| Tamil manuscript digitisation rate | **Over 2.5 lakh (250,000) pages digitised in one year** | Reported 2019 |

**Noolaham's MediaWiki + Islandora stack is directly relevant**: it is a working, volunteer-run, Tamil-language, multi-format archive — the closest thing to a technology answer for the historical-document layer, and it was built by a diaspora volunteer community with no state funding.

---

## F.5 — Land, village and geospatial base layers

| Layer | What exists | Tamil Nadu coverage | Notes |
|---|---|---|---|
| **Local Government Directory (LGD)** | `lgdirectory.gov.in` — the canonical Government of India register of villages, panchayats and local bodies with stable codes; village datasets published on **data.gov.in** | Yes | **The correct spine for the gazetteer's modern-name layer.** LGD codes are the join key between census, revenue and administrative datasets |
| **DataMeet `indian_village_boundaries`** | Open village boundary geometries on GitHub | **Tamil Nadu not listed** among the states referenced (AP, Bihar, Gujarat, Haryana, Karnataka, Kerala, Goa, Telangana, Maharashtra) | A gap. TN village boundaries must come from another source or be assembled |
| **Bhuvan** (ISRO / NRSC) | National geoportal: thematic data dissemination, free GIS data, **OGC web services**, **clip-and-ship**, LULC and land-cover maps, IRS imagery, DEM, ortho | Yes | Layer counts and scales **not published — unverified**. Has a documented API. The obvious base-map and land-cover source |
| **CFR-Potential** (ATREE, `cfr.atree.org`) | Maps **CFR potential areas** (where Community Forest Resource rights could be claimed) against **CFR potential realised** | **Maharashtra, Jharkhand and Chhattisgarh only — Tamil Nadu not covered** | A directly analogous tool, built by an Indian institution, absent for the south. Village counts, data sources, build year and download availability all **unverified** from the tool page |
| **Sathyamangalam WLS Management Plan 2010–2020** | Tamil Nadu Forest Department management plan for the sanctuary | Direct | A primary document, circulating on Academia.edu. The equivalent 2020– plan was not located |
| **Tamil Nadu Archives and Historical Research** | The state's colonial-era record repository | Direct | Holds the forest, revenue and district records the historical layer needs. Digitisation status **unverified** |

---

## F.6 — Water and irrigation heritage: a documented landscape system with no atlas

Tamil Nadu's **eri** (tank) system is a place-based, historically documented, community-managed landscape infrastructure — and the documentation is fragmented.

- One commercial aggregator claims **119,797 lakes, ponds and tanks in Tamil Nadu** (a real-estate utility tool — **provenance unverified and the source should be treated with suspicion**).
- Academic work exists at cascade scale: a 2025 *Frontiers in Water* study of the **Mailam tank cascade**, socio-hydrology framing.
- Historical documentation exists in **mamulnamas** (customary water-management records) — studied in MIDS Working Paper 235.
- **Kudimaramathu**, the traditional community tank-maintenance practice, has been revived as a state scheme.

**No public inventory or atlas of Tamil Nadu's tanks with provenance was found.** This is a second clear build opportunity alongside the gazetteer, and it is exactly the kind of layer that carries competing counts.

---

## F.7 — What Angle F changes about the plan

1. **The PBR layer moves to the top of the source list.** It is statutory, village-indexed, already written, covers the reserve's own settlements, and is undigitised. Nothing else found in two rounds combines those four properties.
2. **Contact the French Institute of Pondicherry before building anything.** They have the inscription corpus, a working historical GIS for exactly this region, and a Western Ghats Portal relationship. Duplicating the Historical Atlas of South India would be the single most wasteful possible error.
3. **The disputed-figures feature now has five demonstration cases in the reserve's own landscape**: reserve area (five figures), tiger population (twelve figures, five methods), elephant corridors (88/101/150 plus a published critique), Western Ghats ESA (129,037 / 59,940 / 56,825 km², unresolved), and sacred groves (13,270 / ~14,000 / 100,000–150,000). That is enough to build and demonstrate the feature without needing a single new field observation.
4. **The gazetteer spine is LGD codes**, not village boundary shapefiles — because TN boundaries are not in the open dataset and LGD is the join key everything else uses.
5. **Noolaham's MediaWiki + Islandora is a proven Tamil-language archive stack** run by volunteers. It deserves evaluation alongside GeoNature, Omeka and Nunaliit.
6. **There is no sacred-grove and no tank inventory for Tamil Nadu.** Both are bounded, culturally legible, and would each make a standalone publishable dataset — the kind of 6-to-12-month first artefact Round 1's §4.3 recommended.

---

## Sources — Round 2, Angle F

**Statutory registers**

- [Mongabay India — Explainer: What is a People's Biodiversity Register? (July 2024)](https://india.mongabay.com/2024/07/explainer-what-is-a-peoples-biodiversity-register/) — 267,608 PBRs created as of May 2023; 11,951 verified; no digitisation; BMC composition; NGT 2020 deadline; quality criticisms; Maduranthakam case
- [National Biodiversity Authority — People's Biodiversity Register](http://www.nbaindia.org/content/105/30/1/pbr.html) — *redirects to nbaindia.nic.in; state-wise figures not retrieved, **unverified***
- [Tamil Nadu Biodiversity Board](https://x.com/TNBB_Secy) — no published PBR count located

**Elephant corridors**

- [Elephant Corridors of India 2023, MoEFCC / Project Elephant (PDF)](https://moef.gov.in/uploads/2023/11/PE-Elephant-Corridor-of-India-2023.pdf) — 150 corridors across 15 range states; Gajah report's 88; references the 2005 and 2017 Right of Passage editions; names the Mudahalli–Talavadi corridor in the BR Hills and Sathyamangalam landscape, and the Segur corridor
- [Wildlife Trust of India — Right of Passage: Elephant Corridors of India, 2nd edition](https://www.wti.org.in/resource_centre/right-of-passage-elephant-corridors-of-india-2nd-edition/) — 101 corridors, mapped by experts with state forest departments
- [Puyravaud, Davidar & Cushman (2024) — A critique of the Right of Passage as a guide to elephant corridors in India, *Conservation Science and Practice* (Oxford ORA copy)](https://ora.ox.ac.uk/objects/uuid:d636e2e2-2d2b-433c-a51e-fdebe954ac72/files/r2514nm77m) — no working definition of corridor; "a compilation of expert opinions with little coordination"; only half of ROP's Nilgiri Biosphere Reserve corridors supported by consistent analyses
- [Publisher version, *Conservation Science and Practice*](https://conbio.onlinelibrary.wiley.com/doi/full/10.1111/csp2.13212) — *HTTP 403 to automated fetch; content taken from the ORA copy*

**Western Ghats Eco-Sensitive Area**

- [Mongabay India — Why Western Ghats states can't agree on Eco-Sensitive Areas (August 2026)](https://india.mongabay.com/2026/08/why-western-ghats-states-cant-agree-on-eco-sensitive-areas/) — Gadgil 129,037 km²; Kasturirangan 59,940 km² / 37%; July 2026 draft 56,825 km²; deadline extended to 2027; Kerala 9,993.7 km²; Goa 108 villages; TN broad acceptance

**Sacred groves**

- [CPREEC ecoheritage — Sacred Grove](https://ecoheritage.cpreec.org/sacred-grove/) — 13,270 documented nationally; Tamil Nadu 1,275; attributed to Malhotra et al. 1998
- [Wikipedia — Sacred groves of India](https://en.wikipedia.org/wiki/Sacred_groves_of_India) — ~14,000 reported vs 100,000–150,000 estimated (Malhotra 1998)
- [FAO — The Spiritual, Socio-Cultural and Ecological Status of Sacred Groves in Tamil Nadu, India](https://www.fao.org/4/XII/0512-A1.htm) — kovil kadu; Theni, Sivagangai, Kanyakumari, Salem examples; size range; Census of Travancore 1891 / Ward and Conner 1827
- [Malhotra — Sacred Groves of India: An Annotated Bibliography (PDF)](https://sacredland.org/wp-content/uploads/2017/07/Malhotra_Sacred-Groves-of-India.pdf)

**Sathyamangalam-specific literature**

- [Baranidharan, Vijayabhama & Bhuvensh (2019) — Faunal diversity of Sathyamangalam tiger reserve, Tamil Nadu, *Journal of Entomology and Zoology Studies* 7(5) (PDF)](https://www.entomoljournal.com/archives/2019/vol7issue5/PartN/7-3-108-524.pdf) — area given as 1,455 km²; 21 mammals, 13 reptiles, 4 amphibians recorded; TN state totals cited
- [A contemporary assessment of tree species in Sathyamangalam Tiger Reserve, Southern India (2017)](https://www.researchgate.net/publication/320923119_A_contemporary_assessment_of_tree_species_in_Sathyamangalam_Tiger_Reserve_Southern_India)
- [Tamil Nadu Forest Department — Management Plan for Sathyamangalam Wildlife Sanctuary 2010–2020](https://www.academia.edu/8273064/TAMILNADU_FOREST_DEPARTMENT_MANAGEMENT_PLAN_FOR_SATHYAMANGALAM_WILDLIFE_SANCTUARY_2010_TO_2020_)
- [Tamil Nadu Forest Department — Sathyamangalam TR page](https://www.forests.tn.gov.in/pages/view/sathyamangalam_tr)

**Gazetteer and epigraphy**

- [French Institute of Pondicherry — Historical Atlas of South India v.1.0](https://www.ifpindia.org/projects/historical-atlas-of-south-india/) — Subbarayalu Yellava and Muthusankar Gowrappan; from August 2005; phase 1 2005–2008; prehistory to 1600 CE; built from south Indian inscriptions, field surveys and ASI reports
- [The Digital Archive of South Indian Inscriptions (DASI) — A First Report, OpenEdition/IFP](https://books.openedition.org/ifp/7841) — Uppsala + Cologne consortium; from 2002; ~25,000 inscriptions estimated in Tamil Nadu, ~6,000 published as of 1973; SGML data model; named team
- [Conserving Digital Archaeological Data in Tamil Nadu (IJSAT 2025, PDF)](https://www.ijsat.org/papers/2025/4/9039.pdf)
- [Tamil Nadu Archives and Historical Research](https://en.wikipedia.org/wiki/Tamil_Nadu_Archives_and_Historical_Research)

**Tamil-language document infrastructure**

- [Noolaham Foundation](https://en.wikipedia.org/wiki/Noolaham_Foundation) — Project Noolaham 2005, incorporated 2010; MediaWiki + Islandora; seven sectors and forty-five processes; chapters in Canada, UK, Norway; Project Madurai 1998, Eelanool 2004, E-Suvadi 2005
- [Tamil Virtual Academy](https://en.wikipedia.org/wiki/Tamil_Virtual_Academy) · [Tamil Digital Library (AISLS)](https://www.aisls.org/tamil-digital-library/) · [Government Oriental Manuscripts Library and Research Centre](https://en.wikipedia.org/wiki/Government_Oriental_Manuscripts_Library_and_Research_Centre) · [Tamil Heritage Foundation](https://en.wikipedia.org/wiki/Tamil_Heritage_Foundation)
- [DT Next — Over 2.5 lakh pages of Tamil manuscripts digitised in a year (2019)](https://www.dtnext.in/tamilnadu/2019/08/16/over-25-lakh-pages-of-tamil-manuscripts-digitised-in-a-year-)

**Land, village and geospatial layers**

- [Local Government Directory (LGD), Government of India](https://lgdirectory.gov.in/) · [LGD villages dataset on data.gov.in](https://www.data.gov.in/resource/local-government-directory-lgd-villages)
- [DataMeet — indian_village_boundaries, reference list](https://github.com/datameet/indian_village_boundaries/blob/master/website/docs/reference.md) — Tamil Nadu **not listed**
- [Bhuvan — Thematic Data dissemination (ISRO/NRSC)](https://bhuvan-app1.nrsc.gov.in/thematic/) · [Bhuvan API](https://bhuvan-app1.nrsc.gov.in/api/) · [Bhuvan free data download](https://bhuvan-app3.nrsc.gov.in/data/download/index.php)
- [CFR-Potential, ATREE](https://cfr.atree.org/potential/index.php) · [ATREE — Estimating and Mapping CFR Potential (report PDF)](https://www.atree.org/sites/default/files/reports/CFR_Potential_Mapping_Report_compressed.pdf) — Maharashtra, Jharkhand, Chhattisgarh only

**Water and irrigation heritage**

- [Frontiers in Water (2025) — Socio-hydrology and sustainable tank management: empirical case from a Mailam tank cascade, Tamil Nadu](https://www.frontiersin.org/journals/water/articles/10.3389/frwa.2025.1597293/full)
- [MIDS Working Paper 235 — Water management of two major system tanks according to mamulnamas (PDF)](https://www.mids.ac.in/assets/doc/WP_235.pdf)
- [India Water Portal — Cascading Tanks of Tamil Nadu](https://www.indiawaterportal.org/drinking-water/ancient-engineering-marvels-tamil-nadu)
- [Kudimaramathu Scheme](https://en.wikipedia.org/wiki/Kudimaramathu_Scheme)

**Other current figures**

- [Deccan Chronicle — Nilgiri Tahr population rises to 1,364 in TN](https://www.deccanchronicle.com/southern-states/tamil-nadu/nilgiri-tahr-population-rises-to-1364-in-tn-survey-1961620)
- [Deccan Herald — Tamil Nadu's wild elephant population rises to 3,170 in 2025](https://www.deccanherald.com/india/tamil-nadu/tamil-nadus-wild-elephant-population-rises-to-3170-in-2025-3755901)

**Traditional knowledge (Soliga)**

- [Frontiers in Conservation Science — Tiger Becomes Termite Hill: Soliga/Solega Perceptions of Wildlife Interactions and Ecological Change](https://www.frontiersin.org/journals/conservation-science/articles/10.3389/fcosc.2021.691900/full)
- [Madegowda & Rao — Traditional Ecological Knowledge of the Soliga Tribe (Antrocom, PDF)](https://antrocom.net/wp/wp-content/uploads/2024/05/madegowda-rao-traditional-ecological-knowledge-soliga-india.pdf)
- [Plant diversity and associated traditional ecological knowledge of the Soliga tribal community of BRT Tiger Reserve — "a biogeographic bridge for Western and Eastern Ghats"](https://www.researchgate.net/publication/303468517_Plant_diversity_and_associated_traditional_ecological_knowledge_of_Soliga_tribal_community_of_Biligiriranga_Swamy_Temple_Tiger_Reserve_BRTTR_A_biogeographic_bridge_for_Western_and_Eastern_Ghats_India)

**Unverified in Angle F**

- Current status of the Historical Atlas of South India (last activity indicator late 2014); whether its data is downloadable
- Whether DASI has a public search interface or URL
- State-wise PBR counts and any Tamil Nadu figure (NBA page redirected)
- Noolaham Foundation's total document/page count
- Bhuvan layer counts, scales and coverage
- CFR-Potential's village counts, data sources, build year and download availability
- Tamil Virtual Academy, Tamil Digital Library, GOML and Tamil Heritage Foundation collection sizes
- The 119,797 Tamil Nadu water-bodies figure (sourced from a commercial real-estate tool; **provenance not established — do not cite**)
- Whether a post-2020 Sathyamangalam management plan exists publicly
- Tamil Nadu Archives digitisation status


---

# Round 2 · Angles A + B — Other Languages, and the Research Literature as a Directory

**Prepared for:** the Sathyamangalam Tiger Reserve conservation atlas project
**Compiled:** 26 August 2026
**What these angles are:** Round 1 searched exclusively in English, via web search, for things that describe themselves as atlases or portals. Angle A searches in the languages the projects are actually named in. Angle B treats the academic literature as a *directory* of projects rather than as a source of verification.
**Headline:** both angles worked, and Angle A **overturned one of Round 1's most consequential conclusions**.

---

## AB.1 — The methodological finding: Round 1's "white spaces" were partly an artefact of English search

Every one of six non-English queries returned at least one project that Round 1 had either missed entirely or been unable to verify. The regions Round 1 declared empty were, in several cases, empty only of *English-language descriptions*.

| Region | Round 1's verdict | What a native-language query found |
|---|---|---|
| **Central Asia** | *"The largest true white space in Asia."* No national portal found for Kazakhstan, Kyrgyzstan, Uzbekistan, Tajikistan, Turkmenistan or Mongolia | **`biokadastr.kz`** — a live Kazakh government biodiversity database |
| **Türkiye** | TÜBİVES "status unverified — no canonical URL confirmed"; `bizimbitkiler.org.tr` timed out | **Nuh'un Gemisi Ulusal Biyolojik Çeşitlilik Veri Tabanı** — the national database, on a government domain, with a public statistics interface |
| **Indonesia** | Only INDOBIOSYS found, "discovery-oriented rather than place-documentation oriented" | **SIBIS** — the Ministry of Environment and Forestry's national biodiversity information system |
| **Francophone West Africa** | *"West African Bird DataBase"* named but never located; region effectively unrepresented | **Atlas de la biodiversité de l'Afrique de l'Ouest** — a multi-volume bilingual atlas from the BIOTA Afrique project |
| **Caribbean** | *"The largest known gap in this report"* | **Atlas de biodiversidad y recursos naturales de la República Dominicana** (2nd ed.) plus the DR environment ministry's Lista Roja de Especies |
| **China** | Catalogue of Life China unreadable — "TLS wrong-version error, checklist year and totals unverified" | Full annual figures, 2024 and 2025 editions, from the Chinese Academy of Sciences' own announcements |

**The lesson for the report as a whole:** an absence recorded in Round 1 should be read as *"not found in English"*, not *"does not exist"* — exactly as Round 1's own method notes warned, but the effect is larger than those notes implied.

---

## AB.2 — A correction that changes a Round 1 conclusion: CRBio (Costa Rica) is alive

> **⚠ THIS SECTION IS ITSELF CORRECTED IN ANGLE C+D, §CD.0. READ THAT FIRST.**
> The figures below come from the Living Atlases *participant detail page*, which describes CRBio at launch, not now. The Living Atlases **participants index** lists CRBio as **Offline**. A successor does exist — CONAGEBIO's **BiodataCR** on `biodiversidad.go.cr`, running ALA software — but its counters render blank and its most recent news item is **7 October 2019**. The accurate reading is *live-but-stale successor*, not *rehoming success*. §CD.0 sets out the full evidence.

This matters because Round 1 built one of its sharpest lessons on it.

**What Round 1 concluded:**

> "The retrospective presents **CRBio** as the successor national portal, integrating over 7 million occurrence records — and **CRBio's domain is now dead too**. So the mitigation failed as well… Anything that existed only as a website is gone from the live web."

**What is actually the case, verified 26 August 2026:**

| Attribute | Finding |
|---|---|
| Status | **Live and active** |
| Launched | **2006** |
| Rebuilt | **2016**, on **Atlas of Living Australia** software |
| Records | **"Nearly seven million georeferenced species occurrence records"**, drawn from **over 900 databases across 36 countries**, roughly half from Costa Rican institutions |
| Species pages | **More than 5,000** — vertebrates, arthropods, molluscs, nematodes, plants, fungi |
| Staff | **Four developers** |
| Operators | Biodiversity Informatics Research Center (CRBio) and the National Biodiversity Institute (INBio) |
| Live hosts | `portal.biodiversidad.go.cr`, `datos.biodiversidad.go.cr`, `geoespacial.biodiversidad.go.cr` — all responding (robots-disallowed, which is a live server refusing crawlers, not a dead one). Spatial Portal running **version 3.1.0** |
| Governance | A **Living Atlases** participant with a public **GitHub organisation** (`AtlasBiodiversidadCostaRica`) |

Round 1's DNS failure on `crbio.cr` was real, but it was the *old* domain. The project migrated to a **`.go.cr` government domain** and onto shared open-source infrastructure.

**This inverts the lesson.** Costa Rica is not the corpus's clearest case of mitigation failing. It is one of its better cases of **institutional rehoming succeeding**: INBio the organisation wound down, its specimens went to mandated institutions, and its portal moved onto a government domain and a maintained open-source stack — where it still serves seven million records with a four-person team. What died was a *domain name*, and Round 1 mistook that for the death of the resource.

The general lesson stands and is arguably strengthened: **register domains long, and never let a project's identity depend on one.** But "the mitigation failed as well" should be struck.

---

## AB.3 — Closing Round 1's "largest true white space in Asia"

### Kazakhstan — `biokadastr.kz`

| Attribute | Finding |
|---|---|
| Name | База данных Биоразнообразия — Biodiversity Database; the live instance is the "Интерактивный портал Биоразнообразие — Область Улытау" (Interactive Biodiversity Portal, **Ulytau Region**) |
| Operator | **Institute of Botany and Phytointroduction**, under the **Forest and Wildlife Committee**, **Ministry of Ecology and Natural Resources** |
| Languages | Russian, with Kazakh and English options |
| Scale | **750 registry records** |
| Composition | **699 vascular plants**, **51 plant communities** — and **zero** for fungi, algae, lichens, molluscs, arthropods, fish, reptiles, birds and mammals |
| Cadence | Site states "данные обновляются ежедневно" — **data updated daily** |
| Features | Interactive mapping, report generation, ecosystem profiles, spatial data layers |

**This is the negative finding Round 1 hypothesised but never confirmed.** Round 1 flagged, for the Assam and Western Ghats portals, that "a state/landscape sub-portal that is live-but-empty would be an important negative finding." Kazakhstan's is exactly that: a government-run, daily-updated, multilingual, well-featured portal holding 750 records, with nine of eleven taxonomic categories at zero.

**The transferable warning:** infrastructure quality and content quality are independent variables. A portal can be technically excellent, actively maintained, government-backed, and substantively empty — and from the outside it looks like a success. This is the strongest argument yet for Round 1's recommendation to **publish coverage gaps as headline numbers** (Toponimia de Galicia's "35% of the territory", NZ Bird Atlas's "97.3%").

### Türkiye — Nuh'un Gemisi (Noah's Ark)

| Attribute | Finding |
|---|---|
| Name | **Nuh'un Gemisi Ulusal Biyolojik Çeşitlilik Veri Tabanı** (Noah's Ark National Biological Diversity Database) |
| Host | `nuhungemisi.tarimorman.gov.tr` — a government domain |
| Operator | **DKMP** (General Directorate of Nature Conservation and National Parks), Ministry of Agriculture and Forestry |
| Public interface | A statistics page with filters on **Bölge** (region), **Şehir** (city), **Canlı Grubu** (organism group), **Tür** (species), **Endemizim** (endemism), **IUCN**, and **İzlenecek** (to be monitored) |
| Scale | **Counts not readable from the fetched page — unverified** |
| Documentation | Described in a published paper on monitoring Türkiye's biological diversity with geographic systems; announced publicly as "opened to the public" via a DKMP news item |

The **"İzlenecek" (to-be-monitored) flag as a first-class filterable field** is unusual and worth noting: it makes conservation-management priority a queryable attribute alongside taxonomy and IUCN status. This supersedes Round 1's unverified TÜBİVES lead as Türkiye's national resource.

### Indonesia — SIBIS

**Sistem Informasi Keanekaragaman Hayati Indonesia**, developed and operated by the **Ministry of Environment and Forestry (KLHK)**. Holds flora, fauna, ecosystems, conservation status and threats. Three declared components: a biodiversity database, a **monitoring system with periodic reporting**, and a public information portal with downloadable reports. **URL, launch date and record counts unverified.** Related: the Indonesian **Balai Kliring Keamanan Hayati** (Biosafety Clearing House) publishes an "Ikhtisar KEHATI 2024" national biodiversity overview.

---

## AB.4 — Figures Round 1 could not read

### Catalogue of Life China — now fully quantified

Round 1 recorded this as "fetch blocked (TLS wrong-version error) — checklist year and totals **unverified**." The Chinese Academy of Sciences publishes the figures directly.

| Edition | Total entries | Species | Infraspecific units | Change vs prior year |
|---|---|---|---|---|
| **2024** | **155,364** | — | — | — |
| **2025** | **162,717** | **148,341** | **14,376** | **+6,857 species, +496 infraspecific units** (animals +4,994, plants +458, fungi +1,405) |

Led by the **Institute of Zoology, Chinese Academy of Sciences** with partner institutions. The CAS announcement makes a claim worth recording: **"China is the only country that releases a biological species catalogue annually."**

**Why this matters for the atlas:** an annually versioned national checklist with a published year-on-year delta is the cleanest working example of Round 1's §4.1.6 primitive *"version the whole resource so any figure is citable as 'as of vX'"* — and the delta itself (+6,857 species in one year) is the object that makes taxonomic disagreement legible rather than invisible.

### Other regional finds

| Project | Detail |
|---|---|
| **Atlas de la biodiversité de l'Afrique de l'Ouest** | Multi-volume series from **Projet BIOTA Afrique** (offices Ouagadougou and Frankfurt/Main). Volume II covers **Burkina Faso**, by **Dorothea Kampmann and Adjima Thiombiano**, 2010, **xxxii + 592 pp.**, illustrated with maps, **bilingual French/English**, glossary and index, ISBN 978-3-9813933-4-7. **Print only — no digital edition found.** A partial fill for Round 1's francophone West Africa gap, and another instance of Round 1's "book-shaped output, thin web layer" failure mode |
| **Atlas de biodiversidad y recursos naturales de la República Dominicana** | 2nd edition, catalogued in the Dominican national archive and university libraries, and listed among the environment ministry's official publications. The ministry also publishes a **Lista Roja de Especies**. Print; digital availability **unverified**. A partial fill for the Caribbean gap |
| **Portal de Biodiversidad de Costa Rica — Spatial Portal** | `geoespacial.biodiversidad.go.cr`, running ALA Spatial Portal **v3.1.0** — spatial analysis, species search, occurrence download, dataset exploration, citizen-science contribution |

---

## AB.5 — Angle B: the literature is a directory, and it is large

### BISS / TDWG proceedings — the single best untapped discovery surface in this sector

*Biodiversity Information Science and Standards* publishes peer-reviewed extended abstracts from the field's main conferences. Round 1 cited it once, for a single Japanese paper. It is, in fact, a structured directory of active projects.

| Collection | Year | Abstracts |
|---|---|---|
| Living Data | 2025 | **100** |
| SPNHC-TDWG Joint Conference | 2024 | **142** |
| TDWG Proceedings | 2023 | **159** |
| TDWG Proceedings | 2022 | **144** |
| TDWG Proceedings | 2021 | **125** |
| TDWG Proceedings | 2020 | **80** |
| Biodiversity_Next | 2019 | **368** |
| SPNHC Proceedings | 2018 | **167** |
| TDWG Proceedings | 2018 | **142** |
| TDWG Proceedings | 2017 | **153** |
| **Total** | 2017–2025 | **1,580** |

Every abstract describes a working project, standard or tool, by named people, with an institution. **1,580 of them.** This is where projects of exactly the Sathyamangalam type announce themselves, and it is systematically searchable in a way that the open web is not.

### A quantitative finding on data papers that changes the credibility calculation

Round 1's §3.2 recommended publishing a data paper as a credibility act (Rung 1) and a citable record stream (Rung 2). A published analysis of publication trends across *Biodiversity Data Journal* and *Scientific Data* qualifies that advice:

- Article numbers in both journals are **"steadily increasing."**
- **31 countries or regions** have five or more data-paper articles; **38 have fewer than five.** The authors conclude there is a **"disequilibrium in the data sharing culture among different geographical regions."**
- **Citation performance is low.** In *Scientific Data*: **77 articles received 0–5 CrossRef citations; 10 received 5–10.** On Scopus: 53 with 0–5, 14 with 5–10.

**The implication is important and slightly uncomfortable.** A data paper buys **permanence and citability** — the record exists, is versioned, is indexed, and survives the website. It does **not** buy readership or citation impact. Round 1's advice to publish one stands, but the reason should be stated correctly: it is an **archival and provenance act**, not a visibility strategy. For visibility, Round 1's §3.1 adoption mechanisms (calendar anchoring, named contributor credit, administrative-unit rollups) are what actually work.

---

## AB.6 — What Angles A + B change

1. **Soften, do not strike, the Costa Rica case — see §CD.0 for the corrected version.** A successor portal exists (CONAGEBIO's BiodataCR, on a government domain, ALA software), so Round 1's "anything that existed only as a website is gone" overstates it. But BiodataCR shows blank counters and no news since October 2019, and CRBio itself is listed Offline by Living Atlases. This is Round 1's *silent stall* failure mode with a working server underneath — not a rehoming success.
2. **Central Asia is no longer a white space.** Kazakhstan has a live government portal — which is also the corpus's first *confirmed* live-but-nearly-empty portal (750 records, nine of eleven taxa at zero). That is a more useful finding than a new full portal would have been.
3. **Three more national systems exist** that Round 1 could not find or verify: Türkiye's Nuh'un Gemisi, Indonesia's SIBIS, and the West Africa atlas series.
4. **Add BISS/TDWG to the standing discovery method.** 1,580 peer-reviewed abstracts, 2017–2025, each describing a real project. No other single surface in this sector comes close.
5. **Restate why to publish a data paper.** It buys permanence and citability, not citations — 77 of 87 sampled *Scientific Data* articles had five or fewer citations.
6. **Adopt annual versioning with a published delta.** Catalogue of Life China is the working model: 155,364 (2024) → 162,717 (2025), with the +6,857 species change published as a figure in its own right.
7. **Treat every Round 1 "not found" as "not found in English."** Six of six language queries returned something Round 1 had missed. The geographic gaps are smaller than Round 1 believed, and the remaining ones (Myanmar, the Gulf, Mongolia, most of the Caribbean) should be re-tested in Burmese, Arabic, Mongolian and Spanish before being called empty.

---

## Sources — Round 2, Angles A + B

**Costa Rica correction**

- [Living Atlases — Atlas of Living Costa Rica (CRBio) participant page](https://living-atlases.gbif.org/participants/atlas_living_costa_rica/) — launched 2006, rebuilt 2016 on ALA software, ~7 million georeferenced records from 900+ databases across 36 countries, 5,000+ species pages, four developers, operated by CRBio and INBio
- [Portal de Biodiversidad de Costa Rica — Spatial Portal](http://geoespacial.biodiversidad.go.cr/) — ALA Spatial Portal v3.1.0, live
- [BiodataCR / Portal de Biodiversidad de Costa Rica](https://biodiversidad.go.cr/) · [portal.biodiversidad.go.cr](https://portal.biodiversidad.go.cr/) · [datos.biodiversidad.go.cr/collectory](https://datos.biodiversidad.go.cr/collectory/) — *live hosts; robots.txt disallows automated fetch, so record counts on these specific hosts are **unverified***
- [GBIF project page — CRBio: Atlas of Living Costa Rica](https://www.gbif.org/project/82989/crbio-atlas-of-living-costa-rica)
- [GitHub — AtlasBiodiversidadCostaRica](https://github.com/AtlasBiodiversidadCostaRica)
- [El Atlas de la Biodiversidad de Costa Rica (CRBio) — RedCLARA (PDF)](https://dspace.redclara.net/bitstream/10786/1295/1/El%20Atlas%20de%20la%20Biodiversidad%20de%20Costa%20Rica.pdf)

**Central Asia**

- [biokadastr.kz — База данных Биоразнообразия / Интерактивный портал Биоразнообразие, Область Улытау](https://biokadastr.kz/) — Institute of Botany and Phytointroduction under the Forest and Wildlife Committee, Ministry of Ecology and Natural Resources; 750 registry records (699 vascular plants, 51 plant communities, zero in nine other categories); daily updates; RU/KK/EN

**Türkiye**

- [Nuh'un Gemisi — Ulusal Biyolojik Çeşitlilik Veritabanı, statistics interface](https://nuhungemisi.tarimorman.gov.tr/public/istatistik) — *filter fields readable; counts **unverified***
- [DKMP — Yeni Nuh'un Gemisi Ulusal Biyolojik Çeşitlilik Veri Tabanı Halka Açıldı](https://www.tarimorman.gov.tr/DKMP/Haber/57/Yeni-Nuhun-Gemisi-Ulusal-Biyolojik-Cesitlilik-Veri-Tabani-Halka-Acildi)
- [Türkiye biyolojik çeşitliliğinin coğrafi sistemleri yardımıyla izlenmesi: Nuh'un Gemisi biyolojik çeşitlilik veritabanı (paper)](https://www.researchgate.net/publication/352686068_TURKIYE_BIYOLOJIK_CESITLILIGININ_COGRAFI_SISTEMLERI_YARDIMIYLA_IZLENMESI_NUH'UN_GEMISI_BIYOLOJIK_CESITLILIK_VERITABANI)
- [Türkiye Bitkileri Listesi — Nezahat Gökyiğit Botanik Bahçesi (bizimbitkiler.org.tr)](https://bizimbitkiler.org.tr/yeni/demos/technical/) — *host responded on this path, unlike Round 1's timeout*

**Indonesia**

- [SIBIS — Sistem Informasi Keanekaragaman Hayati Indonesia (overview)](https://environesia.co.id/blog/Sistem-Informasi-Keanekaragaman-Hayati-Indonesia-SIBIS) — KLHK-operated; database, monitoring with periodic reporting, public portal. *URL, launch date and counts **unverified***
- [Balai Kliring Keamanan Hayati — Ikhtisar KEHATI 2024](https://balaikliringkehati.kemenlh.go.id/ikhtisar-kehati-2024/)
- [The Conversation Indonesia — Membangun 'big data' keanekaragaman hayati kita](https://theconversation.com/membangun-big-data-keanekaragaman-hayati-kita-251145)

**China**

- [Chinese Academy of Sciences — 《中国生物物种名录2025版》发布](https://www.cas.cn/syky/202505/t20250522_5069672.shtml) — 162,717 total (148,341 species + 14,376 infraspecific units); +6,857 species and +496 infraspecific units over 2024; animals +4,994, plants +458, fungi +1,405; led by the CAS Institute of Zoology; "the only country that releases a biological species catalogue annually"
- [CAS — 《中国生物物种名录2024版》发布](https://www.cas.cn/sygz/202405/t20240523_5015532.shtml) · [National Forestry and Grassland Administration — 2024 version, 155,364 species and infraspecific units](https://www.forestry.gov.cn/lyj/1/dzbhml/20240524/664783.html)
- [中国科学院生物多样性委员会](http://www.cncdiversitas.cn/)

**West Africa and the Caribbean**

- [IUCN Library — Atlas de la biodiversité de l'Afrique de l'Ouest, tome II: Burkina Faso](https://portals.iucn.org/library/node/38912) — Projet BIOTA Afrique, 2010, Kampmann & Thiombiano, xxxii + 592 pp., bilingual FR/EN, ISBN 978-3-9813933-4-7, print
- [Atlas de la Biodiversité de l'Afrique de l'Ouest (full PDF, Univ. Frankfurt)](https://www.uni-frankfurt.de/47671163/CI_Atlas_complete.pdf)
- [Atlas de biodiversidad y recursos naturales de la República Dominicana, 2ª ed.](https://bvearmb.do/handle/123456789/232) · [Archivo General de la Nación catalogue record](https://colecciones.agn.gob.do/opac/ficha.php?informatico=00095121PI&codopac=OPPUB&idpag=2109221814)
- [Ministerio de Medio Ambiente y Recursos Naturales (RD) — Lista Roja de Especies](https://ambiente.gob.do/informacion-ambiental/lista-roja-de-especies/)

**Literature as a directory**

- [Biodiversity Information Science and Standards — Collections](https://biss.pensoft.net/collections) — 1,580 peer-reviewed extended abstracts across ten collections, 2017–2025
- [BISS journal home](https://biss.pensoft.net/) · [TDWG journal page](https://www.tdwg.org/journal/) · [Living Data 2025 conference](https://www.tdwg.org/conferences/2025/)
- [Analysis of publication trends of biodiversity data papers, *Biodiversity Science*](https://www.biodiversity-science.net/EN/abstract/abstract8942.shtml) — 31 countries/regions with ≥5 articles, 38 with <5; "disequilibrium in the data sharing culture"; Scientific Data citation distribution (77 articles at 0–5 CrossRef citations, 10 at 5–10)
- [Biodiversity Data Journal (Pensoft)](https://bdj.pensoft.net/journals.php?journal_name=bdj)

**Unverified in Angles A + B**

- Nuh'un Gemisi species, record and contributor counts (statistics page rendered filter fields only)
- SIBIS canonical URL, launch date and record counts
- Whether `crbio.cr` still resolves intermittently, and which of the `biodiversidad.go.cr` hosts is now canonical
- Record counts on `portal.biodiversidad.go.cr` and `datos.biodiversidad.go.cr` (robots-disallowed)
- Digital availability of the Dominican Republic atlas
- Whether the West Africa atlas series has volumes beyond Burkina Faso, and their coverage
- Total article counts and cumulative datasets mobilised across biodiversity data-paper journals (the trends paper gives distributions, not totals)
- Myanmar, the Gulf states, Mongolia and most of the Caribbean remain untested in Burmese, Arabic, Mongolian and Spanish respectively — **these should not be called empty until they are**


---

# Round 2 · Angles C + D — Code Repositories, Registries, and Funder Portfolios

**Prepared for:** the Sathyamangalam Tiger Reserve conservation atlas project
**Compiled:** 26 August 2026
**What these angles are:** Angle C searches the surfaces where projects exist as *infrastructure entries* rather than as websites — participant registries, hosted-portal directories, code repositories, data-repository registries. Angle D treats funders' own project databases as directories of what they funded.
**Headline:** the registries are the strongest discovery surface found in either round. The code surface is much weaker than expected. And this angle **corrects a claim I made in Angle A+B a few hours ago**.

---

## CD.0 — A correction to my own Angle A+B finding, before anything else

In Angle A+B I wrote that Costa Rica's CRBio is "live and active" and that Round 1's conclusion — "the mitigation failed as well" — should be struck. **That was too strong, and I got it from a page that is not a status report.**

The Living Atlases *participant detail page* for CRBio describes ~7 million records, 5,000+ species pages and four developers. But that page reads as written at or near launch, and the Living Atlases **participants index** — the current-status listing — puts **CRBio under "Offline."**

What is actually verifiable, 26 August 2026:

| Evidence | Reading |
|---|---|
| `crbio.cr` failed DNS in Round 1 | The CRBio-branded domain is gone |
| Living Atlases participants index lists **CRBio: Offline** | The CRBio instance is not running |
| `biodiversidad.go.cr` (**BiodataCR**) responds, and is **"based on the Atlas of Living Australia"** | A successor exists, on a government domain |
| BiodataCR is **"managed by the Technical Office of CONAGEBIO"** | Consistent with Round 1's finding that CONAGEBIO took over GBIF representation after April 2015 |
| BiodataCR's counters for *registros*, *juegos de datos*, *especies* and *instituciones* **render blank** | It is not displaying its own holdings |
| BiodataCR's **most recent news item is 7 October 2019** | Nearly seven years stale |
| `portal.`, `datos.` and `geoespacial.biodiversidad.go.cr` all respond (robots-disallowed), Spatial Portal at **v3.1.0** | The infrastructure is up |

**The corrected reading:** Round 1 was directionally right and wrong in detail. A successor portal *does* exist — CONAGEBIO's BiodataCR, on a `.go.cr` government domain, running ALA software — so "anything that existed only as a website is gone from the live web" overstates it. But BiodataCR is **live-but-stale**: blank counters, no news since 2019, and the CRBio brand itself listed offline. This is not a rehoming success. It is Round 1's *silent stall* failure mode (Task 2 §2.6.1, Stage 5) with a working server underneath.

**What I should have done:** treated a participant-registry description as a claim about launch, not about now, and checked the index before making the claim. Flagging it here rather than quietly amending, because the Angle A+B file has already been delivered and §AB.2 of it needs to be read against this section.

---

## CD.1 — Living Atlases participants: a 39-entry directory Round 1 never opened

Round 1 cited individual Living Atlases deployments (Sweden's SBDI, Austria, Croatia, Brazil's SiBBr) but never the participant list itself. It is a maintained, status-labelled directory.

| Status | Count | Members |
|---|---|---|
| **Live** | **22** | Atlas of Living Australia · **BioAtlas** (Croatia) · **Biodiversitäts-Atlas Österreich** · **GBIF Andorra** · **GBIF Benin** · **GBIF France** · **GBIF Portugal** · **GBIF Spain** · **GBIF Togo** · **Israel Citizen Science Center (SMNH)** · **Kew Data Portal** · **NBN Atlas** (5 variants) · **OpenObs** (France) · **Plateforme Régionale des Collections** (West/Central Africa) · **SNIBgt** (Guatemala) · **SiBBr** (Brazil) · **Swedish Biodiversity Data Infrastructure** · **TanBIF** (Tanzania) · **Vlaams Biodiversiteitsportaal** (Flanders) |
| **In development** | 3 | GBIF Chile · GBIF Luxembourg · NDFF (Netherlands) |
| **Offline** | 4 | **CRBio** (Costa Rica) · **Living Atlas of Barbados** · **Living Atlas of Suriname** · **Living Atlas of Trinidad & Tobago / The Caribbean** |
| **In discussion** | 9 | Chile · Habitat Foundation · Iberian Portal · Netherlands initiatives · Germany · New Zealand · Philippines · SCAR Antarctic · South Africa · Taiwan · USA Field Museum |
| **Moved off platform** | 1 | SiB Colombia |
| **Total** | **39** | |

### New to the corpus from this list alone

| Project | Why it matters |
|---|---|
| **SNIBgt** (Guatemala) — `snib.conap.gob.gt` | Round 1 explicitly listed Guatemala's CONAP/SNIBgt under *"search budget exhausted before discovery (never reached)."* It is **live**, administered by **CONAP** (National Council of Protected Areas), self-described as a "network of networks of providers, publishers and users", running **11 modules** (Collectory, Biocache, species, image hosting, spatial tools, user management), with named contacts (Melisa Ojeda, Hector Hernandez). **Record counts unverified** |
| **TanBIF** (Tanzania) | Round 1's Africa pass found nothing comparable for Tanzania beyond Serengeti research programmes |
| **GBIF Benin**, **GBIF Togo**, **Plateforme Régionale des Collections** (West/Central Africa) | Direct fills for Round 1's francophone West and Central Africa gap, where the "West African Bird DataBase" was named but never located |
| **GBIF Andorra** | A micro-state running a full national atlas stack — a useful scale comparator for a single reserve |
| **Vlaams Biodiversiteitsportaal** (Flanders) | Round 1 had `waarnemingen.be` (bot-blocked) but not the regional government portal |
| **Kew Data Portal** | An institution, not a country, running the same stack |

### The Caribbean finding: a four-instance regional cluster, all offline together

Round 1 called the Caribbean *"the largest known gap in this report"* and treated it as under-searched. The registry shows something more useful than a gap: **there was a four-instance Living Atlases deployment for the Caribbean, and all four are offline.**

- **Living Atlas of The Caribbean**, **Living Atlas of Barbados**, **Living Atlas of Suriname**, **Living Atlas of Trinidad & Tobago**
- All four registry entries give **Dimitri Ouboter** as the contact
- The Caribbean instance's URL was **`lac.uvs.edu`** — Anton de Kom University of Suriname
- All four entries have **"?" against code repository, documentation, Twitter account and modules available** — the registry itself never captured their technical details
- **No explanation for the offline status is given anywhere in the registry**

**This is the clearest instance in either round of named-person dependency operating at regional scale.** One person's contact details against four national instances, hosted on a single university domain, all down together, with no succession record and no archived documentation. It is Round 1's E-Flora BC and Alaska Native Place Names lessons — but multiplied across four countries.

---

## CD.2 — GBIF Hosted Portals: 12 in production, and one directly relevant to Sathyamangalam

Round 1 §4.2.5 established that this service is **free** but restricted to GBIF Participants and node-endorsed publishers — and that **India is not a GBIF Participant** ("This country is not a participant of GBIF"). The production list is nonetheless a directory of what the free tier actually produces:

**VertNet** · **GBIF France** · **European Journal of Taxonomy** · **Portal datos de biodiversidad de Argentina** · **Biodiversidad.co** (Colombia) · **GBIF.us** · **Swiss Natural History Collections** · **Pacific Biodiversity Information Facility** · **DiSSCo UK** · **Legume Data Portal** · **SCAR Antarctic Biodiversity Portal** · **VectorNet Data Portal** — with more beyond the featured twelve.

Two entries resolve Round 1 problems:

- **Portal datos de biodiversidad de Argentina** — Round 1 recorded Argentina's SNDB as *"host resolves but all connection attempts failed… possibly down or firewalled."* A working Argentine national portal exists on GBIF's hosted service.
- **Biodiversidad.co** (Colombia) — Round 1 recorded SiB Colombia returning **corrupted placeholder counters** ("repeated 100,000 across unrelated counters"). The hosted-portal instance is the readable one.

And one is a scale model worth studying: the **Legume Data Portal** is a *thematic* hosted portal — not a country, not an institution, a taxon. That is the closest structural precedent for a **place-scoped** hosted portal, and it is the shape a Sathyamangalam portal would take if the eligibility question in Round 1 §4.4.4 (item 1) resolves favourably.

---

## CD.3 — Angle C's negative finding: the code surface is thin

I expected GitHub topics to reveal a hidden layer of atlas software, on the strength of Round 1's evidence that projects like `vncreatures` and the Chinese solo-taxon cluster are invisible to search. **They did not.**

The `biodiversity-data` GitHub topic returns mostly zero-star repositories. What is there is small and specific:

| Repository | What it is |
|---|---|
| `niconoe/arabel` | "Source code for the future **Atlas of spiders of Belgium**", Python — a single-taxon regional atlas being built in the open |
| `nealdoran/biodiversity-gap-audit` | Python ETL pipeline and Streamlit dashboard **auditing 26 million GBIF occurrence records against IUCN Red List criteria** |
| `aminem0/dwc-owl` | An effort to build an **OWL ontology from Darwin Core terms** |
| `galerucinae/galerucinae.github.io` | Galerucinae of the World Digital Catalogue |
| `AtlasBiodiversidadCostaRica` | The CRBio organisation (see §CD.0) |

**The finding is the absence.** This sector does not distribute itself through GitHub topics. The major open-source stacks Round 1 identified — Symbiota, Living Atlases, GeoNature, Nunaliit, the Biodiversity Informatics Platform — are found through **institutional registries and participant lists**, not through code-topic browsing. Anyone hunting for atlas software should search the registries first and GitHub second.

One exception worth noting for the atlas's own build: `biodiversity-gap-audit` demonstrates that a **26-million-record GBIF audit against IUCN criteria** is a one-person Python-plus-Streamlit project. That is a realistic scale marker for what a small team can do with aggregated data rather than collected data — which is the route Round 1 §4.3 recommended.

---

## CD.4 — Angle D: Rufford's project database is a 662-entry directory of Indian conservation work

Round 1 §2.2.1 verified Rufford's *grant tiers* (£7,000 / £8,000 / £12,000 / £18,000, sequential, one application per 12 months). It never opened the project database those grants produced.

| Attribute | Finding |
|---|---|
| Rufford projects listed **for India** | **662**, across **27 pages** |
| Browsable by | **Country** (`/projects/country/IN/`) and by **category** |
| Each entry gives | Grantee name, project title, location, and a project page |

Western Ghats work is well represented in the first page alone — Anand M. Osuri (native tree nurseries and rainforest restoration in the Western Ghats), Rengaian Ganesan (degraded forest, south Western Ghats), Devika M. Anilkumar Madathil (ecorestoration and IUCN assessments of threatened trees endemic to the Western Ghats), Sushanth S (non-volant small mammals in Western Ghats plantations, Kerala), Jithu K. Jose (Western Ghats tree ecology) — plus Tamil Nadu-located work (Brittany Bartlett, small-scale fisheries).

**Why this matters practically:**

1. **It is a named-person directory of exactly the collaborator pool a Sathyamangalam atlas would draw on** — early-career Indian conservation researchers, working at the right scale, in the right landscape, who have already produced a funded project with a report.
2. **Each Rufford project produces a final report**, which is grey literature about specific places — precisely the material the atlas's literature layer is for, and it is not indexed in the journals Angle B covers.
3. **It is the funding ladder Round 1 costed.** Co's Digital Flora of the Philippines — three editors, 10,200 species, 146,000 photos — was Rufford-funded in part. 662 Indian grantees have been through the same door.

**Unverified:** how many of the 662 are Tamil Nadu-located, how many are documentation or database projects rather than field ecology, and whether final reports are downloadable in bulk.

---

## CD.5 — What Angles C + D change

1. **Read §AB.2 against §CD.0.** CRBio is offline; CONAGEBIO's BiodataCR is the live-but-stale successor, blank counters, no news since 2019. Round 1 overstated; my Angle A+B correction also overstated in the other direction. The accurate version is in §CD.0.
2. **Registries beat search, decisively.** The Living Atlases participant list alone produced seven projects new to the corpus plus the Caribbean cluster finding — from one page. Round 1 spent six regional passes and 200-query budgets to build its list; a maintained registry would have been faster and more accurate.
3. **The Caribbean is not a gap; it is a failure.** Four Living Atlases instances covering the Caribbean, Barbados, Suriname and Trinidad & Tobago are all offline, all with the same single contact, all hosted at one university domain, with no documentation captured and no explanation recorded.
4. **Guatemala's SNIBgt is live** — closing another Round 1 "never reached" item, and it runs 11 modules under a protected-areas agency, which is the same institutional shape as a forest-department-hosted reserve atlas.
5. **The GitHub layer does not exist for this sector.** Search registries first. The one useful repository found (`biodiversity-gap-audit`) is a scale marker: 26M GBIF records audited against IUCN criteria by one person in Python and Streamlit.
6. **Rufford's 662 Indian projects are a collaborator directory, not just a funding record.** For a project that needs field partners and grey literature in this exact landscape, it is more immediately useful than any journal index.

---

## Sources — Round 2, Angles C + D

**Registries and participant directories**

- [Living Atlases — participants](https://living-atlases.gbif.org/participants/) — 39 participants; 22 live, 3 in development, 4 offline, 9 in discussion, 1 moved
- [Living Atlases — Atlas of Living Costa Rica (CRBio)](https://living-atlases.gbif.org/participants/atlas_living_costa_rica/) — *launch-era description: ~7M records, 5,000+ species pages, four developers, rebuilt 2016 on ALA. **Listed as Offline in the participants index** — see §CD.0*
- [Living Atlases — SNIBgt (Guatemala)](https://living-atlases.gbif.org/participants/snibgt/) — CONAP-administered, `snib.conap.gob.gt`, 11 modules, named contacts; **record counts unverified**
- [Living Atlases — Living Atlas of The Caribbean](https://living-atlases.gbif.org/participants/living-atlas-caribbean/) — `lac.uvs.edu`, contact Dimitri Ouboter, code/documentation/modules all recorded as "?"
- [Living Atlases — Living Atlas of Suriname](https://living-atlases.gbif.org/participants/living-atlas-suriname/) · [Barbados](https://living-atlases.gbif.org/participants/living-atlas-barbados/) · [Trinidad & Tobago](https://living-atlases.gbif.org/participants/living-atlas-trinidad-tobago/)
- [Living Atlases](https://living-atlases.gbif.org/)
- [GBIF — Hosted portals](https://www.gbif.org/hosted-portals) — 12 featured production portals including Pacific BIF, Legume Data Portal, Argentina, Biodiversidad.co and SCAR Antarctic
- [re3data.org — browse by subject](https://www.re3data.org/browse/by-subject/) · [re3data about](https://www.re3data.org/about)

**Costa Rica status resolution**

- [BiodataCR / biodiversidad.go.cr](https://biodiversidad.go.cr/) — "an information system aimed at systematizing, documenting and publishing information about the biodiversity of Costa Rica"; **managed by the Technical Office of CONAGEBIO**; "based on the Atlas of Living Australia"; **counters render blank; most recent news item 7 October 2019**
- [Portal de Biodiversidad de Costa Rica — Spatial Portal](http://geoespacial.biodiversidad.go.cr/) — ALA Spatial Portal v3.1.0

**Code repositories**

- [GitHub topic — biodiversity-data](https://github.com/topics/biodiversity-data) — `niconoe/arabel` (Atlas of spiders of Belgium), `nealdoran/biodiversity-gap-audit` (26M GBIF records audited against IUCN Red List criteria), `aminem0/dwc-owl`, `galerucinae/galerucinae.github.io`
- [GitHub — AtlasBiodiversidadCostaRica](https://github.com/AtlasBiodiversidadCostaRica)
- [GitHub topic — biodiversity](https://github.com/topics/biodiversity) · [GitHub topic — species](https://github.com/topics/species)

**Funder portfolios**

- [The Rufford Foundation — Projects by country: India](https://www.rufford.org/projects/country/IN/) — **662 projects across 27 pages**
- [The Rufford Foundation — Project Directory](https://www.rufford.org/projects/124/)
- Example Western Ghats grantees: [Anand M. Osuri](https://www.rufford.org/projects/anand-m-osuri/partnering-with-private-landowners-to-expand-native-tree-nurseries-and-restore-tropical-rainforests-in-indias-western-ghats-biodiversity-hotspot/) · [Rengaian Ganesan](https://www.rufford.org/projects/rengaian-ganesan/degraded-forest-south-western-ghats-india/)
- [Rufford — grant criteria and tiers](https://apply.ruffordsmallgrants.org/help/criteria)

**Unverified in Angles C + D**

- Record counts for SNIBgt, TanBIF, GBIF Benin, GBIF Togo, Plateforme Régionale des Collections, GBIF Andorra, Vlaams Biodiversiteitsportaal
- Why the four Caribbean Living Atlases instances went offline, and when
- Whether `lac.uvs.edu` retains any archived content
- The full GBIF hosted-portals list beyond the 12 featured
- How many of Rufford's 662 Indian projects are Tamil Nadu-located, and how many are documentation/database rather than field projects
- Whether Rufford final reports can be retrieved in bulk
- Current record counts on BiodataCR (counters render blank; robots.txt blocks the sub-portals)


---

# Round 2 · Angles E + G + H — Adjacent Domains, Deaths, and Gap-Fill

**Prepared for:** the Sathyamangalam Tiger Reserve conservation atlas project
**Compiled:** 26 August 2026
**What these angles are:** Angle E searches domains Round 1 never entered — chiefly language-documentation archives, which hold the most mature consent and access models anywhere. Angle G hunts dead projects deliberately and looks for *quantitative* failure evidence, which Round 1 had to infer. Angle H re-checks the specific things Round 1 could not read.
**Headline:** the tribal-knowledge access problem has a purpose-built, funded, open-source answer; link rot is now quantified; and the Western Ghats Portal — Sathyamangalam's own landscape portal — is fully resolved, with a consequential name attached.

---

## EGH.1 — Angle H's most important result: the Western Ghats Portal, resolved

Round 1 flagged this as a priority unknown: *"Assam Biodiversity Portal and The Western Ghats Portal activity levels. Both returned HTTP 403 to automated fetching… observation counts and most-recent-observation dates are unverified — which matters, because a state/landscape sub-portal that is live-but-empty would be an important negative finding."*

The French Institute of Pondicherry's own project page answers it.

| Attribute | Finding |
|---|---|
| **Project lead** | **Balasubramanian Dhandapani** |
| **Team** | **Ayyappan Narayanan**, **Muthusankar Gowrappan** |
| **Contact** | `ramesh.br [at] ifpindia [dot] org` |
| **External collaborators** | **Prabhakar R** (Director, Strand Life Sciences) and **Thomas Vattakaven** (Strand Life Sciences) |
| **Start** | **November 2010** |
| **Funder** | **Critical Ecosystem Partnership Fund (CEPF)** |
| **Status** | Ongoing |
| **Architecture** | The project **created the India Biodiversity Portal, which absorbed the earlier Western Ghats Portal** — maps, species pages, citizen science, documents, groups infrastructure |
| **Holdings** | **"Over 10,000 observations from the Western Ghats"**, plus species pages, maps and documents |
| **Governance** | IBP is managed by **a collective of around 17 organisations** forming the IBP consortium |
| Portal About page timestamp | **31 December 2012** |

**Two conclusions follow.**

**First — Round 1's hypothesis is confirmed.** The Western Ghats Portal is a landscape sub-portal holding roughly **10,000 observations for a landscape of 129,037 km² spanning six states**. That is very thin. It is not empty, but it is nowhere near a comprehensive record of the Western Ghats, and it is not an independent portal at all — it was absorbed into IBP. **Sathyamangalam is not already covered by an existing landscape atlas.** The gap Round 1 identified is real.

**Second — the most important contact across both rounds has surfaced.** **Muthusankar Gowrappan appears on the team of both the Western Ghats Portal *and* the Historical Atlas of South India** (Angle F, §F.4) — the same person, at the same institute, on both a biodiversity portal and a historical GIS gazetteer, for the same region.

That combination — inscriptions-derived historical geography **and** a Western Ghats biodiversity portal, under one roof, with named people still contactable — is exactly the Sathyamangalam atlas's two hardest layers, already built once by the same team. **The French Institute of Pondicherry should be contacted before any build decision is made.** Round 1 already had IFP as a Western Ghats Portal partner and as a BIOTIK participant; Round 2 shows it is far more central than that.

### Still unresolved from Round 1's blocked list

| Item | Status after re-check |
|---|---|
| **Assam Biodiversity Portal** | Still **HTTP 403** to automated fetch. Its taxonomy and document-list URLs resolve in search results, so the portal is live; **counts remain unverified** |
| **Internet Archive / Wayback Machine** | **Still unavailable in this environment** (`PROVENANCE_REQUIRED` on fetch). Round 1's inability to date project deaths from snapshots persists into Round 2 |
| **TIGERNET** | Not re-attempted; Round 1's robots block stands |

---

## EGH.2 — Angle E: the tribal-knowledge access problem has a purpose-built answer

Round 1 collected four access-tier precedents — the Portable Antiquities Scheme's account-based findspot precision, ArtenFinder's private-until-released, SILENE's dual interfaces, and Sarawak Biodiversity Centre's documented-but-not-browsable traditional knowledge. All four are *adaptations* of general systems.

**Mukurtu CMS is not an adaptation. It was built for this problem from the start**, and Round 1 mentioned it only once, in passing, as an unverified implementation directory.

| Attribute | Finding |
|---|---|
| Built by | **Center for Digital Scholarship and Curation, Washington State University** |
| Origin | **2007**, the Mukurtu Wumpurrarni-kari Archive, with the **Warumungu community**, **Kim Christen** and **Craig Dietrich** |
| Licence | Open source |
| Funders | **National Endowment for the Humanities**, **Institute of Museum and Library Services**, **Andrew W. Mellon Foundation**, **National Science Foundation** |
| Self-description | "A grassroots project aiming to empower communities to manage, share, narrate, and exchange their digital heritage in culturally relevant and ethically-minded ways" |
| Implementations | **Count not published — unverified** |

### The access model, precisely

This is the mechanism, and it is materially better than anything in Round 1's corpus:

1. **Communities** are the top-level grouping — "tribal nations, repositories, families, classes, governing bodies, professional organizations." Each community manages its own protocols.
2. **Cultural protocols** are the access primitive. **All content must belong to at least one cultural protocol**, assigned at creation, for every media asset and content item.
3. **Two protocol types:**
   - **Open** — "accessible to all visitors, including those without user accounts."
   - **Strict** — "only visible to logged in protocol members."
4. **Multi-protocol assignment produces differential access.** When two or more protocols are assigned to one item, "users may have to be members of **all** assigned protocols to access the content." Set per item. **One asset can therefore be visible to one community and invisible to another**, without duplicating the record.
5. **Protocol stewards**, belonging to the community, control membership and access through role-based permissions — **not the site administrator**.
6. **Traditional Knowledge (TK) Labels** let communities "label third party owned or public domain materials with added information about access, use, circulation and attribution" — i.e. assert community terms over material they do not legally own.

**Why this matters for Sathyamangalam specifically.** Round 1 §4.4.3 established that the tribal-knowledge layer is a live legal matter, not a documentation exercise: nine tribal settlements inside the reserve (seven core, two buffer), Soliga and Oorali (Irula) communities, and Tamil Nadu having distributed only **8,594 Forest Rights Act titles against 34,837 claims (~25%)**. Angle F then found that **People's Biodiversity Registers already contain exactly this knowledge for these villages**, compiled without documented benefit-sharing.

Mukurtu's model answers the design question that raises: **the community holds the protocol, not the atlas.** A PBR-derived record can exist in the system, be indexed, be counted, and be invisible to the public until the community that holds its protocol decides otherwise — and can be visible to that community while invisible to a clearance consultant.

**Point of comparison — PARADISEC** (Pacific and Regional Archive for Digital Sources in Endangered Cultures) takes a different route: a default **CC BY-NC-SA 4.0** licence which "precludes the use of the data in commercial products", with per-item restrictions layered on top, and password-protected access for commercial enterprises limited to **algorithm development only, not model inclusion**. Its public documentation covers *user obligations* rather than depositor-side access tiers, so **its tier structure could not be verified**. The licence-based approach is simpler than Mukurtu's but gives communities less granular control.

---

## EGH.3 — Angle G: link rot, finally quantified

Round 1's failure analysis was built entirely on its own corpus — it inferred death rates from the projects it happened to find. The measurement literature exists and gives hard numbers.

| Study | Sample | Finding |
|---|---|---|
| **Weblock, 2015** | **180,000+ links** from three major open-access publishers | **Half-life ≈ 14 years** |
| **BMC Bioinformatics, 2013** | **~15,000 links** in abstracts | **Median webpage lifespan 9.3 years**; **only 62% were archived** |
| **McCown et al., 2005** (*D-Lib Magazine*) | URLs cited in D-Lib articles | **Half were still active 10 years after publication** |
| **Nelson & Allen, 2002** | Digital library objects | ~3% inaccessible after one year → half-life ≈ 23 years |
| **US Supreme Court study, 2013** | Links cited in Supreme Court opinions | **49% are dead** |
| **Pew Research, 2024** | Web pages and citations | **38% of pages from 2013 were missing by 2023**; **54% of Wikipedia articles had at least one dead reference**; 23% of news articles linked to a dead URL |
| **Fetterly et al., 2003** | General web | ~1 link in 200 broke per week → half-life 138 weeks |
| **COVID-19 dashboard study, 2023** | US state dashboards, Feb 2021 | **23% unavailable by April 2023** — roughly two years |

### What this does to Round 1's conclusions

1. **It confirms the scale.** Round 1's corpus showed solo-project survivors running 15–24 years and grant-funded builds dying at 4–6. The independent measurement — a **~14-year half-life** for scholarly links and a **9.3-year median page lifespan** — sits squarely inside that range. Round 1's inferred numbers were not an artefact of its sample.
2. **It makes the archiving gap explicit.** **Only 62% of cited pages were archived.** Round 1's recommendation to deposit with a persistent identifier (following the Gatehouse Gazetteer's Internet Archive deposit) is not belt-and-braces — roughly two in five cited resources have no archived copy at all.
3. **The COVID dashboard figure is the sharpest warning.** These were high-profile, actively funded, publicly essential resources, and **23% were gone within about two years.** Visibility and importance do not protect a web resource.
4. **The Supreme Court figure justifies the legal layer's design.** If **49% of links cited in Supreme Court opinions are dead**, then an atlas whose legal layer links out to government notification URLs will, on this evidence, be roughly half broken within a decade. **Store the document, not the link** — page images alongside transcriptions, as Round 1 recommended for the DSAL gazetteer model.

---

## EGH.4 — What Angles E + G + H change

1. **Contact the French Institute of Pondicherry first.** Muthusankar Gowrappan is on both the Historical Atlas of South India and the Western Ghats Portal. The two hardest layers of the Sathyamangalam atlas have already been built once, for this region, by people who can be emailed.
2. **The Western Ghats is not already documented.** ~10,000 observations for a 129,037 km², six-state landscape, inside IBP rather than as a standalone portal. Round 1's suspected negative finding is confirmed.
3. **Adopt Mukurtu's protocol model for the tribal-knowledge layer** — or adopt Mukurtu itself. Communities hold protocols; protocol stewards control membership; multi-protocol assignment gives per-item, per-community visibility; TK Labels assert community terms over material the community does not legally own. This is a better answer than any of Round 1's four access-tier precedents, and it is free, funded and maintained.
4. **Store documents, not links.** A ~14-year half-life, 62% archival coverage, and 49% dead links in Supreme Court opinions together mean a link-based legal layer will decay by half within a decade.
5. **Wayback remains unavailable in this environment.** Both rounds have therefore dated project deaths from DNS, HTTP and TLS evidence only. Anyone continuing this work from a normal browser should re-date the dead-project entries in Round 1's Task 1 against archived snapshots — that is a genuine, correctable weakness in both rounds.
6. **Assam Biodiversity Portal remains unverified** after a second attempt.

---

## Sources — Round 2, Angles E + G + H

**Western Ghats Portal and IFP**

- [French Institute of Pondicherry — Western Ghats Portal / India Biodiversity Portal](https://www.ifpindia.org/projects/western-ghats-portal-india-biodiversity-portal/) — lead Balasubramanian Dhandapani; team Ayyappan Narayanan and Muthusankar Gowrappan; contact ramesh.br@ifpindia.org; collaborators Prabhakar R and Thomas Vattakaven (Strand Life Sciences); from November 2010; funded by CEPF; IBP absorbed the Western Ghats Portal; "over 10,000 observations from the Western Ghats"; ~17-organisation IBP consortium
- [The Western Ghats Portal — About](https://thewesternghats.indiabiodiversity.org/about) — "an online open collaborative system for sharing biodiversity information"; page timestamp 31 December 2012
- [French Institute of Pondicherry — Historical Atlas of South India](https://www.ifpindia.org/projects/historical-atlas-of-south-india/) — *cross-reference: Muthusankar Gowrappan appears on both projects*
- [Assam Biodiversity Portal](https://assambiodiversity.indiabiodiversity.org/) · [document list](https://assambiodiversity.indiabiodiversity.org/document/list) — **HTTP 403 to automated fetch; counts remain unverified**

**Indigenous access and consent models**

- [Mukurtu CMS — About](https://mukurtu.org/about/) — Center for Digital Scholarship and Curation, Washington State University; from 2007 with the Warumungu community, Kim Christen and Craig Dietrich; funded by NEH, IMLS, Mellon and NSF; TK Labels
- [Mukurtu User Manual — Understanding Communities and Cultural Protocols](https://docs.mukurtu.org/communities-cultural-protocols-categories/UnderstandingCommunitiesAndCulturalProtocols/) — all content must belong to ≥1 protocol; open vs strict protocols; multi-protocol assignment requiring membership of all assigned protocols; protocol stewards
- [Mukurtu — Traditional Knowledge Labels FAQ](https://mukurtu.org/support/traditional-knowledge-labels-faq/)
- [NEH — Mukurtu: A Digital Platform That Does More than Manage Content](https://www.neh.gov/article/mukurtu-digital-platform-does-more-manage-content)
- [PARADISEC — Access Conditions](https://www.paradisec.org.au/deposit/access-conditions/) — default CC BY-NC-SA 4.0 precluding commercial products; per-item restrictions; password-protected commercial access limited to algorithm development, not model inclusion. **Depositor-side access tiers not documented on this page — unverified**
- [PARADISEC catalogue](https://catalog.paradisec.org.au/) · [PARADISEC](https://www.paradisec.org.au/)

**Link rot and resource decay**

- [Wikipedia — Link rot](https://en.wikipedia.org/wiki/Link_rot), collating: Weblock 2015 (180,000+ links, three open-access publishers, ~14-year half-life); BMC Bioinformatics 2013 (~15,000 links, 9.3-year median lifespan, 62% archived); McCown et al. 2005 (*D-Lib*, half active after 10 years); Nelson & Allen 2002 (~3% lost in year one); Supreme Court study 2013 (49% dead); Pew Research 2024 (38% of 2013 pages missing by 2023; 54% of Wikipedia articles with a dead reference); Fetterly et al. 2003 (half-life 138 weeks); COVID-19 dashboard study 2023 (23% gone in ~2 years)
- [Link rot in LIS literature: a 20-year study of web citation decay, recovery and preservation challenges, *Aslib Journal of Information Management*](https://www.emerald.com/ajim/article/doi/10.1108/AJIM-05-2025-0286/1335399/Link-rot-in-LIS-literature-a-20-year-study-of-web)
- [The life and death of URLs in five biomedical informatics journals](https://www.sciencedirect.com/science/article/abs/pii/S1386505606000086) — *robots-disallowed; cited for completeness, figures not extracted*

**Unverified in Angles E + G + H**

- Number of Mukurtu implementations worldwide
- PARADISEC's depositor-side access tier structure
- Assam Biodiversity Portal observation, species and document counts (403, second attempt)
- TIGERNET record counts and cadence (not re-attempted; robots-blocked in Round 1)
- Internet Archive / Wayback Machine remains **unavailable in this environment**, so no project death in either round has been dated from an archived snapshot
- Whether the Western Ghats Portal's ~10,000 observations figure is current or dates from the project page's last revision
- Round 1's remaining geographic holes — Myanmar, the Gulf states, Mongolia, most of the Caribbean, and 21 US states — were **not** addressed in this angle and remain open


---

# Round 3 — Closing the Flagged Unknowns

**Prepared for:** Vishnuvarthan Venkatapathy
**Compiled:** 26 August 2026
**Purpose:** Round 3 closes the specific items Rounds 1 and 2 flagged as unverified. Nothing already covered is re-searched.
**Status: COMPLETE.** All four priorities finished. Of the twelve research items in Priorities 3 and 4: seven resolved, two partially resolved, three closed as tested negatives or confirmed environment blocks. Nothing is left silently open.

---

## What Round 3 delivered

| Priority | Items | Status |
|---|---|---|
| **P1 — Outreach documents** | 1–2 (plus three extras) | **Complete.** Six sendable documents |
| **P2 — Eligibility and cost blockers** | 3–5 | **Complete.** Two resolved, one shown to be publicly unresolvable |
| **P3 — Reserve and India unknowns** | 6–12 | **Complete.** 6, 7, 8, 12 resolved; 9 partial + a correction; 10, 11 unresolved with routes |
| **P4 — Verification gaps** | 13–18 | **Complete.** 15, 17 resolved; 13 partial; 14, 16 unresolved; 18 confirmed blocked |

## The seven things that changed

1. **Six documents are ready to send** — the IFP email, a Tamil Nadu Biodiversity Board RTI, a GBIF eligibility email, and short enquiries to Keystone Foundation, FES and the Tamil Nadu Forest Department. The Forest Department RTI's item 4 is the cheapest win in the whole report: **₹10 buys an authoritative reserve area with a G.O. number**, against which the five published area figures can finally be arranged. (§P1)

2. **GeoNature is now the default recommendation, not a candidate.** **v2.17.4 released 8 July 2026 — seven releases in ten months**, a live 2.17.5 documentation build, and a national user meeting on 5–6 November 2026. (§P2.2)

3. **India's GBIF hosted-portal eligibility is not publicly determinable — and that is the finding.** Three published GBIF statements conflict and no published rule reconciles them. Do not design around a hosted portal. (§P2.1)

4. **The Wildlife Institute of India's National Wildlife Database exists**, holds **protected-area gazette notifications and a wildlife bibliography** — two of your five layers, nationally — with a named contact. Its description survives only on WII's *legacy* site. (§P3.6)

5. **The demonstration set for the disputed-figures feature is now seven cases, two of them national in scope.** Added this round: **WII's own site giving 1,014 vs 682 protected areas** and **175,169 vs 164,074 km²**, and **Tamil Nadu's tanks at 39,000 vs 119,797**. (§P3.6, §P4.15)

6. **Stop reading a stale copyright as a dead project.** It has now misled three times — Costa Rica, WII, and **India Observatory, which is running a "Code4Nature Challenge 2026" under a 2019 copyright**. Round 1's "likely dormant" call on IBIS was wrong. (§P3.9)

7. **The UK records-centre business model has no Indian counterpart — searched, not assumed.** Indian EIA consultancies do their own fieldwork and cite IUCN/ZSI/BSI as reference material; no records layer exists to sell from. An opportunity with no incumbent, and no proven willingness to pay. (§P3.8)

## Standing limitation, now three rounds old

**The Wayback Machine is confirmed unreachable — three rounds, two distinct refusal mechanisms** (`PROVENANCE_REQUIRED` and a proxy `HTTP 403`). No project death anywhere in this report has been dated from an archived snapshot; all death evidence rests on DNS, HTTP, TLS and on-page staleness at the moment of checking. Closing this needs a normal browser and about an hour, and it is the single highest-value manual task outstanding. (§P4.18)

---


---

# Round 3 · Priority 1 — Outreach Documents (ready to send)

**Compiled:** 26 August 2026
**Status:** These are drafts for you to send. Nothing here has been sent on your behalf.
**Note on addresses:** where a postal address or officer name could not be verified from an official source, the draft says so inline in `[square brackets]` rather than inventing one.

---

## P1.1 — Email to the French Institute of Pondicherry

**Why this is first:** Round 2 established that **Muthusankar Gowrappan** is on the team of both the *Historical Atlas of South India* and the *Western Ghats Portal* at IFP — the atlas's two hardest layers, already built once, for this region. Round 1 also recorded IFP as a documented partner of the Western Ghats Portal and a participant in BIOTIK (whose TLS certificate has since lapsed).

**To:** `ramesh.br@ifpindia.org`
**Cc:** *[IFP lists Muthusankar Gowrappan and Balasubramanian Dhandapani on the Western Ghats Portal project page but does not publish their individual addresses — ask for them in the reply, or use the general contact form at ifpindia.org]*
**Subject:** Historical Atlas of South India — current status, and a possible collaboration on a Sathyamangalam-scoped atlas

> Dear Dr Ramesh,
>
> I am writing from Erode, Tamil Nadu. I am scoping a landscape-scale documentation project for the Sathyamangalam Tiger Reserve — an atlas that would bring together species occurrence records, a place-name gazetteer, legal and government instruments, historical documents and published literature for that one landscape, on one site.
>
> IFP's work is the closest precedent I have found anywhere, and I would rather build on it than duplicate it. Three questions, if you have the time:
>
> **1. The Historical Atlas of South India.** The project page describes phase one running 2005–2008 under Dr Subbarayalu and Dr Muthusankar, covering Tamilagam from prehistory to 1600 CE, built from the south Indian inscription corpus together with field survey and ASI archaeological reports. The most recent activity note I can find is from late 2014.
> - Is the atlas still maintained, and is the GIS interface still reachable?
> - Is the underlying place data available for reuse — as a download, an API, or under a data-sharing agreement?
> - Is there a published description of its data model I could read, particularly how it handles a place carrying several attested names across periods?
>
> **2. The Western Ghats Portal.** I understand it began in November 2010 with CEPF funding and was subsequently absorbed into the India Biodiversity Portal, and that the Western Ghats holdings are on the order of ten thousand observations. Is that still the arrangement, and is IFP still actively involved in the IBP consortium?
>
> **3. Collaboration.** Sathyamangalam sits inside the Western Ghats landscape and adjoins the Nilgiri Biosphere Reserve. If a reserve-scoped atlas were built there, would IFP consider any of the following:
> - advising on, or contributing to, the place-name layer from the inscription corpus and the Historical Atlas;
> - a data-sharing arrangement in either direction, with attribution and licence terms of IFP's choosing;
> - simply telling me what you would do differently if you were starting the Historical Atlas again today.
>
> I am at an early stage and have no funding attached to this, so there is nothing I need from you urgently. Even a short reply on question 1 would save me from rebuilding something that already exists.
>
> With thanks and respect for the work,
>
> **Vishnuvarthan Venkatapathy**
> Erode, Tamil Nadu
> *[your phone / email]*

**Notes on sending**

- Keep it to these three questions. The single highest-value answer is 1(b) — whether the place data can be reused.
- If there is no reply in three weeks, the IFP contact form and the institute's general enquiry address are the fallback; a phone call to the institute asking to be put through to the Indology or Geomatics department is more effective in practice than a second email.
- Ask for Muthusankar Gowrappan by name if you call. He is the person on both projects.

---

## P1.2 — Records request to the Tamil Nadu Biodiversity Board

**Why this matters:** Round 2's largest single finding. As of **August 2026** the National Biodiversity Authority's own dashboard reports **276,653 Biodiversity Management Committees** and **272,648 People's Biodiversity Registers** nationally (up from 267,608 PBRs in May 2023), with **only 11,951 verified** as of the last published verification figure — and **none digitised**. Every panchayat around Sathyamangalam is legally required to hold one, and they are exactly the atlas's subject matter: village-level records of local flora, fauna, medicinal-plant knowledge and landscape.

### Which instrument to use

**File this as a formal RTI application, not a courtesy email.** An RTI carries a statutory 30-day response deadline and an appeal route; an email carries neither.

| Item | Detail |
|---|---|
| Statute | **Right to Information Act 2005** |
| Fee | **₹10** application fee, under **Rule 3(a)**, Tamil Nadu RTI (Fees) Rules 2005 |
| Payment modes accepted | Cash, postal money order, **court fee stamp**, demand draft, or banker's cheque |
| Copying charge | **₹2 per page** for A4/A3 (Rule 3(b)(i)); actual cost for larger sizes |
| Inspection of records | **Free for the first hour**, then **₹5 per hour** (Rule 3(b)(iv)) |
| Electronic media | **₹50 per diskette/floppy** (Rule 3(c)(i)) |
| Addressee | The **State Public Information Officer, Tamil Nadu Biodiversity Board** |
| Deadline | 30 days from receipt |

**Address caveat:** the TNBB website did not expose a contact page or a named Public Information Officer to automated fetch on 26 August 2026 (`robots.txt` fetch failure). Confirm the current postal address and PIO name from the Board directly or via the Tamil Nadu State Information Commission before posting. A phone number of **044-2278 2730** appears on commercial contact aggregators — **treat as unverified** and confirm before relying on it.

### The application text

> **To**
> The State Public Information Officer
> Tamil Nadu Biodiversity Board
> *[confirm current address]*
>
> **Subject:** Application under Section 6(1) of the Right to Information Act, 2005 — information regarding Biodiversity Management Committees and People's Biodiversity Registers in the Sathyamangalam Tiger Reserve landscape, Erode District
>
> Sir/Madam,
>
> I request the following information under the Right to Information Act, 2005. The application fee of ₹10 is enclosed by way of *[court fee stamp / postal order / demand draft]*.
>
> **Scope of this request.** All items below relate to local bodies in **Erode District** whose jurisdiction falls wholly or partly within, or immediately adjoining, the **Sathyamangalam Tiger Reserve**, including but not limited to those overlapping the **Sathyamangalam, Talavadi, Bhavanisagar, Hasanur, Thalamalai, Germalam and Asanur forest ranges**.
>
> **1.** A list of all **Biodiversity Management Committees** constituted under Section 41 of the Biological Diversity Act, 2002 in the area described above, stating for each: the name of the local body, its Local Government Directory (LGD) code if available, the date of constitution, and the name and designation of the current Chairperson.
>
> **2.** For each such Biodiversity Management Committee, whether a **People's Biodiversity Register** has been prepared under Rule 22(6) of the Biological Diversity Rules, 2004, and if so:
> &nbsp;&nbsp;(a) the date of preparation;
> &nbsp;&nbsp;(b) whether it has been **verified**, and if so by which authority and on what date;
> &nbsp;&nbsp;(c) the name of the technical support group or agency that assisted in its preparation;
> &nbsp;&nbsp;(d) whether it exists in **digital form**, and in what format.
>
> **3.** The **total number** of Biodiversity Management Committees constituted and People's Biodiversity Registers prepared in **Tamil Nadu** as a whole, and separately for **Erode District**, as on the date of this application.
>
> **4.** A copy of the **Board's guidelines, template or format** prescribed for the preparation of People's Biodiversity Registers in Tamil Nadu.
>
> **5.** The Board's policy or standing orders governing **public access to People's Biodiversity Registers**, including any restrictions on the disclosure of traditional knowledge recorded in them, and the procedure by which a member of the public or a researcher may inspect or obtain a copy of a Register.
>
> **6.** Whether the Board has undertaken, commissioned or planned any programme for the **digitisation** of People's Biodiversity Registers, and if so, its status, and copies of any related orders, tenders or correspondence.
>
> **7.** Details of any **benefit-sharing** determined or disbursed under Sections 21 or 41(3) of the Biological Diversity Act, 2002 in respect of traditional knowledge recorded in People's Biodiversity Registers in Erode District.
>
> **Manner in which information is required:** by post, in *[English / Tamil]*, in electronic form where available. Where the volume of item 1 or item 2 is large, I am willing to inspect the records under Section 2(j)(i) and will pay the prescribed inspection fee.
>
> I confirm that I am a citizen of India.
>
> Yours faithfully,
> **Vishnuvarthan Venkatapathy**
> *[full postal address]*
> *[phone / email]*
> *[place, date]*

### Notes on filing

- **Items 5 and 6 are the strategically important ones.** Item 5 tells you whether you may lawfully use this material at all and on what terms; item 6 tells you whether someone is already doing the digitisation you are contemplating.
- **Item 7 is sensitive but should stay in.** Round 2 recorded the Maduranthakam case, where medicinal-plant knowledge was collected for a PBR and residents received no compensation. Asking the question on the record establishes that you approached this as a benefit-sharing matter from the start — which matters given the nine tribal settlements inside the reserve and Tamil Nadu's Forest Rights Act record (**8,594 titles distributed against 34,837 claims, ~25%**).
- **File a parallel request to the Erode District Collectorate** (which holds panchayat-level records through the District Rural Development Agency) if the Board's reply is thin on items 1 and 2. Boards routinely hold aggregate numbers while districts hold the actual documents.
- **If the Board refuses items 5 or 7**, the first appeal lies with the departmental First Appellate Authority within 30 days, and the second appeal to the **Tamil Nadu State Information Commission**.
- **Do not ask for the Registers themselves in this first application.** Ask what exists, whether it is verified, and on what terms it can be accessed. Requesting the documents before establishing the access policy invites a blanket refusal on traditional-knowledge grounds.

---

## P1.3 — Email to GBIF on hosted-portal eligibility

**Why:** Round 1 §4.4.4 item 1. GBIF's own country page states India **"is not a participant of GBIF"** and lists it as an Observer Country, yet the India Biodiversity Portal has published to GBIF since **17 April 2019** with **"GBIF India"** named as its endorsing node. Hosted Portals are free but restricted to Participants and node-endorsed publishers. The published documentation does not resolve the contradiction — see §P2.1 below for what the guidance does and does not say.

**To:** `hostedportals@gbif.org`
**Cc:** `participation@gbif.org`
**Subject:** Hosted portal eligibility for a project in India — Observer country status vs an existing endorsing node

> Dear GBIF Secretariat,
>
> I am scoping a place-scoped biodiversity atlas for a single protected area in Tamil Nadu, India — the Sathyamangalam Tiger Reserve — and I am trying to establish whether a GBIF Hosted Portal is an option.
>
> I have read the terms and processes page, which states that hosted portals are offered to representatives of GBIF Participants and to data-publishing institutions endorsed by a node, with priority to Voting Participant nodes, and that national portals are restricted to GBIF Participants.
>
> My difficulty is that the two facts I can verify appear to conflict:
>
> - GBIF's country page for India states that India **is not a participant of GBIF** and lists it as an Observer Country.
> - The India Biodiversity Portal has been a publisher on GBIF since 17 April 2019, with **GBIF India** shown as its endorsing node.
>
> Could you clarify:
>
> 1. Is a hosted portal available at all to an organisation or project in India, given the country's Observer status?
> 2. If endorsement by "GBIF India" is sufficient to qualify a publisher, who currently performs that endorsement function, and how would a new Indian publisher approach them?
> 3. Would a **thematic or place-scoped** portal covering a single protected area — rather than a national portal — fall under a different eligibility rule? I note the Legume Data Portal as an example of a non-national hosted portal.
> 4. If a hosted portal is not available, is there a recommended alternative for a small team wanting a public front end over GBIF-mediated data for one landscape?
>
> I am a small independent project with no institutional host at present, so I would rather understand the constraint now than design around an assumption.
>
> With thanks,
>
> **Vishnuvarthan Venkatapathy**
> Erode, Tamil Nadu, India
> *[your email]*

---

## P1.4 — Three shorter enquiries worth sending in the same week

These close Round 1 and Round 2 unknowns that only a direct approach will resolve. Each is deliberately short.

### To Keystone Foundation, Kotagiri — archive platform (closes Round 1 unverified item 19)

**To:** the contact address on `keystone-foundation.org`
**Subject:** Keystone Archives — platform and approach

> Dear Keystone Foundation,
>
> I am scoping a landscape documentation project for the Sathyamangalam Tiger Reserve, and your archive at keystone-archives.org is the closest thing I have found to what I am trying to build — a landscape-scoped, multi-format collection covering the Nilgiri Biosphere Reserve.
>
> Three short questions:
>
> 1. What software does the archive run on? I have guessed Omeka from the URL structure, but I would rather ask than assume.
> 2. Roughly how much staff time does it take to maintain, and who does it?
> 3. Knowing what you know now, would you choose the same platform again?
>
> I am not asking for access to anything — only for the benefit of your experience before I commit to a stack.
>
> With thanks,
> **Vishnuvarthan Venkatapathy**, Erode

### To Foundation for Ecological Security — the orphaned bibliography (closes Round 1 unverified item, §2.5)

**To:** the contact address on `fes.org.in`
**Subject:** India Observatory / IBIS — status of the bibliographic database

> Dear Foundation for Ecological Security,
>
> The India Observatory tool page for IBIS describes a bibliography of over 500,000 citations relating to Indian ecology. The page carries a 2006–2019 copyright and the reptile and amphibian modules are still described as work in progress, so I am unsure whether the resource is still maintained.
>
> Could you tell me whether the bibliographic database still exists, whether it is still accessible in any form, and whether FES would consider making it available for reuse? It appears to be the largest India-focused ecological literature index anywhere, and it would be a considerable loss if it were not preserved.
>
> With thanks,
> **Vishnuvarthan Venkatapathy**, Erode

### To Tamil Nadu Forest Department — the current management plan (closes Round 2 unverified item)

Round 1 and Round 2 located only the **2010–2020** Sathyamangalam Wildlife Sanctuary management plan. A tiger reserve's Tiger Conservation Plan is a statutory document under **Section 38V of the Wildlife (Protection) Act, 1972**, so a current one should exist.

**Route:** RTI to the **Public Information Officer, Office of the Field Director, Sathyamangalam Tiger Reserve**, or to the Principal Chief Conservator of Forests (Chief Wildlife Warden), Chennai.

> Information sought under the Right to Information Act, 2005:
>
> 1. Whether a Tiger Conservation Plan for the Sathyamangalam Tiger Reserve has been prepared and approved under Section 38V of the Wildlife (Protection) Act, 1972, and if so its period of operation and date of approval by the National Tiger Conservation Authority.
> 2. A copy of the said Tiger Conservation Plan, or of its non-confidential portions, in electronic form.
> 3. Whether the Management Plan for Sathyamangalam Wildlife Sanctuary 2010–2020 has been superseded, and by which document.
> 4. The current notified area of the Sathyamangalam Tiger Reserve, of its core/critical tiger habitat and of its buffer zone, in hectares, together with the G.O. or notification numbers and dates by which each was fixed.
>
> *[fee, address and citizenship declaration as in P1.2]*

**Item 4 is the one that matters most.** Rounds 1 and 2 found **five different total-area figures** for the reserve — 1,408.40 km², 1,411.60 km², 1,408.6 km², 1,408 km² and 1,455 km². An RTI answer stating the notified area with its G.O. number is the authoritative record against which all five can be reconciled — and it becomes the first fully-sourced entry in the disputed-figures layer.

---

## What Priority 1 changes

1. **You now have five sendable documents**: the IFP email, the TNBB RTI, the GBIF eligibility email, and three short enquiries (Keystone, FES, TN Forest Department).
2. **The TNBB request is designed to establish access terms before requesting content.** Items 5 and 6 come before any request for the Registers themselves, deliberately.
3. **The Forest Department RTI item 4 is the cheapest possible first win** for the disputed-figures feature: one ₹10 application produces an authoritative area figure with a citable G.O. number, against which five published figures can be arranged.
4. **Two addresses need confirming before posting** — the TNBB Public Information Officer and postal address, and IFP's individual staff addresses. Both are noted inline.

---

## Sources — Priority 1

- [Tamil Nadu State Information Commission — Fees to be paid under RTI in Tamil Nadu (PDF)](https://tnsic.gov.in/pdf/Fees_RTI_English.pdf) — ₹10 application fee under Rule 3(a); ₹2/page A4–A3 under Rule 3(b)(i); free first hour then ₹5/hour inspection under Rule 3(b)(iv); ₹50 per diskette under Rule 3(c)(i); accepted payment modes
- [Tamil Nadu State Information Commission](https://www.tnsic.gov.in/)
- [National Biodiversity Authority](https://www.nbaindia.nic.in/) — dashboard "BMC / PBR / BHS – Status", **August 2026**: 276,653 BMCs, 272,648 PBRs, 50 Biodiversity Heritage Sites
- [French Institute of Pondicherry — Western Ghats Portal / India Biodiversity Portal](https://www.ifpindia.org/projects/western-ghats-portal-india-biodiversity-portal/) — contact `ramesh.br@ifpindia.org`; team Balasubramanian Dhandapani, Ayyappan Narayanan, Muthusankar Gowrappan; from November 2010; CEPF-funded
- [French Institute of Pondicherry — Historical Atlas of South India](https://www.ifpindia.org/projects/historical-atlas-of-south-india/) — Subbarayalu Yellava and Muthusankar Gowrappan; phase 1 2005–2008
- [GBIF — Terms and processes for hosted portal applications](https://www.gbif.org/article/49Ulh5tfiJyqifDpCKvtPO/terms-and-processes-for-hosted-portal-applications) — `hostedportals@gbif.org`
- [GBIF — India participation page](https://www.gbif.org/country/IN/participation) — "This country is not a participant of GBIF"
- [Tamil Nadu Biodiversity Board](https://tnbb.tn.gov.in/) — *contact page robots-disallowed; PIO and postal address **unverified***

**Unverified in Priority 1**

- TNBB's current postal address and the name of its State Public Information Officer (site contact page could not be fetched — `robots.txt` failure, 26 Aug 2026)
- The TNBB phone number 044-2278 2730 (appears only on commercial contact aggregators — **do not rely on it without confirming**)
- Individual email addresses for Muthusankar Gowrappan and Balasubramanian Dhandapani at IFP (only the project contact `ramesh.br@ifpindia.org` is published)
- Whether a Tiger Conservation Plan for Sathyamangalam exists publicly — see §P3.6
- State-wise PBR/BMC breakdown for Tamil Nadu — NBA publishes national totals only on its homepage; the SBB/BMC dashboard returned a server error (see §P4.1)


---

# Round 3 · Priority 2 — Eligibility and Cost Blockers

**Compiled:** 26 August 2026

| # | Question | Status | Short answer |
|---|---|---|---|
| 3 | Is a free GBIF Hosted Portal available to an Indian project? | **Unresolved from published guidance — and that is the finding** | The published documentation is genuinely contradictory and cannot be reconciled from outside. Email drafted (§P1.3) |
| 4 | Is GeoNature actively released in 2026? Deployments? Cost? | **Activity: resolved. Deployments and cost: unverified** | **v2.17.4, 8 July 2026 — seven releases in ten months.** Decisively active. No deployment census and no published pricing exist |
| 5 | Is Nunaliit still maintained? | **Probably yes, on inference** | Releases page shows `2.4.0-SNAPSHOT` dated **29 July** and `2.3.0` dated **8 April**, with no year rendered. GitHub omits the year for current-year dates, so both infer to 2026 |

---

## P2.1 — Item 3: India's GBIF status cannot be resolved from published sources

Round 1 flagged this as its highest-priority unknown. Round 3 attempted to close it from documentation alone. **It cannot be closed that way**, and the reason is worth recording precisely.

The two published facts remain, and they still conflict:

| Source | What it says |
|---|---|
| [GBIF country page for India](https://www.gbif.org/country/IN/participation) | **"This country is not a participant of GBIF."** India is listed as **"A GBIF Observer Country from Asia."** |
| [GBIF publisher record for the India Biodiversity Portal](https://www.gbif.org/publisher/62431bdf-fa0c-452b-afce-be14884a47ff) | Publisher since **17 April 2019**, endorsing node **"GBIF India"** |
| [GBIF hosted-portal terms](https://www.gbif.org/article/49Ulh5tfiJyqifDpCKvtPO/terms-and-processes-for-hosted-portal-applications) | Free; open to Participant nodes and **node-endorsed publishing institutions**; priority to Voting Participants; **national portals restricted to GBIF Participants** |

Attempts to resolve it further in this round:

- `gbif.org/the-gbif-network/participant-list` — **HTTP 404**
- `gbif.org/publisher/search?country=IN` — **robots.txt disallowed**
- Search for published guidance on Observer-country eligibility — nothing found that addresses the case

**The finding, stated as a finding:** *whether an Indian project can obtain a free GBIF Hosted Portal is not publicly determinable.* The terms hinge on node endorsement; a node called "GBIF India" demonstrably performs endorsement for at least one publisher; and GBIF's own country page says India is not a participant. Three published statements, no published rule that reconciles them.

**Practical consequence:** do not design around a hosted portal until `hostedportals@gbif.org` replies. The email in §P1.3 asks the four questions that would settle it, including the one that matters most — whether a **place-scoped** portal (like the Legume Data Portal, which is thematic rather than national) falls under a different rule than a national portal.

---

## P2.2 — Item 4: GeoNature is unambiguously alive, and moving fast

Round 1 could not verify GeoNature's release status, deployment count or cost. Two of those three are now settled.

### Release activity — resolved

| Version | Date |
|---|---|
| **2.17.4** | **8 July 2026** |
| 2.17.3 | 1 July 2026 |
| 2.17.2 | 9 June 2026 |
| 2.17.1 | 13 April 2026 |
| **2.17.0** — "Pipistrellus kuhlii 🦇" | 12 March 2026 |
| 2.16.4 | 17 November 2025 |
| 2.16.3 | 29 September 2025 |

**Seven releases in ten months**, with fixes covering Docker configuration, metadata handling and authentication. The documentation site is already serving **"Documentation GeoNature 2.17.5"**, so a further release exists beyond the seven listed.

This is a materially stronger signal than Round 1 could establish. For a small team choosing a stack, a monthly-to-quarterly release cadence from a consortium of French national parks is about as good a maintenance guarantee as open-source infrastructure offers.

**National meeting confirmed:** *"Les rencontres nationales GeoNature 2026 auront lieu les 5 et 6 novembre 2026 à Paris"* — programme and registration listed as *à venir*.

### Deployment count — still not published

- The GeoNature site points to a **Framacalc "liste des utilisateurs"** — **robots.txt disallowed**, could not be read.
- **Comptoir du Libre** records only **two self-declared users**: **Parc national des Écrins** and **Mairie de Bayonne**. This is a self-declaration registry, not a census, and the page says so — users and providers "declare themselves."
- A regional instance beyond the national parks is visible: **GeoNat'îdF**, run by the Agence Régionale de la Biodiversité en Île-de-France.

**Verdict: no reliable deployment count exists.** Anyone needing one should ask the PnX-SI team directly or attend the November meeting.

### Cost — no published pricing, but a commercial support market exists

GeoNature is **GPLv3** and free to licence. Nobody publishes hosting or deployment pricing. What does exist:

| Provider | Role |
|---|---|
| **Makina Corpus** and **Makina Corpus Territoires** | Listed on Comptoir du Libre as service providers for GeoNature |
| **Natural Solutions** | Publishes a GeoNature service offer and wrote the practitioner guide Round 2 cited |

**The transferable reading:** the real cost of GeoNature is not licence fees, it is **a server plus a competent administrator** — exactly as Round 1 §4.2.4 estimated — with an optional paid-support market if you want it. That estimate now stands on firmer ground.

---

## P2.3 — Item 5: Nunaliit is probably active, and here is exactly how confident to be

Round 1 recorded "2,640 commits, latest commit date not exposed" and marked the framework's 2026 status **unverified**. Round 3 got closer but not all the way.

**What the releases page shows:**

| Release | Date shown | Note |
|---|---|---|
| **2.4.0-SNAPSHOT** | **July 29, 17:05** | Latest development build |
| 2.3.0 | April 8, 19:54 | Requires Java 11 minimum |
| 2.2.9 | April 8, 19:44 | Requires Java 8 minimum |

**The inference, and its limit.** GitHub omits the year when a date falls in the current calendar year and shows it when it does not. All three dates render **without a year**, which by that convention places them in **2026** — implying a stable 2.3.0 in April 2026 and a development snapshot in July 2026, i.e. an actively maintained framework.

**But the year is not stated on the page**, and the commit history at `/commits/master` is **robots.txt disallowed**, so this is an inference from a rendering convention, not a fetched fact. **Marked as inferred, not verified.**

**How to close it in thirty seconds:** open [github.com/GCRC/nunaliit/releases](https://github.com/GCRC/nunaliit/releases) in a normal browser and hover over the relative date — the full timestamp appears in the tooltip.

**Why it matters:** Nunaliit is the only framework in the entire corpus purpose-built for community-editable multimedia place atlases with document relations, flexible schemas and offline tablet sync — and Round 2 established that it transferred to the Global South through a short GCRC workshop (the Pa Ipai Astronomical Atlas in Mexico). If it is maintained, it is a serious candidate. If it is not, the tribal-knowledge layer should go to **Mukurtu** (Round 2 §EGH.2) and the occurrence layer to GeoNature.

---

## What Priority 2 changes

1. **GeoNature moves from "worth evaluating" to "the default recommendation" for the occurrence and observation layer.** Seven releases in ten months, a live 2.17.5 documentation build, a national user meeting in November, a commercial support market, and a working single-site reference implementation in Biodiv'Écrins. Nothing else in the corpus has that combination.
2. **Do not plan around a GBIF hosted portal.** The eligibility question is not publicly answerable. Treat it as a possible bonus, not a component.
3. **Nunaliit stays a live candidate**, pending a thirty-second browser check you can do yourself.
4. **The stack recommendation now has a shape**: GeoNature for occurrence and observation, Mukurtu for the community-knowledge layer with its cultural-protocol access model, Linked Places Format for the gazetteer, and a boring CMS (Drupal, as Terras Indígenas uses) for legal instruments and documents. Four components, all free, all maintained, none of them built from scratch.

---

## Sources — Priority 2

- [GBIF — India participation](https://www.gbif.org/country/IN/participation) · [India summary](https://www.gbif.org/country/IN/summary) — "This country is not a participant of GBIF"; "A GBIF Observer Country from Asia"
- [GBIF — Terms and processes for hosted portal applications](https://www.gbif.org/article/49Ulh5tfiJyqifDpCKvtPO/terms-and-processes-for-hosted-portal-applications)
- [GBIF — India Biodiversity Portal publisher record](https://www.gbif.org/publisher/62431bdf-fa0c-452b-afce-be14884a47ff) — endorsing node "GBIF India", publisher since 17 April 2019
- [GitHub — PnX-SI/GeoNature releases](https://github.com/PnX-SI/GeoNature/releases) — 2.17.4 (8 Jul 2026) through 2.16.3 (29 Sep 2025)
- [Documentation GeoNature 2.17.5](https://docs.geonature.fr/)
- [GeoNature](https://geonature.fr/) — Rencontres nationales 2026, 5–6 November 2026, Paris; user list hosted on Framacalc
- [Comptoir du Libre — GeoNature](https://comptoir-du-libre.org/fr/softwares/125) — GPLv3; two self-declared users (Parc national des Écrins, Mairie de Bayonne); service providers Makina Corpus and Makina Corpus Territoires
- [Natural Solutions — GeoNature service offer](https://www.natural-solutions.eu/geonature)
- [GeoNat'îdF — Agence Régionale de la Biodiversité Île-de-France](https://geonature.arb-idf.fr/presentation-de-geonat-ile-de-france)
- [GitHub — GCRC/nunaliit releases](https://github.com/GCRC/nunaliit/releases) — 2.4.0-SNAPSHOT (July 29), 2.3.0 and 2.2.9 (April 8); **years not rendered**
- [GitHub — GCRC/nunaliit](https://github.com/GCRC/nunaliit) — 2,640 commits

**Unverified in Priority 2**

- Whether an Indian project is eligible for a GBIF Hosted Portal — **not publicly determinable**; participant-list URL returns 404, publisher search is robots-disallowed
- The number of GeoNature deployments worldwide — Framacalc user list robots-disallowed; Comptoir du Libre shows only two self-declarations
- GeoNature hosting, deployment or support pricing from any provider — none published
- The **year** of Nunaliit's 2.4.0-SNAPSHOT and 2.3.0 releases — inferred as 2026 from GitHub's date-rendering convention; commit history robots-disallowed
- Whether the GeoNature 2.17.5 documentation build corresponds to a tagged release


---

# Round 3 · Priorities 3 and 4 — Complete

**Compiled:** 26 August 2026
**Status:** All twelve items attempted. Seven resolved, two partially resolved, three closed as tested negatives or confirmed blocks. Nothing is left silently open.

| # | Item | Status |
|---|---|---|
| 6 | WII National Wildlife Database | **Resolved** — exists; produced a new national-scale figure dispute |
| 7 | TIGERNET record counts and cadence | **Resolved** — 1,581 mortality events 2012–2025, via NTCA's public page |
| 8 | Does anyone in India sell biodiversity desk studies for EIA? | **Resolved as a negative** — searched, and the LERC model does not exist here |
| 9 | FES 500,000-citation bibliography | **Partial — and Round 1 was wrong.** India Observatory is active; bibliography access still unverified |
| 10 | FLAME Gazetteer / Districts Project | **Unresolved; contact obtained** |
| 11 | Post-2020 Sathyamangalam management plan | **Unresolved** — none public; RTI is the route |
| 12 | Keystone Archives platform | **Resolved** — Omeka S |
| 13 | State-wise PBR counts, Tamil Nadu | **Partial** — national totals current, no state breakdown published |
| 14 | Assam Biodiversity Portal | **Unresolved** — 403 on a third attempt, by two routes |
| 15 | Tamil Nadu tank count | **Partial — and the discrepancy is now explained** |
| 16 | Bhuvan layer counts and scales | **Unresolved** — catalogue does not render specifications |
| 17 | Myanmar / Gulf / Mongolia / Caribbean in native languages | **Tested negative** — now a finding, not a gap |
| 18 | Wayback Machine for dating dead projects | **Confirmed blocked** — three rounds, two mechanisms |

---

## P3.6 — Item 6: the WII National Wildlife Database exists. Three findings.

Round 1 flagged this as a significant unknown: *"The WII page that search engines associate with this name does not, when opened, describe a national wildlife database — only labs and repositories. Whether the NWD is defunct, renamed, or simply internal is unresolved."*

**It is none of those three. It exists, it is substantial, and its description survives only on the legacy version of WII's own website.**

### Finding 1 — The National Wildlife Database Cell is real and directly relevant

The **National Wildlife Database Cell (NWDC)** at the Wildlife Institute of India develops the **National Wildlife Information System (NWIS)** covering India's protected-area network.

| Holding | Detail |
|---|---|
| Protected area coverage | National Parks, Wildlife Sanctuaries, Conservation Reserves, Community Reserves, **Tiger Reserves**, Biosphere Reserves, Elephant Reserves, Ramsar Wetland Sites, World Heritage Sites |
| **Protected Area Gazette Notifications** | **Directly relevant to the atlas's legal-instrument layer** |
| **Wildlife bibliography and research literature** | **Directly relevant to the atlas's literature layer** |
| Other datasets | Conservation status of animal species, biogeographic regions, administrative units, habitat types, plant species data |
| Named contact | **Dr J. S. Kathayat** — `jsk [at] wii [dot] gov [dot] in` |
| Page last updated | **27 November 2023** |

Two of the atlas's five domains — gazette notifications and wildlife literature — are already assembled here, nationally, by the institute that also holds the All-India Tiger Estimation data. **This is a data-partner conversation, not a competitor.**

### Finding 2 — A new disputed-figures case, inside one institution's own website

| Page | Protected Areas | Composition | Total area |
|---|---|---|---|
| `v1.wii.gov.in/nwdc_aboutus` (updated 27 Nov 2023) | **1,014** | 106 National Parks · 573 Wildlife Sanctuaries · 115 Conservation Reserves · 220 Community Reserves | **175,169.42 km²** |
| `v1.wii.gov.in/national_wildlife_database` | **682** | 102 National Parks · 520 Wildlife Sanctuaries · 56 Conservation Reserves · 4 Community Reserves | **~164,074 km²** (4.99% of India) |

**1,014 versus 682 protected areas — a 49% discrepancy — and 175,169 versus 164,074 km², on two pages of the same site describing the same database.**

The likeliest explanation is that one page was updated and the other was not, with Community Reserves growing from 4 to 220 as the newer category filled in. But **neither page carries a date against its figures**, so a reader cannot tell which is current. That is precisely the failure the atlas's disputed-figures feature exists to fix, now demonstrable at **national** level from a **government research institute**.

### Finding 3 — "Migrations drop non-spatial content first", confirmed in India

The **current** WII site at `wii.gov.in/nwdc_aboutus` does **not** describe the NWDC. It lists departments, cells and labs — Tiger Cell, Elephant Cell, Wildlife Forensic & Conservation Genetics Cell, M-STrIPES, ONOS, IRINS — with no NWDC section, no datasets, no access information. The description survives only at **`v1.wii.gov.in`**.

Round 1 drew this lesson from Oregon Explorer's re-platforming. Here is the same pattern in an Indian government institution, and what was dropped is again the **bibliographic and documentary description** rather than the tools.

---

## P3.7 — Item 7: TIGERNET resolved, through NTCA's public mortality page

TIGERNET itself blocks automated fetch. **NTCA publishes the same underlying data openly**, which is the alternate route Round 3 was asked to find.

| Metric | Figure | Period |
|---|---|---|
| **Total tiger mortality events** | **1,581** | **2012–2025** |
| Cases closed after examination | **72.6%** | 2012–2025 |
| Cases pending scrutiny | **27.4%** | 2012–2025 |
| Deaths inside tiger reserves | **52.5%** | 2012–2025 |
| Deaths outside tiger reserves | **47.5%** | 2012–2025 |
| Seizures inside reserves | 19.3% | 2012–2025 |
| Seizures outside reserves | **80.7%** | 2012–2025 |
| Recorded events, 2021 | **130** | |
| Recorded events, 2022 | **122** | |

Tamil Nadu entries are present in the record — a seizure case in Sethumadai Taluk on 02.02.2021 (outside reserve), and a male adult death in Coimbatore Forest Division on 20.09.2021 (outside reserve).

**The most valuable thing on the page is not a number — it is the inclusion rule**, stated explicitly:

> *"No tiger death is entered into the database, unless an authentic source from the State Government reports a tiger mortality."*

Round 1 §3.2 Rung 6 identified "publish your inclusion rule" as a credibility mechanism, citing Pladias, Virginia's Excluded Taxa and Land Conflict Watch's two-source rule. **NTCA already does this**, and the rule has a direct consequence worth recording: a tiger death not reported by a state government does not exist in the national database. That is exactly the kind of definitional boundary the atlas's disputed-figures layer should surface alongside the number.

**Still unverified:** cumulative and state-wise totals are rendered as charts rather than tables, so they could not be extracted. A 2016 NTCA/WII technical report on tiger mortality patterns is published as a PDF and was not read.

---

## P3.8 — Item 8: nobody in India sells biodiversity desk studies. Confirmed by search, not assumed.

Round 3 was asked to establish this as a finding rather than an absence of searching. **It has been searched, and the answer is no.**

What exists in India:

| Provider type | What they sell | Evidence |
|---|---|---|
| **NABET-accredited EIA consultancies** | "Ecology & Biodiversity Study (Biodiversity Assessment)" as a component of an EIA — identifying floristic and faunal communities in the impact zone, computing **frequency, abundance, Importance Value Index (IVI)** and the **Shannon-Wiener Index** | Akone Services, HECS, Perfect Pollucon, BEIPL; NABET maintains an accredited-consultant register |
| Reference sources they cite | **IUCN, WCMC, ZSI, BSI** classifications and the Wildlife (Protection) Act 1972 | Akone service page |

**What does not exist:**

- **No provider draws on a species-records database of its own.** The IUCN/ZSI/BSI references are reference material, not a holding.
- **No provider publishes prices.** Contrast the UK, where Round 3's earlier work established a published tariff — Gloucestershire's records centre charges **£155 for a 1 km search, £295 for 5 km, 5 working days**, and Buckinghamshire's from **£150+VAT** with **up to 100% discount for non-commercial users**.
- **No records-centre layer exists at all.** In the UK the buyer is a consultant on a planning deadline and the seller is a county records centre with a database. In India the consultant does the fieldwork themselves and there is no records centre to buy from.

**The finding, stated as a finding:** *the UK LERC earned-revenue model has no Indian counterpart, and the reason is structural — the records-holding layer that model sells from does not exist in India.* Round 1 §3.4.2 identified this as the one earned-revenue model that transfers to India. It transfers as an **opportunity**, not as a market to enter: there is no incumbent, and also no established willingness to pay.

---

## P3.9 — Item 9: a third correction. India Observatory is active.

Round 1 recorded India Observatory / IBIS as **"likely dormant"**, reasoning from a copyright notice reading 2006–2019 and reptile/amphibian modules still marked work-in-progress.

**That call was wrong.** The site's navigation carries a **"Code4Nature Challenge 2026"**, and IBIS remains listed among **ten active tools**:

Groundwater Monitoring Tool (GMT) · Data Platform (DP) · Composite Landscape Assessment & Restoration Tool (CLART) · GIS Enabled Entitlement Tracking System (GEET) · Integrated Forest Management Toolkit (IFMT) · **Indian Biodiversity Information System (IBIS)** · Crop Water Budgeting (CWB) · Common Land Mapping (CLM) · CLART + Design Estimation Tool (DET) · Location at a Glance (LAG)

IBIS is described as "a group of web-based, modular and searchable biodiversity portals" for birds, mammals and flora.

**The methodological lesson, now three times over.** Round 3 has found three cases where a stale copyright or a stale page was read as a dead project:

| Case | What was inferred | What is true |
|---|---|---|
| Costa Rica CRBio (Round 2 §CD.0) | Dead | Offline under one brand; a live-but-stale successor exists |
| WII National Wildlife Database (§P3.6) | Possibly defunct | Live, on the legacy site build |
| **India Observatory / IBIS** | Likely dormant | **Active — 2026 programming under a 2019 copyright** |

**A stale copyright notice is weak evidence of death.** Both Round 1 and Round 2 leaned on it, and it has now misled all three times. Any future pass should treat copyright dates as a prompt to look harder, never as a verdict.

**Still unverified:** whether the 500,000+ citation bibliography is accessible, and in what form. The IBIS tool page returned `PROVENANCE_REQUIRED` to automated fetch. The FES enquiry drafted in §P1.4 remains the route — and it is now a better-founded enquiry, because the organisation is demonstrably active and therefore likely to reply.

---

## P3.10 — Item 10: FLAME's Districts Project publishes aims, not outputs

The Centre for Knowledge Alternatives page describes the framework and states only:

> *"We evolve a framework and do a pilot for a few districts to begin with."*

**No published district volumes, no status, no team roster, no partners, no funding, no timeline.** The July 2021 concept note by **Prof. Yugank Goyal** remains the only dated artefact, as in Round 1.

**Contact obtained:** `cka@flame.edu.in`. Given the overlapping ambition — reviving the gazetteer form for Indian districts — a short enquiry is worth sending before any Sathyamangalam gazetteer work begins.

**Status unchanged from Round 1: uncertain.** But it is now uncertain *with a contact address*, rather than uncertain in the abstract.

---

## P3.11 — Item 11: no public Sathyamangalam management plan after 2010–2020

No Tiger Conservation Plan or post-2020 management plan for Sathyamangalam was found in public circulation. NTCA maintains a documents page; the reserve's own site publishes an About page with figures but no plan.

A Tiger Conservation Plan is **statutory under Section 38V of the Wildlife (Protection) Act, 1972**, so one should exist. Its absence from the public web is a publication gap, not evidence that none was prepared.

**Route:** the RTI drafted in §P1.4, addressed to the Field Director, Sathyamangalam Tiger Reserve. Item 4 of that application — asking for the notified area with G.O. numbers — remains the highest-value single question in the whole Round 3 outreach set.

---

## P3.12 — Item 12: Keystone Archives runs Omeka S

Round 1 inferred Omeka from URL structure and flagged it unverified. Round 3 confirms the platform family and adds detail.

| Attribute | Finding |
|---|---|
| Platform | **Omeka S** — evidenced by the URL pattern `keystone-archives.org/archive/s/keystone-foundation-archives` (Omeka S's site-scoping convention) and "item-sets" navigation terminology. *Still structural evidence rather than an explicit statement — the site does not name its software* |
| Images | **464** |
| Audio files | **53** |
| Videos | **22** |
| Text documents | **503** |
| Other items | **13** |
| **Total** | **≈1,055 items** |
| Extra tooling | **Kumu.io** integration powering "Browse by People" — a network visualisation over the collection |
| Contribution | The archive has a login and an **"Upload Your Content"** feature — it accepts external contributions |
| Last updated | **No date published** |

**Two things worth taking from this.** First, the counts are **identical to Round 1's**, which means either the archive has not grown in a year or the counts are static text — a live illustration of Round 1's advice to publish a data-modified date. Second, the **Kumu.io network view for people** is an idea worth stealing: a collection of ~1,000 items made navigable by *who* appears in it rather than only by subject or place.

---

## P4.13 — Item 13: national PBR totals updated; Tamil Nadu breakdown still unpublished

| Metric | Figure | As of |
|---|---|---|
| Biodiversity Management Committees constituted | **276,653** | **August 2026** |
| People's Biodiversity Registers prepared | **272,648** | **August 2026** |
| Biodiversity Heritage Sites | **50** | **August 2026** |

Against Round 2's **267,608 PBRs as of May 2023**, growth is roughly **5,040 in three years** — about 1,680 a year on a base of a quarter-million. **The programme is essentially complete in creation terms and has been for years. The gap is verification and digitisation**, and Round 2's verified figure of **11,951** is not superseded by anything on the dashboard, leaving the verified share around **4.4%**.

**No state-wise breakdown is published.** Three routes failed: `nbaindia.org/content/20/35/1/bmc.html` (302 redirect to the homepage), `nbaindia.nic.in/.../bmcdashboard.html` (server error), and the Tamil Nadu Biodiversity Board site (robots failure). **Item 3 of the RTI in §P1.2 is now the primary route to this figure, not a redundancy.**

---

## P4.14 — Item 14: the Assam Biodiversity Portal remains unreadable

Third attempt, two routes, both refused:

- `assambiodiversity.indiabiodiversity.org/document/list` — **HTTP 403**
- `indiabiodiversity.org/observation/list?state=Assam` — **HTTP 403**

The portal is live (its taxonomy and document URLs resolve in search results) but its observation counts, species counts and last-updated date cannot be obtained by automated means.

**Remaining route:** direct enquiry to the **Assam State Biodiversity Board** (`asbb.assam.gov.in`), or a browser visit. Given that Round 2 already resolved the more important of the two Indian sub-portals — the Western Ghats Portal, which holds "over 10,000 observations" and was absorbed into IBP — the Assam figure is now a completeness item rather than a decision-relevant one.

---

## P4.15 — Item 15: the Tamil Nadu tank figure, and why the two numbers differ

Round 2 flagged **119,797 lakes, ponds and tanks** as sourced from a commercial real-estate tool and unreliable. Round 3 found a peer-reviewed figure.

| Figure | Source | Class counted |
|---|---|---|
| **"Over 39,000 tanks"** | *Current Status of Irrigation Systems in Tamil Nadu*, peer-reviewed review article (stated twice) | Appears to be **irrigation tanks** |
| **119,797** | Commercial real-estate "Water Bodies Finder" tool | Appears to be **all water bodies** — lakes, ponds and tanks |

**Verdict: neither figure is debunked, and neither is authoritative.**

- The 39,000 figure is peer-reviewed but **cites no primary source and carries no year**.
- The 119,797 figure has **no established provenance** and should still not be cited.
- **No government source publishing an official tank count was located.** The Water Resources Department's own descriptions do not carry the number; the classification breakdown (system vs non-system tanks, WRD-managed vs panchayat-union-managed, ayacut areas) was not found anywhere.

**This is itself a disputed-figures case, and a good one** — two published figures differing threefold, almost certainly because they count different classes, with neither publishing its inclusion rule. It belongs in the atlas's demonstration set alongside the reserve area, tiger counts, elephant corridors, Western Ghats ESA, sacred groves and now WII's protected-area totals.

---

## P4.16 — Item 16: Bhuvan's catalogue does not publish its specifications

Confirmed present on Bhuvan: **Land Cover Map**, **Land Use Map (LULC)**, **DEM / CartoDEM**, **IRS high-resolution satellite data**, **thematic services and layers**, **historical flood mapping products**, and **hyperspectral (HySI) data**. OGC web services and clip-and-ship were confirmed in Round 2.

**Not obtainable:** specific resolutions, map scales (1:50,000 and similar), coverage years, tile counts or product specifications. The download portal renders navigation rather than a catalogue to automated fetch.

**Practical consequence for a reserve-scale base map:** the products needed almost certainly exist — CartoDEM and LULC at national coverage — but the specifications must be read from the portal in a browser or obtained from NRSC. This is a thirty-minute manual task, not a research question.

---

## P4.17 — Item 17: Myanmar, Mongolia and the Gulf tested in-language. Now a finding.

Round 2 warned that six of six native-language queries had returned projects English search had missed, and that the remaining regions should not be called empty until tested. **They have now been tested.**

| Region | Language tested | Result |
|---|---|---|
| **Myanmar** | Burmese (မြန်မာ ဇီဝမျိုးစုံမျိုးကွဲ ဒေတာဘေ့စ်) | **No national biodiversity portal found.** What exists: CBD national reports, the Biodiversity and Nature Conservation Association (BANCA), and a Myanmar-language news item on snow leopard research. Environmental data appears to reach the public through reports, not a portal |
| **Mongolia** | Mongolian (биологийн олон янз байдлын мэдээллийн сан) | **No national portal found.** What exists: CBD 5th and 6th national reports in Mongolian, *Монгол орны биологийн олон янз байдал* (2017) as volume III of a five-volume national environment work, and the **Mongolian Plant Red List series II (2019)** — all **print and PDF, not databases** |
| **Gulf states** | Arabic (قاعدة بيانات التنوع البيولوجي) | **Nothing new.** The query re-surfaced only **RSCN Jordan's national biodiversity database page** — already catalogued in Round 2 as the case where the parent page still advertises a database whose URL returns 404 |

**The finding:** for Myanmar and Mongolia, the national biodiversity record genuinely appears to be **a printed and PDF literature, not a portal** — CBD reports, national environment volumes and Red List series. This matches Round 1's Central Asia observation that "Central Asian species knowledge appears to reach the public mainly through journal-published checklists."

**Honest limitation:** this is **one query per language**. It is enough to convert "untested" into "tested and nothing surfaced", which is what Round 3 was asked for. It is not enough to assert absence with confidence. A researcher reading Burmese or Mongolian would do better in an hour than these three queries did.

---

## P4.18 — Item 18: the Wayback Machine is confirmed unreachable

Round 3 attempted archive.org twice more, and was refused by **two different mechanisms**:

| Attempt | URL | Refusal |
|---|---|---|
| Round 2 | `web.archive.org/web/2024/http://www.biotik.org/` | `PROVENANCE_REQUIRED` |
| Round 3 | `web.archive.org/web/2024/http://www.biotik.org/` | `PROVENANCE_REQUIRED` |
| Round 3 | `web.archive.org/web/20240101000000*/rebioma.net` | **`PROXY_REJECTED` — HTTP 403 at the proxy** |

**Conclusion, stated plainly:** archive.org is not reachable from this environment, and three rounds of attempts across two mechanisms have established that it will not become reachable. **No project death anywhere in this report has been dated from an archived snapshot.** Every death claim in Task 1 — BISON, NBII, papuaweb, pngtrees, nwatlas, hbmp.hawaii.edu, INBio/CRBio, REBIOMA, GB1900, Arctic Bay Atlas, New Forest Knowledge and the rest — rests on DNS failure, HTTP status, TLS state or on-page staleness at the moment of checking.

**This is the single most valuable outstanding manual task.** In a normal browser, opening `web.archive.org/web/*/[domain]` for each dead project in Task 1 would establish when each actually went offline, and would convert a list of "dead as of 25 August 2026" into a dated mortality record — which is exactly the evidence base Round 2 §EGH.3 showed the link-rot literature provides in aggregate but which this report lacks for its own corpus. **Budget about an hour.**

---

## What Priorities 3 and 4 change

1. **WII holds two of your five layers already** — protected-area gazette notifications and a national wildlife bibliography — with a named contact, Dr J. S. Kathayat. Add him to the outreach set in §P1.
2. **You now have seven demonstration cases for the disputed-figures feature**, three of them added this round: reserve area (five figures), tiger population (twelve figures), elephant corridors (88/101/150), Western Ghats ESA (129,037/59,940/56,825 km²), sacred groves (13,270 vs 100,000–150,000), **WII protected areas (1,014 vs 682)** and **Tamil Nadu tanks (39,000 vs 119,797)**. Two of the seven are national in scope, so the feature no longer demonstrates only on one reserve.
3. **NTCA already publishes an inclusion rule** — "no tiger death is entered into the database unless an authentic source from the State Government reports a tiger mortality." Cite it, and use it as the model for your own.
4. **The UK records-centre business model has no Indian counterpart**, and now that has been searched rather than assumed. Indian EIA consultancies do their own fieldwork; there is no records layer to sell from. That is an opportunity with no incumbent — and no proven willingness to pay.
5. **Stop treating stale copyright as evidence of death.** It has misled three times across three rounds — Costa Rica, WII, and now India Observatory, which is running a **Code4Nature Challenge 2026** under a 2019 copyright.
6. **Keystone Archives runs Omeka S** with ~1,055 items and a Kumu.io people-network view. Its counts are unchanged from a year ago, which is either stagnation or static text — worth asking about in the enquiry drafted in §P1.4.
7. **Myanmar and Mongolia have no national portal** in their own languages either. Their biodiversity record is a printed literature. Tested, not assumed — but on one query each.
8. **Wayback is permanently out of reach here.** Dating the dead projects is now the top manual task, and it is worth about an hour.

---

## Sources — Priorities 3 and 4

**WII National Wildlife Database**

- [WII — National Wildlife Database Cell (legacy site)](https://v1.wii.gov.in/nwdc_aboutus) — NWIS; 1,014 Protected Areas (106 NP, 573 WLS, 115 CR, 220 Community Reserves), 175,169.42 km²; last updated 27 November 2023
- [WII — National Wildlife Database (legacy site)](https://v1.wii.gov.in/national_wildlife_database) — 682 Protected Areas (102 NP, 520 WLS, 56 CR, 4 Community Reserves), ~164,074 km², 4.99% of India; Protected Area Gazette Notifications; wildlife bibliography; contact Dr J. S. Kathayat, `jsk@wii.gov.in`
- [WII — current site, NWDC path](https://wii.gov.in/nwdc_aboutus) — **does not describe the NWDC**

**Tiger mortality**

- [NTCA — Tiger Mortality](https://ntca.gov.in/tiger-mortality/) — 1,581 events 2012–2025; 72.6% closed / 27.4% pending; 52.5% inside / 47.5% outside reserves; seizures 19.3% / 80.7%; 130 events (2021), 122 (2022); Tamil Nadu entries; inclusion rule quoted
- [NTCA / WII — Tiger mortality pattern, technical report 2016 (PDF)](https://ntca.gov.in/assets/uploads/Reports/Tiger_mortality_pattern.pdf) — *identified, not read*
- [NTCA — Documents](https://ntca.gov.in/documents/)

**India's EIA biodiversity-assessment market**

- [Akone Services — Ecology & Biodiversity Study (Biodiversity Assessment)](https://akone.in/service/consultancy-services/ecology-biodiversity-study-biodiversity-assessment/index.html) — IVI, Shannon-Wiener; references IUCN, WCMC, ZSI, BSI; **no own database, no prices**
- [NABET — EIA accreditation](https://nabet.qci.org.in/eia/) · [NABET accredited EIA consultant portal](https://eia.nabet.qci.org.in/Accredited_EIA_Consultant.aspx)
- [HECS — NABET-accredited environmental consultancy](https://hecs.in/service/consultancy-services) · [Perfect Pollucon Services — EIA](https://www.ppsthane.com/environmental-impact-assessment)

**FES / India Observatory**

- [India Observatory — Our Tools](https://www.indiaobservatory.org.in/our-tools) — ten tools including IBIS; copyright 2006–2019; **"Code4Nature Challenge 2026"** in navigation
- [India Observatory — About](https://indiaobservatory.org.in/about-io) · [IBIS tool page](https://www.indiaobservatory.org.in/tool/ibis) — *`PROVENANCE_REQUIRED` to automated fetch*
- [Indian Biodiversity Information System](https://www.indianbiodiversity.org/) · [Foundation for Ecological Security](https://fes.org.in/)

**FLAME Districts Project**

- [FLAME University — The Districts Project](https://www.flame.edu.in/cka/the-districts-project.php) — aims only; contact `cka@flame.edu.in`
- [FLAME University — The Gazetteer Project](https://www.flame.edu.in/cka/the-gazetteer-project.php) · [Concept Note, Yugank Goyal, July 2021 (PDF)](https://www.flame.edu.in/cka/pdfs/Concept-Note.pdf)

**Keystone Archives**

- [Keystone Foundation Archives](https://keystone-archives.org/archive/) — Omeka S (structural evidence); 464 images, 53 audio, 22 videos, 503 text documents, 13 other ≈ 1,055 items; Kumu.io "Browse by People"; "Upload Your Content"

**PBR / Assam / tanks / Bhuvan**

- [National Biodiversity Authority](https://www.nbaindia.nic.in/) — August 2026: 276,653 BMCs, 272,648 PBRs, 50 BHS
- [Assam Biodiversity Portal](https://assambiodiversity.indiabiodiversity.org/) — **HTTP 403** · [IBP observation list, Assam filter](https://indiabiodiversity.org/observation/list?state=Assam) — **HTTP 403** · [Assam State Biodiversity Board](https://asbb.assam.gov.in/)
- [Current Status of Irrigation Systems in Tamil Nadu (PDF)](https://gssrr.org/JournalOfBasicAndApplied/article/download/17606/6881/48049) — "over 39,000 tanks", stated twice, **no primary source or year given**
- [Tamil Nadu PWD Water Resources Department Citizen's Charter (PDF)](https://agritech.tnau.ac.in/agriculture/agri_resourcemgt_water_waterresourceorg.pdf) · [Department of Water Resources (Tamil Nadu)](https://en.wikipedia.org/wiki/Department_of_Water_Resources_(Tamil_Nadu)) — **no tank count published**
- [Bhuvan — free data download](https://bhuvan-app3.nrsc.gov.in/data/download/index.php) — Land Cover, Land Use, CartoDEM, IRS, thematic services, historical flood mapping, HySI; **specifications not rendered**

**Native-language re-tests**

- Myanmar: [Wildlife of Myanmar](https://en.wikipedia.org/wiki/Wildlife_of_Myanmar) · [BANCA](https://en.wikipedia.org/wiki/Biodiversity_and_Nature_Conservation_Association) · [Sources of Environmental and Biodiversity Data (Myanmar)](https://www.slideshare.net/ethicalsector/4-sources-of-environmental-and-biodiversity-data)
- Mongolia: [CBD 5th National Report, Mongolian (PDF)](https://www.cbd.int/doc/world/mn/mn-nr-05-mn.pdf) · [CBD 6th National Report, Mongolian (PDF)](https://www.cbd.int/doc/nr/nr-06/mn-nr-06-mn.pdf) · [*Монгол орны биологийн олон янз байдал* (2017)](https://www.researchgate.net/publication/323545292) · [Mongolian Plant Red List series II (2019)](https://www.researchgate.net/publication/337047867)
- Gulf: [RSCN Jordan — National Biodiversity Database](https://www.rscn.org.jo/national-biodiversity-database) — already catalogued in Round 2; linked BIMS URL 404s

**Unverified after Priority 3 and 4**

- TIGERNET's own interface (blocks automated fetch); NTCA cumulative and state-wise totals (rendered as charts, not tables); the 2016 mortality-pattern report (not read)
- Whether the FES 500,000-citation bibliography is accessible, and in what form
- Whether the FLAME Districts Project has produced any output since July 2021
- Whether a post-2020 Sathyamangalam Tiger Conservation Plan exists (statutory under s.38V; not public)
- An explicit statement of Keystone Archives' software (Omeka S is structural inference); whether its item counts are current or static
- State-wise BMC/PBR figures for Tamil Nadu (three published routes failed)
- Assam Biodiversity Portal counts and last-updated date (403 by two routes, third attempt)
- Any official government count of Tamil Nadu's tanks, and the classification behind the 39,000 and 119,797 figures
- Bhuvan resolutions, scales, coverage years and tile counts
- Myanmar, Mongolia and Gulf coverage rests on **one query per language** — sufficient to say "tested and nothing surfaced", not sufficient to assert absence
- **archive.org remains unreachable** — no death date in this report rests on an archived snapshot
