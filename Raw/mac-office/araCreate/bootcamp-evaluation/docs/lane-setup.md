# Lane setup — one agent, alone, before any lane starts

**Nobody else runs while this is happening.** This is the work that makes four
parallel lanes possible. If lanes start before it is finished, they will each
invent their own routing and their own table, and the merging will cost more
than the parallelism saved.

Do all five steps, commit them, push, and report. Build nothing else.

---

## 1. Make routing automatic

Today `App.jsx` picks the page with a hardcoded chain:

```jsx
page === "projects" ? <Work me={me} /> : page === "home" ? <Today .../> : ...
```

Every lane would have to edit those same ten lines. That is the single biggest
source of conflict in this plan, and it is removable.

Replace it with a registry built from the files themselves:

```jsx
const modules = import.meta.glob("./pages/*.jsx", { eager: true })
const PAGES = {}
for (const m of Object.values(modules)) {
  if (m.route && m.default) PAGES[m.route] = m.default
}
```

Then each page declares its own key:

```jsx
export const route = "projects"
export default function Work({ ctx }) { ... }
```

Add `export const route` to all nine existing pages, and leave `Login.jsx` out
of the registry — it is not a destination.

Keep the `Placeholder` fallback for any key with no file yet. It is what stops
an unmoved screen looking broken.

## 2. One prop for every page

Page props differ today — `me`, `go`, `jumpTo`, `clearJump`, in different
combinations. That means a lane has to edit `App.jsx` to pass a new one, which
is the conflict again.

Give every page exactly one prop:

```jsx
<Page ctx={{ me, go, jumpTo, clearJump }} />
```

Convert all nine existing pages to read what they need out of `ctx`. **The
shape of `ctx` is then fixed** — `docs/lane-rules.md` tells lanes not to change
it.

## 3. Build the shared components the admin screens will need

The student screens needed almost none of these. The fifteen admin screens are
mostly tables, filters and dialogs, and **every lane will want the same ones**.
Build them now, once:

- `table.jsx` — the whole thing: header, rows, an empty state, and the
  horizontal scroll a seven-column table needs on a phone
- `toolbar.jsx` — the strip above a table: title, count, filters, actions
- `tabs.jsx` — `@radix-ui/react-tabs` is already installed
- `modal.jsx` — a form dialog with Save and Cancel, on
  `@radix-ui/react-dialog`. `confirm.jsx` already handles yes/no; this is the
  one that holds a form
- `select.jsx`, `checkbox.jsx`, `textarea.jsx` — native elements, styled to the
  tokens. Do not add a Radix package for these
- `badge.jsx` — status pills for scored / submitted / open / closed

Match `src/public/app.css` for how these look today — the `.ac-table`,
`.ac-toolbar` and `.ac-card` rules are the reference. Every value points at an
`--ac-*` token.

Also port `dept_switch()` and `wire_dept_switch()` from `app.js`. Nearly every
admin screen has the Both / EEE / ECE filter, and four lanes writing it four
times is four subtly different filters.

## 4. Install every package the lanes could need

A lane running `npm install` changes `package.json` and `package-lock.json`,
and two lanes doing it is a conflict in a generated lock file — the worst kind
to resolve.

So install everything now, in one commit. Look through the fifteen remaining
screens in `docs/v3-ui-migration.md` sections 6.4 to 6.7, decide what they need,
and install it. If you are unsure whether something is needed, install it — an
unused dependency costs nothing next to a lock-file conflict.

The charts in step 11 (`attendance_chart`, `points_chart`, `spread_html`) are
the ones to think hardest about. They are hand-drawn HTML and CSS today, using
`.ac-chart__*` and `.ac-bars__*` from the design system. **Keeping them
hand-drawn is the better answer** — they are simple bars, the design system
already styles them, and a charting library would be a second visual language.
Only add one if you find something those three genuinely cannot do.

## 5. Check it still works

The nine existing screens must behave exactly as before. Same routes, same
behaviour, same look. This step moves code; it changes nothing a person sees.

The production build must pass with no errors and no warnings.

---

## Report

1. Confirm the nine existing pages still work and the build passes.
2. List every shared component you built.
3. List every package you installed, and why.
4. Anything you found while doing this that a lane needs to know.

Then stop. Vishnu starts the lanes.

---

## One thing to fix while you are here

`web/src/lib/nav.js` carries a comment saying the old separate "My team" and
"Quiz results" tabs "fold into Today and the leaderboard". **That is not true**
— `pages_for()` in `app.js` never had those tabs for a student, and the code
below the comment correctly matches the old list. Delete the sentence. A
comment that describes a change nobody made will send somebody looking for it.
