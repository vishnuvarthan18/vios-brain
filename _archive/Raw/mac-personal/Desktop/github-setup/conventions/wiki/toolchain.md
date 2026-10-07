# TOOLCHAIN

**This is a reference note, not a convention.** It records what is currently used and why. Nothing here is binding — a project with a reason to choose differently should choose differently and say so in its own README. What *is* binding is [repo](../repo/readme.md) and [git](../git/git-conventions.md): the structure, the headers and the commit format hold regardless of which tools sit inside them.

Revise this page when a choice actually changes. A toolchain note that lists what was used two years ago is worse than no note, because it is read as current.

## Web and application software

| Layer | Tool | Why |
| --- | --- | --- |
| Runtime | Node 22 LTS | LTS, so the support window outlasts the project |
| Language | TypeScript, `strict` | Not partially typed. Half-strict costs the discipline and gives back none of the guarantees |
| Framework | Next.js (App Router) | Routing, rendering and API surface in one thing, deployable anywhere |
| Package manager | pnpm | Fast, strict about phantom dependencies, sane in a monorepo. `pnpm-lock.yaml` is committed |
| Styling | Tailwind CSS | The utility set is a design system with no maintenance cost. Tokens in `styles/` for anything reused |
| Lint / format | ESLint + Prettier | Formatting is not a thing to have opinions about. `make lint` in CI, blocking |
| Testing | Vitest, Playwright for end-to-end | Vitest shares the build config, so tests do not need a second toolchain |
| Database | PostgreSQL | Default until something makes it wrong. Migrations are committed and forward-only |
| Deploy | Vercel for Next.js, container for anything else | Whatever it is, it deploys from `main` on a green build |

## Data, ML and research

| Layer | Tool | Why |
| --- | --- | --- |
| Language | Python 3.12 | Pinned per project in `.python-version` |
| Environment | uv | Resolves and installs fast enough that recreating an environment is not a decision. `uv.lock` is committed |
| Project file | `pyproject.toml` | One file for dependencies, tool config and packaging |
| Notebooks | Jupyter, with `nbstripout` as a git filter | Outputs never reach git — see [data layout](../repo/layouts/data.md#notebooks) |
| Core | pandas, NumPy, scikit-learn | The baseline before anything heavier is justified |
| Deep learning | PyTorch | When the problem needs it, which is less often than it looks |
| Experiment tracking | MLflow, local backend | Runs, params and metrics logged automatically or not at all |
| Lint / format | Ruff | Lint and format in one tool, fast enough to run on save |
| Testing | pytest | Including on pipelines — an untested pipeline is a result nobody can defend |
| Large files | Git LFS above 1 MB, external storage above 100 MB | Thresholds in the [data layout](../repo/layouts/data.md#the-data-folder) |

## Shared

| Layer | Tool | Why |
| --- | --- | --- |
| Task runner | GNU Make | Not because make is good, but because `make dev` means the same thing in every repo regardless of what is inside |
| CI | GitHub Actions | Already where the code is. Fast checks block the PR, slow ones run on `main` |
| Releases | semantic-release | Version and changelog derived from commits, in CI only — see [github-conventions §5](../git/github-conventions.md#5-releases) |
| Banner | figlet, ANSI Shadow | `make help` prints the project name, which is how you notice you are in the wrong terminal |
| Secrets | GitHub repository secrets; `.env` locally, never committed | `.env.example` is the contract — see [repo §4.3](../repo/readme.md#43-environment) |
| Editor config | `.editorconfig` | Settles indentation and line endings before a formatter has to |

## Deliberately not used

Worth recording, so the same discussion does not happen twice:

- **Rebase-merge and merge commits on `main`.** Squash-merge only, so one issue is one commit — see [github-conventions §3](../git/github-conventions.md#3-pull-requests).
- **Hand-written changelogs.** Generated from commits, which is the entire reason the commit format is strict.
- **Committed notebook outputs.** Unreadable diffs, and data leaking into git history.
- **Version numbers anywhere but `VERSION`.** Every duplicate is a copy that will be wrong.
