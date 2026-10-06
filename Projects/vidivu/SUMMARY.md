---
tags: project
status: paused
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: vidivu
State: [[Projects/vidivu/STATE]] · Log: [[Projects/vidivu/LOG]]

## 1. What this project is
- Vidivu is Vishnu's own tech studio / startup (Claude project name: "vidivu.in"). Domain: vidivu.in.
- It is separate from his full-time job at [[Companies/araCreate Group]]. Vishnu runs Vidivu part-time and solo.
- Public face: a general "end-to-end tech company". Tagline (kept): "Your entire tech side. One studio."
- First plan (Aug 2026): IT studio for startups and SMEs — websites, apps, UI/UX, internal tools, digital presence, support.
- Latest plan (Rev 3, 21 Aug 2026): core product is **process automation** — turning manual, document- and data-heavy work into automated flows. Open to any industry and any country.
- The three patterns sold: capture & extract, route & approve, gather & report.
- Website: [[Tools/Next.js]] + [[Tools/Tailwind CSS]] site, code on [[Tools/GitHub]], to be hosted on [[Tools/Vercel]].
- Brand: black, uppercase Inter, sharp corners, tricolor stripe (blue → deep blue → red). Documented in `DESIGN.md`.

## 2. Status now (as of 2026-10-06)
- Paused (not sure — Vishnu may be working on it elsewhere). Last coding session: 2026-08-10. Last strategy chat: 2026-08-21 (pivot blueprint + job book). No newer Vidivu work seen.
- Website: Next.js 16 + React 19 + Tailwind v4, repo `vishnuvarthan18/vidivu.in-`, local copy `~/vidivu.in`. 15 pages built: Home, Services hub + 6 service pages, Industries, Process, Technology, About, Contact, /startups, /smes; SEO (sitemap, robots, metadata, JSON-LD). (earlier note said inner pages were not built — the code has them.)
- Real `vidivu-logo.svg` in navbar and footer.
- Not deployed on [[Tools/Vercel]].
- Contact details (WhatsApp, email, founder info) are still TODO placeholders — a lead cannot reach Vidivu yet. Media is dummy; case studies "coming soon".
- Strategy: Rev 3 pivot blueprint is the current plan. It replaces the old "Buyer-Readiness Audit" entry offer. Site copy still reflects the older general-tech plan.
- No paying clients and no case studies recorded yet (not sure).
- Domain vidivu.in registered 2 Aug 2026, on [[Tools/Cloudflare]]. Subdomain `ops.vidivu.in` is the India Data Platform console of [[Projects/india-data-atlas/SUMMARY]] (live since 2026-09-21) — not Vidivu work.

## 3. Next steps
- Fill real contact details in app/contact and app/about.
- Deploy the website on [[Tools/Vercel]] (first locked tech step); get real Lighthouse numbers.
- Rewrite the site to be capability-led: show the three patterns with examples from several industries. English, for global readers.
- Replace dummy media; delete unused build-loop.mp4.
- Self-host [[Tools/n8n]] with [[Tools/Docker]]; automate Vidivu's own enquiry intake and proposal writing.
- Sell three "One Process, Fixed" jobs to warm network — in three different industries.
- Write up each job the same way: process before, numbers before, numbers after. Get written permission to name the client (put it in the contract).
- Open the international channel with dollar pricing (week 8–12).
- Week 12 review: which industry closed fastest, paid most, referred anyone.
- Pre-launch attention: proof-of-work pieces for named businesses, [[Tools/LinkedIn]] build-in-public posts 2–3 times a week, message 15–20 warm (non-araCreate) contacts, one local association contact per week ([[Companies/CODISSIA]], [[Companies/TANSTIA]], [[Companies/BNI]] Erode, [[Companies/StartupTN]]).
- Learn: Layer 1 delivery stack now (n8n, LLM APIs, [[Tools/Python]]); Layer 3 process interviewing from day one.

## 4. Decisions
- 2026-08-07 — Own the website code in [[Tools/Next.js]], not Framer or a paid template — Vishnu wants to own the code; free templates cannot match the studio look he wants #decision
- 2026-08-07 — Brand name "Vidivu", text-only wordmark, no icon — Vishnu's choice #decision
- 2026-08-07 — Design system: original "motorsport-engineering" look adapted from a BMW M spec (black canvas, UPPERCASE Inter, 0px radius, tricolor stripe as divider only), saved as `DESIGN.md` — Vishnu approved this over the lime-green "Nodable" draft #decision
- 2026-08-07 — Stack: Next.js 16 + [[Tools/Tailwind CSS]] v4, repo on [[Tools/GitHub]], free hosting on [[Tools/Vercel]] — simple, free, code-owned #decision
- 2026-08-08 — Logo: hand-set the type in [[Tools/Figma]], stripe under the wordmark; do not trust AI image tools for the final logo — they misspell text #decision
- 2026-08-10 — Work mode: part-time, solo; max 3–5 active clients = one build at a time + 3–5 retainers in parallel — hours are the real limit #decision
- 2026-08-10 — Retainers are the real business; builds are the way in #decision
- 2026-08-10 — Price zone: ₹40,000–1,50,000 per build, ₹5,000–20,000/month retainer; never publish prices — avoid the cheap freelancer tier #decision
- 2026-08-10 — Payments: 30–40% advance, milestones, balance at handover; never 100% upfront — trust and cash safety #decision
- 2026-08-10 — Never say "solo" or "part-time" in public material #decision
- 2026-08-10 — Entry offer: "Buyer-Readiness Audit", ₹15,000–25,000, 10 working days, 100% credited to a build — local buyers pay ₹0 for "strategy" alone (later replaced on 2026-08-21) #decision
- 2026-08-10 — Behind-the-scenes target: export textile SMEs in Tiruppur–Erode–Coimbatore; local clients = cash and portfolio, international/funded clients = premium later — from 4 rounds of research (later dropped on 2026-08-21) #decision
- 2026-08-10 — Website shows a general tech company, no niche wording — Vishnu rejected niche positioning on the site #decision
- 2026-08-10 — Six services: Web Dev, App Dev, UI/UX & Product Design, Internal Tools & Systems, Digital Presence Setup, Ongoing Support & Maintenance #decision
- 2026-08-10 — Five pages (Home, Services, Work, About, Contact); [[Companies/Netguru]] homepage layout as structure template; [[Tools/Google Stitch]] prompts must lock exact copy — vague prompts gave fake enterprise content #decision
- 2026-08-10 — [[Tools/WhatsApp]] Business as main contact channel, form as backup #decision
- 2026-08-10 — Content/SEO is not a lead source for the first 6+ months #decision
- 2026-08-21 — No AI blog at 10 posts/day before launch — hurts SEO and trust; launch amplifies an audience, it does not create one #decision
- 2026-08-21 — Open to any industry and company type; textile beachhead dropped as main target — Vishnu said it was misleading #decision
- 2026-08-21 — Do not use [[Companies/araCreate Group]] contacts or networks for Vidivu — conflict of interest with his job #decision
- 2026-08-21 — Pre-launch plan: proof-of-work for named businesses, [[Tools/LinkedIn]] build-in-public 2–3x/week, 15–20 warm contacts, one association per week #decision
- 2026-08-21 — Market gap: main gap = AI-readiness and implementation for non-metro SMEs; GEO/AEO = later upsell; AI agent security = not now (revisit in 2–3 years) — four-filter research #decision
- 2026-08-21 — Pivot Rev 3: industry, geography, client size open. Locked: capability (process automation), problem shape (messy info retyped by hand), proof standard (measured before/after) — no niche before real client data #decision
- 2026-08-21 — Services restructure: Internal Tools = flagship (process automation); Support = ops retainer; UI/UX kept as operator interfaces; Web Dev and Digital Presence demoted; App Dev only on request #decision
- 2026-08-21 — New offer ladder replaces Buyer-Readiness Audit: One Process, Fixed (₹35–60k / $700–1,500, 2 wks); Ops System (₹1–3L / $4–12k, 3–6 wks); Ops Retainer (₹15–35k/mo / $1.2–4k/mo); Process Diagnostic ($1.5–3k, international, only after case studies) #decision
- 2026-08-21 — Keep tagline "Your entire tech side. One studio." — product change, not a rebrand #decision
- 2026-08-21 — Keep local (₹) pricing off the public site #decision
- 2026-08-21 — Not learning: model training, computer vision, heavy MLOps, multi-agent, AI security — wrong decade for a solo operator #decision
- 2026-08-21 — Niche later only when 2 of 3 signals appear (best clients cluster, turning work down, charging more for one type); review at client 15 #decision
- 2026-08-21 — Kill test: 20 conversations, 4+ industries; fewer than 3 paid jobs in 8 weeks = fix offer, price or proof, not the industry #decision
- 2026-08-10 — Use the real SVG wordmark in navbar and footer. Hold Tier-2 city landing pages until there is real capacity; top nav capped at 6 items; no Tamil script on site. #decision
- 2026-08-21 — Pricing formula: build = 1/3 to 1/2 of year-one savings; retainer = 15–25% of build per year; never quote before getting their numbers #decision

## 5. Timeline
- 2026-08-02 — vidivu.in registered. Tried /design-sync to Claude Design from an empty folder.
- 2026-08-07 — Researched UI libraries ([[Tools/Watermelon UI]], [[Tools/shadcn-ui]], Aceternity, Magic UI and others). Chose to own code in Next.js.
- 2026-08-07 — Built "Nodable" HTML homepage draft (lime-green style), then VIDIVU homepage in the black motorsport style. Vishnu approved.
- 2026-08-07 — Scaffolded Next.js 16 + Tailwind v4 project (Navbar, Hero, SpecBand, ModelGrid, Magazine, Motorsport, CtaBand, Footer). Wrote `DESIGN.md`.
- 2026-08-07 — Pushed code to GitHub repo `vidivu.in-` after fixing folder-name and remote errors. Vercel deploy not done.
- 2026-08-08 — Logo specs pulled from DESIGN.md; logo image prompt written.
- 2026-08-10 — Business model set (part-time, 1 build + retainers). Four research rounds analysed. Buyer-Readiness Audit chosen as entry offer.
- 2026-08-10 — Website planned: 5 pages, 6 services, Netguru structure; Google Stitch prompts written for all pages.
- 2026-08-10 — `VIDIVU_PROJECT_STATE.md` written and memory updated.
- 2026-08-10 — Repo cloned to `~/vidivu.in`, run locally; real logo put in navbar and footer.
- 2026-08-21 — Rejected AI-blog plan. Opened target to any industry. Ruled out araCreate contacts.
- 2026-08-21 — Market gap research saved (`claude/market-gap-research.md`).
- 2026-08-21 — Pivot blueprint went Rev 1 → Rev 2 → Rev 3 (Vishnu pushed back twice). Saved as project doc + artifact.
- 2026-08-21 — Job Book (8 worked example jobs, pricing formula, scripts) saved as project doc + artifact.
- 2026-09-21 — `ops.vidivu.in` subdomain went live for [[Projects/india-data-atlas/SUMMARY]] (other project, same domain).
- 2026-10-02 — Project moved into viOS.

## 6. Key facts
- **Owner:** [[People/Vishnu]] (Mac user `vishnuvarthanv`). Employer: [[Companies/araCreate Group]] (full-time). Vidivu is part-time.
- **Haraq:** real company, not his employer (confirmed 2026-10-06) → [[Companies/Haraq]].
- **Repo:** github.com/vishnuvarthan18/vidivu.in- (on [[Tools/GitHub]]). Local copy `~/vidivu.in` (since 2026-08-10). Older local folder had a trailing space in its name; folders scattered across Desktop and Downloads (cleanup paused).
- **Domain:** vidivu.in on [[Tools/Cloudflare]]. Subdomain `ops.vidivu.in` used by [[Projects/india-data-atlas/SUMMARY]].
- **Hosting:** [[Tools/Vercel]] planned (free tier). Not deployed (as of last session 2026-08-10).
- **Dev history:** [[Projects/vidivu/DEV-LOG]] (Claude Code sessions, personal Mac). Chats index: [[Projects/vidivu/chats/INDEX]]. Project docs copied to `docs/`.
- **Not Vidivu (filed here by folder only):** "Halo / USD Halo" stablecoin landing page (2026-08-03, `~/Desktop/vidivu.in`, practice build, not sure); gift portfolio site for photographer [[People/Nevin Xavier]] (21 Aug 2026); "Waterminal AI for buildings" idea chat (separate idea).
- **Stack:** [[Tools/Next.js]] 16, [[Tools/Tailwind CSS]] v4, shadcn-compatible components ([[Tools/shadcn-ui]]).
- **Design tools:** [[Tools/Google Stitch]] (page mockups), [[Tools/Figma]] (logo).
- **Planned delivery stack:** [[Tools/n8n]] self-hosted via [[Tools/Docker]], LLM APIs, [[Tools/Python]], [[Tools/PostgreSQL]] + pgvector, [[Tools/WhatsApp]] for alerts.
- **Brand (in code, DESIGN.md / globals.css):** black canvas, white text, cards #1a1a1a, body #bbbbbb weight 300, Inter 800 uppercase headlines, 0px radius, stripe #3ba0e0 → #1c69d4 → #e22718 (accent only).
- **Brand (from logo chat):** canvas #0A0A0A, stripe #6CB4E4 → #0B3D91 → #E4002B — does not match (see open questions).
- **Reference sites:** [[Companies/Netguru]] (structure), [[Companies/Halo Lab]], [[Companies/Lemberg Solutions]]; earlier: cerebrium.ai, twofoldny.com, soma.ca, noartmusic.com, alethia.earth.
- **Channels:** warm network (non-araCreate), [[Companies/CODISSIA]], [[Companies/TANSTIA]], Erode DSIA, [[Companies/BNI]] Erode, [[Companies/StartupTN]] Erode hub, adjacent-vendor partners (ISO/compliance consultants), [[Tools/LinkedIn]].
- **AraCreate network:** ruled out on 21 Aug (conflict of interest), but listed as warmest lead in the Rev 3 blueprint (see open questions). Related: [[Companies/araCreate Group]].
- **Market numbers:** IDP market 28.9% CAGR ($1.5B 2022 → $17.8B 2032); "AI integration" demand +178% YoY (Upwork); SME AI adoption 5–10%; Tamil Nadu ~5M MSMEs.
- **Where work was done:** [[Tools/Claude]] chats in the "vidivu.in" project (claude.ai). Code built in Claude's sandbox and pushed from Vishnu's Mac terminal.
- **Artifacts:** Pivot Blueprint https://claude.ai/code/artifact/512fba09-b09d-4b9d-849c-db44f86ca041 · Job Book https://claude.ai/code/artifact/0cd8a760-ed92-4524-8c53-04230dca11df

## 7. Files and documents
- `claude/market-gap-research.md` — Claude project doc (2026-08-21). Four-filter market research; main gap = AI-readiness for non-metro TN SMEs.
- `claude/vidivu-pivot-blueprint.md` — Claude project doc (2026-08-21). Rev 3 pivot: what is open/locked, 3 patterns, offer ladder, learning path, 90-day plan, kill test, risks. Artifact: https://claude.ai/code/artifact/512fba09-b09d-4b9d-849c-db44f86ca041
- `claude/vidivu-job-book.md` — Claude project doc (2026-08-21). 8 example jobs in 8 industries (garment export, CA firm, diagnostic lab, school, freight forwarder, auto parts, wholesale distributor, overseas bookkeeping), 4 interview questions, pricing formula, case study template, first-conversation script. Artifact: https://claude.ai/code/artifact/0cd8a760-ed92-4524-8c53-04230dca11df
- `DESIGN.md` — brand/design system, in the GitHub repo.
- `VIDIVU_PROJECT_STATE.md` — full state file (business model, offer, trust stack, site plan) made 2026-08-10 in Claude's sandbox; Vishnu was told to commit it to the repo (not sure if done).
- `PROJECT.md` / `DECISIONS.md` — planned for the repo (not sure if made).
- `vidivu-nextjs.zip` — Next.js project download (2026-08-07).
- Google Stitch prompts for Home, Services, Work, About, Contact — kept only inside the 2026-08-10 chat.
- Logo image-generator prompt — in the 2026-08-08 chat.

## 8. Open questions and problems
- Is Vidivu still active? Last code 10 Aug, last chat 21 Aug 2026 (status paused, not sure).
- Is vidivu.in pointed at any site besides ops.vidivu.in?
- Contact details still placeholders.
- AraCreate network: ruled out in the 21 Aug chat but named as warmest lead in Rev 3 blueprint. Which is right? Is "AraCreate Academy" the same as [[Companies/araCreate Group]]?
- Brand colours: code uses #3ba0e0/#1c69d4/#e22718; logo chat used #6CB4E4/#0B3D91/#E4002B. Which does the logo SVG use?
- The chat where Rev 1 → Rev 3 pivot and the job book were made was not found in chat search.
- Website copy still shows the older service plan; needs rewrite for Rev 3 (capability-led, 3 patterns).
- Have any 90-day steps started (n8n setup, first One Process jobs, outreach)?
- Was `VIDIVU_PROJECT_STATE.md` committed to the repo?
- Mac project folder cleanup was paused.
- GST registration / PAN / SAC code before first invoice — not decided.
- Risk: "open" can slide into an unfocused dev shop; horizontal automation is crowded; remote trust needs named case studies.

## 9. All chats in this project
- Waterminal AI for buildings feasibility — 2026-08-07 (separate idea, not Vidivu)
- Logo design specifications — 2026-08-08
- Project progress update — 2026-08-10
- Launching startup with block pages for SEO visibility — 2026-08-21
- (Chat that made the market research, pivot blueprint and job book — 2026-08-21, not found in search, not sure)
- Moving project to viOS (5) — 2026-10-02
- Code sessions: see [[Projects/vidivu/DEV-LOG]].
- Note: dates are the last-updated dates of each chat.
