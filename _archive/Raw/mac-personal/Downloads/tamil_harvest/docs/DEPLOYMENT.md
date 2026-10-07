# Deployment

## What www.semmozhi.online shows right now
`website/production-mode.txt` decides it. While it says **construction**, production serves only the one-page "Under construction" notice in [construction/index.html](../construction/index.html) for every address (the contact API is switched off), and the real site is visible only on https://dev.semmozhi.online. To publish the real site, change that word to **site**, merge to `main` and click Run workflow.

## Two environments
| | Staging | Production |
|---|---|---|
| URL | https://dev.semmozhi.online | https://www.semmozhi.online |
| Branch | `dev` | `main` |
| Cloudflare Pages project | `semmozhi-dev` (we always deploy `dev` code into its "main") | `semmozhi` (production branch = `main`) |
| Starts | By itself on every push to `dev` | **Only when you start it by hand** (a push or merge to `main` never goes live by itself) |
| Contents | Every page, marked `noindex` so search engines skip it | Every page **except** those listed in `website/staging-only.txt` |

Only `website/` is uploaded, via `scripts/package_site.py`. Before any deploy, `scripts/check_website.py` must pass. Workflow: `.github/workflows/deploy.yml`. Checks on pull requests: `.github/workflows/checks.yml`.

## Release steps
1. Merge your pull request into `dev`. Staging updates by itself; look at https://dev.semmozhi.online.
2. Open a pull request `dev` -> `main`, get approval, merge.
3. Go live: GitHub -> Actions -> "Deploy website" -> **Run workflow** -> branch `main`. That click is the approval. Then load the live site.

## The contact function
`functions/api/contact.js` is deployed with the site (see [CONTACT.md](CONTACT.md) for the WhatsApp secrets).

## Responsive test
`make responsive` (or `node scripts/check_responsive.mjs [page.html]`) loads every live page in headless Chrome at 11 widths from 320 to 1920 px, in English and Tamil, and fails on any sideways scroll or element outside the screen. `node scripts/check_behaviour.mjs` clicks through what visitors do (language dialog and cookie, Contact panel, dark mode, converter, 404). Both accept `BASE=https://www.semmozhi.online` to test the live site. Run them before a release; together about 7 minutes.

## Staging-only pages
List a page in `website/staging-only.txt` and it appears on staging but is removed from production. The production build also fails if a live page links to a staging-only one, so there are no dead links. To take a page live, delete its line, merge `dev` to `main`, approve.

## One-time setup (done in the dashboards, not in git)
1. **Cloudflare:** create Pages project `semmozhi-dev` (Direct Upload; leave its production branch as `main`, the workflow deploys `dev` code into it); add custom domain `dev.semmozhi.online` to it.
2. **GitHub -> Settings -> Environments:** create `staging` and `production` (no rules on the free plan).
3. Branch protection and required reviewers need GitHub Pro/Team (or a public repo); the repo is private on the free plan, so the rule "do not push straight to `main`" is by agreement until then. The manual production start is what keeps `main` from going live by accident.

## The private admin site (engine.semmozhi.online)
Home page: `engine-site/index.html`. Built by `scripts/package_engine.py` into: the home page (staging vs production, links to Go live), the design system (`design/`, from `design/showcase/`), the crawl dashboard (`dashboard/`) and `status.json`. Workflow: `.github/workflows/engine.yml` (Cloudflare project `semmozhi-engine`, rebuilt on every push to `dev` or `main`; it creates the project the first time).

**It is only private once Cloudflare Access is in front of it** (dashboard steps, done once):
1. Cloudflare -> Zero Trust -> Access -> Applications -> Add -> Self-hosted.
2. Application domain: `engine.semmozhi.online`. Add a second domain `semmozhi-engine.pages.dev` so the Pages address is locked too.
3. Policy: Allow, include Emails = the people who may enter (for now only the owner). Login is a one-time code sent to that email.
4. In the Pages project `semmozhi-engine` -> Custom domains, add `engine.semmozhi.online` (CNAME `engine` -> `semmozhi-engine.pages.dev`).

The site contains no secrets (design files and published crawl statistics), is `noindex`, and is never cached.

## Secrets (GitHub -> Settings -> Secrets and variables -> Actions)
| Secret | What | Where it comes from |
|---|---|---|
| `CLOUDFLARE_API_TOKEN` | Token with **Account -> Cloudflare Pages -> Edit** only | Cloudflare dashboard -> My Profile -> API Tokens |
| `CLOUDFLARE_ACCOUNT_ID` | The Cloudflare account id | Cloudflare dashboard, account home |
| `DASHBOARD_DEPLOY_KEY` | SSH key for the separate crawl-dashboard repo | Used by the crawl workflow only |

Rotate the token if anyone who had access leaves. Never paste a token into an issue, pull request or chat.

## Deploy by hand (emergency or first setup)
```bash
git checkout main && git pull
bash deploy.sh          # refuses unless you are on a clean main
```
Needs `wrangler login` as someone with access to the Cloudflare account.

## Roll back
Cloudflare dashboard -> Workers & Pages -> `semmozhi` -> Deployments -> pick an earlier deployment -> **Rollback**. Then revert the bad commit on `main` so the next deploy does not bring it back.

## Domains
- Registrar: **BigRock**. DNS: **Cloudflare** (BigRock nameservers are set to the two Cloudflare ones).
- `semmozhi.online`: zone on Cloudflare, attached to the Pages project, with proxied CNAME records `@` and `www` -> `semmozhi.pages.dev`. Status at the time of writing: `www` active, apex still being verified by Cloudflare.
- `semmozhi.info`: owned at BigRock, **not yet added to Cloudflare**. To connect it: add it in Cloudflare, set its BigRock nameservers to the two Cloudflare gives, wait for Active, then attach it to the Pages project and add the CNAMEs.
- To add a domain to the site later: Pages project -> Custom domains -> Set up a domain, then add the CNAME in the zone's DNS.

## Never
- Do not deploy from a feature branch to production. Only `main` is production.
- Do not upload anything other than `website/`.
- Do not delete or change the other Pages projects in the account (Sathyamangalam Atlas).
