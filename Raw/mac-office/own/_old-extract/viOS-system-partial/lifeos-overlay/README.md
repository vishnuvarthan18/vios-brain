# lifeos-overlay — viOS on top of LifeOS

viOS uses **LifeOS** (github.com/danielmiessler/LifeOS, MIT, © Daniel Miessler) as its engine on the Mac.
This folder is the thin viOS layer. **Powered by LifeOS.** Upstream license: `upstream/LICENSE`.

## Run

```bash
bash lifeos-overlay/install.sh --domain example.com            # install or update
bash lifeos-overlay/install.sh --domain example.com --dry-run  # show plan only
vios                                                           # start LifeOS + viOS rules in ~/own/viOS
```
`mac/install.sh` calls this for you.

## How upstream is kept (and why)

- **Fetched at install time, pinned** to commit `5e2f2e8c…` (LifeOS 7.40.4), see `UPSTREAM`.
- Why not vendored: 25 MB / ~2000 files would bloat the viOS repo; a pinned git fetch is
  content-verified (git object hashes = the commit SHA), needs no checksums, and an upgrade
  is a one-line change you can review with `git diff <old>..<new> -- LifeOS/`.
- Offline: `--src /path/to/LifeOS-checkout`.
- Upgrade: edit `UPSTREAM` → run `tests/run-linux-test.sh` → run `install.sh` (it runs the
  upstream `OverlaySystem` update path once, then re-applies the viOS layer).
- Do not use the LifeOS self-update (it follows the *latest* release, not the pin).

## What the overlay adds

| Area | What | Where |
|---|---|---|
| Identity | `core/` stays the one source of truth. Raw read-only links `LIFEOS/USER/VIOS/{CORE,USER,GOALS,VALUES,RULES}.md` → vault `core/`, @-imported in CLAUDE.md | `hooks/lib/vios-config.ts` |
| | LifeOS files that LifeOS tools parse (PRINCIPAL_IDENTITY, TELOS, PRINCIPAL_TELOS, OPERATIONAL_RULES) are **generated** from `core/` every session start. Edits to them (AI or LifeOS memory loop) go to `ai/inbox/core-suggestions/`, never to `core/` | `hooks/ViosCoreSync.hook.ts` |
| Memory | `LIFEOS/MEMORY/KNOWLEDGE` → `ai/knowledge/lifeos/`, `LIFEOS/MEMORY/WORK` → `ai/logs/lifeos/work/` (symlinks; old dirs kept in `~/.local/state/vios/backups/`) | `Tools/ViosOverlay.ts` |
| Skills | 12 `vi-*` skills linked into `~/.claude/skills` | `Tools/ViosOverlay.ts` |
| Zones | PreToolUse hook blocks Edit/Write/Bash-writes to `core/` (also through symlinks) and any tool touching `~/viOS-Private` (+ Cryptomator vault, disk image). Same rules in `permissions.deny` | `hooks/ViosZoneGuard.hook.ts` |
| Rules | viOS block at top of `~/.claude/CLAUDE.md`; `LIFEOS/VIOS_SYSTEM_PROMPT.md` = upstream constitution + viOS addendum (used by `vios`) | `templates/` |
| Remote | `settings.json` → `vios` options block + `env.VIOS_*`; LifeOS MCP profile `~/.claude/MCPs/vios-MCP.json` (`lifeos -m vios`, bearer from `$VIOS_API_TOKEN`) | `Tools/ViosOverlay.ts` |
| Gateway | Optional: route Claude Code via LiteLLM: `bun Tools/ViosOverlay.ts gateway on --apply` (sets `ANTHROPIC_BASE_URL` + `apiKeyHelper` = `vios-secret litellm`; key stays in Keychain) | |
| Branding | "viOS · powered by LifeOS" in CLAUDE.md, DA identity (DA name `viOS`), launcher | |

## Rules it follows

- Upstream files are never edited in the payload. Only viOS-owned files are added
  (LifeOS updates never touch them); CLAUDE.md / settings.json get idempotent, marked patches.
- LifeOS conventions: bun TS tools, JSON reports, dry-run unless `--apply`, backups before writes,
  hooks read stdin once and exit 2 to block.
- Settings layer: if `settings.system.json` + `USER/CONFIG/settings.user.json` exist, viOS entries
  also go into the user overlay (so the SessionStart merge keeps them).
- Never deletes; old files/dirs are renamed aside with a timestamp.

## Tools

```bash
bun Tools/ViosOverlay.ts apply  [--apply]   # viOS layer only (after upstream tools)
bun Tools/ViosOverlay.ts verify             # checks: links, generated files, hooks, guard probes
bun Tools/ViosOverlay.ts sync-core --apply  # regenerate identity from core/ now
bun Tools/ViosOverlay.ts gateway on|off --apply
```

Config: `~/.config/vios/vios.json` · logs: `~/.local/state/vios/lifeos-install/`.
Test: `bash tests/run-linux-test.sh /tmp/builderD` (fake HOME, no real `~/.claude` touched).
