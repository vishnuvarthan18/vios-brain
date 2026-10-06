repo: aracreate-group/aracreate-design-system
branch: main
path: src/claude-design-system

## Sync history

- 2026-08-17T12:29:44Z — imported the full design system tree (tokens, foundations cards, components, UI kits, assets) from `src/claude-design-system/`; dropped generated files and `uploads/`; re-pointed namespace references, added a project thumbnail.

## Last sync

date: 2026-08-20T09:18:12Z

### Updated in this project

- Checked upstream `main` (tokens, styles.css, components/core Button+Card, ui_kits/website Navbar, ui_kits/deck index.html) against local files — all byte-identical, no drift found.
- Local `tokens/colors.css` remains ahead of upstream (four live-Webflow variables added 2026-08-17: `--ac-true-black`, `--ac-photo-overlay`, `--ac-gray-translucent`, `--ac-button-gray-light`) — not yet pushed upstream.
- No commit sha was recorded from the prior sync, so this pass diffed file contents directly rather than via `github_compare`; recording the tree root `3dfa6b3212da` here for the next incremental sync.

commit: 3dfa6b3212da

## Screen map

| Screen / area | Repo files |
| --- | --- |
| Tokens + `styles.css` | `src/claude-design-system/styles.css`, `tokens/*.css` |
| Foundation cards | `src/claude-design-system/foundations/*.card.html` |
| Core + content components | `src/claude-design-system/components/core/*`, `components/content/*` |
| Website UI kit | `src/claude-design-system/ui_kits/website/*` |
| Deck UI kit / slides | `src/claude-design-system/ui_kits/deck/*` |
| Assets (logos, icons, illustrations, imagery, fonts) | `src/claude-design-system/assets/**` |
