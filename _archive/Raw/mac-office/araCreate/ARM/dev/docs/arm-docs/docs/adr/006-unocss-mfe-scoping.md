# ADR-006 — UnoCSS per-app instances with cross-MFE source scanning

**Status**: Decided, implemented — with one verified-broken path in the implementation.

## Context

Each of the three frontend apps needs utility-class CSS generation, but they're independently built (§ ADR-002) and two of them (`calendar-fe`, `admin-fe`) ship their own bundled CSS rather than relying on the shell to generate it for them.

## Decision

Each app runs its own UnoCSS instance rather than a single shared instance across the whole platform. `core-fe` additionally cross-scans `calendar-fe`'s source (not `admin-fe`'s) so that utility classes used inside calendar's federated components — which core doesn't otherwise know about at build time — still generate correctly when calendar is loaded into the shell. `core-fe` and `calendar-fe` share the same design-token preset (`@aracreate/test-arm-ui`'s `acPreset()`); `admin-fe` uses UnoCSS's own `(secret removed)()` directly instead.

## Alternatives considered

No original decision document was found explaining why `calendar-fe` gets cross-scanned but `admin-fe` doesn't, or why `admin-fe` uses a different preset than the other two apps — [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §6 notes this divergence is "presumably deliberate... but no explanation for the choice exists anywhere in either app's `CLAUDE.md`." Recorded here as an open question, not a verified rationale.

## Consequences

- **The cross-scan path for calendar is verified broken in one of its two config locations.** `core-fe/vite.config.ts`'s UnoCSS plugin `include` glob resolves to `core/calendar/src` — a directory that doesn't exist in this workspace (the real path is `apps/arm-app-calendar/src/frontend/src`). The *correct* path exists in the same app's separate `uno.config.ts` file, just not in `vite.config.ts`. Because Vite's build pipeline still transitively processes calendar's federated chunks regardless, this hasn't caused visible breakage historically — but the explicit cross-scan configuration is currently a no-op in the one place it's supposed to matter most. See [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §6 for the exact fix.
- **`admin-fe`'s utility classes aren't guaranteed to resolve to the same design tokens** (colors, spacing) as the rest of the shell, since it doesn't use the shared preset the other two apps do. No visual mismatch has been reported as of this writing, but nothing prevents one — this is a latent risk, not a confirmed bug.
- **Two apps (`calendar-fe`, `admin-fe`) ship a single bundled CSS file instead of relying on cross-scanning at all** — this sidesteps the whole scanning-accuracy problem for those two apps' *own* styles, at the cost of the CSS-isolation complexity documented in [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §5 (the standalone-vs-federated split that causes calendar's FullCalendar theme overrides to never load in production).

## Related documents

- [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §5–§6 — full CSS isolation strategy and the verified broken scan path
- [`troubleshooting/mfe-not-loading.md`](../troubleshooting/mfe-not-loading.md) §5 — the operational symptom and fix for the broken path
