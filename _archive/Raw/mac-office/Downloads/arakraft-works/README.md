# araKraft Works

A responsive, English-language website and product catalogue for araKraft Works, a laser engraving and CNC workshop in Batticaloa, Sri Lanka. Every enquiry opens a product-specific WhatsApp conversation; no checkout, accounts or form backend is required.

Live: https://arakraft.works

## What it does

- Leads with the four laser collections (photo frames, souvenirs, key tags, restaurant menus) and their sourced starting prices.
- Shows CNC work as an auto-playing gallery; CNC is quote-only.
- Tracks Meta and GA4 enquiry events when IDs are configured, and nothing is downloaded when they are not.
- Ships as a pre-rendered static site with structured data, sitemap, robots and a branded 404 page.

## Stack

| Layer | Tool |
| --- | --- |
| UI | React 19 + TypeScript, plain CSS |
| Build | Vite 8, pre-rendered by `src/scripts/finish-build.mjs` |
| Lint | oxlint |
| Tests | Playwright (Chrome + WebKit) with axe-core |
| Hosting | Cloudflare Pages, built from `main` on every push |
| Task runner | make |
| Release | semantic-release (changelog, exec, git) |

This is a static site, so the conventions' default is Astro. It uses Vite + React instead: Vite pre-renders a single page with no server.

Everything the website needs lives in `src/`, including `package.json`, `node_modules` and the tool config, and every `make` target runs from there. The root keeps only the convention files and folders.

## Layout

```
src/
├── index.html        # HTML shell, metadata and social preview tags
├── main.tsx          # Client entry; hydrates the pre-rendered page
├── app.tsx           # Page layout, header, footer, 404 view
├── components/       # One file per section or shared UI piece
├── lib/              # Analytics, WhatsApp links, prices, scroll motion
├── types/            # Shared TypeScript types
├── config/site.ts    # Contact details, location, links, trust messages
├── data/             # products.json, studio-images.json
├── public/           # Static assets copied as-is; images are AVIF/WebP with JPEG fallbacks
├── scripts/          # finish-build.mjs: pre-render step run after vite build
├── index.css         # Base fonts, reset, shared layout
├── studio.css        # Art direction, sections, responsive styles
├── package.json      # Dependencies and npm scripts
└── *.config.ts, tsconfig*.json, .oxlintrc.json, .releaserc.json, .node-version, .env.*
tests/                # Playwright suite; config is src/playwright.config.ts
```

- `src/data/products.json`: the four laser collections and their sourced range starting prices.
- `src/data/studio-images.json`: product, banner and workshop photographs with source paths and optimised dimensions.
- `docs/`: launch notes, verification results, image provenance and hosting (see [docs/readme.md](docs/readme.md)).

## Commands

Use Node.js 22.12 or newer and npm.

| Command | Does |
| --- | --- |
| `make install` | Install dependencies and the Playwright WebKit engine |
| `make setup` | Create or top up `src/.env.local` from `src/.env.example` |
| `make dev` | Run locally at http://127.0.0.1:5173 (override with `HOST=` / `PORT=`) |
| `make build` | Type-check, build and pre-render to `src/dist/` |
| `make preview` | Serve the production build |
| `make lint` | Lint the source |
| `make test` | Run the Playwright suite (uses installed Google Chrome plus WebKit) |
| `make release` | Cut a release: bumps `VERSION`, writes `CHANGELOG.md`, tags (needs git and conventional commits) |
| `make clean` | Remove build artefacts |

## Configuration

`make setup` copies any keys from `src/.env.example` that are missing from `src/.env.local`, without changing values already set. All `VITE_*` values are public browser configuration, never secrets. Rebuild after changing them.

- **Analytics**: set `VITE_GA4_MEASUREMENT_ID` and `VITE_META_PIXEL_ID`. Tracked events are Meta PageView, ViewContent, Contact and product-card Lead, and GA4 product views, WhatsApp clicks, frame design selection and directions. Without IDs, events stay in the local `dataLayer`. Tests check event payloads without sending WhatsApp messages.
- **Location**: the map shows the labelled Kallady area until `VITE_MAP_QUERY`, `VITE_WORKSHOP_ADDRESS` and `VITE_OPENING_HOURS` are set to confirmed values.
- **Socials**: verified URLs in `VITE_INSTAGRAM_URL`, `VITE_FACEBOOK_URL` and `VITE_TIKTOK_URL` render automatically. Unverified handles are not guessed.
- **Public origin**: `src/.env.production` sets `https://arakraft.works`, used for canonical metadata, social images, structured data, robots and sitemap.

## Hosting

Cloudflare Pages builds the site from `main` on every push and serves it at https://arakraft.works. Setup steps are in [docs/cloudflare-hosting.md](docs/cloudflare-hosting.md). The Pages project builds from the `src` root directory. Node 22 is pinned in `src/.node-version`.

The previous host was Netlify. Its notes and `netlify.toml` are archived in [.archives/netlify-host/](.archives/netlify-host/).

## Conventions

See [aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions). Project-specific rules:

- Keep collection prices in `src/data/products.json` as starting prices.
- React component and hook identifiers keep React's required casing (`PascalCase` components, `useX` hooks).

## License

Proprietary, Copyright (C) 2026, araCreate Group. See [LICENSE](LICENSE).
