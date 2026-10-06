# ARM Notification Service

Notification service for the ARM platform. Consumes Kafka events published by `arm-core-be` and sends transactional email via SMTP — login OTP codes and welcome emails today, nothing else. Stateless: no database, purely consume-render-send.

- Consumes events over Kafka (`@nestjs/microservices`, one consumer group: `notification-group`)
- Renders Handlebars templates (`template/login-otp.hbs`, `template/welcome.hbs`) and sends via SMTP (`@nestjs-modules/mailer`)
- Failed sends are retried with backoff on startup/connect, then routed to a dead-letter topic (`notification.dlq`) rather than dropped
- **Full Kafka topology, DLQ design, retry philosophy, and consumer-crash/recovery behavior**: [`KAFKA-ARCHITECTURE.md`](../../docs/arm-docs/docs/KAFKA-ARCHITECTURE.md) — not repeated in this README, which stays focused on what a developer working in this repo day-to-day needs

## Stack

| Layer | Tool |
| --- | --- |
| Framework | NestJS (hybrid: HTTP app + Kafka microservice in one process) |
| Messaging | Kafka (`@nestjs/microservices`, `kafkajs`) |
| Email | `@nestjs-modules/mailer` + Handlebars (`strict: true`), SMTP |
| Metrics | `@willsoto/nestjs-prometheus`, `/metrics` (internal-network only) |
| Language | TypeScript |
| Package manager | pnpm |
| Tests | Jest |

## Architecture

```text
Kafka topic → NotificationController → NotificationService → MailService → SMTP
                                                  │
                                                  └─ on failure → DlqService → notification.dlq
                                                                        │
                                                                        └─ same process, same consumer group,
                                                                           DlqConsumerController → logs only
```

**`main.ts` runs a hybrid app, not a microservice-only one**: `NestFactory.create()` builds a normal HTTP-capable Nest app first, then `connectMicroservice({ transport: Transport.KAFKA, ... })` attaches the Kafka consumer to the *same* app instance — `/v1/health/*` and `/metrics` (HTTP) and the three `@EventPattern()` handlers (Kafka) all run in one process, one consumer group. `DlqConsumerController` (logs `notification.dlq` messages, does nothing else — no replay, per `KAFKA-ARCHITECTURE.md` §4) is wired into the same module as `NotificationController`, so it's the *same* consumer group reading `notification.dlq` too, not a separate service or process.

- `src/app.module.ts` / `app.controller.ts` / `app.service.ts` — `NotificationController`'s two `@EventPattern()` Kafka handlers (§Kafka topics below), delegating to `NotificationService` → `MailService`
- `src/mail/` — `MailService` + `MailModule` (SMTP transport config, Handlebars adapter, template dir wiring)
- `src/kafka/dlq.service.ts` + `dlq-consumer.controller.ts` — publishes failed messages to `notification.dlq` (via a second, dedicated `ClientKafka`, `clientId: notification-dlq-producer`) and passively logs them back
- `src/health/` — `KafkaHealthService` (a real broker round-trip via a throwaway admin client, not a reused long-lived connection — see `KAFKA-ARCHITECTURE.md` §8 for why that distinction matters) + `HealthController` (`/v1/health/live`, `/v1/health/ready`)
- `src/lib/retry.ts` — shared retry-with-backoff helper used by both the bootstrap Kafka connection (`main.ts`) and `DlqService`'s own connect
- `src/common/metrics/` — `notification_kafka_messages_total{topic,status}` and `notification_kafka_message_duration_seconds{topic}` Prometheus providers, incremented in `NotificationController`

## Kafka topics this service consumes

All three via the single `notification-group` consumer group (`KAFKA_GROUP_ID`, default `notification-group`):

| Topic | Handler | Payload (`SendNotification`, `src/lib/interface.ts`) | On failure |
|---|---|---|---|
| `send_welcome_email` | `NotificationController.handleWelcomeEmail` | `{ userId?, email, name?, type? }` | Logged + published to `notification.dlq`, message still marked consumed (offset commits regardless — see `KAFKA-ARCHITECTURE.md` §8) |
| `send_login_otp_email` | `NotificationController.handleLoginOtpEmail` | `{ userId?, email, otp?, type? }` — `userId` is genuinely absent here: the login-OTP request happens before core-be knows whether the account exists | Same |
| `notification.dlq` | `DlqConsumerController.handleDlqMessage` | `{ originalTopic, originalMessage, error, failedAt }` (`DlqPayload`) | Logged only — this *is* the failure path, nothing downstream of it |

Both producer-side topics are published by `arm-core-be`'s `NotificationProducerService` — full producer-side detail (exact call sites, payload construction) is in `KAFKA-ARCHITECTURE.md` §2, not duplicated here since this is the consumer, not the producer.

## Email templates

Handlebars, `template/` **at the repo root**, not under `src/` (`MailModule`'s `template.dir` is `process.cwd() + '/template/'` — a build-relative path, worth knowing if the Docker build context or working directory ever changes, since a `dist/`-relative path would silently break template resolution). `options: { strict: true }` — a template referencing a context variable that isn't provided throws instead of rendering blank, so a new template must either always receive every variable it uses or guard optional ones with `{{#if}}` the way `welcome.hbs` already does for `name`.

| Template | Subject | Variables used |
|---|---|---|
| `template/login-otp.hbs` | "Your Login Code" | `{{otp}}`, `{{year}}` (both always supplied by `NotificationService.sendLoginOtpEmail`) |
| `template/welcome.hbs` | "Welcome to Our Service" | `{{name}}` (guarded with `{{#if}}` — genuinely optional), `{{year}}` |

`year` is stamped by `NotificationService` itself (`new Date().getFullYear()`), not part of the Kafka payload — every other field in `SendNotification` passes through to the template context as-is via `{ email, ...data }` destructuring, so a new payload field is automatically available in the template without any wiring change.

## Mail provider configuration

Plain SMTP via `@nestjs-modules/mailer`, one provider for every environment — no per-environment SMTP provider switch exists in code. `buildMailerConfig` (`mail.module.ts`) does one thing worth knowing if you're debugging a connection failure: `SMTP_PORT` is read as a string by `ConfigService.getOrThrow`, then explicitly `Number()`-coerced before comparing to `465` to set `secure: true` — the type parameter on `getOrThrow<T>` only asserts a type, it never coerces, so skipping that explicit `Number()` call would silently leave TLS off even on port 465. Full env var reference (which are secret, which are required): [ENV-VARS.md](../../docs/arm-docs/docs/ENV-VARS.md) §8 — not repeated here.

## Local dev — testing without a live Kafka broker or real SMTP

**There's no local SMTP catcher (Mailhog/Maildev) anywhere in this workspace** — confirmed, no such service in any compose file. Real email sending requires real `SMTP_*` credentials; skipping them at `make env-init` disables email platform-wide (per the workspace root `CLAUDE.md`), it doesn't fall back to a local catch-all inbox.

The practical way to exercise this service's logic without either a live broker or real SMTP creds is its own test suite (`pnpm test` / `make test`) — `NotificationController`'s Kafka handlers are tested directly (not over HTTP) against mocked `MailService`/`DlqService`, covering welcome-email send, login-OTP send, and the DLQ-routing-on-failure path in one file. See [TESTING-STRATEGY.md](../../docs/arm-docs/docs/TESTING-STRATEGY.md) §1 for how this compares to the platform's other test suites — this is one of the few "e2e"-named suites on the platform that's genuinely useful, not renamed boilerplate.

To exercise the real Kafka path locally, `make local-dev` (from `deploy/arm-deploy-make/`) brings up a real broker and this service together — trigger a message by going through the actual flow that produces one (e.g. logging in via email-OTP), or publish directly to `send_welcome_email`/`send_login_otp_email` with a Kafka console producer if you need to test a template change without a full login flow.

## Commands

```sh
make install    # install dependencies
make setup      # create .env from .env.example if missing
make dev        # run locally with hot reload
make build      # compile to dist/
make test       # unit tests
make test-e2e   # e2e tests
make lint       # eslint (check only, used in CI)
make lint-fix   # eslint --fix
make release    # cut a semantic release (CI-only — needs PAT_TOKEN)
make clean      # remove build artefacts
```

`make help` (default) prints the full target list.

In the full ARM workspace, this service normally runs via `make local-dev` from `deploy/arm-deploy-make/` — see the workspace [CLAUDE.md](../../CLAUDE.md) for the standard startup flow. This repo's own `Makefile` is a standalone entry point for working in this repo directly.

## Conventions

Repo conventions (file headers, naming, versioning, Makefile targets) follow [aracreate-template-codebase](../../aracreate-template-codebase). Template-conformance work for this repo is tracked in `TASK_SHEET.md`.

## License

Proprietary — see [LICENSE](./LICENSE), Copyright (C) 2026, araCreate Group.
