# Application host (`arm-master`) unmanaged state

Captured into the repo 2026-09-06. These files live at fixed paths on the
application host and were **not** version-controlled before — the same gap
`logserver/` closed for the log server. A host-only firewall script is one
`rm` or one rebuild away from being lost, and nothing would report it missing.

| File | Installs to |
|---|---|
| `arm-docker-fw.sh` | `/usr/local/sbin/arm-docker-fw.sh` (0755) |
| `private-egress-route.sh` | `/usr/local/sbin/private-egress-route.sh` (0755) — writes `/etc/netplan/99-private-egress.yaml` (0600) |

`arm-docker-fw.service` already exists on the host and runs the script after
`docker.service`.

## This host has no public IPv4, and that breaks everything outbound

Its public v4 was detached at the Hetzner cloud level once `2.28.50.255`
became the estate's sole public entry point. Only IPv6 remains on `eth0`, so
outbound IPv4 has to leave via the NAT masquerade on the gateway
(`10.0.0.3`) — which needs one local route that nothing restores:

```text
default via 10.0.0.1 dev enp7s0 metric 2000
```

`private-egress-route.sh apply` installs it and persists it. Run
`private-egress-route.sh verify` after any reboot.

**The failure is silent, which is the whole problem.** Without the route the
host and every container lose IPv4 egress at once — Calendar sync, Google
OAuth login, and outbound mail — while the site keeps serving normally,
because inbound traffic arrives from the gateway over the *private* network,
the databases are on that network too, and `docker pull` still works over
IPv6. On 2026-09-07 sync was dead for ~30 minutes after a reboot with every
dashboard green.

It also does not look like a network fault in the logs. Node reports an
`AggregateError` with an **empty** reason — the line ends at `failed, reason:`
with nothing after it — because Happy Eyeballs got `ENETUNREACH` on v6 (Docker bridges are
v4-only) and `ETIMEDOUT` on v4 (no route), leaving no single message to print.
It reads like neither an auth nor a quota error.

The route must live in a `99-` netplan file: cloud-init rewrites
`50-cloud-init.yaml` from Hetzner metadata on every boot, and that metadata is
now IPv6-only. That is exactly how this broke.

**Do not set Docker's MTU.** The bridges are 1500 and this path is 1450, which
looks like it wants `"mtu": 1450` in `daemon.json`. Path-MTU discovery already
handles it — a 378 KB TLS fetch from inside `arm-calendar-backend` completed
in 326 ms.

## Why this script needs two different mechanisms

Docker publishes IPv4 ports by DNAT, so those packets traverse **FORWARD** and
are filtered in `DOCKER-USER` — and because DNAT has already rewritten the
port by then, the rules must match `--ctorigdstport`, not `--dport`.

There is no IPv6 DNAT on this host, so IPv6 publishing falls to userland
`docker-proxy`: the connection terminates on the host and is filtered in
**INPUT**, where the port has *not* been rewritten and `--dport` is correct.

The consequence is the reason this file exists: for months the `ip6tables`
`DOCKER-USER` rules read as IPv6 protection while sitting in a chain those
packets never enter. All ten had zero counters. The session service and all
three frontends were reachable on the host's global IPv6 address, held back
only by a Hetzner cloud firewall that no host-side check can see.

**Check counters, not rule lists.** `arm-docker-fw.sh status` prints both
chains and labels which one actually filters IPv6.
