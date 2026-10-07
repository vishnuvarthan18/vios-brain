# Getting started — viOS v1.0

> **Just want it on your Mac first?** Skip this page: `bash mac/install.sh --with-stack`, then open
> http://app.localhost:8088 (see `local/README.md`). This page is for the cloud server (Option B).

About 45 minutes. Do the steps in order.

## 0. What you need to buy or create (I can't do these for you)

| Item | Where | Cost |
|---|---|---|
| Server: Ubuntu 24.04, 2 vCPU, 4 GB RAM | Hetzner (CX32) or any VPS | ~€7–10 / month |
| Domain (or a subdomain you own) | any registrar | ~€10–15 / year |
| AI keys (at least one) | console.anthropic.com · platform.openai.com · aistudio.google.com · openrouter.ai · groq.com | pay per use |
| Telegram bot | @BotFather in Telegram → `/newbot` → copy token | free |
| Your Telegram user ID | @userinfobot in Telegram | free |
| Off-site backup (recommended) | any S3 / Backblaze B2 / sftp | ~€1–4 / month |
| Authenticator app | Aegis / 2FAS / any TOTP app | free |

## 1. DNS (5 min)

Point a wildcard to the server IP: `*.vios.yourdomain.com → <server IP>`
(or separate A records: app, mcp, chat, notes, tasks, ai, auth).

## 2. Server (15 min)

On your Mac, in this repo:
```bash
server/scripts/make-bundle.sh
scp server/dist/vios-server-*.tar.gz root@<server-ip>:
```
Tip: pass `--telegram-token` to install.sh and step 4 is done automatically.

On the server:
```bash
tar xzf vios-server-*.tar.gz
sudo ./vios/server/install.sh
```
- It asks: domain, email, username (`vishnu`), API keys (you can skip and add later).
- At the end it prints your password + next steps. Save the password in KeePassXC.
- Then add keys any time: `vios keys add anthropic` (also openai, gemini, openrouter, groq).

## 3. First login (5 min)

1. Open `https://app.vios.yourdomain.com` → log in → set up 2FA (`vios user code` on the server shows the code).
2. On your phone: open the same link → Share → **Add to Home Screen**. That's your viOS app.
3. `https://chat.…` → **Log in with viOS** → choose model `vios-smart` → the `vios` tools are already connected.

## 4. Telegram bot (5 min)

On the server:
```bash
sudo nano /srv/vios/.env
#   (secret removed):ABC...
#   TELEGRAM_ALLOWED_USERS=<your user id>
#   TELEGRAM_HOME_CHANNEL=<your user id>
#   COMPOSE_PROFILES=bots
sudo vios up                                      # builds + starts the bot (server/hermes/README.md)
```
Say "hi" to your bot, then "what's on today?". WhatsApp (optional, QR pairing): `server/hermes/README.md` §5.

## 5. Mac (15 min)

```bash
bash mac/install.sh --server <server-ip> --domain vios.yourdomain.com
```
- Your old `~/own/viOS` (v0.1) is moved to a backup folder, never deleted.
- Clones the vault, installs LifeOS (viOS edition), links the 12 vi-* skills, connects Claude Code / Claude Desktop / Cursor / OpenCode to your viOS MCP, and turns on auto-sync every 5 min.
- Private zone: `bash ~/own/viOS/setup/private-zone.sh` → choose Cryptomator.

## 6. Connect other AI apps (optional)

- **Claude.ai**: Settings → Connectors → Add custom connector → `https://mcp.vios.yourdomain.com/mcp` → log in with viOS.
- **ChatGPT**: Settings → Connectors → Developer mode → add the same URL (plan support varies).
- Any tool that takes an OpenAI-style endpoint: `https://ai.vios.yourdomain.com/v1` + a LiteLLM key (make one at `https://ai.…/ui`).

## Daily use

- Phone app: capture, today, projects, tasks · Chat: any model with your memory · Telegram: talk to viOS
- Any AI: **"resume <project>"** at the start, **"handoff"** at the end
- Server health: `vios status` · `vios doctor` · backups run nightly (`vios snapshots`)

## First week plan
1. Fill `core/` (CORE, USER, GOALS, VALUES) — in notes.… or on the Mac.
2. Add your top 3 projects and your existing AI agents.
3. Use resume/handoff every day. Read the 07:45 brief on Telegram.
