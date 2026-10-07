# Server migration plan — old M+ → new L+, in two boxes

Written 28 September 2026 from a live inventory of both servers. Rewritten the same day for the two-box layout.

> **THE ANSWER, for anyone who only reads one line here: the real production
> server is `217.160.93.75`, the app lives inside its `webapp` container
> (reach it with `ssh (secret removed)` then `inbox webapp` or
> `machinectl shell webapp`), and the public site is
> `https://apps.b-halle.de`. The old server, `212.227.213.174` /
> `feedback.arametrics.app`, was switched off 30 September 2026 and is not
> coming back. If any other file in this repo names `212.227.213.174` or
> `feedback.arametrics.app` as where production is, that file is wrong —
> this line is the source of truth, checked again whenever this repo is
> touched for a deploy.**

| | Old server | New server |
|---|---|---|
| IP | **212.227.213.174** | **217.160.93.75** |
| Size | 4 cores, 3.8 GB RAM, 120 GB | 4 cores, 7.9 GB RAM, 237 GB |
| System | Debian 12.15 | Debian 12.15, empty |
| Network card | — | `ens6`, managed by netplan rule `match: en*` |
| Login | `ssh -i ~/.ssh/halle_agent root@…` | same key, added 28 Sept |

## Status — 30 September 2026 (final: migration complete; only the old contract's cancellation is left)

**Done**
- ✅ Phase B — full local backup of the old server (`~/araCreate/HLE/server/halle-old-server-2026-09-28/`, sealed)
- ✅ Part 1 — host prepared (swap, key-only SSH, firewall 22/80/443, private network `br-boxes`, Apache, certificates)
- ✅ Part 2 — **Jupyter box live since 30 Sept 05:21 UTC** (~3 min offline). Final copy verified: 88,679 files, 0 unexpected differences; all rows and both logins match. Fresh backup at switch: `~/araCreate/HLE/server/jupyter-switch-2026-09-30/`
- ✅ Tested on the new server: real Jupyter login (`admin`), notebooks, pgAdmin login + database, product API from the website, full reboot
- ✅ Public traffic reaches the new box via **forwarding on the old server** (old Jupyter / API / pgAdmin are stopped there)

**Pending — Jupyter side**
1. ✅ **DNS done 30 Sept:** IONOS A `ttqvgsran` → `217.160.93.75`; all 4 IONOS nameservers + 1.1.1.1 / 8.8.8.8 / 9.9.9.9 confirmed. Old forwarding stays until old caches (TTL 3600) expire and until cancellation.
2. ✅ `certbot renew --cert-name ttqvgsran.b-halle.de --dry-run` on the host: succeeded (30 Sept)
3. ✅ **Old server switched off, 30 Sept afternoon** (on Vishnu's instruction, without waiting the clean day): `jupyterhub`, `halle-feedback` and `halle-feedback-hybrid-render` disabled and stopped there; Apache kept running for the forwarding. **Part 4 done the same time:** the temporary key `(secret removed)` removed from the old server's `authorized_keys`, and `/root/.ssh/migrate_old*` removed on the new host.
4. **Only step left:** Jakob cancels the old IONOS M+ (contract 111321277) after a day without problems. Until then its Apache keeps forwarding for any old DNS caches.

**Part 3, web app box — ✅ LIVE since 30 Sept 06:22 UTC on `apps.b-halle.de`** (Webflow tag published 06:25; live page loads `apps.b-halle.de/v1.js` + config 200, no calls to the old name). Switch: 1-freeze 06:20 → Mac backup `~/araCreate/HLE/server/webapp-switch-2026-09-30/` → 2-copy fail=0 → 3-verify ALL PASSED → 4-start all 200. Old server: halle-feedback + renderer stopped, not disabled. Done since: Vishnu's admin login and real reports with a tester link; old app and renderer disabled on the old server; Part 4 (temp key removal) — see item 3 above. History below.
- Decided 30 Sept (Vishnu): **Node 22**; the app moves to **`apps.b-halle.de` only** (hard cut, `feedback.arametrics.app` is not kept or forwarded, no Cloudflare change); the Webflow tag is changed **by hand** (Vishnu or Jakob).
- ✅ Box `webapp` 10.10.0.20 (3G cap), Node 22.23.3, Postgres 18.6, user `halle-feedback` 999:994 like old
- ✅ Copy old → box direct (`/root/copy-webapp.sh`): 36,016 files md5 0 diffs, 41,258 owner/perm 0 diffs; DB 9 tables rows + login MATCH
- ✅ Deliberate changes: `APP_URL=https://apps.b-halle.de` (original at `/opt/halle-feedback/env.before-migration`); widget built with `WIDGET_API_ORIGIN=https://apps.b-halle.de` — with the hostname swapped back its `v1.js` is byte-identical to live; `capture.js` identical
- ✅ Host vhosts `halle-feedback.conf` / `-le-ssl.conf` for `apps.b-halle.de` → `10.10.0.20:3000`; new Let's Encrypt cert (to 29 Dec). Publicly reachable **now** with the rehearsal data — new reports still go to the old server until the switch
- ✅ Backup + retention timers installed in the box and run once (backup had never worked: needed a pg_ident map `halle-feedback`→`halle_feedback`, added in the box; backups of `pg_hba`/`pg_ident` as `*.before-map`). Journal cap installed. Repo logrotate file NOT installed (Debian's apache2 rule already covers `*.log`)
- ✅ Separation: neither box reaches the other's Postgres; box restart brings everything back
- ✅ Switch scripts in the host's `/root/switch-webapp/` (1-freeze, 2-copy, 3-verify, 4-start, rollback); 3-verify dry run ALL PASSED
- ✅ End-to-end test report 30 Sept 06:19 UTC: real halle-dev page, widget script swapped to `apps.b-halle.de` in the test browser only (`.demo/uiwork/migration-e2e.mjs`), throwaway tester added to the rehearsal DB → config 200, server capture 200 (renderer `ok 41416b`), report 201, upload 201; the old server got nothing. The widget shows no launcher without a tester link — same on the live site, expected. The test report and tester disappear at 2-copy.
- **Still to do before switching:** Vishnu admin login on `https://apps.b-halle.de` (queue = 26 incl. the test report, old screenshot shows); agree a switch time
- **At the switch:** 1-freeze → Mac backup → 2-copy → 3-verify → 4-start → Webflow tag `<script src="https://apps.b-halle.de/v1.js" data-key="pk_live_66c10589" defer>` + publish → test report → later `old 'systemctl disable halle-feedback halle-feedback-hybrid-render'`. Widget is offline from 1-freeze until the Webflow publish.

**Pending — Part 4, afterwards**
7. Remove the temporary migration key from the old server (after Part 3's final copy)
8. Jakob cancels the old M+ contract (111321277) — only after the new contract's 30-day window and after both parts run cleanly
9. Final archive of the old server before cancelling; update `deploy/runbook.md`; delete local backups 30 days after cancellation

**Known and accepted:** product order inside a category may differ from before (no code change, decided 30 Sept); pgAdmin 9.14 copied as files, so no automatic updates; box Postgres is 18.6 (old 18.3).

## The layout on the new server

```
                         internet
                            │  80 / 443 only
┌───────────────────────────┴────────────────────────────────────────────┐
│ NEW SERVER (host)   Apache + HTTPS certificates + firewall + swap      │
│                     private network br-boxes 10.10.0.1/24              │
│        ┌──────────────────────────┐     ┌───────────────────────────┐  │
│        │ JUPYTER BOX  10.10.0.10  │     │ WEB APP BOX  10.10.0.20   │  │
│        │ JupyterHub        :8000  │     │ feedback app      :3000   │  │
│        │ Halle product API :9000  │     │ screenshot render :4600   │  │
│        │ pgAdmin           :8080  │     │ Postgres (local only)     │  │
│        │ Postgres (local only)    │     │   halle_feedback          │  │
│        │   halle-db, halle-test-db│     │ screenshots               │  │
│        │ notebooks, admin, jakob  │     │                           │  │
│        └──────────────────────────┘     └───────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

- **Boxes are systemd-nspawn containers**: Debian's own tool (package `systemd-container`), open source, no background daemon. Each box is a small Debian 12 with its own files, users, services and database.
- **The host only runs the front door:** Apache receives every visitor and passes them to the right box. HTTPS certificates live here.
- **Each box's database listens only inside that box**, so the Jupyter box can never reach the web app's data, and the other way round.
- **Memory limits per box**, so one box can never starve the other: Jupyter 3 GB, web app 3 GB, the rest for the host.
- **What goes where** was decided by sorting every one of the 161,240 non-Debian files on the old server (0 unsorted). The lists are in `~/araCreate/HLE/server/halle-migration-manifest-2026-09-28/` (`sorted-JUPYTER.txt`, `sorted-WEBAPP.txt`, …).

## Order

1. **Phase B — full local backup** ✅ done 28 Sept, see below.
2. **Part 1 — host:** prepare the new server once (firewall, swap, Apache, private network).
3. **Part 2 — Jupyter box:** build, copy, test fully, switch `ttqvgsran.b-halle.de`.
4. **Part 3 — web app box:** only after Part 2 has run cleanly. Switch `feedback.arametrics.app`.
5. **Part 4 — after:** watch, tidy, cancel the old server.

The two switch-overs are independent: the feedback app doesn't use the product API or Jupyter, and nothing in the Jupyter box uses the feedback app.

## The rules that keep this safe

1. **The backup exists and is verified before anything else** — done.
2. **Nothing is ever changed on the old server** except stopping services at each switch-over. It stays a complete, working fallback until the very end.
3. **Keep every web address the same.** Webflow loads `https://feedback.arametrics.app/v1.js` and calls `https://ttqvgsran.b-halle.de/api/` (confirmed from Apache logs, referer `halle-dev.webflow.io`). Only the DNS A records move, so **no change to Webflow at all**. `apps.b-halle.de` (already pointing at the new server) is left for later.
4. **Copy, don't reinvent.** Inside each box: the same Debian 12, the same Python 3.11.2, the same program versions, the same files copied byte for byte. The Jupyter box even keeps Node 20 like the old server. Only what the box layout forces is changed, and each such change is listed below under "Deliberate changes".
5. **Not a single file missed.** After every copy, a checksum comparison of every file on the box's list, old vs new. It must be empty.
6. **Always use IP addresses in commands**, never hostnames. After a DNS switch, the hostname means the *new* server.
7. **Every step has a check.** Don't start the next step until it passes.
8. **Test everything before any DNS change**, from the Mac only, with the real hostnames pointed at the new IP in `/etc/hosts`.

### Deliberate changes (forced by the box layout)

| Change | Why |
|---|---|
| JupyterHub `bind_url` `127.0.0.1:8000` → `10.10.0.10:8000` | The front door is now outside the box |
| pgAdmin served by the Jupyter box's own Apache on `:8080`, proxied by the host | pgAdmin runs inside Apache; the host's Apache can't load a program inside a box |
| One Postgres per box instead of one shared | Separation — each box has only its own databases |
| Firewall, swap, key-only SSH on the host | The old server had none of these |
| Port 9000 no longer open to the internet | Checked 28 Sept: nobody calls it directly — every real API call comes through Apache (`/api/…`); direct hits were only scanning bots |

## Decisions needed

- **A. Node version in the web app box.** The feedback app runs on Node 20 today (end-of-life; breaks `make user-*`). Recommended: Node 22 in the web app box, proven by the Part 3 test. The Jupyter box stays on Node 20 (only PM2 and Jupyter's proxy use it).
- **B. Port 9000 — resolved, stays closed.** On the old server the product API was open on port 9000 to the whole internet. Its log (79,268 lines) shows **no direct callers at all**: every real call arrives through Apache over HTTPS (`ttqvgsran.b-halle.de/api/…`; the addresses logged with `:0` are visitors passed on by Apache). So the new server keeps 9000 closed.
  **IONOS has its own firewall in the Cloud Panel** (Netzwerk → Firewall-Richtlinien), separate from the server's. The new server's policy must allow **TCP 22, 80 and 443** — Jakob checks this. Check from the Mac: `nc -vz -w 5 217.160.93.75 443` (and 80).
- **C. Switch-over times.** Two short windows (Jupyter, then web app), each ~20 minutes offline for that part only. Agree with Jakob.
- **D. Who changes DNS.** `ttqvgsran.b-halle.de` — IONOS (Jakob, or the IONOS user he adds for Vishnu). `feedback.arametrics.app` — Cloudflare, us?
- **E. IONOS console.** If a network mistake ever cut SSH, the only way back in is the IONOS Cloud Panel's remote console — Jakob's account. Ask him to be reachable during Part 1.

## Setup on the Mac (once, in every Terminal window used)

```
old() { ssh -i ~/.ssh/halle_agent (secret removed) "$@"; }
new() { ssh -i ~/.ssh/halle_agent (secret removed) "$@"; }
```
Check: `old hostname` and `new hostname` both print `my-vps`, `new nproc` prints `4`.

Files are copied **through the Mac** (`old 'tar …' | new 'tar …'`), so the two servers never need keys to each other.

Commands **inside a box** are sent through a helper on the host (installed in step 1.7):
```
new 'inbox jupyter' <<'EOF'
apt update
EOF
```
Everything between the `EOF` lines runs inside the box as root.

---

## Phase B — Full local backup ✅ DONE 28 Sept 2026, 18:06

In `~/araCreate/HLE/server/halle-old-server-2026-09-28/` (4.6 GB, 19 files):
- `all-databases.sql` (everything, with logins) and `db-halle_feedback.dump`, `db-halle-db.dump`, `db-halle-test-db.dump`
- `files-feedback-app.tgz`, `files-screenshots.tgz`, `files-backend.tgz`, `files-jupyter.tgz`, `files-pgadmin.tgz`, `files-system-config.tgz`
- `full-system.tgz` — the **whole filesystem** (3.7 GB); every file on the sorted lists was checked present
- `list-*.txt` — every Debian, pip and npm package with versions; enabled services; git commits
- `SHA256SUMS` — prove nothing has changed: `cd ~/araCreate/HLE/server/halle-old-server-2026-09-28 && shasum -a 256 -c SHA256SUMS`

**It contains every secret on the server.** Keep it only there (FileVault is on). Never put it in the repo, iCloud, Google Drive, Slack or email. Keep it until 30 days after the old server is cancelled.

A second, fresh backup of the changing data is taken at each switch-over (steps 2.9 and 3.9).

---

## Part 1 — Prepare the host (no downtime; the old server is untouched)

**1.1 Updates and base programs**
```
new 'apt update && DEBIAN_FRONTEND=noninteractive apt -y full-upgrade'
new 'DEBIAN_FRONTEND=noninteractive apt -y install systemd-container debootstrap curl ca-certificates gnupg rsync ufw apache2 libapache2-mod-evasive certbot python3-certbot-apache'
```
Check: `new 'systemd-nspawn --version | head -1; debootstrap --version; apache2 -v | head -1'`.

**1.2 Swap (2 GB)** — the old server had none:
```
new 'fallocate -l 2G /swapfile && chmod 600 /swapfile && mkswap /swapfile && swapon /swapfile && echo "/swapfile none swap sw 0 0" >> /etc/fstab'
```
Check: `new 'free -m | grep Swap'` shows ~2047.

**1.3 Key-only SSH** (the initial password has been in chat):
```
new 'printf "PasswordAuthentication no\nKbdInteractiveAuthentication no\n" > /etc/ssh/sshd_config.d/10-key-only.conf && sshd -t && systemctl reload ssh'
```
Check: in a **second** Terminal, `new true` still logs in. Only then close the first.

**1.4 Private network for the boxes** — a bridge `br-boxes` with its own addresses and internet access through the host. Its name doesn't start with `en`, so the netplan rule for `ens6` (the server's own connection) never touches it:
```
new 'cat > /etc/systemd/network/10-br-boxes.netdev <<EOF
[NetDev]
Name=br-boxes
Kind=bridge
EOF
cat > /etc/systemd/network/10-br-boxes.network <<EOF
[Match]
Name=br-boxes
[Network]
Address=10.10.0.1/24
IPMasquerade=ipv4
ConfigureWithoutCarrier=yes
EOF
networkctl reload && sleep 3 && ip -br addr show br-boxes && ip -br addr show ens6'
```
Check: `br-boxes` shows `10.10.0.1/24`, **and `ens6` still shows `217.160.93.75`**. Then `new true` from a second Terminal. **Never run `netplan apply`.**

**1.5 Firewall** — SSH allowed *before* enabling; forwarding allowed only from the boxes to the internet:
```
new 'ufw allow OpenSSH && ufw allow 80/tcp && ufw allow 443/tcp && ufw route allow in on br-boxes out on ens6 && ufw allow in on br-boxes to 10.10.0.1 && ufw --force enable && ufw status verbose'
```
Check: status `active`, rules for 22, 80, 443 and the br-boxes route — **no 9000**. Then `new true` from a second Terminal.

**1.6 Apache modules**
```
new 'a2enmod proxy proxy_http proxy_wstunnel ssl headers rewrite evasive && systemctl restart apache2'
```
Check: `new 'apache2ctl -M | grep -cE "proxy_module|proxy_http|proxy_wstunnel|ssl_module|headers|rewrite|evasive"'` → 7.

**1.7 The `inbox` helper** (runs a script from stdin inside a box):
```
new 'printf "#!/bin/sh\nexec systemd-run -M \"\$1\" --quiet --wait --pipe --collect /bin/bash -s\n" > /usr/local/sbin/inbox && chmod 755 /usr/local/sbin/inbox && cat /usr/local/sbin/inbox'
```

**1.8 Certificates** — copied now so HTTPS works for testing before any DNS change (valid until 4 Nov and 8 Dec; renewal works once DNS points here):
```
old 'tar -C / -czf - etc/letsencrypt var/lib/letsencrypt' | new 'tar -C / -xzpf -'
```
Check: `new 'ls /etc/letsencrypt/live'` shows both names.

---

## Part 2 — Jupyter box

Everything on `sorted-JUPYTER.txt` goes into this box, except the Apache site files and the `ttqvgsran` certificate, which belong to the host's front door. That is: JupyterHub (`/usr/local`), the notebook environment `/opt/jupyterhub-env`, kernels, `/etc/jupyterhub`, `/shared/notebooks`, users `admin` and `jakob` with their homes, the product API `/root/github/halle-app-backend` with PM2 and `/root/.pm2`, `/root/.refractiveindex.info-database`, `/root/.git-credentials`, `/root/.gitconfig`, `/root/.jupyter`, `/root/.local/share/jupyter`, pgAdmin (`/usr/pgadmin4`, `/var/lib/pgadmin`, `/var/log/pgadmin`), the global Node tools `pm2` and `configurable-http-proxy`, and the databases `halle-db` and `halle-test-db` with the login `halle_user`.

### 2.1 Create the box (a fresh Debian 12)

```
new 'debootstrap --include=systemd,dbus,systemd-resolved,iproute2,ca-certificates,curl,gnupg,locales bookworm /var/lib/machines/jupyter http://deb.debian.org/debian'
new 'mkdir -p /etc/systemd/nspawn && cat > /etc/systemd/nspawn/jupyter.nspawn <<EOF
[Exec]
Boot=yes
[Network]
Bridge=br-boxes
EOF
mkdir -p /etc/systemd/system/systemd-nspawn@jupyter.service.d && cat > /etc/systemd/system/systemd-nspawn@jupyter.service.d/limits.conf <<EOF
[Service]
MemoryMax=3G
MemoryHigh=2600M
CPUWeight=100
EOF
systemctl daemon-reload'
```
Give the box its fixed address before first start (from the host, by path):
```
new 'mkdir -p /var/lib/machines/jupyter/etc/systemd/network && cat > /var/lib/machines/jupyter/etc/systemd/network/80-box.network <<EOF
[Match]
Name=host0
[Network]
Address=10.10.0.10/24
Gateway=10.10.0.1
DNS=212.227.123.16
DNS=212.227.123.17
EOF
echo jupyter > /var/lib/machines/jupyter/etc/hostname
systemd-nspawn -D /var/lib/machines/jupyter --pipe /bin/sh -c "systemctl enable systemd-networkd systemd-resolved"'
new 'machinectl enable jupyter && machinectl start jupyter && sleep 8 && machinectl list'
```
Check — all must pass:
```
new 'inbox jupyter' <<'EOF'
hostname; cat /etc/debian_version; ip -br addr show host0; getent hosts deb.debian.org; curl -sI https://deb.debian.org | head -1
EOF
```
→ `jupyter`, `12.x`, `10.10.0.10/24`, an address for deb.debian.org, `HTTP/… 200`. And from the host: `new 'ping -c1 -W2 10.10.0.10'`.

### 2.2 Install the same programs as the old server, inside the box

Versions from `list-debian-packages.txt` / `list-node-global.txt`: Python 3.11.2 (Debian's), Node **20** (NodeSource, like the old server), Postgres 18 (pgdg), pgAdmin 9.14.
```
new 'inbox jupyter' <<'EOF'
set -e
export DEBIAN_FRONTEND=noninteractive
apt update && apt -y full-upgrade
apt -y install python3 python3-pip python3-venv python3-dev build-essential git apache2 libapache2-mod-wsgi-py3 sudo
curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt -y install nodejs
apt -y install postgresql-common && /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh -y && apt -y install postgresql-18 postgresql-client-18
apt -y install libpq5 libgssapi-krb5-2 python3-dbus
python3 -V; node -v; npm -v; psql -V; apache2 -v | head -1
EOF
```
Check: `Python 3.11.2`, `v20.20.2`, npm `10.8.2`, Apache `2.4.68` — all identical to the old server. Postgres comes as the newest 18.x (18.6 on 28 Sept vs 18.3 on the old server): same major version, identical data format, newer security fixes.

**pgAdmin 9.14 is no longer downloadable** (checked 28 Sept: the repository starts at 9.15, and the 9.14 files return 404). pgAdmin keeps its whole program in one folder with its own Python (`/usr/pgadmin4`), so the exact 9.14 is **copied from the old server in 2.4**; the three system packages it needs (`libpq5`, `libgssapi-krb5-2`, `python3-dbus`, plus Apache and `libapache2-mod-wsgi-py3`) are installed above. It gets no automatic updates this way — upgrade it deliberately later, after the migration.

### 2.2b Debian's Python libraries that JupyterHub uses

```
new 'inbox jupyter' <<'EOF'
DEBIAN_FRONTEND=noninteractive apt-get -y install python3-attr python3-blinker python3-certifi python3-cffi-backend python3-chardet python3-charset-normalizer python3-configargparse python3-configobj python3-cryptography python3-debconf python3-debian python3-distro python3-httplib2 python3-icu python3-idna python3-jinja2 python3-json-pointer python3-jsonpatch python3-jsonschema python3-jwt python3-markdown-it python3-markupsafe python3-mdurl python3-netifaces python3-oauthlib python3-openssl python3-parsedatetime python3-pycurl python3-pygments python3-pyparsing python3-pyrsistent python3-pysimplesoap python3-requests python3-rfc3339 python3-rich python3-serial python3-six python3-tz python3-urllib3 python3-yaml python3-josepy
python3 -m pip check
EOF
old 'python3 -m pip check'
```
Check: both print only `pygobject 3.42.2 requires pycairo, which is not installed.`

### 2.3 Users with the same IDs and passwords

```
new 'inbox jupyter' <<'EOF'
useradd -m -u 1000 -s /bin/bash admin && useradd -m -u 1002 -s /bin/bash jakob && id admin && id jakob
EOF
old 'getent shadow admin jakob | cut -d: -f1,2' | new 'systemd-run -M jupyter -q --wait --pipe --collect chpasswd -e'
```
Check (Jupyter logs in with this password):
```
for u in admin jakob; do [ "$(old "getent shadow $u | cut -d: -f2 | sha1sum")" = "$(new 'inbox jupyter' <<< "getent shadow $u | cut -d: -f2 | sha1sum")" ] && echo "$u PASSWORD MATCH" || echo "$u PASSWORD DIFFERENT"; done
```

### 2.4 Copy every file (rehearsal — the old server keeps running)

Straight from the live old server into the box's folder on the host, byte for byte, with owners and permissions. Global Node tools included:
```
old 'tar -C / -czf - usr/local opt/jupyterhub-env etc/jupyterhub shared home/admin home/jakob root/github root/.pm2 root/.refractiveindex.info-database root/.git-credentials root/.gitconfig root/.jupyter root/.local/share/jupyter usr/pgadmin4 var/lib/pgadmin var/log/pgadmin usr/lib/node_modules/pm2 usr/lib/node_modules/configurable-http-proxy' | new 'tar -C /var/lib/machines/jupyter -xzpf - && ln -sf ../lib/node_modules/pm2/bin/pm2 /var/lib/machines/jupyter/usr/bin/pm2 && ln -sf ../lib/node_modules/configurable-http-proxy/bin/configurable-http-proxy /var/lib/machines/jupyter/usr/bin/configurable-http-proxy'
old 'tar -C / -czf - etc/systemd/system/jupyterhub.service etc/systemd/system/pm2-root.service' | new 'tar -C /var/lib/machines/jupyter -xzpf -'
```
`socket ignored` messages are harmless (PM2 recreates its sockets).

**Check — not a single file missed.** Checksums of every file on the list, old vs box:
```
J='usr/local opt/jupyterhub-env etc/jupyterhub shared home/admin home/jakob root/github root/.pm2 root/.refractiveindex.info-database root/.git-credentials root/.gitconfig root/.jupyter root/.local/share/jupyter usr/pgadmin4 var/lib/pgadmin var/log/pgadmin usr/lib/node_modules/pm2 usr/lib/node_modules/configurable-http-proxy'
diff <(old "cd / && find $J \( -type f -o -type l \) -print0 | sort -z | xargs -0 md5sum") <(new "cd /var/lib/machines/jupyter && find $J \( -type f -o -type l \) -print0 | sort -z | xargs -0 md5sum") > /tmp/jupyter-diff.txt; echo "differences: $(grep -c '^[<>]' /tmp/jupyter-diff.txt)"
```
During the rehearsal a few **live** files may differ (PM2 logs, `jupyterhub.sqlite`, pgAdmin sessions) — check that every difference is one of those. At switch-over (2.9) the result must be **0**.

Also check the count against the sorted list — every path from `sorted-JUPYTER.txt` except the Apache and certificate entries must exist in the box:
```
M=~/araCreate/HLE/server/halle-migration-manifest-2026-09-28
grep -vE '^/etc/(apache2|letsencrypt)/|^/var/www/|^/etc/apt/|^/usr/bin/|^/etc/systemd/system/' $M/sorted-JUPYTER.txt | new 'cd /var/lib/machines/jupyter && n=0; while IFS= read -r f; do [ -e ".$f" ] || [ -L ".$f" ] || { echo "MISSING $f"; n=$((n+1)); }; done; echo "missing: $n"'
```
→ `missing: 0` (a few PM2 log files rotated since 28 Sept are acceptable only if named `*.log` under `/root/.pm2/logs`).

### 2.5 Databases `halle-db` and `halle-test-db`

Only the Jupyter box's login `halle_user` (with its password hash) and its two databases:
```
old 'sudo -u postgres pg_dumpall --roles-only' | grep -wE 'halle_user' | new 'systemd-run -M jupyter -q --wait --pipe sudo -u postgres psql -q'
old 'sudo -u postgres pg_dumpall --roles-only' | grep -E '^ALTER ROLE postgres WITH' | new 'systemd-run -M jupyter -q --wait --pipe sudo -u postgres psql -q'
for db in halle-db halle-test-db; do old "sudo -u postgres pg_dump -Fc --create '$db'" | new "systemd-run -M jupyter -q --wait --pipe sudo -u postgres pg_restore --create --clean --if-exists -d postgres"; done
```
Check — row counts identical, table by table (helper written once to both):
```
for s in old new; do $s 'cat > /root/rowcounts.sh' <<'EOF'
for db in "$@"; do
  sudo -u postgres psql -d "$db" -Atc "select format('select %L, count(*) from %I.%I;', '$db.'||schemaname||'.'||relname, schemaname, relname) from pg_stat_user_tables order by 1" | sudo -u postgres psql -d "$db" -At
done
EOF
done
new 'cp /root/rowcounts.sh /var/lib/machines/jupyter/root/'
diff <(old 'bash /root/rowcounts.sh halle-db halle-test-db') <(new 'systemd-run -M jupyter -q --wait --pipe bash /root/rowcounts.sh halle-db halle-test-db') && echo "ROWS MATCH"
```
**The `postgres` login's password must be copied too** — pgAdmin's saved connection (`halle-dev`) logs in as `postgres` over `localhost:5432`, which asks for a password. Missed on the first pass (28 Sept), caught 30 Sept. Check both logins' password fingerprints match:
```
Q="select rolname||' '||md5(coalesce(rolpassword,''))||' super='||rolsuper from pg_authid where rolname in ('postgres','halle_user') order by 1"
diff <(old "sudo -u postgres psql -At -c \"$Q\"") <(new 'inbox jupyter' <<< "sudo -u postgres psql -At -c \"$Q\"") && echo "LOGINS MATCH"
```

Also: `new 'inbox jupyter' <<< 'sudo -u postgres psql -Atc "select rolname from pg_roles where rolname like '"'"'halle%'"'"'"'` → only `halle_user` (no `halle_feedback` — that belongs to the web app box).

### 2.6 The deliberate changes

**JupyterHub listens on the box's address** (the only line changed in its config):
```
new "sed -i \"s#^c.JupyterHub.bind_url = 'http://127.0.0.1:8000'#c.JupyterHub.bind_url = 'http://10.10.0.10:8000'#\" /var/lib/machines/jupyter/etc/jupyterhub/jupyterhub_config.py && grep '^c.JupyterHub.bind_url' /var/lib/machines/jupyter/etc/jupyterhub/jupyterhub_config.py"
```
→ `c.JupyterHub.bind_url = 'http://10.10.0.10:8000'`.

**pgAdmin on the box's own Apache, port 8080:**
```
new 'inbox jupyter' <<'EOF'
set -e
echo "Listen 8080" > /etc/apache2/ports.conf
cat > /etc/apache2/sites-available/pgadmin.conf <<'X'
<VirtualHost *:8080>
    WSGIDaemonProcess pgadmin processes=1 threads=25 python-home=/usr/pgadmin4/venv
    WSGIScriptAlias /pgadmin4 /usr/pgadmin4/web/pgAdmin4.wsgi
    <Directory /usr/pgadmin4/web/>
        WSGIProcessGroup pgadmin
        WSGIApplicationGroup %{GLOBAL}
        Require all granted
    </Directory>
</VirtualHost>
X
a2dissite 000-default; a2disconf pgadmin4 2>/dev/null || true; a2ensite pgadmin; a2enmod wsgi
chown -R www-data:www-data /var/lib/pgadmin /var/log/pgadmin
apache2ctl configtest && systemctl restart apache2
EOF
```

### 2.7 Start the box's services

```
new 'inbox jupyter' <<'EOF'
systemctl daemon-reload
systemctl enable --now jupyterhub
systemctl enable pm2-root && systemctl start pm2-root
sleep 15
systemctl is-active jupyterhub pm2-root apache2 postgresql
pm2 ls
ss -tlnp | grep -E ':(8000|9000|8080|5432) '
EOF
```
Check: all `active`, PM2 shows `halle-python-server` `online`, ports 8000 and 9000 and 8080 listening, **5432 only on 127.0.0.1**. If PM2 shows nothing: `pm2 resurrect` then `pm2 save`.

From the host:
```
new 'curl -s -o /dev/null -w "%{http_code}\n" http://10.10.0.10:8000/jupyter/hub/login; curl -s -o /dev/null -w "%{http_code}\n" http://10.10.0.10:9000/health; curl -s -o /dev/null -w "%{http_code}\n" http://10.10.0.10:8080/pgadmin4/login'
```
→ `200`, `200`, `200`. (The API's `/docs` page is switched off in its code, so `/health` is the check. Check the Jupyter login page, not `/` or `/hub/login` — see agent-rules.md.)

**Real session test without the password** (a login starts a personal notebook session; this starts one the same way, with a temporary hub key that disappears at the final copy because `jupyterhub.sqlite` is copied fresh from the old server):
```
new 'inbox jupyter' <<'EOF'
cd /shared/notebooks; T=$(jupyterhub token -f /etc/jupyterhub/jupyterhub_config.py admin | tail -1); H=http://10.10.0.10:8000/jupyter/hub/api
curl -s -o /dev/null -w "start %{http_code}\n" -X POST -H "Authorization: token $T" $H/users/admin/server; sleep 15
curl -s -o /dev/null -w "session %{http_code}\n" -H "Authorization: token $T" http://10.10.0.10:8000/jupyter/user/admin/api/status
curl -s -o /dev/null -w "stop %{http_code}\n" -X DELETE -H "Authorization: token $T" $H/users/admin/server
EOF
```
→ `start 201`, `session 200`, `stop 204`. Passed 30 Sept.

**All notebooks, old vs box** — run each on a temporary copy as `admin` (`jupyter nbconvert --execute`); the pass/fail list must be identical. 30 Sept: 6 pass, the same 3 fail on both (old notebooks in `Archive`/`Trash`: a missing logo file, and two calling older code).

### 2.8 The host's front door for the Jupyter box

The two old site files, with every `localhost` / `127.0.0.1` target pointed at the box:
```
old 'tar -C / -czf - etc/apache2/sites-available/000-default.conf etc/apache2/sites-available/000-default-le-ssl.conf etc/apache2/mods-available/evasive.conf var/www/html' | new 'tar -C / -xzpf -'
new 'cd /etc/apache2/sites-available && sed -i "s#http://localhost:9000/#http://10.10.0.10:9000/#g; (secret removed):(secret removed):8000#g" 000-default.conf 000-default-le-ssl.conf && grep -nE "ProxyPass|RewriteRule" 000-default.conf 000-default-le-ssl.conf'
```
Add pgAdmin to the HTTPS site (it was served by the same Apache before):
```
new 'sed -i "s#^ProxyPass /jupyter http://10.10.0.10:8000/jupyter#ProxyPass /pgadmin4 http://10.10.0.10:8080/pgadmin4\nProxyPassReverse /pgadmin4 http://10.10.0.10:8080/pgadmin4\n&#" /etc/apache2/sites-available/000-default-le-ssl.conf && grep -n pgadmin4 /etc/apache2/sites-available/000-default-le-ssl.conf'
```
Enable:
```
new 'a2ensite 000-default 000-default-le-ssl && apache2ctl configtest && systemctl reload apache2'
```
Check: `Syntax OK`, then `new 'ss -tlnp | grep -E ":(80|443) "'` shows Apache on both.

### 2.9 Full test before switching (the public still uses the old server)

On the **Mac**, point only this name at the new server:
```
sudo sh -c 'echo "217.160.93.75 ttqvgsran.b-halle.de" >> /etc/hosts' && sudo dscacheutil -flushcache && sudo killall -HUP mDNSResponder
```
Every line must pass:
- [ ] `curl -sI https://ttqvgsran.b-halle.de/jupyter/hub/login | head -1` → `200`, and `new 'tail -1 /var/log/apache2/access.log'` shows your request (proves you're on the new server)
- [ ] Browser: `https://ttqvgsran.b-halle.de/jupyter` — log in as **admin** with the usual password
- [ ] Both folders are there (`Jupyter Stuff`, `R_retardation_curves`) with every notebook
- [ ] Open a notebook in `R_retardation_curves`, choose kernel **Python (jupyterhub-env)**, **run all cells** — no errors, the plots appear (proves numpy, matplotlib, refractiveindex)
- [ ] Save a small test notebook, reload the page, it's still there; then delete it
- [ ] `https://ttqvgsran.b-halle.de/pgadmin4` — log in, the saved server connections are listed and open
- [ ] On `halle-dev.webflow.io` (in the same browser), open 3 product pages — their data loads (DevTools → Network: calls to `ttqvgsran.b-halle.de/api/…`, status 200)
- [ ] `curl -s -o /dev/null -w "%{http_code}\n" https://ttqvgsran.b-halle.de/api/health` → `200`
- [ ] `nc -vz -w 5 217.160.93.75 9000` → **fails** (port 9000 is closed to the internet, Decision B)
- [ ] `new 'free -m; systemctl status systemd-nspawn@jupyter | grep -E "Memory|Active"'` — box running, under its 3 GB limit
- [ ] **Separation:** `new 'inbox jupyter' <<< 'curl -s -m 3 http://10.10.0.20:5432 && echo REACHABLE || echo blocked'` → `blocked` (the web app box doesn't exist yet; re-checked in Part 3)
- [ ] **Reboot test:** `new 'reboot'`, wait 2 minutes, then repeat the first four checks — everything comes back by itself

Then remove the test line:
```
sudo sed -i '' '/ttqvgsran.b-halle.de/d' /etc/hosts && sudo dscacheutil -flushcache && sudo killall -HUP mDNSResponder
```
**Stop here if anything failed.** Nothing public has changed; fix and re-test.

### 2.10 Jupyter switch-over (~20 minutes, agreed time)

**Decided 30 Sept: no TTL change needed from Jakob.** The DNS record keeps its 1-hour TTL; instead, for that hour the **old server forwards** `ttqvgsran.b-halle.de` visitors to the new server (tested 30 Sept under a made-up test name: `/api/health`, `/jupyter/hub/login`, `/pgadmin4/login` all 200 through old → new, a real product call returned all 24 products, the Jupyter websocket reached the new hub). Jakob's only task: change one A record at the agreed moment.

Everything is scripted on the new server in `/root/switch/` (written and syntax-checked 30 Sept). Run each from the Mac with `new 'bash /root/switch/<script>'` and **don't start the next until the previous one's last line says DONE / PASSED**:

| # | What | Script / action | Must end with |
|---|---|---|---|
| 0 | Jakob tells Jupyter users to **save their work** — sessions are closed at step 1 | — | — |
| 1 | Stop Jupyter, product API and pgAdmin on the old server and in the box (feedback app untouched) | `1-freeze.sh` | `STEP 1 DONE` |
| 2 | **Fresh local backup** of the Jupyter-side data to the Mac (below) | Mac | `SHA256SUMS` written |
| 3 | Final copy: wipe each path in the box and copy it fresh, re-apply `bind_url`, both logins, both databases | `2-copy.sh` | `STEP 2 DONE fail=0` |
| 4 | Every file, owner/permission, row and login compared | `3-verify.sh` | `STEP 3 ALL CHECKS PASSED` |
| 5 | Start the box (PM2 from `ecosystem.config.js` — the copied `dump.pm2` is stale) | `4-start.sh` | three `200` |
| 6 | Old server forwards to new | `5-forward-on.sh` | three `200` old-address → new, feedback app `200` |
| 7 | **Jakob:** IONOS → Domains → b-halle.de → DNS → `ttqvgsran` → IP `212.227.213.174` → **`217.160.93.75`** → Save | Jakob | — |
| 8 | Watch it arrive: `dig +short ttqvgsran.b-halle.de @ns1071.ui-dns.org` → new IP at once; public resolvers within ≤ 1 hour (forwarding covers the gap) | Mac | new IP |
| 9 | Public tests: the 2.9 list, now without `/etc/hosts`; **Jakob logs in to Jupyter and pgAdmin** | Mac + Jakob | all pass |
| 10 | `new 'certbot renew --cert-name ttqvgsran.b-halle.de --dry-run'` (after public DNS shows the new IP) | host | success |

Step 2, fresh local backup (Mac, after step 1):
```
cd ~/araCreate/HLE/server && D=jupyter-switch-$(date +%F) && mkdir -p $D && cd $D && for db in halle-db halle-test-db; do old "sudo -u postgres pg_dump -Fc --create '$db'" > "db-$db.dump"; done && old 'tar -C / -czf - shared home etc/jupyterhub root/.pm2 var/lib/pgadmin' > files-jupyter-data.tgz && shasum -a 256 * > (secret removed) && ls -lh
```

**Rollback** (any step fails and can't be fixed in 15 minutes): `new 'bash /root/switch/rollback.sh'` — puts the old server's front-door files back from `/root/apache-before-migration/`, re-enables pgAdmin, restarts Jupyter and the product API on the old server. If step 7 was done: Jakob sets the IP back to `212.227.213.174`. Anything saved in the box in between is copied back with the same `tar`/`pg_dump` commands, reversed.

**After a clean day:** on the old server, keep the Jupyter side stopped (`old 'systemctl disable jupyterhub'`). Leave the forwarding in place until the old server is cancelled — it costs nothing and catches any stale DNS.

**Part 3 starts only after the Jupyter box has run cleanly for at least a few days.**

---

## Open question: product order

`POST /api/product/products` has no `ORDER BY`, so the order of products it returns is decided by the database, not the code. Checked 28 Sept: the content is identical on the old server and in the box for all 32 categories, but the order differs for 31 of them (e.g. *Achromatic Waveplates*: old `1, 2, 3, 4…`, box `4, 3, 2, 1, 8, 7…`). The old server's order isn't guaranteed either — it can change by itself after the database's next automatic statistics refresh.

If the Webflow product pages show products in the order the API returns them, the order on the site will change after the switch. The proper fix is one line in `halle-app-backend/app/catalog/router.py` (`get_products`): add an `ORDER BY` (e.g. `p.id`) before `LIMIT`, which makes the order the same on every server, forever. **Decided 30 Sept (Vishnu): leave it.** No code change. After the switch the order of products inside a category may differ from today (same products, same data) — that is expected, not a fault. On the old server today the order is already jumbled in places (e.g. *Right Angle Prisms*: `UPS 0.10 … UPB 1.30, UPB 1.40, UPS 1.10 …`).

## Part 3 — Web app box (after Part 2)

Only what our repo (`halle-feedback-widget`) runs: `src/web`, `src/widget`, `src/render`, the database `halle_feedback` with login `halle_feedback`, and the screenshots. Same pattern as Part 2 — each step below has the same kind of check.

**3.1 Create the box** — exactly as 2.1 with `webapp` instead of `jupyter`, address `10.10.0.20`, and `MemoryMax=3G` / `MemoryHigh=2600M`.

**3.2 Programs** — Node 22 (Decision A), Postgres 18, git, build-essential; no Apache inside.

**3.3 User** — `useradd --system --home-dir /opt/halle-feedback --shell /usr/sbin/nologin halle-feedback`.

**3.4 Copy** — from the old server into `/var/lib/machines/webapp`:
```
old 'tar -C / -czf - --exclude=node_modules --exclude=.next --exclude=src/widget/dist --exclude=audit-out opt/halle-feedback var/lib/halle-feedback etc/systemd/system/halle-feedback.service etc/systemd/system/halle-feedback-hybrid-render.service' | new 'tar -C /var/lib/machines/webapp -xzpf -'
```
Then inside the box: `chown -R halle-feedback:halle-feedback /opt/halle-feedback /var/lib/halle-feedback`, `chmod 600 /opt/halle-feedback/app/src/web/.env`, `git checkout -- package-lock.json`, check `git log -1` = `ab552a1` and `git status` empty. Checksum diff of `var/lib/halle-feedback` and `opt/halle-feedback/app` (without the excluded build folders) → 0.

**3.5 Build** (always as `halle-feedback`, never root):
```
cd /opt/halle-feedback/app && sudo -u halle-feedback -H npm ci
sudo -u halle-feedback -H npx playwright install chromium && npx playwright install-deps chromium
sudo -u halle-feedback -H env NODE_OPTIONS=--max-old-space-size=1536 npm run build --workspace halle-feedback-web
sudo -u halle-feedback -H env WIDGET_API_ORIGIN=https://feedback.arametrics.app npm run build --workspace halle-feedback-widget-embed
```
Check: `grep -c localhost:3000 src/widget/dist/v1.js` → **0** (without `WIDGET_API_ORIGIN` the widget silently dies — the 23 Sept outage).

**3.6 Database** — the `halle_feedback` role line from `pg_dumpall --roles-only` and `pg_dump -Fc --create halle_feedback`, into the box. Row counts must match. Locale of this database is `C.utf8` (built into Debian).

**3.7 Install the repo's timers that were never installed on the old server** — nightly database backup and weekly screenshot clean-up (`deploy/halle-feedback-backup.*`, `deploy/halle-feedback-retention.*`, per `deploy/runbook.md`).

**3.8 Start** — `systemctl enable --now halle-feedback-hybrid-render halle-feedback`; wait 15 s; ports 3000 and 4600 listening on the box, 5432 only on 127.0.0.1.

**3.8b Front door** — copy `halle-feedback.conf` and `halle-feedback-le-ssl.conf` from the old server to the host, change `http://127.0.0.1:3000/` → `http://10.10.0.20:3000/`, enable, `configtest`, reload.

**3.9 Test before switching** — `/etc/hosts` on the Mac for `feedback.arametrics.app`, then:
- [ ] `curl -s https://feedback.arametrics.app/v1.js | grep -c localhost:3000` → 0
- [ ] `curl -s https://feedback.arametrics.app/v1.js | grep -o '#[0-9a-f]\{6\}' | sort -u` → only the five brand greens + `#2a2924 #ebebeb #d93025 #5c5a52 #000000 #0f171b`
- [ ] Admin login; the queue shows the same reports as the old server; an old report's screenshot shows
- [ ] On `halle-dev.webflow.io`, send a test report with a screenshot; it arrives; the renderer log shows `ok`
- [ ] `node --experimental-strip-types -e "const x: number = 1; console.log(x)"` inside the box prints `1` (the `make user-*` scripts now work — don't run a real one as a test)
- [ ] **Separation:** from the Jupyter box, `curl -s -m 3 http://10.10.0.20:5432` → blocked/refused; from the web app box, `curl -s -m 3 http://10.10.0.10:5432` → blocked/refused
- [ ] Reboot test, as in 2.9

**3.10 Switch-over** — same shape as 2.10: stop `halle-feedback` (old and new box), fresh local backup of `halle_feedback` + screenshots, final copy + checks, start, Cloudflare `feedback.arametrics.app` A → `217.160.93.75` (DNS-only, grey cloud), re-test publicly, `certbot renew --cert-name feedback.arametrics.app --dry-run`, disable on old.

Production `projects.config` still holds the old `thanksSub` subtitle — it moves with the database; clear it in admin Wording as before.

---

## Part 4 — After both switch-overs

- **Day 1:** `new 'free -m; machinectl list'`, each box's logs (`journalctl -M jupyter -u jupyterhub --since today`, `journalctl -M webapp -u halle-feedback --since today`).
- **Day 14:** everything clean → tell Jakob the old server can go. **He must cancel the M+ contract (111321277) himself, after the new contract's 30-day money-back window has ended** (IONOS's condition).
- Before cancelling: one last archive of the old server into the backup folder: `old 'tar -C / -czf - --one-file-system --anchored --exclude=./proc --exclude=./sys --exclude=./dev --exclude=./run --exclude=./tmp .' > ~/araCreate/HLE/server/old-server-final.tgz`.
- Update `deploy/runbook.md` for the box layout and the new IP.
- Delete the local backups 30 days after cancellation.

## Traps already known

- **Root-owned files** in the app tree break later `git pull`/`npm` as `halle-feedback` → always `sudo -u halle-feedback -H`, and `chown -R` after any root copy.
- **The widget build must have** `WIDGET_API_ORIGIN=https://feedback.arametrics.app`.
- **A check straight after a restart** can hit a service before it's up — wait 15 s.
- **Paste one line at a time** into Terminal; a multi-line paste after an `ssh` line loses everything after the first line. (The `new '…' <<'EOF'` blocks are the exception — paste each whole block, from `new` to `EOF`.)
- **Never run `netplan apply`** on the new server; don't touch `ens6`.
- **Running `git status` as root** in the app tree rewrites `.git/index` as root-owned.
- **`systemd-run` blanks out anything starting with `$` in its arguments** (it treats it as a variable). Password hashes (`$y$…`) and anything else with `$` must go into a box through **stdin** (`… | systemd-run … chpasswd -e`, or inside an `inbox` script) — never as a command-line argument. Found 28 Sept in step 2.3.
- **Files are copied server to server, not through the Mac** (decided 28 Sept after a 23-minute copy through the Mac died on a Wi-Fi drop). A temporary key on the new server (`/root/.ssh/migrate_old`, comment `(secret removed)`) is allowed on the old server **only from 217.160.93.75**, no forwarding, no terminal. The old server's original key file is saved as `/root/.ssh/authorized_keys.before-migration`. **Remove the key after the last switch-over:** `old 'cp -p /root/.ssh/authorized_keys.before-migration /root/.ssh/authorized_keys'`. The old server has no `rsync`, so the new server pulls with `tar` per folder (`/root/copy-jupyter.sh`, 25 s for all 20 folders).
- **On the old server, `sites-enabled/000-default*.conf` are real files, not links, and differ from `sites-available`** (enabled: `/api/` → `localhost:9000`; available: stale `localhost:4000`). Always copy Apache site files from **`sites-enabled`**. Found 28 Sept in step 2.8.
- **The old server's PM2 start list is stale.** `/root/.pm2/dump.pm2` (saved 18 May) points to `/root/github/halle-nodejs-server/…`, which no longer exists, so after a reboot the old server would **not** bring the product API back. In the box it's started from the project's own `ecosystem.config.js` and saved (`pm2 start ecosystem.config.js && pm2 save --force`). Re-do this after the final copy in 2.10, because the copy brings the stale `dump.pm2` back.
- **JupyterHub also needs Debian's own Python libraries** (installed by apt on the old server, not pip): Jinja2, MarkupSafe, PyJWT, oauthlib, cryptography, requests and ~35 more. Step 2.2b installs them in the box; `python3 -m pip check` must give the same output on both (one harmless `pygobject … pycairo` line). Only host tools (certbot, cloud-init, reportbug, unattended-upgrades, apt helpers, gyp) are left out.
- **`POST /product/products` returns rows in no fixed order** (its query has no `ORDER BY`). The content is identical old vs box (32 of 32 categories), but the order differs, because the database chooses a different join. The old server's order isn't guaranteed either. See "Open question: product order".
- **The jupyterhub.sqlite** lives in two places (`/etc/jupyterhub/` and `/shared/notebooks/`) — both are copied.
