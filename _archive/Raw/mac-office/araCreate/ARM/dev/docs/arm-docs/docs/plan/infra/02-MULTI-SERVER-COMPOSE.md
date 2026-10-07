# 02 — Multi-Server Compose Files

> New compose files that split the current single-server stack across 4 servers.

---

## How S2 Connects to S1

App services read database hostnames from env vars. Today those default to Docker DNS names. In multi-server mode, `.env` overrides them with private IPs:

```
# Single-server (Docker DNS)     →  Multi-server (.env override)
DB_HOST=arm-postgres-prod        →  DB_HOST=10.0.1.1
MONGODB_URI=...@arm-mongodb:27017 → MONGODB_URI=...@10.0.1.1:27017
KAFKA_BROKERS=arm-kafka1:9092    →  KAFKA_BROKERS=10.0.1.1:9092
REDIS_HOST=arm-redis-prod        →  REDIS_HOST=arm-redis-prod  (stays — Redis is on S2)
```

No application code changes. The existing `${VAR:-default}` pattern in descriptors handles this.

---

## 2.1 `docker-compose.infra.multi.yml` — S1 (Database)

Based on `docker-compose.infra.prod.yml`. Changes:

- **Remove** Redis (moves to S2)
- **Add** port exposure: `0.0.0.0:5432`, `0.0.0.0:27017`, `0.0.0.0:9092`
  - S1 has no public IP, so `0.0.0.0` only means "reachable on the private network"
- Keep health checks, resource limits, volumes, log rotation unchanged
- Optional: add PgBouncer on port 6432 (evaluate if connection pressure warrants it)

---

## 2.2 `docker-compose.app.multi.yml` — S2 (App + Redis)

Based on `docker-compose.yml`. Changes:

- **Keep:** Caddy, Kong, Authelia, all 8 generated app service includes
- **Add:** Redis container (same config as current infra prod)
- **Remove:** All observability containers (prometheus, exporters, loki, alloy, tempo, grafana)
- **Remove:** `arm-infra-network: external: true` (infra is on S1, not same Docker network)
- **Keep:** `arm-prod-network` (local to S2)
- Caddy keeps ports 80, 443 published

---

## 2.3 `docker-compose.observe.multi.yml` — S3 (Observability)

Based on observability section of `docker-compose.yml`. Changes:

- **Move here:** Prometheus, Grafana, Loki, Tempo, Alloy, node-exporter, cAdvisor, all 3 DB exporters
- Exporter connection strings → private IPs:
  - postgres-exporter: `arm-postgres-prod` → `10.0.1.1`
  - mongodb-exporter: `arm-mongodb-prod` → `10.0.1.1`
  - redis-exporter: `arm-redis-prod` → `10.0.1.2` (Redis is on S2)
- Grafana behind Authelia (or SSH tunnel — see [(secret removed)]((secret removed)))

---

## 2.4 Monitoring Config Changes

### Prometheus (`observability/prometheus.multi.yml`)

App metrics scraped over private network from S2. Publish metrics ports on S2's private IP only:

```yaml
scrape_configs:
  - job_name: core-be
    static_configs:
      - targets: ["10.0.1.2:3000"]    # S2 private IP
  - job_name: session
    static_configs:
      - targets: ["10.0.1.2:5000"]
  # ... all 5 backends + Kong at 10.0.1.2
  
  - job_name: postgres
    static_configs:
      - targets: ["arm-postgres-exporter:9187"]  # co-located on S3
```

### Alloy (Log Collection)

Docker socket is host-local. Run a lightweight Alloy on S1 and S2, each pushing to Loki on S3:

- **S1 Alloy:** tails Postgres/MongoDB/Kafka logs → `http://10.0.1.3:3100/loki/api/v1/push`
- **S2 Alloy:** tails app container logs → same Loki endpoint
- **S3 Alloy:** tails observability logs locally + handles OTLP trace ingestion from S2

### OTLP Traces

App services on S2: `OTEL_EXPORTER_OTLP_ENDPOINT=http://10.0.1.3:4317`

---

## 2.5 New Env Vars

Add to `.env.example`:

```bash
# Multi-server topology
DB_SERVER_IP=
APP_SERVER_IP=
OBSERVE_SERVER_IP=
BACKUP_SERVER_IP=
```

On S2's `.env`: `DB_HOST=10.0.1.1`, `KAFKA_BROKERS=10.0.1.1:9092`, etc.

---

## 2.6 New Makefile Targets

**File:** `deploy/arm-deploy-make/make/multi-server.mk`

```makefile
infra-up-multi:        # S1: Postgres + MongoDB + Kafka (no Redis)
app-deploy-multi:      # S2: Redis + 8 apps + Caddy + Kong + Authelia
observe-deploy-multi:  # S3: Full observability stack
backup-setup-multi:    # S4: Install backup cron, client tools
```
