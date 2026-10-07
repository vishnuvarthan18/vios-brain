---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: arm-ui (araMetrics platform UI/UX)
State: [[Projects/arm-ui/STATE]] · Log: [[Projects/arm-ui/LOG]]

## 1. What this project is
- A Claude project called "arm-ui". It is the full v1 UI/UX for **araMetrics**.
- araMetrics is a modular "super-app": one web platform, one login, and each user sees only the apps their role allows.
- Built from scratch. Nothing existed before (no shell, no modules, no design system).
- Owner company: maybe [[Companies/araCreate Group]] (not sure). Dev domain: dev.arametrics.app.
- [[People/Vishnu]] is the designer. Goal: design the whole v1 UI and hand a real frontend codebase to the engineering team.
- 3 surfaces in v1:
  - **Core shell** — login, left sidebar with permitted apps, global search, role-based Home, profile/settings.
  - **Admin Portal** — first product. Ops console that replaces terminal work. 9 feature areas under 7 nav sections (Overview, Users, Monitoring, Calendar Ops, Logs, Security, Audit Log).
  - **Calendar module** — Calendar Merger. Second app. Proves new apps plug into the shell cleanly. Uses [[Tools/Google Calendar]].
- 3 roles: Admin (full control), Operator (near read-only), Platform User (never sees the Admin Portal).
- Work follows an 8-stage AI-native pipeline: Research → Define → Flows/IA → Design system → Generate → Refine → Prototype → Handoff. One Claude chat per stage.
- Also a test of Vishnu's new way of working: AI-native design, less [[Tools/Figma]], design + frontend in one role.
- Related (not sure): [[Projects/timer/SUMMARY]] — also "arm", maybe the same araMetrics family.

## 2. Status now (as of 2026-10-06)
- **Active.** (earlier: marked "paused since 2026-07-14" — but Claude Code work on ARM ran from 2026-08-29 to 2026-10-05, see [[Projects/arm-ui/DEV-LOG]].)
- Real ARM work now happens in the team repos (cloned 10 ARM repos into `~/araCreate/ARM` on 2026-09-08):
  - `arm-core-fe`: araMetrics landing page (hero SVG, `dataset.svg` section) pushed to `feature/aramerics-landing-page` (2026-08-30).
  - `arm-website` (marketing site, Vite + React): content pass, cookie banner, animated Projects/Vendors SVGs, responsive pass, legal fixes. PR #1 `feat/landing-content-pass` → `main` open (2026-09-03).
  - `arm-service-notification`: 4 email templates restyled to araMetrics brand. PR #1 into `dev` open (2026-09-03).
  - New `timer` app (module-federation remote, `localhost:10001/timer/`) started 2026-10-05: basic stopwatch with ARM UI components; runs standalone, not yet tested inside the shell. See [[Projects/timer/SUMMARY]].
- Original v1 UI/UX pipeline (Claude chats, June–July): Stages 1–4 done and locked; Stage 5 (Generate) in progress, Stage 6 (Polish) started in a Next.js + Radix Themes codebase. Done in code: Radix theme + brand tokens + Poppins; Core shell sidebar (expanded + collapsed). Calendar Merger rebuilt once (review not sure). Login/sign-up on hold. Not started: top bar polish, Home payloads, Admin Portal screens, prototype, handoff.
- 2026-07-14: Vishnu was moving the project to an enterprise Claude account (not sure if this finished).

## 3. Next steps
Repo work (newest):
- Timer app: add task/project label next; test inside the shell.
- Get arm-website PR #1 and notification PR #1 reviewed/merged.
- Website: real endpoint for the contact form; GDPR rewrite + Impressum for legal pages.
- Move all apps from `@aracreate/test-arm-ui` (deprecated) to `@aracreate/arm-ui`.

v1 UI/UX pipeline:
1. Settle the open spec conflicts first (see section 8 — mainly self-serve sign-up vs admin-only provisioning).
2. Polish the shell top bar (search, admin context label, avatar menu).
3. Build the regular-user Home payload, then the admin Home (= Admin Overview).
4. Come back to login/sign-up: full-page, left brand panel with geometric animation, right form, 3-step sign-up with email verification.
5. Run the impeccable + taste-skill polish passes on finished shell screens.
6. Build the Admin Portal Phase 1 slice: Users (5 P0 screens), with the permission fence for Operator.
7. Polish the Calendar Merger (6 P0 screens) for stakeholder review.
8. Push built screens into [[Tools/Figma]] (code → canvas) for stakeholders.
9. Stage 7 prototype and test, then Stage 8: push the codebase to [[Tools/GitHub]] for the dev team.

## 4. Decisions
- 2026-06-19 — Use design-native AI tools, not code-generation tools, for the design work — Vishnu wants to move away from Figma-heavy work #decision
- 2026-06-19 — One role-aware shell for all users, no separate interfaces; differs in 3 ways: sidebar filtered by permission, Home payload by role, quiet admin context label — simpler and safer #decision
- 2026-06-19 — Calendar is an app inside the shell, a peer of Admin Portal (not a separate surface) — Vishnu's correction #decision
- 2026-06-19 — Build order: Core shell → Admin Portal → Calendar — shell is the frame everything needs #decision
- 2026-06-19 — Design system built once on Radix Themes + amber #F9BF3B + Sand gray; clean, minimal, corporate — one consistent system #decision
- 2026-06-19 — Web only for v1; shell responsive-aware; Admin screens desktop-first — mobile later #decision
- 2026-06-19 — Quality bar: MVP scope but production-ready, no throwaway mockups #decision
- 2026-06-19 — 8-stage pipeline, one Claude chat per stage, with project instructions + requirements file in the Claude project — keep chats consistent #decision
- 2026-06-21 — Operator = near read-only watcher, almost no write actions — Vishnu cut scope #decision
- 2026-06-21 — Dashboard kept simple: little data, few actions #decision
- 2026-06-21 — Audit log is view-only, no download/export in v1 #decision
- 2026-06-21 — Operator "note / report a problem" has no UI in v1 (out-of-band) #decision
- 2026-06-21 — Same login for all roles; only the view payload changes #decision
- 2026-06-21 — Search is shell-level only (apps + settings), role-scoped; users see no admin results #decision
- 2026-06-21 — Provisioning only through Admin Portal → Users, no self-serve (later overridden, see below) #decision
- 2026-06-21 — App bundles / industry categories scoped out of v1 #decision
- 2026-06-21 — Calendar personas "privacy-conscious user" and "scheduler" merged into 2 modes of one persona #decision
- 2026-06-21 — Stage 2: about 19 P0 screens — Shell 8 (all P0), Admin 14 (5 P0 = Users slice, 9 P1), Calendar 6 (all P0); every screen needs empty/loading/error/success #decision
- 2026-06-21 — Permission fence: governance controls are absent from the Operator DOM, not greyed out — security #decision
- 2026-06-21 — Stage 3: Admin Overview (A1) and admin Home (S4) are the same screen — avoids two dashboards #decision
- 2026-06-21 — Shell→app mount contract: shell owns frame; app gives sidebar entry, permission, routes, section-nav yes/no, and can ask for the context label #decision
- 2026-06-21 — Claude Design output is reference/exploration only, not source of truth; build surface by surface, starting with the shell — avoid visual drift #decision
- 2026-06-21 — Go code-first: build screens in Claude Code on tuned Radix; skip Figma→code generation; Figma Radix file and code share the same tokens — both are Radix #decision
- 2026-06-21 — Code is source of truth for screens; push code → Figma for review, never edit Figma frames expecting them to flow back #decision
- 2026-06-21 — Next.js App Router + Radix Themes, no Tailwind — avoids CSS order conflicts #decision
- 2026-06-21 — Custom Radix colour scales: accent from #F9BF3B, gray from #555555; theme-overrides.css loaded after Radix styles; Poppins 400/500/600 #decision
- 2026-06-21 — Work in VS Code + Claude Code extension — see files, preview and agent in one window #decision
- 2026-06-21 — Visual direction: Cal.com-style restraint, light mode as primary; amber only as a signal (primary buttons, active nav, focus rings) #decision
- 2026-06-21 — Install Claude Code skills pbakaus/impeccable and leonxlnx/taste-skill; run them after each build #decision
- 2026-06-21 — Calendar Merger redefined: mirror busy/free across multiple Google accounts (not an in-app unified view) — Vishnu's correction #decision
- 2026-06-21 — Two-level selection: pick source accounts + calendars inside each; one target account; target can't be a source #decision
- 2026-06-21 — Target writes to its primary calendar by default (no calendar picker on target) #decision
- 2026-06-21 — Privacy default on target = "Busy only"; option "Show event titles" #decision
- 2026-06-21 (not sure of date) — Add self-serve sign-up (overrides admin-only provisioning) — Vishnu's call; not yet reconciled with spec #decision
- 2026-06-21 (not sure of date) — Login: full page, left brand/geometric animated panel, right form; email + Google sign-in (no Apple); 3-step sign-up (email + terms → verify email by link or 6-digit OTP → create password with strength bar) #decision
- 2026-06-22 — Keep and fix the bar-chart illustration in the "One platform / 24%" login card, do not remove it #decision
- 2026-06-22 — Leave auth for now and move to the Core shell #decision
- 2026-06-22 — The 14-item app list in the reference image is layout only, not real araMetrics content #decision
- 2026-06-22 — Spec override: active nav uses an amber filled box (as in the reference), not an amber left bar #decision
- 2026-06-22 — Collapsed sidebar swaps arm-logo.svg for arm-icon.svg, with a chevron toggle #decision
- 2026-08-29 — Repo work: do only what Vishnu asks, in exact pixel steps; no "outsmarting". #decision
- 2026-08-30 — Exclude the 21 MB "Figma design build request" folder from git. #decision
- 2026-09-03 — Website copy: no dashes, humanised text, only `#555555` as dark grey. arm-website uses feature branch + PR into `main` (no `dev` branch). #decision
- 2026-09-03 — Emails: Poppins Light + Regular only, inline SVG logo, no change to API/topics/triggers; push to `dev`, never `main`. #decision
- 2026-10-05 — Timer app uses ARM UI components only. #decision


## 5. Timeline
- 2026-06-19 — Chat on AI tools for UI/UX; araMetrics introduced; full scope defined; admin portal proposal read; 8-stage pipeline made; project instructions + requirements brief written.
- 2026-06-21 — "Where we stopped": Stage 1 recap; 3 open questions found.
- 2026-06-21 — Stage 1 (Research) done and locked; 3 open questions answered.
- 2026-06-21 — Prompt for Claude Design reference mockups written; advice: use as reference only.
- 2026-06-21 to 06-22 — Long build chat: Stages 2, 3, 4 locked; repo scaffolded (Next.js + Radix); tokens + Poppins set; Calendar Merger redefined and rebuilt; login redesign prompt written; chat ran out of tokens.
- 2026-06-22 — Stage 6 polish prompt for login/auth, incl. fixing the bar chart.
- 2026-06-22 — Auth put on hold; Core shell sidebar built and fixed (amber, logo swap, toggle). Expanded + collapsed work.
- 2026-07-14 — Moving to enterprise Claude account; status summary made; araMetrics-requirements.md exported. admin-portal-proposal.md found missing.
- 2026-08-29 to 08-30 — arm-core-fe landing hero SVG + `dataset.svg`; pushed `feature/aramerics-landing-page`.
- 2026-08-31 to 09-02 — arm-website run locally; cookie banner, hero, animated Projects/Vendors SVGs, responsive pass.
- 2026-09-03 — arm-website content pass + legal fixes, PR #1. Notification emails redesigned, PR #1 into `dev`.
- 2026-09-08 — Cloned 10 ARM repos into `~/araCreate/ARM`.
- 2026-10-03 — Project moved into viOS.
- 2026-10-05 — New `timer` app started in ARM with ARM UI components.

## 6. Key facts
- **People:** [[People/Vishnu]] — designer/owner. Engineering team gets the handoff (names not known).
- **Companies:** [[Companies/araCreate Group]] (not sure), [[Companies/Google]] (Google accounts / Calendar), [[Companies/Cal.com]] (visual reference only).
- **Tools:** [[Tools/Claude]] (project "arm-ui", stage chats), [[Tools/Claude Code]] (builds code), [[Tools/Claude Design]] (reference mockups), [[Tools/VS Code]], [[Tools/Next.js]], [[Tools/React]], [[Tools/Radix Themes]], [[Tools/Figma]] (Radix community file with brand tokens; Dev Mode MCP at localhost:3845 for code → canvas), [[Tools/GitHub]] (final handoff), [[Tools/Google Calendar]].
- **Not used:** [[Tools/Tailwind CSS]] — chosen against.
- **Design tokens:** accent amber #F9BF3B, gray from #555555 (Sand), Poppins 400/500/600, radius small, scaling 95%, panelBackground solid. Made at radix-ui.com/colors/custom.
- **Claude Code skills:** pbakaus/impeccable, leonxlnx/taste-skill.
- **Code:** Next.js App Router repo folder `arametrics`; `app/layout.tsx`, `app/theme-overrides.css`, `public/arm-logo.svg`, `public/arm-icon.svg`, `support/spec.md` (spec for Claude Code), mock data in `/lib/mock`. Runs on Vishnu's computer (exact path not known).
- **Links:** dev.arametrics.app (dev domain; also used as placeholder Terms/Privacy links).
- **GitHub repo:** not created yet (planned for Stage 8).
- **Admin roles:** admin — everything incl. role assignment, user delete, audit export. operator — view all, enable/disable users, revoke sessions.
- **Admin phases:** engineering proposal has 5 phases over about 15–16 weeks; Phase 1 = user management.
- **ARM repos** (`~/araCreate/ARM`): arm-make, arm-docs, arm-cli, arm-app-admin, arm-app-calendar, arm-session, arm-core-fe, arm-core-be, arm-service-notification, arm-util-clockify; plus arm-website and timer.
- Dev history: [[Projects/arm-ui/DEV-LOG]]
- **Where work was done:** Claude chats (thinking + prompts) → prompts pasted into Claude Code in VS Code.


## 7. Files and documents
- Claude project doc: "araMetrics Platform — UI/UX Requirements (v1)" — full prose brief.
- Claude project instructions: locked spec, 8-stage pipeline, working rules.
- `araMetrics-requirements.md` — downloaded copy of the brief (2026-07-14). Where it was saved: not sure.
- `admin-portal-proposal.md` — engineering proposal (9 Admin areas, nav, role matrix). Read on 2026-06-19 but **missing** from the project files now.
- `support/spec.md` — Stage 2 inventory + Stage 3 flows, used by Claude Code.
- `app/theme-overrides.css` — brand colour scales.
- `public/arm-logo.svg`, `public/arm-icon.svg` — logo and icon.
- Figma: Radix Themes community file with brand tokens (link not known).
- Stage 1–4 outputs live inside the chats (not saved as separate files — not sure).

## 8. Open questions and problems
- Self-serve sign-up conflicts with "provisioning only through Admin Portal". Needs a clear decision.
- Login screen: redesign written but not finished; did the polish prompt land? (not sure)
- Did the Calendar Merger rebuild pass review? (not sure)
- Operator UI is very thin — scope risk if operators later need more.
- Spec says audit export is admin-only, but Stage 1 said "no export in v1" — which is true? (not sure)
- Amber filled-box active nav breaks the "amber only as signal" rule — override was approved, but keep an eye on consistency.
- `admin-portal-proposal.md` is missing — need to find it.
- Did the move to the enterprise Claude account finish? (not sure)
- Where is the code folder on Vishnu's computer, and is it under git? (not sure)
- Who are the stakeholders and engineering team? (not known)
- Website legal pages describe an Indian entity (DPDP Act) but the company is German — needs GDPR rewrite and Impressum; Terms conflict with marketing.
- Website contact form saves locally only.
- Emails: "araMETRICS" spelling in artwork; "just hit reply" vs "No Reply" sender; old subjects.
- `pnpm` not on PATH on the Mac.
- Should the arm-website / timer work be split into its own project? (timer already has [[Projects/timer/SUMMARY]])

## 9. All chats in this project
- Index: INDEX (archived: Projects/arm-ui/chats/INDEX.md) (16 chats; imported from the personal account on 2026-10-06) · Claude Code sessions: see [[Projects/arm-ui/DEV-LOG]] session index
- Project doc: [[Projects/arm-ui/docs/araMetrics Platform — UI-UX Requirements (v1)]]
- AI tools for UI/UX design — 2026-06-19
- Where we stopped — 2026-06-21
- Stage 1 — go — 2026-06-21
- Full app reference UI prompt for Claude — 2026-06-21
- Role-based platform architecture with admin, operator, and user personas — 2026-06-22
- Last prompt clarification — 2026-06-22
- Resuming previous discussion — 2026-06-22
- Migrating Claude Projects to enterprise accounts — 2026-07-14
- Moving project to viOS (2 chats, viOS meta) — 2026-10-02
