> **Ported verbatim from the araCreate Design System repository, `docs/assets.md`.**
> This is araCreate's own record and has not been rewritten. Path references
> (`src/`, `docs/`, `tests/`, `make` targets) point at that repository, not at
> this project.
>
> This project copied 21 of the 32 logo files, all 17 icons, all 8 illustrations, all 5 duotint images and both brand applications. The Affinity source sheets were not copied — they are working files.

---

# ASSETS

What is in `src/assets/`, which file to reach for, and the four rules that stop
a correct asset being used incorrectly.

Everything here came from araCreate’s own brand asset library by way of the
team’s earlier design system (ACDS), 20 August 2026. Nothing was redrawn.

---

## The four rules

1. **Prefer SVG, always.** Every logo has one. The wordmark inside it is
   outlined to vector paths, so it needs no font, stays crisp at any size, and
   can be recoloured with `fill`.
2. **Monument Extended is the wordmark and nothing else.** Not headings, not
   titles, not slide titles, not statistics. See [Type](#type) below.
3. **Imagery is duotint.** Warm gold and grey, never full colour. See
   [Imagery](#imagery).
4. **Icons are isometric line drawings.** No emoji, no icon fonts, no unicode
   glyphs as icons. See [Icons](#icons).

---

## Logos

`src/assets/logos/` — 32 files.

The naming is systematic once you know the code. `t-w-b-g` reads
*type white, background graphite*; `t-w-b-y` reads *type white, background
gold*.

| Background | File | What you get |
| --- | --- | --- |
| Canvas or white | `aracreate-logo-default.svg` | graphite type, gold chip behind CREATE |
| **Gold** | `aracreate-logo-default.svg` | the same file — its gold chip vanishes into the background, leaving clean graphite type |
| **Graphite `#555` — the dark band** | `aracreate-logo-t-w-b-g.svg` | white type, graphite chip. The chip vanishes into the band, leaving clean white type |
| Any other dark fill | `aracreate-logo-t-w-b-y.svg` | white type on a gold chip. A deliberate lockup where you want the gold to register |
| Monogram — favicon, avatar, app icon | `aracreate-icon-default.svg` | |
| Monogram, on graphite / on gold | `aracreate-icon-t-w-b-g.svg` / `-t-w-b-y.svg` | |
| Wordmark alone, no chip | `aracreate-wordmark-default.svg` | |

**Read the chip, not the name.** These files are not simply light and dark
versions of each other: each carries a coloured block behind CREATE, sized for
one specific background. Put `-t-w-b-g` on anything other than graphite and its
chip shows up as a grey rectangle that looks like a rendering fault. Every variant
was checked against every background before the table above was written — the
combinations not listed are the ones that looked wrong.

`aracreate-logo-negative.svg` is for placing over imagery, where the chip
becomes a knockout. It is not the dark-background variant, despite the name.

**Group and regional lockups**

| Entity | File |
| --- | --- |
| araCreate Group | `aracreate-group-logo-default.svg`, `-t-w-b-g.svg` |
| araCreate India | `aracreate-india-logo-default.svg`, `-t-w-b-g.svg` |
| araCreate Lanka | `aracreate-lanka-logo-default.svg`, `-t-w-b-g.svg` |

**Division lockups** — one per vertical. Use the division lockup on a division
site, and the plain logo everywhere else.

| Vertical | File |
| --- | --- |
| Engineering | `aracreate-engineering-logo.svg` |
| Manufacturing | `aracreate-manufacturing-logo.svg` |
| Media | `aracreate-media-logo.svg` |

`aracreate-brand.svg` is the palette reference sheet — the file the original
audit read the brand colours out of. It is not a logo; do not place it.

PNGs exist alongside the SVGs for the handful of contexts that cannot take
vector (email signatures, some office software). Everywhere on the web, use the
SVG.

The original Affinity multi-artboard export sheets are in
`.archives/logo-source-sheets/` as `source-logo-sheet.svg`,
`source-icon-sheet.svg` and `source-variants-sheet.svg`. They are working files, not deliverables —
they contain every variant on one canvas and will render wrong if placed.

### Placing a logo

- Give it clear space. Nothing else inside a margin of one monogram-width on
  every side.
- Never stretch it. Set one dimension and let the other follow.
- Never recolour it outside the four supplied variants. There is a file for
  each background; picking the wrong one and overriding `fill` is how a
  wordmark ends up gold on gold.
- Never add a shadow, outline, gradient or rotation to it.
- Never place the default logo over imagery. Use the negative variant, or a
  solid band behind it.

---

## Type

Three families, separated strictly by role. The separation is the point — this
is what makes an araCreate page recognisable before you have read a word of it.

| Family | Job | Token |
| --- | --- | --- |
| **Monument Extended** | the logo wordmark **only** | `--ac-font-logo` |
| **Poppins** | everything else — headings, body, UI, documents | `--ac-font-text` |
| **Red Hat Mono** | specs, data, code | `--ac-font-mono` |

**Monument Extended is not in the bundle**, on purpose. It is a commercial
licence, and linking it publishes the `.otf` to every visitor. Since the logo
SVGs have the wordmark outlined already, almost nothing needs it.

If the wordmark genuinely has to be live text — a `<title>`-driven header, a
templating system that cannot place an image — opt in:

```html
<link rel="stylesheet" href="src/wordmark-font.css">
<span class="ac-wordmark">araCreate</span>
```

That file declares the two weights and exactly one selector. If you find
yourself widening that selector, the answer is Poppins.

`src/assets/fonts/` holds the two OTFs — `monument-extended-regular.otf` and
`monument-extended-ultrabold.otf`. **They are excluded by `.gitignore` and must
never be committed anywhere**, because a commit is one `git remote add` away
from being public. A fresh clone will not have them, and `wordmark-font.css`
will quietly fall back to Arial Black — which is the safe failure, and another
reason to use the SVGs. `make starter` copies them into a new project on the
same machine, which is fine. See [`licence-policy.md`](licence-policy.md).

---

## Icons

`src/assets/icons/` — 17 files.

The brand’s icon language is **isometric line illustration**: thin,
single-weight strokes, drawn in three-quarter perspective. This is a
guideline-mandated style, not a preference, and it is why an araCreate icon
never looks like a Material icon.

**Service and capability icons** — `icon-product-design.svg`,
`icon-web-development.svg`, `icon-brand-identity.svg`,
`icon-building-a-brand-identity.svg`, `icon-graphic-design.svg`,
`icon-advertising-campaigns.svg`, `icon-brand-strategy.svg`,
`icon-strategy-and-marketing.svg`, `icon-execution-and-production.svg`,
`icon-we-are-multidisciplinary.svg`.

**UI chrome** — `arrowhead-left.svg`, `arrowhead-right.svg`, `arrow-top.svg`,
`link-sharp.svg`, `locations-white.svg`. Simple stroked shapes, not isometric.

**Social** — `logo-instagram.svg`, `logo-youtube.svg`. These are other
companies' marks; do not restyle them.

**When the icon you need does not exist**, the order of preference is: draw it
in the isometric line style; or borrow one that matches the thin single-weight
line style and flag the substitution in your commit message. Never fall back to
an emoji or an icon font — the brand uses neither.

---

## Illustrations

`src/assets/illustrations/` — 8 files. Larger isometric scenes for heroes and
section art: `circuit-board.svg`, `charts-pie-and-bars.svg`,
`image-creation.svg`, `human-computer-interaction.svg`,
`customer-service.svg`, plus `decorative-element-01/02/04.svg`.

The live site animates bespoke multi-layer isometric "machine" sprites in its
hero. Those are not here and are not recreated — use these static
illustrations instead, and treat the animated machine as site-specific art
rather than part of the system.

---

## Imagery

`src/assets/imagery/` — 5 files.

Photography is **duotint**: composited over a wash so it reads warm gold and
grey rather than full colour. The wash is `#2e419e`, which is why araCreate
photography has that consistent cool-shadow, warm-highlight look. It is a fact
about how the supplied imagery was produced, **not a token** — it was
`--ac-photo-overlay` until 2026-08-23, when the token was removed because
nothing in the system composited with it. The images in `assets/imagery/` are
already treated. Never use this navy as a text or surface colour.

- `duotint-hero-about.jpg`, `duotint-hero-team.jpg`,
  `duotint-our-process.jpg` — treated and ready to place.
- `aracreate-aravinth.jpg` — the founder. A real person; use it only where a
  photograph of the founder is what is meant.
- `character-design.jpg` — illustration work, not a photograph of anyone.

**A raw full-colour photograph is not an araCreate image.** Treat it first, or
use an illustration instead. There are no photographic gradients and no
glassmorphism anywhere in this brand.

---

## Brand applications

`src/assets/brand/` — `aracreate-stamp-266x256.png` (the seal) and
`aracreate-letter-header.png` (document letterhead). Stationery, not web
furniture — a stamp on a web page reads as decoration and dilutes it.

---

See also: [`brand-facts.md`](brand-facts.md) for the facts these assets sit
beside, [`licence-policy.md`](licence-policy.md) for what may be published, and
[`foundations.html`](foundations.html) for the palette and type in the browser.
