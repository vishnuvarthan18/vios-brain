# ARM Platform — Kafka / Message Queue Architecture

Kafka on this platform has exactly one job: carrying email-send requests from `arm-core-be` to `arm-service-notification`. It is not a general-purpose event bus — nothing else on the platform produces or consumes Kafka messages. **Do not confuse this with BullMQ**, the separate Redis-backed queue system `calendar-be` uses extensively for sync/webhook/poll jobs ([SYNC-FLOW.md](calendar/SYNC-FLOW.md) §2, [REDIS-ARCHITECTURE.md](REDIS-ARCHITECTURE.md)) — the two share no infrastructure, no code, and not even the same failure-handling philosophy; calendar-be's own DLQ mechanism (`events/dlq.service.ts`) is BullMQ-based and unrelated to the `notification.dlq` Kafka topic this document covers.

Verified against `core/arm-core-be/src/libs/kafka/`, `services/arm-service-notification/src/`, and `deploy/arm-deploy-make`'s `make/infra.mk` + Kafka compose blocks, 2026-08-11.

---

## 1. Topology

```text
core-be (producer, fire-and-forget)
    ↓ send_welcome_email
    ↓ send_login_otp_email
Kafka
    ↓
notification-service (consumer)
    → renders Handlebars template → sends via SMTP (nodemailer)
    ↓ (on failure)
notification.dlq
    ↓
notification-service's own DLQ consumer (logs only — no replay, no persistence)
```

Only two `package.json`s in the whole workspace reference `kafkajs` — `core/arm-core-be` and `services/arm-service-notification`. No other backend touches Kafka.

---

## 2. Producer — `core-be`

`libs/kafka/kafka.module.ts` registers a NestJS `ClientsModule` Kafka client: `clientId` from `KAFKA_CLIENT_ID` (default `nestjs-core-producer`), `brokers: [process.env.KAFKA_BROKERS || 'localhost:9092']`. **`KAFKA_BROKERS` is read as a single string, not split on commas** — even in production, where the env var is set to a comma-joined 3-broker list, only the first address is actually used as the connection seed (§6 covers why this mostly doesn't matter in practice, and when it would).

**`NotificationProducerService`** (`libs/kafka/producers/notification-producer.service.ts`) is the only thing that calls `.emit()`:

| Method | Topic | Payload | Triggered by |
|---|---|---|---|
| `sendWelcomeEmail(userId, email, name)` | `send_welcome_email` | `{ userId, email, name, type: 'welcome-email' }` | New-user auto-provision via email-OTP verify, Google login, Microsoft login |
| `sendLoginOtpEmail(email, otp)` | `send_login_otp_email` | `{ email, otp, type: 'login-email' }` (no `userId` — the account may not exist yet at OTP-request time) | Every `POST /v1/auth/login/email/request-otp` call |

**Fire-and-forget by explicit design**, not an oversight — every call site invokes these methods without `await`, and `emit()` itself only logs on failure (with the full payload, "so ops can manually replay the message if needed"). **The triggering HTTP request always succeeds regardless of whether the Kafka publish actually succeeded.** There is no outbox pattern, no DB record of a pending send, no retry queue on the producer side — if the publish itself fails (broker unreachable, 5s timeout exceeded), the OTP or welcome email is silently lost with only a log line as evidence, and the user-facing request that triggered it reports success. If "the OTP email never arrived" is reported and the notification-service side looks healthy, check core-be's own producer logs next — the loss may have happened before the message ever reached Kafka.

**Boot-time connection**: 10 retry attempts, jittered ~2s delay between attempts. If all 10 fail, core-be logs an error and **boots normally anyway** — a fully Kafka-unreachable core-be still serves HTTP traffic, just with every OTP/welcome-email publish failing silently per the paragraph above.

---

## 3. Consumer — `notification-service`

Also NestJS's Kafka transport (`@nestjs/microservices`, KafkaJS underneath), not a hand-rolled `consumer.run()`. `clientId: 'notification-service'`, `groupId` from `KAFKA_GROUP_ID` (default `notification-group`), connects to a single `KAFKA_BROKER` (singular env var — different name than core-be's `KAFKA_BROKERS`, worth knowing when setting env vars for this service specifically).

Two `@EventPattern` handlers in `app.controller.ts` — NestJS internally subscribes to whichever topics have a registered pattern and dispatches per-message:

- `send_welcome_email` → renders `template/welcome.hbs`
- `send_login_otp_email` → renders `template/login-otp.hbs`

Both go through `MailService` → `@nestjs-modules/mailer` → nodemailer, using `SMTP_HOST`/`SMTP_PORT`/`SMTP_USER`/`SMTP_PASS`. The Handlebars adapter runs in `strict: true` mode — a template referencing a variable the payload doesn't provide **throws** rather than rendering blank, which becomes a handled failure per §4, not a silently-broken email.

Each handler wraps its work in try/catch, times it with a Prometheus histogram, and increments `notification_kafka_messages_total{topic,status}` (`status` ∈ `success`/`error`/`dlq`) — this is the metric to graph if building alerting on notification volume/failure rate (see [MONITORING.md](MONITORING.md) for what's currently wired to Grafana and what isn't).

---

## 4. DLQ — application code, not a KafkaJS feature

`notification.dlq` is an ordinary Kafka topic that application code publishes to and consumes from — nothing Kafka-native about it.

**On any handler failure** (template render, SMTP send, anything else thrown), the `catch` block calls `DlqService.send()`, which builds `{ originalTopic, originalMessage, error: error.message, failedAt }` and emits it to `notification.dlq` via a dedicated second Kafka client (`clientId: 'notification-dlq-producer'`). If *this* publish also fails, that failure is only logged — a failure reporting a failure has no further fallback.

**Something does consume `notification.dlq`** — `DlqConsumerController` — but it does **nothing except log the message**. No re-processing, no persistence to a database, no re-publish/retry, no alerting hook. This keeps the topic itself from growing unbounded on the broker (offsets get committed), but a failed email's only durable trace is whatever log aggregation the platform has — there is currently no dashboard, alert, or replay tool built on top of `notification.dlq`'s contents.

---

## 5. Retry behavior — fail once, then DLQ

No KafkaJS-level per-message retry is configured on the consumer, and no app-level retry loop wraps the mail-send path. A single failure goes straight to the DLQ on the first attempt — there is no "try 3 times before giving up" step anywhere in this pipeline, unlike calendar-be's BullMQ-based jobs which do have per-queue retry/backoff configuration ([SYNC-FLOW.md](calendar/SYNC-FLOW.md) §2). If a transient SMTP blip is the actual cause of a failed send, this pipeline DLQs it immediately rather than self-healing on a retry — every DLQ'd message currently requires manual intervention to resend, since nothing automated replays from the DLQ.

---

## 6. Topic configuration

| Topic | Partitions | Replication factor (dev/local) | Replication factor (prod) |
|---|---|---|---|
| `send_welcome_email` | 1 | 1 | 3 |
| `send_login_otp_email` | 1 | 1 | 3 |
| `notification.dlq` | 1 | 1 | 3 |

Created by `deploy/arm-deploy-make`'s `infra-wait` (dev/local) or its dedicated prod counterpart (`_wait-kafka-init-prod`) — prod correctly uses `(secret removed)` to match the 3-broker cluster, not the same dev-oriented command run everywhere. **Every topic stays at 1 partition in every environment** — this caps per-topic consumer parallelism at one active consumer regardless of consumer-group size or broker count; the 3-broker prod cluster buys replication/availability, not throughput parallelism, for this specific workload. That's almost certainly fine given the actual volume (login OTPs and welcome emails, not high-throughput event streaming), but worth knowing if notification volume ever grows enough that partition count becomes the bottleneck rather than SMTP throughput.

---

## 7. KRaft mode — no Zookeeper anywhere

Confirmed via grep across every compose file in `deploy/arm-deploy-make` — zero references to Zookeeper. Kafka runs in KRaft (Kafka Raft) mode, combined broker+controller role, in both environments:

- **Dev/local** (`docker-compose.infra.yml`): single `kafka` service, `KAFKA_NODE_ID: 1`, `KAFKA_PROCESS_ROLES: broker,controller`, `KAFKA_CONTROLLER_QUORUM_VOTERS: 1@localhost:9093`, offsets-topic replication factor 1. (A comment in this file attributes KRaft to "Kafka 4.x" — the actual image is `apache/kafka:3.8.1`, which does support KRaft; the version number in that comment is stale, but the KRaft claim itself is accurate.)
- **Prod** (`docker-compose.infra.prod.yml`): three services (`kafka1`/`kafka2`/`kafka3`), each `broker,controller`, shared `KAFKA_CONTROLLER_QUORUM_VOTERS` across all three, `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3`, `KAFKA_DEFAULT_REPLICATION_FACTOR: 3`, `KAFKA_MIN_INSYNC_REPLICAS: 2`, `KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2` — a genuinely production-shaped 3-broker cluster, not a scaled-down dev config relabeled.

**Broker seed-list gap**: prod's app compose sets both `KAFKA_BROKER` (singular, one address) and `KAFKA_BROKERS` (plural, all 3 comma-joined) — but per §2, core-be only ever reads `KAFKA_BROKERS` as one unsplit string, and notification-service only reads `KAFKA_BROKER` (singular) — **neither client ever connects with more than one seed broker address**, regardless of which env var was intended to provide the full list. This is not necessarily broken day-to-day (KafkaJS discovers full cluster metadata from any single reachable seed broker after the initial connection succeeds), but it does mean there's no seed-level failover: if the specific broker named in the single-address env var is down at connection/reconnect time, the client can't fall back to trying `kafka2`/`kafka3` as an alternate entry point even though they're both listed in `KAFKA_BROKERS` and reachable.

---

## 8. Consumer crash and recovery — and a real gap in prod

**Per-message failures don't crash the process.** Every handler's try/catch always resolves normally (logged + DLQ'd), so a bad template render or SMTP rejection is contained to that one message.

**Broker-connectivity failures do crash the consumer.** This is a directly-documented, previously-verified finding (from the health-check fix done earlier in this documentation pass — see [MONITORING.md](MONITORING.md)): `KafkaJSNumberOfRetriesExceeded` kills the consumer connection with nothing else noticing, silently. This is exactly why `notification-service`'s health check was split into `/v1/health/live` (process alive) and `/v1/health/ready` (a real broker round-trip via a separate throwaway admin client) — a dead-consumer-but-alive-process state now correctly reports `ready: false`, distinguishable from a genuinely healthy service. Recovery is **not** self-healing in-process — nothing reconnects a died consumer connection on its own; the actual recovery mechanism is an external restart triggered by whatever's polling the readiness endpoint.

**This was a real gap, since fixed.** `docker-compose.yml`'s `notification-service` block had **no `healthcheck:` stanza at all** — dev and local-dev both correctly polled `/v1/health/ready` ([restart-services.md](runbooks/restart-services.md) §3), but prod, specifically the environment with the 3-broker cluster and the largest blast radius, had nothing triggering the restart that's the actual recovery path for this exact failure mode. Fixed 2026-08-11 in `deploy/arm-deploy-make`: added the same `wget -qO- http://localhost:4001/v1/health/ready` healthcheck (15s interval, 5s timeout, 10 retries, 60s start period) that dev/local-dev already use, verified against `docker compose config` and confirmed no other service `depends_on` `notification-service` (so this was a purely additive change with no ordering side effects). Production now has the same automated-restart recovery path the health-check split was originally built to enable.

**Offset commit behavior**: no explicit auto-commit config is set anywhere — KafkaJS's defaults apply, committing after each handler resolves. Since every handler's try/catch always resolves (never rethrows to the transport), a per-message failure is deliberately treated as "consumed" once it's logged/DLQ'd — this is by design, not an accidental gap. The one narrower risk case: an uncaught, connection-killing failure (the `KafkaJSNumberOfRetriesExceeded` scenario above) that crashes the whole consumer *before* a message's try/catch/DLQ path runs at all — whether that specific in-flight message's offset was already committed depends on KafkaJS's own auto-commit timing relative to the crash, not on this pipeline's own logic. This is a real but narrow edge case, not the common failure path (which is safely caught and DLQ'd every time).

---

## Related documents

- [SYNC-FLOW.md](calendar/SYNC-FLOW.md) §2 — BullMQ's queue/retry model, for contrast with this document's Kafka pipeline (different system, different failure philosophy)
- [REDIS-ARCHITECTURE.md](REDIS-ARCHITECTURE.md) — the Redis/BullMQ side this document deliberately doesn't cover
- [MONITORING.md](MONITORING.md) — the `notification-service` health-check fix (`KafkaHealthService`, live/ready split) referenced in §8, and current Grafana/alerting coverage (or lack of it) for the `notification_kafka_messages_total` metric from §3
- [INFRASTRUCTURE.md](INFRASTRUCTURE.md) — full compose-file inventory this document's topic/broker config was verified against
- [runbooks/restart-services.md](runbooks/restart-services.md) §3 — per-service healthcheck reference, including which environments actually poll notification-service's `/v1/health/ready`
- [ENV-VARS.md](ENV-VARS.md) — `KAFKA_BROKERS`/`KAFKA_BROKER`/`KAFKA_CLIENT_ID`/`KAFKA_GROUP_ID` reference
- [ADR-004](adr/004-bullmq-vs-kafka.md) — why Kafka is scoped to notifications only, and BullMQ handles calendar sync instead
