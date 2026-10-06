---
tags: chat
project: "[[Projects/ac-training/SUMMARY]]"
date: 2026-09-16
source: Claude office account (Cowork)
session: cse_01YWTR1L3wnvbitW9Mx3rL3D
---
# Progress so far

- Project: [[Projects/ac-training/SUMMARY]]
- Date: 2026-09-16
- Full chat, word for word (tool steps shortened).

---

**Vishnu:** so what is done so far

**Claude:**   _[tool: Projects — claude/index.md]_

**Claude:**   _[tool: Projects — claude/handover.md]_

**Claude:**   _[tool: Projects — claude/go-live-checklist.md]_

**Claude:** **Done so far — Basic Electronics Workshop app**

Built and live
- App is live at https://vcet.aracreate.academy (HTTPS, Hetzner server, Debian 13 + Node 20 + PostgreSQL 17 + Caddy)
- Code pushed to the private repo `aracreate-group/aca-bootcamp-dashboard`

Work completed, in order
1. UX audit of the working app — 5 blockers, 10 high, 11 medium — all fixed
2. ECE roster loaded: 38 teams, 151 students (on top of EEE's 14 teams, 55 students) → total 52 teams, 206 students
3. All team codes standardised to `DEPT-TNN-TEAMNAME`
4. Front end rebuilt on the araCreate design system — Home page per role, charts, toasts, breadcrumbs, empty states
5. Onboarding day screen built (so 17 Sep is not a blank page)
6. Repo brought to araCreate conventions — structure, SPDX headers, snake_case, conventional commits
7. Deployed to the server, behind HTTPS

Also in place
- Roles working: student, team lead, mentor, admin
- Sign-in: students = email + code `ARA2026`; staff = email + password from `.env`
- Points to teams only, 5 project + 5 quiz per day, leaderboard kept live by a DB trigger
- Privacy: daily posts readable only by the student and admin
- Nightly database backup at 01:00, 14 days kept
- Both student login sheets ready (EEE and ECE CSVs)
- 71 Playwright checks passing across 4 suites, against a real server and database
- Docs written: index, handover, deploy, design, go-live checklist, UX audit, team codes, ECE load

Not done yet (before Friday 18 Sep)
- Day 1 and Day 2 quiz questions to be pasted in
- 52 Google Drive folders, one per team
- Mentors assigned to teams
- Tomorrow (17 Sep): share login sheets, put `ARA2026` on the board

Sources: `claude/index.md`, `claude/handover.md`, `claude/go-live-checklist.md`

**Vishnu:** ok give me the promt i will fix the one by one in the direct vs code

**Claude:** Paste these into Claude Code in VS Code, one at a time, from `~/araCreate/bootcamp-dashboard`.

**0. Start every session with this context line**

```
Read docs/handover.md and docs/go-live-checklist.md first. This is the live bootcamp app at vcet.aracreate.academy — 52 teams, 206 students, onboarding is 17 Sep, Day 1 is 18 Sep. Never run load-eee.sql or load-ece.sql. Never change start_date. Follow the araCreate conventions in the repo.
```

**1. Quiz questions — check the bulk paste works**

```
Open the Admin > Quizzes > Paste many flow end to end. Confirm the parser accepts lines in the format "question | A | B | C | D | correct letter", trims spaces, is case-insensitive on the correct letter, rejects bad lines with a clear message instead of silently skipping, and shows how many were added. Write a Playwright test for it in tests/. Do not change the quiz lock behaviour.
```

**2. Mentor assignment — make it bulk**

```
Right now mentors are assigned one team at a time in Admin > Teams > Edit. Add a way to assign mentors to many teams at once from the Teams list (select teams, pick a mentor, apply), respecting the Both/EEE/ECE filter. Server-side validation that the user is a mentor. Add a test. Use the existing design system components, do not write new CSS.
```

**3. Drive folder links — bulk entry for 52 teams**

```
Add an admin screen or a paste-many box to set the Google Drive folder link for many teams at once, format "TEAMCODE | drive-url" per line. Validate the URL looks like a Drive link and the team code exists. Show a per-line result. Add a test.
```

**4. Onboarding-day dry run**

```
Run the full onboarding-day path as a student against a local server and real Postgres: sign in with email + ARA2026, submit an old resume Drive link, write a goal, fill the profile. Confirm Home shows only those three tasks, the banner says Day 1 is tomorrow, and the daily post box explains posting opens on Day 1. Report anything broken, then fix it.
```

**5. Day 1 dry run**

```
With start_date set so today is Day 1 in a LOCAL database only (never on the server, and restore it after), run: lead marks attendance, lead hands in the team Drive link, team takes the quiz once, mentor scores 0-5, leaderboard updates. Confirm teams.total_points is correct via the trigger. Then restore start_date and confirm it is back.
```

**6. Privacy re-check before go-live**

```
Audit every API route in src/server.js and confirm no endpoint lets one student read another student's daily post or profile, and that mentors cannot read posts. Check /api/posts, /api/profile, team page, journey and progress. Write a Playwright test that tries each of these as the wrong role and expects 403.
```

**7. Final go-live check**

```
Run make test — all four suites. Then check the live site: sign in as each of the four test accounts in docs/go-live-checklist.md and confirm the right Home page loads. Report a short pass/fail list. Do not deploy anything.
```

**8. Deploy (only when a fix is ready)**

```
Deploy the current branch using the exact steps in docs/deploy.md. Commit with conventional commit format first. After deploy, tail the logs and confirm the service is healthy and the site loads.
```

**Vishnu:** what are tehse

**Claude:** Those are **ready-made instructions you copy and paste into Claude Code inside VS Code** — one at a time.

How it works

- You open your project folder in VS Code
- You open Claude Code there
- You paste one block
- Claude does that one job in your code, you check it, then move to the next block

Why I split them

- Each block = one pending item before your bootcamp goes live
- Doing one at a time is safer than asking for everything at once

What each one is for

- **0** — context line, paste first so Claude knows the project and the rules (never re-run the roster scripts, never move start_date)
- **1** — make sure pasting quiz questions in bulk works
- **2** — assign mentors to many teams at once instead of one by one
- **3** — add 52 team Drive links in one go
- **4** — test tomorrow's onboarding day as a student
- **5** — test Day 1 fully on your laptop only
- **6** — re-check that no student or mentor can read another student's private post
- **7** — final check before go-live
- **8** — deploy the fix to the server

Short version: 0 first, then 4 and 7 before tomorrow morning. 1, 2, 3 before Friday. 5, 6 whenever you have time. 8 only after a fix is ready.

**Vishnu:** save all

**Claude:**   _[tool: Projects — claude/vscode-prompts.md]_

**Claude:** Saved to the project as `claude/vscode-prompts.md`.

- All 9 prompt blocks, ready to copy
- A short table saying what each one is for
- The suggested order: 0 → 4, 7 → 1, 2, 3 → 5, 6 → 8
- The two "never do" rules at the bottom

You can open it from the project in any Claude session or on claude.ai.
