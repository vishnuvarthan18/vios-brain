# Operation Full Record
## The data-acquisition plan to take the Sathyamangalam Atlas from ~15% to 80%

**Date:** 13 August 2026 · **Target:** 80% coverage of the complete Sathyamangalam record
**Stack (superseded 21 Aug 2026 — see Harvest Engine migration/architecture docs for the current all-Cloudflare design):** originally scoped around Cloudflare Workers + R2 + D1 + KV + Queues + Workers AI, borrowing the pattern of `crawler.js` (a separate, unrelated job-vacancy scraper — do not confuse the two)

---

## 0. Reality check, before we spend a month on this

**Scraping alone will not reach 80%. It tops out around 62–68%.** Here is the honest arithmetic, because a plan that promises 80% from crawlers will fail in month three and you'll have wasted the month.

| Route | Coverage delivered | Effort |
|---|---|---|
| Automated harvest (this plan) | +47 points → ~62% | 3–4 weeks build, then continuous |
| One-document extraction (the 391-page management plan) | +8 points → ~70% | 2 weeks of structured OCR + review |
| Archive full-text mining (gazetteers, Buchanan) | +5 points → ~75% | 1 week |
| Physical archives + department relationship | +4 points → ~79% | 2–3 visits over a quarter |
| Your own camera | +3 points → **~82%** | Ongoing, unavoidable |

So: **build the harvest, but budget for the four things it can't reach.** The plan below covers all five routes; the crawler is just the biggest single block.

One more constraint stated up front: the last ~18% — oral history, unpublished working plans, Tamil manuscript material, current staff and budget figures, original photography — does not exist in machine-readable form anywhere. No amount of engineering closes it. Anyone promising 100% is selling you something.

---

## 1. Architecture

Reuse what you already built. Your `crawler.js` pattern — scheduled Worker → fetch → hash → dedupe in KV → store raw in R2 → extract text via Workers AI `toMarkdown` → push to Queue — is exactly right. Extend it from one crawler to a fleet.

```
                    ┌─────────────────────────────────────┐
   CRON (staggered) │  Worker: orchestrator                │
        ──────────► │  reads /sources/*.yaml, fans out     │
                    └──────────────┬──────────────────────┘
                                   │
        ┌──────────────┬───────────┼───────────┬──────────────┐
        ▼              ▼           ▼           ▼              ▼
   api-harvester   pdf-crawler  html-scraper  fulltext-miner  tile-fetcher
   (GBIF, eBird,   (gov PDFs,   (allowlisted (archive.org    (GEE exports,
    iNat, OpenAlex, notifications, pages only) djvu.txt)      FSI alerts)
    Overpass, ...)  reports)
        │              │           │           │              │
        └──────────────┴─────┬─────┴───────────┴──────────────┘
                             ▼
                    R2 (raw immutable)  ──►  Queue  ──►  normaliser Worker
                                                              │
                                                              ▼
                                              D1 / Postgres (entities + provenance)
                                                              │
                                                              ▼
                                              Static JSON/GeoJSON build artefacts
                                                              │
                                                              ▼
                                                        The Atlas site
```

**Non-negotiable design rules**

1. **Raw first, always.** Every fetch lands in R2 unmodified, keyed by SHA-256, before anything parses it. If your parser is wrong you re-run it; you never re-fetch.
2. **Provenance on every field.** Nothing enters the database without `source_url`, `retrieved_at`, `licence`, `confidence`. A fact without a source is a liability on a site whose entire value proposition is citation.
3. **Idempotent by hash.** Same rules as your job crawler — KV pre-check before D1 or R2 writes.
4. **Fail one, continue all.** Already in your code. Keep it.
5. **Every source gets a YAML descriptor**, not hardcoded arrays. You'll have 40+ sources; a `TARGETS` const won't survive that.

**Geographic constant** — define once, use everywhere:

```
STR_BBOX   = 76.83, 11.48, 77.46, 11.82        # minLon, minLat, maxLon, maxLat
STR_WKT    = <WDPA polygon, fetched once from Protected Planet>
BUFFER_KM  = 10                                 # zone of influence
```

---

## 2. The eight harvest streams

### Stream 1 — Biodiversity occurrence (the species problem)

**Fixes:** birds 8%→85%, mammals 40%→90%, herps 2%→60%, flora 2%→55%, invertebrates 2%→40%

| Source | Endpoint | Expected volume | Licence |
|---|---|---|---|
| **GBIF** | `api.gbif.org/v1/occurrence/search?geometry={WKT}` | 20k–60k records, 1,500–3,000 taxa | CC0/CC BY per record |
| **eBird** | `api.ebird.org/v2/data/obs/geo/recent`, `/product/spplist/{regionCode}`, hotspot endpoints | 280–350 bird species, full checklists | eBird terms, key free |
| **iNaturalist** | `api.inaturalist.org/v1/observations?nelat=…&swlat=…` | 2k–6k observations **plus CC-licensed photos** | CC BY / BY-NC per obs |
| **India Biodiversity Portal** | public API | Indian records, **vernacular Tamil names** | CC |
| **Xeno-canto** | `xeno-canto.org/api/2/recordings?query=…` | Bird and frog audio | CC |
| **GBIF species API** | `/v1/species/match` | Taxonomic backbone for reconciliation | CC0 |
| **IUCN Red List API** | `apiv3.iucnredlist.org` | Conservation status for every taxon | key required, free for non-commercial |

**Pipeline:** fetch by polygon → resolve every name against the GBIF backbone → deduplicate across sources → cross-reference against the management plan checklist → **publish the diff**. The delta between "species the department documented in 2010" and "species with a verifiable open record today" is itself a publishable finding, and it is exactly the kind of thing WILDLABS and JoTT pick up.

**Note the trap:** GBIF returns records from a bounding box that includes chunks of Mudumalai and BRT. Clip to the WDPA polygon, and keep a separate "landscape" tier for the 10 km buffer. Don't silently claim Mudumalai's birds.

---

### Stream 2 — The gazetteer (the biggest single unlock)

**Fixes:** places 15% → 90%. This is the highest-value stream in the entire plan and it is the easiest.

**OpenStreetMap Overpass API** returns every named feature in the bounding box: villages, hamlets, peaks, streams, springs, tracks, temples, forest rest houses, check dams. Expect **800–2,000 named toponyms**.

```
[out:json][timeout:180];
(
  node["place"](11.48,76.83,11.82,77.46);
  node["natural"~"peak|spring|water"](11.48,76.83,11.82,77.46);
  way["waterway"](11.48,76.83,11.82,77.46);
  way["highway"~"track|unclassified|tertiary"](11.48,76.83,11.82,77.46);
  node["amenity"~"place_of_worship"](11.48,76.83,11.82,77.46);
  relation["boundary"="protected_area"](11.48,76.83,11.82,77.46);
);
out body geom;
```

Then join to:

- **Census of India 2011** village directory — population, literacy, SC/ST proportion, amenities, for every village in Sathyamangalam, Thalavady and Gobichettipalayam taluks. Downloadable as tables; join on village code.
- **LGD (Local Government Directory)** — official village/panchayat codes, the canonical join key.
- **WDPA / Protected Planet** — the reserve polygon.
- **Bhuvan / FSI** — forest cover class per location.
- **Wikidata SPARQL** — anything already modelled, so you link out rather than duplicate.
- **The management plan's beat and stream lists** — 40+ named streams, 25+ ponds, all four ranges' beats. Reconcile OSM names against these; every mismatch is a data-quality finding.

**Output:** one `place` entity per toponym with coordinates, type, admin range, census join, and a stable slug. **This alone is several hundred pages of unique, zero-competition search surface.**

**Status as of 21 Aug 2026: still blocked** — needs a management decision to treat Overpass as an authenticated API client rather than a crawler bound by its robots.txt. See the Harvest Engine planning docs for the current build order (this is Stage 3 in the new plan).

---

### Stream 3 — Scientific literature (the bibliography)

**Fixes:** bibliography 15% → 85%. Target 400+ entries, not 150.

**Use scholarly APIs, not Google Scholar.** Scholar prohibits automated access and will block you; the open alternatives are better anyway.

| Source | Why | Access |
|---|---|---|
| **OpenAlex** | 250M+ works, no key, generous rate limit. The workhorse. | `api.openalex.org/works?search=Sathyamangalam` |
| **Crossref** | DOI metadata, references, funders | `api.crossref.org/works?query=…` |
| **Semantic Scholar** | Citation graph — find papers that cite the STR tiger paper | `api.semanticscholar.org/graph/v1` |
| **CORE** | 200M+ open-access full texts | `core.ac.uk/services/api` |
| **Unpaywall** | Legal OA link for any DOI — critical for "link, don't host" | `api.unpaywall.org/v2/{doi}` |
| **Shodhganga** | Every Indian PhD thesis, **OAI-PMH harvestable** | `shodhganga.inflibnet.ac.in/oai/request` |
| **BHL** | Biodiversity Heritage Library — historic natural history literature | `biodiversitylibrary.org/api3` |
| **Internet Archive** | Scholar + text search APIs | `archive.org/advancedsearch.php` |
| **PubMed / EuropePMC** | Zoonoses, anthrax, HD, FMD in the cattle-wildlife interface | free APIs |

**Query set** — run every one against every source, and store the union:

```
"Sathyamangalam" · "Satyamangalam" · "Sathyamangalam Tiger Reserve"
"Moyar" · "Moyar valley" · "Bhavani river" · "Bhavanisagar"
"Talamalai" · "Talaimalai" · "Hasanur" · "Hassanur" · "Bargur"
"Nilgiri Biosphere Reserve" + Tamil Nadu
"Erode district" + (forest | wildlife | biodiversity | tribal)
"Soliga" · "Sholaga" · "Irula" · "Oorali" · "Kurumba" + forest rights
"Senna spectabilis" + India · "Lantana camara" + Western Ghats
"Gyps indicus" + Moyar · vulture + "Nilgiri"
"human-elephant conflict" + Tamil Nadu
```

**Then a second pass by citation graph:** take the 20 core papers, pull everything that cites them and everything they cite, two hops deep. That's how you find the theses and regional-journal notes no keyword search surfaces.

**Legal line:** harvest *metadata* freely. Store full text only where the licence permits (OA, CC, public domain). For everything else, store the DOI, the abstract, and an Unpaywall link. **Do not scrape ResearchGate or Academia.edu** — both prohibit it in their terms. Where a document is only on Academia (like the management plan), download it manually as a human, once.

---

### Stream 4 — Historical full-text mining

**Fixes:** history 25% → 75%. One week of work for an outsized return.

Six Internet Archive `_djvu.txt` files, each 5–20 MB:

| Item ID | Document |
|---|---|
| `in.ernet.dli.2015.105586` | Nicholson, *Manual of the Coimbatore District*, 1887 |
| `dli.csl.3313` | *Madras District Gazetteers: Coimbatore*, W. Francis, 1908 |
| `in.ernet.dli.2015.280833` | *Madras District Manuals: Coimbatore* Vol. II |
| `dli.ministry.08433` | *Madras District Gazetteers: Coimbatore*, Baliga |
| `in.ernet.dli.2015.177459` | *Madras District Gazetteers: Coimbatore* Vol. II |
| `journeyfrommadra01hami` | Buchanan, *A Journey from Madras…*, 1807 |

**Method:** download whole (Internet Archive explicitly permits this; use their API and identify yourself in the User-Agent), then run a local fuzzy-match pass. OCR of 19th-century type is dirty — exact matching will miss half of it. Use Levenshtein distance ≤2 against this variant list:

```
Satyamangalam · Sathyamangalam · Sattiamangalam · Sittimungulum
Sattiamungulum · Sathinungulum · Satyamanagalam
Talamalai · Talaimalai · Thalamalai
Hasanur · Hassanur · Hasanoor
Bargur · Burgoor · Bargoor
Danaiken kottai · Danayakankottai · Danaikenkottai
Gajalhatti · Gejjalhatti · Gujjalhatti
Moyar · Mayar · Moyaar
Bhavani · Bowani · Bhawani
Sholaga · Sholagar · Soliga · Solega
Irula · Irular · Iruler
Kurumba · Kurumbar · Curumba
Sandal · sandalwood · Santalum
```

For every hit: extract ±500 words of context, page reference, and store as a `historical_passage` entity linked to the place and topic it mentions. **Expect several thousand words of genuinely uncited primary material.** Then have a human read and annotate — this is the part where the machine hands off.

**Also in this stream:** Endangered Archives Programme EAP314 and EAP458 (free, downloadable), British Library India Office photographs (public domain), and the National Library of Scotland Survey of India sheets for georeferencing.

---

### Stream 5 — Government and legal documents

**Fixes:** governance 50% → 85%. This is a straight port of your existing `crawler.js`.

| Target | What |
|---|---|
| `sathytiger.tn.gov.in` | GOs, tenders, notifications, gallery |
| `forests.tn.gov.in` | Policy PDFs, TN PIPER, wildlife circulars, permit fee schedules |
| `tamilnaduarchives.tn.gov.in` | Government Orders — **GOMS No.122 (2008) and the 2013 notification live here** |
| `ntca.gov.in` | Tiger reserve notifications, MEE reports, AITE cycles |
| `wii.gov.in` / `mee-tr.wii.gov.in` | TCP references, MEE scores, downloads |
| `moef.gov.in` / Parivesh | Clearances, EIA documents for projects in the landscape |
| `erode.nic.in` | District administration, forest, census |
| `indiankanoon.org` | **Madras HC judgments on invasive removal, grazing, FRA, night traffic** — free API |
| `egazette.gov.in` | Gazette notifications |

**Method:** exactly your existing pattern. Crawl listing page → extract PDF links → hash → dedupe → R2 → Workers AI `toMarkdown` → queue. Add an OCR fallback for scanned GOs (Workers AI handles text-native PDFs; scans need Tesseract or a vision model).

**Rate limits:** your 1,500 ms sleep is right for `.gov.in`. Keep it. Identify your bot honestly in the User-Agent with a contact URL. Respect `robots.txt` — write a checker into the orchestrator, not into each crawler.

---

### Stream 6 — Remote sensing and environmental layers

**Fixes:** climate 60% → 90%, plus the entire live-data proposition.

| Layer | Source | Cadence |
|---|---|---|
| Sentinel-2 surface reflectance, NDVI, NDWI | Google Earth Engine | 5-day |
| *Senna* flowering phenology composite | GEE, tuned to the yellow signal | seasonal |
| Post-clearance regrowth change detection | GEE, your existing pipeline | monthly |
| Landsat archive back to 1984 | GEE | annual composites |
| Active fire + burnt area | FIRMS (MODIS/VIIRS) + FSI alerts | daily |
| Rainfall, temperature | CHIRPS, ERA5-Land, IMD | daily/monthly |
| Forest cover change | Hansen GFC, FSI ISFR | annual |
| Elevation, slope, aspect | SRTM / Copernicus DEM | static |
| Historical land cover | **NLS Survey of India sheets, georeferenced** | one-off |

**Run GEE exports on a schedule via a service account**, write COGs and GeoJSON to R2, serve as tiles. This is the stream that makes the site infrastructure rather than an encyclopedia — and it's the one only you can build. **Note:** this stream overlaps with the separate `senna-regrowth-verification` project (see that project's own docs) — coordinate rather than duplicate the GEE work.

---

### Stream 7 — Media and journalism

**Fixes:** contemporary record and the events timeline.

Harvest **metadata and links, never full text.** News copy is under copyright; a headline, date, byline, outlet, URL and a 25-word summary is fair and sufficient.

- **GDELT** — free global news event database with API; query by location and theme.
- **Mongabay India** — CC BY-NC-ND, so you may republish with attribution. Check each piece.
- **The Hindu, Indian Express, Times of India, Deccan Herald, The Federal** — link + metadata only.
- **Tamil media** — Dinamalar, Dinamani, Vikatan. **Critical: the Tamil record of this landscape is nearly absent from English-language coverage.** This stream alone materially moves the Tamil-content number.
- **Google News RSS** for standing queries in English and Tamil.

Output: a `news_event` entity feeding a live "what's happening" timeline — the thing that gives people a reason to come back.

---

### Stream 8 — The management plan (not scraping, but the biggest single win)

**+8 coverage points from one document.** Download the 391-page PDF once, manually, as a human. Then:

1. Split into sections and appendices.
2. Table extraction via `camelot` / `tabula` for the structured appendices — species lists, village inventories, budget tables, stream registers.
3. Vision-model pass for scanned pages and maps.
4. **Human review of every extracted table.** This is the point where accuracy matters more than speed; a wrong species list on a site claiming to be authoritative is worse than no species list.
5. Emit as structured open data with a Zenodo DOI, licensed CC BY.

That last step matters: **releasing the management plan's appendices as clean open data is itself a citable contribution**, and it's the kind of thing that gets you cited by people who then link to the atlas.

---

## 3. Data model

Nine entity types. Everything is one of these, everything has provenance.

```sql
place(id, slug, name_en, name_ta, lat, lon, precision, type,
      range, division, osm_id, census_code, wikidata_id, elevation)

taxon(id, slug, scientific_name, authority, rank, common_en, common_ta,
      common_soliga, iucn, wpa_schedule, gbif_key, endemic_flag)

occurrence(id, taxon_id, place_id, lat, lon, coord_uncertainty, date,
           basis, recorded_by, source, source_id, licence, public_precision)

document(id, type, title, authors, year, doi, url, oa_url, licence,
         full_text_permitted, r2_key, abstract)

historical_passage(id, document_id, page, matched_term, text, annotation,
                   place_ids[], topic_ids[])

legal_instrument(id, kind, number, date, title, issuing_body, pdf_r2_key, summary)

news_event(id, date, headline, outlet, url, summary_25w, place_ids[], topic_ids[])

observation_layer(id, kind, date_from, date_to, r2_key, format, method, params)

claim(id, subject_type, subject_id, field, value, source_id, retrieved_at,
      confidence, conflicts_with)
```

**`claim` is the important one.** Every disputed fact — the 793.49 vs 917.27 km² core area, tiger counts across methods, fee schedules — is stored as competing claims with sources, not resolved silently. The site renders both and says so. **That single design decision is what makes this a reference work rather than another blog.**

**Note (21 Aug 2026):** the current Harvest Engine rebuild extends this schema with a `reserve` table and `reserve_id` on every row, so the same design can template to other protected areas later — see the Harvest Engine architecture docs.

---

## 4. Legal and ethical rules — read before writing code

**Do:**
- Respect `robots.txt`. Build the check into the orchestrator.
- Identify the bot honestly: `SathyamangalamAtlas/1.0 (+https://sathyamangalam.org/bot; contact@…)`.
- Rate-limit hard. 1,500 ms between requests to `.gov.in`; 1 req/sec to APIs; honour `Retry-After`.
- Cache aggressively. Never re-fetch what hasn't changed.
- Use official APIs wherever one exists, even if scraping would be faster.
- Store licence per record and render attribution.

**Don't:**
- Scrape ResearchGate, Academia.edu, Google Scholar, or anything whose terms prohibit it. Where a document exists only there, download it manually, once.
- Republish paywalled full text. Metadata, abstract, and an Unpaywall link.
- Republish news copy. Headline, link, 25-word summary.
- Publish precise coordinates for tigers, leopards, vultures, pangolins, star tortoises or sandalwood. **Enforce this in code**: an `occurrence.public_precision` field that rounds sensitive taxa to 5 km before anything reaches the front end. Not a policy document — a database constraint.
- Scrape anything about individual people. Community material comes through consent, not crawlers.
- Ingest photographs of tribal people from any source without documented consent.

---

## 5. Phasing and coverage math

**Phase 1 — Foundation (week 1).** Source registry, orchestrator, R2/D1/KV/Queue wiring, robots checker, provenance schema, STR polygon fetched and frozen. *No coverage gain. Skipping this costs you a month later.*

**Phase 2 — Gazetteer (weeks 2–3).** Overpass + Census 2011 + LGD + WDPA + Wikidata. **→ 15% → 34%.** Biggest single jump in the plan; do it first after foundation.

**Phase 3 — Biodiversity (weeks 3–4).** GBIF + eBird + iNaturalist + IBP + IUCN + Xeno-canto, with taxonomic reconciliation. **→ 34% → 48%.**

**Phase 4 — Literature (weeks 4–5).** OpenAlex + Crossref + Semantic Scholar + CORE + Unpaywall + Shodhganga OAI-PMH + citation-graph expansion. **→ 48% → 56%.**

**Phase 5 — Government and legal (weeks 5–6).** Port `crawler.js` across the nine government targets + Indian Kanoon. **→ 56% → 62%.**

**Phase 6 — History mining (week 6).** Six archive texts, fuzzy variant matching, human annotation. **→ 62% → 67%.**

**Phase 7 — Management plan extraction (weeks 6–8).** Manual download, table extraction, full human review, Zenodo release. **→ 67% → 75%.**

**Phase 8 — Remote sensing (weeks 7–10, parallel).** GEE scheduled exports, FIRMS/FSI fire, CHIRPS/ERA5 climate, NLS historical georeferencing. **→ 75% → 79%.**

**Phase 9 — Field and archive (continuous, from week 1).** Gass Forest Museum, Tamil Nadu Archives, the Field Director's office, and your camera. **→ 79% → 82%+.**

**Start Phase 9 on day one, not at the end.** Relationships and archive visits have long lead times; the crawler doesn't care whether you've written to the Field Director yet, but month four does.

---

## 6. What this still won't reach — say it on the site

At 82% you will still be missing:

- **Oral history and Soliga/Irula ethnobotanical knowledge.** Consent-based fieldwork only.
- **Unpublished working plans and compartment maps.** Department relationship only.
- **Current staff, budget and visitor figures.** RTI request or the department's goodwill.
- **Tamil manuscript and local print material.** Roja Muthiah Library, in person.
- **Original photography.** Your camera. Nothing else.
- **Anything about the last ~18% that simply isn't recorded.** Reptiles, amphibians and invertebrates have no baseline — the department's own plan says so. For those, near-zero *is* the state of knowledge, and publishing that honestly is more valuable than pretending otherwise.

A `/data/coverage` page that shows exactly these numbers, per domain, updated automatically — **that transparency is itself a differentiator.** No other reserve resource in India tells you what it doesn't know.

---

## Status note (added 21 Aug 2026)

This plan's Cloudflare architecture recommendation was correct in direction but got superseded in detail — see the Harvest Engine planning docs for the current, agreed build (all-Cloudflare: D1 + R2 + Workers + Cron + Queues + Workers AI, multi-reserve schema, free tier to start). The 8 streams and their priority order above are still the reference for what to build, one stage at a time, inside `harvest-engine/` in the `sathyamangalam-atlas` repo.

## Sources

- [GBIF API](https://www.gbif.org/) · [eBird API](https://ebird.org/) · [iNaturalist API](https://www.inaturalist.org/) · [Xeno-canto](https://xeno-canto.org/)
- [OpenAlex](https://openalex.org/) · [Crossref](https://www.crossref.org/) · [Semantic Scholar API](https://www.semanticscholar.org/product/api) · [CORE](https://core.ac.uk/) · [Unpaywall](https://unpaywall.org/)
- [Shodhganga](https://shodhganga.inflibnet.ac.in/) · [Biodiversity Heritage Library](https://www.biodiversitylibrary.org/) · [Internet Archive](https://archive.org/)
- [OpenStreetMap Overpass API](https://overpass-api.de/) · [Protected Planet / WDPA](https://www.protectedplanet.net/) · [Wikidata Q2226064](https://www.wikidata.org/wiki/Q2226064)
- [Survey of India maps 1870–1966, NLS](https://maps.nls.uk/india/survey-of-india/) · [Historical Survey of India maps, Zenodo](https://zenodo.org/records/3596387)
- [Endangered Archives EAP314](https://eap.bl.uk/project/EAP314) · [EAP458](https://eap.bl.uk/project/EAP458) · [BL India Office photographs](https://commons.wikimedia.org/wiki/Category:British_Library_India_Office_photographs_collection)
- [Tamil Nadu Archives](https://tamilnaduarchives.tn.gov.in/WEB/EN/RECORDS/IMPGO) · [TN Forest Department](https://www.forests.tn.gov.in/) · [STR portal](https://sathytiger.tn.gov.in/) · [WII MEE-TR](https://mee-tr.wii.gov.in/Home/Geospatial/sathyamangalam-tiger-reserve) · [Erode District Forest](https://erode.nic.in/departments/forest/)
- [Management Plan for Sathyamangalam WLS 2010–2020](https://www.academia.edu/8273064/TAMILNADU_FOREST_DEPARTMENT_MANAGEMENT_PLAN_FOR_SATHYAMANGALAM_WILDLIFE_SANCTUARY_2010_TO_2020_) — download manually, do not scrape
- Archive.org full texts: [Nicholson 1887](https://archive.org/stream/in.ernet.dli.2015.105586/2015.105586.Manual-Of-The-Coimbatore-District_djvu.txt) · [Francis 1908](https://archive.org/stream/dli.csl.3313/3313_djvu.txt) · [Madras District Manuals Vol. II](https://archive.org/stream/in.ernet.dli.2015.280833/2015.280833.Madras-District_djvu.txt) · [Buchanan 1807](https://archive.org/details/journeyfrommadra01hami)
