# Getting started

## You need
- Git, **Python 3.12**, **Node 22+** (only for the JavaScript syntax check), a modern browser.
- For deploying by hand: `npm i -g wrangler` and `wrangler login` (normally CI does this for you).
- For running the crawler locally: `pip install scrapy itemadapter`.

## 1. Get the code
```bash
git clone https://github.com/vishnuvarthan18/tamil-data-collector.git
cd tamil-data-collector
git checkout dev
```
The repository is large (about 450 MB of history, because crawled data is committed). The first clone takes a while. See [KNOWN_ISSUES.md](KNOWN_ISSUES.md).

## 2. Look at everything locally
```bash
make local
```
Opens http://localhost:8000/local/, a hub that links to every page in the project, grouped by part. The website pages are under `website/`. Nothing to build or install.

## 3. Check your work
```bash
make check
```
Runs: Python compile of the engine code, a link and asset check of every website page, JSON validity of `website/data`, JavaScript syntax, and a guard that nothing private is inside `website/`. CI runs the same command on every pull request.

## 4. Make a change
Follow [CONTRIBUTING.md](../CONTRIBUTING.md). In short: branch from `dev`, change, `make local`, `make check`, pull request into `dev`.

## What lives where (30-second tour)
```
website/            the live site (static HTML, CSS, JS, JSON data, fonts)
design/             design system work; design/showcase/ is the workbench (never published)
tamil_harvest/      Scrapy crawler (spiders, pipelines, settings)
engines/            topic engines that turn crawled data into website JSON
scripts/            build scripts and check_website.py
vps/                server setup (Docker, Caddy, cron) for running the engine on a VPS
viewer_app/         local browser for the cleaned corpus (python3 viewer_app/app.py)
dashboard/          crawl status dashboard (published separately by the crawl workflow)
archive/            old things kept on purpose
docs/               you are here
local/              the local page hub
```

## Useful commands
| Command | What it does |
|---|---|
| `make local` | Serve the project, page hub at /local/ |
| `make check` | All checks |
| `make site-data` | Rebuild website JSON from crawled data (needs `data_classified/`) |
| `python3 viewer_app/app.py` | Browse the cleaned corpus at http://127.0.0.1:8000 |
| `python3 -m engines.run list` | List the topic engines and their spiders |
| `scrapy crawl <spider>` | Run one crawler locally |
