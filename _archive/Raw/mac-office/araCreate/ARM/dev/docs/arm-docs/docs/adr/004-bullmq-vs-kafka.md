# ADR-004 — BullMQ for calendar sync jobs, Kafka reserved for notifications

**Status**: Decided, implemented. Two separate queue systems coexist on the platform by design, not by accident — this ADR records why they weren't unified onto one.

## Context

The platform needs background job processing for two very different workloads: `calendar-be`'s sync engine (initial sync, webhook processing, polling, teardown — all Redis-adjacent, already depending on Redis for sessions/caching elsewhere) and `core-be` → `notification-service`'s fire-and-forget email delivery (a simple one-hop pipeline with no interdependency on the rest of the platform's Redis usage).

## Decision

**BullMQ** (Redis-backed) for every calendar sync/webhook/teardown job. **Kafka** exclusively for the `core-be` → `notification-service` email pipeline. The two systems share no infrastructure, no code, and — notably — not even the same failure-handling philosophy (see Consequences).

## Alternatives considered

- **Kafka for calendar sync jobs too**: would have unified the platform onto one queue technology, but calendar-be's workload is fundamentally job-queue-shaped (retry-with-backoff, delayed/repeatable jobs, per-job state tracking, a real dead-letter queue with configurable policy) rather than event-stream-shaped. BullMQ is purpose-built for exactly this; Kafka's consumer-group/partition model is a better fit for the simpler, higher-throughput-potential, at-least-once email pipeline it's actually used for. Not adopted for calendar-be.
- **BullMQ for notifications too**: would have avoided standing up Kafka infrastructure at all (one broker, later a 3-node KRaft cluster in production — see [KAFKA-ARCHITECTURE.md](../KAFKA-ARCHITECTURE.md) §7) for a comparatively low-volume workload. Not chosen — no source document explains why Kafka specifically was picked over just reusing BullMQ here; treat this half of the decision as unexplained rather than justified, unlike the calendar-side reasoning above.

## Consequences

- **Two DLQ mechanisms exist, shaped completely differently, and must not be conflated**: calendar-be's DLQ (`events/dlq.service.ts`) is BullMQ's own `OnQueueEvent('failed')` mechanism, moving exhausted-retry jobs into a dedicated BullMQ queue — see [SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §2. `notification.dlq` is a completely separate, hand-rolled Kafka topic with its own producer/consumer application code — see [KAFKA-ARCHITECTURE.md](../KAFKA-ARCHITECTURE.md) §4. Neither replays into the other; neither shares tooling with the other.
- **Retry philosophy differs materially between the two systems** — most BullMQ queues on the calendar side have real per-queue retry/backoff configuration (though notably `initial-sync` itself doesn't — a verified gap, [SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §2), while the Kafka notification pipeline is fail-once-then-DLQ everywhere, with zero per-message retry ([KAFKA-ARCHITECTURE.md](../KAFKA-ARCHITECTURE.md) §5). An engineer moving between the two codepaths should not assume either's failure-handling conventions carry over to the other.
- **Operational knowledge doesn't transfer either** — debugging a stuck calendar sync means BullMQ tooling (Redis inspection, Bull Board if wired up, `[REDIS-ARCHITECTURE.md]`); debugging a lost notification email means Kafka tooling (consumer group offsets, topic inspection) and reading application logs, since the DLQ consumer itself only logs rather than exposing state anywhere queryable.

## Related documents

- [SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §2 — BullMQ queue configuration, retry/backoff per queue
- [KAFKA-ARCHITECTURE.md](../KAFKA-ARCHITECTURE.md) — the Kafka pipeline in full, explicitly contrasted against BullMQ throughout
- [REDIS-ARCHITECTURE.md](../REDIS-ARCHITECTURE.md) — Redis/BullMQ infrastructure shared with the rest of calendar-be's Redis usage
