# Wedding2day (W2D)

B2B trade connection platform for the Tamil Nadu wedding industry (Expo + Firebase).

## Authoritative docs

**`DECISIONS.md` is the single source of truth** for product and architecture decisions. If anything else disagrees with it, `DECISIONS.md` wins.

| File | Role |
|---|---|
| [`DECISIONS.md`](./DECISIONS.md) | Locked decisions — what to build, what not to |
| [`PRODUCT_CONTEXT.md`](./PRODUCT_CONTEXT.md) | Why those decisions exist (companion / reasoning) |

Older v1 docs live under `docs/archive/` with a `SUPERSEDED_` prefix. Do not follow them.

## Project structure

```
w2d/
├── DECISIONS.md              # authoritative decisions (read first)
├── PRODUCT_CONTEXT.md        # reasoning behind decisions
├── AGENTS.md / CLAUDE.md     # agent guidance
├── app/                      # Expo Router app (screens, layouts)
├── constants/                # shared constants (districts, listing enums)
├── lib/                      # Firebase and shared libs
├── scripts/                  # seed and tooling scripts
├── docs/
│   └── archive/              # SUPERSEDED_*.md — old v1 scope (do not use)
├── desing/                   # design references / stitch exports
├── firestore.rules           # Firestore security rules
├── storage.rules             # Storage security rules
├── package.json
└── app.json
```
