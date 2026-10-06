# Tamil Data Collector ("semmozhi") — Status

Last updated: 2026-09-05

## Project purpose
Collect Tamil language/culture/history data (literature, dynasties, history, culture) comprehensively and **accurately** — explicitly NOT to "prove" Tamil is the "first" or "best" language/culture. Hard constraint on tone/scope for any future work.

## Architecture
- **Engine:** Scrapy · **Execution:** GitHub Actions, free tier, scheduled cron · **Storage:** raw `.jsonl` in repo's `data/` folder, no external DB
- **Repo:** private, GitHub user `vishnuvarthan18`, local working copy at `~/Downloads/tamil_harvest/`
- **Dashboard:** separate PUBLIC repo `tamil-data-dashboard`, GitHub Pages: https://vishnuvarthan18.github.io/tamil-data-dashboard/ — user's own parallel effort with the other AI agent.

## CRITICAL workflow rule
User has a **separate AI coding agent** that writes/edits all code. Claude (this chat) writes PROMPTS for that agent, then reviews/verifies its reported output against real numbers before trusting it. Carries forward into any new session.

## Verification discipline
Never trust "it works" — demand real numbers and spot-check actual content. The agent has no memory across its own invocations — always give exact file paths. The agent has repeatedly caught and corrected Claude's own mistakes this session — treat its pushback as signal. Also learned: GitHub re-running an old workflow run replays its ORIGINAL definition, not the current one on main — always trigger a fresh `workflow_dispatch` run rather than "Re-run" an old one when testing a just-merged workflow change.

## STATUS: CORE ENGINE IS DONE AND VERIFIED LIVE — commit `fa7be1a`. Day 7-8/15 check-in (2026-09-05): HEALTHY, no action needed.

### Day 7-8 progress check (2026-09-05)
- **Repo size:** `.git` = 5.3MB. On track for the expected 100-170MB by day 15 — no GitHub free-tier size concerns.
- **Data volume:** 96 files in `data/`.
- **Scheduled runs:** last 5 scheduled Actions runs (2026-09-04 to 2026-09-05) all `completed`/`success`, runtimes 25m–37m. No failures.
- **Content spot-check:** `wikipedia_tamil_categories_20260828T045844Z.jsonl` — 385 records, real full-paragraph extracts (not empty; only 1 record had an empty extract). Content is in English by design (this spider = English Wikipedia articles about Tamil topics; the separate `tamil_wikipedia_categories` spider is the one that pulls Tamil-script content from ta.wikipedia.org — see naming convention below). Confirmed this is expected behavior, not a bug.
- **Conclusion:** everything green. No fixes needed this check-in. Next full review/next-phase planning conversation is day 15.

All 40 spiders, the two-job crawl.yml split, and all bug fixes are committed to `main` AND have been proven to work end-to-end:

**Run #11** (manual `workflow_dispatch`, `group: both`, triggered 12:21 PM, commit `fa7be1a`): **SUCCESS**. Both jobs green — `wikimedia` (37m 37s) and `other-sources` (24m 48s), total 1h 2m 32s. This is the first-ever real execution of all 40 spiders together. Only cosmetic warnings (Node.js 20 deprecation notice on GitHub's runners — unrelated to spiders/data, safe to ignore).

**Scheduled crons are live and firing correctly**: Job A (wikimedia) `23 0,12 * * *` UTC, Job B (other-sources) `23 6,18 * * *` UTC — same file, same proven-working definition as run #11. Confirmed still succeeding as of the day 7-8 check-in above.

## KNOWN OPEN ISSUE (non-blocking): dashboard not reflecting run #11's data
The public dashboard (https://vishnuvarthan18.github.io/tamil-data-dashboard/) still showed stale data from an earlier run ("run #10", 27 sources, 23,963 records) even after run #11 succeeded and a new commit landed in the `tamil-data-dashboard` repo, as of last check (2026-08-28). Not rechecked during the 2026-09-05 day 7-8 check-in (user didn't report on it this time — worth asking about at day 15 if not resolved sooner). Checked so far:
- Hard refresh (Cmd+Shift+R) — no change.
- Cache-busting query string (`?v=2`) — no change.
- A new commit DID land in `tamil-data-dashboard` after run #11 (confirmed by user) — **not yet confirmed** whether that commit's timestamp/message actually corresponds to run #11 specifically, or whether it's an intermediate/different commit.
- NOT yet checked: the `tamil-data-dashboard` repo's own Pages deployment status (Settings → Pages), which would show whether GitHub Pages actually redeployed from that commit.
**Decision: deferred by user ("fix after some days") — explicitly NOT blocking, since this is a display/publish-layer issue only.** The underlying `.jsonl` data files in the `tamil-data-collector` repo's `data/` folder are correct and already updated regardless of what the dashboard shows.
**To resume when picked back up:** check the exact commit message/timestamp in `tamil-data-dashboard`, check Settings → Pages deployment status/history there, and check the "Publish dashboard" step's actual log output inside run #11 (or the next scheduled run) for silent failures.

## Two remaining loose ends (not blocking, both minor — separate from the dashboard issue above)
- `project_madurai_texts.py` and `wikipedia_tamil_categories.py` — last known state had pre-existing uncommitted local modifications (MAX_LINKS_PER_RUN=150 decision + throttle pin). Should double check these landed correctly; not reconfirmed since the fa7be1a push. `wikipedia_tamil_categories` spot-checked healthy on 2026-09-05 (see above) but the specific MAX_LINKS_PER_RUN setting itself wasn't directly inspected.
- `scratchpad/gen_crawl.py` — untracked, now covered by `.gitignore`.

## Two near-identical spider name pairs (confirmed real, not typos)
Naming convention: `tamil_X` = the Tamil-language wiki, `X_tamil` = the English wiki about Tamil topics.
- `tamil_wikipedia_categories` (ta.wikipedia.org) vs `wikipedia_tamil_categories` (en.wikipedia.org) — both in Job A
- `tamil_wikipedia_texts` (ta.wikipedia.org) vs `tamil_wikisource_texts` (ta.wikisource.org) — both in Job A

## crawl.yml design decisions (approved and verified working this session)
- Two jobs split by **domain contention, not runtime** — Wikimedia-family spiders rate-limit each other if run together.
- **Job A "wikimedia" (19 spiders)**, cron `23 0,12 * * *` UTC: tamil_wikisource, tamil_wikisource_texts, tamil_wikisource_classics, tamil_wikipedia_stats, tamil_wikipedia_texts, tamil_wikipedia_categories, wikipedia_tamil_articles, wikipedia_tamil_categories, wikipedia_tamil_multilingual, wikidata_tamil_entities, wikidata_tamil_works, wikimedia_commons_tamil, wikivoyage_tamil_nadu, english_wikisource_tamil, tamil_wiktionary, wiktionary_tamil_etymology, tamil_wikiquote, tamil_wikibooks, tamil_wikinews
- **Job B "other-sources" (21 spiders)**, cron `23 6,18 * * *` UTC: project_madurai, project_madurai_texts, internet_archive_tamil, internet_archive_tamil_language, internet_archive_tamil_collections, internet_archive_tamil_fulltext, thevaaram_thirumurai, openlibrary_tamil, openalex_tamil_studies, crossref_tamil_studies, arxiv_tamil_nlp, doaj_tamil, zenodo_tamil, huggingface_tamil_datasets, met_museum_tamil, cleveland_museum_tamil, art_institute_chicago_tamil, overpass_tamil_heritage, tamil_nlp_catalog, ai4bharat_catalog, mozilla_data_collective
- `workflow_dispatch` input `group: [both, wikimedia, other]` — confirmed working correctly (used for run #11).
- `fetch-depth: 0` on both checkouts — required for `git pull --rebase` to have a reliable merge base.
- **One shared concurrency group** — both jobs write to `data/` and force-push the public dashboard repo; serializes A then B on a manual `both` run (confirmed: run #11 ran wikimedia then other-sources sequentially, matching design).
- Commit messages carry a job label.
- Rebase step uses **`-X theirs`** for `dashboard/data/*.json` — corrected mid-session (upstream is "ours" during rebase, replayed commits are "theirs"; "this run's data wins" requires `theirs`).

## Known bugs — status

### FIXED, verified, and LIVE on main (all proven working end-to-end via run #11)
- Scrapy 2.18 `start_requests()` → `async def start()` API change.
- Wikidata wrong hardcoded entity IDs — fixed via runtime `wbsearchentities` lookup.
- thevaaram_thirumurai depth-84 runaway — `DEPTH_LIMIT: 3`.
- DSAL spiders — DROPPED, genuinely blocked by robots.txt.
- dbpedia_tamil — built then dropped (stale/empty data).
- wikipedia_tamil_multilingual rate-limiting — spider-scoped throttle. Verified 255→0 errors.
- met_museum_tamil 403s — WAF-driven, fixed with concurrency=1 + 403 in retry codes + AutoThrottle pinned.
- met_museum_tamil Tanjavur→Thanjavur typo — fixed.
- tamil_wikibooks/wikinews/wikiquote sampling — converted to deterministic full sweeps.
- tamil_wikipedia_categories missing from schedule — added to Job A.
- **thevaaram_thirumurai field mislabeling + off-canon content** (commit `b3f26ca`) — verified 103 items, 0 leakage.
- **tamil_wikisource_texts 97.5% empty** (commit `b3f26ca`) — root cause was ProofreadPage transclusion. Fix: `action=parse&prop=text` + `clean_html()`. Verified 40/40 non-empty, real inspected Tamil prose.
- **settings.py AutoThrottle/concurrency upgrade** (commit `38021fd`) — verified per-spider custom_settings precedence holds.
- **project_madurai_texts.py missing throttle pin** — fixed with a conservative, well-reasoned pin.
- **28 new spiders + final crawl.yml** (commit `fa7be1a`) — all 40 spiders confirmed loadable via `scrapy list`, and now proven to actually run correctly in CI via run #11.
- **Day 7-8 check-in (2026-09-05)**: 5/5 recent scheduled runs succeeded; repo size and data volume on track; spot-checked file confirmed real, non-empty content.

### FOUND, fix drafted/sent, NOT yet confirmed back
- **Dashboard not reflecting run #11's fresh 40-source data** — see "KNOWN OPEN ISSUE" above. Deferred by user, not rechecked 2026-09-05.

### Deliberately left alone (not bugs, in-scope decisions)
- `tamil_wiktionary.py` and full `tamil_wikisource_texts` (37,773 articles) still use random-single-batch sampling — needs `apcontinue` pagination, a real rewrite, correctly deferred.
- Dashboard's `runlogs/` is gitignored — stale detail until each job's own next run.
- Wikisource index/TOC pages contribute boilerplate rather than prose — flagged, no filter built.

## Dashboard (user's own parallel project, reviewed for awareness not built by me)
Live at https://vishnuvarthan18.github.io/tamil-data-dashboard/. Last known to be showing STALE data (see open issue above) as of 2026-08-28 — deferred, not rechecked 2026-09-05. Sections: Engine health, Data collected, Spot-checks, Browse records. Built by `scripts/build_dashboard_data.py`, force-push single-commit each run, direct to `main`.

## Pending tasks (in priority order)
1. **(Deferred)** Diagnose why the dashboard didn't reflect run #11's data: check exact commit timestamp/message in `tamil-data-dashboard`, check Settings → Pages deployment status there, check the "Publish dashboard" step log inside a completed run. Worth asking about at day 15 review if not resolved sooner.
2. Double-check `project_madurai_texts.py` (MAX_LINKS_PER_RUN=150) and `wikipedia_tamil_categories.py` throttle settings are correctly committed as expected (content itself confirmed healthy 2026-09-05, but the specific settings weren't directly inspected).
3. Consider un-gitignoring `runlogs/` for dashboard freshness (not decided).
4. Consider a prose-only filter for Wikisource index/TOC pages (not decided).
5. **Day 15 full review**: repo size final check, all-spider health review, next-phase planning conversation (dataset use, dashboard fix, any new sources).

## User context
- Non-technical, needs plain-English step-by-step instructions with exact button names/locations, especially for GitHub UI navigation.
- GitHub username: `vishnuvarthan18`
- Runs a separate AI coding agent independently — Claude writes prompts, verifies output, does not write code directly. Agent has no memory across invocations.
- Building the dashboard themselves in parallel with that same agent — Claude reviews for awareness, doesn't drive that track.
</content>
</invoke>
