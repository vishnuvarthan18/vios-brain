# Gateway — `arm-htz-srvr-vpn` (10.0.0.3, public 2.28.50.255)

The estate's single public entry point. Today it is the WireGuard endpoint, the
split-horizon DNS resolver (dnsmasq) and the NAT router for every private host.
This directory adds the fourth role: **L4 ingress for the application**, so the
app host can lose its public IP.

Design and phasing: [`../../../EDGE-GATEWAY-PLAN.md`](../../../EDGE-GATEWAY-PLAN.md)
§2, Phase 5–6.

---

## Why this box already carries the risk it looks like it is taking on

`iptables -t nat -S POSTROUTING` on this host:

```
-A POSTROUTING -s 10.0.0.0/16 -o eth0 -j MASQUERADE
-A POSTROUTING -s 10.8.0.0/24 -o enp7s0 -j MASQUERADE
```

Every private host's internet egress **already** masquerades out through this
box — that is how the db and git hosts reach the internet with no public IP of
their own. Adding ingress widens an existing single point of failure rather
than creating a new one. The mitigation is a floating IP (Phase 7), not
avoidance.

## Current state — as of 2026-09-07

| | |
|---|---|
| haproxy | **3.0.11-1+deb13u3 installed** from Debian 13 apt. No third-party repo, no build |
| service | **enabled, stopped.** Still running the distro default config, which binds nothing |
| `haproxy.cfg` here | **validated against the real binary** (`haproxy -c` → exit 0), **not yet installed** |
| ufw | `51820/udp`, `22`, `53` — no `80`, no `443` |
| Hetzner Cloud Firewall | `51820/udp` only |
| `FORWARD` policy | `ACCEPT` — EDGE-20, still open |

The config is deliberately **not** installed yet. Its backends
(`10.0.0.2:8443`, `10.0.0.2:8080`) do not exist, so installing it would only
buy a health-check failure every 5 seconds in a journal that now ships to
central Loki. The distro default is inert and a reboot starts something
harmless.

## The two decisions baked into `haproxy.cfg`

**Layer 4 only — TLS is never terminated here.** The TCP stream is forwarded
untouched and Caddy on the app host keeps the certificate. Terminating at the
gateway would hand this box the private key for the public site and put a
second ACME client in the path.

**The backend ports are `8443`/`8080`, not `443`/`80`.** Caddy's
`proxy_protocol` listener wrapper *requires* the header, so it cannot be
enabled on the port that also serves direct public traffic — every direct
connection would break the moment it was switched on, and direct traffic stays
live right through the cutover. The app host therefore grows a second pair of
listeners that require PROXY protocol and accept it only from `10.0.0.3`.
Public `:443`/`:80` on the app host keep working untouched, and go away with
the public IP rather than before it.

PROXY protocol is not cosmetic. In plain passthrough every request reaches
Caddy sourced from `10.0.0.3`, which collapses Kong's rate limiting into one
bucket for the whole internet, empties the client IP from the auth audit trail,
and makes the client-IP label in Loki a constant.

## Remaining steps, and their gates

1. App host: publish Caddy on `8443`/`8080` with the `proxy_protocol` listener
   wrapper scoped `allow 10.0.0.3/32`, and add `DOCKER-USER` rules restricting
   both to that source — same shape as the existing `9090` federation rule in
   `../appdeploy/arm-docker-fw.sh`
2. Install this config, start haproxy, confirm both backends report `UP`
3. `ufw allow 80/tcp && ufw allow 443/tcp` here, then open the same two in the
   Hetzner Cloud Firewall
4. `FORWARD` policy `ACCEPT` → `DROP` with explicit rules (EDGE-20). **Do this
   with console access available** — this box is the NAT router for the whole
   estate and a wrong rule takes every host's egress with it
5. Rehearse on `edge-test.arametrics.app`, already in the SNI allow-list. Check
   TLS handshake, all three Module Federation `remoteEntry.js`, **the real
   client IP in Caddy's logs**, and an SSE stream open past two minutes
6. DNS cutover: `dev.arametrics.app` and the apex → `2.28.50.255`
7. Only then Phase 6 — reroute the app host's default route through this box
   and remove its public IP

## Rollback

Nothing here is load-bearing until step 3 opens the firewall. After the DNS
cutover, rollback is repointing one A record back to `188.245.177.70`, which
stays reachable until Phase 6.

**Add a hostname to the `arm_sni` allow-list before pointing it at this edge**,
or the handshake is rejected before it reaches a backend.
