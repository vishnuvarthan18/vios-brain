# arm-website

The araCreate Group public marketing site — a separate project from `arm-ui`
(the araMetrics product app). Plain TypeScript React SPA, Vite, React Router.

## Status

Skeleton only. The real design/content hasn't been handed off yet — see
`src/pages/LandingPage.tsx`'s placeholder. Do not build the landing page
against guessed content; wait for the actual design.

## Structure

- `src/` — the website app (pages, routing)
- `packages/ds/` — `@aracreate/ds`, this site's own design system, a real
  npm workspace package (same pattern as `arm-ui/packages/ds`). Separate
  from `@arametrics/ds` — this is a different brand (araCreate Group, not
  araMetrics) with its own tokens/components.
- `stories/` — app-level Storybook stories that don't belong in the DS
  package (page-level composition, if any end up needed)
- `reference/` (sibling, not committed — see `../.gitignore`) lives in
  `arm-ui/reference/aracreate-design-system/`, not here. It's a saved copy
  of the source design system (github.com/aracreate-group/aracreate-design-system)
  to port `@aracreate/ds` from — real tokens (`--ac-*`), real components
  (Button, SectionLabel, Card, etc.), and a ready-made `ui_kits/website/`
  recreation of the live site. Not directly installable (built for a
  different runtime, plain global-script JSX) — components get ported into
  real React/TypeScript here, same approach used for `@arametrics/ds`.

## Scripts

Same shape as `arm-ui`: `npm run dev` / `build` / `preview` (Vite),
`npm run storybook` / `build-storybook` (app-level Storybook, port 6008).
`packages/ds` has its own `npm run storybook` (port 6009).
