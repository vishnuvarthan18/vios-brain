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
