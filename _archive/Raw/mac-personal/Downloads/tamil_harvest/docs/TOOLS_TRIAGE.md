# Reference and comparison tools: what to do with them

Proposal only. Nothing is moved or deleted ("clean" means organise, never delete). Sizes measured 2026-10-03.

| Folder | Size | What it is | Proposal |
|---|---|---|---|
| `design/showcase/` | 6 MB | Style guide, identity gallery, CSS source of truth | **Keep. Already published on the private admin site** (`/design/`). |
| `design/svg_kit/` | 0.7 MB | Our own SVGs; the identity kit is generated | **Keep.** |
| `dashboard/` | 1 MB | Crawl status page | **Keep. Already on the admin site** (`/dashboard/`). |
| `design/reference_engine/` | 19 MB | Reference-photo database (`refs.db`) and viewer | **Keep local only** (decided). |
| `design/realism/` | 368 MB | Realism engine: measured materials, checks, reports, backups | **Keep local only.** Largest tool; do not put on a server or the admin site. Never delete `backups/`. |
| `design/references/` | 2.1 GB | Licensed reference photos (git-ignored) | **Keep local, back up off the laptop** (external drive or the server). Never commit. |
| `viewer_app/` | 280 MB | Python browser for the cleaned corpus (SQLite) | **Keep local.** Needs a server process, so it is not on the admin site. Rebuild its index from the audit-cleaned data. |
| `data_clean/`, `data_classified/` | 116 + 118 MB | Cleaned and classified copies of the crawl | **Keep; dedupe** to the latest file per source (see `DATA_AUDIT.md`). Archive, do not delete. |
| `data/` | 1.2 GB | Raw crawl, 96% repeats | **Archive after a deduped copy exists.** |
| `tools/font*`, `v2/`, `local/` | 2 MB | Font build tools, old v2, local hub | **Keep.** |
| `archive/` | 8 MB | Retired pages and pending Explore/Engine pages | **Keep.** |

## What goes where
- **Admin site (engine.semmozhi.online, private):** design system, style guide, crawl dashboard, staging vs production. Next candidate: data-audit numbers.
- **Server (OVH):** the collector and its data only.
- **Your laptop:** realism engine, reference photos, corpus viewer.
- **Public website:** nothing from these folders.

## Decisions
1. **Decided 2026-10-03: the reference-photo viewer stays local only.** It is not published to the admin site or the server.
2. Open: is a second copy of `design/references/` (2.1 GB) wanted on the server or an external drive? It is your only copy.
