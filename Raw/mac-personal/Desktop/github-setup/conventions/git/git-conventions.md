# Git Conventions

**What this is.** How a commit message is written, and what may appear in one. Nothing here depends on where the repo is hosted — it holds on GitHub, on a self-hosted remote, and on a bare repo with no tracker at all.

**Why it exists.** Commit format is the one convention that has to be identical in every repo, because release tooling reads it. `semantic-release` decides the version bump and writes `CHANGELOG.md` from commit messages ([repo §4.4](../repo/readme.md#44-versioning)), so a repo that phrases commits differently produces different release notes without ever failing loudly.

The history is also the only record that outlives the host. Issues, boards and review threads live on a platform that can be migrated, archived or lost; `git log` travels with the code. Keeping it readable is what makes the work legible in five years.

**How it fits.**

| Layer | Covers |
| --- | --- |
| [repo](../repo/readme.md) | What a repo *contains* — structure, headers, naming, versioning |
| **This document** | What a commit message looks like |
| [github-conventions](github-conventions.md) | How work is *tracked and reviewed* on GitHub |

## Contents

| Part | Covers |
| --- | --- |
| [1 Format](#1-format) | Type · subject · body |
| [2 Reference trailer](#2-reference-trailer) | Pointing at the tracked issue |
| [3 Wording](#3-wording) | Articles · short forms · what never appears |
| [4 What goes in one commit](#4-what-goes-in-one-commit) | Scope · staging · splitting |
| [5 Publishing](#5-publishing) | Committing and pushing are separate acts |
| [6 Quick reference](#6-quick-reference) | The rules in one table |

## 1 Format

Conventional Commits. The subject is what release tooling parses, so its shape is not negotiable:

```
<type>: <lowercase imperative summary>

<body — why, where the diff does not say it. Wrapped, normal sentences.>

#<issue number>
```

- **Types**: `feat` `fix` `refactor` `perf` `docs` `build` `test` `chore`.
  `feat` bumps the minor version, `fix` bumps the patch, the rest bump nothing.
- **Breaking changes**: a `!` after the type (`feat!:`) or a `BREAKING CHANGE:` paragraph in the body bumps the major version. Use the body form and explain what breaks and what to do instead — a bare `!` tells the reader nothing.
- **Subject**: imperative, lowercase after the prefix, no trailing full stop, no articles ([§3](#3-wording)). Keep it under ~72 characters so `git log --oneline` stays readable.
- **Body**: required whenever the reason is not obvious from the diff. It explains *why*, not *what* — the diff already says what. Wrap it; write full sentences.

A scope is optional and goes in brackets after the type — `refactor(api):`. Use it where the area is not obvious from the subject, and keep the set of scopes small enough to remember.

## 2 Reference trailer

Where the work is tracked, the issue reference goes at the foot of the body, on its own last line, as a bare `#<n>` and nothing else:

```
feat: derive session key from device id

Default key collided across instances on the same host, so two clients
could not be told apart in the audit log.

#136
```

No closing keyword, no URL, no PR number. A closing keyword in a commit message fires when the commit reaches the **default branch**, which is long after the work was done and outside the review that should have decided it. On GitHub, closing belongs in the pull request description, where merging triggers it — see [github-conventions §3](github-conventions.md#3-pull-requests).

The bare `#136` is a pointer for whoever reads the log, short enough that the message still reads outside the host. Omit the trailer where there is no tracked issue.

## 3 Wording

**Use the project's short forms** for its recurring nouns, and define them in that project's own docs rather than here. Long forms are for prose aimed at people outside the project; in a subject line they only pad it. This does not extend to code — a tool's `--help` output still spells things out for whoever runs it cold.

**Drop the articles** — `a`, `an`, `the` — from every commit subject. They carry no information in a summary line, and leaving them out keeps the phrasing general rather than tied to one instance:

```
add a retry wrapper around the upload call   ->  add retry wrapper around upload call
fix the timeout on cold start                ->  fix timeout on cold start
update the makefile for a fresh checkout     ->  update makefile for fresh checkout
```

This is a summary-line rule, not a prose rule. Commit bodies are written as normal sentences.

**Never name people or credentials.** No emails, usernames, real names, tokens or account IDs in a commit message. Keep it generic; the detail belongs in the file or in the secret store.

**No trailers beyond the issue reference.** No `Co-Authored-By`, no tool attribution, no generated-by lines. A commit message describes the change, not the process that produced it.

The line is *people, not references*. An issue reference is exactly what a commit should carry — `#136` at the foot of the body ([§2](#2-reference-trailer)) points at the work without naming anyone, which is the whole reason that trailer is allowed. `@handle` belongs in issue and PR descriptions, because that is how the host notifies someone; it stays out of commit messages, where it notifies no one and only dates the history.

## 4 What goes in one commit

**One concern per commit**, and every commit must stand on its own — build, test and read correctly without the commits after it.

**Stage deliberately.** Name the paths going into a commit rather than staging everything present. An unrelated file swept in by a blanket `git add` lands under a message that does not describe it, and nothing later will catch that.

**Split a file that spans two concerns** rather than letting the smaller change ride along. Stage the hunks belonging to this commit and leave the rest — `git add -p`, or a patch built from `git diff` and applied with `git apply --cached`. A commit carrying someone else's change is a commit whose message is wrong.

**A zero-context patch places hunks by line number.** `git diff -U0` gives the finest split, but `git apply --cached --unidiff-zero` trusts those numbers instead of matching surrounding context, so omitting an earlier hunk silently misplaces every later one — the file still applies and no longer parses. Either keep context (`-U3`, so git locates each hunk by matching) or build the intended file content and write it to the index, then re-diff after every commit so the offsets stay valid.

**Prove the split lost nothing.** Two checks: selecting *all* hunks must reproduce the working tree byte for byte, and the finished branch must diff identically against its base to a patch taken before the split started. Keep that patch until the last commit lands.

**`git mv` stages the rename immediately.** Both paths sit in the index from that moment, so the next `git add <paths>` and commit sweeps them in although they were never named. Check `git diff --cached --stat` before committing after a move, and `git reset -- <path>` anything belonging to a later commit.

**Refactoring needed to make a fix possible goes in its own commit**, so the fix is legible on its own and can be reverted without taking the refactor with it.

## 5 Publishing

Committing and pushing are separate acts, and **each needs its own explicit instruction**. Approval to make a change is never approval to commit it, and approval to commit is never approval to push.

None of the following is an instruction, however strongly it implies one is coming:

- approving a plan whose steps say the work will be committed or pushed;
- a conditional or permissive remark — "it can be committed", "this should go on branch X";
- being asked to draft or prepare a commit message;
- having committed something earlier in the same session.

Prepare the work, draft the message to a file, say what is ready, and wait.

Cutting a release is publishing too: it commits `VERSION` and `CHANGELOG.md`, creates a tag, and pushes both. **Releases run in CI only** — see [github-conventions §5](github-conventions.md#5-releases). Run outside CI, `semantic-release` falls back to dry-run while still printing `✔ Published release <version>` and the full notes, so a laptop run looks exactly like a real one and changes nothing. Never tag by hand to work around it.

## 6 Quick reference

| | Rule |
| --- | --- |
| Subject | `<type>: <lowercase imperative>` · no articles · no full stop · ~72 chars |
| Types | `feat` `fix` `refactor` `perf` `docs` `build` `test` `chore` |
| Bumps | `feat` minor · `fix` patch · `BREAKING CHANGE:` major · rest none |
| Body | why, not what · normal sentences · required when the diff is not self-explanatory |
| Trailer | bare `#<n>` on the last line · no closing keyword · no URL · omitted when untracked |
| Never in a message | emails · usernames · real names · tokens · `Co-Authored-By` · tool attribution |
| Scope | one concern · staged by path · stands on its own |
| Splitting | keep context or build the blob · all-hunks rebuild must match · diff against a pre-split patch |
| Publishing | explicit instruction for each act — see [§5](#5-publishing) |
| Releases | CI only · never tag by hand · a laptop `semantic-release` run is a silent no-op |
