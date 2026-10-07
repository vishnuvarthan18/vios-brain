# Mac Storage Maintenance Guide

A simple routine to keep your Mac from filling up again. Tuned to how you
actually use this machine (heavy Adobe + video work, lots of cloud accounts,
a very full Downloads folder).

## The one habit that matters most

**Treat Downloads as a temporary folder.** Yours grew to 50 GB / 29,000
files because nothing ever left it. Every week or two, move finished files to
their real home (project folder, external drive, or cloud) and clear the rest.
This alone prevents most of the problem.

## Weekly (2 minutes)

- Empty the Trash. Deleting a file doesn't free space until Trash is emptied.
- Glance at Downloads. Move or delete anything you're done with.

## Monthly (10 minutes)

- **Big files check:** System Settings → General → Storage → click the (i)
  next to *Documents* / *Applications* to see the largest items. Delete or
  archive anything you no longer need.
- **Clear Adobe media cache** (rebuilds automatically, safe): After Effects /
  Premiere → Settings → Media Cache → **Delete Unused**. Can reclaim GBs.
- **Empty browser caches / old downloads** in Chrome, Arc, Brave.

## Quarterly (20 minutes)

- **Archive old projects.** Finished video/design projects don't belong on
  your internal drive. Move them to an external SSD or cloud. Your Downloads
  alone had ~30 GB of these.
- **Uninstall apps you stopped using.** Use the App's own uninstaller or drag
  to Trash, then remove leftover data in `~/Library/Application Support`.
- **Review cloud sync.** You sync 3 Google Drive accounts + iCloud locally.

## Keep cloud storage "online-only" (recommended, permanent fix)

This was your single biggest silent space user (~65 GB). Files stay safe in
the cloud and download only when you open them:

- **Google Drive** (per account): menu-bar Drive icon → gear → **Preferences**
  → Google Drive → select **"Stream files"** (not "Mirror files").
- **iCloud:** System Settings → [your name] → iCloud → turn on
  **"Optimize Mac Storage."**

## About the hangs (separate from storage)

You have ~90 GB free, so the beachballs are almost certainly **memory**, not
disk. When it hangs:

1. Open **Activity Monitor → Memory** tab.
2. Look at **Memory Pressure** (the graph at the bottom). Yellow/red = you're
   out of RAM.
3. Quit what you're not using — especially extra browsers (Chrome + Arc +
   Brave + Safari all at once is heavy) and idle Adobe apps.
4. Restarting once a week clears memory leaks from long-running creative apps.

## Signs you should act

- Free space drops under ~10% of the disk (≈50 GB for you) → clean now.
- "System Data" balloons past ~60–80 GB → clear caches, old iOS backups,
  and check `~/Library/Application Support` for a bloated app.
- Frequent beachballs → check Memory Pressure, not storage.
