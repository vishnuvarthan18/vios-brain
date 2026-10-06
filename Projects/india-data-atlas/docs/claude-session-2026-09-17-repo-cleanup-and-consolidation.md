# Session 2026-09-17 — local repo cleanup, consolidation and sync

_Continues `session-2026-09-17-console-redesign-and-hardening.md`. No platform code changed and nothing was deployed. This session was housekeeping on the Mac: cleaning the repos, putting them in one folder, and getting every one backed up on GitHub._

## Result

All 10 repos live under `~/india-platform/` and **all 10 are on GitHub and in sync with origin**. The console's first push landed 2026-09-21. The ops console deploy is still the one open blocker.

## 1. Cleanup

- **8 stale `.git/index.lock` files removed**, one in every repo. They would have blocked the next commit anywhere.
- 18 `__pycache__` dirs and 64 `.pyc` files cleared (outside the venv).
- `README.md.d1-era.bak` deleted from `india-data-platform`.
- Left alone on purpose: `harvest-engine/.venv` (263M of the platform repo's 265M, gitignored, working) and `india-data-core/Claude outputs/` (4 untracked HTML previews). Both are the user's call.

## 2. Consolidation into `~/india-platform/`

```
~/india-platform/
  india-data-core/        india-data-platform/    india-ops-console/
  india-culture-engine/   india-extinct-engine/   india-forest-engine/
  india-geo-engine/       india-laws-engine/      india-species-engine/
  india-water-engine/
```

- `india-ops-console` was lifted out of `india-data-platform`, which ends the nested-repo problem.
- Each repo was checked after the move by file count and `git log`: same HEAD, same files. The 9 old `~/india-<name>` folders were then deleted by the user with `rmdir`.
- **Breaks:** `CORE_REPO` for the console's `./scripts/run-tests.sh` must now point at `~/india-platform/india-data-core`, or the 99 database-backed tests fail. Editor workspaces need updating too. The VPS is unaffected.

## 3. GitHub sync — the VPS is a second author

Both pushes from the Mac were rejected at first. The Mac was **2 commits behind** on `india-data-platform` and **1 behind** on `india-culture-engine`, because earlier sessions pushed from the VPS to the same branches. Nobody knew until push time.

Fixed by rebasing, not merging, and never force-pushing:

| Repo | Remote had | Pushed as |
|---|---|---|
| `india-data-platform` | `9d523c2`, `199d4f4` (D-66–D-70 deploy work, Overpass grid note) | `9f2bd16` |
| `india-culture-engine` | `ff6219e` (TLS cert, secrets, systemd unit) | `87400ba` |

**The D-71 fix is now on GitHub.** Before this it lived only on the Mac and the VPS, so a fresh clone would have brought back the monthly false failure on `culture-fra-jk`. Checked on the pushed commit: line 247 reads `return 0 if status in ("success", "partial") else 2`.

**Rule going forward:** run `git fetch` on the Mac before starting local work in any of these repos.

## 4. `india-ops-console` gets a remote

- `origin` is set to `https://github.com/vishnuvarthan18/india-ops-console.git`, under the personal account like the other nine.
- The branch was renamed from `master` to `main`.
- **Secret scan before first push:** only `.env.example` and `control/ops-control.env.example` are tracked, and both have empty values. The one history match is a test fixture literal (`DATA_GOV_IN_API_KEY=(secret, removed) Nothing needed scrubbing.
- **First push done 2026-09-21** (`* [new branch] main -> main`, 144 objects), with upstream tracking set. The console is no longer stored only on one Mac.

## 5. Limits of working through the device bridge

These cost several round trips. Read this before the next session touches these repos from the cloud:

- **The shell on the Mac has no git identity and no GitHub credentials.** It cannot commit, fetch or push. Private repos can't even be fetched. Commits and pushes have to be run by the user in Terminal.
- **Plain `git status` in that shell leaves an `index.lock`** it cannot clean up. It happened twice and blocked the user's own rebase both times. Use `git --no-optional-locks status` for any read-only check.
- A rebase run from that shell stopped halfway because it had no identity. It was aborted cleanly with nothing lost. Don't try it again.
- Each connected folder is its own mount. Moving a repo between folders copies it and then deletes the original, and the mount root can't be removed while connected (`Device or resource busy`). Delete permission has to be granted per folder.

## Still open (unchanged)

The deploy is still the blocker. See `open-items-next-session.md` for the full list.
