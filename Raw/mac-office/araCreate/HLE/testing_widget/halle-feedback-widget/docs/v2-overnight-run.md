# V2 OVERNIGHT RUN

**This is an unattended run.** No one is available to answer a question
until morning. This document is the run brief — how to move through the
night without stopping. `v2-build-plan-for-agent.md` is the technical
content for what to build; this document is how to build it without a human
in the loop.

Same shape as the M0–M3 overnight run this project has already done once
successfully — this document plays the role `docs/overnight-run.md` played
then. Read `docs/agent-rules.md` (12 nevers, 7 always) and
`docs/quality-gate.md` (the mechanical standard) in full before starting;
both still apply, unchanged, for the whole night.

**Read `v2-build-plan-for-agent.md §0.1` before anything else.** An earlier
draft of that plan claimed screenshot capture and storage were not built.
They are. That correction is the difference between adapting working,
privacy-critical code and rewriting it in the dark.

---

## 1 What "non-stop" means here, precisely

- Run **M7 → M8 → M9 → M10**, in that order, in one continuous pass. Do not
  stop and wait between them — there is no one to wait for. (M6a and M6b are
  already built and committed; the numbering starts at M7 for that reason.)
- Every milestone still needs to pass its own tests and its own quality-gate
  pass (§4) before the next one starts. "Non-stop" means no waiting for a
  human, not skipping verification. A milestone that hasn't earned its own
  green tests is not a foundation for the next one.
- **Commit at the end of each milestone**, once its tests and quality-gate
  pass are green. Starting this run is Vishnu's explicit authorization for
  these four commits — the same standing rule ("committing needs Vishnu's
  own explicit instruction at the time") is satisfied by that authorization,
  the same way it was for the M0–M3 run. No `Co-Authored-By` trailer, ever,
  on any of them.
- **Stage by path, never wholesale.** There is pre-existing staged work in
  the index that is not part of this run
  (`v2-build-plan-for-agent.md §0.1`, §7). Every commit stages only the
  exact files that milestone touched. No `git add -A`, no `git add .`, no
  `git commit -a`.
- **Never push to any remote.** Nothing in this project has a remote yet.
- One commit per milestone, four commits total by morning if all four
  complete. Do not squash them together and do not split one milestone
  across multiple commits.
- **Code only.** The only files in `docs/` this run may touch are
  `docs/v2-blocked.md` and `docs/v2-overnight-log.md`. Every other document
  is read-only tonight, including the ones that turn out to be wrong.

## 2 When something is genuinely not answered by the plan

`v2-build-plan-for-agent.md §1` pre-answers the decisions most likely to
block this build. If something still comes up that isn't covered there:

1. **First, re-read the plan.** Most things that feel unanswered are
   answered somewhere in it — §0.1's corrections, §1's table, or the
   relevant milestone section.
2. **If the plan and the repo disagree about what already exists, the repo
   wins.** Log the disagreement in `docs/v2-blocked.md` and build against
   what is actually there. This already happened once with the screenshot
   milestones; assume it can happen again.
3. If it's truly not covered: make the most **reversible, conservative**
   choice available — the one that's cheapest to undo or override later,
   not the one that seems most feature-complete. Write down what you chose
   and why in `docs/v2-blocked.md` (create it if it doesn't exist, same
   shape as `docs/blocked.md` from the last run — one dated entry per
   judgement call). Then **keep going.**
4. The only things that should actually stop the run before all four
   milestones are attempted:
   - A migration that would destroy data that might matter. The plan says
     the local database is disposable seed data — verify that by checking
     for anything that looks like a real submitted report before running a
     destructive M7 migration; if you find something real, stop and log it.
   - A later milestone turning out to depend on something the repo doesn't
     have, in a way no reversible workaround covers.
   - Needing information this project has repeatedly refused to let an
     agent invent — the real 49 page URLs, principally. If a task seems to
     require inventing one, stub it (a clearly-marked placeholder, not a
     guessed real address) and log it. Restated because M7's template
     backfill touches every page row.
   - Anything that would require deleting, weakening or rewriting the
     privacy stripping in `capture.ts`. That is not a judgement call an
     unattended run gets to make. Stop and log it.
   If a stop condition is hit, finish and commit whatever milestone was in
   progress if it's in a genuinely green state; do not leave a half-built
   milestone committed. Log clearly in `docs/v2-blocked.md` and in the final
   log entry (§3) exactly where the run stopped and why.

## 3 Logging

Keep `docs/v2-overnight-log.md` — timestamped entries, same shape as
`docs/overnight-log.md` from the last run. One entry per milestone at
minimum: start time, what was done, test results, quality-gate result,
commit hash. Add extra entries for anything logged to `docs/v2-blocked.md`,
at the time it happened, not reconstructed at the end.

End the log with a plain summary for the morning: which milestones
completed, test counts, what's in `docs/v2-blocked.md` (if anything), and
what would be next if the run stopped early.

## 4 The quality gate, applied to this run specifically

`docs/quality-gate.md`'s standard applies to all four milestones, same as
M0–M6b: break something on purpose in the new code and prove the tests
actually catch it, not just that they pass. Three places are worth being
specifically suspicious of:

- **The privacy stripping in `capture.ts`** — which this run *modifies
  around* rather than writes. It is now the only protection left, because
  the tester's option to decline the picture has been removed
  (`widget-v2-spec.md §11`). Disable `strip_clone()` deliberately and prove
  the existing privacy test goes red. If it stays green, the test is the
  bug — fix the test, and say so in the log.
- **The authenticated image route.** Prove a request without a valid
  session is actually rejected, not just that a request with one succeeds —
  the same class of gap the M3 quality gate caught before (an unscoped
  query returning another tenant's data under concurrency) was a "the happy
  path works" bug that only a deliberately adversarial test surfaced.
- **The failed-capture path.** Force the capture to return null and prove
  the report still sends, with a null `screenshot_key` and nothing shown to
  the tester. A report that silently dies because its picture failed would
  be invisible in testing and fatal in a real round.

## 5 What Vishnu will do in the morning

Same as every previous milestone: **the report will not be trusted at face
value.** The repo gets inspected directly — migrations applied to a fresh
database, the widget clicked through by hand, the admin app logged into,
`docs/v2-blocked.md` read in full — before anything from tonight is treated
as done. This is not a comment on the agent; it's the same process every
milestone so far has gone through, including the ones that turned out to
have real gaps a summary alone wouldn't have shown.

## 6 Start-of-run checklist

- [ ] `docs/agent-rules.md` and `docs/quality-gate.md` read in full.
- [ ] `v2-build-plan-for-agent.md` read in full — **§0.1 first**, then §1's
      pre-answered decisions.
- [ ] `widget-v2-spec.md` and `admin-v2-spec.md` read in full — the build
      plan references both throughout rather than repeating them.
- [ ] The four files listed in `v2-build-plan-for-agent.md §4.1` read before
      any picture-path code is written.
- [ ] `git status` recorded in the log, so the pre-existing staged work is
      on record as it was before the run started.
- [ ] Confirmed the local database has no real submitted reports before
      M7's migration runs (§2, first stop condition).
- [ ] `docs/v2-overnight-log.md` created with a start-time entry before M7
      begins.
