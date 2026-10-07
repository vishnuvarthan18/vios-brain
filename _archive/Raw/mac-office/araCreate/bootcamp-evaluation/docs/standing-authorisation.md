# Standing authorisation — run to the end

Given 19 Sep. Applies to Track 2 and Track 3 Phase A.

## The line

> **You may decide anything that is local and reversible.
> You may decide nothing that reaches the server.**

That is the whole rule. Everything below is it, spelled out.

## Pre-authorised — do not ask, just do it

- Install **devDependencies** — Playwright, any test or screenshot tooling.
- Create files and folders under `docs/`, `scripts/`, `tests/`, `src/modules/`,
  `src/db/migrations/`.
- Write additive migrations, each with a working `down`.
- Create, reset and reload the **local** database as often as needed.
- Create branches, commit, push.
- Choose naming, file layout and test structure where the brief is silent.
- Follow the nearest existing pattern when a detail is unspecified — the quiz is
  the pattern for the survey.

## Still forbidden — no authorisation overrides these

- Deploying. Anything.
- `ssh` to the production server.
- Connecting to the production database.
- Committing to `main`.
- Adding a **runtime** dependency to `package.json` — devDependencies only.
- Editing `scripts/setup-server.sh`, the deploy scripts, or anything under
  `.env`.
- `git stash`, `reset --hard`, rewriting a pushed branch.
- Running `load-eee.sql` or `load-ece.sql`.
- Deleting data anywhere.

## When something is unclear

**Never stop the run.** In order:

1. Is it local and reversible? **Decide it.** Log the decision under `ASSUMED:`
   with what breaks if it is wrong.
2. Is there an existing pattern in the codebase? **Follow it.** Log which.
3. Is it expensive to undo, or does it reach the server? Write it to
   `docs/questions-for-vishnu.md`, mark the item `BLOCKED`, **take the next
   item.**

A question is never a reason to stop working. It is a reason to move to the next
item.

## Sabotage — a guard test is not done until you have watched it fail

> **A test is not a control until you have watched it fail.**

A *guard test* is one whose job is to stop a mistake rather than to check a
feature: a lint, an invariant, a permission check, a "this must never happen"
assertion. Those are written precisely because nobody will notice when they
stop working — so a guard that has only ever been seen to pass is not evidence
of anything.

**The rule.** Before a guard test is reported done: break the thing it guards,
run it, and watch it go red. Then restore, and run it again. Both halves go in
the log.

**The sabotage must be at the level the test claims to protect.** Breaking
something nearby and watching a failure proves only that the test runs.

### The worked example — a near-miss, 19 Sep

`tests/survey.js` guards the three-state rule: every proof endpoint must report
yes, no *and* not asked, because folding "never asked" into "said no" inflates
every gain and the inflation is invisible.

The first sabotage removed `not_asked` from the initialiser that builds each
count block:

```js
const blank = () => ({ yes: 0, no: 0, answered: 0, of: 0, by_venue: {} });
```

**The test passed.** Four checks that exist to catch exactly this reported
green against code with the field deleted — because a later line,
`side.not_asked = cohort`, simply created the property again. The sabotage was
one level below what the test inspects: the test reads the *response*, and the
response was still correct.

The second sabotage stripped `not_asked` from the response itself. Four checks
went red, which is what the guard is for.

Had the first result been taken as proof, the repo would carry a test that
looks like a control, is cited as one, and does not work. That is worse than
no test, because it stops anyone looking again.

---

## Run to the end

Work the queue from the top until every item is `DONE`, `BLOCKED` or `FAILED`.
Do not pause for approval. Do not wait for a reply. Do not end the run because
an item is blocked — only because the queue is empty, or a hard stop from
`docs/unattended-operation.md` section 2 has fired.
