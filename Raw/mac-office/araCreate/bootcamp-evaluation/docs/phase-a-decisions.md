# Phase A — decisions pre-answered

Written so no Phase A item has to stop and ask. Read with
`docs/work-queue.md` and `docs/v3-agent-brief.md`.

**Order changed 19 Sep:** A1 → A2 → A4 → A5 → A3. The module split runs **last**,
after the session test harness exists and after the small items are banked. A
night that lands four items and starts the fifth is a good night; a night that
starts with the hardest item and stalls is a wasted one.

---

## A1 — fake-data generator

- 209 students, 53 teams, two venues (EEE 14 teams / 55 students, ECE 39 / 154),
  nine days.
- Shapes matter more than realism: some teams complete, some partial, some
  silent. Some students with no team. Some attendance marked absent, some rows
  simply missing. That is what makes tests find things.
- **Names and phone numbers are invented.** Never copy real students' details
  into a fixture file — fixtures get committed, real contact details must not be.
- Deterministic: a seed argument, so the same seed gives the same data twice.
- It writes only to the **local** database.

## A2 — session test harness

This is the safety net for everything after it. Get it right.

- Every test signs in through the **real** login and carries the session. A test
  that hits an endpoint without a session does not count and must not be written.
- Four roles: student, team lead, mentor, admin.
- **Credentials:** read the staff password from the local `.env`. **Never print
  it, never commit it, never ask for it.** If the local `.env` has none, set a
  local-only password directly in the local database and say in the log that you
  did — that is a local fixture, not a secret.
- The student join code is `ARA2026`.
- **Every route gets a test for every role that can reach it, and for at least
  one role that must not.** A route that only proves the happy path is untested —
  the worst v2 bug was a lead locked out of their own work while 41 checks passed.
- Run at full volume: 209 students, 53 teams.

## A4 — migration ledger

- `src/db/migrations/readme.md` lists 9; the directory holds 16. The dump has
  all 16 — that was proved in Track 1.
- Produce the real run order, and record how each was verified as applied
  (column marker, table marker, or by reading a view's own definition — the
  method Track 1 used for the two view-only ones).
- This is documentation. Change no migration and run nothing against any
  database.

## A5 — `v_student_progress`

The view still exposes `has_photo` and `has_education` after
`src/routes/profile-completion.js` dropped both from the weights.

**Do not drop the columns.** That is a Phase B job, once nothing reads them.
Dropping a view column overnight breaks whatever silently depends on it.

Do this instead, in order:

1. **Find every reader** — the view's columns, across routes, front end, scripts
   and tests. List them in the log, each with file and line.
2. **Fix the readers** that use it to decide whether a profile is complete, so
   they use the JS weights instead.
3. **Mark the view** — a comment header on the view definition saying the two
   columns are deprecated, kept only until Phase B, and are not a completeness
   signal.
4. If a reader turns out to genuinely need them for something other than
   completeness, **leave it alone** and write it up. That is a finding, not a bug.

Nothing here should stop the run. If step 2 is larger than expected, fix what is
clearly completeness-related, log the rest, and move on.

## A3 — module split, LAST and in small pieces

**One module per queue item.** Never one commit that moves 3,000 lines — a single
large move cannot be reviewed, and a clean move is indistinguishable from a
subtle behaviour change.

Order, chosen so each module only depends on ones already moved:

`auth` → `students` → `teams` → `venues` → `releases` → `activities` →
`submissions` → `scoring` → `attendance` → `reports` → `storage` → `programs`

For each one:

- Move routes, service and queries into `src/modules/<name>/`.
- **Behaviour does not change.** The same assertions are green before and after.
- No cross-module SQL. If a module needs another's data, it calls that module.
- Commit and push that module alone, then take the next.
- If a module cannot move cleanly, **stop that module, log why, move to the next
  one.** Do not force it and do not refactor to make it fit.

`server.js` is reduced to wiring only in the final item, not the first.
