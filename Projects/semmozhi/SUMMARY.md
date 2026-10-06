---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: Semmozhi (செம்மொழி) — Tamil heritage website + data engine

## 1. What this project is
- A non-profit website that shows the greatness of Tamil (language, literature, history, culture) to a global audience, including people with no link to Tamil.
- Two parts: (1) a data engine that collects Tamil data from the open web (started as the "Tamil Data Collector"), and (2) the Semmozhi website built on that data.
- Rule on tone: be accurate. Do NOT claim "Tamil is the first / oldest language". Use the strong, sourced case instead: 2,000+ years of unbroken literature, independent of Sanskrit, 60,000+ inscriptions, Roman-era trade, UNESCO "Great Living Chola Temples". Plus a "Myths vs Evidence" page.
- Design theme: the materials Tamil was written on — palm leaf, stone, copper plate, pottery, coins, rings, seals — one per site section.
- Also builds 4 original fonts for old Tamil scripts: Tamil-Brahmi, Grantha, Vatteluttu, Tamil.
- Way of working: Vishnu is non-technical. [[Tools/Claude]] writes prompts and checks results; code is written by an AI coding agent ([[Tools/Claude Code]] sessions on the personal Mac — see DEV-LOG).
- Three parts kept separate (2026-09-28): data engine, website, design system.
- Sister project: [[Projects/india-data-atlas/SUMMARY]].

## 2. Status now (as of 2026-10-06)
- **Active.** Last work: 2026-10-05. Vishnu said "we are going to do something big" next; not yet described.
- **Website online (Cloudflare Pages):**
  - Production www.semmozhi.online shows an **"Under construction"** page (since 5 Oct; full site was live there briefly).
  - The **full site is on staging dev.semmozhi.online** (from `dev`, auto-deploys on push, hidden from search).
  - Private admin site engine.semmozhi.online behind a Cloudflare Access login (only Vishnu).
  - (earlier, to 2026-10-02: site ran only on the Mac, hosting paused)
- **Release flow:** push to `dev` → staging. Production (`main`) deploys only on a manual "Run workflow". To show the real site on production: change `website/production-mode.txt` from `construction` to `site`, merge to `main`, run the workflow.
- **Git:** only `main` and `dev` (old branches kept as `archive/<name>` tags). `dev` and `main` match; nothing uncommitted.
- **Website content:** folder `website/`. Public pages: Home (with Contact panel), Scripts, Brahmi Lab, Grantha, Vatteluttu, Tamil. Chola, Literature, About and All fonts pages hidden but kept. Contact form saves to a Cloudflare database (empty now).
- **Design system:** Web Awesome Core (MIT) with the original cream and terracotta colours (since 4 Oct). Header, footer and Lab form controls still custom. Realism experiment pages deleted. (earlier: custom "showcase" redesign with Prompt 6 / realism engine)
- **Fonts:** all four at v3.0 ("tapered ends" style). Vatteluttu is a draft; must not be published until [[People/Elmar Kniprath]] reviews it — not sent yet (as of 2026-09-25). Permission received 24 Sep to base it on e-Vatteluttu OT (SIL OFL) with changed letterforms.
- **Data engine:** crawls off since 5 Sep. Collector installed on the OVH server in `/srv/semmozhi/` but idle by choice. 10 topic engines never run. Audit: 513,541 raw records, ~96% repeats; ~5,300 records are real Tamil text (~15.9 million characters); most text lacks a licence field.
- **Team docs in repo:** README, docs/ARCHITECTURE, DEPLOYMENT, KNOWN_ISSUES, DATA_AUDIT, TOOLS_TRIAGE, SERVER_PLAN, PROJECT_MAP.
- **Domains:** semmozhi.online and semmozhi.info registered 26 Aug 2026 (BigRock); semmozhi.online DNS on Cloudflare since 2–3 Oct. BigRock auto-renew is off (expires 26 Aug 2027).

## 3. Next steps
1. Hear Vishnu's "something big" plan.
2. Decide when to switch production from "Under construction" to the full site.
3. Turn on BigRock auto-renew.
4. Send the Vatteluttu font draft to Elmar Kniprath for review before publishing.
5. Credits: 2 photos (temple, coin) need visible credit or swap; licence cleanup and credits page.
6. Tamil-speaker check of all Tamil text; source-check 2 timeline facts (Keeladi date, "first Indian language printed in its own script").
7. Data: remove repeats, add licence and source URL per record, test topic engines once before any server run.
8. Later: Atlas project set up the same way; Astro (site) / React (admin) moves discussed, not started.

## 4. Decisions
- 2026-08 (not sure of day) — Goal is accurate, complete data, NOT to "prove" Tamil is first/best — credibility. #decision
- 2026-08 (not sure) — Claude writes prompts and checks numbers; a separate AI agent writes all code — Vishnu is non-technical. #decision
- 2026-08 (not sure) — Crawler = [[Tools/Scrapy]] on [[Tools/GitHub]] Actions free tier, raw `.jsonl` in repo, no database — free and simple. #decision
- 2026-08 (not sure) — Drop DSAL spiders (blocked by robots.txt) and dbpedia_tamil (stale data) — respect robots, no junk. #decision
- 2026-08 (not sure) — Split crawl into 2 jobs by domain (Wikimedia vs others), one shared concurrency group, rebase with `-X theirs` — Wikimedia spiders rate-limit each other; newest data must win. #decision
- 2026-08-28 (about) — Defer fixing the stale public dashboard ("fix after some days") — display issue only, data is fine. #decision
- 2026-09-05 — Turn off scheduled crawls (manual run still works) — repo growing too big. #decision
- 2026-09 (not sure) — Merge the two repos into one private repo, no public dashboard for now. #decision
- 2026-09-06 — Do not lead with "Tamil is the oldest language"; build the sourced "airtight" case — the claim is fact-checked false. #decision
- 2026-09-23 — Build order: Chola section first, classical literature reader second; script timeline waits for real sources — that is where the rich data is. #decision
- 2026-09-24 — Raw data goes to [[Tools/Google Drive]] with [[Tools/rclone]], not the server — Drive has 50–500 GB, server only 38 GB. #decision
- 2026-09-24 — Web pages from the mission engine shown as excerpt + link only — copyright. #decision
- 2026-09-24 — Photos: free licence only, with author + licence caption. #decision
- 2026-09-24 — Visual style v2 "Temple and palm-leaf" — Vishnu rejected v1 as "worst". #decision
- 2026-09-24 (about) — Pause server and domain; "build the website first" (later replaced: hosting set up 2026-10-02/03). #decision
- 2026-09-25 — One shared font style for all 4 scripts: "C Tapered ends"; Brahmi rebuilt from Noto Sans Brahmi, old hand-drawn pipeline retired — consistency and quality. #decision
- 2026-09-25 — Use [[People/Elmar Kniprath]]'s e-Vatteluttu OT as reference (SIL OFL 1.1): do not copy letterforms exactly, credit him, send for review before publishing — his conditions. #decision
- 2026-09-25 — Theme = all writing surfaces, one per section: stone+pottery = origins, palm leaf = literature, copper+seals = kings, temple stone = Chola, coins+rings = trade. #decision
- 2026-09-25 — Thirukkural reader: one Kural per leaf strip. #decision
- 2026-09-25 — Change `website_live` directly, but git commit first, branch `redesign-design-system`, and a backup copy. #decision
- 2026-09-25 — Only public domain / CC0 / CC BY images may ship; CC BY-SA, CC BY-NC (e.g. CICT) = reference only; draw original SVGs where possible; rings and seals drawn from scratch — licence safety. #decision
- 2026-09-25 — Keep Indus seals off the site — Indus–Tamil link is contested. #decision
- 2026-09-25 — Do not use Vishnu's Mac Chrome; never work around blocked sites. #decision
- 2026-09-26 — Realism plan: prototype the palm leaf in WebGL ([[Tools/Three.js]]) with physics, then a decision gate; keep a light 2D fallback for slow phones. #decision
- 2026-09-27 — Showcase phase: STOP adding animation; keep only functional motion under 400 ms; one material registry for all parts; polish only the hero set (home, Chola, Kural reader, palm leaf, stone, copper, coins); small reviewed batches, no overnight runs. Reverses the 2026-09-26 "more physics/animation" direction. #decision
- 2026-09-28 — Project has three parts: data engine, website, design system; keep them separate. #decision
- 2026-10-02 — "Clean" means organize, not delete. Two branches only: `main` = production, `dev` = work. The Lab is part of the public website. #decision
- 2026-10-03 — Staging from `dev`, production from `main`; nothing skips staging; production needs Vishnu's approval. engine.semmozhi.online = private admin. Website on Cloudflare, collector on the server, kept idle. Focus on Semmozhi first, Atlas later. #decision
- 2026-10-04 — Web Awesome Core (MIT) as the design system; keep the original cream and terracotta colours. #decision
- 2026-10-05 — Public site shows only Home, Scripts and the script pages; Contact inside Home; contact messages saved in Cloudflare (no WhatsApp). Production shows "Under construction"; full site only on dev. #decision

## 5. Timeline
- 2026-08 (not sure of day) — Project started as the "Tamil Data Collector" (Scrapy crawler, private GitHub repo).
- 2026-08 (before 08-28) — Many spider bugs fixed (Scrapy 2.18 API, Wikidata IDs, thevaaram depth, Met Museum 403s, Wikisource empty texts, throttle settings).
- 2026-08-28 (about) — Run #11: all 40 spiders ran together for the first time, both jobs green (1 h 2 m). Commit `fa7be1a`. Public dashboard still showed old data.
- 2026-09-05 — Day 7–8 check: healthy. 5/5 scheduled runs OK, 96 data files, `.git` 5.3 MB. Scheduled crawls turned off this day. `clean_data.py` cut 513,541 records to 28,691.
- 2026-09-06 — Two research reports done: "Is Tamil a Language or a Civilization?" and "The Tamil Heritage Gap" (~70 Tamil sites checked; nothing like this project exists).
- 2026-09-23 — Data cleaned and classified: 28,691 records, 776 files; only 6,692 records are real readable text. Website plan saved. `.git` now 367 MB.
- 2026-09-24 — Permission received to base an open-source Vatteluttu font on e-Vatteluttu OT. Website v1 (Home, Cholas, Tirukkural reader, Brahmi Lab, Font, About, Explore), 9 topic engines and a whole-web "mission" engine built (offline tests only). Google Drive decision. Site restyled "Temple and palm-leaf".
- 2026-09-25 — All 4 fonts reach v3.0 (scores 99.7–99.99% vs reference). Scripts and Fonts pages done. Palm-leaf reference catalog and 7-surface media plan written. 256 reference images collected on the Mac. Redesign handoff saved; mapping and Kural reader decided; Prompt 1 (design system) written.
- 2026-09-26 — Realism plan written (WebGL, physics, material study). Realism Engine unattended run T0–T16 (finished 27 Sep, 340 of 382 checks pass; 93 runs died on usage limits). Coin, copper plate, stone in 3D; palm leaf, pottery, ring, seal stay 2D.
- 2026-09-27 — Showcase-quality plan. Owner found 3 bugs (floating copper ring, flat coin text, mismatched fanned leaves). Prompt 6 planned.
- 2026-09-28 — Project review and cleanup: three parts named.
- 2026-10-02 — Project moved into viOS. Sites merged into `website/` with new UI; branches cut to `main` and `dev`; semmozhi.online DNS moved to Cloudflare; auto-deploy via GitHub Actions.
- 2026-10-03 — Staging, production and admin domains set up; team docs and data audit written; collector installed idle on OVH server.
- 2026-10-04 — Web Awesome Core chosen; all pages rebuilt; realism experiment pages and `archive/` deleted.
- 2026-10-05 — Home page polished; contact form to Cloudflare; Brahmi font download moved into Brahmi Lab. Live briefly on www, then production set to "Under construction".

## 6. Key facts
- **People:** [[People/Vishnu]] (owner, GitHub `vishnuvarthan18`). [[People/Elmar Kniprath]] (made e-Vatteluttu OT font, gave permission; review pending; do NOT store his email). [[People/George Hart]] (UC Berkeley Tamil scholar, cited in research; never claims Tamil is "first").
- **Companies / sources:** [[Companies/Wikimedia]] (Wikipedia, Wikisource, Commons — main data and photo source), [[Companies/Wikidata]], [[Companies/Internet Archive]], [[Companies/Project Madurai]] (richest free Tamil e-texts), [[Companies/OpenAlex]], [[Companies/Cleveland Museum of Art]], [[Companies/Art Institute of Chicago]], [[Companies/The Met]] (open-access APIs, CC0), [[Companies/CICT]] (palm-leaf library, CC BY-NC = reference only), [[Companies/UNESCO]] (Chola temples citation), [[Companies/Google]] (Drive, Noto fonts).
- **Tools:** [[Tools/Cloudflare]] (Pages, Access, database, DNS), Web Awesome Core, [[Tools/Scrapy]], [[Tools/GitHub]] (Actions + Pages), [[Tools/Python]], [[Tools/Docker]], [[Tools/Caddy]], [[Tools/rclone]], [[Tools/Google Drive]], [[Tools/SQLite]] (local viewer, frontier.db), [[Tools/Three.js]], [[Tools/GSAP]], [[Tools/Claude]].
- **Live addresses:** www.semmozhi.online (production, "Under construction"), dev.semmozhi.online (staging, full site), engine.semmozhi.online (admin, login). Old semmozhi.pages.dev addresses still respond.
- **Dev history:** [[Projects/semmozhi/DEV-LOG]] (138 Claude Code sessions, personal Mac). Chats index: [[Projects/semmozhi/chats/INDEX]]. Project docs copied to `docs/`.
- **GitHub repos:** `vishnuvarthan18/tamil-data-collector` (private, code + data; history ~455 MB); `vishnuvarthan18/tamil-data-dashboard` (public, GitHub Pages: https://vishnuvarthan18.github.io/tamil-data-dashboard/ — to be merged away).
- **Mac folder:** `~/Downloads/tamil_harvest` — the ONE project folder. Inside: `website/` (the site; was `website_live/` until 2026-10-02), `engines/`, `tamil_harvest/` (crawler), `design/`, `viewer_app/`, `PROJECT_MAP.md`.
- **Server:** Vishnu's OVH server `ubuntu@40.160.137.239` (Ubuntu 26.04, 38 GB disk) — same VPS as [[Projects/india-data-atlas/SUMMARY]] (its `core-infra` services — do not touch). Collector in `/srv/semmozhi/`, idle.
- **Data sizes:** 40 spiders (19 Wikimedia + 21 other). 28,691 records; 44 named literary works, 555,394 words; 12 major works ready (Tirukkural, Cilappatikaram, Puranānūru, etc.).
- **Where work was done:** the "semmozhi" Claude project (plans) and [[Tools/Claude Code]] on the personal Mac (code).
- **Competitors:** English Wikipedia (main), itihaas.ai (AI-made, no citations), tamiltimeline.com ("coming soon" for years).

## 7. Files and documents
**Claude project docs (in the "semmozhi" Claude project):**
- `claude/tamil-data-collector-status.md` — crawler status, 40 spiders, crawl.yml design (2026-09-05).
- `claude/tamil-website-plan.md` — mission, positioning, competitors, data inventory, build order (2026-09-23).
- `claude/website-build-status.md` — website v1, engines, Drive plan, server steps (2026-09-24).
- `claude/website-live-2page-status.md` — local site pages and temple/palm-leaf style (2026-09-24).
- `claude/scripts-progress-status.md` — 4 fonts v3.0, scores, Vatteluttu conditions (2026-09-25).
- `claude/ollai-chuvadi-reference-catalog.md` — palm-leaf photo/vector sources and licences (2026-09-25).
- `claude/ancient-tamil-media-collection-plan.md` — 7 writing surfaces, collection phases (2026-09-25).
- `claude/reference-collection-status.md` — what downloads work, museum API yield (2026-09-25).
- `claude/redesign-handoff-2026-09-25.md` — full handoff: rules, design decision, image library, tools (2026-09-25).
- `claude/redesign-decisions-2026-09-25.md` — mapping, Kural reader, folder choice, Prompt 1 (2026-09-25).
- `claude/realism-plan-2026-09-26.md` — WebGL/physics realism plan (2026-09-26).
- `claude/showcase-quality-plan-2026-09-27.md` — showcase phase, 3 bugs, Prompt 6 (2026-09-27).

**Key files on the Mac (`~/Downloads/tamil_harvest`):**
- `PROJECT_MAP.md` — folder map. `DESIGN_SYSTEM.md` — design system (location not sure). `CONTENT_TO_VERIFY.md` — facts to check.
- `prompt-1-design-system.md` — Prompt 1 for the dev agent.
- `design/references/LICENSES.csv` — 256 reference images with licences; `design/fetch_commons_v2.py`, `design/curate_refs.py`, `design/specs/palm_leaf_visual_notes.md`.
- `tools/font_v3/` (sf.py, best.py) — font pipeline. Zips `SemmozhiX-3.0.zip`.
- `engines/archive.py`, `vps/DRIVE-SETUP.md`, `vps/docker-compose.yml`, `vps/crontab.txt`, `engines/mission/config.py`.
- `.github/workflows/crawl.yml`, `scripts/build_dashboard_data.py`, `viewer_app/app.py`.
- Research reports (in old chats, Sept 6): "Is Tamil a Language or a Civilization?", "The Tamil Heritage Gap".
- Architecture diagram: `semmozhi_architecture.html`.

## 8. Open questions and problems
- What is the "something big" Vishnu mentioned on 5 Oct?
- BigRock auto-renew is off (domain expires 26 Aug 2027).
- Repo history is large (~455 MB) from committed raw data: clean history, or new code-only repo? Not decided.
- Old semmozhi.pages.dev addresses still respond.
- Branch protection needs GitHub Pro (not enforced now).
- Header, footer and Lab form controls still custom, not Web Awesome.
- WhatsApp copies of contact messages would need a Meta token (optional).
- Two crawler bugs: `project_madurai` indexes nav links (2,686 junk records); `english_wikisource_tamil` pulls unrelated texts.
- Most data lacks licence fields; 10 topic engines never tested.
- Vatteluttu review not sent to Kniprath; no expert has reviewed any font.
- Brahmi weak glyphs (candrabindu dot, 𑀴𑀼/𑀴𑀽) not fixed; Grantha conjuncts not built.
- Adinatha font licence not checked; About-page footnotes (4 claims) missing; Tevaram sample thin.
- Script-evolution timeline has no sourced content yet. Chera/Pandya/Pallava content too thin.
- Commercial or non-commercial site? Decides if CC BY-NC images can ever ship.
- Old public dashboard (tamil-data-dashboard) showed stale data; to be merged away.

## 9. All chats in this project
- [[Projects/semmozhi/chats/2026-10-02 Moving project to viOS (8)|Moving project to viOS (8)]] — 2026-10-02
- Code sessions: see [[Projects/semmozhi/DEV-LOG]] (session index).
- Work sessions recorded in the project docs (titles unknown):
- Crawler build + run #11 — about 2026-08-28 (title unknown)
- Day 7–8 crawler check-in — 2026-09-05 (title unknown)
- Research: Tamil language vs civilization + heritage gap — 2026-09-06 (title unknown)
- Data classification + website plan — 2026-09-23 (title unknown)
- Website build sessions 1–3 + restyle — 2026-09-24 (title unknown)
- Fonts v3.0 + reference collection + redesign handoff — 2026-09-25 (title unknown)
- Redesign decisions, Prompt 1 — 2026-09-25 (title unknown)
- Realism plan — 2026-09-26 (title unknown)
- Showcase-quality plan — 2026-09-27 (title unknown)
