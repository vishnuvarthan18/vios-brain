# Two lanes — file ownership and rules

Running two agents to go faster. This file is the ownership contract. **An agent
that edits a file it does not own is a stop-work event**, whatever the reason.

Both lanes follow `docs/standing-authorisation.md` and
`docs/unattended-operation.md` unchanged. Neither lane deploys.

---

## The split

| | **Lane A** | **Lane B** |
| --- | --- | --- |
| Work | Track 2 — the survey | Track 3 Phase A — foundation |
| Branch | `survey` off `main` | `v3-dev` off `main` |
| Worktree | the main checkout | **its own worktree**, e.g. `.worktrees/lane-b` |
| Queue | `docs/work-queue.md` Track 2 | `docs/work-queue.md` Phase A |

**One worktree per lane, always.** Two agents in one working tree caused a
production incident in v2 and a commit on the wrong branch. Lane B works in its
own worktree or it does not start.

---

## File ownership

### Lane A owns — Lane B must not touch

```
src/server.js
src/public/app.js
src/public/app.css
src/db/migrations/*survey*
tests/survey*.js
docs/survey-spec.md
```

### Lane B owns — Lane A must not touch

```
tests/harness/**          the session test harness
tests/gates.js            the gate budgets
scripts/seed/**           the fake-data generator
src/db/migrations/readme.md
docs/phase-a-decisions.md
docs/migration-ledger.md
```

### Shared, append-only — both may write

```
docs/agent-log.md               append your own entries, never edit another's
docs/questions-for-vishnu.md    append only
docs/work-queue.md              tick only your own lane's rows
```

If either lane hits a merge conflict in a shared file: **keep both sides**, never
discard. Log it.

---

## What cannot run in parallel

**T3-A3, the module split, is serial.** It rewrites `src/server.js`, which Lane A
owns for the whole of Track 2. A3 does not start until Track 2 is `DONE` or
`BLOCKED` and `survey` is merged to `main`. No exceptions — this is the single
highest-risk collision in the project.

**A5 is split**, because fixing `v_student_progress`'s readers means editing
`src/server.js`:

- **A5a — audit only.** Find every reader of the view, with file and line.
  Read-only. **Lane B does this.**
- **A5b — the fix.** Edit the readers. **After `survey` merges**, by whichever
  lane is free.

---

## Lane B's order

`A1` fake-data generator → `A2` session test harness → `A1b` make the `releases`
and `tasks` suites run against the generator → `A4` migration ledger →
`A5a` reader audit → then **stop and wait** for `survey` to merge before A3.

Lane B has roughly 7–10 hours of work that touches nothing Lane A owns.

---

## Merging

1. Lane A finishes Track 2 → reports → **Vishnu deploys it** in a window.
2. `survey` merges to `main`.
3. `v3-dev` rebases onto `main`.
4. Only then does A3 begin, as 12 module items, in one lane alone.

---

## What this actually buys

Not 2×. Realistically **1.5–1.7×**, because the largest single item — the module
split — cannot be parallelised at all, and because two lanes cost coordination.

It also costs roughly **double the tokens**. Worth it while there is genuinely
independent work; not worth it once everything funnels back into `server.js`.
