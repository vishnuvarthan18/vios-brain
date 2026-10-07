# DreamSpace Academy Design System

## About
DreamSpace Academy (DSA) is a non-profit social enterprise based in Batticaloa, Sri Lanka. It empowers underserved communities through challenge-based learning, grassroots innovation, and impact-venture building. Its work spans learning "labs," grassroots innovations, venture incubation, and community achievements — tracked through impact statistics. The website is content-rich, organized around these programme areas.

## Sources provided
- Brand notes (colors, type, spacing) — pasted brief, no external link.
- Logo files: `uploads/dsa-colour.svg`, `uploads/dsa-white.svg`, `uploads/dsa-white-icon.svg`, `uploads/dsa-white-vertical.svg`, `uploads/dsa-white-vertical-1.svg`.
- No Figma file, codebase, or slide deck was attached. This system is built from the brand brief plus the logo files — a from-scratch component set, sized to the brand's needs, not copied from an existing product.

## Index
- `styles.css` — root stylesheet, imports all tokens.
- `tokens/` — colors, typography, spacing, radius, shadows.
- `assets/logo/` — logo lockups (see Iconography section): `dsa-colour.svg` (full colour, light backgrounds), `dsa-white.svg` (white wordmark, dark backgrounds), `dsa-white-icon.svg` (mark only), `dsa-white-vertical.svg` / `dsa-white-vertical-1.svg` (stacked lockups for square formats, white and multicolour).
- `components/core/` — Button, Badge, Tag, Card, SectionHeading, StatCard
- `components/forms/` — Input, Select, Checkbox, Radio, Switch
- `components/feedback/` — Toast, Tooltip
- `components/navigation/` — Tabs
- `ui_kits/website/` — DreamSpace Academy marketing site recreation: Header, Footer, Home, Labs, Ventures, Impact screens (`index.html` to view, click-through navigation)
- `ui_kits/homepage/` — closer recreation of the live homepage: HomepageHeader, Hero, Programs, About, CtaFooter
- `design-system-book.html` / `design-system-book-print.html` — printable brand/design-system book
- `The Honeycomb.html`, `guidelines/honeycomb-device*.html` — the honeycomb brand device
- `brand-book-fulltext.txt`, `uploads/DreamSpace-Brand-Book-v2.pdf` — source brand book text and PDF
- `guidelines/` — foundation specimen cards (colors, type, spacing, radius/elevation, brand logos)
- `SKILL.md` — Claude Code-compatible skill file
- `thumbnail.html` — project thumbnail

## Content fundamentals
- **Tone**: purpose-driven, grounded, hopeful. Community-first, never corporate-speak. Avoid buzzwords ("synergy", "disruption", "leverage") — write plainly, like explaining a programme to a neighbour.
- **Voice**: DreamSpace speaks about the communities it works with, not down to them. Prefer "young people in Batticaloa built…" over "we empower youth to…" — foreground the people doing the work, DSA as the enabler.
- **Casing**: sentence case for headings and buttons ("Apply for a lab", not "Apply For A Lab"). Programme and venture names are proper nouns and stay capitalized.
- **Numbers**: impact stats are stated plainly and specifically ("1,200+ youth trained", "38 ventures launched") — no vague superlatives ("countless", "many").
- **Emoji**: not used. The brand relies on colour and iconography, not emoji, for warmth.
- **Sentence style**: short, direct sentences. Avoid stacking adjectives; one concrete detail beats three abstract ones ("a solar dryer built from scrap metal" over "an innovative and impactful solution").
- **Example (site-appropriate copy)**: "In 2023, twelve young innovators from Batticaloa built a low-cost water filter now used in three villages."

## Visual foundations
- **Colour**: warm cream base (#FDF9F6), with white (#FFFFFF) and light grey (#FAFAFA) as secondary card surfaces. Purple 700 (#6A0BB2) carries brand-coloured elements; orange 500 (#E45B00) marks a single CTA per block. Never both at full saturation together. Full ramps (purple, orange/Poppy Flower, neutral/black, 50–950) are in `tokens/colors.css`, taken from the live Webflow build. Secondary accents — blue #253DCC, coral #FE753F, coral-2 #FFC971, misty rose #FFDEDE, notification dark #001B38 — are used sparingly and are not core brand.
- **Type**: Poppins only, display and body — no secondary typeface. (The live site also loads Bricolage Grotesque and Roboto; the brand standard is Poppins throughout and those are ignored here.) Scale: H1 56/600, H2 44/600, H3 36/600, H4 32/700, H5 26/700, H6 16/600, Body 18/400 at 1.3 line-height with 0.7px letter-spacing, Caption 14/400.
- **Text & links**: body text #010101; links #000000, shifting to purple 700 on hover.
- **Backgrounds**: flat cream, purple, or orange fills — no gradients, no textures, no patterns. The one gradient in the source material lives inside the icon mark itself (orange, top-to-bottom) and should not be reused elsewhere.
- **Imagery**: none supplied. When real photography is added, it should be warm and documentary (community/workshop settings), not stock-corporate. Until then, use plain colour blocks or the logo mark as placeholders — never invented illustration.
- **Animation**: not specified by the brand; default to short, functional transitions (150ms ease) on hover/focus only — no bounce, no decorative motion.
- **Hover/press states**: hover darkens filled buttons one step and tints ghost/secondary buttons with a light neutral fill; press states are handled the same way (no scale/shrink effects).
- **Borders & shadows**: a three-level shadow system (sm/md/lg) for cards, dropdowns, and toasts; 1px neutral borders — #F1F1F1 (`--border-hairline`), #F5F5F6 (`--border-subtle`), #D4D4D4 (`--border-default`) — rather than colour borders. No left-border accent bars.
- **Radius**: sm (4px) for checkboxes/inputs, md (8px) for buttons/tags, lg (16px) for cards, pill for buttons/badges.
- **Semantics**: a standard status set sits alongside the brand ramps and is used only for status/feedback — success #1E9E5A, error #D92D20, warning #F59E0B, info #253DCC. Core brand purple and orange are never used to signal status.
- **Layout**: content-first, section-based marketing site — alternating white/tinted sections rather than a fixed app-shell layout.
- **Transparency/blur**: not used; the brand is opaque and flat.

## Iconography
No icon system, icon font, or icon SVG set was supplied in the brand materials. Two CDN sets are used as substitutes: [Lucide](https://lucide.dev) for UI/category glyphs (stroke-based, 1.5–2px weight) and [Simple Icons](https://simpleicons.org) for social brand marks — Lucide dropped brand glyphs from its set, so those four filenames 404 there. **Both are substitutions, not brand assets** — swap in the real icon set if DreamSpace has one. No emoji or unicode-glyph icons are used.

## Intentional additions
No source defined a component inventory, so a standard set was authored, sized to a content-heavy nonprofit site: Button, IconButton (folded into Button variants), Badge, Tag, Card, Input, Select, Checkbox, Radio, Switch, Tabs, Toast, Tooltip. Stat/impact-number display and a Section Header pattern were added as `components/core/StatCard.jsx` and `components/core/SectionHeading.jsx` because the brief explicitly calls out "impact stats" and content sections as core to the site.
