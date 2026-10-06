---
tags: project
status: paused
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: Niborra — handwritten letters at scale

## 1. What this project is
- **Goal:** B2B SaaS + pen plotter for handwritten letters at scale. Software is the product; the machine is the output device.
- **Client:** [[Companies/Niborra]] (araCreate Group client, Clockify tag #noa). Client contact: [[People/Ahmad Taleb]]. araCreate PM: [[People/Shyam]]. Developer: [[People/Kishor Arjunan]].
- **Two surfaces:** Niborra Studio (Electron desktop app for operators/managers) + Web Portal (sign-up, billing, teams, config).

## 2. Status now (as of 2026-10-06)
- Status in viOS: paused since 2026-08-18 (waiting for client flow choice). Not sure if the build plan below is running — ask Vishnu.
- Client PRD ACG-NOA-001 v1.0 approved.
- Build plan (20 Jul 2026): 11 epics, 15 user stories, 100 tasks, 306 story points, 10 two-week sprints 13 Jul – 27 Nov 2026; tracked as GitHub issues.
- Hard dates: hardware MVP mid-Sep 2026, V1.0 software launch Dec 2026, first pilot batch of 10 machines Feb 2027.
- Main risk: [[People/Kishor Arjunan]] is the only developer; plan needs 23–28 pts/sprint vs his 12–14 pace. Options: extend, cut to Phase 2 (Pro analytics first), or add a developer.
- Done: competitor analysis (Signascript, UUNA TEK) with 28 findings; gap analysis (12 gaps); UX research docs; 41 [[Tools/Figma]] Make prompts for v3; client Word doc; Miro architecture v1; pricing tiers locked.
- 3 flow versions in Figma: Web-First, App-First, Role-Based v3 (recommended). Client choice pending.
- Project had 0 saved docs in Claude.

## 3. Next steps
1. Get client choice of flow (v3 recommended).
2. Build v3 wireframes.
3. Decide telemetry scope, integrations scope, per-tier feature gates, data storage.
4. Hi-fi design + design system; dev roadmap.
5. Decide how to close the developer capacity gap (extend, cut scope, or add a developer).
6. Decide EU + US data residency.

## 4. Decisions
- (before 2026-08-18) — Recommend Role-Based v3 flow. #decision
- (before 2026-08-18) — Pricing tiers locked. #decision
- 2026-07-20 (plan) — E5 Automations (Shopify trigger) deferred to post-launch, because REST API/webhooks are post-launch. #decision
- 2026-07-20 (plan) — Hosting on Hetzner (EU/Frankfurt). #decision

## 5. Timeline
- 2026-07-20 — Build plan written (10 sprints, 13 Jul – 27 Nov); client PRD v1.0 approved.
- 2026-06-04 — Chat: computer analysis and SWOT.
- 2026-06-03 — Chat: new project requirements and scope overview.
- Aug 2026 — Low-fidelity design for the flow (Clockify entries 6–7 Aug); planning and flow evaluation (18 Aug).
- 2026-08-18 — Context check chat.

## 6. Key facts
- **Personas:** Camille, Sofia, Marcus, Priya, + Operator archetype.
- **Figma file key:** `oul2s4ac1NaWZXvnrmteUr` (pages: Flows, Wireframes Portal, Wireframes Studio, UI Kit, Hi-fi).
- **Tools:** [[Tools/Figma]], Miro, [[Tools/GitHub]] issues.
- **UX research:** earlier name "InkWave"; hypothesis only — no real user interviews yet.
- **Chats index:** [[Projects/niborra/chats/INDEX]]
- **Working prefs noted:** concise bullets, direct recommendations, no em dashes, docx + Miro deliverables, one focus at a time.

## 7. Files and documents
- Research docs, 41 prompts, proof pack, gap analysis — not stored in the Claude project (location unknown).

## 8. Open questions and problems
- Where are the research docs and prompts saved?
- Which flow did the client pick?
- Is the build running to the July plan (hardware MVP mid-Sep), or is the project paused?
- EU + US data residency still open.
- No real user interviews yet.

## 9. All chats in this project
- [[Projects/niborra/chats/2026-06-03 New project requirements and scope overview|New project requirements and scope overview]] — 2026-06-03
- [[Projects/niborra/chats/2026-06-04 Computer analysis and SWOT assessment|Computer analysis and SWOT assessment]] — 2026-06-04
- [[Projects/niborra/chats/2026-08-18 Context check|Context check]] — 2026-08-18
- (Empty "save chat" note moved to Archive/empty-save-chats/.)
