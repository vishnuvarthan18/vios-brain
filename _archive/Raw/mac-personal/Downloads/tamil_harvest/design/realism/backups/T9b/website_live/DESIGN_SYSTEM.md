# Semmozhi design system — writing surfaces

The site is themed on the materials Tamil was written on. Each section takes the colours of one material. This file says what each token is for and when to use it. `styleguide.html` shows every token and component in every state.

## Files and load order

```html
<link rel="stylesheet" href="css/tokens.css">     <!-- colours, fonts, spacing, radii, shadows, motion, base rules -->
<link rel="stylesheet" href="css/textures.css">   <!-- texture layers (gradients + inline SVG noise) -->
<link rel="stylesheet" href="css/components.css"> <!-- components -->
<link rel="stylesheet" href="css/materials.css">  <!-- real material surfaces (Prompt 2); optional -->
<script>document.documentElement.classList.add('js')</script>   <!-- in <head>: avoids a menu flash -->
<!-- optional GSAP motion (Prompt 2), end of <body>, in this order: -->
<script src="js/vendor/gsap/gsap.min.js"></script>              <!-- + the plugins the page uses, see "Motion (GSAP)" -->
<script src="js/motion.js"></script>
<script src="js/components.js"></script>                         <!-- end of <body>: leaf flip, bundle untie, menu; hands motion to Motion -->
<!-- optional measured 3D materials (Prompt 4 T9), after components.js, in this order: -->
<script src="js/realism/stage.js"></script>                      <!-- GENERATED from the measured rig -->
<script src="js/realism/materials.params.js"></script>           <!-- GENERATED from the T6 material cards -->
<script src="js/realism/materials3d.js"></script>                <!-- attaches a stage to every [data-real-material] -->
```

| File | Size | Notes |
|---|---|---|
| css/tokens.css | 10.9 KB | The only file with raw colour values |
| css/textures.css | 5.7 KB | Every texture is under 1.1 KB |
| css/components.css | 26.5 KB | Colours only via `var(--…)` |
| js/components.js | 4.4 KB | No dependencies; reads the motion tokens |
| css/materials.css | 25.5 KB | Real materials (Prompt 2, Part A). Includes four inlined SVG masks |
| img/tex/* | 41.5 KB total | 17 procedural SVG tiles (each under 2 KB) and 3 photo detail tiles (WebP, largest 13 KB) |
| js/realism/stage.js | 19.0 KB | Generated (`design/realism/tools/build_site_stage.mjs`); do not edit by hand |
| js/realism/materials.params.js | 6.8 KB | Generated (`design/realism/tools/build_site_materials.mjs`); measured material numbers |
| js/realism/materials3d.js | 6.3 KB | Site glue for the measured materials; no dependencies |
| fonts/web/*.woff2 | 333 KB | Noto Serif Tamil (Tamil subset) and Fraunces (roman + italic, Latin + Latin Extended). SIL OFL 1.1; licences sit next to the files |

The CSS added comes to 46.6 KB, including the style guide's own 3.6 KB `<style>` block. No external requests: everything loads from the site folder.

## Ground rules

1. **No raw colours outside `tokens.css`.** Use the semantic roles (`--text`, `--bg`, `--accent-text` and so on) first. Use material tokens only inside material components.
2. **Text sits on flat areas only**: the leaf centre, the slab face, the polished copper panel, paper labels and caption bands. Never put text on a leaf edge, the stone frame, patina, or the sherd's grit.
3. **Tamil gets `lang="ta"`**, on the element or an ancestor. That selects Noto Serif Tamil, a size of at least 18px and a line height of at least 1.75. Tamil is always real text, never an image.
4. **State is never colour alone.** Hover lifts or thickens the underline, focus adds a 3px ring, active presses down, selected adds ✓, current page is bold with a bar, disabled is dashed.
5. **Motion is short and optional.** Nothing plays on its own. With `prefers-reduced-motion: reduce` every duration is 0 and every change is instant.
6. **Images** come only from rows marked `SHIP_OK` in `design/references/LICENSES.csv` (public domain, CC0, CC BY), always with a `credit-line`.

## Colour tokens

### Material palette

| Token | Value | Use it for |
|---|---|---|
| `--leaf-centre` | #C98A3F | Flat text area of a palm leaf |
| `--leaf-edge` | #A86A2C | Darker long edges and ends of a leaf. **No text.** Also the literature accent |
| `--leaf-bundle` | #4A3426 | Bundle seen edge-on; footer background |
| `--ink-soot` | #1E1712 | Text written on a leaf |
| `--thread` | #E9A23B | Binding thread, footer top rule. Decorative |
| `--catalogue-red` | #A33A2A | Catalogue numbers. **On paper only** (fails on the leaf) |
| `--wood`, `--wood-dark` | #7A5436, #553823 | Bundle boards |
| `--stone-light` | #B9B4A8 | Flat face of a slab (text goes here) |
| `--stone-mid` | #8A857A | Noisy slab frame. **No text** |
| `--stone-dark` | #4B4842 | Carved headings: **large text only** (≥ 18.7px bold) |
| `--stone-shadow` | #2B2925 | Body text on stone |
| `--stone-highlight` | #DCD8CF | Light edge of carved letters |
| `--temple-stone`, `--temple-shadow` | #C8BBA6, #5A4D3E | Chola temple-wall slab face and its carved headings or accent |
| `--terracotta` | #A84E34 | Primary button, accents. **Adjusted** from #B5553A |
| `--terracotta-deep` | #8F3F29 | Hover and pressed states, links |
| `--terracotta-soft` | #E8C9B0 | Tags, clay seal impressions |
| `--terracotta-light` | #C8704F | Sherd texture only |
| `--copper` | #8C5A3C | Copper plate |
| `--copper-sheen` | #D4A27F | Polished panel on a plate that carries text |
| `--copper-patina` | #56705E | Patina; chips in the kings section. **Adjusted** from #5E7A66 |
| `--copper-dark` | #3E2A1E | Engraved text, ring, seal |
| `--coin-gold` / `-light` / `-dark` | #C9A24A / #E4C97E / #8E6D24 | Gold coin face, highlight, rim; signet ring |
| `--coin-silver` / `-light` / `-dark` | #B7B9BC / #E1E2E4 / #7D8084 | Silver coin |
| `--coin-ink` | #2E2412 | Struck legend on a coin |
| `--paper` | #F6EEDD | Page ground; text on dark surfaces |
| `--paper-deep` | #EFE3CB | Alternate bands; disabled buttons |
| `--ink` | #241B14 | Body text |
| `--ink-muted` | #5E4E40 | Secondary text, captions, credit lines |
| `--rule` | #D8C7A6 | Hairlines. Decorative, carries no meaning |
| `--mask-on`, `--mask-off` | #000, transparent | Alpha helpers inside CSS masks. Never visible as colour |

**Adjusted to pass contrast** (the rule was kept; the tokens moved): `--terracotta` (#B5553A gave only 4.21:1 with paper text on the primary button) and `--copper-patina` (#5E7A66 gave only 4.09:1 with paper text).

**Added**, all derived from the given palette: `--catalogue-red` stays paper-only, plus `--terracotta-deep`, `--terracotta-light`, `--copper-sheen`, `--stone-highlight`, `--temple-*`, `--wood*`, `--coin-*-light/-dark`, `--coin-ink`, `--paper-deep`, `--ink-muted` and `--rule`.

### Semantic roles and section surfaces

Components read these roles, not material colours. Put one class on `<body>` or a `<section>` and the roles switch together, so visitors feel the material change between sections.

| Role | Default (paper) | What reads it |
|---|---|---|
| `--bg`, `--bg-alt` | paper, paper-deep | Page and alternate bands |
| `--text`, `--text-muted` | ink, ink-muted | Body and secondary text |
| `--accent` | terracotta | Top rule on nav/panels, borders, large italic accents. Not for small text |
| `--accent-text` | terracotta-deep | Links, eyebrows, `h1 em` |
| `--button-bg`, `--button-bg-hover`, `--button-text` | terracotta, terracotta-deep, paper | Primary button (the same in every section) |
| `--focus-ring`, `--focus-ring-on-dark` | ink, paper | Keyboard focus ring; dark areas (footer) swap to the on-dark ring |

| Class | Section | Material |
|---|---|---|
| `.surface-leaf` | Literature (Tirukkuṟaḷ, Sangam) | Palm leaf |
| `.surface-origins` | Origins, Tamil-Brahmi | Stone and pottery sherds |
| `.surface-kings` | Kings and grants | Copper plate and seals |
| `.surface-chola` | Chola | Temple-wall stone |
| `.surface-trade` | Trade, Roman contact | Coins and rings |

## Typography

| Token | Value | Use it for |
|---|---|---|
| `--font-tamil` | Noto Serif Tamil → Noto Sans Tamil → system Tamil fonts | All Tamil (applied automatically by `lang="ta"`) |
| `--font-display` | Fraunces (with italic) | Latin headings; `em` in headings gives the italic accent |
| `--font-body` | Fraunces | Latin running text |
| `--font-ui` | system-ui stack (Noto Serif Tamil for any Tamil inside) | Buttons, tags, nav, captions, credit lines |
| `--font-brahmi` | Semmozhi Brahmi → Noto Sans Brahmi | Tamil-Brahmi. Add class `script-brahmi` |
| `--font-grantha` | Semmozhi Grantha → Noto Sans Grantha | Grantha. Add class `script-grantha` |
| `--font-vatteluttu` | Semmozhi Vatteluttu (draft) | Vatteluttu. Add class `script-vatteluttu` |
| `--font-mono` | system monospace | Code in the style guide |

The historical-script fonts are the project's own Semmozhi fonts, already shipped in `fonts/` and used by `font.html` and `fonts.html`. The prompt mentioned Adinatha, but the project does not use it: only `website/font.html` mentions it, with an unverified licence. So it was not added.

**Sizes**: `--text-xs` 14px, `--text-sm` 15px, `--text-base` 17px, `--text-md` 20px, then fluid `--text-lg` (22–28px), `--text-xl` (28–40px) and `--text-2xl` (36–60px). **Line heights**: `--lh-latin` 1.55, `--lh-display` 1.15, `--lh-tamil` 1.75. **Measure**: `--measure` 68ch.

**Tamil minimum (18px, line height ≥ 1.7)**: `:lang(ta)` sets `--ta-floor: 18px` and `--ta-lh: 1.75`; outside Tamil they are 0. A component that sets its own small size must write it as

```css
font: 600 max(var(--text-sm), var(--ta-floor)) / max(1.2, var(--ta-lh)) var(--font-ui);
```

Then Tamil can never drop below 18px or 1.7, and Latin keeps its own size. Headings keep their larger sizes. The style-guide check scans every Tamil text node at 360, 768 and 1200px.

## Spacing, radii, shadows, motion

- **Spacing**: `--space-1` … `--space-8` = 4, 8, 12, 16, 24, 32, 48, 64px (in rem). `--gutter` is 16px, or 24px from 768px up. `--page-max` is 1200px. `--touch` is 44px, the minimum size for every control.
- **Radii**: `--radius-1` 4px (tags, labels), `--radius-2` 8px (figures, bands), `--radius-3` 16px (panels), `--radius-pill` (buttons, chips).
- **Shadows**: `--shadow-rest` / `--shadow-lift` for boxes. `--drop-rest` / `--drop-lift` are `filter` shadows for shapes cut by masks or clip-paths (leaf, sherd, bundle). `--carve-light` / `--carve-dark` do carved lettering. `--inset-rim` is the raised copper rim. `--shade-soft` / `--shade-strong` darken inside textures.
- **Motion** (CSS fallback; GSAP motion is described below): `--dur-hover` 150ms (2px lift via `--lift`); `--dur-flip` 350ms (leaf flip, half out and half in); `--dur-untie` 500ms (X thread slides off a bundle); `--dur-open` 200ms (boards open, overlapping the last 100ms, so the total is 600ms); `--ease-out`. Nothing is longer than 600ms. Reduced motion sets every duration to 0 and switches off all transitions and animations.
- **Breakpoints** (mobile first): 360px base, `48rem` (768px), `75rem` (1200px). The leaf strip uses a container query instead (horizontal at ≥ 55rem of its own width).

## Textures (`textures.css`)

Each texture is a custom property (`--tx-leaf`, …) for components, and a class (`.tx-leaf`, …) for plain blocks.

| Texture | Size | Flat area for text |
|---|---|---|
| `--tx-leaf` / `--tx-leaf-v` | 0.4 KB each | Centre band (15–85% across the leaf); grain lines only near the edges |
| `--tx-bundle` | 0.5 KB | None: text goes on the paper label |
| `--tx-wood` | 0.2 KB | None |
| `--tx-stone` / `--tx-temple` | 1.0 KB each (includes the noise tiles) | The slab face (`.stone-slab__face`) is flat |
| `--tx-copper` | 0.9 KB | The polished panel (`.copper-plate__panel`) is flat |
| `--tx-sherd` | 1.0 KB | Flat centre for the scratched mark; the caption sits below on the page |
| `--tx-coin-gold` / `--tx-coin-silver` / `--tx-clay` | 0.1 KB each | Smooth radial gradient; the legend sits in the lighter centre |

## Real materials (`materials.css`, Prompt 2)

Load it after `components.css`. It turns the flat textures into surfaces that look like the objects: leaf fibre, granite grain, hammered copper, fired clay, worn metal, twisted thread. `textures.css` stays as it was; the style guide shows both side by side under **Real materials**.

**How it works.** Every texture is an SVG filter (`feTurbulence` → `feDiffuseLighting` / `feSpecularLighting`, `feDisplacementMap` for rough edges) baked into a small file in `img/tex/`. The files are grey height/tone maps or alpha-only masks, never colours. CSS paints the token colours and blends the grey files over them (`overlay` for relief, `multiply` for stains and spots). So the rule "no raw colour outside `tokens.css`" still holds. The browser rasterises each file once and caches it; nothing animates a filter.

| Material | Class / variable | Layers (top first) | Flat area for text |
|---|---|---|---|
| Palm leaf | `.mat-leaf`, `.mat-leaf--holes`, `.mat-leaf-v`; `--mat-leaf` | age spots at the edges (multiply) · fibres (lit, overlay) · real fibre photo (overlay, horizontal leaf only) · darker ends · edge-to-centre gradient. Mask: torn edge + nicks. Holes: dark worn rim + bevel | Centre band; spots stay in the outer 20% |
| Stone | `.mat-stone`, `.mat-temple`, `.mat-rough`, `.mat-stone-face` | chisel marks · weathering streaks (multiply) · granite grain with mica and quartz specks · real rock photo · gradient. Mask: rough-cut edge | `.mat-stone-face` / `.stone-slab__face`: the same grain at a third of the strength |
| Copper | `.mat-copper`, `.mat-copper-panel` | hammered dents with dull specular light and slow tarnish · gradient; `::before` green patina (crusty mask, strongest at the edges); `::after` dull highlight that follows the pointer (`--sheen-x`) | `.mat-copper-panel` / `.copper-plate__panel` (polished) |
| Terracotta | `.mat-clay`; `--mat-clay` | slip-colour bands · sandy grain with pores and temper · real clay photo · gradient | None: the mark is decorative, the caption sits below |
| Coin | `.mat-gold`, `.mat-silver` | soft shine · fine pitting (lit) · Prompt 1 metal gradient; raised rim from inset shadows | Legend is large, raised relief |
| Thread, bundle | `.mat-bundle`, `.mat-wood` | stacked edges (lines of different tone) · worn boards with real fibre photo; thread twist = diagonal alpha bands; soft shadow on the leaves | Paper label |

**Text treatments**: `.mat-inked` (leaf: dark ink in a thin lighter scratch groove), `.mat-carved` (stone: shaded groove, dust in the groove, lit lower edge; uses `background-clip:text`), `.mat-engraved` (copper: patina trace on the groove's lower lip).

**Gotchas found while building it**
- On `file://`, Chrome blocks `mask-image` files (masks are CORS requests) but not background images. Every mask is therefore inlined in `materials.css` as a `data:` URI. The readable sources are `img/tex/leaf-edge.svg`, `leaf-edge-v.svg`, `stone-edge.svg` and `copper-patina.svg`. After editing one, re-inline it (URL-encode it, including `#` as `%23`).
- A relative `url()` stored in a custom property resolves against the stylesheet when used from a CSS file, but against the page when used from an inline `style`. Use the `--mat-*` stacks from `css/` files only.
- A tile drawn at a non-uniform size (`480px 100%`) needs `preserveAspectRatio="none"`, or the SVG is letterboxed with transparent strips.
- `feTurbulence` also puts noise in the alpha channel. Make it opaque (`feColorMatrix … 0 0 0 0 1`) before any `arithmetic` composite, or premultiplied alpha skews the result.

**Photo detail tiles.** Three tiles come from real photos: stone (STO-193, CC0), leaf and board fibre (PAL-081, CC0) and clay (POT-018, CC BY 3.0). Each is from a row with tier SHIP and a credit in `design/references/LICENSES.csv`. Each is a grey, high-passed, seamless crop with no text or faces, used at low contrast under the procedural layers. `credits-data.json` has the id, object, licence, credit, source URL and crop for each; the style guide shows the credit lines. `LICENSES.csv` repeats some ids, so rows are matched by surface + file name.

**Low-quality fallback.** A small script in `<head>` sets `data-quality="low"` on `<html>` for Save-Data, 2G, `deviceMemory ≤ 2`, `prefers-reduced-data: reduce`, or `?quality=low`. Every material then becomes a plain token colour, with no texture files and no SVG masks. `@media (prefers-reduced-data: reduce)` does the same without JS.

### Contrast on the real materials (measured on rendered pixels)

The token table below still holds, but textures shift the background. `_checks/material_contrast.mjs` + `material_contrast.py` capture each text element with and without its glyphs. They compare the glyph core with the local background, and with its worst 5% of pixels (the texture tone closest to the text).

| Text | Width | Glyph median | Background median | Ratio (median) | Ratio (worst 5%) | Needs | Result |
|---|---|---|---|---|---|---|---|
| Kural on the real leaf (horizontal) | 1200 | L=0.009 | L=0.282 | 5.59:1 | 5.04:1 | body ≥ 4.5 | PASS |
| Transliteration on the leaf | 1200 | L=0.009 | L=0.284 | 5.63:1 | 5.05:1 | body ≥ 4.5 | PASS |
| Meaning on the leaf | 1200 | L=0.009 | L=0.283 | 5.61:1 | 5.13:1 | body ≥ 4.5 | PASS |
| Kural on the real leaf (vertical, 360px) | 360 | L=0.009 | L=0.283 | 5.61:1 | 5.04:1 | body ≥ 4.5 | PASS |
| Meaning on the vertical leaf (360px) | 360 | L=0.009 | L=0.277 | 5.50:1 | 4.87:1 | body ≥ 4.5 | PASS |
| Inked text on .mat-leaf | 1200 | L=0.009 | L=0.280 | 5.56:1 | 5.04:1 | body ≥ 4.5 | PASS |
| Carved heading on the slab face | 1200 | L=0.063 | L=0.470 | 4.58:1 | 4.30:1 | large ≥ 3.0 | PASS |
| Body text on the slab face | 1200 | L=0.022 | L=0.472 | 7.21:1 | 6.75:1 | body ≥ 4.5 | PASS |
| Carved heading on temple stone | 1200 | L=0.076 | L=0.519 | 4.52:1 | 4.28:1 | large ≥ 3.0 | PASS |
| Body text on temple stone | 1200 | L=0.022 | L=0.518 | 7.86:1 | 7.41:1 | body ≥ 4.5 | PASS |
| Carved Tamil on .mat-stone-face | 1200 | L=0.060 | L=0.472 | 4.74:1 | 4.46:1 | large ≥ 3.0 | PASS |
| Engraved heading on the copper panel | 1200 | L=0.028 | L=0.395 | 5.73:1 | 5.53:1 | large ≥ 3.0 | PASS |
| Engraved body text on the copper panel | 1200 | L=0.028 | L=0.395 | 5.73:1 | 5.55:1 | body ≥ 4.5 | PASS |
| Engraved text on .mat-copper-panel | 1200 | L=0.028 | L=0.396 | 5.73:1 | 5.53:1 | body ≥ 4.5 | PASS |
| Legend on the gold coin | 1200 | L=0.019 | L=0.460 | 7.41:1 | 5.02:1 | large ≥ 3.0 | PASS |
| Legend on the silver coin | 1200 | L=0.019 | L=0.540 | 8.56:1 | 4.71:1 | large ≥ 3.0 | PASS |
| Tamil-Brahmi on the cave-bed strip | 1200 | L=0.058 | L=0.473 | 4.82:1 | 4.52:1 | large ≥ 3.0 | PASS |
| Body text on the cave-bed face | 1200 | L=0.022 | L=0.472 | 7.22:1 | 6.74:1 | body ≥ 4.5 | PASS |
| Vatteluttu line on the copper panel (decorative) | 1200 | L=0.028 | L=0.395 | 5.73:1 | 5.55:1 | body ≥ 4.5 | PASS |
| Vatteluttu lines on the top plate (decorative facsimile) | 1200 | L=0.028 | L=0.387 | 5.62:1 | 4.88:1 | body ≥ 4.5 | PASS |
| Manuscript facsimile lines (decorative) | 1200 | L=0.009 | L=0.277 | 5.52:1 | 4.70:1 | body ≥ 4.5 | PASS |
| Kural on the vertical leaf, 360px (after the precision pass) | 360 | L=0.009 | L=0.283 | 5.61:1 | 5.04:1 | body ≥ 4.5 | PASS |

0 failing target(s).

The "(decorative)" rows are facsimile lines marked `aria-hidden`. They are checked anyway and pass.

## Motion (GSAP, `js/motion.js`, Prompt 2)

GSAP 3.15.0 is self-hosted in `js/vendor/gsap/`: `gsap`, `ScrollTrigger`, `Draggable`, `DrawSVGPlugin`, `MorphSVGPlugin`, `SplitText`, `CustomEase` and `Flip` (Flip is used by the identity page), 219 KB in total. Pages load only the plugins they use. There is no CDN, so the site opens by double-click. **Licence** (`js/vendor/gsap/LICENSE.txt`): the Standard "No Charge" GSAP License lets any website use GSAP and all these plugins free of charge. It forbids only use in no-code visual animation builders that compete with Webflow, and removing GSAP's notices. `MotionPathPlugin` is not used, so it is not shipped.

`js/motion.js` holds every timeline under a clear name. `components.js` calls them through data attributes, so a page needs no extra script:

| Call | Markup hook | What moves | Time |
|---|---|---|---|
| `Motion.leafFlip(stack)` | `[data-leaf-stack]` | Top leaf lifts, swings on its left hole (≤ 7°), turns over (rotateX) with a two-part skew bend, fades; the next leaf rises from underneath; the lift shadow follows. Draggable: drag left = next, right = previous, otherwise it springs back. The Prev/Next buttons and ← → keys always work | 0.7 s |
| `Motion.untieBundle(el, {fan, reader})` | `a.leaf-bundle[href]`, `button.leaf-bundle[aria-controls]` | Knot loosens (MorphSVG), thread unwinds (DrawSVG), top board swings on its hinge, 5 leaves fan out (stagger), reader opens; `reverse()` ties it up again | 1.14 s (link: 0.76 s) |
| `Motion.inkWrite(el)` | `[data-ink]` (plays on first view) | Iron nib runs along each line: the scratch groove first, then the ink 16px behind. The real text stays in the DOM; two `aria-hidden` copies are clipped | 1.15 s |
| `Motion.carveStone(el)` | `[data-carve]` | Letters struck one by one (SplitText, Tamil-safe grapheme split), dust falls, then the flat letters fade into the carved groove | 0.72 s |
| `Motion.copperSheen(el)` | `.copper-plate`, `[data-sheen]` | Dull highlight follows the pointer, or device tilt where no permission prompt is needed | follows input |
| `Motion.ringSpring(plate)` | `.copper-plate` | Ring swings on its hole with a small elastic spring when the plate first scrolls into view | 1.1 s |
| `Motion.scrollStory(section)` | `[data-story]` | One pinned scene: stone → copper → leaf; the back layers cross-fade with parallax and a soft blur (depth of field) | scroll-linked |
| `Motion.intro({title})` | `<body data-intro>`, `?intro=1` | One leaf slides in, both holes catch the light, the title is inked. Any key, click, scroll or Skip ends it. Once per session | 1.75 s |
| `Motion.micro(root)` | automatic | Buttons and chips press in with a spring (`--press` → `scale`), card links lift (`--card-y` → `translate`); stylus cursor over leaves (CSS, fine pointers only) | 0.08 s / 0.55 s |
| `Motion.flipOpen(card, panel)`, `Motion.rise(els)` | identity page | Card grows into the detail view (Flip); emblems rise and settle | 0.55 s / ≤ 1.1 s |

**Rules kept in code**
- `prefers-reduced-motion: reduce`: all setup runs inside `gsap.matchMedia()`, and every call first checks `Motion.reduced()`. No tween runs; final states show at once (the next leaf, the open reader, the finished text). The check counts GSAP ticks with an active tween: 0.
- Without JS, or without GSAP, `Motion.ok` is false and the Prompt 1 CSS motion (the tokens above) stays in charge.
- Only transform and opacity move, except the copper sheen (one gradient layer's position), the ink clip (per line) and the carving depth (one custom property on one heading). Nothing longer than 1.2 s except scroll-linked motion.
- The only loop is the dust in the scroll story: 14 specks at opacity 0.12, paused while the tab is hidden.
- `will-change` is set only while something moves (leaf flip, drag, untie) and on the pinned story layers.

**Gotchas found while building it**
- DrawSVG cannot measure a path with `vector-effect: non-scaling-stroke` inside a non-uniformly stretched SVG. `untieBundle` redraws the thread 1:1 in pixels before it unties, then restores the markup.
- GSAP folds the CSS `scale` property into its own transform (and sets `scale: none` inline). Size an element with CSS `scale` only if you expect that.
- Blur on large layers costs a lot. The story's back layers are drawn at half size with half the blur, then scaled 2×.
- Headless Chrome caps `requestAnimationFrame` at 30 fps even on an idle page. The frame-rate check therefore runs with vsync and the frame-rate limit off, and reports how many frames Chrome can make.

## Precision spec (Prompt 2c)

Measured on reference photos in `design/references/`. Only rows with tier SHIP and a credit in `LICENSES.csv` were used, matched by surface + file name. "Measured" means pixels were counted: segmentation, row/column profiles, or a 5% grid on the crop. "Estimate" means read by eye, taken from general knowledge, or not measurable from a 2-D photo. Side-by-side sheets: `_checks/precision-<component>.png`. Regenerate them with `node _checks/precision.mjs && python3 _checks/precision.py`; the reference crops appear only in those check images and are never used on the site.

| Component | What | Value (range) | Source ids | How | Estimate? |
|---|---|---|---|---|---|
| Leaf strip | Length : breadth | 6.8–8.1 : 1 (7.2, 8.1, 7.6, 6.8, 6.8). Built as 7.6 default, 6.8 `--short`, 9 `--long` (9 = the brief's Tirukkural scan) | PAL-016, PAL-017, PAL-028, PAL-029 (2 leaves) | segmentation of flat scans | measured |
| Leaf strip | Hole centres along the length | first 27–33%, second 65–75%. Built 30% / 70% | PAL-016, -017, -028, -029 | 5% grid on the crop | measured, ±1% |
| Leaf strip | Hole diameter ÷ breadth | 10–24%, about 15% typical. Built about 14% | same | grid, by eye | estimate, ±3% |
| Leaf strip | Script lines per leaf | 8–12 (the brief said 5–7). Facsimile built with 8 | PAL-016 (11–12), -017 (~11), -028 (9), -029 (8–9) | row profile + by eye | measured |
| Leaf strip | Text around holes | lines run past each hole above and below; only the middle lines break. Built: columns that stop at each hole | PAL-029, PAL-016 | by eye | simplification |
| Leaf strip | Margin note | short column, about 10–13% of the length, at the left | PAL-028, PAL-029 | by eye | estimate |
| Leaf strip | Folio number | written on the leaf at the right end. Facsimile does the same; the reading leaf keeps its paper tag (contrast) | PAL-016, PAL-017 | by eye | measured |
| Leaf strip | Colour | centre #F27704–#F8A554 on bright scans, #A9691A on a dark one; edges always darker. Tokens #C98A3F / #A86A2C kept (inside the range) | PAL-016, -017, -028, -029 | median pixels | measured, camera-dependent |
| Leaf strip | Cord | cream cotton through the left hole, about as thick as the hole is wide. Built 3px | PAL-084, -087, -088, -090 | by eye | estimate |
| Leaf bundle | Thread | undyed cream cotton, new token `--thread-cotton` #E6DDC8 | PAL-083, -084, -088, -090 | by eye | estimate (colour) |
| Leaf bundle | Tight turns at the label end | 3–5. Built 3 | PAL-083, -088 | counted | measured |
| Leaf bundle | Criss-cross passes | 8–10 along the whole bundle, about 9% apart. Built 3, 9% apart, on the last third (keeps the title readable) | PAL-083, -084, -088 | counted | measured; built differently on purpose |
| Leaf bundle | Knot | over the first hole, 25–35% along. Built 30% | PAL-083, -088 | by eye | estimate |
| Leaf bundle | Boards | a little longer than the leaves (about 2–4% each end), rounded ends. Built 2.5% | PAL-084, PAL-081 | by eye | estimate |
| Stylus, knife | Proportions | point about 55% of the length, taper over the last tenth. The case shows all-iron tools and hooked knives | PAL-035, PAL-036 | by eye | estimate (no scale in the photo) |
| Stone | Cave bed | smooth weathered rock, dark stains. No SHIP photo of the Arittapatti or Mangulam Tamil-Brahmi beds | STO-193 | by eye | estimate |
| Stone | Letter stroke ÷ letter height | about 1/8 on temple walls; letter depth is not measurable from photos | STO-064, STO-024 | by eye | estimate |
| Copper plate | Plate body w : h | 1.54 and 1.53 (without the lug). Built 1.55 | COP-001, COP-002 | segmentation | measured |
| Copper plate | Ring hole | near the left edge, mid-height. Built at 6% of the width | COP-001, -002, -028 | by eye | estimate |
| Copper plate | Seal size ÷ plate height | about 0.45. Built about 0.45 | COP-028 | by eye, photo at an angle | estimate |
| Seal | Parts | raised rim, Grantha legend band, raised emblem (tiger, two fish, bow, parasol, lamps). Built with a placeholder emblem | COP-027 | by eye | estimate |
| Coin | Roundness (narrowest ÷ widest width) | 0.89–0.92 and 0.86–0.87 (notched). Built about 0.91; denarius about 0.98 | COI-006, COI-013 | segmentation, 36 angles | measured |
| Coin | Strike | off-centre, the bead border runs off one side. Built 4% off | COI-006, COI-013, COI-050 | by eye | estimate |
| Coin | Punch-marked | no SHIP photo yet (COI-045 renders black). Squarish flan, 4 symbols | — | general description | estimate |
| Sherd | Black-and-red ware | black rim and upper body, red below, soft boundary | POT-018 | by eye | estimate |
| Sherd | Thickness, marks | no SHIP photo of a single inscribed sherd | — | — | estimate |
| Ring | Bezel ÷ ring width, band ÷ ring width | about 0.55 and 0.12. No SHIP photo of an early Tamil ring | RIN-027, RIN-048, RIN-051 | by eye | estimate |
| Scale | Real sizes | leaf ~400 × 53 mm, plate ~280 × 175 mm, stylus ~200 mm, sherd ~90 × 70 mm, coin ~19 mm, slab 1–2 m | — | general knowledge | estimate |

**Physical rules (checked by `_checks/checks_p2.mjs` → `physics`)**
- A leaf flexes but does not fold: the bend (skewX) stays at 5° or less, and the in-plane swing at 7° or less.
- Copper does not bend: plates turn as rigid bodies around the ring (no scale or skew).
- Stone does not move: slab links have no hover lift. Hover underlines the carved title and deepens the shadow.
- Coins and sherds tilt a few degrees (3°) and never lift away.
- One light, top-left: every shadow falls down and to the right. `--drop-thin` (leaf, plate), `--drop-mid` (coin, sherd, seal) and `--drop-thick` (slab, bundle) follow the object's thickness.
- Scale: the **Object scale** section of the style guide draws every object at one millimetre = `--mm`.

**Historical script on each object**: palm leaf in modern round Tamil; stone (cave bed) and pottery in Tamil-Brahmi (Semmozhi Brahmi); copper plates in Vatteluttu shapes (Semmozhi Vatteluttu, a draft font); the seal legend in Grantha. All sample inscriptions are placeholders and say so in their captions. The temple-wall slab keeps modern Tamil, because there is no font for the Chola-period letterforms.

**Facsimile exception**: the manuscript leaf's script lines and the copper set's engraved lines are pictures of text. They are real Tamil text drawn at object scale, marked `aria-hidden`, with the readable text in the caption at 18px or more. The 18px Tamil floor applies to reading text; these facsimiles are excluded from it on purpose.

## Identity kit (`identity.html`, Prompt 2b)

`design/svg_kit/identity_build.py` generates everything: one SVG per item in `identity/<group>/<slug>.svg` (copied to `design/svg_kit/identity/`), `identity.json`, `identity-data.js` (the same data, because `file://` blocks fetch), `identity-svg.js` (the markup, loaded on first open) and `CONTENT_TO_VERIFY.md`. Edit the items there and rebuild. Do not edit the output files.

- **One family**: 512 viewBox, 32-unit margin, 16-unit round strokes. Three `<symbol>` variants per file (`-solid`, `-line`, `-material`) plus `<view id="solid|line|material">`, so `<img src="x.svg#material">` shows one. Colours are tokens with fallbacks, so a file works both inline and standalone. Every file is under 8 KB.
- **Materials**: dynasties are embossed on copper, people and script carved in stone, books inked on palm leaf.
- **Script letters** are real outlines from the project fonts, read by `design/svg_kit/ttf_outline.py` (no dependencies). Book titles and the letter grid are real text.
- **Page**: cards use lazy `<img>`; only the open item is inlined. Filter chips, a search that works on Tamil and English (கோவில் and கோயில் count as the same word), arrow keys between cards (one tab stop), Enter or Space opens, Escape closes, Tab stays inside the dialog. The card grows into the detail view with `Motion.flipOpen`, and the hero emblems use `Motion.rise`.
- **Honesty**: `verified` is false on every row until a person checks it. Fields marked "unverified" are never shown. People are symbolic figures, or a crown and emblem for kings.
- **Checks**: `node _checks/checks_p2.mjs identity.html`, `node _checks/checks_identity.mjs`, `python3 _checks/identity_svgs.py`.

## Components (`components.css`)

Plain HTML and CSS; add a class. `.is-hover`, `.is-focus` and `.is-active` exist only so the style guide can show states statically.

| Class | When to use | Notes |
|---|---|---|
| `leaf-strip` | One kural or couplet: number, Tamil, transliteration, meaning | Needs an inner `.leaf-strip__leaf` whose first child is `<span class="leaf-strip__surface" aria-hidden="true">`. It is 9:1 with holes at 30% and 70% when it has ≥ 880px; otherwise a 3:4 vertical leaf with holes top and bottom. `--vertical` forces the phone layout. Put several in `[data-leaf-stack]` with `[data-leaf-prev]` / `[data-leaf-next]` buttons for the flip |
| `leaf-bundle` | Navigation card to a text or collection | `<a href>`; the paper label (`.leaf-bundle__label`, Tamil title) sits inside `.leaf-bundle__art`. For "coming soon", use `role="link" aria-disabled="true"` with no href |
| `stone-slab` | Origins content, inscriptions | Heading `.stone-slab__title` (carved, large); body in `.stone-slab__face`. `--temple` is the Chola variant |
| `copper-plate` | Kings, grants | Ring and seal are inline SVG (`aria-hidden`); text goes in `.copper-plate__panel` |
| `coin-card` | Coins | `--silver` variant. The coin is `role="img"` with an `aria-label` naming its legend; `.coin-card__band` is the caption |
| `ring-seal` | Signet ring and its impression | The ring's legend is mirrored; the impression reads correctly. Both are `role="img"` with labels |
| `sherd-card` | Inscribed pottery | Irregular clip-path shard, scratched mark, caption below |
| `button`, `button--secondary`, `button--icon` | Actions | Primary is terracotta; secondary is outlined; disabled is dashed |
| `a` (base) | Links | Always underlined; hover thickens the underline |
| `tag` (`--stone`, `--patina`, `--gold`) | Static labels | Not interactive |
| `chip` | Filter toggles | `<button aria-pressed>`; selected shows ✓ and a heavier border |
| `source-cite` | "Source:" line under a text | `.source-cite__label` + `<cite>` |
| `breadcrumb` | Page location | `<nav aria-label="Breadcrumb"><ol>`; current page `aria-current="page"` |
| `topnav` | Site navigation | Below 768px a Menu button (`aria-expanded`; Escape closes). Current page = bold + accent bar |
| `site-foot` | Footer | Bundle-brown ground, thread-coloured rule, on-dark focus ring |
| `topnav__tools` + `theme-switch` | Day / System / Night switch in the nav (T10) | Three `<button aria-pressed>` in a `role="group"`; selected = filled ground **and** bold **and** ✓. Sets `data-theme` on `<html>`, kept in `localStorage` (`sem-theme`, the key the older pages use); System removes the attribute and follows `prefers-color-scheme`. `?theme=dark|light` forces one for a screenshot |
| `subnav` | Sub-navigation inside one section (T10) | `<nav><ul class="subnav__list">`; scrolls sideways when it is too long (give it `tabindex="0" role="region" aria-label`), current item `aria-current="true"` = bold + accent bar |
| `page-foot` | Footer that follows the page ground (T10) | Use instead of `site-foot` when the footer should change with the mode. Its own component, not a modifier: `site-foot` pins the roles to the bundle material |
| `timeline` | Dated events (T10) | `<ol>`; each `.timeline__item` has a `.timeline__dot` (`aria-hidden`), `.timeline__date`, `.timeline__title`, `.timeline__body`. `--uncertain` = hollow dot, and add a `.timeline__flag` reading "unverified". `.timeline--across` runs sideways from 768px with the same markup |
| `search` | Search field, filters, results, empty state (T10) | `<form data-search role="search">` with `.search__field` (own 3px focus ring), chips as filters, `.search__count` (`role="status"`), `.search__results` `<ul>`, `.search__empty` (`hidden` until nothing matches). `js/components.js` filters the results already in the page; there is no index file yet |
| `map-ph` | Placeholder where a map will go (T10) | A 4:3 box with a graticule, an invented land shape, pins with text labels, and a `.map-ph__note` that says in the picture that it is not a map of anywhere |
| `card`, `card-grid` | Text cards and their grid (T10) | `card--link` (the title's `<a>` covers the card: one tab stop), `card--row` (media beside the text from 768px), `card--quiet` (outline only), `aria-disabled="true"` (dashed + muted). Object cards — leaf, slab, plate, coin, ring, sherd — are the rows above |
| `figure` + `credit-line` | Any image | `credit-line` = object name · licence · credit · source, from a `SHIP_OK` row of `design/references/LICENSES.csv` |
| `quote` | Quotations | Tamil original, translation, source |
| `leaf-strip--short` / `--long` | Leaves of different lengths in a stack | Same breadth; 6.8 : 1 and 9 : 1 (default 7.6 : 1) |
| `leaf-strip__cord` | Cord through the left hole | `<span class="leaf-strip__cord" aria-hidden="true">` after the surface; horizontal leaves only |
| `leaf-strip--manuscript` | A written leaf as an object (facsimile) | `.leaf-strip__script` holds `__margin`, `__col--a/b/c`, `__folio` (all `aria-hidden`); readable text in the `figcaption` |
| `leaf-bundle__tag`, `leaf-bundle__knot` | Catalogue number tag, knot | Inside `.leaf-bundle__art`; both `aria-hidden` |
| `tool` | Stylus, leaf knife | Inline SVG with `role="img"` and a `<title>`; parts `tool__wood`, `tool__iron`, `tool__band`, `tool__shine` |
| `stone-slab--cavebed` | Early cave inscriptions | `.stone-slab__strip` (smoothed band) with `.stone-slab__brahmi` (`lang="ta-Brah"`), then the usual face |
| `copper-set` | A grant as a set of plates | `[data-copper-set]`; plates `.copper-set__plate.mat-copper` with `--i` (0 = top); a `[data-copper-fan]` button with `aria-pressed` fans them around the ring |
| `seal` | Cast seal | Inline SVG: rim, Grantha legend on a `textPath` (unique ids per seal), raised emblem drawn three times (shadow, highlight, relief) |
| `coin-card__thick` | Wraps `.coin-card__coin` | Shows the coin's thickness; variants `coin-card--punch`, `coin-card--denarius` |
| `sherd-card--brw` | Black-and-red ware | Black rim, red body; the broken edge's thickness shows below right on every sherd |
| `ring-seal__top` | Bezel seen from above | Between the side view and the impression |
| `obj-scale` | Relative real sizes | Items take `--w` and `--h` in millimetres |

## Dark mode (T10)

Three states, the same as the older pages (`css/style.css`, `js/app-1.js`):

| `<html>` | Mode |
|---|---|
| no `data-theme` | follow the operating system (`prefers-color-scheme`) |
| `data-theme="dark"` | night, whatever the system says |
| `data-theme="light"` | day, whatever the system says |

**Only the page ground and the semantic roles change.** Every material colour — `--paper`,
`--ink`, `--leaf-centre`, `--leaf-bundle`, `--stone-light`, `--copper-sheen`, `--coin-gold` … — is
measured from the objects (Prompt 4, T1/T6) and is identical in both modes: a leaf is the same amber
at night, and the bundle-brown footer stays bundle-brown. Two reasons, not one: those colours are
data, and `--paper`/`--ink` are also used as *text* colours on dark materials (paper on
`--leaf-bundle`, on `--copper-dark`), so redefining them would break pairs that already pass.

The night ground and the lifted accents (`tokens.css`):

| Token | Value | Role at night |
|---|---|---|
| `--night` | `#16120E` | `--bg` |
| `--night-deep` | `#201A14` | `--bg-alt` |
| `--night-raise` | `#2A2219` | `--surface` (cards, fields, panels) |
| `--night-text` | `#F1E7D6` | `--text`, `--focus-ring` |
| `--night-muted` | `#C3B49E` | `--text-muted` |
| `--night-rule` | `#4A3E30` | `--rule` (decorative hairlines) |
| `--night-edge` | `#8F7A5C` | `--edge` (borders that carry meaning; 3.57:1 on the darkest ground) |
| `--terracotta-bright` | `#E08A62` | `--accent`, `--accent-text`, `--button-bg` |
| `--leaf-bright` / `--copper-bright` / `--temple-bright` / `--coin-gold-bright` | `#E0AE6A` / `#D89C74` / `#CFC0A6` / `#E0BE6A` | the accent inside `.surface-leaf` / `-kings` / `-chola` / `-trade` |

Day added two tokens as well: `--paper-raise` `#FFFBF2` (`--surface`) and `--edge` `#87734F`
(3.15:1 on the lightest ground; `--rule` stays the decorative hairline at 1.6:1).

Each of the five section surfaces becomes a dark tint of its own material
(`--dk-leaf-bg` `#1C150D`, `--dk-origins-bg` `#171613`, `--dk-kings-bg` `#1D140E`,
`--dk-chola-bg` `#1A170F`, `--dk-trade-bg` `#1B1608`, each with an `-alt` and a `-raise`).

`.theme-night` and `.theme-day` put one mode inside the other, for showing a component in both modes
on one page (the style guide's `#components` section does this). `contrast_components.py` asserts
those two classes resolve to exactly the same colours as the real modes, so the preview cannot lie.

**Checks.** `python3 design/realism/eval/contrast_components.py` (tokens, both modes; exit 1 on any
failing pair) and `node design/realism/tools/t10_modes_probe.mjs` (the same components measured on the
rendered page in both modes, so a rule that never applies is caught too; result saved as
`design/realism/eval/t10_modes.json`). Numbers at T10: 182 of 182 token pairs pass, and 44 of 44
rendered pairs pass — worst 4.78:1 in light, 5.68:1 in dark, against 4.5:1 for body text.

## Accessibility summary

- Focus ring: 3px solid, 3px offset, ink on light surfaces and paper on dark ones. Tab order follows the DOM.
- Decorative textures and SVG are `aria-hidden="true"`. Drawn objects that carry meaning (coins, seals, sherds, the figure drawing) are `role="img"` with a text label.
- Touch targets are ≥ 44 × 44px, except links inside running text, which are allowed to be smaller.
- A scrollable table gets `tabindex="0" role="region" aria-label`, so keyboard users can scroll it.

## Contrast table

WCAG 2.x: body text ≥ 4.5:1; large text (≥ 24px, or ≥ 18.7px bold) ≥ 3:1; focus rings and borders ≥ 3:1. Generated from `css/tokens.css` by `python3 _checks/contrast.py` (run from `website_live/`; it exits 1 if any pair fails). Re-run it after any token change.

### Text and focus pairs

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--ink` #241B14 | `--paper` #F6EEDD | 14.65:1 | body ≥ 4.5 | PASS | Body text on the page |
| `--ink-muted` #5E4E40 | `--paper` #F6EEDD | 6.89:1 | body ≥ 4.5 | PASS | Secondary text, captions, credit lines |
| `--ink` #241B14 | `--paper-deep` #EFE3CB | 13.30:1 | body ≥ 4.5 | PASS | Body text on alternate bands |
| `--ink-muted` #5E4E40 | `--paper-deep` #EFE3CB | 6.26:1 | body ≥ 4.5 | PASS | Secondary text on alternate bands |
| `--terracotta-deep` #8F3F29 | `--paper` #F6EEDD | 6.25:1 | body ≥ 4.5 | PASS | Links, secondary-button label |
| `--terracotta-deep` #8F3F29 | `--paper-deep` #EFE3CB | 5.68:1 | body ≥ 4.5 | PASS | Links on alternate bands |
| `--terracotta` #A84E34 | `--paper` #F6EEDD | 4.78:1 | body ≥ 4.5 | PASS | Accent text (eyebrows, italic accents) |
| `--paper` #F6EEDD | `--terracotta` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Primary button label |
| `--paper` #F6EEDD | `--terracotta-deep` #8F3F29 | 6.25:1 | body ≥ 4.5 | PASS | Primary button label, hover/pressed |
| `--ink` #241B14 | `--terracotta-soft` #E8C9B0 | 10.80:1 | body ≥ 4.5 | PASS | Tag / chip text |
| `--ink-muted` #5E4E40 | `--terracotta-soft` #E8C9B0 | 5.08:1 | body ≥ 4.5 | PASS | Muted text inside a chip |
| `--catalogue-red` #A33A2A | `--paper` #F6EEDD | 5.69:1 | body ≥ 4.5 | PASS | Catalogue number on its paper tag |
| `--ink-soot` #1E1712 | `--leaf-centre` #C98A3F | 6.06:1 | body ≥ 4.5 | PASS | Kural text on the flat leaf centre |
| `--paper` #F6EEDD | `--leaf-bundle` #4A3426 | 10.05:1 | body ≥ 4.5 | PASS | Text on a dark bundle |
| `--thread` #E9A23B | `--leaf-bundle` #4A3426 | 5.36:1 | large ≥ 3 | PASS | Thread-coloured large lettering on a bundle |
| `--ink` #241B14 | `--stone-light` #B9B4A8 | 8.18:1 | body ≥ 4.5 | PASS | Body text on a stone-slab face |
| `--stone-shadow` #2B2925 | `--stone-light` #B9B4A8 | 7.02:1 | body ≥ 4.5 | PASS | Warm-shadow text on a slab face |
| `--stone-dark` #4B4842 | `--stone-light` #B9B4A8 | 4.41:1 | large ≥ 3 | PASS | Carved heading on a slab (large only) |
| `--paper` #F6EEDD | `--stone-dark` #4B4842 | 7.89:1 | body ≥ 4.5 | PASS | Text on a dark slab |
| `--ink` #241B14 | `--temple-stone` #C8BBA6 | 8.95:1 | body ≥ 4.5 | PASS | Body text on the Chola temple-stone face |
| `--temple-shadow` #5A4D3E | `--temple-stone` #C8BBA6 | 4.33:1 | large ≥ 3 | PASS | Carved Chola heading (large only) |
| `--copper-dark` #3E2A1E | `--copper-sheen` #D4A27F | 5.96:1 | body ≥ 4.5 | PASS | Text engraved on the polished plate area |
| `--paper` #F6EEDD | `--copper` #8C5A3C | 4.99:1 | body ≥ 4.5 | PASS | Text on plain copper |
| `--paper` #F6EEDD | `--copper-dark` #3E2A1E | 11.70:1 | body ≥ 4.5 | PASS | Text on dark copper |
| `--paper` #F6EEDD | `--copper-patina` #56705E | 4.69:1 | body ≥ 4.5 | PASS | Chip on patina (kings section) |
| `--coin-ink` #2E2412 | `--coin-gold` #C9A24A | 6.35:1 | large ≥ 3 | PASS | Coin legend on a gold face |
| `--coin-ink` #2E2412 | `--coin-silver` #B7B9BC | 7.75:1 | large ≥ 3 | PASS | Coin legend on a silver face |
| `--ink` #241B14 | `--coin-gold` #C9A24A | 7.05:1 | body ≥ 4.5 | PASS | Text on a gold band |
| `--ink` #241B14 | `--coin-silver` #B7B9BC | 8.60:1 | body ≥ 4.5 | PASS | Text on a silver band |
| `--copper-dark` #3E2A1E | `--terracotta-soft` #E8C9B0 | 8.63:1 | large ≥ 3 | PASS | Seal impression lettering (raised clay) |
| `--terracotta-soft` #E8C9B0 | `--terracotta` #A84E34 | 3.53:1 | non-text ≥ 3 | PASS | Scratched mark on a sherd (decorative; the image has a text label) |
| `--ink` #241B14 | `--paper` #F6EEDD | 14.65:1 | non-text ≥ 3 | PASS | Focus ring on the page |
| `--ink` #241B14 | `--leaf-centre` #C98A3F | 5.79:1 | non-text ≥ 3 | PASS | Focus ring on a leaf |
| `--paper` #F6EEDD | `--leaf-bundle` #4A3426 | 10.05:1 | non-text ≥ 3 | PASS | Focus ring on dark surfaces (on-dark ring) |
| `--paper` #F6EEDD | `--copper-dark` #3E2A1E | 11.70:1 | non-text ≥ 3 | PASS | Focus ring on dark copper (on-dark ring) |
| `--terracotta` #A84E34 | `--paper` #F6EEDD | 4.78:1 | non-text ≥ 3 | PASS | Secondary-button border |

### Section surface `.surface-leaf`

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #241B14 | `--bg` #F7EAD3 | 14.22:1 | body ≥ 4.5 | PASS | Body text |
| `--text` #241B14 | `--bg-alt` #EDD5AE | 11.86:1 | body ≥ 4.5 | PASS | Body text on alternate band |
| `--text-muted` #5E4E40 | `--bg` #F7EAD3 | 6.69:1 | body ≥ 4.5 | PASS | Secondary text |
| `--text-muted` #5E4E40 | `--bg-alt` #EDD5AE | 5.58:1 | body ≥ 4.5 | PASS | Secondary text on alternate band |
| `--accent-text` #7E4A1C | `--bg` #F7EAD3 | 6.13:1 | body ≥ 4.5 | PASS | Links, eyebrows, italic accents |
| `--accent-text` #7E4A1C | `--bg-alt` #EDD5AE | 5.11:1 | body ≥ 4.5 | PASS | Links on alternate band |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Primary button label |
| `--accent` #A86A2C | `--bg` #F7EAD3 | 3.71:1 | non-text ≥ 3 | PASS | Top rule, borders |

### Section surface `.surface-origins`

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #241B14 | `--bg` #EEEBE5 | 14.21:1 | body ≥ 4.5 | PASS | Body text |
| `--text` #241B14 | `--bg-alt` #DEDAD1 | 12.12:1 | body ≥ 4.5 | PASS | Body text on alternate band |
| `--text-muted` #524E47 | `--bg` #EEEBE5 | 6.95:1 | body ≥ 4.5 | PASS | Secondary text |
| `--text-muted` #524E47 | `--bg-alt` #DEDAD1 | 5.93:1 | body ≥ 4.5 | PASS | Secondary text on alternate band |
| `--accent-text` #8F3F29 | `--bg` #EEEBE5 | 6.06:1 | body ≥ 4.5 | PASS | Links, eyebrows, italic accents |
| `--accent-text` #8F3F29 | `--bg-alt` #DEDAD1 | 5.17:1 | body ≥ 4.5 | PASS | Links on alternate band |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Primary button label |
| `--accent` #A84E34 | `--bg` #EEEBE5 | 4.64:1 | non-text ≥ 3 | PASS | Top rule, borders |

### Section surface `.surface-kings`

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #241B14 | `--bg` #F5E9DE | 14.17:1 | body ≥ 4.5 | PASS | Body text |
| `--text` #241B14 | `--bg-alt` #E9D3C1 | 11.72:1 | body ≥ 4.5 | PASS | Body text on alternate band |
| `--text-muted` #5A4536 | `--bg` #F5E9DE | 7.52:1 | body ≥ 4.5 | PASS | Secondary text |
| `--text-muted` #5A4536 | `--bg-alt` #E9D3C1 | 6.22:1 | body ≥ 4.5 | PASS | Secondary text on alternate band |
| `--accent-text` #76492F | `--bg` #F5E9DE | 6.36:1 | body ≥ 4.5 | PASS | Links, eyebrows, italic accents |
| `--accent-text` #76492F | `--bg-alt` #E9D3C1 | 5.26:1 | body ≥ 4.5 | PASS | Links on alternate band |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Primary button label |
| `--accent` #8C5A3C | `--bg` #F5E9DE | 4.83:1 | non-text ≥ 3 | PASS | Top rule, borders |

### Section surface `.surface-chola`

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #241B14 | `--bg` #EFE9DF | 14.00:1 | body ≥ 4.5 | PASS | Body text |
| `--text` #241B14 | `--bg-alt` #DFD5C5 | 11.64:1 | body ≥ 4.5 | PASS | Body text on alternate band |
| `--text-muted` #544A3F | `--bg` #EFE9DF | 7.16:1 | body ≥ 4.5 | PASS | Secondary text |
| `--text-muted` #544A3F | `--bg-alt` #DFD5C5 | 5.96:1 | body ≥ 4.5 | PASS | Secondary text on alternate band |
| `--accent-text` #5A4D3E | `--bg` #EFE9DF | 6.78:1 | body ≥ 4.5 | PASS | Links, eyebrows, italic accents |
| `--accent-text` #5A4D3E | `--bg-alt` #DFD5C5 | 5.64:1 | body ≥ 4.5 | PASS | Links on alternate band |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Primary button label |
| `--accent` #5A4D3E | `--bg` #EFE9DF | 6.78:1 | non-text ≥ 3 | PASS | Top rule, borders |

### Section surface `.surface-trade`

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #241B14 | `--bg` #F7EFD8 | 14.73:1 | body ≥ 4.5 | PASS | Body text |
| `--text` #241B14 | `--bg-alt` #ECDFB8 | 12.73:1 | body ≥ 4.5 | PASS | Body text on alternate band |
| `--text-muted` #584A32 | `--bg` #F7EFD8 | 7.50:1 | body ≥ 4.5 | PASS | Secondary text |
| `--text-muted` #584A32 | `--bg-alt` #ECDFB8 | 6.48:1 | body ≥ 4.5 | PASS | Secondary text on alternate band |
| `--accent-text` #6E520E | `--bg` #F7EFD8 | 6.36:1 | body ≥ 4.5 | PASS | Links, eyebrows, italic accents |
| `--accent-text` #6E520E | `--bg-alt` #ECDFB8 | 5.50:1 | body ≥ 4.5 | PASS | Links on alternate band |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Primary button label |
| `--accent` #8E6D24 | `--bg` #F7EFD8 | 4.19:1 | non-text ≥ 3 | PASS | Top rule, borders |

### Surfaces that carry no text (by design)

| Pair | Ratio | Rule |
|---|---|---|
| `--ink-soot` on `--leaf-edge` | 4.01:1 | Leaf edges and ends: text stays in the flat centre |
| `--catalogue-red` on `--leaf-centre` | 2.25:1 | Red numbers never go straight on the leaf: they sit on a paper tag |
| `--ink` on `--copper-patina` | 3.12:1 | Patina gradient zone: no dark text on patina |
| `--ink` on `--stone-mid` | 4.60:1 | Noisy stone frame: text sits on the flat face |

0 failing pair(s).

## Measured realism (`js/realism/*`, Prompt 4 T9)

Everything in this section is a number from `design/realism/`: a spec card measured from reference
photos, a scoreboard row, or an estimate that is marked as one. Nothing here is called "real"
without its number, and every number is a distance to photographs of uncalibrated colour — not to
the objects. Sources: `design/realism/specs/<material>.json` (spec cards), `design/realism/eval/scoreboard.md`
(scores, generated 2026-09-26), `design/realism/DECISIONS.md` (every choice the owner did not make).

### The stack

Plain WebGL2, no library: one fragment shader per surface (colour ramp through the measured CIELAB
percentiles, fbm noise in a rotated and stretched grain frame, height → normals by finite
difference, carved or painted letters from a text mask, one directional light plus an optional oil
lamp). three.js was rejected because it ships only ES-module builds since r160 and the site must run
by double-click from `file://` with no bundler (DECISIONS, T3a, 2026-09-26 10:01:21).

| File | What it is | Size |
|---|---|---|
| `js/realism/stage.js` | **Generated** from `design/realism/rig/stage.js` by `node design/realism/tools/build_site_stage.mjs`. Same shader and same defaults as the rig the scoreboard measures; the only edit is that the renderer takes a canvas instead of owning `#c`, so one page can hold several materials. `--check` fails if the shipped file has drifted from the rig. | 19 KB |
| `js/realism/materials.params.js` | **Generated** from `design/realism/t6/*_light.params.js` by `node design/realism/tools/build_site_materials.mjs`. The measured numbers, with each value's source in its own `_source` block. | 7 KB |
| `js/realism/materials3d.js` | Site glue: attaches a stage to every `[data-real-material]` block, wires the light, age and lamp controls, handles the fallbacks. | 6 KB |

All three are classic scripts (no module, no network, no build step at page load), so `file://`
works. A block renders nothing until it is scrolled into view (`IntersectionObserver`, 200 px
margin), and makes **no WebGL call at all** when `data-quality="low"` is set or the browser has no
WebGL2 — the plain fallback panel stays instead. Under `prefers-reduced-motion` the oil lamp is lit
but steady: the flicker loop never starts.

### Gate per object

T3b, the automatic gate task, is blocked, so `eval/gates/*.json` does not exist. The gate used here
is the scoreboard: 3D ships where the engine beats the flat CSS version on the scored rows, the 2D
version stays where it does not (DECISIONS, T9). Both are on the style guide: the Prompt 2 versions
under **Real materials**, the engine versions under **Real materials · measured**.

| Object | Visual rows (engine) | Visual rows (2D baseline) | Colour dE2000 median / p95 (engine) | Light-angle test | Shipped |
|---|---|---|---|---|---|
| Copper plate | **7 / 7** | 1 / 7 | **2.50 / 6.19** (2D: 23.74 / 26.5) | PASS 6.35 dL* at 35°, 9.63 dL* at 15° (2D: no light input) | 3D |
| Stone | **4 / 6** | 2 / 6 | 6.46 / 13.2 — **both FAIL** (2D: 6.28 / 8.7) | PASS 20.39 dL* at 35°, 24.39 dL* at 15° (2D: no light input) | 3D |
| Palm leaf | no engine material (T3b, T4-palm_leaf blocked) | 6 / 7 | 3.89 / 7.29 (2D) | FAIL (2D: no light input) | 2D |
| Terracotta sherd | no engine material (T4-pottery blocked) | 7 / 8 | 5.80 / 7.92 (2D) | FAIL | 2D |
| Coin | no engine material (T4-coins blocked) | 1 / 8 | 28.5 / 37.0 (2D) | FAIL | 2D |
| Seal | no engine material (T4-seals blocked) | 0 / 8 | 23.78 / 31.89 (2D) | FAIL | 2D |
| Ring | no engine material (T4-rings blocked) | 0 / 1 | not scorable (2D) | FAIL | 2D |

Targets (`design/realism/eval/targets.json`, owner's, unchanged): colour dE2000 median < 6.0, p95 <
12.0, spectral slope difference < 0.3, anisotropy difference < 0.15, local contrast ratio 0.8–1.25,
mean L\*a\*b\* inside the 5–95% range of the photo means, light-angle ≥ 4 dL\* with ≥ 2 dL\* edge
excess.

**The stone fails two rows and is shipped anyway**: it passes 4 of 6 where the flat version passes
2 of 6, and it is the only version that answers a light angle at all. Real reference photos pass
these same targets only sometimes (leave-one-out, `eval/self_test/real_photo_pass_rates.json`):
spectral slope 33% for stone and 23% for copper, colour dE2000 median 27% and 23%. A failing row is
not proof of an unreal material, and a passing row is not proof of a real one.

### Copper plate — every value used

| Parameter | Value | Where it comes from |
|---|---|---|
| `color.p5 / p50 / p95` (CIELAB) | [18.9, −5.2, −6.7] / [37.4, 5.0, 9.4] / [60.2, 38.24, 29.9] | `specs/copper_plate.json`, 22 reference photos |
| `color.contrast` | 1.1 | picked by `t6/probe_copper.mjs`, scored with `eval/visual.py` |
| `noise.scale` | 25 | same probe |
| `noise.gain` | 1.0 | measured spectral slope −1.9524 → H = −0.024 → gain 1.017, clamped to 1.0 |
| `noise.octaves` / `lacunarity` | 6 / 2.0 | rig defaults |
| `grain.angle_deg` | 0.4087 | `specs/copper_plate.json` grain direction relative to the long axis |
| `grain.stretch` | 1.23 | estimate: 1 + measured anisotropy 0.2305 |
| `surface.roughness / metal / spec` | 0.45 / 0.2 / 0.35 | **estimate** (patinated, not polished): metal 0.2 passes all 7 rows, 0.5 fails 1, 0.8 fails 4 |
| `surface.spec_tint` | 1.00 : 0.78 : 0.58 | copper F0 0.95 / 0.74 / 0.55, normalised |
| `height.letter_depth` | 0.003 object units ≈ **1.2 mm** on a 400 mm plate | **estimate, unverified**: no source gives the groove depth of a Chola charter. Set by the lighting test (0.5 mm gave 2.625 dL\*, under the 4 dL\* target); stays under half the 2.5 mm spec plate thickness |
| `height.noise_amp` | 0.0003 | estimate |
| `letters.mode / size` | carved / 0.16 | engraved charter text |
| Sample text | தமிழ் செப்பேடு | **unverified**: no Tamil speaker has checked it |

### Stone — every value used

| Parameter | Value | Where it comes from |
|---|---|---|
| `color.p5 / p50 / p95` (CIELAB) | [31.7, −2.7, −4.0] / [64.2, 1.8, 10.4] / [93.24, 16.6, 33.8] | `specs/stone.json`, 62 reference photos |
| `color.contrast` | 0.90 | picked by `t6/probe.mjs` (48 renders scored with `eval/visual.py`) |
| `noise.scale` | 16 | same probe. The correlation-length derivation (12.75 px → scale 80) measured back as a 2 px correlation length because the fbm octaves dominate, so the scale is set by measurement instead |
| `noise.gain` | 0.824 | measured spectral slope −2.5571 → H = 0.279 → gain 2^−H |
| `grain.angle_deg` | 0.5572 | `specs/stone.json` |
| `grain.stretch` | 1.26 | estimate: 1 + measured anisotropy 0.2577 |
| `speckle` | off | no speckle number in the spec card; the shader speckle was inert at every density the probe tried |
| `surface.roughness / spec` | 0.85 / 0.08 | **estimate**: dry granite |
| `height.letter_depth` | 0.008 object units ≈ **5 mm** on a 600 mm slab | **estimate** |
| `height.noise_amp` | 0.0003 | estimate |
| Sample text | தமிழ் கல்வெட்டு | **unverified** |

### Age and wear (T7)

One control, `wear.age` 0..1, drives four things at once: crack lines (frequency 22 per object unit,
half-width 0.05 at age 1, groove 0.0015 units deep), stain blotches (frequency 2.3), edge loss (2%
of the long side at age 1) and an overall 18% L\* darkening at age 1. Age 0 is bit-for-bit the
unworn material (tested by pixel hash). Every wear amount is a **T7 estimate**: no source measures
crack density or stain area on these objects. The style-guide sliders start at 0; the engine default
is 0.25, the largest value that keeps both T6 materials' visual scores within 10% of their
unworn numbers (0.35 breaks copper's colour row, 0.5 breaks both materials' spectral slope).

### Page cost and compatibility

Measured by `node design/realism/eval/run_all.mjs` (headless Chrome for Testing 151, 4× CPU
throttle, 1200×800 and 360×740 at DPR 2 — not a real phone).

| Row (styleguide.html, with the measured blocks on it) | Value | Target | Result |
|---|---|---|---|
| `p95_frame_ms_4x_throttle` | 18.7 ms | ≤ 20 ms | PASS |
| `console_errors_desktop` / `_mobile` | 0 / 0 | 0 | PASS |
| `file_url_requests_ok_desktop` / `_mobile` | 0 / 0 failed requests over `file://` | 0 | PASS |
| `fallback_quality_low` | renders | no errors, page not blank | PASS |
| `fallback_webgl_off` | renders | no errors, page not blank | PASS |
| `reduced_motion_static` | 0 changed pixels over 1500 ms | ≤ 0.001 | PASS |

The scoreboard still calls this page the "site 2D baseline": the 2D samples it measures are the
Prompt 2 ones, which were not touched, but the perf and compat rows above are the whole page,
measured blocks included.

T9's own checks — `node design/realism/t9/checks.mjs` — are **25 of 25 PASS**: both generated files
still match their sources; both blocks start a stage and paint every pixel; the parameters the page
renders with are the T6 material cards value for value, and the surface the page draws matches the
rig's render of the same parameters to within 0.06 / 255 (copper) and 0.00 / 255 (stone) per sRGB
channel; the light angle and the age slider both change the pixels; `?quality=low` and a browser
with WebGL switched off leave both fallbacks visible with no console error; under reduced motion
the lamp lights but the canvas does not change over 1.5 s.

The old site checks stay green: `node design/realism/tools/run_site_check.mjs website_live/_checks/checks_p2.mjs styleguide.html`
— keyboard 104 of 104 focus stops at 1200 px and 93 of 93 at 360 px, all in DOM order, no missing
focus ring — and `python3 _checks/contrast.py` (`tokens.css` untouched, 0 failing pairs).

**One side effect, reported, not hidden:** the page is now longer, so the 2D sherd sample sits at a
different scroll position and the baseline capture of it lands on different device pixels. Its
`spectral_slope_diff` moved 0.28 → 0.397 against a target of 0.3, and the scoreboard's pottery
column went 7/8 → 6/8. No pottery CSS was changed; only where the screenshot falls did. Every other
2D baseline number moved by less than 0.03.


### What is not rebuilt

- **Physics.** The engine's XPBD scenes (bundle, plate set on a ring, coin, ring, seal, sherd,
  static stone: 128 of 128 physics rows pass) run on their own demo pages under `design/realism/t5/`;
  the site's motion is still the Prompt 2 GSAP layer. Wiring the solver into the style guide is not
  part of T9.
- **Sound.** `design/realism/t8/` has a muted-by-default synthesized sound layer. It is not on the
  site.
- **Palm leaf, sherd, coin, ring, seal in 3D.** The tuner tasks for those materials (T4-*) and the
  palm-leaf gate (T3b) are blocked, so no measured material exists for them; the flat CSS versions
  stay, with their baseline numbers in the table above.

## Known limits

- **The measured materials are measured against photographs**, not against the objects: the reference photos have unknown white balance, and two of the six stone rows fail (see § Measured realism). The sample Tamil text on those blocks is unverified.
- **Only copper plate and stone have an engine material.** Palm leaf, sherd, coin, ring and seal are still the flat CSS versions, because their tuner tasks are blocked.
- **Dark mode is ground-only (T10).** The page ground and the roles flip; the measured material colours do not, so a bright leaf, slab or coin still sits on a dark page by design. Whether that reads well at night is a judgement nobody has made yet — it has been measured for contrast, not reviewed by eye on a real screen.
- **The search is a demo.** `js/components.js` filters the entries already in the page; there is no index file, no Tamil folding (the identity page has its own), and the four sample records are placeholders.
- **The map is a placeholder.** The shape is invented, the pins are at made-up positions, and there are no coordinates. The box says so in the picture.
- **Every date, place name and Tamil string in the T10 style-guide section is unverified** sample text, including the timeline dates, until a Tamil speaker and a source check them.
- **Tested in Google Chrome only** (desktop, headless, via Playwright), not in Safari or Firefox. The CSS needs 2023-or-later browsers (container queries, `color-mix()`, unprefixed `mask-composite`); `-webkit-mask-composite` is included for older Safari.
- Noto Sans Tamil (the named fallback) is not self-hosted. It is used only if it is installed on the device.
- Semmozhi Vatteluttu is a draft font (see `fonts/README.txt`).
- Style-guide captions, dates and names are layout samples, not catalogue records.

## Contrast table — page components, light and dark (T10)

<!-- BEGIN T10 CONTRAST -->

### Page components — light mode (day)

| Foreground | Background | Ratio | Needs | Result | Component | Used for |
|---|---|---|---|---|---|---|
| `--text` #241B14 | `--bg` #F6EEDD | 14.65:1 | body ≥ 4.5 | PASS | `.topnav` | Brand and navigation links |
| `--text` #241B14 | `--bg-alt` #EFE3CB | 13.30:1 | body ≥ 4.5 | PASS | `.topnav` | Navigation link, hover ground |
| `--text` #241B14 | `--rule` #D8C7A6 | 10.18:1 | body ≥ 4.5 | PASS | `.topnav` | Navigation link, pressed ground |
| `--text-muted` #5E4E40 | `--bg` #F6EEDD | 6.89:1 | body ≥ 4.5 | PASS | `.topnav` | Disabled navigation link (also struck through) |
| `--accent` #A84E34 | `--bg` #F6EEDD | 4.78:1 | non-text ≥ 3 | PASS | `.topnav` | 4px top rule and the current-page underline |
| `--text` #241B14 | `--bg-alt` #EFE3CB | 13.30:1 | body ≥ 4.5 | PASS | `.theme-switch` | Unselected segment label (day / system / night) |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | `.theme-switch` | Selected segment label (also bold + check mark) |
| `--text` #241B14 | `--rule` #D8C7A6 | 10.18:1 | body ≥ 4.5 | PASS | `.theme-switch` | Segment label on hover |
| `--edge` #87734F | `--bg-alt` #EFE3CB | 3.59:1 | non-text ≥ 3 | PASS | `.theme-switch` | Switch outline |
| `--text` #241B14 | `--bg` #F6EEDD | 14.65:1 | body ≥ 4.5 | PASS | `.subnav` | Section sub-navigation link |
| `--text` #241B14 | `--bg-alt` #EFE3CB | 13.30:1 | body ≥ 4.5 | PASS | `.subnav` | Sub-navigation link, hover ground |
| `--accent` #A84E34 | `--bg` #F6EEDD | 4.78:1 | non-text ≥ 3 | PASS | `.subnav` | Current sub-navigation item underline |
| `--paper` #F6EEDD | `--leaf-bundle` #4A3426 | 10.05:1 | body ≥ 4.5 | PASS | `.site-foot` | Footer text on the bundle colour |
| `--thread` #E9A23B | `--leaf-bundle` #4A3426 | 5.36:1 | non-text ≥ 3 | PASS | `.site-foot` | Thread-coloured footer rule |
| `--paper` #F6EEDD | `--leaf-bundle` #4A3426 | 10.05:1 | non-text ≥ 3 | PASS | `.site-foot` | Focus ring inside the footer (on-dark ring) |
| `--text` #241B14 | `--bg-alt` #EFE3CB | 13.30:1 | body ≥ 4.5 | PASS | `.page-foot` | Footer text |
| `--text-muted` #5E4E40 | `--bg-alt` #EFE3CB | 6.26:1 | body ≥ 4.5 | PASS | `.page-foot` | Footer meta line |
| `--accent-text` #8F3F29 | `--bg-alt` #EFE3CB | 5.68:1 | body ≥ 4.5 | PASS | `.page-foot` | Footer link |
| `--accent` #A84E34 | `--bg-alt` #EFE3CB | 4.34:1 | non-text ≥ 3 | PASS | `.page-foot` | Footer top rule |
| `--accent-text` #8F3F29 | `--bg` #F6EEDD | 6.25:1 | body ≥ 4.5 | PASS | `.timeline` | Event date |
| `--text` #241B14 | `--bg` #F6EEDD | 14.65:1 | body ≥ 4.5 | PASS | `.timeline` | Event title and body |
| `--accent` #A84E34 | `--bg` #F6EEDD | 4.78:1 | non-text ≥ 3 | PASS | `.timeline` | Event dot (filled = dated, hollow = uncertain) |
| `--text-muted` #5E4E40 | `--bg` #F6EEDD | 6.89:1 | body ≥ 4.5 | PASS | `.timeline` | “unverified” flag label |
| `--edge` #87734F | `--bg` #F6EEDD | 3.96:1 | non-text ≥ 3 | PASS | `.timeline` | Dashed border of the “unverified” flag |
| `--text` #241B14 | `--surface` #FFFBF2 | 16.37:1 | body ≥ 4.5 | PASS | `.search` | Typed query |
| `--text-muted` #5E4E40 | `--surface` #FFFBF2 | 7.70:1 | body ≥ 4.5 | PASS | `.search` | Placeholder and result location line |
| `--edge` #87734F | `--surface` #FFFBF2 | 4.42:1 | non-text ≥ 3 | PASS | `.search` | 2px field border |
| `--accent` #A84E34 | `--surface` #FFFBF2 | 5.34:1 | non-text ≥ 3 | PASS | `.search` | Field border when focused, result left bar |
| `--text` #241B14 | `--bg-alt` #EFE3CB | 13.30:1 | body ≥ 4.5 | PASS | `.search` | Clear button |
| `--text` #241B14 | `--rule` #D8C7A6 | 10.18:1 | body ≥ 4.5 | PASS | `.search` | Clear button on hover |
| `--accent-text` #8F3F29 | `--surface` #FFFBF2 | 6.99:1 | body ≥ 4.5 | PASS | `.search` | Result title link |
| `--text` #241B14 | `--surface` #FFFBF2 | 16.37:1 | body ≥ 4.5 | PASS | `.search` | Result snippet |
| `--ink` #241B14 | `--coin-gold-light` #E4C97E | 10.43:1 | body ≥ 4.5 | PASS | `.search` | Matched words inside a snippet (<mark>) |
| `--text` #241B14 | `--bg-alt` #EFE3CB | 13.30:1 | body ≥ 4.5 | PASS | `.search` | “No results” panel |
| `--edge` #87734F | `--bg-alt` #EFE3CB | 3.59:1 | non-text ≥ 3 | PASS | `.search` | Dashed border of the “No results” panel |
| `--text-muted` #5E4E40 | `--bg` #F6EEDD | 6.89:1 | body ≥ 4.5 | PASS | `.search` | Result-count line |
| `--text` #241B14 | `--bg` #F6EEDD | 14.65:1 | body ≥ 4.5 | PASS | `.map-ph` | Place labels drawn on the land shape |
| `--edge` #87734F | `--bg-alt` #EFE3CB | 3.59:1 | non-text ≥ 3 | PASS | `.map-ph` | Box border |
| `--edge` #87734F | `--bg` #F6EEDD | 3.96:1 | non-text ≥ 3 | PASS | `.map-ph` | Outline of the land shape |
| `--accent` #A84E34 | `--bg` #F6EEDD | 4.78:1 | non-text ≥ 3 | PASS | `.map-ph` | Pins (each has a text label beside it) |
| `--text` #241B14 | `--surface` #FFFBF2 | 16.37:1 | body ≥ 4.5 | PASS | `.map-ph` | “Not a real map” note |
| `--text-muted` #5E4E40 | `--bg` #F6EEDD | 6.89:1 | body ≥ 4.5 | PASS | `.map-ph` | Caption under the box |
| `--text` #241B14 | `--surface` #FFFBF2 | 16.37:1 | body ≥ 4.5 | PASS | `.card` | Card title and body |
| `--text-muted` #5E4E40 | `--surface` #FFFBF2 | 7.70:1 | body ≥ 4.5 | PASS | `.card` | Card foot / meta line |
| `--accent-text` #8F3F29 | `--surface` #FFFBF2 | 6.99:1 | body ≥ 4.5 | PASS | `.card` | Card eyebrow |
| `--edge` #87734F | `--bg` #F6EEDD | 3.96:1 | non-text ≥ 3 | PASS | `.card--quiet` | Border of a card with no fill |
| `--text-muted` #5E4E40 | `--bg-alt` #EFE3CB | 6.26:1 | body ≥ 4.5 | PASS | `.card[aria-disabled]` | Disabled card text |
| `--edge` #87734F | `--bg-alt` #EFE3CB | 3.59:1 | non-text ≥ 3 | PASS | `.card[aria-disabled]` | Dashed border of a disabled card |
| `--focus-ring` #241B14 | `--surface` #FFFBF2 | 16.37:1 | non-text ≥ 3 | PASS | `.card` | Keyboard focus ring on a card |
| `--focus-ring` #241B14 | `--bg` #F6EEDD | 14.65:1 | non-text ≥ 3 | PASS | `.all` | Keyboard focus ring on the page ground |
| `--focus-ring` #241B14 | `--bg-alt` #EFE3CB | 13.30:1 | non-text ≥ 3 | PASS | `.all` | Keyboard focus ring on an alternate band |

### Page components — dark mode (night)

Same components, same markup. Only the ground and the roles are rebound; the material
colours (`--paper`, `--ink`, `--leaf-bundle`, `--coin-gold-light` …) are measured from the
objects and do not change, so the footer and the `<mark>` rows repeat their day numbers.

| Foreground | Background | Ratio | Needs | Result | Component | Used for |
|---|---|---|---|---|---|---|
| `--text` #F1E7D6 | `--bg` #16120E | 15.21:1 | body ≥ 4.5 | PASS | `.topnav` | Brand and navigation links |
| `--text` #F1E7D6 | `--bg-alt` #201A14 | 14.06:1 | body ≥ 4.5 | PASS | `.topnav` | Navigation link, hover ground |
| `--text` #F1E7D6 | `--rule` #4A3E30 | 8.47:1 | body ≥ 4.5 | PASS | `.topnav` | Navigation link, pressed ground |
| `--text-muted` #C3B49E | `--bg` #16120E | 9.18:1 | body ≥ 4.5 | PASS | `.topnav` | Disabled navigation link (also struck through) |
| `--accent` #E08A62 | `--bg` #16120E | 7.08:1 | non-text ≥ 3 | PASS | `.topnav` | 4px top rule and the current-page underline |
| `--text` #F1E7D6 | `--bg-alt` #201A14 | 14.06:1 | body ≥ 4.5 | PASS | `.theme-switch` | Unselected segment label (day / system / night) |
| `--button-text` #241B14 | `--button-bg` #E08A62 | 6.43:1 | body ≥ 4.5 | PASS | `.theme-switch` | Selected segment label (also bold + check mark) |
| `--text` #F1E7D6 | `--rule` #4A3E30 | 8.47:1 | body ≥ 4.5 | PASS | `.theme-switch` | Segment label on hover |
| `--edge` #8F7A5C | `--bg-alt` #201A14 | 4.19:1 | non-text ≥ 3 | PASS | `.theme-switch` | Switch outline |
| `--text` #F1E7D6 | `--bg` #16120E | 15.21:1 | body ≥ 4.5 | PASS | `.subnav` | Section sub-navigation link |
| `--text` #F1E7D6 | `--bg-alt` #201A14 | 14.06:1 | body ≥ 4.5 | PASS | `.subnav` | Sub-navigation link, hover ground |
| `--accent` #E08A62 | `--bg` #16120E | 7.08:1 | non-text ≥ 3 | PASS | `.subnav` | Current sub-navigation item underline |
| `--paper` #F6EEDD | `--leaf-bundle` #4A3426 | 10.05:1 | body ≥ 4.5 | PASS | `.site-foot` | Footer text on the bundle colour |
| `--thread` #E9A23B | `--leaf-bundle` #4A3426 | 5.36:1 | non-text ≥ 3 | PASS | `.site-foot` | Thread-coloured footer rule |
| `--paper` #F6EEDD | `--leaf-bundle` #4A3426 | 10.05:1 | non-text ≥ 3 | PASS | `.site-foot` | Focus ring inside the footer (on-dark ring) |
| `--text` #F1E7D6 | `--bg-alt` #201A14 | 14.06:1 | body ≥ 4.5 | PASS | `.page-foot` | Footer text |
| `--text-muted` #C3B49E | `--bg-alt` #201A14 | 8.48:1 | body ≥ 4.5 | PASS | `.page-foot` | Footer meta line |
| `--accent-text` #E08A62 | `--bg-alt` #201A14 | 6.55:1 | body ≥ 4.5 | PASS | `.page-foot` | Footer link |
| `--accent` #E08A62 | `--bg-alt` #201A14 | 6.55:1 | non-text ≥ 3 | PASS | `.page-foot` | Footer top rule |
| `--accent-text` #E08A62 | `--bg` #16120E | 7.08:1 | body ≥ 4.5 | PASS | `.timeline` | Event date |
| `--text` #F1E7D6 | `--bg` #16120E | 15.21:1 | body ≥ 4.5 | PASS | `.timeline` | Event title and body |
| `--accent` #E08A62 | `--bg` #16120E | 7.08:1 | non-text ≥ 3 | PASS | `.timeline` | Event dot (filled = dated, hollow = uncertain) |
| `--text-muted` #C3B49E | `--bg` #16120E | 9.18:1 | body ≥ 4.5 | PASS | `.timeline` | “unverified” flag label |
| `--edge` #8F7A5C | `--bg` #16120E | 4.53:1 | non-text ≥ 3 | PASS | `.timeline` | Dashed border of the “unverified” flag |
| `--text` #F1E7D6 | `--surface` #2A2219 | 12.78:1 | body ≥ 4.5 | PASS | `.search` | Typed query |
| `--text-muted` #C3B49E | `--surface` #2A2219 | 7.71:1 | body ≥ 4.5 | PASS | `.search` | Placeholder and result location line |
| `--edge` #8F7A5C | `--surface` #2A2219 | 3.81:1 | non-text ≥ 3 | PASS | `.search` | 2px field border |
| `--accent` #E08A62 | `--surface` #2A2219 | 5.95:1 | non-text ≥ 3 | PASS | `.search` | Field border when focused, result left bar |
| `--text` #F1E7D6 | `--bg-alt` #201A14 | 14.06:1 | body ≥ 4.5 | PASS | `.search` | Clear button |
| `--text` #F1E7D6 | `--rule` #4A3E30 | 8.47:1 | body ≥ 4.5 | PASS | `.search` | Clear button on hover |
| `--accent-text` #E08A62 | `--surface` #2A2219 | 5.95:1 | body ≥ 4.5 | PASS | `.search` | Result title link |
| `--text` #F1E7D6 | `--surface` #2A2219 | 12.78:1 | body ≥ 4.5 | PASS | `.search` | Result snippet |
| `--ink` #241B14 | `--coin-gold-light` #E4C97E | 10.43:1 | body ≥ 4.5 | PASS | `.search` | Matched words inside a snippet (<mark>) |
| `--text` #F1E7D6 | `--bg-alt` #201A14 | 14.06:1 | body ≥ 4.5 | PASS | `.search` | “No results” panel |
| `--edge` #8F7A5C | `--bg-alt` #201A14 | 4.19:1 | non-text ≥ 3 | PASS | `.search` | Dashed border of the “No results” panel |
| `--text-muted` #C3B49E | `--bg` #16120E | 9.18:1 | body ≥ 4.5 | PASS | `.search` | Result-count line |
| `--text` #F1E7D6 | `--bg` #16120E | 15.21:1 | body ≥ 4.5 | PASS | `.map-ph` | Place labels drawn on the land shape |
| `--edge` #8F7A5C | `--bg-alt` #201A14 | 4.19:1 | non-text ≥ 3 | PASS | `.map-ph` | Box border |
| `--edge` #8F7A5C | `--bg` #16120E | 4.53:1 | non-text ≥ 3 | PASS | `.map-ph` | Outline of the land shape |
| `--accent` #E08A62 | `--bg` #16120E | 7.08:1 | non-text ≥ 3 | PASS | `.map-ph` | Pins (each has a text label beside it) |
| `--text` #F1E7D6 | `--surface` #2A2219 | 12.78:1 | body ≥ 4.5 | PASS | `.map-ph` | “Not a real map” note |
| `--text-muted` #C3B49E | `--bg` #16120E | 9.18:1 | body ≥ 4.5 | PASS | `.map-ph` | Caption under the box |
| `--text` #F1E7D6 | `--surface` #2A2219 | 12.78:1 | body ≥ 4.5 | PASS | `.card` | Card title and body |
| `--text-muted` #C3B49E | `--surface` #2A2219 | 7.71:1 | body ≥ 4.5 | PASS | `.card` | Card foot / meta line |
| `--accent-text` #E08A62 | `--surface` #2A2219 | 5.95:1 | body ≥ 4.5 | PASS | `.card` | Card eyebrow |
| `--edge` #8F7A5C | `--bg` #16120E | 4.53:1 | non-text ≥ 3 | PASS | `.card--quiet` | Border of a card with no fill |
| `--text-muted` #C3B49E | `--bg-alt` #201A14 | 8.48:1 | body ≥ 4.5 | PASS | `.card[aria-disabled]` | Disabled card text |
| `--edge` #8F7A5C | `--bg-alt` #201A14 | 4.19:1 | non-text ≥ 3 | PASS | `.card[aria-disabled]` | Dashed border of a disabled card |
| `--focus-ring` #F1E7D6 | `--surface` #2A2219 | 12.78:1 | non-text ≥ 3 | PASS | `.card` | Keyboard focus ring on a card |
| `--focus-ring` #F1E7D6 | `--bg` #16120E | 15.21:1 | non-text ≥ 3 | PASS | `.all` | Keyboard focus ring on the page ground |
| `--focus-ring` #F1E7D6 | `--bg-alt` #201A14 | 14.06:1 | non-text ≥ 3 | PASS | `.all` | Keyboard focus ring on an alternate band |

### `.surface-leaf` components — light mode

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #241B14 | `--surface` #FFFBF2 | 16.37:1 | body ≥ 4.5 | PASS | Card / search / map text on this section's card ground |
| `--text-muted` #5E4E40 | `--surface` #FFFBF2 | 7.70:1 | body ≥ 4.5 | PASS | Card foot, placeholder, caption |
| `--accent-text` #7E4A1C | `--surface` #FFFBF2 | 7.05:1 | body ≥ 4.5 | PASS | Card eyebrow, timeline date, result link |
| `--accent` #A86A2C | `--surface` #FFFBF2 | 4.27:1 | non-text ≥ 3 | PASS | Timeline dot, result bar, focused field border |
| `--accent` #A86A2C | `--bg` #F7EAD3 | 3.71:1 | non-text ≥ 3 | PASS | Nav rule, timeline dot on the section ground |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Selected theme segment / primary button |
| `--edge` #87734F | `--surface` #FFFBF2 | 4.42:1 | non-text ≥ 3 | PASS | Field and card borders |
| `--focus-ring` #241B14 | `--surface` #FFFBF2 | 16.37:1 | non-text ≥ 3 | PASS | Keyboard focus ring |

### `.surface-origins` components — light mode

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #241B14 | `--surface` #FFFBF2 | 16.37:1 | body ≥ 4.5 | PASS | Card / search / map text on this section's card ground |
| `--text-muted` #524E47 | `--surface` #FFFBF2 | 8.01:1 | body ≥ 4.5 | PASS | Card foot, placeholder, caption |
| `--accent-text` #8F3F29 | `--surface` #FFFBF2 | 6.99:1 | body ≥ 4.5 | PASS | Card eyebrow, timeline date, result link |
| `--accent` #A84E34 | `--surface` #FFFBF2 | 5.34:1 | non-text ≥ 3 | PASS | Timeline dot, result bar, focused field border |
| `--accent` #A84E34 | `--bg` #EEEBE5 | 4.64:1 | non-text ≥ 3 | PASS | Nav rule, timeline dot on the section ground |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Selected theme segment / primary button |
| `--edge` #87734F | `--surface` #FFFBF2 | 4.42:1 | non-text ≥ 3 | PASS | Field and card borders |
| `--focus-ring` #241B14 | `--surface` #FFFBF2 | 16.37:1 | non-text ≥ 3 | PASS | Keyboard focus ring |

### `.surface-kings` components — light mode

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #241B14 | `--surface` #FFFBF2 | 16.37:1 | body ≥ 4.5 | PASS | Card / search / map text on this section's card ground |
| `--text-muted` #5A4536 | `--surface` #FFFBF2 | 8.69:1 | body ≥ 4.5 | PASS | Card foot, placeholder, caption |
| `--accent-text` #76492F | `--surface` #FFFBF2 | 7.36:1 | body ≥ 4.5 | PASS | Card eyebrow, timeline date, result link |
| `--accent` #8C5A3C | `--surface` #FFFBF2 | 5.58:1 | non-text ≥ 3 | PASS | Timeline dot, result bar, focused field border |
| `--accent` #8C5A3C | `--bg` #F5E9DE | 4.83:1 | non-text ≥ 3 | PASS | Nav rule, timeline dot on the section ground |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Selected theme segment / primary button |
| `--edge` #87734F | `--surface` #FFFBF2 | 4.42:1 | non-text ≥ 3 | PASS | Field and card borders |
| `--focus-ring` #241B14 | `--surface` #FFFBF2 | 16.37:1 | non-text ≥ 3 | PASS | Keyboard focus ring |

### `.surface-chola` components — light mode

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #241B14 | `--surface` #FFFBF2 | 16.37:1 | body ≥ 4.5 | PASS | Card / search / map text on this section's card ground |
| `--text-muted` #544A3F | `--surface` #FFFBF2 | 8.37:1 | body ≥ 4.5 | PASS | Card foot, placeholder, caption |
| `--accent-text` #5A4D3E | `--surface` #FFFBF2 | 7.92:1 | body ≥ 4.5 | PASS | Card eyebrow, timeline date, result link |
| `--accent` #5A4D3E | `--surface` #FFFBF2 | 7.92:1 | non-text ≥ 3 | PASS | Timeline dot, result bar, focused field border |
| `--accent` #5A4D3E | `--bg` #EFE9DF | 6.78:1 | non-text ≥ 3 | PASS | Nav rule, timeline dot on the section ground |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Selected theme segment / primary button |
| `--edge` #87734F | `--surface` #FFFBF2 | 4.42:1 | non-text ≥ 3 | PASS | Field and card borders |
| `--focus-ring` #241B14 | `--surface` #FFFBF2 | 16.37:1 | non-text ≥ 3 | PASS | Keyboard focus ring |

### `.surface-trade` components — light mode

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #241B14 | `--surface` #FFFBF2 | 16.37:1 | body ≥ 4.5 | PASS | Card / search / map text on this section's card ground |
| `--text-muted` #584A32 | `--surface` #FFFBF2 | 8.33:1 | body ≥ 4.5 | PASS | Card foot, placeholder, caption |
| `--accent-text` #6E520E | `--surface` #FFFBF2 | 7.07:1 | body ≥ 4.5 | PASS | Card eyebrow, timeline date, result link |
| `--accent` #8E6D24 | `--surface` #FFFBF2 | 4.66:1 | non-text ≥ 3 | PASS | Timeline dot, result bar, focused field border |
| `--accent` #8E6D24 | `--bg` #F7EFD8 | 4.19:1 | non-text ≥ 3 | PASS | Nav rule, timeline dot on the section ground |
| `--button-text` #F6EEDD | `--button-bg` #A84E34 | 4.78:1 | body ≥ 4.5 | PASS | Selected theme segment / primary button |
| `--edge` #87734F | `--surface` #FFFBF2 | 4.42:1 | non-text ≥ 3 | PASS | Field and card borders |
| `--focus-ring` #241B14 | `--surface` #FFFBF2 | 16.37:1 | non-text ≥ 3 | PASS | Keyboard focus ring |

### `.surface-leaf` components — dark mode

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #F1E7D6 | `--surface` #302415 | 12.34:1 | body ≥ 4.5 | PASS | Card / search / map text on this section's card ground |
| `--text-muted` #C3B49E | `--surface` #302415 | 7.45:1 | body ≥ 4.5 | PASS | Card foot, placeholder, caption |
| `--accent-text` #E0AE6A | `--surface` #302415 | 7.51:1 | body ≥ 4.5 | PASS | Card eyebrow, timeline date, result link |
| `--accent` #E0AE6A | `--surface` #302415 | 7.51:1 | non-text ≥ 3 | PASS | Timeline dot, result bar, focused field border |
| `--accent` #E0AE6A | `--bg` #1C150D | 8.97:1 | non-text ≥ 3 | PASS | Nav rule, timeline dot on the section ground |
| `--button-text` #241B14 | `--button-bg` #E0AE6A | 8.40:1 | body ≥ 4.5 | PASS | Selected theme segment / primary button |
| `--edge` #8F7A5C | `--surface` #302415 | 3.68:1 | non-text ≥ 3 | PASS | Field and card borders |
| `--focus-ring` #F1E7D6 | `--surface` #302415 | 12.34:1 | non-text ≥ 3 | PASS | Keyboard focus ring |

### `.surface-origins` components — dark mode

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #F1E7D6 | `--surface` #292823 | 12.05:1 | body ≥ 4.5 | PASS | Card / search / map text on this section's card ground |
| `--text-muted` #C3B49E | `--surface` #292823 | 7.27:1 | body ≥ 4.5 | PASS | Card foot, placeholder, caption |
| `--accent-text` #E08A62 | `--surface` #292823 | 5.61:1 | body ≥ 4.5 | PASS | Card eyebrow, timeline date, result link |
| `--accent` #E08A62 | `--surface` #292823 | 5.61:1 | non-text ≥ 3 | PASS | Timeline dot, result bar, focused field border |
| `--accent` #E08A62 | `--bg` #171613 | 6.88:1 | non-text ≥ 3 | PASS | Nav rule, timeline dot on the section ground |
| `--button-text` #241B14 | `--button-bg` #E08A62 | 6.43:1 | body ≥ 4.5 | PASS | Selected theme segment / primary button |
| `--edge` #8F7A5C | `--surface` #292823 | 3.59:1 | non-text ≥ 3 | PASS | Field and card borders |
| `--focus-ring` #F1E7D6 | `--surface` #292823 | 12.05:1 | non-text ≥ 3 | PASS | Keyboard focus ring |

### `.surface-kings` components — dark mode

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #F1E7D6 | `--surface` #312218 | 12.49:1 | body ≥ 4.5 | PASS | Card / search / map text on this section's card ground |
| `--text-muted` #C3B49E | `--surface` #312218 | 7.54:1 | body ≥ 4.5 | PASS | Card foot, placeholder, caption |
| `--accent-text` #D89C74 | `--surface` #312218 | 6.50:1 | body ≥ 4.5 | PASS | Card eyebrow, timeline date, result link |
| `--accent` #D89C74 | `--surface` #312218 | 6.50:1 | non-text ≥ 3 | PASS | Timeline dot, result bar, focused field border |
| `--accent` #D89C74 | `--bg` #1D140E | 7.70:1 | non-text ≥ 3 | PASS | Nav rule, timeline dot on the section ground |
| `--button-text` #241B14 | `--button-bg` #D89C74 | 7.19:1 | body ≥ 4.5 | PASS | Selected theme segment / primary button |
| `--edge` #8F7A5C | `--surface` #312218 | 3.72:1 | non-text ≥ 3 | PASS | Field and card borders |
| `--focus-ring` #F1E7D6 | `--surface` #312218 | 12.49:1 | non-text ≥ 3 | PASS | Keyboard focus ring |

### `.surface-chola` components — dark mode

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #F1E7D6 | `--surface` #2D281B | 11.98:1 | body ≥ 4.5 | PASS | Card / search / map text on this section's card ground |
| `--text-muted` #C3B49E | `--surface` #2D281B | 7.23:1 | body ≥ 4.5 | PASS | Card foot, placeholder, caption |
| `--accent-text` #CFC0A6 | `--surface` #2D281B | 8.21:1 | body ≥ 4.5 | PASS | Card eyebrow, timeline date, result link |
| `--accent` #CFC0A6 | `--surface` #2D281B | 8.21:1 | non-text ≥ 3 | PASS | Timeline dot, result bar, focused field border |
| `--accent` #CFC0A6 | `--bg` #1A170F | 10.01:1 | non-text ≥ 3 | PASS | Nav rule, timeline dot on the section ground |
| `--button-text` #241B14 | `--button-bg` #CFC0A6 | 9.46:1 | body ≥ 4.5 | PASS | Selected theme segment / primary button |
| `--edge` #8F7A5C | `--surface` #2D281B | 3.57:1 | non-text ≥ 3 | PASS | Field and card borders |
| `--focus-ring` #F1E7D6 | `--surface` #2D281B | 11.98:1 | non-text ≥ 3 | PASS | Keyboard focus ring |

### `.surface-trade` components — dark mode

| Foreground | Background | Ratio | Needs | Result | Used for |
|---|---|---|---|---|---|
| `--text` #F1E7D6 | `--surface` #2E2611 | 12.23:1 | body ≥ 4.5 | PASS | Card / search / map text on this section's card ground |
| `--text-muted` #C3B49E | `--surface` #2E2611 | 7.38:1 | body ≥ 4.5 | PASS | Card foot, placeholder, caption |
| `--accent-text` #E0BE6A | `--surface` #2E2611 | 8.38:1 | body ≥ 4.5 | PASS | Card eyebrow, timeline date, result link |
| `--accent` #E0BE6A | `--surface` #2E2611 | 8.38:1 | non-text ≥ 3 | PASS | Timeline dot, result bar, focused field border |
| `--accent` #E0BE6A | `--bg` #1B1608 | 10.08:1 | non-text ≥ 3 | PASS | Nav rule, timeline dot on the section ground |
| `--button-text` #241B14 | `--button-bg` #E0BE6A | 9.46:1 | body ≥ 4.5 | PASS | Selected theme segment / primary button |
| `--edge` #8F7A5C | `--surface` #2E2611 | 3.64:1 | non-text ≥ 3 | PASS | Field and card borders |
| `--focus-ring` #F1E7D6 | `--surface` #2E2611 | 12.23:1 | non-text ≥ 3 | PASS | Keyboard focus ring |

### Decorative pairs (no threshold)

| Pair | Light | Dark | Why it carries no meaning |
|---|---|---|---|
| `--rule` on `--bg` | 1.44:1 | 1.79:1 | Timeline spine, card hairline, sub-nav divider: the events are an ordered list <ol> with a visible date on each item, so the line adds nothing |
| `--rule` on `--bg-alt` | 1.31:1 | 1.66:1 | Map graticule: a texture behind the shape, never read |
| `--rule` on `--surface` | 1.61:1 | 1.51:1 | Card and result hairline: the card already has its own fill (--surface against --bg), so the border only softens the edge |
| `--wood` on `--leaf-bundle` | 1.74:1 | 1.74:1 | Footer divider above the small print |
| `--thread` on `--leaf-bundle` | 5.36:1 | 5.36:1 | Footer rule: decoration; the footer's structure is headings and lists |

**182 of 182 pairs pass; 0 fail.** (51 component pairs x 2 modes + 8 pairs x 5 section surfaces x 2 modes.) Generated by `design/realism/eval/contrast_components.py`.

<!-- END T10 CONTRAST -->
