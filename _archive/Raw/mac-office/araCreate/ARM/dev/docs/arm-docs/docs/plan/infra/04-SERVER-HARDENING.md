# 04 — Server Hardening

> Applied to every server (S1-S4) after first boot. Ubuntu 24.04 LTS.

---

## 4.1 System Updates + Auto-Patches

```bash
apt update && apt upgrade -y
apt install -y unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades
```

## 4.2 Deploy User (Not Root)

```bash
useradd -m -s /bin/bash deploy
mkdir -p /home/deploy/.ssh
cp /root/.ssh/authorized_keys /home/deploy/.ssh/
chown -R deploy:deploy /home/deploy/.ssh
chmod 700 /home/deploy/.ssh && chmod 600 /home/deploy/.ssh/authorized_keys
usermod -aG docker deploy
```

## 4.3 SSH Hardening

`/etc/ssh/sshd_config`:
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers deploy
X11Forwarding no
AllowTcpForwarding no
```

## 4.4 Kernel Hardening

`/etc/sysctl.d/99-arm-hardening.conf`:
```
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv6.conf.all.disable_ipv6 = 1
kernel.shmmax = (removed)
vm.swappiness = 10
```

## 4.5 Swap (4 GB, All Servers)

```bash
fallocate -l 4G /swapfile && chmod 600 /swapfile
mkswap /swapfile && swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

## 4.6 Fail2Ban

```bash
apt install -y fail2ban
```
`/etc/fail2ban/jail.local`: ban IP for 1 hour after 3 failed SSH attempts in 10 min.

## 4.7 Time Sync

```bash
apt install -y chrony && systemctl enable chrony
```

Critical for Kafka, JWT expiry, TLS certs, log timestamps.

## 4.8 Remove Unnecessary Packages

```bash
apt purge -y telnet rsh-client rsh-server && apt autoremove -y
```

## 4.9 Docker Daemon Hardening

`/etc/docker/daemon.json`:
```json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "50m", "max-file": "3" },
  "storage-driver": "overlay2",
  "live-restore": true,
  "no-new-privileges": true,
  "icc": false,
  "default-ulimits": { "nofile": { "Name": "nofile", "Soft": 65536, "Hard": 65536 } }
}
```

- `live-restore` — containers survive daemon restarts
- `no-new-privileges` — prevents privilege escalation
- `icc: false` — containers can't talk unless linked by networks

## 4.10 Docker Socket

Only mount where necessary:

| Server | Socket needed? | Why |
|---|---|---|
| S1 | No | Databases don't need it |
| S2 | Only Alloy | Log collection |
| S3 | Alloy + cAdvisor | No public-facing services — acceptable risk |
| S4 | No | No containers |

---

## Verification

```bash
ssh root@<ip>              # rejected
ssh deploy@<ip>            # with password → rejected
ufw status verbose         # correct rules, default deny
sysctl net.ipv4.conf.all.rp_filter  # returns 1
swapon --show              # 4G
fail2ban-client status sshd # jail active
chronyc tracking           # synchronized
docker info                # no-new-privileges: true
```
