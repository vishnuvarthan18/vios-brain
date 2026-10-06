---
tags: project
status: paused
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# STATE: wedding2day.com — mandap marketplace + wedding packages

Full history: [[Projects/wedding2day-com/SUMMARY]] · Log: [[Projects/wedding2day-com/LOG]]
Sister project (B2B app, same brand): [[Projects/wedding2day-app/SUMMARY]]

## Where we are (as of 2026-10-06)
- Status: **paused** (not sure). Last commit 2026-07-28; investor deck Aug 2026.
- Repo found: `~/Desktop/mura/w2d/d2c/wedding2day.com` (old folder `~/Desktop/wedding2day.com` is empty).
- Code: mandap marketplace + wedding packages built; ~31 of 39 tickets; 132 tests pass.
- New build **not live**: W-031 deploy, remote D1 migration, `R2_PUBLIC_BASE_URL`, W-018, W-036, W-029 still to do.
- Old 2025 site (Hostinger) is gone: hosting expired 13 Jun 2026.
- **Domain risk:** Cloudflare zone deleted 12 Aug; Hostinger renewal failed 20 Aug — domain may be lapsing.
- Venue sign-ups: 0 (as of 2026-07-28).

## Next steps
1. Check and renew the wedding2day.com domain.
2. Sign 15–20 mandapams in the first city using the call sheet.
3. Decide who answers inquiries and how fast.
4. Start demand: [[Tools/Instagram]] real weddings, muhurtham-date content, referrals.
5. Go-live tasks, then W-031 deploy via a [[Tools/Cursor]] queue prompt.
6. Decide W-038 schema (column vs table).

## Blockers
- Domain may be lapsing.
- Zero signed venues — a city cannot open below 15–20.
- Delivery owner not decided.

## Key places
- Repo: `~/Desktop/mura/w2d/d2c/wedding2day.com`
- Stack: [[Tools/Cloudflare]] Pages (Workers runtime) + D1 + R2, [[Tools/Next.js]] 16, [[Tools/Drizzle ORM]]
- Docs in repo: `docs/PLAN.md`, `docs/PRD.md`, `docs/ARCHITECTURE.md`, `docs/BACKLOG.md`, `AGENTS.md`
- Call sheet: `data/wedding2day-ops-call-sheets.xlsx`
