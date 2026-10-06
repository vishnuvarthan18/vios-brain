# Kuzhali Tuition Centre & Academy — Website

Static marketing site for **Kuzhali Tuition Centre** and **Kuzhali Academy**, Erode.
No build step, no npm, no framework. Plain HTML, CSS and JavaScript.

---

## Run it

1. Open this folder in VS Code (`File → Open Folder…`).
2. Install the **Live Server** extension when VS Code offers it (it's in `.vscode/extensions.json`).
3. Right-click `index.html` → **Open with Live Server**.

It opens at `http://127.0.0.1:5500`. Saving any file reloads the browser.

No Live Server? Any static server works:

```bash
python3 -m http.server 5500      # then open http://localhost:5500
npx serve .                      # if you have Node
```

Opening `index.html` directly with `file://` mostly works too, but Live Server is better.

---

## Folder layout

```
kuzhali-academy/
├── index.html                  ← all page content lives here
├── css/
│   ├── tokens.css              ← colours, shadows, spacing scale  ← START HERE for colours
│   ├── base.css                ← reset, typography, buttons
│   ├── layout.css              ← nav, hero, sections, footer
│   └── components.css          ← course cards, category cards, tabs, carousel
├── js/
│   └── main.js                 ← sticky nav, scroll reveal, filter tabs, WhatsApp forms
├── assets/
│   ├── illustrations/          ← 17 unDraw SVGs (MIT licence)
│   ├── icons/                  ← 6 Fluent Emoji SVGs (MIT licence)
│   └── favicon/                ← favicon, apple-touch-icon, og-image
├── site.webmanifest
├── robots.txt
├── sitemap.xml
└── .vscode/                    ← recommended extensions + Live Server port
```

---

## Things you'll want to change

### Phone number — 4 places

Your flyers show three different numbers (`(removed)`, `(removed)`, `(removed)`).
The site currently uses the first two. To change them:

| File | What to search for |
|---|---|
| `js/main.js` | `WHATSAPP_NUMBER` — country code + number, digits only, e.g. `(removed)` |
| `index.html` | `tel:(removed)` and `tel:(removed)` — appears in the nav, contact block, footer and mobile bar |
| `index.html` | `(removed)` / `(removed)` — the display text |
| `index.html` | `"telephone"` inside the `<script type="application/ld+json">` block in `<head>` |

Search-and-replace across the project: `Cmd + Shift + F` in VS Code.

### Address

Currently just "Erode, Tamil Nadu". Search `Erode, Tamil Nadu` in `index.html` — it appears in the contact block, the footer, and the structured-data block. Add the street and area, and consider adding a Google Maps link.

### Testimonials

**There are none on the page right now, on purpose** — inventing student quotes or marks would be dishonest. When you have real ones, add a section using the same card pattern as `.ccard`.

### Fees

Every course card says **"Fees on call"**. Search `Fees on call` in `index.html` and replace with real figures when you're ready to publish them.

### Colours

All in `css/tokens.css`:

| Token | Value | Used for |
|---|---|---|
| `--brand` | `#4338CA` | Headings, primary buttons, links |
| `--brand-2` | `#FF5A1F` | Accent words, CTA buttons |
| `--mint` | `#E3E0FB` | Hero background, panels |
| `--gray` | `#696984` | Body text |
| `--star` | `#FFB800` | Ratings, gold accents |
| `--red` | `#FF2D55` | The admissions band |

Change a value there and it updates everywhere.

### Fonts

**Raleway** (headings, body) and **Inter** (the big centred section titles).
Loaded from Google Fonts in `<head>`. To self-host them later, download the woff2 files into `assets/fonts/` and swap the `<link>` for `@font-face` rules.

---

## Before going live

- [ ] Replace `https://kuzhaliacademy.in/` with the real domain in `index.html` (`canonical`, `og:url`, `og:image`, `twitter:image`), `robots.txt` and `sitemap.xml`
- [ ] Confirm the correct phone numbers
- [ ] Add the full street address
- [ ] Add real testimonials and top-scorer results
- [ ] Update `<lastmod>` in `sitemap.xml`
- [ ] Test the WhatsApp button on a real phone

## Deploying

It's a static site, so any host works.

- **Netlify / Vercel / Cloudflare Pages** — drag the folder onto their dashboard, done
- **Hostinger / cPanel / any shared host** — upload the whole folder to `public_html` via FTP
- **GitHub Pages** — push to a repo, enable Pages on the `main` branch

There's nothing to build. What you see in the folder is what gets served.

---

## Licences

| Asset | Source | Licence |
|---|---|---|
| Illustrations | [unDraw](https://undraw.co) | MIT — free for commercial use, no attribution required |
| Emoji icons | [Microsoft Fluent Emoji](https://github.com/microsoft/fluentui-emoji) | MIT |
| Raleway, Inter | Google Fonts | SIL Open Font License |

All illustrations have been recoloured to the brand indigo `#4338CA` and orange `#FF5A1F`.

---

## Browser support

Chrome, Safari, Firefox and Edge — current versions and one back.
Tested clean from **280px to 1920px** wide: no horizontal scroll, no layout breaks.
The scroll animations are disabled automatically for anyone with "Reduce Motion" turned on.
