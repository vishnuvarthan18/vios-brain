# Session close — 26 Aug 2026 (tech audit + overnight fix run)

Read this first in the next chat. Covers the technical audit work done today and the overnight fix Vishnu is about to run with Claude Code. Separate from (and later than) the earlier `session-close-2026-08-26.md`, which covers the life-direction research thread.

## What happened this session, in order

1. Did a full technical audit of the harvest-engine (all 29 data streams, database setup, error handling, rate limiting, security, idempotency) by reading the actual code on Vishnu's Mac via the device bridge — saved as `sathyamangalam/harvest-engine-full-audit-2026-08-26.md`. Found and diagnosed the exact root cause of the gazetteer under-querying bug (query only asks for villages/hamlets/towns, missing hills/water/temples/forest tags) plus 22 other findings, ranked BLOCKING/IMPORTANT/MINOR.
2. Audited git/branch/dev-prod setup and dead code — found 3 branches exist but only `main` is actually used, zero dev/production separation exists anywhere (testing touches the real database and real site directly), no CI/CD, and one real bloat item (a 16MB PDF committed into the repo). Folded into the same audit doc as findings #17-23.
3. Wrote and delivered `sathyamangalam/coding-agent-instructions-2026-08-26.md` — a 10-step, copy-paste-ready instruction set for a coding agent, in safe order (dev branch setup first, then fixes tested on staging, then merge).
4. Vishnu ran a SEPARATE full-project audit using Claude Code directly on the repo (covering site/, scripts/, exports/, docs — everything the harvest-engine audit deliberately excluded). This came back and is saved verbatim (with our plain-English translation) as `sathyamangalam/claude-code-full-project-audit-2026-08-26.md`. This is the more authoritative audit since it was run directly against live code and a running local copy of the site, not read-only inspection.

## CORRECTIONS from the Claude Code audit — supersede earlier understanding

- **The `claim` table is NOT empty.** 82 real rows, conflict detection correctly finds 3 (before the date-bug fix). Earlier "empty differentiator" framing is stale.
- **The tiger dispute is NOT "12 vs 46."** Real pair: 112 (2024-25, TN Forest Dept) vs 8-10 (2009, old Management Plan) — same population, different years, not two disagreeing sources.
- **"Sacred groves" and "Western Ghats ESA boundary" as documented disputes don't exist anywhere in the actual project.** Grepped the whole repo — no hits. Don't reference these as real disputes going forward.

## THE most important finding from the whole audit process

The "3 disputed figures" the site was about to publish (tigers, elephants, leopards — old count vs new count) are **not real disputes at all** — they're the same growing population measured at two different points in time, mislabeled as a contradiction because the conflict-detection code doesn't check dates. This is literally the reserve's celebrated tiger-recovery story (TX2 Award, 2022) that the homepage already brags about elsewhere. If shipped as-is, this would have been a real credibility problem — a journalist or forest officer would have caught "Tigers: 112 vs 8-10" under a "disputed figures" heading immediately, undermining the site's entire rigor-focused pitch. Caught before publishing. Fix: add the date into the conflict-grouping logic (this drops real conflicts to 0, which is correct and honest).

## Also found: real content for 7 "Under Construction" pages was NOT lost

`land`, `life`, `history`, `record`, `govern`, `people`, `visit` all show placeholder "Under Construction" text on the live site, but their real, complete content was deliberately set aside in commit `3aeacc5` (17 Aug) with an explicit plan to bring it back page-by-page — a plan that was never followed through. The content is fully recoverable from git commit `a367949` (the `dev` branch snapshot). Caveat: `dev` also has OLDER/worse versions of about.html and contact.html than current `main` — restore only the body content of the 7 stub pages into the CURRENT page design, don't checkout the whole branch.

## Also found, urgent: personal documents at risk of accidental public leak

`docs/claude-project/` (containing Vishnu's personal career/money planning docs — government-scientist-master-plan.md, scientist-money-plan.md, etc.) sits in the repo, untracked, but NOT in `.gitignore`. One routine `git add -A` would push these permanently to the public GitHub repo. This is Step 0 of the overnight fix — must happen before anything else.

## The overnight fix plan

Wrote and delivered `sathyamangalam/overnight-full-fix-2026-08-26.md` — a single 13-step file combining fixes from BOTH audits (harvest-engine + full-site), in dependency order, meant to be handed to Claude Code in one sitting for an unattended overnight run. Structure:

- Step 0: `.gitignore` fix for the personal-docs leak (urgent, first)
- Step 1: set up `dev` branch + Cloudflare Pages preview deploys (safety net before any other change)
- Step 2: staging D1 database for the harvest engine
- Step 3: fix the gazetteer query bug (tested on staging first)
- Step 4: build the missing D1 → atlas.db sync bridge (tested on staging first)
- Step 5: fix the date-blind conflict-detection bug (the tiger 112-vs-8-10 issue)
- Step 6: restore the 7 real pages' content + rebuild the disputed-figures table on record.html
- Step 7: fix SEO/noindex on any pages still unfinished after Step 6
- Step 8: fix dead-end navigation links
- Step 9: remove the visible debug error text on the homepage
- Step 10: re-insert the one real historical dispute (core area 793.49 vs 917.27 km²) marked as resolved/superseded, not deleted
- Step 11: batch of small fixes (credits page, stale hardcoded number, open-data links, video poster image, security headers, etc.)
- Step 12: update README.md and CHANGELOG.md to match reality
- Step 13: final review — explicitly does NOT auto-merge `dev` into `main`; stops and reports what's done vs deferred

Includes an explicit "if you hit a usage limit or get cut off" section: finish or fully revert the current step, never leave `main` touched, commit progress on `dev` with a clear note of what's done/in-progress/not-started, write a status note, and resume from there rather than restarting — added specifically because Vishnu was worried about Claude Code running out of usage mid-run overnight.

**Explicitly deferred to Vishnu personally, not attempted by the coding agent:**
- Getting the 3 missing API keys (BHL, WDPA, data.gov.in) — needs his own sign-ups
- Deciding override-vs-exclude for the permanently robots.txt-blocked `wii` stream
- Turning on Cloudflare Web Analytics — one dashboard click only he can do
- Re-encoding the 11MB hero video — flagged, not attempted unsupervised
- The full visual "history over time" UI for the tiger-recovery story — data structure only tonight
- Final decision to merge `dev` into `main` and go live

## Rough expected outcome, told to Vishnu

~80-85% of all identified issues (both audits combined) should be fixed by morning if the run completes cleanly. The ~15-20% left is deliberately gated on Vishnu's own action (API sign-ups, one decision, one dashboard click, final go-live approval) — not because those are hard, but because they need him specifically. Flagged honestly that Step 6 (restoring 7 real pages) is the biggest, most judgment-heavy step and might not fully complete for all 7 pages in one night — the file instructs the agent to report accurately rather than claim false completeness.

## Status as of this session close

Overnight fix file has been handed to Vishnu to paste/point Claude Code at. **Not yet run as of this close.** Next chat should:
1. Ask Vishnu how the overnight run went — did it complete, get cut off, or hit issues.
2. If it produced a status/summary, review it against the 13 steps above to confirm what's actually done vs deferred.
3. If cut off partway, check for the status note it was instructed to write, and help resume from there rather than restarting.
4. Do NOT re-run any step already confirmed complete.
5. Do NOT merge `dev` to `main` without Vishnu explicitly looking at the preview and approving — Step 13 is designed to stop and wait for this.

## Files to re-read first in the next chat, in this order

1. This file (session close)
2. `sathyamangalam/overnight-full-fix-2026-08-26.md` — the plan that was run
3. Whatever status/summary Vishnu brings back from Claude Code
4. `sathyamangalam/claude-code-full-project-audit-2026-08-26.md` — full findings list, to check off against
5. `sathyamangalam/harvest-engine-full-audit-2026-08-26.md` — full findings list, to check off against
