# viOS v1.0 — Vishnu's personal AI operating system

One brain for all your projects, AI agents and (non-private) life. Reach it from anywhere, use any AI.
100% open source. Built on proven projects: engine = **LifeOS** (ourlifeos.ai, MIT).

## What you get

| Where you are | How you use viOS |
|---|---|
| Phone / any browser | **app.** dashboard PWA (install to home screen) · **chat.** chat with any AI (LibreChat) · **notes.** edit your vault (SilverBullet) · **tasks.** task board |
| Telegram / WhatsApp | Message the viOS bot: capture, ask, "what's today", run agents. Morning brief 07:45, nightly clean-up 23:30, Friday review |
| Claude.ai / ChatGPT / Cursor / any MCP app | Add connector `https://mcp.<domain>/mcp` (OAuth login) → your memory, projects, tasks |
| Mac terminal | **LifeOS** (viOS edition) in Claude Code + OpenCode / Cursor / goose — same vault, same skills |
| Any AI model | One gateway (LiteLLM): Claude, OpenAI, Gemini, OpenRouter, Groq, local Ollama — `vios-smart`, `vios-fast`, … with cost tracking + fallbacks |

## How it fits together

```
 Phone/Web ──► Caddy (HTTPS) ──► Authelia (login + 2FA, OIDC)
                   │
   app./api ──► vios-api ◄── mcp. (OAuth/bearer) ◄── Claude.ai, ChatGPT, Cursor, LibreChat, Hermes
   chat.    ──► LibreChat ──┐
   notes.   ──► SilverBullet│        all models
   tasks.   ──► Backlog.md  ├──► LiteLLM ──► Anthropic · OpenAI · Gemini · OpenRouter · Groq · Ollama
   Telegram ──► Hermes ─────┘
                   │
              /srv/vios/vault  (Markdown + Git)  ◄── git sync ──►  Mac ~/own/viOS  (+ LifeOS, Private zone)
```

## Zones (your privacy)
- `core/` — who you are, goals, values, rules. Only you edit. AI reads.
- `ai/` — projects, tasks, agents, knowledge, daily. AI reads + writes.
- `~/viOS-Private` — **Mac only, encrypted, never on the server, never seen by any AI.**

## Start

**Option A — run everything on your Mac (no server, no domain):**
```bash
cd ~/own
tar xzf viOS-v1.2.tar.gz
cd viOS-system
bash mac/install.sh --with-stack
```
The code lives in `~/own/viOS-system`, your vault in `~/own/viOS`. Then open http://app.localhost:8088 · details: `local/README.md`

**Option B — cloud server (reach it from anywhere):** `docs/GETTING-STARTED.md` (~45 min).
You can start with A and move to B later — same vault, just add a git remote.

## Repo map
| Folder | What |
|---|---|
| `local/` | Whole stack on your Mac (Docker, *.localhost:8088), `vios-local` CLI |
| `server/` | Cloud stack, installer, `vios` CLI, backups (see `server/README.md`) |
| `services/vios-api/` | viOS API + remote MCP + PWA dashboard |
| `server/hermes/` | Telegram / WhatsApp bot + scheduled agents |
| `lifeos-overlay/` | LifeOS fork layer (viOS branding, vault wiring, zone guard) |
| `mac/` | Mac installer: vault clone, LifeOS, MCP config, auto-sync |
| `vault-template/` | Starting vault (AGENTS.md, core/, ai/, 12 vi-* skills, templates, tools) |
| `docs/` | Getting started, architecture, testing, sources |
| `SPEC.md` | The contract all parts follow |
