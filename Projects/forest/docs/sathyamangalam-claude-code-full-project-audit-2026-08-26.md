# Full project audit (site/, scripts/, exports/, docs — everything except harvest-engine/) — 26 Aug 2026

Run by Claude Code directly on the repo. This is the authoritative, code-verified audit — it corrects two things earlier document-based research got wrong, and finds real issues the harvest-engine-only audit couldn't see (since that one deliberately excluded site/).

## Corrections to earlier understanding

1. **The `claim` table is NOT empty.** It holds 82 real rows, and conflict detection correctly finds 3 conflicts from them. Earlier "empty differentiator" framing was accurate as of whenever it was checked, but is now stale — the data layer is built.
2. **The tiger dispute is NOT "12 vs 46."** The real pair is 112 (2024–25, TN Forest Dept) vs 8–10 (2009, old Management Plan). "Sacred groves" and "Western Ghats ESA boundary" as disputes don't exist anywhere in this project — grepped the whole repo, no hits.
3. **The real story: the data mostly exists. The web page that showed it was deliberately deleted on 17 Aug (commit `3aeacc5`), and the conflict-detection logic itself has a bug that would make it look wrong if restored as-is.**

## BLOCKING — fix before anything else matters

**B1 — Stub pages are telling Google they have content they don't have.**
All 7 "Under Construction" stub pages (`land`, `life`, `people`, `history`, `govern`, `visit`, `record`) carry full SEO tags AND FAQ structured-data markup describing the FINISHED page content (real facts like "824mm annual rainfall", "112 tigers, 651 elephants"). Google can show these as rich search results — a user clicks "How many tigers are in Sathyamangalam Tiger Reserve?" and lands on "Life is being built." This is exactly the pattern that gets a whole domain penalized for thin content, not just the bad pages — it can drag down the good pages (homepage, About) too.
**Fix:** in each of the 7 stub pages: change `<meta name="robots" content="index,follow,...">` to `noindex,follow`; delete the entire `<script type="application/ld+json">` block; replace the description with something honest like "This section is still being built." In `sitemap.xml`, remove the 7 URL entries for these pages, leaving only `/`, `/about`, `/contact`. Reverse each page's fix once real content is restored.

**B2 — Every navigation link except About and Contact is a dead end. 175 total links point into stub pages.**
The homepage's "Recognition" section has 4 polished cards and multiple call-to-action buttons ("See the Five Corridors", "See the Full Species Atlas", "Read the Full Record") — every single one leads to "is being built." There is no path from the homepage into real content today. 5 links also point at page-sections (#anchors) that don't even exist on the destination.
**Fix:** either restore the real content (see B3), or as an interim step, remove the dead links from index.html/about.html/404.html and the site nav, showing plain "coming soon" text instead of clickable dead ends.

**B3 — The site's headline feature ("we show disputed figures side by side, we never silently pick a winner") has no working page, but is still advertised on two live pages.**
The homepage says "3 disputed figures shown side by side, not silently resolved." About page says "See the live conflicts table on Sources & Data" — that link goes to a stub. The actual table and its code were deleted in the same 17 Aug commit that created all the stubs, but the code still exists in git history and can be recovered.
**Fix:** recover the old table's HTML/JS from git history (`git show a367949:site/record.html`), rebuild it into the current page style (not a wholesale branch checkout — the old branch also has OLDER, worse versions of About/Contact that would be a downgrade), fix the underlying data bug first (B4), then wire it up.

**B4 — The most important finding. The "3 disputed figures" are not disputes — they're the same measurement at two different points in time, mislabeled as contradictions.**
Tigers: 112 (measured 2024–25) vs 8–10 (measured 2009). Same for elephants and leopards — old count vs new count, not two sources disagreeing. This is because the code that finds "conflicts" groups values by subject and field, but ignores the date entirely — so a 15-year-old count and a fresh count of the same growing population look like a contradiction. In reality this is the reserve's celebrated tiger recovery story (they won an international TX2 Award for exactly this in 2022) — and the site would have been about to publish it as if it were a data error. **This would have been actively embarrassing if shipped as-is** — a journalist or forest officer would catch it immediately, and it directly undermines the site's core claim of being more rigorous than Wikipedia.
**Fix:** in `export_from_db.py`, add the date (`as_of`) into the grouping logic so only same-period values can be flagged as genuinely conflicting. This will correctly drop the count to zero real conflicts for now (honest, and the code already handles "no conflicts" gracefully). Separately, build a proper "how this number changed over time" view — that's the actually interesting, true story here, not a manufactured dispute.

## IMPORTANT — real gaps, fix soon

**I1 — The one genuine, real dispute that ever existed in this project was deleted from the record instead of being marked resolved.**
There was a real one: the reserve's core area was reported as both 793.49 km² and 917.27 km² by different sources, and it got correctly resolved using the primary 2013 government notification. But instead of keeping the old, wrong figure marked "superseded — see resolution," it was just deleted outright. This is the opposite of the site's own stated philosophy (track it openly, then show how it got resolved).
**Fix:** put the old 917.27 figure back into the data with a "superseded" marker pointing at what replaced it and why, so the site can show the actual, true best example of "we track disagreements until we resolve them" — currently it can't show this at all, because the evidence trail was deleted.

**I2 — A developer error message is shown live to real visitors on the homepage.**
If a background data-load fails, the homepage literally displays the text "(load failed, serve over http)" to visitors — a message meant for a developer, not the public. Documented as having already happened once in production.
**Fix:** one-line change to show a normal fallback like "—" instead.

**I3 — 13 of 14 published data files are never linked to from anywhere on the site.**
The site publicly promises its data is open and licensed for reuse, but 1.8MB of real data files are invisible — nothing links to them.
**Fix:** add a simple "Open Data" section to the About page linking directly to the actual files.

**I4 — The "Media Credits" link downloads a file instead of showing a readable page.** Clicking it just downloads raw text instead of showing a proper page — breaks the site's polish and makes photo credit attribution (a real legal/licensing obligation) hard to access.
**Fix:** convert it into a real page using the existing site style.

**I5 — Zero analytics of any kind.** No way to know if anyone visits, what they click, or whether any of this SEO work does anything.
**Fix:** turn on Cloudflare's free built-in analytics (no cookie banner needed, no extra privacy paperwork) — one setting plus one line of code per page.

**I6 — Two internal documents (README and CHANGELOG) are significantly out of date and actively describe a version of the site that no longer exists** — e.g. CHANGELOG's newest entry describes "11 finished pages and a working conflicts table," both now false. This matters because it's exactly the kind of document a future AI session or collaborator would trust at face value.
**Fix:** update both with what's actually true today; add the missing entries covering the 17 Aug stub-creation commit and later work.

**I7 — The About page brags "nobody hand-types this number to make it look better" right next to a number that IS hand-typed and already wrong** (31.5% written in the text vs the real live 31.6%).
**Fix:** remove the hardcoded number from that sentence, or make it pull the real number automatically like the homepage does.

**I8 — Personal, unrelated documents (career plans, money plans) are sitting in the project folder, not yet committed to git, but also not excluded from it — one routine `git add -A` away from accidentally being pushed permanently to your public GitHub.**
**Fix:** add that folder to `.gitignore` right now, before any other work, so this can't happen by accident.

## MINOR

- 11MB autoplay video with no preview image, no compression — slow and costly especially on Indian mobile data. Needs a poster image and re-encoding to ~2-3MB.
- 4.6MB of unused images left over from now-stubbed pages — fine to keep, just note in the credits file that those pages are temporarily offline.
- No caching/security header file — minor, easy one-time fix.
- One unescaped special character causes a (harmless but real) HTML validation error on one stub page.
- Several stat numbers are hardcoded directly into the homepage HTML instead of being pulled live — they currently match, but this is exactly the pattern that already caused I7's problem once.
- An old undocumented snapshot folder (`.baseline/`) exists from before the stub commit — genuinely useful as a record of what was there, just needs one line of documentation explaining what it is.

## What's confirmed actually fine — no action needed

Scripts (export, build, local dev server) are all complete, correct, well-built, no stubs, no duplication with harvest-engine. The build/deploy process correctly fails loudly rather than shipping a broken site — this already caught a real bug once. All 7 database exports match the live database exactly, no drift. SEO tag hygiene, mobile responsiveness, image alt-text, internal link integrity, the 404 page, and the contact form are all done properly. No lorem ipsum, no fake data anywhere — every real figure has a traceable source, and estimated map coordinates are honestly marked as estimates. The `places.html` redirect page is a deliberate, well-documented, correct design choice — leave it alone. The code comments throughout the project explaining editorial decisions (why certain figures are estimated, what got left out and why) are unusually thoughtful — this is a real asset of the project, not something to clean away.

## Recommended order (from the audit)

1. **First, before anything else:** I8 — add the personal-docs folder to `.gitignore`. One line, prevents an accidental permanent leak.
2. **To make the site safe to leave live as-is (~1 hour, no data work):** B1 (stop telling Google there's content that isn't there) → I2 (remove the visible error message) → B2 interim fix (remove dead-end links).
3. **To actually deliver the site's core promise:** B4 (fix the date bug so real conflicts are found, not fake ones) → I1 (restore the one real historical dispute, marked as resolved) → B3 (rebuild the conflicts table page using the fixed data).
4. **Then:** the remaining IMPORTANT items (I3–I7) and the MINOR items, in any order.

**One important caveat from the audit:** don't just copy pages wholesale from the old `dev` branch — it also contains OLDER, worse versions of About and Contact than what's live now. Recover just the body content of the stub pages from the specific old commit, and rebuild it inside the current, better page design — not a wholesale branch restore.

## Status
Audit complete and verified against real code and a running local copy of the site. This supersedes the earlier document-based understanding of the `claim` table and the "12 vs 46" tiger figure. Next step: action item 1 (the .gitignore fix) immediately, then work through the order above with the coding agent.
