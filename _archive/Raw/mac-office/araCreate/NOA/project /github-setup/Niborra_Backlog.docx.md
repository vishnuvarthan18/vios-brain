---
source: office Mac ~/araCreate/NOA/project /github-setup/Niborra_Backlog.docx
---

Niborra
Product Backlog — Epics, User Stories & GitHub Setup
Prepared for project kickoff · 20 July 2026

How to read this document
This backlog turns the completed research — personas, competitor gap analysis, and UX flows — into 11 epics, each with user stories and acceptance criteria, and each linked to a compressed 10-sprint / 20-week technical plan (see below). Every story and every technical task below becomes one GitHub issue. Epics become GitHub milestones or a top-level label; sprints become milestones with due dates; the sprint plan's Task IDs are preserved from the original v0.2 plan so nothing already agreed gets renumbered — only the sprint they're scheduled in has changed.
Source documents: Niborra Full UX Research (personas, JTBD, journey maps, IA, flows), Niborra Competitor Analysis (sourced findings, opportunity map), Niborra Sprint Planning v0.2 (87 tasks, originally 12 sprints, now compressed to 10), and the formal Project Requirement Document (ACG-NOA-001, v1.0, client-approved).
Reconciled against the client PRD (ACG-NOA-001)
This backlog has been checked against the formal client-approved PRD. Most of it held up unchanged — competitors, positioning, personas, verticals, subscription tiers, and machine limits all match. Four things changed or need a decision:
Roles confirmed: Admin/Owner, Manager, and Operator (three tiers, no separate Viewer role at v1).
E5 Automations deferred to Phase 2 (post-launch). The PRD confirms the REST API and webhooks — which E5 depends on entirely — are a post-launch feature, not a v1 requirement. No sprint capacity was ever allocated to E5 build work, so this costs the schedule nothing; it only affects the Priya/e-commerce launch story and the Pro-tier “REST API/Webhooks” pricing line, which should be caveated as “coming post-launch” until it ships.
Hosting: infrastructure is being built on Hetzner (EU / Frankfurt) for v1 and staging. The PRD calls for EU + US data residency — that multi-region requirement is not yet resolved and is flagged below as an open decision.
Hard dates now confirmed: hardware MVP mid-September 2026, V1.0 software launch December 2026, first pilot batch of 10 machines ships February 2027. This plan's Sprint 10 (14–27 Nov 2026) already lands ahead of both.
Gap-analysis passes (20 Jul 2026): 13 tasks added
Two structured reviews of this backlog against the PRD found real gaps. Thirteen tasks were added as a result — G01–G13, tagged in every list below and in GitHub. Pass 1 (G01–G09): Linux support (testing + installer, PRD requires all three OSes at launch), the Studio-side OTA firmware push, a proactive low-consumables prompt (the % display already existed, the prompt didn't), data encryption at rest/in transit, an ISO 27001-aligned access review, trial-expiration/auto-downgrade handling, a dedicated and separately-pointed handwriting humanization engine (the product's core IP, previously had no owning task), and an earlier hardware-in-the-loop smoke test. Pass 2 (G10–G13): OTA failure handling and rollback, the Pro-tier in-app analytics dashboard the PRD requires at launch, pilot beta account provisioning ahead of the Feb 2027 shipment (not just the Sprint 10 release itself), and incremental cross-platform smoke checks starting Sprint 5 instead of one batched pass in Sprint 8. G06 (access review) was also moved from Sprint 6 to Sprint 7 so it evaluates a stable system rather than one still being built that same sprint. Total scope grew by 53 points (253 → 306) across both passes — this is real added work, not free.
Open risks / decisions needed
Hosting region — DECIDED (20 Jul 2026, Vishnu): locking to EU-only (Hetzner Frankfurt) for v1. The PRD's EU+US data residency requirement is not being built now; if it's genuinely needed, it comes back as a change request rather than blocking the current plan.
Handwriting capture vs. style library (E1.2) — DECIDED (20 Jul 2026, Vishnu): proceeding with live capture for v1 as originally scoped. Still worth a quick confirmation with Ahmad/Shyam that this matches what the PRD's “5 styles / Full library” pricing line meant, but it is not blocking Sprint 2 build start.
niborra_format_matrix.xlsx, referenced in the PRD for card format specs, has not been supplied yet — needed to finalise E2 acceptance criteria.
Ask when the team reaches that task (not needed at kickoff)
Studio↔Core communication protocol is marked TBD in the client PRD. Sprint 3's machine-pairing work (S7-01) currently assumes mDNS discovery as a working assumption. Confirm with the firmware team before Sprint 3 starts (not before kickoff) — the Job compilation engine (S9-01, Sprint 5) depends on this being locked by then.
Humanization engine architecture (G08, Sprint 8): is stroke/pressure variation computed in Studio or in firmware? Ask Ahmad's firmware team when G08 is approaching, not now.
OTA bundle format and AN3155 handshake (G03, Sprint 8): ask the firmware team when G03 is approaching.
OTA rollback support (G10, Sprint 8): ask what the bootloader supports when G10 is approaching.

Personas
Camille  — Guest Experience Manager · Hospitality (80-room boutique hotel). Wants every VIP guest to feel personally recognised on arrival, without her team handwriting notes by hand.
Sofia  — Clienteling Manager · Luxury Retail (flagship + 6 boutiques). Wants consistent, genuinely personal outreach to top-tier VIP clients across every boutique.
Marcus  — Real Estate Agent & Team Lead · Residential (4-agent team). Wants neighbourhood-farming mail that actually gets opened, at a volume he can’t write by hand.
Priya  — Retention Marketing Manager · D2C Beauty (~€8M revenue). Wants every Shopify order over €100 to trigger a note automatically, with zero manual work.
The Operator  — Front desk / retail / fulfilment staff · appears in every vertical. Just needs to press the right button and know immediately if something’s wrong.
Team structure
Delivery team
Vishnu  — Designer. Owns UX/UI design across Studio and Portal.
Shyam  — PM — client point of contact. Owns sprint planning and status tracking; the only person who speaks to Ahmad on the client side.
Kishor  — Developer. Covers both frontend (Electron/Next.js) and backend (Node/Postgres) build work — the sprint plan's separate 'Frontend Dev' and 'Backend Dev' roles both map to Kishor.
Ragul  — QA / Tester. Owns testing and hardening — including the dedicated hardening sprint, reassigned from the original plan's generic 'All' / dev-owned testing tasks.
Client
Ahmad  — Client — hardware/firmware owner. Point of contact for this engagement. Scope is software-only (Studio + Portal); the machine and firmware are Ahmad's side. He is the source for the firmware API spec that Sprint 1 (S1-06) depends on.
Compressed schedule: 10 sprints, 20 weeks (13 Jul – 27 Nov 2026)
Dates corrected 20 Jul 2026: Sprint 1 actually started 13 Jul, not 20 Jul as first planned. Every sprint after it shifted to match, keeping the same 2-week rhythm — scope and story points are unchanged, only the calendar moved. Net effect: this plan now lands ahead of where it started, not behind.
The original 24-week plan ran design (Sprints 2–5) and build (Sprints 6–10) back to back — Kishor was largely idle during the first 10 weeks. This version runs them in parallel: Kishor starts real build work from Sprint 1 on whatever has no design dependency, and picks up each screen as soon as Vishnu signs off its hi-fi design. That reclaims roughly 6 weeks of otherwise-idle capacity.
Honest tradeoff: after two gap-analysis passes, Sprints 3–8 now run Kishor at 23–28 points per 2-week sprint — meaningfully above his historical solo pace (peaked at 12–14 on spikes) and above what the original plan budgeted even for two developers combined. This is no longer a minor stretch on a couple of sprints; it is six consecutive sprints at or above capacity. Three real options, not mutually exclusive: (1) extend the timeline — the actual hard deadline is the February 2027 pilot shipment, not the December 2026 V1.0 date, and this plan currently has roughly 8 weeks of buffer against that real deadline, some of it can be spent; (2) descope specific items to Phase 2 — candidates include G11 (Pro analytics dashboard, 5 points) and parts of E10 polish; (3) add a second developer for overflow/spike work in the Sprint 4–8 window. This needs a real decision with Ahmad, not a plan that quietly assumes it will work out.
Why Niborra wins — the opportunity map
Every product principle below is traceable to a sourced competitor pain point (Signascript / UUNA TEK). This is the argument for the client: not a feature list, but a direct response to documented failure.

Epic summary
E1 — Onboarding & Machine Pairing
Niborra Studio  ·  13 linked tasks  ·  43 story points
Persona: Camille (hospitality) · applies to every first-time user · the Operator
Get a first-time, non-technical user from unboxing to a real printed letter in under 10 minutes, with the machine auto-detected and invisible.
User stories
Story E1.1
As a new Niborra owner, I want the app to auto-detect my machine over the network, so that I never install a driver or configure anything manually.
Acceptance criteria:
Discovery completes within 5 seconds on a standard network
UI shows "Looking for your Niborra..." then a green "Connected" state
If not found within 15 seconds, show a plain-language error with a retry action

Story E1.2
As a new Niborra owner, I want to capture my own handwriting once during setup, so that every future letter looks genuinely mine, with no external tool or account.
STATUS: Proceeding for v1 as scoped (decided 20 Jul 2026, Vishnu). The PRD's subscription table ("handwriting styles: 5 / Full library" by tier) reads like a preset style library, which may not be the same thing as this story's live capture — worth a quick confirmation with Ahmad/Shyam, but not blocking Sprint 2 build start.
Acceptance criteria:
Capture happens in-app on a connected tablet/pad
User writes each letter of the alphabet once
Process takes under 5 minutes and requires no external website

Story E1.3
As Camille, I want to hold a real printed letter within 10 minutes of unboxing, so that I trust the product works before I commit my team to a campaign.
Acceptance criteria:
Full flow (unbox → connect → capture → template → preview → print) completes in under 10 minutes
No driver dialog, cell reference, or line of code is ever shown
Validated via an onboarding usability test before general release

Story E1.4
As an Operator, I want a single Start button with no visible settings, so that I can run today's queue correctly without training or calling IT.
Acceptance criteria:
Operator Home shows only today's queue and machine status
One primary action — no settings visible
A jam shows a plain-language fix plus Resume — never an error code

Linked technical backlog (from Sprint Plan v0.2)

E2 — Template & Campaign Designer
Niborra Studio  ·  10 linked tasks  ·  40 story points
Persona: Sofia (luxury retail) · Marcus (real estate)
Let a non-technical manager build an on-brand, personalised letter without touching Excel, cell references, or a developer.
User stories
Story E2.1
As Sofia, I want to pick a saved template and drag my client list onto it, so that I can send 500 personalised birthday notes without starting from scratch each month.
Acceptance criteria:
Template gallery opens by default — never a blank canvas
Drag-and-drop CSV import with automatic header reading
Visual column mapping (drag “First Name” onto the greeting) — no cell references

Story E2.2
As Marcus, I want every letter in a 200-piece batch to look genuinely handwritten and unique, so that my mailers don’t look mass-produced next to everyone else’s printed flyers.
Acceptance criteria:
Natural variation is applied automatically via a single Natural ↔ Neat slider
No manual spacing fields (font size / word spacing / gaps) are ever exposed
Preview flips through real recipient samples before send

Story E2.3
As a designer or marketer, I want to add PNG/SVG/PDF assets and format text, so that I can build an on-brand letter without a developer.
Acceptance criteria:
Drag-and-drop or file-browser upload, whitelisted to PNG/SVG/PDF (JPG excluded per PRD)
All property changes (font, size, colour, alignment) update the canvas in real time
Unsupported file type shows a clear error, not a silent failure

Linked technical backlog (from Sprint Plan v0.2)

E3 — Contacts & Data Import
Niborra Studio  ·  1 linked tasks  ·  4 story points
Persona: Sofia · Priya
Remove the Excel-knowledge barrier that both competitors expose (cell references, row/column splitting).
User stories
Story E3.1
As a non-technical manager, I want to drop in my CRM export and have the app read the columns automatically, so that I never type a cell reference or learn what “splitting with rows” means.
Acceptance criteria:
CSV (Papa Parse) and XLSX (SheetJS) both supported
Preview of the first 5 rows shown before mapping
Send is blocked until every canvas variable field is mapped

Linked technical backlog (from Sprint Plan v0.2)

E4 — Send & Machine Monitor
Niborra Studio  ·  11 linked tasks  ·  34 story points
Persona: Sofia · Camille · the Operator
One clear action to send, and full visibility into a running job — including graceful handling of jams and disconnects.
User stories
Story E4.1
As Sofia, I want to click one “Start writing 500 notes” action and get notified when it’s done, so that I don’t have to babysit the job or guess whether it finished.
Acceptance criteria:
Single confirm action after validation passes
Upload progress shown; email confirmation on completion
Every job is logged automatically for later reporting

Story E4.2
As an Operator, I want to see live progress and clear a paper jam myself, so that I can get back to my main job without calling for help.
Acceptance criteria:
Live “Writing 12 of 47” counter with pause / resume / cancel controls
Jam shows a plain-language fix plus Resume, never an error code
On disconnect: a banner appears, job state is preserved, and status reconciles automatically on reconnect

Linked technical backlog (from Sprint Plan v0.2)

E5 — Automations (No-Code Triggers) — Deferred to Phase 2
Niborra Web Portal  ·  0 linked tasks  ·  0 story points
Persona: Priya
DEFERRED TO PHASE 2 (post-launch): the client PRD confirms the REST API and webhooks this feature depends on are not a v1 requirement. Kept here so the story isn't lost, and to make clear it is roadmap, not a v1 commitment. No v1 sprint capacity is allocated to it. Original intent: let a marketer wire up an automatic trigger without a developer, a laptop running a web server, or a CURL command — the exact failure mode of both competitors' APIs.
User stories
Story E5.1
As Priya, I want a handwritten thank-you note to fire automatically when a Shopify order over €100 is placed, so that every qualifying customer gets one with zero manual work, set up once and forgotten.
Acceptance criteria:
One-click Shopify connection — no code, no API key handling by the user
Plain condition builder (“Only when order is over [ ]”)
A plain-English rule summary is shown before the automation is switched on

Linked technical backlog (from Sprint Plan v0.2)

E6 — Web Portal: Account, Team & Billing
Niborra Web Portal  ·  9 linked tasks  ·  34 story points
Persona: Sofia (admin) · Camille (admin)
Let an account owner manage people and subscription without contacting support, with every plan limit enforced server-side.
User stories
Story E6.1
As an account admin, I want to invite teammates and set their roles, so that the right people can create campaigns without me being a bottleneck.
Acceptance criteria:
Invite by email, role change, and remove-with-confirmation all supported
Starter plan enforces a 3-user maximum on the server, not just in the UI

Story E6.2
As an account owner, I want to see my plan, upgrade, and view billing history, so that I can manage cost without contacting support.
Acceptance criteria:
Plan comparison table across Free / Starter / Pro / Enterprise
Upgrade via Stripe Checkout; cancel/manage via Stripe Customer Portal
Invoice history with PDF download

Linked technical backlog (from Sprint Plan v0.2)

E7 — Fleet Management
Niborra Web Portal  ·  1 linked tasks  ·  4 story points
Persona: Camille · Marcus (multi-machine teams)
Give an admin visibility and control over every machine on the account without being physically on-site — something neither competitor offers.
User stories
Story E7.1
As an admin, I want to see every machine’s status and add new ones from the portal, so that I can manage a growing fleet remotely.
Acceptance criteria:
Machines table shows name / online status / last seen / firmware version
Add machine generates a pairing token used once in Studio
Machine limits are enforced server-side (Free=1, Starter=2, Pro=4, Enterprise=unlimited)

Linked technical backlog (from Sprint Plan v0.2)

E8 — Reporting & Job History
Cross-cutting (Studio + Portal + Backend)  ·  3 linked tasks  ·  10 story points
Persona: Camille · Sofia — proving ROI to leadership
Give buyers something no competitor offers: a complete, exportable record of every job that ran.
User stories
Story E8.1
As a manager, I want every job logged with a downloadable report, so that I can prove to leadership exactly what ran, when, and for whom.
Acceptance criteria:
PDF and CSV export per job (name / date / machine / completed / failed / errors / duration)
Job history is filterable by date, machine, and status in the portal

Linked technical backlog (from Sprint Plan v0.2)

E9 — Platform & Infrastructure
Foundational — all repos  ·  18 linked tasks  ·  42 story points
Persona: Internal (Product, Backend, Frontend)
Definition of done: repos live with branch protection, all infrastructure hosted in the EU (GDPR), CI runs lint + test on every PR, and the auth API is live before any feature work depends on it.
Linked technical backlog (from Sprint Plan v0.2)

E10 — Design System & Visual Foundation
Cross-cutting — enables every screen  ·  12 linked tasks  ·  37 story points
Persona: Internal (Design, PM) — enables all personas
Definition of done: tokens and component library match the brief (premium, calm, editorial — deep teal/navy + warm gold), and every hi-fi screen has a written spec with acceptance criteria before a line of UI code is written.
Linked technical backlog (from Sprint Plan v0.2)

E11 — Launch Readiness
Cross-cutting — hardening & go-live  ·  22 linked tasks  ·  58 story points
Persona: Internal (all roles) — gates the public launch
Definition of done: zero P0 bugs, a real-hardware end-to-end test passed, signed installers on macOS and Windows, GDPR / security / accessibility checks passed, monitoring live — nothing ships without PM sign-off.
Linked technical backlog (from Sprint Plan v0.2)

GitHub project structure
This backlog maps directly onto GitHub as follows:
Repo: a single monorepo, aracreate-group/external-niborra-app, with folders for Studio (Electron), Web (Next.js portal), API (backend), and shared/meta work — one repo, one CI pipeline, branch protection on main. (Supersedes the original S1-03 plan of 4 separate repos — a monorepo is a better fit for a 4-person team with one developer covering both frontend and backend.)
Milestones: one per sprint (Sprint 1–Sprint 10), kickoff locked to Monday 13 July 2026. Sprint 1 runs 13–24 Jul 2026; Sprint 10 (Launch) runs 14–27 Nov 2026 — ahead of the February 2027 pilot batch forcing function, with roughly 9-10 weeks of buffer.
Labels — epic:E1..E11; priority:P0/P1/P2; sprint:1..10; area:studio/web/api/meta; owner:vishnu / owner:shyam / owner:kishor / owner:ragul; type:story / type:task.
Project board (GitHub Projects): columns Backlog → Ready → In Progress → In Review → Done, grouped/filterable by epic, sprint, and assignee.
Issues: every user story and every technical task becomes one issue, titled with its ID, body containing acceptance criteria / description / assignee, labelled by epic + priority + sprint + owner, assigned to its sprint milestone. Phase-2 items (currently just E5) go to a separate "Phase 2 (Post-Launch)" milestone instead of a sprint, so they stay visible without being mistaken for committed v1 work.
Next steps
Connect GitHub access (Claude in Chrome or a GitHub connector) to create the org, repos, labels, and milestones live.
Bulk-create all issues from the accompanying CSV (niborra_github_issues.csv) via GitHub CLI or the Projects import — see import_to_github.py.
Raise the solo-developer capacity risk (Sprints 3–7) with Ahmad before treating the 20-week timeline as fixed.
Resolve the hosting region question (EU-only vs EU+US) with Ahmad/Shyam — see Open risks above.