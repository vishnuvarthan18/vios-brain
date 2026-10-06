# LOGSERVER

Repo-side copy of the central log server's stack — Grafana + Loki + Alloy + Prometheus + node-exporter behind Caddy. The **live copy is `/opt/arm-logs` on `10.0.0.4`** (Hetzner, public `49.13.169.44`), and until 2026-09-02 it existed *only* there: unmanaged host state, the same class of problem as an unmanaged credentials file. Nothing here is deployed by anything.

**The two copies can drift and nothing detects it.** This is a mirror for review and for rebuilding the box by hand, not an automated deploy. Changes are still made on the host and copied back here; a change made only here reaches nothing. Layout matches the host exactly so `scp -r` in either direction is the whole "deploy".

`.env` is **not** mirrored — see `.env.example` for the key names. Neither are the host's `*.bak-prealerting` files (pre-2026-09-02 snapshots, kept on the box only).

## What runs where

Only Grafana has a public path, via Caddy at `grafana.arametrics.app`, gated by Grafana's own login. Loki is published on this host's private interface only (`LOG_BIND_IP`, `10.0.0.4`), so remote hosts push over `10.0.0.0/16` and nothing reaches it from the internet. Prometheus, node-exporter, Alloy and blackbox are on the `logs` Docker network with no host publication at all.

`LOG_BIND_IP` deliberately has no default: Compose refuses to start rather than falling back to `0.0.0.0`, which on a box with a public IP would serve every log line the estate ships to the internet, silently. A hand-rebuild with no `.env` therefore takes the log server down — the better of the two failures.

Caddy is built locally (`caddy-build/Dockerfile`) because the stock image cannot do ACME (secret removed), which this host now requires: `grafana.arametrics.app` has no public A record any more, so Let's Encrypt cannot reach port 80 and HTTP-01 is impossible. That needs `CF_API_TOKEN` — without it renewal fails silently until the certificate expires.

## Loki has no authentication, and its firewall rule does not work

`loki/loki-config.yml` sets `auth_enabled: false`. That is not a tenancy setting with a security default — it means Loki authenticates nobody, on read *and* write. Anything that can reach `10.0.0.4:3100` can query every production log line and push fabricated ones.

Two separate reasons the intended restriction is not in force. Verified 2026-09-04.

**1. The ufw rule cannot see this traffic.** Loki's port is published by `docker-proxy`, and Docker DNATs published ports in `nat/PREROUTING`, which is traversed *before* `filter/INPUT` where ufw's rules live. The traffic then crosses `FORWARD` via `DOCKER-USER` — a chain that is empty on this host. So the `3100/tcp ALLOW IN` rule in `ufw status` has never restricted anything, whatever source it names. This is the same trap that makes `scripts/db-firewall.sh` the wrong tool on the native database host.

**2. `from 10.0.0.0/16` means "from any VPN peer", estate-wide.** The gateway carries `-A POSTROUTING -s 10.8.0.0/24 -o enp7s0 -j MASQUERADE`, so every tunnel peer enters the private network wearing the gateway's own address, `10.0.0.3`. A peer is therefore indistinguishable from an estate host at every host's firewall. Confirmed by reaching this port from a laptop at `10.8.0.2`.

**This generalises, and it matters for work not yet done.** Any rule written as "allow from `10.0.0.0/16`" grants access to everyone holding a peer config, not to the five hosts. The pending task to open Postgres and MongoDB "to ARM prod only" must name `10.0.0.2` specifically — written as the private network it would expose both databases to every VPN peer, and if those ports are Docker-published it would need a `DOCKER-USER` rule to have any effect at all.

The fix here is a `DOCKER-USER` rule pinned to the ingress interface, which is how the app host restricts its own published Prometheus port:

```sh
# On the log server. Bridge traffic is untouched: Grafana reaches Loki at its
# container address on 172.18.0.0/16, not via 10.0.0.4, so an interface-scoped
# rule cannot break the datasource.
iptables -I DOCKER-USER -i enp7s0 -p tcp --dport 3100 ! -s 10.0.0.2 -j DROP
iptables -I DOCKER-USER -i eth0   -p tcp --dport 3100 -j DROP
```

`10.0.0.2` is the only legitimate client: the app host's Alloy is the sole `loki.write "central"` in the platform, and this box's own Alloy pushes to `loki:3100` over the Docker network. Adding a second shipping host means adding a second rule — deliberately explicit.

**iptables rules do not survive a reboot.** Persist them (`iptables-persistent`, or a systemd unit) or the restriction disappears at the next restart, silently.

## The Prometheus datasource uid must stay `prometheus`

`grafana/provisioning/datasources/datasources.yml` sets `uid: prometheus`. All 8 ARM alert rules reference `datasourceUid: prometheus` verbatim. Renaming the uid **silently breaks every one of them** — Grafana does not error on an unresolvable datasource uid in a provisioned rule, the rule simply stops evaluating. If the datasource is ever recreated, recreate it with the same uid.

## rules-arm.yaml is generated, not mirrored

The host carries `grafana/provisioning/alerting/rules-arm.yaml`. It is **not** mirrored here, because it is derived from a file this repo already owns and two copies would be two sources of truth:

```
observability/grafana/provisioning/alerting/rules.yaml   (this repo, authoritative)
  + a header recording the state and why
  + noDataState / execErrState inserted per rule
  = /opt/arm-logs/grafana/provisioning/alerting/rules-arm.yaml
```

`make estate-alert-rules` performs that transformation into `generated/rules-arm.yaml`; copy the result to the host. Edit `observability/.../rules.yaml` and re-render — never hand-edit either copy.

The two state fields are **variables, not edits**: `ARM_RULE_NODATA_STATE` and `ARM_RULE_EXECERR_STATE`. Rendering the silent combination requires `CONFIRM_SILENT_RULES=yes`, because it cannot be an inherited default — see below.

### All 8 ARM rules are live

Verified 2026-09-04 against the host: all 8 carry `noDataState: NoData` and `execErrState: Alerting`, and `up{host="arm-prod"}` returns real series through the `arm-prod-federate` job. An unreachable ARM exporter now fires.

They were silent (`OK` / `OK`) until 2026-09-03, and the reason is worth keeping because it is the trap to avoid on the way back: every rule queries `datasourceUid: prometheus`, so provisioning them while this box could not reach ARM prod would have sent eight "no data" emails immediately. `OK` kept them quiet — at the cost of a genuinely dead exporter also reading OK.

**The two halves belong together.** These states and the `arm-prod-federate` scrape job in `prometheus/prometheus.yml` must move in the same change. Live rules with no federation job turn every ARM rule into a permanent alert; a federation job with silent rules means the box scrapes ARM and then declines to tell you when it breaks. That coupling is why the render target refuses `noDataState=OK` without an explicit confirmation.

`rules-logserver.yaml` and `rules-certs.yaml` (both mirrored here) query metrics this box produces itself and have always been live.

## Prometheus carries both a time cap and a size cap

`docker-compose.yml` passes `--storage.tsdb.retention.time=15d` **and** `--storage.tsdb.retention.size=5GB`. The size cap is the point: Loki on the same box has only a time-based cap (`retention_period: 720h`) and nothing watches the disk, so a growth surprise in Loki has no ceiling other than the filesystem. Prometheus is capped on both axes so it cannot be the process that fills the volume the log store shares. Do not drop the size cap when tuning retention.

## Alert notification works — and how it broke

Delivering as of 2026-09-04: no `535` in the last seven days of `logs-grafana`, sending as `alerts@aracreate.group`.

It used to fail with:

```
535 5.7.8 Username and Password not accepted
```

which is what Gmail returns for an *account* password on a (secret removed) account. The fix was a Google App Password in `SMTP_PASS`. Two things about that are worth keeping:

- **App passwords get revoked.** This one was, silently, and the only symptom was alerts going nowhere — the container stayed healthy and Grafana logged the 535 without raising anything. Nothing watches for it; a `535` in `logs-grafana` is the signal.
- **Recreate, don't restart.** `docker restart` keeps the container's original environment, so a new `SMTP_PASS` in `.env` has no effect until `docker compose up -d grafana` replaces the container.

## Labels are written at ingest

`alloy/config.alloy` stamps constant `host` and `env` labels on every stream. Loki writes labels into chunks at ingest and they **cannot** be added retroactively — a stream pushed without `host` stays unattributable forever. Any new host pushing here must set them before it starts pushing, not after.
