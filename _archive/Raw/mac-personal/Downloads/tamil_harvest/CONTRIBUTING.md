# Contributing

Thanks for working on Semmozhi. This page is the whole workflow; the rest of the detail is in [docs/](docs/).

## Branches
- **`main`** is production. It deploys automatically to the live site. Never commit to it directly.
- **`dev`** is where work is integrated. Its preview site is https://dev.semmozhi.online.
- Work on a short-lived branch **from `dev`**: `feature/<topic>`, `fix/<topic>`, `content/<topic>`, `design/<topic>`, `engine/<topic>`.
- Open a pull request **into `dev`**. A release is a pull request from `dev` into `main`, opened by the owner.

## Every change
1. `git checkout dev && git pull && git checkout -b fix/short-name`
2. Make the change. Keep it to one purpose.
3. `make local` and look at the pages you touched, at desktop and phone width.
4. `make check` must pass. CI runs the same thing.
5. Commit with a clear message: what changed and why, present tense ("Fix broken nav on phone").
6. Push, open a pull request using the template, and ask for review.

## Rules that protect the project
1. **Only `website/` is published.** Do not link to or copy design pages, `design/showcase/`, `_checks`, or data folders into it.
2. **No secrets, ever.** No tokens, keys or passwords in the repo, issues or pull requests. Deploy credentials live in GitHub Secrets only. If one leaks, tell the owner immediately so it can be revoked.
3. **No licensed photos in git.** `design/references/` is git-ignored on purpose. Only public-domain, CC0 and CC-BY files may ship on the site, each with a credit. CC-BY-SA, CC-BY-NC and unclear-licence files are for measuring only. See [docs/CONTENT_RULES.md](docs/CONTENT_RULES.md).
4. **Do not invent facts.** Tamil text, dates, names and numbers need a source. Unchecked text must be labelled "sample · unverified".
5. **Do not rename the inner `tamil_harvest/` folders.** Scrapy needs the package name, and the Dockerfile and crawl workflow use these paths.
6. **Do not edit generated files by hand.** For example `design/svg_kit/identity_build.py` generates the identity SVGs and JSON.
7. **Never delete data to "clean up".** Move things to `archive/` and say so in the pull request.

## Where things go
| You are changing | Put it in | Merge target |
|---|---|---|
| A page, text or style on the live site | `website/` | `dev` |
| A design token or component | `design/showcase/css/`, then copy to `website/css/` (see [design/README.md](design/README.md)) | `dev` |
| A crawler, source or builder | `tamil_harvest/`, `engines/`, `scripts/` | `dev` |
| Docs | `docs/`, or the README next to the code | `dev` |

## Review
- One approval from a code owner (see `.github/CODEOWNERS`) before merging.
- Reviewers check: does it work at phone width, is anything unsourced, did anything private end up in `website/`.

## Getting help
Open an issue (templates: Bug, Content or source problem), or ask the owner.
