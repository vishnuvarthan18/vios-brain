# vios-api

viOS REST API + remote MCP server + PWA, over the viOS vault (Markdown + Git).
Python 3.12 · FastAPI · FastMCP 4 (streamable HTTP) · SQLite FTS5. MIT.

## What it does
- `/api/*` — REST contract from `SPEC.md` (dashboard, projects, handoff, tasks, capture, search, notes, daily, agents).
  Extra: `GET /api/config` (user + app links, used by the PWA), `GET /api/list?path=`, `POST /api/sync`, docs at `/api/docs`.
- `/mcp` — MCP tools `vios_resume`, `vios_handoff`, `vios_capture`, `vios_search`, `vios_read`, `vios_write`, `vios_list`,
  `vios_tasks`, `vios_task_create`, `vios_task_update`, `vios_today`, `vios_project_create`, `vios_dashboard`.
- `/` — PWA (dashboard, project + handoff, tasks, capture with offline queue, search, today note). No build step.
- Zones enforced in one place (`vios_api/zones.py`): write only under `ai/`, `core/` read-only, existing
  `ai/knowledge/raw/` files read-only, any `private` / `viOS-Private` segment refused, no `..`, no symlink escape,
  notes with `zone: private` refused, secrets refused.
- Every write = one git commit, author `viOS <vios@localhost>`, trailer `Agent: <client>`.
- Background: search index refresh (by mtime) + git sync loop (`add -A` → commit → `pull --rebase` → `push`).
  Conflict → merge, keep ours in place, save theirs as `<file>.conflict-<time>-remote.md`, note in `ai/inbox/INBOX.md`.
- Tasks are Backlog.md files (`ai/tasks/tasks/vi-<N> - <Title>.md`); verified with `backlog` CLI 1.53.

## Auth
- REST: `Remote-User` header is trusted **only** if the TCP peer is in `VIOS_TRUSTED_PROXIES` (set this to Caddy's
  IP/subnet). Otherwise `Authorization: Bearer $VIOS_API_TOKEN`. `X-Forwarded-For` is ignored (uvicorn `proxy_headers=False`).
- MCP: bearer `VIOS_API_TOKEN` always. OAuth 2.1 for Claude.ai / ChatGPT connectors when `VIOS_OIDC_CLIENT_SECRET` +
  issuer + public URL are set (FastMCP `OIDCProxy` → Authelia, DCR supported, id_token checked, only `VIOS_ALLOWED_USERS`).
- Authelia client for MCP (builder B): id `vios-mcp`, confidential, secret `VIOS_OIDC_CLIENT_SECRET`,
  redirect URI `https://mcp.$VIOS_DOMAIN/auth/callback`, scopes `openid profile email offline_access`,
  grant types `authorization_code refresh_token`, `token_endpoint_auth_method: client_secret_basic`.
- Caddy for `mcp.$VIOS_DOMAIN`: **no** forward-auth (OAuth/bearer are checked by the app); proxy the whole host to
  `vios-api:8080` (needs `/mcp`, `/.well-known/*`, `/authorize`, `/token`, `/register`, `/revoke`, `/consent`, `/auth/callback`).

## Env vars
| Var | Default | Use |
|---|---|---|
| `VIOS_VAULT` | `/vault` | vault git working copy |
| `VIOS_API_TOKEN` | — | bearer token (REST + MCP) |
| `VIOS_DOMAIN` | — | builds app links + default OIDC issuer / MCP URL |
| `VIOS_TRUSTED_PROXIES` | `172.16.0.0/12,10.0.0.0/8,192.168.0.0/16` | peers allowed to set `Remote-User` |
| `VIOS_ALLOWED_USERS` | `vishnu` | users allowed via `Remote-User` and OAuth |
| `VIOS_SYNC_INTERVAL` | `60` | git sync seconds (`0` = off) |
| `VIOS_GIT_REMOTE` | — | added as `origin` if the vault has none |
| `VIOS_OIDC_ISSUER` | `https://auth.$VIOS_DOMAIN` | Authelia issuer |
| `VIOS_OIDC_CLIENT_ID` | `vios-mcp` | |
| `VIOS_OIDC_CLIENT_SECRET` | — | turns MCP OAuth on |
| `VIOS_PUBLIC_MCP_URL` | `https://mcp.$VIOS_DOMAIN` | public base of the MCP server |
| `VIOS_JWT_SIGNING_KEY` | derived from client secret | signs FastMCP-issued tokens |
| `VIOS_DATA_DIR` | `/data` | search DB (`search.db`); keep on a volume |
| `FASTMCP_HOME` | `/data/fastmcp` (Docker) | OAuth client/token store (encrypted) |
| `VIOS_TZ` / `TZ` | `UTC` | dates for daily notes, `as_of` |
| `VIOS_INDEX_INTERVAL` | `30` | full index rescan seconds |
| `VIOS_LINK_CHAT/NOTES/TASKS/AI` | from domain | override PWA links |
| `VIOS_HOST` / `VIOS_PORT` / `VIOS_LOG_LEVEL` | `0.0.0.0` / `8080` / `INFO` | server |

## Run
```bash
# local
python3.12 -m venv .venv && .venv/bin/pip install --require-hashes -r requirements.lock
VIOS_VAULT=~/viOS VIOS_DATA_DIR=/tmp/vios-data VIOS_API_TOKEN=change-me .venv/bin/python -m vios_api

# docker (vault mounted at /vault, uid 1000 must be able to write it; build arg VIOS_UID to change)
docker build -t vios-api .
docker run -p 8080:8080 -v /srv/vios/vault:/vault -v vios-data:/data -e VIOS_API_TOKEN=... vios-api
```
MCP client config (bearer): URL `https://mcp.$VIOS_DOMAIN/mcp`, header `Authorization: Bearer $VIOS_API_TOKEN`.

## Test
```bash
.venv/bin/pip install pytest==9.1.1 pytest-asyncio==1.4.0 httpx==0.28.1
npm i -g backlog.md   # optional: enables the Backlog.md CLI compatibility test
.venv/bin/python -m pytest -q     # uses a temp git copy of ../../vault-template
```
