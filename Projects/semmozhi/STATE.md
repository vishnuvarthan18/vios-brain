---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# STATE: Semmozhi

Full summary: [[Projects/semmozhi/SUMMARY]] · Log: [[Projects/semmozhi/LOG]] · Dev: [[Projects/semmozhi/DEV-LOG]]

## Where we are (as of 2026-10-05)
- Site is online on Cloudflare Pages. www.semmozhi.online shows "Under construction"; the full site is on staging dev.semmozhi.online (hidden from search). (earlier: "runs only on the Mac, not online")
- Private admin site engine.semmozhi.online behind Cloudflare Access (only Vishnu).
- Release: push to `dev` → staging; production (`main`) only on manual "Run workflow". To go live: set `website/production-mode.txt` to `site`, merge to `main`, run workflow.
- Git: only `main` and `dev`; they match; nothing uncommitted.
- Public pages: Home (with Contact), Scripts, Brahmi Lab, Grantha, Vatteluttu, Tamil. Chola, Literature, About, All fonts hidden but kept.
- Design system: Web Awesome Core with original cream and terracotta colours.
- 4 fonts at v3.0; Vatteluttu draft waits for [[People/Elmar Kniprath]]'s review (not sent).
- Data: crawls off since 5 Sep; collector idle on OVH server (`/srv/semmozhi/`); ~5,300 real Tamil-text records.
- Next big step: Vishnu said "we are going to do something big" — not yet described.

## Next steps
1. Get Vishnu's "something big" plan.
2. Decide when production shows the full site.
3. Turn on BigRock auto-renew (expires 26 Aug 2027).
4. Send Vatteluttu draft to Elmar Kniprath.
5. Photo credits (temple, coin); Tamil-speaker review; source-check 2 timeline facts.

## Blockers
- Expert/Tamil-speaker reviews not done.
- Data lacks licence fields; topic engines never tested.

## Key places
- Production www.semmozhi.online · Staging dev.semmozhi.online · Admin engine.semmozhi.online
- Mac folder: `~/Downloads/tamil_harvest` (see `PROJECT_MAP.md`)
- Repo: `vishnuvarthan18/tamil-data-collector` (private)
- Server: OVH VPS, collector in `/srv/semmozhi/`
