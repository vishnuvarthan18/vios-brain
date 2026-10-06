# Questions for Vishnu

Written by the agent when an item needs a decision that is not already on record.
Each entry: what is blocked, the options, and the recommendation.

Nothing here stops the run -- the agent skips the item and continues.

---

## Q1. Track 2 needs a branch off `main`, and the queue says to start it

**Blocked:** T2-01 through T2-13, the whole daily survey track.

**Why this is not something to guess.** The queue says to start Track 2 once
Track 1 is DONE, and Track 1 is now DONE. But the brief's Part 3 says Track 2 is
`survey` off `main`, and unattended-operation.md section 7 says the queue must
only ever contain work whose decisions are already made. Track 2 fails that test
on one specific point, below. Everything else about it is decided.

**The undecided point: what the survey questions are for.** v3-decisions.md
lists this under "Still open", item 1: *"What the survey questions are actually
for -- Vishnu had more to add."* The schema, the release model, the screens and
the results views are all decided (D7-D14) and I could build every one of them.
But D14 says results must show "the same question tracked across days", and
whether a question is the same question across days is a content decision I
cannot make. Build it one way and a question reworded on Day 4 breaks its own
trend line; build it the other and two genuinely different questions get merged.

**Options:**

1. **Match on exact question text.** Simple, no admin work. A single typo or
   reword on a later day silently starts a new trend line.
2. **A stable `key` column** the admin sets when pasting, blank meaning
   "not tracked". Correct across rewording, and the paste-many loader stays one
   line per question with an optional `key: text` prefix. Costs the admin a
   decision per question.
3. **Track nothing across days**, ship the per-day counts, add trends later.

**Recommendation: option 2.** It is the only one that survives a reworded
question, which is certain to happen across nine days written the evening
before. The cost is one optional prefix in the paste box, and blank means the
current behaviour. It is also additive, so option 1 or 3 remains available.

**What I did instead:** stopped at the end of Track 1 rather than starting T2-01.
Track 2 ships to the live app, which makes an overnight guess about its data
model expensive to undo -- unattended-operation.md section 4 says an assumption
that is expensive to undo is a question, not an assumption. The rest of Track 2
is unblocked the moment this is answered.

---

## Q2. Three seeded test accounts are excluded from every chase list

**Not blocking** -- decided and implemented, flagged here because it is the one
judgement call in Track 1 that changes who gets rung.

TEST0001, TEST0002, TEST0003 in ECE-T99-TESTTEAM are `is_active = true` in the
dump. Left in, they appear on all nine lists every day. `_chased.sql` excludes
them by a named rule and `00-flags.sql` reports how many were excluded, so the
rule can never silently swallow a real student.

Excluding them is also what makes the ECE cohort read 151, which matches the
figure in the brief. With them included it reads 154.

**If they are real students, say so and I will remove the exclusion.**


---

## Q3. `tests/gates.js` already exists on `survey`, and Lane B now owns it

**Not blocking** — flagged because it is an ownership collision that already
happened, before `docs/lanes.md` existed.

**What happened.** I wrote `tests/gates.js` (the enumeration budgets) and wired
it into `make test` earlier today, on branch `survey`, committed and pushed in
e9ae524 and the commit before it. `docs/lanes.md` now assigns that file to Lane
B. I have not touched it since reading the lanes doc, and I will not.

**State it is in.** Four budgets, each naming the phase that clears it:

| Budget | Today | Cleared by |
| --- | --- | --- |
| activity type gates, server-side | 17 | B4, by hand |
| venue literals, server-side | 13 | B2, by hand |
| venue literals, `app.js` | 24 | C1/C2, by deletion |
| page routes, `app.js` | 10 | C2, by deletion |

All four were sabotage-tested individually — broken, watched red, restored.
`Makefile` runs it first in `make test`, before anything needing a server.

**Why it matters at merge.** `v3-dev` was cut from `cb7ac5d`, which predates it,
so Lane B's branch does not have this file. When `survey` merges to `main` and
`v3-dev` rebases, Lane B will find `tests/gates.js` and a `Makefile` line
arriving underneath it. If Lane B has written its own by then, that is a
conflict in a file the lanes doc says is theirs.

**Recommendation.** Lane B takes mine as the starting point rather than writing a
second one — the budgets are calibrated against real counts and the survival
split (B2 is 13 sites, not 37) is the part worth keeping. If Lane B has already
written one, keep both sides per the lanes rule and reconcile the budgets, since
the numbers are the valuable part, not the harness.

**What I need.** Nothing to proceed. Lane B should know the file is coming.

---

## Q4. Handing in a task by text or Drive link is broken, in code neither lane owns

**Found by:** LANE B, T3-A1b, running the existing `tests/tasks.js` against the
generated fixture. Not fixed — `src/server.js` and `src/routes/` are Lane A's
for the whole of Track 2, and the brief's rule 19 says a finding is written
down, not fixed.

**What happens.** `POST /api/tasks/:id/submit` returns **500 "Something went
wrong"** for any task whose `submission_type` is `text` or `drive`. The student
sees a generic error and their work is not recorded.

**Why.** `src/server.js:1374` does:

```sql
ON CONFLICT (task_id, team_id) DO UPDATE
```

There is no longer a plain `UNIQUE (task_id, team_id)` to infer. The
`a-quiz-per-student` migration replaced it with two **partial** unique indexes:

```
idx_task_sub_team    (task_id, team_id)      WHERE NOT per_student
idx_task_sub_student (task_id, submitted_by) WHERE per_student AND submitted_by IS NOT NULL
```

Postgres will not use a partial index as an ON CONFLICT arbiter unless the
statement repeats its predicate, so it raises `42P10, there is no unique or
exclusion constraint matching the ON CONFLICT specification`.

**This is not a fixture artefact.** It reproduces against `bootcamp_local`,
loaded from the real dump — the same statement fails there, and adding
`WHERE NOT per_student` makes the planner pick `idx_task_sub_team`.

**Why nobody has hit it.** All 56 `task_submissions` rows in the dump are
`per_student = true`, i.e. the photo/file upload path, which goes through
`src/routes/drive-uploads.js` and does not reach this statement. The text and
Drive-link hand-in paths appear never to have been used in anger.

**There is a second site, with the same defect.**
`src/routes/drive-uploads.js:389` uses the identical `ON CONFLICT (task_id,
team_id)`, and its comment still cites the `UNIQUE (task_id, team_id)` that the
migration removed. It has not been reproduced, because that path needs a file
upload, but it is the same statement against the same table and will fail the
same way whenever `per_student` is false.

**The fix, in both places:**

```sql
ON CONFLICT (task_id, team_id) WHERE NOT per_student DO UPDATE
```

**Evidence it is the whole problem.** With that one line changed — on a
throwaway copy, never on the real file — `tests/tasks.js` goes from
**9 passing / 5 failing** to **43 passing**.

**Recommendation.** One-line change in each of the two sites, by whichever lane
owns `server.js` when Track 2 lands. It is additive, needs no migration, and has
a test that already proves it. **Do not** add a plain UNIQUE constraint back:
that would break the per-student tasks the partial indexes exist for.

**Also worth knowing:** `tests/tasks.js:283` asserts a unique **constraint**
exists by counting `pg_constraint WHERE contype = 'u'`. That assertion is stale
for the same reason — the uniqueness is now an index, not a constraint — so it
fails even against a correct database. Three of the remaining five failures are
of this kind and are the test being out of date, not the code being wrong.

---

## Q5. A5b is nearly empty, and probably does not need to wait for the merge

**Found by:** LANE B, T3-A5a, the read-only reader audit. Not acted on — the
order says stop after A5a.

**What the audit found.** `v_student_progress` has exactly **one** reader in
the whole repo:

```
src/server.js:3430   SELECT * FROM v_student_progress
```

It feeds `/api/admin/progress`, whose screen (`src/public/app.js:2546`) renders
eight columns — `has_goal`, `has_goal_3y`, `has_goal_5y`, `has_resume_v1`,
`name`, `posts`, `roll_no`, `team_code`. **Neither `has_photo` nor
`has_education` is among them**, and a repo-wide grep finds no other use of
either name outside the files that define the view.

`src/routes/profile-completion.js`, the authoritative weights, does not read
the view at all — it reads `student_profiles` directly.

**Why this matters for the plan.** `docs/phase-a-decisions.md` A5 has four
steps: find the readers, fix the completeness readers, mark the view, write up
anything that genuinely needs the columns.

- Step 1 — done, one reader.
- Step 2 — **no work exists.** No reader uses the view to decide completeness.
- Step 3 — a comment header on the view definition.
- Step 4 — **no work exists.** Nothing needs the columns.

So A5b is step 3 alone: a comment on a migration file saying the two columns
are deprecated, kept until Phase B, and are not a completeness signal.

**The decision.** `docs/lanes.md` splits A5 across the merge because "fixing
`v_student_progress`'s readers means editing `src/server.js`". There are no
readers to fix, so that reason does not apply. A5b touches no file Lane A owns.

**Options:**

1. **Do A5b now, before the merge** — one comment header, no `server.js`
   change. *Recommended.* It is the cheapest item left in Phase A and the
   reason for deferring it has gone.
2. Leave it queued behind the merge as written. Costs nothing but keeps a
   five-minute job on the critical path for no reason.

**One thing to decide either way:** which file the comment goes in.
`src/db/migrations/*.sql` is not in this lane's ownership list, and the view is
currently defined across four migrations. Writing a new
`2026-09-20-a-progress-view-deprecation.sql` that does `CREATE OR REPLACE VIEW`
with the comment is additive and keeps the history honest; editing an
already-applied migration in place would not.
