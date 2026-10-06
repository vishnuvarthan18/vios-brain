# Phase 3, Cohort B — results, 2026-09-09

Branch: `deploy/sensitive-species-fix-verify`. Nothing deployed, no staging created, main untouched.

## bhl — on hold, not actioned

24 Aug doc claimed BHL_API_KEY was issued; current repo (sources/bhl.json, .dev.vars) says never obtained. Conflict unresolved — Vishnu needs to check his email/BHL account directly before anything happens here. No agent action taken.

## wdpa — confirmed dead end, left open per Vishnu's call

Re-verified live (WebFetch + WebSearch, this session, 2026-09-09): protectedplanet.net's India country page still states "India chooses to restrict some data on its protected areas... data on 900 protected area(s) are not publicly available." Kaziranga (a public World Heritage entry) has a real indexed page; Sathyamangalam has none at all. Confirms the 24 Aug finding — a WDPA_API_KEY would not deliver Sathyamangalam's polygon regardless. Vishnu chose to leave this open/revisit later rather than mark permanently_excluded now — no code/config change made. If revisited, the real path to a boundary polygon is OSM/Overpass (needs its own robots.txt exemption decision), the 2013 TR notification text (already in the corpus), or an archived pre-restriction WDPA release.

## management-plan — real bug found and fixed

The stream's code was already correct (reads from R2 by content-addressed key, no local-file fallback). The actual bug: the PDF was uploaded to R2 on 2026-08-22 using `wrangler r2 object put --local` — the local dev sandbox, not the real bucket. Confirmed via a real `--remote` get: "The specified key does not exist." A live harvest run would have hit the stream's own clean "Expected PDF not found in R2" error — not silent, but never fixed either.

Fixed: re-uploaded the same file (verified same sha256, `8cc0c8ad...79989`) to the real bucket with `--remote` and `--content-type=application/pdf` (missing from the original upload). Round-trip verified: 15,855,991 bytes, hash matches. `sources/management-plan.json`'s note corrected to say what actually happened (today's date, `--remote`) rather than just swapping the flag on the old date.

Also: `git rm --cached` on the 16MB PDF (still on disk, unmodified), added to root `.gitignore`. Does not shrink the existing 71MB `.git` history — that's a separate, optional job.

**Left for later:** `management-plan.js`'s own runtime error message still tells the reader to re-upload with `--local` — the exact instruction that caused this bug. One-word fix, intentionally left since it's a code change outside this task's original scope. Worth folding into the next commit that touches this stream.

## Cohort B status: effectively closed for now

- bhl — blocked pending Vishnu checking key status
- wdpa — deliberately left open, not pursued
- management-plan — fixed and verified

Not run end-to-end (no live harvest triggered). npm test unchanged: 39 pass / 5 fail.
