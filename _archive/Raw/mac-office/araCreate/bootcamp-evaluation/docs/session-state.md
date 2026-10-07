# Session state — read this first in a new chat

Last updated **19 Sep 2026, Day 2 of the bootcamp**.

A fresh session should read this file, then `docs/lanes.md`, then
`docs/work-queue.md`. Everything else is reference.

---

## 1. Situation

| | |
| --- | --- |
| Live | https://vcet.aracreate.academy |
| Running | Fri 18 Sep → Sat 26 Sep 2026, nine days |
| Scale | **209 students, 53 teams** — EEE 14/55, ECE 39/154 |
| Repo | `~/araCreate/bootcamp-dashboard` on the Mac |
| Server | `ssh hetzner` · Debian 13 · `89.167.82.144` · **UTC, not IST** |

The bootcamp is live. v2 shipped overnight 17–18 Sep. v3 is being built to the
side. **Nothing is deployed without Vishnu doing it himself, at night.**

---

## 2. TWO LANES ARE RUNNING

This is the most important thing to know.

| | **Lane A** | **Lane B** |
| --- | --- | --- |
| Work | Track 2 — the daily survey | Track 3 Phase A — foundation |
| Branch | `survey` off `main` | `v3-dev` off `main` |
| Worktree | main checkout | `.worktrees/lane-b` |
| Owns | `src/server.js`, `src/public/app.js`, `app.css`, survey migrations, `tests/survey*.js`, `docs/survey-spec.md` | `tests/harness/**`, `tests/gates.js`, `scripts/seed/**`, `src/db/migrations/readme.md`, `docs/migration-ledger.md` |

**`docs/lanes.md` is the ownership contract.** An agent editing a file it does
not own is a stop-work event.

**A3, the module split, is serial** — it rewrites `server.js`, which Lane A owns.
It starts only after `survey` merges to `main`.

---

## 3. Progress

| Track | State |
| --- | --- |
| **Track 1 — chase lists** | **DONE.** 14 items, 17 checks, pushed on `chase-lists` |
| **Track 2 — survey** | **8 of 18**, Lane A, on T2-08 |
| **Phase A — foundation** | Starting, Lane B |
| **Phases B–F** | Not started |

**About 8% of the total build** (~12 h of ~180 h). Item count flatters it —
Phase D, rebuilding every screen in React, is 40–60 h on its own.

Roughly **75% of what is needed before 26 September** is done.

---

## 4. Documents, and what each is for

| File | Purpose |
| --- | --- |
| **`session-state.md`** | This file. Start here |
| **`lanes.md`** | The two-lane file-ownership contract |
| **`work-queue.md`** | What the agents work from. Status per item |
| `v3-agent-brief.md` | The authority for agents. 21 hard rules, schema facts |
| `standing-authorisation.md` | Local and reversible = decide it. Reaches the server = never |
| `unattended-operation.md` | How agents run without Vishnu. Never stop for a question |
| `ownership-model.md` | **Authority on ownership.** Who owns what, how it scores |
| `survey-spec.md` | The survey, fully specified, all decisions pre-answered |
| `phase-a-decisions.md` | Phase A decisions pre-answered |
| `v3-restructure-plan.md` | What is being rebuilt and why |
| `v3-build-and-cutover.md` | Staging, migration, gate list, cutover runbook |
| `v3-decisions.md` | Numbered decision log |
| `agent-working-rules.md` | Rules learned from the v2 build |
| `ops-findings-19-sep.md` | Backup bug, cron schedule, dump row counts |
| `agent-log.md` | Agent reports, append-only, prefixed LANE A / LANE B |
| `questions-for-vishnu.md` | What agents blocked on. **Check this every time** |

---

## 5. Decisions that must not be relitigated

**Working**

- The agent **never** deploys, never `ssh`s to the server, never touches the
  production database. Vishnu deploys, at night, when nobody is working.
- No calendar deadlines on deploys. Fully tested first, then the next window.
- One build, one cutover, **between batches, never during one**.
- Agents report at the end of each item and keep going. A question is never a
  reason to stop — it goes to `questions-for-vishnu.md` and the item is skipped.
- **A test is not a control until you have watched it fail.** Every guard test is
  proven by sabotage at the level it claims to protect.

**Ownership** — `docs/ownership-model.md`

- **Project** — team-owned, **team lead only**, mentor scores 0–5.
- **Task** — **per student**, auto-marked: handed in 5, not 0. Team mark is the
  average across **all** members.
- **Quiz** — per student, team mark is the average of those who **attempted**.
  The inconsistency with task is deliberate and documented.
- **Attendance** — lead marks, **lead's dashboard only**.
- Survey, post, assessment, CV — per student.
- Every mark rolls up to the team. No individual leaderboard.
- **Not to be applied to the live app.** It changes visible scores. v3 cutover.

**Survey** — `docs/survey-spec.md`

- Proof of learning gain. Daily round before each day's teaching; **final round
  at the end re-asks every question from all nine days**, reusing the same
  question rows so pairing is automatic.
- Yes/No, per student, zero points. First in My work, tagged Required, does not
  lock other items. First tap is locked — screen must warn before, not after.
- Final round **hides** the earlier answer. Admin only sees the proof report.
- **Three states always: yes · no · not asked, with the denominator.** Never a
  bare percentage. `not_asked` derives from the roster, never `total − yes`.

**UI rebuild**

- React + Tailwind + shadcn/ui, Vite, served as static files. Backend untouched.
- araCreate colours, **light mode only**, phone-first at 390px.
- **Phase C3 is a hard stop** — three screens approved before 20+ more are built.

---

## 6. Live findings, open

- **No project has ever been opened.** All 159 are `is_open=false`. Admin team
  handling it. Agents must not touch it.
- **50 orphan Drive files** from 27 teams against Day 1's task. Real work,
  counted by nobody.
- **Quiz questions: 0** across all nine days. Only Vishnu can write them.
- **EEE has never had the pre-assessment.** 154 attempts = exactly ECE.
- **`personal_email` is empty for all 209.** Phone only, everywhere.
- 36 CVs await their Drive copy. A count, never names.
- Backups were broken from 16–19 Sep and are now fixed. Check after **07:00 IST**
  daily: `ssh hetzner 'sudo ls -lh /var/backups/bootcamp'` — want ~80K or larger.

## 7. Gate budgets, measured

| Budget | Today | Survives v3? | Cleared by |
| --- | --- | --- | --- |
| activity type gates, server-side | 17 | yes | B4, by hand |
| venue literals, server-side | 13 | yes | B2, by hand |
| venue literals, `app.js` | 24 | no | C1/C2, by deletion |
| page routes, `app.js` | 10 | no | C2, by deletion |

Four bugs so far are all one bug: **a new value added to a gated concept, and an
old list enumerating the values that nobody updated.** Three of the four were the
agent's own, in one feature, in one day — knowing about the pattern did not
prevent it. That is the argument for the v3 activity registry: the fix cannot be
vigilance.

---

## 8. Still unanswered by Vishnu

1. Can a **team lead see which of their six members** finished today's work?
   Recommendation: yes for status, no for content.
2. Can a **mentor** open a student profile? Recommendation: yes, minus posts.
3. What counts as **"done"** for a task with no hand-in type?
4. **Absent vs not-yet-marked** must be two different colours in the matrix.
5. Profile photos are on the server only, in no backup. Move to Drive or accept.
6. **Certificates, journey page, transcript** — deferred, not dropped.

---

## 9. What a new session should do

1. Read `questions-for-vishnu.md`. Answer anything there first.
2. Read the newest `agent-log.md` entries for both lanes.
3. Keep both lanes moving. Answer, do not ask — decide anything local and
   reversible on Vishnu's behalf and tell him what you decided.
4. **Do not write files into the repo while the agents are working in it.**
   Two writers in one tree already caused one near-miss. Hand Vishnu the text,
   or have a lane commit it.
5. Vishnu wants **points, simple English, no long paragraphs.**
