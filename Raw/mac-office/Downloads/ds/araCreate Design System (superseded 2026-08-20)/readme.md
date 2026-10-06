# araCreate Design System

One design system for every araCreate Group surface — the group website,
araCreate Academy, araCreate Meditate, the service landing pages, and the
pitch deck. This project is a port of the group's own design system into a
Claude design system: the CSS, the assets, the brand facts and the voice rules
are the originals, not a reinterpretation.

**araCreate** — founded in 2003 by Aravinth Panch. From a solo digital studio
to a group of companies across Europe and Asia, working as one
interdisciplinary ecosystem delivering 360° services. Master tagline:
*Empowering ideas from mind to market.* Headquarters: araCreate GmbH,
Hubertusstr. 5, 12163 Berlin, Germany. ask@aracreate.group · +49 30 91437357.

## Sources

Everything here came from one attached, read-only codebase:

- `aracreate-design-system/` — the group's own design-system repository
  (proprietary, © 2026 araCreate Group; author Vishnu <vishnu@aracreate.group>).
  Ported files: `src/tokens.css`, `src/base.css`, `src/signature.css`,
  `src/components.css`, `src/sections.css`, `src/deck.css`,
  `src/signature.js`, `src/components.js`, `src/deck.js`, and the whole of
  `src/assets/`. Prose read for this readme: `README.md`, `docs/brand-facts.md`,
  `docs/guidance.md`, `docs/assets.md`.
- That repository in turn cites araCreate Brand Guidelines v1.0 (2026), the
  araCreate deck of 4 February 2026, and a measured audit of 72 pages of
  aracreate.group. Its own conventions repo is
  `https://github.com/aracreate-group/aracreate-conventions` (not read — no
  access from here).

No Figma file, no live-site access and no slide binaries were provided. The
deck layouts here are the ones `src/deck.css` defines, not a new deck.

## Products represented

| Surface | What it is | Kit |
| --- | --- | --- |
| aracreate.group | the group's marketing site — verticals, ecosystem, projects, contact | `ui_kits/group_website/` |
| araCreate Academy | course marketing and enrolment, a consumer of this system | `ui_kits/academy/` |
| A signed-in web app | dashboard, data table, settings, sign-in | `ui_kits/web_app/` |
| araCreate pitch deck | ten slide layouts, 16:9, built from the same tokens | `ui_kits/deck/` |

The first two and the deck are **recreations** — they rebuild something that
exists. The web app is a **demonstration**: no app screens, wireframes or product
copy were provided, so nothing in it is copied from anywhere. It exists because
24 application components with no screen behind them are a guess, and it earned
its keep by surfacing two contrast bugs on the first run. If araCreate has a real
internal tool, point this system at its code and the kit becomes a recreation
instead — which is the more useful thing for it to be.

araCreate Meditate is named in the sources as an intended consumer but no
screens, copy or layouts for it were provided, so no kit is built for it.

---

## CONTENT FUNDAMENTALS

araCreate sounds **professional, clear and friendly** — simple enough for
someone in the middle of a real project to read once and understand. Four
pillars at once: innovative (energetic), trustworthy (has authority),
professional (dependable), friendly (hospitable). The shorthand: **confident,
industrial, human.**

**Person.** "We" for araCreate, "you" for the client. Not "clients can" —
"you can". Not "araCreate delivers" in a sentence araCreate is writing; "we
deliver".

**Casing.** Sentence case for headlines and body — never Title Case. "Where
complex ideas need broader services", not "Where Complex Ideas Need Broader
Services". Wide-tracked UPPERCASE is reserved for buttons, eyebrows and small
labels, which is why a heading set in caps looks wrong here: caps are already
spoken for. Section eyebrows are numbered — `01 — araCreate Group`,
`02 — About`, `03 — Services` — and the number is part of a page's rhythm.

**Sentences.** Short, one idea each. Say the thing, then stop: no wind-up, no
"in today's fast-moving world". Concrete over abstract — "Rapid prototyping
through series production" beats "end-to-end manufacturing solutions". No
jargon the client has not used first. Section headlines are the one place the
brand is allowed to be aspirational, and they trail off with an ellipsis:
*Where boundless passion meets performing results …*

**Microcopy.** Buttons say what happens, in the visitor's terms: "Send
message", not "Submit". "Let's talk about your idea" is the brand's own CTA and
is worth reusing. Errors say what to do, not that something went wrong — "That
email address is missing an @", never "Invalid input", and never blame the
visitor. Empty states say what to do next, not "No data". Hints go under the
field, phrased as help rather than warning.

**Never.**

- **No emoji.** Not in copy, not as an icon, not in a heading, not as a bullet.
  This is absolute; the brand has an isometric line-illustration language
  instead.
- **No exclamation marks** in body copy. Friendly comes from directness.
- **No invented facts.** Every figure, company name and address a page may
  state is fixed. `300+ clients` is the fact — the plus sign is what makes it
  true; `300 clients` is a different and untrue claim. If a number is not on
  the list, ask; do not estimate. Where filler is genuinely needed use obvious
  filler (`Client name`, `00+`, `Lorem ipsum`): a visible placeholder gets
  replaced, a plausible invention does not.
- **No "click here".** Link text says where it goes.
- **No shouting a whole sentence** in caps.

**Fixed strings — never reworded.** The tagline *Empowering ideas from mind to
market*. The positioning line *where boundless passion meets performing
results*. The three vertical couplets, which are part of the vertical's name:
Engineering *from Prototype to Production*; Manufacturing *from Drawing to
Delivery*; Media *from Sketch to Screen*. The four values, first-person plural:
*We pursue excellence · We collaborate seamlessly · We deliver results · We
stay curious*.

**The name.** araCreate — lowercase a-r-a, capital C. Not "AraCreate", not
"Aracreate", not "ARACREATE". Full capitals ARACREATE is correct **only** as
the logo wordmark set in Monument Extended: never body copy, never a heading,
never a page title. `araCreate Group` is the parent; a single company is
`araCreate GmbH`. Legal suffixes are part of the name — `GmbH`, `gUG`,
`Pvt Ltd`, `CLG` are not interchangeable and not optional. `rlgtechdesign` is
one lowercase word.

**The figures you may state**, verbatim, plus signs included: 300+ clients ·
60+ new products · 8+ group companies · 3+ service verticals · 50+ service
capabilities · 3+ country offices · 50+ team members · 20+ years of service.
Clients range from SMBs and social enterprises to unicorn startups, global
corporations and governments. Do not compute anything from these — "20+ years
since 2003" is arithmetic on a rounded figure and will be wrong.

**The 8+ group companies.** Germany: araCreate GmbH, BatchOne GmbH,
rlgtechdesign GmbH, DreamSpace Foundation gUG. India: araCreate India Pvt Ltd,
Micro-Tech CNC Pvt Ltd. Sri Lanka: araCreate Lanka Pvt Ltd, DreamSpace
Foundation CLG.

**Not facts, absent on purpose** — revenue, headcount by office, growth
figures, named clients, named team members other than the founder,
certifications, awards, pricing, or any date other than the 2003 founding.

---

## VISUAL FOUNDATIONS

**Colour.** Two brand values, taken from araCreate's own logo files, which
contain those two and nothing else: Golden Sun `#f9bf3b` and Graphite Gray
`#555555`. Brand black `#222222`, stroke `#cecece` and canvas `#f6f6f6` are the
three named neutrals. The page is canvas off-white, not white; white is the
*raised* surface. A page's rhythm is full-width bands alternating canvas → pale
gold `#fdf3d8` → canvas, with occasional brand-black and Golden Sun bands.

**Golden Sun is never text.** Gold on white measures 1.67:1 and white on gold
measures the same — it fails in both directions at every size, including a
105px slide title. There is deliberately no token for gold text. Gold is
something you put *behind* type; brand black on gold measures 9.51:1. On a dark
surface gold is legible (8.11:1), which is why `.ac-eyebrow` and the decorative
quote mark turn gold inside an inverse surface and are brand black everywhere
else. Two colours are the whole palette; error `#b3261e` and success `#186a43`
are the only added hues, warning reuses gold and information is charcoal.
`#2e419e` is the duotint photo wash and is not a text or surface colour.

**Surfaces, not variants.** Four surfaces — page, raised, inverse, accent, plus
subtle — and each re-points every colour token inside it. Put the surface class
on the *band*, not on each thing inside it; that is why no component in this
system has a dark variant, and forgetting it produces exactly one bug (light
components with white text).

**Dark theme.** `data-ac-theme="dark"` on any element re-themes everything inside
it; `data-ac-theme="auto"` follows the operating system. It cost one file
(`css/theme-dark.css`) and touched no component, because no component contains a
raw colour. Three things needed more than a token swap, and all three are
documented in that file. **Golden Sun does not move** — lightening it for dark
mode would give the brand two different yellows, and it already reads correctly
on a dark surface. **The inverse surface flips to light**, because a dark band on
a dark page is invisible; that is the one structural change. **Error and success
needed dark-specific values** — the light pair measures 2.51:1 and 2.79:1 on
brand black, so dark uses `#f28b82` (8.42:1) and `#81c995` (8.73:1). Body copy is
`#d8d8d8` rather than white: 15.91:1 is more contrast than long-form reading
wants, and it glares.

**Density and target size.** Comfortable is the default.
`data-ac-density="compact"` on a region tightens control padding, table cells,
card bodies and stack gaps for signed-in, data-dense screens — scoped to
`pointer: fine`, so on a touch device it tightens the type rhythm but the target
floor holds. **Every interactive control is at least 44px** in both directions,
applied as padding and min-height, never by growing the type: a 12px label
inside a 44px target is fine, a 12px target is not. Checkboxes and radios stay
18px and their `<label>` carries the target, which is why the control is wrapped
in one. Links inside running text are exempt — one cannot be 44px tall without
wrecking the line.

**Type.** Poppins for everything; Red Hat Mono for specs, data and code;
Monument Extended for the logo wordmark and nothing else. The heading ladder is
locked to the live site exactly: 36 / 33 / 31 / 28 / 21 / 16px, around 3px
apart, in Light (300) — a known, deliberate deviation. Body is 16px Poppins
Light with a 2em line-height and .02em tracking. **Links are 15px** — raised
from the site's 12px on 20 August 2026, the one locked deviation that was fixed
rather than kept; navigation, footer, breadcrumb and pagination links also carry
a 44px hit area. Links inside running text still inherit the paragraph's size. A
separate display tier sits above the ladder for the two or three places per site
that are display type: 64 / 48 / 36 / 28px, and it is the one place weight
600–700 is used for size rather than emphasis. Statistics are 42px in Thin (200)
with -.02em tracking.

**Spacing.** A 10-based ladder, named by its own value: 4, 8, 10, 12, 15, 16,
20, 24, 30, 40, 60, 80, 90, 100, 120, 140. `--ac-space-20` is the most-used
value on the live site (153 times). Sections are 90px top and bottom; a tall
band is 140px. Containers: 940px is what every `.container` on aracreate.group
actually measures, 1200px is the outer bound, and body copy caps at 74ch.

**Corners: there are none.** Every radius token is `0` — cards, panels, inputs,
buttons, badges, tags, avatars, switches, step markers, dots, the spinner. A
square dotted panel beside a pill-shaped switch and a circular avatar reads as
three systems on one page. Do not add a `border-radius` to a component; if
something must be round that is a decision about the whole system and it
belongs in `css/tokens.css`. One consequence: radio buttons are square like
checkboxes, distinguished by what appears inside them.

**Borders and the signature edge.** The signature device is a 1px edge that is
**dotted on the top and right, solid on the bottom and left**, with square
corners — the panel reads as drawn by hand and still open on two sides. It is
applied **by default** to panels, cards, modals, text inputs, textarea, select,
table, blockquote and `figure img`. `.ac-edge-none` opts out; the exception is
what you declare, not the rule. Buttons are excluded on purpose — a dotted
border round a yellow fill reads as a rendering fault. Mirror variants move
which corner is "closed"; adjacent panels drop the shared edge so two 1px
borders never stack into a 2px seam. Two border colours exist because they
answer different questions: dividers may be faint `#cecece`, a control edge
must reach 3:1 and uses graphite.

**Cards.** Canvas-white box, square, the signature edge, no resting shadow. A
card at rest is flat — the edge is what separates it from the page, and a
resting shadow would say the same thing twice. On hover a card that is a link
*closes*: the dotted sides resolve to solid, it lifts 4px, takes
`--ac-shadow-lift`, and its image scales to 1.04. Six kinds exist — course,
service, article, person, stat, project — and if none fits, that is a
conversation before inventing a seventh.

**Shadows.** Three, and a component picks one. `--ac-shadow-lift` for hover,
`--ac-shadow-float` for a dropdown or toast, `--ac-shadow-overlay` for a modal.
Nothing composes its own box-shadow and nothing is ever coloured — araCreate
reads industrial and flat, not glassy.

**Transparency and blur.** No glassmorphism anywhere, no backdrop blur, no
frosted panels. Transparency appears in exactly four places: the modal scrim
(charcoal at 70%, so the page reads as dimmed rather than switched off), the
gold hover/pressed states (90% and 80% of Golden Sun), the token-level
translucent feedback surfaces (`#b3261e14` and friends, roughly 8%), and the
marquee's mask, which fades its two ends so items appear rather than being
clipped. There are no protection gradients — text over imagery uses a solid
band or the negative logo, not a scrim gradient.

**Motion.** Four durations — 200 / 400 / 500 / 800ms — and one easing,
`cubic-bezier(.165, .84, .44, 1)`: a fast start settling gently, never a
bounce, never a spring. Named devices: the animated 2px underline, which wipes
in from the left on hover rather than fading (the single most-used device on the
live site, 2,259 elements); scroll reveals that rise 20px and fade in, staggered
80ms per child; a line that draws itself by scaling from its left edge; the
40-second marquee, paused on hover so a visitor can read an item they spotted;
a sticky header that shrinks 15px once the page scrolls; a 3px gold scroll-
progress bar. Every animation stops for a visitor who has asked their operating
system for less motion, and nothing is ever hidden from someone who cannot
receive the animation.

**Hover and press.** Buttons lift: `translateY(-5px) scale(1.02)` with the
lift shadow on hover, `translateY(-2px) scale(1.01)` on press, and the gold goes
to 90% then 80% — darker, never lighter. Ghost buttons neither lift nor shadow.
Links darken from graphite to brand black and grow their underline. Cards lift
4px and close their dotted edge. Service-list rows indent 12px and their arrow
slides 6px right. Logo tiles come out of greyscale. The focus ring is one
global style — 2px solid brand black, 2px offset, white on an inverse surface —
and removing it is one of three things you may not undo (the others being
reduced-motion handling and the inverse-surface context).

**Imagery.** Photography is **duotint**: composited over `#2e419e` at low
opacity so it reads warm gold and cool grey rather than full colour — warm
highlights, cool shadows, no grain, no colour cast beyond that. A raw
full-colour photograph is not an araCreate image; treat it, or use an
illustration instead. No photographic gradients. `figure img` carries the
signature edge; a bare `<img>` used as a logo or icon does not.

**Layout.** Sticky header, 90px falling to 80px at 1280 and 75px at 1440, with
`:target` clearance to match. Edge-to-edge bands use a measured viewport width
(`--ac-vw`) rather than `100vw`, because `100vw` includes the scrollbar and is
the usual cause of a few pixels of horizontal scroll. A sticky side column
holds a heading while content scrolls past it, and stops being sticky below
991px where there is no column beside it. Breakpoints: 479 / 767 / 991 down,
1280 / 1440 / 1920 up.

**Slides.** A slide is 16:9 with `container-type: size`, and every size inside
it is written in `cqw`, so the same markup is a thumbnail, a projected slide and
a PDF page with no media queries. 1280px is the reference. Three slide
surfaces — canvas, gold, brand black — and the modifier must be paired with the
surface class. Emphasis in a slide title is *weight*, not colour. Slide numbers
come from a CSS counter, so inserting a slide renumbers the deck. The deck's
own motif is three chevrons at the bottom right, drawn in CSS from the
monogram's shape so they inherit the slide's colour.

---

## ICONOGRAPHY

**The brand's icon language is isometric line illustration** — thin,
single-weight strokes drawn in three-quarter perspective. This is
guideline-mandated, not a preference, and it is why an araCreate icon never
looks like a Material icon. All of it is SVG. Every file in `assets/icons/` and
`assets/illustrations/` came from araCreate's own library; nothing was redrawn
and nothing was generated.

- **Service and capability icons** (isometric, large — 48–96px): 
  `icon-product-design`, `icon-web-development`, `icon-brand-identity`,
  `icon-building-a-brand-identity`, `icon-graphic-design`,
  `icon-advertising-campaigns`, `icon-brand-strategy`,
  `icon-strategy-and-marketing`, `icon-execution-and-production`,
  `icon-we-are-multidisciplinary`.
- **UI chrome** (simple stroked shapes, not isometric): `arrowhead-left`,
  `arrowhead-right`, `arrow-top`, `link-sharp`, `locations-white`.
- **Social**: `logo-instagram`, `logo-youtube`. Other companies' marks — do not
  restyle them.
- **Illustrations** — larger isometric scenes for heroes and section art:
  `circuit-board`, `charts-pie-and-bars`, `image-creation`,
  `human-computer-interaction`, `customer-service`, plus
  `decorative-element-01/02/04`.

**There is no icon font and no icon-library dependency.** Nothing is loaded
from a CDN. Small interface glyphs — arrow, check, close, chevron, clock, mail,
pin, cog, alert — are drawn as an inline `<symbol>` sprite in the page and
referenced with `<use href="#ac-i-…">`, matching the `.ac-icon` contract:
1.15em square, `fill:none`, `stroke:currentColor`, `stroke-width:1.5`, round
caps and joins. The `Icon` component in `components/core/` wraps that contract.
Icons inherit `currentColor`, so never set their colour directly — set the text
colour of what contains them.

**Emoji are never used**, as icons or anywhere else. Neither are unicode
characters standing in for icons; the one exception is genuine punctuation used
as punctuation (the `·` separator in a marquee or a meta row, and the `»` in the
slide page marker).

**Every icon is one of two things and must declare which**: decoration
(`aria-hidden="true"`) or content (`role="img"` with a label). The system draws
a dashed red outline round any icon that declares neither, and round any
icon-only button without an `aria-label`, so the omission is visible rather than
silent. An icon is rarely enough on its own — icon-only is for controls people
meet constantly.

**When the icon you need does not exist**: draw it in the isometric line style,
or borrow one that matches the thin single-weight line style and flag the
substitution. Never fall back to an emoji or an icon font.

## Logos

`assets/logos/` holds the real files — 21 SVGs, nothing redrawn. **Read the
chip, not the name:** each logo carries a coloured block behind CREATE, sized
for one specific background.

| Background | File |
| --- | --- |
| Canvas, white, **or gold** | `aracreate-logo-default.svg` (on gold the chip vanishes, leaving clean graphite type) |
| Brand black | `aracreate-logo-t-w-b-y.svg` — white type on a gold chip |
| Graphite `#555` | `aracreate-logo-t-w-b-g.svg` |
| Over imagery | `aracreate-logo-negative.svg` (a knockout — *not* the dark-background variant, despite the name) |
| Favicon, avatar, app icon | `aracreate-icon-default.svg`, `-t-w-b-g`, `-t-w-b-y`, `-negative` |
| Wordmark alone, no chip | `aracreate-wordmark-default.svg`, `-t-w-b-g` |

Group and regional lockups: `aracreate-group-logo-*`, `aracreate-india-logo-*`,
`aracreate-lanka-logo-*`. Division lockups, one per vertical:
`aracreate-engineering-logo.svg`, `aracreate-manufacturing-logo.svg`,
`aracreate-media-logo.svg` — use the division lockup on a division site and the
plain logo everywhere else. `aracreate-brand.svg` is the palette reference
sheet, not a logo; do not place it.

Placing one: clear space of one monogram-width on every side; never stretch,
recolour, shadow, outline, gradient or rotate it; never place the default logo
over imagery.

`assets/brand/` holds the seal and the letterhead. They are stationery, not web
furniture — a stamp on a web page reads as decoration and dilutes it.

## Type files

Poppins and Red Hat Mono are both SIL Open Font License and are loaded from
Google Fonts by `css/fonts.css` — the real families, not substitutes.

**Monument Extended is now self-hosted** from `assets/fonts/`, in both supplied
weights — Regular 400 and Ultrabold 800 — and declared in `css/fonts.css`,
which `styles.css` imports. `.ac-wordmark` is the only selector allowed to use
it.

**Two things follow from that, and both matter.** It is a commercial licence,
and declaring it in the imported stylesheet publishes the `.otf` to every
visitor of every page that links `styles.css` — a licence question, not a
technical one. And it is almost never needed: the logo SVGs have the wordmark
outlined to vector paths, so they are crisp at any size, recolourable, and need
no font at all. Prefer the SVG; reach for live wordmark text only where a
template genuinely cannot place an image.

The wordmark is UPPERCASE ARACREATE in Ultrabold. Never a heading, never a page
title, never a slide title, never body copy, never a statistic. Everything that
is not the wordmark is Poppins.

---

## Index

**Root**

- `readme.md` — this file.
- `changelog.md` — what this project changed relative to the source repository, and why.
- `SKILL.md` — Agent Skills front matter, so this system works as a Claude Code skill.
- `styles.css` — the single stylesheet a consumer links. `@import` lines only.
- `thumbnail.html` — the homepage tile.

**`tests/`** — `checks.html`. A browser-runnable port of two of the repository's
seven gates: contrast (compositing translucent backgrounds before measuring) and
accessibility (accessible names, alt text, heading order, ARIA references,
duplicate ids, `th` scope, table captions, positive tabindex, undeclared icons),
plus two brand rules (no emoji, no vague link text). It **globs rather than
lists** — add a path to `PAGES` and it is covered from then on. Open it and it
runs: 55 pages, no install.

**`css/`** — `fonts.css` (webfonts) · `tokens.css` (every colour, size, space,
speed; primitives then semantics) · `base.css` (element defaults, focus ring,
the four surface contexts) · `signature.css` (the araCreate devices — edge,
dash, eyebrow, underline, bleed, marquee, reveal, sticky bar, progress,
scroller, sticky side, dot) · `components.css` · `sections.css` · `app.css`
(the signed-in surfaces — shell, toolbar, table extras, empty state, skeleton,
segmented control, number, slider, combobox, date picker, upload, drawer,
banner, chart shell) · `density.css` (the 44px target floor and the compact
density scope) · `theme-dark.css` · `deck.css`.

**`js/`** — `signature.js` (viewport measurement, reveals, sticky, progress,
scrollers) · `components.js` (tabs, dropdowns, modals, mobile nav, table
sorting, toasts) · `deck.js` (optional arrow-key deck navigation). Vanilla, no
dependencies.

**`assets/`** — `logos/` (21) · `icons/` (17) · `illustrations/` (8) ·
`imagery/` (5 duotint) · `brand/` (2 stationery).

**`components/`** — React primitives, grouped by concern. Each has
`<Name>.jsx`, `<Name>.d.ts` and `<Name>.prompt.md`, and each directory has one
`@dsCard` HTML specimen.

- `core/` — Button, Icon, Badge, Tag, TagGroup, Avatar, AvatarGroup, Card,
  CardMedia, CardBody, CardFooter, CardGrid, Stat, StatRow, Panel
- `forms/` — Field, Input, Search, Textarea, Select, Choice, ChoiceGroup,
  Switch, SegmentedControl, NumberInput, Slider, Combobox, DatePicker,
  FileUpload
- `feedback/` — Alert, Toast, ToastRegion, Modal, Tooltip, ProgressBar
- `navigation/` — Tabs, Accordion, AccordionItem, Breadcrumb, Pagination,
  Dropdown
- `data/` — Table, Steps, PriceCard, Quote, LogoTile, LogoStrip
- `signature/` — Eyebrow, Dash, Underline, Marquee, Chevrons
- `sections/` — Header, Hero, Section, ServiceList, CtaBand, Footer, PostList,
  NotFound
- `app/` — AppShell, AppBrand, NavList, NavFooter, Toolbar, FilterBar, BulkBar,
  DataTable, CellStack, EmptyState, Skeleton, SkeletonStack, SkeletonTable,
  Drawer, Banner, KpiTile, ChartShell, Bars

**Which set to reach for.** `core`, `forms`, `feedback`, `navigation`, `data`
and `signature` serve every surface. `sections` is the marketing website — full
page slabs. `app` is the signed-in product: the shell, the selectable table, and
the states a product has that a marketing page does not (empty, loading,
bulk-selected). A dashboard uses `app` + `core` + `forms`; a landing page uses
`sections` + `core`.

**`ui_kits/`** — `group_website/` · `academy/` · `web_app/` · `deck/`. Each has
its own `README.md` and an `index.html` you can click through.

**`templates/`** — starting folders a consuming project copies. `marketing-page/`
(a full araCreate section page) and `deck/` (six slide layouts). Each has a
`ds-base.js` with one line to point at the bound design system, and a `README.md`
listing what to change.

**`guidelines/`** — the foundation specimen cards that populate the Design
System tab (colour, type, spacing, shape, signature devices, brand assets, dark
theme, density, targets, grid), plus the written documentation.

**Written here, for this project:**

- `components.md` — **the component reference.** All 78: what each is for, what
  each is *not* for, its states, its keyboard behaviour, and its maturity
  (Stable, New, Frame). Start here when choosing a component.
- `accessibility.md` — the contract, every measured contrast figure in both
  themes, the keyboard table, and an honest list of what is **not** verified.
- `contributing.md` — how to add a component without pulling the system apart,
  which group it belongs in, and how this project stays in step with the source
  repository.
- `api-audit.md` — a pass over the prop surface of all 78, ranked by what each
  finding would cost a consumer. Two fixed, one closed, five recorded.
- `back-port.md` — **the reconciliation list.** What this project added that the
  source repository does not have, what should go back into `src/`, what should
  not, and an eleven-line checklist. Read this before the two ever diverge
  further.
- `screen-reader-pass.md` — a forty-minute script for the one check nothing
  automated can do, with the eight places most likely to fail already named.

**Ported verbatim from the source repository**, each with a provenance header:

- `brand-facts.md` — **the source of truth for every fact copy may state.** The
  CONTENT FUNDAMENTALS section above summarises it; if the two disagree, that
  file wins.
- `guidance.md` — when to use each CSS class and, more usefully, when not to.
- `decisions.md` — the decision record. What changed, when, why, and the
  measurement that forced it. Read this before arguing with a value.
- `licence-policy.md` — what is copied and what is rebuilt. araCreate's live
  site was built on a Webflow template under a single-use licence; this explains
  the line the system works to and why the code is original.
- `assets.md` — which logo, icon and image file, on which background.

### Intentional additions

Two wrappers over classes the source styles but never marks up:

- **`Icon`** — the source defines `.ac-icon` as a contract for inline SVG but
  ships no component. The wrapper exists so the `aria-hidden` / `role="img"`
  rule is enforced in code rather than remembered.
- **`Panel`** — `.ac-panel` is styled by `signature.css` (it is the primary
  carrier of the signature edge) but has no markup of its own in the source.

And the whole **`app/` group plus six `forms/` additions**, added 20 August 2026
when the brief widened from "the marketing site" to "the website, the app, the
web app, the deck — all the places". `components.css` was reverse-engineered from
a marketing stylesheet: it had no shell, no selectable table, no empty state, no
skeleton, no date picker. These have no counterpart in the source repository and
are **new design in the araCreate idiom**, not recreations — square, dotted edge
where the thing reads as a panel, gold as the single accent, semantic tokens
only.

They reuse rather than duplicate wherever something already fitted: a KPI tile
*is* `.ac-card--stat`, an inline message *is* `.ac-alert`, a combobox *is* an
`.ac-input` plus a menu, filter tokens *are* `.ac-tag`. Building a second card
that looks 95% like the first is how a system ends up with two of everything and
nobody knowing which is current.

Everything else maps one-to-one onto a class family in `css/components.css`,
`css/signature.css` or `css/sections.css`.

## Rules that outrank convenience

1. **Never write a raw value.** No hex colours, pixel sizes or timings in a
   component. A missing token is a signal: use the nearest one, or add one
   deliberately.
2. **Name the job, not the colour.** `--ac-action-bg`, never `--ac-yellow`.
3. **Yellow is never text.**
4. **Nothing invents a fact.**
5. **Do not undo three things the system handles**: the global `:focus-visible`
   ring, reduced-motion handling, and the inverse-surface context.
6. **Screenshot every new component and look at it.** A whole build of the
   source system rendered card body copy bold — a card that was a link
   inherited the link's font weight — and every automated gate passed. It was
   found by reading a screenshot. This project reproduced the same class of bug
   twice; see the ledger in `guidelines/accessibility.md`.

How to add to the system: [`guidelines/contributing.md`](guidelines/contributing.md).

## The Monument Extended licence decision

**Recorded here because it is a decision, not a side effect.**

The source repository deliberately excludes the two Monument Extended `.otf`
files from version control, and its `licence-policy.md` explains why: the font
is a commercial licence, and linking it publishes the file to every visitor.

You uploaded both weights to this project on 20 August 2026, so they are
self-hosted from `assets/fonts/` and declared in `css/fonts.css` — which
`styles.css` imports. **Consequence: any page that links `styles.css` now serves
the `.otf` to everyone who loads it.** That is the whole of the exposure, and it
is why this paragraph exists rather than sitting in a commit message.

The decision stands on the grounds that you supplied the files knowingly and the
Design System tab needs the real face to show a truthful specimen. **It is
reversible in one line.** To make the font opt-in instead — matching the
repository's policy — delete the `@import url("css/fonts.css")` line's font-face
half by moving the two `@font-face` rules out of `css/fonts.css` into a separate
file that is not imported by `styles.css`, and link that file only on pages that
need live wordmark text. Nothing else breaks: `--ac-font-logo` falls back to
Arial Black, and every logo SVG already has the wordmark outlined to vector
paths, so almost nothing needs the font at all.

**Either way, the rule is unchanged**: the wordmark is uppercase ARACREATE in
Ultrabold, and everything that is not the wordmark is Poppins.

## What this project does not have

Stated plainly so nobody assumes otherwise. The source repository runs seven
gates — conventions, adherence, contrast, accessibility, behaviour, deck, and a
proof that a real page needs no custom CSS. **None of them run here.**
Verification in this project was by hand and is recorded, with its limits, in
[`guidelines/accessibility.md`](guidelines/accessibility.md). There is also no
screen-reader pass, no release process, and nothing has shipped in production on
the twenty-four application components.

The highest-value thing anyone could add is the contrast and accessibility
gates, globbing rather than listing, so a new card or kit is covered the moment
it exists.

Licence: the ported CSS, JavaScript and assets are proprietary, © 2026
araCreate Group.
