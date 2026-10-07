# SOURCE

The admin backend's own source, structured by NestJS module:

```
src/
├── main.ts                             # Bootstrap: prefix, CORS, pipes, filter, port
├── app.module.ts                       # Root module — wires everything
├── config/                             # Joi env-var validation schema
├── database/
│   ├── entities/{core,admin}/          # TypeORM entities for both datasources (arm_core read, arm_admin owned)
│   ├── migrations/                     # arm_admin schema migrations (run on startup)
│   └── seeders/                        # First-admin seeder, deny-list Redis re-population
├── auth/
│   ├── guards/                         # KongJwtGuard, AdminRoleGuard
│   └── decorators/                     # @CurrentAdmin()
├── redis/                              # Deny-list service (disabled users blocked at the gateway)
├── users/                              # Admin user list/disable/enable endpoints
├── audit/                              # Audit log write + query endpoints
├── health/                             # /health endpoint
└── common/
    ├── filters/                        # TypeORM error → safe JSON response
    └── metrics/                        # Prometheus HTTP metrics middleware
```

See this app's own [CLAUDE.md](../CLAUDE.md) for the full architecture writeup — the dual TypeORM connection setup, the deny-list mechanism, and the auth guard stack.
