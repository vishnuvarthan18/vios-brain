# Conventions

The canonical reference for the conventions every repository here follows. New repos are set up to match the structure and standards documented here, and each project's own README links back to this repo rather than repeating the rules.

Conventions are grouped by domain, one folder each. **A folder exists when there is a convention to put in it** — the set of folders is the catalogue, so a tool or a platform with nothing written for it yet has no folder. Reference material that is not a convention lives in [`wiki/`](wiki/).

## What lives here

| Folder | Covers |
| --- | --- |
| [repo/](repo/) | Repo structure, copyright, file headers, naming, Makefile, motd, environment, versioning |
| [git/](git/) | Commit message format, plus the platform document for GitHub |
| [wiki/](wiki/) | Reference notes that are not conventions — currently the [toolchain](wiki/toolchain.md) |

Read [`git/git-conventions.md`](git/git-conventions.md) whatever the repo, then the platform document for wherever it is hosted. **Commit format is defined there once and nowhere else**, because `semantic-release` parses it to decide version bumps and write `CHANGELOG.md`.

## Starting a new repo

Start from [project-template](https://github.com/vishnuvarthan18/project-template) with **Use this template**, then follow [repo §1](repo/readme.md#1-starting-a-repo). The template carries the full folder set, the `Makefile`, `scripts/motd`, `VERSION`, `LICENSE`, `.gitignore`, the GitHub workflows and the issue and PR templates, with placeholders ready to replace.

The scaffold is a separate repository rather than a folder in this one so that **Use this template** works — GitHub only offers it at repository level. Keep the two in step: a structural change here gets the matching change there in the same sitting.

## Maintaining this

This repo is only worth having if it stays true, so:

- **It follows its own conventions.** Same commit format, same headers, same `VERSION`. A rule that is annoying to follow here is annoying everywhere, and that is the signal to change the rule rather than the exception.
- **Every change is a commit with the reason in the body.** `git log` then answers *why is this rule here*, which is the question that kills undocumented standards.
- **One rule, one place.** The moment a rule appears in two documents the copies diverge, and the one nearer the work wins. Link, never restate.
- **A rule earns its place by having been broken.** Do not pre-write conventions for situations that have not come up. Speculative rules are what make a conventions repo feel dead.
- **Tag releases.** A project scaffolded against a specific version can link the tag rather than a moving `main` that silently invalidates it.
- **Check for drift periodically.** Open the two or three most active repos and compare headers, structure and commit messages against these documents. Then either fix the repo or change the rule — the one thing not to do is notice and leave it.

## License

MIT. See [LICENSE](LICENSE).
