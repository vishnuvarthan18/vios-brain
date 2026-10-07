# Harvest Engine — Master Build Prompt, Stages 1 through 8

## Working directory — read this first, every session

Work exclusively in `sathyamangalam-atlas-clean`. Before writing any code in any session, confirm:
```
git remote -v   # must show https://github.com/vishnuvarthan18/sathyamangalam-atlas.git
git log --oneline -1
```
If either check fails or looks wrong, stop and report it instead of proceeding.

Never work in `sathyamangalam-atlas-main` — that folder is not a git repository and is being phased out.

## How this prompt works

This covers all remaining stages of Harvest Engine (1 through 8). Work through them **in order, one at a time**. After finishing and self-verifying a stage, commit and push it, update `harvest-engine/PROGRESS.md` (create it if it doesn't exist) with a status entry, then move to the next stage automatically — do not wait for a human prompt between stages unless you hit a blocker described below.

**Only pause and ask a human when:**
- A required API key/credential is missing and there's no keyless workaround (see per-stage notes below for which stages need what).
- A source that was supposed to be freely accessible turns out to require payment, registration approval, or explicit permission you don't have.
- Verification evidence contradicts what the stage's non-negotiable rules require (e.g. sensitive-species coarsening doesn't fire) — do not silently ship broken behavior; stop and report exactly what failed.
- Two of Harvest Engine's existing rules genuinely conflict for a new source and you can't resolve it by inference from Stage 0/1 precedent.

Otherwise, keep going stage to stage without asking for permission to proceed.

## Non-negotiable rules — apply to every stage, every stream, no exceptions

1. **Raw before parse.** Every fetched response's raw body goes to R2 via `saveRaw()` before any parsing happens. Never parse-then-discard raw.
2. **Provenance on every fact.** Every row written to any fact table carries a `source_id`, traceable back to the raw R2 object and the specific API/page response it came from.
3. **robots.txt respected**, via the existing `src/lib/robots.js` checker, for every non-API-key-authenticated fetch. For source registries where an API's own terms of use supersede robots.txt, note that explicitly in that source's `sources/*.json` entry — don't silently bypass.
4. **Rate limiting** tuned to each source's actual documented limits — via `src/lib/rate-limit.js`. Look the real limit up per source; don't guess or reuse a different source's number without checking.
5. **Sensitive-species coordinate coarsening** is a DB-level trigger from Stage 0 — never reimplement in application code. Every stream that writes coordinates must verify the trigger actually fires for a sensitive-species test case before being marked done.
6. **Idempotency.** Reuse `src/lib/hash.js` and the `job_run` dedupe pattern — no stream may double-write on a second run.
7. **Error boundary.** Every stream run wrapped in `withJobRun()` so failures land as a `job_run` row with `errors_json`, never a silent crash.
8. **Never scrape sources whose terms prohibit it** — Google Scholar, ResearchGate, Academia.edu are explicitly off-limits; use the open alternatives listed per stage instead. Where a document exists only on a prohibited site, flag it for manual human download rather than scraping it.
9. **Never republish paywalled full text or news copy** — metadata + link + short summary only, per stage notes below.
10. **Reuse Stage 0/1 infrastructure** — `src/lib/robots.js`, `rate-limit.js`, `hash.js`, `raw-storage.js`, `job-run.js`, `coarsen.js`, the `src/streams/index.js` registry pattern, `sources/*.json` shape, and the dashboard. Do not duplicate or reinvent these per stage.

## Definition of done, every stage (self-verify before moving on)

For every stream added in a stage: a real run against the live source (not mocked), a `job_run` row showing success, real fact rows written with correct provenance, R2 raw copy confirmed present via a bucket listing, and — for any stream touching occurrence/coordinate data — confirmed coarsened coordinates for a sensitive-species test case. Capture this evidence (row counts, job_run rows, R2 listing, before/after coords) in `PROGRESS.md` under that stage's entry. Do not mark a stage done on a prose summary alone.

---

## Stage 1 — Biodiversity (in progress / may already be done when you read this)

GBIF, eBird, iNaturalist. GBIF and iNaturalist need no key. eBird needs `EBIRD_API_KEY` — already set in `.dev.vars` and as a Cloudflare Worker secret as of this writing; if it's missing, pause and ask.

Also add, if not already covered: **India Biodiversity Portal** (public API, no key, yields Tamil vernacular names) and **Xeno-canto** (public API, no key, bird/frog audio). **IUCN Red List API** needs a free non-commercial key from apiv3.iucnredlist.org — if missing, pause and ask.

Scope all queries to the Sathyamangalam Tiger Reserve WDPA polygon (fetch and freeze this once, reuse everywhere) plus a defined buffer zone — do not silently include neighboring reserves (Mudumalai, BRT) in results.

## Stage 2 — Scientific literature

Sources, all keyless/public: OpenAlex, Crossref, Semantic Scholar, CORE, Unpaywall, Shodhganga (OAI-PMH), Biodiversity Heritage Library, Internet Archive scholar/text search, PubMed/EuropePMC.

Run the full query set from `sathyamangalam/harvest-plan.md` Stream 3 (place names, tribal names, invasive species terms, etc.) against every source, store the union. Second pass: citation graph, two hops from the 20 core papers already identified. Store metadata + DOI + abstract + Unpaywall OA link only — full text only where licence explicitly permits (OA/CC/public domain). Never scrape ResearchGate, Academia.edu, or Google Scholar.

## Stage 3 — Gazetteer

OpenStreetMap Overpass API (keyless), joined to Census of India 2011 village directory, LGD codes, WDPA polygon, Bhuvan/FSI forest-cover class, Wikidata SPARQL, and the management plan's beat/stream lists (manual reference data, not a live fetch).

**Known blocker from prior planning:** this stream was flagged as blocked pending a decision on whether to treat Overpass as an authenticated API client (higher, negotiated rate limits) versus a plain crawler bound by Overpass's own robots.txt/fair-use policy. If you hit this, pause and ask rather than guessing — it changes the rate-limit and identification approach.

## Stage 4 — Government and legal documents

Targets: sathytiger.tn.gov.in, forests.tn.gov.in, tamilnaduarchives.tn.gov.in, ntca.gov.in, wii.gov.in/mee-tr.wii.gov.in, moef.gov.in/Parivesh, erode.nic.in, indiankanoon.org (check if API key required — free tier historically didn't need one, verify current terms), egazette.gov.in.

This is a port of the existing (separate, do-not-touch) `crawler.js` pattern, not a rebuild: listing page → extract PDF links → hash → dedupe → R2 → text extraction → queue. Add OCR fallback (Workers AI or equivalent) for scanned Government Orders. 1500ms rate limit for `.gov.in` domains specifically — do not use a faster default. Identify the bot honestly in the User-Agent with a real contact reference.

## Stage 5 — Historical full-text mining

Internet Archive `_djvu.txt` full texts (six specific volumes — Nicholson 1887, Francis 1908, Madras District Manuals, Buchanan 1807, etc. — see `sathyamangalam/source-atlas.md` for exact item IDs). Download whole per Internet Archive's terms (identify the bot). Fuzzy-match (Levenshtein ≤2) against the spelling-variant list in `sathyamangalam/harvest-plan.md` Stream 4. Store each hit as a `historical_passage` row with page reference and ±500 words context, linked to `place`/topic where identifiable. This stage produces raw extracted material for human annotation — don't try to auto-annotate meaning, just extract and link accurately.

## Stage 6 — Remote sensing and environmental layers

Sentinel-2/Landsat via Google Earth Engine (needs a GEE service account — Google Cloud project + service account credentials, more setup than a simple key; pause and ask if not yet provisioned), FIRMS fire data (free key from firms.modaps.eosdis.nasa.gov — pause and ask if missing), CHIRPS/ERA5-Land/IMD rainfall data, Hansen Global Forest Change, SRTM/Copernicus DEM.

**Coordinate with, don't duplicate:** the separate `senna-regrowth-verification` project already does GEE work for *Senna* regrowth detection — check that project's docs before building overlapping GEE export logic.

## Stage 7 — Media and journalism

GDELT (keyless API), Mongabay India (CC BY-NC-ND, check per-article), major English outlets (metadata + link only, never full text), Tamil media (Dinamalar, Dinamani, Vikatan — same metadata-only rule), Google News RSS for standing English + Tamil queries. Output feeds a `news_event` table. Never store full article text — headline, outlet, date, URL, and a ≤25-word summary only.

## Stage 8 — Management-plan extraction

This is different from the other stages: it operates on one already-downloaded document (the 391-page management plan PDF — confirm it's already been manually downloaded; if not, pause and ask a human to do that download, since Academia.edu can't be scraped). Table extraction via a library like camelot/tabula for structured appendices (species lists, village inventories, stream registers). Vision-model pass for scanned pages/maps. Flag every extracted table for human review rather than auto-publishing — accuracy matters more than speed here, per the standing project rule that a wrong species list is worse than no species list. Emit as structured open data.

---

## API keys/credentials needed across all stages — for Vishnu, not the agent

Collect these ahead of the stage that needs them (agent will pause and ask if one's missing when it gets there):

- **IUCN Red List API key** — apiv3.iucnredlist.org, free, non-commercial (Stage 1)
- **Indian Kanoon** — verify current terms, may or may not need a key (Stage 4)
- **Google Earth Engine service account** — Google Cloud project + service account JSON, more involved than a simple key (Stage 6)
- **FIRMS API key** — firms.modaps.eosdis.nasa.gov, free (Stage 6)

Everything else across all 8 stages is keyless/public.
