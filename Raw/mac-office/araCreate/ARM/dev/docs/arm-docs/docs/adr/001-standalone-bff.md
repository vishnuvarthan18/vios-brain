# ADR-001 — Standalone `arm-bff` vs. embedding BFF in `arm-core-be`

**Status**: Decided, implemented. **Date**: pre-dates this documentation pass; source plan dated in `docs/architecture/session-architecture.md`.

> **Naming, since renamed.** This record deliberately keeps the language of the decision as it was made — the service was called `arm-bff` and the pattern "BFF". It is now **`arm-session`**, living at `core/arm-session`, and the platform rule reads "one session service serves all". The decision itself is unchanged; only the name is. See [ADR-009](009-rename-bff-to-session.md). Note the GitHub repository is still `arm-core-bff` — the rename never reached the remote.

## Context

The platform is a multi-repo workspace with more than one frontend+backend pair — `core` (shell), `calendar`, `admin`, and room for more. Each needs session management and a way to call its own backend without exposing tokens to the browser (IETF's `draft-ietf-oauth-browser-based-apps` and OWASP both recommend a Backend-for-Frontend pattern for exactly this). The question was where that BFF should live.

## Decision

One standalone BFF (`arm-bff`) serves **every** frontend app on the platform — not one BFF per app, and not a BFF embedded inside `arm-core-be`.

## Alternatives considered

- **BFF embedded in `arm-core-be`**: rejected. Every other app's backend (`calendar-be`, `admin-be`) would have had to route through `core-be` as a proxy for their own traffic, since the session cookie would be owned by core-be alone. This makes `core-be` a "god proxy" that every new app adds routes to, blocks independent per-app deploys (a calendar release would require touching core), and bleeds token scope across services with no isolation. Directly violates the BFF pattern's own premise — one BFF per client, not one client's backend acting as BFF for everyone.
- **One BFF per app** (`calendar-bff`, `admin-bff`, etc.): considered and rejected — this is the platform's own explicit, standing rule (root [CLAUDE.md](../../../../CLAUDE.md), in its current wording: "Do not create a `calendar-session` or any other per-app session service. One session service serves all."). A single shared BFF means one session model, one cookie, one place tokens ever touch server-side memory, and Module Federation remotes (which share the shell's browser session) don't need to reconcile multiple BFF cookies.

## Consequences

- **Adding a new backend to the platform means adding one line** to the BFF's `SERVICE_ENV_MAP`, not standing up new session infrastructure — see [`core/arm-session/README.md`](../../../../core/arm-session/README.md)'s "Proxying to backends."
- **The BFF is a single point of coupling for every frontend** — an outage there affects every app, not just one. Mitigated by statelessness (Redis holds all session state, so any number of BFF instances can run behind a load balancer) rather than by splitting the BFF up.
- **East-west calls (BFF → backend) go direct, never through Kong** — Kong is edge-only for external traffic; see [KONG-CONFIG.md](../KONG-CONFIG.md) for what Kong actually fronts instead (a separate, direct-to-backend API surface, not this BFF's path).

## Related documents

- [`docs/architecture/session-architecture.md`](../architecture/session-architecture.md) — the original research/proposal this decision was made from
- [SESSION-FLOW.md](../SESSION-FLOW.md) — the implemented session lifecycle
- [`core/arm-session/README.md`](../../../../core/arm-session/README.md) — implementation guide
