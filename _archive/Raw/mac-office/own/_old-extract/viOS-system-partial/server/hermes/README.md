# viOS — Hermes (Telegram, WhatsApp, cron)

[Hermes Agent](https://github.com/NousResearch/hermes-agent) (Nous Research, MIT) is viOS's
**messaging + automation layer**. You chat with viOS on Telegram or WhatsApp. It also runs the
scheduled jobs (morning brief, nightly lint, weekly review).

- Pinned version: **Hermes v2026.9.24 (v0.21.5)**, commit `f97608f178d1ffeca59860195ab7da295f7c8e5f`.
- Brain: LiteLLM at `http://litellm:4000/v1`, model `vios-smart` (side tasks: `vios-fast`).
- Vault access: **only** through the viOS MCP server `http://vios-api:8080/mcp` (bearer `VIOS_API_TOKEN`).
- Skills: the vault's `vi-*` skills, mounted read-only.

## Files

| File | What it does |
|---|---|
| `Dockerfile` | Two stages. Clones the pinned tag, checks the commit hash, `uv sync --frozen` (upstream `uv.lock`), pre-installs the WhatsApp bridge. Runs as uid 10000, tini as PID 1. |
| `entrypoint.sh` | Checks env, writes `$HERMES_HOME/.env` (0600) and `config.yaml`, installs `SOUL.md`, syncs cron jobs, starts the gateway. Runs on every start. |
| `config/config.yaml.tmpl` | Hermes config. `@@NAME@@` = filled from non-secret env. `${NAME}` = Hermes reads it from `.env` at runtime. |
| `config/SOUL.md` | The "viOS assistant" persona (Hermes system-prompt slot #1). |
| `config/cron-jobs.json` | The 3 scheduled jobs. Edit here, restart = applied. |
| `scripts/sync_cron.py` | Adds or updates those jobs by name, using `hermes cron create/edit/pause`. |
| `scripts/healthcheck.py` | Docker healthcheck: gateway pid alive and state `running`. |
| `tests/` | Mock LiteLLM + mock viOS MCP + `run_tests.sh` (61 checks, run natively). |

## Environment

Pass **only** these to the container (do not use `env_file: .env` for the whole stack file).

| Var | Needed | Notes |
|---|---|---|
| `VIOS_API_TOKEN` | yes | Bearer for viOS MCP. |
| `VIOS_LLM_API_KEY` | yes | A **LiteLLM virtual key** for Hermes only (budget + model limits). If it is missing, falls back to `LITELLM_MASTER_KEY` with a warning. Do not do that in production. |
| `TELEGRAM_BOT_TOKEN` | for Telegram | From @BotFather. |
| `TELEGRAM_ALLOWED_USERS` | with Telegram | Your numeric Telegram user ID(s), comma-separated. `*` is refused. |
| `TELEGRAM_HOME_CHANNEL` | no | Where cron briefs go. Default: first allowed user (your DM). |
| `WHATSAPP_ENABLED` | no | `true` to use WhatsApp. |
| `WHATSAPP_ALLOWED_USERS` | with WhatsApp | Phone numbers with country code, digits only (e.g. `9198xxxxxxxx`). |
| `WHATSAPP_MODE` | no | `bot` (default: separate number) or `self-chat`. |
| `WHATSAPP_HOME_CHANNEL` | no | WhatsApp chat ID for deliveries (optional). |
| `VIOS_TIMEZONE` | no | Default `Asia/Kolkata` (cron times are in this zone). |
| `VIOS_MODEL_SMART` / `VIOS_MODEL_FAST` | no | Default `vios-smart` / `vios-fast`. |
| `VIOS_LLM_BASE_URL` / `VIOS_MCP_URL` | no | Defaults `http://litellm:4000/v1` / `http://vios-api:8080/mcp`. |
| `VIOS_CRON_SYNC` | no | `false` = do not touch cron jobs on start. |

The entrypoint **refuses to start** if: an allow-list is empty or `*`, any `*_ALLOW_ALL_USERS=true`,
`VIOS_API_TOKEN` or the LLM key is missing, or a value could inject extra config lines.

## Setup

### 1. Telegram bot
1. In Telegram open **@BotFather** → send `/newbot`.
2. Pick a name (e.g. `viOS`) and a username ending in `bot` (e.g. `vishnu_vios_bot`).
3. Copy the token (`123456789:ABC...`) → `TELEGRAM_BOT_TOKEN` in `/srv/vios/.env`.
4. Optional: `/setprivacy` → Enable (bot only sees commands/mentions in groups). viOS is DM-only anyway.

### 2. Your Telegram user ID
1. Message **@userinfobot** (or @get_id_bot). It replies with a number like `123456789`.
2. Put it in `TELEGRAM_ALLOWED_USERS=123456789`. This is a number, not your @username.

### 3. LiteLLM virtual key for Hermes
Create a key allowed to use `vios-smart` and `vios-fast` only (LiteLLM admin UI → Virtual Keys,
or `POST /key/generate` with the master key), then set `VIOS_LLM_API_KEY`.

### 4. Start and check
```bash
docker compose up -d hermes
docker compose logs -f hermes              # "starting Hermes gateway (telegram=true ...)"
docker compose exec hermes hermes mcp test vios   # "Tools discovered: N"
docker compose exec hermes hermes cron list       # vi-morning / vi-nightly / vi-weekly
```
Send your bot "hi". Then "what's on today?" — it should call `vios_today`.

### 5. WhatsApp (optional, pairs by QR code)
WhatsApp uses the Baileys bridge (WhatsApp Web emulation, **unofficial**, small ban risk).
Use a **separate number** for the bot if you can (`WHATSAPP_MODE=bot`); `self-chat` uses your own number
(you message yourself).

1. Set `WHATSAPP_ENABLED=true`, `WHATSAPP_ALLOWED_USERS=<your number, digits only>`, restart.
   The log says "WhatsApp is not paired yet; starting without it" — that is expected. Telegram keeps running.
2. Pair (needs a terminal ≥ 60 columns):
   ```bash
   docker compose exec -it hermes hermes whatsapp
   ```
   Pick the mode, confirm the allowed number, a **QR code** appears.
3. On the phone that owns the bot number: WhatsApp → **Settings → Linked devices → Link a device** → scan.
   QR codes refresh about every 20 s; rerun the command if it expires.
4. `docker compose restart hermes`. Log shows `whatsapp=true` and no "not paired" warning.
5. Session is saved in the `hermes-data` volume (`platforms/whatsapp/session` or `whatsapp/session`).
   It is a full login to that WhatsApp account: never copy or commit it. To unlink: phone → Linked devices.

Why the gate: Hermes stops the **whole** gateway when WhatsApp is on but unpaired. The entrypoint only
switches WhatsApp on (and adds its config block) once `creds.json` exists.

## Scheduled jobs (built-in Hermes cron)

Defined in `config/cron-jobs.json`, times in `Asia/Kolkata`, delivered to Telegram (`TELEGRAM_HOME_CHANNEL`):

| Job | When | Skill | Output |
|---|---|---|---|
| `vi-morning` | daily 07:45 | `vi-today` | Top 3, blockers, 1 thing to drop → Telegram. Writes the daily note via MCP. |
| `vi-nightly` | daily 23:30 | `vi-lint` | Safe fixes in `ai/`, report in `ai/logs/runs/<date>-lint.md`, short summary → Telegram. |
| `vi-weekly` | Fri 18:00 | `vi-review` | `ai/daily/<YYYY-Www>-review.md`, summary → Telegram. |

- Change a job: edit the JSON, restart. Matching is by `name`; your own chat-created reminders are never touched.
- Pause: `docker compose exec hermes hermes cron pause <id>` (stays paused across restarts), or `"enabled": false` in the JSON.
- Run now: `docker compose exec hermes hermes cron run <id>`.
- Cron runs use `vios-smart`, may use only skills + todo + viOS MCP, and deny anything that needs approval.

## Rule of Two — what Hermes can and cannot do

Hermes reads **untrusted input** (chat messages, forwarded text). So it gets **no** raw private data
access and **no** way to act outward except replying in the chat.

| Boundary | How it is enforced (verified) |
|---|---|
| Who can talk to it | `TELEGRAM_ALLOWED_USERS` / `WHATSAPP_ALLOWED_USERS` (Hermes default = deny all). `unauthorized_dm_behavior: ignore` (strangers get silence, no pairing codes). WhatsApp groups: `WHATSAPP_GROUP_POLICY=disabled`. Allow-all refused by entrypoint. |
| Tools | `agent.disabled_toolsets` removes terminal, file, code_execution, browser, web, search, vision, image/video/tts, delegation, computer_use, homeassistant, kanban, connections **on every surface**. Per-surface allow-list `platform_toolsets`: skills, memory, todo, clarify, session_search, cronjob (chat only), `vios` MCP. `hermes tools list --platform <p>` confirms. |
| Vault | Only `vios_*` MCP tools (vios-api enforces zones: core/ read-only, private refused). The container mounts **only** `/vault/.agents/skills` read-only — not the vault. `HERMES_WRITE_SAFE_ROOT=/vault/ai` (not mounted) as a second lock on file writes. |
| Skills | External dir read-only; `skills.write_approval: true` (any skill write waits for your `/skills approve`); `inline_shell: false`; curator + background self-review off. |
| Network | Hermes talks to: Telegram API, WhatsApp servers (bridge), `litellm`, `vios-api`. Web/browser tools are off. Update checks, model catalog, telemetry, Nous tool connectors, runtime `pip` installs, tirith download: off. Hermes still fetches public model metadata from models.dev (no user data). For hard egress control, put the container behind an HTTPS proxy allow-list (Hermes honours `HTTPS_PROXY`). |
| Secrets | Only in `$HERMES_HOME/.env` (0600). `config.yaml` has `${VAR}` references only. Entrypoint removes secrets and `TELEGRAM_*`/`WHATSAPP_*` from the process env before starting Hermes. MCP stdio env filtering is irrelevant (no stdio servers). `security.redact_secrets: true`. |
| Outward actions | No send-message tool exists; the bot can only reply to you. SOUL.md: ask before anything goes outward; cron runs never act outward. Approvals `manual`, cron/unattended `deny`. |
| Updates | Image carries `/etc/hermes/image-provenance.json`, so `hermes update` and in-chat `/update` refuse. Upgrade = bump the pin and rebuild. |

## docker-compose (for builder B)

```yaml
  hermes:
    build: ./hermes
    image: vios/hermes:v2026.9.24
    container_name: hermes
    restart: unless-stopped
    networks: [vios]
    depends_on: [litellm, vios-api]
    environment:
      VIOS_API_TOKEN: ${VIOS_API_TOKEN}
      VIOS_LLM_API_KEY: ${HERMES_LITELLM_KEY:-}      # virtual key; falls back to master key if empty
      LITELLM_MASTER_KEY: ${LITELLM_MASTER_KEY}      # only used as the fallback above
      TELEGRAM_BOT_TOKEN: ${TELEGRAM_BOT_TOKEN}
      TELEGRAM_ALLOWED_USERS: ${TELEGRAM_ALLOWED_USERS}
      WHATSAPP_ENABLED: ${WHATSAPP_ENABLED:-false}
      WHATSAPP_ALLOWED_USERS: ${WHATSAPP_ALLOWED_USERS:-}
      VIOS_TIMEZONE: Asia/Kolkata
    volumes:
      - hermes-data:/opt/data
      - /srv/vios/vault/.agents/skills:/vault/.agents/skills:ro
    security_opt: [no-new-privileges:true]
    cap_drop: [ALL]
    mem_limit: 1g
    cpus: 1.0
    pids_limit: 256
    logging: { driver: json-file, options: { max-size: 10m, max-file: "3" } }
volumes:
  hermes-data:
```
No ports: Hermes is outbound only. `HERMES_LITELLM_KEY` is not in the SPEC env list yet; add it to
`install.sh` (generate a LiteLLM virtual key) or leave it empty to use the fallback.

## Operations

- Logs: `docker compose logs hermes`, or inside the volume `logs/gateway.log`, `logs/agent.log` (secrets redacted).
- Health: `docker compose exec hermes hermes cron status` · `docker inspect --format '{{.State.Health.Status}}' hermes`.
- Full check (renders config, then `config check`, `mcp list`, `skills list`, `doctor`):
  `docker compose run --rm hermes check`. Doctor warnings about Nous/Codex/xAI auth, OpenRouter,
  boto3 and "no API key in .env" are expected (we use a custom provider via `key_env`).
- Bad bot token → Hermes exits with code 78 ("token rejected") and Docker restarts it. Fix the token.
- Memory: Hermes keeps a small memory (`memories/MEMORY.md`, `USER.md`) for chat preferences.
  Real facts go to the vault via `vios_capture`.
- Upgrade Hermes: change `HERMES_REF`/`HERMES_SHA` in the Dockerfile, check config keys against the new
  `hermes_cli/config_defaults.py`, bump `_config_version`, run the tests, rebuild.

## Test natively (what builder C ran)

```bash
git clone --depth 1 --branch v2026.9.24 https://github.com/NousResearch/hermes-agent.git /tmp/builderC/hermes-src
python3 -m venv /tmp/builderC/uvtool && /tmp/builderC/uvtool/bin/pip install uv==0.11.6
cd /tmp/builderC/hermes-src
UV_PROJECT_ENVIRONMENT=/tmp/builderC/venv /tmp/builderC/uvtool/bin/uv sync --frozen --no-dev \
  --no-install-project --extra messaging --extra mcp -p python3.13
VIRTUAL_ENV=/tmp/builderC/venv /tmp/builderC/uvtool/bin/uv pip install --no-deps -e .
/tmp/builderC/uvtool/bin/uv venv -p python3.12 /tmp/builderC/mockenv
VIRTUAL_ENV=/tmp/builderC/mockenv /tmp/builderC/uvtool/bin/uv pip install "mcp>=1.20,<2" fastapi uvicorn
bash server/hermes/tests/run_tests.sh      # passed: 61  failed: 0
```
