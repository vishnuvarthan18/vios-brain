# araCreate Design System

The design systems for **araCreate Group** — the brand's tokens, components,
foundations and asset library, kept under version control so every platform that
consumes them works from the same source.

The brand itself is defined once — Golden Sun `#F9BF3B` and Graphite Gray
`#555555`, Monument Extended for the logo wordmark and Poppins for everything
else — and each platform folder under [`src/`](src/) expresses it in the format
that platform expects.

## This repo is the master

**All design system edits happen here.** This repository is the single source of
truth for the araCreate design system.

Platforms that render or design with the system — Claude Design among them —
consume a **snapshot imported from this repo**. That import is one-way. A canvas,
a preview, or a rendered design is never the master: changes made there are not
authoritative and will be overwritten by the next import.

To change the design system, change it here, then re-import downstream.

## What it holds

- **Design tokens** — colors, typography, spacing, radii, shadows and motion, as
  CSS custom properties (`--ac-*`)
- **Components** — React components with type definitions and usage docs
- **Foundations** — preview cards for the palette, type scale, spacing and brand marks
- **UI kits** — recreations of the marketing site and the pitch deck
- **Assets** — logos, isometric icons and illustrations, duotint imagery, fonts
- **Sources** — the brand guidelines, deck and brand-application PDFs everything traces back to

## Stack

| Layer | Tool |
| --- | --- |
| Tokens | CSS custom properties, entry point `styles.css` |
| Components | React (JSX) + `.d.ts` contracts |
| Preview cards | Static HTML |
| Typography | Monument Extended (logo only), Poppins, Red Hat Mono, Inconsolata |
| Consumers | [Claude Design](https://claude.ai/design) — imports a snapshot of this repo |

## Layout

```
aracreate-design-system/
├── src/                        # One folder per platform (see src/readme.md)
│   └── claude-design-system/   # Claude Design export — consumed as-is
├── docs/                       # Documentation
├── tests/                      # Tests and prototypes
├── releases/                   # Shippable outputs
├── logs/                       # Project-log media
├── .archives/                  # Approaches trialled but not shipped
├── scripts/                    # Helper scripts + motd
├── Makefile
├── VERSION
├── LICENSE
└── README.md
```

Each folder under `src/` is self-contained and keeps its own platform's layout,
so it can be consumed directly with no build step in between. Further platforms
(for example a Storybook-based system) are added as sibling folders.

### `src/claude-design-system`

Originally extracted from the Claude Design project
[`aracreate-design-system`](https://claude.ai/design/p/9b36ca8f-4c81-4048-b714-7d9efa992b03)
to seed this repo. From here on the flow is the other way: this folder is
authoritative and Claude Design imports it. Its own
[`readme.md`](src/claude-design-system/readme.md) is the full reference —
brand rules, content voice, visual foundations, iconography and a component
index. Consumers link `styles.css`; components are bundled into `_ds_bundle.js`
and exposed on `window.AraCreateDesignSystem_9b36ca`.

## Commands

```
make help       # Default target — banner and target list
make list       # List the design systems in src/
make release    # Cut a semantic release
make clean      # Remove OS and editor cruft
```

## Conventions

See [aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions).

Project-specific rule: those conventions — file headers, naming, README casing —
apply to the **repo root only**. The contents of each folder under `src/` are
kept in their platform's own format, byte-for-byte as that platform expects, and
are not reformatted to match. `src/claude-design-system/` in particular is read
directly by Claude Design; restructuring it breaks that.

## License

Proprietary. Copyright (C) 2026, araCreate Group. See [LICENSE](LICENSE).
