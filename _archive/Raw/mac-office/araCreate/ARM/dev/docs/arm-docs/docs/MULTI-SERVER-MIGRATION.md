# ARM Platform — Multi-Server Architecture Migration Plan

## Context

ARM currently runs ~25 containers on a single Hetzner server (~13.3 GB memory). This is a single point of failure in every dimension — disk, compute, network. The goal is to split into 4 servers on a Hetzner private network, with a 5th component being Hetzner Object Storage for offsite backups:

- **S1 — Database:** PostgreSQL, MongoDB, Kafka
- **S2 — Application + Redis:** Caddy, Kong, Authelia, 8 app services, Redis (master)
- **S3 — Observability:** Prometheus, Grafana, Loki, Tempo, Alloy, exporters
- **S4 — Backup:** Receives dumps from S1, runs restore drills, syncs to S3-compatible object storage

Redis lives on S2 (not S1) because the session service hits Redis on every request for session lookup — keeping it on the same host as the session service avoids the ~0.2-0.5ms private network latency penalty on the hottest path. Calendar-be's BullMQ usage is background processing that tolerates network latency.

---

## What Changes

### Phase 0: Pre-split fixes on the current single server

These are risks that exist today and should be fixed before the migration, not during it.

#### 0.1 — Fix Redis eviction policy

**File:** `deploy/arm-deploy-make/docker-compose.infra.prod.yml:141`

Change the default from `noeviction` to `volatile-lru`:
```
--maxmemory-policy ${TUNE_REDIS_MAXMEMORY_POLICY:-noeviction}
```
becomes:
```
--maxmemory-policy ${TUNE_REDIS_MAXMEMORY_POLICY:-volatile-lru}
```

Also fix in `docker-compose.infra.redis-sentinel.yml` (lines 19, 51, 83) — all three instances hardcode `noeviction`:
```
--maxmemory-policy noeviction
```
becomes:
```
--maxmemory-policy volatile-lru
```

#### 0.2 — Bump PostgreSQL max_connections

**File:** `deploy/arm-deploy-make/docker-compose.infra.prod.yml:55`

Change default from 200 to 250:
```
- max_connections=${TUNE_PG_MAX_CONNECTIONS:-200}
```
becomes:
```
- max_connections=${TUNE_PG_MAX_CONNECTIONS:-250}
```

#### 0.3 — Configure offsite backups

**File:** `deploy/arm-deploy-make/.env` (not committed — manual step)

Set `(secret removed)`, `(secret removed)`, `(secret removed)`, `(secret removed)`, `(secret removed)` to point at Hetzner Object Storage. The GitHub Actions workflow already has these as vars/secrets (`(secret removed)`).

Run `make backup-all` to verify end-to-end (local dump + S3 sync + prune).

---

### Phase 1: New compose files for multi-server topology

The current single-server uses Docker bridge networks for inter-container communication. In the multi-server setup, containers on different hosts communicate over the Hetzner private network (10.x.x.x) instead. This means:

- Docker DNS names (`arm-postgres-prod`, `arm-redis-prod`, etc.) are replaced by private IPs or hostnames for cross-host connections
- Containers on the same host still use Docker DNS within their local compose stack
- Each server runs its own independent `docker compose` stack

#### 1.1 — Create `docker-compose.infra.multi.yml` (S1 — Database server)

**New file:** `deploy/arm-deploy-make/docker-compose.infra.multi.yml`

Based on `docker-compose.infra.prod.yml` with these changes:
- Postgres, MongoDB, Kafka stay as-is (they run locally on S1)
- **Remove** Redis from this file (Redis moves to S2)
- **Add** port exposure on `0.0.0.0` for Postgres (5432), MongoDB (27017), Kafka (9092) — these are only reachable on the private network since S1 has no public IP
- Keep existing health checks, resource limits, volumes, log rotation
- **Add** PgBouncer container (optional, evaluate during Phase 1):
  - Image: `edoburu/pgbouncer` or `bitnami/pgbouncer`
  - Listens on 6432, proxies to localhost:5432
  - Pool mode: `transaction` (best for TypeORM)
  - Max client connections: 400, default pool size: 50
  - Expose 6432 instead of 5432 to app servers

#### 1.2 — Create `docker-compose.app.multi.yml` (S2 — Application server)

**New file:** `deploy/arm-deploy-make/docker-compose.app.multi.yml`

Based on `docker-compose.yml` with these changes:
- **Keep:** Caddy, Kong, Authelia, all 8 generated app service includes
- **Add:** Redis container (moved from infra to here) — same config as `docker-compose.infra.prod.yml` lines 129-165
- **Remove:** All observability containers (prometheus, node-exporter, cadvisor, exporters, loki, alloy, tempo, grafana)
- **Update generated compose env vars** to point at S1's private IP:
  - `DB_HOST` default: `arm-postgres-prod` → `${DB_HOST}` (set to S1 private IP in .env)
  - `MONGO_HOST` default: similarly
  - `KAFKA_BROKERS` default: similarly
  - `REDIS_HOST` stays as Docker DNS name since Redis is co-located on S2
- **Keep:** `arm-prod-network` (created here, local to S2)
- **Remove:** `arm-infra-network: external: true` (no longer needed — infra is on a different host)
- Caddy keeps ports 80, 443 exposed publicly

#### 1.3 — Create `docker-compose.observe.multi.yml` (S3 — Observability server)

**New file:** `deploy/arm-deploy-make/docker-compose.observe.multi.yml`

Based on the observability section of `docker-compose.yml` with these changes:
- **Move here:** Prometheus, Grafana, Loki, Tempo, Alloy, node-exporter, cadvisor
- **Move here:** postgres-exporter, mongodb-exporter, redis-exporter
- **Add:** Caddy instance (or nginx) for TLS + Authelia if Grafana is publicly accessible. Alternatively, access via SSH tunnel only.
- **Update exporter connection strings** to point at private IPs:
  - postgres-exporter `DATA_SOURCE_NAME`: `arm-postgres-prod` → S1 private IP
  - mongodb-exporter `MONGODB_URI`: `arm-mongodb-prod` → S1 private IP
  - redis-exporter `REDIS_ADDR`: `arm-redis-prod` → S2 private IP (Redis is on S2)
- **Update Prometheus config** — new file `observability/prometheus.multi.yml`:
  - App scrape targets (`core-be`, `session`, etc.) → S2 private IP + container port
  - DB exporter targets → localhost (co-located on S3)
  - Kong target → S2 private IP
  - node-exporter, cadvisor → localhost (S3's own host metrics)
  - **Add** additional node-exporter scrape jobs for S1, S2, S4 if running node-exporter there
- **Update Alloy config** — new file `observability/alloy/config.multi.alloy`:
  - Docker log collection only works for local containers (requires docker.sock mount)
  - S3's Alloy collects logs from S3 containers only
  - For S1 and S2 logs: either run a lightweight Alloy instance on each, or use Docker's `loki` logging driver to push directly to Loki on S3
- **Update Grafana datasources** — new file `observability/grafana/datasources-multi/datasources.yml`:
  - Prometheus, Loki, Tempo URLs stay as localhost (all co-located on S3)
- **Update Alloy trace forwarding:**
  - App services on S2 send OTLP traces to S3's Alloy/Tempo endpoint over private network
  - Update `OTEL_EXPORTER_OTLP_ENDPOINT` in app services to point at S3

#### 1.4 — Create backup automation for S4

**New file:** `deploy/arm-deploy-make/scripts/backup-remote.sh`

- Runs on S4, connects to S1 over private network
- `pg_dump -h <S1-private-ip>` instead of `docker exec arm-postgres-prod pg_dump`
- `mongodump --host=<S1-private-ip>` instead of `docker exec arm-mongodb-prod mongodump`
- Requires PostgreSQL and MongoDB client tools installed on S4 (not Docker)
- Stores dumps locally on S4, syncs to Hetzner Object Storage
- Schedule: daily cron at 02:00 (same as current `backup-cron-install`)

**New file:** `deploy/arm-deploy-make/scripts/restore-drill-remote.sh`

- Runs weekly on S4
- Pulls latest backup, restores into throwaway database on S4's own local PostgreSQL/MongoDB instances (lightweight, for verification only)
- Verifies row/collection counts, drops drill databases
- Emails result (success/failure)

---

### Phase 2: Environment variable changes

#### 2.1 — New env vars for multi-server mode

Add to `.env.example`:

```bash
# ── Multi-server topology ──────────────────────────────────
# Leave empty/unset for single-server mode (defaults to Docker DNS names).
# Set to private IPs when running on separate servers.
MULTI_SERVER_MODE=false

# S1 — Database server private IP
DB_SERVER_IP=
# S2 — App server private IP (also where Redis runs)
APP_SERVER_IP=
# S3 — Observability server private IP
OBSERVE_SERVER_IP=
# S4 — Backup server private IP
BACKUP_SERVER_IP=
```

#### 2.2 — Update existing env var defaults

When `MULTI_SERVER_MODE=true`, these defaults change:

| Env var | Single-server default | Multi-server value |
|---|---|---|
| `DB_HOST` | `arm-postgres-prod` | `${DB_SERVER_IP}` |
| `DB_PORT` | `5432` | `6432` (if PgBouncer) or `5432` |
| `MONGO_HOST` | `arm-mongodb-prod` | `${DB_SERVER_IP}` |
| `MONGODB_URI` | `mongodb://...@arm-mongodb-prod:27017/...` | `mongodb://...@${DB_SERVER_IP}:27017/...` |
| `KAFKA_BROKERS` | `arm-kafka1-prod:9092,...` | `${DB_SERVER_IP}:9092` |
| `REDIS_HOST` | `arm-redis-prod` | `arm-redis-prod` (unchanged — Redis is local to S2) |

These are set in the `.env` file on each server, not changed in code.

#### 2.3 — Update generated compose defaults

**Files:** `orchestrator/templates/prod/backend-core.yml.tmpl`, `orchestrator/templates/prod/backend-app.yml.tmpl`

The `${DB_HOST:-arm-postgres-prod}` defaults in generated compose files need to work in both modes:
- Single-server: default `arm-postgres-prod` (Docker DNS) works
- Multi-server: `.env` overrides with private IP

No template changes needed — the existing `${VAR:-default}` pattern already supports this. The `.env` on S2 just sets `DB_HOST=10.0.1.1` etc.

---

### Phase 3: Prometheus and monitoring config changes

#### 3.1 — Create `prometheus.multi.yml`

**New file:** `deploy/arm-deploy-make/observability/prometheus.multi.yml`

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # App services — on S2, reached over private network
  - job_name: core-be
    metrics_path: /v1/metrics
    static_configs:
      - targets: ["${APP_SERVER_IP}:3000"]  # core-be container port

  - job_name: session
    metrics_path: /metrics
    static_configs:
      - targets: ["${APP_SERVER_IP}:5000"]

  # ... (all 5 app backends + Kong at APP_SERVER_IP)

  # Exporters — co-located on S3
  - job_name: postgres
    static_configs:
      - targets: ["arm-postgres-exporter:9187"]

  # ... (mongodb-exporter, redis-exporter similarly)

  # Host metrics — local to S3
  - job_name: node
    static_configs:
      - targets: ["arm-node-exporter:9100"]
```

**Problem:** Prometheus can't reach app services on S2 via Docker container ports — those ports aren't published to the host. Two options:

**Option A (recommended):** Publish metrics ports on S2's private IP only:
- Add `ports: ["${APP_SERVER_IP}:3000:3000"]` to each app service in the multi-server compose
- Prometheus on S3 scrapes `${APP_SERVER_IP}:3000`
- Ports are only reachable on the private network

**Option B:** Run a lightweight Prometheus on S2 that scrapes local containers via Docker DNS, and use Prometheus federation on S3 to aggregate.

#### 3.2 — Alloy log collection strategy

**Current:** Single Alloy instance mounts `docker.sock`, tails all container logs, pushes to co-located Loki.

**Multi-server:** Docker socket is host-local. Options:

**Option A (recommended):** Run a lightweight Alloy on S1 and S2, each pushing to Loki on S3:
- S1 Alloy: tails Postgres, MongoDB, Kafka logs → pushes to `http://${OBSERVE_SERVER_IP}:3100/loki/api/v1/push`
- S2 Alloy: tails all app container logs → pushes to same Loki endpoint
- S3 Alloy: tails observability container logs locally, handles OTLP trace ingestion from S2

**Option B:** Use Docker's `loki` logging driver on S1 and S2 — each container ships logs directly to Loki without Alloy intermediary.

#### 3.3 — OTLP trace routing

App services on S2 currently send OTLP traces to `arm-alloy:4317` (Docker DNS). In multi-server mode, they need to send to S3:

- Set `OTEL_EXPORTER_OTLP_ENDPOINT=http://${OBSERVE_SERVER_IP}:4317` in app service env vars
- Alloy on S3 receives traces and forwards to co-located Tempo (unchanged)

---

### Phase 4: Network and firewall

#### 4.1 — Hetzner Cloud Network setup

- Create a Hetzner Cloud Network: `10.0.0.0/16`
- Subnet: `10.0.1.0/24`
- Assign private IPs:
  - S1 (DB): `10.0.1.1`
  - S2 (App): `10.0.1.2`
  - S3 (Observe): `10.0.1.3`
  - S4 (Backup): `10.0.1.4`

#### 4.2 — Firewall rules (Hetzner Cloud Firewall or iptables)

**S1 (Database) — no public IP:**

| Allow from | Port | Purpose |
|---|---|---|
| 10.0.1.2 (S2) | 5432 (or 6432) | App → PostgreSQL (or PgBouncer) |
| 10.0.1.2 (S2) | 27017 | App → MongoDB |
| 10.0.1.2 (S2) | 9092 | App → Kafka |
| 10.0.1.3 (S3) | 9187, 9216 | Exporters → DB metrics |
| 10.0.1.4 (S4) | 5432, 27017 | Backup dumps |
| 10.0.1.0/24 | 22 | SSH from any server in the network |

**S2 (App) — public IP for Caddy:**

| Allow from | Port | Purpose |
|---|---|---|
| 0.0.0.0/0 | 80, 443 | Browser traffic (Caddy TLS) |
| 10.0.1.3 (S3) | app metrics ports | Prometheus scraping |
| 10.0.1.0/24 | 22 | SSH |

**S3 (Observe) — optional public IP for Grafana:**

| Allow from | Port | Purpose |
|---|---|---|
| 10.0.1.2 (S2) | 3100 | Alloy on S2 → Loki |
| 10.0.1.1 (S1) | 3100 | Alloy on S1 → Loki |
| 10.0.1.2 (S2) | 4317, 4318 | OTLP traces → Alloy/Tempo |
| 0.0.0.0/0 | 443 | Grafana (if publicly exposed, behind Authelia) |
| 10.0.1.0/24 | 22 | SSH |

**S4 (Backup) — no public IP:**

| Allow from | Port | Purpose |
|---|---|---|
| 10.0.1.0/24 | 22 | SSH |

---

### Phase 5: CI/CD workflow changes

#### 5.1 — Update `deploy-production.yml`

The current workflow SSHes into one server and runs everything there. In multi-server mode, it needs to deploy to multiple servers.

**Key changes:**
- Add secrets: `PROD_DB_SERVER_HOST`, `PROD_OBSERVE_SERVER_HOST`, `PROD_BACKUP_SERVER_HOST` (S2 keeps the existing `PROD_SERVER_HOST`)
- **Prepare job:** rsync deploy files to S1, S2, S3 (each gets what it needs)
- **Deploy job sequence:**
  1. SSH to S1: `make infra-up-multi` (Postgres, MongoDB, Kafka)
  2. SSH to S1: `make infra-wait-multi` (wait for DBs)
  3. SSH to S2: `make app-deploy-multi IMAGE_TAG=$TAG` (Redis + all app services + edge)
  4. SSH to S3: `make observe-deploy-multi` (observability stack)
- **Verify job:** check containers on all 3 servers
- **Rollback job:** only needs to rollback S2 (app images) — databases on S1 are not touched

#### 5.2 — New Makefile targets

**File:** `deploy/arm-deploy-make/make/multi-server.mk`

```makefile
# New targets for multi-server deployment
infra-up-multi:        # S1: Postgres + MongoDB + Kafka (no Redis)
app-deploy-multi:      # S2: Redis + 8 app services + Caddy + Kong + Authelia
observe-deploy-multi:  # S3: Full observability stack
backup-setup-multi:    # S4: Install backup cron, client tools
```

---

### Phase 6: Migration execution order

This is the actual cutover sequence. Plan for ~2 hours of downtime.

1. **Announce maintenance window**
2. **Take final backup** on current server: `make backup-all`
3. **Provision S1, S2, S3, S4** on Hetzner Cloud Network
4. **Set up S4 first** — copy latest backup from current server, verify restore
5. **Set up S1** — install Docker, start database containers, restore data from backup
6. **Verify S1** — run health checks, verify data integrity
7. **Set up S3** — install Docker, start observability stack, point exporters at S1
8. **Set up S2** — install Docker, copy app images/configs, update .env with private IPs, start app stack
9. **Verify end-to-end** — S2 app services connect to S1 databases, S3 scrapes metrics from both
10. **DNS cutover** — point domain A record to S2's public IP
11. **Verify public access** — TLS cert, login flow, calendar sync
12. **Set up S4 automated backups** — cron job pulling from S1 over private network
13. **Decommission old server** (keep it around for 1 week as fallback)

---

### Phase 7: Post-migration improvements (not blocking)

- Enable Redis Sentinel: master on S2, replica on S1, 3 sentinels across S2+S3
- Add PgBouncer on S1 if connection count becomes an issue
- Put Cloudflare in front of S2 for CDN + DDoS protection
- Add TLS cert expiry alert in Prometheus
- Run load test to establish baseline concurrent user capacity
- Write Ansible playbook for server provisioning (disaster recovery runbook)
- Evaluate MongoDB replica set (needs 3 members minimum — could use S1 primary, S4 secondary, S3 arbiter)

---

---

# Part 2 — Production Hardening & Secure Hosting

Everything below covers how to provision, harden, secure, and maintain the 4-server setup from bare metal to running platform.

---

## 8. Hetzner Account & Project Setup

### 8.1 — Hetzner Cloud Project

- Create project: `arm-production`
- Enable **2FA** on the Hetzner account; add a second admin user as backup
- Create an API token (read+write) for automation — store in password manager, never in git

### 8.2 — Cloud Network

- Name: `arm-private`
- IP range: `10.0.0.0/16`
- Subnet: `10.0.1.0/24`, zone: `eu-central` (Falkenstein or Nuremberg)
- All 4 servers attach to this network

### 8.3 — Object Storage

- Bucket: `arm-backups`
- Region: same DC as servers
- Generate S3 access key + secret for `(secret removed)` env vars

---

## 9. Server Provisioning

### 9.1 — Specs

| Server | Hetzner Type | RAM | vCPU | Disk | Public IP | Purpose |
|---|---|---|---|---|---|---|
| S1 — DB | CX32 | 16 GB | 4 | 80 GB NVMe | No | Postgres, MongoDB, Kafka |
| S2 — App | CX22 | 8 GB | 4 | 80 GB NVMe | Yes (IPv4) | App services, Redis, Caddy, Kong |
| S3 — Observe | CX22 | 8 GB | 4 | 80 GB NVMe | Optional | Prometheus, Grafana, Loki, Tempo |
| S4 — Backup | CX11 | 4 GB | 2 | 40 GB + Volume | No | Backup dumps, restore drills |

- S4 gets a **Hetzner Volume** (100-200 GB, expandable) mounted at `/mnt/backups`
- OS: **Ubuntu 24.04 LTS** on all 4 servers
- SSH key only at creation — never enable password auth

---

## 10. OS Hardening (All 4 Servers)

Run every step below on each server after first boot.

### 10.1 — System Updates + Auto-Patches

```bash
apt update && apt upgrade -y
apt install -y unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades
```

`unattended-upgrades` applies CVE patches automatically. Critical for a small team.

### 10.2 — Create Deploy User

```bash
useradd -m -s /bin/bash deploy
mkdir -p /home/deploy/.ssh
cp /root/.ssh/authorized_keys /home/deploy/.ssh/
chown -R deploy:deploy /home/deploy/.ssh
chmod 700 /home/deploy/.ssh
chmod 600 /home/deploy/.ssh/authorized_keys
usermod -aG docker deploy    # after Docker install
```

All operations run as `deploy`. Root login disabled next.

### 10.3 — SSH Hardening

`/etc/ssh/sshd_config`:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthenticationMethods publickey
MaxAuthTries 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers deploy
X11Forwarding no
AllowTcpForwarding no
```

`systemctl restart sshd` — after this only `deploy` can SSH in, only with a key, idle sessions killed after 10 min.

### 10.4 — Host Firewall (UFW)

Defense-in-depth on the host, even with Hetzner Cloud Firewall at the network edge.

**S1 (Database):**
```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow from 10.0.1.0/24 to any port 22         # SSH
ufw allow from 10.0.1.2 to any port 5432          # S2 → Postgres
ufw allow from 10.0.1.2 to any port 27017         # S2 → MongoDB
ufw allow from 10.0.1.2 to any port 9092          # S2 → Kafka
ufw allow from 10.0.1.3 to any port 5432          # S3 exporter → Postgres
ufw allow from 10.0.1.3 to any port 27017         # S3 exporter → MongoDB
ufw allow from 10.0.1.4 to any port 5432          # S4 → Postgres backup
ufw allow from 10.0.1.4 to any port 27017         # S4 → MongoDB backup
ufw allow from 10.0.1.3 to any port 3100          # Alloy on S1 → Loki on S3
ufw enable
```

**S2 (App):**
```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow from 10.0.1.0/24 to any port 22
ufw allow 80/tcp                                    # Caddy ACME
ufw allow 443/tcp                                   # Caddy HTTPS
ufw allow from 10.0.1.3 to any port 3000           # Prometheus → core-be
ufw allow from 10.0.1.3 to any port 4001           # Prometheus → calendar-be
ufw allow from 10.0.1.3 to any port 5000           # Prometheus → session service
ufw allow from 10.0.1.3 to any port 8100           # Prometheus → Kong
ufw allow from 10.0.1.3 to any port 10001          # Prometheus → admin-be
ufw allow from 10.0.1.3 to any port 6379           # S3 redis-exporter → Redis
ufw enable
```

**S3 (Observe):**
```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow from 10.0.1.0/24 to any port 22
ufw allow from 10.0.1.1 to any port 3100           # S1 Alloy → Loki
ufw allow from 10.0.1.2 to any port 3100           # S2 Alloy → Loki
ufw allow from 10.0.1.2 to any port 4317           # S2 → Tempo OTLP gRPC
ufw allow from 10.0.1.2 to any port 4318           # S2 → Tempo OTLP HTTP
ufw allow 443/tcp                                    # Grafana (if public, behind Authelia)
ufw enable
```

**S4 (Backup):**
```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow from 10.0.1.0/24 to any port 22
ufw enable
```

### 10.5 — Hetzner Cloud Firewall (Second Layer)

Mirror the UFW rules above in Hetzner Cloud Firewall. This blocks at the hypervisor level before traffic reaches the host. Two layers: Hetzner blocks at the edge, UFW blocks at the OS.

### 10.6 — Kernel Hardening

`/etc/sysctl.d/99-arm-hardening.conf`:

```
# Prevent IP spoofing
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Disable source routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0

# Disable ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0

# SYN flood protection
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048

# Disable IPv6 if not needed
net.ipv6.conf.all.disable_ipv6 = 1

# Shared memory (for Postgres)
kernel.shmmax = (removed)

# Prefer RAM over swap
vm.swappiness = 10
```

Apply: `sysctl --system`

### 10.7 — Swap (All Servers)

```bash
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

4 GB per server. Not a crutch — a buffer before the OOM killer.

### 10.8 — Fail2Ban

```bash
apt install -y fail2ban
```

`/etc/fail2ban/jail.local`:
```
[sshd]
enabled = true
port = 22
maxretry = 3
bantime = 3600
findtime = 600
```

Bans an IP for 1 hour after 3 failed SSH attempts in 10 minutes.

### 10.9 — Remove Unnecessary Packages

```bash
apt purge -y telnet rsh-client rsh-server
apt autoremove -y
```

### 10.10 — Time Sync

```bash
apt install -y chrony
systemctl enable chrony
```

Critical — Kafka, JWT expiry, TLS certs, and log timestamps all depend on synchronized clocks.

---

## 11. Docker Hardening (All Servers)

### 11.1 — Install

```bash
curl -fsSL https://get.docker.com | sh
usermod -aG docker deploy
apt install -y docker-compose-plugin
```

### 11.2 — Daemon Configuration

`/etc/docker/daemon.json`:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "live-restore": true,
  "no-new-privileges": true,
  "icc": false,
  "default-ulimits": {
    "nofile": { "Name": "nofile", "Soft": 65536, "Hard": 65536 }
  }
}
```

Key settings:
- **`live-restore: true`** — containers survive Docker daemon restarts (critical for upgrades)
- **`no-new-privileges: true`** — prevents privilege escalation inside containers
- **`icc: false`** — containers can't talk unless explicitly linked by Docker networks

### 11.3 — Docker Socket Access

`/var/run/docker.sock` gives full root access to the host. Only mount where absolutely necessary:

| Server | Docker socket needed? | Why |
|---|---|---|
| S1 (DB) | No | Databases don't need it |
| S2 (App) | Only Alloy | Log collection; consider `loki` logging driver instead |
| S3 (Observe) | Alloy + cAdvisor | Accept the risk — S3 has no public-facing services |
| S4 (Backup) | No | No containers |

---

## 12. Database Security (S1)

### 12.1 — PostgreSQL

- `POSTGRES_PASSWORD`: 32+ character random string (generated by `make env-init`)
- `arm_monitoring`: separate read-only user with `pg_monitor` role
- All connections require password auth (no `trust` entries in `pg_hba.conf`)
- Listens on `0.0.0.0:5432` in the container, exposed to host — but S1 has no public IP
- UFW + Hetzner firewall restrict to S2, S3, S4 private IPs only
- Hetzner NVMe drives are encrypted at hardware level
- Application-level encryption already in place: `GOOGLE_TOKEN_ENCRYPTION_KEY` for OAuth tokens

### 12.2 — MongoDB

- Root credentials: `MONGO_ROOT_USER` / `MONGO_ROOT_PASSWORD`
- `--auth` flag enforced in start command
- `arm_monitoring` user: `clusterMonitor` role only (read-only metrics)
- Same network lockdown as Postgres

### 12.3 — Redis (on S2)

- `--requirepass` with strong password
- **Not exposed to host network** — no `ports:` mapping since all consumers (session service, calendar-be) are co-located on S2
- Only exception: S3's redis-exporter needs access — expose Redis port to S3's IP only via UFW

### 12.4 — Credential Rotation (Quarterly, Manual)

1. Generate new password
2. Update `.env` on S2 and GitHub Secrets
3. `ALTER USER` / MongoDB `updateUser` on S1
4. Restart app services on S2 (re-read env vars on boot)
5. Verify health checks pass

---

## 13. Secrets Management

### 13.1 — At Rest

| Location | How secured |
|---|---|
| On servers | `~/arm-deploy/.env` with `chmod 600` (only `deploy` user can read) |
| In CI | GitHub Secrets (encrypted, masked in logs) |
| Source of truth | Password manager (1Password/Bitwarden team vault) |

### 13.2 — In Transit

- `.env` shipped via `scp` (encrypted SSH channel), never as inline shell strings
- Deploy workflow already uses `env:` binding (not `${{ }}` interpolation in script body) — prevents shell injection from secrets with special characters

### 13.3 — Never Do

- Commit `.env` to git (already gitignored)
- Log env vars in application code (`pino` `redact` gap still open — see MONITORING.md)
- Pass secrets as Docker build args (persists in image layers)
- Use `docker inspect` to debug in production (shows all env vars in cleartext)

---

## 14. TLS / HTTPS

### 14.1 — Caddy Auto-TLS (S2)

Caddy auto-provisions Let's Encrypt certs via ACME HTTP-01. Already works. Verify:

- Port 80 open to internet on S2 (Caddy needs it for ACME challenge)
- DNS A record for `SITE_DOMAIN` and `GRAFANA_DOMAIN` → S2's public IP
- Cert storage: `arm_caddy_prod_data` volume — must persist across deploys
- Add Prometheus alert for cert expiry < 14 days as safety net

### 14.2 — Internal Traffic

S1↔S2↔S3↔S4 traffic is **unencrypted** but on Hetzner's VLAN-isolated private network. Acceptable for current threat model. If compliance requires internal encryption later: WireGuard mesh between servers is the simplest option.

### 14.3 — Grafana Access (S3)

**Option A — SSH tunnel (more secure, recommended):**
- S3 has no public IP
- Access: `ssh -L 3000:localhost:3000 deploy@10.0.1.3` via S2 as jump host
- Zero attack surface

**Option B — Public with Authelia (current pattern):**
- S3 gets a public IP, Caddy terminates TLS, Authelia gates access
- More convenient, more attack surface

---

## 15. Security-Focused Monitoring Alerts

### 15.1 — Existing (Already Configured in Grafana)

8 alert rules: service down, DB down, 5xx spike, disk space, Redis memory (warning + critical).

### 15.2 — Add These

| Alert | Condition | Why |
|---|---|---|
| SSH brute force | fail2ban ban count > 5/hour | Someone is probing |
| TLS cert expiry | < 14 days remaining | Prevent accidental outage |
| Disk usage on S4 | > 80% | Backup volume filling |
| Container restart loop | > 3 restarts in 5 min | Crash or resource exhaustion |
| Kong 401/403 spike | > 20% auth failures in 5 min | Credential stuffing or broken auth |

### 15.3 — Log Security

All container logs flow to Loki on S3. This means S3 holds potentially sensitive data.

- Loki retention: 30 days (auto-deleted after)
- Grafana/Loki gated by Authelia
- **Open gap:** `pino` `redact` config not deployed — tokens in error logs go to Loki in cleartext. Most urgent security follow-up.

---

## 16. Backup & Disaster Recovery

### 16.1 — Backup Chain

```
S1 (Database)
  │
  ├── daily 02:00 ──→ S4 (Backup Server)
  │                     │
  │                     ├── local disk (/mnt/backups)
  │                     │   retention: 14 days
  │                     │
  │                     └── Hetzner Object Storage (arm-backups bucket)
  │                         retention: 30 days (S3 lifecycle rule)
  │
  └── weekly ──→ S4 restore drill (automated verification)
```

### 16.2 — What Gets Backed Up

| Data | Method | Frequency | Stored on |
|---|---|---|---|
| PostgreSQL (`arm_core`, `arm_admin`) | `pg_dump` over network | Daily | S4 + Object Storage |
| MongoDB (`arm-calendar`) | `mongodump` over network | Daily | S4 + Object Storage |
| Redis | Not backed up (sessions are ephemeral; BullMQ jobs are replayable) | — | — |
| Kafka | Not backed up (fire-and-forget notifications) | — | — |
| Caddy TLS certs | Docker volume on S2 (Caddy re-provisions if lost) | — | — |
| `.env` files | Password manager + GitHub Secrets | Manual | — |
| Grafana dashboards | JSON in git (`observability/grafana/dashboards/`) | Every deploy | Git |

### 16.3 — Automated Restore Drill (S4, Weekly)

1. Pull latest `arm_core` dump from `/mnt/backups`
2. Restore into throwaway database on S4's local PostgreSQL
3. Verify table count, row count in critical tables
4. Drop throwaway database
5. Repeat for MongoDB
6. Email result to `GRAFANA_ALERT_EMAIL`

### 16.4 — Disaster Recovery Runbook

**If S1 (Database) dies:**

| Step | Action |
|---|---|
| 1 | Provision new CX32, attach to `arm-private` at `10.0.1.1` |
| 2 | Run OS hardening (Section 10) |
| 3 | Install Docker (Section 11) |
| 4 | Pull latest backup from S4 or Object Storage |
| 5 | Start DB containers, restore data |
| 6 | Verify from S2: health checks pass |
| **RTO** | ~1 hour |
| **RPO** | Up to 24 hours (last backup) |

**If S2 (App) dies:**

| Step | Action |
|---|---|
| 1 | Provision new CX22 with public IP |
| 2 | OS hardening + Docker |
| 3 | `make app-deploy-multi` |
| 4 | Update DNS A record to new public IP |
| 5 | Caddy auto-provisions new TLS cert |
| **RTO** | ~30 min |
| **RPO** | Zero (stateless — all state is on S1) |

**If S3 (Observe) dies:**

| Step | Action |
|---|---|
| 1 | Provision new CX22, deploy observability stack |
| 2 | Historical metrics/logs/traces are lost (acceptable — not backed up) |
| **RTO** | ~20 min |

**If S4 (Backup) dies:**

| Step | Action |
|---|---|
| 1 | Provision new CX11 + volume |
| 2 | Object Storage still has all backups |
| **RTO** | ~15 min, no data loss |

---

## 17. CI/CD Security

### 17.1 — Already Good

- Secrets injected via `env:` binding, not `${{ }}` interpolation (prevents shell injection)
- `.env` shipped via `scp`, never constructed in remote shell
- Concurrency control prevents deploy races
- Automatic rollback on verification failure
- `IMAGE_TAG` is commit SHA, not `latest`

### 17.2 — Harden Further

| Action | Why |
|---|---|
| Pin Actions to SHA (`actions/checkout@<sha>`) | Tag (`@v5`) can be repointed; SHA is immutable |
| Add `permissions:` block to workflow | Least-privilege per job |
| Rotate `PROD_SSH_PRIVATE_KEY` annually | Limit blast radius if key leaks |
| IP allowlist on SSH (S2) | Only GitHub Actions runners + team IPs |

---

## 18. Maintenance Procedures

### 18.1 — Automated (No Human Intervention)

| Task | Frequency | How |
|---|---|---|
| OS security patches | Daily | `unattended-upgrades` |
| Database backup | Daily 02:00 | Cron on S4 |
| Backup prune (local) | Daily (part of backup-all) | 14-day retention |
| Backup prune (Object Storage) | Automatic | S3 lifecycle rule, 30 days |
| TLS cert renewal | Auto | Caddy + Let's Encrypt (~60 days) |
| Log rotation | Automatic | Docker `json-file` driver, 50m x 3 |

### 18.2 — Monthly (~30 min)

| Task | What to do |
|---|---|
| Review Grafana dashboards | Disk trends, memory trends, error rates |
| Manual restore drill | `make restore-postgres-drill` + `make restore-mongodb-drill` |
| Review fail2ban | `fail2ban-client status sshd` — persistent attackers? |
| Docker base image updates | `docker pull` base images, rebuild, redeploy |
| Review UFW logs | `grep UFW /var/log/syslog` — unexpected blocks? |

### 18.3 — Quarterly (~1 hour)

| Task | What to do |
|---|---|
| Rotate database passwords | New passwords → `.env` + GitHub Secrets → restart services |
| Rotate SSH keys | New key pair → `authorized_keys` on all servers → GitHub Secret |
| Update Docker | `apt update && apt install docker-ce` (`live-restore` keeps containers running) |
| OS version check | Verify Ubuntu 24.04 still gets security patches (LTS until 2029) |
| Review Hetzner firewall rules | No accidental additions? |
| Capacity review | Any server consistently above 80% RAM/disk? Time to upsize? |

### 18.4 — Annual

| Task | What to do |
|---|---|
| Full DR drill | Kill S1, restore from backup onto new server, verify end-to-end |
| Dependency audit | CVE check on Node.js, npm packages, Docker images |
| Cost review | Right-size servers based on actual usage |

---

## 19. Access Control Summary

| Who/What | Can access | How |
|---|---|---|
| Admin (you) | All 4 servers | SSH key, `deploy` user |
| GitHub Actions | S1, S2, S3 | `PROD_SSH_PRIVATE_KEY` |
| S2 (App) | S1 databases | Private network + DB credentials in `.env` |
| S3 (Observe) | S1 DB metrics, S2 app metrics | Private network + monitoring credentials |
| S4 (Backup) | S1 databases (dump only) | Private network + DB credentials |
| Browser users | S2 port 443 only | Caddy TLS → Kong → app services |
| Grafana viewers | S3 port 443 (if public) | Authelia + Grafana login |

**No server trusts another server's Docker network.** Cross-server communication is TCP over private IPs with firewall rules. A compromised S3 cannot reach S2's Redis — UFW on S2 doesn't allow it.

---

## 20. Execution Order (Updated with Hardening)

Full cutover sequence incorporating hardening at every step.

### Pre-Migration (on current single server)

- [ ] Fix Redis eviction policy (`noeviction` → `volatile-lru`)
- [ ] Bump Postgres `max_connections` to 250
- [ ] Create Hetzner Object Storage bucket
- [ ] Configure `BACKUP_S3_*` env vars, verify `make backup-all` + S3 sync
- [ ] Take a final full backup

### Server Provisioning (all 4 in parallel)

For **each** server (S1, S2, S3, S4):
- [ ] Create Hetzner Cloud server, attach to `arm-private` network
- [ ] OS hardening: updates, deploy user, SSH lockdown (Section 10.1–10.3)
- [ ] Install UFW, apply per-server rules (Section 10.4)
- [ ] Kernel hardening (Section 10.6)
- [ ] Swap setup (Section 10.7)
- [ ] Fail2Ban (Section 10.8)
- [ ] Chrony time sync (Section 10.10)
- [ ] Install Docker + daemon hardening (Section 11)

### S4 (Backup) — First

- [ ] Mount Hetzner Volume at `/mnt/backups`
- [ ] Install `postgresql-client`, `mongosh` (client tools, not full servers)
- [ ] Copy latest backup from current server
- [ ] Verify restore drill works

### S1 (Database) — Second

- [ ] Start `docker-compose.infra.multi.yml` (Postgres, MongoDB, Kafka)
- [ ] Restore data from backup
- [ ] Create `arm_monitoring` users for exporters
- [ ] Start Alloy forwarder (pushes DB logs to S3 Loki)
- [ ] Verify: health checks pass, data integrity OK

### S3 (Observability) — Third

- [ ] Start `docker-compose.observe.multi.yml`
- [ ] Verify: exporters scrape S1 databases
- [ ] Verify: Grafana loads, dashboards show data

### S2 (Application) — Fourth

- [ ] Create `.env` with private IPs (`DB_HOST=10.0.1.1`, etc.)
- [ ] Build and start `docker-compose.app.multi.yml`
- [ ] Verify: all 8 app containers healthy
- [ ] Verify: S2 → S1 database connectivity
- [ ] Verify: S2 Alloy → S3 Loki log shipping
- [ ] Verify: S2 OTLP → S3 Tempo trace shipping

### Cutover

- [ ] DNS A record: `SITE_DOMAIN` → S2 public IP
- [ ] DNS A record: `GRAFANA_DOMAIN` → S3 public IP (or leave SSH-only)
- [ ] Verify: TLS cert issued by Caddy
- [ ] Verify: login, calendar sync, admin portal all work
- [ ] Set up backup cron on S4 (daily 02:00 from S1 over private network)
- [ ] Set up restore drill cron on S4 (weekly)
- [ ] Apply Hetzner Cloud Firewall rules (Section 10.5)
- [ ] Update `deploy-production.yml` GitHub Secrets for multi-server
- [ ] Verify: CI/CD deploy pipeline works end-to-end
- [ ] Keep old server running 1 week as fallback, then decommission

---

## Files to Create/Modify

### New files
| File | Purpose |
|---|---|
| `docker-compose.infra.multi.yml` | S1: Database stack (Postgres, MongoDB, Kafka — no Redis) |
| `docker-compose.app.multi.yml` | S2: App stack + Redis + edge (Caddy, Kong, Authelia) |
| `docker-compose.observe.multi.yml` | S3: Full observability stack |
| `observability/prometheus.multi.yml` | Prometheus config with private IP targets |
| `observability/alloy/config.multi.alloy` | Alloy config for S3 (receives logs from S1/S2 Alloy instances) |
| `observability/alloy/config.multi-forwarder.alloy` | Lightweight Alloy config for S1/S2 (forwards to S3 Loki) |
| `observability/grafana/datasources-multi/datasources.yml` | Grafana datasource config (all localhost on S3) |
| `observability/grafana/provisioning/alerting/rules-security.yaml` | Security-focused alert rules (SSH brute force, cert expiry, restart loops) |
| `make/multi-server.mk` | Makefile targets for multi-server deploy |
| `scripts/backup-remote.sh` | S4: Remote backup script (pg_dump/mongodump over network) |
| `scripts/restore-drill-remote.sh` | S4: Automated restore verification |
| `scripts/harden-server.sh` | OS hardening script (Sections 10.1–10.10 automated) |
| `docs/arm-docs/docs/MULTI-SERVER-MIGRATION.md` | Migration runbook with hardening steps |
| `docs/arm-docs/docs/DISASTER-RECOVERY.md` | DR runbook per server (Section 16.4) |
| `docs/arm-docs/docs/MAINTENANCE-SCHEDULE.md` | Monthly/quarterly/annual maintenance checklist |

### Modified files
| File | Change |
|---|---|
| `docker-compose.infra.prod.yml:141` | Redis eviction: `noeviction` → `volatile-lru` |
| `docker-compose.infra.prod.yml:55` | Postgres max_connections: `200` → `250` |
| `docker-compose.infra.redis-sentinel.yml:19,51,83` | Redis eviction: `noeviction` → `volatile-lru` |
| `.env.example` | Add `MULTI_SERVER_MODE`, `DB_SERVER_IP`, `APP_SERVER_IP`, `OBSERVE_SERVER_IP`, `BACKUP_SERVER_IP` |
| `.github/workflows/deploy-production.yml` | Multi-server deploy steps (Phase 5.1) |
| `Makefile` | Include `make/multi-server.mk` |

### NOT modified (important)
- Generated compose files in `generated/prod/` — these already use `${DB_HOST:-default}` patterns that work with `.env` overrides
- Application source code — all connections are env-var driven, no code changes needed
- `Caddyfile.prod` — Caddy references Docker service names that remain valid on S2
- `kong/kong.yml.tmpl` — Kong references Docker service names that remain valid on S2

---

## Verification

### Phase 0 verification (pre-split fixes)
- Restart Redis container, verify `CONFIG GET maxmemory-policy` returns `volatile-lru`
- Check `SHOW max_connections;` on PostgreSQL returns 250
- Run `make backup-all`, verify S3 bucket has new backup files

### Hardening verification (per server)
- `ssh root@<ip>` is rejected (root login disabled)
- `ssh deploy@<ip>` with password is rejected (password auth disabled)
- `ufw status verbose` shows correct rules, default deny incoming
- `sysctl net.ipv4.conf.all.rp_filter` returns 1
- `swapon --show` shows 4G swap
- `fail2ban-client status sshd` shows jail active
- `chronyc tracking` shows synchronized
- `docker info` shows `no-new-privileges: true`
- From S4: `psql -h 10.0.1.1` works (firewall allows)
- From S3: `psql -h 10.0.1.2` fails (firewall blocks — S3 shouldn't reach S2's Postgres because S2 has no Postgres)
- From internet: `nmap 10.0.1.1` returns nothing (no public IP)

### Connectivity verification (after migration)
- From S2: `psql -h 10.0.1.1 -U admin -d arm_core -c "SELECT 1;"` (Postgres)
- From S2: `mongosh mongodb://...@10.0.1.1:27017/admin --eval "db.ping()"` (MongoDB)
- From S2: `redis-cli -h localhost ping` (Redis is local)
- All 8 app containers healthy: `docker ps` on S2
- Grafana dashboards show metrics from all services: check on S3
- Loki shows logs from S1 and S2 containers: check on S3
- Run backup on S4, verify dump files created
- Run restore drill on S4, verify success
- Browser test: login, navigate calendar, create event, admin portal
- Prometheus alerts firing: trigger a test alert

### Security verification
- `https://SITE_DOMAIN` loads with valid TLS cert (check in browser)
- `http://SITE_DOMAIN` redirects to HTTPS
- Kong rate limiting: hit `/api/core/v1/auth/login` > 20 times/min → 429
- Grafana behind Authelia: `https://GRAFANA_DOMAIN` redirects to Authelia login
- `.env` permissions: `stat ~/arm-deploy/.env` shows `-rw-------` (600)
- Database ports not reachable from internet: `curl http://SITE_DOMAIN:5432` times out

### arm-cli verification

**Phase 0-1:** `arm-cli --version` works, no infra generator code remains.

**Phase 2 (descriptor fix):**
- Generate a service: `arm-cli scaffold --kind service --db PostgreSQL --port 4300 -y`
- Inspect Makefile descriptor: `EXTRA_ENV_PROD_*` uses `${DB_HOST:-arm-postgres-prod}` (not bare value)
- `EXTRA_ENV_LOCAL_*` uses bare `arm-postgres-dev` (no override needed locally)
- Add to `services.conf`, run `make generate-compose-prod` — succeeds
- Set `DB_HOST=10.0.1.1` in `.env`, re-run — compose file contains `DB_HOST: 10.0.1.1`

**Phase 5 (validate):**
- `arm-cli validate` against full workspace — all 8 repos pass
- Break a descriptor intentionally — `arm-cli validate` catches it
- Remove an env var from `.env.example` — `arm-cli validate` flags it
- Assign duplicate port to two services — `arm-cli validate` detects collision

**Phase 6 (check):**
- `arm-cli check` from laptop shows all 4 servers green
- Stop a container on S2 — `arm-cli check health` reports the failure
- Fill Redis to 85% — `arm-cli check redis` shows warning

**Phase 7 (end-to-end):**
- Full lifecycle: scaffold → validate → generate-compose → deploy → check

---

---

# Part 3 — arm-tool-cli: The Platform CLI

## Context

`arm-tool-cli` (at `arm/arm-tool-cli/`) is currently a scaffolding tool that generates new ARM repos. This plan expands it into the **single developer-side CLI** for the ARM platform — scaffolding, validation, and server health checks — while keeping compose generation and deployment in `arm-deploy-make` where they belong (no Node.js on production servers).

**Existing plan:** `ok-then-plan-all-gentle-ocean.md` covers Phases 0-5 (make the CLI runnable, fix scaffolding). This plan supersedes it with a complete 8-phase roadmap.

### Responsibility Split

| arm-cli (developer laptop) | arm-deploy-make (servers) |
|---|---|
| Scaffold new repos | Generate compose files from descriptors |
| Validate all repos against ARM contract | Run compose (up/down/build) |
| Validate descriptors | Manage infra (backup, restore) |
| Check server health over SSH | CI/CD make targets |
| Port allocation & collision detection | Kong config rendering |
| Template conformance enforcement | Env var quick-check (`make env-check`) |

The **descriptor contract** is the clean interface between them. arm-cli produces and validates descriptors; arm-deploy-make consumes them.

```
Developer laptop                           Servers (S1-S4)
────────────────                           ────────────────
arm-cli scaffold ───── descriptor ─────→   orchestrator generate-compose
arm-cli validate ───── descriptor ─────→   (lightweight re-check only)
arm-cli check ──────── SSH ────────────→   docker ps / redis-cli / pg_isready
                                           make ci-deploy-prod
                                           docker compose up -d
```

---

## Phase 0: Make the CLI Runnable

> From existing plan `ok-then-plan-all-gentle-ocean.md` — unchanged.

**Goal:** `arm-cli --help` works after `pnpm build && pnpm link --global`.

**Files:**
- `arm-tool-cli/package.json` — fix build scripts (tsup, tsc-alias, copy-templates)
- `arm-tool-cli/tsconfig.json` — alias resolution
- `arm-tool-cli/bin/cli.ts` — entry point

**Verify:** `arm-cli --version` prints `1.0.0`.

---

## Phase 1: Delete Generic Infrastructure Generators

> From existing plan — unchanged.

**Goal:** Remove ~3,300 lines of code that generate CI/CD workflows, Docker infrastructure, Kafka/Redis configs. ARM's infra is centrally owned by `arm-deploy-make` — generated repos must not duplicate it.

**Delete:**
- CI/CD generator (open issue CLI-002)
- Docker/Kafka/Redis generators
- AWS/Hetzner workflow templates

**Keep:** Everything related to scaffolding (NestJS backend, React frontend, Makefile, descriptor).

**Verify:** `arm-cli scaffold --kind service --db PostgreSQL --port 4300 -y` generates a repo with no CI/CD files, no docker-compose, no infra config.

---

## Phase 2: Fix Descriptors for Multi-Server

**Goal:** Generated descriptors use the `${VAR:-default}` pattern for all infrastructure hostnames, matching existing hand-written services.

### The Bug

**File:** `arm-tool-cli/src/core/descriptor/build-descriptor.ts:62-76`

`prodDbEnv` hardcodes bare Docker DNS names:
```typescript
// Current — breaks in multi-server mode:
{ key: "DB_HOST", value: "arm-postgres-prod" }

// Fixed — .env can override with private IP:
{ key: "DB_HOST", value: "${DB_HOST:-arm-postgres-prod}" }
```

### Changes

| File | Change |
|---|---|
| `build-descriptor.ts:65` | `"arm-postgres-prod"` → `"${DB_HOST:-arm-postgres-prod}"` |
| `build-descriptor.ts:66` | `"5432"` → `"${DB_PORT:-5432}"` |
| `descriptor.test.ts` | Update test expectations |

`localDbEnv` (bare `arm-postgres-dev`) stays unchanged — local dev doesn't need overrides.

MongoDB `${MONGODB_URI}` is already correct — reads the full URI from `.env`.

### Multi-Server Conformance Rule

Add to `docs/service-descriptor.md`:

> Every `EXTRA_ENV_PROD_*` that references a shared infrastructure hostname MUST use `${VAR:-default}`, not a bare Docker DNS name.

| Env var | Pattern |
|---|---|
| `DB_HOST` | `${DB_HOST:-arm-postgres-prod}` |
| `DB_PORT` | `${DB_PORT:-5432}` |
| `MONGODB_URI` | `${MONGODB_URI}` (full URI from .env) |
| `REDIS_HOST` | `${REDIS_HOST:-arm-redis-prod}` |
| `KAFKA_BROKERS` | `${KAFKA_BROKERS:-arm-kafka1-prod:9092,...}` |

Inter-service URLs (`http://core-backend:3000`) do NOT need this — they're Docker service names on the same host.

**Verify:** Generate a service, inspect its Makefile descriptor, confirm `EXTRA_ENV_PROD_*` lines use the `${VAR:-default}` pattern.

---

## Phase 3: ARM-Conformant Templates

> From existing plan — unchanged.

**Goal:** Generated React frontends and NestJS backends match the structure and conventions of existing ARM repos (arm-admin, arm-app-calendar).

- React 19 + Vite 7 + UnoCSS frontends
- NestJS backends with TypeORM/Mongoose
- File headers, versioning, naming per `TEMPLATE-CONFORMANCE.md`

**Verify:** Generated repo passes a diff-check against arm-admin's structure.

---

## Phase 4: Emit Registration Patch

> From existing plan — unchanged.

**Goal:** Instead of auto-editing `services.conf` and `.env.example` (which risks breaking things), arm-cli emits a `workspace-registration.patch` and a checklist. The developer applies it manually.

The patch adds:
- One row to `services.conf`
- Env var keys to `.env.example`

**Verify:** `git apply workspace-registration.patch` applies cleanly, `make env-check` passes.

---

## Phase 5: `arm-cli validate` — Workspace-Wide Validation

**Goal:** Replace the scattered validation in arm-deploy-make's bash scripts with a proper TypeScript validator that catches problems before compose generation or deployment.

### Commands

```bash
arm-cli validate              # validate all repos in the workspace
arm-cli validate core-be      # validate one repo
arm-cli validate --fix        # auto-fix what can be fixed (e.g., missing .env.example keys)
```

### What It Checks

#### 5.1 — Descriptor Validation

For each repo listed in `services.conf`:
- Run `make descriptor` in the repo, parse output
- Validate every field against the contract (`descriptor-fields.ts` rules)
- Check `EXTRA_ENV_PROD_*` lines use `${VAR:-default}` for infra hostnames (Phase 2 rule)
- Verify `REQUIRED_ENV_VARS` names all exist as keys in `deploy/arm-deploy-make/.env.example`
- Validate `KIND`, `FE_BE`, `PORT`, `HEALTH_PATH` are present and sane

#### 5.2 — Port Collision Detection

- Collect all `PORT`, `HOST_PORT`, `PROD_HOST_PORT` from every descriptor
- Flag any collision (two services claiming the same port)
- Use the existing `ALLOCATED_PORTS` map in `src/core/registration/ports.ts`

#### 5.3 — Structure Check

For each repo:
- `Dockerfile` exists
- `Dockerfile.local.dev` exists
- `Makefile` has a `descriptor` target
- `package.json` exists with expected scripts
- Health endpoint responds at `HEALTH_PATH` when the service is running

#### 5.4 — Template Conformance

Check against `TEMPLATE-CONFORMANCE.md` rules:
- File headers present (SPDX license, copyright, author)
- Versioning file exists
- Git conventions (branch naming, commit format)
- Required Makefile targets present

#### 5.5 — Env Var Completeness

- Every `REQUIRED_ENV_VARS` in every descriptor → must exist in `.env.example`
- Every `EXTRA_ENV_PROD_*` that uses `${VAR}` or `${VAR:-default}` → the `VAR` part must exist in `.env.example`
- Flag vars in `.env.example` that no descriptor references (dead vars)

### Output

```
arm-cli validate

  core-be          ✓ descriptor  ✓ ports  ✓ structure  ✓ conformance  ✓ env vars
  arm-session          ✓ descriptor  ✓ ports  ✓ structure  ✓ conformance  ✓ env vars
  calendar-be      ✓ descriptor  ✓ ports  ✓ structure  ✓ conformance  ✓ env vars
  admin-be         ✓ descriptor  ✓ ports  ✓ structure  ✓ conformance  ✓ env vars
  notification     ✓ descriptor  ✓ ports  ✓ structure  ✗ conformance  ✓ env vars
                     └── missing SPDX header in 3 files

  8/8 repos checked, 1 warning, 0 errors
```

### What Stays in arm-deploy-make

`make env-check` stays as a lightweight bash check for CI (no Node.js dependency). `orchestrator/generate-compose.sh` keeps its basic "does the env var exist in .env" check as a safety net. But the thorough validation is `arm-cli validate`.

### Files

| File | Purpose |
|---|---|
| `arm-tool-cli/src/commands/validate.ts` | Command entry point |
| `arm-tool-cli/src/core/validation/descriptor-validator.ts` | Runs `make descriptor`, parses and validates |
| `arm-tool-cli/src/core/validation/port-validator.ts` | Port collision detection |
| `arm-tool-cli/src/core/validation/structure-validator.ts` | File existence checks |
| `arm-tool-cli/src/core/validation/conformance-validator.ts` | TEMPLATE-CONFORMANCE rules |
| `arm-tool-cli/src/core/validation/env-validator.ts` | Env var completeness |

**Verify:** Run `arm-cli validate` against the full workspace (all 8 existing repos). All pass. Intentionally break a descriptor, re-run, confirm it catches the error.

---

## Phase 6: `arm-cli check` — Server Health Checks

**Goal:** One command to check the health of the multi-server production setup from a developer's laptop, replacing the manual monthly checklist (Part 2, Section 18.2).

### Commands

```bash
arm-cli check                 # check everything
arm-cli check health          # container health on all servers
arm-cli check backups         # last backup timestamp, size, S3 sync status
arm-cli check firewall        # ufw status on each server, diff against expected rules
arm-cli check certs           # TLS cert expiry on S2
arm-cli check disk            # disk usage on all servers
arm-cli check redis           # memory usage, eviction policy, connected clients
arm-cli check postgres        # connection count vs max_connections, replication lag
arm-cli check logs            # recent errors from Loki (last 1h)
```

### How It Works

arm-cli SSHes into each server using the developer's local SSH key and runs standard commands — no Node.js or make targets needed on the server.

```
arm-cli check health
  ├── SSH 10.0.1.1 → docker ps --format '{{.Names}} {{.Status}}'
  ├── SSH 10.0.1.2 → docker ps --format '{{.Names}} {{.Status}}'
  ├── SSH 10.0.1.3 → docker ps --format '{{.Names}} {{.Status}}'
  └── SSH 10.0.1.4 → ls -la /mnt/backups/ | tail -5
  
  → Parses output, prints dashboard
```

### Server Config

arm-cli reads server IPs from a config file (gitignored, developer-local):

```yaml
# arm-tool-cli/servers.yml
servers:
  db:      { host: 10.0.1.1, user: deploy }
  app:     { host: 10.0.1.2, user: deploy }
  observe: { host: 10.0.1.3, user: deploy }
  backup:  { host: 10.0.1.4, user: deploy }
```

Or reads from `deploy/arm-deploy-make/.env` if `DB_SERVER_IP` etc. are set.

### Output

```
arm-cli check

  ┌─────────────────────────────────────────────────────────────────┐
  │ ARM Platform Health                           2026-08-24 15:30  │
  ├─────────────────────────────────────────────────────────────────┤
  │                                                                 │
  │ S1 (DB)       10.0.1.1                                         │
  │   ✓ postgres    healthy    connections: 87/250                  │
  │   ✓ mongodb     healthy    wiredTiger cache: 0.3/0.4 GB        │
  │   ✓ kafka       healthy    1 broker                            │
  │   ✓ disk        34% used                                       │
  │                                                                 │
  │ S2 (App)      10.0.1.2                                         │
  │   ✓ 8/8 services healthy                                       │
  │   ✓ redis       45% mem    policy: volatile-lru                │
  │   ✓ caddy       cert expires in 58 days                        │
  │   ✓ kong        healthy                                        │
  │   ✓ disk        28% used                                       │
  │                                                                 │
  │ S3 (Observe)  10.0.1.3                                         │
  │   ✓ prometheus  healthy    30d retention                       │
  │   ✓ grafana     healthy    behind authelia                     │
  │   ✓ loki        healthy    30d retention                       │
  │   ✓ tempo       healthy                                        │
  │   ✓ disk        41% used                                       │
  │                                                                 │
  │ S4 (Backup)   10.0.1.4                                         │
  │   ✓ last backup: 6h ago    arm_core: 42MB  arm-calendar: 18MB │
  │   ✓ S3 sync:    6h ago     arm-backups bucket                  │
  │   ✓ last drill:  3d ago    passed                              │
  │   ✓ disk        22% used   /mnt/backups                        │
  │                                                                 │
  └─────────────────────────────────────────────────────────────────┘
```

### What arm-cli check Does NOT Do

- **Deploy** — that's GitHub Actions (`deploy-production.yml`)
- **Rollback** — that's `make rollback TAG=<sha>` via CI
- **Mutate server state** — all checks are read-only SSH commands
- **Replace Grafana alerts** — Grafana alerts are 24/7 automated; `arm-cli check` is on-demand from a developer's laptop

### Files

| File | Purpose |
|---|---|
| `arm-tool-cli/src/commands/check.ts` | Command entry point + subcommands |
| `arm-tool-cli/src/core/server/ssh-client.ts` | SSH connection wrapper (uses `node:child_process` + system `ssh`) |
| `arm-tool-cli/src/core/server/health-checker.ts` | Parse docker ps, health endpoints |
| `arm-tool-cli/src/core/server/backup-checker.ts` | Parse backup file listing, S3 sync status |
| `arm-tool-cli/src/core/server/resource-checker.ts` | Disk, memory, Redis, Postgres stats |
| `arm-tool-cli/servers.yml.example` | Example config (committed); `servers.yml` is gitignored |

**Verify:** Run `arm-cli check` from laptop against the running multi-server setup. All green. Stop one container on S2, re-run, confirm it reports the failure.

---

## Phase 7: End-to-End Validation

**Goal:** Prove the full lifecycle works — scaffold a service, validate it, deploy it.

### Test Sequence

1. `arm-cli scaffold --kind service --db PostgreSQL --port 4300 -y` in a scratch workspace
2. `arm-cli validate` — confirm the generated repo passes all checks
3. Apply the registration patch to `services.conf` + `.env.example`
4. `make generate-compose-prod` — confirm compose generation succeeds
5. Set `DB_HOST=10.0.1.1` in `.env`, re-run compose generation — confirm the override takes effect
6. `arm-cli check` — confirm servers are healthy before and after
7. Clean up scratch workspace

### Register arm-cli Itself

Apply `workspace-registration.patch` to add arm-tool-cli to `repos.conf` at `tools/arm-tool-cli`. Document installation in `ONBOARDING.md`.

**Verify:** A new developer can clone the workspace, `cd tools/arm-tool-cli && pnpm install && pnpm build && pnpm link --global`, then use `arm-cli` everywhere.

---

## Summary: arm-cli Command Map

```
arm-cli
├── scaffold                    # Phase 1-4: Generate new ARM repos
│   ├── --kind app|service
│   ├── --db PostgreSQL|MongoDB|None
│   ├── --port <N>
│   └── --frontend-port <N>     # apps only
│
├── validate                    # Phase 5: Workspace-wide validation
│   ├── [service-name]          # optional: validate one repo
│   └── --fix                   # auto-fix trivial issues
│
└── check                       # Phase 6: Server health checks (SSH)
    ├── health                  # container status on all servers
    ├── backups                 # backup recency, S3 sync, drill status
    ├── firewall                # ufw rules diff
    ├── certs                   # TLS cert expiry
    ├── disk                    # disk usage all servers
    ├── redis                   # memory, eviction policy
    ├── postgres                # connection count, replication
    └── logs                    # recent errors from Loki
```

### What Lives Where

| Concern | arm-cli | arm-deploy-make |
|---|---|---|
| Scaffold new repos | ✓ | |
| Validate repos & descriptors | ✓ (thorough, TypeScript) | lightweight bash safety net |
| Port allocation | ✓ | |
| Template conformance | ✓ | |
| Compose generation | | ✓ (bash, no Node.js on servers) |
| Deployment | | ✓ (via GitHub Actions) |
| Rollback | | ✓ (`make rollback`) |
| Infra management | | ✓ (`make infra-up`, `backup-*`) |
| Server health checks | ✓ (SSH, read-only) | |
| Alerting (24/7) | | ✓ (Grafana/Prometheus) |

### Files to Create/Modify

**New files in arm-tool-cli:**

| File | Phase | Purpose |
|---|---|---|
| `src/commands/validate.ts` | 5 | `arm-cli validate` entry point |
| `src/core/validation/descriptor-validator.ts` | 5 | Descriptor field + multi-server pattern checks |
| `src/core/validation/port-validator.ts` | 5 | Port collision detection |
| `src/core/validation/structure-validator.ts` | 5 | File existence (Dockerfile, Makefile, package.json) |
| `src/core/validation/conformance-validator.ts` | 5 | TEMPLATE-CONFORMANCE rules |
| `src/core/validation/env-validator.ts` | 5 | Env var completeness |
| `src/commands/check.ts` | 6 | `arm-cli check` entry point |
| `src/core/server/ssh-client.ts` | 6 | SSH wrapper |
| `src/core/server/health-checker.ts` | 6 | Parse docker ps output |
| `src/core/server/backup-checker.ts` | 6 | Backup recency and drill status |
| `src/core/server/resource-checker.ts` | 6 | Disk, memory, DB stats |
| `servers.yml.example` | 6 | Example server config |

**Modified files:**

| File | Phase | Change |
|---|---|---|
| `src/core/descriptor/build-descriptor.ts:65-66` | 2 | `${DB_HOST:-arm-postgres-prod}` pattern |
| `src/utils/__tests__/descriptor.test.ts` | 2 | Update test expectations |
| `deploy/arm-deploy-make/docs/service-descriptor.md` | 2 | Add multi-server conformance rule |
| `bin/cli.ts` | 5-6 | Register `validate` and `check` commands |
| `package.json` | 5-6 | Add SSH dependency if needed (or use system ssh) |

**NOT modified:**

| File | Why |
|---|---|
| `build-descriptor.ts` `localDbEnv` | Local dev uses Docker DNS correctly — no override needed |
| `orchestrator/generate-compose.sh` | Stays in arm-deploy-make — no Node.js on servers |
| `orchestrator/templates/prod/*.yml.tmpl` | Templates are generic, paste `EXTRA_ENV_PROD_*` as-is |
| `make/ci.mk` | Deployment stays in make + GitHub Actions |
| `Caddyfile.prod`, `kong/kong.yml.tmpl` | Centrally managed, not per-service |
