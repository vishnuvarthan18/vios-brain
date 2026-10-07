# GitHub Conventions

**What this is.** How work is tracked, branched, reviewed and released on GitHub. Commit format is not here — it is in [git-conventions](git-conventions.md), defined once because release tooling parses it.

**Why it is separate.** Everything on this page is a property of the host. Branch protection, pull requests, labels and Actions exist because the repo lives on GitHub; move it elsewhere and this document is replaced while [git-conventions](git-conventions.md) survives unchanged.

## Contents

| Part | Covers |
| --- | --- |
| [1 Branches](#1-branches) | Naming · protection · lifetime |
| [2 Issues](#2-issues) | Templates · labels · what gets an issue |
| [3 Pull requests](#3-pull-requests) | Title · description · merge strategy |
| [4 Actions](#4-actions) | Required checks · secrets |
| [5 Releases](#5-releases) | Tags · changelog · what ships |
| [6 Repository settings](#6-repository-settings) | The checklist for a new repo |

## 1 Branches

`main` is the default branch and is always releasable. Nothing is committed to it directly.

Work branches are named `<type>/<issue>-<slug>`, where `<type>` is a commit type from [git-conventions §1](git-conventions.md#1-format):

```
feat/12-oauth-login
fix/48-timeout-on-cold-start
docs/51-api-reference
chore/60-bump-node-22
```

The issue number makes the branch traceable without opening it, and reusing the commit types means one vocabulary rather than two. Omit the number for work with no issue: `chore/rotate-deploy-key`.

**Branch protection on `main`** — required for every repo, set at creation:

- Require a pull request before merging.
- Require status checks to pass, and require branches to be up to date first.
- Block force pushes and deletion.
- Allow the release job to push (`VERSION`, `CHANGELOG.md` and the tag) — see [§5](#5-releases).

**Delete the branch on merge.** Enable it in repository settings so it happens without anyone remembering. A merged branch left behind is a branch someone eventually commits to.

**Branches are short.** A branch that has been open long enough to need a rebase against a moved `main` is a branch that should have been split into two.

## 2 Issues

**What gets an issue.** Anything that takes more than one commit, anything anyone else might ask about, and anything you would otherwise write down somewhere less durable. A one-line fix you are making right now does not need one.

**Templates** live in `.github/ISSUE_TEMPLATE/` and ship with the project scaffold. Two forms — bug and feature — plus a `config.yml` that turns off blank issues, because a blank issue is a title and nothing else three months later.

**Labels.** A fixed, small set, prefixed so they group in the picker. Add to it deliberately; a label nobody filters on is noise in every list.

| Prefix | Labels |
| --- | --- |
| `type:` | `bug` `feature` `docs` `chore` |
| `area:` | one per major component of the repo — defined per project, not here |
| `priority:` | `now` `next` `later` |
| standalone | `blocked` `good-first-issue` |

`priority:` is three values on purpose. A five-point scale collapses into "everything is a 2" within a month.

## 3 Pull requests

**Title follows the commit format.** `feat: add oauth login`, not `Add OAuth login`. Squash-merge uses the PR title as the commit subject, so a title in the wrong shape produces a commit that release tooling silently ignores.

**The description carries the closing keyword.** `Closes #12` on its own line. This is the one place a closing keyword belongs — merging the PR closes the issue at exactly the moment the work lands. Commit messages keep the bare `#12` trailer and no keyword ([git-conventions §2](git-conventions.md#2-reference-trailer)).

Beyond that the description says what changed and why, and names anything the diff cannot show: a manual migration step, a setting that has to change in the deploy environment, a decision that was close.

**Squash-merge is the default.** One issue, one commit on `main`, and the branch's working history — the false starts, the typo fixes — does not become permanent. Merge commits are for the rare branch whose individual commits each stand on their own and are each worth keeping; rebase-merge is off.

**Review.** Self-merging is fine on a solo repo, but open the PR anyway and read the diff on the PR page before merging. Reading your own change in a different view catches things the editor does not.

**Draft PRs** are for work you want CI to run on before it is ready. Open one early rather than pushing to a branch nobody is watching.

## 4 Actions

Two workflows ship with the scaffold:

| Workflow | Trigger | Does |
| --- | --- | --- |
| `ci.yml` | pull request, push to `main` | Install, lint, test. This is the required status check. |
| `release.yml` | push to `main` | Runs `semantic-release` — see [§5](#5-releases). |

**Pin actions to a major version** (`actions/checkout@v4`). Pin to a commit SHA for any action outside the `actions/` and `github/` namespaces, because a tag on a third-party action can be moved under you.

**Secrets are named `SCREAMING_SNAKE_CASE`** and scoped to the repository unless more than one repo genuinely needs the same value, in which case they go at the account level. Never read a secret into a step that echoes it; a masked value still leaks through a `set -x`.

**Keep CI fast enough to wait for.** If the required check takes longer than a few minutes, split the slow part into a job that runs on `main` only and leave the fast checks blocking the PR.

## 5 Releases

`semantic-release` runs in `release.yml` on every push to `main` and does all of it: reads the commits since the last tag, decides the bump, writes `VERSION` and `CHANGELOG.md`, commits them, tags, and publishes a GitHub Release with the generated notes.

**Consequences of that, all of which are the point:**

- **Never edit `CHANGELOG.md` by hand.** It is generated. A manual edit is overwritten at the next release, and the commit that should have carried the note is the place to fix it.
- **Never create a tag by hand.** A hand-made tag desynchronises the tool's idea of the last release from the repo's, and the next automated release either skips versions or fails.
- **Never run `semantic-release` from a laptop.** With no CI environment detected it silently dry-runs while printing a full success message ([git-conventions §5](git-conventions.md#5-publishing)).
- **The release commit is `chore(release): <version> [skip ci]`.** The skip marker keeps it from triggering the workflow again.

**Version zero.** `VERSION` starts at `0.0.1` and stays below `1.0.0` until the project has something depending on it. Until then a `feat` bumps the minor and nothing is promised.

**What ships in `releases/`** is separate from the tag: built artefacts, fabrication outputs, anything a consumer downloads rather than builds. The tag records the state of the source; `releases/` holds the product of it.

## 6 Repository settings

The checklist for a new repo, in order. Everything here is once-only and easy to forget.

1. Create from the [project template](https://github.com/vishnuvarthan18/project-template) — "Use this template", not a fork.
2. Set the description and topics. An untagged repo is invisible in your own list within a year.
3. **Settings → General**: enable "Automatically delete head branches"; disable rebase-merge and merge-commit, leave squash-merge on; set squash-merge commit message to "Pull request title and description".
4. **Settings → Branches**: protect `main` per [§1](#1-branches).
5. **Settings → Actions → General**: set workflow permissions to read and write, so the release job can push.
6. Add the labels from [§2](#2-issues).
7. Replace the placeholders in the scaffold and regenerate `scripts/motd` — see [repo §1.1](../repo/readme.md#11-from-the-template).
8. Set `VERSION` to `0.0.1` and make the first commit `chore: initial project scaffold`.
