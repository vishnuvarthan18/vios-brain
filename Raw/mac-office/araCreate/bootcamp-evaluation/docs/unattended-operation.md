# Unattended operation — how the agent runs without Vishnu

This replaces the "report at the end of each feature, then stop" rule in
`docs/v3-agent-brief.md` Part 1 rule 18, **for queued work only**. Every other
hard rule in that file still applies without exception — above all: **the agent
never deploys, never ssh's to the server, never touches the production
database.**

That is what makes this safe. The worst possible outcome of a bad overnight run
is a messy local branch that gets deleted in the morning. Nothing the agent can
do reaches the 209 students.

---

## 1. The shape

The agent works a **written queue**, in order, from `docs/work-queue.md`.

- Take the next unblocked item.
- Build it. Test it. Commit it. Push it.
- Append a report to `docs/agent-log.md`.
- Tick the item in the queue, commit that too.
- Take the next item. Do not wait for permission.

It keeps going until the queue is empty, or a stop condition fires.

---

## 2. When it must STOP completely

Only these. Everything else is handled by skipping, below.

1. **It would need to touch production** — any ssh, any production database
   connection, any deploy. Stop and write why.
2. **A data-loss risk it cannot avoid** — a migration with no working `down`, or
   anything that would destroy work in the repo.
3. **A verification gate fails in a way that suggests the data is wrong** — row
   counts not matching, team totals moving. Stop; do not "fix" it overnight.
4. **The same item fails three times** and the failures are not independent.
5. **The queue is empty.**

On a stop: write the reason at the top of `docs/agent-log.md`, push, and halt.

---

## 3. When it must SKIP and continue — the important one

Most overnight failures are not emergencies. They are questions.

**If an item needs a decision Vishnu has not already given:**

- Do **not** guess.
- Do **not** stop the whole run.
- Write the question to `docs/questions-for-vishnu.md` — what is blocked, the
  options, and which one you would choose and why.
- Mark the queue item `BLOCKED`.
- **Move to the next item that does not depend on it.**

Same for an item that fails twice for a reason you cannot resolve: log it, mark
it `FAILED`, move on. A night spent on one stuck item is a wasted night; a night
that clears eight of ten items is a good one.

**Never** loop on the same failing command more than three times. Never widen
scope to "fix" something in order to unblock yourself — log it and move on.

---

## 4. Assumptions

Every assumption made without a decision on record goes in the log, under
`ASSUMED:`, with what would change if it is wrong.

An assumption that would be **expensive to undo** is not an assumption — it is a
question. Write it to `questions-for-vishnu.md` and skip the item.

---

## 5. Git discipline, non-negotiable overnight

- **Commit after every item.** Not every night, every item.
- **Push after every commit.** Work that exists only on the laptop does not
  exist. Twelve commits once sat unpushed for a full night.
- One branch per track. Never `main`.
- No `git stash`. No `reset --hard`.
- A broken item is committed on its own, clearly labelled `WIP` in the message,
  so the morning review can see exactly where it stopped.

---

## 6. What Vishnu reads in the morning

Three files, in this order:

| File | What it answers |
| --- | --- |
| `docs/agent-log.md` | What happened, newest at the top. Any STOP reason is the first line |
| `docs/questions-for-vishnu.md` | What is blocked and needs a decision |
| `docs/work-queue.md` | What is done, what is blocked, what is left |

Plus `git log --oneline` on the branch.

### Log entry format — one per item

```
## [item id] [DONE | BLOCKED | FAILED]  <timestamp>
WHAT:     one line
TESTS:    what was run, what passed, at what data volume
OLD PATH: what the old code still does. Never "n/a"
ASSUMED:  every assumption, and what breaks if it is wrong
FOUND:    noticed, not fixed
NEXT:     the item being started
```

The `OLD PATH` line is not optional and is not skippable overnight. It is the
line that found the two worst bugs in the v2 build.

---

## 7. The cost of running unattended

Stated plainly so it is a choice, not a surprise.

- **A wrong assumption runs for hours instead of minutes.** Mitigated by
  question-and-skip, not eliminated.
- **You lose the conversation.** The agent's first report on this repo was
  excellent *because* it stopped and asked. Overnight it cannot.
- **Reviewing a night's work takes real time in the morning.** Budget twenty
  minutes before the day starts, not two.

**Therefore:** the queue must only ever contain work whose decisions are already
made. Anything still being argued about stays out of the queue. That is the
single rule that makes unattended running work.

---

## 8. Launching it

On the Mac, before leaving:

```sh
# stop the Mac sleeping while the agent works (lid must stay open)
caffeinate -i -w $$ &

cd ~/araCreate/bootcamp-dashboard
git status                 # must be clean before starting
git branch --show-current  # must NOT be main
```

Then start the agent with auto-accept for edits on, and this prompt:

```
Work docs/work-queue.md from the top, following docs/unattended-operation.md.
Do not wait for me. Report into docs/agent-log.md after every item.
Questions go to docs/questions-for-vishnu.md — never guess, never stop the run
for a question. Commit and push after every item.
```

### Before you walk away, check

- [ ] `git status` clean, branch is not `main`
- [ ] The queue contains only work whose decisions are already made
- [ ] A fresh dump is on the Mac and loaded locally
- [ ] `caffeinate` running, lid open, power connected
- [ ] `docs/agent-log.md` and `docs/questions-for-vishnu.md` exist, even if empty
