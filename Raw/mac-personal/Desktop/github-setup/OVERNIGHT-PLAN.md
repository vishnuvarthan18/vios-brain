# Repo cleanup — end-to-end plan

Audit first, fixes second. Nothing is changed tonight.

---

## Before anything: what I can and cannot do while you are away

I cannot run this for you. Your repos are private, my sandbox has no GitHub
credentials, and I am blocked from typing into your Terminal by macOS. Anything
that touches your account has to run from your machine, under your `gh` login.

So "overnight" means: **you start it, then go to bed.** It is not a job I can
trigger at 3am.

It also does not take all night. Fifteen repos, one API call each for the file
tree and one for commits — about **two to four minutes**. You will have the
report before you close the laptop.

---

## Tonight — one command

```sh
cd ~/Desktop/github-setup && bash audit.sh
```

Read-only. Every call is a GET. It creates no branches, no commits, no pull
requests, and changes no setting. The worst it can do is fail.

If you want it to survive the lid closing:

```sh
cd ~/Desktop/github-setup && caffeinate -i bash audit.sh
```

Output lands at `~/Desktop/github-setup/audit-<today>.md`.

---

## What the audit measures

Per repo, against [conventions](https://github.com/vishnuvarthan18/conventions):

| Checked | How |
| --- | --- |
| Expected files present | `README.md` `LICENSE` `VERSION` `.gitignore` `Makefile` |
| Expected folders present | `src/ docs/ tests/ releases/ scripts/`, plus `scripts/motd` |
| GitHub scaffolding | `.github/workflows/`, issue templates, PR template |
| Default branch | `main`, or still `master` |
| Commit format | share of the last 30 subjects matching `<type>: <summary>` |
| Housekeeping | description, topics, fork, archived |

It deliberately does **not** check file headers, README quality, or naming.
Those need reading the code, and a script that guesses at them produces
confident nonsense.

---

## In the morning — read the report, then decide

The report sorts every repo into one of four verdicts:

- **close** — a couple of files away
- **worth fixing** — a session's work
- **large gap** — decide whether the project has earned the effort
- **skip** — fork or archived

**The decision that matters is which repos are worth it at all.** A dead client
site from 2024 with twelve missing files is not a problem; it is a project the
conventions were never meant for. Bringing it into line buys nothing and costs
an evening.

My expectation: two or three repos are worth the full treatment, the rest want
a `README` and a `LICENSE` and nothing more.

---

## Phase 2 — the fixes, once you have chosen

For each repo you pick, in this order. The order matters: the cheap, safe,
reversible things first, so that if you stop halfway the repo is still better
than it was.

1. **Description and topics.** One `gh repo edit`. Thirty seconds, no risk.
2. **`LICENSE` and `.gitignore`.** Additive, cannot break a build.
3. **Default branch to `main`** where it is still `master`. GitHub retargets
   open PRs automatically; local clones need one command, which the report will
   print.
4. **`README` rewritten to the standard shape** — title, description, stack,
   commands, link back to the conventions. This is the one with real value and
   the one no script can do, because it requires knowing what the project is.
5. **`Makefile` with the standard targets.** Only where the project has commands
   worth wrapping. A `Makefile` whose targets echo "nothing to do" is worse than
   none.
6. **`.github/` scaffolding and workflows.** Last, because CI on a project you
   are not actively working in is noise.

**Commit history is not on this list.** The share of conforming commit messages
is a measurement, not a task. Rewriting history to fix old subjects means
force-pushing every branch of every repo, which breaks every clone and every
open PR for a cosmetic gain. What matters is the next commit.

---

## What never gets automated

- **File headers.** A script prepending SPDX comments to hundreds of files
  unattended will eventually corrupt a shebang, a JSON file, or a generated
  file that gets rewritten anyway. Do these by hand, per repo, when you are
  next editing the file.
- **Merging anything.** No automated flow of mine should merge to `main`.
- **Deleting anything.** Including branches that look stale.

---

## Handoff for tomorrow's conversation

Paste this into a new conversation, with the report attached:

> I have a conventions repo at `vishnuvarthan18/conventions` and a scaffold at
> `vishnuvarthan18/project-template`, both private. Attached is an audit of all
> my repos against those conventions. Help me work through it: start by telling
> me which repos are actually worth bringing into line and which to leave alone,
> then take the top one and do it properly. My folder `~/Desktop/github-setup`
> has the scripts from last time. `gh` is installed and authenticated. I am on
> macOS with bash 3.2, and I cannot have anything typed into my Terminal for me
> — give me commands to run, without inline `#` comments, because interactive
> zsh executes them as arguments.

That last sentence saves the next session the mistake this one made twice.

---

## If the audit fails tonight

- `gh not authenticated` → `gh auth login`
- Empty report → the `gh repo list` call returned nothing; check
  `gh api user --jq .login` matches `vishnuvarthan18`
- Anything else → the error prints and the script stops. Nothing is
  half-changed, because nothing is changed at all. Send me the output.
