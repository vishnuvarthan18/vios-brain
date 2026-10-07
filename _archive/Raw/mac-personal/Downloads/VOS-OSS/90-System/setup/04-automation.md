---
type: system
status: stable
created: 2026-08-21
updated: 2026-08-21
tags: [vos, setup]
---

# 4. Automation — the loops that keep it alive

## Nightly, on the Mac (launchd)

```bash
cp 90-System/scripts/com.vos.nightly.plist ~/Library/LaunchAgents/
# edit the two CHANGEME paths inside it first
launchctl load ~/Library/LaunchAgents/com.vos.nightly.plist
```

`vos tidy` runs at 02:30: git snapshot, frontmatter validation, broken-link
check, stale-note list, three random notes to consider deleting, regenerated
dashboards, `qmd` reindex if installed, and a dated health report in
`90-System/audits/`.

Because each report is a dated file, vault health becomes a time series. In
three months you can see whether the system is getting healthier or rotting.

## Nightly, on a Linux box or VPS (cron)

```cron
30 2 * * * cd /srv/vos && /srv/vos/bin/vos tidy >> /var/log/vos.log 2>&1
```

## Weekly and monthly

These are agent jobs, not scripts, because they require judgement:

- **Friday:** `/weekly` — the review, built from your dailies, git activity and
  project notes. It must report what stalled 14+ days and where your actions
  contradicted your stated plan.
- **Monthly, 15th:** `/decide --revisit` — score the predictions whose review
  date has passed.
- **Monthly:** read the newest health report and compare it to the first one.

Run them by hand at first. Automate them only once you trust the output.

## Deconflicting Syncthing

```bash
90-System/scripts/deconflict.sh
```

Union-merges any `*.sync-conflict-*` files in `10-Journal/` (append-only, so a
union merge is always correct) and lists conflicts elsewhere for you to resolve
by hand. Add it to the nightly job if you edit on both devices a lot.
