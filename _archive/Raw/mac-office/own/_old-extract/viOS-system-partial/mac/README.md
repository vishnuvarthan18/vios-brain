# viOS on the Mac

One command sets up viOS on this Mac (macOS, Apple Silicon, zsh). Powered by LifeOS.

```bash
bash mac/install.sh                      # LOCAL mode (default): whole stack on this Mac
bash mac/install.sh --with-stack         # … and also run local/setup.sh (docker stack)
bash mac/install.sh --server <ssh-host> --domain <domain>   # REMOTE mode (VPS)
bash mac/install.sh --dry-run            # plan only, changes nothing
```
`--yes` = say yes to all prompts. Safe to run again.

## Local mode (default, no `--server`)

- No SSH, no clone. Vault = `~/own/viOS`:
  - missing → created from `vault-template` + `git init` + first commit.
  - v0.1 vault there (`AGENTS.md` + `core/` + `ai/`) → **kept in place**; only missing
    files/folders are added from `vault-template` (nothing overwritten; the list is printed).
  - some other folder there → moved to `~/own/viOS-backup-<date>` (never deleted).
- MCP in Claude Code, Claude Desktop, Cursor, OpenCode → `http://mcp.localhost:8088/mcp`,
  bearer = `VIOS_API_TOKEN` from `<repo>/local/.env` (or `--token`). No token yet → MCP is
  added after `--with-stack` creates it, or on the next run.
- Gateway (LiteLLM) → `http://ai.localhost:8088` (`--litellm-key` adds it to OpenCode;
  `--gateway on` routes Claude Code through it). `--local-port` changes 8088.
- Auto-sync = commit only every 5 min (no remote). The pre-commit hook still blocks
  secrets/private paths. Add a remote later (`git remote add origin …`) → the same agent
  starts pull/push automatically.
- `--with-stack` runs `bash <repo>/local/setup.sh` at the end (skipped with a note if missing).
- Stack not running → MCP/health checks are warnings, not errors.

## Remote mode — what it does

1. Checks macOS + finds Homebrew (`~/.homebrew`, `/opt/homebrew`, `/usr/local`).
2. Installs: git, node, bun, uv, jq (brew) · backlog.md, opencode, Claude Code (npm, pinned) ·
   basic-memory (optional, `--with-basic-memory`) · Logseq, Cryptomator, KeePassXC (asks).
3. SSH key `~/.ssh/vios_ed25519` + `~/.ssh/config` entry. Shows the public key to add on the server.
4. Clones `ssh://vios@<server>/srv/vios/vault.git` → `~/own/viOS`.
   An old v0.1 `~/own/viOS` is **moved** to `~/own/viOS-v0.1-backup-<date>` (never deleted).
5. Stores the token in the Keychain (+ `~/.config/vios/secrets/`, mode 600). Never in git.
6. Installs LifeOS + the viOS overlay (`lifeos-overlay/install.sh`).
7. Links the 12 `vi-*` skills into `~/.claude/skills`, `~/.config/opencode/skills`, `~/.agents/skills`.
8. Adds the remote viOS MCP (`https://mcp.<domain>/mcp`, bearer) to Claude Code, Claude Desktop
   (via `mcp-remote`), Cursor, OpenCode. With `--litellm-key`: OpenCode gets the viOS gateway models.
9. Auto-sync: launchd agent `group.aracreate.vios.sync`, every 5 min.
10. Private zone helper `vios-private` (reuses `setup/private-zone.sh`).
11. Final check (tools, remote, skills, MCP entries, launchd, LifeOS verify, server health).

## Daily use

- `vios` — start LifeOS with viOS rules, in the vault.
- `vios-sync` — sync now. Log: `~/Library/Logs/vios-sync.log`.
- Conflict? Sync pauses, your work is pushed to a `mac-conflict-<time>` branch on the server.
  Fix: `cd ~/own/viOS && git pull --rebase`, resolve, then `vios-sync --clear`.
- A commit with a secret is blocked by the pre-commit hook → sync waits until you remove it.
- Claude Code via the gateway: `bun lifeos-overlay/Tools/ViosOverlay.ts gateway on --apply`.

## Test (Linux, simulated Mac)

`bash mac/tests/run-mac-sim-test.sh /tmp/builderD` — stubbed uname/brew/launchctl, local bare repo as
server; covers remote mode and local mode (fresh vault, v0.1 merge, localhost MCP, commit-only sync, --with-stack).
