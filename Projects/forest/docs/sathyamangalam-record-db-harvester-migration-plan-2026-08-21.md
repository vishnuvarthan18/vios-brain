# Harvest Engine & Record DB — End-to-End Migration Plan

**Written:** 21 Aug 2026. Nothing gets touched until this plan is approved.

## Where things stand today

- **Record DB** = one file called `atlas.db`, sitting inside your GitHub repo (`sathyamangalam-atlas`). 74,180 wildlife records, 1,839 species, 6,914 papers, 19 places, and more.
- **Harvest Engine** = Python scripts on your own computer. You run them by hand. Nothing happens automatically.
- **Website** (sathyamangalam.online) reads a set of summary files that get generated from `atlas.db`. It never touches the database directly.

## Where we're taking it

- **Record DB** moves to Turso — a hosted database service built for exactly this format. Free at your size. Backed up automatically. Reachable from anywhere, not just your laptop.
- **Harvest Engine** moves to GitHub Actions — a free "robot" built into GitHub that runs your scripts on a schedule (e.g. once a day) with nobody touching a keyboard.
- **Website** keeps working exactly as it does now, for every visitor. Only the wiring behind it changes.

Nothing about what the site shows, how it looks, or how visitors use it changes. This is entirely a backstage move.

---

## The stages, in order

### Stage 0 — Safety net (before anything else moves)
- Take a full backup copy of `atlas.db` exactly as it is today, saved somewhere outside GitHub too (e.g. sent to you directly).
- Confirm the GitHub repo itself is untouched and still works as a fallback the whole way through.
- **Nothing is deleted at any stage until the new setup is proven working for at least a few days.**

### Stage 1 — Get the database a new home
- You create a free Turso account (2 minutes, needs your email — I can't do this step for you).
- I move all the data from `atlas.db` into Turso, table by table, then compare row counts against the original to prove nothing was lost or changed.
- At this point: Turso has a full copy. The GitHub file is still there, untouched, still the "official" copy.

### Stage 2 — Prove the copy is trustworthy
- I run checks comparing Turso's data against the original file: same row counts, same species names, same disputed-figures table, spot-checks on a sample of records.
- You don't do anything technical here — I show you a short before/after comparison and you say yes or no.

### Stage 3 — Move the harvester to run automatically
- I write a GitHub Actions schedule (a free built-in feature of GitHub, no new account) that runs your existing harvest scripts once a day (or whatever cadence you want — daily, weekly, your call).
- The scripts are changed to write into Turso instead of the local file.
- First few runs are watched closely — if a run fails, nothing bad happens automatically; it just doesn't update that day, and I get notified.

### Stage 4 — Point the website at the new database
- The website's "read data" step is switched from the old summary files to Turso.
- Tested on a private preview link first — **not** the live site — so nothing public breaks while we test.
- Only once that preview looks correct does it go live on sathyamangalam.online.

### Stage 5 — Retire the old setup
- Once Turso + GitHub Actions have run cleanly for a set period (I'd suggest at least a week), the old `atlas.db` file in the repo is marked as archived, not deleted — kept as a historical backup, just no longer the "live" copy.
- Nothing is ever hard-deleted without you explicitly saying so.

---

## What you need to do, total

Two things, both one-time:
1. Sign up for a free Turso account (I'll give you the exact steps when we get there).
2. Say "yes, go live" at Stage 4, after seeing the preview.

Everything else — the actual moving, coding, testing, checking — I do.

## Cost

Free at every stage, at your current data size (a few tables, under 100,000 rows total). If the project grows dramatically — millions of records — there could eventually be a small monthly cost, but that's far off and I'll flag it well before it happens, not after.

## Risk / rollback

At every stage, the old GitHub file stays exactly as-is until the new thing is proven. If anything goes wrong at any point, we just... keep using the old file — nothing forces a cutover before it's ready.

## What this plan does *not* cover

- It doesn't add any new data streams (gazetteer, government docs, etc.) — that's separate work, for after this migration.
- It doesn't change anything about the website's design or content.
- It doesn't touch `senna-regrowth-verification` or `crawler.js` — separate, untouched.

---

## Open decision needed from you

**How often should the harvester run automatically?** Options: daily, weekly, or "only when I ask" (manual trigger, but from anywhere, not just your laptop). No wrong answer — daily is normal for a project actively growing its data, but there's no cost difference between the choices.

Nothing proceeds until you approve this plan as a whole.
