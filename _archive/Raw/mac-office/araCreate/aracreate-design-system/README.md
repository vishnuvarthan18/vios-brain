# araCreate Design System

The design system for **araCreate Group** — tokens, components, foundations,
templates and the brand asset library, kept under version control as the backup
and audit trail of the master.

The brand is defined once — Golden Sun `#F9BF3B` and Graphite Gray `#555555`,
Monument Extended for the logo wordmark and Poppins for everything else — and
each platform folder under [`src/`](src/) expresses it in the format that
platform expects.

## Where the master is

**The master is the Claude Design project ACDS** (`acds-aracreate-design-system`).
All design-system edits happen there. This repository receives exports from it
and holds the history; nothing is edited here and imported back.

Settled 24 August 2026. If a file here and the master disagree, the master is
right and this repo is behind.

## What it holds

- **Design tokens** — colours, typography, spacing, radii, shadows and motion as
  CSS custom properties (`--ac-*`). Two entry points: `tokens.css` (variables
  only) and `system.css` (everything).
- **Components** — 81 React components in eight groups, each with a `.d.ts`
  contract and a usage prompt.
- **Foundations** — 35 specimen cards for palette, type, spacing and brand marks.
- **Templates** — `web`, `app` and `deck` starting folders.
- **Assets** — logos, isometric icons and illustrations, duotint imagery, fonts.
- **Docs** — brand facts, guidance, decisions, accessibility, deck conventions.

## Layout

```
aracreate-design-system/
├── src/
│   └── claude-design-system/   # the exported master — do not edit here
├── docs/  tests/  releases/  logs/  .archives/  scripts/
├── Makefile · VERSION · LICENSE · README.md
```

### `src/claude-design-system`

An export of the master, byte-for-byte. Its own
[`readme.md`](src/claude-design-system/readme.md) is the full reference and its
[`changelog.md`](src/claude-design-system/changelog.md) the version history;
[`github.md`](src/claude-design-system/github.md) records each export.
Components are bundled into `_ds_bundle.js` and exposed on
`window.AraCreateDesignSystem_4716e7`. Consumers link `system.css`.

## Commands

```
make help       # banner and target list
make list       # list the design systems in src/
make release    # cut a semantic release
make clean      # remove OS and editor cruft
```

## Conventions

See [aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions).
They apply to the **repo root only**; each folder under `src/` keeps its
platform's own format and is not reformatted.

## License

Proprietary. Copyright (C) 2026, araCreate Group. See [LICENSE](LICENSE).
