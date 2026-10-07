# COMPONENT REFERENCE

All 78 components, what each is for, and — more usefully — what each is *not*
for. A component without a "do not use it for" note gets misused; that rule comes
from [`guidance.md`](guidance.md) and applies to everything added since.

`guidance.md` covers the CSS classes and is araCreate's own text. This file
covers the React API layered over them, including the twenty-four application
components that have no counterpart in the source repository.

**Maturity.** Every component is one of three things:

| Status | Meaning |
| --- | --- |
| **Stable** | Ported from `components.css` / `sections.css` / `signature.css`. The values are audited from the live site and reviewed. |
| **New** | Authored for this project, 20 August 2026, in the araCreate idiom. Rendered, measured and screenshotted, but not yet used in production. |
| **Frame** | Deliberately incomplete — supplies the furniture and expects you to supply the content. |

---

## core — 16 components · Stable

| Component | Use for | Do NOT use for | States |
| --- | --- | --- | --- |
| `Button` | something that *happens* — submit, open, save, apply | navigation. If it goes somewhere pass `href` and it becomes a real `<a>` | rest, hover (lifts 5px), active, focus-visible, disabled, loading (`aria-busy`) |
| `Icon` | a small interface glyph | the brand's isometric service icons — place those SVG files directly | inherits `currentColor`; no states of its own |
| `Badge` | state, in a word — Live, Closed, Starts September | anything interactive. A badge is not a button | static. Seven tones, each paired with a word |
| `Tag` | a token the visitor can take off — a filter, a chosen option | state (that is `Badge`) | rest, hover on the remove control, focus-visible |
| `Avatar` | a person, as a photograph or initials | a generic silhouette for a real person — initials are more informative | static. Group avatars overlap with a page-coloured ring |
| `Card` | a repeating thing in a list — a course, a project, a person | a single item. A lone card is a box round nothing | rest (flat), hover (edge closes, lifts 4px, image scales 1.04), focus-within |
| `Stat` | one figure doing the work | a figure you computed. Every number comes from `brand-facts.md`, plus signs included | static |
| `Panel` | the box to reach for before inventing a component | a case `Card` already covers | rest, `interactive` (hover closes the edge and lifts) |

Sub-parts: `CardMedia`, `CardBody`, `CardFooter`, `CardGrid`, `StatRow`,
`TagGroup`, `AvatarGroup`.

**Accessibility.** Icon-only buttons require `aria-label` — the system draws a
red dashed outline round any that lacks one, so the omission is visible rather
than silent. The same applies to an `Icon` that declares neither `aria-hidden`
nor `role="img"`. Card images take `alt=""` when the title already says what
they are.

---

## forms — 14 components · Field/Input/Select/Choice/Switch Stable, the rest New

| Component | Use for | Do NOT use for | Keyboard |
| --- | --- | --- | --- |
| `Field` | every form control — label, hint, error as one unit | a control with no label. A placeholder is not a label | n/a |
| `Input` | text, email, password, number | a label. Never label a field with a placeholder alone | native |
| `Textarea` | more than one line | two questions. Two questions need two fields | native |
| `Select` | choosing a value from a short list | actions (that is `Dropdown`) | native — gets the phone's picker and autofill |
| `Choice` | checkbox or radio | an immediate effect (that is `Switch`) | native. Square radios; the tick versus filled square distinguishes them |
| `Switch` | something that takes effect **immediately** | anything that waits for Save | Space toggles |
| `SegmentedControl` | 2–4 mutually exclusive options, all visible | more than four — that is a `Select`; or multiple choice — that is `ChoiceGroup` | arrow keys, via real radios |
| `NumberInput` | a quantity where nudging by one is a real action | a wide range where position matters more (`Slider`) | arrows on the input; the steppers disable at the bounds |
| `Slider` | a threshold or tolerance where position matters | a number the visitor cares about exactly | arrows, Home, End. Always shows its value |
| `Combobox` | filtering roughly twenty options or more | fewer than twenty — a `Select` is better | ↓ opens and moves, ↑ moves, Enter picks, Escape closes |
| `DatePicker` | picking a date from a month | a date someone already knows — always offer a typed alternative. Never for a date of birth | arrows within the grid; every day has a full `aria-label` |
| `FileUpload` | attaching files, with per-file progress and errors | a single required document with no feedback — say the limits first | the drop zone is a real `<label>` over a real file input |

Sub-parts: `Search`, `ChoiceGroup`.

**Accessibility.** Errors say what to do, not that something went wrong: *"That
email address is missing an @"*, never *"Invalid input"*, and never blaming the
visitor. Required is an asterisk, not a colour. Hints sit under the field, never
in a tooltip — a tooltip is invisible on a touchscreen. In the date picker,
**today is an outline and the selection is a fill**; a filled today is
indistinguishable from a selection, which is the commonest date-picker bug there
is.

---

## feedback — 6 components · Stable

| Component | Use for | Do NOT use for | Notes |
| --- | --- | --- | --- |
| `Alert` | something on the page that stays | something that just happened (`Toast`) | four tones, only two added hues. `live="assertive"` interrupts a screen reader — blocking errors only |
| `Toast` | something that happened and needs no action | anything necessary. It disappears | region is `aria-live="polite"` deliberately |
| `Modal` | when the page must not continue until something is decided | anything that could be a page. Modals cannot be linked, bookmarked or found again | `<dialog>`, so Escape, focus trapping and the backdrop are the browser's |
| `Tooltip` | something genuinely optional | anything required. Invisible on touch, gone when the pointer moves | opens on keyboard focus as well as hover |
| `ProgressBar` | a length you know, or an indeterminate wait | faking progress. A bar that stops at 90% is worse than a spinner | percentage lives in `aria-valuenow` and is mirrored into CSS |

Sub-parts: `ToastRegion`.

---

## navigation — 6 components · Stable

| Component | Use for | Do NOT use for | Keyboard |
| --- | --- | --- | --- |
| `Tabs` | alternative views of the same kind of thing | sequential steps (`Steps`), or hiding bulk on a long page | ← → move, Home/End jump to the ends |
| `Accordion` | questions with long answers | anything important. Collapsed content is not read and is missed by skimmers | `<details>`, so it works with no script |
| `Breadcrumb` | position in a hierarchy | a substitute for a back button | native links; last item is `aria-current="page"`, not a link |
| `Pagination` | page-by-page through a long list | infinite content. Prefer `hrefFor` so pages can be shared | native links or buttons |
| `Dropdown` | actions | choosing a value in a form (`Select`) | Escape closes and returns focus to the trigger |

Sub-parts: `AccordionItem`.

---

## data — 6 components · Stable

| Component | Use for | Do NOT use for | Notes |
| --- | --- | --- | --- |
| `Table` | data where rows and columns both mean something | layout. Use a grid | every `th` has `scope`; numeric columns right-align in the mono face |
| `Steps` | a real sequence where order matters | alternative views (`Tabs`) | done steps carry a tick as well as the fill — colour is never the only signal |
| `PriceCard` | one plan | more than four plans; nobody compares them | absent features are shown, not hidden |
| `Quote` | a testimonial someone actually said | an invented endorsement. Use `Client name` until one is signed | the decorative mark is ink on light, gold on dark |
| `LogoTile` | a client or group-company mark | a client araCreate has not published | greyscale until hovered |

Sub-parts: `LogoStrip`.

---

## signature — 5 components · Stable

| Component | Use for | Do NOT use for |
| --- | --- | --- |
| `Eyebrow` | the uppercase kicker above a heading | gold text on a light surface. The component handles that; it turns gold only on a dark band |
| `Dash` | the 55px rule above a heading or hero title | decoration elsewhere. It means "a section starts here" |
| `Underline` | the animated wipe-in underline on a link | a link that is already underlined by `base.css` |
| `Marquee` | an endless horizontal band between two sections | anything a visitor must read. It moves |
| `Chevrons` | the deck's bottom-right motif | a web page. It is the deck's signature |

---

## sections — 8 components · Stable · marketing website only

`Header`, `Hero`, `Section`, `ServiceList`, `CtaBand`, `Footer`, `PostList`,
`NotFound`.

**Use these for aracreate.group and the landing pages, not inside an app.** A
`Section` is 90px of vertical rhythm and a 1200px container; inside an
`AppShell` that is wasted space. The app equivalent is `Toolbar` plus the main
region's own padding.

Two rules carried over verbatim: never cover the current-page link (mark it with
`aria-current` and style it — the template this replaced put an invisible
click-blocker over it and killed 27 links at once), and five or six top-level
links is the limit.

---

## app — 18 components · New · signed-in surfaces only

| Component | Use for | Do NOT use for | States |
| --- | --- | --- | --- |
| `AppShell` | the signed-in frame | a marketing page | expanded, `rail` (64px icons), drawer below 992px |
| `NavList` | the sections of an app | a marketing nav (`Header`) | rest, hover, `aria-current` (gold rule + tint + weight) |
| `Toolbar` | the strip above a table or list | a page heading on a marketing page | — |
| `BulkBar` | what to do with a selection | a floating bar. It replaces the toolbar, because a floating one covers the rows being checked | appears only when something is selected |
| `DataTable` | rows that are picked, sorted or acted on | static content (`Table`) | rest, hover, selected (tint **and** a gold left mark), sorted, loading, empty |
| `EmptyState` | saying what to do next | saying "No data" | three kinds: nothing yet, nothing found, nothing due |
| `Skeleton` | a placeholder shaped like what is coming | a generic spinner | container carries `aria-busy` and an off-screen "Loading"; bars are `aria-hidden` |
| `Drawer` | a record beside the list, not instead of it | anything that could be a page | `<dialog>`, so Escape and focus trapping are the browser's |
| `Banner` | page-level and it stays — maintenance, expiry, a failed sync | an in-page message (`Alert`). One banner at a time | `tone="error"` uses `role="alert"` |
| `KpiTile` | a dashboard figure | a second card design. It *is* `.ac-card--stat` | delta carries an arrow as well as a colour |
| `ChartShell` | **Frame.** The title, legend, gridlines and axis | the chart itself. Put a real chart inside it | gold is series one; the rest are neutrals, so two series survive greyscale |

Sub-parts: `AppBrand`, `NavFooter`, `FilterBar`, `CellStack`, `SkeletonStack`,
`SkeletonTable`, `Bars`.

**One rule specific to this group.** Row actions fade in on hover only where
`pointer: fine`. On touch and for keyboard they are always present — a control
that exists only on hover does not exist on a phone.

---

## Choosing between the three groups that overlap

| You want | Marketing page | App screen |
| --- | --- | --- |
| A page heading | `Section title` | `Toolbar title` |
| A table | `Table` | `DataTable` |
| A figure | `Stat` | `KpiTile` |
| A message | `Alert` | `Alert`, or `Banner` if it is page-level |
| Navigation | `Header` | `AppShell` + `NavList` |
| Nothing to show | — | `EmptyState` |
| Loading | — | `Skeleton` |

`core`, `forms`, `feedback`, `navigation`, `data` and `signature` serve both.
