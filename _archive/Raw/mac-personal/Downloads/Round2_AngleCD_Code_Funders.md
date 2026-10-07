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
