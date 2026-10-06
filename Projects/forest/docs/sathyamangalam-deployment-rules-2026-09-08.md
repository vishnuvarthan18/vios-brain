# Deployment rules — locked 8 Sep 2026

These are the permanent rules for how the Sathyamangalam Atlas project deploys. Any future session (human or agent) should follow these unless Vishnu explicitly changes them here.

## STATUS: both rules live and verified as of 8 Sep 2026, 14:58 UTC

- Production (sathyamangalam.online): confirmed serving only Home/About/Contact live, "under construction" on every other page.
- Dev (dev.sathyamangalam.online): confirmed live and serving the full, real, unfiltered site.

## 1. Production — sathyamangalam.online (main branch)

- Only three pages are live: **Home, About, Contact.**
- Every other section (Land, Life, People, History, Governance, Visit, Sources & Data, Places) shows an **"under construction"** placeholder instead of real content. The placeholder reuses the real site header/nav/footer so it doesn't look broken — only the body content is swapped.
- This is what the public sees. It lives on the `main` branch and deploys via the `sathyamangalam-atlas` Cloudflare Pages project.
- Deploy production ONLY with `scripts/deploy-prod.sh` — it refuses to run unless you're on `main`, synced with origin, and have a clean working tree, and requires typing "DEPLOY TO PRODUCTION" to confirm. Do not bypass this script.
- Do not deploy unfinished or unverified data here. Production is deliberately minimal until sections are ready to go live for real.
- Implemented 8 Sep 2026, commit `66164c8` on main. Placeholder pages: land.html, life.html, people.html, history.html, govern.html, visit.html, record.html, places.html.

## 2. Dev — dev.sathyamangalam.online (dev branch)

- Shows everything that's actually built: all harvested data, all sections, **completely unfiltered** — exactly as collected, no sensitivity masking, no gatekeeping.
- This includes exact coordinates for sensitive/vulnerable species. Confirmed explicitly by Vishnu on 8 Sep 2026: dev is meant to be the raw, working view, not a second public-safe copy.
- This is the internal working view — for Vishnu and anyone he shares the dev link with, not the general public. It lives on the `dev` branch.
- Deployed to a **separate Cloudflare Pages project**, `sathyamangalam-atlas-dev` (created 8 Sep 2026), NOT the same project as production. This keeps the two deploy targets fully independent — deploying dev never touches production and vice versa.
- To deploy dev: `git checkout dev && make build && npx wrangler pages deploy dist --project-name sathyamangalam-atlas-dev --commit-dirty=true`. This runs the detail-page generators (build_history_pages.py, build_species_pages.py, build_place_pages.py) then build_dist.py, so it always ships whatever is currently on the dev branch, in full.
- DNS: `dev.sathyamangalam.online` is a CNAME to `sathyamangalam-atlas-dev.pages.dev`, proxied, created 8 Sep 2026 (record id `f7631739a222cbfdf0238867309ec2a4`). Custom domain attached to the Pages project (domain id `6d577966-8e6d-4f2a-bc43-75261512002e`).
- Confirmed live 8 Sep 2026: 2,908 files deployed (2,664 species pages, 92 place pages, 83 history pages), all real, unfiltered.

## 3. Engine dashboard — engine.sathyamangalam.online/dashboard

- This is the harvest-engine's own operational dashboard (the data-collection tool), not the public-facing atlas site.
- Stays as-is. Not affected by the main/dev split above.

## 4. Cleanup baseline (completed 8 Sep 2026)

Before any of the above was built, a full cleanup pass was done. State as of this date:

**GitHub** (`vishnuvarthan18/sathyamangalam-atlas`, private):
- All branches now pushed and safe: `dev`, `main`, `restructure-explore-nav`, `feat/stream-health`, `fix/failing-streams`, `fix/lgd-fields-and-empty-run-guard`, `deploy/sensitive-species-fix`.
- Previously, `dev` was 18 commits and `main` was 3 commits ahead of GitHub — i.e. only existed on Vishnu's Mac. This was the most urgent risk fixed in this pass.
- A stray 0-byte file named `main` in the repo root (which broke `git log main` and similar commands) was moved to `_to_delete/` in the repo, then removed.
- A duplicate/stale full clone of the repo at `~/Downloads/sathyamangalam-atlas-clean` (769MB, last updated Aug 23, same GitHub remote) was deleted — confirmed redundant before deletion.
- Known friction: this local repo's `.git/index.lock` recurs when git write operations run from a sandboxed/automated shell (permission wall on unlinking it from that context) — always run git write operations (checkout, commit, stash) directly in Vishnu's own Terminal, not from an automated remote shell. If it appears there too, `rm -f .git/index.lock` clears it.

**Cloudflare** (account: Vishnu88varthan@gmail.com's Account, id `aa523b5d2ceed84e54997db0dc6cbaec`):
- Workers: only `harvest-engine` remains. `harvest-engine-staging` (unused, confirmed by Vishnu) was deleted along with its queue bindings.
- Queues: only the 3 actively-used ones remain (`harvest-engine-jobs` + its DLQ, plus the two now-orphaned `forest-job-*` were deleted, not kept — see note below). Orphaned queues with 0 producers/consumers (`forest-job-queue`, `forest-job-dlq`, and the harvest-engine-staging queues) were deleted.
- Pages: two projects now exist — `sathyamangalam-atlas` (production, main branch) and `sathyamangalam-atlas-dev` (dev branch, created 8 Sep 2026). 13 of 14 stale deployment snapshots on the production project (Aug 18–26, none live) were deleted; Cloudflare doesn't allow deleting the active one.
- DNS as of 8 Sep 2026: root + www → `sathyamangalam-atlas` Pages project; `engine.sathyamangalam.online` → harvest-engine Worker; `dev.sathyamangalam.online` → `sathyamangalam-atlas-dev` Pages project (new).

## 5. Access notes for future sessions

- GitHub: needs a fine-grained personal access token scoped to `sathyamangalam-atlas` with Contents (read/write) and Administration (read/write) to push/manage branches.
- Cloudflare: needs a custom API token with Account-level Workers Scripts (Edit), Workers Tail (Read), D1 (Edit), Queues (Edit), Cloudflare Pages (Edit), Account Settings (Read), and Zone-level DNS (Edit) scoped to `sathyamangalam.online`.
- Cloud sandbox terminals (this session's `device_bash` equivalent) cannot reach `api.cloudflare.com` — network blocked at the proxy level, regardless of token. Cloudflare API work (curl calls) runs fine from Vishnu's own Terminal — that's where all Cloudflare and git-write commands should run, with Claude supplying exact commands to paste. GitHub API calls (read-only or push via token) work fine from the sandbox.
- Account ID: `aa523b5d2ceed84e54997db0dc6cbaec`. Zone ID (sathyamangalam.online): `252554fdf33cbdae85970b82e396b881`.
- After a new DNS record is created, the requester's own machine may show "could not resolve host" for a few minutes due to local DNS cache, even though the record is already live at Cloudflare's edge. Verify with `dig <host> @1.1.1.1 +short` or `curl --resolve <host>:443:<ip> ...` instead of waiting/retrying blindly.
