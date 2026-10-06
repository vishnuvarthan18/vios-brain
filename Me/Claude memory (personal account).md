---
tags: me
source: Claude personal account memory export
---
# Claude memory (personal account)

A cleaned copy of what Claude remembers about Vishnu. Private items (health, money, family, IDs, contact details) are left out.

## Work context
- Tech professional at the meeting point of technology, design and strategy.
- Goal: simplify systems, improve efficiency, build scalable, UX-led solutions.
- Works with araCreate India (araCreate Group): client work, content/media shoots, and helps with araCreate Academy (not founder).
- ISMS Coordinator, UX Designer and Security Lead at AraCreate India for the ISO 27001:2022 project.
- Holds ISO 27001:2022 Internal Auditor certificate (QHSE Solutions, March 2026). Interested in Lead Auditor next.
- Does freelance design and build work: Webflow sites, packaging design, web apps.
- Product manager and UI/UX designer on araMetrics (works with a separate dev team).
- Non-coder who builds with AI tools: Claude writes prompts, Claude Code / Cursor write code.

## Personal context
- Based in Chithode, Erode, Tamil Nadu (Kongu region).
- Cares about environment, wildlife, forests and tribal communities.
- Special interest in the Western Ghats, Nilgiris and Sathyamangalam.
- Daily cyclist, tracks rides on Strava; has explored a multi-activity fitness community in Erode.
- Follows a 16:8 eating window with fasted morning rides.
- New hobby: embedded systems and electronics (NodeMCU ESP8266).
- Learning harmonica.
- Keeps a personal second brain called viOS.
- Treats Wedding2day as a personal project, not office work.

## Preferences (how to answer)
- Short, dense answers. Bullets and tables over prose.
- Practical and usable over theory.
- Factor in environmental and community values when recommending.
- Answer first, no preamble, no filler, no closing summary.
- One clear recommendation with the reason, not a long menu of options.
- Plain, simple English; avoid jargon.
- One step at a time for technical setup.

## Per-project memories

### How to use Claude (no project page)
- Wants to get the most out of Claude Pro with low usage.
- Built a four-layer custom instruction setup: Core System, Reasoning Mode, Coding Tasks, Project Context template.
- Learned: thinking mode, model choice and long chats drive usage; limits work on a rolling window.
- Pushes back directly when an answer misses.

### wedding2day.app → [[Projects/wedding2day-app/SUMMARY]]
- B2B trade platform for the Tamil Nadu wedding decoration industry: buy/sell used, new, rental items and post needs.
- Stack: React Native (Expo) + Firebase + NativeWind, built via Cursor. Old stack fully dropped.
- DECISIONS.md is the source of truth; AI agents must read it first.
- Five post types in one `listings` collection; user types Vendor / Manufacturer.
- Auth moving to WhatsApp OTP via a Cloudflare Worker.
- Main success metric: interests per listing.
- Open risks: WhatsApp-first habits of vendors; Meta WhatsApp verification not started.
- Rules: exact Cursor prompts pasted as-is; one new Cursor chat per phase; reject diffs touching unrelated files.
- Landing page live on Cloudflare Pages.

### ac-iso-27001-isms → [[Projects/ac-iso-27001-isms/SUMMARY]]
- ISO/IEC 27001:2022 certification for AraCreate India; Vishnu is the contact for the external auditor.
- Stage 1 audit (May 2026) closed; two NCs closed, CAR accepted in June.
- Drive ISMS folder rebuilt into a clean 0–7 folder system.
- Open: SoA reference column shifted, unclear Annex A exclusions, Stage 2 result unconfirmed.
- Learned: organise by document type with a master register; keep restore-test evidence; one risk register.
- Likes checklists mapped to real folders and short, formal auditor emails.

### ac-sinolink → [[Projects/sinolink/SUMMARY]]
- Website for SinoLink Deutschland (Dresden, China sourcing consultancy), built in Webflow for araCreate.
- Done: coming-soon page, legal pages, mobile nav, services grid, Web3Forms, launch post.
- Learned: custom code only runs on published site; JS in footer; scope CSS with prefixed classes.
- Likes minimal output and natural, people-first copy.

### ac-niborra → [[Projects/niborra/SUMMARY]]
- InkWave (formerly Niborra): handwriting automation, pen-plotter machine + software.
- Positioning: the software is the product.
- Research done: competitor UX (Signascript, UUNA TEK), proof pack, gap analysis, five personas, JTBD, flows, Miro process board.
- Likes depth and evidence, but plain language.

### v (personal planning, no project page)
- Fitness: daily fasted zone-2 cycling, two-phase plan (add strength, then running later).
- Built a habit board and a daily routine with rotating evening projects.
- Learning harmonica via Tomlin Leckie's beginner playlist.
- Compiled a 22-project portfolio; a fuller career story is still pending.
- Likes clarifying questions asked up front, then one directive plan.

### arm → [[Projects/arm-ui/SUMMARY]]
- araMetrics: modular super-app for deep-tech companies and SMEs ("mind to market").
- v1 scope: Core shell + Admin panel + Calendar Merger.
- Design: Radix Themes, amber accent, Sand grays, desktop-first.
- Produced: UX flow doc, design spec, PRD, developer spec, QA test plan.
- Rule: never mix existing behaviour and new requirements in specs.

### arm-ui → [[Projects/arm-ui/SUMMARY]]
- AI-native build of the araMetrics UI with Claude Code, Next.js and Radix Themes.
- Three roles: Admin, Operator (near read-only), Platform User; one shell.
- Permission fencing is structural (not in the DOM), not just hidden.
- 8-stage pipeline: Research → Define → Flows → Design System → Generate → Refine → Prototype → Handoff.
- Paused (last active July 2026); open: login scope, signup vs admin-only provisioning.

### web-bala → [[Projects/web-bala/SUMMARY]]
- Subscription status checker web app for an Adobe plan resale business (public lookup + admin panel).
- Next.js + Supabase on Hostinger.
- Learned: never commit `.env.production`; set every env var on the host.

### araCreate academy → [[Projects/aracreate-academy/SUMMARY]]
- In-school STEM and robotics programme in Coimbatore, Erode and Salem, grades 5–9.
- Student journey: Learner → Innovator → Changemaker.
- Proof: 700+ students, 4 schools, 7+ projects selected at state level (SIDP).
- Built: master reference doc, 10-panel carousel, 7-post launch plan, Instagram + LinkedIn strategy.
- Copy rules: plain, warm South Indian English; no AI buzzwords; never imply teachers are failing; use real student photos.

### HR and leads - bala (no project page)
- WFH HR + lead tracking PWA for a BPO client (geofenced attendance, leave, leads, product catalog).
- Stack: React + Vite PWA, Supabase.
- Must-have: duplicate phone check on leads. Build attendance first.

### vidivu.in → [[Projects/vidivu/SUMMARY]]
- Vidivu: part-time IT studio, "Your entire tech side. One studio."
- Latest direction: process-automation services, any industry; paused since Aug 2026.
- Homepage built in Next.js + Tailwind, not yet deployed.
- Learned: local SMEs pay for delivery, not "strategy"; milestone payments build trust; AI blog spam hurts.
- Keeps a strict conflict-of-interest boundary with his day job.

### whatsapp api (no project page)
- Two move-to-viOS attempts failed: project had no content and viOS was offline.

### forest → [[Projects/forest/SUMMARY]]
- Forest-tech career plan: field conservation work using tech skills.
- Sathyamangalam Atlas: public species and places database.
- Senna invasive-species regrowth check using Google Earth Engine.
- Plans IGNOU MSc Geoinformatics (Jan 2027 intake); decided against a permanent government job.
- GovTech job alert pipeline on Cloudflare (free tier) for research-institute jobs.

### career → [[Projects/career/SUMMARY]]
- Only a move-to-viOS request was captured; no project details in memory.

### trip-meghalaya → [[Projects/trip-meghalaya/SUMMARY]]
- 3-day Meghalaya self-drive trip, Sep 12–14, 2026, with two friends.
- Goal: as many waterfalls and places as possible.
- Done Day 1: Wei Sawdong and Nohkalikai falls. Nongriat skipped (closed Sundays).
- Likes tables and map views over written directions; short answers.

### semmozhi → [[Projects/semmozhi/SUMMARY]]
- Non-profit site showcasing Tamil language, literature and history.
- Custom data-collection engine from open sources; four original fonts for historic Tamil scripts.
- Avoids unsupported claims; uses only open, licensed sources.
- Sister project: [[Projects/india-data-atlas/SUMMARY]].

### India Data Atlas → [[Projects/india-data-atlas/SUMMARY]]
- Citation-backed data platform on Indian protected areas, forests, species, tribal communities and laws.
- Started as Ecotourism Atlas (Scrapy + GitHub Actions + Cloudflare); now six layers on a VPS with an ops console.
- Every fact needs source, date, licence and confidence.
- Prefers plain, readable code over black-box AI pipelines.
- Related: Sathyamangalam Atlas, Tamil Data Collector.

### Tech to me → [[Projects/tech-to-me/SUMMARY]]
- No memory summary in the export.

### arm-timer → [[Projects/timer/SUMMARY]]
- Deep walkthrough of Toggl Track (30-day trial) as time-tracker research for araMetrics.
- Status paused; true goal of the project still an open question.

### space app → [[Projects/nasa-space-apps-erode-2026/SUMMARY]]
- Local Lead for NASA Space Apps Challenge 2026, Erode (Nov 14–15, 2026, VCET Thindal).
- Handles registrations, emails and the participant WhatsApp group.
- Fixed a waitlist bug caused by a low virtual-capacity cap.
- Virtual teams judged on submission only; key updates go by email.
- Learned: confirm a fix is live before telling people it is fixed.

### clockify enty → [[Projects/clockify/SUMMARY]]
- Rebuilt Clockify time entries for Sep 11–30, 2026 from Slack and Calendar evidence (173 entries).
- Projects tracked: #AC, #ARM, HALLE, FUTURE STATE, SINOLINK, #DSA.
- Rules: no invented entries; non-meeting times should look natural, not rounded.
