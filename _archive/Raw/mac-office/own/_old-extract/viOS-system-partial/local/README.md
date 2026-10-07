# viOS local — the whole viOS stack on your Mac

No VPS. No domain. Everything runs in Docker on your Mac. Only this Mac can open it (127.0.0.1).

Same apps as the server: dashboard + API + MCP, LibreChat, SilverBullet, Backlog.md board,
LiteLLM (all models), Hermes bot (optional). No Authelia: being on this Mac *is* the login.

## Requirements

- Mac with Apple Silicon (M1 or newer), macOS 14+.
- 16 GB RAM recommended (Docker gets 6 GB). About 15 GB free disk (images + data).
- A Docker runtime: OrbStack, Docker Desktop or Colima. None? `setup.sh` installs **Colima** (open source)
  with Homebrew.
- Git (`xcode-select --install`). Optional: API keys (Anthropic, OpenAI, Gemini, OpenRouter, Groq), Ollama.

## Start (one command)

```
./local/setup.sh
```

- Finds or starts Docker (Colima: 4 CPU, 6 GB, 60 GB disk, `vz` + `virtiofs`, home folder writable).
- Vault: uses `~/own/viOS`. Missing? Made from `vault-template` (git, first commit).
  Already there? Kept as is; only missing folders are added. Nothing is ever deleted.
- Writes `local/.env` (secrets, chmod 600). Asks for API keys (Enter = skip).
- Builds + starts everything, waits until healthy, creates your LibreChat login.
- Prints the URLs. Passwords go to `local/credentials.txt` (chmod 600).

Options: `-y` (no questions), `--anthropic-key K`, `--openai-key K`, `--vault DIR`, `--port N`,
`--telegram-token T --telegram-users ID`, `--dry-run` (show steps, change nothing). Safe to run again.

## URLs

| App | URL |
|---|---|
| Dashboard (PWA + API) | http://app.localhost:8088 |
| Chat (LibreChat) | http://chat.localhost:8088 — email `vishnu@aracreate.group`, password in `local/credentials.txt` |
| Notes (SilverBullet) | http://notes.localhost:8088 |
| Tasks (Backlog.md) | http://tasks.localhost:8088 |
| AI gateway (LiteLLM) | http://ai.localhost:8088/ui — user `admin`, password = (secret removed) in `local/.env` |
| MCP | http://mcp.localhost:8088/mcp — bearer token |

`*.localhost` always points to your Mac. No DNS setup needed.

## Connect AI apps (MCP)

Token: (secret removed) token`

- **Claude Code**:
  `claude mcp add --transport http vios http://mcp.localhost:8088/mcp --header "Authorization: Bearer <TOKEN>"`
- **Cursor** (`~/.cursor/mcp.json`):
  `{"mcpServers": {"vios": {"url": "http://mcp.localhost:8088/mcp", "headers": {"Authorization": "Bearer <TOKEN>"}}}}`
- **Claude Desktop** (Settings → Developer → Edit Config), via the `mcp-remote` bridge:
  `{"mcpServers": {"vios": {"command": "npx", "args": ["-y", "mcp-remote", "http://mcp.localhost:8088/mcp", "--allow-http", "--header", "Authorization: Bearer <TOKEN>"]}}}`
- Models for any OpenAI-compatible app: base URL `http://ai.localhost:8088/v1`, a LiteLLM key, model `vios-smart`
  (or `vios-fast`, `vios-gpt`, `vios-gemini`, `vios-open`, `vios-local`).
- Ollama on the Mac works as `vios-local` (reached at `host.docker.internal:11434`).

If an app cannot resolve `mcp.localhost`, use `http://127.0.0.1:8088/mcp` with header `Host: mcp.localhost:8088`,
or add `127.0.0.1 mcp.localhost` to `/etc/hosts`.

## Daily use

```
local/bin/vios-local status        # containers + URLs
local/bin/vios-local open chat     # open in browser
local/bin/vios-local logs litellm
local/bin/vios-local keys add anthropic
local/bin/vios-local update        # new pinned images, rebuild
local/bin/vios-local backup        # restic -> ~/viOS-backups (vault + .env + chat/LiteLLM databases)
local/bin/vios-local doctor
```

Tip: `ln -s "$PWD/local/bin/vios-local" /opt/homebrew/bin/vios-local` to use it from anywhere.

## Stop

- `local/bin/vios-local down` — stops everything. Data stays (Docker volumes + your vault folder).
- Colima users: `colima stop` frees the RAM. Start again: `colima start`, then `local/bin/vios-local up`.

## From your phone (later, optional)

- Easiest: install **Tailscale** on the Mac and the phone. Then use `tailscale serve` to share port 8088
  inside your tailnet only. Keep the port bound to 127.0.0.1 — do not open it to the LAN/internet
  (the local mode has no login screen for app/notes/tasks).
- For real remote access with login + 2FA, use the server version (`server/`).

## Good to know

- The vault on your Mac is the source of truth. vios-api commits edits locally (author `viOS`) and never
  pushes. Your own git sync (e.g. to the server) keeps working as before.
- Databases (chat, LiteLLM spend) are Docker volumes: `vios-local_mongo-data`, `vios-local_litellm-pg`, ...
  These scripts never delete volumes or vault files.
- Memory limits add up to ~4.3 GB (with Hermes); a 6 GB Docker VM is enough.
