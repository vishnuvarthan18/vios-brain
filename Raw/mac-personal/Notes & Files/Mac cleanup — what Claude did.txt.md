---
source: personal Mac ~/Notes & Files/Mac cleanup — what Claude did.txt
---

MAC CLEANUP & ORGANIZATION — SUMMARY
Done by Claude, 22 Jul 2026.  Nothing was deleted. Every change is a move you can undo.

====================================================================
*** BIGGEST FINDING (added later) ***
====================================================================
The mystery "223 GB System Data" is mostly your ADOBE AFTER EFFECTS CACHE:
  ~/Library/Caches/Adobe/After Effects .......... 47 GB
  ~/Library/Caches/Adobe/After Effects (Beta) ... 93 GB
  = ~140 GB of rendered-preview cache. This is NOT your data — After
    Effects rebuilds it automatically. Clearing it is 100% safe.

I MOVED it (did not delete) into:
  ~/Library/Caches/Adobe/_SAFE TO DELETE - After Effects cache/

TO RECLAIM THE 140 GB:
  1. Finder > press Cmd+Shift+G
  2. Paste:  ~/Library/Caches/Adobe/
  3. Drag "_SAFE TO DELETE - After Effects cache" to Trash
  4. Empty the Trash.
  (Going forward, empty it inside After Effects:
   Preferences > Media & Disk Cache > Empty Disk Cache.)

Also tidied your PERSONAL Google Drive ((removed)):
grouped 20 resume files into "resume/" and 24 career docs into
"job-career/". Work/shared drives were left untouched.

iCloud: only ~1 GB local, so "Optimize Mac Storage" won't help — skip it.

====================================================================
THE HEADLINE
====================================================================
Your disk: 494 GB total, ~90 GB free. It is NOT critically full, so the
hanging/beachballs are most likely RAM pressure (many heavy apps open at
once: Photoshop, Illustrator, After Effects + Chrome/Arc/Brave/Safari),
not disk space. Freeing space helps, but check Activity Monitor > Memory
when it hangs.

Where your space actually goes (biggest first):
  • System Data ....... 223 GB  (mostly the two items below)
  • CloudStorage ....... 65 GB   Google Drive (3 accounts) + iCloud, mirrored offline
  • Application Support  49 GB   Adobe / browser / app data
  • Downloads .......... 50 GB   (29,514 files — now organized, see below)

====================================================================
WHAT I ORGANIZED (no deletions)
====================================================================
DOWNLOADS
  • Created "NEED REVIEW - safe to delete/" and moved ~19 GB of clear junk
    into it (13 GB backup.zip, 4.3 GB "untitled folder", a duplicate
    project folder, redundant zips, installers). See its README.txt.
  • Sorted all 324 loose files into "_Sorted/" by type:
    Images(111), PDFs(72), Videos(61), Archives(25), Design files(19),
    Documents(15), Misc(12), Audio(5), Web & data(4).
  • Left your 53 project folders (act-training, Ara, Repair_Rewards, etc.)
    where they are — those are already projects.

DESKTOP
  • Screenshots/  <- 25 loose screenshots & recordings
  • Videos/       <- the 2 movie files (both 0 bytes — cloud placeholders)
  • W2D/          <- the event poster files
  • Left dev projects (w2d, w2d-landing, my-flutterflow-project) untouched
    so nothing in your code/IDE breaks.

DOCUMENTS
  • "After Effects Projects/"  <- the two .aep files
  • Left "Adobe" and "AEJuice" untouched (app-managed data).

====================================================================
YOUR NEXT STEPS (to actually free space)
====================================================================
1. Open Downloads > "NEED REVIEW - safe to delete", skim it, drag back
   anything you want, then move the folder to Trash AND empty the Trash.
   (Space frees only after emptying Trash — I'm not allowed to do that.)

2. Biggest win, zero data loss — switch cloud to online-only:
   • Google Drive: menu-bar Drive icon > gear > Preferences > Google Drive
     > choose "Stream files" (not "Mirror files") for each account. Frees
     most of the 65 GB; files stay in the cloud, download on demand.
   • iCloud: System Settings > [your name] > iCloud > turn on
     "Optimize Mac Storage".

3. Old project folders (~30 GB in Downloads, listed in the review README):
   if they're finished work, move them to an external drive or cloud.

See "Storage maintenance guide.md" (also on your Desktop) for the ongoing routine.
