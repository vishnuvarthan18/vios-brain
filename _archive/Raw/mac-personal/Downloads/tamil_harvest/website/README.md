# website/

The production site. Static HTML, CSS and JavaScript, no build step. **Everything in this folder is published**, so nothing private goes here.

## Pages
| Page | Purpose |
|---|---|
| `index.html` | Home: hero, three ways in, today's Kural, the evidence, what is inside, roadmap |
| `chola.html` | The Cholas: list and article reader (28 articles, English and Tamil) |
| `literature.html` | Classical literature reader: 17 works including the complete Tirukkural |
| `about.html` | Mission, what we claim and do not, sources and licences, method |
| Lab: `brahmi-lab.html`, `scripts.html`, `grantha.html`, `vatteluttu.html`, `tamil.html`, `font.html`, `fonts.html` | Script converters, lessons, letter charts and the free Semmozhi fonts. A sub-nav links them. |
| `404.html` | Not-found page (uses root-absolute paths on purpose) |

The Explore and Engine pages were deleted on 2026-10-04 (git history keeps them); rebuild them when the engine produces their data.

## Folders
```
css/    tokens.css, components.css  <- copies of the design system (source: design/showcase/css)
        content.css                 <- maps the content pages' classes onto the tokens
        lab.css                     <- Lab pages
js/     content.js   shared header/footer/nav/theme + helpers (Site.init, Site.esc, Site.copy, ...)
        brahmi.js, maps.js, grantha-map.js   script conversion
data/   JSON read by the pages: articles/, works/, articles.json, works.json, images.json
fonts/  Semmozhi fonts (OFL 1.1) and the self-hosted web fonts
img/    textures and cursor used by the design system CSS
```

## Rules
- Pages start with `<script src="js/content.js">` and call `Site.init('<key>')`. The key highlights the nav item.
- Colours, spacing and fonts come from tokens (`var(--accent)`, `var(--text)`, `var(--space-4)`, ...). No hard-coded colours.
- Use relative paths (`css/...`, not `/css/...`) except in `404.html`.
- Data pages must cope with a missing file (show a message, do not crash).
- Run `make local` to look, `make check` before a pull request.

## Run
`make local` from the repository root, then http://localhost:8000/website/ (or the hub at /local/). Or `cd website && python3 -m http.server 8000`.

## Refresh the data
See [../docs/ENGINE.md](../docs/ENGINE.md) ("Refresh the website data").

## Web Awesome
UI components come from Web Awesome Core (MIT), copied into `vendor/webawesome/` so nothing loads from a CDN. Each page head loads `vendor/webawesome/styles/webawesome.css`, `css/wa-theme.css`, `js/wa-head.js` and the loader module (the 404 page uses absolute paths). Do not edit `vendor/`; to upgrade, copy a newer `dist-cdn` from the `@awesome.me/webawesome` npm package.

## What is on the site now (2026-10-05)
Home, Scripts (hub plus four script pages: Brahmi, Grantha, Vatteluttu, Tamil). Contact is not a page: it is a panel that slides in from the side on every page (`js/contact.js`). A page loader (`.loader`, removed by `js/wa-head.js`) shows while a page and its fonts load. Chola, Literature, About and the all-fonts page are listed in `staging-only.txt`: they stay in the project and on staging but are left out of production.

**Language:** English and Tamil. `js/i18n.js` keeps the choice in a cookie (`sem-lang`); first-time visitors get a one-time dialog with two buttons. `?lang=ta` or `?lang=en` in a link also sets it. Strings live next to each page (`js/home.js`, `js/scripts-page.js`, `js/contact.js`) and in `js/i18n-shared.js` for the header, footer and menu. Mark text with `data-i18n="key"`.

**Contact:** see `docs/CONTACT.md`.

## Photos
Home's nine tiles use photos from `design/references` (rows marked SHIP, public domain / CC0 / CC BY), resized to WebP in `img/photos/`; each tile shows its credit and links to the source. Full list: `img/photos/CREDITS.md`.
