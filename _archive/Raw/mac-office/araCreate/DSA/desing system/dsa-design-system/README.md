# DSA Design System

The design system for **DreamSpace Academy (DSA)** — tokens, components,
foundations and brand assets for the DreamSpace Academy brand, kept under
version control so every platform that consumes them works from the same
source.

## What it holds

- **Design tokens** — colors, typography, spacing, radius and shadows, as CSS
  custom properties
- **Components** — React components with type definitions and usage docs
- **Foundations / guidelines** — preview cards for the palette, type scale,
  spacing, logo usage and the honeycomb brand device
- **UI kits** — recreations of the DreamSpace Academy marketing site and
  homepage
- **Assets** — logo lockups in colour, white and black variants
- **Sources** — the DreamSpace brand book everything traces back to

## Stack

| Layer | Tool |
| --- | --- |
| Tokens | CSS custom properties, entry point `styles.css` |
| Components | React (JSX) + `.d.ts` contracts |
| Preview cards | Static HTML |
| Typography | Poppins |
| Consumers | [Claude Design](https://claude.ai/design) — imports a snapshot of this repo |

## Layout

```
dsa-design-system/
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

Each folder under `src/` is self-contained and keeps its own platform's
layout, so it can be consumed directly with no build step in between.

### `src/claude-design-system`

Exported from the Claude Design project for the DreamSpace Academy brand.
This folder is authoritative and Claude Design imports it. Its own
[`readme.md`](src/claude-design-system/readme.md) is the full reference —
brand rules, content voice, visual foundations, iconography and a component
index.

## Commands

```
make help       # Default target — banner and target list
make list       # List the design systems in src/
make release    # Cut a semantic release
make clean      # Remove OS and editor cruft
```

## Conventions

See [aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions).

Project-specific rule: those conventions — file headers, naming, README
casing — apply to the **repo root only**. The contents of `src/` are kept in
their platform's own format, byte-for-byte as that platform expects, and are
not reformatted to match. `src/claude-design-system/` in particular is read
directly by Claude Design; restructuring it breaks that.

## License

Proprietary. See [LICENSE](LICENSE).
