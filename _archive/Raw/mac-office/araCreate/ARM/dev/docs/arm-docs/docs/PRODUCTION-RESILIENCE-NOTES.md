# Production Resilience & Overload Handling — Reference Notes

> A condensed reference on why systems overload, how to handle it, and the surrounding production-readiness concerns (data layer, infra, observability, release process, security, DR). Not ARM-specific — general production engineering knowledge to apply when hardening any of the services in this workspace.

---

## 1. Overload: why it happens

### The physics

- **Little's Law:** `concurrency = arrival_rate × latency`. Rising latency raises concurrency, which exhausts resources, which raises latency further — a positive feedback loop.
- **Universal Scalability Law:** throughput doesn't scale linearly — contention (locks, DB) and coherency (cross-node sync) make it fall past a peak. There is a load level where adding traffic *reduces* goodput.
- **Metastable failure:** a trigger (deploy, cache flush, spike) pushes the system into a state that sustains itself after the trigger is gone. Retry storms and cold caches are the classic amplifiers. Recovery requires shedding load, not waiting.
- **Retry amplification:** 3 layers × 3 retries = 27× load exactly when the system is weakest.
- **Unbounded queues are latency bombs:** they convert overload into 100% timeout with full CPU burn (work is done on requests the client already abandoned).

### The two different "throttles" people conflate

- **Request throttling** — you reject/delay traffic (rate limit, load shed).
- **CPU throttling** — the kernel CFS quota stops your container mid-slice. `cpu: 0.5` on a Node process with an 8-core-parallel GC will throttle hard at low average utilization. Check `container_cpu_cfs_throttled_seconds_total`. Prefer CPU *requests* with generous/no limits for latency-sensitive services; always set memory limits.

---

## 2. Handling overload

Order of preference: **prevent → shed → degrade → fail fast**.

- **Concurrency limits, not just rate limits.** Rate limits are open-loop and mis-tuned by definition. Bounded in-flight limits (semaphore per endpoint/dependency) directly enforce Little's Law. Adaptive versions: Netflix concurrency-limits (Gradient2/Vegas), Envoy adaptive concurrency — they infer capacity from the latency gradient.
- **Rate limiting algorithms:** token bucket (bursty, most common), GCRA/leaky bucket (smooth), sliding window counter (cheap approximation), sliding window log (accurate, expensive). Know the key: per-IP is trivially defeated; prefer per-API-key/tenant/user + a global backstop.
- **Kong:** local policy is per-node (N nodes = N× the limit); Redis is correct but adds latency + a SPOF. Pick deliberately.
- **Load shedding:** bounded queue + queue-time deadline (CoDel-style). If a request has waited > 100ms in queue, drop it — it's likely dead already. Under sustained overload, LIFO beats FIFO (serve fresh requests successfully rather than every request slowly).
- **Priority classes:** shed batch/analytics/retry traffic before interactive; shed anonymous before paying. Propagate a criticality header.
- **Timeout budget:** deadlines must decrease down the stack, and be propagated (`X-Request-Deadline` / gRPC deadline). Client 3s → gateway 2.5s → service 2s → DB 1.5s. A downstream timeout longer than upstream is pure wasted capacity.
- **Retries:** exponential backoff + full jitter; retry only idempotent ops; retry budget (cap retries at ~10% of requests — Google SRE); retry at one layer only; never retry on 429/503 without `Retry-After`.
- **Circuit breakers** on every remote dependency, with half-open probing.
- **Bulkheads:** separate pools/threads per dependency so one slow downstream can't consume all workers.
- **Backpressure end-to-end:** queue depth must be visible to producers. Kafka/SQS decouple but hide backpressure — monitor consumer lag as a first-class SLI.
- **Graceful degradation:** stale cache, reduced page, skip recommendations, read-only mode. Decide these *before* the incident.
- **Cold-start protection:** request coalescing/singleflight, cache warming, staggered restarts, jittered TTLs (prevent synchronized expiry).

---

## 3. App layer (NestJS / FastAPI)

- **Node:** event-loop lag as an SLI (`perf_hooks.monitorEventLoopDelay`); no sync crypto/JSON of large payloads on the loop; `--max-old-space-size` set below the container memory limit (else OOMKill 137 instead of a heap error); keep-alive agents on outbound HTTP; `server.headersTimeout`/`requestTimeout` set explicitly.
- **Python:** any blocking call inside `async def` stalls the entire loop — this is the #1 FastAPI prod bug. `def` endpoints run in a threadpool (default 40 threads = hidden concurrency limit). Sizing: `uvicorn --workers` ≈ cores; gunicorn+uvicorn workers for process supervision. Check for GIL-bound CPU work → move to a worker/process pool.
- **Graceful shutdown:** SIGTERM → fail readiness probe → drain (wait > LB check interval) → finish in-flight → close pools → exit. `terminationGracePeriod` > drain time. Docker sends SIGTERM only to PID 1 — use `--init` or `tini` or handle signals properly.
- **Liveness vs readiness are different things.** Liveness that checks the DB will restart your whole fleet during a DB blip. Liveness = "is the process wedged"; readiness = "can I serve".
- **Idempotency keys** on all mutating public endpoints.
- **Pagination is mandatory** — no unbounded list endpoints, no unbounded request bodies, no unbounded fan-out.
- **Connection pools** sized and monitored (acquire wait time is the metric that matters, not pool size).
- **Config via env, validated at boot**, fail fast on missing config. No hot-path feature detection.
- **Structured JSON logs**, correlation/trace ID propagated across every hop and into DB comments.

---

## 4. Data layer (Postgres)

- **`max_connections` vs app pools:** N services × M replicas × pool size must be < `max_connections`. Use PgBouncer (transaction mode) — and know what it breaks (session state, prepared statements, `SET`, advisory locks).
- **Pool size heuristic:** ~(2 × cores) + effective_spindles. Bigger pools usually make things slower.
- **Set globally:** `statement_timeout`, `lock_timeout`, `idle_in_transaction_session_timeout`. Missing the last one is how a stuck app holds a lock and blocks a migration for hours.
- **`pg_stat_statements` on.** Know your top-10 queries by total time. Kill N+1s (`EXPLAIN (ANALYZE, BUFFERS)`).
- **Autovacuum tuning**, bloat and XID-wraparound monitoring, index bloat, unused indexes.
- **Migrations:** expand → migrate → contract. Never a breaking schema change in the same deploy as code. Avoid `ACCESS EXCLUSIVE` locks; `CREATE INDEX CONCURRENTLY`; add `NOT NULL` via `CHECK NOT VALID` + `VALIDATE`; always `SET lock_timeout` + retry in migrations.
- **Backups:** untested backup = no backup. PITR via WAL archiving (pgBackRest/wal-g), off-host + off-provider copy, encrypted, and a documented restore drill with a measured RTO. Do it quarterly. Also: backup restore is your ISO 27001 A.8.13 evidence.
- **Replication:** replica lag as an SLI; know whether your reads tolerate staleness.

---

## 5. Infra / Linux / Docker (Hetzner)

- Single server = SPOF. At minimum: IaC (so you can rebuild), automated snapshots, and a documented rebuild runbook with measured time-to-restore.
- **Resource limits on every container** (mem limit always; CPU limits carefully — see CFS throttling above). Watch OOMKilled (exit 137) and restart loops.
- **Kernel/ulimits:** file descriptors, `net.core.somaxconn`, `tcp_max_syn_backlog`, ephemeral port range, `nf_conntrack_max` (silently drops packets when full — classic mystery outage), TIME_WAIT behaviour.
- **Disk:** capacity and inode alerts, log rotation, IOPS ceiling, fsync behaviour. Full disk kills Postgres.
- **Time sync** (NTP/chrony) — token/JWT validation and log correlation break without it.
- **Image hygiene:** pinned base images, non-root user, read-only rootfs where possible, multi-stage builds, vuln scanning (Trivy), SBOM, reproducible tags (never `:latest` in prod).
- **Firewall:** default deny, DB never publicly reachable, SSH key-only + fail2ban, private network between services.
- **Egress control** (also an ISO 27001 talking point).

---

## 6. Edge (Kong / Caddy)

- **Timeouts:** connect/read/write/idle explicitly set at every proxy hop; idle timeouts consistent up the chain (upstream idle < downstream idle, else 502s).
- Request/response body size limits, header size limits, slowloris protection.
- Rate limiting + `Retry-After` + correct 429/503 semantics.
- **TLS:** auto-renew monitored (Caddy is good, but alert on cert expiry independently), TLS 1.2+, HSTS, OCSP.
- Upstream health checks (active + passive), and understand ejection behaviour under partial failure.
- HTTP/2 or keep-alive to upstreams; connection reuse.
- Basic DDoS/L7 protection; Hetzner gives L3/L4 only.
- CORS, security headers, WAF if public.

---

## 7. Observability

- **RED per service** (Rate, Errors, Duration) + **USE per resource** (Utilization, Saturation, Errors). Saturation is the one everyone forgets and it's the leading indicator.
- **Percentiles, never averages.** p50/p95/p99/p99.9. Histograms, not summaries, if you need to aggregate.
- **Distributed tracing** with sampling (tail-based for errors) — OpenTelemetry.
- **Alert on symptoms** (SLO burn rate), not causes. Multi-window multi-burn-rate alerts. Every alert must be actionable and have a runbook link. Delete alerts nobody acts on.
- **Define SLIs/SLOs + error budget before launch.** "99.9%" = 43min/month.
- **Dashboards:** one "is it healthy" per service, one dependency map, one capacity view.
- Log volume/cost controls; no PII/secrets in logs (ISO 27001 + GDPR).

---

## 8. Release & change management

- **CI gates:** tests, lint, type-check, SAST, dependency audit, image scan, migration lint.
- **Deploy strategy:** blue/green or canary with automated rollback on SLO breach. Rolling with readiness gates at minimum.
- **Feature flags decoupled from deploy** — the fastest rollback is a flag, not a redeploy.
- **Every deploy must be rollback-able including the DB.** If not, it's a one-way door — treat it as such.
- Deploy markers on dashboards. Freeze windows. Change log.

---

## 9. Security & compliance

- **Secrets in a manager** (not env files in git), rotation policy, no secrets in images or logs.
- **AuthN/AuthZ tested at the object level** (IDOR is the most common real-world bug), rate-limited auth endpoints, token expiry/revocation.
- **Input validation at the boundary** (zod / pydantic), output encoding, parameterized queries.
- **Audit logging** for privileged actions (append-only, retained).
- Dependency and container CVE monitoring with an SLA to patch.
- Pen test findings triaged before launch; DPA/data residency; retention & deletion policy.

---

## 10. Resilience / DR

- Explicit RTO/RPO per system, signed off by the business.
- **Documented failure modes:** what happens if DB is down / Redis is down / a third-party API is slow (slow is worse than down)? Each must have a defined behaviour.
- **Chaos/failure injection:** kill a pod, add 500ms latency, blackhole a dependency, fill the disk. With no pre-production environment, do it on the server during business hours, with a tested restore ready — see [runbooks/db-backup-restore.md](runbooks/db-backup-restore.md).
- **GameDay before launch:** simulate an outage, time your response, fix the gaps in the runbook.
- On-call rota, escalation path, incident severity definitions, comms template, blameless postmortems.

---

## 11. Pre-prod testing you must actually run

- **Load test to failure** (k6/Locust) — you need the knee of the curve, not "it handled expected traffic". Record max sustainable RPS and what breaks first.
- **Soak test 12–24h** — finds memory/FD/connection leaks that a 5-min test never will.
- **Spike test** — 10× in 10s. Validates autoscaling, cold start, shedding.
- **Recovery test** — how does it behave after overload is removed? (metastability check).
- **Restore-from-backup drill**, with a stopwatch.

---

## 12. Reading list (highest leverage, in order)

1. **Google SRE Book + SRE Workbook** — SLOs, error budgets, overload chapters (20, 21, 22 especially).
2. **Release It! (Nygard)** — stability/capacity patterns; the single best fit for this topic.
3. **Designing Data-Intensive Applications (Kleppmann)** — data layer reasoning.
4. **Systems Performance (Gregg)** — USE method, Linux tooling.
5. **Papers:** "Metastable Failures in Distributed Systems", "Controlling Queue Delay" (CoDel), Netflix adaptive concurrency limits, AWS Builders' Library (timeouts/retries/jitter, load shedding, static stability) — the Builders' Library articles are short and the highest ROI of anything here.
6. **PostgreSQL 14 Internals (Rogov)** — free, excellent, and directly relevant since Postgres is where most production pain originates.
