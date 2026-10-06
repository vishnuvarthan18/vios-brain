# @arametrics/ds

Design-system components shared by the arm-ui app. Started as 10 generic primitives
(`AccentHeader`/`AccentCard`, `AccountCard`, `CardFooter`, `ConfirmDialog`, `EventBlock`,
`IconTile`, `MonthGrid`, `PausedBadge`, `Skeleton`, `Toast`), then a later pass moved the
auth screens, app shell (Sidebar/Topbar/SearchOverlay), and the whole calendar module in too —
see **Scope** below for the current boundary. Full current list: `src/index.ts`.

Built on Radix Themes. `react-router-dom` is a peer dependency for the shell/nav/calendar
components that render real navigation links (`Sidebar`, `Topbar`, `AppCard`, etc.) — that's
a real coupling, not an oversight; components that don't navigate stay router-free. No app
React context (`useAuth`, `useIntegrations`, `useCalendar`, `useAdmin`) is imported anywhere
in `src/` — app-owned data is always taken as props/callbacks instead.

## Development

```
npm install
npm run storybook        # dev server, port 6007
npm run build-storybook  # static build
npm run typecheck
```

Storybook coverage: every exported component has a story under `stories/`.
(`useConnectAccount` is a hook, not a component — no story expected. `OnboardingStepDots` is a
deprecated alias for `OnboardingStepper`, which has the story — see that component's own
docstring.)

## Files duplicated from the app (by design, not by accident)

These exist so the package has **zero import path back into `arm-ui/src/`**. Each is a
deliberately narrow copy — not the app's full file — kept in sync by hand, since there's no
tooling that enforces it. If you change a shared value in the app, check whether it needs to
change here too.

- **`src/layout-tokens.ts`** — a subset of `arm-ui/src/lib/layout-tokens.ts`'s `layout` object
  (17 of the app's ~30 constants — only the ones this package's own components reference).
  Values are checked to match the app's copy as of this writing; the gap is coverage
  (constants added to the app since aren't here yet), not drift.
- **`src/tokens.css`** — utility classes and custom properties the components in `src/`
  actually use. Despite the name suggesting a `theme-overrides.css` subset, most of its content
  (`.data-table-row`, `.shell-nav-item`, `.topbar-search`, `.otp-box`, `.metric-card`, etc.) was
  actually pulled from the app's `globals.css`, not `theme-overrides.css` — only `.dark` /
  `.radix-themes` come from the latter. Consumers must load Poppins themselves; this package
  doesn't self-host font files.
- **`src/calendar-types.ts`** — type shapes only (`CalendarAccount`, `CalendarSync`, etc.),
  duplicated from `arm-ui/src/lib/calendar-mock.ts`. The app's actual mock/seed data
  (`CALENDAR_ACCOUNTS` and friends) never lives here — components take real data as props.
- **`src/appearance-context.tsx`** — duplicated from `arm-ui/src/lib/appearance-context.tsx`,
  used only by `.storybook/preview.tsx`'s theme decorator. No component in `src/` calls
  `useAppearance()` itself — this is Storybook-only plumbing that happens to live in `src/`.
- **`ProviderIcon.tsx` / `Sidebar.tsx`** — each duplicate one small type (`CalendarProvider`,
  `Role`) from the app's `src/lib/mock.ts`, same reasoning as `calendar-types.ts`.

## Extraction note

This package was split out of `arm-ui/src/components/ds/` via `git subtree split`, preserving
file history, then expanded in a later pass to absorb `arm-ui/src/components/{auth,shell,calendar}/`
in full. If you're reading this inside a freshly extracted standalone repo (not the `arm-ui`
monorepo), a few things differ from how it lives as a workspace member:

- **`package.json`'s `"private": true`** — drop this line. It's required for local npm
  workspace members but wrong for a standalone repo meant to be published or consumed via git
  URL.
- **Consuming app** — `arm-ui` depends on this package by name (`@arametrics/ds`), including
  its stylesheet via `@arametrics/ds/tokens.css` (see `package.json`'s `exports` map — add a
  subpath entry there for any new CSS file this package ships). Once this repo has its own
  remote, point `arm-ui/package.json`'s dependency at a git URL or published npm version
  instead of the local workspace `"*"` range.

## Scope

Components with app-state dependencies (`useAuth()`, `useCalendar()`, etc.) were converted to
take that data as props rather than excluded — see **Files duplicated from the app** above for
how the type shapes got here without a live import. The only things that stay in `arm-ui/src/`
are actual app wiring: route guards (`RequireAuth`/`RequireRole`), layout files that supply
real context data to these components, and pages that compose them with real app data/routing.
