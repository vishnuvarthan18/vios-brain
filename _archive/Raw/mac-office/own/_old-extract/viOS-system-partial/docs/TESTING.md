# What was tested (2026-10-01)

Build sandbox limits: Docker registries (Docker Hub, ghcr, quay) blocked → **no container was built or run**.
Every component was tested natively instead, with the real binaries/packages.

| Part | Command | Result |
|---|---|---|
| vios-api (zones, auth, tasks, handoff, capture, search, MCP, OAuth wiring, git sync) | `cd services/vios-api && pytest -q` | 92 passed |
| PWA | Playwright/Chromium: all views, mobile+desktop, light+dark, offline capture | pass, screenshots reviewed |
| Server stack | `server/tests/run_all.sh` (compose config, shellcheck, caddy, authelia validate + live SSO/2FA/OIDC flow, LiteLLM live proxy + fallback, LibreChat schema, installer + CLI sandbox) | 7/7 groups, 113 checks |
| Hermes bot | `server/hermes/tests/run_tests.sh` (config, tool lock-down, MCP, skills, CLI chat, cron) | 61 passed |
| LifeOS overlay | `lifeos-overlay/tests/run-linux-test.sh` (real pinned LifeOS install, links, hooks, skills) | 29 passed |
| Mac installer | `mac/tests/run-mac-sim-test.sh` (simulated macOS) | 28 passed |
| End-to-end | Hermes (real) → vios-api (real) MCP → daily note created + git commit | pass |

## Not tested yet (first real deploy will show)
- Docker images build/run, real HTTPS certificates, real Telegram/WhatsApp, real Claude.ai/ChatGPT OAuth connector flow.
- LiteLLM with Postgres (spend DB, key generation), LibreChat + MongoDB runtime, SilverBullet runtime.
- Real Mac: Homebrew casks, Keychain, launchd, Claude Desktop/Cursor loading MCP.

## After deploy — smoke test (10 min)
1. `vios doctor` → all green. 2. `curl https://app.<domain>/api/health`.
3. PWA: capture a note → appears in `ai/inbox/INBOX.md` on the Mac within 5 min.
4. Chat: `vios-smart` → "resume vios" → it calls vios_resume.
5. Telegram: "what's on today?" → reply + daily note.
6. Claude.ai connector → OAuth login → `vios_dashboard` works.
7. `vios backup && vios snapshots`.
