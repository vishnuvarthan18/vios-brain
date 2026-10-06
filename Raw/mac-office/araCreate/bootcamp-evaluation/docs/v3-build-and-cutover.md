# v3 — one build, one deploy

**Decision:** stop patching v2. Build everything in `docs/v3-restructure-plan.md`
in one go, off to the side, and put it live in **one cutover**.

This document is the *how* and *when*. The *what* is in the restructure plan.

---

## 1. The rule that makes a big-bang safe

> **Nothing goes to the live server until v3 is finished and proven on a copy of
> the real data.**

v2 stays frozen and untouched until the cutover night. There is no half-migrated
state, no feature flags, no two systems running different rules on the same
students. That was the whole problem with v2 — it landed in pieces onto a live
bootcamp.

---

## 2. Timing

The bootcamp runs to 26 September with 209 students using v2 every day. v3 must
not touch that.

| | |
| --- | --- |
| **Build window** | On a `v3-dev` branch and, later, a staging box. Nothing deployed |
| **v2 during this time** | Frozen. Only the daily survey ships, once fully tested |
| **Cutover** | One night, **between batches**, never during one |

**The cutover is not urgent.** The current batch finishes on v2 and finishes
fine. v3 exists for batch 2. There are **no calendar deadlines on deploys** — a
change goes out when development and testing are finished, not on a date.

### The gap that could not wait

Knowing **who** has not done today's work. Solved with no deploy at all: a set of
read-only SQL queries producing CSVs with phone numbers. See Track 1 in
`docs/v3-agent-brief.md`.

---

## 3. What v3 is, and what it is not

**It is not a rewrite from scratch.** v3 keeps:

- The PostgreSQL database and all its data
- Every business rule that works — per-venue opening, the release gate, 30s per
  question, team mark = average of attempters, points capped at 5 a day
- The Drive integration and the storage layer
- The deploy hardening already learned

**It restructures** the code, the navigation, the data model above the tables,
and the screens — and it **replaces the front end entirely** with React +
Tailwind + shadcn/ui, phone-first, light mode only, araCreate colours.

---

## 4. Scope

Nothing is dropped from this list without a written decision.

**A. Model and naming** — `programs` and `venues`; one `activities` shape for
task / project / quiz / assessment / survey; `releases.is_open_for()` as the
single gate; one name per field; `load-*.sql` deleted, replaced by an additive
CSV import screen.

**B. Visibility** — every count links to the named list behind it, including the
negative side; activity detail page; completion matrix; CSV export and copy
phone numbers everywhere.

**C. The missing pages** — full student profile, full team profile, every name
in the app a link.

**D. Navigation** — five admin groups; one release board; student "My work".

**E. Code** — `src/modules/*`, one file per screen, `server.js` as wiring only,
every count endpoint paired with a list endpoint.

**F. Front end** — Vite + React + Tailwind + shadcn/ui, built to static files.
Backend untouched.

**G. Tests** — every route, every role, through a **real signed-in session**, at
209 students and 53 teams.

**H. Docs** — one `docs/` tree, one index, stale facts fixed.

**Deferred:** certificates, journey page, transcript.

---

## 5. How it is built

**Branch and tree.** `v3-dev`. The new front end is built alongside the old one,
not on top of it. The old tree is deleted in the **last** commit before cutover,
not the first.

**Staging, when it exists,** is a second database and a second systemd unit on
the same box, loaded from a **real production dump**, behind a staging hostname
and basic auth, refreshed every few days. Vishnu owns it; the agent does not.

**Everything is tested against real data** — 209 students, 53 teams, 159
projects. The "tested with three students" mistake does not repeat.

**Migrations are additive and reversible.** Every migration has a down. The
migration set runs against a production dump **at least five times** before the
real one.

---

## 6. Data migration

| Step | What |
| --- | --- |
| 1 | Create `programs`, insert one row: VCET Basic Electronics, 18–26 Sep, 9 days |
| 2 | Create `venues`, insert EEE and ECE, linked to that program |
| 3 | Backfill `program_id` and `venue_id` on students, teams, releases, submissions |
| 4 | Create `activities`; backfill from tasks, projects, quizzes, assessments, surveys |
| 5 | Point submissions and scores at `activity_id` |
| 6 | Verify: row counts match, total points per team match, leaderboard order identical |
| 7 | Old tables kept, read-only, for one month. Dropped only after batch 2 runs clean |

**Step 6 is the gate.** If one team's total moves by one point, the cutover does
not happen that night.

---

## 7. Before cutover — the gate list

- [ ] Every route has a session-based test, for every role that can reach it
- [ ] The full suite passes on a fresh production dump
- [ ] Migration run five times from a clean dump, with a verified down each time
- [ ] Team totals, leaderboard order and attendance counts identical before and after
- [ ] Every count on every screen opens its list — checked by hand, screen by screen
- [ ] Every screen checked at 390px with no horizontal scroll
- [ ] A full cutover rehearsal, timed, with the rollback also rehearsed
- [ ] **Batch 2 created through the UI with no code change** — the real proof v3 is a product
- [ ] `docs/ops/cutover.md` written, with the exact commands
- [ ] A verified production dump, checked for row counts, not just valid gzip

---

## 8. Cutover night — the runbook

One evening, nobody using the system, all lanes stopped.

1. Announce the window. Nothing else deploys that night.
2. `pg_dump` production. **Verify the row counts inside it.**
3. Copy the dump off the server to a second machine.
4. Stop the app. Put Caddy on a maintenance page.
5. Run the migration set. Reassign object ownership **in the same step**.
6. Deploy the payload with `git archive` of an explicit tagged commit.
7. Start the app. **Watch the log across the restart.**
8. Run the verification queries — totals, counts, leaderboard order.
9. Walk the smoke list by hand: sign in as student, lead, mentor, admin. Open a
   thing per venue. Hand something in. Score it. Click a number and get its list.
10. Lift the maintenance page.
11. Watch the log for thirty minutes.

**Rollback**, at any step: restore the dump, deploy the `v2-final` tag, restart.
Rehearsed on staging, so it is a known number of minutes, not a guess.

---

## 9. What could go wrong with one big deploy

| Risk | Answer |
| --- | --- |
| No partial rollback — it all goes or none | Full rehearsal, and the rollback is rehearsed too |
| Migration wrong on real data | Run five times from real dumps; totals verified, not assumed |
| A screen nobody rebuilt, found live | Gate list walks every screen by hand. This bit v2 twice |
| Scope grows and cutover never comes | Section 4 is the scope. Anything new goes to a v3.1 list |
| Old code paths still open underneath | Old front end deleted in the last commit, not left beside the new one |
| Something breaks in front of students | Cutover is between batches. No student is mid-bootcamp |
| Lost data | Dump before, verified, copied off the box, rollback rehearsed |

---

## 10. The one thing to hold to

**The cutover happens between batches, never during one.** Every serious problem
in this project came from changing a live system while people were standing in a
room using it. The single biggest benefit of one deploy is that it can be
scheduled for a day when nobody is there.
