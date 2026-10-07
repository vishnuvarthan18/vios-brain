# 09 — Monitoring & Security Alerts

---

## 9.1 Existing Alerts (8, Already in Grafana)

Service down, Postgres down, MongoDB down, Redis down, high 5xx ratio, disk space low, Redis memory warning, Redis memory critical.

## 9.2 Add These

| Alert | Condition | Why |
|---|---|---|
| SSH brute force | fail2ban bans > 5/hour | Someone probing |
| TLS cert expiry | < 14 days remaining | Prevent accidental outage |
| Disk usage on S4 | > 80% | Backup volume filling |
| Container restart loop | > 3 restarts in 5 min | Crash or resource exhaustion |
| Kong 401/403 spike | > 20% auth failures in 5 min | Credential stuffing |

## 9.3 Log Security

All container logs flow to Loki on S3 — S3 holds potentially sensitive data.

- Loki retention: 30 days (auto-deleted)
- Grafana/Loki gated by Authelia
- **Open gap:** `pino` `redact` not deployed — tokens in error logs go to Loki in cleartext. Most urgent security follow-up.
