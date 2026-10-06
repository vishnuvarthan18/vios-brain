# ARM Platform — Release Process

**The most important fact to know before reading further**: there is no **code-promotion** pipeline for application repos on this platform today, and with dev and staging removed there is no pre-production environment to promote through either. `deploy/arm-deploy-make`'s `repos.conf` pins every application repo to the `dev` branch — deploying to the server clones and builds each application repo's `dev` branch HEAD, not a promoted or tagged release ([CICD.md](CICD.md) §2, [`runbooks/deploy-rollback.md`](runbooks/deploy-rollback.md) §3). If you came here looking for a merge-checklist/promotion-gate process, it doesn't exist yet — this document describes what actually happens instead.

---

## 1. Two separate concepts that are easy to conflate

**"Release" and "deploy" are unrelated mechanisms on this platform, triggered by completely different things:**

- **Release** = versioning + changelog + GitHub release, via `semantic-release` on a push to `main`/`dev` in an *application* repo. **No deploy happens as part of this** — confirmed directly in every repo that has it: the workflow builds, runs `semantic-release`, and stops.
- **Deploy** = `arm-deploy-make`'s own `deploy-production.yml`, triggered by a push to `main` **in `arm-deploy-make` itself** (or a manual `workflow_dispatch`). It SSHes into the server and runs `make ci-deploy-prod`, which clones fresh source and builds Docker images directly — never pulling a versioned/released artifact from anywhere.

Pushing to `dev` in, say, `core-be` runs *that repo's* CI and (if it has one) release workflow — neither deploys anything. A deploy only happens when someone pushes to (or manually triggers) one of `arm-deploy-make`'s own three branches. These two systems don't reference each other at all — a semantic-release version bump in an application repo has no effect on what the next deploy actually runs.

---

## 2. Release automation, per repo — real and confirmed, not assumed uniform

| Repo | Has `semantic-release`? | Notes |
|---|---|---|
| `arm-session` | Yes (`dev-release.yml`) | Versioning + changelog + GitHub release only |
| `arm-service-notification` | Yes (`dev-release.yml`) | Same |
| `arm-ui-library` | Yes (`release.yml`) | The one repo where this actually matters for a real published artifact — see §3 |
| `arm-core-be` | **No** — no separately-named release workflow found | Only has `ci.yml` (lint/test) confirmed; its `deploy.yaml` is a live, broken, unrelated Kubernetes workflow ([CICD.md](CICD.md) §5) |
| `arm-core-fe` | **No** | Only `ci.yml` (lint/test) + `e2e.yml` (Playwright) |
| `arm-app-calendar` | **No** | Only `ci.yml` (backend/frontend/SAST jobs) |
| `arm-admin` | **No** | **Zero `.github/workflows/` at all** — no CI, no release automation of any kind |

Don't assume every repo gets an automatic version bump and changelog on merge — only 3 of the 7 application repos actually have this wired up, and none of the 3 that do deploy anything as a result of it.

---

## 3. `arm-ui-library` — the one real published artifact, and a verified gap in its automation

`@aracreate/test-arm-ui` is the only thing on this platform that ships as an installable package, so its release process is worth being precise about.

**Verified directly in `.releaserc.json`: `@semantic-release/npm` is configured with `"npmPublish": false`.** `release.yml`'s CI-driven `semantic-release` run handles version computation (from conventional commits), `CHANGELOG.md` generation, a git tag, and a GitHub release — but **does not itself run `npm publish`**. The actual publish to the npm registry is a manual step, documented in [`docs/guides/npm-publish.md`](guides/npm-publish.md): `npm version patch|minor|major` followed by `npm publish --access public`, run locally by a person.

**This means two separate version-bumping mechanisms exist in the same repo, and how they're meant to coordinate isn't documented anywhere**: semantic-release computes and commits a version bump automatically on every push to `main`/`dev` (via `@semantic-release/git`, committing `package.json`/`CHANGELOG.md`/`pnpm-lock.yaml` back to the branch); `npm-publish.md`'s manual process also runs `npm version`, which bumps `package.json` and creates its own git tag. Whether these are meant to be used together, or `npm-publish.md`'s manual flow supersedes semantic-release's automatic one for actual releases, isn't stated anywhere in either the workflow or the guide. Until that's clarified, follow [`npm-publish.md`](guides/npm-publish.md) as the authoritative step-by-step for actually getting a new version onto the npm registry — it's the only path that includes the `npm publish` step at all.

(Verdaccio is not part of this flow — it was removed from the platform entirely; see [ADR-007](adr/007-verdaccio-registry.md).)

---

## 4. What "deploying" actually involves

See [CICD.md](CICD.md) for the full workflow inventory and [`runbooks/deploy-rollback.md`](runbooks/deploy-rollback.md) for the operational procedure — not repeated here. In short: a deploy is `make ci-deploy-<env>`, always a fresh clone-and-build, never a promoted artifact. Post-deploy verification is the same health-check reference used for restarts — see [`runbooks/restart-services.md`](runbooks/restart-services.md) §3, including which service healthchecks are weaker signal than others.

---

## 5. Hotfix process

Because there's no promotion pipeline to bypass, a "hotfix" on this platform is procedurally identical to a normal change: commit the fix to the affected application repo's `dev` branch (the branch every environment currently builds from, per §0 above), then either wait for `arm-deploy-make`'s own branch push to trigger the relevant deploy workflow, or manually trigger it via `workflow_dispatch` in GitHub Actions if a push already happened and needs a manual kick. There is no separate "hotfix branch" convention documented anywhere in this workspace — don't invent one without confirming it against how `repos.conf`'s single-branch-per-repo model would actually handle it (it currently can't target one environment without affecting the others, per [`runbooks/deploy-rollback.md`](runbooks/deploy-rollback.md) §3's identical finding about rollback).

---

## Related documents

- [CICD.md](CICD.md) — the full workflow inventory this document's §1–§2 summarize
- [`runbooks/deploy-rollback.md`](runbooks/deploy-rollback.md) — the operational deploy/rollback procedure, and the identical `repos.conf` branch-pinning finding from the rollback angle
- [`runbooks/restart-services.md`](runbooks/restart-services.md) §3 — post-deploy health verification
- [`docs/guides/npm-publish.md`](guides/npm-publish.md) — the authoritative manual steps for actually publishing `@aracreate/test-arm-ui`
- [ADR-007](adr/007-verdaccio-registry.md) — Verdaccio's removal, relevant to this document's §3
