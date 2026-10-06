---
tags: project
updated: 2026-10-06
---
# DEV LOG: Semmozhi (Claude Code sessions, personal Mac)

Source: 138 Claude Code sessions, 28 Aug to 5 Oct 2026. Repo: github vishnuvarthan18/tamil-data-collector. Local folder: ~/Downloads/tamil_harvest.
93 of the 138 sessions are automated "Realism Engine" task runs that stopped at once on a Claude usage limit (26 Sep). About 30 more are automated task runs that did finish. Fewer than 15 are real working chats with Vishnu.

## What was built
- **Data engine (crawler).** About 40 Scrapy spiders that collect Tamil content from open sources (Wikipedia, Wikisource, Project Madurai, Internet Archive, OpenLibrary, OpenAlex, museums, OpenStreetMap). Ran on GitHub Actions in two scheduled jobs. Schedule switched off on 5 Sep.
- **Data cleaning and audit.** `clean_data.py` turned 513,541 raw records into 28,691 cleaned ones (96% were repeats). A classifier sorted them by topic. Later audit: only about 5,300 records are real Tamil text (about 15.9 million characters); most of the rest are catalogue entries. The 10 newer "topic engines" have never run.
- **Local data viewer** (`viewer_app/`, Flask) to browse the cleaned data.
- **Design system.** Tokens, components, style guide, light and dark mode, own fonts (Semmozhi Brahmi, Vatteluttu, Grantha, Tamil). Rule set in `DESIGN_SYSTEM.md`.
- **Reference engine.** A Wikimedia Commons photo harvester (`fetch_commons_v3.py`) that collected about 2,400 licence-checked reference photos of real objects (stone, palm leaf, pottery, coins, copper plates, rings, seals), plus a local reference viewer.
- **Realism Engine.** An unattended task queue (T0 to T16) run by a supervisor script. It built a test rig, physics scenes (palm-leaf bundle, copper plates on a ring, coins, seals, sherds), auto-tuned materials against real photos, lighting tests, wear and age, a sound layer, identity illustrations (121 SVGs), and test pages. Final report: 340 of 382 checks pass. Coin, copper plate and stone moved to 3D; palm leaf, pottery, ring and seal stay 2D.
- **Website.** Merged into one `website/` folder with the new UI. Pages: Home (nine themes, timeline, photo tiles, miniatures, contact panel), Scripts, Brahmi Lab, Grantha, Vatteluttu, Tamil. English and Tamil switch, loader, running banner, light and dark icons. Built with Web Awesome Core components in the original cream and terracotta colours.
- **Hosting and release setup.** Cloudflare Pages. Production www.semmozhi.online (from `main`, deploys only on a manual "Run workflow"). Staging dev.semmozhi.online (from `dev`, deploys on every push, hidden from search). Private admin site engine.semmozhi.online behind a Cloudflare Access login. Contact form saves messages to a Cloudflare database. Automatic page tests at 11 screen sizes in both languages.
- **Server.** The collector is installed on Vishnu's OVH server in its own folder (`/srv/semmozhi/`) but is idle. The server also runs another project's services, which were not touched.

## Timeline (newest first)
- **5 Oct.** Many Home page edits (text-only hero, real photos not cropped, credits removed, timeline animation, miniatures shelf, AI-style wording and em dashes removed, same text size in English and Tamil, clean phone menu). Site went live on www, then production was switched to an "Under construction" page. Full site lives only on dev.semmozhi.online. Contact form saves to Cloudflare. Brahmi font page removed; font download moved into the Brahmi Lab page.
- **4 Oct.** Deleted the realism experiment pages and the `archive/` folder. Chose Web Awesome Core as the design system. Tried new colour palettes, then went back to the original colours. Rebuilt all pages with Web Awesome; released to staging, then production (5 Oct).
- **3 Oct.** Set up staging, production and admin domains, release workflow and Access login. Wrote team docs (README, ARCHITECTURE, DEPLOYMENT, KNOWN_ISSUES, DATA_AUDIT, TOOLS_TRIAGE, SERVER_PLAN). Looked at the OVH server over SSH; installed the collector there, idle.
- **2 Oct.** Folder cleanup went wrong at first (Claude deleted about 1.5 GB of test renders when Vishnu meant "organize"). Then: `website_live/` renamed to `website/`, new UI applied to the real pages, Lab made part of the site, branches reduced to `main` and `dev` (old branches kept as `archive/<name>` tags), local page hub (`local/serve.sh`). semmozhi.online moved to Cloudflare DNS (from BigRock). Auto-deploy via GitHub Actions.
- **29 Sep.** Ran the style guide locally.
- **28 Sep.** Project review: three parts (engine, website, design system). Removed old font zips, scratch files and backups.
- **27 Sep.** Prompt 6 (consistency and minimal motion) on `realism-polish`. Prompt 7 (clean design system) on `realism-clean`.
- **26 Sep.** Prompt 4: Realism Engine autonomous run. Supervisor queue T0 to T16. Many runs died on usage limits in the morning; run restarted on a new account and finished overnight into 27 Sep.
- **25 Sep.** Prompt 1 design system (style guide). Prompt 2 GSAP real materials and emblems. Prompt 3 Commons harvester, overnight run.
- **23 Sep.** Classified the cleaned data by topic; found it thin on real prose.
- **6 Sep.** Built the local data viewer.
- **5 Sep.** Turned off scheduled crawls (kept manual trigger). Fixed slow git (large repo, not a stuck login). Wrote `clean_data.py`.
- **28 Aug.** Throttle settings for the Project Madurai spider. Added 28 spiders (40 total) and split the crawl into two scheduled jobs.

## Decisions
- #decision 2026-09-05: Stop scheduled crawls by commenting out the schedule in `crawl.yml`, keep manual trigger.
- #decision 2026-09-05: Clean data into a separate `data_clean/` folder; never change the raw data.
- #decision 2026-09-26: Realism Engine runs unattended; it never asks, it logs decisions. Usage-limit stops count as waits, not crashes.
- #decision 2026-09-26: A 3D material is used only if it passes the automatic gate (coin, copper plate, stone passed; palm leaf stays 2D).
- #decision 2026-09-27: Design rule: no bright top-edge highlight strip on cards anywhere.
- #decision 2026-09-28: Project has three parts: data engine, website, design system. Keep them clearly separate.
- #decision 2026-10-02: "Clean" means organize, not delete.
- #decision 2026-10-02: Two branches only: `main` is production, `dev` is where work happens.
- #decision 2026-10-02: The Lab (script tools and fonts) is part of the public website.
- #decision 2026-10-03: Staging dev.semmozhi.online from `dev`; production www.semmozhi.online from `main`; nothing skips staging; production needs Vishnu's approval; some staging pages never go to production.
- #decision 2026-10-03: engine.semmozhi.online is the private admin site, only Vishnu can log in.
- #decision 2026-10-03: Two places only: the website on Cloudflare, the collector on the server.
- #decision 2026-10-03: Do not run the engine or schedule crawls on the server for now; keep it idle.
- #decision 2026-10-03: Reference-photo viewer stays local only.
- #decision 2026-10-03: Focus on Semmozhi first; Atlas project later.
- #decision 2026-10-04: Use Web Awesome Core (open source, MIT) as the design system; keep the original cream and terracotta colours.
- #decision 2026-10-05: Public site shows only Home, Scripts and the script pages; Contact lives inside Home, not a separate page. Remove unfinished identity illustrations.
- #decision 2026-10-05: Contact messages saved in Cloudflare for now (no WhatsApp needed).
- #decision 2026-10-05: Production shows an "Under construction" page; the full site is only on dev.semmozhi.online. Switch back by changing `website/production-mode.txt` from `construction` to `site`.

## State at last session (5 Oct 2026)
- Git: `dev` and `main` match, all pushed, nothing uncommitted.
- www.semmozhi.online shows "Under construction". Full site on dev.semmozhi.online. Admin site at engine.semmozhi.online behind login.
- Contact database set up and empty.
- Collector on the OVH server installed but idle. Crawls off since 5 Sep.
- Hidden from the live site but kept in the project: Chola, Literature, About, All fonts pages.
- Vishnu said "we are going to do something big" next; not yet described.

## Open items
- Turn on BigRock auto-renew (domain expires 26 Aug 2027).
- Two photos (temple, coin) legally need a visible credit, or swap them.
- A Tamil speaker should check all Tamil text.
- Source-check two timeline facts (Keeladi date, "first Indian language printed in its own script").
- Optional: merge the script pages into one Scripts page with tabs.
- Data: remove repeats, add licence and source URL to each record, test the topic engines once before any server run.
- Repo history is large (about 455 MB) from committed raw data.
- Old semmozhi.pages.dev addresses still respond.
- Branch protection needs GitHub Pro (not enforced now).
- Header, footer and Lab form controls are still custom, not Web Awesome.
- Astro (site) and React (admin) moves discussed, not started.
- Set up the Atlas project the same way, later.
- WhatsApp copies of contact messages need a Meta token (optional).

## Session index
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-10-03_b8caa415]] — 3 to 5 Oct — staging, production and admin domains; server look; team docs; Web Awesome rebuild; Home page work; "Under construction" on production
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-10-02_93e0545c]] — 2 to 3 Oct — folder cleanup, website merge to new UI, main/dev branches, semmozhi.online on Cloudflare, auto-deploy
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-29_e7f7b8d7]] — 29 Sep — run style guide locally
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-28_e5cb127c]] — 28 Sep — project review (engine, website, design system) and file cleanup
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-27_96348158]] — 27 Sep — Prompt 7 clean design system; rule: no top-edge highlight on cards
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-27_b49c44ef]] — 27 Sep — Prompt 6 consistency and minimal motion on realism-polish
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_4ce6ca39]] — 26 to 27 Sep — Prompt 4 Realism Engine: rig, self-tests, supervisor, long run, restart on new account
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_d88496dd]] — 27 Sep — T16 final report (340 of 382 checks pass)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_37e30607]] — 27 Sep — T15 browser, accessibility, keyboard, reduced motion checks pass
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_b817dea9]] — 27 Sep — T14 licence cleanup report (271 findings)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_6a0e6b91]] — 27 Sep — T13 reference viewer (2,412 photos)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_133d89b2]] — 27 Sep — T12b identity illustrations batch 3 (121 total)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_954da73c]] — 27 Sep — T12a identity illustrations batch 2 (96 total)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_eb94ec1c]] — 27 Sep — T11c Thirukkural leaf reader page (10 kurals)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_78eecf64]] — 27 Sep — T11b Chola stone wall page
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_70b3cada]] — 27 Sep — T11a realism home page
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_b3ec9e2d]] — 26 Sep — T9b gate results and tuned materials into the site
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_813b25a1]] — 26 Sep — T4 seals material (7 of 8)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_8ee1e8f2]] — 26 Sep — T4 rings material (5 of 8)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_4eca7ca2]] — 26 Sep — T4 coins material (8 of 8)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_ee034518]] — 26 Sep — T4 pottery material (7 of 8)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_e08aa66a]] — 26 Sep — T4 copper plate material (7 of 7)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_6a6c444c]] — 26 Sep — T4 stone material (6 of 6)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_ac08dc47]] — 26 Sep — T4 palm leaf material and reusable tuner
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_d68b6764]] — 26 Sep — T3b palm leaf 3D gate failed, keep 2D
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_7da3cd79]] — 26 Sep — T3a palm leaf 3D build
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_494474aa]] — 26 Sep — T10 page components, dark mode, contrast tables
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_ffc3deb8]] — 26 Sep — T9 style guide rebuild with 3D materials
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_bcccea28]] — 26 Sep — T8 optional sound layer
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_440ba7b2]] — 26 Sep — T7 wear and age system
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_bba84e9c]] — 26 Sep — T6b copper plate lighting
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_fbaedb7b]] — 26 Sep — T6a stone lighting
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_388e5afe]] — 26 Sep — T5d physics: sherd and stone
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_a8dfb91a]] — 26 Sep — T5c physics: coin, ring, seal
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_976174fc]] — 26 Sep — T5b physics: copper plates on ring
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_a26efd3e]] — 26 Sep — T5a physics: leaf bundle and thread
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_f0a62329]] — 26 Sep — T4 thread and board skipped (no photos)
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_d7b1a954]] — 26 Sep — T3a first attempt, cut short
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-25_18759e2c]] — 25 to 26 Sep — Prompt 2 GSAP real materials and emblems; harvester files committed
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-25_1be78540]] — 25 Sep — Prompt 3 Commons harvester built, overnight run started
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-25_8f855d03]] — 25 Sep — Prompt 1 design system and style guide
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-23_a8e19cdf]] — 23 Sep — topic classification of cleaned data
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-06_288b3082]] — 6 Sep — local data viewer app
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-05_f73349ae]] — 5 Sep — crawl schedule off, slow git, `clean_data.py`
- [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-08-28_2a9fde37]] — 28 Aug — Project Madurai throttle; 28 new spiders, crawl split in two jobs
- 26 Sep, automated runs stopped by usage limit (no work done, 93 sessions): [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_0015ad1c]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_0018775b]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_01302d1e]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_034f49cf]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_0544c64f]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_07c440e1]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_08769737]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_09896fbb]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_09ace1b4]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_0e8cea1c]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_10660f58]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_10e0d1ac]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_12c5f4f8]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_134951e6]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_15c18525]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_179a2bb4]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_1946a8cc]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_1e506a87]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_1e8f837c]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_2142097e]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_2366b48e]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_2373cd52]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_28398aea]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_29651167]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_2a7f43d1]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_2e2ef616]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_309e6c3f]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_335a2dd2]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_354ca567]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_35da178f]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_3a2fdf8f]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_3c1d5a68]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_3d42223d]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_427395f1]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_489736fb]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_4b6b2ca9]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_4f04762c]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_4fedc973]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_51bacbb1]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_5206833a]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_53b65bad]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_573c4a84]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_597917c1]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_5c6867d8]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_63b4ec75]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_668cfc65]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_679cc832]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_6ab2a7f6]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_6b9375c9]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_72e81928]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_7b4954bd]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_7ca53482]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_7dab0ee8]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_7db0dd14]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_86d83f9a]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_89b04686]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_8c8deece]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_99a8e25b]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_99fb7371]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_9c01e01b]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_9f069d23]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_9fb9e3fc]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_a009adca]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_a2fd0008]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_a4c73e4c]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_a598a21c]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_a5a52304]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_a6268be9]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_a75d7e31]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_b1a81145]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_bd600062]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_be9e02ee]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_c729a23f]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_c94f4273]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_ca14fb9f]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_caa3041b]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_cfaf503b]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_d0e134c7]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_da2e8832]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_db6caabe]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_dc615546]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_df5455d4]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_e1d9261c]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_e38c4a98]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_e42b02c3]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_ead1f82b]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_ee631bfa]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_f1d465b7]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_f6d0506c]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_f7be1294]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_fd3f5cf8]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_fe355786]] · [[Projects/semmozhi/claude-code/personal-mac__Downloads-tamil-harvest__2026-09-26_fe5ece91]]
