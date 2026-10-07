# Agent task — URGENT: roll the live server back to the previous version

22 September 2026. Vishnu tested the last deploy live and reports it made
things WORSE, not better: the pointer box is now wrong in every case
(previously wrong in some), and he also reports the SCREENSHOT IMAGES
THEMSELVES are now worse too — a new, separate complaint from the box
issue, not yet diagnosed. He is frustrated and wants this off the live
site now, correctly — stop making changes and get back to the last known
working state first.

**Do this now, nothing else. No investigation, no fixing-in-place, no
"while I'm here" changes. Roll back, verify, stop.**

## What to roll back

The live server currently runs commit `38aa6ba` (this round: `8c6b185`,
`e247408`, `38aa6ba`). Roll the deployed APP back to `0f01492` — the
commit that was live and working before this round.

## What NOT to roll back

**Leave the database migration in place.** `reports.capture_method` is a
new, nullable, additive column — the old app code simply never writes to
it. Rolling the schema back is unnecessary and adds risk for no benefit.
Do not run any migration in reverse.

## Steps

1. Health check first: memory, both services, current git SHA (should be
   `38aa6ba`).
2. On the server, get the app code back to `0f01492` — a `git checkout`
   to that commit (detached HEAD is fine, this is a rollback not a new
   branch) or equivalent, then rebuild the widget and web app from that
   checkout exactly as the last two deploys did, and restart BOTH
   `halle-feedback` and `halle-feedback-hybrid-render` (both were touched
   by the deploy being undone).
3. Health check after: memory, both services up, git SHA now `0f01492`,
   `/health` on the renderer OK.
4. Submit one real test report against the live site to confirm the
   ROLLED-BACK version behaves the way it did before this round (box
   sometimes right, sometimes not — that was the known, accepted state
   before this round, not what we're chasing right now).
5. Report back briefly: confirmation the rollback is live and healthy.
   That is all — do not start diagnosing the new "images are worse" report
   in this task. That is separate, upcoming work once Vishnu has sent
   examples and everyone has taken a breath.

## Do not do

- Do not attempt to fix the box/letterbox bug in this task.
- Do not touch the database or its migrations.
- Do not install anything or change any safety limits.
