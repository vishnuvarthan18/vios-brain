# 10 — Maintenance Schedule

---

## Automated (No Human Intervention)

| Task | Frequency | How |
|---|---|---|
| OS security patches | Daily | `unattended-upgrades` |
| Database backup | Daily 02:00 | Cron on S4 |
| Backup prune (local) | Daily | 14-day retention |
| Backup prune (S3) | Automatic | S3 lifecycle rule, 30 days |
| TLS cert renewal | Auto | Caddy + Let's Encrypt (~60 days) |
| Log rotation | Automatic | Docker json-file driver, 50m x 3 |

## Monthly (~30 min)

| Task | What to do |
|---|---|
| Review Grafana dashboards | Disk trends, memory trends, error rates |
| Manual restore drill | `make restore-postgres-drill` + `make restore-mongodb-drill` |
| Review fail2ban | `fail2ban-client status sshd` |
| Docker base image updates | `docker pull` base images, rebuild, redeploy |
| Review UFW logs | `grep UFW /var/log/syslog` |

**Or run `arm-cli check` (Phase 6) to cover most of the above in one command.**

## Quarterly (~1 hour)

| Task | What to do |
|---|---|
| Rotate database passwords | New pw → `.env` + GitHub Secrets → restart |
| Rotate SSH keys | New key → `authorized_keys` all servers → GitHub Secret |
| Update Docker | `apt install docker-ce` (`live-restore` keeps containers up) |
| OS version check | Ubuntu 24.04 LTS → patches until 2029 |
| Review Hetzner firewall rules | No accidental additions? |
| Capacity review | Any server > 80% RAM/disk? |

## Annual

| Task | What to do |
|---|---|
| Full DR drill | Kill S1, restore from backup to new server, verify end-to-end |
| Dependency audit | CVE check on Node.js, npm, Docker images |
| Cost review | Right-size servers based on actual usage |

## Access Control

| Who/What | Can access | How |
|---|---|---|
| Admin | All 4 servers | SSH key, `deploy` user |
| GitHub Actions | S1, S2, S3 | `PROD_SSH_PRIVATE_KEY` |
| S2 (App) | S1 databases | Private network + credentials |
| S3 (Observe) | S1 DB metrics, S2 app metrics | Private network + monitoring creds |
| S4 (Backup) | S1 databases (dump only) | Private network + credentials |
| Browser users | S2 :443 only | Caddy TLS → Kong → apps |
| Grafana viewers | S3 :443 (if public) | Authelia + Grafana login |
