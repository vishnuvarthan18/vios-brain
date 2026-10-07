# Running the Realism Engine unattended

## Start and stop

Start (from any terminal; survives closing it):

```
cd ~/Downloads/tamil_harvest && nohup design/realism/run_forever.sh >> design/realism/state/supervisor.out 2>&1 &
```

Stop (the running task finishes its current step, puts itself back to `pending`, REPORT.md is rewritten):

```
touch ~/Downloads/tamil_harvest/design/realism/state/STOP
```

Start again later: remove the STOP file (`rm ~/Downloads/tamil_harvest/design/realism/state/STOP`), then the start command.
Only one supervisor can run at a time (`state/supervisor.pid`); a second start refuses and exits.

## Where to look

| file | what |
|---|---|
| `state/REPORT.md` | status line, scoreboard summary, decisions, blocked tasks, NOT-verified list (rewritten after every task) |
| `state/heartbeat.txt` | time, current task, step, free disk (rewritten every 60 s) |
| `state/log.md` | append-only: every start, restart, exit code, wait, decision |
| `state/queue.json` | tasks: status, depends_on, notes, commit, crash count |
| `state/runs/<task>-<time>.jsonl` | full transcript of each headless run (stream-json), `.err` = stderr |
| `state/supervisor.out` | supervisor console output |
| `eval/scoreboard.md` | every score |

Each run is also a normal Claude Code session named `realism-<task>` (`claude --resume` lists them).

## What `claude --help` offers (installed version 2.1.263) and what is used

Checked on 2026-09-26 with `claude --help` and a live probe (not guessed):

| flag | used | why |
|---|---|---|
| `-p, --print` | yes | non-interactive: run one prompt and exit |
| `--output-format stream-json` + `--verbose` | yes | full transcript per run; the final `result` line carries `subtype`, `permission_denials`, `total_cost_usd` |
| `--permission-mode <acceptEdits, auto, bypassPermissions, manual, dontAsk, plan>` | `dontAsk` | anything not pre-allowed is denied at once; nothing is skipped |
| `--permission-prompts <host, none>` | `none` | "nobody: anything that would prompt is denied automatically" - a run never waits for a person |
| `--settings <file>` | `agent_settings.json` | the allow/deny lists below |
| `--strict-mcp-config` (no `--mcp-config`) | yes | no MCP servers (Gmail, Drive, etc.) in unattended runs |
| `--no-chrome` | yes | no browser-extension integration (Playwright drives its own Chrome) |
| `--name realism-<task>` | yes | readable session names |
| `--dangerously-skip-permissions`, `--allow-dangerously-skip-permissions`, `bypassPermissions` | **no** | Prompt 4 forbids skipping all checks unless the owner configured it; `~/.claude/settings.json` has no such setting |
| `--max-budget-usd` | no | auth is a claude.ai Pro subscription (`claude auth status`), where usage limits, not dollars, stop runs; the supervisor waits them out |
| `--model`, `--effort` | not passed | the owner's defaults apply (`~/.claude/settings.json`: model `opus`, Opus 5.5 effort `xhigh`). To save usage add e.g. `--model sonnet --effort high` in `run_forever.sh` |
| `--no-session-persistence` | no | sessions are kept so the owner can inspect or resume a run |

## Permissions (narrowest set that still lets tasks run unattended) - `agent_settings.json`

Allowed: Read, Glob, Grep, TodoWrite; Edit/Write anywhere inside `~/Downloads/tamil_harvest`; Bash: `node`, `npm`,
`python3`, the project venv's `python`/`pip`, `git status|diff|log|show|add|commit|revert|rev-parse|ls-files`,
`git branch --show-current`, `ls`, `cat`, `head`, `tail`, `wc`, `du`, `df`, `date`, `shasum`, `diff`, `sort`, `file`,
`which`, `echo`, `pwd`, `true`, `grep`, `mkdir -p`, `cp -n` (no-overwrite copies for backups). ffmpeg is not installed; Playwright is driven through `node`.

Denied (deny beats allow): edits to `design/references/**`, `refs.db`, `website_live_backup_2026-09-25/**`, the 7 old
pages, `.git/**`, the supervisor, its settings, `rs.py`, `AGENT_BRIEF.md`, `tasks/**`, this file; Bash `rm`, `mv`, `npx`,
`curl`, `wget`, `claude`, `git push|reset|checkout|switch|rebase|stash|clean|rm|mv`, `git branch -d/-D`, `npm publish`;
WebFetch, WebSearch, subagents.

Probe (2026-09-26 09:52, headless, these exact settings): `node --version` allowed; `rm ...` denied; Write to
`run_forever.sh` denied; Write to `state/perm_probe.txt` allowed; no prompt, no waiting.

When a run needs something that is denied, it is refused immediately. The agent is told to mark its task
`blocked: needs approval <command>` and end; the supervisor also blocks any task whose run ended (not done) with a
denial after ending normally (a crashed run with an earlier harmless denial counts as a crash), logs the command, and moves on. To approve: add the rule to `agent_settings.json` `allow`, then
`python3 design/realism/state/rs.py set <task> pending --note "owner approved <command>"`.

## Supervisor rules (`run_forever.sh`)

- Wrapped in `caffeinate -ims` (the Mac does not idle-sleep; with the lid closed on battery it still sleeps).
- Picks the first `running` (crashed earlier) or `pending` task whose dependencies are done, skipped or blocked
  (a blocked dependency is handed to the agent, which works around it or blocks too), runs ONE headless agent,
  logs the exit code, rewrites REPORT.md, sleeps 30 s, repeats.
- 95-minute hard cap per run (tasks stop themselves at 80-90 min) -> task `blocked: time cap`.
- A run that ends without marking its task done/blocked counts as a crash; 5 in a row on the same task -> `blocked`.
- Usage limit (claude.ai Pro, HTTP 429 "You've hit your session limit · resets 3:30pm"): not a crash; the supervisor
  sleeps until the stated reset time + 3 minutes (a time more than 5 h 10 min away is treated as stale: retry in 1 h;
  no time given: 20, 40, then 60 minutes) and resumes the same task. Any error run that cost nothing and ended within
  90 s never reached the task: waited out (10 min), not counted as a crash; 6 in a row stop the supervisor.
  (Fixed 2026-09-26 18:30 after the first day: the old pattern missed 'session limit', so 18 tasks were wrongly
  blocked as '5 failed runs'; they were requeued.)
- Login or network failure within 2 minutes: waits 10 minutes; 6 in a row -> supervisor stops.
- Stops when: STOP exists, free disk < 20 GB, the branch is not `realism-engine`, or no runnable task is left.
- Dry-run tested with a fake agent (done, crash x2 -> blocked, usage limit -> wait -> done, over cap -> killed ->
  blocked, permission denial -> blocked, dependency order, STOP mid-task -> pending + clean exit, second copy refused).

## To unblock or rerun a task

`python3 design/realism/state/rs.py set T5b pending --note "owner: <what changed>"`, then start the supervisor.
