# Agent task — fix html2canvas's button-ring issue, then check it across the whole site

## Background

`docs/agent-task-tune-in-browser-capture.md` and its results found that
switching the capture library from `modern-screenshot` to `html2canvas`
fixes the known text/element positioning bug (open, unfixed upstream issue
in `modern-screenshot`, #104) and is about 2x faster (93ms vs 191ms,
measured on this machine). One new, smaller defect was found: html2canvas
draws a faint grey ring/outline around some buttons (seen on the "Send
Mail" buttons) that the real browser doesn't show.

Vishnu's decision: fix that button issue, then properly check html2canvas
across the whole site (not just the pages already tested) before deciding
whether to actually switch the widget over to it.

## What to do

1. **Fix the button-ring artifact.** Likely cause: a default
   `box-shadow`/`outline`/focus-ring style html2canvas applies to buttons
   that the real page suppresses via its own CSS but html2canvas isn't
   picking up correctly, or a default browser UA style html2canvas paints
   that the clone doesn't override. Investigate and fix properly — don't
   just force `box-shadow: none !important` blindly without confirming
   that's actually the correct root cause and doesn't hide a real style
   the site does want on buttons in some state (e.g. a real focus ring
   for keyboard users, or a real hover/active style).
2. **Check html2canvas across the full site**, not just `/contact` and
   Home — every page in the site's page list (see `docs/widget-v2-
   spec.md` / the admin's Pages list for the real 49-page inventory this
   project uses, or at minimum the 3 pages the existing audit tool already
   covers, extended to any page with buttons, icons, gradients, or
   masked/pseudo-element backgrounds, since those are exactly the CSS
   features `capture.ts` already had to special-case for modern-screenshot
   — html2canvas may handle them differently again).
3. Run the existing `scripts/audit-capture.mjs` (or a version of it
   pointed at an html2canvas-based capture path) across all 16 screens
   (desktop + mobile, all scroll depths) and report the real pass/fail
   numbers, same format as before.
4. Check bundle size impact — `capture.js` is currently a defined, tracked
   size (see `docs/decision-admin-shadcn-rebuild.md`'s size discussion for
   how seriously this project tracks widget size). Report html2canvas's
   real size cost, gzipped, compared to `modern-screenshot`'s.

## What NOT to do yet

- Do not make html2canvas the widget's actual default capture method.
  This task is "test it properly," not "ship it." Keep the change behind
  a way to compare both (a flag, a separate test build, whatever's
  simplest) rather than replacing `capture.ts`'s real behavior outright.
- Do not commit the swap as the new default — commit test/diagnostic code
  if useful, but the actual capture path stays on `modern-screenshot`
  until Vishnu says to switch it.
- Do not push anything without being asked.

## Report back

- Confirmation the button-ring issue is actually fixed, with the real root
  cause explained (not just "added a CSS override").
- Full-site pass/fail numbers with html2canvas, same audit format as
  before.
- Any new quirks found on pages not tested yet.
- Real bundle-size number for html2canvas vs the current library.
- A clear final recommendation: is this actually ready to become the
  widget's real capture method, or does it need more work first.
