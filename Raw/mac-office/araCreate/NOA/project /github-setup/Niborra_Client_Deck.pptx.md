---
source: office Mac ~/araCreate/NOA/project /github-setup/Niborra_Client_Deck.pptx
---

NIBORRA
Client Kickoff — Project Plan Overview
20 July 2026  ·  araCreate ↔ Ahmad Taleb

What we'll cover today

1
How we organized the project
PRD → Epics → Stories → Tasks → Sprints

2
The 15 user stories
What we're actually building, epic by epic

3
Sprint plan & timeline
10 sprints, 13 Jul – 27 Nov 2026

4
Team & ownership
Who's doing what

5
The one open risk
Kishor's capacity, and three options

6
Questions for today
What we need from you to keep moving
Niborra — Client Kickoff, 20 Jul 2026
2
How we split this project

PRD
Client's formal, approved requirements (ACG-NOA-001)
→

EPIC
11 big feature areas, e.g. Campaign Designer
→

STORY
15 real user needs — As a [user], I want [x], so that [y]
→

TASK
100 pieces of engineering work, 306 story points
→

SPRINT
10 two-week blocks with real deadlines
Two review passes after the PRD landed added 13 more tasks it implied but didn't spell out — OTA rollback, data encryption, cross-platform testing, and more.
Worked example
Epic: Campaign Designer   →   Story: Send a handwritten card automatically at check-in   →   Tasks: build trigger, connect PMS, design template, test print queue
Niborra — Client Kickoff, 20 Jul 2026
3
Where the project stands

11
Epics

15
User stories

100
Tasks

306
Story points

10
Sprints
10 sprints · 20 weeks · 13 Jul – 27 Nov 2026 — ahead of the Feb 2027 pilot shipment
Design and development run in parallel rather than back to back — that reclaimed roughly 6 weeks against the original 24-week plan.
Niborra — Client Kickoff, 20 Jul 2026
4
The 15 user stories — Niborra Studio
What a first-time user experiences: unbox, design, send

E1
Onboarding & Machine Pairing  
Unbox to a real printed letter in under 10 minutes.
New owner: auto-detect my machine, so I never install a driver
Camille: hold a printed letter in 10 minutes, so I trust the product

E2
Template & Campaign Designer  
Build an on-brand letter without Excel or a developer.
Sofia: drag my client list onto a template, so I can send 500 notes fast
Marcus: every letter looks genuinely handwritten, so mailers aren't mass-produced

E3
Contacts & Data Import  
Remove the Excel-knowledge barrier both competitors expose.
Manager: the app reads my CRM export automatically, so I never type a cell reference

E4
Send & Machine Monitor  
One clear action to send, full visibility while it runs.
Sofia: one Start action with a completion notification
Operator: clear a paper jam myself, without calling for help
Niborra — Client Kickoff, 20 Jul 2026
5
The 15 user stories — Niborra Web Portal
Managing the account, the team, and the machine fleet remotely

E5
Automations (No-Code Triggers)  · DEFERRED TO PHASE 2
Depends on the REST API/webhooks — client PRD confirms these ship post-launch.
Priya: a thank-you note fires automatically on a Shopify order over €100

E6
Account, Team & Billing  
Manage people and subscription without contacting support.
Admin: invite teammates and set their roles
Owner: see my plan, upgrade, and view billing history

E7
Fleet Management  
Manage every machine on the account remotely — a gap neither competitor covers.
Admin: see every machine's status and add new ones from the portal
Niborra — Client Kickoff, 20 Jul 2026
6
The 15 user stories — Cross-cutting & Foundational
The one user-facing story, plus three epics with no single story — they're "definition of done"

E8
Reporting & Job History  
A complete, exportable record of every job that ran.
Manager: every job logged with a downloadable report

E9
Platform & Infrastructure  
No discrete story — done when repos are live, EU-hosted, CI runs on every PR.
Definition of done, not a single feature — foundational to every other epic

E10
Design System & Visual Foundation  
No discrete story — done when the visual language is locked before UI code starts.
Definition of done — the tokens and components every screen is built on

E11
Launch Readiness  
No discrete story — done when testing, security, and accessibility checks pass.
Definition of done — nothing ships without this sign-off
Niborra — Client Kickoff, 20 Jul 2026
7
Sprint plan — 13 Jul to 27 Nov 2026
Dates corrected 20 Jul — Sprint 1 actually started 13 Jul, not 20 Jul as originally planned.
Niborra — Client Kickoff, 20 Jul 2026
8
Who owns what


V
Vishnu
Designer
E1, E2, E4, E6, E10
Owns hi-fi design across Studio + Portal; design system foundation


S
Shyam
PM — client contact
E9, E10, E11
PRD, sprint planning, sign-off; only person who talks to Ahmad


R
Ragul
QA / Tester
E11
Leads hardening/testing sprint; hardening tasks reassigned to him


K
Kishor
Developer
E1–E9 (nearly all build work)
Frontend + backend, solo dev — the capacity risk on the next slide
Niborra — Client Kickoff, 20 Jul 2026
9
The one open risk — Kishor's capacity
Sprints 3–8 run Kishor at 23–28 story points per two-week sprint. His demonstrated solo pace is 12–14. Six sprints at or above capacity — not guaranteed.
Sprints 3–8 planned load

25.5 pts / sprint
Kishor's historical solo pace

13 pts / sprint
Three options on the table

Extend the timeline
~8 weeks of real buffer before the Feb 2027 pilot shipment

Cut scope to Phase 2
Pro analytics dashboard (G11) is the obvious first candidate

Add a second developer
For the Sprint 4–8 window specifically
Niborra — Client Kickoff, 20 Jul 2026
10
Questions for today

1
Capacity
Extend timeline, cut scope, or add a developer for Sprints 4–8?

2
E5 Automations
Confirm the deferral to Phase 2 — and soften the "REST API: Yes" pricing line until it ships.

3
Format spec file
niborra_format_matrix.xlsx was referenced in the PRD but never attached — can you send it?

4
Handwriting: capture vs. presets
Confirm live handwriting capture is what the pricing table's "5 styles / Full library" means.
Niborra — Client Kickoff, 20 Jul 2026
11
Ask later, and already decided

Ask later — not needed today
Studio↔Core protocol — before Sprint 3 (mid-Aug)
Humanization engine: Studio or firmware? — before Sprint 8
OTA bundle format + AN3155 handshake — before Sprint 8
OTA rollback support — before Sprint 8
Pilot beta customer list — before Sprint 9 (mid-Nov)

Already decided
Hosting: EU-only (Hetzner Frankfurt) for v1 — change request later if genuinely needed
E1.2 handwriting capture: proceeding as scoped for v1
Repo structure: single monorepo, not 4 separate repos
E5 Automations: deferred to Phase 2 — costs the schedule nothing
Niborra — Client Kickoff, 20 Jul 2026
12
After this meeting

1
Capacity decision → move issues to Phase 2, or shift sprint due dates

2
E5 sign-off → close the question in GitHub (issue already in Phase 2)

3
Format matrix file → finalise E2 acceptance criteria

4
Handwriting confirmation → keep or rewrite E1.2 and its acceptance criteria

5
Everything lives in GitHub — Backlog → Ready → In Progress → In Review → Done
Thank you.