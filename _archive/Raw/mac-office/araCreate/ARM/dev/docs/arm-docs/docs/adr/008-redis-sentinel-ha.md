# ADR-008 — Redis Sentinel for HA vs. Redis Cluster

**Status**: Decided, **partially implemented** — the topology exists and is opt-in, but application-level readiness for it is inconsistent across services. This ADR records the decision and flags the rollout gap explicitly.

## Context

Redis backs sessions (`session`), OTP storage and the admin role cache (`core-be`/`admin-be`), and BullMQ queues plus SSE pub/sub (`calendar-be`) — see [REDIS-ARCHITECTURE.md](../REDIS-ARCHITECTURE.md) for the full inventory. Production needs an HA story for Redis that doesn't lose sessions/queue state on a single-node failure.

## Decision

Redis Sentinel HA, layered in as an **opt-in** topology (`docker-compose.infra.redis-sentinel.yml`, which replaces rather than supplements the default single-instance Redis service) — not Redis Cluster. See [INFRASTRUCTURE.md](../INFRASTRUCTURE.md) §5 for the container topology and failover parameters this decision produced.

## Why Sentinel over Cluster (as inferred from the platform's own usage pattern)

The platform's Redis usage is not sharded-data-shaped — every service reads/writes a manageable single-node-sized keyspace (sessions, caches, queues), not a dataset large enough to need horizontal partitioning. Sentinel gives automatic failover and a single logical Redis endpoint without the added complexity of Cluster's hash-slot model, which would additionally require every client to be cluster-aware (multi-key operations, `MULTI`/transactions, and Lua scripts all behave differently under Cluster). No source document was found stating this reasoning explicitly — recorded as the most plausible inference from the implemented topology, not a verified rationale.

## Consequences — the rollout is incomplete, verified directly against each service's Redis client code

**Only 2 of the 4 Redis-using services are actually Sentinel-aware.** `core-be` and `calendar-be` construct their Redis clients with Sentinel awareness and would follow a failover correctly. **`session` and `admin-be` do not** — both are configured against a fixed `host`/`port`, and if that host stops being the master during a failover, both keep trying to reach a node that's no longer accepting writes (or is down entirely) until someone manually updates `REDIS_HOST`/`REDIS_PORT` and restarts them.

**This is the platform's single highest-severity Redis finding**, per [REDIS-ARCHITECTURE.md](../REDIS-ARCHITECTURE.md) §8: a production Sentinel failover, if triggered today, would break every active session platform-wide (`session` owns session state) with no automatic recovery, plus the admin disable/enable flow (`admin-be`). Sentinel HA is opt-in and not yet turned on for production as of this writing — which is exactly why this gap must be closed **before** it is turned on, not discovered after a real failover in production.

**Not the same problem as `session`'s own documented Redis Cluster/replica scaling roadmap** — that roadmap (in `session`'s own `CLAUDE.md`) is about read-throughput scaling (read replicas before Cluster), and doesn't mention Sentinel failover awareness at all. Two genuinely separate Redis-resilience conversations that shouldn't be conflated when planning the fix.

## Related documents

- [REDIS-ARCHITECTURE.md](../REDIS-ARCHITECTURE.md) §8 — the full per-service Sentinel-awareness verification and evidence
- [INFRASTRUCTURE.md](../INFRASTRUCTURE.md) §5 — Sentinel HA container topology, failover parameters, opt-in activation
