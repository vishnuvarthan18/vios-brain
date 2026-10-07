# API AUDIT

A pass over the prop surface of all 78 components, looking for names that mean
different things in different places. Findings are ranked by what they would cost
a consumer, not by how tidy they are.

Audited 20 August 2026 by reading every `.d.ts`.

---

## Fixed

### `label` meant three different things — RESOLVED 20 August 2026

The finding below was recorded as "not fixed, do it deliberately before anyone
adopts this". It was then done deliberately, the same day, before anyone adopted
it.

`label` now means **one** thing across all 78 components: **text the visitor
can see.** The two other senses were renamed:

| Was | Is | On |
| --- | --- | --- |
| `label` (accessible name) | **`a11yLabel`** | `Icon`, `Search`, `Tabs`, `Breadcrumb`, `ProgressBar` |
| `label` (group heading) | **`heading`** | `NavList` |

`label` is unchanged and still visible text on `Field`, `Choice`, `Switch`,
`Slider` and `Stat`, and inside every `items`/`options` array entry.

**Nothing broke, because nothing was removed.** Every renamed component accepts
both spellings — `a11yLabel || label` — and the old name is marked
`@deprecated` in the `.d.ts` with the date and the reason. All thirteen call
sites across the four UI kits, the cards and the `prompt.md` examples were
updated in the same pass, so the deprecated path is documented but unused.

The distinction is now visible at the call site, which was the whole point:

```jsx
<Switch label="Keep me signed in" />        {/* renders text        */}
<Icon name="alert" a11yLabel="Warning" />   {/* renders nothing      */}
<NavList heading="Work" items={nav} />      {/* renders a heading    */}
```

**Why it was worth doing rather than documenting.** `<Icon label="Save" />` and
`<Switch label="Save" />` are indistinguishable in review and behave completely
differently. A JSDoc comment does not help the person skimming a diff. Renaming
does.

**`caption` stays as it is** on `Table` and `DataTable` — a table genuinely has
a `<caption>`, and naming it anything else would be worse HTML for the sake of
symmetry. That was finding 2 below, and it closes with this one.

---

**`align="center"` in a British codebase.** `EmptyState` was the only component
using American spelling, in a system that writes `colour`, `licence`, `centred`
and `organised` throughout — including `Section centred` and `Hero centred`
two directories away. Now `align="centred"`, with `"center"` accepted as an
alias so the American spelling does not silently do nothing.

---

## Found and NOT fixed, with reasons

### 1 · `label` means three different things — HIGH · **now fixed, see above**

| Component | `label` is |
| --- | --- |
| `Field`, `Choice`, `Switch`, `Slider`, `Stat` | a **visible** label |
| `Icon`, `Search`, `Tabs`, `Breadcrumb`, `ProgressBar` | an **accessible name** (`aria-label`) |
| `NavList` | a **group heading** above the items |

This is the one finding a consumer will actually trip on: `<Icon label="Save" />`
renders nothing visible, and `<Switch label="Save" />` renders text. Both look
identical in a code review.

**The correct fix** is `label` for visible text and `a11yLabel` (or
`accessibleName`) for the aria case, everywhere.

**Done 20 August 2026** — see the resolution at the top of this file. The
original reasoning for deferring was that the rename would break four UI kits,
eleven cards and every example. It did not, because both names are accepted and
every call site was updated in the same pass. Deferring it would have been the
wrong call: the cost was one pass, and it would only have grown.

The standing rule: **if it is not visible, it is `a11yLabel`.**

### 2 · `caption` versus `label` for a hidden name — MEDIUM · **closed**

`Table` and `DataTable` take `caption` (a visually hidden `<caption>`).
`ProgressBar` takes `label` for the same job. Both are correct HTML for their
element — a table genuinely has a `<caption>` — so this is defensible.

**Closed by finding 1:** `ProgressBar` now takes `a11yLabel`, and `caption`
stays on the two tables because it names a real HTML element. Two different words
for two different things, rather than one word for two.

### 3 · `variant` versus `tone` — LOW, and intentional

`Button` has `variant` (`primary`, `outline`, `ghost`, `inverse`, `danger`).
`Badge`, `Alert`, `Toast` and `Banner` have `tone` (`success`, `error`,
`warning`…).

**Keep both.** They are different axes: `variant` changes a component's
*structure and prominence*, `tone` changes its *semantic colour*. A ghost button
is not a "tone" and a success badge is not a "variant". This distinction should
be documented rather than flattened — it now is, here.

### 4 · Body copy is sometimes `text`, sometimes `children` — LOW

`CtaBand`, `EmptyState`, `NotFound` and `Tooltip` take `text`. `Alert`, `Toast`
and `Banner` take `children`. `Card` uses neither and expects you to write the
`<p>` yourself.

Roughly explicable — components with a fixed one-paragraph slot use `text`,
components accepting arbitrary content use `children` — but not a rule anyone
would guess. **Recommendation: prefer `children` for anything that might contain
a link.**

### 5 · `size` is a scale in most places and a spacing knob in one — LOW

`Button`, `Input`, `Avatar`, `Icon` use `size` as a scale (`sm`/`md`/`lg`/`xl`).
`Section` uses `size` for vertical rhythm (`tight`/`default`/`tall`). Same word,
different concept. `Section` would read better as `rhythm` or `spacing`.

### 6 · Ad-hoc booleans — LOW, but worth a rule

Nineteen one-off boolean props across the set: `tight`, `wide`, `flat`, `padSm`,
`noEdge`, `large`, `square`, `accent`, `dot`, `pill`, `block`, `bleed`, `centred`,
`dash`, `grid`, `single`, `row`, `rail`, `invalid`.

Most are fine. Two are worth noting: **`padSm` is the only abbreviated prop in
the system** (it should be `padding="sm"` or `compact`), and **`noEdge` is the
only negative one** — negative booleans read badly at the call site
(`noEdge={false}` is a double negative). Both are single components.

**Rule going forward: no abbreviations, no negatives.** Name the state you want
on, not the one you want off.

### 7 · `onChange` signatures differ — LOW, and deliberate

`Input` and `Textarea` pass the DOM event (they are thin wrappers over the real
element). `Slider`, `NumberInput`, `Combobox` and `SegmentedControl` pass the
**value**, because the event is noise when the component owns parsing.

Defensible, and the `.d.ts` states which. Worth knowing before you write
`e.target.value` on a `Slider` and get `undefined`.

---

## Checked and consistent

Worth recording so nobody re-audits them:

- **`href` turns a component into a real `<a>`** — `Button`, `Card`, `Underline`,
  `PostList` items, `ServiceList` items. Consistent, and the right default: if it
  goes somewhere, it is a link.
- **`surface`** takes the same five values everywhere it appears (`Panel`,
  `Section`, `CtaBand`, `Footer`).
- **`onClose`** means "the visitor dismissed this" on `Modal`, `Drawer`, `Alert`,
  `Toast` and `Banner`.
- **`actions`** is the right-hand slot on `Toolbar`, `ChartShell`, `Hero`,
  `BulkBar`; `action` (singular) is a single control on `EmptyState`, `CtaBand`,
  `NotFound`, `Banner`. Plural means a group. Consistent.
- **`items`** is always the array a component maps over.
- **Every component spreads `...rest` onto its root element**, so `id`,
  `style`, `data-*` and `aria-*` always work.
- **Sub-parts are named `ParentPart`** — `CardBody`, `CardMedia`, `StatRow`,
  `TagGroup`, `AvatarGroup`, `ChoiceGroup`, `ToastRegion`, `AccordionItem`,
  `SkeletonStack`, `SkeletonTable`, `NavFooter`, `AppBrand`. One exception:
  `Search` is part of the `Input` file but not called `InputSearch`, which is
  correct — it is its own thing.

---

## Coverage gaps found while auditing

Not naming problems, but missing pieces:

- **No `Textarea` in the app inputs card.** It exists and is documented; the card
  just does not show it.
- **`ChartShell` has no axis-label prop for the Y axis** — only the X. A chart
  with an unlabelled Y axis is a shape.
- **`DatePicker` has no range mode**, though `app.css` styles
  `[data-ac-in-range]` for one. The CSS is ahead of the component.
- **`Pagination` has no page-size control**, which a data table usually needs
  next to it.
- **No `Tooltip` on a disabled control.** A disabled button with no explanation
  is the most common "why can't I click this" complaint, and `pointer-events:
  none` means the tooltip cannot fire. Needs a wrapper.

---

## Verdict

The prop surface is consistent on the things that matter structurally — `href`,
`surface`, `onClose`, `actions`, `items`, `...rest` — and, since the `label`
rename, on the one thing that mattered practically as well.

**Two findings fixed** (`label`, `align`), **one closed** (`caption`), **five
recorded as cosmetic with reasons** (`variant`/`tone` kept on purpose,
`text`/`children`, `size` as two concepts, `padSm` and `noEdge`, the two
`onChange` signatures).

The rules that came out of it, for anything added later:

1. **If it is not visible, it is `a11yLabel`.**
2. **No abbreviations** — `padding="sm"`, never `padSm`.
3. **No negative booleans** — name the state you want on.
4. **`variant` changes structure; `tone` changes semantic colour.** Both, not one.
5. **Plural `actions` is a group; singular `action` is one control.**

The five coverage gaps below are still open and are the most useful remaining
work on the component set.
