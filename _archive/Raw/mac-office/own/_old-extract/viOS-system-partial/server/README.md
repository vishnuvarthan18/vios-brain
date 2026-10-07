# viOS server

One small VPS runs the online part of viOS. All open source.

- **app** — viOS dashboard (PWA) + API · **mcp** — remote MCP for any AI app
- **chat** — LibreChat · **notes** — SilverBullet (edit the vault) · **tasks** — Backlog.md board
- **ai** — LiteLLM gateway (all models, one key, cost tracking) · **auth** — Authelia (login + 2FA)
- Hermes bot (Telegram / WhatsApp), optional
- Caddy in front: HTTPS for every subdomain, automatic

## Requirements

- Ubuntu 24.04 VPS, 2 vCPU, **4 GB RAM**, 40 GB disk (e.g. Hetzner CX32).
- A domain you control (e.g. `vios.example.com`).
- Root SSH access. Ports 22, 80, 443 open.
- Optional: API keys (Anthropic, OpenAI, Gemini, OpenRouter, Groq), a Telegram bot token.

## DNS

Point these names to the server IP (A record; AAAA too if you use IPv6):

```
app  mcp  chat  notes  tasks  ai  auth   .vios.example.com  ->  <server IP>
```

Or one wildcard: `*.vios.example.com -> <server IP>`. Do this before install so certificates work at once.

## Install

On your computer:

```
server/scripts/make-bundle.sh            # -> server/dist/vios-server-<version>.tar.gz
scp server/dist/vios-server-*.tar.gz root@<server>:
```

On the server:

```
tar xzf vios-server-*.tar.gz
sudo ./vios/server/install.sh            # asks domain, emails, keys
# or without questions:
sudo ./vios/server/install.sh -y --domain vios.example.com --email you@example.com \
  --user vishnu --user-email you@example.com --ssh-key-file mac.pub --anthropic-key sk-ant-...
./vios/server/install.sh --dry-run ...   # show every step, change nothing
```

What it does:

- Docker (official repo), ufw (22/80/443), fail2ban, unattended upgrades, 2 GB swap.
- User `vios` (git over SSH only). Vault: bare repo `/srv/vios/vault.git` + working copy `/srv/vios/vault`.
- All secrets in `/srv/vios/.env` (chmod 600). Authelia keys, users, LiteLLM keys for LibreChat + Hermes.
- Starts the stack, waits for health, creates your LibreChat account, prints DNS + next steps.
- Safe to re-run: keeps secrets, vault, users. Code goes to `/opt/vios`, CLI to `/usr/local/bin/vios`.

Connect the Mac vault: `git remote add server ssh://vios@<server>/srv/vios/vault.git`.

## First login

1. Open `https://app.<domain>`. Log in as your user with the password install.sh printed
   (also in `/root/vios-credentials.txt` — delete it after saving the password).
2. Authelia asks for 2FA. It "sends" a code — there is no mail server, so run `vios user code` on the server.
3. Scan the QR code with an authenticator app. Done: one login for app, notes, tasks and ai.
4. Chat: `https://chat.<domain>` → **Log in with viOS** (same account). The vault is the MCP server `vios`.
5. AI apps (Claude.ai, ChatGPT): add a remote MCP connector `https://mcp.<domain>/mcp` (OAuth login via Authelia).
   Terminal agents: use `Authorization: Bearer $VIOS_API_TOKEN` (`grep VIOS_API_TOKEN /srv/vios/.env`).
6. Models: use the aliases `vios-smart`, `vios-fast`, `vios-gpt`, `vios-gemini`, `vios-open`, `vios-local`
   at `https://ai.<domain>/v1` with a LiteLLM key (make one in the UI at `https://ai.<domain>/ui`).

## Daily use: `vios`

```
vios status | vios doctor | vios logs litellm | vios restart librechat
vios keys add anthropic        # add/replace an AI key (restarts LiteLLM)
vios user passwd | vios user reset-2fa
vios update                    # pull pinned images, rebuild, recreate, prune
vios backup | vios snapshots | vios restore [id]
```

Backups: nightly (systemd `vios-backup.timer`, 03:30 ± 30 min) with restic: vault, vault.git, `.env`,
Authelia, MongoDB dump, Postgres dump. Keeps 7 daily / 4 weekly / 6 monthly.
**Set `RESTIC_REPOSITORY` to off-site storage** (S3, B2, sftp) in `/srv/vios/.env`; the default is on the same disk.

## Costs (approx., check current prices)

- VPS 4 GB (Hetzner CX32 class): about €7–10 / month.
- Domain: about €10–15 / year. Off-site backup storage: €1–4 / month.
- AI usage: pay per token to each provider. LiteLLM tracks spend; defaults: gateway cap $100 / 30 days,
  LibreChat key $50, Hermes key $20 (change in `litellm/config.yaml` or the LiteLLM UI).
- All software: free (open source).

## Memory plan (4 GB)

Limits: LiteLLM 900 MB, LibreChat 700 MB, MongoDB 400 MB, Hermes 512 MB, Postgres / Meilisearch /
SilverBullet / Backlog / vios-api 256 MB each, Caddy / Authelia 128 MB. Typical use is well below the limits.

## Troubleshooting

- **Run `vios doctor` first.** It checks secrets, vault, containers, disk, DNS, HTTPS, backups.
- No certificate: DNS not pointing here yet, or port 80/443 blocked. `vios logs caddy`.
- Login loop: clock wrong (TOTP) → `timedatectl`; cookie domain must be your `VIOS_DOMAIN`.
- Lost 2FA device: `vios user reset-2fa`. Forgot password: `vios user passwd`.
- "Log in with viOS" fails in LibreChat: `vios logs librechat` and `vios logs authelia`; the Authelia user
  email must match the LibreChat account email.
- Model errors: `vios keys list`; add the missing key with `vios keys add <provider>`.
- A container restarts: `vios logs <service>`; low memory → `free -h`, stop Hermes (`COMPOSE_PROFILES=''`).
- Vault conflicts: vios-api keeps both versions and writes a note in `ai/inbox/`.

## Files

```
docker-compose.yml   services, pinned versions, health checks, memory limits, networks
caddy/Caddyfile      subdomains, HTTPS, Authelia forward-auth, security headers
authelia/            configuration.yml (template), users_database.yml.tmpl
litellm/config.yaml  model aliases, fallbacks, budgets
librechat/           librechat.yaml (LiteLLM endpoint + vios MCP); its env is in docker-compose.yml
backlog/             tiny image for the Backlog.md web board
install.sh           installer · bin/vios CLI · systemd/ backup timer · scripts/make-bundle.sh
tests/               run_all.sh (compose, shellcheck, caddy, authelia, litellm, librechat schema, installer)
```

Security notes: only Caddy publishes ports; databases sit on internal networks; identity headers from
clients are always stripped; `mcp.` is not behind Authelia (vios-api does OAuth / bearer itself);
the private zone never exists on the server (install.sh refuses it).
