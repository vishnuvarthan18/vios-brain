# Connect any AI to viOS

Run `setup/install-mac.sh` first. viOS only shares the **AI zone** (`~/own/viOS/ai`) and reads `core/`.
The Private zone (`~/viOS-Private`) is never connected.

## Coding / agent tools (open the viOS folder)

| Tool | How | Already set up in viOS |
|---|---|---|
| **OpenCode** (open source) | `cd ~/own/viOS && opencode` | `opencode.json` (rules, permissions, MCP), `.opencode/skills` |
| **goose** (open source) | `cd ~/own/viOS && goose session` | add `setup/mcp/goose-extensions.yaml` to `~/.config/goose/config.yaml` |
| **Claude Code** | `cd ~/own/viOS && claude` | `CLAUDE.md` → `AGENTS.md`, `.claude/settings.json` (zone locks), `.mcp.json`, `.claude/skills` |
| **Cursor** | Open folder `~/own/viOS` | `AGENTS.md` (read natively), `.cursor/mcp.json` |
| **Kiro / Cline / Trae / Copilot** | Open folder `~/own/viOS` | They read `AGENTS.md`. Add MCP servers from `.mcp.json` in their settings. |

First thing to type in any of them: **`resume vios`**

## Working inside another code repo

Use `vi-new-project <name> --repo <path>` — it adds a "viOS link" block to that repo's `AGENTS.md`.
Any AI in that repo will then read/write the project's STATE and LOG in viOS.
(User-level skill links from the install script make `vi-resume` / `vi-handoff` work in every repo.)

## Chat apps (no file access) — via MCP

| App | How |
|---|---|
| **Claude Desktop** | Merge `setup/mcp/claude-desktop.json` into `~/Library/Application Support/Claude/claude_desktop_config.json`, restart |
| **ChatGPT** | Needs a *remote* MCP server (developer mode). Option: `basic-memory mcp --transport streamable-http --project vios` behind a secure tunnel. **Not recommended yet** — only do it with auth. |
| **Local model (Ollama)** | `opencode` → `/models` → choose an Ollama model (e.g. `qwen3:8b`). Fully private. |

## MCP servers viOS uses

| Server | What it gives the AI | Scope |
|---|---|---|
| `basic-memory mcp --project vios` | read / write / search notes, knowledge graph | `ai/` only |
| `backlog mcp start` | task board (create, update, list tasks) | `ai/tasks/` |
| `qmd mcp` | search by meaning | `ai/` only |

## Safety checklist when adding any new AI or MCP server

- [ ] It is open source or from a trusted vendor, and I read what it does
- [ ] It sees only `~/own/viOS` (never `~/viOS-Private`)
- [ ] Rule of Two: not all three of (reads untrusted web/email, sees private data, can send out)
- [ ] Added to `ai/agents/REGISTRY.md` if it runs on its own
