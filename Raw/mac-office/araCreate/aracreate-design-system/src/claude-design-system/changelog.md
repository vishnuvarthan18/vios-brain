# Changelog — araCreate Design System (ACDS)

**Current version: v2.0.1** — 1 September 2026.

Every change to this project, newest first. Each entry carries a version; the
same version is stated at the top of [`readme.md`](readme.md) and is what an
export to the repository is tagged with.

## How versions are assigned

- **Major** — a change a consumer cannot absorb without editing their code: a
  token renamed or removed, a component's prop surface broken, an entry-point
  stylesheet changing meaning.
- **Minor** — new tokens, components, cards, kits, templates or documentation;
  a token value changing within its role (a contrast repair, a palette move).
- **Patch** — a fix that changes nothing a consumer relies on: a corrected
  card, a doc correction, a gate fix.

## Version index

| Version | Date | What | In the repo |
| --- | --- | --- | --- |
| **v2.0.1** | 1 Sep 2026 | `styles/app.css` still read the removed `--ac-gray-600` in seven places | UNVERIFIED |
| v2.0.0 | 1 Sep 2026 | Clean master: aliases removed, `styles.css` → `tokens.css`, deck template tokenised, readme rewritten | UNVERIFIED |
| v1.13.0 | 1 Sep 2026 | Deck conventions written down; chrome unified across surfaces | UNVERIFIED |
| v1.12.0 | 1 Sep 2026 | `ui_kits/` is gone: the website kit merged into `acds-template-web` | UNVERIFIED |
| v1.11.0 | 1 Sep 2026 | The 4.45:1 pairing narrowed from eleven components to one | UNVERIFIED |
| v1.10.0 | 1 Sep 2026 | Ara's weight scale swept across every stylesheet | UNVERIFIED |
| v1.9.0 | 1 Sep 2026 | `acds-template-app` created; the web-app kit folded into it | UNVERIFIED |
| v1.8.0 | 1 Sep 2026 | araCreate Academy kit removed | UNVERIFIED |
| v1.7.0 | 1 Sep 2026 | One web template, `acds-template-web`; the duplicate page files removed | UNVERIFIED |
| v1.6.0 | 1 Sep 2026 | One deck: `ui_kits/deck/` folded into `templates/deck/` and removed | UNVERIFIED |
| v1.5.0 | 1 Sep 2026 | Ara's weight scale: emphasis and numbers are medium, not bold | UNVERIFIED |
| v1.4.0 | 1 Sep 2026 | Brand black retired; Graphite Gray is the only neutral | UNVERIFIED |
| v1.3.0 | 1 Sep 2026 | Graphite Gray is the text colour at every level, headings included | UNVERIFIED |
| v1.2.0 | 24 Aug 2026 | Repository record corrected; source of truth settled; this changelog versioned | UNVERIFIED |
| v1.1.0 | 23 Aug 2026 | `--ac-text-muted` passes contrast; four dead tokens removed | yes |
| v1.0.0 | 21 Aug 2026 | The 20 August system merged into ACDS | yes |
| v0.5.2 | 20 Aug 2026 | Graphite surfaces corrected | yes |
| v0.5.1 | 20 Aug 2026 | Token-layer housekeeping | yes |
| v0.5.0 | 20 Aug 2026 | Black leaves the palette | yes |
| v0.4.1 | 20 Aug 2026 | Test gate added; its first eleven findings triaged | yes |
| v0.4.0 | 20 Aug 2026 | Documentation, API audit, governance | yes |
| v0.3.0 | 20 Aug 2026 | Application surfaces | yes |
| v0.2.0 | 20 Aug 2026 | Dark theme, density scale, 44px targets | yes |
| v0.1.0 | 20 Aug 2026 | Initial port | yes |

**In the repo** is filled only from a sha Claude Code reports after a push;
`UNVERIFIED` means no such report has arrived. It records whether that version has been exported to
`aracreate-group/aracreate-design-system` (`main`, `src/claude-design-system`).
ACDS here is authoritative; the repository is backup and audit trail, and
changes flow one way — out. Sync state and dates live in
[`github.md`](github.md); this column is the summary.

## Releasing

1. Add the entry here under a new version heading and add its row to the index.
2. Update the version line at the top of this file and of `readme.md`.
3. Export and commit to the repository on `main`.
4. Set the row's **In the repo** cell to yes and refresh `## Last sync` in
   `github.md` with the date and commit sha.

---

## v2.0.1 — 1 September 2026

### The alias sweep missed one file

Found by Claude Code while pushing v2.0.0: `styles/app.css` still referenced
`var(--ac-gray-600)` seven times (the accent banner and bulk bar's `--ac-text-muted`
and `--ac-focus`, the selected-row muted text, the dark banner's subtle surface,
and the second chart-key swatch), and the token no longer exists. Effect: those
properties resolved to nothing and inherited; the chart swatch had no colour. The
v2.0.0 sweep covered `base.css` and `theme-dark.css` and skipped `app.css`.

All seven now read `--ac-graphite-gray`, the value the alias always resolved to,
so nothing changes on screen relative to v1.13.0. A tree-wide grep for every
retired name now hits only changelog and history prose. Two stale comments
corrected on the way: `colors.css` named `--ac-gray-600` as its primitive
example, and `deck-conventions.md` still called `--ac-slide-foot-lg` an alias.

## v2.0.0 — 1 September 2026

### A clean master

Ara's instruction: ACDS is the source of truth, the Webflow site is today's only
live consumer and will be rebuilt on this system, so the project should be in a
clean state rather than carry names kept for consumers that do not exist yet.
Major bump because names were removed, not because anything looks different —
no value changed on screen apart from the two colour repairs noted below.

**Removed, not aliased.** `--ac-black`, `--ac-gray-900`, `--ac-gray-700` and
`--ac-gray-600` (every reference now reads `--ac-graphite-gray`, 47 in
`theme-dark.css` and `base.css`), `--ac-white` (→ `--ac-canvas`; it was off-white,
not white, and the name invited the wrong use), `--ac-weight-semibold`,
`--ac-slide-foot-lg`, and the `components/content/` compatibility layer
(`ServiceCard` → `Card kind="service"`, `StatBlock` → `Stat`). 81 components in
eight groups. The gray ramp now lists only real rungs.

**Two entry points, named for what they do.** `styles.css` is gone; `tokens.css`
is the four token imports, `system.css` imports `tokens.css` and every stylesheet.
Every reference updated: the six foundation cards, `thumbnail.html`, the three
`ds-base.js`, `SKILL.md`, the template READMEs, `tokens/fonts.css`.

**`fonts.css` loads only what the system uses.** Poppins 600 and Inconsolata are
dropped from the Google Fonts request; Red Hat Mono loads 400 and 500. Neither
removed face is referenced anywhere in the tree.

**`acds-template-deck` obeys rule one.** Every colour is `var(--token, fallback)`
— no raw hex or `rgba()` remains outside a fallback — and every chrome and type
size reads the `--ac-slide-*` knobs from `styles/deck.css` the same way
(`font-size: var(--ac-slide-heading, 71px)`), with `container-type: size` on
each slide so the `cqw` values resolve. `ds-base.js` now loads `tokens.css` and
`styles/deck.css`, so the whole deck retunes from one block. The slide canvas
`#f3f3f1`, until now a hex only the template knew, is unified on `--ac-canvas`
`#f6f6f6` through `--ac-surface-slide` (one off-white in the system; the printed
deck's value was 1% darker and slightly warm), and `.ac-slide` in `deck.css`
uses the same token. Two
repairs while there: the case-study figures on slide 08 were Golden Sun *text*
on canvas (1.67:1, against rule three) and are graphite at 500 like every other
figure; the project-card placeholders and muted labels use `--ac-gray-200` and
`--ac-text-muted` rather than one-off values.

**`tests/checks.html` reads `_ds_manifest.json`.** The hand-typed page list is
gone; every `@dsCard` is walked automatically, with an `EXTRA` array for pages
that are not cards.

**Documentation.** `readme.md` rewritten from 339 lines to describe the system
as it stands — the merge narrative, sources and the retired-names list moved to
`docs/history.md`, the merge provenance table from `github.md`. Corrected on the way: readme still said `--ac-surface-dark`
was brand black, links darkened to black, muted text failed at 3.19:1, headings
and body were 300, there were 36 cards and the deck template held six slides;
`typography.css` comments pointed at repository paths (`src/`, `css/`,
`docs/deck.html`); `decisions.md` said links went to 15px. `uploads/` duplicate
of `deck-conventions.md` deleted. Rule eight is new: **no aliases**.

**Export package.** `docs/export/EXPORT.md` gives the procedure for pushing this
version to the repository (replace the subtree wholesale, 56 deletions, the
breaking-name list), with corrected copies of the repository's `README.md` and
`src/readme.md` beside it — both still claim the repo is the master.
`docs/back-port.md` retired; `github.md` rewritten as the sync record.

**Export rules (1 Sep 2026, from Claude Code).** Remote state is never asserted
from here: `commit:`, the "In the repo" column and `state.json` → `remote` read
`UNVERIFIED` until a sha arrives. The export gate is a generic
references-resolve-to-definitions sweep (`docs/export/EXPORT.md`), not a name
list. The zip ships `docs/export/state.json` as the machine-readable handshake
and no longer carries `HANDOFF.md`. `decisions.md` still named `--ac-gray-600`
as its primitive example; fixed. v2.0.0 was committed locally as `9ef7587`,
unpushed.

## v1.13.0 — 1 September 2026

### Deck conventions, written down and partly enforced

Vishnu's working notes on deck building are now `docs/deck-conventions.md`, at
the system's 1280 reference rather than the 1920 they were written at, and
`styles/deck.css` enforces the parts CSS can reach. Two rules he declined
(a numbered section eyebrow; the contents-slide and per-section-divider rhythm)
are recorded as not adopted rather than dropped, so the next deck can revisit
them.

**Chrome no longer changes with the surface.** Gold and graphite slides took a
126px footer against canvas's 90px, and grew the chevron motif to 4cqw to match.
The effect was that the standing line and the motif moved every time the deck
crossed a surface — the one thing chrome is meant not to do. One footer height
and one motif size now; `--ac-slide-foot-lg` becomes an alias of
`--ac-slide-foot` and is marked deprecated. The logo heights still differ by
surface, which is deliberate: the wordmark and logo SVGs have different crops,
so equal `height` renders unequal marks.

**New in the stylesheet:** `.ac-slide__standing` (the footer's standing line, in
Poppins — never the wordmark typeface, which reads as a watermark in chrome),
`.ac-slide__note` (a footnote above the footer, so a caveat stops displacing the
standing line), `.ac-slide--end` (mirrors the motif on the final slide as an end
mark), and `.ac-slide__heading--step` (drops one wrapping heading a step instead
of retuning the whole deck's ladder). `.ac-slide__mark` gains a fixed
`min-width` so the marker does not shift at the tenth slide;
`.ac-slide__grid` gains `justify-items: start`.

**Three decisions the notes left open, settled here.**

*The type floor* binds prose only. Card metadata — a category, a country, a role
in a dense grid — is exempt down to 13px, because it is scanned rather than read
and the alternative is carrying fewer items than the content needs. Slide 07 of
the template is the case it exists for.

*The heading-weight conflict.* The notes said titles and headings alike are never
bold; Ara's instruction the same day was that section headings are bold. They are
about different classes, and both stand: `.ac-slide__title` at 105px stays
regular 400, `.ac-slide__heading` at 71px is bold 700. A 105px display line does
not need weight to dominate; a 71px heading over a subhead does.

*The footer wordmark.* The notes warn that a wordmark in chrome reads as a
watermark. That is an objection to the Monument Extended *typeface*, not to the
logo asset, so the logo SVG stays bottom-left and the standing line is Poppins.

**The sibling-consistency rule, applied as well as written.** On slide 10 — the
Values slide Ara commented on — the four descriptions ran 3 / 2 / 3 / 3 lines.
The two-line one is now three, matching its siblings up rather than cutting them
down.

Slides 07 and 09 could not be fixed that way, and the document now records why:
slide 09's paragraphs are **attributed client testimonials**, where padding a
line means putting invented words in a named person's mouth, and slide 07's
uneven lines are project **facts** — categories and countries, where matching
line counts means inventing geography. Both got the structural fix instead:
`justify-content: space-between` on the project card bodies, at a fixed 113px
height sized for the worst case rather than a min-height — a min-height still
grew with each card’s own copy, and the `flex:1` media block above absorbed the
difference, leaving the twelve grey placeholders at three distinct heights inside
twelve identical cards. And
`margin-top: auto` on the testimonial quotes, so uneven copy sits at a
consistent depth across siblings. The rule in the document now carves out quoted
and factual copy explicitly.

**The rule is about consistency, not brevity** — Vishnu’s
correction. Where the notes said every list line must fit on one line, the rule
is now that sibling items occupy the same number of lines, whatever that number
is: if one item needs two, give them all two. Matching the others up is the
repair, not cutting the long one down.

**The template was then reconciled with the chrome rules.** The three gold
slides dropped from a 126px footer to 90px, their chip wordmark from 60px to
46px to sit in the shorter band, and their 115×60 motif to the same 44×23 every
canvas slide uses. All ten page markers gained `min-width:154px` and right
alignment.

Two things fell out of that. Five slides were re-pasting the chevron vector
inline — the three gold ones plus slides 02 and 03, which the v1.6.0 dedup had
missed — so all ten footers now `<use>` the single `<defs>` copy, and the file
is **7.5KB smaller**. And slides 02 and 03 carried `viewBox="0 0 44 23"` on a
vector authored at 208.74×109.1, scaled back down by a `scale(0.2108)` on each
path; consistent, but it meant the motif could not be edited in one place.

The slides still use inline styles rather than the classes in `deck.css`,
because a Design Component styles inline by format. `docs/deck-conventions.md`
now says that plainly: the classes are enforcement for pages built on
`deck.css`, and documentation for the template.

## v1.12.0 — 1 September 2026

### `ui_kits/` is gone. The website kit merged into `acds-template-web`.

The last kit was a six-screen click-through of aracreate.group. In content it
was not a duplicate of the template — its About screen carried the founder
card, the eight group companies as a table and the ventures grid, and none of
that existed anywhere else. But it was still a second description of the same
website, which is what the consolidation has been removing all day.

**`acds-template-web` is now three pages**, switched by the nav or by a new
`page` tweak:

| Page | What is on it |
| --- | --- |
| home | the fourteen bands, one scroll, anchors for services, projects, journal, contact |
| about | breadcrumb, split hero, four-stat graphite band, founder split with the values list, group-companies table, ventures grid |
| 404 | oversized code, two ways back |

The kit's Services, Projects and Contact screens did **not** become pages: their
content is already bands on the home page, which is how a site this size is
built. The About content is transcribed unchanged, including the eight legal
company names.

**What was lost, and where to test it instead.** The kit's Contact screen was
the only place that demonstrated `Modal`, `Toast`, `Dropdown`, `Tooltip`,
`TagGroup` and `ProgressBar` composed together. Those components are unchanged
and still have their own cards under `components/feedback/` and
`components/overlays/`; `docs/screen-reader-pass.md` section 1 now points at the
template's contact band for the form path and names the cards for the overlay
path.

**One bug the merge exposed.** `page` reads `this.state.page ?? this.props.page`,
not the other way round: a declared prop default arrives on every render, so
reading props first made every nav click a silent no-op. The state field starts
`null` so the tweak still works as the initial value.

**Nine files deleted.** `ui_kits/` no longer exists — the deck kit became
`templates/deck/`, the web-app kit `templates/app/`, this one `templates/web/`,
and the Academy kit was deleted at the client's request. `tests/checks.html`,
`SKILL.md`, `readme.md`, `github.md`, `docs/live-site.md` and
`docs/contributing.md` no longer name the folder. The Design System tab loses
its "Website" group, one card.

## v1.11.0 — 1 September 2026

### The gold shortfall was eleven components, not one button

v1.4.0 retired brand black and pointed `--ac-text-on-accent` at graphite. That
single edit moved **every** component with a full Golden Sun fill from 9.51:1 to
4.45:1 — and v1.4.0's own note, repeated into `readme.md`,
`tokens/colors.css` and `docs/accessibility.md`, described it as "the primary
button's label". It was the primary button *and ten others*, several of them
11–14px state indicators: which segmented chip is selected, how many rows are
picked, which day is chosen, which step is done. The large-text reading that
excuses a 105px gold title was never available to any of them.

**One rule, applied to every accent fill:** a Golden Sun fill that carries text
below 19px takes **Golden Soft** `#fdf3d8` (graphite 6.63:1, passes at every
size). A fill that carries no text keeps Golden Sun.

Moved: `.ac-badge--accent`, `.ac-avatar--accent`, `.ac-card--accent`,
`.ac-bulk-bar`, `.ac-segmented input:checked + span`,
`.ac-pagination__link[aria-current]`, `.ac-datepicker__day[aria-selected]`,
`.ac-steps__marker` on a done step, and `::selection` — joining
`.ac-banner--accent`, moved in v1.9.0 for the same reason. The done step
marker's border goes to `--ac-accent-active` so it still reads as a filled
marker against the softer fill.

Unchanged: ticks, switches, slider tracks, bars, rules, chart swatches and
progress fills, none of which carry text; and `.ac-surface-accent`,
`.ac-band--accent` and `.ac-slide--gold`, the band classes whose text is display
sized by rule.

**The primary button is now genuinely the only one**, and it stays parked: the
repair is a graphite-filled primary button with a white label (7.44:1), which is
a brand decision for Vishnu, not an accessibility fix. All four places that
describe the shortfall now say "the last such case" rather than implying it was
always the only one.

## v1.10.0 — 1 September 2026

### The weight scale, applied everywhere it should have been in v1.5.0

v1.5.0 wrote Ara's Poppins scale into `tokens/typography.css` and applied it to
the deck template and to `strong`. It never swept the component stylesheets, so
pre-scale weights survived in the components every page actually uses — the app
template put four of them on screen at once. Every use of `extralight` and
`semibold` in `styles/` is now reconciled by role.

**Numbers are medium.** `.ac-card__number`, `.ac-stat__value` and
`.ac-price__figure` were **extralight 200** — the thinnest weight in the scale
on the system's largest figures. Ara's original note was "numbers /
highlightened text weight : medium", so all three are now 500. This is the same
correction the deck's stat figures got in v1.5.0, on the classes the app and
marketing pages use.

**Display lines are regular.** `.ac-hero__title` (up to 52px) and
`.ac-404__code` (up to 180px) move from 200 to **400**, matching the deck's
88–105px titles.

**600 is gone from the stylesheets.** Fourteen selectors still used it. Uppercase
micro-labels and active states — `.ac-card__eyebrow`, `.ac-footer__title`,
`.ac-nav-list__label`, `.ac-slide__eyebrow`, the signature caption, the
segmented and date-picker selected states, the combobox match highlight, the
inline link — go to **medium 500**: at 11px with wide tracking, the scale's
label weight of 300 is too thin to read, and emphasis is the honest slot for
them. `.ac-slide-block__title` is a heading and goes to **700**.
`--ac-weight-semibold` stays defined for consumer code and is marked retired.

**The deck page number** (`.ac-slide__mark`, 14px) moves 200 → **300**: it is a
caption, not a paragraph.

**One left for Ara.** `.ac-toolbar__title` renders the app's page heading
("Overview", "Jobs", "Settings") at **light 300**. Under the scale a heading is
700, but a toolbar title is arguably chrome rather than a section heading, and
at 21px it is not obviously wrong. Unchanged pending her call.

## v1.9.0 — 1 September 2026

### `acds-template-app` — the app is a template now, not a kit

`ui_kits/web_app/` held four screens and a shell that nothing else in the
project could start from: a kit is something you read, a template is something
you copy. `templates/app/App.dc.html` is that same app as
**`acds-template-app`**, and the kit is deleted — six files.

It was not merged into `acds-template-web`, which was the original request. The
two share the token layer and every component but no layout: a marketing page is
a vertical scroll of full-bleed bands with no state, the app is a fixed shell
with routed screens and theme and density switches on `<html>`. Merged, every
consumer's first act would be deleting the half they did not want. Two templates
give the same deduplication without that.

**What moved.** The shell — `.ac-app` grid, side nav with counts, collapsing
64px rail, top bar, dismissible banner — is now **static markup in the
template**, so nav items, brand, banner copy and bar buttons are directly
editable rather than React props. The four screens stay `.jsx` and are
`<x-import>`ed: `SignIn`, `Dashboard`, `Jobs`, `Settings` (renamed from
`*Screen`). That is the split the DC format wants — chrome as markup, component
demos as imports.

**Two tweaks:** `showBanner` drops the banner and gives the shell full viewport
height; `startSignedIn: always-out` lands on the sign-in screen, which is
otherwise skipped.

**Two bugs found while porting the shell to markup.** The banner used
`.ac-banner__inner` and `.ac-banner__text`, neither of which exists — the real
component is `.ac-banner__body` with the title as an inline `<span>`, and
without it the title ran straight into the text with no space. And the nav
needed `hidden` when closed under 992px, where `.ac-app__nav` is
`position: fixed`; without it the nav covered the page at tablet width. The
logic class now computes `navHidden` from a resize listener and the hamburger
switches meaning — rail toggle when wide, open/close when narrow. This is the
third instance of that same class of bug, after `Header.jsx` in v1.7.0.

**Two rules this template broke on arrival, both fixed at the source.**

The maintenance banner is `.ac-banner--accent`, which was full Golden Sun with
a 14px title and 14px message — graphite at 4.45:1, exactly what v1.7.0 wrote
into `docs/accessibility.md` two entries earlier. A banner is body-size text by
definition, so the fix belongs in `styles/app.css` rather than in this page:
`.ac-banner--accent` now takes **Golden Soft** `#fdf3d8` (6.63:1, passes at
every size). Golden Sun remains the accent surface for bands whose text is
large. Every consumer of the accent banner gets the repair.

The sign-in screen was copied verbatim from the kit and had never met Ara's
Poppins scale from v1.5.0: its 36px `h1` was weight 200 (body-paragraph weight
on a display line) and its 28px `h2` was 300. Now 400 and 700. `Dashboard`,
`Jobs` and `Settings` carry no inline weights and needed nothing.

**One real defect the port introduced, worth recording.** The kit loaded
`_ds_bundle.js` as a blocking `<script>` and its screens with `defer` after it,
so order was guaranteed. `ds-base.js` injects the bundle dynamically, which
makes it async, and `<x-import>` fetches each screen as soon as the template
parses — so a module-scope `const { Toolbar, … } = window.<Namespace>` races the
bundle and throws whenever it loses. All four screens now read the namespace
inside the component body, and `Jobs`'s `STATUS` map (which builds `<Badge>`
elements) moved in with them. That alone was not enough — React calls a
component the moment x-import mounts it, which is still before the bundle
executes, so the destructure threw on first render and the dc-runtime's retry
hid it behind two console errors and an error-boundary flash. The deterministic
fix is a `dsReady` gate in the logic class: `componentDidMount` polls for the
namespace at 30ms and the four `sc-if` conditions require it, so a screen is
never called before the bundle exists. `s.async = false` was added to all three
templates' `ds-base.js` too, though it only orders the injected scripts against
each other, not against x-import.

**Also updated:** `tests/checks.html` drops the path, and
`docs/screen-reader-pass.md` section 2 points at the template instead of the
kit. `ui_kits/` is down to one kit, the group website.

## v1.8.0 — 1 September 2026

### The araCreate Academy kit is removed

`ui_kits/academy/` — `index.html`, `CoursesScreen.jsx`, `TutorsScreen.jsx`,
`ApplyScreen.jsx` and its `README.md` — is deleted at Vishnu's request. Five
files.

The kit was a click-through recreation of the Academy's course marketing, built
from the same components as the group website because the Academy has no
separate visual identity. Nothing in the system depended on it: no component,
stylesheet or template referenced it, and the three screens it put on
`window.<Namespace>` were not imported anywhere.

**Two consequences.** The Design System tab loses its "Academy" group, one card.
And `docs/screen-reader-pass.md` named `ui_kits/academy/index.html` as its form
path — "the richest form in the system". That pass now points at
`ui_kits/website/index.html` → Contact, which carries the same controls: live
validation, a select, a choice group, an alert and a polite toast. Its section 4
also still named `ui_kits/deck/index.html`, removed in v1.6.0, and now points at
`templates/deck/Deck.dc.html`.

`tests/checks.html` drops the path. The Academy remains listed in
`docs/brand-facts.md` and `readme.md` as one of the group's properties and an
intended consumer of this system — that is a fact about araCreate, not a file
reference.

## v1.7.0 — 1 September 2026

### One web template. `acds-template-web`.

There were three descriptions of the same website in this project: the
`marketing-page/` template, the click-through kit in `ui_kits/website/`, and
eight loose section files inside that kit — `SiteHero`, `SiteFooter`, `Navbar`,
`TrustedBy`, `About`, `Services`, `Projects`, `Contact` — which nothing loaded
and which the kit's own README did not list. Three copies of a page is three
places to forget.

**`templates/web/Web.dc.html`** is now the single web template, named
`acds-template-web`. It carries every band the site has, in page order: sticky
header with a working mobile toggle, split hero, marquee, trusted-by logos,
service list, graphite stats band, values grid, gold ecosystem table, projects
grid, testimonial, journal listing, contact details with a real form, gold CTA
and footer. Two tweaks — `showTrustedBy` and `showJournal` — drop the two bands
a new page rarely has copy for.

**The marketing-page template was broken, and this was why.** Its `ds-base.js`
linked `styles.css`, which imports the four token files and nothing else, so
every `.ac-*` class in the template was unstyled: the page rendered as bare
browser HTML with a logo in it. The new template links `system.css`, the whole
system, and `ds-base.js` says so in a comment so the next copy does not repeat
it.

**`Header` had the same class of bug.** `hidden={!open ? undefined : undefined}`
evaluates to `undefined` either way, so the mobile nav panel was never hidden
and the links sat stacked down the page under 992px. Now `hidden={open ?
undefined : true}`.

**Small text never sits on Golden Sun.** The first draft put the ecosystem
table (14px cells) and a 12px eyebrow on a Golden Sun band, where graphite
measures 4.45:1 — outside the large-text exception v1.4.0 recorded. The band is
now Golden Soft `#fdf3d8` (6.63:1, passes at every size), and the gold CTA band
lost its 14px contact line, so gold carries only the 31px title and the button.
The rule is written into `docs/accessibility.md`.

**One shortfall this exposed, recorded rather than papered over.** `.ac-btn`
takes its label colour from `--ac-text-on-accent`, so v1.4.0 moved every gold
button in the system to graphite on gold: 4.45:1 at 12–14px/500, outside the
large-text rule. Graphite is now the darkest colour in the system, so nothing
inside the palette repairs it. The alternative — a graphite-filled primary
button with a white label at 7.44:1 — is a brand decision and is not being made
unilaterally. `styles/components.css`, `tokens/colors.css`, `readme.md` and
`docs/accessibility.md` all state it in the same words.

**Removed:** `templates/marketing-page/` and the eight unreferenced section
files in `ui_kits/website/`.

**Kept:** the click-through kit at `ui_kits/website/` — its six `*Screen.jsx`
screens, the nav that switches them and the live-validating contact form. That
is a working demo of the components, not a second template.

## v1.6.0 — 1 September 2026

### One deck. `ui_kits/deck/` is gone.

The project carried three parallel deck systems: this template, a React slide
library (`ui_kits/deck/slides.jsx` plus a navigable `index.html` and three
`card-*.html` specimens), and ten `*.slide.html` cards built on a third
vocabulary, the `.ac-slide` classes. Six slide types existed twice, styled
differently in each place. `templates/deck/Deck.dc.html` is now the only source.

**Merged into the template** — the five layouts that existed nowhere else, in
approved geometry and on the Poppins scale:

| # | Slide | Came from |
| --- | --- | --- |
| 06 | Service vertical — blurb + service domains | `VerticalSlide` |
| 07 | Projects — twelve dashed cards, six across | `ProjectGridSlide` |
| 08 | Case study — impact prose + eight figures | `CaseStudySlide` |
| 09 | Testimonials — three dashed quote cards | `TestimonialsSlide` |
| 10 | Values — two-by-two with a gold rule | `ValuesSlide` / `values.slide.html` |

Content is transcribed unchanged from the source deck. Stat figures land at
medium 500 per v1.5.0 rather than the 600 the JSX used.

**Removed as duplicates** — 16 files. `title`, `statement`, `statistics`,
`section-opener` and `closing` duplicated template slides 01, 04, 03, 04 and 05.
`ecosystem`, `verticals`, `photo-split` and `split-art` were `.ac-slide`
restatements of the two-column and three-column structures slides 06–10 now
carry. `slides.jsx`, `index.html`, the three `card-*.html` files and the kit
`README.md` went with them.

**Two consequences worth stating.** The Design System tab loses its "Slides"
group — 14 cards — because every card in it lived in that folder; the deck is
now reached as the `acds-template-deck` template. And `window.<Namespace>`
loses the ten slide components `slides.jsx` exported (`SlideFrame`, `Header`,
`Chevrons`, `TitleSlide` and the rest). Nothing in the project imported them.

**Chevron dedup.** The triple-chevron vector was pasted inline on every slide,
about 2.8KB each. It is now one `<g id="ac-chev">` in a `<defs>` block at the
top of the template, referenced by three `<use>` elements per footer. Slides
01–05 keep their original inline copies; 06–10 use the reference.

**Also updated:** `tests/checks.html` drops the 14 deleted paths, and the
"locked / pinned by decision" notes in `readme.md` and `github.md` no longer
list `ui_kits/deck/`. `templates/deck/README.md` is rewritten as the single
source of record, with the ten-slide table and the geometry rules.

## v1.5.0 — 1 September 2026

### The Poppins weight scale, set by Ara

Ara reviewed the deck template and named a weight for each class of type. The
scale is now written into `tokens/typography.css` rather than living as inline
values on one deck. **It governs Poppins.** Poppins does all type work here, so
in practice that is everything you set — but Monument Extended is logo-only and
keeps its own weight (`.ac-wordmark`), off this ladder, as does the mono stack:

| Weight | Job |
| --- | --- |
| 200 extralight | body paragraphs |
| 300 light | subheadings, captions, stat labels |
| 500 medium | emphasis (`strong`/`b`) and numbers or highlighted text |
| 700 bold | section headings |

400 regular is Poppins display type only — the 88–105px title slides. **600 semibold is
retired for emphasis**: at 600, Poppins beside 200/300 body reads as a different
typeface, so `strong, b` in `styles/base.css` and `.ac-slide__prose strong` in
`styles/deck.css` both move to medium. The token stays defined and is marked
legacy.

**In the deck template.** The seven stat figures on slide 3 move from 700 to
500, joining the `300+` figure Ara had already set by hand. Headings (700),
subheadings (300), paragraphs (200) and the inline `strong` (500) were already
at the new values from Ara's own edits and are unchanged.

**Body copy outside decks is unaffected** — documents and app screens stay at
light 300. Extralight 200 is a deck voice: at 29px on a 1280px slide the
thinness is deliberate, and the note in `typography.css` says so.

## v1.4.0 — 1 September 2026

### Brand black is retired. Graphite Gray is the only neutral.

v1.3.0 moved headings to graphite and left brand black `#222222` holding the
strong border and the focus ring. That was a colour kept for two jobs graphite
already does. It is now out of the palette: araCreate has two colours, Golden
Sun and Graphite Gray, and the system says so without a third near-black.

**Retired as values, kept as aliases.** `--ac-black`, `--ac-gray-900` and
`--ac-gray-700` all resolve to `--ac-graphite-gray` (`#555555`). Nothing a
consumer references breaks; everything referencing them gets graphite. Do not
use them in new work.

**Moved with it:**

| Token | Was | Is |
| --- | --- | --- |
| `--ac-focus` | `#222222` | graphite — 6.90:1 on canvas |
| `--ac-surface-dark` | `#222222` | graphite — the footer, section openers, table headers, tooltips, code blocks, the app side nav and the featured price card |
| `--ac-text-on-accent` | `#222222` | graphite |
| `--ac-border-strong` | graphite already | unchanged |

**The third accessibility exception closes.** `--ac-text-on-accent` was held at
brand black because graphite on Golden Sun measures 4.45:1, 0.05 under 4.5:1.
With black gone the exception has nothing to point at, so the shortfall is
accepted deliberately and recorded here: set text on a gold band at 19px/600 or
larger, where the requirement is 3:1 and graphite passes with room. Two
exceptions remain — `--ac-focus` and `--ac-success`.

**The shadows were still black.** `tokens/spacing.css` claimed in a comment
that every shadow was graphite while the values were `rgba(0,0,0)` and one
stray `rgba(8,8,8)`. All four shadow tokens are now `rgba(85,85,85)`, each alpha
multiplied by 255/170 so they composite to the grey black produced.

**Every remaining black surface.** The gold-rule card's two black chips are
graphite and carry graphite's real figures (4.45:1 on gold, 4.58:1 for gold on
graphite) rather than black's 9.51:1 and 8.11:1. The logos card swaps its black
cell for white. The deck template's desk is graphite, not `#111`. Component
prose in `Chevrons`, `Footer`, `Section` and `AppShell` no longer says "brand
black".

**Chart keys.** `.ac-chart__key:nth-child(4)` read `--ac-gray-900`, which now
resolves to the same graphite as key 2. It moves to `--ac-gray-450` (`#6f6f6f`)
so four series stay distinguishable.

**Cards and specimens updated.** The neutrals card drops its Ink chip and now
reads five columns; the graphite ramp starts at 600 rather than 900 and 700; the
monogram card sits the negative icon on graphite instead of black and moves the
fourth cell to canvas; the project thumbnail's swatch strip loses its black
band. `SiteFooter` reads `--ac-surface-dark`, the deck template's fallback hex
is `#555555`, and `readme.md`, `SKILL.md` and `docs/accessibility.md` state the
palette as two colours.

Historical notes in `docs/` and earlier changelog entries still describe
`#222222` as the brand black. They are records of what was true then and are
left as written.

## v1.3.0 — 1 September 2026

### Graphite Gray is the text colour, headings included

`--ac-text-heading` moves from `--ac-gray-900` (brand black `#222222`) to
`--ac-gray-600` (Graphite Gray `#555555`). The page now has one text colour
across every level — heading, body, `strong`, table header, stat figure, button
label, breadcrumb, tab — rather than a black tier sitting above a graphite one.
Graphite measures 6.90:1 on canvas and 7.44:1 on white, so nothing drops below
the bar.

Because every rule in `styles/` and every component reads the semantic token,
this is one value change. `--ac-code-text` follows `--ac-text-heading` and moves
with it.

**Brand black's role narrows again.** `--ac-black` is now the strong border and
the focus ring, and nothing else. It is no longer a text colour and was already
not a fill. `--ac-gray-900`'s comment changes to match.

**Dark theme.** Inside the dark theme the inverse band is the LIGHT band, so its
heading, inverse and base text values move to graphite alongside the light
theme. The dark page's own ramp (`#ffffff` headings, `#d8d8d8` body) is measured
against graphite and does not change.

**Kept as brand black:** `--ac-text-on-accent`. Graphite on Golden Sun measures
4.45:1 and misses the bar by 0.05 — the exception recorded in v1.0.0 stands, so
text on a gold band stays `#222222`.

Also updated: three `ui_kits/website` files and two colour foundation cards that
named `--ac-black` directly for text now read `--ac-text-heading`; `SKILL.md`'s
quick reference states the narrowed roles.

## v1.2.0 — 24 August 2026 — versioning, and the repository record corrected

### The repository exists

`github.md`, `readme.md` and `CLAUDE.md` all stated that
`aracreate-group/aracreate-design-system` had never existed, on the strength of
a 404 and an empty search on 23 August. The repository is **private**, and a
private repo returns 404 to any unauthorized caller. Read directly on 24 August:
177 files, 166 of them under `src/claude-design-system/`. All three files are
corrected.

**Direction of truth settled.** ACDS here is authoritative; the repository is
backup and audit trail. Changes flow from Claude Design to the repository via
export, never back. The repository's own `README.md` and `src/readme.md` still
assert the reverse and need editing there — recorded in `github.md` as
outstanding.

No commit sha is on record: the tree read resolves to a tree hash, not a commit.

### Sync, 24 August 2026 10:10 UTC

The remote was compared against this tree: no upstream change exists. The
repository carries **v1.1.0** — its `changelog.md` ends at the 23 August entry,
and `tokens/colors.css`, `CLAUDE.md`, `styles.css` and `system.css` are
byte-identical here. Nothing was imported and no screen was rebuilt. The
subtree holds 393 files, not the 166 previously recorded, which was a capped
listing. Details in [`github.md`](github.md).

### This changelog is versioned

Entries now carry version numbers, indexed above with their repository state,
and `readme.md` states the current version. Previously the only handle on a
given state of the system was a date, and four separate changes share
20 August 2026.

---

## v1.1.0 — 23 August 2026 — muted text passes, four dead tokens removed

Two colour decisions taken after a full audit of the token layer and every raw
colour outside it.

### Muted text now passes contrast

`--ac-text-muted` was ACDS's `#8a8a8a` at 3.19:1 on canvas, carried as a
knowingly-kept failure since the merge. It now points at `--ac-gray-450`
(`#6f6f6f`), the derived accessible grey already declared in the same file.

| Muted text on | Ratio | |
| --- | --- | --- |
| `--ac-surface-page` (`#f6f6f6`) | **4.65:1** | passes |
| `--ac-surface-card` (`#ffffff`) | **5.02:1** | passes |
| `--ac-surface-accent` (`#f9bf3b`) | **9.51:1** | passes — every gold surface re-points muted to `--ac-gray-900` |

The gold figure is the one worth reading twice. `#6f6f6f` on Golden Sun measures
3.00:1 and would fail, but that pairing does not render: `.ac-surface-accent`
(base.css), the gold bulk bar and `.ac-banner--accent` (app.css) and both
accent blocks in theme-dark.css all re-point `--ac-text-muted` to ink. The
surface-context pattern is what makes the new value safe there — it was already
carrying the old value, which measured 2.06:1 on gold.

The dark theme's own `--ac-text-muted` (`#cecece`, 4.68:1 on the graphite page)
is untouched. Different value, different background, already passing.

### Four dead tokens removed

`--ac-photo-overlay` (`#2e419e`), `--ac-true-black` (`#000000`),
`--ac-gray-translucent` (`rgba(46,46,46,.5)`) and `--ac-button-gray-light`
(`rgba(47,53,69,.6)`) are gone from `tokens/colors.css`.

All four arrived in the 17 August Webflow reconciliation because they existed as
live Webflow variables. **None was ever referenced** — not by a stylesheet, a
component, a card or a template, verified by grepping the whole project for each
name. Each is also outside the palette: a navy, a blue-grey, a grey off the
ramp, and a pure black in a system whose black is `#222222`.

`foundations/colors-photo-wash.card.html` displayed the navy and nothing else,
so the card is deleted and its line removed from `tests/checks.html`.
`docs/assets.md` keeps the wash colour as a fact about how the supplied
photography was produced, which is what it always was.

## v1.0.0 — 21 August 2026 — merged into ACDS

This project has been folded into **ACDS**, the araCreate Design System of
17 August 2026. ACDS is the base. Where the two systems disagreed on structure,
file naming or a token value, **ACDS won** — with three deliberate exceptions
and one knowingly-kept failure, all four recorded below rather than left in a
commit message.

### What came across from the 20 August system

- **78 components across 57 files in 8 groups** — core, forms, feedback,
  navigation, data, signature, sections, app. Plus `content/`, which is not a
  ninth group: it is the ACDS compatibility layer, old names that still import
  as thin wrappers over the merged component named in each `.prompt.md`.
- **Ten stylesheets.** `base`, `signature`, `components`, `sections`, `app` and
  `deck` now live in `styles/`; `density` and `theme-dark` in `tokens/`. The
  single `tokens.css` the port arrived with was split four ways —
  `fonts` / `colors` / `typography` / `spacing` — to follow ACDS's
  four-files-by-kind token layer. The split changed no value and no name.
- **The dark theme.** `data-ac-theme="dark"` and `"auto"`, one file, no
  component touched.
- **The density scale and the 44px target floor.** Comfortable by default,
  `data-ac-density="compact"` scoped to `pointer: fine`, and every interactive
  control at least 44px in both directions.
- **Thirty specimen cards** in `foundations/`.
- **Four UI kits** — website, academy, web_app, deck.
- **A browser test gate** — `tests/checks.html`, contrast and accessibility,
  globbing rather than listing.
- **Eleven documents** in `docs/`.

### Value reversions to ACDS

Every one of these is the 20 August value being replaced by the ACDS value.

- **Radii restored.** `--ac-radius-xs` 2px, `--ac-radius-sm` 4px,
  `--ac-radius-md` 9px, `--ac-radius-lg` 20px, `--ac-radius-pill` 200px,
  `--ac-radius-none` 0. The 20 August system had set every radius to 0.
  **Anything carrying the signature edge stays square**, because
  `--ac-edge-radius` governs those and ACDS has no such device — so the two
  decisions do not collide.
- **Four shadows**, including a resting `--ac-shadow-sm`. The 20 August system
  had three and no resting shadow.
- **Three durations** — `.25s` / `.5s` / `.8s`, down from four. This is a value
  change as well as a rename and cannot be otherwise: 400ms and 500ms both land
  on `--ac-duration`.
- **`--ac-surface-dark` back to `#222222`** from graphite. The dark band is
  brand black again.
- **`--ac-text-muted` back to `#8a8a8a`.** See the kept failure below —
  **no longer kept as of 23 August 2026**, see the entry at the top of this file.
- **`--ac-danger` `#c0492f`** — 4.59:1 on canvas.
- **`--ac-info` graphite** `#555555`.
- **`--ac-border-strong` graphite** `#555555`.
- **`--ac-text-sm` and `--ac-text-link` both 14px.**
- **`--ac-leading-body` 1.8.**
- **`--ac-tracking-label` `0.07em`.**

### Three deliberate exceptions — ACDS's value was not taken

On the client's instruction, each an accessibility repair rather than a
preference. These are the only three places the merge overrode the base.

| Token | ACDS value | Value here | Measured |
| --- | --- | --- | --- |
| `--ac-focus` | Golden Sun | brand black `#222222` | Gold measures **1.55:1** on canvas. A focus ring nobody can see is not a focus ring. |
| `--ac-success` | `#4f9d69` | `#186a43` | `#4f9d69` measures **3.06:1**; `#186a43` measures 6.11:1 on canvas. |
| `--ac-text-on-accent` | Graphite Gray | brand black `#222222` | Graphite on gold measures **4.45:1** — it misses 4.5:1 by 0.05. |

### One knowingly-kept failure — REVERSED 23 August 2026

`--ac-text-muted` was ACDS's `#8a8a8a`, measuring **3.19:1 on canvas** — below
the 4.5:1 that normal text requires. It was **kept at the client's explicit
instruction**, and written down here so that nobody would find it in an audit
and assume it was missed.

That instruction was withdrawn on 23 August 2026 and the failure is repaired:
`--ac-text-muted` is now `var(--ac-gray-450)` (`#6f6f6f`, 4.65:1). The reversal
was the one line this entry predicted, and the prediction that nothing else
needed touching held — no component holds a raw colour. Only prose and one
swatch card stated the old figure.

### Spacing

ACDS's **indexed names** won: `--ac-space-1` … `--ac-space-16`, carrying the
sixteen values the live site actually measures (4, 8, 10, 12, 15, 16, 20, 24,
30, 40, 60, 80, 90, 100, 120, 140).

The names came from ACDS and the values from the audit because **nothing
consumed ACDS's declared ten-rung ladder** — verified against the compiled
bundle and against every source file before the decision was taken. Keeping
ACDS's ten values would have broken every measured page; keeping the 20 August
system's value-named tokens would have broken ACDS's naming convention. Nothing
was consuming the one, so the other survives.

### Naming

**American spelling in identifiers** (`gray`, `color`, `center`), **British
spelling in prose**. `--ac-graphite-gray` is the token; grey is the colour.
Foundation cards renamed accordingly — `colour-*.card.html` became
`colors-*.card.html`.

### Locked and untouched

Pinned by decision, not merged and not rewritten:

- `templates/deck/` — `Deck.dc.html`, `ds-base.js`, `support.js`
- `foundations/brand-icons.card.html`
- `styles.css`

### styles.css and system.css

**`styles.css` still means what it always meant** — the four token imports and
nothing else — because the locked files link it. It is not to be widened.

**The full system is `system.css`**: the four token files plus `base`,
`signature`, `components`, `sections`, `app`, `density`, `theme-dark` and
`deck`. New consumers link `system.css`; a page that links `styles.css` gets
tokens and no component CSS, which is the intended behaviour.

### Open after this merge — all closed, 23 August 2026

This section previously listed three defects. **None of them were real.** They
described the state of a scratch build folder, not this tree, and they were
repeated here without being checked. Verified on the tree itself:

- `styles.css` — **present.** `foundations/brand-icons.card.html` — **present.**
- `assets/` — **present**, 69 files across `logos`, `icons`, `illustrations`,
  `imagery`, `brand` and `fonts`. `tokens/fonts.css` and `thumbnail.html` point
  into it correctly.
- `tests/checks.html` — **65 referenced paths, 65 resolve, 0 broken.** No
  `guidelines/*` or `ui_kits/group_website/` names remain anywhere in the tree.
  The gate works.

### Genuinely open

- **The spacing ladder.** `--ac-space-3` … `--ac-space-10` carry the
  site-measured values, not ACDS's, under ACDS's names. Conversion table at the
  head of `tokens/spacing.css`. Nothing in the tree is broken by this; code
  written against pre-21-August ACDS is.
- **249 merge-added files sit in the published ACDS project** and cannot be
  removed from it — bulk delete requires project ownership.
- **No remote repository exists.** Local git only, commit `12c7b85`, no push.
  See `github.md`.

---

## v0.5.2 — 20 August 2026 — the graphite surfaces, corrected

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

## v0.5.1 — 20 August 2026 — token-layer housekeeping

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

## v0.5.0 — 20 August 2026 — black leaves the palette

Requested: no pure black anywhere, and no black backgrounds. Graphite `#555555`
replaces both. Nothing outside the token layer changed, because no component
holds a raw colour.

**Retired.** `--ac-black` (`#000000`), `--ac-gray-700` (`#2e2e2e`) and
`--ac-gray-700` (`#383838`) are now aliases, not values — every consumer of
them keeps working and gets graphite. `--ac-brand-black` is renamed
**`--ac-black`**, kept as an alias, and its role is narrowed in writing: text,
strong borders and the focus ring. Never a fill. The old name was inviting the
exact use that is no longer allowed.

**The dark band is graphite.** `--ac-surface-dark` moves from `#222222` to
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
danger mixes toward graphite (`--ac-danger-bg-hover`). And the dark slides and
sign-in screen now carry `aracreate-logo-t-w-b-g.svg` rather than the gold-chip
lockup — per `guidelines/assets.md`, the graphite chip is the one that vanishes
into a graphite band.

## v0.4.1 — 20 August 2026 — the gate's first eleven findings, triaged

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

## v0.4.0 — 20 August 2026 — phases 3 and 4: documentation, audit, governance

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

## v0.3.0 — 20 August 2026 — phase 2: application surfaces

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
(`.ac-nav-item` asks for `--ac-text-primary`, the current one for
`--ac-text-heading`) kept resolving against the light page.

`.ac-banner` and `.ac-bulk-bar` had the same fault. All three now re-point the
full token set, the way `base.css` does for `.ac-surface-inverse`.

**And a second-order bug inside that fix.** Pinning `--ac-text-heading` to white
is correct in the light theme and wrong in the dark one, where the inverse
surface flips to light — white-on-canvas for the current item. Both text tokens
now follow `--ac-text-inverse`, which is right in both themes; the current
item keeps its emphasis from weight, tint and the gold rule instead of a second
colour. The alpha-based values (`#ffffffb3` muted, `#ffffff33` dividers) cannot
follow a token, so `theme-dark.css` overrides them.

Measuring that override found one more: `--ac-gray-450` is the lightest grey
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

## v0.2.0 — 20 August 2026 — phase 1: theme, density, targets

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

- **`--ac-text-link`: 12px → 15px.** The one locked deviation of three that was
  unlocked, at the client's request. Two separate problems lived in that number
  and both are now fixed: readability (15px, matching `--ac-text-sm`, still
  visibly quieter than 16px body copy) and target size (a 44px hit area from
  `density.css`).

  Navigation, footer and breadcrumb links also moved off `--ac-text-xs`
  onto `--ac-text-link`. They were 12px because a caption is 12px, but they are
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

## v0.1.0 — 20 August 2026 — initial port

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
