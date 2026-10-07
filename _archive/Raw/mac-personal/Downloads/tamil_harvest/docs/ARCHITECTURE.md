# Architecture

## The flow, end to end
```
 open sources                 DATA ENGINE (private)                         WEBSITE (public)
 Wikipedia, Wikisource,   ->  tamil_harvest/  Scrapy spiders        ->     website/data/*.json
 Project Madurai,             data/*.jsonl    raw crawled records          (articles, works, images)
 Internet Archive, ...        scripts/ + engines/ build + clean                     |
                              site_data/      website-ready JSON  --copy-->  website/*.html read it in the browser
                                                                                    |
 DESIGN SYSTEM (private)                                                   git push main -> GitHub Action
 design/showcase/css  ---copy tokens.css + components.css--->  website/css          |
                                                                                    v
                                                                     Cloudflare Pages  ->  semmozhi.pages.dev
                                                                                           semmozhi.online
```

## The website is static
Plain HTML, CSS and JavaScript plus **Web Awesome Core** (MIT, web components, self-hosted in `website/vendor/webawesome/`, no CDN). No build step, no server code. Buttons, cards, tags, callouts and breadcrumbs are `<wa-button>`, `<wa-card>`, `<wa-tag>`, `<wa-callout>`, `<wa-breadcrumb>`. Web Awesome reads our tokens through `css/wa-theme.css` (source: `design/showcase/css/wa-theme.css`), so colours, fonts and shape stay ours; `js/wa-head.js` keeps its light/dark class in step with the Day/System/Night switch. Use only Core components; Web Awesome Pro is not licensed here. Pages such as Cholas and Literature fetch JSON from `website/data/` in the browser and render it (`website/js/content.js` provides the shared header, footer and helpers). The Lab pages run entirely in the browser: conversion code is in `website/js/brahmi.js`, `maps.js`, `grantha-map.js`.

| Piece | Where |
|---|---|
| Pages | `website/*.html` |
| Shared shell (nav, footer, theme switch, Lab sub-nav) | `website/js/content.js` |
| Script conversion engine | `website/js/brahmi.js`, `maps.js`, `grantha-map.js` |
| Styles | `website/css/tokens.css`, `components.css` (design system), `content.css`, `lab.css` (page-level) |
| Data the pages read | `website/data/` (tracked in git on purpose) |
| Fonts | `website/fonts/` (Semmozhi fonts under SIL OFL 1.1, plus self-hosted Fraunces and Noto Serif Tamil) |

## The design system
Source of truth for the CSS is `design/showcase/css/`. The website keeps its own copy of `tokens.css` and `components.css` so it is self-contained and deploys alone. After changing the design system, copy the two files into `website/css/` (see [design/README.md](../design/README.md)). Colours and sizes come only from tokens; do not hard-code them in pages. The design workbench (`design/showcase/`: style guide, identity gallery, realism sample pages) is never published.

## The data engine
- **Crawlers** (`tamil_harvest/spiders/`): one small Scrapy spider per source. Output is JSON Lines, one record per line.
- **Topic engines** (`engines/<topic>/config.py`): language, literature, history, archaeology, culture, places, people, world, mission, media. Each config lists what to collect (for example Wikipedia article titles). `python3 -m engines.run` orchestrates crawling and building.
- **Builders** (`scripts/build_site_data.py`, `postprocess_site_data.py`): turn raw records into the website JSON.
- **Where it runs**: locally for development, or on a VPS (`vps/`: Docker, Caddy, cron). The GitHub crawl workflow (`.github/workflows/crawl.yml`) exists but its schedule is switched off.
- **Viewers**: `viewer_app/` (browse the cleaned corpus locally), `dashboard/` (crawl status page, published by the crawl workflow to a separate repo), `design/reference_engine/viewer.html` (reference-photo viewer).

Explore and Engine pages were removed from the site and deleted on 2026-10-04 (recoverable: `git log --diff-filter=D -- archive/pending`). They read `hub.json`, `search.json` and `engines/*/index.json`, which the engine has not produced yet.

## Hosting
- **Cloudflare Pages**, project `semmozhi`, in the owner's Cloudflare account. Production branch: `main`.
- **Domains**: `semmozhi.online` and `www.semmozhi.online` (registered at BigRock, DNS on Cloudflare). `semmozhi.info` is owned but not yet connected.
- The same Cloudflare account also hosts the unrelated Sathyamangalam Atlas projects. Do not touch them.
