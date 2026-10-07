# 12 — Execution Checklist

> Step-by-step cutover sequence. Check off as you go.

---

## Pre-Migration (Current Single Server)

- [ ] Fix Redis eviction: `noeviction` → `volatile-lru` ([01](01-PRE-SPLIT-FIXES.md))
- [ ] Bump Postgres `max_connections` to 250 ([01](01-PRE-SPLIT-FIXES.md))
- [ ] Create Hetzner Object Storage bucket
- [ ] Configure `BACKUP_S3_*` env vars
- [ ] Verify `make backup-all` + S3 sync works
- [ ] Take final full backup

---

## Hetzner Setup

- [ ] Create Hetzner Cloud project `arm-production`
- [ ] Enable 2FA on account
- [ ] Create Cloud Network `arm-private` (10.0.1.0/24)
- [ ] Provision S1 (CX32, no public IP, 10.0.1.1)
- [ ] Provision S2 (CX22, public IP, 10.0.1.2)
- [ ] Provision S3 (CX22, optional public IP, 10.0.1.3)
- [ ] Provision S4 (CX11, no public IP, 10.0.1.4)
- [ ] Attach Hetzner Volume to S4, mount at `/mnt/backups`

---

## Server Hardening (All 4, in parallel)

For **each** server ([04](04-SERVER-HARDENING.md)):
- [ ] `apt update && apt upgrade`
- [ ] Install + enable `unattended-upgrades`
- [ ] Create `deploy` user, copy SSH keys
- [ ] SSH hardening (`PermitRootLogin no`, password auth off)
- [ ] Install UFW, apply per-server rules ([03](03-NETWORK-AND-FIREWALL.md))
- [ ] Kernel hardening (`/etc/sysctl.d/99-arm-hardening.conf`)
- [ ] 4 GB swap
- [ ] Fail2Ban
- [ ] Chrony time sync
- [ ] Remove telnet/rsh
- [ ] Install Docker + daemon hardening
- [ ] Verify: SSH root rejected, password rejected, UFW active, swap active

---

## S4 (Backup) — First

- [ ] Install `postgresql-client`, `mongosh`
- [ ] Copy latest backup from current server to `/mnt/backups`
- [ ] Verify restore drill works on S4

---

## S1 (Database) — Second

- [ ] Deploy `docker-compose.infra.multi.yml`
- [ ] Restore Postgres data from backup
- [ ] Restore MongoDB data from backup
- [ ] Create `arm_monitoring` users (Postgres + MongoDB)
- [ ] Start Alloy forwarder (logs → S3 Loki)
- [ ] Verify: `pg_isready`, `mongosh ping`, Kafka topics exist

---

## S3 (Observability) — Third

- [ ] Deploy `docker-compose.observe.multi.yml`
- [ ] Verify: postgres-exporter scrapes S1
- [ ] Verify: mongodb-exporter scrapes S1
- [ ] Verify: redis-exporter scrapes S2 (after S2 is up)
- [ ] Verify: Grafana loads, datasources connected

---

## S2 (Application) — Fourth

- [ ] Create `.env` with private IPs:
  - `DB_HOST=10.0.1.1`
  - `MONGODB_URI=mongodb://...@10.0.1.1:27017/...`
  - `KAFKA_BROKERS=10.0.1.1:9092`
  - `REDIS_HOST=arm-redis-prod` (local Docker DNS)
- [ ] Deploy `docker-compose.app.multi.yml`
- [ ] Verify: all 8 app containers healthy (`docker ps`)
- [ ] Verify: `psql -h 10.0.1.1` from S2 works
- [ ] Verify: `mongosh ...@10.0.1.1` from S2 works
- [ ] Verify: `redis-cli -h localhost ping` on S2
- [ ] Verify: Alloy shipping logs to S3 Loki
- [ ] Verify: OTLP traces arriving at S3 Tempo

---

## Cutover

- [ ] DNS A record: `SITE_DOMAIN` → S2 public IP
- [ ] DNS A record: `GRAFANA_DOMAIN` → S3 IP (or skip for SSH-only)
- [ ] Verify: `https://SITE_DOMAIN` loads with valid TLS cert
- [ ] Verify: `http://SITE_DOMAIN` redirects to HTTPS
- [ ] Verify: Login flow works end-to-end
- [ ] Verify: Calendar sync works
- [ ] Verify: Admin portal works
- [ ] Verify: Kong rate limiting (hit login > 20 times/min → 429)
- [ ] Verify: Grafana behind Authelia
- [ ] Verify: `.env` permissions are 600 on all servers

---

## Post-Cutover

- [ ] Set up backup cron on S4 (daily 02:00 from S1)
- [ ] Set up restore drill cron on S4 (weekly)
- [ ] Apply Hetzner Cloud Firewall rules (mirrors UFW)
- [ ] Update `deploy-production.yml` GitHub Secrets for multi-server
- [ ] Test CI/CD deploy pipeline end-to-end
- [ ] Verify: Prometheus alerts work (trigger test alert)
- [ ] Keep old server 1 week as fallback
- [ ] Decommission old server

---

## Post-Migration Improvements (Not Blocking)

- [ ] Enable Redis Sentinel (master on S2, replica on S1)
- [ ] Put Cloudflare in front of S2 (CDN + DDoS)
- [ ] Add TLS cert expiry Prometheus alert
- [ ] Run load test (baseline concurrent users)
- [ ] Write Ansible playbook for server provisioning
- [ ] Evaluate MongoDB replica set
- [ ] Evaluate PgBouncer on S1
