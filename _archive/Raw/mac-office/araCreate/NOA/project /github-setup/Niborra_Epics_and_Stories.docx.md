---
source: office Mac ~/araCreate/NOA/project /github-setup/Niborra_Epics_and_Stories.docx
---

Niborra
Epics, User Stories & Tasks — Full Evaluation Copy
Reconciled against client PRD ACG-NOA-001 · 20 July 2026
11 epics · 15 user stories · 43 features (acceptance criteria) · 100 technical tasks · 306 story points across the 10-sprint plan
E1 — Onboarding & Machine Pairing
Niborra Studio   •   In scope v1
Get a first-time, non-technical user from unboxing to a real printed letter in under 10 minutes, with the machine auto-detected and invisible.
Story E1.1
As a new Niborra owner, I want the app to auto-detect my machine over the network, so that I never install a driver or configure anything manually.
Features:
Discovery completes within 5 seconds on a standard network
UI shows “Looking for your Niborra...” then a green “Connected” state
If not found within 15 seconds, show a plain-language error with a retry action

Story E1.2
As a new Niborra owner, I want to capture my own handwriting once during setup, so that every future letter looks genuinely mine, with no external tool or account.
STATUS: Proceeding for v1 as scoped (decided 20 Jul 2026, Vishnu). Worth a quick confirmation with Ahmad/Shyam that this matches the PRD's "5 styles / Full library" pricing line, but not blocking Sprint 2 build start.
Features:
Capture happens in-app on a connected tablet/pad
User writes each letter of the alphabet once
Process takes under 5 minutes and requires no external website

Story E1.3
As Camille, I want to hold a real printed letter within 10 minutes of unboxing, so that I trust the product works before I commit my team to a campaign.
Features:
Full flow (unbox → connect → capture → template → preview → print) completes in under 10 minutes
No driver dialog, cell reference, or line of code is ever shown
Validated via an onboarding usability test before general release

Story E1.4
As an Operator, I want a single Start button with no visible settings, so that I can run today’s queue correctly without training or calling IT.
Features:
Operator Home shows only today’s queue and machine status
One primary action — no settings visible
A jam shows a plain-language fix plus Resume — never an error code

Linked technical tasks (13 tasks · 43 points)

E2 — Template & Campaign Designer
Niborra Studio   •   In scope v1
Let a non-technical manager build an on-brand, personalised letter without touching Excel, cell references, or a developer.
Story E2.1
As Sofia, I want to pick a saved template and drag my client list onto it, so that I can send 500 personalised birthday notes without starting from scratch each month.
Features:
Template gallery opens by default — never a blank canvas
Drag-and-drop CSV import with automatic header reading
Visual column mapping (drag “First Name” onto the greeting) — no cell references

Story E2.2
As Marcus, I want every letter in a 200-piece batch to look genuinely handwritten and unique, so that my mailers don’t look mass-produced next to everyone else’s printed flyers.
Features:
Natural variation is applied automatically via a single Natural ↔ Neat slider
No manual spacing fields (font size / word spacing / gaps) are ever exposed
Preview flips through real recipient samples before send

Story E2.3
As a designer or marketer, I want to add PNG/SVG/PDF assets and format text, so that I can build an on-brand letter without a developer.
Features:
Drag-and-drop or file-browser upload, whitelisted to PNG/SVG/PDF (JPG excluded per PRD)
All property changes (font, size, colour, alignment) update the canvas in real time
Unsupported file type shows a clear error, not a silent failure

Linked technical tasks (10 tasks · 40 points)

E3 — Contacts & Data Import
Niborra Studio   •   In scope v1
Remove the Excel-knowledge barrier that both competitors expose (cell references, row/column splitting).
Story E3.1
As a non-technical manager, I want to drop in my CRM export and have the app read the columns automatically, so that I never type a cell reference or learn what “splitting with rows” means.
Features:
CSV (Papa Parse) and XLSX (SheetJS) both supported
Preview of the first 5 rows shown before mapping
Send is blocked until every canvas variable field is mapped

Linked technical tasks (1 tasks · 4 points)

E4 — Send & Machine Monitor
Niborra Studio   •   In scope v1
One clear action to send, and full visibility into a running job — including graceful handling of jams and disconnects.
Story E4.1
As Sofia, I want to click one “Start writing 500 notes” action and get notified when it’s done, so that I don’t have to babysit the job or guess whether it finished.
Features:
Single confirm action after validation passes
Upload progress shown; email confirmation on completion
Every job is logged automatically for later reporting

Story E4.2
As an Operator, I want to see live progress and clear a paper jam myself, so that I can get back to my main job without calling for help.
Features:
Live “Writing 12 of 47” counter with pause / resume / cancel controls
Jam shows a plain-language fix plus Resume, never an error code
On disconnect: a banner appears, job state is preserved, and status reconciles automatically on reconnect

Linked technical tasks (11 tasks · 34 points)

E5 — Automations (No-Code Triggers)
Niborra Web Portal   •   DEFERRED — Phase 2
Originally: let a marketer wire up an automatic trigger without a developer, a laptop running a web server, or a CURL command. Deferred because the client PRD confirms the REST API/webhooks it depends on are post-launch, not a v1 requirement. No v1 sprint capacity is allocated to this epic.
Story E5.1
As Priya, I want a handwritten thank-you note to fire automatically when a Shopify order over €100 is placed, so that every qualifying customer gets one with zero manual work, set up once and forgotten.
Features:
One-click Shopify connection — no code, no API key handling by the user
Plain condition builder (“Only when order is over [ ]”)
A plain-English rule summary is shown before the automation is switched on

No technical tasks are tagged to this epic in the sprint plan.

E6 — Web Portal: Account, Team & Billing
Niborra Web Portal   •   In scope v1
Let an account owner manage people and subscription without contacting support, with every plan limit enforced server-side.
Story E6.1
As an account admin, I want to invite teammates and set their roles, so that the right people can create campaigns without me being a bottleneck.
Features:
Invite by email, role change, and remove-with-confirmation all supported
Starter plan enforces a 3-user maximum on the server, not just in the UI

Story E6.2
As an account owner, I want to see my plan, upgrade, and view billing history, so that I can manage cost without contacting support.
Features:
Plan comparison table across Free / Starter / Pro / Enterprise
Upgrade via Stripe Checkout; cancel/manage via Stripe Customer Portal
Invoice history with PDF download

Linked technical tasks (9 tasks · 34 points)

E7 — Fleet Management
Niborra Web Portal   •   In scope v1
Give an admin visibility and control over every machine on the account without being physically on-site — something neither competitor offers.
Story E7.1
As an admin, I want to see every machine’s status and add new ones from the portal, so that I can manage a growing fleet remotely.
Features:
Machines table shows name / online status / last seen / firmware version
Add machine generates a pairing token used once in Studio
Machine limits are enforced server-side (Free=1, Starter=2, Pro=4, Enterprise=unlimited)

Linked technical tasks (1 tasks · 4 points)

E8 — Reporting & Job History
Cross-cutting (Studio + Portal + Backend)   •   In scope v1
Give buyers something no competitor offers: a complete, exportable record of every job that ran.
Story E8.1
As a manager, I want every job logged with a downloadable report, so that I can prove to leadership exactly what ran, when, and for whom.
Features:
PDF and CSV export per job (name / date / machine / completed / failed / errors / duration)
Job history is filterable by date, machine, and status in the portal

Linked technical tasks (3 tasks · 10 points)

E9 — Platform & Infrastructure
Foundational — all repos   •   In scope v1
No discrete user story — definition of done: repos live with branch protection, all infrastructure hosted in the EU (GDPR; US region flagged as open decision), CI runs lint + test on every PR, and the auth API is live before any feature work depends on it.
No discrete user stories for this epic — it is infrastructure / cross-cutting work with a definition of done rather than a persona-facing story.
Linked technical tasks (18 tasks · 42 points)

E10 — Design System & Visual Foundation
Cross-cutting — enables every screen   •   In scope v1
No discrete user story — definition of done: tokens and component library match the brief (premium, calm, editorial — deep teal/navy + warm gold), and every hi-fi screen has a written spec with acceptance criteria before a line of UI code is written.
No discrete user stories for this epic — it is infrastructure / cross-cutting work with a definition of done rather than a persona-facing story.
Linked technical tasks (12 tasks · 37 points)

E11 — Launch Readiness
Cross-cutting — hardening & go-live   •   In scope v1
No discrete user story — definition of done: zero P0 bugs, a real-hardware end-to-end test passed, signed installers on macOS and Windows, GDPR / security / accessibility checks passed, monitoring live — nothing ships without PM sign-off.
No discrete user stories for this epic — it is infrastructure / cross-cutting work with a definition of done rather than a persona-facing story.
Linked technical tasks (22 tasks · 58 points)
