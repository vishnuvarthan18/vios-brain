# SOURCE

The service's own source — a NestJS microservice, no defined layout in the araCreate template (Node/NestJS isn't one of the covered project types), so this follows Nest's own module conventions instead:

```
src/
├── main.ts              # Microservice bootstrap (Kafka transport, no HTTP)
├── app.module.ts         # Root module — wires ClientsModule, MailModule
├── app.controller.ts     # Kafka @EventPattern handlers
├── app.service.ts        # Notification logic (welcome / login-OTP emails)
├── mail/                 # MailerModule config + MailService
├── kafka/                # Dead-letter queue service
└── lib/                  # Shared types and the retry-with-backoff helper
```

No third-party code is vendored here.
