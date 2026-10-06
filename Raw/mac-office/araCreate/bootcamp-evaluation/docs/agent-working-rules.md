# Running agents on a live app — what worked, 17–19 Sep 2026

Written after building and shipping v2 onto a live bootcamp with 209 students, using
two coding agents over about 36 hours. Every rule here was learned by something going
wrong, or nearly going wrong. Keep it with the repo docs.

## The shape that worked

Two agents, called Lane A and Lane B. One person (Vishnu) running the rooms, one
assistant reading every report and writing the next prompt.

- **One step per prompt. Never two.** Every time the scope widened mid-prompt, it cost
  more than it saved.
- **Report, then act.** Each agent reported before deploying, and the report was read
  before the next step was authorised.
- **The agent proposes, the reviewer decides.** Both agents refused instructions that
  would have caused damage — a `checkout -B` that would have orphaned a commit, an
  impossible test plan, an `app.js` edit with unclear ownership. Every refusal was
  correct. Build agents that push back and then listen when they do.

## Git rules, all learned from incidents

- **One worktree per lane, always.** Two agents in one working tree caused an
  uncommitted edit to reach production and a commit to land on `main` instead of a
  feature branch.
- **No `git stash`, ever.** Worktrees share one stash stack. It swallowed another
  lane's work once and silently committed staged changes another time.
- **Commit and push after every step.** Twelve commits sat on one laptop for a full
  night. Push was raised four times before it happened.
- **Never `reset --hard` without rescuing uncommitted work first.** A cleanup nearly
  destroyed four half-finished doc files belonging to someone else.
- **Declare file ownership before parallel work**, and re-declare it whenever a job
  moves between lanes.

## Deploy rules

- **Build the payload with `git archive` of an explicit commit.** Never `rsync` from a
  working tree — `rsync` ships whatever is on disk, including another agent's
  uncommitted edits and stale worktrees.
- **Exclude runtime data from `--delete`:** `uploads`, `.archives`, `logs`,
  `node_modules`. The uploads directory held student CVs that are in no database dump,
  and one exclude list would have deleted the pre-deploy backup itself.
- **Reassign object ownership immediately after migrations, in the same step as the
  restart.** Migrations run as `postgres` leave objects the app user cannot read. A
  2m 40s gap between the fix and the restart was a live outage.
- **Take a fresh dump immediately before deploying**, and verify it *contains the rows
  you expect* — not merely that it is valid gzip.
- **Watch the log across the restart**, not just that the service came back up. The one
  outage was found this way and missed the time it was not done.
- **Do not deploy while people are using it.** The one mid-morning deploy cost 19
  seconds of downtime and a student's photo. Every other deploy was in a quiet window
  and cost nothing.

## Testing rules

- **Prove, do not assert.** "Verified against the database, not the script's output"
  caught a false all-clear that would have let a deletion job run against copies that
  were never made.
- **Go through a real session.** 41 checks passed while every team lead was locked out
  of their own work, because no test ever signed anyone in.
- **A passing suite can be a false green.** One suite went green on its own; the
  question "why did that change" was worth more than the green.
- **When a test fails, ask whether the test is wrong.** Twice, a failing assertion was
  the test's fault, and "fixing" the code would have broken working behaviour. Print
  the actual response instead of assuming.
- **Test with real data volumes.** Everything was tested with three students until
  someone ran 209 concurrent. The app held; the assumption had not been checked.

## The pattern worth internalising

**Eight serious bugs were found in three days. The new code was correct every time.
The bug was always in the code underneath it.**

Old routes still open. Old booleans still read. Old screens never rebuilt after their
backend changed. A field renamed in one place and read by its old name in another.

So the standing instruction to any agent finishing a feature is: *before you report,
go and look at what the old path still does.* That instruction found the two worst
bugs of the whole build — a quiz screen locked to 156 of 209 students, and a
department gate that leaked one venue's work to the other.

## Handling secrets

- **Never paste a private key, password or token into a chat.** One was, and had to be
  rotated the next morning.
- Agents should be told explicitly not to ask for credentials. Both did ask once;
  after the rule was set, both proposed workarounds instead — a human clicking the
  admin UI, or a shared non-secret login code.
- A password passed inline on a shell command lands in shell history and the process
  list. Read it from a file.

## Working with someone who is running the room

- The person running a live event has minutes, not hours. Lead with the decision they
  need to make, not the reasoning.
- Give them the exact clicks when they must do something themselves.
- Tell them plainly when something is their job and no agent can do it — writing quiz
  questions, chasing a room that never signed in.
- When they overrule a recommendation, say the risk once, then help them do it well.
