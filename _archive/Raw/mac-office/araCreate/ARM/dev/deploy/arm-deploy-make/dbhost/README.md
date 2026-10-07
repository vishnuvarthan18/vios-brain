# Database host (`arm-htz-srvr-db`) unmanaged state

Captured 2026-09-06. This host is the live production database tier and runs
PostgreSQL 16 and MongoDB 8 as **native packages** — no Docker. Several pieces
of its configuration existed only on the box.

| File | Purpose |
|---|---|
| `fix-firewall-ordering.sh` | One-shot: makes the two firewalls deterministic and the nftables ruleset safe standing alone |
| `alloy/config.alloy` | Installs to `/etc/alloy/config.alloy` (0644). Ships the journal **and both database log files** to the central Loki |

## Alloy is native here, and the database logs are FILES

Alloy is a systemd service (v1.19.2, deb package), not a container — the app and
log hosts run it in Docker, so their configs are not interchangeable. It runs as
`alloy`, which is in `adm` and `systemd-journal`, with
`--storage.path=/var/lib/alloy/data` set by the packaged unit.

**Neither engine writes to the journal.** `journalctl -u postgresql@16-main` and
`-u mongod` both return **zero** lines: Postgres has `logging_collector=off` but
Debian's `pg_ctlcluster` redirects to a file anyway, and mongod is configured
`destination: file`. So until the file sources were added on 2026-09-08 this host
shipped only OS noise — about 85 lines/hour of cron, ssh and systemd — and the
production database tier's own logs were **not searchable anywhere**.

| Engine | Path | Mode as shipped |
|---|---|---|
| Postgres | `/var/log/postgresql/postgresql-16-main.log` | `postgres:adm 0640` — already readable |
| Mongo | `/var/log/mongodb/mongod.log` | `mongodb:mongodb 0600` — **was not readable** |

Mongo's log needed `chgrp adm` + `chmod 0640`. Deliberately **not** done by
adding `alloy` to the `mongodb` group: that group owns `/var/lib/mongodb`, the
actual data directory, so it would grant far more than log read. `mongod` still
owns the file and writes to it normally at 0640.

Both files rotate under logrotate with **`copytruncate`**, which truncates the
existing inode instead of creating a new file. That is why the one-time `chgrp`
persists across rotations — but if `mongod.log` is ever *deleted* rather than
rotated, mongod recreates it at 0600 under its own umask and the `chgrp` must be
reapplied.

### mongod.log is ~40 MB/day and 94% of it is noise

Measured over one rotated 42 MB file, 65,165 lines sampled: **68.5% NETWORK,
25.6% ACCESS**. The dominant client is `mongodb_exporter` — 3,246 connections in
an 8 MB sample against **36** from the application's own Mongoose pool — so most
of the volume is monitoring reconnect churn, not information about the database.

`loki.process "mongo_denoise"` drops five NETWORK message types at the source
(`Connection accepted`, `Connection ended`, `Ingress TLS handshake complete`,
`No SSL certificate provided by peer`, `client metadata`). Measured effect on
first run: **485 lines dropped against 251 kept, ~66% reduction**, matching the
~64% predicted from the sample. ACCESS lines are kept deliberately — they are
auth events and worth having even at this volume.

Check it with the counter, not by eye:

```sh
curl -s http://localhost:12345/metrics | grep loki_process_dropped_lines_total
```

**Worth a separate look:** the exporter's connection churn is the underlying
cause of the volume. Roughly 17,600 connections/day where the application makes
a few dozen suggests it opens a fresh connection per scrape. Fixing that would
shrink this log at the source rather than filtering it — `mongodb_up` is 1, so
it is a churn problem, not an outage.

## This host has two firewalls

That is the single most surprising thing about it, and it cost hours to find.

`/etc/nftables.conf` installs `inet filter` via `nftables.service` — **and it
is the only host in the estate where that service is enabled.** ufw installs
`ip filter` / `ip6 filter`. A packet must be accepted by **both**, so a ufw
rule here is necessary but not sufficient.

The way this presents is deceptive: ufw's own ACCEPT counters increment on
every SYN, because ufw genuinely does accept them. The other table then drops
them. `tcpdump` showed SYNs arriving with no SYN-ACK, no `SYN_RECV` socket and
no kernel drop counter — the packet was accepted by one firewall and discarded
by the other.

## And `flush ruleset` makes them hostile to each other

`/etc/nftables.conf` opens with `flush ruleset`, which erases ufw's tables.
Nothing ordered the two services, so whichever started last won. The nftables
ruleset also accepted SSH from anywhere, with ufw the only thing narrowing it —
so an inverted boot order did not merely lose rules, it would have put port 22
on the live database host on the public internet.

`fix-firewall-ordering.sh` addresses both halves: `Before=ufw.service` so ufw
installs last at boot, and a source-scoped SSH rule so the nftables ruleset is
safe without ufw at all.

**Still true afterwards:** a manual `nft -f /etc/nftables.conf` flushes ufw's
tables and must be followed by `ufw reload`. The scoped SSH rule is what makes
forgetting that survivable rather than an exposure.
