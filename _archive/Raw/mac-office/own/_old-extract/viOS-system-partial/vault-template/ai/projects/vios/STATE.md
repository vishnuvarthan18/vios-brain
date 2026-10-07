---
title: vios/STATE
type: state
zone: ai
project: vios
as_of: 2026-10-01
updated_by: claude-opus (cowork, v1.0 build)
status: active
---
## For future agent
Current state of [[vios]]. READ FIRST when resuming. True as of the date above.

# STATE — vios

## Now
- viOS v1.0 built (2026-10-01): server stack + vios-api/PWA/MCP + Hermes bots + LifeOS overlay + Mac installer.
- Tested here: 92 vios-api tests, 113 server checks, 61 Hermes checks, 57 LifeOS/Mac checks, Hermes → vios-api end-to-end.
- Not yet deployed: needs a server, domain and API keys from Vishnu.

## Next (in order)
1. Vishnu: buy VPS (Ubuntu 24.04, 4 GB) + point DNS (see docs/GETTING-STARTED.md in the viOS repo).
2. Run `sudo ./vios/server/install.sh` on the server; log in at app.<domain>; set up 2FA.
3. Add API keys: `vios keys add anthropic` (and others).
4. Telegram: create bot with @BotFather, set TELEGRAM_BOT_TOKEN + TELEGRAM_ALLOWED_USERS, `vios restart hermes`.
5. Mac: `bash mac/install.sh --server <ip> --domain <domain>` (clones vault, installs LifeOS overlay, MCP, auto-sync).
6. Fill core/ files; add top 3 projects; connect Claude.ai / ChatGPT to https://mcp.<domain>/mcp.

## Blockers / questions for Vishnu
- Server provider + domain name to use?
- Which AI provider keys do you have (Anthropic, OpenAI, Gemini, OpenRouter, Groq)?

## How to run / test
- Server: `vios status`, `vios doctor` · API: `curl https://app.<domain>/api/health`
- Local tests: see docs/TESTING.md in the viOS repo

## Key files
- viOS repo: SPEC.md, server/, services/vios-api/, server/hermes/, lifeos-overlay/, mac/, docs/
