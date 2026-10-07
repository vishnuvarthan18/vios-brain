# SOURCE

The service's own source — a NestJS application, no defined layout in the araCreate template (Node/NestJS isn't one of the covered project types), so this follows Nest's own module conventions instead:

```
src/
├── main.ts               # Bootstrap
├── app.module.ts          # Root module
├── auth/                  # Email-OTP + Google/Microsoft SSO login, JWT issuance, KongJwtGuard
├── users/                 # User CRUD (create/getAll/remove permanently disabled on the controller)
├── health/                 # /health endpoint (TypeORM + Redis indicators)
├── registry/               # Placeholder org/role management
├── database/
│   └── migrations/         # TypeORM migrations, run at startup
├── common/                 # Shared decorators and filters
├── libs/                   # Kafka producer, Redis service, (secret removed) token encryption
└── scripts/                # Deploy/startup helper scripts (not the root-level scripts/)
```

No third-party code is vendored here. See this repo's own [CLAUDE.md](../CLAUDE.md) for the full module breakdown.
