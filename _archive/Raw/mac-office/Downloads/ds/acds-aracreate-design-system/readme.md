# araCreate Design System (ACDS)

A design system for **araCreate Group** — built from the official Brand Guidelines v1.0 (2026), the brand asset library, the Monument Extended typeface, the araCreate deck of 4 February 2026, and the production Webflow website. Nothing here is invented; every colour, font, radius and motif traces to a source below.

This project is **ACDS (17 August 2026)**. On **21 August 2026** a second, parallel design system — the "araCreate Design System" built on 20 August — was folded into it. ACDS is the base: its structure, its file naming and its token values won. What the 20 August system brought with it is listed under [What the merge brought in](#what-the-merge-brought-in), and every value that moved is recorded in [`changelog.md`](changelog.md).

---

## Which stylesheet to link

**Read this before anything else. There are two entry points and they are not interchangeable.**

- **`system.css` — the whole system.** Tokens, element defaults, the signature devices, components, page sections, the application surfaces, the density scale, the dark theme and the deck. **New consumers link this.**
- **`styles.css` — the smaller, older entry point.** Four token imports (`fonts`, `colors`, `typography`, `spacing`) and nothing else. It is pinned by decision, because the locked files listed under [Locked files](#locked-files) link it and must keep working unchanged. It has always meant exactly that, and it is not to be "upgraded" into a second full-system file.

A page that links `styles.css` gets tokens and no component CSS. That is the intended behaviour, not a bug. If a page renders unstyled components, it linked the wrong file.

`styles.css` sits at the root and is referenced by `system.css`, `thumbnail.html`, the two `templates/*/ds-base.js` files and `tokens/fonts.css`. It is exactly what it claims to be: four `@import` lines — `tokens/fonts.css`, `tokens/colors.css`, `tokens/typography.css`, `tokens/spacing.css` — and no rules of its own.

---

## Company context

**araCreate** — founded in **2003 by Aravinth Panch** — has evolved from a solo digital studio into a thriving **group of companies in Europe and Asia**, operating as an interdisciplinary ecosystem that delivers **360° services**. It empowers *"ideas from mind to market"* — crafting hardware and software, manufacturing industrial products, creating immersive media, and delivering performance marketing — across **three service verticals**:

- **Engineering** — *from Prototype to Production* (hardware and software technologies)
- **Manufacturing** — *from Drawing to Delivery* (industrial products: rapid prototyping through series production)
- **Media** — *from Sketch to Screen* (immersive storytelling and digital marketing)

**By the numbers (from the deck):** 300+ clients · 60+ new products · 8+ group companies · 3+ service verticals · 50+ service capabilities · 3+ country offices · 50+ team members · 20+ years of service. Clients range from **SMBs and social enterprises to unicorn startups, global corporations and governments**.

**Ecosystem (8+ group companies):** araCreate GmbH, BatchOne GmbH, rlgtechdesign GmbH, DreamSpace Foundation gUG (Germany); araCreate India Pvt Ltd, Micro-Tech CNC Pvt Ltd (India); araCreate Lanka Pvt Ltd, DreamSpace Foundation CLG (Sri Lanka). **HQ:** araCreate GmbH, Hubertusstr. 5, 12163 Berlin · ask@aracreate.group · +49 30 91437357.

Brand positioning: **innovation and reliability** — *"where boundless passion meets performing results."* The look is **bold, industrial and human**: engineering precision softened by a warm golden accent.

**Sub-brands and ventures.** **DTF — Deep Tech Foundry** is an araCreate venture running this same system under a parallel `-dtf` namespace; a sub-brand, not a separate brand. **araCreate Academy** and **araCreate Meditate** are the group's own properties and among the intended consumers of this system.

### Sources

Primary repository: **https://github.com/aracreate-group/aracreate-design-system** (`src/claude-design-system/`) — ACDS was imported from it; see [`github.md`](github.md). The source PDFs and deck page renders lived in that repo's `uploads/` folder and are **not** copied into this project; fetch them from the repo if you need them.

- `uploads/aracreate-brand-guidelines.pdf` — Brand Guidelines v1.0 (logo, colour, type, voice, do's and don'ts)
- `uploads/aracreate-brand.pdf` — group and sub-brand lockups, colour and type recap
- `uploads/aracreate-deck.pdf` — **araCreate Deck (04.02.2026)**, 28 pages. The richest source of copy, statistics, service domains, project portfolio, the Jodel case study, team, testimonials, values, ecosystem and contact details.
- `uploads/aracreate-business-card.pdf`, `aracreate-document-template.pdf`, `aracreate-stamp.pdf`, `aracreate-tape.pdf` — brand applications.
- Logo library (`.png` / `.eps`, icon variants) and the **vector SVGs** split from the Affinity multi-artboard sheets, with the Monument wordmark **outlined to vector paths** (no font dependency).
- `MonumentExtended-Regular.otf`, `MonumentExtended-Ultrabold.otf` — display typeface, commercial licence.
- **Webflow site export** — production HTML and CSS; source of UI values, copy, imagery and iconography.
- **Live CMS website** (published June 2026) — confirms the live palette, the WebFont stack and the CMS-driven surfaces. Everything verified against production is recorded in [`docs/live-site.md`](docs/live-site.md).

The second system folded in on 21 August 2026 was itself a port of araCreate's own design-system repository (proprietary, © 2026 araCreate Group; author Vishnu). Its ported CSS and JavaScript are the files now in `styles/` and `js/`; its documentation is in `docs/`. That repository cites the same Brand Guidelines and deck, plus a measured audit of 72 pages of aracreate.group.

---

## CONTENT FUNDAMENTALS

**Voice (from the guidelines):** professional, clear and friendly — *"simple to understand for people working on real projects."* Four tone pillars at once: **Innovative** (energetic, dynamic), **Trustworthy** (has authority), **Professional** (formal, dependable), **Friendly** (hospitable, inclusive). The shorthand: **confident, industrial, human.**

**Person.** "We" for araCreate, "you" for the client. Not "clients can" — "you can". Not "araCreate delivers" in a sentence araCreate is writing; "we deliver". Warm and direct.

**Casing.** Sentence case for headlines and body — never Title Case. **Wide-tracked UPPERCASE** is reserved for buttons, eyebrows and small labels (`LET'S TALK ABOUT YOUR IDEA`, `TRUSTED BY`), which is why a heading set in caps looks wrong here: caps are already spoken for. Section eyebrows are numbered — `01 — araCreate Group`, `02 — About`, `03 — Services` — and the number is part of a page's rhythm.

**Sentences.** Short, one idea each. Say the thing, then stop: no wind-up, no "in today's fast-moving world". Concrete over abstract — "Rapid prototyping through series production" beats "end-to-end manufacturing solutions". No jargon the client has not used first. Section headlines are the one place the brand is allowed to be aspirational, and they trail off with an ellipsis: *"Where boundless passion meets performing results …"*, *"Where complex ideas need broader services …"*, *"Where exceptional ideas become success stories …"*

**Microcopy.** Buttons say what happens, in the visitor's terms: "Send message", not "Submit". "Let's talk about your idea" is the brand's own CTA and is worth reusing. Errors say what to do, not that something went wrong — "That email address is missing an @", never "Invalid input", and never blame the visitor. Empty states say what to do next, not "No data". Hints go under the field, phrased as help rather than warning.

### Never

- **No emoji.** Not in copy, not as an icon, not in a heading, not as a bullet. This is absolute; the brand has an isometric line-illustration language instead.
- **No exclamation marks** in body copy. Friendly comes from directness.
- **No invented facts.** Every figure, company name and address a page may state is fixed in [`docs/brand-facts.md`](docs/brand-facts.md). `300+ clients` is the fact — **the plus sign is what makes it true**; `300 clients` is a different and untrue claim. If a number is not on the list, ask; do not estimate. Where filler is genuinely needed use obvious filler (`Client name`, `00+`, `Lorem ipsum`): a visible placeholder gets replaced, a plausible invention does not.
- **No "click here".** Link text says where it goes. This is also an accessibility requirement — a screen reader can list links out of context.
- **No shouting a whole sentence** in caps.
- **No Monument Extended for headings**, titles, slide titles, body copy or statistics. It is the logo wordmark and nothing else.
- **No ninth group company.** The ecosystem is the eight named below, and `8+` is how the figure reads.

**Fixed strings — never reworded.** The tagline *Empowering ideas from mind to market*. The positioning line *where boundless passion meets performing results*. The three vertical couplets, which are part of each vertical's name: Engineering *from Prototype to Production*; Manufacturing *from Drawing to Delivery*; Media *from Sketch to Screen*. The four values, first-person plural: *We pursue excellence · We collaborate seamlessly · We deliver results · We stay curious*.

**The name.** araCreate — lowercase a-r-a, capital C. Not "AraCreate", not "Aracreate", not "ARACREATE". Full capitals ARACREATE is correct **only** as the logo wordmark set in Monument Extended: never body copy, never a heading, never a page title. `araCreate Group` is the parent; a single company is `araCreate GmbH`. Legal suffixes are part of the name — `GmbH`, `gUG`, `Pvt Ltd`, `CLG` are not interchangeable and not optional. `rlgtechdesign` is one lowercase word.

**The figures you may state**, verbatim, plus signs included: 300+ clients · 60+ new products · 8+ group companies · 3+ service verticals · 50+ service capabilities · 3+ country offices · 50+ team members · 20+ years of service. Do not compute anything from these — "20+ years since 2003" is arithmetic on a rounded figure and will be wrong.

**The 8+ group companies.** Germany: araCreate GmbH, BatchOne GmbH, rlgtechdesign GmbH, DreamSpace Foundation gUG. India: araCreate India Pvt Ltd, Micro-Tech CNC Pvt Ltd. Sri Lanka: araCreate Lanka Pvt Ltd, DreamSpace Foundation CLG.

**Not facts, absent on purpose** — revenue, headcount by office, growth figures, named clients, named team members other than the founder, certifications, awards, memberships, pricing, or any date other than the 2003 founding.

[`docs/brand-facts.md`](docs/brand-facts.md) is the source of truth. This section summarises it; if the two ever disagree, that file wins.

---

## VISUAL FOUNDATIONS

**Naming convention.** American spelling in identifiers (`gray`, `color`, `center`), British spelling in prose. `--ac-graphite-gray` is the token; grey is the colour.

**Colour.** A deliberately tight palette: **Golden Sun `#f9bf3b`** (the accent) and **Graphite Gray `#555555`** (primary text and secondary brand), grounded by brand black `#222222`, hairline stroke `#cecece` and off-white canvas `#f6f6f6`. White `#ffffff` is the *raised* surface, not the page. Golden Sun appears at full strength on CTAs, at 90% and 80% for hover and pressed, and as a pale `#fdf3d8` wash for tinted bands. *Do not introduce hues outside this palette.* `#2e419e` is the duotint photo wash and is not a text or surface colour.

**Golden Sun is never text.** Gold on white measures 1.67:1 and white on gold measures the same — it fails in both directions at every size, including a 105px slide title. There is deliberately no token for gold text. Gold is something you put *behind* type; brand black on gold measures 9.51:1. On a dark surface gold is legible (8.11:1), which is why the eyebrow and the decorative quote mark turn gold inside an inverse surface and are brand black everywhere else.

**Feedback colours.** araCreate has no brand colours for these, so warning reuses gold and information is graphite; only danger and success introduce a hue. `--ac-danger` is `#c0492f` (4.59:1 on canvas) and `--ac-success` is `#186a43` (6.11:1 on canvas).

**Surfaces, not variants.** Surfaces — page, card/raised, subtle, dark, accent and accent-soft — each re-point every colour token inside them. Put the surface class on the *band*, not on each thing inside it; that is why no component in this system has a dark variant, and forgetting it produces exactly one bug (light components with white text). `--ac-surface-dark` is brand black `#222222`.

**Dark theme.** `data-ac-theme="dark"` on any element re-themes everything inside it; `data-ac-theme="auto"` follows the operating system. It is one file — `tokens/theme-dark.css` — and it touched no component, because no component contains a raw colour. Three things needed more than a token swap and are documented in that file: **Golden Sun does not move** (lightening it would give the brand two yellows, and it already reads correctly on a dark surface); **the inverse surface flips to light**, because a dark band on a dark page is invisible; and **error and success need dark-specific values**, because the light pair fails on a dark page.

**Density and target size.** Comfortable is the default. `data-ac-density="compact"` on a region tightens control padding, table cells, card bodies and stack gaps for signed-in, data-dense screens — scoped to `pointer: fine`, so on a touch device it tightens the type rhythm but the target floor holds. **Every interactive control is at least 44px** in both directions, applied as padding and min-height, never by growing the type: a 12px label inside a 44px target is fine, a 12px target is not. Checkboxes and radios stay 18px and their `<label>` carries the target, which is why the control is wrapped in one. Links inside running text are exempt — one cannot be 44px tall without wrecking the line.

**Type.** Strictly separated by role:

- **Monument Extended** — the **logo wordmark ONLY** (uppercase ARACREATE, Ultrabold 800). Never headings, titles, slide titles, body or statistics. Exposed as `--ac-font-logo`; `.ac-wordmark` is the only selector allowed to use it. The logo ships as outlined-path SVG, so this token is only for typesetting the wordmark when vector art cannot be used.
- **Poppins** — **everything else**: headings, titles, body, UI, documents. The system leans on Light (300) for headings and body, Medium (500) for UI labels, Bold (700) for emphasis. Exposed as both `--ac-font-text` and `--ac-font-display`.
- **Red Hat Mono** — specs, data and code. `--ac-font-mono`.

`tokens/fonts.css` loads Poppins and Red Hat Mono from Google Fonts (both SIL Open Font License) and declares the two Monument Extended weights. The `.otf` files those `@font-face` rules point at are in this tree, in `assets/fonts/`.

**The type scale.** The heading ladder is locked to the live site exactly: 36 / 33 / 31 / 28 / 21 / 16px, around 3px apart, in Light (300) — a known, deliberate deviation, recorded at the foot of `tokens/typography.css`. A separate display tier sits above it for the two or three places per site that are display type: 64 / 48 / 36 / 28px, and it is the one place weight 600–700 is used for size rather than emphasis. Body is 16px with `--ac-leading-body` 1.8 and `.02em` tracking. `--ac-text-sm` and `--ac-text-link` are both **14px**; `--ac-tracking-label` is `0.07em`. Statistics are 42px. Navigation, footer, breadcrumb and pagination links carry a 44px hit area from `tokens/density.css`.

**Spacing.** ACDS's indexed ladder, `--ac-space-1` through `--ac-space-16`: 4, 8, 10, 12, 15, 16, 20, 24, 30, 40, 60, 80, 90, 100, 120, 140. `--ac-space-7` (20px) is the most-used value on the live site, 153 times. Sections are 90px top and bottom; a tall band is 140px. Containers: `--ac-container` 940px is what every `.container` on aracreate.group actually measures, `--ac-container-wide` 1200px is the outer bound, and body copy caps at 74ch.

**Corners.** ACDS's radii: `--ac-radius-xs` 2px, `--ac-radius-sm` 4px (inputs and small controls), `--ac-radius-md` 9px (panels and image frames), `--ac-radius-lg` 20px (cards), `--ac-radius-pill` 200px (pills and avatars), `--ac-radius-none` 0 (buttons are square). **Anything carrying the signature edge stays square regardless**, because `--ac-edge-radius` governs those and it is a device ACDS does not have. Do not write a `border-radius` in a component; if something must change shape, that is a decision about the whole system and it belongs in `tokens/spacing.css`.

**Borders and the signature edge.** Hairline `1px #cecece`. The signature device is a 1px edge that is **dotted on the top and right, solid on the bottom and left**, with square corners — the panel reads as drawn by hand and still open on two sides. It is applied by default to panels, cards, modals, text inputs, textarea, select, table, blockquote and `figure img`. `.ac-edge-none` opts out; the exception is what you declare, not the rule. Buttons are excluded on purpose — a dotted border round a yellow fill reads as a rendering fault. Adjacent panels drop the shared edge so two 1px borders never stack into a 2px seam. Two border colours exist because they answer different questions: a divider may be faint `#cecece` (`--ac-border`), a control edge must reach 3:1 and uses graphite (`--ac-border-control`, `--ac-border-strong`).

**Shadows.** Four, and a component picks one — never composes its own, and never a coloured one. `--ac-shadow-sm` is the resting shadow, `--ac-shadow-lift` is hover, `--ac-shadow-md` is a dropdown or a toast, `--ac-shadow-lg` is a modal. araCreate reads industrial and flat, not glassy.

**Motion.** Three durations — `--ac-duration-fast` .25s, `--ac-duration` .5s, `--ac-duration-slow` .8s — and one easing, `cubic-bezier(.165, .84, .44, 1)`: a fast start settling gently, never a bounce, never a spring. Named devices: the animated 2px underline, which wipes in from the left on hover rather than fading (the single most-used device on the live site, 2,259 elements); scroll reveals that rise 20px and fade in, staggered 80ms per child; a line that draws itself by scaling from its left edge; the 40-second marquee, paused on hover; a sticky header that shrinks 15px once the page scrolls; a 3px gold scroll-progress bar. Every animation stops for a visitor who has asked their operating system for less motion, and nothing is ever hidden from someone who cannot receive the animation.

**Hover and press.** Buttons lift: `translateY(-5px) scale(1.02)` with the lift shadow on hover, `translateY(-2px) scale(1.01)` on press, and the gold goes to 90% then 80% — darker, never lighter. Ghost buttons neither lift nor shadow. Links darken from graphite to brand black and grow their underline. Cards lift and close their dotted edge. Service-list rows indent 12px and their arrow slides 6px right. Logo tiles come out of greyscale. The focus ring is one global style — 2px, 2px offset, `--ac-focus` — and removing it is one of three things you may not undo.

**Transparency and blur.** No glassmorphism, no backdrop blur, no frosted panels. Transparency appears in a handful of declared places: the modal scrim (graphite at 80%, so the page reads as dimmed rather than switched off), the gold hover and pressed states, the token-level translucent feedback surfaces at roughly 8%, and the marquee's mask, which fades its two ends so items appear rather than being clipped. There are no protection gradients — text over imagery uses a solid band or the negative logo, not a scrim gradient.

**Backgrounds and layout.** Off-white canvas is the default; the page rhythm is full-width bands alternating canvas → pale gold `#fdf3d8` → canvas, with occasional brand-black and Golden Sun bands — **max one or two background colours per page**. Sticky header, 90px falling to 80px at 1280 and 75px at 1440. Edge-to-edge bands use a measured viewport width (`--ac-vw`) rather than `100vw`, because `100vw` includes the scrollbar and is the usual cause of a few pixels of horizontal scroll. Breakpoints: 479 / 767 / 991 down, 1280 / 1440 / 1920 up.

**Imagery.** Photography is **duotint**: composited over `#2e419e` at low opacity so it reads warm gold and cool grey rather than full colour. A raw full-colour photograph is not an araCreate image; treat it, or use an illustration instead. No photographic gradients. `figure img` carries the signature edge; a bare `<img>` used as a logo or icon does not.

**Slides.** A slide is 16:9 with `container-type: size`, and every size inside it is written in `cqw`, so the same markup is a thumbnail, a projected slide and a PDF page with no media queries. 1280px is the reference. Three slide surfaces — canvas, gold, brand black — and the modifier must be paired with the surface class, or a heading on gold renders at 4.45:1. Emphasis in a slide title is *weight*, not colour. Slide numbers come from a CSS counter, so inserting a slide renumbers the deck. The deck's own motif is three chevrons at the bottom right, drawn in CSS from the monogram's shape so they inherit the slide's colour.

### The three accessibility exceptions

Three values are **deliberately not ACDS's**, on the client's instruction, each a repair rather than a preference. They are the only places the merge overrode the base.

| Token | ACDS value | Value here | Why |
| --- | --- | --- | --- |
| `--ac-focus` | Golden Sun | brand black `#222222` | Gold measures **1.55:1** on canvas — a focus ring nobody can see is not a focus ring. |
| `--ac-success` | `#4f9d69` | `#186a43` | `#4f9d69` measures **3.06:1**; `#186a43` measures 6.11:1 on canvas. |
| `--ac-text-on-accent` | Graphite Gray | brand black `#222222` | Graphite on gold measures **4.45:1** — it misses 4.5:1 by 0.05. |

### The one knowingly-kept failure

`--ac-text-muted` is `#8a8a8a`, ACDS's value, and it measures **3.19:1** on canvas — below the 4.5:1 that normal text requires. It is kept at the client's explicit instruction, recorded here rather than in a commit message so that nobody discovers it in an audit and assumes it was missed.

**It is reversible in one line.** In `tokens/colors.css`, change `--ac-text-muted` to `var(--ac-gray-450)` (`#6f6f6f`, 4.65:1 on canvas, already declared in the same file as the derived accessible grey). Nothing else needs touching, because no component holds a raw colour.

---

## ICONOGRAPHY

**The brand's icon language is isometric line illustration** — thin, single-weight strokes drawn in three-quarter perspective. This is guideline-mandated, not a preference, and it is why an araCreate icon never looks like a Material icon. All of it is SVG.

- **Service and capability icons** (isometric, 48–96px): product design, web development, brand identity, graphic design, advertising campaigns, brand strategy, strategy and marketing, execution and production, multidisciplinary.
- **UI chrome** (simple stroked shapes, not isometric): arrowhead left and right, arrow-top, link-sharp, locations.
- **Social**: Instagram and YouTube marks. Other companies' marks — do not restyle them.
- **Illustrations** — larger isometric scenes for heroes and section art: circuit board, charts, image creation, human-computer interaction, customer service, plus decorative elements.

**There is no icon font and no icon-library dependency.** Nothing is loaded from a CDN. Small interface glyphs — arrow, check, close, chevron, clock, mail, pin, cog, alert — are drawn as an inline `<symbol>` sprite in the page and referenced with `<use href="#ac-i-…">`, matching the `.ac-icon` contract: 1.15em square, `fill:none`, `stroke:currentColor`, `stroke-width:1.5`, round caps and joins. The `Icon` component in `components/core/` wraps that contract. Icons inherit `currentColor`, so never set their colour directly — set the text colour of what contains them.

**Emoji are never used**, as icons or anywhere else. Neither are unicode characters standing in for icons; the one exception is genuine punctuation used as punctuation (the `·` separator in a marquee or a meta row, and the `»` in the slide page marker). Webflow ships a hidden `webflow-icons` font on the live site for internal slider and nav chrome only — it is not for design use.

**Every icon is one of two things and must declare which**: decoration (`aria-hidden="true"`) or content (`role="img"` with a label). The system draws a dashed red outline round any icon that declares neither, and round any icon-only button without an `aria-label`, so the omission is visible rather than silent.

**When the icon you need does not exist**: draw it in the isometric line style, or borrow one that matches the thin single-weight line style and flag the substitution. Never fall back to an emoji or an icon font.

---

## VISUAL ASSETS

The brand asset library is **carried in this tree**, under `assets/`, in six folders: `brand/` (the letterhead and the stamp, 2 files) · `fonts/` (the two Monument Extended `.otf` files) · `icons/` (17 SVG) · `illustrations/` (8 SVG) · `imagery/` (5 duotint `.jpg`) · `logos/` (21 SVG and 11 PNG, plus three `_source-*.svg` Affinity multi-artboard sheets the individual files were split from). `tokens/fonts.css` points its two `@font-face` rules at `assets/fonts/` and `thumbnail.html` loads `assets/logos/aracreate-logo-default.svg`; both resolve. The rules for using the assets hold regardless of where they are served from.

**Logos — read the chip, not the name.** Each logo carries a coloured block behind CREATE, sized for one specific background.

| Background | File |
| --- | --- |
| Canvas, white, **or gold** | `aracreate-logo-default.svg` (on gold the chip vanishes, leaving clean graphite type) |
| Brand black | `aracreate-logo-t-w-b-y.svg` — white type on a gold chip |
| Graphite `#555` | `aracreate-logo-t-w-b-g.svg` |
| Over imagery | `aracreate-logo-negative.svg` (a knockout — *not* the dark-background variant, despite the name) |
| Favicon, avatar, app icon | `aracreate-icon-default.svg`, `-t-w-b-g`, `-t-w-b-y`, `-negative` |
| Wordmark alone, no chip | `aracreate-wordmark-default.svg`, `-t-w-b-g` |

Group and regional lockups: `aracreate-group-logo-*`, `aracreate-india-logo-*`, `aracreate-lanka-logo-*`. Division lockups, one per vertical: `aracreate-engineering-logo.svg`, `aracreate-manufacturing-logo.svg`, `aracreate-media-logo.svg` — use the division lockup on a division site and the plain logo everywhere else. `aracreate-brand.svg` is the palette reference sheet, not a logo; do not place it.

Placing one: clear space of one monogram-width on every side; never stretch, recolour, shadow, outline, gradient or rotate it; never place the default logo over imagery. The seal and the letterhead are stationery, not web furniture — a stamp on a web page reads as decoration and dilutes it.

**The Monument Extended licence.** It is a commercial licence, and declaring it in an imported stylesheet publishes the `.otf` to every visitor of every page that links that stylesheet. It is also almost never needed: the logo SVGs have the wordmark outlined to vector paths, so they are crisp at any size, recolourable, and need no font at all. Prefer the SVG; reach for live wordmark text only where a template genuinely cannot place an image. To make the font opt-in, move the two `@font-face` rules out of `tokens/fonts.css` into a file that no entry point imports, and link that file only where live wordmark text is needed. `--ac-font-logo` falls back to Arial Black and nothing else breaks. See [`docs/licence-policy.md`](docs/licence-policy.md).

---

## What the merge brought in

ACDS is the base and won on structure, file naming and token values. Folded in from the 20 August system on 21 August 2026:

- **78 components in eight groups** — the component index below.
- **Ten stylesheets** — `base`, `signature`, `components`, `sections`, `app` and `deck` in `styles/`, plus `density` and `theme-dark` in `tokens/`, plus the `tokens.css` the port arrived with, which was split four ways into `fonts` / `colors` / `typography` / `spacing` to follow ACDS's four-files-by-kind token layer.
- **The dark theme** — `tokens/theme-dark.css`.
- **The density scale and the 44px target floor** — `tokens/density.css`.
- **Thirty specimen cards** in `foundations/`, alongside ACDS's own six.
- **Four UI kits** — website, academy, web_app, deck.
- **A browser test gate** — `tests/checks.html`.
- **Eleven documents** in `docs/`.

Everything the merge moved in value terms is in [`changelog.md`](changelog.md), dated 21 August 2026.

---

## Index

**Root**

- `system.css` — **the whole system.** Link this. Twelve `@import` lines, nothing else.
- `styles.css` — the older entry point: four token imports and nothing else.
- `readme.md` — this file. `changelog.md` — what changed and why. `SKILL.md` — Agent Skills front matter. `CLAUDE.md` — the standing rules. `github.md` — sync history with the source repository.
- `thumbnail.html` — the project tile.
- `_ds_bundle.js`, `_ds_manifest.json`, `_adherence.oxlintrc.json` — generated by the app, not hand-edited; they were stale at the time of writing.

**`tokens/`** — the token layer, four files by kind plus two scoped layers.

- `fonts.css` — webfonts and the `@font-face` declarations.
- `colors.css` — every colour, primitives then semantics. Nothing anywhere else writes a raw colour.
- `typography.css` — families, weights, the display tier, the heading ladder, sizes, leading, tracking, and the known-deviations note.
- `spacing.css` — the space ladder, radii, shadows, durations and easing, border widths, z-index layers, container widths.
- `density.css` — the 44px target floor and the compact density scope.
- `theme-dark.css` — the dark theme.

**`styles/`** — `base.css` (element defaults, focus ring, the surface contexts) · `signature.css` (the araCreate devices — edge, dash, eyebrow, underline, bleed, marquee, reveal, sticky bar, progress, scroller, sticky side, dot) · `components.css` · `sections.css` · `app.css` (the signed-in surfaces) · `deck.css`.

**`js/`** — `signature.js` (viewport measurement, reveals, sticky, progress, scrollers) · `components.js` (tabs, dropdowns, modals, mobile nav, table sorting, toasts) · `deck.js` (optional arrow-key deck navigation). Vanilla, no dependencies.

**`components/`** — React primitives grouped by concern. Each has `<Name>.jsx`, `<Name>.d.ts` and `<Name>.prompt.md`, and each directory carries at least one `@dsCard` HTML specimen. Nine directories: the eight groups below plus `content/`, the ACDS compatibility layer.

**`assets/`** — the brand asset library, six folders: `brand/` (letterhead, stamp) · `fonts/` (the two Monument Extended `.otf` files) · `icons/` (17 SVG) · `illustrations/` (8 SVG) · `imagery/` (5 duotint `.jpg`) · `logos/` (21 SVG, 11 PNG and three `_source-*.svg` Affinity sheets).

**`foundations/`** — thirty-six specimen cards that populate the Design System tab: eight brand cards (icons, logos, sublockups, monogram, stamp, illustrations, imagery, signature edge), twelve colour cards (brand, core, golden, the ramp, neutrals, text, surfaces, feedback, the gold rule, the photo wash, dark theme, dark-theme components), eight spacing and shape cards (scale, grid, in-use, radii, shadows, motion, density, targets), and eight type cards (display, headings, body, labels, mono, weights, figures, wordmark).

**`ui_kits/`** — `website/` (the group marketing site: Home, About, Services, Projects, Contact, NotFound, TrustedByStrip) · `academy/` (Courses, Tutors, Apply) · `web_app/` (Dashboard, Jobs, Settings, SignIn) · `deck/` (ten slide layouts plus a navigable `index.html`). Each has its own `README.md` and an `index.html` you can click through.

**`templates/`** — starting folders a consuming project copies. `marketing-page/` (a full araCreate section page) and `deck/` (six of the ten slide layouts in one 16:9 deck). Each has a `ds-base.js` with one line to point at the bound design system, a `support.js`, and a `README.md` listing what to change.

**`tests/`** — `checks.html`. A browser-runnable gate covering contrast (compositing translucent backgrounds before measuring) and accessibility (accessible names, alt text, heading order, ARIA references, duplicate ids, `th` scope, table captions, positive tabindex, undeclared icons), plus two brand rules (no emoji, no vague link text). It **globs rather than lists** — add a path to `PAGES` and it is covered from then on. Open it and it runs; no install.

**`docs/`** — eleven documents.

- `live-site.md` — **the production Webflow snapshot**: verified site identity, page skeleton, component inventory with instance counts, CMS surfaces, and the DTF namespace. Read this before treating anything about the live site as current.
- `components.md` — **the component reference.** All 78: what each is for, what each is *not* for, its states, its keyboard behaviour, and its maturity. Start here when choosing a component.
- `brand-facts.md` — **the source of truth for every fact copy may state.**
- `guidance.md` — when to use each CSS class and, more usefully, when not to.
- `decisions.md` — the decision record. What changed, when, why, and the measurement that forced it. Read this before arguing with a value.
- `accessibility.md` — the contract, measured contrast figures in both themes, the keyboard table, and an honest list of what is not verified.
- `assets.md` — which logo, icon and image file, on which background.
- `licence-policy.md` — what is copied and what is rebuilt, and the line the system works to.
- `contributing.md` — how to add a component without pulling the system apart.
- `api-audit.md` — a pass over the prop surface of all 78, ranked by what each finding would cost a consumer.
- `back-port.md` — the reconciliation list: what lives only here and should go back to the source repository.
- `screen-reader-pass.md` — a forty-minute script for the one check nothing automated can do.

### Component index — 78 components, eight groups

Counts and maturities are from [`docs/components.md`](docs/components.md), which is the reference.

| Group | Count | Maturity | What it is for |
| --- | --- | --- | --- |
| `core/` | 15 | Stable | Button, Icon, Badge, Tag, TagGroup, Avatar, AvatarGroup, Card, CardMedia, CardBody, CardFooter, CardGrid, Stat, StatRow, Panel, SectionLabel |
| `forms/` | 14 | Field/Input/Select/Choice/Switch Stable, rest New | Field, Input, Search, Textarea, Select, Choice, ChoiceGroup, Switch, SegmentedControl, NumberInput, Slider, Combobox, DatePicker, FileUpload, Checkbox, Radio |
| `feedback/` | 6 | Stable | Alert, Toast, ToastRegion, Modal, Tooltip, ProgressBar |
| `navigation/` | 6 | Stable | Tabs, Accordion, AccordionItem, Breadcrumb, Pagination, Dropdown |
| `data/` | 6 | Stable | Table, Steps, PriceCard, Quote, LogoTile, LogoStrip |
| `signature/` | 5 | Stable | Eyebrow, Dash, Underline, Marquee, Chevrons |
| `sections/` | 8 | Stable, marketing website only | Header, Hero, Section, ServiceList, CtaBand, Footer, PostList, NotFound |
| `app/` | 18 | New, signed-in surfaces only | AppShell, AppBrand, NavList, NavFooter, Toolbar, FilterBar, BulkBar, DataTable, CellStack, EmptyState, Skeleton, SkeletonStack, SkeletonTable, Drawer, Banner, KpiTile, ChartShell, Bars |

**`content/` is not a ninth group.** It holds the ACDS compatibility layer — `ServiceCard` and `StatBlock`, old names that still import, each a thin wrapper over the merged component named in its `.prompt.md`. Write the merged name in new code.

**Which set to reach for.** `core`, `forms`, `feedback`, `navigation`, `data` and `signature` serve every surface. `sections` is the marketing website — full page slabs. `app` is the signed-in product: the shell, the selectable table, and the states a product has that a marketing page does not (empty, loading, bulk-selected). A dashboard uses `app` + `core` + `forms`; a landing page uses `sections` + `core`.

**Intentional additions.** `Icon` and `Panel` are wrappers over classes the source styles but never marks up — `.ac-icon` is a contract with no component, and `.ac-panel` is the primary carrier of the signature edge with no markup of its own. The whole `app/` group and six `forms/` additions are **new design in the araCreate idiom**, not recreations: the stylesheet they extend was reverse-engineered from a marketing site and had no shell, no selectable table, no empty state, no skeleton, no date picker. They reuse rather than duplicate wherever something already fitted — a KPI tile *is* a stat card, an inline message *is* an alert, a combobox *is* an input plus a menu, filter tokens *are* tags. Building a second card that looks 95% like the first is how a system ends up with two of everything and nobody knowing which is current.

---

## Rules that outrank convenience

1. **Never write a raw value.** No hex colours, pixel sizes or timings in a component. A missing token is a signal: use the nearest one, or add one deliberately.
2. **Name the job, not the colour.** `--ac-accent`, never `--ac-yellow`.
3. **Gold is never text.**
4. **Nothing invents a fact.**
5. **Do not undo three things the system handles**: the global `:focus-visible` ring, reduced-motion handling, and the inverse-surface context.
6. **Screenshot every new component and look at it.** A whole build of the source system once rendered card body copy bold — a card that was a link inherited the link's font weight — and every automated gate passed. It was found by reading a screenshot. The same class of bug has recurred twice since; see the ledger in [`docs/accessibility.md`](docs/accessibility.md).
7. **Link the right stylesheet.** `system.css` for the system, `styles.css` for tokens alone. Do not widen `styles.css`.
8. **Do not edit the locked files.** See below.

How to add to the system: [`docs/contributing.md`](docs/contributing.md).

### Locked files

These are pinned by decision and were not touched by the merge. Changing one is a conversation, not a commit.

- `ui_kits/deck/` — the deck kit and its `README.md`.
- `templates/deck/` — `Deck.dc.html`, `ds-base.js`, `support.js` and its `README.md`.
- `styles.css` — the four token imports and nothing else.
- `foundations/brand-icons.card.html` — the iconography card.

All four are present in this tree.

---

## What this project does not have

Stated plainly so nobody assumes otherwise.

- **The source repository runs seven gates** — conventions, adherence, contrast, accessibility, behaviour, deck, and a proof that a real page needs no custom CSS. **None of them run here.** `tests/checks.html` is a browser-runnable port of two of them.
- **No screen-reader pass** has been run. [`docs/screen-reader-pass.md`](docs/screen-reader-pass.md) is the script for one, not a record of one.
- **No release process.**
- **Nothing has shipped in production** on the eighteen `app/` components or the six `forms/` additions that came with them.
- **No Figma file.** The design lives in CSS.
- **No araCreate Meditate kit.** It is named in the sources as an intended consumer, but no screens, copy or layouts were provided, so none was built.
- **One known contrast failure**, `--ac-text-muted` at 3.19:1, kept on instruction and documented above.

The highest-value thing anyone could add is the contrast and accessibility gates from the source repository, globbing rather than listing, so a new card or kit is covered the moment it exists.

---

## Open threads

**The live-site record is a snapshot**, not a live check. It was verified on 17 August 2026 and the site is under active CMS management. See [`docs/live-site.md`](docs/live-site.md).

---

Licence: the ported CSS, JavaScript and assets are proprietary, © 2026 araCreate Group.
