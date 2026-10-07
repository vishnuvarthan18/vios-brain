## 20 August 2026 — the graphite surfaces, corrected

Caught in review, and worth recording because the mistake was reasoning rather
than a typo. Moving the dark surface to graphite, I sent the *raised* surface
lighter (`#6f6f6f`) — which is what a dark theme normally does, and which is
wrong here. Graphite is mid-grey: on `#6f6f6f` essentially nothing but white
clears 4.5:1 (`#f0f0f0` reaches 4.40:1), so card body copy measured **3.53:1**
and the eyebrow **3.19:1**, and raising the text values could not have fixed it.

All the headroom on a graphite page is *below* it. Raised is `#4f4f4f` and
subtle `#464646` — graphite tinted with ink, dark grey rather than black — in
the dark theme, in `.ac-surface-inverse` and in the app nav. Every text token
now measures at or above its on-page figure. Separation comes from the signature
edge, which is already this system's rule for a panel at rest.

The feedback hues failed the same way for the same reason: `#fbc7c2` and
`#a8dfb8` passed on the page and measured about 3.9:1 on their own 13% wash,
because the wash lightens the background. Both went one step lighter and both
washes down to 8% — `#fdd8d4` and `#c3e9cf`, passing on the page (5.65:1,
5.62:1), on the wash (4.77:1, 4.76:1) and on raised.

`accessibility.md` now carries two columns, page and raised, because one column
is what let this through.

## 20 August 2026 — token-layer housekeeping

Raised by the design-system check, and all of it the same fault: values that
behaved like tokens but were never declared as tokens, so no consumer could
reach them.

- **The deck's twelve knobs** (`--ac-slide-pad` … `--ac-chevron-size`) sat on
  `.ac-deck`. They are at `:root` now, with `@kind` annotations so the type and
  spacing ones classify. `cqw` is unaffected: a custom property is substituted
  where it is *used*, so each still measures against the slide's container.
- **`--ac-card-min`** existed only inside `.ac-card-grid--wide` and
  `--narrow`, with the real default hidden in a `var()` fallback. Declared at
  `:root`; the two modifiers still override it.
- **`--ac-code-bg` and `--ac-code-text`** were declared only inside the surface
  contexts that re-point them, so `base.css` carried a `var()` fallback and a
  plain page had no token to change. Both are at `:root`; the fallbacks are gone.

What remains flagged is by design: 94 declarations are surface and modifier
classes *re-pointing* registered tokens — `.ac-surface-inverse` re-pointing the
text set, `.ac-edge--thick` re-pointing `--ac-edge-width`. Every name now exists
at `:root`. Moving those overrides out would delete the mechanism that makes a
dark band work without a dark variant per component.

## 20 August 2026 — black leaves the palette

Requested: no pure black anywhere, and no black backgrounds. Graphite `#555555`
replaces both. Nothing outside the token layer changed, because no component
holds a raw colour.

**Retired.** `--ac-black` (`#000000`), `--ac-charcoal` (`#2e2e2e`) and
`--ac-grey-800` (`#383838`) are now aliases, not values — every consumer of
them keeps working and gets graphite. `--ac-brand-black` is renamed
**`--ac-ink`**, kept as an alias, and its role is narrowed in writing: text,
strong borders and the focus ring. Never a fill. The old name was inviting the
exact use that is no longer allowed.

**The dark band is graphite.** `--ac-surface-inverse` moves from `#222222` to
`#555555`, which carries the footer, section-opener and values slides, the app
side nav, table headers, tooltips, code blocks and the featured price card. White
on it measures 7.44:1 — down from 15.91:1, still clear at every size — so every
white-alpha value inside an inverse surface was raised: muted text 70% → 85%
(4.47:1 would have shipped as a fail), dividers 20% → 30%, control borders
50% → 60%. A panel lifted off the band now goes **lighter** (`#6f6f6f`), not
darker; `#383838` on graphite was a near-black panel on a grey one.

**Shadows and the scrim are graphite.** All three shadows and the modal scrim
were black at various alphas. They are graphite with alphas raised to land at the
same strength. The marquee's mask keeps its gradient — mask-mode is alpha, so the
colour there was never rendered.

**The dark theme was re-measured, not re-pointed.** Its page is graphite too,
which is a lighter page than before and moved everything measured against it:
body copy 10.86:1 → 5.22:1, muted text `#a3a3a3` → `#cecece` (`#a3a3a3`
measures **2.99:1** on graphite and would have shipped invisible), and the
feedback pair had to go *lighter* — `#f28b82` and `#81c995` measure 2.95:1 and
3.01:1 here, so error is `#fbc7c2` (4.96:1) and success `#a8dfb8` (4.94:1).
Neither the light pair nor the previous dark pair survives on this page.

**Two knock-ons.** The dark and danger buttons darkened toward `#000` on hover;
with black gone the dark button lifts *lighter* (`--ac-inverse-bg-hover`) and
danger mixes toward graphite (`--ac-error-bg-hover`). And the dark slides and
sign-in screen now carry `aracreate-logo-t-w-b-g.svg` rather than the gold-chip
lockup — per `guidelines/assets.md`, the graphite chip is the one that vanishes
into a graphite band.

# Changelog — the Claude design system

What this project changed relative to the araCreate Design System repository it
was ported from. The repository's own decision record is
[`guidelines/decisions.md`](guidelines/decisions.md); this file only records what
is different **here**.

Dates are the date of the change. Newest first.

---

## 20 August 2026 — The gate's first eleven findings, triaged

`tests/checks.html` found 11 findings on its first run. All eleven are now
closed: **0 findings across 55 pages, 2,040 elements.** Nine were real, two were
faults in the gate itself.

### Fixed — two real defects in the CSS

Both are the same underlying mistake: a **tinted background changes what "muted"
is safe against**, and neither band re-pointed the token.

- **`.ac-surface-subtle` (`css/base.css`)** — `--ac-text-muted` (grey-450) on
  grey-300 measures **3.19:1**. The comment above the rule claimed every text
  token was safe on this surface; it was wrong. Muted and placeholder now drop
  one rung to grey-600, and the comment records the correction.

- **A selected table row (`css/app.css`)** — the gold tint warms the row enough
  to take secondary cell text to **4.35:1**. Muted re-points to grey-600 within
  the row. Worth noting how this escaped the hand pass: the row only fails
  *once selected*, and nobody had clicked one.

Both were fixed at the band rather than at the component, so every current and
future component inside either context is covered.

### Fixed — two defects in the specimen cards

- **`colour-gold-rule.card.html`** — the three captions were 10px at 75% opacity
  and inherited page-dark text, so they measured 4.45:1 on gold and **2.13:1** on
  brand black. Opacity dropped; each caption now carries the correct explicit
  colour for the chip it sits on. The two deliberately-failing headline
  specimens are marked `data-contrast-exempt` — that card's whole job is to show
  a failure, and the gate should not argue with a demonstration.

- **`density.card.html`** — both tables lacked a caption. Added, visually hidden.

### Fixed — two faults in the gate

Worth recording, because a gate that cries wolf gets ignored, which is worse
than no gate.

- **The disabled exemption used `matches()` where it needed `closest()`.** The
  text of a disabled button is usually owned by a child `span`, so the exemption
  never applied to the node actually being measured. One false failure.

- **The client-count rule tested `innerText`.** Flattened across lines, a `12`
  badge in a nav sitting above a `Clients` label read as "12 clients". Now walks
  text nodes individually. One false failure.

### Still true

The gate covers two of the repository's seven. Conventions, adherence,
behaviour, deck and proof-page still need Node and Playwright, and the
screen-reader pass in `guidelines/screen-reader-pass.md` still needs a person.

---

The answer to "what do you need from me" was "go do all six". Two of the six were
doable without you. Four are not — they need material only araCreate has, and
inventing it would break the one rule this system exists to enforce.

### Added

- **`tests/checks.html`** — a browser-runnable port of two of the repository's
  seven gates. It **globs rather than lists**: add a path to `PAGES` and it is
  covered from then on, which was the repo's own reason for making its gates glob
  (its sweep went from 10 pages to 26 the day it changed).

  Contrast, compositing translucent backgrounds before measuring — a see-through
  panel measured naively is a false pass or a false fail, and this project has
  already produced one of each. Accessibility: accessible names, alt text,
  heading order, ARIA references pointing at elements that exist, duplicate ids,
  `th` scope, table captions, positive tabindex, icons declaring neither
  `aria-hidden` nor `role`. Plus two brand rules from the adherence gate: no
  emoji, no link text that says nothing on its own.

  **First run: 55 pages, 2,038 elements, 11 findings** — 8 contrast, 2
  accessibility, 1 brand. **Not yet triaged.** That is the next job, and it is
  exactly what the gate is for: it found things a hand pass over four kits did
  not.

  What it does not replace: conventions, adherence, behaviour, deck and
  proof-page. Those need Node and Playwright.

- **`guidelines/back-port.md`** — the reconciliation list. Which files here are
  copies that a re-port overwrites, which additions should go back to `src/`
  (the dark theme and the density scale, in that order), which should **not**
  (the React components, on the repo's own recorded reasoning; `app.css`, until
  araCreate builds a signed-in tool and it can be written against real screens),
  and an eleven-line checklist. Also the three bug fixes that are faults in the
  *pattern* rather than in new code and belong upstream regardless.

- **`guidelines/screen-reader-pass.md`** — a forty-minute script for the check
  nothing automated can do, with the eight places most likely to fail already
  named: the rail's visually-hidden labels, `aria-selected` on table rows, the
  combobox's unannounced match count, "today" versus "selected" in the date
  picker, upload error association, the danger-zone buttons whose consequence
  sits in an unassociated sibling, and the deck's CSS-counter slide numbers.

### The licence decision, made rather than left

Monument Extended stays self-hosted. The files were supplied knowingly and the
Design System tab needs the real face to show a truthful specimen. **The
consequence is now written into `readme.md` in plain terms** — any page linking
`styles.css` serves the `.otf` to everyone who loads it — along with the one-line
reversal, because a licence exposure recorded only in a commit message is not
recorded.

### The four that could not be done, and what each needs

| Item | Blocked on |
| --- | --- |
| araCreate Meditate kit | screens, a URL, or the code. Nothing about it exists in the sources beyond its name |
| Real figures for the placeholders | Academy fees, engagement pricing, cohort dates, team names, client names |
| Turning the web-app kit into a recreation | whether a real internal tool exists, and its code |
| The screen-reader pass | a person with VoiceOver or NVDA. The script is written; someone has to hear it |

---

## 20 August 2026 — Phases 3 and 4: documentation, audit, governance

### Added

- **`guidelines/components.md`** — the component reference. All 78, grouped, each
  with what it is for, **what it is not for**, its states, its keyboard
  behaviour, and a maturity status (Stable / New / Frame). Includes a
  "choosing between the groups that overlap" table, because `Stat` versus
  `KpiTile` and `Table` versus `DataTable` are the two decisions a consumer will
  get wrong first.

- **`guidelines/accessibility.md`** — the contract and the evidence: every
  measured contrast figure in both themes, the keyboard table, the
  colour-is-never-the-only-signal list, and the ledger of the six faults found
  in this project with what each measured. Ends with **what is not verified** —
  no gates run here, no screen-reader pass, and the compact density has not been
  swept. An accessibility claim without its limits is worth less than none.

- **`guidelines/contributing.md`** — the five rules, the ten-step process for
  adding a component, a test for which group it belongs in, how to change a
  token, the three locked deviations, and how this project stays in step with the
  source repository (which files are copies and will be overwritten, which are
  additions).

- **`guidelines/api-audit.md`** — a pass over the prop surface of all 78, read
  from every `.d.ts` rather than recalled.

### Changed

- **`EmptyState` `align="center"` → `"centred"`.** The only American spelling in
  a codebase that writes `colour`, `licence` and `centred` throughout — including
  `Section centred` two directories away. `"center"` is accepted as an alias so
  the American spelling does not silently do nothing.

### The `label` rename — recorded as deferred, then done

The audit's one finding that mattered, fixed the same day.

`label` now means **one** thing across all 78 components: text the visitor can
see. The accessible-name sense became **`a11yLabel`** on `Icon`, `Search`,
`Tabs`, `Breadcrumb` and `ProgressBar`; the group-heading sense became
**`heading`** on `NavList`. `label` is unchanged where it was already visible
text — `Field`, `Choice`, `Switch`, `Slider`, `Stat` — and inside every
`items` / `options` entry. `caption` stays on the two tables, because a table
genuinely has a `<caption>`.

**Nothing broke.** Every renamed component accepts both spellings
(`a11yLabel || label`), the old name carries `@deprecated` with its date and
reason, and all thirteen call sites across the four kits, the cards and the
example files were updated in the same pass — so the deprecated path is
documented but unused.

The point was legibility at the call site:

```jsx
<Switch label="Keep me signed in" />        // renders text
<Icon name="alert" a11yLabel="Warning" />   // renders nothing
<NavList heading="Work" items={nav} />      // renders a heading
```

The deferral reasoning — that it would break four kits and eleven cards — turned
out to be wrong, because accepting both names costs one line per component. It
was one pass, and it would only have grown.

### The audit's original finding, as recorded

**`label` means three different things**: a visible label on `Field`, `Choice`,
`Switch`, `Slider` and `Stat`; an accessible name on `Icon`, `Search`, `Tabs`,
`Breadcrumb` and `ProgressBar`; and a group heading on `NavList`.
`<Icon label="Save" />` renders nothing visible and `<Switch label="Save" />`
renders text, and the two are indistinguishable in review.

**Not fixed, deliberately.** The correct fix renames props on ten components,
breaking four kits, eleven cards, two templates and every `prompt.md` example.
Doing it half-way is worse than either end state. It should be one deliberate
pass **before** any consumer adopts this — recorded in `api-audit.md` with that
recommendation rather than left as a surprise.

Six further findings are recorded and not fixed, each with its reason: `caption`
versus `label`, `variant` versus `tone` (kept — different axes, now documented),
`text` versus `children`, `size` as a scale versus a spacing knob, the two
awkward booleans (`padSm` abbreviated, `noEdge` negative), and the two
`onChange` signatures. A rule going forward: **no abbreviations, no negatives.**

Also recorded: five coverage gaps found while auditing — no Y-axis label on
`ChartShell`, no range mode on `DatePicker` though `app.css` already styles one,
no page-size control beside `Pagination`, no tooltip that works on a disabled
control, and `Textarea` missing from its card.

---

## 20 August 2026 — Phase 2: application surfaces

The brief widened from "the marketing site" to "the website, the app, the web
app, the deck — all the places". `components.css` was reverse-engineered from a
marketing stylesheet, so it had no shell, no selectable table, no empty state,
no skeleton, no date picker. This adds them.

### Added

- **`css/app.css`** — the signed-in surfaces. App shell (side nav, 64px icon
  rail, top bar, scrolling main), toolbar and filter bar, bulk-action bar, table
  selection and row actions, empty state, skeleton, segmented control, number
  input, slider, combobox, date picker, file upload, drawer, page banner, chart
  shell.

- **24 new components** across `components/app/` and `components/forms/`:
  AppShell, AppBrand, NavList, NavFooter, Toolbar, FilterBar, BulkBar,
  DataTable, CellStack, EmptyState, Skeleton, SkeletonStack, SkeletonTable,
  Drawer, Banner, KpiTile, ChartShell, Bars, SegmentedControl, NumberInput,
  Slider, Combobox, DatePicker, FileUpload. Four new cards.

- **`ui_kits/web_app/`** — a click-through web app: sign in, overview, jobs and
  settings. **This kit is a demonstration, not a recreation** — no app screens
  or product copy were provided, so nothing in it is copied from anywhere and
  the domain is deliberately thin. It exists because 24 application components
  with no screen behind them are a guess, and it earned its keep immediately by
  surfacing both bugs below. Dark mode and compact density are switchable from
  its top bar, which makes phase one demonstrable rather than described.

### Reused rather than duplicated

The instruction was to reuse where something already fitted, and it applies
more often than it looks:

| Wanted | Used | New CSS |
| --- | --- | --- |
| KPI tile | `.ac-card--stat` | none |
| Inline message | `.ac-alert` | none |
| Filter tokens | `.ac-tag` | none |
| Combobox field | `.ac-input` + a menu | one class, for the wiring |
| Bulk actions | `.ac-btn` in a bar | the bar only |
| Upload progress | `.ac-progress-bar` | none |
| Dialogs, pagination | `.ac-modal`, `.ac-pagination` | none |

Building a second card that looks 95% like `.ac-card--stat` is how a system ends
up with two of everything and nobody knowing which is current.

### Decisions worth recording

- **The rail keeps its labels in the DOM.** Collapsed, they are hidden visually
  rather than removed, so a screen-reader user gets the same navigation.
- **A selected table row carries a tint AND a gold left mark.** The tint alone
  fails for anyone who cannot distinguish it — the same reasoning as badges
  never resting on colour.
- **Row actions fade in on hover only where the pointer is fine.** On touch and
  for keyboard they are always present. A control that exists only on hover does
  not exist on a phone.
- **The bulk bar replaces the toolbar rather than floating.** A floating bar
  covers the rows the visitor is trying to check.
- **"Today" in the date picker is an outline, a selected day is a fill.** A
  filled today is indistinguishable from a selection — the commonest
  date-picker bug there is. Monday first, and every day carries a full
  `aria-label`, because "14" alone tells a screen-reader user nothing.
- **The combobox marks matches by weight, not colour.** Gold on white is
  1.67:1, and a coloured highlight fights the selected state.
- **The drop zone is dashed, not the dotted signature edge.** Dashed reads as
  provisional, which is what a drop target is.
- **Gold is series one in a chart**, and every other series is a neutral — so a
  two-series chart survives greyscale printing.
- **`ChartShell` is a frame, not a charting engine.** A chart's marks come from
  data; what a design system owns is the title, legend, gridlines and axis,
  which is the part that goes inconsistent first.

### Two bugs, both found by looking rather than by a check passing

**1 · The table's select-row checkboxes rendered as the browser's blue default**
— round, blue, nothing like the rest of the system. `components.css` styles
checkboxes only inside `.ac-choice`, because on a marketing page every checkbox
has a visible label beside it. A select-row box has no visible label — its name
is an `aria-label` — so it fell through every selector. Fixed in `app.css` with
the values from `components.css` §4, indeterminate included as a dash rather
than a tick: "some of these" is a different statement from "all of these".

**2 · The side nav rendered graphite on brand black at 1.55:1, and its current
item at 1.00:1 — invisible.**

This is **bug 5 from `guidelines/decisions.md`, reproduced in a new file on the
first attempt**: "the inverse surface never re-pointed the surface tokens … the
step markers rendered at 1.08:1". `.ac-app__nav` painted a dark background and
set `color`, which is not enough — every child naming its own token
(`.ac-nav-item` asks for `--ac-text-default`, the current one for
`--ac-text-strong`) kept resolving against the light page.

`.ac-banner` and `.ac-bulk-bar` had the same fault. All three now re-point the
full token set, the way `base.css` does for `.ac-surface-inverse`.

**And a second-order bug inside that fix.** Pinning `--ac-text-strong` to white
is correct in the light theme and wrong in the dark one, where the inverse
surface flips to light — white-on-canvas for the current item. Both text tokens
now follow `--ac-text-on-inverse`, which is right in both themes; the current
item keeps its emphasis from weight, tint and the gold rule instead of a second
colour. The alpha-based values (`#ffffffb3` muted, `#ffffff33` dividers) cannot
follow a token, so `theme-dark.css` overrides them.

Measuring that override found one more: `--ac-grey-450` is the lightest grey
that passes on plain canvas at 4.61:1, but the current nav row is canvas **plus
a tint**, which drops it to **4.18:1**. Graphite is the right muted there —
6.90:1 on canvas, 6.20:1 on the tinted row.

Final measurement, both themes, translucent backgrounds composited before
measuring: **22 elements, 0 failures.**

Neither bug would have been caught by a passing build. The markup is valid, the
controls work, nothing errors. Both were found by rendering the thing and
reading the numbers off it — which is the source repo's standing instruction and
the reason it is written down there.

---

## 20 August 2026 — Phase 1: theme, density, targets

### Added

- **Dark theme** — `css/theme-dark.css`. `tokens.css` had left an empty
  `[data-ac-theme="dark"]` block with the note "skip it, but leave the door
  open". The door is now open. `data-ac-theme="dark"` on any element re-themes
  everything inside it; `data-ac-theme="auto"` follows the operating system.

  The prediction in `tokens.css` held exactly — **no component was touched**,
  because no component contains a raw colour. Three things needed more than a
  token swap:

  - **Golden Sun does not move.** `#f9bf3b` reads correctly on a dark surface
    and lightening it for one theme would give the brand two yellows.
  - **The inverse surface flips to light.** A dark band on a dark page is
    invisible. This is the only structural change, and it required re-pointing
    every token that `base.css` re-points, in the opposite direction — including
    the eyebrow and the decorative quote mark, which turn gold on a dark band
    and must go back to ink on the flipped light one.
  - **Error and success needed dark-specific values.** `#b3261e` measures
    **2.51:1** on brand black and `#186a43` measures **2.79:1** — both fail
    badly. Dark uses `#f28b82` (8.42:1) and `#81c995` (8.73:1). Derived
    accessible values for one theme, not new brand colours.

  Body copy is `#d8d8d8` (10.86:1) rather than white. `#ffffff` on `#222222` is
  15.91:1, which is more contrast than long-form reading wants — it glares. This
  is the value `tokens.css`'s own sketch proposed.

- **A 44px touch-target floor** — `css/density.css`. Every interactive control is
  at least 44px in both directions, applied as padding and `min-height`, never by
  growing the type: a 12px label inside a 44px target is fine, a 12px target is
  not.

  Checkboxes and radios stay 18px — the `<label>` carries the target, which is
  why the control is wrapped in one. Links inside running text are exempt; one
  cannot be 44px tall without wrecking the line.

- **A compact density** — `data-ac-density="compact"` on any region. Tightens
  control padding, table cells, card bodies, field gaps and section rhythm for
  signed-in, data-dense screens. **Scoped to `pointer: fine`** — on a touch
  device it tightens the type rhythm but the 44px floor holds, because a dense
  table on a phone is still operated by a finger.

- **Named control geometry** — `--ac-target-min`, `--ac-control-h`,
  `--ac-row-h`, `--ac-cell-pad-x/y`, `--ac-stack-gap` and siblings. These sizes
  existed as paddings inside `components.css`; naming them is what lets a density
  scope move all of them at once. Same rule-one reasoning as the eighteen raw
  font sizes named in the source repo on 20 August.

- **Monument Extended, self-hosted** — `assets/fonts/`, both supplied weights,
  declared in `css/fonts.css`. The source repo deliberately excludes the OTFs
  from version control; the client supplied them directly for this project.
  **This is a licence decision, knowingly taken**: linking `styles.css` now
  publishes the `.otf` to every visitor of every page. The outlined logo SVGs
  remain the right answer wherever an image will do.

- **Ported prose** — `guidelines/decisions.md`, `licence-policy.md`,
  `guidance.md`, `brand-facts.md`, `assets.md`, each verbatim with a provenance
  header.

- **Foundation cards** — dark theme palette, dark theme in use, touch targets,
  density comparison, grid and breakpoints, Monument Extended specimen.

### Changed

- **`--ac-size-link`: 12px → 15px.** The one locked deviation of three that was
  unlocked, at the client's request. Two separate problems lived in that number
  and both are now fixed: readability (15px, matching `--ac-size-small`, still
  visibly quieter than 16px body copy) and target size (a 44px hit area from
  `density.css`).

  Navigation, footer and breadcrumb links also moved off `--ac-size-caption`
  onto `--ac-size-link`. They were 12px because a caption is 12px, but they are
  links, not captions. Overridden in `density.css` rather than `sections.css` so
  that file stays diffable against the source.

  **Visible consequence, recorded so nobody reports it as a bug:** footer and
  navigation links are larger than on aracreate.group, and footer columns are
  taller. That is the fix, not a side effect.

- **`--ac-chevron-size` hoisted to `.ac-deck`.** It was declared only inside
  `.ac-chevrons__mark` with no base value — the one property in the system with
  no declaration in a token scope.

- **`/* @kind other */` annotations** on the nine tokens whose kind cannot be
  inferred from name or value (durations, easings, z-indexes, border widths).

### Not changed, on purpose

- **The heading ladder** (36 / 33 / 31 / 28px) and **body weight** (Poppins Light
  300) remain locked. The client kept both when offered the chance to unlock
  them.
- **Golden Sun is still never text.** No token was added for it.
- **The signature edge, the surface contexts and the reduced-motion handling**
  are untouched. The source repo lists these as three things not to undo.

---

## 20 August 2026 — Initial port

The system as delivered: 279 tokens, 54 React components across seven groups,
51 specimen cards, three UI kits (group website, Academy, deck), two templates,
and the full asset library.

**What differs structurally from the source repository.** That repo is
class-based CSS with no React, and its own decision record explains why. This
project wraps the same classes as React components because a Claude design
system's components are consumed as React — the CSS remains the single source of
truth and every component reads it rather than duplicating values inline.

**Two intentional additions**, both wrappers rather than new design: `Icon`
(enforces the decoration-versus-content rule in code) and `Panel` (`.ac-panel` is
styled by `signature.css` but had no markup of its own). Both are listed in
`readme.md`.

**Deliberately blank.** Pricing, team members other than the founder, client
names, journal articles and Academy fees are visible placeholders — `00+`,
`Client name`, `Article title` — because none of that is in the sources.

**Not built.** No araCreate Meditate kit: it is named as an intended consumer but
no screens, copy or layouts were provided.
