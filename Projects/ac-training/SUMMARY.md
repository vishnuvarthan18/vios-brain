---
tags: project
status: done
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: araCreate Academy — VCET bootcamp dashboard

## 1. What this project is
- **Goal:** Run a 9-day "Basic Electronics" bootcamp for year-2 ECE and EEE students at [[Companies/Velalar College of Engineering and Technology]] (VCET), with one web dashboard for profiles, teams, hand-ins, quizzes, attendance, points and reports.
- **Who it is for / client:** VCET (college client), run by [[Companies/araCreate Group]] under the araCreate Academy name. 206 students / 52 teams after test data was removed (EEE 14 teams / 55 students, ECE 38 teams / 151 students) — earlier count 209 students / 53 teams incl. the staff test team (to confirm). Two venues (EEE, ECE).
- **Why it exists:** Vishnu wanted one place for each student's profile and all activity, and a live points board, instead of Google Sheets. Built with AI agents, no hired developer.

## 2. Status now (as of 2026-10-06)
- **Done.** The bootcamp ran 18–26 Sep 2026 (Day 9 = 26 Sep). Last work: 28 Sep (client pack, late hand-ins, certificate check). Only clean-up items are left.
- Live app still up: https://vcet.aracreate.academy — React "v3" front end (old UI at /old/), automatic scoring, team pages, completion matrix, marking + evaluation screens, CSV/XLSX exports, read-only Viewer role for college staff.
- Stack: Node 22 + Express + Postgres 16 + Caddy on Hetzner (to confirm — an older note said Node 20 / Postgres 17); team points by DB trigger.
- Auto points: quiz 1 per correct, hand-in 5 on time (late = 0), attendance 2/day, survey 1/day; admin +/− adjustments. Since 24 Sep: rank by total points, attendance + survey scaled to a team of 4 (Option B).
- **Final results (26 Sep) (to confirm):**
  - Final scores loaded from `final-team-scores.pdf` (top ECE-T28 Link Force 220.0, lowest 104.8) + project evaluation marks (out of 50). Independent marks check found 0 differences.
  - Top four then set by hand: ECE-T36-SWITCHSQUAD and EEE-T02-COREX joint 1st, ECE-T34-CODETEAM 3rd, EEE-T09-SPARKX 4th.
  - Two notes name a different winner (Link Force by score vs SWITCHSQUAD/COREX by hand-set rank) — to confirm which is the official result.
- Project marks are hidden from students and most staff screens (only Evaluation screen and admin exports). Leaderboard closed to students; board shows only the final score. Tied teams share a rank.
- Certificates: 206 made from araCreate's own SVG artwork (serials ARA-2026-NNNN, guest serials G0NN). Open to students; 38 of 206 downloaded by 28 Sep (from file open times; app does not log downloads).
- Client pack ready (28 Sep): "Bootcamp Final Projects - 2026" folder (final project photo for 50 of 52 teams) + team details xlsx. Missing photos: Live Wire (ECE-T02), Spark Shift (EEE-T13).
- Training report: one note says delivered to VCET on 28 Sep 2026, another says the bootcamp report for the client is still to do (to confirm).
- 247 student GitHub repos reviewed; common gaps: no circuit diagram, weak commit messages.
- All CVs and hand-in files are in Google Shared Drive (team folders); server uploads folder is empty. Local backup on Mac (uploads-backup-21sep).
- Not done / open: secret rotation (staff password, Google key); evaluation branch not yet merged into `dev`; admin exports still include project marks; quiz/survey questions were mostly never filled; EEE pre-assessment never opened; server still running.

## 3. Next steps
1. Merge the evaluation branch (shared-rank change, commit `c867933`) into `dev` before any new deploy.
2. Confirm the official winner and whether the client report was delivered.
3. Rotate the staff/admin password and the Google service-account key (both were exposed in chats; Drive is now the only copy of files).
4. Collect results: export CSVs, team photo form responses, feedback form responses.
5. Decide what to do with the server and data now the bootcamp is over (keep, archive, shut down).
6. Clean up: delete ECE-T99-TESTTEAM Drive folder, test accounts, stray share on 2-backend folder.
7. Commit and merge all code on `dev` into `main`; regenerate schema.sql (15 vs 26 live tables).
8. Fix deploy.md rsync trap (must keep `--exclude uploads`) before any reuse.
9. Write a lessons-learned note for the next college bootcamp.

## 4. Decisions
- 2026-09-16 — Discord (free cloud) for chat only; all records on own server; no Discord bot. — free, simple. #decision
- 2026-09-16 — No Google Sheets; projects per team; student profile with resume + goal in form fields. — one place for progress. #decision
- 2026-09-16 — Dates locked: onboarding 17 Sep, Day 1–9 = 18–26 Sep 2026. #decision
- 2026-09-16 — Student login = email + one shared code; staff = email + shared password. — no OTP needed. #decision
- 2026-09-16 — Google Drive links instead of uploads; daily team quiz in the app. #decision
- 2026-09-16 — Team code format DEPT-TNN-TEAMNAME; ECE sections A/B/C used as tracks. #decision
- 2026-09-16 — Host on Hetzner VPS behind Caddy HTTPS; follow aracreate-conventions; araCreate design system, light theme only. #decision
- 2026-09-16 — Never re-run load-eee.sql / load-ece.sql; never change start_date on server. — would wipe/shift live data. #decision
- 2026-09-17 — v2: progress bar = profile completion; quiz per student, 30 s per question; tasks replace projects; everything opens per venue by admin hand. #decision
- 2026-09-17 — Build v2 overnight with two parallel Claude Code agents (Lane A / Lane B). #decision
- 2026-09-18 — Deploy v2 and Tinkercad feature before 9 AM Day 1 (Vishnu's call against advice). #decision
- 2026-09-18 — Projects: one per team, lead only, all formats, files to Drive. #decision
- 2026-09-19 — Tasks per student, projects per team; site taken down ~1 min for the fix. — teammates' hand-ins were overwritten. #decision
- 2026-09-19 — Rebuild UI now in React + Tailwind + shadcn (Claude advised waiting; Vishnu overrode). #decision
- 2026-09-19 — Git: only two branches, main and dev. #decision
- 2026-09-19 — VCET only; multi-college out of scope. #decision
- 2026-09-19 — Marking fully automatic; manual marking removed; one plain total ranking. #decision
- 2026-09-19 — "No deploy until all built" rule, then overridden: go live night of 19–20 Sep. #decision
- 2026-09-20 — Attendance window auto 09:00–10:00 IST; Open popup with optional timer, never auto-closes. #decision
- 2026-09-20 — Survey renamed "Daily questions"; assessment per day; both data only, no marks; only Vishnu adds questions. #decision
- 2026-09-20 — "A link" hand-ins accept any http/https link. #decision
- 2026-09-20 — Profile photo feature removed. #decision
- 2026-09-21 — Read-only Viewer role for college staff. #decision
- 2026-09-21 — All files to Google Drive, nothing on server; Vishnu set "Anyone with the link" himself (Claude refused to code public sharing). #decision
- 2026-09-26 — Team photo + project name Google Form; anonymous 5-star feedback form. #decision
- 2026-09-23 — Certificate uses araCreate's own artwork, served from the server, download only (no LinkedIn share). #decision
- 2026-09-24 — Rank by total points; scale only attendance and survey to team size 4 (Option B). #decision
- 2026-09-25 — Drop the CV comparison tool; only a final-resume upload, analysis done outside the app. Leaderboard closed to students until final results. #decision
- 2026-09-26 — Final scores set from the PDF as adjustment entries; evaluation out of 50 (not 45), added on top. Project marks hidden from students and most staff screens; tied teams share a rank. #decision

## 5. Timeline
- 2026-09-16 — Plan made; v1 app built, re-skinned, UX audit fixed, EEE + ECE rosters loaded, deployed live at vcet.aracreate.academy.
- 2026-09-17 — Onboarding day; v2 plan written; v2 built overnight by two agents.
- 2026-09-18 — Day 1. v2 deployed (short outages: DB ownership, 19 s EACCES); Tinkercad codes; 129 CVs moved to Drive; project formats deployed.
- 2026-09-19 — Day 2. Project groups; per-student tasks fix; v3 plan; React UI rebuild; git cleanup; scoring v3 built; Track 5 (team page, matrix, new nav).
- 2026-09-20 — ~00:49 IST v3 go-live (GRANT bug fixed). Attendance window, any-link, upload capacity fixes. Marking screen, CSV export, search/sort, submit bug fix.
- 2026-09-21 — Viewer role, points-zero bug fixed, CV open fix, all 189 CVs to Drive, server uploads emptied.
- 2026-09-23 — Certificates built and 206 generated on production; guest certificate screen.
- 2026-09-24 — Day 6 social media carousel text corrected. Leaderboard rank bug fixed (Option B), 4 teams moved.
- 2026-09-25 — Leaderboard closed to students; "Request points" button; final resume upload; marking exports.
- 2026-09-26 — Day 9 (last day). Full export pack; final scores from PDF; project evaluation form live; top four set by hand. Team photo form and feedback form built.
- 2026-09-28 — 16 late hand-ins exported; client photo folder made; certificate downloads checked (38 of 206). Training report delivered to VCET (to confirm).

## 6. Key facts
- **People:** [[People/Vishnu]] — owner, sole admin, runs the bootcamp (not a tech person; wants simple words, one step at a time). No other named staff.
- **Companies:** [[Companies/Velalar College of Engineering and Technology]] (client), [[Companies/araCreate Group]], araCreate Academy, Hetzner (server), [[Companies/Google]] (Drive / Cloud).
- **Tools:** [[Tools/Claude Code]], [[Tools/Claude]], [[Tools/VS Code]], [[Tools/PostgreSQL]], [[Tools/Caddy]], [[Tools/systemd]], [[Tools/React]], [[Tools/Vite]], [[Tools/Tailwind CSS]], [[Tools/shadcn-ui]], [[Tools/GitHub]], [[Tools/Google Drive]], [[Tools/Google Sheets]], [[Tools/Figma]], [[Tools/Instagram]], [[Tools/LinkedIn]], Node/Express, Playwright, Discord, Tinkercad, Google Forms, Canva.
- **Links / repos / servers / file paths:**
  - Live: https://vcet.aracreate.academy (old UI at /old/)
  - Repo (private): https://github.com/aracreate-group/aca-bootcamp-dashboard (branches main, dev)
  - Design system repo: https://github.com/aracreate-group/aracreate-design-system ; conventions: https://github.com/aracreate-group/aracreate-conventions
  - Mac: ~/araCreate/bootcamp-dashboard ; backups ~/araCreate/uploads-backup-21sep, ~/araCreate/dumps/
  - Server: Hetzner VPS aca-htz-vcet (ssh alias `hetzner`), Debian 13, app /opt/bootcamp-dashboard, systemd `bootcamp`, DB `bootcamp`, backups /var/backups/bootcamp (nightly 01:00 UTC)
  - Shared Drive: ac-vcet (team folders "CODE - Team Name")
  - Google service account: aracreate-academy-1 (GCP project aracreate-academy); key (secret, not saved)
  - Passwords, join code, Tinkercad codes: (secret, not saved)
- **Related:** [[Projects/ac-ds/SUMMARY]], [[Projects/aracreate/SUMMARY]]
- Dev history: [[Projects/ac-training/DEV-LOG]] (note: most ARA-VCET Claude Code sessions 31 Jul–11 Aug are about [[Projects/the-regen-room/SUMMARY]], not VCET)

## 7. Files and documents
- `bootcamp-plan.md`, `features-and-risk.md`, `go-live-checklist.md`, `handover.md`, `index.md` — v1 plan and handover (project docs)
- `ux-audit.md`, `ux-fixes-applied.md` — UX audit and fixes (project docs; 5 of 16 claimed fixes later found missing)
- `vscode-prompts.md`, `v2-plan.md`, `v2-build-prompts.md`, `agent-working-rules.md` — v2 plan and agent prompts (project docs)
- `v3-restructure-plan.md`, `v3-decisions.md`, `v3-agent-brief.md`, `ownership-model.md`, `scoring-v3-plan.md`, `track5-track6.md` — v3 plan (project docs)
- `SESSION-STATE.md`, `work-queue.md`, `known-issues.md` — running state (project docs + repo)
- `deploy-record-20-sep*.md`, `capacity-209-uploads.md`, `any-link-hand-in.md`, `resumes-and-drive.md`, `viewer-role-college-staff.md`, `dead-points-columns.md`, `verification-report-20-sep.md` — deploy and fix records (project docs)
- `docs/deploy.md`, `docs/go-live.md`, `docs/cv-drive-migration.md`, `docs/v3-ui-migration.md`, `src/db/migrations/readme.md` — repo docs
- `scripts/go-live.sh`, `scripts/ui.sh`, `scripts/update.sh`, `scripts/migrate-cvs.js`, `scripts/chase/` — ops scripts (repo)
- User stories, HTML architecture visuals — 16 Sep artifacts

## 8. Open questions and problems
- Staff password and Google service-account key were exposed in chats — rotation not confirmed.
- Quiz and daily survey questions stayed empty for most/all days (not sure if filled later); EEE pre-assessment never given (55 students).
- Which result is official: Link Force (top score 220.0) or SWITCHSQUAD + COREX (hand-set joint 1st)? (to confirm)
- Client training report: delivered 28 Sep or still to do? (to confirm)
- v3 React rebuild (~180 h, planned in parallel lanes) went live 20 Sep — does further v3 work continue after the bootcamp?
- 168 students had not downloaded certificates by 28 Sep.
- Admin exports still include project marks; Adjust screen still lets staff work out a project mark.
- Hand-in marking: only 15 of 279 marked on 21 Sep (auto scoring covers on-time points).
- No mentor or viewer accounts created on live.
- Code: latest work on `dev`; some changes not committed in chat; schema.sql out of date; 4 old tasks.js test failures.
- Deploy trap: rsync without `--exclude uploads` wipes files; migrations run as postgres need GRANTs to app role.
- Files on Drive are "Anyone with the link" (student CVs) — privacy risk Claude flagged.
- Unexplained attendance row jump (419 -> 598) on 20 Sep night.
- Social posts used student photos — consent was suggested, not confirmed.
- Team form branching and feedback form (no sign-in) not confirmed working.

## 9. All chats in this project
- Index: INDEX (archived: Projects/ac-training/chats/INDEX.md) · Claude Code sessions: see [[Projects/ac-training/DEV-LOG]] session index
- Bootcamp infrastructure setup (archived: Projects/ac-training/chats/2026-09-16 Bootcamp infrastructure setup.md) — 2026-09-16
- The plan (archived: Projects/ac-training/chats/2026-09-16 The plan.md) — 2026-09-16
- UI/UX production workflow (archived: Projects/ac-training/chats/2026-09-16 UI-UX production workflow.md) — 2026-09-16
- Progress so far (archived: Projects/ac-training/chats/2026-09-16 Progress so far.md) — 2026-09-16
- Student progress tracking platform (archived: Projects/ac-training/chats/2026-09-17 Student progress tracking platform.md) — 2026-09-17
- Rework status report (archived: Projects/ac-training/chats/2026-09-19 Rework status report.md) — 2026-09-19
- Dual development lanes (archived: Projects/ac-training/chats/2026-09-19 Dual development lanes.md) — 2026-09-19
- Completing and completed (archived: Projects/ac-training/chats/2026-09-19 Completing and completed.md) — 2026-09-19
- Session state review (archived: Projects/ac-training/chats/2026-09-19 Session state review.md) — 2026-09-19
- Track 5 student profile page (archived: Projects/ac-training/chats/2026-09-19 Track 5 student profile page.md) — 2026-09-19
- Deployment tasks (archived: Projects/ac-training/chats/2026-09-20 Deployment tasks.md) — 2026-09-20
- Project bugs and dashboard issues (archived: Projects/ac-training/chats/2026-09-20 Project bugs and dashboard issues.md) — 2026-09-20
- Daily poster for social media (archived: Projects/ac-training/chats/2026-09-24 Daily poster for social media.md) — 2026-09-24
- Team photo and project form (archived: Projects/ac-training/chats/2026-09-26 Team photo and project form.md) — 2026-09-26
