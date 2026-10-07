---
title: vios/STATE
type: state
zone: ai
project: vios
as_of: 2026-10-01
updated_by: claude-opus (cowork, local install session)
status: active
---
## For future agent
Current state of [[vios]]. READ FIRST when resuming. True as of the date above.

# STATE — vios

## Now
- **Paused (2026-10-01):** everything stopped by Vishnu. Restart: `colima start && cd ~/own/viOS-system && local/bin/vios-local up`.
- **viOS runs locally on Vishnu's Mac** (since 2026-10-01). All 9 services healthy in Docker (Colima).
  - Dashboard http://app.localhost:8088 · Chat http://chat.localhost:8088 · Notes http://notes.localhost:8088
  - Tasks http://tasks.localhost:8088 · AI gateway http://ai.localhost:8088/ui · MCP http://mcp.localhost:8088/mcp
- Code: `~/own/viOS-system` (v1.2 + fixes). Vault: `~/own/viOS` (git, auto-commit every 5 min via launchd).
- Mac side done: LifeOS viOS edition (`vios` command), 12 vi-* skills linked, MCP added to Claude Code / Claude Desktop / Cursor / OpenCode, Logseq + Cryptomator + KeePassXC installed.
- LibreChat account created (vishnu@aracreate.group) via temporary sign-up.
- Vishnu has started using the dashboard (moved VI-1, created today's note).
- **No AI API key yet** → chat models fail until a key is added.

## Next (in order)
1. Get a free Gemini key (aistudio.google.com/apikey) → `local/bin/vios-local keys add gemini` → use model `vios-gemini`.
2. Turn sign-up off again: `sed -i '' 's/ALLOW_REGISTRATION: "true"/ALLOW_REGISTRATION: "false"/' local/docker-compose.yml && local/bin/vios-local up`.
3. In chat: enable MCP Servers → vios → "show my viOS dashboard" (test memory).
4. Run `vios` (LifeOS) → "resume vios" → "handoff" (test any-AI-continues).
5. Fill core/CORE.md + core/GOALS.md (Vishnu, own words).
6. Private zone: `vios-private` → Cryptomator.
7. Fix OpenCode: `npm approve-scripts opencode-ai && npm install -g opencode-ai`.
8. Add first real project: bootcamp-evaluation (~/araCreate/bootcamp-evaluation).
9. Later: Telegram bot (local profile `bots`), phone access (Tailscale or cloud server), Ollama for free local model.

## Blockers / questions for Vishnu
- Needs one AI API key (Gemini free tier recommended) for chat/Telegram.
- Cloud server later? (provider + domain)

## Known issues to fix in the code (`~/own/viOS-system`)
- LibreChat `npm run create-user` exits silently in v0.8.7 → setup.sh can't create the chat user (workaround: temporary ALLOW_REGISTRATION).
- `mac/install.sh` asks again about Basic Memory / Private zone on re-runs — should remember answers.

## How to run / test
- `local/bin/vios-local status | logs <svc> | up | down | doctor`
- `docker logs --tail 40 vios-local-api` if the API restarts

## Key files
- `~/own/viOS-system/local/.env` (secrets, 600) · `local/credentials.txt` · `README.md` · `local/README.md`
