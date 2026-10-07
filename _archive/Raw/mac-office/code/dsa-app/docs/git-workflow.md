# Git Workflow & Conventional Commits

This document is the source of truth for branching, commit messages, pull requests, and releases on **DreamSpace Academy (dsa-app)**. Follow the rules here whenever you create branches, write commits, or cut releases — both for human developers and for Claude Code.

It is adapted from the proven workflows on Viyanix and KathiraGreens. Two things differ on dsa-app and are baked in below:

- This is a **pnpm monorepo with four workspaces** (`api/`, `web/`, `mobile/`, `public/`), not a two-workspace npm repo.
- Before opening a PR, also run the **[Known-Issues Verification Checklist](./KNOWN-ISSUES-CHECKLIST.md)** for any area you touched.

## Branch Model

dsa-app uses a Git Flow style model with long-lived branches and short-lived supporting branches.

- **`main`** — Production. Protected. Every commit on `main` is deployable. Only receives merges from `staging` (release PRs) or `hotfix/*` branches. Production tags `vX.Y.Z` are cut from `main`.
- **`staging`** — Release candidate. Protected. Hosts the next release candidate for QA. After sign-off, `staging` is merged into `main` and tagged. (Create when the first release candidate is cut; until then PR straight to `develop`.)
- **`develop`** — Integration branch. Protected. Default branch for new work. All `feature/*`, `bugfix/*`, `chore/*`, and `docs/*` branches PR into `develop`.
- **Supporting branches** — Short-lived, deleted after merge:
  - `feature/*` — new functionality
  - `bugfix/*` — non-urgent bug fixes
  - `hotfix/*` — urgent production fixes (branch from `main`)
  - `release/*` — release preparation (version bump, changelog)
  - `chore/*` — tooling, dependency bumps, non-functional housekeeping
  - `docs/*` — documentation-only changes

```
                feature/*  bugfix/*  chore/*  docs/*
                       \      |      /      /
                        \     |     /      /
                         v    v    v      v
                         +----------------+
                         |    develop     |
                         +----------------+
                                  |
                                  v
                            release/X.Y.Z
                                  |
                                  v
                         +----------------+
                         |    staging     |
                         +----------------+
                                  |
                                  v
                         +----------------+
                         |     main       |---> tag vX.Y.Z
                         +----------------+
                                  ^
                                  |
                              hotfix/*
                                  |
                              (also merged back to develop)
```

## Branch Naming

Use kebab-case after the prefix slash. Keep names short but descriptive.

| Prefix | Purpose | Source | Target | Example |
|---|---|---|---|---|
| `feature/` | New functionality | `develop` | `develop` | `feature/attendance-offline-sync` |
| `bugfix/` | Non-urgent bug fix | `develop` | `develop` | `bugfix/payment-cron-duplicate` |
| `hotfix/` | Urgent production fix | `main` | `main` + `develop` | `hotfix/cert-verify-base-url` |
| `release/` | Release prep | `develop` | `staging` then `main` | `release/1.2.0` |
| `chore/` | Tooling, deps, housekeeping | `develop` | `develop` | `chore/bump-prisma-6` |
| `docs/` | Docs-only changes | `develop` | `develop` | `docs/git-workflow` |

## Workflow

Branch off `develop` and merge back into `develop` via PR.

1. Pull latest `develop` before starting new work.

   ```bash
   git checkout develop
   git pull origin develop
   ```

2. Create your branch.

   ```bash
   git checkout -b feature/attendance-offline-sync
   ```

3. Commit often using Conventional Commits (see below).

   ```bash
   git add api/src/routes/attendance.ts
   git commit -m "feat(attendance): upsert on offline sync to dedupe double-sync"
   ```

4. Push and open a PR into `develop` — **only after human verification** (see Claude Code rules).

   ```bash
   git push -u origin feature/attendance-offline-sync
   gh pr create --base develop --title "feat(attendance): upsert on offline sync to dedupe double-sync"
   ```

5. Request review.

6. Address review feedback with additional commits on the same branch. Do not force-push a branch someone else is reviewing unless you coordinate first.

7. Merge via **Squash and merge** (recommended for most PRs) or a regular merge commit when the per-commit history is meaningful (e.g. release branches). Avoid rebase-merge on shared PRs.

8. Delete the branch after merge — remotely (GitHub does this on squash) and locally:

   ```bash
   git checkout develop
   git pull origin develop
   git branch -d feature/attendance-offline-sync
   ```

9. Sync long-running branches with `develop` frequently. Prefer rebase while the branch is unshared, switch to merge once others are reviewing.

   ```bash
   git fetch origin
   git rebase origin/develop      # unshared branch
   # or
   git merge origin/develop       # shared / under review
   ```

## Conventional Commits

All commits on all branches must follow Conventional Commits.

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | Description | Example |
|---|---|---|
| `feat` | New feature visible to a user | `feat(portfolios): coordinator approve → public` |
| `fix` | Bug fix | `fix(payments): cap paidAmount at remaining balance` |
| `docs` | Documentation only | `docs(api): document cert verify endpoint` |
| `style` | Formatting, whitespace, semicolons | `style(web): prettier on attendance pages` |
| `refactor` | Code change that is not a feature or fix | `refactor(rbac): extract can() resolution order` |
| `perf` | Performance improvement | `perf(api): index Payment(studentId,month)` |
| `test` | Add or update tests | `test(attendance): cover lock-after-submit override` |
| `chore` | Maintenance, no production code change | `chore(deps): bump prisma to 6.2` |
| `ci` | CI configuration | `ci: cache pnpm store on GitHub Actions` |
| `build` | Build system, bundler, package manager | `build(web): enable vite-plugin-pwa precache` |
| `revert` | Revert a previous commit | `revert: feat(portfolios): coordinator approve → public` |

### Scopes

Pick the most specific scope that fits. Module scopes mirror the CLAUDE.md domains.

| Scope | Area |
|---|---|
| `api` | Express server, generic backend |
| `web` | React PWA, generic frontend (all roles) |
| `mobile` | Capacitor app (derived slice) |
| `public` | Next.js no-auth app (verify + showcase) |
| `auth` | Auth0 (staff) + phone-OTP (parents), JWT, sessions |
| `rbac` | Roles, `can()` matrix, permission overrides, hub scope |
| `hubs` | Hubs (Hatton, Batticaloa) and hub config |
| `users` | User CRUD, profile, staff-platform link |
| `students` | Enrollment, DSA IDs, slot assignment, demographics |
| `consent` | PDPA consent, T&C acceptance, data requests |
| `sessions` | Timetable slots, session instances, cover trainer, holidays |
| `attendance` | Mark, lock, override, offline sync, outcomes, proof of work |
| `payments` | Monthly fee, payment cron, sponsorship, waive/reverse |
| `receipts` | Receipt generation (`RCP-XXXXXX`) |
| `portfolios` | Portfolio approval flow, MongoDB body |
| `certs` | Certificate flow, QR verify |
| `notifications` | In-app (Realtime) + SMS + push + email + prefs |
| `announcements` | Broadcast announcements + rate limit |
| `open-sessions` | Categories, tags, tiers, registration, waitlist |
| `reports` | Impact reports, CSV exports |
| `audit` | Audit log + viewer |
| `avatar` | DSA robot avatar generator |
| `jobs` | Cron jobs (`api/src/jobs/`) |
| `db` | Prisma schema, migrations, sequences, indexes |
| `infra` | Deployment (Railway, Vercel), Nixpacks |
| `deps` | Dependency upgrades |
| `docs` | Project docs |
| `config` | Env vars, runtime config |

### Subject rules

- Imperative mood: "add", "fix", "remove" — not "added", "fixes", "removing".
- Lowercase the first letter. No trailing period. Max 72 characters.

### Body rules

- Optional, but include one when the change is not obvious from the subject.
- Wrap at 72 characters. Explain **why** — the diff already shows **what**.

### Footer rules

- `Closes #123` — auto-closes the linked issue on merge to default.
- `Refs #456` — references without closing.
- `BREAKING CHANGE: <description>` — see Breaking Changes.
- `Co-Authored-By: Name <email>` — when pairing.

### Good example

```
fix(payments): make monthly cron idempotent on re-run

The 1st-of-month job created duplicate Payment rows when the cron
fired twice. Rely on @@unique([studentId, month]) and switch to
upsert so a second run is a no-op rather than a constraint crash.

Closes #42
```

### Bad examples

```
update stuff                  # no type, no scope, no content
Fixed bug.                    # past tense, capitalised, period
feat: WIP                     # not shippable, not descriptive
```

## Breaking Changes

Mark breaking changes with `!` after the type/scope or a `BREAKING CHANGE:` footer. Either bumps MAJOR on the next release.

```
feat(api)!: require Idempotency-Key on POST /payments

BREAKING CHANGE: payment writes without an Idempotency-Key header
are now rejected with 400. Clients must send a UUID per attempt.
```

## Pull Request Flow

### Title

The PR title must match Conventional Commits format and becomes the squash-merge commit message.

### Description template

```
## Summary
One or two sentences on what this PR does.

## Why
Context. Link to issue or feature spec if relevant.

## How to test
Step-by-step manual test plan. Include seed data or env vars needed.
(AUTH_MODE=mock works with no external services.)

## Screenshots
For UI changes only. Mobile and desktop. Verify <480px.

## Related Issues
Closes #123

## Checklist
- [ ] Typecheck passes (`pnpm typecheck`)
- [ ] API smoke harness green (`cd api && pnpm exec tsx scripts/smoke.ts`)
- [ ] API unit tests pass (`cd api && pnpm test`)
- [ ] Web builds (`cd web && pnpm build`)
- [ ] TypeScript strict — zero errors, no `any`
- [ ] Prisma migration committed if schema changed
- [ ] Known-Issues Checklist run for the area touched (docs/KNOWN-ISSUES-CHECKLIST.md)
- [ ] Tamil default respected for Hatton/Batticaloa strings (no hardcoded English)
- [ ] No student names/photos in operational UI (DSA ID only)
```

### Review rules

- At least one approval before merge. No self-merge without approval (exception: `docs/*` PRs with no functional change, after notifying the other developer).
- Resolve all review comments before merge — change the code or reply with a justification.
- CI must be green.

## Releases and SemVer

Versions follow Semantic Versioning: `vMAJOR.MINOR.PATCH`.

| Bump | Trigger |
|---|---|
| MAJOR | Breaking change (`feat!`, `fix!`, or `BREAKING CHANGE:`) |
| MINOR | Backwards-compatible new feature (`feat`) |
| PATCH | Backwards-compatible bug fix (`fix`, `perf`) |

`CHANGELOG.md` follows Keep a Changelog: **Added / Changed / Deprecated / Removed / Fixed / Security**. The first production release is `v1.0.0`, cut after deployment sign-off.

## Release Process

```bash
# 1. Cut release branch from develop
git checkout develop && git pull origin develop
git checkout -b release/1.2.0

# 2. Bump versions in each workspace package.json + update docs/CHANGELOG.md
git commit -am "chore(release): prepare 1.2.0"
git push -u origin release/1.2.0

# 3. PR release/1.2.0 into staging, deploy, run QA
gh pr create --base staging --title "chore(release): 1.2.0"

# 4. After QA sign-off, PR staging into main
gh pr create --base main --head staging --title "release: 1.2.0"

# 5. Tag on main
git checkout main && git pull origin main
git tag -a v1.2.0 -m "Release 1.2.0"
git push origin v1.2.0

# 6. Merge main back into develop so tags + bumps flow back
git checkout develop && git pull origin develop
git merge --no-ff main && git push origin develop

# 7. Delete the release branch
git branch -d release/1.2.0 && git push origin --delete release/1.2.0
```

## Hotfix Process

Hotfixes branch from `main`, take a PATCH bump, and must be merged back into `develop` (and `staging`) so the fix is not lost.

```bash
git checkout main && git pull origin main
git checkout -b hotfix/cert-verify-base-url
git commit -am "fix(certs): read verify base from PUBLIC_VERIFY_BASE_URL"
git push -u origin hotfix/cert-verify-base-url
gh pr create --base main --title "fix(certs): read verify base from PUBLIC_VERIFY_BASE_URL"
# after merge: tag patch on main, then merge main back into develop and staging
```

## Rules for Claude Code

Claude Code runs git commands on behalf of the developer. These rules are non-negotiable.

- **Never commit directly to `main`, `staging`, or `develop`.** Always branch first. If invoked on one of these branches, stop and create a feature branch.
- **Always use Conventional Commits** for every commit message, including squashed PR titles.
- **Always build and test before declaring work done.** The CI gate is `pnpm typecheck`, the API smoke harness (`cd api && pnpm exec tsx scripts/smoke.ts` — 106 checks, spins up its own embedded Postgres), the API unit tests (`cd api && pnpm test`), and the web build (`cd web && pnpm build`). Run at least the parts covering the workspaces you touched. Report failures with `file:line` and propose a fix.
- **Never force-push to shared branches** (`main`, `staging`, `develop`, or any branch with an open PR). Force-pushing your own unshared feature branch before review is allowed.
- **Never use `--no-verify`, `--no-gpg-sign`, or any hook-skipping flag** unless the user explicitly asks in the current session. If a hook fails, diagnose and fix the root cause.
- **Never push without explicit user approval.** Stage and commit freely, but wait for the user to say "push" or "open the PR".
- **Wait for human verification before opening a PR.** Even after commits land and build/tests are green, do not run `gh pr create` (and do not push for review) until the human has smoke-tested the running feature and explicitly confirms ("open the PR", "ready for PR", "push it"). A green build is **necessary but not sufficient** — a human eye on the running feature is required. Provide a short verification handoff first:
  1. Branch name and one-line summary.
  2. Files touched, grouped by workspace (`api/` / `web/` / `mobile/` / `public/`).
  3. Build / test status.
  4. Manual smoke-test steps for the golden path (URLs, sample inputs, expected result).
  5. Any flags, env vars, or seed commands needed first (e.g. `AUTH_MODE=mock`, `pnpm db:seed`).
  Then stop and wait.
- **Never tag releases automatically.** Only create or push a `vX.Y.Z` tag when the user says "release vX.Y.Z" or equivalent.
- **If unsure which branch to base on, ask.** Default is `develop`; use `main` only for `hotfix/*`.
- **Prefer a new commit over amending.** Amend only when asked, and never amend a commit already pushed to a shared branch.
- **Do not stage `.env`, credentials, or other secrets.** Stage files by name; avoid `git add -A` / `git add .` unless the user confirms there are no sensitive files in the tree.
- **Schema changes require a committed Prisma migration.** Never hand-edit the database without a migration in the same PR. The one-time SQL setup (sequences + audit-log `REVOKE`) is documented in HANDOVER — do not silently re-run it.

## Pre-PR Checklist

Before opening or marking a PR ready for review, confirm:

- `pnpm typecheck` succeeds with zero TypeScript errors (strict, no `any`).
- API smoke harness green: `cd api && pnpm exec tsx scripts/smoke.ts` (106 checks, embedded Postgres — no Docker needed).
- API unit tests pass: `cd api && pnpm test`.
- Web builds: `cd web && pnpm build` (tsc + vite + PWA generation).
- No `console.log`, `debugger`, or debug statements left in the diff.
- All commit messages follow Conventional Commits.
- Branch is rebased on / merged with the latest `develop`.
- No `.env`, credentials, or secrets committed.
- Prisma migration committed if `schema.prisma` changed.
- **[Known-Issues Verification Checklist](./KNOWN-ISSUES-CHECKLIST.md)** run for every area touched.
- **Human verification done** — change smoke-tested in the running app by the requester.

---

Last reviewed: 2026-05-31
Document version: 1.0
