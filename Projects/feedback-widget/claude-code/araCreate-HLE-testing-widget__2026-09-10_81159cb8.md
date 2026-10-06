**Vishnu** (2026-09-10T01:29): Read docs/agent-task-capture-succeeds-but-ui-shows-no-picture.md and follow it. Do not commit or push without asking me first.

**Claude** (2026-09-10T01:29): I'll start by reading the task document.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): Let me find the file.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): The docs folder is empty at the top level. Let me check inside the project directory.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): The task points at the caller of `capture_screenshot()`. Let me explore the widget source and check my memory notes for context.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): Now let me trace the capture call in `app.ts` — that's where the doc says the hand-off bug lives.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): Interesting — `capture_screenshot` isn't referenced in `app.ts` at all. Let me find where it actually is called.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): The caller is `loader.ts`, not `app.ts`. Line 208 of the test file mentions a never-resolving promise — suggesting the loader has its own timeout. Let me read it.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): `load_and_capture` has no timeout of its own. So the caller's timeout must be in `app.ts`. Let me look at the state machine and how the capture result flows to the review screen.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): Found it. Line 476-485 is a `Promise.race` with `CAPTURE_TIMEOUT_MS` in the caller. Let me read that section in full.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): This confirms the diagnosis. Let me check `CAPTURE_TIMEOUT_MS` and the internal capture budget.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): Root cause confirmed exactly as hypothesised in the doc's first bullet: the caller gives up at **3s** while the capture library is allowed **12s**. The trace shows the capture resolving at 11.576s — 8.5s after the UI had already committed to "(no picture)".

The stale-result guard makes it worse: the late blob is discarded. Let me read the surrounding code before deciding on the fix.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): Now I understand the full picture. The doc forbids touching `capture.ts`'s budget, so the fix belongs in the caller. Let me check the existing tests to understand the contract they encode — particularly the 3s-timeout test at line 208.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): The existing tests only assert the never-resolving case degrades gracefully — none assert that a *slow but successful* capture reaches the screen. That's the missing regression test.

The right fix has two parts: raise the cap above the library's own budget, and stop discarding a late-arriving valid image. Let me confirm how `capture_promise` and the send path interact.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): Note `capture_promise` is never read at send time — the review screen's blob is the only path to upload. Let me check how the blob gets from the review screen into the send call.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): I have the complete picture. Let me check the specs the code cites, since my memory notes say spec precedence matters here.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:30): Confirmed: this repo's standard is to verify by actually exercising behavior, not by code reading. Let me confirm the diagnosis empirically before changing anything — reproducing with a slow-but-successful capture as the doc's step 3 asks.

**Claude** (2026-09-10T01:30): _[tool: Bash]_

**Claude** (2026-09-10T01:31): Now let me write a reproduction test that models the real-world case: a capture that succeeds but takes longer than the caller's 3s cap.

**Claude** (2026-09-10T01:31): _[tool: Bash]_

**Claude** (2026-09-10T01:31): Now let me write the reproduction test. It serves a chunk whose capture succeeds but takes 5s — longer than the 3s cap, shorter than the 12s budget.

**Claude** (2026-09-10T01:31): _[tool: Bash]_

**Claude** (2026-09-10T01:31): No existing helper. Let me write the reproduction test using a real decodable WebP produced in-browser via canvas.

**Claude** (2026-09-10T01:31): _[tool: Bash]_

**Claude** (2026-09-10T01:31): Reproduced exactly. The image never appears even though the capture succeeded and returned a valid WebP — it waited the full 15s and the picture never arrived.

Now the fix. Two independent defects, and both need addressing:

1. **The cap (3s) is shorter than the library's budget (12s)** — so a capture inside its own budget gets abandoned.
2. **A late result is discarded even when valid** — the race resolves `null`, renders "(no picture)", and the real blob that lands later is thrown away.

Fixing only the cap would still leave the UI blank until the capture finishes. Let me fix both: raise the cap past the budget, and upgrade the screen in place when a late image arrives.

**Claude** (2026-09-10T01:31): _[tool: Bash]_

**Claude** (2026-09-10T01:31): _[tool: Bash]_

**Claude** (2026-09-10T01:31): Now the core fix in `render_review` — paint early, then adopt a late-arriving image instead of discarding it.

**Claude** (2026-09-10T01:32): _[tool: Bash]_

**Claude** (2026-09-10T01:32): A late re-render rebuilds the comment box, which would wipe whatever the tester typed while waiting. Let me check how `render_review_screen` builds the textarea and preserve that text.

**Claude** (2026-09-10T01:32): _[tool: Bash]_

**Claude** (2026-09-10T01:32): A re-render would discard the tester's typed comment and reset focus/caret. Since the picture can now arrive after the tester has started typing, I need to carry that text across.

**Claude** (2026-09-10T01:32): _[tool: Bash]_

**Claude** (2026-09-10T01:32): Let me make `render_review_screen` accept and restore the in-progress comment, so adopting a late picture doesn't destroy the tester's typing.

**Claude** (2026-09-10T01:32): _[tool: Bash]_

**Claude** (2026-09-10T01:32): Now wire `paint` to read the live textarea before rebuilding, so the carried text is actually captured.

**Claude** (2026-09-10T01:32): _[tool: Bash]_

**Claude** (2026-09-10T01:32): One more leak to close: the first paint creates an object URL that the rebuild would orphan. Let me check the revoke paths.

**Claude** (2026-09-10T01:32): _[tool: Bash]_

**Claude** (2026-09-10T01:32): All four revokes are on user-initiated exits (send/cancel/close/focus-trap). A re-render replaces the DOM without hitting any of them — so the prior paint's object URL would leak. In practice the first paint has `blob === null` (no URL created), but the guard should not depend on that ordering. Let me make the revoke explicit.

**Claude** (2026-09-10T01:33): _[tool: Bash]_

**Claude** (2026-09-10T01:33): Now let me check the other `render_review_screen` call site and typecheck.

**Claude** (2026-09-10T01:33): _[tool: Bash]_

**Claude** (2026-09-10T01:33): Single call site — clean. Let me check the Makefile for the build/typecheck targets and rebuild the bundle.

**Claude** (2026-09-10T01:33): _[tool: Bash]_

**Claude** (2026-09-10T01:33): The widget bundle is built, so my `.ts` edits need compiling before the browser tests exercise them. Let me check how the test server serves the widget.

**Claude** (2026-09-10T01:33): _[tool: Bash]_

**Claude** (2026-09-10T01:33): Tests run against `src/widget/dist/`, so I must rebuild before testing.

**Claude** (2026-09-10T01:33): _[tool: Bash]_

**Claude** (2026-09-10T01:33): Build succeeded. Now let me verify the fix against the reproduction test.

**Claude** (2026-09-10T01:33): _[tool: Bash]_

**Claude** (2026-09-10T01:33): Fixed — passes in 5.7s, adopting the picture right when the capture resolves. Let me verify the typecheck and confirm the first paint still happens fast (the tester isn't blocked for 5s).

**Claude** (2026-09-10T01:33): _[tool: Bash]_

**Claude** (2026-09-10T01:33): Lint and typecheck clean. Now let me write the real regression tests for the tracked tree, replacing my scratch repro. These need to assert the behaviours the old code got wrong.

**Claude** (2026-09-10T01:33): _[tool: Bash]_

**Claude** (2026-09-10T01:33): Now let me add the regression tests to the tracked suite, matching the file's existing style and comment density.

**Claude** (2026-09-10T01:33): _[tool: Bash]_

**Claude** (2026-09-10T01:34): _[tool: Bash]_

**Claude** (2026-09-10T01:34): Let me verify `mock_api` supports the `uploadUrl` option as I used it, then run the new tests.

**Claude** (2026-09-10T01:34): _[tool: Bash]_

**Claude** (2026-09-10T01:34): _[tool: Bash]_

**Claude** (2026-09-10T01:34): There's a built-in `onUpload` hook — cleaner than my duplicate `page.route`, which would also have conflicted with the fixture's own route. Let me use the intended API.

**Claude** (2026-09-10T01:34): _[tool: Bash]_

**Claude** (2026-09-10T01:34): Nine pass, but the upload assertion fails — worth checking whether this is a test-timing artifact or a real second bug. The upload happens asynchronously after "Thank you" renders, so the assertion may simply be racing it.

**Claude** (2026-09-10T01:34): _[tool: Bash]_

**Claude** (2026-09-10T01:34): `finish_send` calls `go('sent')` immediately without awaiting `send_report`, so "Thank you" renders before the upload completes. My assertion was racing it — a test defect, not a product one. Let me confirm by polling for the upload.

**Claude** (2026-09-10T01:34): _[tool: Bash]_

**Claude** (2026-09-10T01:35): All 10 pass, including the pre-existing never-resolving-capture test (3.9s), which confirms the cap still works as a first-paint deadline.

Now the mutation proof this repo's quality gate requires — I'll revert the fix and confirm the new tests actually scream.

**Claude** (2026-09-10T01:35): _[tool: Bash]_

**Claude** (2026-09-10T01:35): Mutation proof passes: exactly the 3 new behavioural tests fail under the old logic, and the first-paint test still passes (correctly — the cap was never the part that broke). Restoring the fix:

**Claude** (2026-09-10T01:35): _[tool: Bash]_

**Claude** (2026-09-10T01:35): Now let me run the full widget suite to confirm nothing else regressed, and check the size budget since I added code to the bundle.

**Claude** (2026-09-10T01:35): _[tool: Bash]_

**Claude** (2026-09-10T01:35): All 48 pass. Let me check the size budget and lint.

**Claude** (2026-09-10T01:36): _[tool: Bash]_

**Claude** (2026-09-10T01:36): Size within budget, lint clean. Now the doc's step 3 asks me to reproduce against a real image-heavy page and read `window.__halleCaptureLog` — verifying the actual end-to-end path, not just my synthetic delay.

**Claude** (2026-09-10T01:36): _[tool: Bash]_

**Claude** (2026-09-10T01:36): Let me run the real capture pipeline against the image-heavy fixture and read the actual trace, as the doc prescribes.

**Claude** (2026-09-10T01:36): _[tool: Bash]_

**Claude** (2026-09-10T01:36): This is important — the trace shows the real capture **succeeded** (9,644 bytes in 98ms, well under the cap), yet the UI still shows "(no picture)". A second, distinct bug that my synthetic-delay tests could not reach. Let me investigate.

The capture succeeded fast, so this isn't a timing issue. Let me check what target the click selected.

**Claude** (2026-09-10T01:36): _[tool: Bash]_

**Claude** (2026-09-10T01:36): Both host pages configure the widget identically. The capture succeeded but the blob didn't reach the screen — let me check whether my click actually landed on a valid target, since `render_review` keys off `capture_promise` identity.

**Claude** (2026-09-10T01:36): _[tool: Bash]_

**Claude** (2026-09-10T01:36): That existing test uses **Screenshot** mode and already asserts no "(no picture)" — and it passes. My scratch test used **pointer** mode and clicked at `(40,40)` on `body`. Let me test whether the mode is the differentiator.

**Claude** (2026-09-10T01:36): _[tool: Bash]_

**Claude** (2026-09-10T01:36): Screenshot mode also shows "(no picture)" here, yet the near-identical existing test passes. The difference must be the viewport — the existing test sets 900x700. My scratch test used the default 1280x720. Let me isolate that.

**Claude** (2026-09-10T01:36): _[tool: Bash]_

**Claude** (2026-09-10T01:37): Still "(no picture)" even with identical setup to the passing test. The only remaining difference is my `waitForTimeout(6000)`. That's suspicious — it suggests the image appears and then *disappears*. Let me check the timeline.

**Claude** (2026-09-10T01:37): _[tool: Bash]_

**Claude** (2026-09-10T01:37): There's the second bug, and it's one I introduced. The image shows correctly, then at exactly **3000ms** — `CAPTURE_FIRST_PAINT_MS` — my unconditional `setTimeout(() => paint(null), ...)` fires and *replaces* the good picture with "(no picture)".

My `painted_blob === blob` guard doesn't catch it: `painted_blob` is the real blob, incoming is `null`, so they differ and it repaints. The fast-capture path was broken by my own fix. Let me correct it.

**Claude** (2026-09-10T01:37): _[tool: Bash]_

**Claude** (2026-09-10T01:37): The fix: the deadline must only ever paint the *empty* state, and never downgrade a screen that already has a picture. Let me make that explicit rather than relying on blob comparison.

**Claude** (2026-09-10T01:37): _[tool: Bash]_

**Claude** (2026-09-10T01:38): The picture now stays on screen past the 3s mark. That regression was invisible to the existing suite because every test asserts state only *once*, immediately — never after the deadline elapses. Let me add a test that closes that gap.

**Claude** (2026-09-10T01:38): _[tool: Bash]_

**Claude** (2026-09-10T01:38): Let me mutation-proof the new guard specifically, to confirm the test actually catches that regression.

**Claude** (2026-09-10T01:38): _[tool: Bash]_

**Claude** (2026-09-10T01:38): Mutation proof confirmed — removing the guard fails exactly that one test, and the fix is restored. Let me run the full suite and lint for a final check.

**Claude** (2026-09-10T01:38): _[tool: Bash]_

**Claude** (2026-09-10T01:39): All 49 tests pass, lint clean, size within budget. Let me verify the working tree contains only intended changes and no scratch files.

**Claude** (2026-09-10T01:39): _[tool: Bash]_

**Claude** (2026-09-10T01:39): My changes are confined to `app.ts` and `capture.spec.ts` (the other modified files were already dirty before I started — untouched by me). Scratch files cleaned up. Note the `dist/` bundle is gitignored but rebuilt.

Let me confirm the built bundle is current and review my final diff.

**Claude** (2026-09-10T01:39): _[tool: Bash]_

**Claude** (2026-09-10T01:39): The diff is clean and contained. Let me record the non-obvious finding in memory, since the "assert once" test-blindness is a reusable lesson for this repo.

**Claude** (2026-09-10T01:39): _[tool: Bash]_

**Claude** (2026-09-10T01:39): Done. The hand-off bug is fixed, verified against the real capture pipeline, and mutation-proofed. I have not committed anything, as you asked.

## What was wrong

The doc's first hypothesis was correct: **the caller's timeout was shorter than the library's budget.** [app.ts](halle-feedback-widget/src/widget/src/app.ts) capped its wait at 3s while `capture_screenshot` is allowed 12s, and `Promise.race` rendered whichever won. Any capture slower than 3s was discarded outright — the 11.6s/29KB capture in your trace was a real image the UI threw in the bin. `capture.ts` was never touched.

I reproduced it first (a capture succeeding at 5s → image never appeared), then fixed it by making the 3s cap a **first-paint deadline** rather than a give-up: the comment box still opens fast, the capture is still awaited, and a late picture upgrades the screen in place.

## The second bug — introduced by that fix

Following your step 3 against the real pipeline on the image-heavy fixture, the trace said `captured, 9644 bytes, 98ms` but the UI *still* showed "(no picture)". Sampling the DOM every 500ms instead of once revealed why:

```
t=2500ms img=1 nopic=0
t=3000ms img=0 nopic=1   ← deadline fires, tears the picture off
```

The deadline's timer fired unconditionally and downgraded an already-good screen. So the **fast** path — the common case — would have broken in production, showing testers a picture that vanished three seconds later. Fixed with an explicit never-downgrade guard (comparing blobs isn't enough: real-blob vs `null` legitimately differ).

Worth flagging: **all 48 existing tests were blind to this** because each asserts the screen once, immediately. That's the gap the new time-sampling test closes.

Handling the re-render also required carrying across the tester's typed comment, the Send button's enabled state, and caret position, plus revoking the superseded object URL — the existing revokes only run on tester-initiated exits, never on a re-render.

## Verification

- **49 tests pass** (5 new), lint + typecheck clean, bundle within budget (53.1% / 35.5%).
- **Mutation-proofed both fixes**: reverting the race fix fails exactly the 3 hand-off tests; removing the downgrade guard fails exactly the no-downgrade test.

## Secondary finding (your item 2, not blocking)

I left capture performance alone as instructed. One datum for when you pick it up: on the 300-image fixture the viewport-scoped capture pulled only 10 images and finished in **98ms** — so the 11.6s you saw live isn't inherent to image count. It's more likely per-asset network fetches for cross-origin images on the real page. That's worth measuring before optimising quality or image limits, since the fix now means testers see the comment box in 3s regardless and the picture simply appears when ready.

I also noted the timer-vs-single-shot-assertion lesson in memory. Want me to commit this?

**Vishnu** (2026-09-10T01:44): What was wrong
The doc's first hypothesis was correct: the caller's timeout was shorter than the library's budget. app.ts capped its wait at 3s while capture_screenshot is allowed 12s, and Promise.race rendered whichever won. Any capture slower than 3s was discarded outright — the 11.6s/29KB capture in your trace was a real image the UI threw in the bin. capture.ts was never touched.

I reproduced it first (a capture succeeding at 5s → image never appeared), then fixed it by making the 3s cap a first-paint deadline rather than a give-up: the comment box still opens fast, the capture is still awaited, and a late picture upgrades the screen in place.

The second bug — introduced by that fix
Following your step 3 against the real pipeline on the image-heavy fixture, the trace said captured, 9644 bytes, 98ms but the UI still showed "(no picture)". Sampling the DOM every 500ms instead of once revealed why:


t=2500ms img=1 nopic=0
t=3000ms img=0 nopic=1   ← deadline fires, tears the picture off
The deadline's timer fired unconditionally and downgraded an already-good screen. So the fast path — the common case — would have broken in production, showing testers a picture that vanished three seconds later. Fixed with an explicit never-downgrade guard (comparing blobs isn't enough: real-blob vs null legitimately differ).

Worth flagging: all 48 existing tests were blind to this because each asserts the screen once, immediately. That's the gap the new time-sampling test closes.

Handling the re-render also required carrying across the tester's typed comment, the Send button's enabled state, and caret position, plus revoking the superseded object URL — the existing revokes only run on tester-initiated exits, never on a re-render.

Verification
49 tests pass (5 new), lint + typecheck clean, bundle within budget (53.1% / 35.5%).
Mutation-proofed both fixes: reverting the race fix fails exactly the 3 hand-off tests; removing the downgrade guard fails exactly the no-downgrade test.
Secondary finding (your item 2, not blocking)
I left capture performance alone as instructed. One datum for when you pick it up: on the 300-image fixture the viewport-scoped capture pulled only 10 images and finished in 98ms — so the 11.6s you saw live isn't inherent to image count. It's more likely per-asset network fetches for cross-origin images on the real page. That's worth measuring before optimising quality or image limits, since the fix now means testers see the comment box in 3s regardless and the picture simply appears when ready.

I also noted the timer-vs-single-shot-assertion lesson in memory. Want me to commit this?

**Claude** (2026-09-10T01:44): You

**Vishnu** (2026-09-10T01:44): [Request interrupted by user]

**Vishnu** (2026-09-10T01:44): Yes, go ahead and commit. Don't push.Yes, go ahead and commit. Don't push.

**Claude** (2026-09-10T01:44): Committing now. First let me check the repo state and the draft-commit convention from my notes.

**Claude** (2026-09-10T01:44): _[tool: Bash]_

**Claude** (2026-09-10T01:44): Two of the modified files (`src/web/package.json`, `tests/db/tenant-import-guard.test.ts`) were already dirty before I started and aren't mine. Let me confirm that before staging so I commit only my fix.

**Claude** (2026-09-10T01:44): _[tool: Bash]_

**Claude** (2026-09-10T01:44): Confirmed — those two belong to unrelated password-script work, not my fix. I'll stage only my two files.

**Claude** (2026-09-10T01:44): _[tool: Bash]_

**Claude** (2026-09-10T01:44): The `git add` succeeded; only my status flag was wrong.

**Claude** (2026-09-10T01:44): _[tool: Bash]_

**Claude** (2026-09-10T01:44): Exactly my two files staged. Let me check the existing commit message style before writing mine.

**Claude** (2026-09-10T01:44): _[tool: Bash]_

**Claude** (2026-09-10T01:45): Clear convention: `type: summary`, then prose explaining the *why* and what only surfaced by testing, then verification counts. This is a `fix:`. Let me write it.

**Claude** (2026-09-10T01:45): _[tool: Bash]_

**Claude** (2026-09-10T01:45): Committed. Let me verify the result and that nothing was pushed.

**Claude** (2026-09-10T01:45): _[tool: Bash]_

**Claude** (2026-09-10T01:45): Committed as `253d97b` on `dev`, not pushed — it shows as the one unpushed commit ahead of the remote.

The commit contains only my two files:
- [app.ts](halle-feedback-widget/src/widget/src/app.ts) — the hand-off fix
- [capture.spec.ts](halle-feedback-widget/tests/widget/capture.spec.ts) — 5 regression tests

I deliberately left two files out of it, still dirty in your tree: `src/web/package.json` and `tests/db/tenant-import-guard.test.ts`. Those were already modified before I started and belong to the `user-password`/`check-demo-password` script work, not this fix — they're yours to commit separately whenever that work lands.

The commit message records both defects (including that the second was introduced by the fix for the first), the mutation-proof results, and the performance finding as an explicit non-goal with the 98ms/300-image datum for whoever picks it up.