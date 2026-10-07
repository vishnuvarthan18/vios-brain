# ARM Infrastructure Plan — Multi-Server + Hardening + arm-cli

**Date:** 2026-08-24
**Status:** Planning
**Current state:** All ~25 containers on a single Hetzner server (~13.3 GB)
**Target state:** 4 hardened servers on Hetzner private network + arm-cli as the platform CLI

---

## Architecture Overview

```
                     Hetzner Cloud Network (10.0.1.0/24)
    ┌────────────────┬────────────────┬────────────────┬─────────────────┐
    │                │                │                │                 │
┌───▼────┐     ┌─────▼─────┐   ┌──────▼──────┐   ┌────▼────┐    ┌──────▼──────┐
│   S1   │     │    S2     │   │     S3      │   │   S4    │    │  Hetzner    │
│Database│     │   App +   │   │ Observ-     │   │ Backup  │    │  Object     │
│        │     │   Redis   │   │ ability     │   │         │    │  Storage    │
│Postgres│     │ Caddy     │   │ Prometheus  │   │ pg_dump │    │            │
│MongoDB │     │ Kong      │   │ Grafana     │   │mongodump│    │  Offsite   │
│Kafka   │     │ Authelia  │   │ Loki/Tempo  │   │ Restore │    │  backups   │
│        │     │ 8 apps    │   │ Alloy       │   │ drills  │    │            │
│        │     │ Redis     │   │ Exporters   │   │         │    │            │
└────────┘     └───────────┘   └─────────────┘   └─────────┘    └────────────┘
No public IP   Public IP       Optional pub IP   No public IP    S3 API
10.0.1.1       10.0.1.2        10.0.1.3          10.0.1.4
```

Redis on S2 (not S1) — session service hits Redis every request; same-host = microseconds, cross-network = 0.2-0.5ms.

---

## Plan Documents

| # | Document | What it covers |
|---|---|---|
| 1 | [01-PRE-SPLIT-FIXES.md](01-PRE-SPLIT-FIXES.md) | Risks to fix on the current single server before migration |
| 2 | [02-MULTI-SERVER-COMPOSE.md](02-MULTI-SERVER-COMPOSE.md) | New compose files, env var changes, monitoring config for 4-server split |
| 3 | [03-NETWORK-AND-FIREWALL.md](03-NETWORK-AND-FIREWALL.md) | Hetzner network setup, UFW rules per server, Cloud Firewall |
| 4 | [04-SERVER-HARDENING.md](04-SERVER-HARDENING.md) | OS hardening, Docker hardening, kernel tuning, fail2ban, swap |
| 5 | [(secret removed)]((secret removed)) | Postgres/MongoDB/Redis auth, credential rotation, encryption |
| 6 | [(secret removed)]((secret removed)) | Secrets management, Caddy auto-TLS, internal traffic |
| 7 | [07-BACKUP-AND-DR.md](07-BACKUP-AND-DR.md) | Backup chain, restore drills, disaster recovery runbook per server |
| 8 | [08-CICD-AND-DEPLOY.md](08-CICD-AND-DEPLOY.md) | Multi-server deploy workflow, rollback, CI/CD security hardening |
| 9 | [09-MONITORING-AND-ALERTS.md](09-MONITORING-AND-ALERTS.md) | Security alerts, log aggregation, Grafana access |
| 10 | [10-MAINTENANCE.md](10-MAINTENANCE.md) | Automated, monthly, quarterly, annual maintenance schedules |
| 11 | [11-ARM-CLI.md](11-ARM-CLI.md) | arm-cli phases: scaffold, validate, check — the full platform CLI |
| 12 | [12-EXECUTION-CHECKLIST.md](12-EXECUTION-CHECKLIST.md) | Step-by-step cutover sequence with checkboxes |

---

## Server Sizing

| Server | Hetzner Type | RAM | vCPU | Disk | Public IP | Est. Monthly |
|---|---|---|---|---|---|---|
| S1 — DB | CX32 | 16 GB | 4 | 80 GB NVMe | No | ~15-30 |
| S2 — App | CX22 | 8 GB | 4 | 80 GB NVMe | Yes | ~10-20 |
| S3 — Observe | CX22 | 8 GB | 4 | 80 GB NVMe | Optional | ~10 |
| S4 — Backup | CX11 | 4 GB | 2 | 40 GB + Volume | No | ~5 + storage |

**Total: ~45-70/month**

---

## Responsibility Split

| arm-cli (developer laptop) | arm-deploy-make (servers) |
|---|---|
| Scaffold new repos | Generate compose files from descriptors |
| Validate repos & descriptors | Run compose (up/down/build) |
| Check server health (SSH) | CI/CD targets (deploy, rollback) |
| Port allocation | Kong/Caddy config rendering |
| Template conformance | Infra management (backup, restore) |

No Node.js on production servers. The descriptor contract is the clean boundary.
