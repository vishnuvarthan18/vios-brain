---
tags: project
---
# DEV LOG: Feedback widget (Claude Code sessions)

Built by Vishnu with Claude Code as developer (Vishnu as tech lead / product owner). Repo: `halle-feedback-widget` (GitHub `aracreate-group/halle-widget`, moved to `halle-app-widget`), branch `dev`. Client: B. Halle (Jakob). 52 sessions, 7 Sep – 6 Oct 2026.

## What was built (features)

**Widget (tester side, `v1.js` on the Webflow site)**
- Shows only for testers with a `?t=` token link; expired/revoked link shows a clear "ask for a new link" message.
- Launcher "Report a Bug" ("Bug" on phones, editable in Wording).
- Two modes: point at the problem (pick an element, tap-confirm on touch) or take a screenshot.
- Review screen: picture with red box around the picked element, marker pen (Clear only), comment box, Send, info tooltip "What else we send with this".
- Problems only (no "page was fine" button). Strings all editable from admin.
- React + Tailwind, shadow-DOM isolated, own green brand palette (5 greens), mobile/tablet sheet layouts.
- Screenshots: server-side renderer first (headless browser on our server, fed a light page snapshot), falls back to in-browser capture (modern-screenshot). Privacy stripping of form fields.

**Admin dashboard (Next.js + shadcn, `apps.b-halle.de/app`)**
- Login (email + password), lockout, revoke/disable, session signing.
- Overview dashboard (stat tiles, 30-day trend, status donut, template bars), one screen.
- Queue → classify each report: Bug or custom types (Classifications screen, colours, hide).
- Tracked items (bugs): tabs Processing / Fixed (To check) / Closed / Deleted; strict order Processing → Fixed → Closed, reopen.
- Classified page: one tab per custom type, Open → Done.
- Report popup (not a page): who sent it, when (CET), picture, numbered step line, Activity timeline (history + team comments).
- Pages (URL → template list, 99 Halle pages imported), Testers (name, copy link, revoke), Wording (string editor with revisions), Users (admin only), Settings (own profile/password).
- Filters (template, tester, mode, dates, search), CSV export, white collapsible sidebar with icons, pale blue page, mobile layout (cards, menu).
- Prefetching for faster clicks; HTTP/2 on server.

**Ops**
- `make demo` local stack, `make tunnel`, deploy runbook, systemd services (app + hybrid renderer), backups, 180-day screenshot retention, e2e tests (`make test-e2e`).
- New server (IONOS VPS L+) with two systemd-nspawn boxes: `webapp` and `jupyter`.

## Timeline (newest first)

- 2026-10-06 — Made Vishnu admin on prod (Users link was missing). Added 4 tag colours, then removed Navy (primary colour); "Phase-2" auto-recoloured to Teal. Both deployed.
- 2026-10-05 — Queue filter UI fixes (widths, own chevron for selects, date icon gap), bigger step circles, line fix, refresh on popup returns to list, copy cleanup, new end-to-end test plan (70 checks). Pushed and deployed. First deploy went to the OLD server by mistake (stale docs); then deployed to apps.b-halle.de. Fixed stale docs with warning banners. Added Kishor's SSH key to new server.
- 2026-10-05 — Bug flow redesign: strict Processing → Fixed → Closed, "Next: developer/stakeholder", numbered progress step line (local; Vishnu first said not to production).
- 2026-10-02 — Built Classifications (custom report types), Users admin + Settings page, UI declutter, 21 e2e tests, full conventions pass (snake_case rename, migration 0015). 8 commits pushed.
- 2026-10-02 — Root URL `apps.b-halle.de/` now redirects to login (deployed). viOS project created for Halle.
- 2026-09-30 — Admin overhaul: report popup, activity history + comments, Tracked tabs, CET times, All reports removed (Deleted tab, tester filter, CSV to Overview), 99 pages imported, prefetch, HTTP/2. Launch reset: test data cleared, 4 real logins (Vishnu, Jakob, Rahul, Shyam).
- 2026-09-30 — Red box accuracy fixed (box around element, keeps tapped element on phone, server places the box). Live.
- 2026-09-30 — Server capture speed-up: light page snapshot, 3 renders at once, warm pages, crash recovery, TCP tweak. Most pages ~0.5 s. Single loader in widget. Tablet Send button fix. Halle emblem favicon.
- 2026-09-30 — Web app moved to new server, now at `apps.b-halle.de` (Webflow tag changed). Jupyter/pgAdmin/product API moved; DNS for ttqvgsran switched. Old server services disabled.
- 2026-09-28 — Client could not upgrade old VPS; bought new one. Full backup of old server to Mac (`~/araCreate/HLE/server/`). Migration plan; Jupyter box built.
- 2026-09-23 — Admin mobile responsive, flat nav, smooth collapse. Widget mobile/tablet layout, "Bug" label, info panel fix. Deployed; Bug button missing on live because widget built without WIDGET_API_ORIGIN — fixed.
- 2026-09-22 — Widget UI rework (green palette, icons, review dialog 70%, loader, overlay, tooltip). Admin UI rework (logo, pale blue, white sidebar, dashboard). Renderer fixes (hero images, speed). Highlight box drawn on raster. Letterbox fix. Deployed (Vishnu ran server steps).
- 2026-09-21 — Admin migrated to shadcn (11 screens), widget to React. Screenshot speed fix live (fonts off). Hero blank / pseudo-element / scroll capture fixes. Hybrid server renderer built and turned on in production.
- 2026-09-10 — Capture timeout fix; lazy-image bug (11.6 s → 1.6 s); server-side capture A/B test (decided keep browser method then). Deployed to feedback.arametrics.app.
- 2026-09-09 — Brand design system applied. Conventions commit rewrite. Deploy prep (runbook, systemd). Capture fixes on real pages, CSV token strip, retention, backups. Admin v3: tester revoke, no assignments, expired-link message, sidebar.
- 2026-09-08 — M3 fixes, M4 issues, M5 admin, M6a storage, M6b capture + privacy. Overnight v2 run: M7 DB simplify, M8 widget v2, M9 picture flow, M10 admin v2. `make demo`.
- 2026-09-07 — M0 scaffold, M1 public API, M2 widget, M3 read-only app. Committed M0–M3.

## Decisions

- #decision 2026-09-07 — Problems only: no "this page was fine" button; main screen is a report grid, never "coverage".
- #decision 2026-09-07 — English only, five fixed option texts, strings seeded and not reworded.
- #decision 2026-09-08 — Screenshots on local disk behind a storage interface, not S3.
- #decision 2026-09-08 — v2: roles and issues tables dropped; status column on reports; consent step removed.
- #decision 2026-09-09 — Removing a tester = revoke (link dies, reports kept); no assignments, one link works on all pages.
- #decision 2026-09-09 — Commits follow aracreate-conventions: no Co-Authored-By, no real names, no articles in subjects.
- #decision 2026-09-10 — Keep in-browser capture method; install nothing on prod (later reversed on 21 Sep).
- #decision 2026-09-21 — Override agent rules: widget moves to React + Tailwind (bigger bundle accepted).
- #decision 2026-09-21 — EMBED_WEB_FONTS = false (fast path, under 200 ms captures).
- #decision 2026-09-22 — Widget uses only the 5 brand greens; runtime accent colour from DB removed.
- #decision 2026-09-22 — Highlight box drawn on the finished picture, not on the DOM clone.
- #decision 2026-09-28 — New server uses systemd-nspawn boxes (webapp, jupyter), no Docker/Podman; Jupyter moved first.
- #decision 2026-09-30 — Feedback app lives at `apps.b-halle.de` (not feedback.arametrics.app).
- #decision 2026-09-30 — Screenshots taken on our server (like Marker.io/Usersnap); no close-up picture.
- #decision 2026-09-30 — Report detail opens as popup; small edits/confirms as popups; times in CET.
- #decision 2026-09-30 — All reports page removed; Deleted becomes a Tracked tab.
- #decision 2026-10-02 — One user type plus Admin flag; Users screen admin only.
- #decision 2026-10-02 — Custom classifications: Bug keeps Processing → Fixed → Closed; custom types Open → Done; types hidden, never deleted.
- #decision 2026-10-02 — Keep old server until 14 clean days and money-back window end.
- #decision 2026-10-05 — Bug steps strict: Closed means a stakeholder checked the fix.
- #decision 2026-10-05 — Pages screen kept (corrects URL → page-type guesses).
- #decision 2026-10-06 — Navy removed from tag colours (it is the primary UI colour).

## Current state (as of 2026-10-06)

- Live at `https://apps.b-halle.de` (new server, `webapp` box). Widget on halle-dev.webflow.io loads `apps.b-halle.de/v1.js`.
- Latest deploy: tag colours change (Navy removed). Code on `origin/dev`, pushed.
- Production has 4 dashboard logins; Vishnu is admin. 99 pages registered.
- Server renderer takes almost all screenshots, ~0.5 s on most pages.
- Jupyter, pgAdmin, product API on the `jupyter` box (ttqvgsran.b-halle.de). Old server kept as backup, not to be used.
- Local: all local servers stopped.

## Open bugs / next steps

- Revoke the GitHub token found in plain text in the server git remote (both servers); switch to a deploy key.
- Re-stop the feedback service on the OLD server (it was started by mistake on 5 Oct).
- Decide if Jakob/Rahul/Shyam need admin.
- Vishnu asked for Classifications to be admin-only — not done yet (currently any login).
- Update local git remote to `halle-app-widget`.
- Test in Safari / iPad; check login open-redirect.
- Old server: after 14 clean days (~14 Oct) check logs, take final archive, then Jakob cancels old contract after money-back window.
- Fix two broken links on Best Form Lenses page in Webflow (connector has no access to Halle site).
- Node 20 `.mts` scripts issue on old server is moot; new box runs Node 22.
- Always check viOS STATE before touching any server (stale repo docs caused the wrong-server deploy).

## Session index

- araCreate-HLE-testing-widget__2026-09-07_eeeabb0f (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-07_eeeabb0f.md) — 2026-09-07 — M0 scaffold: DB, migrations, append-only reports guard, Makefile.
- araCreate-HLE-testing-widget__2026-09-07_35aa90df (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-07_35aa90df.md) — 2026-09-07 — M1 public API: config and reports endpoints, grouping, rate limit.
- araCreate-HLE-testing-widget__2026-09-07_c59814c5 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-07_c59814c5.md) — 2026-09-07 — M2 widget: states, shuffled options, tap-confirm.
- araCreate-HLE-testing-widget__2026-09-07_bc872be0 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-07_bc872be0.md) — 2026-09-07 — M3 read-only app: auth, report grid, list, CSV.
- araCreate-HLE-testing-widget__2026-09-07_446a7276 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-07_446a7276.md) — 2026-09-07 — Overnight-run prep; committed M0–M3.
- araCreate-HLE-testing-widget__2026-09-08_db1e8676 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-08_db1e8676.md) — 2026-09-08 — M3 auth fixes and M4 issue workflow.
- araCreate-HLE-testing-widget__2026-09-08_6194b9bc (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-08_6194b9bc.md) — 2026-09-08 — M5 admin: string editor, pages, testers.
- araCreate-HLE-testing-widget__2026-09-08_29a15db1 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-08_29a15db1.md) — 2026-09-08 — M6a storage and signed uploads.
- araCreate-HLE-testing-widget__2026-09-08_1cf3f0c5 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-08_1cf3f0c5.md) — 2026-09-08 — M6a fixes and M6b capture with privacy stripping.
- araCreate-HLE-testing-widget__2026-09-08_2ee9dfb2 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-08_2ee9dfb2.md) — 2026-09-08 — Split M6 commits, build-plan corrections.
- araCreate-HLE-testing-widget__2026-09-08_1b1024c2 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-08_1b1024c2.md) — 2026-09-08 — `make demo`, widget served from app, tunnel setup.
- araCreate-HLE-testing-widget__2026-09-08_32aac108 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-08_32aac108.md) — 2026-09-08 — Unattended v2 run: M7–M10.
- araCreate-HLE-testing-widget__2026-09-08_8b3715b6 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-08_8b3715b6.md) — 2026-09-08 — Verification pass on v2; found two bugs.
- araCreate-HLE-testing-widget__2026-09-08_7e111804 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-08_7e111804.md) — 2026-09-08 — Fixed `make demo` fixture and viewer zoom.
- araCreate-HLE-testing-widget__2026-09-09_59e128f1 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-09_59e128f1.md) — 2026-09-09 — Brand design system on admin and widget.
- araCreate-HLE-testing-widget__2026-09-09_53355724 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-09_53355724.md) — 2026-09-09 — Conventions pass, commit message rewrite.
- araCreate-HLE-testing-widget__2026-09-09_2456cd15 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-09_2456cd15.md) — 2026-09-09 — Conventions audit (rewrite already done).
- araCreate-HLE-testing-widget__2026-09-09_3e348634 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-09_3e348634.md) — 2026-09-09 — Production deploy config and runbook.
- araCreate-HLE-testing-widget__2026-09-09_690ff28a (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-09_690ff28a.md) — 2026-09-09 — Fixed user scripts after roles removal.
- araCreate-HLE-testing-widget__2026-09-09_4b59b0fe (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-09_4b59b0fe.md) — 2026-09-09 — Capture fixes, retention, backups; admin v3 rebuild.
- araCreate-HLE-testing-widget__2026-09-10_81159cb8 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-10_81159cb8.md) — 2026-09-10 — Capture hand-off timeout fix.
- araCreate-HLE-testing-widget__2026-09-10_e9d000a9 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-10_e9d000a9.md) — 2026-09-10 — Screenshot speed: lazy images bug.
- araCreate-HLE-testing-widget__2026-09-10_e8b521fc (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-10_e8b521fc.md) — 2026-09-10 — A/B test server vs browser capture.
- araCreate-HLE-testing-widget__2026-09-10_3a06758e (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-10_3a06758e.md) — 2026-09-10 — Real-conditions test stopped; deployed live.
- araCreate-HLE-testing-widget__2026-09-21_bbee31ee (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-21_bbee31ee.md) — 2026-09-21 — shadcn admin, React widget, capture fixes, deploys.
- araCreate-HLE-testing-widget__2026-09-21_4c6c4eaa (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-21_4c6c4eaa.md) — 2026-09-21 — Capture investigations; hybrid server renderer live.
- araCreate-HLE-testing-widget__2026-09-22_337ea8ad (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-22_337ea8ad.md) — 2026-09-22 — Box timing, speed, marker fixes, capture_method; deploy.
- araCreate-HLE-testing-widget__2026-09-22_ea3e1952 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-22_ea3e1952.md) — 2026-09-22 — Letterbox box-offset fix.
- araCreate-HLE-testing-widget__2026-09-22_f2b37da0 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-22_f2b37da0.md) — 2026-09-22 — Local test environment setup.
- araCreate-HLE-testing-widget__2026-09-22_4445288e (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-22_4445288e.md) — 2026-09-22 — Highlight box drawn on raster; deploy.
- araCreate-HLE-testing-widget__2026-09-22_ebdfb0b7 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-22_ebdfb0b7.md) — 2026-09-22 — Deployed two renderer fixes.
- araCreate-HLE-testing-widget__2026-09-22_8fcf2c46 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-22_8fcf2c46.md) — 2026-09-22 — Widget UI rework in green; deploy.
- araCreate-HLE-testing-widget__2026-09-22_d566d728 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-22_d566d728.md) — 2026-09-22 — Admin UI rework; guided manual deploy 23 Sep.
- araCreate-HLE-testing-widget__2026-09-23_e926b4cf (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-23_e926b4cf.md) — 2026-09-23 — Admin mobile responsiveness.
- araCreate-HLE-testing-widget__2026-09-23_3c7af8aa (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-23_3c7af8aa.md) — 2026-09-23 — Widget mobile/tablet; deploy; API origin fix.
- araCreate-HLE-testing-widget__2026-09-28_005e5468 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-28_005e5468.md) — 2026-09-28 — New server, full backup, migration plan, Jupyter box move.
- araCreate-HLE-testing-widget__2026-09-30_133124dc (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-30_133124dc.md) — 2026-09-30 — Web app moved to apps.b-halle.de.
- araCreate-HLE-testing-widget__2026-09-30_99772fcb (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-30_99772fcb.md) — 2026-09-30 — Server capture under 1 s; old server shut down; favicon.
- araCreate-HLE-testing-widget__2026-09-30_68c8e7ea (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-30_68c8e7ea.md) — 2026-09-30 — Red box accuracy fix.
- araCreate-HLE-testing-widget__2026-09-30_f9745f33 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-09-30_f9745f33.md) — 2026-09-30 — Admin popup, history, tabs, pages import, launch reset.
- araCreate-HLE-testing-widget__2026-10-02_0e09829a (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-10-02_0e09829a.md) — 2026-10-02 — Root URL to login; viOS project link.
- araCreate-HLE-testing-widget__2026-10-02_3d95566b (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-10-02_3d95566b.md) — 2026-10-02 — Title stub only.
- araCreate-HLE-testing-widget__2026-10-02_56e32277 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-10-02_56e32277.md) — 2026-10-02 — Title stub only.
- araCreate-HLE-testing-widget__2026-10-02_98e1c427 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-10-02_98e1c427.md) — 2026-10-02 — Title stub only.
- araCreate-HLE-testing-widget__2026-10-02_04bafd0b (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-10-02_04bafd0b.md) — 2026-10-02 — Classifications, Users/Settings, e2e, snake_case; bug flow step line (5 Oct).
- araCreate-HLE-testing-widget-halle-feedback-widget-src-web__2026-10-02_b9a10023 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget-halle-feedback-widget-src-web__2026-10-02_b9a10023.md) — 2026-10-02 — Eval stub (classification spec).
- araCreate-HLE-testing-widget-halle-feedback-widget-src-web__2026-10-02_fd36ab86 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget-halle-feedback-widget-src-web__2026-10-02_fd36ab86.md) — 2026-10-02 — Eval stub (UI cleanup).
- araCreate-HLE-testing-widget__2026-10-05_a10d30d3 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-10-05_a10d30d3.md) — 2026-10-05 — Title stub only.
- araCreate-HLE-testing-widget__2026-10-05_ab7137fb (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-10-05_ab7137fb.md) — 2026-10-05 — Title stub only.
- araCreate-HLE-testing-widget-halle-feedback-widget__2026-10-05_837081a1 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget-halle-feedback-widget__2026-10-05_837081a1.md) — 2026-10-05 — Eval stub (select arrow fix).
- araCreate-HLE-testing-widget__2026-10-05_76dfd452 (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-10-05_76dfd452.md) — 2026-10-05 — Server access question (pointed at old docs).
- araCreate-HLE-testing-widget__2026-10-05_aafacd4d (archived: Projects/feedback-widget/claude-code/araCreate-HLE-testing-widget__2026-10-05_aafacd4d.md) — 2026-10-05 — Filter UI fixes, deploy (wrong server first), tag colours (6 Oct).
