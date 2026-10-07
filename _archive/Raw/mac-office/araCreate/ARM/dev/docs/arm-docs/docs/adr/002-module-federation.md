# ADR-002 — Module Federation over alternative MFE approaches

**Status**: Decided, implemented.

## Context

The platform ships three independently-built frontend apps (`core-fe`, `calendar-fe`, `admin-fe`) that need to appear as one seamless application to the end user — a single shell with a shared navbar, auth state, and theme, into which calendar and admin UI mount as if native.

## Decision

Module Federation (`@module-federation/vite`), with `core-fe` as the sole host and `calendar-fe`/`admin-fe` as remotes exposing specific components (not whole apps) that the host lazy-loads and mounts inside routes it owns.

## Alternatives considered (as commonly weighed against MF at the time)

- **iframes**: would isolate each app cleanly but at the cost of a fully separate DOM/CSS/JS context per app — no shared auth state without extra plumbing, awkward routing (the shell's URL bar wouldn't reflect the embedded app's state naturally), and a visibly "embedded" feel rather than a seamless single application. Not chosen.
- **single-spa** (or similar meta-framework orchestration): a viable alternative with a similar "multiple independently-deployed apps, one shell" goal, but Module Federation's Webpack/Vite-native shared-dependency negotiation (§ below) and its more direct component-level `import()` API were the deciding factor over single-spa's more app-lifecycle-centric model. Not chosen.
- **A monorepo with one build**: would have eliminated the coordination problem the other options above have to solve, but at the cost of the platform's core organizing principle — independent per-app deploy and ownership (root [CLAUDE.md](../../../../CLAUDE.md) describes the workspace as "assembled from independent git repositories," not a monorepo). Not chosen.

## Consequences

- **Shared-dependency negotiation is real and needs active maintenance** — `react`/`react-dom` are marked `singleton: true` across host and both remotes, meaning the host's version wins negotiation regardless of what a remote's own `package.json` pins. This is a verified, currently-harmless drift (`19.2.0` host vs. `19.1.1` remotes) documented in [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §4 — the risk is latent, not realized, but requires someone to notice if the versions ever diverge in a breaking way.
- **CSS isolation had to be solved by hand** — MF doesn't scope CSS between host and remote automatically. The `.calendar-scope` wrapper convention and a standalone-only/federated-only split in `calendar-fe`'s own CSS (some rules only apply outside federation) is the platform's answer, and it has a known incomplete spot ([MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §5.2 — FullCalendar theme overrides never load in the federated path).
- **Two remotes reach back into the host at runtime** (`core/toast`, `core/theme-store`) with no static declaration and no compile-time contract check — a working pattern, but one that fails silently if the host ever renames or removes either export ([MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §3.1).
- **A cross-MFE UnoCSS scan path is required** for the host to generate utility classes used only in remote source — and this scan path has a verified bug (resolves to a nonexistent directory) that's gone unnoticed because the build still mostly works anyway ([MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §6).

## Related documents

- [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) — full implementation detail, all verified pitfalls
- [`troubleshooting/mfe-not-loading.md`](../troubleshooting/mfe-not-loading.md) — symptom-first triage for the consequences above
