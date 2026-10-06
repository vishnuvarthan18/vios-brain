# BLOCKED / DEVIATIONS

One entry per genuinely two-sided decision or a deviation from a planning
doc, per agent-rules.md §1.6 and overnight-run.md §8: what the choice was,
the options, what was picked, and why. Not a place to re-litigate a settled
instruction — only where two docs actually disagree or where the task
message overrides an earlier planning doc.

## M6a — storage backend: local disk, not S3

`docs/build-plan.md` §2/§9 and the original `.env.example` both describe
screenshot storage as S3-compatible object storage, presigned via the
provider's own SDK. The M6a task message given directly by the tech lead
supersedes this explicitly: "there is no S3 to presign against, sign it
yourself" and "one local-disk implementation" — per the standing convention
in this repo (the current task message is the more authoritative, current
spec when it and a planning doc disagree — see the M2 precedent), the task
message wins.

**What was picked:** `src/web/lib/storage/` defines the `Storage` interface
(`put`, `get`, `delete`, `signed_upload_url`) exactly as `docs/overnight-run.md`
§7 specifies, with one implementation (`local-disk.ts`) backed by the
filesystem, path from `STORAGE_DIR` (default `.storage/`, gitignored).
Upload urls are signed with an HMAC over the key and an expiry, using the
existing `SESSION_SECRET` — not a new secret — per the task message's exact
wording.

**Why this doesn't block anything:** the interface is the contract every
caller (the reports route, a future retention job) is written against, not
the backend. `.env.example`'s S3 block is replaced with a `STORAGE_DIR`
comment explaining the swap, so a future cloud implementation is a second
file behind the same interface and an env change, not a rewrite of anything
that calls `get_storage()`.

**Flagging for the tech lead:** `docs/build-plan.md` §2 and §9 should be
corrected to describe local-disk storage (or note that cloud storage is a
future swap) rather than S3, so the two docs stop disagreeing. Not edited
here — agent-rules.md §1.6, `build-plan.md` is the tech lead's.

## M6b — capture clone attachment point: document.documentElement, not a shadow root

The capture module (`src/widget/src/capture.ts`) needs to render an
off-screen, privacy-stripped clone of `document.body` somewhere the browser
will actually lay it out and paint it — a fully detached node (never
attached anywhere) reports zero for `getBoundingClientRect`/
`getComputedStyle`, so `display:none`/no-attachment at all is not an option.

**The options considered:** (a) attach the clone directly as a child of
`document.body` — rejected during development because it nests the clone
inside its own live original, transiently duplicating every id on the page
for the ~150-200ms a capture takes (caught by this repo's own acceptance
suite: a query for one id started resolving to two elements). (b) attach as
a sibling of `<body>`, directly under `document.documentElement` — avoids
the duplication, but the clone is still reachable by the host page's own
`document.getElementById`/`querySelector` during that window, and its
insertion could fire a host page's own `MutationObserver` (a Webflow
interactions engine, a lazy-load script, an analytics tracker watching for
DOM changes). (c) attach inside a shadow root (the widget's own, or a fresh
one on the wrapper) instead — tried, and rejected on hard evidence: decoding
an actual captured image this way showed the host page's own CSS had not
applied at all — no colours, no layout-affecting rules, a screenshot that
would be useless in a bug report, since a shadow root's styling isolation
cuts both ways.

**What was picked:** option (b) — attach as a sibling of `document.body`.
The window between attaching and removing the clone is kept as short as
possible (build, capture, remove, no unrelated `await`s in between), and
this is the only option of the three that keeps the host page's own CSS
cascade applying to the clone, without permanently corrupting the live DOM
the way option (a) did transiently.

**What is NOT verified yet, and must be checked at M2b against the real
Webflow site, not just the local test harness:** whether attaching this
clone (even briefly, even off-screen) triggers anything observable on a real
production page — a Webflow interaction/animation firing, a lazy-load
script reacting to a DOM insertion it wasn't expecting, an analytics
snippet recording a spurious event. Nothing in the local acceptance suite
can exercise this, since none of the local fixture pages run that kind of
production tooling. This should be one of the first things checked once M2b
puts the widget on the actual client site.

## M6b — a `<select>`'s chosen option is not stripped from the capture

`strip_clone()` blanks `input`, `textarea`, and `[contenteditable]` content,
and blanks `[data-fb-block]` elements outright, but deliberately leaves a
`<select>`'s currently-chosen option showing in the captured image.

**Why this is a genuine two-sided question, not an oversight:** a chosen
option can itself be sensitive (a diagnosis in a symptom picker, an income
bracket, anything from a closed list rather than free text) — the same
privacy concern `data-fb-block` and input-stripping exist for. But blanking
it outright would make some categories of bug report unreadable — "the
wrong option came up pre-selected" is exactly the kind of thing this widget
exists to catch, and a screenshot with every dropdown showing its default
or blank state can't show that. Neither "always show it" nor "always blank
it" is clearly correct without knowing what kind of `<select>` elements
B. Halle's actual test pages contain.

**What was picked:** left showing, unchanged, for now — the safer default
until a real decision is made, since a form on a build-plan.md-listed page
(`data/pages.json`'s placeholders) is not yet known to contain anything
sensitive in a `<select>`. **Flagging for the tech lead:** decide whether a
`<select>`'s chosen option should be stripped, and if so, whether uniformly
or via a `data-fb-block`-style opt-in/opt-out attribute.
