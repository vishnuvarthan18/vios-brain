# design/

The design system and the work around it. **Nothing in this folder is published.**

| Path | What |
|---|---|
| `showcase/` | The workbench: `styleguide.html`, `identity.html`, sample pages, and the **source of truth for the CSS** (`showcase/css/`: `tokens.css`, `components.css`, `materials.css`, `textures.css`). Docs: `showcase/DESIGN_SYSTEM.md`. |
| `svg_kit/` | Our own redrawn SVGs. The identity kit is **generated**: edit `svg_kit/identity_build.py`, never the output files. |
| `realism/` | The realism engine: measured materials (leaf, stone, copper, clay), checks, reports, prompts. Local only. |
| `reference_engine/` | Reference-photo database (`refs.db`) and viewer (`viewer.html`). |
| `references/` | Reference photos with `LICENSES.csv`. **Git-ignored. Never commit.** |
| `prompts/`, `specs/` | The numbered design prompts and the palette, texture and layout specs |
| `fetch_commons*.py`, `curate_refs.py` | Tools that collect reference photos with their licences |

## Changing the design system
1. Edit the files in `design/showcase/css/`.
2. Open `design/showcase/styleguide.html` (via `make local`) and check every component in light and dark mode.
3. Copy the changed files to the site: `cp design/showcase/css/tokens.css design/showcase/css/components.css website/css/`
4. `make check`. It fails if the two copies differ.
5. Check the website pages, then open a pull request.

## Rules
- Only public-domain, CC0 and CC-BY photos ship, each with a credit. CC-BY-SA, CC-BY-NC and unclear files are for measuring only (`../docs/CONTENT_RULES.md`).
- Reference photos are never copied into `website/`.
- Do not delete backups under `realism/backups/`; they are the record of what each step changed.
- Tools that need a browser (`realism/tools`, `showcase/_checks`) need `playwright-core`; see `realism/package.json` and `realism/RUNNING.md`.

## Removed experiment pages (2026-10-04)
The nine realism demo pages (rig stage, 3D leaf, bundle, copper set, sherd and stone, small objects, copper light, stone light, sound) and the files only they used were deleted. Everything is in git history (`git log --diff-filter=D -- design/realism`). Kept because the generators and the shipped materials still read them: `rig/stage.js`, `rig/rig.mjs`, `t3/leaf3d.params.*`, `t6/*_light.params.*`, the check scripts, `registry/`, `materials/`.
