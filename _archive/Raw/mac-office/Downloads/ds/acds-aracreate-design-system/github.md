repo: aracreate-group/aracreate-design-system
branch: main
path: src/claude-design-system

## Sync history

**Closed.** These entries record ACDS's sync relationship with the upstream
repository up to 20 August 2026. Nothing below has been re-run since the merge
of 21 August 2026, and the tree has moved a long way from the shape these
entries describe — see the merge entry that follows.

- 2026-08-17T12:29:44Z — imported the full design system tree (tokens,
  foundations cards, components, UI kits, assets) from `src/claude-design-system/`;
  dropped generated files and `uploads/`; re-pointed namespace references, added
  a project thumbnail. **Closed.**
- 2026-08-20T09:18:12Z — diffed key files against upstream `main` directly (no
  commit sha on record) — all identical; local `colors.css` was ahead with 4
  live-Webflow vars, pushed upstream manually after this sync. **Closed.**
- 2026-08-20T09:29:38Z — compared tree root `3dfa6b3212da` against `main`: 83
  files changed upstream. `tokens/colors.css` upstream now matches local
  exactly — the manual push of the 4 live-Webflow variables landed. Verified the
  other modified files (`Button.d.ts`, `ServiceCard.d.ts`, `Hero.jsx`,
  `TrustedBy.jsx`, `ui_kits/website/index.html`) byte-identical to local — no
  rebuild needed. Upstream removed `slides.jsx` and `uploads/`, and added its own
  `templates/`, `thumbnail.html`, `CLAUDE.md`, `github.md` — all parallel to work
  this project already had independently.
  commit: `main@83-file-diff-from-3dfa6b3212da`. **Closed.**

- 2026-08-21 — **merged.** The 20 August 2026 "araCreate Design System" project
  was folded into ACDS. ACDS is the base: its structure, file naming and token
  values won, with three deliberate accessibility exceptions and one
  knowingly-kept failure. See [`changelog.md`](changelog.md) for the full entry
  and the measured contrast figures.

  What this means for syncing:

  - **The tree is no longer a subset of `src/claude-design-system/`.** It now
    also carries the ported CSS and JavaScript of araCreate's own design-system
    repository (`styles/`, `js/`), that repository's documentation (`docs/`),
    the dark theme, the density layer, the application component group and the
    browser test gate. None of that came from `src/claude-design-system/`.
  - **Two upstreams, not one.** Changes to `styles/` and `js/` belong back in
    araCreate's own repository, not in `src/claude-design-system/`;
    [`docs/back-port.md`](docs/back-port.md) is the action list for that and is
    the file to read before the two diverge further.
  - **No diff has been run against `main` since the merge.** The 83-file
    comparison above predates it and should not be treated as current.
  - Naming changed in the merge — `guidelines/` became `foundations/`,
    `colour-*.card.html` became `colors-*.card.html`, `ui_kits/group_website/`
    became `ui_kits/website/`, and `css/tokens.css` was split four ways into
    `tokens/`. A path-level diff against either upstream will report far more
    change than there actually is.

## Last sync

date: 2026-08-21
commit: none — merge performed locally, not synced upstream

### State

- Local tree is **ahead of both upstreams** and has not been pushed.
- The next sync has to be run twice, once per upstream, using the screen map
  below.

## Screen map

Local paths in this project, and where each belongs upstream.

| Screen / area | Local | Upstream |
| --- | --- | --- |
| Token layer | `tokens/{fonts,colors,typography,spacing}.css` | `src/claude-design-system/tokens/*.css`; values also in araCreate `src/tokens.css` |
| Density and dark theme | `tokens/{density,theme-dark}.css` | araCreate repo — see `docs/back-port.md` |
| System entry point | `system.css` | new here; no upstream counterpart |
| Element and device CSS | `styles/{base,signature}.css` | araCreate `src/base.css`, `src/signature.css` |
| Component and page CSS | `styles/{components,sections,app,deck}.css` | araCreate `src/components.css`, `src/sections.css`, `src/deck.css`; `app.css` is new here |
| Behaviour | `js/{signature,components,deck}.js` | araCreate `src/signature.js`, `src/components.js`, `src/deck.js` |
| Foundation cards | `foundations/*.card.html` | `src/claude-design-system/foundations/*.card.html` |
| React components | `components/*/` | `src/claude-design-system/components/core/*`, `components/content/*`; the other seven groups are new here |
| Website UI kit | `ui_kits/website/*` | `src/claude-design-system/ui_kits/website/*` |
| Deck UI kit | `ui_kits/deck/*` | `src/claude-design-system/ui_kits/deck/*` — **locked** |
| Academy and web-app kits | `ui_kits/{academy,web_app}/*` | new here; no upstream counterpart |
| Templates | `templates/{deck,marketing-page}/*` | `src/claude-design-system/templates/*` — `templates/deck/` is **locked** |
| Test gate | `tests/checks.html` | port of two of the araCreate repo's seven gates |
| Documentation | `docs/*.md` | araCreate `docs/*.md` for the five ported files; the rest written here |
| Assets (logos, icons, illustrations, imagery, fonts) | not present in this tree | `src/claude-design-system/assets/**` and araCreate `src/assets/` |
