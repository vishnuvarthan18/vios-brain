# V OS — start here

This is a starter vault, not a finished system. Everything in it is plain text you
own. Nothing here needs a subscription to keep working.

## Open it (10 minutes)

1. Install Obsidian (free) on your Mac. **Do not install a single plugin today.**
2. `Open folder as vault` → point it at this folder.
3. Settings → Files and links → set `Templates folder` to `90-System/templates`.
4. Settings → Core plugins → turn on **Daily notes**, **Templates**, **Bases**,
   **File recovery** (set it to 5 minutes / 365 days). Turn on nothing else.
5. Daily notes settings: folder `10-Journal`, format `YYYY/MM/YYYY-MM-DD`,
   template `90-System/templates/daily`.
6. In Terminal, from this folder:
   ```
   git init && git add -A && git commit -m "V OS day zero"
   ```
   That is your undo button for everything that follows.

## The only rule for week one

Open the daily note. Dump anything into `## Capture`. That's it. Do not create
folders, do not install plugins, do not reorganise. Seven days.

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
| `90-System/` | Templates, schema, scripts, logs, audits, dashboards. |
| `99-Archive/` | Finished and dead things. Nothing is deleted, it moves here. |

## The important files

- **`CLAUDE.md`** — the contract your AI agent must obey. Read it once; it is the
  difference between an assistant and a vandal.
- **`90-System/schema.md`** — the closed list of `type` and `status` values. If you
  only enforce one thing, enforce this.
- **`90-System/scripts/validate.py`** — `python3 90-System/scripts/validate.py`.
  Run it whenever an agent has been writing.
- **`90-System/scripts/random-three.py`** — shows three random notes so you can
  delete them. This is the habit that stops the vault becoming a graveyard.
- **`90-System/scripts/nightly-tidy.sh`** — mechanical health check, writes a dated
  report into `90-System/audits/`. Schedule with `90-System/vos-nightly.plist`.
- **`.claude/skills/`** — your four commands: `/inbox`, `/meeting`, `/weekly`,
  `/decide`. Four is deliberate. A fifth needs a reason.

## Phase order (do not skip ahead)

1. **Week 1** — vault + daily note + sync + git. Nothing else.
2. **Week 2** — agent layer: Obsidian CLI, `kepano/obsidian-skills`, this `CLAUDE.md`.
3. **Week 3** — automation: nightly tidy on a schedule, `/weekly` on Fridays.
4. **Week 4** — capture on Android, voice, meetings, people.
5. **Month 2+** — decisions with predictions, output pipeline, dashboards.

The blueprint document that came with this vault explains why each phase is in
that order and what breaks if you reorder them.
