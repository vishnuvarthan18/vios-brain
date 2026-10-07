# 03 — Network & Firewall

> Hetzner Cloud Network setup and per-server firewall rules (two layers: Hetzner Cloud Firewall + host UFW).

---

## 3.1 Hetzner Cloud Network

- Name: `arm-private`
- Range: `10.0.0.0/16`
- Subnet: `10.0.1.0/24`, zone: `eu-central`

| Server | Private IP | Public IP |
|---|---|---|
| S1 (DB) | 10.0.1.1 | None |
| S2 (App) | 10.0.1.2 | Yes (IPv4) |
| S3 (Observe) | 10.0.1.3 | Optional |
| S4 (Backup) | 10.0.1.4 | None |

---

## 3.2 Firewall Rules

Two identical layers — Hetzner Cloud Firewall at the hypervisor, UFW on the host. Defence-in-depth.

### S1 (Database) — No Public IP

| Allow from | Port | Purpose |
|---|---|---|
| 10.0.1.2 (S2) | 5432 | App → Postgres |
| 10.0.1.2 (S2) | 27017 | App → MongoDB |
| 10.0.1.2 (S2) | 9092 | App → Kafka |
| 10.0.1.3 (S3) | 5432, 27017 | Exporters → DB metrics |
| 10.0.1.4 (S4) | 5432, 27017 | Backup dumps |
| 10.0.1.0/24 | 22 | SSH from private network |

```bash
ufw default deny incoming && ufw default allow outgoing
ufw allow from 10.0.1.0/24 to any port 22
ufw allow from 10.0.1.2 to any port 5432
ufw allow from 10.0.1.2 to any port 27017
ufw allow from 10.0.1.2 to any port 9092
ufw allow from 10.0.1.3 to any port 5432
ufw allow from 10.0.1.3 to any port 27017
ufw allow from 10.0.1.4 to any port 5432
ufw allow from 10.0.1.4 to any port 27017
ufw enable
```

### S2 (App) — Public IP

| Allow from | Port | Purpose |
|---|---|---|
| 0.0.0.0/0 | 80, 443 | Caddy (browser traffic + ACME) |
| 10.0.1.3 (S3) | 3000, 4001, 5000, 8100, 10001 | Prometheus → app metrics |
| 10.0.1.3 (S3) | 6379 | redis-exporter → Redis |
| 10.0.1.0/24 | 22 | SSH |

### S3 (Observe) — Optional Public IP

| Allow from | Port | Purpose |
|---|---|---|
| 10.0.1.1, 10.0.1.2 | 3100 | Alloy → Loki |
| 10.0.1.2 | 4317, 4318 | OTLP traces → Tempo |
| 0.0.0.0/0 | 443 | Grafana (if public, behind Authelia) |
| 10.0.1.0/24 | 22 | SSH |

### S4 (Backup) — No Public IP

| Allow from | Port | Purpose |
|---|---|---|
| 10.0.1.0/24 | 22 | SSH |

---

## 3.3 Key Isolation Properties

- **No server trusts another's Docker network** — all cross-server communication is TCP over private IPs
- **S1 is unreachable from the internet** — no public IP, databases never exposed
- **S4 is unreachable from the internet** — backup data physically isolated
- **A compromised S3 cannot reach S2's Redis** — UFW on S2 only allows S3 to scrape metrics ports, not 6379 (correction: redis-exporter on S3 needs 6379 — that's the only exception)
