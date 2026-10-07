# Live Site CSS Audit — araCreate Website (production)

- **Source:** `01-input/live-site/araCreate Website/_assets/cdn.prod.website-files.com/63780fb6eec282197fc5547f/css/aracreate.webflow.shared.f91f6cfc4.min.css`
- **Size:** 567,526 bytes, minified, single line.
- **Method:** grep/regex extraction (file not read in full — too large for context). Counts are raw occurrence counts of each exact declaration/value string in the minified file (i.e. how many rules use that value, not how many elements render with it).
- **Scope:** step 1 of the design-token pipeline — cataloging only, no normalization or redesign decisions made here.

---

## 1. Colors

All color values in this file are **hex** (3/4/6/8-digit, including alpha hex). No `rgb()`/`rgba()`/`hsl()`/`hsla()` functions are used anywhere in the file (confirmed: 0 matches each). 159 unique hex strings total.

### Likely real brand colors (high frequency, used in `color:`/`background-color:`)

| Value | Total occurrences | Role (inferred) |
|---|---|---|
| `#2e2e2e` (+ alpha variants `#2e2e2e33`, `#2e2e2e80`, `#2e2e2e66`, `#2e2e2e99`, `#2e2e2ecc`, `#2e2e2e0d`) | 6 solid + 40+24+... (alpha variants heavily used, esp. `33`=20% and `80`=50%) | **Primary near-black / body text color.** Also used as shadow color base. This is effectively the "ink" neutral. |
| `#fff` / `#ffffff` + alpha (`fff0`, `fff6`, `fff9`, `fffc`, `ffffffb3`, `ffffff8a`) | 36 solid + several alpha | White — base background / text-on-dark. |
| `#000` / `#000000` + many alpha variants (`0000`, `0003`, `0006`, `0009`, `000c`, `00000014`, `0000001a`, `00000021`, etc.) | 49 solid + ~25 alpha variants | Black — mostly used as **shadow color**, not surface fill (see box-shadow section). |
| `#f6f6f6` | 21 (18 as background-color) | Light gray surface — likely a standard "subtle background" token. |
| `#f9bf3b` | 9 (8 as background-color) | **Gold/yellow accent** — reads as a real brand accent color (buttons/highlights), not noise. |
| `#3347a0` (+ `#3347a099`, 4 solid + 5 alpha) | 9 total | **Brand blue** — used as background and in box-shadow (`0 4px 9px -3px #3347a099`), paired directly with `#f9bf3b` usage pattern. Likely the secondary brand color alongside the gold. |
| `#3898ec` | 7 (incl. 2x `box-shadow:0 0 3px 1px #3898ec`) | Webflow's **default focus-outline blue** — this is a Webflow template artifact, not an intentional brand color (every Webflow site ships this exact hex for default link/focus states). Flag as noise/template leftover. |
| `#252525` | 11 | Secondary near-black, close to `#2e2e2e` (6-point drift) — likely meant to be the same ink color, drifted. |
| `#555` | 17 | Mid-gray — secondary text color candidate. |
| `#969696` | 7 | Mid-gray — possible disabled/placeholder text. |

### Near-duplicate clusters (likely unintentional drift)

- **Near-black ink cluster:** `#2e2e2e` (6), `#252525` (11), `#222` (6), `#1c1d21` (2), `#1a1b1f` (1+alpha) — four to five different "almost black" values where there's likely supposed to be one or two. `#2e2e2e` and its alpha variants dominate by volume, so that's the real token; the others are drift or distinct secondary darks worth a design call.
- **White-adjacent cluster:** `#fff` (36), `#fafafa` (5), `#f6f6f6` (21), `#f5f5f5` (2), `#f3f3f3` (1), `#f0f0f0` (1) — a gradient of near-whites. `#fff` and `#f6f6f6` are the two with real volume; the rest (`#fafafa`, `#f5f5f5`, `#f3f3f3`, `#f0f0f0`) are each used 1-5 times and are candidates for consolidation.
- **Mid-gray cluster:** `#ddd` (6), `#ccc` (6), `#cecece` (4), `#c8c8c8` (4), `#e5e5e5` (1), `#e2e2e2` (1), `#e9e9ea` (1) — border/divider grays, no single dominant value; likely arbitrary per-component choices rather than a scale.
- **Blue cluster (non-focus-outline):** `#0766c4` (2), `#0082f3` (2), `#0050bd` (2), `#2895f7` (1), `#2672cc` (1), `#1a50a2` (1), `#2060b6` (1), `#0096ec` (1), `#164190` (1), `#0d2973` (1), `#3b79c3` (1), `#5792d8` (1), `#82aee2` (1) — a long tail of one-off and twice-used blues, likely link-state variations or inline per-page overrides rather than tokens. Worth checking if these map to a single blue with computed hover/active shades.

### One-off / noise (appear 1-2 times each)

~110 of the 159 unique hex values appear only once or twice. Many look like per-section accent colors for specific content blocks (e.g. `#ea384c`, `#c94138`, `#b82e48`, `#a8243f`, `#9a1b37`, `#810c28` — a red/maroon ramp; `#87ca81`, `#4fb247`, `#2aa120`, `#21791b`, `#1d6719`, `#164a15` — a green ramp; `#f7da92`, `#f4ce6e`, `#f0c146`, `#d5a237`, `#bd862a`, `#a86f1e`, `#814207` — a gold/amber ramp). These look like **deliberate multi-step color ramps** (possibly for a chart, icon set, or category-tagging system) rather than accidental drift — worth flagging to the person who built the site rather than discarding, since the pattern (5-7 steps per hue, descending lightness) is too systematic to be random.

**Assessment:** Real brand palette is almost certainly **gold `#f9bf3b` + blue `#3347a0` + ink `#2e2e2e` + white `#fff` + light gray `#f6f6f6`**. `#3898ec` is a Webflow template default, not brand. The red/green/gold ramps are likely an intentional secondary system (status colors or category tags) — confirm with stakeholder before treating as noise.

---

## 2. Typography

### Font families (11 unique declarations)

| Value | Count |
|---|---|
| `Poppins,sans-serif` | 57 |
| `Red Hat Mono,sans-serif` | 2 |
| `webflow-icons!important` / `webflow-icons` | 1 + 1 (icon font, not content) |
| `unset` | 1 |
| `serif` | 1 |
| `sans-serif` | 1 |
| `monospace` | 1 |
| `Inconsolata,monospace` | 1 |
| `Helvetica Neue,Helvetica,Ubuntu,Segoe UI,Verdana,sans-serif` | 1 (Webflow default fallback stack, likely unused/template remnant) |
| `Arial,sans-serif` | 1 |

**Assessment:** This is clean — **Poppins is the dominant and essentially only real font** (57 declarations vs. everything else at 1-2). `Red Hat Mono` appears twice, likely an intentional monospace accent (code/label use). Everything else is template boilerplate or unused fallback noise, not real content typography.

### Font sizes (76 unique values — px and rem mixed)

Top values by count: `14px`(72), `20px`(66), `16px`(54), `12px`(42), `13px`(40), `18px`(38), `36px`(28), `15px`(28), `24px`(25), `21px`(17), `17px`(15), `19px`(14), `42px`(12), `32px`(12), `28px`(11), `26px`(11), `35px`(10), `30px`(9), `22px`(8), `34px`(7).

A separate **rem-based scale** also exists, concentrated at: `1rem`(7), `1.25rem`(4), `1.125rem`(4), `.75rem`(3), `.8rem`(2), `.889rem`(2), `.875rem`(2), `.65rem`(2), `2.25rem`(2), `1.875rem`(2), `1.5rem`(2), plus one-offs `3.052rem`, `2.441rem`, `1.953rem`, `1.802rem`, `1.602rem`, `1.563rem`, `1.424rem`, `1.266rem`. The rem sequence (`1, 1.266, 1.424/1.563, 1.602/1.802, 1.953, 2.441, 3.052...`) looks like a **type-scale ratio system** (roughly ~1.125–1.25 ratio steps), likely from a rich-text/CMS block style rather than the main UI — this is the strongest evidence of an actual modular scale anywhere in the file.

Outliers: very large display sizes `240px`, `220px`, `200px`, `180px`, `145px`, `120px`, `110px`, `65px`, `58px`, `51px`, `39px` — these are likely hero/display headings per-page, not part of a system.

**Assessment:** The **px values do not follow a clean 4/8 grid** — e.g. `13px`, `17px`, `19px`, `21px`, `23px`, `26px`, `27px`, `31px`, `33px`, `34px`, `35px`, `37px`, `38px` all appear, which is inconsistent with a disciplined scale and suggests organic, page-by-page sizing in Webflow's visual editor. The **rem values do follow a scale** (see above) and should be the reference point for the real type-scale token if one is to be extracted. Recommend building the new type scale from the rem sequence, not the px sprawl.

### Font weights (8 values — clean)

`400`(65), `500`(54), `300`(44), `600`(23), `700`(7), `200`(3), `100`(1), `unset`(1).

**Assessment:** Clean set, maps directly to Poppins' available weights (100–700). 400/500/300/600 cover the vast majority of usage — likely the real weight scale is {300, 400, 500, 600, 700}.

### Line-heights (55 unique values — mixed em/px/unitless)

Dominant: `1.5em`(39), `1.4em`(28), `34px`(22), `32px`(15), `2em`(12), `19px`(12), `1em`(11), `1.7em`(11), `1.5`(unitless, 11), `1.8em`(10), `56px`(8), `21px`(8), `27px`(7), `1.6em`(7).

**Assessment:** Mixed units (em, px, unitless) for the same concept is itself a finding — three different authoring conventions were used over time, likely across different build phases of the site. The em-based values (`1.4em`, `1.5em`, `1.6em`, `1.7em`, `1.8em`, `2em`) look like a deliberate step pattern (0.1 increments) and are the better candidate for a token scale vs. the px values, which are mostly literal "pixel line-height matched to a specific font-size" and won't generalize.

### Letter-spacing (16 unique values — mostly clean)

`.03em`(14), `.05em`(11), `.02em`(8), `0`(5), `.15em`(5), `.07em`(3), `.04em`(2), `50px`(2, likely a typo/outlier — 50px letter-spacing is extreme), plus one-offs `1.05px`, `.25px`, `.08em`, `.02px`, `-.03em`, `-.02em`, `unset`, `normal`.

**Assessment:** The em-based values form a reasonably clean micro-scale (.02–.05em common, .07/.15em as wider tracking for caps/labels). `50px` appears twice and is almost certainly a mistake or an intentional "spread apart" decorative treatment — verify visually before deciding if it's a bug.

---

## 3. Spacing (margin/padding)

4,116 total margin/padding declarations (shorthand + individual sides) extracted; 208 unique numeric token values after splitting shorthand into individual values.

### Most-used values

| Value | Count | | Value | Count |
|---|---|---|---|---|
| `0` | 1603 | | `20px` | 337 |
| `40px` | 267 | | `10px` | 167 |
| `30px` | 125 | | `80px` | 102 |
| `auto` | 92 | | `90px` | 61 |
| `60px` | 58 | | `100px` | 55 |
| `5px` | 48 | | `70px` | 46 |
| `15px` | 40 | | `140px` | 36 |
| `120px` | 31 | | `8px` | 29 |
| `24px` | 27 | | `1px` | 21 |

Plus a parallel **rem-based set**, each appearing exactly 36 times (strong signal of a shared utility class system): `1rem`, `2rem`, `3rem`, `4rem`, `5rem`, `6rem`, `8rem`, `10rem`, `1.5rem`, `.5rem` — and their negative-margin counterparts `-1rem` through `-10rem` (18 each). This is clearly Webflow's generated **spacing utility class set** (`.margin-xsmall`, `.padding-large`, etc.) — a real, consistent system, distinct from the inline px values below.

### Scale assessment

- The **rem utility set is a clean, deliberate 1/2/3/4/5/6/8/10 rem scale** (i.e. roughly 8px/16px/24px/32px/40px/48px/64px/80px at 16px root) — this is a real, importable scale.
- The **px values are not on a consistent grid.** They roughly cluster around multiples of 5 and 10 (`5, 10, 15, 20, 30, 40, 60, 70, 80, 90, 100, 120, 140px`), but not strictly on 4px or 8px — e.g. `9px`, `26px`, `35px`, `45px`, `94px`, `77px`, `132px` all appear with count ≥3, which breaks a clean grid. This is typical of Webflow visual-editor drag-resizing rather than a token system.
- Large negative values (`-245px`, `-229px`, `-215px`, `-206px`, `-195px`, `-189px`, `-187px`, `-153px`, `-125px`, `-164px x2`) and large fixed widths used as spacing (`600px`, `850px`, `950px`, `480px`, `770px`, `750px`, `580px`, `560px`, `550px`, `510px`) look like layout-positioning hacks (offsetting elements, container max-widths) rather than spacing-scale tokens — exclude these from the spacing token extraction; they belong in a layout/breakpoints conversation instead.

**Recommendation:** Build the spacing scale from the rem utility set (1/2/3/4/5/6/8/10rem + .5rem), treat the px sprawl as evidence of inconsistent historical practice to be normalized away, not preserved.

---

## 4. Border-radius

20 unique values:

| Value | Count |
|---|---|
| `20px` | 13 |
| `50%` | 9 |
| `4px` | 5 |
| `9px` | 4 |
| `0` | 4 |
| `3px` | 3 |
| `100%` | 3 |
| `2px` | 2 |
| `999px` | 1 |
| `5px` | 1 |
| `3px!important` | 1 |
| `200vw` | 1 |
| `200px` | 1 |
| `1rem` | 1 |
| `1px` | 1 |
| `100px` | 1 |
| `1.5rem` | 1 |
| `.5rem` | 1 |
| `.25rem` | 1 |

**Assessment:** `50%`/`100%` = circular elements (avatars/icons), `999px`/`200vw`/`200px`/`100px` = pill-shaped buttons/badges, `20px` is the dominant real corner-radius token (13 uses). `4px`, `9px`, `3px`, `2px` look like smaller-component radii (cards, inputs, tags) but don't follow a clean doubling scale. The rem values (`.25rem`, `.5rem`, `1rem`, `1.5rem`) each appear once — possibly from a CMS rich-text block, low confidence as a real pattern given n=1 each.

---

## 5. Box-shadow

18 unique declarations, 5 of which are "real" (non-`none`/`unset`) and repeated:

| Value | Count |
|---|---|
| `0 4px 9px -3px #0003` | 7 |
| `none` | 5 |
| `0 4px 9px -3px #3347a099` | 5 |
| `0 0 3px 1px #3898ec` | 2 (Webflow focus-ring default) |
| `0 4px 9px -3px #2e2e2e33` | 1 |

The remaining ~10 are **one-off, multi-layer shadows** (2-4 shadow layers stacked), e.g.:
- `0 4px 8px -9px #0000000f,8px 8px 22px -9px #00000017,0 22px 51px -9px #00000021`
- `0 4px 7px -4px #0000000a,0 8px 14px -4px #0000000d,0 14px 27px -4px #0000000f,0 22px 47px -4px #00000014`
- `0 1px 2px #0000000a,0 2px 7px #0000000d,0 7px 16px #00000017`

**Assessment:** There is one real, reused elevation token: **`0 4px 9px -3px` with varying color** (black 20% alpha, brand-blue 60% alpha, and ink 20% alpha versions) — this is the closest thing to a deliberate "card elevation" shadow in the file, used 13 times combined across 3 color variants. The multi-layer shadows are likely Webflow's default "elevation preset" library (these look like standard layered-shadow presets, e.g. Material-style small/medium/large elevation sets) applied ad hoc to specific components — each used once, so treat as candidates for 2-3 elevation levels (sm/md/lg) rather than noise, but confirm visually which components use which.

---

## 6. Transitions / Animations

11 unique `transition` declarations, all simple (no complex multi-property choreography beyond 2 properties):

| Value | Count |
|---|---|
| `transform .5s` | 8 |
| `opacity .4s` | 5 |
| `background-color .8s cubic-bezier(.165,.84,.44,1),transform .8s cubic-bezier(.165,.84,.44,1)` | 5 |
| `transform .8s cubic-bezier(.165,.84,.44,1),background-color .8s cubic-bezier(.165,.84,.44,1)` | 1 (same pair, reversed order) |
| `transform .35s` | 1 |
| `padding-top .2s linear,padding-bottom .2s linear` (and reversed order variant) | 1 each |
| `color .6s` | 1 |
| `background-color .1s,color .1s` | 1 |
| `all .3s` | 1 |

**Durations in use:** `.1s`, `.2s`, `.3s`, `.35s`, `.4s`, `.5s`, `.6s`, `.8s` — a reasonably clean set, no default easing specified except where `cubic-bezier(.165,.84,.44,1)` appears (5+1 = 6 uses) — this is a named/standard "ease-out-quart"-family curve, likely the one deliberate custom easing in the system. Everything else defaults to browser `ease`.

One `@keyframes spin` with `animation:.8s linear infinite spin` — standard loading-spinner animation.

**Assessment:** Real tokens to extract: **durations `.2s`/`.3s`/`.5s`/`.8s`** (fast/base/slow/slower) and **one custom easing curve `cubic-bezier(.165,.84,.44,1)`**. The `.1s`, `.35s`, `.6s` are one-offs and likely not worth preserving as named tokens.

---

## 7. Breakpoints (media queries)

7 unique `@media` rules, all standard Webflow responsive breakpoints:

| Breakpoint | Count | Typical Webflow tier |
|---|---|---|
| `max-width:991px` | 5 | Tablet |
| `max-width:767px` | 5 | Mobile landscape |
| `max-width:479px` | 5 | Mobile portrait |
| `min-width:1920px` | 2 | Large desktop (custom, non-default) |
| `min-width:1440px` | 2 | Desktop (custom, non-default) |
| `min-width:1280px` | 2 | Laptop (custom, non-default) |
| `min-width:768px` | 1 | Tablet-up (custom, non-default) |

**Assessment:** The three `max-width` breakpoints (991/767/479) are **Webflow's standard default breakpoint set** — expected and not a custom decision. The four `min-width` queries (1920/1440/1280/768) are **custom additions on top of Webflow's defaults**, suggesting someone added large-screen handling beyond the template. These seven values are the real breakpoint set to carry into tokens — no noise here, this category is clean.

---

## Summary: what's signal vs. noise across all categories

| Category | Signal (real system) | Noise (template/drift/one-off) |
|---|---|---|
| Colors | `#f9bf3b` gold, `#3347a0` blue, `#2e2e2e` ink, `#fff`, `#f6f6f6` | `#3898ec` (Webflow default), near-black/near-white drift clusters, long tail of 1-2x blues |
| Typography | Poppins (dominant), weights {300,400,500,600,700}, rem type-scale, em line-height steps | Fallback font stacks, px line-heights, `50px` letter-spacing outlier |
| Spacing | rem utility scale (.5/1/2/3/4/5/6/8/10rem) | px value sprawl not on a 4/8 grid, large negative offsets (layout hacks) |
| Border-radius | `20px` cards, `50%`/`100%` circles, pill radii | `4px`/`9px`/`3px`/`2px` small radii (no clean scale), rem one-offs |
| Box-shadow | `0 4px 9px -3px` elevation (3 color variants) | Webflow focus-ring, assorted multi-layer one-offs (candidate elevation presets) |
| Transitions | durations .2/.3/.5/.8s, one custom cubic-bezier | `.1s`, `.35s`, `.6s` one-offs |
| Breakpoints | All 7 values are real/clean | None |

Full raw grep output (all categories) is in the scratchpad at `/private/tmp/claude-501/-Users-vishnuvarthanvenkatapathy-araCreate-AC-ACDS/09a74d35-ae80-4c3c-b4d9-6bb6203f1909/scratchpad/audit/` if deeper drill-down is needed later.
