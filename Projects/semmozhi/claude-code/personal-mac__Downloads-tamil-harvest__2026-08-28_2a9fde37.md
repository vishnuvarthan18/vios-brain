**Vishnu** (2026-08-28T06:19): Add a custom_settings throttle pin to project_madurai_texts.py, matching the pattern used in met_museum_tamil.py or wikipedia_tamil_multilingual.py — this spider currently has no pin, so under the new global concurrency (32 concurrent, 0.1s delay) it has no error-driven backoff if Project Madurai (a small volunteer-run site) starts fast-failing.

Suggest something conservative like:
custom_settings = {
    "CONCURRENT_REQUESTS": 2,
    "CONCURRENT_REQUESTS_PER_DOMAIN": 1,
    "DOWNLOAD_DELAY": 1.0,
    "AUTOTHROTTLE_TARGET_CONCURRENCY": 1.0,
}

Use your judgment on exact values based on what you know of the site's behavior from prior runs. Commit this alone (don't touch the 28 untracked spiders or crawl.yml). Report the diff, commit hash, and confirm nothing else changed.

**Claude** (2026-08-28T06:19): _[tool: Bash]_

**Claude** (2026-08-28T06:19): _[tool: Bash]_

**Claude** (2026-08-28T06:19): _[tool: Bash]_

**Claude** (2026-08-28T06:20): Pre-existing wrinkle: [project_madurai_texts.py](tamil_harvest/spiders/project_madurai_texts.py) already has an uncommitted change in your working tree (`MAX_LINKS_PER_RUN` 60 → 500). Let me check prior run data before picking values.

**Claude** (2026-08-28T06:20): _[tool: Bash]_

**Claude** (2026-08-28T06:20): _[tool: Bash]_

**Claude** (2026-08-28T06:20): _[tool: Bash]_

**Claude** (2026-08-28T06:20): _[tool: Bash]_

**Claude** (2026-08-28T06:20): _[tool: Bash]_

**Claude** (2026-08-28T06:20): _[tool: Bash]_

**Claude** (2026-08-28T06:20): _[tool: Bash]_

**Claude** (2026-08-28T06:20): _[tool: Bash]_

**Claude** (2026-08-28T06:20): _[tool: Bash]_

**Claude** (2026-08-28T06:20): _[tool: Bash]_

**Claude** (2026-08-28T06:21): Good grounding. Key numbers from prior runs:

- The index lists **2,931 qualifying links** (html/pdf) — the 26 items/run came from dedup, not a small catalogue.
- Prior runs pulled **6.6 MB of response bodies for 26 items** — median ~250 KB, 10 of 26 were PDFs.
- 26 of 27 unique requests succeeded, so the site is healthy at the current rate; the risk is volume, not flakiness.

So the load here is **bytes**, not request count — and your working tree's uncommitted `MAX_LINKS_PER_RUN = 500` would multiply it. Writing the pin:

**Claude** (2026-08-28T06:21): _[tool: Bash]_

**Claude** (2026-08-28T06:21): _[tool: Bash]_

**Claude** (2026-08-28T06:21): _[tool: Bash]_

**Claude** (2026-08-28T06:22): Staged diff is the pin alone. Verifying the spider still loads before committing:

**Claude** (2026-08-28T06:22): _[tool: Bash]_

**Claude** (2026-08-28T06:22): _[tool: Bash]_

**Claude** (2026-08-28T06:22): Effective settings resolve correctly, and `429` is already in the inherited retry codes — no need to redefine them. Committing:

**Claude** (2026-08-28T06:22): _[tool: Bash]_

**Claude** (2026-08-28T06:22): _[tool: Bash]_

**Claude** (2026-08-28T06:22): Done. Commit `63efa6c`, one file, 26 insertions, 0 deletions.

## The diff

```python
     name = "project_madurai_texts"
     start_urls = ["https://www.projectmadurai.org/pmworks.html"]
 
+    # projectmadurai.org is a single volunteer-run static host, and this
+    # spider is the heaviest thing pointed at it: the index lists ~2900
+    # qualifying links, and the pages behind them are large - a past run
+    # pulled 6.6MB of response bodies for 26 items, median ~250KB, with
+    # PDFs making up a third of them. The cost here is bytes on someone
+    # else's bandwidth bill, not request count, so it is pinned well below
+    # the project-wide concurrency rather than left to adapt.
+    #
+    # AutoThrottle alone is not enough cover. It steers on latency, so it
+    # only backs off once the server is already struggling enough to answer
+    # slowly - a host that sheds load by returning quick 503s reads as
+    # headroom and it speeds up (the same trap met_museum_tamil documents).
+    # MAX_DELAY is raised from the project's 10s so that when latency does
+    # climb there is real room to back off into. RETRY_TIMES covers the
+    # transient 5xx a small shared host throws under load; 403 is
+    # deliberately NOT added to the retry codes, because on plain Apache it
+    # means the file is genuinely not served, and retrying is just noise.
+    custom_settings = {
+        "CONCURRENT_REQUESTS": 2,
+        "CONCURRENT_REQUESTS_PER_DOMAIN": 1,
+        "DOWNLOAD_DELAY": 1.5,
+        "AUTOTHROTTLE_TARGET_CONCURRENCY": 1.0,
+        "AUTOTHROTTLE_MAX_DELAY": 30,
+        "RETRY_TIMES": 5,
+    }
+
     MAX_LINKS_PER_RUN = 60
```

## Where I deviated from your suggested values

Your four keys are all in, with `DOWNLOAD_DELAY` at 1.5 instead of 1.0, plus two additions. Grounds, from [data/](data/):

- **`DOWNLOAD_DELAY` 1.0 → 1.5.** The last run pulled 6.6 MB of bodies for 26 items — median ~250 KB, 10 of 26 were PDFs. At 1 req/s that's a sustained ~250 KB/s draw off a volunteer host; 1.5s puts it nearer 170 KB/s. Request rate is the wrong meter for this spider.
- **`AUTOTHROTTLE_MAX_DELAY: 30`** (project default is 10). This is the actual backoff headroom — when the host does get slow, AutoThrottle otherwise caps out at 10s.
- **`RETRY_TIMES: 5`**, matching both reference spiders. I did *not* add 403 to `RETRY_HTTP_CODES` the way [met_museum_tamil.py](tamil_harvest/spiders/met_museum_tamil.py) does — that spider's 403 is an Incapsula throttle response, but from plain Apache a 403 is a genuine refusal and retrying it 5× is just noise. `429` needed no work: it's already in Scrapy's inherited defaults.

Verified with `scrapy list` (loads, 40 spiders) and by resolving the merged settings — all six land as written.

## Two things to flag

**1. This file already had an uncommitted change, and it's the reason the pin matters.** Your working tree had `MAX_LINKS_PER_RUN` 60 → 500, from before this session. You said commit the pin alone, so I staged the pin against the `HEAD` version and **left the 500 uncommitted** — it's still there as the sole unstaged hunk in this file, exactly as you had it.

**2. The index is far bigger than the 26 items/run suggests.** It carries **2,931 qualifying html/pdf links**. The 26 isn't a small catalogue — the first 60 links in index order contain only 27 unique URLs (nav repeats), and 26 of those 27 returned items. So at `MAX_LINKS_PER_RUN = 500` the unique fan-out is several hundred requests at ~250 KB each, roughly 100 MB per run, four runs a day. The pin makes that polite in rate but not in volume; you may want to weigh the 500 separately, which is partly why I left it for you rather than folding it in.

Nothing else changed: `crawl.yml` and `wikipedia_tamil_categories.py` are untouched, and all 28 spiders remain untracked.

**Vishnu** (2026-08-28T06:25): Now merge the final crawl.yml. Context: this repo has 40 total spiders. 12 are already live in the current committed crawl.yml (single job, 6h cron). thevaaram_thirumurai and tamil_wikisource_texts are already committed and fixed. 28 spider files currently sit untracked in the working tree and are NOT yet part of any commit or workflow — do not touch/stage those files, this task is only about crawl.yml.

Target structure — two jobs, split by domain contention (Wikimedia-family spiders rate-limit each other if run together, this is NOT a runtime constraint):

Job A "wikimedia" (19 spiders), cron `23 0,12 * * *` UTC:
tamil_wikisource, tamil_wikisource_texts, tamil_wikisource_classics, tamil_wikipedia_stats, tamil_wikipedia_texts, tamil_wikipedia_categories, wikipedia_tamil_articles, wikipedia_tamil_categories, wikipedia_tamil_multilingual, wikidata_tamil_entities, wikidata_tamil_works, wikimedia_commons_tamil, wikivoyage_tamil_nadu, english_wikisource_tamil, tamil_wiktionary, wiktionary_tamil_etymology, tamil_wikiquote, tamil_wikibooks, tamil_wikinews

Job B "other sources" (21 spiders), cron `23 6,18 * * *` UTC:
project_madurai, project_madurai_texts, internet_archive_tamil, internet_archive_tamil_language, internet_archive_tamil_collections, internet_archive_tamil_fulltext, thevaaram_thirumurai, openlibrary_tamil, openalex_tamil_studies, crossref_tamil_studies, arxiv_tamil_nlp, doaj_tamil, zenodo_tamil, huggingface_tamil_datasets, met_museum_tamil, cleveland_museum_tamil, art_institute_chicago_tamil, overpass_tamil_heritage, tamil_nlp_catalog, ai4bharat_catalog, mozilla_data_collective

Requirements:
1. Both jobs must keep the existing dashboard-publish steps (build_dashboard_data.py + SSH deploy to the public dashboard repo) — check the current committed crawl.yml for exactly what those steps do and preserve them unmodified.
2. Add a `workflow_dispatch` input `group: [both, wikimedia, other]` since GitHub Actions has no true per-job manual trigger — use it to conditionally run Job A, Job B, or both on manual trigger, while the cron schedules always run their own job only.
3. Add a concurrency guard so overlapping runs of the same job don't collide, and `git pull --rebase --autostash` before any commit-and-push step in the workflow itself, since CI commits data/dashboard JSON on its own schedule.
4. IMPORTANT: since only 12 of the 40 spiders in these lists are currently committed to the repo (thevaaram_thirumurai + tamil_wikisource_texts + the original 12, plus dashboard spiders if any overlap), the other 28 do NOT exist in the working tree as committed files yet — so this crawl.yml, once merged, will reference spiders that don't exist until a later commit adds them. Do not try to make crawl.yml "work" against the current tree — just build the correct final YAML. Flag clearly in your report which spider names in the two job lists are not yet present as files, so we can double check nothing is misspelled before that later commit lands.
5. Do NOT commit or push this yet. Just produce the final crawl.yml content and show me the full diff against the current committed version, plus your list from point 4. I'll review before we commit.

**Claude** (2026-08-28T06:25): Let me read both versions of crawl.yml and inventory the actual spider names.

**Claude** (2026-08-28T06:25): _[tool: Bash]_

**Claude** (2026-08-28T06:25): _[tool: Bash]_

**Claude** (2026-08-28T06:26): The working tree already has a WIP two-job split. Let me verify the spider lists against reality before finalizing.

**Claude** (2026-08-28T06:26): _[tool: Bash]_

**Claude** (2026-08-28T06:26): _[tool: Bash]_

**Claude** (2026-08-28T06:27): Every name resolves to a real spider — no misspellings. Let me verify the confusable near-duplicate pairs are genuinely distinct, and check the committed workflow's actual count.

**Claude** (2026-08-28T06:27): _[tool: Bash]_

**Claude** (2026-08-28T06:27): _[tool: Bash]_

**Claude** (2026-08-28T06:27): Confirmed distinct — `tamil_wikipedia_*` hits **ta.**wikipedia.org, `wikipedia_tamil_*` hits **en.**wikipedia.org. Also note: the committed workflow runs **11** spiders, not 12.

Now generating the final YAML. I'll template the shared job body so the two jobs can't drift apart:

**Claude** (2026-08-28T06:28): _[tool: Bash]_

**Claude** (2026-08-28T06:28): Now validating: YAML parses, structure is right, and the two job bodies differ only where intended.

**Claude** (2026-08-28T06:28): _[tool: Bash]_

**Claude** (2026-08-28T06:28): _[tool: Bash]_

**Claude** (2026-08-28T06:28): _[tool: Bash]_

**Claude** (2026-08-28T06:29): That "1 iteration" is a zsh artifact (zsh doesn't word-split unquoted expansions; GitHub Actions runs `bash`). Re-testing under bash, which is what actually runs:

**Claude** (2026-08-28T06:29): _[tool: Bash]_

**Claude** (2026-08-28T06:29): Verified: 21 iterations under bash. Now the full diff against the committed version:

**Claude** (2026-08-28T06:29): _[tool: Bash]_

**Claude** (2026-08-28T06:29): _[tool: Bash]_

**Claude** (2026-08-28T06:29): Final [crawl.yml](.github/workflows/crawl.yml) is written to the working tree — **not staged, not committed**. HEAD is still `63efa6c`. Full diff is above (244 insertions, 23 deletions).

## Point 4 — spider name audit

I checked all 40 names in your two lists against the `name = "..."` attribute declared in every spider file, not against filenames.

**No misspellings. All 40 resolve to a real spider.** Also: no name appears in both lists, none is duplicated within a list, and no declared spider is missing from the lists — the 19 + 21 split covers exactly the 40 that exist.

**28 are not yet committed** (each exists as an untracked file, so `scrapy crawl` will fail until a later commit lands them) — this matches your 28 exactly:

*Job A / wikimedia — 13 of 19 not committed:*
`tamil_wikisource_classics`, `tamil_wikipedia_texts`, `tamil_wikipedia_categories`, `wikipedia_tamil_multilingual`, `wikidata_tamil_works`, `wikimedia_commons_tamil`, `wikivoyage_tamil_nadu`, `english_wikisource_tamil`, `tamil_wiktionary`, `wiktionary_tamil_etymology`, `tamil_wikiquote`, `tamil_wikibooks`, `tamil_wikinews`

*Job B / other — 15 of 21 not committed:*
`internet_archive_tamil`, `internet_archive_tamil_language`, `internet_archive_tamil_collections`, `internet_archive_tamil_fulltext`, `openlibrary_tamil`, `openalex_tamil_studies`, `crossref_tamil_studies`, `arxiv_tamil_nlp`, `doaj_tamil`, `zenodo_tamil`, `huggingface_tamil_datasets`, `met_museum_tamil`, `cleveland_museum_tamil`, `art_institute_chicago_tamil`, `overpass_tamil_heritage`

**Two near-identical pairs worth a second look** — both are real, distinct spiders, not typos. The convention is `tamil_X` = the Tamil-language wiki, `X_tamil` = the English wiki about Tamil topics:
- `tamil_wikipedia_categories` (ta.wikipedia.org, untracked) vs `wikipedia_tamil_categories` (en.wikipedia.org, committed) — both in Job A
- `tamil_wikipedia_texts` (ta.wikipedia.org) vs `tamil_wikisource_texts` (ta.wikisource.org) — both in Job A

## Corrections to the premise

The committed crawl.yml runs **11** spiders, not 12. There are 12 committed spider *files* — the 12th is `thevaaram_thirumurai`, which is committed but absent from the current workflow, so it has never run in CI. Putting it in Job B fixes that; no action needed, just so the count reconciles.

## What I changed beyond the literal spec

- **`fetch-depth: 0` on both checkouts.** `actions/checkout@v4` defaults to a depth-1 shallow clone, and `git pull --rebase` on a shallow clone has no reliable merge base. Adding the rebase without this would have made requirement 3 flaky rather than safe.
- **One shared concurrency group** (`tamil-harvest-crawl`) rather than per-job groups. Your wording was "the same job", but both jobs commit to `data/` here *and* force-push the same public dashboard repo — a parallel `both` dispatch would have them racing on both. The shared group serialises A then B instead. Consequence to accept: a manual `both` run takes A+B sequentially, not in parallel. Say the word if you'd rather have per-job groups.
- **Group labels in the two commit messages** (`chore: crawl run (wikimedia)` / `(other sources)`). This is the *only* deviation from "preserve the dashboard-publish steps unmodified" — I verified `Record recent workflow runs` and `Build dashboard data` are byte-identical to the committed versions, and `Publish dashboard` differs by that label alone. Trivial to drop if you want strict preservation.

## Verification

YAML parses; jobs are `wikimedia` (19 spiders) and `other-sources` (21); both gate strings match their `on.schedule` cron literals exactly (asserted programmatically, since a typo there silently disables a job); the `for spider in $SPIDERS` loop yields 21 iterations under **bash** — my first check ran under zsh and reported 1, which is a zsh non-word-splitting artifact, not a workflow bug. `actionlint` isn't installed here, so no lint pass.

One residual risk I did not paper over: the rebase can still conflict on `dashboard/data/*.json`, which is regenerated wholesale every run, if main genuinely moved. Timestamped files in `data/` can't conflict. With crons 6h apart and the shared concurrency group, this needs a human pushing mid-run to trigger. A `-X ours` on that path would close it if you want.

The generator I used to keep the two job bodies from drifting is at `scratchpad/gen_crawl.py` (untracked). Nothing else in the tree was touched — the other two modified files are your pre-existing `MAX_LINKS_PER_RUN` change and `wikipedia_tamil_categories.py`, both untouched by me, and all 28 spiders remain untracked.

**Vishnu** (2026-08-28T06:32): Approved — keep all three deviations (fetch-depth: 0, shared concurrency group, commit message labels).

One addition: add -X ours for the dashboard/data/*.json path in the rebase step, since those files are fully regenerated each run and there's nothing worth preserving from a conflicting version.

Now: commit crawl.yml together with all 28 currently-untracked spider files in ONE commit. Do not include scratchpad/gen_crawl.py (leave it untracked, or gitignore scratchpad/ if that's clean to do alongside). Push to main. If CI has landed commits in the meantime, rebase with autostash as before, and checksum any files NOT part of this commit (there shouldn't be any left, but confirm) before/after.

Report: commit hash, confirmation `scrapy list` now shows all 40 spiders, confirmation crawl.yml is on main, and confirmation main == origin/main.

**Claude** (2026-08-28T06:33): _[tool: Bash]_

**Claude** (2026-08-28T06:34): No CI commits landed (`origin/main` is still at `38021fd`, we're ahead by the pin commit only). Applying the rebase-strategy change — with one important correction:

**Claude** (2026-08-28T06:34): _[tool: Bash]_

**Claude** (2026-08-28T06:34): _[tool: Bash]_

**Claude** (2026-08-28T06:34): Recording checksums of everything **not** in this commit, then staging exactly the intended set:

**Claude** (2026-08-28T06:34): _[tool: Bash]_

**Claude** (2026-08-28T06:34): _[tool: Bash]_

**Claude** (2026-08-28T06:34): _[tool: Bash]_

**Claude** (2026-08-28T06:34): All 30 staged correctly. Verifying all 40 spiders load before committing:

**Claude** (2026-08-28T06:34): _[tool: Bash]_

**Claude** (2026-08-28T06:35): _[tool: Bash]_

**Claude** (2026-08-28T06:35): Exact 1:1 match between `scrapy list` and the workflow. Committing:

**Claude** (2026-08-28T06:35): _[tool: Bash]_

**Claude** (2026-08-28T06:35): _[tool: Bash]_

**Claude** (2026-08-28T06:35): No divergence (0 behind, 2 ahead) — no rebase needed. Pushing:

**Claude** (2026-08-28T06:35): _[tool: Bash]_

**Claude** (2026-08-28T06:35): _[tool: Bash]_

**Claude** (2026-08-28T06:35): _[tool: Bash]_

**Claude** (2026-08-28T06:36): Done. All checks pass.

## Results

**Commit:** `fa7be1a` — *Add 28 spiders and split the crawl into two scheduled jobs*
30 files changed, 2,626 insertions: [crawl.yml](.github/workflows/crawl.yml), [.gitignore](.gitignore), and the 28 new spiders.

**`scrapy list` shows all 40 spiders** — and I reconciled that list against the workflow rather than just counting it: 40 in `scrapy list`, 40 across the two `SPIDERS` blocks, **exact 1:1 match**, no name in the workflow that Scrapy can't load, and no loadable spider missing from the workflow. `origin/main` now carries 41 files in `tamil_harvest/spiders/` (40 spiders + `__init__.py`).

**crawl.yml is on main** — verified by reading it back out of `origin/main`, not from disk: both crons, both jobs (`wikimedia` / `other-sources`), both `concurrency` blocks, both `fetch-depth: 0`, both rebase lines.

**`main == origin/main`** — both at `fa7be1a`. No CI commits landed while we worked (`origin/main` was still at `38021fd` at push time, 0 behind / 2 ahead), so the push was a plain fast-forward and no rebase was needed.

## One correction to the requested change

You asked for `-X ours`; I used **`-X theirs`**, which is what actually produces the behaviour you described.

During a rebase git replays your commits *onto* the upstream branch, so mid-rebase the sides are inverted from the intuitive reading: upstream is `ours`, and the commits being replayed — this run's — are `theirs`. `-X ours` would have kept the *stale* JSON already on main and discarded the dashboard the running job just built, the exact opposite of "the copy from this run wins." The comment in the file spells this out so it doesn't get "fixed" back later.

Also note `-X` cannot be scoped to a path — it applies to the whole rebase. That's harmless here because the step stages only `data/` and `dashboard/data/`, and `data/` filenames carry a per-run timestamp, so `dashboard/data/*.json` is the only path that can ever conflict. The effect is path-scoped by construction rather than by flag.

## Files not in the commit — confirmed unchanged

There were two left, not zero. Both are pre-existing tracked modifications, so they were never candidates for a commit of *untracked* spiders:

| File | Why it stayed out |
|---|---|
| [project_madurai_texts.py](tamil_harvest/spiders/project_madurai_texts.py) | your uncommitted `MAX_LINKS_PER_RUN` 60 → 500 |
| [wikipedia_tamil_categories.py](tamil_harvest/spiders/wikipedia_tamil_categories.py) | in-progress edits from before this session |

SHA-256 before and after the commit+push: **identical for both**, plus `scratchpad/gen_crawl.py`. `git status` is now exactly those two modified files and nothing else — no untracked files remain, since `scratchpad/` went into `.gitignore` alongside the existing `runlogs/` and `_site/` entries.

Worth knowing: the next scheduled run fires at **06:23 UTC** (group B, other sources) and will be the first to exercise 21 spiders that have never run in CI, including the newly-wired `thevaaram_thirumurai`. The `.github/workflows` change also means group A's first run at 00:23 UTC is the real test of the cron-literal gating — if either `if:` string ever drifts from `on.schedule`, that job silently never runs rather than failing loudly.