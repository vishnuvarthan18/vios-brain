# Sathyamangalam Atlas — Session Memory
## Full project state as of 13 August 2026

This doc is the handoff record for picking this project back up in any future session — read this first before anything else in the project.

---

## What this project is

An open-reference website for Sathyamangalam (Erode district, Tamil Nadu) — the tiger reserve, the wider landscape, its people, history and wildlife — built to be the single authoritative place online that researchers, filmmakers, tourists and anthropologists all land on. Secondary goal: it's Vishnu's highest-leverage visibility move for his conservation-tech/geospatial career pivot (see other project docs for that broader context).

Domain: not yet registered. **This is the single biggest open blocker** — nothing can go live, rank, or be shown to the Field Director until this happens.

---

## What exists right now, and where

### 1. Research documents (saved permanently in this Project)
- `sathyamangalam/atlas-website-dossier.md` — this file
- `sathyamangalam/source-atlas.md` — full source inventory: colonial gazetteers (Nicholson 1887, Francis 1908), Buchanan's 1807 survey, the Battle of Sathyamangalam (1790, catalogued as "Sittimungulum"), Survey of India historical maps (1870-1966, georeferenced via NLS), the 391-page TN Forest Dept Management Plan 2010-2020 (the single richest document — full species checklists, range/beat tables, climate data, working-circle history)
- `sathyamangalam/harvest-plan.md` — "Operation Full Record": the plan to take data coverage from ~15% to ~80% via automated scraping + manual archive work + fieldwork. Honest ceiling: scraping alone tops out ~62-68%; the rest needs the management plan extraction, physical archive visits (Gass Forest Museum Coimbatore, Tamil Nadu Archives), a Field Director relationship, and original photography.

### 2. The website (v2, single-page HTML — NOT saved to Project, only delivered as a file)
`sathyamangalam-atlas.html` — a complete, modern, dark-mode single-page site with:
- 74-species filterable database (mammals/birds/herps/fish/flora/invasives) with Tamil names, IUCN status, WPA schedule
- Interactive MapLibre gazetteer with 15 places
- 21-event history timeline, 405 AD to 2026, including the 1790 battle, 1807 Buchanan survey, 1887 gazetteer, 1919 sandalwood depot, Veerappan era, TX2 award
- 39-source bibliography, filterable
- Full FRA/Soliga/Irula/Kurumba community section
- Three-track permit guide (visitor/filmmaker/researcher)
- Full SEO schema (JSON-LD WebSite/Place/Dataset/FAQPage/Event), bilingual hreflang
- ⌘K command palette search across everything
- **This file was only ever sent to Vishnu as a downloadable file card — if he hasn't saved it locally, it needs to be rebuilt or re-fetched from chat history.**
- **UPDATE (later session):** the full repo, including this site rebuilt as multi-page, was recovered from a GitHub zip on 21 Aug 2026 — see `sathyamangalam/repo-inventory-2026-08-21.md`.

### 3. The data harvester (Python package — NOT saved to Project, only delivered as a zip: `sathyamangalam-harvester.zip`)
`atlas-harvester/` — a full data-acquisition pipeline, 15 harvesters across 7 phases:
1. `boundary` — WDPA reserve polygon
2. `gazetteer` — OpenStreetMap Overpass + Census 2011 join (**highest single coverage jump**)
3. `gbif`, `inaturalist`, `ebird`, `iucn` — biodiversity occurrences
4. `openalex`, `crossref`, `citations`, `unpaywall`, `shodhganga` — literature/bibliography
5. `govdocs`, `caselaw` — government PDFs, Madras HC judgments
6. `archive` — fuzzy OCR mining of colonial texts (handles spelling corruptions like "Sittimungulum")
7. `news` — Google News RSS, English + Tamil, metadata only

**Design principles baked into the code, not just documented:**
- Every fact has a source, retrieval timestamp, and licence (provenance-first)
- Sensitive-species coordinates (tigers, leopards, elephants, vultures, pangolins, sandalwood, star tortoises, pythons) are automatically coarsened to ~5km grid **at the database insert layer** — cannot be bypassed by a future harvester
- Conflicting facts (e.g. reserve core area: 793.49 km² per TNFD vs 917.27 km² per Wikipedia) are preserved as rival `claim` rows, never silently resolved — the site is designed to show both
- robots.txt obeyed by default, honest User-Agent required, no scraping of ResearchGate/Academia.edu/Google Scholar (all prohibit it), no news body text or paywalled full text stored

**Bug found and fixed (13 Aug):** SQLite `UNIQUE(url, sha256)` and `UNIQUE(kind,title,year)` constraints silently failed to dedupe because SQL treats every `NULL` as distinct — caused exact-doubling of claims and drift in document counts on repeat `seed` runs. Fixed with explicit dedup keys + an automatic in-place migration (`atlas/migrate.py`) that self-heals any already-corrupted database on next open. 86/86 offline tests pass. Verified fix on the real CLI: `seed` run twice now produces zero new duplicates.

**Current state of the actual data (at the time this doc was written):** only seed data loaded (74 species, 19 places, 39 documents, 82 claims). Coverage: **2.0%** against the honest weighted targets in `atlas/coverage.py`. No live harvest phases had been run yet as of this doc.

**IMPORTANT — later finding:** as of 21 Aug 2026, the harvester Python code described above was confirmed **not present anywhere in the GitHub repo** — only its output (the populated `atlas.db`) survived. It was apparently built fresh in a temporary session each time and never committed. See `sathyamangalam/repo-inventory-2026-08-21.md` and the Harvest Engine rebuild plan.

### Decision made this session: outsourcing the harvest run
Vishnu is going to hand the harvester off to another AI agent to actually execute (not Gemini web chat — confirmed that doesn't have the shell/filesystem/sustained-network access this needs; something like Claude Code, Gemini CLI, or a coding agent on his own machine does). A complete standalone prompt for that handoff was written earlier in this conversation (system/task framing, exact run order, hard rules — no robots.txt bypass, no scraping prohibited sites, no touching the sensitive-coordinate logic, rate limits fixed). **That prompt should be reconstructed/reused when the handoff actually happens** — worth asking Vishnu to paste his agent's final report back so a fresh session can pick up the numbers.

---

## Timeline estimate given to Vishnu (for the harvest run itself)

Rough wall-clock if another AI/agent runs all 7 phases attentively, assuming API keys are set (eBird, IUCN, ATLAS_EMAIL) and no major blocking errors:

- **Phase 1 (boundary) + Phase 2 (gazetteer):** under an hour of runtime, but this is the highest-value phase — expect 800-2,000 place records from one Overpass query. Coverage jumps from 2% to roughly 25-30%.
- **Phase 3 (biodiversity — GBIF/iNat/eBird/IUCN):** several hours, mostly GBIF pagination (rate-limited at 0.4s/request) and iNaturalist (1s/request). Could run overnight.
- **Phase 4 (literature — OpenAlex/Crossref/citations/Unpaywall/Shodhganga):** several hours; Shodhganga OAI-PMH harvesting in particular can be slow (2s/request, potentially hundreds of pages).
- **Phase 5 (govdocs/caselaw):** an hour or two; small target list, but caselaw needs a manual Indian Kanoon token first or it's a no-op.
- **Phase 6 (archive text mining):** an hour or two once the six archive.org texts are downloaded (large files, one-time).
- **Phase 7 (news):** under an hour.

**Total active engineering/running time: roughly 1-2 full days if run back-to-back with attention paid to errors.** But realistically, spread across a normal week with the agent checking in, debugging API hiccups, and Vishnu reviewing — **budget 3-5 days** for the full harvest to complete and get to the ~60-65% automated ceiling. That does NOT include the manual-only work (management plan PDF extraction, archive visits, Field Director outreach, photography) which the harvest plan estimates in weeks/months separately.

**When Vishnu returns with the report:** ask for the final `atlas coverage` output and `atlas conflicts` output specifically — that's the fastest way to assess what actually got done and pick up the site rebuild against real numbers instead of the 15-place/74-species seed data.

---

## Immediate next steps (in priority order)

1. **Register the domain.** Blocks the bot's honest User-Agent, the Field Director letter, and anything going live.
2. Vishnu's AI agent runs the harvest phases (in progress / about to start).
3. **Write to the STR Field Director** — should happen in parallel, not after the harvest. Non-commercial framing, links to official portal for bookings, offers to take down anything flagged sensitive.
4. Once harvest data comes back: rebuild the site as **multi-page** (driven by `out/places.json` / `out/species.json` / etc.) rather than the current single-page hardcoded HTML — the export format was deliberately designed to support this jump.
5. Longer-term, manual-only: management plan PDF full extraction, Gass Forest Museum + Tamil Nadu Archives visits, original photography (the one gap automation can never close).
