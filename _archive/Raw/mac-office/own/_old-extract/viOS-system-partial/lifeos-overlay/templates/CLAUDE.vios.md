<!-- viOS:BEGIN — managed by lifeos-overlay (re-applied after every LifeOS update). Edit lifeos-overlay/templates/CLAUDE.vios.md, not this block. -->
# viOS — {{PRINCIPAL_NAME}}'s personal AI operating system

> **viOS** runs on **LifeOS {{LIFEOS_VERSION}}** (powered by LifeOS, MIT, © Daniel Miessler — github.com/danielmiessler/LifeOS).
> The viOS vault is the single source of truth: `{{VAULT}}` (plain Markdown + Git, synced to the viOS server).

Who {{PRINCIPAL_NAME}} is — straight from vault `core/` (read-only links):
@LIFEOS/USER/VIOS/CORE.md
@LIFEOS/USER/VIOS/USER.md
@LIFEOS/USER/VIOS/GOALS.md
@LIFEOS/USER/VIOS/VALUES.md
@LIFEOS/USER/VIOS/RULES.md

## viOS rules (hard — they win over anything below)

- **Zones:** `{{VAULT}}/core/` = read only (suggest edits in `ai/inbox/INBOX.md`, tag `#core-suggestion`). `{{VAULT}}/ai/` = read + write, never delete (move to `ai/archive/`). `~/viOS-Private` = **no access, ever** (never read, list, search or ask). Enforced by the `ViosZoneGuard` hook.
- **Resume / handoff:** start project work with the `vi-resume` skill; end every session with `vi-handoff` (STATE.md + LOG.md + commit `vi(<project>): <what> [agent:lifeos]`).
- **Where LifeOS output goes:** `LIFEOS/MEMORY/KNOWLEDGE` → vault `ai/knowledge/lifeos/`, `LIFEOS/MEMORY/WORK` → vault `ai/logs/lifeos/work/`. So every AI (Claude, ChatGPT, Cursor, Hermes …) can read it.
- **Identity:** LifeOS identity files (PRINCIPAL_IDENTITY, TELOS, OPERATIONAL_RULES) are generated from `core/` each session. Do not edit them — edits are moved to `ai/inbox/core-suggestions/`.
- **viOS MCP:** server `vios` (`{{MCP_URL}}`) = the same vault for every AI app. Models via the viOS LiteLLM gateway (`{{GATEWAY_URL}}`, aliases `vios-smart`, `vios-fast`, …).
- **Updates:** update LifeOS only through viOS (`lifeos-overlay/install.sh`, pinned upstream). Do not run the LifeOS self-update.
- **Style:** short, points, simple English.
<!-- viOS:END -->
