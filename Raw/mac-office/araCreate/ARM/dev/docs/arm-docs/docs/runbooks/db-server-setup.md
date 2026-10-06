# Runbook — Database Server (S1) Setup

> How to stand up ARM's databases on a dedicated server, separate from the application server.
>
> This is the execution guide for **S1** in the multi-server plan. It assumes the topology from
> [03-NETWORK-AND-FIREWALL.md](../plan/infra/03-NETWORK-AND-FIREWALL.md), the compose split from
> [(secret removed)](../plan/infra/02-MULTI-SERVER-COMPOSE.md), and the credential rules from
> [05-DATABASE-SECURITY.md](../plan/infra/05-DATABASE-SECURITY.md). Read those for *why*; this document is *how*.

**Status of the plan itself:** every box in
[12-EXECUTION-CHECKLIST.md](../plan/infra/12-EXECUTION-CHECKLIST.md) is still unchecked as of 2026-08-25 —
nothing below has been run on real hardware yet. Sections 3, 4, and 11 describe files and fixes that **do not
exist in the repo yet** and that you have to create as part of this procedure. They are called out explicitly.

---

## 0. What lands on S1, and what does not

| Runs on S1 | Stays elsewhere |
|---|---|
| PostgreSQL (`arm_core`, `arm_admin`, `arm_authelia`) | **Redis → S2.** The session service hits Redis on every request; co-locating it with the apps keeps that at microseconds instead of 0.2–0.5 ms over the network |
| MongoDB (`arm-calendar`) | All 8 app containers, Caddy, Kong, Authelia → S2 |
| Kafka (KRaft, 1 or 3 brokers) | Prometheus, Grafana, Loki, Tempo, the DB exporters → S3 |
| A lightweight Alloy log forwarder (optional, ships DB logs to S3's Loki) | Backup cron + restore drills → S4 |

S1 has **no public IP**. Everything reaching it does so over the Hetzner private network `10.0.1.0/24`.

Target: `10.0.1.1`, Hetzner CX32 (16 GB / 4 vCPU / 80 GB NVMe).

---

## 1. Prerequisites

Before starting, S1 must already be:

- Provisioned on the `arm-private` network with private IP `10.0.1.1` and **no public IPv4**
- Hardened per [04-SERVER-HARDENING.md](../plan/infra/04-SERVER-HARDENING.md) — `deploy` user, SSH keys only,
  `PermitRootLogin no`, unattended-upgrades, 4 GB swap, fail2ban, chrony
- Running Docker with the daemon hardening from that same document

You reach it by hopping through S2 (S1 has no public IP):

```sh
ssh -J deploy@<S2-public-ip> deploy@10.0.1.1
```

Add that as a `ProxyJump` entry in `~/.ssh/config` now — every step below assumes you can get a shell on S1.

---

## 2. Put the deploy tooling on S1

S1 needs `arm-deploy-make` for its compose files, `.env`, and make targets. It does **not** need any
application repo — no `clone-repos`, no Node.js, no image builds. Only infra containers run here.

```sh
# From your laptop, in the workspace root
rsync -az --delete \
  --exclude=.env --exclude=backups --exclude=generated --exclude=node_modules \
  deploy/arm-deploy-make/ deploy@10.0.1.1:~/arm-db/
```

Keep it at `~/arm-db` rather than `~/arm-deploy` so it is obvious on sight which server you are on.

---

## 3. Create `docker-compose.infra.multi.yml`  ⚠️ does not exist yet

`docker-compose.infra.prod.yml` assumes Postgres, MongoDB, Kafka **and** Redis all share one Docker bridge
network on one host, and it publishes **no ports at all** — app containers reach the databases by Docker DNS.
Neither holds once the apps live on another machine. Copy it and make exactly three changes:

```sh
cp docker-compose.infra.prod.yml docker-compose.infra.multi.yml
```

**3.1 — Publish the ports on the private network.** Add a `ports:` block to each service:

```yaml
  postgres:
    ports:
      - "0.0.0.0:5432:5432"
  mongodb:
    ports:
      - "0.0.0.0:27017:27017"
  # and in generated/docker-compose.kafka.yml, per broker:
  kafka1:
    ports:
      - "0.0.0.0:9092:9092"
```

`0.0.0.0` is safe here **only because S1 has no public IP** — the bind can only ever be reached from the
private network, and UFW (§8) narrows it further to the three IPs that need it. If S1 ever gains a public
IP, these binds become internet-facing databases; change them to `10.0.1.1:5432:5432` before that happens.

**3.2 — Delete the `redis` service block** and its `arm_redis_prod_data` volume entry. Redis moves to S2.

**3.3 — Leave everything else alone.** Health checks, `ulimits`, `deploy.resources.limits`, log rotation,
named volumes, and the `${TUNE_*}` variables all carry over unchanged. The volume names in particular
(`arm_postgres_prod_data`, `arm_mongodb_prod_data`, `arm_kafka1_prod_data`) must stay identical, or the
restore in §7 lands in a fresh volume and the data appears to vanish.

---

## 4. Fix Kafka's advertised listener  ⚠️ hard blocker

This one silently breaks cross-server Kafka and is worth understanding before you hit it.

`make tune-infra` generates `generated/docker-compose.kafka.yml` with the advertised address **hardcoded**
to the Docker hostname — `make/tune.mk:132` (single broker) and `:177` (3-broker loop):

```
KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://arm-kafka1-prod:9092
```

A Kafka client does not keep talking to the address it bootstrapped against. It connects to
`10.0.1.1:9092`, receives the cluster's *advertised* address in the metadata response, and reconnects to
**that**. On S2 there is no `arm-kafka1-prod` in DNS, so core-be and the notification service will bootstrap
successfully and then fail on every produce and consume — with a resolution error that points at Kafka
rather than at the config.

Make the advertised host a variable in `make/tune.mk` before generating:

```
KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://${KAFKA_ADVERTISED_HOST:-arm-kafka1-prod}:9092
```

Then set `KAFKA_ADVERTISED_HOST=10.0.1.1` in S1's `.env`. The default keeps single-server and local dev
working exactly as they do today.

`KAFKA_CONTROLLER_QUORUM_VOTERS` also names `arm-kafka1-prod:9093`, but the controller quorum is
broker-to-broker traffic *inside* S1's Docker network, so it can stay on Docker DNS. Only the client-facing
listener needs the private IP.

---

## 5. Write S1's `.env`

S1 needs a much smaller `.env` than S2 — only what the infra containers themselves read. `make env-check`
will complain about app variables that are legitimately absent here, so treat its output as advisory on this
box and check these by hand instead:

| Variable | Value on S1 | Read by |
|---|---|---|
| `POSTGRES_USER` | same as S2 (`admin` by default) | Postgres container, health check, `_ensure-*` targets |
| `POSTGRES_PASSWORD` | 32+ chars, **identical to S2's** | Postgres container |
| `POSTGRES_DB` | `arm_core` | Postgres container |
| `MONGO_ROOT_USER` | same as S2 (`arm_admin` by default) | MongoDB container, health check |
| `MONGO_ROOT_PASSWORD` | **identical to the password embedded in S2's `MONGODB_URI`** | MongoDB container |
| `POSTGRES_MONITORING_PASSWORD` | 16+ chars, same as S3's exporter config | `_ensure-monitoring-roles-prod` |
| `MONGO_MONITORING_PASSWORD` | 16+ chars, same as S3's exporter config | `_ensure-monitoring-roles-prod` |
| `KAFKA_ADVERTISED_HOST` | `10.0.1.1` | the fix from §4 |
| `TUNE_*` | written by `make tune-infra` in §6 — do not hand-edit | Postgres/Mongo/Kafka commands |

Then lock it down: `chmod 600 .env`.

**The Mongo password trap applies across servers now.** `MONGODB_URI` lives in S2's `.env` with the password
embedded in the connection string, while `MONGO_ROOT_PASSWORD` lives in S1's. Changing one without the other
gives calendar-be an opaque auth failure, and the two files are no longer next to each other to compare.

---

## 6. Tune for the box

`make tune-infra` sizes Postgres shared buffers, the WiredTiger cache, and the Kafka broker count from the
host's RAM. Run it **on S1**, so it reads S1's 16 GB rather than a laptop's:

```sh
cd ~/arm-db && make tune-infra
```

It writes `TUNE_*` into `.env` and regenerates `generated/docker-compose.kafka.yml`. Re-run §4's edit if
`tune-infra` overwrote the advertised listener — that generated file carries a `do not hand-edit` header for
exactly this reason, which is why the fix belongs in `make/tune.mk` and not in the output.

Two values to override deliberately while you are here — both are
[01-PRE-SPLIT-FIXES.md](../plan/infra/01-PRE-SPLIT-FIXES.md) items that are **still unapplied in the repo**
(verified 2026-08-25):

```sh
TUNE_PG_MAX_CONNECTIONS=250        # default is 200 — 4 services x 50 pool = zero headroom for pg_dump or a migration
TUNE_REDIS_MAXMEMORY_POLICY=volatile-lru   # set on S2, not here — noeviction fails every write at the 768 MB cap
```

The Redis one is S2's problem, but fix it in the same pass so it does not get lost.

---

## 7. Start the databases and restore the data

```sh
cd ~/arm-db
docker network create arm-infra-network                          # the Kafka include expects it to exist
docker compose -f docker-compose.infra.multi.yml up -d
```

Wait for health, then let the make targets finish provisioning:

```sh
make infra-wait-prod
```

That target does more than poll health checks — worth knowing, because it means several
[12-EXECUTION-CHECKLIST.md](../plan/infra/12-EXECUTION-CHECKLIST.md) lines are already automated:

- `_ensure-postgres-databases-prod` creates `arm_admin` and `arm_authelia` if absent (`arm_core` comes from
  `POSTGRES_DB`)
- `_ensure-monitoring-roles-prod` creates the Postgres `arm_monitoring` role with `pg_monitor`, and the
  MongoDB `arm_monitoring` user with `clusterMonitor` — both skip with a warning if the matching
  `*_MONITORING_PASSWORD` is unset, which is how you get exporters that connect on S3 but return nothing
- `_wait-kafka-init-prod` creates the platform's topics

Both `_ensure-*` targets shell out via `docker exec` against `arm-postgres-prod` / `arm-mongodb-prod`, so
they only work **on S1**. They are not runnable from S2 or your laptop.

Now restore from the final pre-migration backup:

```sh
# Copy the dumps up from wherever `make backup-all` last wrote them
scp -J deploy@<S2-public-ip> backups/postgres/arm_core-*.sql.gz \
    backups/postgres/arm_admin-*.sql.gz \
    backups/mongodb/arm-calendar-*.archive deploy@10.0.1.1:~/arm-db/backups/

# Postgres — one database at a time
gunzip -c backups/postgres/arm_core-<ts>.sql.gz | \
  docker exec -i arm-postgres-prod psql -U admin -d arm_core
gunzip -c backups/postgres/arm_admin-<ts>.sql.gz | \
  docker exec -i arm-postgres-prod psql -U admin -d arm_admin

# MongoDB
docker cp backups/mongodb/arm-calendar-<ts>.archive arm-mongodb-prod:/tmp/restore.archive
docker exec arm-mongodb-prod mongorestore \
  -u arm_admin -p "$MONGO_ROOT_PASSWORD" --authenticationDatabase admin \
  --archive=/tmp/restore.archive --gzip
```

See [db-backup-restore.md](db-backup-restore.md) for the backup side and the two drill targets that prove a
dump is actually restorable.

---

## 8. Firewall

Host UFW, matching [03-NETWORK-AND-FIREWALL.md](../plan/infra/03-NETWORK-AND-FIREWALL.md) §3.2. Apply the
Hetzner Cloud Firewall with the same rules afterwards — two layers, hypervisor and host.

```sh
ufw default deny incoming
ufw default allow outgoing
ufw allow from 10.0.1.0/24 to any port 22      # SSH from the private network only
ufw allow from 10.0.1.2 to any port 5432       # S2 apps  → Postgres
ufw allow from 10.0.1.2 to any port 27017      # S2 apps  → MongoDB
ufw allow from 10.0.1.2 to any port 9092       # S2 apps  → Kafka
ufw allow from 10.0.1.3 to any port 5432       # S3 postgres-exporter
ufw allow from 10.0.1.3 to any port 27017      # S3 mongodb-exporter
ufw allow from 10.0.1.4 to any port 5432       # S4 backup dumps
ufw allow from 10.0.1.4 to any port 27017      # S4 backup dumps
ufw enable
```

**Docker publishes ports by writing iptables rules that bypass UFW's `INPUT` chain.** A `ports:` mapping is
reachable even under `ufw default deny incoming`, so the rules above are not what is actually protecting
these databases — S1 having no public IP is. Setting `"iptables": false` on the Docker daemon is not the
answer either; it breaks container networking outright. Treat the **Hetzner Cloud Firewall as the enforcing
layer** and UFW as defence-in-depth, and confirm it empirically: from a machine outside `10.0.1.0/24`, check
that 5432 is unreachable rather than assuming the UFW rule closed it.

---

## 9. Verify

On S1:

```sh
docker exec arm-postgres-prod pg_isready -U admin                      # accepting connections
docker exec arm-postgres-prod psql -U admin -c "SHOW max_connections;" # expect 250
docker exec arm-postgres-prod psql -U admin -l                         # arm_core, arm_admin, arm_authelia
docker exec arm-postgres-prod psql -U admin -c "\du arm_monitoring"    # role exists, has pg_monitor
docker exec arm-mongodb-prod mongosh -u arm_admin -p "$MONGO_ROOT_PASSWORD" \
  --authenticationDatabase admin --eval "db.adminCommand('ping')"
docker exec arm-kafka1-prod /opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 --list                             # platform topics present
```

From S2 — this is the half that catches the cross-server mistakes:

```sh
psql -h 10.0.1.1 -U admin -d arm_core -c "SELECT 1;"
mongosh "mongodb://(secret removed)@10.0.1.1:27017/arm-calendar?authSource=admin" --eval "db.stats()"

# Kafka: prove the ADVERTISED listener resolves, not just the bootstrap address.
# A bare --list can succeed against the bootstrap connection while every
# produce still fails, so exercise a real produce/consume round trip.
docker run --rm apache/kafka:3.8.1 /opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server 10.0.1.1:9092 --list
```

If §4 was skipped, the `--list` above may well pass and the platform still breaks. That is the failure mode
to watch for.

---

## 10. Point S2 at S1

No application code changes. Every generated prod compose file already reads its DB host through
`${VAR:-default}` — verified in `generated/prod/`: `DB_HOST: ${DB_HOST:-arm-postgres-prod}`,
`KAFKA_BROKERS: ${KAFKA_BROKERS:-arm-kafka1-prod:9092,...}`, `ARM_CORE_DB_HOST`, `ARM_ADMIN_DB_HOST`,
`MONGODB_URI`. Setting them in S2's `.env` is the whole change:

```sh
DB_HOST=10.0.1.1
DB_PORT=5432
ARM_CORE_DB_HOST=10.0.1.1
ARM_ADMIN_DB_HOST=10.0.1.1
MONGO_HOST=10.0.1.1
MONGO_PORT=27017
MONGODB_URI=mongodb://(secret removed)@10.0.1.1:27017/arm-calendar?authSource=admin
KAFKA_BROKERS=10.0.1.1:9092
REDIS_HOST=arm-redis-prod        # unchanged — Redis is local to S2
```

Then restart the app services on S2. Note the GitHub Actions deploy writes S2's `.env` from `PROD_*`
secrets, so these values have to be updated **there** as well or the next deploy reverts them to Docker DNS
names.

---

## 11. Known gaps in the tooling

Found by reading the repo, not by running the migration. Each one will bite during this procedure:

| Gap | Where | Consequence |
|---|---|---|
| `docker-compose.infra.multi.yml` does not exist | spec'd in [02](../plan/infra/02-MULTI-SERVER-COMPOSE.md) §2.1 only | §3 is authoring work, not a checkout |
| Kafka advertised listener hardcoded to `arm-kafka1-prod` | `make/tune.mk:132`, `:177` | Cross-server Kafka fails after a successful bootstrap — §4 |
| `TUNE_PG_MAX_CONNECTIONS` still defaults to `200` | `docker-compose.infra.prod.yml:55` | No headroom for `pg_dump`, migrations, or a manual `psql` — §6 |
| `TUNE_REDIS_MAXMEMORY_POLICY` still defaults to `noeviction`; hardcoded in `docker-compose.infra.redis-sentinel.yml:23,57,89` | same | On S2: every Redis write fails at the 768 MB cap — sessions break, BullMQ stops |
| `scripts/backup-postgres.sh` and `backup-mongodb.sh` hardcode `docker exec arm-postgres-prod` / `arm-mongodb-prod` | both scripts | `make backup-all` can only run **on S1**, not from S4 as the plan's "daily 02:00 from S1" implies. Either run the cron on S1 and sync to S4, or rewrite the scripts to take `-h <host>` |
| `make/multi-server.mk` and its four targets do not exist | spec'd in [02](../plan/infra/02-MULTI-SERVER-COMPOSE.md) §2.6 | Every step here is a raw `docker compose` invocation until they are written |
| `make env-check` has no per-server profile | `make/config.mk` `SERVER_REQUIRED_VARS` | It reports app variables as missing on a DB-only box; §5's table is the manual substitute |

---

## 12. Rollback

Nothing is destroyed until the old single server is decommissioned, which
[12-EXECUTION-CHECKLIST.md](../plan/infra/12-EXECUTION-CHECKLIST.md) deliberately holds for a week after
cutover. To back out:

1. Revert S2's `.env` DB overrides to the Docker DNS defaults (or delete the lines — the `${VAR:-default}`
   fallbacks restore single-server behaviour on their own)
2. Restart the app services on S2 — or point DNS back at the old server if S2 is also new
3. Leave S1 running and untouched, so a second attempt does not need another restore

**`make rollback` will not help here.** It swaps image tags only and says so; it has no notion of where a
database lives. Any migration that ran against S1 during the attempt stays applied — take a backup before
cutover, and gate any migration-carrying deploy on `make backup-all` plus `make restore-postgres-drill`.
