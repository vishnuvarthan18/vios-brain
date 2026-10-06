---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# STATE — forest (Sathyamangalam Atlas)
Full details: [[Projects/forest/SUMMARY]] · History: [[Projects/forest/LOG]] · Dev: [[Projects/forest/DEV-LOG]]

## Where we are (as of 2026-10-06)
- Active. All work saved in one local commit on `main` (2,940 files); **not pushed** (main 4 ahead of origin/main).
- 2026-10-03 data refresh deployed to `dev.sathyamangalam.online` only; production `sathyamangalam.online` not updated (still Home/About/Contact + "under construction").
- Data: 92 places, 2,664 species, 78,467 occurrences, 928 public documents (+6,632 other-tier, 1,424 unreviewed). Coverage 32.5%. Places are the biggest gap (vs 800–2,000 target).
- Dev site has generated detail pages: 2,664 species, 92 places, 83 history pages.
- Harvest engine: production Worker b1cd0cce (sensitive-species fix). Later fixes (dedup freeze, lgd paging, retries, stale-run reaper, stream_health, DLQ alert) on branch `deploy/sensitive-species-fix-verify`.
- Not sure if the cron is running: paused 2026-09-05, but a 2026-09-07 deploy re-attached the schedules.
- Domain was suspended 2 Sep 2026 (unverified registrant email) — not sure if fixed.
- Career side (IGNOU, jobs, portfolio): see [[Projects/career/STATE]].

## Next steps
1. Push `main` to GitHub; production deploy only with Vishnu's go-ahead.
2. Check harvest-engine cron state.
3. Production coarsening gate + backfill of sensitive coordinates.
4. Confirm data.gov.in key is a secret; grow places via LGD. Get WDPA key; set DLQ email address.
5. Build CORE stream; make tiger/elephant/leopard "conflicts" date-aware.
6. Move FIRMS to NOAA-21 before 1 Nov 2026.

## Blockers
- Exposed keys need rotating (data.gov.in in a doc; Anthropic key pasted 10 Aug).
- No staging environment; dev/main diverged.
- Every deploy needs Vishnu's explicit go-ahead; git/Cloudflare commands run in Vishnu's own Terminal.

## Key places
- GitHub: `vishnuvarthan18/sathyamangalam-atlas`, `vishnuvarthan18/senna-regrowth-verification` (both private).
- Sites: `sathyamangalam.online` (prod), `dev.sathyamangalam.online` (dev), `engine.sathyamangalam.online/dashboard`.
- Local: `~/sathyamangalam-atlas` (personal Mac).
