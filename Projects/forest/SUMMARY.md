---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: forest

Claude project name: "forest". It holds 4 linked threads:
1. Forest-tech career plan (how [[People/Vishnu]] gets field work in forests and mountains using tech skills). Main career page: [[Projects/career/SUMMARY]].
2. **Sathyamangalam Atlas** — the main build. A website + database that aims to be a full, sourced reference for every species and place in Sathyamangalam Tiger Reserve.
3. Senna regrowth verification — a [[Tools/Google Earth Engine]] script that checks if invasive Senna grows back after clearing.
4. Forest job crawler — a [[Tools/Cloudflare]] Worker that watched .gov.in sites for GIS/IT jobs (stopped, deleted 8 Sep). Details in [[Projects/career/SUMMARY]].

Note: [[Projects/india-data-atlas/SUMMARY]] links to `Projects/sathyamangalam-atlas/SUMMARY`. That Atlas lives inside this page. The all-India "Ecotourism Atlas" is a separate project (india-data-atlas), not this one.

## 1. What this project is
- **Goal (non-negotiable):** work in forests and mountains, field-based, using tech skills. Started 10 Aug 2026.
- **Career plan (locked 12 Aug):** no permanent government job. Do contract/project work at WII, IFGTB, FSI, NRSC, SACON, DGRE. Study [[Companies/IGNOU]] M.Sc. Geoinformatics (MSCGI) by distance from January 2027. Build a public portfolio first.
- **Sathyamangalam Atlas:** one connected database + site at `sathyamangalam.online`. Every plant, animal and place gets its own page with facts, sightings, trends, threats, photos, Tamil/local names, sources, and an honest "how complete is this" label. Disputed numbers are shown side by side, never silently picked.
- Data comes from a **harvest engine** (Cloudflare Worker + D1 database, ~29–35 "streams" pulling from [[Companies/GBIF]], [[Tools/iNaturalist]], [[Tools/eBird]], [[Tools/OpenStreetMap]], [[Companies/Wikidata]], [[Companies/data.gov.in]], Crossref, Europe PMC, [[Companies/OpenAlex]], FIRMS ([[Companies/NASA]]), [[Tools/Google Earth Engine]], news sites, government sites).
- Site reads a separate SQLite file (`atlas.db`). D1 → atlas.db sync is a manual script.
- Out of scope for now: new fieldwork, interviews, oral history.
- Work was mostly done in [[Tools/Claude Code]] and Cowork sessions on Vishnu's Mac, plus [[Tools/Claude]] chat. Session notes were saved as project docs.

## 2. Status now (as of 2026-10-06)
- **Status: active.** Work resumed on the personal Mac: 2026-10-03 data refresh deployed to the dev site; 2026-10-06 all work saved in one local commit on `main` (2,940 files) — **not pushed** (main 4 ahead of origin/main). See [[Projects/forest/DEV-LOG]].
- **Data now:** 92 places, 2,664 species, 78,467 occurrences, 928 public documents (+6,632 other-tier, 1,424 unreviewed). Coverage 32.5%.
- **Dev site** `dev.sathyamangalam.online`: refreshed 2026-10-03 (2,911 files). Full unfiltered site with generated detail pages: 2,664 species, 92 places, 83 history pages; new sections history/land/life/credits. People = honest empty page.
- **Public site** `sathyamangalam.online`: not updated since 10 Sep; only Home, About, Contact are real; other pages "under construction" (rule set 8 Sep). The domain was suspended on 2 Sep 2026 for an unverified registrant email — needs fix (not sure if fixed).
- **Harvest engine:** production Worker is version b1cd0cce (sensitive-species fix, 8 Sep). Later fixes (dedup freeze, lgd paging, retries, stale-run reaper, stream_health, DLQ alert) are on branch `deploy/sensitive-species-fix-verify`. 17 fix commits were deployed 10 Sep with crons back on (3x/day: 03:00, 11:00, 19:00 UTC) — not sure the cron is running now (paused 5 Sep; a 7 Sep deploy re-attached the schedules).
- **Coverage 10 Sep:** taxa 2,664 (100%), occurrences 78,467 (100%), places 7.7%, history 0.2%, legal 1.7%, news 0.2%, layers/media/community 0%.
- **Biggest gap:** places/gazetteer (92 places vs 800–2,000 target).
- **Career:** see [[Projects/career/SUMMARY]]. IGNOU MSCGI apply window ~16 Dec 2026 – 31 Jan 2027. Portfolio (Senna notebook, WILDLABS post) — not sure if shipped.
- **Outreach:** Tier 1 (12 emails + 6 WhatsApp) sent 18 Aug. Tier 2 (11 drafts, 28 Aug) — not sure if sent.
- **Job crawler:** stopped 10 Aug; its Cloudflare queues deleted 8 Sep.

## 3. Next steps
1. Push `main` to GitHub; decide when to deploy to production (needs Vishnu's go-ahead).
2. Check if the harvest-engine cron is running or paused.
3. Fix the domain suspension (verify the registrant email) if still open.
4. Production coarsening gate for sensitive coordinates; backfill 3 unflagged taxa and 6 exact-coordinate rows in production.
5. 1,839 synced documents have no relevance tier (not shown on site) — tier them.
6. Delete stray Preview deployment c243affa on Cloudflare. Back up `data/db-backups/` (134 MB, not in git).
7. Check the OpenAlex one-term-per-run fix finishes a full ~20-term chain, and confirm its commit is in git (was due 14 Sep).
8. Check crossref / europepmc / ia-scholar stay clean; check if management-plan / unpaywall runs get stuck every cycle.
9. Confirm `DATA_GOV_IN_API_KEY` is set as a secret, then grow places via LGD/Census (biggest win).
10. Build the new CORE stream (name it `core-oa` or similar — `core` is taken).
11. Fix the 3 "conflicts" (tiger/elephant/leopard): they are counts from different years, not real disputes. Make conflict check date-aware.
12. Run the manual sync → export → build → deploy chain when new data should go live.
13. (career) Call IGNOU Regional Centre Madurai to confirm MSCGI January intake (was planned for September).
14. (career) Set reminder 15 Dec 2026; apply to MSCGI from 16 Dec.
15. (career) Ship portfolio: Senna/GEE notebook public + WILDLABS post; gazetteer dataset with a DOI.
16. Decide on Tier 2 outreach emails.
17. Before 1 Nov 2026: move FIRMS to NOAA-21/20 (Suomi NPP retires). Before 31 Dec 2026: BHL API v2 retires.
18. Rotate keys that were exposed (see section 8).

## 4. Decisions
- 2026-08-10 — Job crawler: drop PI contact email from the data — scraping personal emails is risky under India's DPDP Act #decision
- 2026-08-10 — Job crawler: stay on Cloudflare free tier; use Queues instead of paid Workflows #decision
- 2026-08-10 — Job crawler: remove the "admin role" regex; cap LLM batches at 5–8 docs — avoid missing mixed postings, keep output reliable #decision
- 2026-08-12 — No permanent government job (Forester, ACF, Scientist-B out). Contract/project posts stay in — dislikes the closed circle, not the pay #decision
- 2026-08-12 — Degree = IGNOU M.Sc. Geoinformatics (MSCGI), distance, part-time; drop M.Sc. Environmental Science — it only served ICFRE rules #decision
- 2026-08-12 — Join January 2027 intake, not July 2026 — use Aug–Dec to build a portfolio #decision
- 2026-08-12 — First portfolio project: invasive species (Senna/Lantana) mapping recommended — data, tools and gap all within reach (not sure it was formally picked) #decision
- 2026-08-14 — Hold off Astro migration — the bottleneck is data, not tooling #decision
- 2026-08-14 — GitHub repo is the single source of truth; office Mac is pull-only — two Macs, two Claude accounts #decision
- 2026-08-21 — Genomics ideas: animal/wildlife only, no plant genetics — Vishnu corrected scope #decision
- 2026-08-24 — WDPA API ruled out — India withholds ~900 protected areas #decision
- 2026-08-25 — Conflicting figures: show both, no winner; `superseded_by` only when the same authority corrects itself #decision
- 2026-08-25 — Pause Atlas building until the comparable-projects deep research returns #decision
- 2026-08-26 — Career plan confirmed by research, not reopened (Kunte model: institution salary + own body of work) #decision
- 2026-08-26 — Do the gazetteer (with DOI) before People's Biodiversity Register work — PBR has consent problems #decision
- 2026-08-26 — Execution-plan research brief deferred on purpose; run only when Vishnu says #decision
- 2026-08-27 — Never merge/push/deploy to `main` or production without Vishnu's explicit go-ahead in that same turn — after an accidental production deploy #decision
- 2026-08-27 — Site goes to production only via `scripts/deploy-prod.sh`; no Cloudflare Pages git integration #decision
- 2026-08-27 — Leave harvest engine running 15 days as-is; same morning, run cron 3x/day #decision
- 2026-09-05 — Pause the cron and do a deep review before any fix — dedup freeze found #decision
- 2026-09-05 — Refetch window 7 days for polling sources; one-shot sources keep infinite window #decision
- 2026-09-05 — WII national-scope reports must not go in the reserve-level legal table #decision
- 2026-09-06 — Coarsen sensitive-species coordinates only on production; dev/staging may keep exact (forest dept permission) #decision
- 2026-09-06 — Scope locked: full record per species/place, one connected database, 4 sections Land/Life/History/People, no new fieldwork #decision
- 2026-09-08 — Production shows only Home/About/Contact; dev site shows everything unfiltered on a separate Pages project #decision
- 2026-09-08 — Dev agent works code-only, no remote commands — there is no staging environment #decision
- 2026-09-09 — Deploy the sensitive-species fix to production engine (approved) #decision
- 2026-09-10 — Skip Email Routing for DLQ alerts for now; comment out `send_email` block #decision
- 2026-09-10 — Deprioritize Shodhganga stream — ~50% timeouts, thesis-only, low yield #decision
- 2026-09-10 — Register a free OpenAlex API key and build a new CORE stream — free tiers are enough #decision
- 2026-09-06 — Place match key is wikidata → osm → slug; merged Sathyamangalam/Satyamangalam duplicate (93 → 92 places). #decision


## 5. Timeline
- 2025-07 — Background: letters to [[Companies/ATREE]], [[Companies/Keystone Foundation]], NCF, WWF India and others.
- 2026-06-14 — Chat on the Nilgiri biosphere (personal account).
- 2026-08-10 — Project starts. Career context doc written. Forest job crawler built and deployed on [[Tools/Cloudflare]] with [[Tools/Wrangler]] + [[Tools/Telegram]] alerts; hit 404s and subrequest limit; stopped.
- 2026-08-11 — Six-track plan (Forester, ACF, Scientist, tech, NGO, naturalist).
- 2026-08-12 — No-government decision. IGNOU MSCGI, January 2027 intake. Market context saved.
- 2026-08-13 — WILDLABS profile plan written. Locked plan saved.
- 2026-08-14 — Atlas repo pushed to [[Tools/GitHub]] (private). Senna regrowth repo pushed. Data bundle: 89 places, 1,893 taxa, 74,900 occurrences.
- 2026-08-17 — Sathyamangalam Record launched at sathyamangalam.online (1,893 species/taxa). Real page content set aside; 7 pages became "under construction" stubs.
- 2026-08-18 — Outreach Tier 1 sent: 12 emails + 6 WhatsApp to STR officers and NGOs. Reply from Gowtham (WhatsApp), call from Bharathidasan ([[Companies/Arulagam]]). [[People/Kartik Shanker]] ([[Companies/IISc CES]]) said share with TN Forest Department; ICFRE pointed to WII. Joined eBird and WILDLABS (Aug).
- 2026-08-21 — Harvest engine stages 0–3 done. Genomics ideas explored (elephant TB, elephant gene flow).
- 2026-08-23 — Position check: source coverage 56.8%, real content ~35–40%, gazetteer ~4%.
- 2026-08-24 — 5 API keys resolved; WDPA ruled out.
- 2026-08-25 — Reserve polygon found (791.62 km², OSM relation 4192204). Two databases found diverged. `request_hash` bug found. Building paused.
- 2026-08-26 — Deep research verdict: field is real, solo work can last, grants won't fund a life alone. Two tech audits. Overnight fix plan written.
- 2026-08-26/27 — Accidental merge + production deploy, reverted. Harvest engine deployed; first cron run (19 of 28 streams failed). Cron set to 3x/day.
- 2026-08-28 — Tier 2 outreach: 11 drafts, not sent.
- 2026-09-02 — Worldwide comparables library (602 rows, 585 projects) + "Forest and Wildlife Atlas" overview as one shareable HTML file. Domain sathyamangalam.online suspended (unverified registrant email).
- 2026-09-05 — Dedup freeze found (streams said "success" but stopped fetching). Cron paused.
- 2026-09-06 — Dashboard deployed without go-ahead (incident). Coordinate policy set. Scope locked. Full freeze.
- 2026-09-07 — Overnight run: 9 sensitive species added, documents tiered, places 43→81. 22-table schema migration on dev.
- 2026-09-08 — History (83 pages) + People pages built on dev. Prod/dev split live. Cloudflare cleanup (job-crawler queues deleted). data.gov.in key obtained; WDPA token requested. Stream audit: 13 healthy, 11 silently broken, 5 blocked. Deep fix plan written. Vishnu felt frustrated that nothing finishes.
- 2026-09-08 — Sensitive-species fix deployed to production engine (b1cd0cce). Dedup freeze fixed (7-day window); trust layer (DLQ alert, stream_health).
- 2026-09-09 — Phase 1–3 fixes done on a branch. Overnight run: 56 tests, 5xx retries, 'blocked' status, stale-run reaper; lgd paging bug fixed; management-plan PDF re-uploaded to the real R2 bucket.
- 2026-09-10 — 17 fix commits deployed; crons back on. Shodhganga dropped. OpenAlex/CORE decision. D1 → site sync + deploy (news sync bug fixed). 13 sensitive occurrences re-coarsened in D1. 11 untracked scripts committed. 4 more bugs fixed (upsert crash, OpenAlex key, OpenAlex CPU limit, missing migrations 0004–0007).
- 2026-09-14 — Planned follow-up check (no record it happened).
- 2026-09-28 — (career) TN Forest Dept vacancies page checked (11 notices). Best fit: Technical Assistant (GIS), Fire Control Centre, closing 29 Sep.
- 2026-10-02 — Moved into viOS.
- 2026-10-03 — Exports regenerated; deployed to dev site only (2,911 files). Production untouched.
- 2026-10-06 — All work committed locally on `main` (2,940 files). Not pushed.

## 6. Key facts
**People**
- [[People/Vishnu]] — owner. B.Sc. CS, born 2002, Chithode, Erode. Linked to [[Companies/araCreate Group]].
- [[People/K Rajkumar]] — Field Director, Sathyamangalam Tiger Reserve. Contacted 18 Aug.
- [[People/S Gowtham]] — Deputy Director, Sathyamangalam division. Replied on WhatsApp.
- [[People/Bharathidasan]] — [[Companies/Arulagam]]. Called back; advised going wide.
- DD Hasanur — name not confirmed (Sudhagar or Garg).
- [[People/Kartik Shanker]] — [[Companies/IISc CES]]; replied to outreach.

**Organisations**
- Government / research: [[Companies/TN Forest Department]], [[Companies/WII]], [[Companies/SACON]], [[Companies/IFGTB]], [[Companies/ISRO]] (IIRS/NRSC), [[Companies/IGNOU]].
- NGOs: [[Companies/Keystone Foundation]], [[Companies/ATREE]], [[Companies/NCF]], [[Companies/Arulagam]], [[Companies/Junglescapes]], [[Companies/Technology for Wildlife]].
- Community: [[Companies/WILDLABS]].
- Data sources: [[Companies/GBIF]], [[Companies/Wikidata]], [[Companies/data.gov.in]], [[Companies/OpenAlex]], [[Companies/NASA]] (FIRMS), [[Companies/Google]] (Earth Engine).

**Tools**
- [[Tools/Cloudflare]] (Workers, D1, R2, Queues, Pages), [[Tools/Wrangler]], [[Tools/GitHub]], [[Tools/Python]], [[Tools/Google Earth Engine]], [[Tools/QGIS]], [[Tools/OpenStreetMap]], [[Tools/iNaturalist]], [[Tools/eBird]], [[Tools/Telegram]], [[Tools/Gmail]], [[Tools/WhatsApp]], [[Tools/Claude Code]], [[Tools/Claude]].

**Links and places**
- GitHub: `vishnuvarthan18/sathyamangalam-atlas` (private), `vishnuvarthan18/senna-regrowth-verification` (private).
- Sites: `sathyamangalam.online` (prod), `dev.sathyamangalam.online` (dev), `engine.sathyamangalam.online/dashboard` (engine dashboard), `harvest-engine.mrdecors.workers.dev`.
- Cloudflare Pages projects: `sathyamangalam-atlas` (prod), `sathyamangalam-atlas-dev` (dev). Worker: `harvest-engine`.
- Local folders: `~/sathyamangalam-atlas` (working copy), `~/forest-automation` (job crawler).
- Branches: `main`, `dev`, `deploy/sensitive-species-fix-verify` and others. dev and main have diverged (30+ commits each side).
- Site pipeline: harvest engine (D1) → `scripts/sync_d1_to_atlas.py` (manual) → `atlas.db` → `make export` → `make build` → `make deploy`.
- Reserve facts: core/WLS ~791.62–793.49 km²; Tiger Reserve 1,408.40 km².
- Rules: never deploy without explicit go-ahead; git write commands run in Vishnu's own Terminal; cloud sandbox can't reach Cloudflare API.
- Keys live as Cloudflare secrets and `harvest-engine/.dev.vars` — not here.
- Production Worker version: b1cd0cce. Harvest-engine fixes branch: `deploy/sensitive-species-fix-verify`.
- Dev history: [[Projects/forest/DEV-LOG]]


## 7. Files and documents
- Claude project "forest" holds ~90 docs + 1 PDF. Copies of the first 44 docs also sit in the repo at `docs/claude-project/` (must stay git-ignored — personal plans).
- **Career:** `forest-tech-career-context-and-findings.md` (10 Aug), `claude/current-plan-locked-aug-2026.md` (the live plan), older superseded plans (six-track, no-government, government-scientist, scientist-degree, jump-now-90-day, scientist-money, forest-life-options), `claude/market-context-aug-2026.md`, `claude/wildlabs-profile-setup.md`, `claude/claude-science-wildlife-genomics-research-2026-08-21.md`, notes `parts`, website lists (WILDLABS, Technology for Wildlife, TNC India, "Forest 4.0").
- **PDF:** "Conservation Tech Entry Routes for a CS Graduate in Erode: A Nilgiris Field-First Playbook".
- **Atlas planning:** `harvest-plan.md`, `source-atlas.md`, `atlas-website-dossier.md`, `path-to-70-percent-coverage.md`, `atlas-scope-locked-2026-09-06.md`, `deployment-rules-2026-09-08.md`, `sensitive-species-coordinate-policy.md`.
- **Atlas status/handoffs:** session closes and handoffs from 14 Aug to 10 Sep; `master-project-log-index-2026-08-25.md`; `project-position-2026-08-23.md`.
- **Audits/plans:** harvest-engine full audit + Claude Code full project audit (26 Aug), stream audit, world-class plan, coverage plan, master build plan, deep plan (8–9 Sep).
- **Research:** deep-research briefs + reports (comparable projects, tech-forest future, geoinformatics career).
- **Outreach:** `record-outreach-2026-08-18.md`, `record-outreach-2026-08-28.md`.
- **Senna:** `senna-regrowth/verification-repo-2026-08-14.md`, `primer-satellite-invasive-mapping.md`.
- **Jobs:** `tnfd-vacancies-snapshot-2026-09-28.md`.
- Published artifacts: technical blueprint, repository audit, 602-project library, connected-database blueprint (claude.ai artifact links in project docs).

## 8. Open questions and problems
- **Security:** a data.gov.in API key was written in plain text in a project doc (8 Sep) — rotate it. An Anthropic API key was pasted in chat on 10 Aug — confirm it was revoked.
- GitHub secret-scanning alert on `vishnuvarthan18/w2d-admin` (opened 19 Aug) — unread (separate project).
- Was the 14 Sep follow-up check ever done? OpenAlex chaining commit not confirmed in git.
- Gazetteer still tiny; real growth needs LGD via data.gov.in key.
- 3 "disputed" counts on site are really year-over-year changes — need date-aware fix or editorial note.
- `claim` table: docs disagree (0 rows in D1 vs 82 in atlas.db).
- No staging environment. dev/main branches diverged; needs its own planning session.
- 3 untracked test files — commit or not? 5 tests still fail (pre-existing).
- 5 failing tests in place-identity (Sathyamangalam/Satyamangalam duplicate in D1).
- WDPA API key missing; DLQ email alert needs a verified address; real data.gov.in key page size not measured.
- Domain suspension (2 Sep, unverified registrant email) — fixed?
- Small bugs: crocodile/king cobra wrong category, empty "Trees" category, no `family` field, tier-C citability.
- 48 photo contributors found — use them or not?
- Email Routing for DLQ alerts not set up. `.workers.dev` dashboard URL is public.
- Atlas vs Record naming never settled.
- NDVI 2026 anomaly / "drier forest" framing — editorial call pending.
- Senna script: Sentinel-2 fallback (2025+) and field validation not written; Senna vs native step needs hand-drawn points.
- Job crawler: unfinished — keep or drop? (Queues already deleted.)
- Career: was IGNOU January intake confirmed by phone? Was the portfolio shipped? "Product Manager" vs "ran a studio" CV framing unresolved. Mohamed bin Zayed grant window 15 Oct 2026.
- Did Vishnu apply for the Fire Control Centre Technical Assistant (GIS) post (closed 29 Sep)?

## 9. All chats in this project
- Index: [[Projects/forest/chats/INDEX]] · Claude Code sessions: see [[Projects/forest/DEV-LOG]] session index · 95 project docs in `docs/` (imported 2026-10-06)
- Spec review and implementation recommendations — 2026-08-10 (now filed under [[Projects/career/SUMMARY]])
- [[Projects/forest/chats/2026-06-14 Nilgiri biosphere|Nilgiri biosphere]] — 2026-06-14
- [[Projects/forest/chats/2026-10-02 Moving project to viOS (9)|Moving project to viOS (9)]] — 2026-10-02 (viOS meta chat)
- Most build work happened in [[Tools/Claude Code]] / Cowork sessions and is recorded as project docs, not chats.
