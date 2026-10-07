# GUARD

Repo-side configuration for the estate's gateway — `arm-htz-srvr-vpn`, private `10.0.0.3`,
public `2.28.50.255`. Captured from the running box on **2026-09-04**.

This box is the estate's only public IP for operators, its NAT egress, its split-horizon
resolver, and — once HAProxy lands — its single public front door. Two of the five hosts have
already dropped their public IPv4, so it is **already the only route to them**. That is why its
config belongs in version control and why a timed rebuild procedure exists: see
[rebuild.md](rebuild.md).

## What is here

| Path | Purpose | Applied? |
|---|---|---|
| [dnsmasq/arm-internal.conf](dnsmasq/arm-internal.conf) | Split-horizon DNS. Answers estate names with private addresses so a split-tunnel client reaches them at all | **Live** — matches `/etc/dnsmasq.d/arm-internal.conf` |
| [ufw/before.rules.nat](ufw/before.rules.nat) | The `*nat` fragment: estate egress **and** the peer-to-private rule relocated out of `wg0.conf` | **Partly** — the estate rule is live; the relocation is not |
| [ufw/rules.sh](ufw/rules.sh) | The gateway's ufw allow set, idempotent | **Live** — matches the four active rules |
| [haproxy/haproxy.cfg](haproxy/haproxy.cfg) | The L4 front door: SNI routing on `:443`, ACME on `:80`, PROXY v2 to backends | **No** — `haproxy` is not installed. Valid on 3.0.11, the version Debian 13 offers |
| [haproxy/test-sni-routing.sh](haproxy/test-sni-routing.sh) | Proves the routing behaviour on a throwaway Docker network. `make test-gateway` | Passing, off-estate |
| [guard-setup.sh](guard-setup.sh) | Provisioner: installs and wires everything this directory owns | Not run against the live box — written from what is already there |

## What is deliberately NOT here

**No keys, and no peer list.** `/etc/wireguard/wg0.conf` holds the server private key and every
peer's public key. Neither belongs in a git repository, and the key is not recoverable if lost —
every peer config would need reissuing. A copy is held off-host outside every git working
tree, pending transfer into the password manager, which is where it belongs.

`guard-setup.sh` therefore does **not** generate or write WireGuard config. That is
[`scripts/vpn-wireguard-setup.sh`](../scripts/vpn-wireguard-setup.sh)'s job and it already does
it well — `setup`, `add-peer`, `status`. The two are complementary, and the split is on purpose:
that script owns WireGuard and the things WireGuard needs; this directory owns everything else
the box does. Nothing in it touches `wg0.conf`.

**No running HAProxy.** The config now exists — [haproxy/haproxy.cfg](haproxy/haproxy.cfg) —
but the package is not installed and `:80`/`:443` are still closed in ufw and unused. What the
config does is settled and recorded in `EDGE-GATEWAY-PLAN.md` §0: SNI frontend on `:443` in
`mode tcp`, a single default backend on `:80` for the app host's HTTP-01 (the only remaining
consumer), PROXY protocol v2 to every backend, `check inter 3s fall 3 rise 2`, stick-table
connection-rate limiting.

**No production backend is attached, and that is not an oversight.** Every backend line carries
`send-proxy-v2`, and a receiver that is not expecting a PROXY header treats it as malformed
protocol and drops the connection. `Caddyfile.prod` declares neither `proxy_protocol` nor
`trusted_proxies` (checked 2026-09-04), so attaching the app host today would take the
application down on the first request. The `app` backend is written out at the foot of the config,
commented, beside the two preconditions for uncommenting it.

**What has and has not been proven.** `make test-gateway` stands up two throwaway backends with
distinct certificates on a private Docker network and asserts the four properties that
`haproxy -c` cannot: a hostname reaches the backend it is meant to, an unknown hostname reaches
none, plaintext on `:443` is refused, and the backend learns the real client address rather than
the gateway's. It passes, and it has been mutation-tested — adding a `default_backend` or
dropping `send-proxy-v2` both make it fail. That proves the *pattern*. It does not prove this
box — the throwaway hostname end-to-end on the real gateway, including a forced certificate
renewal, is still outstanding.

The rehearsal backend points at `127.0.0.1:8443` on the gateway itself rather than at another
estate host. `10.0.0.4:443` is the obvious-looking target and the wrong one — it is the log
server's real Caddy, which does not speak PROXY protocol either, so a `send-proxy-v2` backend
aimed there would break Grafana instead of rehearsing anything.

## The state of the box as captured

| | |
|---|---|
| OS | Debian 13 (trixie) |
| Sizing | 2 cores, 3826 MB RAM, 38 GB disk, load 0.16 — **no resize needed** |
| WireGuard | 1.0.20210914-3, `wg0` on `10.8.0.1/24`, `:51820`, one peer |
| dnsmasq | 2.91, listening `10.8.0.1` and `127.0.0.1` only |
| ufw | 0.36.2-9, active |
| HAProxy | not installed |
| Enabled units | `wg-quick@wg0`, `dnsmasq`, `ufw` |
| Listening | `:22` (sshd), `10.8.0.1:53` and `127.0.0.1:53` (dnsmasq). `:80` and `:443` free |

## Two things about this box that are easy to get wrong

**It filters nothing crossing it.** `-P FORWARD ACCEPT`, verified 2026-09-04. Every packet
between any peer and any estate host passes unexamined. Tightening it is the
highest-blast-radius change on the estate: get it wrong and you sever operator access and estate
egress simultaneously. Do it with a deadman timer and a second session already open — Appendix C
Trap 3 of `EDGE-GATEWAY-PLAN.md` has the pattern.

**Its NAT makes every peer look like an estate host.** `-A POSTROUTING -s 10.8.0.0/24 -o enp7s0
-j MASQUERADE` means a peer arrives at every backend as `10.0.0.3`. So a firewall rule anywhere
on the estate written `from 10.0.0.0/16` grants access to **everyone holding a peer config**, not
to the five hosts. This is not hypothetical: it is how the log server's Loki port turned out to be
readable and writable by any peer. Per-peer FORWARD scoping is the fix. Until then, name
specific host addresses in rules and never the private network.
