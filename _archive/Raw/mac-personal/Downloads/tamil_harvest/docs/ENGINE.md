# Data engine

Collects open Tamil texts and images, keeps the source and licence with every record, and builds the JSON the website reads.

## Pieces
| Folder | Role |
|---|---|
| `tamil_harvest/spiders/` | One Scrapy spider per source (Wikipedia, Wikisource, Project Madurai, Internet Archive, Commons, museums, ...) |
| `tamil_harvest/pipelines*.py`, `settings*.py` | Output format and politeness settings. `settings_vps.py` is the server profile. |
| `engines/<topic>/config.py` | What each topic engine collects. Edit the lists to change it. |
| `engines/run.py` | Orchestrator (see commands below) |
| `scripts/build_site_data.py`, `postprocess_site_data.py` | Raw records -> website JSON |
| `clean_data.py` | `data/` -> `data_clean/`: drop empty and duplicate records, flag broken text (originals untouched) |
| `vps/` | Docker, Caddy, cron and run scripts for running it all on a server (`vps/README-VPS.md`) |
| `viewer_app/` | Local browser for `data_clean/` |
| `dashboard/` | Crawl status page (built by `scripts/build_dashboard_data.py`) |

## Data folders
| Folder | In git? | What |
|---|---|---|
| `data/` | yes (776 files, large) | Raw crawled records, one `.jsonl` file per run |
| `data_clean/`, `data_classified/` | no | Cleaned and categorised copies, made locally |
| `site_data/` | no | Output of the site-data build |
| `website/data/` | **yes** | The JSON the live site reads. Copy reviewed output here. |

## Commands
```bash
python3 -m engines.run list                 # engines and their spiders
python3 -m engines.run crawl history --limit 20      # small test crawl of one engine
python3 -m engines.run build all            # raw data -> website JSON (set HARVEST_DATA)
python3 -m engines.run status               # record counts and last run per engine
scrapy crawl ai4bharat_catalog -o /tmp/test.jsonl     # run a single spider
make site-data                              # legacy build from data_classified/ (sets HARVEST_ROOT for you)
python3 viewer_app/app.py                   # browse data_clean/ at http://127.0.0.1:8000
```
Environment: `HARVEST_DATA` (default `/data`, the server layout), `HARVEST_SITE_DATA`. Locally, set `HARVEST_DATA` to a scratch folder so you never write into `data/` by accident.

## Add a source
1. Copy a small spider in `tamil_harvest/spiders/` (for example `ai4bharat_catalog.py`). Set `name`, the start URL(s) and the fields you yield. Always yield `source` and `url`; keep the licence if the source states one.
2. Run it with a limit and look at the output: `scrapy crawl <name> -o /tmp/test.jsonl`.
3. Check the source's terms and `robots.txt`. Respect the rate limits already set in `settings.py`.
4. If a topic engine should use it, add it to that engine's config.
5. Rebuild site data, review the diff in `website/data/`, and open a pull request.

## Add or change a topic engine
Edit `engines/<topic>/config.py`: titles, Wikipedia article lists and other lists. `engines/registry.py` lists the engines and their order.

## Refresh the website data
1. Build: `python3 -m engines.run build all` (server) or `make site-data` (legacy local build).
2. Copy the output into `website/data/` and look at `git diff --stat`. Large or surprising changes need a second pair of eyes.
3. `make check`, then a pull request into `dev`.

## Scheduling
- GitHub Actions: `.github/workflows/crawl.yml`, manual run only. The schedule lines are commented out on purpose.
- Server: `vps/crontab.txt` runs one engine per night slot. Whether the server is currently running is not recorded in the repo; ask the owner.

## Rules
- Respect each source's terms and rate limits. The Wikimedia-family spiders share one rate limit.
- Keep source and licence on every record. Do not publish anything whose licence is unclear.
- Never write into `data/` from a script that is not a crawler. Cleaning writes to `data_clean/`.
