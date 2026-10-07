# ADR-007 — Verdaccio for internal package registry

**Status**: **Superseded.** Adopted at some point, then removed entirely — this ADR exists specifically to preserve that reversal, since nothing else in the current codebase records that Verdaccio was ever there in the first place.

## Original context and decision

`@aracreate/test-arm-ui` (the shared UI component library) needed somewhere to publish to that both CI and local developer machines could pull from without depending on the public npm registry. Verdaccio (a self-hosted, lightweight private npm proxy/registry) was adopted for this.

## What changed

Verdaccio has been **removed from the stack entirely**. `@aracreate/test-arm-ui` now publishes via plain `npm publish --access public` to the real public npm registry, not an internal proxy — confirmed in [`docs/guides/npm-publish.md`](../guides/npm-publish.md) and [INFRASTRUCTURE.md](../INFRASTRUCTURE.md) §1. `make/local-dev.mk`'s own header comment documents the removal explicitly, and `clone-repos`/`update-repos` now auto-detects and deletes any stale `.npmrc` file left over from when a service pointed at `localhost:4873` (Verdaccio's default port) — a cleanup step that only makes sense as residue from a real, once-live piece of infrastructure.

## Why this ADR exists at all

Without this record, "Verdaccio" appears nowhere except as a cleanup edge case in a Makefile comment — someone reading the codebase fresh would have no way to know an internal registry was ever part of the plan, why it was removed, or that `DOC-P25` (Verdaccio Internal Registry Setup) in the documentation inventory should be read as **obsolete**, not as a pending doc to write. That inventory entry is explicitly marked obsolete for exactly this reason.

## Consequences

- **No internal registry exists to document, and none is needed** — this closes DOC-P25 as a non-task rather than a gap.
- **Publishing the shared UI library now depends on real npm credentials/access**, not an internally-controlled proxy — a different trust and availability model than a self-hosted registry, not evaluated further here since no source document explains the reasoning behind the removal either (matching this ADR's own honesty pattern: the reversal is recorded as a fact, not as a reasoned trade-off, because no reasoning was found anywhere in the workspace).

## Related documents

- [`docs/guides/npm-publish.md`](../guides/npm-publish.md) — the current publish process
- [INFRASTRUCTURE.md](../INFRASTRUCTURE.md) §1 — where this removal was first verified
