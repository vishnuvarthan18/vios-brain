# 07 — Backup & Disaster Recovery

---

## 7.1 Backup Chain

```
S1 (Database)
  │
  ├── daily 02:00 ──→ S4 (Backup Server)
  │                     ├── /mnt/backups (14-day retention)
  │                     └── Hetzner Object Storage (30-day lifecycle)
  │
  └── weekly ──→ S4 restore drill (automated)
```

## 7.2 What Gets Backed Up

| Data | Method | Frequency | Location |
|---|---|---|---|
| PostgreSQL (arm_core, arm_admin) | `pg_dump` over network | Daily | S4 + Object Storage |
| MongoDB (arm-calendar) | `mongodump` over network | Daily | S4 + Object Storage |
| Redis | Not backed up | — | Sessions ephemeral, BullMQ replayable |
| Kafka | Not backed up | — | Fire-and-forget notifications |
| Caddy TLS certs | Docker volume on S2 | — | Caddy re-provisions if lost |
| `.env` files | Password manager + GitHub Secrets | Manual | — |
| Grafana dashboards | JSON in git | Every deploy | Git |

## 7.3 Automated Restore Drill (S4, Weekly)

1. Pull latest `arm_core` dump from `/mnt/backups`
2. Restore into throwaway database on S4's local PostgreSQL
3. Verify table count, row count
4. Drop throwaway database
5. Repeat for MongoDB
6. Email result to `GRAFANA_ALERT_EMAIL`

---

## 7.4 Disaster Recovery Runbook

### S1 (Database) Dies

| Step | Action |
|---|---|
| 1 | Provision new CX32, attach to `arm-private` at 10.0.1.1 |
| 2 | Run OS hardening ([04-SERVER-HARDENING.md](04-SERVER-HARDENING.md)) |
| 3 | Install Docker |
| 4 | Pull latest backup from S4 or Object Storage |
| 5 | Start DB containers, restore data |
| 6 | Verify from S2: health checks pass |
| **RTO** | ~1 hour |
| **RPO** | Up to 24 hours (last backup) |

### S2 (App) Dies

| Step | Action |
|---|---|
| 1 | Provision new CX22 with public IP |
| 2 | Harden + Docker |
| 3 | `make app-deploy-multi` |
| 4 | Update DNS A record |
| 5 | Caddy auto-provisions new TLS cert |
| **RTO** | ~30 min |
| **RPO** | Zero (stateless) |

### S3 (Observe) Dies

| Step | Action |
|---|---|
| 1 | Provision new CX22, deploy observability |
| 2 | Historical data lost (acceptable) |
| **RTO** | ~20 min |

### S4 (Backup) Dies

| Step | Action |
|---|---|
| 1 | Provision new CX11 + volume |
| 2 | Object Storage still has all backups |
| **RTO** | ~15 min, no data loss |
