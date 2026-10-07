---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: Timer (araMetrics Time model)
State: [[Projects/timer/STATE]] · Log: [[Projects/timer/LOG]]

## 1. What this project is
- **Goal:** Build time tracking inside araMetrics (the ARM platform, the "super app" by [[Companies/araCreate Group]]). People log hours on projects, managers approve, hours go to billing and payroll.
- **Who it is for:** Internal araCreate product, for individuals and teams.
- **Why it exists:** Most work time is meetings. araMetrics already has calendar data (Calendar Merger), so the week can fill itself from the calendar. Today a Python script (arm-util-clockify) copies Google Calendar events into [[Tools/Clockify]].
- **Big change (2026-09-28):** Timer is NOT a standalone app. It is one implementer of the "Time" model inside the Operations collection of the araMetrics shared data model (Bundle → Collection → Model → Field → Implementer).
- **arm-timer (merged in):** "arm-timer" was a separate Claude project holding only the research for Timer: a full live walkthrough of [[Tools/Toggl Track]] (Toggl 2.0) on 23 Sep 2026 — 143 screenshots, 2 flowcharts, 43 edge cases. It is the Toggl live test listed below.

## 2. Status now (as of 2026-09-28)
- Done: 12 ARM repos cloned to ~/araCreate/ARM and sorted into deploy/ core/ apps/ services/ library/ docs/.
- Done: desk research on 13 competitors; only Timely does calendar-to-timesheet, and it is buggy.
- Done: real live tests (Admin + Member) for [[Tools/Clockify]], [[Tools/Toggl Track]] (the arm-timer walkthrough), Harvest, Hubstaff. Jibble skipped (desk research only).
- Done: Miro board ("My First Board") with research and product findings as one connected story, plus "Start here" map and "Next steps".
- Done: product findings: 10 insights, positioning, 4 personas, 10 JTBD scored, 2 journey maps, proposed approval flow, metrics, MoSCoW scope (25 features), 8 open decisions, 5 risks.
- Done: meeting notes page and a simple explainer for Vishnu.
- Done: new architecture reading saved (`arm-architecture-v2.md`). Old standalone plan put aside.
- Not done: 8 flowchart PNGs and 24 key screenshots not on Miro yet (upload blocked).
- Not done: answers to the 7 architecture questions and 8 product decisions; Timer plan rebuilt on the new architecture; MVP user stories.
- No work since 28 Sep (as of 2026-10-06).

## 3. Next steps
1. Vishnu answers the 7 architecture questions (Clients owner, Leaves/Budget, Risks service, phase meaning, who says Yes/No, Operations DB, diagram label mistakes).
2. Rebuild the Timer plan on the Time-model base (`arm-architecture-v2.md`).
3. Vishnu answers the 8 product decisions (start with monitoring yes/no).
4. Update the spec, then write MVP user stories.
5. Upload the 8 flowcharts + 24 screenshots from `comparison analysis/_for_miro` to the Miro slots.
6. Optional: test Toggl Manager, Time tracker, Guest roles before the trial ends (about 23 Oct 2026); then clean up the trial org and test accounts.
7. Pilot with one team for 2 to 4 weeks (proposed).

## 4. Decisions
- 2026-09-08 — Claude must share the plan first and ask before saving or sending anything. — Vishnu was unhappy Claude did work without asking. #decision
- 2026-09-08 — Timer works for individuals and teams, has calendar auto-entries from day one, and has billing/invoicing. #decision
- 2026-09-08 — Assume trust-based (no screenshots/monitoring) for now. (Still an open decision.) #decision
- 2026-09-08 — Old docker/ folder (1.4GB stale copies) deleted when reorganising repos. #decision
- 2026-09-10 — Do real click-by-click live tests of competitors, shown visually on Miro. #decision
- 2026-09-23 — Toggl test: study the new "Toggl 2.0" (focus.toggl.com), not classic Toggl Track — this is what a new customer gets today. #decision
- 2026-09-23 — Toggl test: 2 real accounts, Owner/Admin (Vishnu) and Member ([[People/Thalaivan Ugam]]), on the 30-day Premium trial (no card). #decision
- 2026-09-23 — Toggl test: no typing passwords, no destructive actions, no OAuth, no export downloads; skip Manager/Time tracker/Guest roles and the paid Time Off add-on. #decision
- 2026-09-23 — Skip the live Jibble test. — It is an attendance (clock-in) app, not a project-billing tracker. #decision
- 2026-09-23 — Research and product findings shown as one connected story on Miro. #decision
- 2026-09-28 — Old plan is wrong. Timer = one implementer of the Time model in Operations; Projects, Tasks, Clients are shared models; invoicing lives in Finance; users/access come from Admin; Calendar publishes events and Time subscribes. #decision

## 5. Timeline
- 2026-10-06 — arm-timer project merged into Timer.
- 2026-10-02 — Project moved into viOS.
- 2026-09-28 — "Where we stopped": live tests done for 4 apps; findings on Miro; architecture change confirmed; `arm-architecture-v2.md` saved. Data-flow picture is a mockup.
- 2026-09-23 — Vishnu shared 4 super-app diagrams; Claude wrote back its understanding ("miro planning" chat).
- 2026-09-23 — Meeting notes page made; simple explainer saved (`timer-explainer-simple.md`).
- 2026-09-23 — Miro: research + product findings joined as one story. Image upload blocked; files put in `_for_miro`.
- 2026-09-23 — Toggl 2.0 walkthrough (arm-timer): sign-up, timer, manual entry, projects, client, rates, tags, invites (role bug seen), full approval loop, reports, admin, integrations; 43 edge cases. Saved as `toggl-walkthrough.md`.
- 2026-09-23 — Hubstaff (54 screens), Toggl (143), Harvest (168) live-test docs saved; 4 tuned live-test prompts saved.
- 2026-09-10 — Clockify real live-test doc saved (98 screens, 2 flows).
- 2026-09-08 — 13-competitor deep dive; "Timer Competitor Dossier" page published.
- 2026-09-08 — Repos cloned and reorganised; first spec and analysis docs written.

## 6. Key facts
- **People:** [[People/Vishnu]] — product manager / owner (new to PM work, wants simple explanations). [[People/Thalaivan Ugam]] — Member account in the Toggl test.
- **Companies:** [[Companies/araCreate Group]] — builds araMetrics. Competitors studied: Clockify, [[Tools/Toggl Track]] ([[Tools/Toggl Track]]), Harvest, Hubstaff, TimeCamp, Everhour, RescueTime, Timely, Time Doctor, ClickUp, Paymo, Jibble, QuickBooks Time.
- **Tools:** [[Tools/Clockify]], [[Tools/Toggl Track]], [[Tools/Miro]], [[Tools/GitHub]], [[Tools/Python]], [[Tools/PostgreSQL]], [[Tools/Google Calendar]], [[Tools/Claude in Chrome]].
- **Repos (github aracreate-group):** arm-ui-library, arm-make, arm-website, arm-docs, arm-cli, arm-app-admin, arm-app-calendar, arm-session, arm-core-fe, arm-core-be, arm-service-notification, arm-util-clockify.
- **Local folders:** ~/araCreate/ARM; test data in ~/araCreate/ARM/under build/timer/comparison analysis (with `_for_miro`).
- **Architecture (from Vishnu's diagrams, 2026-09-23/28):**
  - Layers: Bundle (e.g. Productivity) → Collection (e.g. Calendar, Operations) → Model (Events, Projects, Tasks, Time, Expenses) → Field (Time: entry, project, phase, tag) → Implementer (app/service).
  - Data flow (mockup): Clients → Projects → Tasks → Expenses → Time; Time → HR/Payroll; Time → Yes/No approval gate → Finance/Invoicing; Risks (from Leaves and Budget) feed the gate.
  - Apps today: Calendar (PUB Events, SUB Tasks/Notifications), Finance (Invoices, POs, Quotations), Admin (Users, Access, Apps), Core (list of Collections).
  - Servers: 2 Hetzner servers on a private network. DB server: Postgres "core" + MongoDB "calendar". Main server: Calendar FE, Calendar BE, Core FE.
- **Key competitor findings:** Clockify reject note never reaches the worker and shows a fake "failed" toast; Toggl lets admin silently edit approved hours and hides the reject note on hover; Harvest has no reject button, accepts negative values and 25h days; Hubstaff paywall popups cannot be closed.
- **Toggl 2.0 facts:** merges Track, Plan and Focus. Roles: Owner, Tool admin, Manager, Member, Time tracker, Guest (+ Admin toggle). Approvals weekly only (Mon–Sun): Not submitted → Pending review → Changes requested / Approved. Free plan up to 5 members; Time Off add-on $2/user/month.
- **Toggl gaps (design rules for our Timer):** approved entries editable with no warning; empty and future weeks can be submitted; approver can approve an unsubmitted week; cell being edited during Submit is lost; change-request note only on hover, no notification; invite flow forces new org and switches session; start-after-end saved as 22h, 1000h accepted; duplicate project names allowed; blocked URLs redirect silently.
- **Proposed answers to 8 decisions (Claude's draft, not confirmed):** no screenshots; replace the Clockify script later; approvals in MVP; invoices in V1.1; reopened week → "changes requested" with reason; weekly Mon–Sun; manager approves; no attendance features.
- **North star metric (proposed):** share of hours captured without typing.
- **Chats index:** INDEX (archived: Projects/timer/chats/INDEX.md)
- **Related:** [[Projects/arm-ui/SUMMARY]], [[Projects/clockify-automation-project/SUMMARY]], [[Projects/clockify/SUMMARY]]

## 7. Files and documents
(Claude project docs unless noted.)
- `status.md` — where work stands; read first in a new chat.
- `arm-architecture-v2.md` — new architecture reading + 7 open questions. Read before anything else.
- `arm-platform-reference.md` — ARM platform reference.
- `timer-app-analysis.md`, `timer-app-spec.md` — first analysis and spec (old standalone plan; needs rework).
- `timer-competitor-deep-dive.md` — 13-competitor desk research; also the "Timer Competitor Dossier" page.
- `timer-e2e-clockify-REAL.md`, `timer-e2e-toggl-REAL.md`, `timer-e2e-harvest-REAL.md`, `timer-e2e-hubstaff-REAL.md` — live test write-ups; `timer-e2e-jibble.md` — desk research only.
- `docs/claude-toggl-walkthrough.md` — the arm-timer Toggl walkthrough (from Claude project "arm-timer"). Its `shots/` (143 images) and `flow/` (Mermaid + PNG) files: location not known.
- `timer-live-test-tasks.md` — prompts for running live tests with another agent.
- `timer-product-findings.md` — insights, JTBD, scope, decisions, risks.
- `timer-explainer-simple.md` — plain-words explanation + "words to know".
- "Timer: Research to Product - Meeting Notes" — editable page.
- Miro "My First Board" — Timer sections.

## 8. Open questions and problems
- Who owns Clients (Operations, Finance, other)?
- Where do Leaves and Budget live? Is there an HR app?
- Will Risks be its own service?
- What does "phase" in the Time model mean?
- Who decides approval Yes/No: a manager, automatic rules, or both?
- Does Operations get its own back end and database?
- Diagram may have mistakes: calendar DB drawn to FE not BE; two boxes labelled "CALENDAR-APP-FE-NODE".
- Data-flow picture is only a mockup.
- 8 product decisions still open (monitoring first).
- Miro image upload blocked; 32 files waiting in `_for_miro`.
- Where are the Toggl `shots/` and `flow/` files? Toggl trial org and test accounts still exist (trial ends ~23 Oct 2026).

## 9. All chats in this project
- Timer project analysis (archived: Projects/timer/chats/2026-09-08 Timer project analysis.md) — 2026-09-08
- Progress update (archived: Projects/timer/chats/2026-09-08 Progress update.md) — 2026-09-08
- Project progress review (archived: Projects/timer/chats/2026-09-10 Project progress review.md) — 2026-09-10
- miro planning (archived: Projects/timer/chats/2026-09-23 miro planning.md) — 2026-09-23
- Where we stopped (archived: Projects/timer/chats/2026-09-28 Where we stopped.md) — 2026-09-28
- Moving project to viOS (7) (archived: Projects/timer/chats/2026-10-02 Moving project to viOS (7).md) — 2026-10-02 (personal, arm-timer)
