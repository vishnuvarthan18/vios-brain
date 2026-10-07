# Setup

Three repositories, built and verified, with every link pointing at
`github.com/vishnuvarthan18`. No placeholders left — push as-is.

**They are on your machine**, unpacked at `~/Desktop/github-setup/`. The three
`.zip.done` files sitting alongside them are the spent archives — delete them,
I was not permitted to.

The commands below use `gh`. Without it, create the repo on github.com first and
swap the last line for:

```sh
git remote add origin git@github.com:vishnuvarthan18/<repo>.git
git push -u origin main
```

## 0 The short way

```sh
cd ~/Desktop/github-setup
bash push.sh
```

It runs `git init`, makes the first commit in each of the three repos, and wires
up `origin` — using `gh` to create the repos if you have it, and printing what
to do at github.com/new if you do not. **It does not push.** It prints the three
push commands and stops, so you read the commits first.

Sections 1–3 below are the same thing by hand.

## 1 Push `conventions`

```sh
cd conventions
git init -b main
git add .
git commit -m "chore: add conventions repo"
gh repo create conventions --public --source=. --push
```

Public matters — every project README links here, and a private target makes
those links dead for anyone but you.

## 2 Push `project-template`

```sh
cd ../project-template
git init -b main
git add .
git commit -m "chore: add project scaffold"
gh repo create project-template --public --source=. --push
```

Then **Settings → General → tick "Template repository"**. Without it there is no
"Use this template" button and the whole scaffold reverts to copy-paste.

## 3 Push the profile repo

The repo name must match your username **exactly**, or GitHub does not render it
on your profile — and it fails silently.

```sh
cd ../vishnuvarthan18
git init -b main
git add .
git commit -m "chore: add profile readme"
gh repo create vishnuvarthan18 --public --source=. --push
```

Public is required too; a private profile repo renders nothing. It appears on
[github.com/vishnuvarthan18](https://github.com/vishnuvarthan18) within a minute.

Before pushing, check the two things I had to guess: the display name at the top
(`Vishnu Varthan`) and the positioning line under it. The `Elsewhere` section has
LinkedIn and website badges commented out — uncomment and fill in the URLs, or
leave them out.

## 4 Settings on the two code repos

Per [github-conventions §6](conventions/git/github-conventions.md#6-repository-settings):

- **Settings → General** — enable "Automatically delete head branches"; disable
  rebase-merge and merge-commit, leave squash-merge on; set the squash commit
  message to "Pull request title and description".
- **Settings → Branches** — protect `main`: require a PR, require status checks,
  block force pushes and deletion.
- **Settings → Actions → General** — workflow permissions to **read and write**,
  or the release job cannot push `VERSION`, `CHANGELOG.md` and the tag.
- Add the labels: `type:bug` `type:feature` `type:docs` `type:chore`,
  `priority:now` `priority:next` `priority:later`, `blocked`, `good-first-issue`.

## 5 Starting a project from the template

```sh
# "Use this template" on GitHub, then:
make identity SLUG=my-project NAME="My Project"
rm scripts/set-identity.sh          # and delete the 'identity' target in the Makefile
```

### The banner

`make identity` regenerates `scripts/motd`, which needs a figlet. Either:

```sh
brew install figlet          # then check the font exists:
figlet -f "ANSI Shadow" TEST
```

If that font is missing, Homebrew's figlet did not ship it — install the font
into `/opt/homebrew/share/figlet/`, or take the simpler route:

```sh
pip3 install pyfiglet
```

`scripts/gen-motd.sh` tries figlet first, falls back to pyfiglet, and exits with
a message if neither is there rather than writing a broken banner. Install one
before `make identity`.

Both were tested end to end: `make identity` rewrites every placeholder and
regenerates the ANSI Shadow banner; `make setup` creates `.env` from
`.env.example` and, on an existing `.env`, appends only the missing keys without
touching a value already set.

## 6 The first real test

Retrofit one existing repo — pick the most active of your seven — to these
conventions end to end: structure, file headers, commit format, branch
protection. Anything the retrofit shows to be unworkable gets fixed **in the
conventions**, not worked around in the project. Until that has happened once,
these are rules nothing has tested.

## Assumptions made

Each is a single find-and-replace if wrong:

| Assumed | Where it appears |
| --- | --- |
| MIT licence | `LICENSE`, every SPDX header, `README.md` |
| `Copyright (C) 2026, Vishnu` | `LICENSE`, file headers, `scripts/motd`, `Makefile` |
| `Vishnu <(removed)>` | `Author:` line in file headers |
| pnpm + Node 22, uv + Python 3.12 | `wiki/toolchain.md`, template `Makefile`, `ci.yml` |
| Repo names `conventions` and `project-template` | Links throughout both repos |

The copyright holder is the human name, not the GitHub handle — change it to
whatever should appear on a licence if that is wrong.

## What is deliberately not here

- **No `.github` organisation repo.** That is an organisation feature and does
  nothing on a personal account. The profile repo in §3 is the equivalent.
- **No firmware or hardware `src/` layouts.** Add one the day there is such a
  repo; a layout written for work nobody is doing is a rule with nothing behind
  it.
