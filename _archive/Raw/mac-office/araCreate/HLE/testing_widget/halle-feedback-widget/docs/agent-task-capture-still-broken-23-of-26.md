# Agent task — push two ready commits, then diagnose why capture still fails on 23 of 26 checks

## Part A — push what's already done and approved

Vishnu approved pushing these two already-committed, already-verified fixes:

- `31c8ac6` — fixed the capture-audit script and its real report
- `a204aac` — fixed the two known bugs in `deploy/runbook.md`

Run `git push origin dev`. Confirm it went through and report the result.
(A previous attempt to push from outside VS Code failed with a git-credential
error — this needs to run from inside your own environment, which has
working GitHub credentials.)

## Part B — diagnose, do not fix yet

`docs/agent-task-followup-21-sept-real-status.md` and the newer results
doc both describe this: the capture-audit tool was fixed and actually run
for the first time today (not the earlier fake report). Real result across
3 pages x 2 viewports x all scroll positions (26 screenfuls): **only 3
passed, 23 failed**, most by a double-digit percent-difference from what a
real browser shows.

This is despite two capture bugs already being fixed and confirmed live
today: the hero-pseudo-element background bug (`0965f27`) and the
scroll-offset misframing bug (`6d73f2c`, `f36c146`, using a
`transform`-based shift). Vishnu tested the scroll fix live himself and it
looked right — but that was one flow at a few depths, not this full
26-screenful sweep.

**Your job for this part: find out why so many screens still fail. Do not
attempt a fix yet — report back with a clear diagnosis first**, the same
way `claude/SESSION-RECORD-21-sept-capture-engine.md` and
`claude/research-scroll-fix-restoreScrollPosition.md` (in the Claude
project, ask if you need them mirrored into the repo) diagnosed the
earlier two bugs before touching code.

Suggested approach:

1. Look at `audit-report.md` and the saved side-by-side images from the
   real run (committed in `31c8ac6`) — what do the 23 failing screenfuls
   actually look like? Wrong content entirely, partially blank, misaligned,
   wrong colors/fonts, something else? Group the failures by what's
   actually wrong, not just "different."
2. Check whether the failures are scroll-position related still (i.e. the
   `f36c146` fix didn't fully solve it), or a new/different class of
   problem (e.g. something specific to `/about-us` or `/contact` that
   wasn't present on `/`, since the earlier hero bug was only found on the
   Home page).
3. Check whether this is related to the widget's recent migration to React
   + shadcn/ui (commits `aea41a3`..`f36c146` today) — i.e. did the rebuild
   change anything about how the capture clone is built or mounted, even
   though the capture logic itself was meant to be left untouched
   (`docs/agent-task-shadcn-widget-migration.md` says the capture pipeline
   was out of scope for that migration — verify that held).
4. Report: a clear breakdown of what's actually wrong, grouped by cause,
   with evidence (specific screenfuls, pixel/visual differences) for each
   group — not a guess.

Do not commit or push anything for Part B — diagnosis only, no code
changes.
