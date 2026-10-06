# OVERNIGHT RUN

**Authorised 7 September 2026 by Vishnu. Valid for one unattended run.**

Nobody is awake. Every decision you would normally stop and ask about is
pre-answered below. Read this in full, then work the order in
[§9](#9-order-of-work).

**M6 (screenshots) is CUT from this run.** Do not start it. Screenshots
capture whatever is on a tester's screen, which makes the input-stripping and
`data-fb-block` logic the only privacy-sensitive code in the system — it gets
built with a human reviewing it. See [§9](#9-order-of-work).

If something is genuinely undecidable and is not answered here: **park it and
keep going.** Do not idle waiting for a human. [§8](#8-when-you-are-blocked)
says exactly how.

| Part | Covers |
| --- | --- |
| [1 Standing authorisations](#1-standing-authorisations) | What you may do without asking |
| [2 Hard stops](#2-hard-stops) | What you may never do, tonight especially |
| [3 Production standard](#3-production-standard) | What "done" means tonight — see [quality-gate.md](quality-gate.md) |
| [4 M4 decisions](#4-m4-decisions) | Issues and resolving, pre-answered |
| [5 M5 decisions](#5-m5-decisions) | Admin, pre-answered |
| [6 M6 decisions](#6-m6-decisions) | Screenshots — **deferred, not tonight** |
| [7 Storage](#7-storage) | Local disk behind an interface — **deferred** |
| [8 When you are blocked](#8-when-you-are-blocked) | Park and continue |
| [9 Order of work](#9-order-of-work) | And when to commit |

## 1 Standing authorisations

**Commits.** Vishnu has explicitly authorised committing, for this run only.
One commit per milestone, after its acceptance passes and all five make
targets are green. Follow
[git-conventions](https://github.com/aracreate-group/aracreate-conventions/blob/main/git/git-conventions.md):
`<type>: <lowercase imperative>`, no articles, no full stop, body explaining
why, **no `Co-Authored-By`**, no names or emails. Stage by path, never a
blanket `git add`.

**No pushing.** There is no remote. Do not add one.

**No dependency is approved tonight.** M6 is cut, so its `modern-screenshot`
pre-approval is withdrawn with it. Add nothing. Where a dependency would have
helped, implement without it and log the trade-off in `docs/blocked.md`.

**Schema changes** are authorised where this document names them. Migration
per change, never a hand edit.

## 2 Hard stops

Tonight is unattended, so these matter more than usual.

1. **No deployment. No hosting. No cloud.** No Vercel, Cloudflare, S3, Neon,
   Docker, tunnels, or any SDK for them. Everything local. If a task seems to
   need hosting, it is M2b and M2b is parked.
2. **Do not invent B. Halle's 49 page URLs.** `pages.json` keeps its
   placeholders. Making up plausible URLs would poison the fixture and the
   assignment generator with data that looks real and is not.
3. **Do not change a single tester-facing string.** Not the wording, not the
   punctuation, not the order in the file.
4. **Do not build anything on the out-of-scope list** in
   [agent-rules §3](agent-rules.md#3-out-of-scope). Not even a small version
   of it. Not even behind a flag.
5. **No mutation path for `reports`**, in code, in a migration, or in a test
   helper.
6. **Do not edit `build-plan.md` or `agent-rules.md`.** Those are the tech
   lead's. Propose changes in `docs/blocked.md` instead.
7. **Do not start M6.** Not the capture code, not the upload route, not the
   `modern-screenshot` import, not the storage interface, not a stub or a
   placeholder. Privacy-sensitive code does not get written unattended. The
   pre-approval of `modern-screenshot` is therefore **withdrawn for tonight** —
   there is no approved dependency in this run.
8. **Do not weaken a test to make it pass.** If a test is wrong, say so in
   `docs/blocked.md` and leave it failing. A green suite that got there by
   lowering the bar is worse than a red one.

## 3 Production standard

**Read [quality-gate.md](quality-gate.md) — it is the standard, and it
outranks this section.** A checklist you grade yourself against is not a
standard; the gate is mechanical. A milestone is done when deliberate attempts
to break it failed, not when the tests are green.

In particular you must, per milestone: introduce each violation in
[gate §1](quality-gate.md#1-mutation-proof) and prove a test screams, hunt
every item in [gate §2](quality-gate.md#2-attack-list), write the
[concurrency tests](quality-gate.md#3-concurrency) — the identical-report race
will fail with a naive read-then-insert, find that before it finds you — run
the [grep checks](quality-gate.md#4-loops-and-runaway-code), and do the
[adversarial review pass](quality-gate.md#5-adversarial-review-pass) with its
findings written down.

**If the gate does not pass, do not commit that milestone.** Log it and move
on. An honest red beats an arranged green.

On top of the gate, "done" also means:

- `make lint`, `make build`, `make test`, `make test-widget`, `make size` all
  green.
- Every acceptance item has a test that would fail if the behaviour broke.
- No `TODO`, `FIXME`, `XXX` or commented-out code left behind.
- No `console.log` in shipped code. Server-side errors logged with enough
  context to debug, never leaked to the client.
- Every external input validated with zod at the boundary — route handlers,
  server actions, form submissions, and anything read from a file.
- Security headers on the app: `X-Content-Type-Options: nosniff`,
  `Referrer-Policy: strict-origin-when-cross-origin`, a restrictive
  `Content-Security-Policy` for the dashboard, and `X-Frame-Options: DENY`.
  The two public API routes keep their open CORS and are exempt from the CSP.
- Every mutation goes through the tenant helper, and every state change writes
  an `issue_events` row where one applies.
- `.env.example` updated for any new key, `make setup` still tops it up.
- `README.md`, `docs/web/readme.md` and `docs/widget/readme.md` reflect what
  now exists. File headers on every new file.
- Accessibility basics on every new screen: real labels on inputs, visible
  focus, 4.5:1 contrast, keyboard operable, 44px targets. No WCAG claim
  anywhere.

## 4 M4 decisions

### Status transitions — allowed, and nothing else

```
new         -> triaged, wont_fix, duplicate
triaged     -> in_progress, wont_fix, duplicate
in_progress -> fixed, triaged, wont_fix
fixed       -> verified, in_progress
verified    -> closed
wont_fix    -> triaged            (reopen)
duplicate   -> triaged            (reopen)
closed      -> triaged            (reopen)
```

Enforce this server-side as a table, not with scattered `if`s. An illegal
transition is a 400 and writes nothing. Test every illegal edge, not only the
legal ones.

### Who may do what

| | staff | developer | client |
| --- | --- | --- | --- |
| Read grid, issues, reports, CSV | yes | yes | yes |
| Change status | any issue | only issues assigned to them | no |
| Assign | yes | no | no |
| Set priority | yes | no | no |
| Set category | yes | yes | no |
| Client-visible comment | yes | yes | yes |
| Internal note | yes | yes | **no — cannot write or read** |
| Mark duplicate | yes | no | no |

A client must not be able to infer an internal note exists. No count, no
placeholder, no gap in numbering.

### Comments

- `client_visible` defaults **false**. An internal note is the default because
  the safer accident is a note the client cannot see.
- A client-role author can only ever create `client_visible = true`. Enforce
  server-side; do not trust the form.
- Comments are append-only for now. No edit, no delete. If someone wants that
  later it is a decision, not a bug.

### Event log

Write an `issue_events` row for: `status_changed`, `assigned`,
`priority_changed`, `category_changed`, `commented`, `report_added`,
`marked_duplicate`, `reopened`. Store `from_value` and `to_value` as plain
text. Never delete an event.

### Category taxonomy — this exact list, in a data file

`src/web/data/categories.json`, with the `_meta` block:

```
content       Text is wrong, missing or out of date
layout        Something is visually broken or out of place
navigation    Could not find or reach something
link          A link or button goes nowhere or errors
image         An image is missing, stretched or wrong
form          A form does not work or does not submit
wording       The words are unclear or confusing
speed         Something was too slow
other         Does not fit the above
```

Nine values, staff/developer set them, never asked of the tester. Free text is
not a category — do not add one.

### Duplicate marking

Sets `duplicate_of` and status `duplicate`. It does **not** move reports
between issues and does not hide anything. The duplicate stays visible and
links to its target.

### Invites — dropped

There is no email in scope, so an invite that cannot be sent is dead code.
Users are created by `make user-create` and disabled by `make user-disable`.
**Do not build the `invites` table.** Log this in `docs/blocked.md` as a
deviation from `build-plan.md §3` so the tech lead can correct the doc.

### UI

No component library. Hand-written CSS, plain server-rendered forms and
server actions. If a screen feels like it needs a library, it is too clever —
simplify the screen.

## 5 M5 decisions

**Pages** — list, add, edit. Bulk import from a pasted list, one URL or path
per line: normalise with the same function the report route uses, ignore
blanks and duplicates, skip paths already present, report how many were added
and how many skipped. `page_type` is a select; bulk-imported pages get
`other`.

**Testers** — create with a generated token, 24 URL-safe characters from
`crypto.randomBytes`. Label auto-numbered `Tester 01`, `Tester 02`. Show the
invitation link for copying: `<site_url>/?t=<token>`. Never a password.

**Assignment generator** — three testers per page, spread as evenly as the
counts allow; Home, Contact and 404 in every tester's set; product-type pages
spread so no tester gets a run of near-identical ones. Accept an optional
integer seed so a run is reproducible and therefore testable. Show a preview
before writing, and make it idempotent — running it twice must not duplicate
assignments.

**String editor** — every tester-facing string editable in one form. Saving
validates against the zod config schema, writes a `config_revisions` row, then
updates `projects.config`. Rollback restores a chosen revision as a **new**
revision — never delete history. A save that fails validation changes nothing.

**Team** — list users with role and disabled state, disable and re-enable.
No creation in the UI; that is `make user-create`.

## 6 M6 decisions — DEFERRED, DO NOT BUILD TONIGHT

**These decisions stand for when M6 is built with review. Nothing in this
section is authorised tonight.** It is recorded here so tomorrow starts from
settled ground, not so you start now.

**The 15 KB budget is for `v1.js` only, and it stays.** `modern-screenshot`
would eat the remaining headroom, so:

- Keep the screenshot code out of `v1.js`. Load it with a dynamic import, as
  a separate chunk fetched from the same origin as `v1.js`, only when a
  capture is actually about to happen.
- `make size` keeps its 15,360-byte gate on `v1.js`.
- Add a second gate for the screenshot chunk at 30,720 bytes gzipped.
- If the capture chunk fails to load: skip the screenshot silently and submit
  the report without one. Never block or delay a report on it.

**Capture** — viewport only, `scale: 1`, WebP quality 0.8. **Capture twice on
Safari and keep the second** — the first is documented as blank across
libraries. Set `crossorigin="anonymous"` on images before capturing.

**Privacy, before the bytes leave the page** — strip the values of every
`input` and `textarea`, and honour a `data-fb-block` attribute on any element
by blanking it. Test both.

**Consent** — show the tester the captured image with a "don't include it"
option before sending, using strings from config. If they decline, no upload
happens and the report is submitted with its `screenshot_key` pointing at
nothing. That is a normal state, not an error.

**Upload** — `POST /api/v1/uploads` returns a short-lived upload URL for the
key already on the report. `screenshot_key` was set at insert and **is never
updated** — the trigger will refuse, correctly.

**Retention** — a 90-day job, wired to `make retention`, that deletes stored
files only. It never touches a `reports` row, not even to null the key.

## 7 Storage — DEFERRED WITH M6

**Not tonight.** The storage layer exists only to serve screenshots, so it
waits with them. Recorded for tomorrow.

Define `src/web/lib/storage/` with an interface — `put`, `get`, `delete`,
`signed_upload_url` — and one implementation backed by local disk.

- Files live outside the tracked tree, under a path from an env var, default
  `.storage/` at the repo root, and `.storage/` goes in `.gitignore`.
- Keys are validated before use: they must match the
  `reports/<uuid>/<uuid>.webp` shape. Reject anything containing `..` or an
  absolute path. Path traversal is the whole risk surface of a disk-backed
  store.
- Cloud storage later is a second implementation of the same interface and
  nothing else changes. Do not import the local one directly anywhere outside
  `lib/storage`.

## 8 When you are blocked

Something not answered here, genuinely two-sided, and nobody to ask.

1. Write it to `docs/blocked.md`: what the choice was, the options, what you
   picked, and why.
2. Pick the **safest reversible** option. Reversible beats clever at 3am.
   Prefer not building something over building the wrong thing — an absent
   feature is a morning conversation, a wrong one is a migration.
3. Keep going. Never stop the whole run on one question.
4. If a whole milestone cannot be completed, log why, leave it in a clean
   state, commit what does work, and start the next milestone.

Also keep `docs/overnight-log.md`: one short line per significant thing done,
in order, with timestamps. It is what the tech lead reads first in the
morning.

## 9 Order of work

M3 is signed off with three fixes owed. Do those first, then M4, then M5.
**Stop there.** Commit after each.

### 0. M3 fixes — do these before anything else

Three items from the M3 auth instruction did not land. All are fully
specified; none need a decision. Fix them, then commit as `fix:` — a separate
commit from M3's own, which stands as it is.

**a. Revocation.** `users.disabled_at timestamptz` in a migration. Load the
user row on every authenticated request and treat a missing OR disabled user
as no session — a signed cookie alone cannot be revoked, and this is what
makes removing someone's access actually work. Add `make user-disable
EMAIL=...` and a re-enable. **Disable, never delete** — deleting a user with
comments and `issue_events` behind them destroys the audit trail.
Test: a live session stops working on the next request after the user is
disabled.

**b. Login lockout.** Lock an email after 5 failed attempts in 15 minutes.
Count from data you already store — no new infrastructure. The error text must
be identical whether or not the email exists, and identical when locked out,
so it leaks nothing. Test all three cases return the same message.

**c. CSV formula injection.** A field whose first character is `=`, `+`, `-`,
`@`, tab or CR must be neutralised before it reaches the file, in both
exports. Tester notes are untrusted text; without this, a note reading `=1+1`
is a live formula when the client opens the export. Test each of the six
characters, and test that a legitimate note starting with a hyphen is still
readable.

Then run the full [quality gate](quality-gate.md) over auth and CSV — the §1
mutations for CSV quoting, formula injection and session forgery apply
directly.

### Then

1. **M4** — issues and resolving, per [§4](#4-m4-decisions). Gate. Commit.
2. **M5** — admin, per [§5](#5-m5-decisions). Gate. Commit.
3. **M6 and the storage layer are cut from this run.** Do not begin them.
4. If time remains: **no new features.** Re-run the full
   [quality gate](quality-gate.md) against the earlier milestones — M3 and M4
   were gated when written, but the attack list is worth a second pass once
   M5 exists and the surface is bigger. Raise test coverage on the paths that
   matter — the permission matrix, the transition table, the CSV writer, the
   grouping rule — and tidy the two `docs/` logs. Do not start M2b.

Leave the working tree clean and every make target green.
