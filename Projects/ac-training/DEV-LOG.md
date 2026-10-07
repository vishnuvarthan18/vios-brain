---
tags: [project]
---
# DEV LOG: VCET bootcamp dashboard (Claude Code sessions)

Built from 38 Claude Code transcripts (30 Jul to 28 Sep 2026). The app is the araCreate Academy bootcamp dashboard at vcet.aracreate.academy, used during the VCET bootcamp (18 to 26 Sep 2026). Note: most of the "ARA-VCET" sessions from 31 Jul to 11 Aug are about The REGEN Room Webflow site, not VCET. They are listed in the index, but kept short.

## What was built

- **Stack:** Node + Express + PostgreSQL, no build step at first. Repo `~/araCreate/bootcamp-dashboard`. Hosted on a Hetzner box (`/opt/bootcamp-dashboard`, systemd service `bootcamp`). Later a React (Vite) front end was added under `web/` and served at `/v3/`, then became the main UI.
- **Cohort:** 2 venues, EEE (14 teams, 55 students) and ECE (38 teams, 151 students). Later counts: 53 teams / 209 students including a staff test team, then 52 teams / 206 students after test data was removed. Team codes use the form `DEPT-TNN-NAME`.
- **Sign-in:** students use email plus one shared bootcamp code. Staff use email plus one shared staff password. Admin rights come from an `is_admin` flag.
- **Student side:** sidebar nav (a drawer on phones), profile with goals (3-year and 5-year goals, about, skills; 10-character minimum), photo, resume upload (PDF/DOCX only, file type checked by bytes), Tinkercad code per team (read-only for students), daily posts, quizzes, tasks, projects, survey, certificate download.
- **Admin side:** Students, Teams, Staff, Progress, Matrix, Open screen (per-venue release of quiz/task/project/attendance/survey/certificates/final resume/leaderboard), Projects create form, Marking, Adjust (extra points), Reports, Certificates, Evaluation (project marks), CSV/XLSX exports.
- **Google Drive:** a service account uploads hand-ins and CVs to the `ac-vcet` Shared Drive, one folder per team. No Google library, just JWT + fetch.
- **Scoring:** first it was hand-typed marks. On 19 Sep it moved to automatic scoring from recorded facts (quiz 1 pt per correct answer, on-time hand-in 5 pts, attendance 2 pts per student per day, survey 1 pt per student), plus admin adjustments.
- **Survey (learning-gain proof):** daily and final rounds, admin opens them per venue, the report shows yes / no / not asked.
- **Certificates:** PDF made from the team's own SVG artwork (flattened to 300dpi) with Playwright/Chromium on the server. Name, "Basic Electronics Mastery Workshop", 18/09/2026 to 26/09/2026, serial `ARA-2026-NNNN`. Guest certificates for people not on the dashboard (serial `G0NN`).
- **Final resume:** a `final_resume` release so the admin can open a "before / after resume" upload. `export-cvs.js` writes a CSV of both resumes.
- **Project evaluation form:** 9 criteria from the judges' sheet, out of 50, admin only, added on top of the team score.
- **Ops:** deploy from `git archive` of an exact commit, never rsync from a working tree. Take a fresh pg_dump before every deploy. Reassign view ownership and restart in one step. Never delete `uploads/`, `.archives/`, `logs/`. All of this is in `docs/deploy.md`.
- **Early work (30 Jul, ARA-VCET repo):** a scroll-driven 3D bootcamp site (Vite + React + React Three Fiber) with one 3D component per day, morphing between stages, a timeline on the left and a rotatable model on the right.

## Timeline (newest first)

- **28 Sep** – Exported 16 late hand-ins (after 26 Sep 10 am, 5 ECE teams) to xlsx. Merged a Google Form list of final project photos (last entry per team). Downloaded the photos into a client folder "Bootcamp Final Projects - 2026" (50 of 52 teams; 2 teams had no photo). Checked certificate downloads: 38 of 206 (about 18%), based on file open times, because the app does not log downloads.
- **26 Sep** – Final scores from `final-team-scores.pdf` loaded as one adjustment per team (Link Force top at 220.0, lowest 104.8). The board shows only the final score ("earned" / "given" lines removed). Project evaluation form went live (Live → Evaluation, 546 + 36 checks). Project marks hidden from students, then from staff screens too (only on the Evaluation screen and admin exports). Joint ranks for tied teams. Top four set by hand: ECE-T36-SWITCHSQUAD and EEE-T02-COREX joint 1st, ECE-T34-CODETEAM 3rd, EEE-T09-SPARKX 4th.
- **25 Sep** – Leaderboard closed for students ("The leaderboard is closed" page), "Request points" button that counts taps, admin Open/Close switch for the student board. Final resume upload deployed, plus `export-cvs.js`. Marking exports: project and task sheets, attendance grid, one `overall.csv` (52 teams, 20 columns incl. EOD posts and extra points), resumes sheet. Late night: full export pack `bootcamp-export-2026-09-26.zip` (all marks xlsx plus before/after resumes). Adjustments table now shows date and time.
- **24 Sep** – Leaderboard bug: a 136-point team ranked 3rd. Cause: `rank_by = per_member` (set 23 Sep). Fixed: rank by total points again, and attendance + survey points scaled to a 4-person team (`norm_team_size = 4`). Quiz and hand-ins not scaled (Option B). Deployed in the evening; 4 teams moved. CV comparison planned, sample before/after CVs and OCR test (about a third of CVs are photos inside PDFs; OCR read 20 of 20).
- **23 Sep** – Per-member ranking tried (local, then live). Certificates built: 4 design options, then the real SVG artwork, many fixes (flicker, white edge, two pages, badge text). Downloads served from the server, not Drive. LinkedIn share removed. 206 certificates generated on production. Guest certificate screen (Reports → Certificates). Production was down about 10 min (Playwright loaded at top of file). Research on a student comparison tool (no build).
- **19 Sep** – Per-student tasks fix (second student was overwriting the first). Site down 58 s for the deploy; 50 orphaned Drive files found and loaded into `task_submission_orphans`. Projects can be opened one by one (project groups). Open tab fix for projects. Chase-list scripts. Survey feature (Track 2, 18 items, Lane A). Fake-data generator and test harness (Lane B). Lanes merged into one. React UI migration: all 26 screens ported, shape-checked against a real dump.
- **18 Sep (Day 1)** – v2 deployed at 06:54 before students came. Tinkercad codes per team (52 loaded). Pre-assessment turned into a 4-question survey. Drive service account fixed (made Content manager on `ac-vcet`). 114 CVs copied to Drive and a folder made for every team. Advisory-lock fix for the Drive folder race. Projects accept five submission types. Projects admin create form. Health check: one 19-second outage during a deploy, one photo upload lost (recorded in `docs/known-issues.md`; no student was contacted, by choice).
- **17 Sep** – Admin can see resumes, answers, who submitted. Project/quiz submission only for team leads, later quiz opened to all. Dates no longer lock anything; admin switches control all. Staff create projects (2 to 3 per day), image upload with comment, open/close like quizzes. Quiz 1 pt per correct answer, project 5. Text contrast fixed. Test team `(secret removed)` added (no real student logins used for testing). New profile questions (3-year / 5-year goals). Night: v2 plan with two agent lanes – per-venue releases, attendance, tasks, profile completion, Drive client, full dry run (46 + 15 checks).
- **16 Sep** – Local setup, EEE then ECE data loaded, team code migration, start date 18 Sep, 9 days. Re-skin with the araCreate design system. Login page with the VCET gate photo. Name "Basic Electronics Workshop". Resume upload to the server (decided over Drive links). Student side rebuilt, mentor role removed, sidebar nav.
- **11 Aug** – (REGEN Room) SEO/GEO work, Search Console, Cloudflare purge, alt text, image compression. Also a personal daily dashboard started in `~/own/dashboard`.
- **3–5 Aug** – (REGEN Room) testimonial carousel, Kartra popup form, mobile nav, an unwanted push to production and the backup/restore fix.
- **31 Jul** – (REGEN Room) Perimenopause Reset Programme page built to Figma, responsive fixes.
- **30 Jul** – 3D bootcamp website (ARA-VCET repo) with real Sketchfab models.

## Decisions

- #decision 16 Sep – Resumes upload straight to the server (disk had plenty of space) instead of only Drive links.
- #decision 16 Sep – Remove the mentor role; staff and admin share the `mentors` table.
- #decision 17 Sep – Dates never lock anything; the admin opens and closes every item.
- #decision 17 Sep – Use a staff test team for live tests, never real student accounts.
- #decision 17 Sep – Quiz = 1 point per correct answer; project = 5 points.
- #decision 18 Sep – Deploy only from `git archive` of an exact commit; fresh pg_dump first; ownership + restart in one step.
- #decision 18 Sep – Do not message the student who lost a photo; just record it.
- #decision 19 Sep – A task is per student (each student hands in their own); a project is per team.
- #decision 19 Sep – Projects open one at a time (project groups), like tasks.
- #decision 19 Sep – Two agent lanes merged into one, because all work ran through `server.js`.
- #decision 23 Sep – Certificate uses the team's own artwork, served from the server, download only (no LinkedIn share).
- #decision 24 Sep – Rank by total points; scale only attendance and survey to team size 4 (Option B).
- #decision 25 Sep – Drop the comparison tool; only an upload place for the final resume, analysis done outside the app. The 10 students without a first resume just get 0.
- #decision 25 Sep – Leaderboard closed to students until final results.
- #decision 26 Sep – Final scores set from the PDF as extra-points entries; evaluation out of 50 (not 45), added on top.
- #decision 26 Sep – Project marks hidden from students and from most staff screens; tied teams share a rank.

## State at last session

- Last session: 28 Sep. The bootcamp is over.
- Live site works. Leaderboard has the final scores plus project evaluation marks and the hand-set top four. Certificates are open to students; 38 of 206 had downloaded by 28 Sep.
- Client pack ready on Vishnu's Mac: final project photos folder + team details xlsx.
- Code: the evaluation branch is what runs on live; `dev` did not yet have all of it (commit `c867933` not pushed at that time).

## Open items

- Merge the evaluation branch (with shared-rank change) into `dev` so a later deploy does not remove it.
- 2 teams had no final project photo (Live Wire ECE-T02, Spark Shift EEE-T13).
- 168 students had not downloaded certificates by 28 Sep. The app does not log downloads (a small change was offered).
- Admin exports still include project marks; the Adjust screen still lets staff work out a project mark.
- Bootcamp report for the client still to be made from the exports.

## Session index

- araCreate-ARA-VCET__2026-07-30_63662178 (archived: Projects/ac-training/claude-code/araCreate-ARA-VCET__2026-07-30_63662178.md) — 30 Jul — 3D scroll website for the bootcamp, Sketchfab models
- araCreate-ARA-VCET__2026-07-31_0b04f0ff (archived: Projects/the-regen-room/claude-code/araCreate-ARA-VCET__2026-07-31_0b04f0ff.md) — 31 Jul — (REGEN Room) hero height, short
- araCreate-ARA-VCET__2026-07-31_214fd31f (archived: Projects/the-regen-room/claude-code/araCreate-ARA-VCET__2026-07-31_214fd31f.md) — 31 Jul — (REGEN Room) PRP apply section, testimonials, responsive
- araCreate-ARA-VCET__2026-07-31_2a648033 (archived: Projects/ac-training/claude-code/araCreate-ARA-VCET__2026-07-31_2a648033.md) — 31 Jul — run project locally, short
- araCreate-ARA-VCET__2026-07-31_7365f964 (archived: Projects/the-regen-room/claude-code/araCreate-ARA-VCET__2026-07-31_7365f964.md) — 31 Jul — (REGEN Room) PRP page nav, video cards to Figma
- araCreate-ARA-VCET__2026-07-31_c147b55b (archived: Projects/the-regen-room/claude-code/araCreate-ARA-VCET__2026-07-31_c147b55b.md) — 31 Jul — (REGEN Room) PRP apply section, responsive (copy of 214fd31f)
- araCreate-ARA-VCET__2026-08-03_399bc7ee (archived: Projects/the-regen-room/claude-code/araCreate-ARA-VCET__2026-08-03_399bc7ee.md) — 3 Aug — (REGEN Room) video thumbnail, testimonial carousel
- araCreate-ARA-VCET__2026-08-03_bd19a94e (archived: Projects/the-regen-room/claude-code/araCreate-ARA-VCET__2026-08-03_bd19a94e.md) — 3–4 Aug — (REGEN Room) Kartra popup, nav button, mobile nav, overlays
- araCreate-ARA-VCET__2026-08-05_759444fb (archived: Projects/the-regen-room/claude-code/araCreate-ARA-VCET__2026-08-05_759444fb.md) — 5 Aug — (REGEN Room) mobile bugs, wrong push to production, backup restore
- araCreate-ARA-VCET__2026-08-11_d0268dd2 (archived: Projects/the-regen-room/claude-code/araCreate-ARA-VCET__2026-08-11_d0268dd2.md) — 11 Aug — (REGEN Room) SEO/GEO, schema, alt text, backlinks
- araCreate-ARA-VCET__2026-08-11_e61dcc0e (archived: Projects/ac-training/claude-code/araCreate-ARA-VCET__2026-08-11_e61dcc0e.md) — 11 Aug — personal daily dashboard (Gmail/Calendar/Slack)
- araCreate-bootcamp-dashboard__2026-09-16_12ce8e08 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-16_12ce8e08.md) — 16 Sep — local setup, EEE data, admin panel check, profile migration
- araCreate-bootcamp-dashboard__2026-09-16_298bcbac (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-16_298bcbac.md) — 16–17 Sep — resume upload to server, student side rebuild, sidebar
- araCreate-bootcamp-dashboard__2026-09-16_4749344d (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-16_4749344d.md) — 16 Sep — design system re-skin check, login background, workshop name
- araCreate-bootcamp-dashboard__2026-09-16_7d165055 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-16_7d165055.md) — 16 Sep — ECE data load, team codes, new secrets
- araCreate-bootcamp-dashboard__2026-09-17_0420ee88 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-17_0420ee88.md) — 17–18 Sep — v2 night build (Lane A), deploy before Day 1, Tinkercad, survey mode
- araCreate-bootcamp-dashboard__2026-09-17_304062ff (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-17_304062ff.md) — 17 Sep — no date locks, admin visibility, lead-only tabs, staff password
- araCreate-bootcamp-dashboard__2026-09-17_51587eed (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-17_51587eed.md) — 17 Sep — team and lead list per department
- araCreate-bootcamp-dashboard__2026-09-17_b0efea84 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-17_b0efea84.md) — 17 Sep — v2 Lane B: profile completion, Drive client, CV migration script
- araCreate-bootcamp-dashboard__2026-09-17_c3517afb (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-17_c3517afb.md) — 17 Sep — required profile answers (10 chars), PDF/DOCX only, deploy
- araCreate-bootcamp-dashboard__2026-09-17_fc9fb15c (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-17_fc9fb15c.md) — 17 Sep — UI polish, staff projects, open/close, scoring, test team, go live
- araCreate-bootcamp-dashboard__2026-09-18_8c3b0c60 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-18_8c3b0c60.md) — 18–19 Sep — Drive go-live, Tinkercad deploy, CVs to Drive, folder lock, projects admin
- araCreate-bootcamp-dashboard__2026-09-18_8da504f0 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-18_8da504f0.md) — 18–19 Sep — health check, known issues, project formats, project groups
- araCreate-bootcamp-dashboard__2026-09-19_2bfb9c3b (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-19_2bfb9c3b.md) — 19 Sep — chase lists, survey Track 2 (Lane A), live 500 fix
- araCreate-bootcamp-dashboard__2026-09-19_575dea2b (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-19_575dea2b.md) — 19 Sep — per-student tasks, orphan Drive files, 58 s deploy
- araCreate-bootcamp-dashboard__2026-09-19_919f31ff (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-19_919f31ff.md) — 19 Sep — Lane B: fake data, harness, handover
- araCreate-bootcamp-dashboard__2026-09-19_b78f928f (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-19_b78f928f.md) — 19 Sep — React UI migration (26 screens), shape check, local run
- araCreate-bootcamp-dashboard__2026-09-23_4ad34fb2 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-23_4ad34fb2.md) — 23 Sep — marking review, per-member rank, certificates built and live
- araCreate-bootcamp-dashboard__2026-09-23_fd2efc80 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-23_fd2efc80.md) — 23–25 Sep — comparison tool research, CV plan, OCR, final resume upload deploy
- araCreate-bootcamp-dashboard__2026-09-24_ac3e9176 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-24_ac3e9176.md) — 24 Sep — leaderboard rank bug, team-size scaling (Option B), deploy
- araCreate-bootcamp-dashboard__2026-09-25_1d4a65ed (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-25_1d4a65ed.md) — 25 Sep — local run, adjustments show date
- araCreate-bootcamp-dashboard__2026-09-25_77f6dee5 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-25_77f6dee5.md) — 25 Sep — marking exports, attendance, overall sheet, resumes sheet
- araCreate-bootcamp-dashboard__2026-09-25_96117e33 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-25_96117e33.md) — 25–26 Sep — full export pack, final scores from PDF, board shows final only
- araCreate-bootcamp-dashboard__2026-09-25_9dee9cca (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-25_9dee9cca.md) — 25 Sep — board closed to students, request points, admin switch
- araCreate-bootcamp-dashboard__2026-09-26_35896f6f (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-26_35896f6f.md) — 26 Sep — hide project score from students
- araCreate-bootcamp-dashboard__2026-09-26_4a7c9e26 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-26_4a7c9e26.md) — 26 Sep — joint 1st place, top four set, project marks hidden from staff
- araCreate-bootcamp-dashboard__2026-09-26_7e7879d5 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-26_7e7879d5.md) — 26 Sep — project evaluation form (out of 50), live and tested
- araCreate-bootcamp-dashboard__2026-09-28_467fca82 (archived: Projects/ac-training/claude-code/araCreate-bootcamp-dashboard__2026-09-28_467fca82.md) — 28 Sep — late submissions export, client photo folder, certificate download count
