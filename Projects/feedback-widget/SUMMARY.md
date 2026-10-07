---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: Feedback widget for B. Halle

## 1. What this project is
- **Goal:** A "Report a Bug" button on the B. Halle [[Tools/Webflow]] site. An invited tester clicks it, points at a problem (or takes a screenshot), the widget takes a picture with a red box on that element, the tester writes a comment and sends it. Reports land in a private admin web app (queue, tracked bugs, picture viewer, CSV export).
- **Who it is for / client:** [[Companies/HALLE]] (B. Halle Nachfl. GmbH, German optics maker). Client contact: [[People/Jakob Silbermann]] (Jakob Silbermann). Proposal was "Prepared for Bernhard Halle". Built by [[Companies/araCreate Group]] as an araCreate product.
- **Why it exists:** The client needs testers (internal B. Halle staff) to report UI/usability problems on the new beta site before launch. araCreate decided to build its own tool (not buy Usersnap/Ybug/Marker.io/BugHerd) so it can be reused and maybe sold to other clients later.

## 2. Status now (as of 2026-10-06)
- **Live at https://apps.b-halle.de** on the new IONOS server (`webapp` systemd-nspawn box) since 30 Sep 2026. Latest deploy `d9b6d11` on 6 Oct (tag colours, Navy removed). Root redirects to login.
  - (earlier: live at feedback.arametrics.app on the old IONOS VPS 212.227.213.174 — now retired. The old VPS could not be upgraded to 8 GB, so the client bought a bigger VPS on 28 Sep.)
- Webflow staging site halle-dev.webflow.io loads the widget from `apps.b-halle.de/v1.js` (only shows with a tester link `?t=` token).
- New server has two boxes: `webapp` (feedback app, screenshot renderer, its DB) and `jupyter` (JupyterHub, pgAdmin, product API, at ttqvgsran.b-halle.de).
- Screenshots are taken on our server (headless browser fed a light page snapshot), ~0.5 s on most pages; in-browser capture only as fallback. Red box sits correctly around the picked element, also on phone and far down a page.
- Admin (React + shadcn/ui, Halle brand tokens): report popup with Activity timeline (history + team comments); Tracked tabs Processing / Fixed (To check) / Closed / Deleted; times in CET; All reports page removed; Classifications (custom report types, Open → Done) + Classified page; Users screen (admin only); Settings page. Admin + widget are mobile/tablet responsive.
- Bug flow is strict: Processing → Fixed → Closed (Closed = stakeholder checked), with a numbered step line in the popup.
- Tag colours: 9 options (Navy removed; "Phase-2" is Teal). 99 Halle pages (EN + DE) registered in Pages.
- Production logins: Vishnu (admin), Jakob, Rahul, Shyam. Test data cleared at launch (30 Sep).
- Code on GitHub `dev` (repo renamed to `halle-app-widget`); follows aracreate-conventions incl. snake_case; 21 e2e tests + 70-check test plan.
- Old server 212.227.213.174: kept as backup only, but its feedback service was restarted by mistake on 5 Oct — must be stopped + disabled again.

## 3. Next steps
1. Revoke the GitHub token found in plain text in the git remote on both servers; switch to a deploy key.
2. Re-stop and disable the feedback service on the old server.
3. ~14 Oct 2026: check old-server logs, take final archive; then Jakob cancels the old IONOS contract (after the money-back window).
4. Decide if Jakob / Rahul / Shyam get admin; make Classifications admin-only (asked by Vishnu, not done).
5. Test in Safari / iPad; check the login `next` open redirect.
6. Update local git remote to `aracreate-group/halle-app-widget`.
7. Fix two broken links on the Best Form Lenses page in Webflow.
8. Older items, status not sure: "Our History" slider shows wrong slide in picture; marker-pen fixes; 16-screen UI review; one real report with `capture_method = server` checked.
9. Security: rotate the root password that was shared in chat; remove `~/.ssh/halle_agent` key at end of dev.

## 4. Decisions
- 2026-08-24 — Build custom, not buy — reusable araCreate product for future clients. #decision
- 2026-08-24 — Bill client 8–12 h; araCreate absorbs ~40 h as internal investment. #decision
- 2026-08-24 — Random tester tokens; EU hosting; shared admin password (later replaced by real logins). #decision
- 2026-08-25 — English only for now; templates Home, Category, Product, Contact, 404 (49 pages). #decision
- 2026-09-01 — Widget only for invited testers; tester token in URL, nothing stored on device (TDDDG §25, no cookie banner). #decision
- 2026-09-01 — Build is done by an AI coding agent; Claude = PM / tech lead / writes task docs only. #decision
- 2026-09-02 — Drop "page looked fine" button and reminder emails. #decision
- 2026-09-07 — Stack: Next.js + Drizzle + Postgres, zero-dependency TypeScript widget; copyright "B. Halle" (client owns code). #decision
- 2026-09-07 — German/language switching, keyboard path, read-aloud out of scope. #decision
- 2026-09-08 — v2 flow: "Report a Bug" with "Point at the problem" / "Screenshot", marker pen, comment required, panel docked in corner. #decision
- 2026-09-08 — Admin v2: one login, no roles; report -> Bug or Delete; tracked item -> Fixed or Closed. #decision
- 2026-09-08 — No consent screen; picture always attempted. #decision
- 2026-09-09 — Strip tester token from CSV; picture retention 180 days. #decision
- 2026-09-09 — No DPA needed (testers are internal B. Halle staff only). #decision
- 2026-09-09 — Follow araCreate conventions; push to `dev` only, `main` only for production. #decision
- 2026-09-09 — Host on existing IONOS VPS (permanent home), own user, own DB; temporary domain feedback.arametrics.app; never touch other projects on the server. #decision
- 2026-09-09 — Testers: name only, no email, no page assignments, remove = revoke. #decision
- 2026-09-10 — First server capture test lost (2.4x slower) — kept in-browser method then. #decision
- 2026-09-21 — Admin and widget move to React + shadcn/ui with brand tokens. #decision
- 2026-09-21 — Email tester when bug fixed: yes; reopen Fixed/Closed: yes; duplicate warning, bulk actions: no. #decision
- 2026-09-21 — Use existing server, no new server; "don't delete anything from the server" (standing rule). Server-first hybrid capture built and deployed. #decision
- 2026-09-22 — Draw red box onto finished picture with canvas (not in page DOM). #decision
- 2026-09-22 — Carousel/slider issue is a separate bug, fix later. #decision
- 2026-09-23 — Do not buy a new server; upgrade existing IONOS VPS to 8 GB. (Reversed 28 Sep: old VPS could not be upgraded; client bought a bigger VPS.) #decision
- 2026-09-28 — New server uses systemd-nspawn boxes (webapp, jupyter), no Docker/Podman; Jupyter moved first. #decision
- 2026-09-30 — Feedback app lives at `apps.b-halle.de` (not feedback.arametrics.app). Screenshots taken on our server; no close-up picture. #decision
- 2026-09-30 — Report detail opens as popup; times in CET; All reports page removed; Deleted becomes a Tracked tab. #decision
- 2026-10-02 — One user type plus Admin flag; Users screen admin only. Custom classifications: Bug keeps Processing → Fixed → Closed; custom types Open → Done; types hidden, never deleted. #decision
- 2026-10-02 — Keep old server until 14 clean days and money-back window end. #decision
- 2026-10-05 — Bug steps strict: Closed means a stakeholder checked the fix. Pages screen kept. #decision
- 2026-10-05 — Rule: check viOS STATE before touching any server (stale repo docs caused a wrong-server deploy). #decision
- 2026-10-06 — Navy removed from tag colours (it is the primary UI colour). #decision

## 5. Timeline
- 2026-08-24 — v1 proposal reviewed; final proposal + client docs written.
- 2026-08-25 — Jakob approved proposal (9 comments); asked for screenshots and spec; per-page questions written.
- 2026-09-01 — Plain-language research; new widget content (point + 5 answers); build-vs-buy = build.
- 2026-09-02 — Scope decisions; tech stack explained; client note "why we changed the form".
- 2026-09-07 — Dev agent builds M0–M6 (DB, API, widget, admin, screenshots); v2 direction from Vishnu.
- 2026-09-08 — v2 spec + overnight run (M7–M10); brand design system applied.
- 2026-09-09 — Conventions fixed, pushed to GitHub `dev`; deployed to IONOS VPS; HTTPS on feedback.arametrics.app; first end-to-end report worked.
- 2026-09-10 — Screenshot speed fix measured; server capture A-B test (server lost); live "Loading"/blank bug.
- 2026-09-11 — Vishnu reports blank screenshots and 8–10 s wait on live.
- 2026-09-21 — React/shadcn migration; capture under 1 s; hero and scroll-blank fixes live; hybrid server capture deployed.
- 2026-09-22 — Box letterbox bug; canvas box fix live (`036883f`); hero-blank fix (`82a471b`) and speed fix (`390662f`) live.
- 2026-09-23 — Slider bug investigated; server check; IONOS upgrade message for Jakob. Admin + widget made mobile/tablet responsive.
- 2026-09-28 — Old VPS could not be upgraded; client bought bigger VPS. Full old-server backup to Mac; migration plan; Jupyter box built.
- 2026-09-30 — Web app moved to new server at apps.b-halle.de; Jupyter moved, DNS switched; old services disabled. Admin overhaul; red box accuracy fix; capture ~0.5 s. Launch reset with 4 real logins.
- 2026-10-02 — Classifications, Users admin, Settings, 21 e2e tests, snake_case conventions pass; root redirects to login.
- 2026-10-05 — Strict bug flow + step line; queue/UI fixes; 70-check test plan. Deployed first to old server by mistake, then apps.b-halle.de. Kishor given SSH key access to new server.
- 2026-10-06 — Vishnu made prod admin; tag colours changed; deployed `d9b6d11`.

## 6. Key facts
- **People:** [[People/Vishnu]] — owner, araCreate; [[People/Jakob Silbermann]] — client contact (Jakob Silbermann, B. Halle); Bernhard Halle — client (proposal addressed to him); [[People/Shyam]] — heard the "isolated instance" pitch (role not sure).
- **Companies:** [[Companies/HALLE]], [[Companies/araCreate Group]], IONOS (server host).
- **Tools:** [[Tools/Webflow]], [[Tools/Next.js]], [[Tools/React]], [[Tools/shadcn-ui]], [[Tools/Tailwind CSS]], [[Tools/Drizzle ORM]], [[Tools/PostgreSQL]], [[Tools/Vitest]], [[Tools/GitHub]], [[Tools/systemd]], [[Tools/Claude Code]], [[Tools/Figma]], [[Tools/Slack]], [[Tools/WhatsApp]], Playwright, modern-screenshot, Apache, Let's Encrypt.
- **Links / repos / servers / file paths:**
  - Live app: https://apps.b-halle.de (widget `/v1.js`). (earlier: feedback.arametrics.app, retired.)
  - Test site: https://halle-dev.webflow.io (needs tester link token, secret, not saved).
  - Repo: github.com/aracreate-group/halle-widget, moved to aracreate-group/halle-app-widget; branch `dev`.
  - Mac repo: `~/araCreate/HLE/testing_widget/halle-feedback-widget`.
  - Server (now): new IONOS server 217.160.93.75, boxes `webapp` (feedback app + renderer + DB, Node 22) and `jupyter` (JupyterHub, pgAdmin, product API at ttqvgsran.b-halle.de).
  - Old server (retired, backup only until ~14 Oct): IONOS VPS 212.227.213.174 (Debian 12, 3.8 GB RAM). App `/opt/halle-feedback/app`, data `/var/lib/halle-feedback/`, services `halle-feedback` and `halle-feedback-hybrid-render` (port 4600).
  - Conventions: github.com/aracreate-group/aracreate-conventions.
  - Passwords, tokens, keys: (secret, not saved).
- **Related:** [[Projects/halle-web/SUMMARY]]
- Dev history: [[Projects/feedback-widget/DEV-LOG]]

## 7. Files and documents
- `claude/SESSION-HANDOVER.md` and `claude/SESSION-HANDOVER-22-sept-burnin-box-fix.md` — main handover (Claude project).
- `claude/BUILD-SPEC.md`, `claude/scope-decisions.md`, `claude/widget-content-FINAL.md` — early spec and decisions (Claude project).
- `claude/server-deployment-plan.md`, `claude/https-domain-live-record.md` — server + HTTPS record (Claude project).
- `claude/decision-use-existing-server-not-new-one.md`, `claude/decision-admin-shadcn-rebuild.md` (Claude project).
- `claude/research-capture-speed-measured-22-sept.md`, `claude/fix-hybrid-renderer-hero-blank-22-sept.md`, `claude/live-evidence-blank-hero-22-sept.md` (Claude project).
- `claude/client-note-why-we-changed-the-form.md` — note for Jakob (Claude project).
- `docs/agent-rules.md`, `docs/widget-v2-spec.md`, `docs/admin-v2-spec.md`, `docs/admin-v3-rebuild-plan.md`, `docs/halle-design-system-draft.md` — repo docs.
- `deploy/RUNBOOK.md` — server install/deploy steps (repo).
- Proposal docs (proposal_final, Page_Feedback_Widget_Proposal_3pg.docx, Page_Feedback_Widget_Simple.docx) — made 2026-08-24 in Claude sandbox.
- Proposal / content Google Doc "beta-feedback-widget" shared with client (link in 2026-08-24 chat).

## 8. Open questions and problems
- Slider ("Our History") wrong slide in picture — fixed? (not recorded since 23 Sep)
- Renderer `TasksMax=64` / idle crash — still relevant on the new server? (old server had only ~1 GB free)
- Login `next` open redirect; Safari/iPad not tested.
- Who else gets admin; Classifications admin-only not done.
- `claude/` project docs are not in git — deploy agents cannot read them.
- Console-error PII filtering declined; GDPR risk flagged (revisit?).
- Notify team on new report — ask client first.
- Exposed GitHub token in git remotes on both servers and root password shared in chat — rotate.
- IONOS price: with or without VAT not clear.
- Dashboard moved to the client's domain (apps.b-halle.de) on 30 Sep — answered.
- Scratch `_tmp-*` files and untracked test files left in `testing_widget`.

## 9. All chats in this project
- Index: INDEX (archived: Projects/feedback-widget/chats/INDEX.md) · Claude Code sessions: see [[Projects/feedback-widget/DEV-LOG]] session index
- Text explanation (archived: Projects/feedback-widget/chats/2026-08-24 Text explanation.md) — 2026-08-24
- Client message (archived: Projects/feedback-widget/chats/2026-08-25 Client message.md) — 2026-08-25
- Content clarity research (archived: Projects/feedback-widget/chats/2026-09-01 Content clarity research.md) — 2026-09-01
- Planning and readiness check (archived: Projects/feedback-widget/chats/2026-09-02 Planning and readiness check.md) — 2026-09-02
- Project tech stack overview (archived: Projects/feedback-widget/chats/2026-09-02 Project tech stack overview.md) — 2026-09-02
- AI agent development setup (archived: Projects/feedback-widget/chats/2026-09-07 AI agent development setup.md) — 2026-09-07
- Progress and pending items (archived: Projects/feedback-widget/chats/2026-09-07 Progress and pending items.md) — 2026-09-07
- Dev agent development (archived: Projects/feedback-widget/chats/2026-09-08 Dev agent development.md) — 2026-09-08
- Discussion starter (archived: Projects/feedback-widget/chats/2026-09-08 Discussion starter.md) — 2026-09-08
- Overnight run completion (archived: Projects/feedback-widget/chats/2026-09-08 Overnight run completion.md) — 2026-09-08
- Feedback widget HTTPS setup (archived: Projects/feedback-widget/chats/2026-09-09 Feedback widget HTTPS setup.md) — 2026-09-09
- Feedback widget end-to-end testing (archived: Projects/feedback-widget/chats/2026-09-09 Feedback widget end-to-end testing.md) — 2026-09-09
- Pending items (NGf2J6) (archived: Projects/feedback-widget/chats/2026-09-09 Pending items (NGf2J6).md) — 2026-09-09
- Pending items (archived: Projects/feedback-widget/chats/2026-09-09 Pending items.md) — 2026-09-09
- Screenshot speed optimization (archived: Projects/feedback-widget/chats/2026-09-10 Screenshot speed optimization.md) — 2026-09-10
- Server-side capture A-B test (archived: Projects/feedback-widget/chats/2026-09-10 Server-side capture A-B test.md) — 2026-09-10
- Returning session (archived: Projects/feedback-widget/chats/2026-09-11 Returning session.md) — 2026-09-11
- Capture engine session record (archived: Projects/feedback-widget/chats/2026-09-21 Capture engine session record.md) — 2026-09-21
- Pending items (archived: Projects/feedback-widget/chats/2026-09-21 Pending items.md) — 2026-09-21
- Project next steps (archived: Projects/feedback-widget/chats/2026-09-21 Project next steps.md) — 2026-09-21
- Continuation (archived: Projects/feedback-widget/chats/2026-09-22 Continuation.md) — 2026-09-22
- Engine fix alignment (archived: Projects/feedback-widget/chats/2026-09-22 Engine fix alignment.md) — 2026-09-22
