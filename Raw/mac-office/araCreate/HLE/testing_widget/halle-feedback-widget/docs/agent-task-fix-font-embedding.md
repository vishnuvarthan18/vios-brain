# Agent task — turn web-font embedding back on with the correct font, measure the cost

## Background

`src/widget/src/capture.ts` currently has `EMBED_WEB_FONTS = false` (set
21 Sept), on the assumption the site's text uses Helvetica Neue, already
installed on every Mac/iOS device. That assumption is wrong: the live site
actually declares `font-family: Inter`, a web font, confirmed by fetching
the real page. `Inter` is not installed on this machine, not bundled with
Chromium, and likely missing on many real testers' machines too — so the
capture currently substitutes a different font, which reflows the whole
page below it. This was diagnosed today as the main cause of 11+ of the 23
failures found in the latest capture audit (`docs/agent-task-capture-
still-broken-23-of-26.md` and its diagnosis, in the Claude project as
`claude/agent-task-capture-still-broken-diagnosis.md`).

Vishnu's decision: fix it properly — turn font embedding back on with the
correct font, and measure what it actually costs in speed before anyone
decides whether that cost is acceptable. Report the number; do not
silently accept or reject it.

## What to do

1. Turn `EMBED_WEB_FONTS` back on (or however the code needs to change to
   correctly embed `Inter` specifically — check whether the flag is a
   simple on/off or needs the correct font family name wired in).
2. Test against the real live site, not a synthetic fixture — same rule
   this project has used throughout (see `claude/live-bug-loading-blank-
   images.md`'s "method notes" section for why fixtures have been
   misleading before).
3. Measure real capture speed with the fix, same way it's been measured
   before in this project (the 500ms budget is the reference point —
   `docs/agent-task-screenshot-speed-500ms.md`). Report the real number,
   don't estimate it.
4. Re-run `scripts/audit-capture.mjs` across all 26 screenfuls (3 pages x
   2 viewports x every scroll depth) to confirm this actually fixes the
   font-related failures. Report the new pass/fail count.
5. Leave the carousel-timing issue on the Home page alone for now (the 12
   failures caused by the audit's own screenshot and the widget's capture
   catching two different carousel slides) — that's a test-methodology
   question, not a capture bug, and is out of scope for this task unless
   it's trivial to note in the report.

## Commit, don't push

Commit this as its own change with a clear message. **Do not push** —
Vishnu wants to see the real speed number before this goes live, same as
every other capture-speed decision in this project so far.

## Report back

- The real measured capture speed, before and after.
- The new audit pass/fail count (was 3 passed / 23 failed).
- Whether the font-related failures are actually resolved, with evidence
  (specific screenfuls that now pass that didn't before).
- Anything unexpected found along the way.
