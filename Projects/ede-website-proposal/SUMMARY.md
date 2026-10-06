---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: EDE Website Proposal

## 1. What this project is
- **Goal:** Understand the EDE Figma design ("EDE Snapshot 25 Sep"), estimate it, build a project plan and prepare the proposal to build EDE's new website on [[Tools/Liferay]] (Liferay DXP).
- **Who it is for / client:** End client is [[Companies/Emirates Drug Establishment]] (EDE), the UAE federal body that regulates medicines, medical devices and health products. araCreate works through [[Companies/Aarini]] (Aarini Consulting B.V., Netherlands), who is the contract client.
- **Why it exists:** EDE needs a new bilingual (English + Arabic, RTL) website, desktop and mobile, built on Liferay. araCreate must send a proposal and plan with a fixed start 12 Oct 2026 and go-live 30 Nov 2026.

## 2. Status now (as of 2026-10-06)
- **Two plans exist — which one is current is not decided (to confirm):**
  - **PM plan (ours):** `EDE_Project_Plan_PM.xlsx` — start 12 Oct, go-live 30 Nov 2026, hypercare to 28 Dec, ~1,413 h total, team PM + 4 full-stack devs + 2 testers.
  - **Developer Delivery Plan v2 (Kishor):** Liferay DXP 2026.Q1 LTS, self-hosted in the UAE; 75 tasks, 71 page templates (EN/AR, desktop + mobile), 500 dev hours (484 to go-live + 16 hypercare); go-live end of W8, hypercare W9–W12 (~4 h/week). v2 vs v1: 62 → 75 tasks, 68 → 71 templates; testing inside section 5 (25 h manual dev testing EN/AR); still 500 h.
  - The 12 conflicts between them (hours, go-live date, testers, hosting, AI items, …) are not yet decided.
- Done: Figma analysis of all 14 ✅ pages + IA site map. 63 desktop pages, about 14 Liferay page templates.
- Done: Analysis doc (Claude Doc) and 7 frame-by-frame audit reports saved in the claude.ai project.
- Done: Compared 3 plans - Claude plan (A), Kishor's Developer Delivery Plan (B, 500 h, go-live end W8), and the Proposal by [[People/Shyam]] (C, master).
- Done (2026-10-01): Expert-PM Excel plan `EDE_Project_Plan_PM.xlsx` (10 tabs) - start Mon 12 Oct 2026, go-live Mon 30 Nov 2026, hypercare to 28 Dec 2026, about 1,413 h, team 1 PM + 4 full-stack devs + 2 testers. Rewritten in simple Indian English.
- Done: Reviewed proposal draft (v0.1): PM-side risk list (18 points) and "points where client can reject" list (18 points) for the team.
- Not done: Price is still blank ([AMOUNT] EUR). Client approver name blank.
- Not done: Plan is not in the Google Sheet yet - Google Sheets connector was never on, so only Excel files exist (the Sheet has only an older one-tab CSV version).
- Not done: Kishor's v2 plan conflicts (12 points) not yet decided; Excel not updated with his 13 extra tasks.
- Not done: Proposal fixes (payment split, missing basic pages, team, tech approach, UAT days) not yet applied.
- Figma itself: no screen approved (all "Under review"), no prototype links, no after-click states, copied placeholder text from other ministries (MOHRE, TDRA, MoF, Singapore, Ireland), Arabic has many errors.

## 3. Next steps
1. Fill the price and day rates; decide the payment split (proposal has 40/30/30; suggestion 20/30/30/20).
2. Fix proposal points where the client can reject (fixed price vs many "Confirmation needed", unrealistic client dates, team/references, tech approach, AI document workspace priced separately, missing basic pages like 404/SEO/privacy).
3. Decide Kishor v2 conflicts (hours, go-live date, testers, hosting, AI items, shortcuts login, booking, hypercare) and update the Excel.
4. Move UAT to 20.11-26.11 (5 days), get UAT server by 09.11, set proposal signing date (suggested 07.10).
5. Check UAE holidays near go-live (Commemoration Day ~30 Nov/1 Dec, National Day 2-3 Dec).
6. Turn on Google Sheets connector and move the final plan into the same Google Sheet.
7. Run the discovery workshop with EDE in week 1 (from 12 Oct) to close the open points.

## 4. Decisions
- Note: Kishor's v2 says self-hosted in the UAE; our decision below says hosting is provided by EDE (we only get access) — part of the 12 open conflicts.
- 2026-10-01 — Build on Liferay; anything not clear in Figma is marked "Undefined" — Vishnu's instruction.
- 2026-10-01 — Working days Mon-Fri only, no holidays counted — Vishnu said only Sat/Sun off. #decision
- 2026-10-01 — No Phase 2; all scope stays in one delivery — Vishnu rejected Claude's Phase 2 split. #decision
- 2026-10-01 — Discovery workshop with client at the start; all unknowns go to client discussion — Vishnu's instruction. #decision
- 2026-10-01 — Proposal (C) is the master scope document — client document. #decision
- 2026-10-01 — AI assistant in scope; hosting provided by EDE (we only get access); Start Application / Track / Book appointment link to EDE systems; shortcuts and service save saved in user account (UAE PASS); hypercare 4 weeks; UI/UX designer outside our hours; build the approved design versions. #decision
- 2026-10-01 — AI document-checking workspace (agentic dossier) kept open for client discussion. #decision
- 2026-10-01 — Hours can grow; dates fixed: start 12 Oct 2026, go-live 30 Nov 2026 — Vishnu's instruction. #decision
- 2026-10-01 — Team: PM (10-year expert level), 4 full-stack developers, 2 testers. #decision

## 5. Timeline
- 2026-10-01 — Figma homepage analysed; analysis doc created.
- 2026-10-01 — All 14 ✅ pages audited: 63 pages, first estimate 242-312 person-days.
- 2026-10-01 — Sign-in/apply flow and rating options worked out (UAE PASS; Customer Pulse vs Liferay-built rating).
- 2026-10-01 — Kishor's Delivery Plan (8 weeks) compared; first Google Sheet plan (go-live 15 Mar 2027).
- 2026-10-01 — Plan with RACI (2,640 h), then 500 h version, then workshop added.
- 2026-10-01 — Compared Developer Delivery Plan .md and Proposal ACG-EDE-002; merged plan (490 h, go-live 4 Dec).
- 2026-10-01 — Fixed window 12 Oct - 30 Nov; expert-PM Excel plan (~1,413 h) made and humanised.
- 2026-10-01 — Proposal gaps filled (dates, dependencies); Kishor v2 plan compared (12 conflicts).
- 2026-10-01 — Proposal reviewed from PM side and for client-rejection points; team summary written.

## 6. Key facts
- **People:** [[People/Vishnu]] — owner, PM lead on proposal; [[People/Shyam]] — author of proposal (Shyam Sathish Kumar); [[People/Kishor Arjunan]] — "Kishor", wrote the Developer Delivery Plans v1 and v2 (not sure same person); [[People/Navaneethan Kandaraj]] — "Navaneethan K", MD, supplier approver in later proposal (not sure same person); [[People/Aravinth Panch]] — "Aravinth Panch", MD, supplier approver in first proposal draft (not sure same person)
- **Companies:** [[Companies/Emirates Drug Establishment]] (end client), [[Companies/Aarini]] (contract client), [[Companies/araCreate India]] (supplier in later proposal, Erode), araCreate GmbH Berlin (supplier in first draft), [[Companies/araCreate Group]]
- **Tools:** [[Tools/Liferay]], [[Tools/Figma]], [[Tools/Google Sheets]], [[Tools/Google Drive]], [[Tools/Claude]]; integrations: UAE PASS, Tatmeen (link out), Bayanat.ae, Customer Pulse, Intercom/live chat, Google Analytics, Hotjar, reCAPTCHA, Azure OpenAI (in Kishor's plan)
- **Links / repos / servers / file paths:**
  - Figma: EDE-Snapshot-25-Sep (file key HX7b7a5itxQWBWPdPFbDwl)
  - Claude Doc: "EDE Website – Figma Analysis & Liferay Build Plan" (claude.ai artifact 33c94339-0a6a-462d-a9bd-1e3c40b910ea)
  - Google Sheet: "EDE Website – Project Plan" (id 18-Aw3rsOfosaBHSAhQJ67jdgxoXuRxBdHsXvmXOxMio) - older one-tab version
  - Proposal IDs: ACG-EDE-002 (first draft), ACI-ARN-001 (later draft), v0.1, 01.10.2026
- **Stack (Kishor v2):** Liferay client extensions + fragments only, React 18 custom elements, PostgreSQL 16, Elasticsearch 8 (Arabic analyzer), Kubernetes, GitLab CI, Spring Boot microservices, Azure OpenAI (UAE North) for the AI assistant.
- **Key numbers:** 63 pages; ~14 templates (Kishor: 68-71 templates); milestones: workshop decisions 14 Oct, BRD/HLD + hosting sign-off 26 Oct, UAT test cases 3 Nov, feature cut-off/code freeze 16 Nov, SIT done 19 Nov, VAPT cleared 24 Nov, UAT sign-off 27 Nov, go-live 30 Nov, hypercare ends 28 Dec.
- **Related:** [[Projects/aracreate/SUMMARY]]

## 7. Files and documents
- `claude/ede-figma-analysis.md` — Figma analysis summary (claude.ai project)
- `claude/ede-project-plan.md` — latest plan notes (claude.ai project)
- `claude/figma-audit/01-07` — frame-by-frame audit reports per section (claude.ai project)
- `EDE_Project_Plan.xlsx`, `EDE_Project_Plan_RACI.xlsx`, `EDE_Project_Plan_Phase1.xlsx`, `EDE_Project_Plan_500h.xlsx`, `EDE_Project_Plan_Merged.xlsx`, `EDE_Project_Plan_30Nov.xlsx` — earlier plan versions (chat outputs)
- `EDE_Project_Plan_PM.xlsx` — latest plan, 10 tabs (Summary, Milestones, Plan, RACI, Team load, Test plan, Dependencies, Open points, Risks, Governance)
- `EDE_Developer_Delivery_Plan.md`, `EDE_Developer_Delivery_Plan_2.md` — Kishor's plans (uploaded)
- EDE Website Development Proposal (v0.1) — by Shyam (pasted in chat)

## 8. Open questions and problems
- Price and day rates not set; plan is ~1,413 h but Kishor's is 500 h - which one is the price based on?
- Payment split 40/30/30 likely to be rejected; last 30% tied to go-live can get stuck.
- Figma not approved; placeholder content and Arabic errors; missing screens (job details/apply, 404, search results, mobile menu, signed-in header, side effect form).
- Client unknowns: hosting/Liferay version and licence, UAE PASS onboarding, APIs (drug registry, dashboards, Tatmeen), rating tool, live chat vendor, AI assistant model, AI document workspace scope.
- Timeline very tight: only 10 working days after cut-off; UAT only 3 days in proposal.
- Go-live 30 Nov may clash with UAE Commemoration Day; National Day 2-3 Dec in hypercare.
- Plan not yet in Google Sheet (connector off).
- Which plan is current: PM plan (~1,413 h, 30 Nov) or Kishor v2 (500 dev h, end of W8)? (to confirm)
- Draft-list items still to confirm with EDE: UAE PASS login, AI assistant (30 h), agentic dossier workspace (35 h), analytics, user training.
- EDE must give API docs + test environments by end of W2 (Drugs Registry, Open Data, GIS, UAE PASS, happiness meter, newsletter).
- Sign-off turnaround, browser support and performance target are open.

## 9. All chats in this project
- Index: [[Projects/ede-website-proposal/chats/INDEX]]
- [[Projects/ede-website-proposal/chats/2026-10-01 EDE Snapshot project scope|EDE Snapshot project scope]] — 2026-10-01
