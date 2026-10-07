# ADR-009 — Rename the BFF to `arm-session`

**Status**: Decided, implemented. **Date**: 2026-08-25.
**Supersedes the naming in** [ADR-001](001-standalone-bff.md), whose architectural decision stands unchanged.

## Context

The platform's browser-facing session layer was named `arm-core-bff` on GitHub, `arm-bff`
on disk, and `bff` as an orchestrator service key. Three problems accumulated:

1. **"BFF" is jargon.** A newcomer reading the repo list learns nothing from it.
2. **The `core-` prefix was architecturally wrong.** The service is platform-wide — every
   MFE shares it — but the prefix implied it belonged to Core alongside `arm-core-be` and
   `arm-core-fe`.
3. **`arm-core-be` and `arm-core-bff` differ by one trailing character**, which is a real
   misread hazard.

The name also disagreed with itself: the GitHub repo, the directory, the package name and
the service key were four different strings.

## Decision

Rename the service to **`arm-session`** — service key `session`, env prefix `SESSION_`,
route prefix `/session/*`.

The name states the architectural invariant the platform already enforces: this service is
the sole owner of the browser session. Its three responsibilities — session lifecycle,
OAuth brokering, and the authenticated reverse proxy — are all consequences of that.

### Names ruled out

- **`arm-gateway`** — "gateway" already denotes Kong, the API edge behind Caddy. A second
  "gateway" would be worse than the jargon it replaced.
- **Any `/api/*` route prefix** — `/api/*` is owned by Caddy → Kong. The session service's
  prefix exists precisely to *bypass* Kong.

### Scope

All seven layers, including runtime state. Redis key prefixes moved from `bff:*` to
`session:*` with **no dual-read migration**, which makes the cutover a flag day: every
logged-in user is signed out once, and in-flight OAuth handoffs are dropped.

Two collisions shaped the final naming:

- `src/session/` already existed, so the old `src/bff/` was **merged into it** as one
  NestJS feature module rather than renamed onto it.
- `SESSION_SECRET` already existed (the cookie HMAC key), so it became
  `SESSION_COOKIE_SECRET` to sit unambiguously beside the new `SESSION_INTERNAL_SECRET`.

## Consequences

**The session cookie name is unchanged** (`SESSION_ID`) — it was already neutral, so no
extra forced logout came from that direction.

**Two external systems must be updated before cutover**, and they are the long pole:

- Google Cloud Console and Microsoft Entra hold the OAuth **redirect URIs**, which contain
  the route prefix. Until the `/session/...` paths are registered, every login and
  add-account attempt fails with `redirect_uri_mismatch`.
- GitHub Actions **secrets and variables** are referenced by name
  (`PROD_SESSION_INTERNAL_SECRET`, `PROD_SESSION_PUBLIC_URL`,
  `PROD_SESSION_COOKIE_SECRET`) and must be renamed in repository settings.

**Historical task IDs (`BFF-01`…`BFF-19`) were deliberately left alone.** They identify
work already done; renaming them would break cross-references for no gain.

**One irregularity survives.** `PROD_NAME` is `arm-session` — the prod compose *service
key* that `depends_on:` and Caddy reference — so it is still not `arm-` + service key, and
`PROD_CONTAINER_NAME` remains a separate explicit descriptor field. Normalising it is a
separate change with its own blast radius.

## Related documents

- [BFF-RENAME-INVENTORY.md](../BFF-RENAME-INVENTORY.md) — the full old→new mapping and the survey it was derived from
- [BFF-RENAME-TASKS.md](../BFF-RENAME-TASKS.md) — the phased execution plan and what each phase actually found
- [SESSION-FLOW.md](../SESSION-FLOW.md) — the implemented session lifecycle
- [ADR-001](001-standalone-bff.md) — the original decision to run this as a standalone service
