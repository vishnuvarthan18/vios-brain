# araCreate Design System (ACDS)

**Version v2.0.1** — 1 September 2026. History and the version index are in [`changelog.md`](changelog.md); how the project came to be is in [`docs/history.md`](docs/history.md).

The design system for **araCreate Group**, built from the Brand Guidelines v1.0 (2026), the brand asset library, the araCreate deck of 4 February 2026 and the production Webflow site. Nothing here is invented; every colour, font, radius and motif traces to one of those sources.

**This project is the master.** The Webflow site is today's only live consumer and will be rebuilt on this system; the GitHub repository is a backup and audit trail, exported to, never imported from. So there are no compatibility aliases, no deprecated names kept "for consumers", and no second way to say anything: when a name changes here, it changes everywhere.

---

## Which stylesheet to link

- **`system.css`** — the whole system: tokens, element defaults, signature devices, components, sections, application surfaces, density, dark theme, deck. A page built from `.ac-*` classes links this.
- **`tokens.css`** — the variables alone. A page that styles itself inline (the deck template, the project tile, the foundation cards) links this.

A page that links `tokens.css` and renders unstyled components linked the wrong file.

---

## Company context

**araCreate** — founded in **2003 by Aravinth Panch** — has evolved from a solo digital studio into a **group of companies in Europe and Asia**, delivering **360° services** that empower *"ideas from mind to market"* across **three service verticals**:

- **Engineering** — *from Prototype to Production*
- **Manufacturing** — *from Drawing to Delivery*
- **Media** — *from Sketch to Screen*

**By the numbers:** 300+ clients · 60+ new products · 8+ group companies · 3+ service verticals · 50+ service capabilities · 3+ country offices · 50+ team members · 20+ years of service.

**Ecosystem (8+ group companies):** araCreate GmbH, BatchOne GmbH, rlgtechdesign GmbH, DreamSpace Foundation gUG (Germany); araCreate India Pvt Ltd, Micro-Tech CNC Pvt Ltd (India); araCreate Lanka Pvt Ltd, DreamSpace Foundation CLG (Sri Lanka). **HQ:** araCreate GmbH, Hubertusstr. 5, 12163 Berlin · ask@aracreate.group · +49 30 91437357.

Positioning: **innovation and reliability** — *"where boundless passion meets performing results."* The look is **bold, industrial and human**. **DTF — Deep Tech Foundry** is a venture on the same system under a parallel `-dtf` namespace; a sub-brand, not a separate brand.

Every fact copy may state is fixed in [`docs/brand-facts.md`](docs/brand-facts.md). If the two ever disagree, that file wins.

---

## Content fundamentals

**Voice:** professional, clear, friendly — confident, industrial, human. "We" for araCreate, "you" for the client. Short sentences, one idea each. Concrete over abstract.

**Casing.** Sentence case for headlines and body. Wide-tracked UPPERCASE is reserved for buttons, eyebrows and small labels, which is why a heading in caps looks wrong here. Section eyebrows are numbered — `01 — araCreate Group`. Section headlines are the one aspirational place and trail off with an ellipsis: *"Where boundless passion meets performing results …"*

**Microcopy.** Buttons say what happens, in the visitor's terms. Errors say what to do. Empty states say what to do next. "Let's talk about your idea" is the brand's own CTA.

**Never:** emoji, anywhere; exclamation marks in body copy; invented facts (`300+ clients` is the fact — the plus sign makes it true); "click here"; a whole sentence in caps; Monument Extended outside the logo wordmark; a ninth group company.

**Fixed strings.** *Empowering ideas from mind to market* · *where boundless passion meets performing results* · the three vertical couplets · the four values: *We pursue excellence · We collaborate seamlessly · We deliver results · We stay curious*.

**The name.** araCreate — lowercase a-r-a, capital C. ARACREATE in full caps is correct only as the logo wordmark. Legal suffixes are part of a company's name. `rlgtechdesign` is one lowercase word.

---

## Visual foundations

**Naming.** American spelling in identifiers (`gray`, `color`), British in prose. Tokens name the job, not the colour: `--ac-accent`, never `--ac-yellow`. Components use semantic tokens only.

**Colour.** Two brand colours: **Golden Sun `#f9bf3b`** (the accent — backgrounds and rules, never text) and **Graphite Gray `#555555`** (every level of text, every strong border, the focus ring, the dark band). **There is no black in this system**; graphite is its darkest colour. Around them: hairline stroke `#cecece`, off-white canvas `#f6f6f6` (the page), white `#ffffff` (the raised surface), Golden Soft `#fdf3d8` (the alternating band and every gold fill carrying small text), and one derived accessible grey, `#6f6f6f`, for muted text. Slides use the same canvas through `--ac-surface-slide`. Feedback: danger `#c0492f`, success `#186a43`; warning reuses gold, information is graphite. `#2e419e` is the duotint photo wash, not a colour. *Do not introduce hues outside this palette.*

**Gold is never text.** Gold on white and white on gold both measure 1.67:1 and fail at every size. Gold goes *behind* type. Graphite on Golden Sun measures 4.45:1 — 0.05 under the bar for small text — so full Golden Sun carries display text (19px/500 and larger) or no text; anything smaller sits on Golden Soft, where graphite measures 6.63:1. The primary button's 14px label is the one accepted shortfall, recorded in [`docs/accessibility.md`](docs/accessibility.md).

**Surfaces, not variants.** Page, card, subtle, dark, accent and accent-soft each re-point every colour token inside them. Put the surface class on the *band*; no component has a dark variant.

**Dark theme.** `data-ac-theme="dark"` on any element; `"auto"` follows the OS. One file, `tokens/theme-dark.css`. Golden Sun does not move; the inverse surface flips to light; error and success take dark-specific values.

**Density and targets.** Comfortable is the default; `data-ac-density="compact"` tightens signed-in screens under `pointer: fine`. Every interactive control is at least 44px in both directions, as padding, never by growing the type. Links in running text are exempt.

**Type.** Three families, strictly by role: **Monument Extended** for the logo wordmark only (Ultrabold 800, `.ac-wordmark` is the only selector allowed to use it; prefer the outlined SVG); **Poppins** for everything else; **Red Hat Mono** for specs, data and code.

**The Poppins weight scale** — four steps, one job each: **200** body paragraphs · **300** subheadings, captions, labels · **500** emphasis and numbers · **700** section headings. **400** is display type only (the 88–105px slide titles). 600 does not exist in this system.

**The type scale.** Headings 36 / 33 / 31 / 28 / 21 / 16px, locked to the live site (a recorded deviation: the steps are close). Display tier above it: 64 / 48 / 36 / 28px. Body 16px, leading 1.8, tracking .02em. Small and link text 14px. Statistics 42px. The deck has its own ladder in `styles/deck.css`, in `cqw`.

**Spacing.** `--ac-space-1` … `--ac-space-16`: 4, 8, 10, 12, 15, 16, 20, 24, 30, 40, 60, 80, 90, 100, 120, 140 — measured from the live site; the index is an order, not a measurement. Sections 90px; tall bands 140px. Containers 940px (`--ac-container`) and 1200px (`--ac-container-wide`); body copy caps at 74ch.

**Corners.** xs 2 · sm 4 (inputs) · md 9 (panels, image frames) · lg 20 (cards) · pill 200 · none 0 (buttons). Anything carrying the signature edge stays square. Never write a `border-radius` in a component.

**Borders and the signature edge.** Hairline `1px #cecece`. The signature device is a 1px edge, **dotted top and right, solid bottom and left**, square corners, applied by default to panels, cards, modals, inputs, tables, blockquotes and `figure img`; `.ac-edge-none` opts out. Buttons are excluded on purpose. Dividers may be faint (`--ac-border`); control edges must reach 3:1 and use graphite (`--ac-border-control`, `--ac-border-strong`).

**Shadows.** Four — `sm` resting, `lift` hover, `md` dropdown or toast, `lg` modal. A component picks one, never composes its own, never a coloured one.

**Motion.** `.25s` / `.5s` / `.8s`, one easing `cubic-bezier(.165, .84, .44, 1)`. Named devices: the 2px underline that wipes in from the left; reveals that rise 20px, staggered 80ms; the self-drawing line; the 40s marquee; the sticky header that shrinks 15px; the 3px gold progress bar. Everything stops under `prefers-reduced-motion` and nothing is hidden from someone who cannot receive it.

**Hover and press.** Buttons lift `translateY(-5px) scale(1.02)` on hover, `-2px / 1.01` on press; gold goes to 90% then 80%, darker never lighter. Ghost buttons neither lift nor shadow. Cards lift and close their dotted edge. The focus ring is one global style — 2px graphite, 2px offset — and removing it is one of three things you may not undo.

**Transparency.** No glassmorphism, no blur, no protection gradients. Transparency appears only in the modal scrim (graphite at 80%), the gold hover and pressed states, the ~8% feedback surfaces and the marquee's end fade.

**Layout.** Canvas is the default; bands alternate canvas → Golden Soft → canvas, with occasional graphite or Golden Sun — **at most one or two background colours per page**. Sticky header 90px, falling to 80 at 1280 and 75 at 1440. Edge-to-edge bands use `--ac-vw`, not `100vw`. Breakpoints: 479 / 767 / 991 down, 1280 / 1440 / 1920 up.

**Imagery.** Photography is **duotint** over `#2e419e`. A raw full-colour photograph is not an araCreate image. `figure img` carries the signature edge; a logo or icon `<img>` does not.

**Slides.** 16:9, `container-type: size`, every size in `cqw` (1280px reference, 1cqw = 12.8px). Two surfaces — slide canvas and gold; no dark slides. Chrome is identical on every surface: 67px gutters, 90px footer, wordmark bottom-left, three chevrons bottom-right (mirrored on the closing slide only), page marker top-right. Emphasis in a title is weight, not colour. The rules are in [`docs/deck-conventions.md`](docs/deck-conventions.md); the reference build is `templates/deck/`.

### Accessibility exceptions and shortfalls

Two ACDS values are deliberately not used, each a repair: `--ac-focus` is graphite, not gold (gold measures 1.55:1 on canvas); `--ac-success` is `#186a43`, not `#4f9d69` (3.06:1). Two shortfalls are accepted and recorded, not hidden: graphite on Golden Sun at 4.45:1 for the primary button's 14px label, and the slide title's canvas-on-gold highlight phrase, which is decoration on a 105px line. Measured figures for both themes, the keyboard table and what is not yet verified are in [`docs/accessibility.md`](docs/accessibility.md).

---

## Iconography

The brand's icon language is **isometric line illustration** — thin, single-weight strokes in three-quarter perspective, all SVG. Service icons at 48–96px; UI chrome as simple stroked shapes; illustrations for heroes and section art. **No icon font, no icon library, nothing from a CDN.** Small glyphs are an inline `<symbol>` sprite referenced with `<use href="#ac-i-…">` under the `.ac-icon` contract (1.15em, `stroke: currentColor`, 1.5 stroke, round caps); the `Icon` component wraps it. Icons inherit `currentColor` — set the container's text colour, never the icon's. **Every icon declares itself** decoration (`aria-hidden="true"`) or content (`role="img"` with a label); the system outlines any that declares neither. When the icon you need does not exist, draw it in the isometric style or flag a substitution — never an emoji, never an icon font.

---

## Visual assets

`assets/` carries the library: `brand/` (letterhead, stamp) · `fonts/` (two Monument Extended `.otf`) · `icons/` (17 SVG) · `illustrations/` (8 SVG) · `imagery/` (5 duotint `.jpg`) · `logos/` (21 SVG, 11 PNG, three `_source-*.svg` Affinity sheets).

**Logos — read the chip, not the name.** Canvas, white or gold → `aracreate-logo-default.svg`. Graphite → `-t-w-b-g.svg` (or `-t-w-b-y.svg`). Over imagery → `aracreate-logo-negative.svg` (a knockout, not a dark-background variant). Favicon and avatar → `aracreate-icon-*`. Wordmark alone → `aracreate-wordmark-*`. Division lockups (`-engineering-`, `-manufacturing-`, `-media-`) on a division site only. `aracreate-brand.svg` is the palette sheet, not a logo. Clear space of one monogram-width; never stretch, recolour, shadow, outline, gradient or rotate. Full table in [`docs/assets.md`](docs/assets.md).

**The Monument Extended licence** is commercial; declaring it in `tokens/fonts.css` serves the `.otf` to every visitor. The logo SVGs have the wordmark outlined, so the font is almost never needed. See [`docs/licence-policy.md`](docs/licence-policy.md).

---

## Index

**Root** — `system.css` (the whole system) · `tokens.css` (variables only) · `readme.md` (this file) · `changelog.md` · `SKILL.md` (agent front matter) · `CLAUDE.md` (standing rules) · `github.md` (repository record) · `thumbnail.html` (project tile). `_ds_bundle.js`, `_ds_manifest.json`, `_adherence.oxlintrc.json` are generated; do not edit them.

**`tokens/`** — `fonts.css` · `colors.css` (every colour, primitives then semantics; nothing else writes a raw colour) · `typography.css` · `spacing.css` (space, radii, shadows, motion, borders, z-index, containers) · `density.css` · `theme-dark.css`.

**`styles/`** — `base.css` (element defaults, focus ring, surfaces) · `signature.css` (the araCreate devices) · `components.css` · `sections.css` · `app.css` · `deck.css` (slides, and the `--ac-slide-*` knobs).

**`js/`** — `signature.js` · `components.js` · `deck.js`. Vanilla, no dependencies.

**`components/`** — React primitives in eight groups; each component is `<Name>.jsx` + `<Name>.d.ts` + `<Name>.prompt.md`, and each directory carries a `@dsCard` specimen.

| Group | Count | For |
| --- | --- | --- |
| `core/` | 15 | Button, Icon, Badge, Tag, TagGroup, Avatar, AvatarGroup, Card, CardMedia, CardBody, CardFooter, CardGrid, Stat, StatRow, Panel, SectionLabel |
| `forms/` | 14 | Field, Input, Search, Textarea, Select, Choice, ChoiceGroup, Switch, SegmentedControl, NumberInput, Slider, Combobox, DatePicker, FileUpload, Checkbox, Radio |
| `feedback/` | 6 | Alert, Toast, ToastRegion, Modal, Tooltip, ProgressBar |
| `navigation/` | 6 | Tabs, Accordion, AccordionItem, Breadcrumb, Pagination, Dropdown |
| `data/` | 6 | Table, Steps, PriceCard, Quote, LogoTile, LogoStrip |
| `signature/` | 5 | Eyebrow, Dash, Underline, Marquee, Chevrons |
| `sections/` | 8 | Header, Hero, Section, ServiceList, CtaBand, Footer, PostList, NotFound — marketing site only |
| `app/` | 18 | AppShell, AppBrand, NavList, NavFooter, Toolbar, FilterBar, BulkBar, DataTable, CellStack, EmptyState, Skeleton, SkeletonStack, SkeletonTable, Drawer, Banner, KpiTile, ChartShell, Bars — signed-in product only |

A dashboard uses `app` + `core` + `forms`; a landing page uses `sections` + `core`. A service tile is `Card kind="service"`; a statistic is `Stat`. [`docs/components.md`](docs/components.md) is the reference: what each is for, what it is not for, states, keyboard behaviour, maturity.

**`foundations/`** — 35 specimen cards for the Design System tab: 8 brand, 11 colour, 8 spacing and shape, 8 type.

**`templates/`** — starting folders a consuming project copies. `web/` (`acds-template-web`: home, about, 404) · `app/` (`acds-template-app`: sign-in, shell, dashboard, jobs, settings) · `deck/` (`acds-template-deck`: ten 1280×720 slides). Each has a `ds-base.js` with one line to point at the bound design system and a `README.md`.

**`tests/`** — `checks.html`. Contrast (compositing translucent backgrounds), accessibility (names, alt, heading order, ARIA references, duplicate ids, `th` scope, captions, tabindex, undeclared icons) and two brand rules (no emoji, no vague links). It reads the card list from `_ds_manifest.json`, so a new card is covered the moment it exists. Open it and it runs.

**`docs/`** — `export/` (export procedure and gate, `state.json`, the two repo READMEs) · `live-site.md` (the Webflow snapshot, verified 17 Aug 2026) · `components.md` · `brand-facts.md` · `guidance.md` (when to use each class, and when not) · `decisions.md` (the decision record, ported from the repository) · `deck-conventions.md` · `accessibility.md` · `assets.md` · `licence-policy.md` · `contributing.md` · `api-audit.md` · `back-port.md` (retired) · `screen-reader-pass.md` · `history.md`.

---

## Rules that outrank convenience

1. **Never write a raw value.** No hex, pixel sizes or timings in a component. In an inline-styled template, every value is `var(--token, fallback)`.
2. **Name the job, not the colour.**
3. **Gold is never text.**
4. **Nothing invents a fact.**
5. **Do not undo three things the system handles**: the focus ring, reduced motion, the surface contexts.
6. **Screenshot every new component and look at it.** Every gate has passed on a broken build at least once; a screenshot caught it.
7. **Link the right stylesheet.** `system.css` for a page, `tokens.css` for variables alone.
8. **No aliases.** When a name changes, change every reference and record it in the changelog. A master carries one name per thing.

How to add to the system: [`docs/contributing.md`](docs/contributing.md).

---

## Not yet

- The repository's seven gates do not run here; `tests/checks.html` ports two.
- No screen-reader pass has been run; [`docs/screen-reader-pass.md`](docs/screen-reader-pass.md) is the script.
- Nothing has shipped in production on `app/` or the six newer `forms/` components.
- No Figma file. The design lives in CSS.
- Remote state is UNVERIFIED from here; `docs/export/EXPORT.md` is the export procedure and gate, `docs/export/state.json` the handshake.

Licence: proprietary, © 2026 araCreate Group.
