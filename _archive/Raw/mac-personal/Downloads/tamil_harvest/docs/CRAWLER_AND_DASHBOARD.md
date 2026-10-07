> Moved from the old root README. It describes the original crawler prototype and the status dashboard. Newer engine docs: [ENGINE.md](ENGINE.md).

# Tamil Data Collector — Scrapy 15-day prototype

Uses **Scrapy**, a real open-source crawling engine (not custom-built), to
collect Tamil language/culture/literature data from 6 sources. Runs on a
free GitHub Actions schedule for 15 days, then you look at what was
collected before deciding what to build next.

## What's in here

- `tamil_harvest/spiders/` — one small file per source. Each just says
  "start at this URL" and "grab this field." That's the minimum Scrapy
  needs to work; nothing else custom was written.
- `.github/workflows/crawl.yml` — runs `scrapy crawl` for all 6 spiders
  every 6 hours, saves results as `.jsonl` files into `/data`, commits
  them to the repo automatically. Free, no server needed.

## Sources being collected

1. `project_madurai` — index of classical Tamil literature texts
2. `tamil_nlp_catalog` — GitHub catalog of Tamil NLP/AI resources
3. `ai4bharat_catalog` — GitHub catalog of Indic NLP resources
4. `tamil_wikisource` — list of Tamil Wikisource pages (via API)
5. `tamil_wikipedia_stats` — Tamil Wikipedia size/activity stats (via API)
6. `mozilla_data_collective` — Tamil Literature Corpus dataset page

## To actually start the 15-day run

1. Create a new GitHub repo (private or public, your call).
2. Push this `tamil_harvest` folder to it.
3. GitHub Actions will start running automatically on the schedule
   (every 6 hours). You can also trigger it manually anytime from the
   repo's "Actions" tab → "Tamil Data Collector - Scrapy Crawl" →
   "Run workflow."
4. After 15 days, look in the `/data` folder — each file is one
   crawl run's results, in `.jsonl` format (one record per line,
   readable as plain text or opened in Excel/Sheets after conversion).

## Status dashboard

`https://vishnuvarthan18.github.io/tamil-data-dashboard/`

A static page showing what the crawler collected and whether it is healthy.
It is rebuilt and republished automatically at the end of every crawl, so
checking in on the project means opening one link — no tools, no logins.

Four sections:

1. **Engine health** — every spider's status in the latest crawl (working /
   partly done / failed), what it collected, any errors it logged, how long
   each crawl took, and how fast each website let us crawl it.
2. **Data collected** — totals per source, growth over time, repository size,
   and coverage bars where a real total exists to measure against (for
   example, how many of Project Madurai's linked works have been downloaded).
3. **Spot-checks** — a fresh random handful of records per source every run,
   shown raw, with automatic warnings for empty text, missing Tamil script,
   raw HTML, and search terms that do not match what was actually collected.
4. **Browse records** — search and filter the collected text by source,
   keyword and date, loaded a page at a time so nothing large is ever
   downloaded at once.

### How it is built

- `scripts/build_dashboard_data.py` runs inside the crawl workflow, after the
  spiders. It reads `data/*.jsonl` and the per-spider Scrapy logs and writes
  small JSON summaries into `dashboard/data/`, which are committed alongside
  the data. Line counts for older files are cached, so it never re-reads data
  it has already measured.
- `dashboard/` holds the page itself: three static files, no frameworks.
- The workflow copies both into `_site/`, adds paginated record shards for
  the browser, and pushes that to the public `tamil-data-dashboard` repo,
  which serves it on GitHub Pages. This repo stays private; only the built
  dashboard is public. Every run also uploads `_site` as a downloadable
  workflow artifact.

Publishing uses an SSH deploy key stored as the `DASHBOARD_DEPLOY_KEY` secret;
it can only write to the dashboard repo. Set the `DASHBOARD_REPO` variable to
publish somewhere else, or remove the secret to stop publishing entirely.

## Running it yourself locally (optional, to test)

```
cd tamil_harvest
pip install scrapy
scrapy crawl project_madurai -o test_output.json
```

Note: this needs a normal internet connection — it won't work from a
locked-down sandbox environment.

## Built-in politeness (Scrapy defaults, not custom code)

- 1 request per website domain at a time
- 1 second delay between requests
- Obeys each website's `robots.txt` rules automatically
- Identifies itself honestly via a User-Agent header

## After the 15 days

Look at `/data/*.jsonl` to see what was actually collected — how much
text, how many pages/links, whether sources returned useful data or
mostly errors. That tells us what's worth building next (a database, a
website, a bigger crawl) instead of guessing upfront.
