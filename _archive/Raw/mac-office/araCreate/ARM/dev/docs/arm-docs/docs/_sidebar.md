- [Home](/)

- Platform Reference

  - [Architecture](ARCHITECTURE.md)
  - [Auth Architecture](AUTH-ARCHITECTURE.md)
  - [Session & Proxy Flow](SESSION-FLOW.md)
  - [Module Federation](MODULE-FEDERATION.md)
  - [Database Design](DATABASE-DESIGN.md)
  - [Redis Architecture](REDIS-ARCHITECTURE.md)
  - [Kafka Architecture](KAFKA-ARCHITECTURE.md)
  - [Security](SECURITY.md)
  - [Env Vars](ENV-VARS.md)
  - [Coding Standards](CODING-STANDARDS.md)
  - [Testing Strategy](TESTING-STRATEGY.md)
  - [Template Conformance](TEMPLATE-CONFORMANCE.md)
  - [Onboarding](ONBOARDING.md)

- Infrastructure & Delivery

  - [Infrastructure](INFRASTRUCTURE.md)
  - [Deploy Orchestration](DEPLOY-ORCHESTRATION.md)
  - [Kong Configuration](KONG-CONFIG.md)
  - [CI/CD](CICD.md)
  - [Release Process](RELEASE-PROCESS.md)
  - [Monitoring](MONITORING.md)
  - [Production Resilience Notes](PRODUCTION-RESILIENCE-NOTES.md)

- Audits & Risk Registers

  - [Production Audit (2026-08-21)](PRODUCTION-AUDIT-2026-08-21.md)
  - [Failure Analysis (2026-08-21)](FAILURE-ANALYSIS-2026-08-21.md)
  - [Infrastructure Risks](INFRASTRUCTURE-RISKS.md)
  - [API Surface Review](API-SURFACE-REVIEW.md)

- Roadmap & Plans

  - [Implementation Roadmap](IMPLEMENTATION-ROADMAP.md)
  - [Phase 1 — Emergency Stabilization](roadmap/PHASE-1-EMERGENCY.md)
  - [Phase 2 — Security Hardening](roadmap/PHASE-2-SECURITY.md)
  - [Phase 3 — Infrastructure Right-Sizing](roadmap/PHASE-3-INFRA-TUNING.md)
  - [Phase 4 — Sync Engine Hardening](roadmap/PHASE-4-SYNC-ENGINE.md)
  - [Phase 5 — Feature Development](roadmap/PHASE-5-FEATURES.md)
  - [Phase 6 — Operational Maturity](roadmap/PHASE-6-OPERATIONS.md)
  - [Phase 7 — UX Bug Fixes](roadmap/PHASE-7-UX-BUGS.md)

- Multi-Server Migration

  - [Migration Plan](MULTI-SERVER-MIGRATION.md)
  - [Infra Plan — Index](plan/infra/README.md)
  - [01 — Pre-Split Fixes](plan/infra/01-PRE-SPLIT-FIXES.md)
  - [02 — Multi-Server Compose](plan/infra/02-MULTI-SERVER-COMPOSE.md)
  - [03 — Network & Firewall](plan/infra/03-NETWORK-AND-FIREWALL.md)
  - [04 — Server Hardening](plan/infra/04-SERVER-HARDENING.md)
  - [05 — Database Security](plan/infra/05-DATABASE-SECURITY.md)
  - [06 — Secrets & TLS](plan/infra/06-SECRETS-AND-TLS.md)
  - [07 — Backup & DR](plan/infra/07-BACKUP-AND-DR.md)
  - [08 — CI/CD & Deployment](plan/infra/08-CICD-AND-DEPLOY.md)
  - [09 — Monitoring & Alerts](plan/infra/09-MONITORING-AND-ALERTS.md)
  - [10 — Maintenance Schedule](plan/infra/10-MAINTENANCE.md)
  - [11 — arm-cli Platform CLI](plan/infra/11-ARM-CLI.md)
  - [12 — Execution Checklist](plan/infra/12-EXECUTION-CHECKLIST.md)

- Architecture Decision Records

  - [Index](adr/readme.md)
  - [ADR-001: Standalone Session Service](adr/001-standalone-bff.md)
  - [ADR-002: Module Federation](adr/002-module-federation.md)
  - [ADR-003: MongoDB for Calendar](adr/003-mongodb-for-calendar.md)
  - [ADR-004: BullMQ vs Kafka](adr/004-bullmq-vs-kafka.md)
  - [ADR-005: OAuth Popup](adr/005-google-oauth-popup.md)
  - [ADR-006: UnoCSS MFE Scoping](adr/006-unocss-mfe-scoping.md)
  - [ADR-007: Verdaccio (Superseded)](adr/007-verdaccio-registry.md)
  - [ADR-008: Redis Sentinel HA](adr/008-redis-sentinel-ha.md)
  - [ADR-009: Rename BFF to Session](adr/009-rename-bff-to-session.md)

- Architecture Proposals

  - [Session Service Architecture](architecture/session-architecture.md)
  - [Admin Portal Proposal](architecture/admin-portal-proposal.md)
  - [Infrastructure Auto-Tune Design](architecture/infra-auto-tune-design.md)
  - [Object Storage Design](architecture/object-storage-design.md)
  - [SSO/OTP Auth Tasks (stale snapshot)](architecture/sso-otp-auth-tasks.md)
  - [Observability Plan (raw HTML)](architecture/observability-plan.html)

- Guides

  - [Local Dev Setup](guides/local-dev-setup.md)
  - [GitHub Setup](guides/github-setup.md)
  - [npm Publish](guides/npm-publish.md)

- Runbooks

  - [Restart Services](runbooks/restart-services.md)
  - [Deploy Rollback](runbooks/deploy-rollback.md)
  - [Database Backup and Restore](runbooks/db-backup-restore.md)
  - [Database Server (S1) Setup](runbooks/db-server-setup.md)

- Troubleshooting

  - [Calendar Sync Not Working](troubleshooting/calendar-sync-not-working.md)
  - [Authentication Failures](troubleshooting/auth-failures.md)
  - [MFE Not Loading](troubleshooting/mfe-not-loading.md)

- Per-Service Reference

  - [core-be API](core-be/API.md)
  - [Calendar: Architecture](calendar/ARCHITECTURE.md)
  - [Calendar: Sync Flow](calendar/SYNC-FLOW.md)
  - [Calendar: Webhooks](calendar/WEBHOOKS.md)
  - [Calendar: Data Model](calendar/DATA-MODEL.md)
  - [Calendar: API Reference](calendar/API.md)
  - [Calendar: Linked Accounts](calendar/LINKED-ACCOUNTS.md)
  - [Calendar: Google OAuth](calendar/GOOGLE-OAUTH.md)
  - [Calendar: Delete Cascade](calendar/DELETE-CASCADE.md)

- History

  - [BFF Rename — Reference Inventory](BFF-RENAME-INVENTORY.md)
  - [BFF Rename — Phased Task Sheet](BFF-RENAME-TASKS.md)
  - [BFF Rename — Phase 0 Record](BFF-RENAME-P0-RECORD.md)

- Meta

  - [Documentation Inventory](meta/documentation-inventory.md)
  - [Miro Diagrams](meta/miro-diagrams.md)
