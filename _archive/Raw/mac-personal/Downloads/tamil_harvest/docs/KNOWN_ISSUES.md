# Known issues and open decisions

Honest list as of 2026-10-03. Fix or decide, then remove the line.

## Unfinished
| Item | State | Next step |
|---|---|---|
| `semmozhi.online` (apex) | Zone is active; apex now answers 200 | Decide: serve the site or redirect to `www`. Turn on BigRock Auto-renew (domain expires 2027-08-26). |
| `semmozhi.info` | Owned at BigRock, not in Cloudflare | Add to Cloudflare, switch nameservers, attach to Pages (see DEPLOYMENT.md) |
| Explore and Engine pages | Deleted 2026-10-04 (in git history) | Run the engine build, copy `hub.json`, `search.json`, `engines/*/index.json` into `website/data/`, restore the pages |
| Home page and Kural text | Some content is labelled "sample · unverified" | Source and check, then remove the label |
| About page claims | Several marked "attach citations" | Add primary-source footnotes before a wide launch |
| Data refresh after crawl | Not automated; crawl schedule is off | Decide the schedule, then add a workflow that rebuilds `website/data` and opens a pull request |

## Not tested
- Lab pages: lessons, quizzes and letter charts were loaded but not exercised by hand.
- Phone width was checked automatically (no sideways scroll, content renders), not by eye on real devices.
- No automated test covers the converters (`brahmi.js`, `maps.js`, `grantha-map.js`).

## Risks
- **Repository size.** About 455 MB of history because raw crawled data (`data/`, 776 files) is committed. Clones are slow. Options: Git LFS, or move raw data to storage (the VPS or Cloudflare R2) and keep only `website/data/` in git. Needs a decision before the team grows.
- **Branch protection and required reviewers.** Not available on a private repo on the free plan. Production therefore deploys only by a manual **Run workflow** (that click is the approval); "never commit to `main`" is a convention.
- **Server (OVH VPS-2).** Shared with another project (data platform, notes, password manager). Collector foundation installed under `/srv/semmozhi` but **idle by decision**; no crawl, no schedule. See `docs/SERVER_PLAN.md`. `vps/docker-compose.yml` and `vps/README-VPS.md` still describe the old "server hosts the website" layout; use `vps/docker-compose.server.yml`.
- **Crawled data is 96% repeats and mostly lacks licence fields.** See `docs/DATA_AUDIT.md`. Do not publish crawled text before licence + URL are attached to each record.
- **Old address still live.** https://semmozhi.pages.dev (and `dev.semmozhi.pages.dev`) still respond. The docs no longer mention them; disable in the Pages project settings if unwanted.
- **Staging is public** (hidden from search engines only). The admin site is login-protected (Cloudflare Access).
- **Shared Cloudflare account.** The same account hosts the Sathyamangalam Atlas projects, storage buckets and KV namespaces. Use a token scoped to Pages only (as the deploy token is) and never delete things you did not create.
- **Duplicated CSS.** `tokens.css` and `components.css` exist in `design/showcase/css/` and `website/css/` and must be copied by hand. A script could check they match.
- **Old tool scripts.** Scripts in `design/realism/` and `design/showcase/_checks/` were written for the old `website_live/` layout and were pointed at `design/showcase/` by search and replace. They pass a syntax check but were not run.
