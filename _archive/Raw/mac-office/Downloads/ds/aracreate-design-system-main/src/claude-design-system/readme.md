# araCreate Design System (ACDS)

A design system for **araCreate Group** — built from the official Brand Guidelines v1.0 (2026), brand asset library, the Monument Extended typeface, and the production Webflow website export. Nothing here is invented; every color, font, radius and motif traces to a source below.

---

## Company context

**araCreate** — founded in **2003 by Aravinth Panch** — has evolved from a solo digital studio into a thriving **group of companies in Europe and Asia**, operating as an interdisciplinary ecosystem that delivers **360° services**. It empowers *"ideas from mind to market"* — crafting hardware and software, manufacturing industrial products, creating immersive media, and delivering performance marketing — across **three service verticals**:

- **Engineering** — *from Prototype to Production* (hardware & software technologies)
- **Manufacturing** — *from Drawing to Delivery* (industrial products: rapid prototyping → series production)
- **Media** — *from Sketch to Screen* (immersive storytelling & digital marketing)

**By the numbers (from the deck):** 300+ clients · 60+ new products · 8+ group companies · 3+ service verticals · 50+ service capabilities · 3+ country offices · 50+ team members · 20+ years of service. Clients range from **SMBs and social enterprises to unicorn startups, global corporations and governments**.

**Ecosystem (8+ group companies):** araCreate GmbH, BatchOne GmbH, rlgtechdesign GmbH, DreamSpace Foundation gUG (Germany); araCreate India Pvt Ltd, Micro-Tech CNC Pvt Ltd (India); araCreate Lanka Pvt Ltd, DreamSpace Foundation CLG (Sri Lanka). **HQ:** araCreate GmbH, Hubertusstr. 5, 12163 Berlin · ask@aracreate.group · +49 30 91437357.

Brand positioning: **innovation + reliability** — *"where boundless passion meets performing results."* The look is **bold, industrial and human**: engineering precision softened by a warm golden accent.

### Sources

Primary repository: **https://github.com/aracreate-group/aracreate-design-system** (`src/claude-design-system/`) — this project was imported from it; see `github.md`. Explore that repo directly for the original source material behind everything below. The source PDFs and deck page renders referenced below lived in that repo's `uploads/` folder and are **not** copied into this project (size); fetch them from the repo if you need them.

- `uploads/aracreate-brand-guidelines.pdf` — Brand Guidelines v1.0 (logo, color, type, voice, do's/don'ts)
- `uploads/aracreate-brand.pdf` — group/sub-brand lockups + color/type recap
- `uploads/aracreate-deck.pdf` — **araCreate Deck (04.02.2026)**, 28 pages. The richest source of copy, statistics, service domains, project portfolio, the Jodel case study, team, testimonials, values, ecosystem and contact details. Recreated as `ui_kits/deck/`.
- `uploads/aracreate-business-card.pdf`, `aracreate-document-template.pdf`, `aracreate-stamp.pdf`, `aracreate-tape.pdf` — brand applications. The letter header lives at `assets/brand/aracreate-letter-header.png`.
- Logo library (`.png` / `.eps`, icon variants) — kept at `assets/logos/`. **Vector SVGs** were later supplied as Affinity multi-artboard sheets; these were split into clean per-variant files with the Monument wordmark **outlined to vector paths** (no font dependency) — see `assets/logos/aracreate-*.svg`. The source sheets are kept as `assets/logos/_source-*-sheet.svg` (`_source-logo-sheet.svg`, `_source-icon-sheet.svg`, `_source-variants-sheet.svg`).
- `assets/fonts/MonumentExtended-Regular.otf`, `MonumentExtended-Ultrabold.otf` — display typeface
- **Webflow site export** (mounted at `website-webflow/`) — production HTML/CSS; source of UI values, copy, imagery and iconography.
- **Live CMS website export** (mounted at `aracreate-cms-website/`, published Jun 2026) — the current production site. Confirms the live palette, the WebFont stack (`Poppins`, `Red Hat Mono`, **`Inconsolata`** — newly added), the animated isometric "machine" hero, and adds CMS-driven surfaces: `index/about/engineering/manufacturing/media/projects/impact/contact`, a **blog** (`blogs.html` + `blogs/*`), and CMS lists (`lists/clients.html`, `lists/team.html`, `lists/testimonials.html`). Legal pages live under `etc/`.

---

## CONTENT FUNDAMENTALS

**Voice (from guidelines):** professional, clear, and friendly — *"simple to understand for people working on real projects."* Four tone pillars: **Innovative** (energetic, dynamic), **Trustworthy** (authority), **Professional** (formal, dependable), **Friendly** (hospitable, inclusive).

**Casing:** Sentence case for headlines and body. **Wide-tracked UPPERCASE** reserved for buttons, eyebrows and small labels (e.g. `LET'S TALK ABOUT YOUR IDEA`, `TRUSTED BY`). Section eyebrows are numbered: `01 — araCreate Group`, `02 — About`, `03 — Services`.

**Person:** "We" for the company, "you/your" for the client (*"Let's Talk About Your Idea"*, *"bring your idea from mind to market"*). Warm and direct.

**Signature phrasings:**

- *"Empowering ideas from mind to market"* (master tagline)
- *"360° services"*
- Vertical couplets: *"from Prototype to Production"*, *"from Sketch to Screen"*, *"from Drawing to Delivery"*
- Section headlines are aspirational and trail off with an ellipsis: *"Where boundless passion meets performing results …"*, *"Where complex ideas need broader services …"*, *"Where exceptional ideas become success stories …"*

**Emoji:** none. The brand never uses emoji. Iconography is isometric line illustration instead.

**Vibe:** confident, industrial, human. Engineering precision softened by a warm golden accent and friendly directness.

---

## VISUAL FOUNDATIONS

**Color.** A deliberately tight palette: **Golden Sun `#F9BF3B`** (primary accent) + **Graphite Gray `#555555`** (primary text/secondary brand), grounded by **Black `#222222`**, hairline **Stroke `#CECECE`**, and off-white **Canvas `#F6F6F6`**. Golden Sun appears at full strength on CTAs and as 80/90% tints for light buttons, plus a pale `#FDF3D8` wash for tinted sections. *Do not introduce hues outside this palette* (a guideline don't). Status colors are kept functional and muted.

**Type.** Strictly separated by role:

- **Monument Extended** — the **logo wordmark ONLY** (UPPERCASE "ARACREATE", Ultrabold 800). Never used for headings, titles, or body. Exposed as `--ac-font-logo`. The logo ships as outlined-path **SVG** (preferred) plus PNG/EPS art, so this token is only for typesetting the wordmark when vector art isn't used.
- **Poppins** — **everything else**: all headings, titles, body, UI, documents. The site leans on **Light (300)** for headings and body, **Medium (500)** for UI labels, **Bold (700)** for emphasis/metrics. Body runs loose (line-height \~1.9) with `0.02em` tracking. Exposed as both `--ac-font-text` and `--ac-font-display` (headings).
- **Red Hat Mono** — specs, data, code (loaded on the live site).
- **Inconsolata** — also requested by the live site's WebFont loader (secondary monospace / tabular use); added to `tokens/fonts.css`. Red Hat Mono remains the primary `--ac-font-mono`.

**Group facts (from the deck, verbatim — never invent beyond these):** 300+ clients · 60+ new products · 8+ group companies · 3+ service verticals · 3+ country offices · 50+ service capabilities · 50+ team members · 20+ years. **Verticals:** Engineering (*Prototype → Production*), Manufacturing (*Drawing → Delivery*), Media (*Sketch → Screen*). **Values:** "We pursue excellence / collaborate seamlessly / deliver results / stay curious." **Ecosystem:** araCreate GmbH, BatchOne GmbH, rlgtechdesign GmbH, DreamSpace Foundation gUG (Germany); araCreate India Pvt Ltd, Micro-Tech CNC Pvt Ltd (India); araCreate Lanka Pvt Ltd, DreamSpace Foundation CLG (Sri Lanka). **HQ:** araCreate GmbH, Hubertusstr. 5, 12163 Berlin · ask@aracreate.group · +49 30 91437357.

**Spacing & layout.** 4px base scale (4 → 128). Content max-width \~940–1200px, centered. Generous vertical rhythm; numbered sections.

**Backgrounds.** Off-white canvas as default; soft golden wash (`#FDF3D8`) and dark (`#222`) for alternating bands — **max 1–2 background colors per page**. Imagery is **duotint** (warm gold/gray treatment, not full color). Decorative **isometric vector illustrations** (the guideline-mandated imagery style) — incl. animated "machine" sprites on the live site. No photographic gradients, no glassmorphism.

**Borders & corners.** Hairline `1px #CECECE`. Radii: **buttons are square (0)**, inputs 4px, panels 9px, **cards 20px**, pills/avatars full. Toggles use square tracks.

**Shadows.** Low, neutral, never colored — soft multi-layer ambient shadows (`--ac-shadow-sm/md/lg`). Hover uses a single soft lift shadow (`--ac-shadow-lift`).

**Motion.** Long, gentle eases — `cubic-bezier(0.165, 0.84, 0.44, 1)`, durations 0.5–0.8s. Signature interaction: elements **translate up \~5px (and slightly scale)** on hover with a soft shadow; underlines wipe in left-to-right. Press states settle back down/scale slightly under 1. Respect `prefers-reduced-motion`.

**Hover/press.** Primary button: Golden Sun → 90% tint + lift. Secondary (graphite) → yellow on hover. Links: underline reveals/brightens. Cards: lift + deepen shadow.

**Transparency & blur.** Sparingly — a frosted (`backdrop-filter: blur`) sticky navbar; golden tints at 80/90% opacity for layered fills. No heavy glass.

---

## ICONOGRAPHY

The brand's icon language is **isometric line illustration**, matching the guideline imagery style. Service/vertical icons are line-drawn isometric SVGs (`assets/icons/icon-*.svg`) — product design, web development, brand identity, graphic design, advertising campaigns, strategy & marketing, execution & production, multidisciplinary. UI chrome uses simple stroked SVGs (arrowheads, arrow-top, link). Social marks (`logo-instagram.svg`, `logo-youtube.svg`) are copied in.

- **No emoji, no icon font for content**, no unicode glyphs as icons. (Webflow ships a hidden `webflow-icons` font for internal slider/nav chrome only — not for design use.)
- Larger **isometric illustrations** (`assets/illustrations/`: circuit-board, charts, image-creation, human-computer-interaction, customer-service, decorative elements) carry hero/section art.
- The live site's animated multi-layer "machine" sprites are bespoke and intentionally **not** recreated here; substitute the simpler shipped illustrations.
- When a needed icon is missing, prefer adding to the isometric line set; if substituting from a CDN, match the thin single-weight line style and flag it.

---

## VISUAL ASSETS

`assets/`

- `fonts/` — Monument Extended Regular + Ultrabold (OTF).
- `logos/` — **vector SVG** (outlined paths, no font dependency) and PNG for: primary wordmark + monogram/icon in default / negative / on-graphite (t-w-b-g) / on-gold (t-w-b-y); araCreate Group, India & Lanka regional lockups; Engineering / Manufacturing / Media division lockups. SVG is preferred everywhere (crisp at any size, recolorable). `_source-*-sheet.svg` are the original Affinity export sheets.
- `brand/` — stamp/seal, letter header.
- `icons/` — isometric service icons + UI/social SVGs.
- `illustrations/` — isometric scene illustrations + decorative elements.
- `imagery/` — duotint hero/process/team photos, founder & character-design imagery.

---

## INDEX / MANIFEST

**Root**

- `styles.css` — entry point (imports only). Consumers link this.
- `tokens/` — `fonts.css` (@font-face + Google import), `colors.css`, `typography.css`, `spacing.css` (radius/shadow/motion).
- `readme.md` — this file. `SKILL.md` — Agent-Skills wrapper.

**Components** (`window.AraCreateDesignSystem_4716e7.*`)

- `components/core/` — Button, SectionLabel, Input, Textarea, Select, Checkbox, Radio, Switch, Card, Badge, Tag, Avatar, Tabs
- `components/content/` — ServiceCard, StatBlock

**Foundations** (Design System tab cards) — `foundations/*.card.html`: color core/tints/ramp; display/heading/body/label type; spacing scale/radii/shadows; brand logos/monogram/stamp/icons. **Slides** group: `ui_kits/deck/` cards.

**Templates** (starting folders for consuming projects)

- `templates/deck/` — `Deck.dc.html`: 1280×720 slides in the brand deck style (gold title, canvas story, stats grid, section divider, contact) with the triple-chevron footer motif.
- `templates/marketing-page/` — `MarketingPage.dc.html`: homepage-style marketing layout (sticky nav, hero, trusted-by, about + stats, services grid, dark contact band, footer) composed from the DS components. Replaces the old `@startingPoint` tags.

**UI kits**

- `ui_kits/website/` — interactive recreation of the araCreate Group marketing site (Navbar, Hero, TrustedBy, About, Services, Projects, Contact, SiteFooter).
- `ui_kits/deck/` — sample pitch deck recreated from `aracreate-deck.pdf`: navigable `index.html` + `slides.jsx` (TitleSlide, SectionSlide, StorySlide, StatsSlide, VerticalSlide, ProjectGridSlide, CaseStudySlide, TestimonialsSlide, ValuesSlide, ContactSlide).

---

*Source PDFs:* the brand-guideline / deck / stationery PDFs and deck page renders are **not** carried in this project — their content is fully absorbed into the tokens, foundations, components and UI kits here. Originals remain in the source repo under `src/claude-design-system/uploads/`.

*Open thread:* the live CMS site is managed in **Webflow**. Verified against production on 2026-08-17 (site `aracreate.group`, id `63780fb6eec282197fc5547f`, last published 2026-06-09):

- **Colors match.** Golden Sun `#f9bf3b`, Graphite Gray `#555`, canvas `#f6f6f6` and the 0.6/0.8/0.9 tints are identical to the live Webflow "Base collection".
- **Four live variables were missing** and have been added to `tokens/colors.css`: `--ac-photo-overlay` (`#2e419e`, the duotint/photo wash), `--ac-true-black` (`#000`, distinct from the brand near-black `#222`), `--ac-gray-translucent` and `--ac-button-gray-light`.
- **Fonts.** Webflow hosts no custom fonts, so Poppins is served from a webfont service. Monument Extended is licensed for the **logo wordmark only** and is never used for headings or body — `tokens/typography.css` already enforces this via `--ac-font-logo`.

### Live page architecture

Every araCreate page follows the same skeleton, confirmed by reading the live element trees:

```
Body
  └ menu / logo / right-line / right-section-seperator   (global component instances)
  └ .sections
      └ .section                       ← default canvas band
          └ .container (+ .hero)
          └ .banner-photo.<page>-page-banner
      └ .section.bg-brand-yellow-light  ← alternating pale-gold band
  └ footer
```

The rhythm is the design's backbone: full-width sections alternating between canvas `#f6f6f6` and the pale gold wash, each holding one centred `.container`. Page-specific banners are `.banner-photo` plus a page modifier (`.clients-page-banner`, `.projects-page-banner`).

**DTF** (Deep Tech Foundry — an araCreate venture) runs the same system under a parallel `-dtf` namespace: `.section-dtf`, `.container-dtf`, `.dtf-bg-brand-yellow-light`, with its own `dtf-header` and `dtf-footer` components. Same alternating bands, same palette; treat it as a sub-brand of the same system rather than a separate one.

### Live component inventory (Webflow)

These are the reusable components defined in the production site — page-level composites rather than primitives. The UI kits compose the equivalents; the `components/` primitives in this project are the smaller pieces those composites are built from.

| Component | Instances | Notes |
| --- | --- | --- |
| `menu` | 37 | Global nav with full-screen overlay |
| `logo` | 36 | Wordmark, image-prop driven |
| `footer` | 28 | Current footer |
| `right-line` | 15 | Fixed right-edge rule |
| `right-section-seperator` | 13 | Section divider on the right margin |
| `cta` | 8 | Call-to-action band |
| `projects-slider-section` | 7 | Project carousel |
| `trusted-by` | 6 | Client logo strip |
| `footer-old` | 6 | Legacy, archive pages only |
| `faq` | 3 | Accordion |
| `loader` | 2 | Page loader |
| `dtf-header` / `dtf-footer` | 2 each | DTF sub-brand chrome |
| `cms-testimonial`, `old-logo-slider` | 0-1 | Unused / legacy |

Live pages worth reading for further design context: Impact, Engineering, Manufacturing, Media, Blogs, Projects, About, Contact, and the `/dtf/` sub-site. Pages under `/archive/` and `/template/` are legacy and should not be used as reference.

*Substitution flag:* none required — Monument Extended and Poppins are both available (Monument from the supplied OTFs, Poppins + Red Hat Mono + Inconsolata from Google Fonts).

*Note on the live site's hero:* the production CMS site animates bespoke multi-layer isometric "machine" SVG sprites (`machine-01…`, `machine-03…`). These are intentionally **not** recreated here — the shipped static isometric illustrations in `assets/illustrations/` substitute for them.
