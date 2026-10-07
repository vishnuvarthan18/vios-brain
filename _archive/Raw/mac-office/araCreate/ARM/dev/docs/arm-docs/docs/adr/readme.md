# Architecture Decision Records

Written retroactively (2026-08-11) for decisions already made and live in the codebase — not decisions pending. Each ADR records the context, the decision, its consequences, and what alternatives were considered, so a future engineer understands *why*, not just *what*. Full technical detail lives in the platform reference docs each ADR cross-links to; ADRs stay short on purpose.

| # | Title | Status |
|---|---|---|
| [ADR-001](001-standalone-bff.md) | Standalone `arm-session` vs. embedding session service in `arm-core-be` | Decided, implemented |
| [ADR-002](002-module-federation.md) | Module Federation over alternative MFE approaches | Decided, implemented |
| [ADR-003](003-mongodb-for-calendar.md) | MongoDB for calendar data vs. PostgreSQL | Decided, implemented |
| [ADR-004](004-bullmq-vs-kafka.md) | BullMQ for calendar sync jobs, Kafka reserved for notifications | Decided, implemented |
| [ADR-005](005-google-oauth-popup.md) | OAuth popup + BroadcastChannel pattern for linking calendar accounts | Decided, implemented (revised from the original postMessage proposal) |
| [ADR-006](006-unocss-mfe-scoping.md) | UnoCSS per-app instances with cross-MFE source scanning | Decided, implemented (with a known-broken path — see the ADR) |
| [ADR-007](007-verdaccio-registry.md) | Verdaccio for internal package registry | **Superseded** — adopted, then removed entirely |
| [ADR-008](008-redis-sentinel-ha.md) | Redis Sentinel for HA vs. Redis Cluster | Decided, partially implemented (2 of 4 services not Sentinel-aware) |
| [ADR-009](009-rename-bff-to-session.md) | Rename the BFF to `arm-session` — service key, env prefix and route prefix | Decided, implemented |

Two ADRs (007, 008) don't end at a clean "decided and done" — that's the point of keeping the record. A decision reversed (007) or a rollout left incomplete (008) is exactly the kind of thing an ADR should preserve instead of silently erasing.
