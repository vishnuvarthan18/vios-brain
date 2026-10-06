# Project map

Folder-by-folder. For the big picture read [README.md](README.md) and [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Website (public): `website/`
The only folder that is published. See [website/README.md](website/README.md).

## Design system (private): `design/`
| Path | What |
|---|---|
| `design/showcase/` | Workbench and source of truth for the CSS; style guide, identity gallery, sample pages |
| `design/svg_kit/`, `design/realism/`, `design/reference_engine/` | SVG kit, realism engine, reference-photo database and viewer |
| `design/references/` | Licensed reference photos (git-ignored) |
| `design/prompts/`, `design/specs/` | Design prompts and specs |

## Data engine (private)
| Path | What |
|---|---|
| `tamil_harvest/` | Scrapy crawler (do not rename the inner folders) |
| `engines/` | Topic engines and the orchestrator (`python3 -m engines.run`) |
| `scripts/` | Site-data builders, dashboard builder, `check_website.py` |
| `vps/`, `v2/` | Server setup (Docker, Caddy, cron) and VPS spiders |
| `viewer_app/` | Local corpus browser |
| `dashboard/` | Crawl status page |
| `tools/` | Font build tools (`font`, `font_v3`) |
| `clean_data.py`, `scrapy.cfg` | Cleaner and Scrapy entry point (must stay at the root) |
| `data/` | Raw crawled records (tracked) |
| `data_clean/`, `data_classified/`, `site_data/` | Derived data (git-ignored, rebuilt locally) |

## Project plumbing
| Path | What |
|---|---|
| `docs/` | Team documentation |
| `local/` | Local page hub (`make local`) |
| `.github/` | Workflows (checks, deploy, crawl), pull request and issue templates, code owners |
| `Makefile`, `deploy.sh` | Everyday commands and the manual deploy |
