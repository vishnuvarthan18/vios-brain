# Runbook — rebuild the gateway from bare metal

**Purpose.** Recreate `arm-htz-srvr-vpn` from a fresh Hetzner box, and **record how long it
takes**. That number is the estate's real RTO for a total gateway loss, and nobody knows it yet.

**Why it matters more than it used to.** Two of the five hosts have dropped their public IPv4.
The log server and Forgejo are reachable *only* through this box, and both resolve only through
its dnsmasq. A gateway loss today is not a degradation — it is a total loss of access to two
hosts and of NAT egress for three.

---

## Before you start

**You cannot rebuild without the WireGuard server key.** It is not recoverable. Without it every
peer config must be reissued, including your own — which you need in order to reach the estate at
all. Backing the key and peer list up to the password manager is **still outstanding**, and
this procedure is untestable until it is done. Do that first.

Have ready:

| | |
|---|---|
| `/etc/wireguard/server.key` and the peer list | Password manager. Not in this repo, ever |
| Hetzner console access | The rebuild's only out-of-band path |
| This repo, on the new box | `git clone`, or `scp` the `guard/` and `scripts/` directories |
| A second terminal | Non-negotiable for the ufw step |
| A stopwatch | The output of this procedure is a number |

Record the start time. The clock runs from "fresh box, SSH as root" to "estate reachable through
the tunnel".

---

## Step 1 — Base box

Debian 13 (trixie), same region as the estate (`nbg1`), attached to the **`arm-private`** Cloud
Network at provisioning time. Attaching afterwards works but adds a reboot.

Verify before continuing — everything downstream depends on both NICs existing:

```sh
ip -brief addr show | grep -vE '^(lo|wg0)'
# expect a public NIC (eth0) and a private one on 10.x (enp7s0)
```

If the private NIC is missing, stop. `guard-setup.sh` will refuse anyway, which is deliberate.

## Step 2 — WireGuard, from the existing script

```sh
./scripts/vpn-wireguard-setup.sh setup
```

Then restore the server key and peer list from the password manager, replacing the freshly
generated ones, and restart:

```sh
systemctl restart wg-quick@wg0
wg show          # expect the interface up, peers listed
```

Restoring the key rather than keeping the generated one is what saves every existing peer config.
If you skip it, every operator needs a new config before they can reach anything.

## Step 3 — Everything else the box owns

```sh
./guard/guard-setup.sh apply
```

Installs dnsmasq and ufw, persists `ip_forward`, writes the split-horizon config, splices the
`*nat` fragment into `before.rules`, and applies the ufw allow set. It does **not** enable ufw.

It will warn if `wg0.conf` still carries a MASQUERADE rule in `PostUp` — on a fresh setup it
will, because `vpn-wireguard-setup.sh` puts it there. Remove those two lines now that
`before.rules` owns the rule, then restart WireGuard and confirm exactly two nat rules:

```sh
systemctl restart wg-quick@wg0
iptables -t nat -S POSTROUTING | grep MASQUERADE
# expect exactly two: -s 10.0.0.0/16 -o eth0, and -s 10.8.0.0/24 -o enp7s0
```

Two is correct. One means the relocation did not take and `ufw reload` will break the tunnel's
onward path. Three means both places are adding it.

## Step 3b — HAProxy *(skip on a like-for-like rebuild)*

Only if the box is being rebuilt as a traffic gateway. A rebuild that restores today's state
should skip this: `haproxy` is not installed on the live box, and installing it here would make
the rebuilt box differ from the one it replaces.

```sh
apt-get install -y haproxy          # Debian 13 offers 3.0.11
install -m 644 guard/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg

# Validate BEFORE starting. A bad config leaves the unit in a failed state,
# and on a box that is the only route to three hosts that is not the moment
# to be reading parse errors.
haproxy -c -f /etc/haproxy/haproxy.cfg
```

The config's only backend is a scratch listener on `127.0.0.1:8443`/`:8080`, so bring that up
first or every check reports DOWN and both frontends close each connection. It must speak PROXY
protocol, because every backend line sends `send-proxy-v2`:

```sh
# Any listener that accepts PROXY v2 will do. The routing properties are
# already covered off-estate by `make test-gateway`; what this proves is
# THIS box — its NICs, its ufw, its systemd.
systemctl enable --now haproxy
ss -lntp | grep -E ':80|:443'       # expect haproxy on both
```

Then, and only then, open the ports — `ufw/rules.sh` keeps them commented precisely so
they are not opened ahead of a listener:

```sh
ufw allow 80/tcp  comment 'HAProxy — ACME HTTP-01 for the app host'
ufw allow 443/tcp comment 'HAProxy — SNI routing'
```

Reboot afterwards and confirm `haproxy` comes back with WireGuard, dnsmasq and ufw. Step 5's
checks all still apply.

---

## Step 4 — Enable ufw, with a deadman

**This is the step that locks you out if done in the wrong order.** `ufw enable` drops the
session that ran it unless SSH is already permitted.

```sh
# 1. arm a rollback FIRST — self-heals in 5 minutes if you lose access
echo 'ufw --force disable' | at now + 5 minutes

# 2. confirm SSH is allowed before enabling, not after
ufw status | grep -E '22/tcp'

# 3. enable
ufw --force enable

# 4. verify from a SECOND terminal, over the path you intend to keep
#    ssh arm-vpn 'echo still-here'

# 5. only then cancel the rollback
atq            # find the job id
atrm <id>
```

If step 4 fails, do nothing — the `at` job restores access in under five minutes. Do not try to
fix it from the session you still have; that is how a five-minute recovery becomes a console
session.

## Step 5 — Verify, end to end

```sh
./guard/guard-setup.sh status
```

Expect: both NICs, `ip_forward = 1`, dnsmasq and `wg-quick@wg0` active, ufw active, both split-DNS
names answering, exactly two nat rules.

Then from a workstation with the tunnel up:

```sh
sudo wg-quick up ~/.config/wireguard/arm-vpn.conf
ssh arm-vpn 'hostname -s'                                    # gateway reachable
dig +short @10.8.0.1 grafana.arametrics.app                   # expect 10.0.0.4
ssh -i ~/.ssh/id_ed25519 deploy@10.0.0.2 'hostname -s'       # estate reachable through the tunnel
```

And prove NAT egress from a host that has no public IP of its own — this is the rule three hosts
depend on, and it fails quietly:

```sh
ssh -i ~/.ssh/id_ed25519 (secret removed) 'curl -s -o /dev/null -w "%{http_code}\n" https://deb.debian.org/'
# expect 200. A hang means the egress MASQUERADE rule is wrong or missing.
```

## Step 6 — Record the number

Stop the clock. Write the elapsed time here, with the date and who ran it:

| Date | Who | Elapsed | Notes |
|---|---|---|---|
| — | — | — | *Not yet performed. Blocked on the server key not being backed up, so a rebuild cannot be completed or rehearsed.* |

Until there is a row in that table, the estate's RTO for a gateway loss is unknown, and
the rebuild procedure is not proven.

---

## What this procedure does not restore

**A running HAProxy.** Step 3b installs it and proves it against a scratch listener, but the
gateway is not a traffic gateway until `arametrics.app` resolves here and the app host's Caddy
accepts PROXY protocol. Neither is part of a rebuild — see the foot of `haproxy/haproxy.cfg`.

**The FORWARD policy.** Today it is `ACCEPT` and this rebuild reproduces that faithfully, because
that is the current state. Once the FORWARD policy is tightened, the rule set joins `guard/ufw/` and this
runbook needs a step for it — with the same deadman pattern as step 4, for the same reason.

**Peer configs for anyone who has lost theirs.** `vpn-wireguard-setup.sh add-peer <name>` issues
a new one, but the peer list from the password manager is what tells you who should have one.
