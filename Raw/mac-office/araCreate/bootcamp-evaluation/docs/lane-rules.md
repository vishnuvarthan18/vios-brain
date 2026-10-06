# Lane rules — read this before any lane work

Four agents build screens at the same time. They never touch the same file, so
their branches merge without a single conflict. That is not a hope; it is
arranged by the rules below, and every one of them exists because breaking it
causes a conflict.

**If you are a lane, read this file and your own `docs/lane-<x>.md`. Nothing
else.**

---

## 1. The one rule

**A lane only ever creates new files inside its own list of screens.**

Everything shared was built before the lanes started. It is finished. You do
not add to it, improve it, or tidy it.

### Files NO lane may edit, for any reason

| File | Why |
| --- | --- |
| `web/src/App.jsx` | Routing is automatic now — see section 2. Editing it is the biggest conflict there is |
| `web/src/components/Shell.jsx` | The frame is done |
| `web/src/components/ui/*` | Shared components. Four lanes improving a table four ways is four conflicts |
| `web/src/lib/api.js`, `useData.js`, `utils.js`, `nav.js`, `icons.jsx` | Shared |
| `web/src/index.css` | Tokens. A missing token is reported, never added |
| `web/package.json`, `package-lock.json` | Every package a lane could need is already installed |
| `docs/v3-ui-migration.md` | Every lane ticking the same table is a conflict on every merge |
| `src/public/app.js`, `app.css`, `index.html` | The old front end. Untouched until cutover |
| `src/server.js`, `src/routes/*` | No lane changes the server. Ever |

### What to do when you need something you are not allowed to add

You need a component that does not exist. You need a token that is missing. You
need a package.

**Do not add it.** Write it in your report, and solve it inside your own page
file for now — a local helper in your own file is fine, even if it duplicates
what another lane is doing. Duplication is cheap. A conflict costs everyone.

The duplicates get pulled into `components/ui/` afterwards, once, by one person
who can see all four lanes at the same time.

---

## 2. How routing works now — why you never edit App.jsx

Every page file declares its own route key:

```jsx
export const route = "marking"          // the key from pagesFor() in lib/nav.js
export default function Marking({ ctx }) { ... }
```

`App.jsx` finds every file in `web/src/pages/` automatically and builds the map
from those `route` exports. **Adding a screen is adding one file.** Nothing
shared changes, so nothing can conflict.

### Every page takes exactly one prop

```jsx
export default function Marking({ ctx }) {
  const { me, go, jumpTo, clearJump } = ctx
  ...
}
```

- `me` — the signed-in person: `kind`, `is_admin`, `is_lead`, `name`, `team_id`,
  `team_code`, `assessOpen`, `surveyOpen`
- `go(page)` — move to another page
- `jumpTo`, `clearJump` — the deep-link slot, used by Today → You

Do not change the shape of `ctx`. If you need something that is not in it, fetch
it in your own page.

---

## 3. What is already built — use it, do not rebuild it

**`@/components/ui/`** — button · input · label · card · alert · confirm ·
table · toolbar · tabs · modal · select · checkbox · textarea · badge

**`@/components/ui/bits`** — `Pill` `Head` `SectionHead` `Note` `Empty` `Muted`
`Bar` `Stats` `Stat` `Skeleton` `Failed` `Field`

**`@/lib/api`** — `api(path, {method, body})` · `upload(path, formData)` ·
`Offline`. Every failure comes back as plain English. **Never use bare `fetch`.**

**`@/lib/useData`** — `useData(load, deps)` gives you `{data, error, loading,
reload}` and the loading and failed states that go with it.

**`@/lib/icons`** — `navIcon(key)` and `<Icon name>`.

Read one existing page before you write yours. `web/src/pages/Work.jsx` and
`Attendance.jsx` are the two worth copying the shape of.

---

## 4. Your branch

Your lane has its own branch and its own worktree. You never leave it.

- Commit **one screen per commit**. Never a half screen.
- Push after every commit. A lane once went 733 lines without committing on this
  project, and one wrong command would have lost all of it.
- Never merge. Never rebase. Never touch another lane's branch. Vishnu merges.

---

## 5. Your report

You write **one file: `docs/lanes/<your-lane>.md`.** Nobody else writes to it,
so it can never conflict. Keep it updated as you go, and commit it with each
screen.

For every screen, record:

1. **Check list** — the two or three things Vishnu should click to prove it
   works. Name the role and the day. "Hand in a photo on Day 2 as a non-lead",
   not "check the work page".
2. Anything the old screen did that you could not reproduce.
3. Any shared component, token or package you needed and worked around.

---

## 6. The standing rules still apply

Everything in `docs/v3-ui-build-prompts.md` under **Standing rules** holds:
nothing is redesigned, light theme only, no raw hex or px or font names, no
`dark:` classes, no Playwright, no screenshots, no deploy, and the production
build must pass before a screen counts as done.

Read the screen's row in `docs/v3-ui-migration.md` before building it — its
endpoints and its "must keep" notes are there. **Read it. Do not edit it.**
