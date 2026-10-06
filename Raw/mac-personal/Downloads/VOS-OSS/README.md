# V OS — start here

A second brain made of plain text files you own. Every component below is
**open source** and **free forever**. Nothing here needs a subscription, an
account, or a company staying in business.

## What this actually is

A folder of Markdown files, kept in git, with a written contract that lets an AI
agent file, link, review and maintain them without wrecking anything. There is
no app you are locked into. Your notes are readable by `cat`, searchable by
`grep`, and openable by every text editor ever written.

## Day one (about 30 minutes)

```bash
mkdir -p ~/VOS
# copy this whole folder's contents into ~/VOS, then:
cd ~/VOS
git init && git add -A && git commit -m "V OS day zero"

echo 'export PATH="$HOME/VOS/bin:$PATH"' >> ~/.zshrc
echo 'export VOS_VAULT="$HOME/VOS"'      >> ~/.zshrc
source ~/.zshrc

vos help
vos daily
vos jot "first thing in my second brain"
```

That is a working system. Everything else is optional improvement.

Then read, in order:

1. `90-System/setup/01-mac.md` — editor and tools on the Mac
2. `90-System/setup/02-android.md` — Markor + Syncthing + two-second capture
3. `90-System/setup/03-agent.md` — pointing Claude at the vault
4. `90-System/setup/04-automation.md` — the nightly and weekly loops
5. `90-System/setup/05-server-optional.md` — SilverBullet in a browser, if you want it

## The only rule for week one

`vos jot` whatever comes into your head. Read the daily note in the evening.
That's it. Do not reorganise, do not add tools, do not restructure. Seven days.

## What's in here

| Folder | What goes in |
|---|---|
| `00-Inbox/` | Anything you haven't filed. Always allowed to be messy. |
| `10-Journal/` | One file per day. Weeklies and monthlies later. |
| `20-Projects/` | Things with a finish line. One folder each, with a `README.md`. |
| `30-Areas/` | Ongoing responsibilities. araCreate ops, finance, health, learning. |
| `40-Library/` | Ideas and knowledge. Flat. Titles are claims, not topics. |
| `50-People/` | One file per human. |
| `60-Raw/` | Transcripts, clippings, exports. **Never edited, ever.** |
| `90-System/` | Templates, schema, scripts, setup guides, dashboards, logs, audits. |
| `99-Archive/` | Finished and dead things. Nothing is deleted, it moves here. |

## The important files

- **`AGENTS.md`** — the contract every AI agent must obey. Read it once. It is
  the difference between an assistant and a vandal. (`CLAUDE.md` just points here.)
- **`bin/vos`** — the whole command line, in one dependency-free Python file.
  Read it; it is short enough to understand and yours to change.
- **`90-System/schema.md`** — the closed list of `type` and `status` values.
  If you enforce one thing, enforce this.
- **`.claude/skills/`** — your four commands: `/inbox`, `/meeting`, `/weekly`,
  `/decide`. Four is deliberate. A fifth needs a reason.
- **`.gitattributes`** — makes journal files union-merge, so your daily notes can
  never produce a conflict between devices.

## The commands

```
vos daily                 today's note (creates it if needed)
vos jot "text"            capture a line, timestamped
vos new <type> "Title"    create a note in the right place, correctly
vos find "term"           search
vos ls --type project --status active
vos stale 30              active notes nobody has touched in 30 days
vos random 3              three notes to consider deleting
vos check                 validate frontmatter, find broken links
vos dash                  regenerate dashboards
vos snapshot              git commit — your undo point
vos tidy                  the nightly job (all of the above + health report)
```

## Phase order (do not skip ahead)

1. **Week 1** — vault + `vos jot` + git + sync to the phone. Nothing else.
2. **Week 2** — agent layer: point Claude at the vault, run `/inbox` once.
3. **Week 3** — automation: nightly `vos tidy`, `/weekly` on Fridays.
4. **Week 4** — capture properly on Android, voice, meetings, the people layer.
5. **Month 2+** — decisions with predictions, publishing, dashboards.

The blueprint document that came with this kit explains why the order matters
and what breaks if you rearrange it.

## Licences of everything recommended

| Component | Licence |
|---|---|
| `vos` and this vault | yours, do what you like |
| git, ripgrep, fd, jq, yq | GPL-2.0 / MIT / Unlicense |
| VSCodium + Foam | MIT + MIT |
| Zettlr | GPL-3.0 |
| Markor (Android) | Apache-2.0 |
| Syncthing / BasicSync | MPL-2.0 / GPL-3.0 |
| Termux, Termux:API, Termux:Widget | GPL-3.0 |
| Voxscribe, Sayboard (voice) | MIT / GPL-3.0 |
| qmd (semantic search) | MIT |
| zk (graph queries, optional) | GPL-3.0 |
| SilverBullet (optional server) | MIT |
| Quartz (optional publishing) | MIT |

Recurring cost: **$0**. Optional: a few euros a month for a server, only if you
want browser access from anywhere.
