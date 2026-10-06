---
tags: project
status: active
owner: "[[People/Vishnu]]"
---
# STATE: araKraft Works website

Full history: [[Projects/arakraft/SUMMARY]] · Log: [[Projects/arakraft/LOG]] · Dev log: [[Projects/arakraft/DEV-LOG]]

## Where we are (as of 2026-10-01)
- Static React + Vite site for the araKraft Works laser/CNC workshop (Batticaloa).
- Follows the araCreate conventions; all website files are in `src/`.
- Builds clean, 18 tests pass, Lighthouse 93-100.
- Pushed to GitHub `aracreate-group/arakraft-works` (`8eb6596`).
- Cloudflare Pages and domain not set up yet — Vishnu does this.
- Commits are not signed yet.

## Next steps
1. Cloudflare Pages project (root dir `src`), remove old redirect, connect `arakraft.works` + `www`.
2. Check live site, clean up Netlify leftovers.
3. Set up SSH commit signing.
4. Fill analytics IDs, socials, address, hours.

## Blockers
- Cloudflare setup waits on Vishnu (domain is in another Cloudflare account).
- Business details (address, hours, socials, analytics) not given yet.

## Key places
- Repo: github.com/aracreate-group/arakraft-works
- Local: `~/Downloads/arakraft-works`
- Hosting guide: `docs/cloudflare-hosting.md`
- Live target: https://arakraft.works
