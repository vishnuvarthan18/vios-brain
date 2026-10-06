# SOURCE

The website's own source: a Vite + React + TypeScript single page that is
pre-rendered at build time and hydrated in the browser.

| Path | Holds |
| --- | --- |
| `index.html` | HTML shell, metadata and social preview tags |
| `main.tsx` | Client entry point; hydrates the pre-rendered page |
| `app.tsx` | Page layout: header, hero, process, contact, footer, 404 view |
| `components/` | One file per page section or shared UI piece |
| `lib/` | Analytics, WhatsApp links, product prices, scroll motion |
| `types/` | Shared TypeScript types: product, analytics |
| `config/` | Business contact details and navigation |
| `data/` | JSON data: collections, prices, studio image dimensions |
| `public/` | Static assets copied as-is into `dist/`: images, favicon, robots, sitemap, headers |
| `index.css`, `studio.css` | Design tokens, layout, responsive styling, motion |
| `scripts/` | `finish-build.mjs`: pre-renders the page, adds structured data, rewrites the public origin and writes `404.html` |
| `package.json`, config files | Dependencies, npm scripts, Vite, TypeScript, Playwright, oxlint, semantic-release, Node version and `.env.*` |

Every `make` target runs from this folder, so Vite's root is here and it
reads static assets from `public/` by default. The layout follows
[`src/` by project type](https://github.com/aracreate-group/aracreate-conventions/blob/main/repo/readme.md#21-src-by-project-type).
