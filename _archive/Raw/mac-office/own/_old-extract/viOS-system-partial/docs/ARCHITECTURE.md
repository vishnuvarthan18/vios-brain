# viOS architecture (v1.0)

## Layers
| Layer | Part | Open-source project (license) |
|---|---|---|
| Data | Vault: Markdown + Git, 3 zones | git |
| Engine (Mac) | LifeOS viOS edition: TELOS, Algorithm, memory, skills, hooks | LifeOS (MIT) + `lifeos-overlay/` |
| Rules | AGENTS.md, core/RULES.md, zone guards (API, hooks, git hook, Claude/OpenCode permissions) | open standards |
| Skills | 12 vi-* skills + 56 LifeOS skills | Agent Skills (SKILL.md) |
| API + MCP | vios-api: REST, remote MCP (13 tools), PWA, git sync, search | FastAPI, FastMCP (Apache-2.0) |
| AI gateway | One endpoint for all models, keys, budgets, fallbacks | LiteLLM (MIT) |
| Chat | Web/phone chat with any model + vios tools | LibreChat (MIT) |
| Notes | Edit the vault in the browser | SilverBullet (MIT) |
| Tasks | Kanban over `ai/tasks` | Backlog.md (MIT) |
| Bots + cron | Telegram, WhatsApp, 07:45 / 23:30 / Fri jobs | Hermes Agent (MIT) |
| Edge + identity | HTTPS, SSO, TOTP 2FA, OIDC for LibreChat + MCP OAuth | Caddy (Apache-2.0), Authelia (Apache-2.0) |
| Ops | `vios` CLI, nightly backups, firewall, fail2ban | restic (BSD-2), ufw |

## Data flow
- **Write from anywhere** (PWA, chat, bot, Claude.ai, Mac agent) → vios-api (zones checked) → file + git commit (`Agent:` trailer).
- **Sync**: server working copy ↔ bare repo (every 60 s) ↔ Mac `~/own/viOS` (every 5 min). Conflicts: both versions kept + inbox note; Mac pushes a side branch and pauses.
- **Models**: every client → LiteLLM (`vios-smart` → fallback `vios-gpt` → `vios-gemini`). Spend tracked per key.

## Security model
- Private zone: Mac only, encrypted (Cryptomator). Server refuses any `private` path.
- core/: read-only for every AI (API 403, LifeOS hook, git pre-commit, Claude/OpenCode deny rules).
- Logins: Authelia password + TOTP; one identity for app/notes/tasks/ai/chat/MCP OAuth. Bearer tokens for machine clients.
- vios-api trusts `Remote-User` only from Caddy's fixed IP (172.30.0.10).
- Hermes (reads untrusted chat) has no file/shell/web tools, vault only via MCP, allow-listed users only (Rule of Two).
- Databases on internal networks without internet. Only Caddy exposes ports. Secrets only in `/srv/vios/.env` (600).

## Known limits
- Scheduled briefs arrive via Telegram; the PWA has no push notifications yet.
- LibreChat has its own chat history store (MongoDB), not in the vault. Save important answers with the vios tools.
- WhatsApp uses an unofficial bridge (small ban risk) — use a separate number.
- Upstream versions are pinned; upgrade by bumping pins and re-running tests.
