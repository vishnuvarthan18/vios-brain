# Sources — what viOS took from where

viOS is assembled from proven open-source work. Ideas and patterns were adapted and rewritten for viOS
(no large code copies). Studied at the commits below on 2026-09-30.

## Patterns and skills (studied, adapted)

| Project | License | Commit | What viOS took |
|---|---|---|---|
| [nrjn — "How I Work" gist](https://gist.github.com/nrjn/2511690a253503e8d9b35cd4c5520666) | — (idea) | — | Starting idea: vault + small MCP tools + skills + app connectors; project ↔ repo link; meeting→tasks flow |
| [Karpathy — LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) | — (idea) | — | `raw/` vs `wiki/`, ingest / query / lint → vi-ingest, vi-query, vi-lint |
| [eugeniughelbur/obsidian-second-brain](https://github.com/eugeniughelbur/obsidian-second-brain) | MIT | b0089f7 | "For future agent" preamble, AI-first note rules, propagation rule, dated facts, scheduled morning/nightly/weekly agents |
| [brkakyldz/second-brain-os](https://github.com/brkakyldz/second-brain-os) | MIT | baa8d39 | `core/` tier loaded every session, git as the only database, `.agents/skills` single copy + links, closeout → vi-handoff, `CLAUDE.md` = `@AGENTS.md` |
| [mattbuildz/obsidian-life-os](https://github.com/mattbuildz/obsidian-life-os) | MIT | 2f81657 | Zones with write contracts, human-owned areas, "pattern only after 3 times", options-then-Vishnu-chooses |
| [anthropics/cwc-long-running-agents](https://github.com/anthropics/cwc-long-running-agents) | Apache-2.0 | ad107a9 | Read progress + git log + smoke test at start; one task per session; commit at end → vi-resume / vi-handoff |
| [huytieu/COG-second-brain](https://github.com/huytieu/COG-second-brain) | MIT | 4cdb601 | People folder, "worker never grades its own homework", AGENTS.md as universal fallback |
| [AgriciDaniel/claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) | MIT | 32ac5a0 | Wiki ingest/lint with citations, verifier idea |
| [danielmiessler/Personal_AI_Infrastructure](https://github.com/danielmiessler/Personal_AI_Infrastructure) | MIT | 5e2f2e8 | TELOS goals → `core/GOALS.md`; "scaffolding > model" |
| [davidhariri/life-system](https://github.com/davidhariri/life-system) | no license → **ideas only** | 3538cb4 | Plan / goals / values / decision records read before planning |
| [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) · [kepano "file over app"](https://stephango.com/file-over-app) | MIT | 3ccff53 | Plain files outlive apps; "don't delegate understanding" |
| [ar9av/obsidian-wiki](https://github.com/ar9av/obsidian-wiki), [heyitsnoah/claudesidian](https://github.com/heyitsnoah/claudesidian) | MIT | 59fa4b2, 6c56f35 | "compile, don't accumulate"; PARA starter layout |
| OpenAI [PLANS.md / ExecPlans](https://developers.openai.com/cookbook/articles/codex_exec_plans) | docs | — | Decision log + surprises in handoff |
| Simon Willison [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/), Meta "Rule of Two" | docs | — | Agent limits in the registry |

## Tools viOS runs (installed, not copied)

| Tool | License | Role |
|---|---|---|
| [Backlog.md](https://github.com/MrLesk/Backlog.md) (69e7b15) | MIT | Task board in `ai/tasks/`, CLI + web + MCP |
| [Basic Memory](https://github.com/basicmachines-co/basic-memory) (88c3990) | AGPL-3.0 | MCP memory for chat apps, over `ai/` |
| [qmd](https://github.com/tobi/qmd) (04e4dbd) | MIT | Local hybrid/semantic search, MCP |
| [Logseq](https://github.com/logseq/logseq) | AGPL-3.0 | Viewer: graph, journals |
| [OpenCode](https://github.com/sst/opencode) | MIT | Agent runner (any model) |
| [goose](https://github.com/block/goose) | Apache-2.0 | Agent runner |
| [Ollama](https://github.com/ollama/ollama) | MIT | Local open models |
| [Cryptomator](https://github.com/cryptomator/cryptomator) | GPL-3.0 | Private zone encryption |
| [KeePassXC](https://github.com/keepassxreboot/keepassxc) | GPL-3.0 | Passwords & keys outside viOS |

Rejected: Obsidian (not open source), Task Master AI (Commons Clause), n8n (fair-code), Llama models (restricted license).

## v1.0 additions (server + engine)

| Project | License | Role in viOS |
|---|---|---|
| [LifeOS](https://github.com/danielmiessler/LifeOS) @ 5e2f2e8 (7.40.4) | MIT | Engine on the Mac (fetched pinned at install; `lifeos-overlay/`) |
| [LiteLLM](https://github.com/BerriAI/litellm) v1.103.1 | MIT (core) | AI gateway |
| [LibreChat](https://github.com/danny-avila/LibreChat) v0.8.7 | MIT | Web/phone chat |
| [SilverBullet](https://github.com/silverbulletmd/silverbullet) 2.11.1 | MIT | Vault editor in the browser |
| [Hermes Agent](https://github.com/NousResearch/hermes-agent) v2026.9.24 | MIT | Telegram / WhatsApp + cron |
| [FastMCP](https://github.com/jlowin/fastmcp) 4.0.10 · MCP Python SDK | Apache-2.0 / MIT | Remote MCP server |
| [Caddy](https://github.com/caddyserver/caddy) 2.11.4 | Apache-2.0 | HTTPS reverse proxy |
| [Authelia](https://github.com/authelia/authelia) 4.39.28 | Apache-2.0 | SSO, 2FA, OIDC |
| [restic](https://github.com/restic/restic) | BSD-2 | Backups |
| MongoDB 8.0 (SSPL — LibreChat dependency), PostgreSQL 17, Meilisearch 1.35 (MIT) | mixed | Databases |

Note: MongoDB's SSPL is source-available, not OSI-approved. It is required by LibreChat. Swap option later: FerretDB (Apache-2.0).
