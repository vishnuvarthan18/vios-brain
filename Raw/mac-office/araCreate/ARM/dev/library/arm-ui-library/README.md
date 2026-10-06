# araMetrics UI

A modular super-app: one web platform, authenticate once, access apps determined by
role + admin-granted permissions. v1 covers three surfaces — the **core shell**, the
**Admin Portal**, and the **Calendar module**.

> The full product spec (screens, roles, flows, states) lives in
> [`../ui-ux/arm-v1-spec.md`](../ui-ux/arm-v1-spec.md) — the source of truth for scope.
> The visual spec is the Figma export PNGs in `../ui-ux/figma/exports/`.

## Stack

- **Vite 8** + **React 19** (plain SPA — no framework, no SSR)
- **react-router-dom 7** (plain `<Routes>`/`<Route>` — no data router/loaders)
- **Radix Themes** for the component layer ([`@radix-ui/themes`](https://www.radix-ui.com/themes))
- **Storybook 10** (`@storybook/react-vite`) + **Vitest 4** for the component workbench
- **TypeScript**, ESLint (flat config, `typescript-eslint`)

## Getting started

```bash
npm install
npm run dev          # app on http://localhost:5173
npm run storybook    # component workbench on http://localhost:6006
```

The root route (`/`) redirects to `/login`. Auth is passwordless: enter an email,
then the 4-digit code shown in the "Dev hint" callout (mock API — no real email).

## Scripts

| Command                   | What it does                               |
| ------------------------- | ------------------------------------------ |
| `npm run dev`             | Start the Vite dev server (port 5173)      |
| `npm run build`           | Production build (`dist/`)                 |
| `npm run preview`         | Serve the production build locally         |
| `npm run lint`            | Run ESLint                                 |
| `npm run storybook`       | Start Storybook (port 6006)                |
| `npm run build-storybook` | Build the static Storybook                 |

## Project layout

```
index.html           Vite entry (loads src/main.tsx)
src/
  App.tsx            Route tree
  routes/            Auth/role guards (RequireAuth, RequireGuest, RequireRole)
  layouts/           Shell / admin / calendar layout wrappers
  pages/             One file per route
  components/        Feature-grouped UI
    admin/ auth/ calendar/ shell/   Surface-specific components
    ds/              Shared design-system primitives
  lib/               React contexts, mock API client (lib/api/), mock data
  styles/            globals.css, theme overrides
stories/             Storybook stories (radix-base, patterns, app)
public/              Brand assets (arm-* logos/icon)
.storybook/          Storybook config
```

Imports use the `@/*` alias mapped to `src/` (e.g. `@/components/ds`, `@/lib/mock`).

## Notes

See [`AGENTS.md`](AGENTS.md) for architecture conventions and the design-system
rules (typography, buttons, component reuse, Figma verification workflow).
