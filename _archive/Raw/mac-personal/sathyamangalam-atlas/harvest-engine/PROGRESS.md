# Harvest Engine — Progress

Tracks stage-by-stage completion per `harvest-engine/harvestenginemasterprompt.md`. Each entry is written only after self-verification against real evidence (live run, job_run row, R2 listing) — never on a prose summary alone.

---

## Stage 0 — Foundation

Done prior to this log's creation. See commit `e152604`.

## Stage 1 — Biodiversity

Confirmed done and verified (commit `cd3ad2b`) before this session began. GBIF, eBird, iNaturalist streams live; India Biodiversity Portal and Xeno-canto and IUCN Red List not yet added as of that commit (not re-verified in this session — Stage 1 was taken as given per explicit instruction).

## Stage 2 — Scientific literature

**Status: done, with two sources blocked on missing credentials/connectivity — see below.**

### What was built

Nine new streams, one per source, following the Stage 1 one-file-per-source convention:

| Stream | File | Source |
|---|---|---|
| `openalex` | `src/streams/openalex.js` | OpenAlex Works API |
| `crossref` | `src/streams/crossref.js` | Crossref Works API |
| `semanticscholar` | `src/streams/semanticscholar.js` | Semantic Scholar Graph API |
| `core` | `src/streams/core.js` | CORE v3 search API |
| `europepmc` | `src/streams/europepmc.js` | Europe PMC REST API (covers PubMed) |
| `unpaywall` | `src/streams/unpaywall.js` | Unpaywall (DOI enrichment, not search) |
| `ia-scholar` | `src/streams/ia-scholar.js` | Internet Archive advancedsearch.php |
| `shodhganga` | `src/streams/shodhganga.js` | Shodhganga OAI-PMH — **blocked, see below** |
| `bhl` | `src/streams/bhl.js` | Biodiversity Heritage Library — **blocked, see below** |

New shared library: `src/lib/document.js` (find-or-create + merge for the `document` table, mirroring `src/lib/taxon.js`'s pattern). New source registries: `sources/openalex.json`, `crossref.json`, `semanticscholar.json`, `core.json`, `europepmc.json`, `unpaywall.json`, `ia-scholar.json`, `shodhganga.json`, `bhl.json`. New shared query-term config: `sources/literature-terms.json`.

**Bug fixed in shared Stage 0 infrastructure**: `src/lib/robots.js`'s `checkRobotsAllowed` treated any non-200/non-404 robots.txt response as fail-closed (disallowed). `api.unpaywall.org/robots.txt` returns HTTP 422 (it requires `?email=` on every route, including robots.txt) — this would have wrongly blocked the entire Unpaywall stream. Fixed to treat any 4xx as "no robots.txt present" (same as 404), fail closed only on 5xx/network errors. Verified this doesn't change behavior for any Stage 1 source (GBIF/eBird/iNaturalist all use plain 200/404).

**Bug found and fixed during verification**: `findOrCreateDocument` (and the OpenAlex stream specifically) didn't guard against a work with neither `title` nor `display_name` set, hitting `document.title`'s `NOT NULL` constraint (`D1_ERROR: NOT NULL constraint failed: document.title`, seen live during the first OpenAlex run — 17 rows affected). Fixed by skipping title-less records before calling the helper (matching the pattern every other Stage 2 stream already used) and added a defensive throw inside `findOrCreateDocument` itself so this class of bug can't recur silently in a future stream.

### Query terms

The master prompt directs this stage to run "the full query set from `sathyamangalam/harvest-plan.md` Stream 3" — **this file does not exist anywhere in the repo** (confirmed by a full-repo search before writing any code). Rather than block all of Stage 2 on a missing planning doc, `sources/literature-terms.json` was derived directly from the reserve's own identity (the `0001_init.sql` seed row), tribal/place names already present in `docs/*.md` (Irula, Sholiga, Kurumba), and Stage 1's own species/place framing — 18 terms across five categories (core, place, tribal, invasive-species, focal-species). If `harvest-plan.md` turns up later, diff it against this file and re-run affected streams; idempotency means new terms only add rows, never duplicate existing ones. The "second pass: citation graph, two hops from the 20 core papers" instruction could not be executed at all, since neither the 20 core papers list nor `sathyamangalam/source-atlas.md` exist in the repo either — not attempted this stage.

### Live verification evidence (2026-08-21, local `wrangler dev`)

All nine streams were triggered via the dashboard's "Run now" API against real, live external hosts (not mocked) and their `job_run` rows inspected directly in local D1.

| Stream | job_run status | rows_written | Notes |
|---|---|---|---|
| openalex | `success` (after fix) | 2226 | Idempotent re-run confirmed: 19 pages skipped as duplicate, 0 new writes, no errors |
| crossref | `success` | 3660 | Clean, no errors |
| semanticscholar | `partial` | 75 | Only 3/15 terms got a 200 before the shared *unkeyed* pool's rate limit was hit (confirmed live and independently reproduced with a bare `curl` — this is real external throttling, not a bug); the other 12 terms logged clear `HTTP 429 after 3 retries` errors and will retry on the next scheduled run (no `source` row was written for them, so they are not falsely deduped) |
| core | `partial` | 60 | 10/15 terms succeeded before hitting CORE's rate limit; same pattern as above |
| europepmc | `success` | 198 | Clean; idempotent re-run confirmed separately: 19 pages skipped, 0 new writes |
| ia-scholar | `success` | 40 | Clean |
| unpaywall | `success` | 6 | Enrichment pass — 6 documents (of the ones already carrying a DOI but no `oa_url`) got a legal OA link filled in this run; 1704 documents total now carry `oa_url` (most set directly by OpenAlex/Semantic Scholar's own OA fields, not just this enrichment pass) |
| shodhganga | `failed` (expected) | 0 | See blocker below |
| bhl | `failed` (expected) | 0 | See blocker below |

**Aggregate**: 4097 `document` rows written, 0 orphaned (every row's `source_id` resolves to a real `source` row), full provenance chain spot-checked (title → DOI → source name → R2 key, all consistent and on-topic).

**Correction (added 2026-08-22, during Stage 2–5 re-verification ahead of Stage 8)**: the per-stream `rows_written` figures in the table above are the raw `job_run` values recorded *before* the `rows_written`-overcounting fix landed in Stage 4 (see that section), and overstate true new-row counts. Real counts, from a direct `SELECT COUNT(*) ... GROUP BY source` against the live `document` table: `crossref-works-search` 2168 (not 3660), `openalex-works-search` 1711 (not 2226), `europepmc-rest-search` 101 (not 198), `ia-scholar` 35 (not 40), `core-works-search` 29 (not 60), `semanticscholar-paper-search` 53 (not 75). No duplication occurred — these are lower, not higher, than the table above; treat the original numbers as "work attempted," these as the real row counts. `unpaywall`'s 6 and the 1704-document `oa_url` aggregate were independently re-confirmed and are accurate as printed.

**R2 raw-before-parse confirmed** via direct inspection of the local R2 object store: `openalex` 50 objects, `crossref` 74, `semanticscholar` 4, `core` 15, `europepmc` 19, `ia-scholar` 19, `unpaywall` 95. (`bhl` and `shodhganga` correctly have 0 — both failed before any network call was made.)

**Sensitive-species coarsening**: not applicable to this stage — Stage 2 writes only to `document`, never `occurrence`/coordinates, so the coarsening trigger from Stage 0 has nothing to verify here (per the master prompt's own definition-of-done wording: "for any stream touching occurrence/coordinate data").

### Blockers (flagged per the master prompt's own pause criteria, not silently worked around)

1. **BHL requires a credential the master prompt calls keyless.** Live test against `api3?op=PublicationSearch` returned `{"Status":"unauthorized","ErrorMessage":"'' is an invalid or unauthorized API key."}`. BHL's key is free and self-service (via a web form at biodiversitylibrary.org), similar in spirit to eBird's Stage 1 key — but no `BHL_API_KEY` exists in `.dev.vars`, and registering one requires a human to submit contact details, which this agent cannot do. The stream (`src/streams/bhl.js`) is fully implemented and registered, but gated to fail cleanly with an actionable message until the key is added. **Action needed from Vishnu**: register a free key at biodiversitylibrary.org and add `BHL_API_KEY` to `.dev.vars` / Worker secrets, then re-run the `bhl` stream from the dashboard.
2. **Shodhganga was unreachable from this environment.** Both `robots.txt` and the OAI-PMH `?verb=Identify` endpoint hung to a connection timeout (not a 4xx/5xx — a network-level failure), confirmed independently with a bare `curl`. This is a known-flaky host, not a terms/credential issue. The stream (`src/streams/shodhganga.js`) is fully implemented per the master prompt's infrastructure-reuse rule but is unverified against live data. It will attempt again on the next scheduled cron tick; `withJobRun`'s error boundary means a continued outage surfaces as a clean `failed` job_run, never a crash or a silent gap.

Per the master prompt's guidance ("otherwise, keep going stage to stage without asking for permission to proceed" — these are single-source blockers, not whole-stage blockers, and rule #4's error-boundary design means a failing stream never stops the others), Stage 2 is being marked done with these two sources explicitly flagged rather than holding up Stages 3–8 waiting on a credential only Vishnu can obtain.

---

## Stage 3 — Gazetteer

**Not started. A real blocker was found during Stage 2's downtime and is flagged here now, ahead of starting Stage 3's implementation, per the master prompt's own anticipation of this exact issue.**

The master prompt already flags this stage as blocked pending a decision on Overpass's robots.txt vs. authenticated-client status. Live verification confirms the blocker is real and is broader than just Overpass:

- **`overpass-api.de/robots.txt`** explicitly disallows `/api/` for all user agents — and `/api/interpreter` (the actual query endpoint Stage 3 needs) is under that exact disallowed path. Verified live 2026-08-21.
- **`query.wikidata.org/robots.txt`** explicitly disallows `/sparql` for all user agents — again, the exact endpoint Stage 3's Wikidata SPARQL requirement needs. Verified live 2026-08-21.

Both disallows are unambiguous, on-path, and unexplained by any terms-of-service page found that would supersede them (attempted to check; found no explicit "our terms override robots.txt for API clients" statement for either service). Per rule #3 ("Where a source registry's API's own terms of use supersede robots.txt, note that explicitly... don't silently bypass") and the master prompt's own explicit Stage 3 pause instruction, **this is not being resolved by inference** — both are surfaced together as one decision needed from Vishnu before Stage 3's Overpass/Wikidata components can be built:

- Treat these as authenticated API clients (common real-world practice for SPARQL/Overpass query endpoints — the disallow typically targets search-engine crawlers indexing query-string URLs as if they were pages, not deliberate API consumers) and proceed, OR
- Respect the robots.txt disallow literally and find alternative sources for gazetteer data (e.g. OSM's export/Planet data via a different access pattern, Wikidata's REST API instead of SPARQL where coverage allows).

**Decision (Vishnu, 2026-08-21): treat Overpass and Wikidata SPARQL as authenticated API clients.** Proceeded to build Stage 3 in full on that basis — both streams identify the bot honestly via User-Agent (per rule #3's spirit even though the disallow is being knowingly not followed literally), pass `acknowledgedOverride: true` explicitly to `checkRobotsAllowed` (a new, narrowly-scoped opt-in added to `src/lib/robots.js` — default `false`, so every other source's behavior, including all of Stage 1/2, is unchanged), and the override plus its justification is recorded in each stream's `sources/*.json` entry under `robots_txt_override`.

### What was built

Six new streams: `overpass`, `wikidata`, `wdpa`, `lgd`, `census`, `bhuvan`. New shared library `src/lib/place.js` (find-or-create for the `place` table, mirroring `document.js`/`taxon.js`). New source registries for all six plus their query-term/config data.

A background research pass (before writing LGD/Census/Bhuvan/WDPA code) found: LGD has no public API of its own — data.gov.in mirrors it; Census 2011 village data is bulk-PDF-only, no API; Bhuvan has a real WMS/WFS; WDPA's bulk shapefile download needs no key but its exact monthly-versioned filename isn't discoverable without the same gated API, so the token-gated search API was used instead as the more stable contract.

### Live verification evidence (2026-08-21, local `wrangler dev`)

| Stream | job_run status | rows_written | Notes |
|---|---|---|---|
| wikidata | `success` | 19 (18 written to `place` after 1 dedupe) | Clean; idempotent re-run confirmed: 1 skipped as duplicate, 0 new writes |
| overpass | `failed` (see note) | 0 | **Not a code bug** — see below |
| census | `failed` (see note) | 0 | **Local-dev-only limitation** — see below |
| wdpa | `failed` (expected) | 0 | Missing `WDPA_API_KEY`; fails cleanly with an actionable message |
| lgd | `failed` (expected) | 0 | Missing `DATA_GOV_IN_API_KEY` AND a real `resource_id` (still a placeholder in `sources/lgd.json` — needs a one-time manual dataset lookup at data.gov.in); fails cleanly |
| bhuvan | `failed` (expected) | 0 | Host unreachable — see Stage 2-style note in `sources/bhuvan.json` |

**Overpass debugging finding**: extensive live debugging (documented in `sources/overpass.json`) traced repeated 406 responses to Cloudflare's local-dev (`wrangler dev`/Miniflare) egress path specifically — the exact same request from a direct `curl` on the same machine, at the same moment, got a clean 200, while every attempt from inside the local Worker got 406, including the `robots.txt` fetch itself. This points to the shared local-dev egress IP pool having been rate-limited/flagged by Overpass (plausibly from cumulative load across many developers' local testing), not a defect in this stream. A retry-on-transient-status (406/429/502/503/504) mechanism was added regardless, since Overpass's own multi-backend pool does genuinely have real, separate transient-failure modes worth retrying. **Needs re-verification against a deployed Worker** (real Cloudflare edge IPs, not the local-dev proxy) before concluding whether it works in production — do not read the local `failed` status as proof this stream is broken.

**Census finding**: `censusindia.gov.in` is fully reachable via `curl` (TLS 1.3, valid cert from `emSign PKI`, an Indian CA) but every fetch from local `wrangler dev` throws `internal error; reference = ...` from the Workers runtime itself, reproduced in isolation with a minimal test Worker. This strongly suggests a gap in Miniflare's local TLS trust store for smaller/regional CAs — again, **needs re-verification against a deployed Worker** before concluding it's broken; local dev's failure here is not evidence against production. **Update (Stage 4, wrangler v4 upgrade)**: this project's `wrangler`/Miniflare was upgraded from 3.114.17 to 4.125.0 during Stage 4 (see that section) to fix an unrelated PDF-extraction bug, and the new Miniflare version is a plausible fix for this TLS gap too, but the `census` stream itself was not re-tested against the new version before this entry was written — re-verify next time Stage 3 or Stage 5 (which also touches `censusindia.gov.in`-adjacent Internet Archive content) is worked on.

**R2 raw-before-parse confirmed**: `overpass` 1 object (the 406 error page itself — rule #1 applies even to failed fetches), `wikidata` 1 object. `census`, `wdpa`, `lgd`, `bhuvan` correctly have 0 (all failed before any successful network call completed).

**Provenance**: 18 new `place` rows, 0 orphaned (every row's `source_id` resolves to a real `source` row).

**Correction (added 2026-08-22, during Stage 2–5 re-verification ahead of Stage 8)**: same pre-fix `rows_written`-overcounting issue as Stage 2's table above applies here too. Real `place` counts from the live database: `overpass-place-query` 40 (not the 42 in the table above), `wikidata-sparql-settlements` 18 (matches the table's "18 written after 1 dedupe" exactly). Small deltas, no duplication.

**Sensitive-species coarsening**: not applicable — Stage 3 writes only to `place`, never `occurrence`/coordinates.

### Blockers and follow-ups (for Vishnu)

1. **WDPA needs a free token.** Register at `api.protectedplanet.net/request`, add `WDPA_API_KEY`. (A keyless bulk-shapefile alternative exists at `protectedplanet.net/country/IND` but its exact monthly filename isn't reliably discoverable without the same API — documented in `sources/wdpa.json` as a fallback, not implemented.)
2. **LGD needs two things**: a free `DATA_GOV_IN_API_KEY` (instant via data.gov.in account signup), AND a one-time manual lookup of the real LGD-villages dataset's `resource_id` at data.gov.in (search "local government directory villages"), to replace the placeholder in `sources/lgd.json`.
3. **Bhuvan's WMS host was unreachable** during this session — best-effort implementation only fetches `GetCapabilities` as raw XML (not yet the actual forest-cover layer, since its real layer name couldn't be confirmed against a live capabilities response). Will need re-attempting once the host is reachable.
4. **Overpass and Census both need re-verification against a deployed Worker**, not just local dev — see the findings above. If either still fails once deployed, that would be a genuine finding worth revisiting; if they work (as circumstantial evidence suggests), no further action needed.
5. `reserve.boundary_geojson` is still empty (only the bbox exists) — Stage 1's own instruction to "fetch and freeze [the WDPA polygon] once, reuse everywhere" was never implemented in Stage 1 (verified: no WDPA fetch exists in the repo before this stage). Once `WDPA_API_KEY` is available, a follow-up increment to `wdpa.js` should fetch the actual polygon geometry (the current implementation only stores a text note that a WDPA match was found, since the search endpoint's summary response doesn't include full geometry — a separate detail-endpoint call is needed, not yet built since it couldn't be verified without a token).

---

## Stage 4 — Government and legal documents

**In progress.** Preliminary connectivity check (opportunistic, during Stage 2 wait time): `mee-tr.wii.gov.in` was unreachable (connection failure) at check time 2026-08-21 — worth re-checking now that this stage is actually starting. `sathytiger.tn.gov.in` explicitly allows all crawling (`Disallow:` empty). `indiankanoon.org`'s API host returned an empty 200 body on its root — its actual auth requirement needs real verification against current docs, per the master prompt's own note to check.

**Missing prior art**: the master prompt directs this stage to be "a port of the existing (separate, do-not-touch) `crawler.js` pattern, not a rebuild" — no `crawler.js` exists anywhere in this repo (confirmed by full-repo search). This is the third master-prompt reference to supporting material that turns out not to exist (`sathyamangalam/harvest-plan.md` and `sathyamangalam/source-atlas.md` in Stage 2, now this). **Decision (Vishnu, 2026-08-21): build the listing-page → PDF-links → hash → dedupe → R2 → text-extraction → queue pipeline fresh**, following the shape Stage 4 describes as closely as possible, rather than pausing to search further or skipping ahead. If a real `crawler.js` turns up later (a different branch, an uncommitted local file, another repo), diff this implementation against it and reconcile.

**Status: done for 4 of 7 candidate sources, with the other 3 explicitly deferred (not silently skipped) — see below.**

### What was built

New shared pipeline `src/lib/crawler.js`: `fetchListingPage` (fetch a listing page, save raw HTML, extract PDF links via regex) and `fetchAndExtractPdf` (fetch one PDF, save raw bytes, extract text via `unpdf`). New shared helper `src/lib/legal-instrument.js` (find-or-create for `legal_instrument`, dedupes by `r2_key` since a PDF's content hash is a stronger identifier than a listing page's often-inconsistent title text). Four working streams: `forests-tn`, `ntca` (two listing pages: nationwide sanction orders + the reserve-specific tiger-reserves table), `wii` (tiger status reports + MEE reports), `erode-nic`.

New dependency: `unpdf` (a Workers-native PDF.js build) for text extraction — chosen over `pdf-parse`/native alternatives specifically because it has no native/binary dependencies, unlike `renderPageAsImage`'s `@napi-rs/canvas` requirement (confirmed incompatible with Workers — that path is why OCR isn't implemented yet, see below).

**Explicit Stage 4 rate-limit requirement applied**: `src/lib/rate-limit.js`'s `.gov.in` floor was raised from Stage 1's 1000ms to the 1500ms Stage 4 specifies ("do not use a faster default"). This also retroactively makes Stage 3's `census`/`bhuvan`/`lgd` streams (which touch `.gov.in`-family hosts) more conservative — confirmed this doesn't change their pass/fail status, only spacing.

### A real, unrelated infrastructure fix made along the way: wrangler v3 → v4

While first testing PDF extraction, every single `unpdf` call failed inside local `wrangler dev` with `TypeError: Cannot set properties of undefined (setting '_isSameOrigin')` — a PDF.js internal error from code that expects a browser-like `window.location`. Reproduced identically with both `unpdf`'s bundled build and the official `pdfjs-dist` package, ruling out an `unpdf`-specific bug. This project was still on `wrangler@3.114.17` (the CLI warned about being out of date on every command run this whole project). **Decision (Vishnu, 2026-08-21): upgrade to `wrangler@4`.** Confirmed live: `npm install --save-dev wrangler@4` (now `4.125.0`) upgraded cleanly with zero config changes needed and **zero `npm audit` vulnerabilities** (all prior audit warnings traced to the old wrangler's own `esbuild`/`sharp`/`undici` dependency chain, unrelated to any harvest-engine code). PDF extraction then worked correctly on the first real test — 83 pages, 4370 characters of real, readable text extracted from a genuine 8.5MB government PDF. The `pdfjs-dist` package installed for testing the official-build path was removed again afterward since `unpdf`'s own bundled build works fine post-upgrade.

**This upgrade is a standing risk worth flagging**: Stage 0–3 were built and verified against `wrangler@3`. Spot-checked after the upgrade: `wikidata` stream still runs and dedupes correctly under `wrangler@4`; general Worker/queue/D1/R2 behavior looks unchanged. A full re-verification pass of every prior stream under `wrangler@4` was NOT done (would substantially duplicate work already recorded as evidence) — if something in Stage 1–3 behaves unexpectedly in a future session, check whether it's actually a `wrangler@4` behavior difference before assuming a regression in that stage's own code.

### A real bug fixed across Stages 2–4: `rows_written` overcounting

Found live: re-running the `ntca` stream reported "20 rows written" when the actual `legal_instrument` table had gained zero new rows (all 20 were dedupe hits against already-known PDFs). Root cause: `findOrCreateDocument`, `findOrCreatePlace`, and `findOrCreateLegalInstrument` all silently returned the existing row on a dedupe hit, with no way for a caller to distinguish that from a genuine insert — and every caller across every Stage 2–4 stream unconditionally incremented `rowsWritten` right after calling one of these, regardless. **This was a metrics-accuracy bug only, not a data-integrity bug** — the underlying tables were never actually duplicated (confirmed: `legal_instrument` count stayed at 20 before and after the miscounted re-run). Fixed by changing all three helpers to return `{ row, created }` and updating every one of the 16 affected call sites (`core.js`, `census.js`, `bhl.js`, `erode-nic.js`, `crossref.js`, `ia-scholar.js`, `lgd.js`, `forests-tn.js`, `europepmc.js`, `ntca.js`, `semanticscholar.js`, `shodhganga.js`, `openalex.js`, `wii.js`, `wikidata.js`, `overpass.js`) to only increment `rowsWritten` when `created` is true. `findOrCreateTaxon` (Stage 1) was deliberately left unchanged — its callers (`gbif.js`/`ebird.js`/`inaturalist.js`) already gate `rowsWritten` on a separate `occurrence`-level dedup check, not on taxon creation, so they never had this bug. **This means every `rows_written` figure recorded in this file for Stage 2 and Stage 3 before this fix landed may be a modest overcount** relative to true new-insert counts (though never an undercount, and never evidence of a data-duplication problem) — treat those historical numbers as "work attempted," not a precise new-row tally, unless re-verified.

### A real bug fixed in shared Stage 0 infrastructure: robots.txt empty-`Disallow` parsing

Found live: the `wii` stream reported `wii.gov.in` as disallowing crawling, but `wii.gov.in/robots.txt` is `User-agent: * / Disallow:` (an empty Disallow value) — which per the robots.txt spec means "disallow nothing," not "disallow everything." `src/lib/robots.js`'s `isAllowed()` was treating an empty-path rule as matching every path, then picking whichever rule (allow or disallow) had that empty path, incorrectly returning "disallowed" for a host with no real restriction. Fixed to skip empty-path rules entirely when finding the best-matching rule. **Blast radius checked**: only `wii.gov.in` (this stage) and `sathytiger.tn.gov.in` (Stage 3, but never implemented as a live stream — deferred for JS-rendering reasons, so the bug never actually manifested there) have this exact empty-`Disallow` shape among every host touched so far; no other Stage 1–3 evidence needs revisiting.

### A design inconsistency fixed: two different robots.txt-override mechanisms

While building the Stage 3 Overpass/Wikidata overrides, added a new `acknowledgedOverride` parameter to `checkRobotsAllowed` itself — without first checking that Stage 1's `ebird`/`inaturalist` streams (which also override a blanket robots.txt disallow, for the same "documented API client, not a generic crawler" reasoning) already used a different, established pattern: `checkRobotsAllowed` always reports the true verdict, and each stream's own code decides whether to proceed anyway based on `sourceConfig.robots_txt_override?.applies` from that source's registry entry. **Reconciled**: reverted the `acknowledgedOverride` parameter from `robots.js` (checking robots.txt now behaves identically to before Stage 3 for every call site) and rewrote `overpass.js`/`wikidata.js` plus their `sources/*.json` entries to match `sources/ebird.json`'s established shape (`applies`/`checked_url`/`checked_result`/`justification`/`decided_by`/`decided_at`) exactly.

### Live verification evidence (2026-08-21/22, local `wrangler dev`, after the wrangler v4 upgrade)

| Stream | job_run status | rows_written | Notes |
|---|---|---|---|
| ntca | `success` | 20 | Real, live sanction-order PDFs fetched from both listing pages, text extracted from 9/20 (the other 11 are genuinely scanned/no-text-layer PDFs — confirmed via job_run's `errors_json` being empty, not a silent extraction failure), 0 orphaned rows |
| forests-tn | `success` | 0 | Listing page fetched and parsed correctly; genuinely zero PDFs on the homepage's "latest live" feed matched the reserve-relevant term filter at check time (confirmed by inspecting the live page's actual PDF filenames — all generic department notices/RFQs, none Sathyamangalam-specific) |
| wii | `success` | 0 (after fix) | Both listing pages fetched correctly (2 R2 objects confirmed) after fixing the `wii.gov.in` robots.txt false-positive above; genuinely zero reserve-relevant PDFs in the current nationwide listings |
| erode-nic | `success` | 0 | Listing page fetched; zero PDFs on the current page matched the relevance filter this run. When a relevant PDF IS found, it is expected to be blocked at fetch time — `cdn.s3waas.gov.in` (where erode.nic.in's PDFs are actually hosted) has `robots.txt: Disallow: /` (a blanket disallow, verified live), and this is treated as a real, unambiguous restriction (unlike Overpass/Wikidata's SPARQL-endpoint special case) — no override applied |

**R2 raw-before-parse confirmed**: real PDF bytes and listing-page HTML both saved before any parsing, per rule #1 — spot-checked by copying a raw PDF blob out of local R2 and independently re-running `unpdf` extraction against it outside the Worker, matching the Worker's own extracted text exactly (83 pages, 4370 characters, byte-identical result).

**Provenance**: 20 `legal_instrument` rows, 0 orphaned.

**Sensitive-species coarsening**: not applicable — Stage 4 writes only to `legal_instrument`, never `occurrence`/coordinates.

### Sources explicitly deferred (not silently skipped)

- **`sathytiger.tn.gov.in`** (the reserve's own site) — confirmed to be a client-side-rendered SPA (a raw GET on its Tenders/G.O.s pages returns an empty ~1KB shell); would need a headless browser (e.g. Cloudflare's Browser Rendering API), a materially different architecture than every other Stage 4 source's plain-HTML-fetch assumption. `sources/sathytiger.json` has an empty `sources` array with the reasoning recorded.
- **`egazette.gov.in`** — classic ASP.NET WebForms search (`SearchMenu.aspx`), which needs `__VIEWSTATE`/postback state simulation rather than a plain GET-with-querystring; judged too fragile to build unverified (a subtly-wrong postback sequence risks silently-malformed requests, not a clean failure). `sources/egazette.json` has an empty `sources` array.
- **`tamilnaduarchives.tn.gov.in`** — confirmed to have no online catalog or finding aid at all; physical-archive-only. `sources/tamilnaduarchives.json` has an empty `sources` array.
- **`indiankanoon.org`** — confirmed live: requires Token-based auth on every request (401 on an unauthenticated search), and no self-service signup form was found (contact-based access per their site, unlike BHL/WDPA/data.gov.in's instant forms). Unlike those three, no stream code was written at all — the actual API response shape can't be verified without first securing access, and building against a guessed contract risks worse-than-nothing behavior once real access is granted. `sources/indiankanoon.json` documents the blocker; `src/streams/indiankanoon.js` does not exist yet.

### OCR fallback — not implemented, here's why

The master prompt asks for "OCR fallback (Workers AI or equivalent) for scanned Government Orders." Live evidence now confirms this is a real need (11/20 real NTCA PDFs have no text layer). Investigated the actual pipeline needed: `unpdf`'s `renderPageAsImage` (the obvious way to turn a scanned PDF page into an image for OCR) explicitly requires `@napi-rs/canvas`, a native Node addon — **confirmed incompatible with Cloudflare Workers**, which cannot load native/binary addons. A viable alternative path exists — `unpdf`'s `extractImages` (pulls a scanned page's single embedded raster image, Workers-compatible, no canvas needed) piped through `@jsquash/png` (a WASM PNG encoder, genuinely Workers-compatible) into a Workers AI vision/OCR model — but this needs its own new binding (`[ai]` in `wrangler.toml`, not yet added), a new WASM dependency, and — critically — cannot be verified in local `wrangler dev` without a live Workers AI binding, which behaves differently locally vs. deployed. Given the added scope and that it can't be confidently verified without deploying, this was deferred rather than built speculatively. **Action needed from Vishnu**: decide whether to (a) add the `[ai]` binding and accept building this pipeline without local verification, relying on post-deploy testing, or (b) treat OCR as a Stage 8-adjacent follow-up once the project already has Workers AI wired up for something else.

### Blockers and follow-ups (for Vishnu)

1. **Indian Kanoon needs contact-based API access** — not a simple free-signup key like BHL/WDPA. See above.
2. **OCR fallback is scoped but not built** — needs a decision on the `[ai]` binding, see above.
3. **`sathytiger.tn.gov.in` needs headless-browser rendering** to ever be crawlable — a bigger investment than this stage's other sources; revisit only if this specific site's content turns out to be uniquely valuable enough to justify it.
4. **eGazette needs ASP.NET postback handling** — same "bigger investment" judgment call as sathytiger.

---

## Stage 5 — Historical full-text mining

**Done.** Six historical Internet Archive volumes downloaded and fuzzy-mined for place-name spelling variants.

### Missing prior art (same pattern as Stages 2 and 4)

The master prompt names six volumes by author/year ("Nicholson 1887, Francis 1908, Madras District Manuals, Buchanan 1807, etc.") and directs using exact item IDs from `sathyamangalam/source-atlas.md`, and a spelling-variant list from `sathyamangalam/harvest-plan.md` Stream 4 — **neither file exists anywhere in this repo** (confirmed by full-repo search; this is the fourth master-prompt reference to supporting material that turns out absent, after `harvest-plan.md`/`source-atlas.md` in Stage 2 and `crawler.js` in Stage 4). Both were derived fresh:

- **Volume identifiers**: found via a dedicated research pass against Internet Archive's own search/metadata APIs, each independently verified to resolve with real metadata and a real `_djvu.txt` file. Documented in `sources/historical-volumes.json`. **One honest gap**: the real "Francis 1908" (W. Francis's Madras District Gazetteers: Coimbatore, the main descriptive volume) could not be found on Internet Archive after an extensive multi-query search — confirmed Francis's gazetteers for *other* districts (Vizagapatam 1907, Bellary 1916) ARE digitized, proving the series is partly on IA, but Coimbatore's own main volume isn't there under any searchable cataloging. Substituted with the closest real relative (a 1915 statistical/appendix Vol. II for the same district/series) rather than fabricating or guessing an identifier — flagged clearly in that file's `notes` field, not silently passed off as the requested volume.
- **Spelling-variant list**: derived from well-documented 18th–19th century British colonial transliteration patterns for this region (Tamil/Kannada sounds rendered inconsistently by different surveyors — `th`/`t`, `v`/`w`/`b`, terminal `-am`/`-um`/`-oor`), stored in `sources/historical-spelling-variants.json`.

### What was built

New shared library `src/lib/fuzzy-match.js` (pure Levenshtein-distance matcher, no external dependency — a Worker has no native fuzzy-string library available and the corpus size doesn't need one) and `src/lib/historical-passage.js` (insert helper for `historical_passage` — no dedup-by-content here, unlike the other find-or-create helpers, since the same term can legitimately match more than once at different positions in the same book). New stream `historical-text.js`.

**A real accuracy issue found and fixed during testing**: an initial, naive flat Levenshtein-distance-2 tolerance produced heavy false-positive noise on short terms — confirmed live against the real 1887 Nicholson text, "Erode" (5 letters) matched common English words like "trade", "prove", "Europe", "rose" at distance 2. Fixed by scaling the effective tolerance down to 1 for terms of 6 characters or fewer, cutting total matches from 86 to 59 on the same test corpus while keeping all the genuine hits (Satyamangalam, Bhavani, Hasanur, Talamalai, Gobichettipalayam all still matched correctly, including real OCR-garbled spellings like "Bhavdni" and "Satyamahgalam"). Some residual noise remains for common short words (documented as a known, accepted limitation of fuzzy-matching short strings against noisy OCR text, not something to fully eliminate without more sophisticated NLP than this stage's scope calls for).

**Page-number attribution is a best-effort estimate, not exact**: DjVu OCR text from Internet Archive doesn't reliably carry page-break markers (confirmed live: zero form-feed characters in a real 1887 volume's full text). `page` is instead estimated by counting standalone digit-only lines encountered before a match's position (page numbers that were printed on their own line in the original scan) — this is documented in the code and here as an approximation, not presented with false precision. ~96% of matches got a non-null estimate in practice.

**A real, retry-safety bug found and fixed**: the stream initially deduped on whether the whole-volume fetch had happened (via `request_hash`), not on whether the resulting `document`/`historical_passage` rows were actually created. A transient Miniflare-internal D1 error during testing crashed the stream mid-run, after one volume's raw text had already been fetched and saved (creating a `source` row) but before its `document`/passages were written — on retry, the fetch-level dedup check then permanently skipped that volume's document/passage creation forever, since it only ever checked "was this fetched," not "did this actually finish." **Fixed**: the dedup check now looks for the volume's own `document` row (matched by its deterministic `archive.org/details/<identifier>` URL) before deciding to skip; if a `source` row exists but no `document` row does, the stream re-reads the already-fetched raw text from R2 (no re-fetch from Internet Archive needed) and completes the document/passage creation. Verified live: the stuck volume completed correctly on the next run, and a further idempotency check confirmed all 6 volumes then dedupe cleanly with 0 new writes.

### Live verification evidence (2026-08-22, local `wrangler dev`)

| Metric | Value |
|---|---|
| `document` rows (type=`historical-volume`) | 6 |
| `historical_passage` rows | 1182 |
| Orphaned passages (no matching document) | 0 |
| Idempotent re-run | Confirmed: 0 new writes, 6/6 correctly skipped as duplicate on a clean re-run |

4 of 6 volumes hit the stream's own `MAX_PASSAGES_PER_VOLUME_PER_RUN = 200` safety cap (only one came in under, at 182) — common terms like "Coimbatore" and "Mysore" genuinely appear very frequently throughout region-specific 19th-century gazetteers, so this reflects real density of relevant content, not a bug. **Not a silent truncation**: documented here and in the stream's own comments — a volume with more than 200 matches has real content beyond what one run captures; a future increment could raise the cap or paginate across runs if more complete coverage is wanted.

**R2 raw-before-parse confirmed**: each volume's whole `_djvu.txt` saved to R2 before any fuzzy-matching, per rule #1.

**Sensitive-species coarsening**: not applicable — Stage 5 writes only to `document`/`historical_passage`, never `occurrence`/coordinates.

**Per the master prompt's own instruction for this stage**: no attempt was made to interpret or annotate the meaning of matched passages — each `historical_passage.annotation` field is left null, ready for human review, exactly as directed ("this stage produces raw extracted material for human annotation... don't try to auto-annotate meaning").

---

## Stage 6 — Remote sensing and environmental layers

**Done — all six GEE layers (Sentinel-2 NDVI, Hansen forest-loss, CHIRPS rainfall, SRTM elevation/slope, ERA5-Land temperature, and the Landsat 1984-onward annual NDVI time series) plus FIRMS live and verified; mechanics and value-correctness both checked, not just HTTP-200 assumed.** See the "Third re-check" update below for the auth/registration history including a real "200 with a wrong number" incident caught and fixed on review, and the "Sixth and last GEE layer" section further down for the Landsat time series' own four-trap verification. `FIRMS_MAP_KEY` and the GEE service account are both configured — spot-checked FIRMS live with a real API call against the reserve bbox, confirmed working (returned a valid CSV header for the VIIRS_SNPP_NRT dataset, correctly wrote 1 `observation_layer` row for 0 detections that day). GEE's own REST API requires signed-JWT OAuth2 token exchange (RS256, using the service account's private key) rather than a simple API key — implemented via the Workers-native Web Crypto API (`src/lib/gee-auth.js`), more complex and security-sensitive than any stream built so far, and kept in its own small file for that reason.

**Config bug found and fixed**: `.dev.vars` had `GEE_SERVICE_ACCOUNT_KEY_PATH=./secrets/gee-service-account.json` — a filesystem path, which a Worker cannot read at runtime (no filesystem access). Nothing in the codebase actually consumed that variable name; it was dead config from an earlier, incorrect assumption. Fixed by replacing it with `GEE_SERVICE_ACCOUNT_JSON`, containing the service account file's actual JSON content inline (matching what `gee-auth.js`/`sources/gee.json` document and expect). `.dev.vars` is gitignored and untracked, confirmed before editing.

**`src/lib/gee-expression.js`**: builds the GEE REST `value:compute` expression-graph JSON for three computations — Sentinel-2 NDVI, Hansen forest-loss pixel count, CHIRPS rainfall total. Two rounds of research against the `google/earthengine-api` GitHub repo's own source (not guessed) caught three real bugs before any live test: (1) `Collection.filterDate`/`filterBounds` aren't real server algorithms — must build `Filter.dateRangeContains`/`Filter.intersects` fed into raw `Collection.filter`; (2) `Image.normalizedDifference`/`Image.select` take the image argument as `input`, not `image` (an inconsistency confirmed real, not a mistake, vs. `Image.reduceRegion`/`Image.gte` which do use `image`/`image1`/`image2`); (3) the DateRange value constructor is the bare, unnamespaced server function `DateRange` — not `DateRangeConstructors.DateRange`, which was an incorrect guess by analogy to the confirmed-real `GeometryConstructors.Polygon` (that namespacing pattern doesn't generalize) — and `Filter.dateRangeContains` takes the DateRange node as `leftValue`, with the per-image date property as `rightField`, not the reverse. Both citations: `python/ee/daterange.py` and `python/ee/tests/algorithms.json` in the earthengine-api repo.

**`src/streams/gee.js`**: new stream tying `gee-auth.js` + `gee-expression.js` together, following the same shape as every other Stage 1–6 stream (`withJobRun`, raw-before-parse to R2, `source`/`observation_layer` rows, request-hash dedup keyed to a rolling date window so it naturally re-runs rather than being deduped forever). Registered in `src/streams/index.js`. Runs all three computations per invocation; a per-computation failure is recorded in `errors` without stopping the other two (same "partial success" convention as Stage 2's rate-limited literature streams).

**Live verification (2026-08-22, local `wrangler dev`, real network calls)**: triggered via the dashboard's run-now API → queue → `job_run` row inspected directly in local D1. Result: `status=failed`, all three computations returned a clean `HTTP 403 PERMISSION_DENIED: Project sathyamangalam-record is not registered to use Earth Engine` — the same known blocker flagged before this session, not a new bug. This is actually strong positive evidence for the implementation: JWT signing + OAuth2 token exchange succeeded (no auth error), and a wrong function name or argument shape in the expression graph would have produced an EE-side `400` (bad request), not a `403` (authorization) — meaning the graph parsed and reached GEE's authorization layer correctly, including the DateRange/`leftValue` fix above. Confirmed 0 `source`/`observation_layer` rows written on this failure (no orphaned rows) and the raw 403 response body correctly saved to R2 before being reported as an error, per rule #1. **Still cannot be confirmed as fully correct end-to-end** (a successful compute call, with sane NDVI/forest-loss/rainfall numbers) until registration is done — the expression graphs are "verified to authorize correctly, not yet verified to compute correctly."

**Action needed from Vishnu**: visit `https://console.cloud.google.com/earth-engine/configuration?project=sathyamangalam-record` and register the project for Earth Engine access (one-time GCP console step, cannot be done by this agent). Once done, re-run the `gee` stream from the dashboard — no code change should be needed, only re-verification. If the response is still an error afterward (e.g. a real 400 from a bad expression), that would be a genuine new finding worth revisiting the DateRange assumption over.

**Re-verification after registration (2026-08-22, later the same day)**: Vishnu confirmed registering `sathyamangalam-record` for Earth Engine via `code.earthengine.google.com` and confirmed the Earth Engine API shows enabled in Cloud Console. Re-ran the `gee` stream live three times from this session (`job_run` ids 63, 65, 66 — id 64 failed instantly on a stale `GEE_SERVICE_ACCOUNT_JSON is not set` error caused by this session's local `wrangler dev` process having started hours before `.dev.vars` was last edited; restarting the dev server fixed that specific error, unrelated to the registration question). All three real, live re-runs after the restart (ids 63, 65, 66, spanning about 3.5 minutes) got the **exact same byte-for-byte 403 `Project sathyamangalam-record is not registered to use Earth Engine`** response as before registration. This is a live, current finding, not a stale one — **registration has not yet taken effect from this environment's calls**, whether due to propagation delay on Google's side (GCP API enablement can lag beyond a few minutes in some cases) or an incomplete registration step (e.g. registered under a personal Earth Engine account not linked to this exact GCP project/service-account pairing, a known real-world gotcha with EE's registration flow). **Action needed from Vishnu**: (1) confirm the registration was completed against the exact project id `sathyamangalam-record` and not a different/personal EE project, (2) allow more time for propagation and ask this agent to re-run the `gee` stream again later, or (3) check `https://console.cloud.google.com/earth-engine/configuration?project=sathyamangalam-record` directly for the project's current registration status. Not marking this resolved until a live call actually succeeds.

**Second re-check (2026-08-23, ~1 hour after the above, during Stage 8 work)**: re-ran the `gee` stream once more (job_run id 72) purely opportunistically, since a `wrangler dev` instance happened to be up for Stage 8 testing. Still the identical byte-for-byte 403 on all three computations. This rules out a short propagation delay (well over an hour has now passed) — the registration/project-linkage question above remains the live blocker.

**Third re-check (2026-08-23, later the same day) — 403 resolved, real bugs surfaced instead.** Vishnu confirmed the GCP project was verified directly in the Earth Engine configuration page (Community/noncommercial tier), not just the general signup flow this time. Re-ran the `gee` stream (job_run id 75): **the 403 `PERMISSION_DENIED` was completely gone** — all three computations reached GEE's compute layer and returned real `400 INVALID_ARGUMENT` errors instead, confirming the registration question is now genuinely resolved and exposing two real bugs in `src/lib/gee-expression.js` that had been unreachable until authorization actually succeeded:

- **`vegetation-ndvi` and `rainfall`** (same root cause): `HTTP 400 reduce.median: Property '.all' with value '<Image<[...]>>' is not a valid operand for Intersects<e:ErrorMargin(0.1 METERS)>`. Root cause found by reading the earthengine-api repo's own `python/ee/filter.py` source (not guessed): the real client-library `Filter.geometry()` (what `Collection.filterBounds()` calls under the hood) wraps its geometry argument in a bare `Feature(...)` constructor call before using it as `Filter.intersects`'s `rightValue` — `filterByBounds()` was passing the raw `GeometryConstructors.Polygon` value directly, producing a malformed filter that GEE only rejected later, confusingly, at the `reduce.median`/`reduce.sum` step rather than at the filter itself.
- **`forest-loss`**: `HTTP 400 Image.gte, argument 'image2': Invalid type. Expected type: Image<unknown bands>. Actual type: Integer. Actual value: 21`. Confirmed from the same repo's `algorithms.json` fixture: `Image.gte`/`Image.lte`'s `image1`/`image2` arguments are both strictly typed `Image`, not raw numbers — `buildForestLossExpression` was passing the bare `lossYearMin`/`lossYearMax` integers directly. Fixed by wrapping each in `Image.constant(value)` first (confirmed a real, correctly-typed algorithm from the same fixture).

**Fixed both, re-ran live (job_run id 76)**: `status=success`, `rows_written=3`, all three returning real computed values instead of error bodies:
```
vegetation-ndvi:   {"result": {"nd": 0.10888092001949569}}
forest-loss:       {"result": {"lossyear": 3350.2705882352939}}
rainfall:          {"result": {"precipitation": 4.9814133424760056}}
```
This was reported as fully resolved at the time — **it wasn't.** A structurally-valid 200 response is not the same as a correct one, and two of these three numbers were wrong in ways that only surfaced by checking them against real-world plausibility, not by any HTTP error. Caught on review: NDVI at 0.109 is far too low for the reserve's real forest canopy (0.6-0.9 expected), and a field named `lossyear` holding `3350.27` makes no sense for a band whose real values are small integers (1-23). Investigating both — and separately re-checking rainfall's units/period on the same pass — found three distinct real issues:

1. **NDVI (`buildNdviExpression`) had no cloud/shadow masking at all.** The date window (2026-07-24 to 2026-08-23) is Tamil Nadu's monsoon season; Sentinel-2 SR's raw reflectance values include heavy cloud/haze contamination during this window that a per-pixel median across a mostly-cloudy month doesn't fully filter out on its own. Fixed by mapping a QA60-bitmask cloud/cirrus mask (bit 10 = opaque cloud, bit 11 = cirrus — Sentinel-2 SR's own documented format) over every image in the collection via `Collection.map` before compositing. This needed a first-class GEE "function value" (`FunctionDefinition`/`argumentReference`), the most structurally complex node built in this file so far, which surfaced its own real bug: GEE's REST API's authoritative discovery document (fetched live from `https://earthengine.googleapis.com/$discovery/rest?version=v1` — not the Python client library's `.py` source, which doesn't publish this shape) defines `FunctionDefinition.body` as a **string reference** into the enclosing `Expression`'s flat `values` map, not an inline nested value node the way every other function argument in this file works. A first attempt nested the mask body inline and got a real, live `400 Starting an object on a scalar field`. Fixed by adding a small slot-allocator (`createSlotAllocator`/`slots.allocate`) so `maskS2Clouds()`'s body gets a real top-level `values` entry, referenced by its string key.
2. **Forest-loss's field name was actively misleading, though the computation itself was legitimate.** Traced precisely: `Image.gte`/`Image.lte` name their output bands "for the longer of the two inputs" (confirmed from the algorithm signature fixture) — since `lossYearBand` carries the name `lossyear` and the `Image.constant` threshold images are unnamed, that name persists through every subsequent step into the final `reduceRegion` result key, even though by then the band holds a boolean in-range mask, and after `Reducer.sum`, a raw pixel count — not a year. The magnitude itself is plausible, not just accepted on faith: 3350 pixels at Hansen's native 30m resolution ≈ 3.02 km² of flagged loss (years 2021-2023) within the ~2585 km² lon/lat bbox this stream currently uses (the bbox, not the true reserve polygon — `sources/wdpa.json`'s still-open blocker), which is a reasonable order of magnitude, not obviously wrong. Fixed with `Image.rename` so the output key is now honestly `loss_pixel_count`, matching what the value actually is.
3. **Rainfall was quietly computed over 8 real days, not the requested 31.** A debug `Collection.size` check (built once, run against a few date-window slices, then removed) against `UCSB-CHG/CHIRPS/DAILY` directly under the exact same `filterByDate`/`filterByBounds` helpers the production code uses found that CHIRPS's real, published data for this dataset ends at **2026-07-31** — 2026-08-01 onward currently has zero images, a genuine external processing-latency gap (CHIRPS is a blended satellite/gauge product, not a same-day feed), not a query bug. The 30-day rolling window (`new Date()` minus 30 days) has no way to know this ahead of time, so `4.98mm` was really an 8-day total silently reported as if it were a 31-day one — correct arithmetic over a truncated window, exactly the "no silent caps" failure this project's own convention (Stage 5's PROGRESS.md) warns about. Not fixed by clamping the date (would need to track CHIRPS's actual publication lag over time, a moving target this stage's scope doesn't cover) — instead, `image_count` was added to both the rainfall and NDVI results (`Dictionary.set` on the `reduceRegion` output, via `Collection.size` on the actual filtered-and-masked collection) so every future consumer can see how many real days/images a value actually covers rather than trusting the requested window blindly.

**Cleared the three known-wrong rows and re-ran live end to end (job_run id 78, partial — NDVI's new `Collection.map` code hit the `FunctionDefinition.body` bug above; job_run id 79, success once fixed)**. Real, current values, pulled directly from D1 and cross-checked against the raw R2 response bodies:
```
vegetation-ndvi:   {"image_count": 32, "nd": 0.40723600014207512}
forest-loss:       {"loss_pixel_count": 3350.270588235294}
rainfall:          {"image_count": 8, "precipitation": 4.981413342476006}
```
NDVI rose from 0.109 to 0.407 with real cloud masking applied (32 real Sentinel-2 images now confirmed feeding the composite, up from an unmasked, cloud-contaminated equivalent). 0.407 is still below a "dense forest canopy" figure — the most plausible remaining explanation, not yet independently re-verified, is that this is a spatial mean over the full lon/lat bbox (which genuinely includes agricultural land, the Bhavanisagar reservoir area, and settlements around Sathyamangalam/Bhavani/Gobichettipalayam per the Stage 8 management-plan text, not just reserve forest interior) rather than evidence of a further masking bug — the same underlying "bbox vs. real reserve polygon" gap already on record from Stage 3. Flagged as a follow-up, not silently accepted as fully correct.

**Idempotency confirmed on the corrected code** (job_run id 80, same rolling window): `rows_written=0`, `rows_skipped_duplicate=3` — a clean re-run with no new writes.

**Stage 6's mechanics (JWT auth, OAuth2 token exchange, GCP registration, all three expression graphs reaching and being accepted by GEE's compute layer) are confirmed working end-to-end, and the three specific value-correctness bugs found on review are fixed and re-verified.** A follow-up round of independent verification (2026-08-23, same session) resolved the one open caveat above by cross-checking each corrected value against real, independent evidence rather than accepting it on the same computation's own say-so:

**1. NDVI (0.407) — cross-checked against Hansen tree-cover data and the management plan's own text; the bbox-dilution explanation was wrong, the real explanation is forest type.** Built a temporary debug endpoint (removed after use, never committed) that scanned a 6×4 grid of cells across the full bbox using Hansen's independent `treecover2000` band. Result: mean tree cover across the bbox is genuinely low (20.46%, cross-confirmed two ways: the grid-cell average and a separate direct bbox-wide `Reducer.mean` call agree to 4 significant figures), with no single coarse cell exceeding ~45%. Ran NDVI over the single highest-cover coarse cell (44.6%) — got 0.399, essentially unchanged from the full bbox's 0.407. **This refuted the bbox-dilution explanation**: if dilution were the whole story, the densest cell should have shown meaningfully higher NDVI. Drilled into a 5×5 fine sub-grid inside that cell and found a genuinely dense 76% tree-cover micro-patch (`77.2035°E–77.2075°E, 11.6985°N–11.7025°N`); NDVI there was 0.547 — trending upward with real tree cover, but still below 0.6. A per-pixel min/max/count check on that same micro-patch (2025 valid 10m pixels after cloud masking) found the single highest NDVI pixel in the entire search was **0.644** — barely inside the requested 0.6-0.9 range, with the mean well below it. Given the max ceiling itself doesn't reach typical dense-evergreen NDVI even in the best patch found, checked the management plan's own vegetation-type text (read live from the actual PDF, `TAMILNADU_FOREST_DEPARTMENT_MANAGEMENT_P.pdf` pages 41-45): Sathyamangalam's vegetation is explicitly documented as **southern tropical dry thorn forest, dry mixed deciduous forest, and semi-evergreen forest** (Champion & Seth 1968 classification) — not dense evergreen/rainforest, which is what a 0.6-0.9 NDVI expectation is really calibrated to. Dry thorn/dry deciduous canopies are naturally sparser and have lower peak NDVI than evergreen forest, even at high tree-cover percentages, as a matter of real forest ecology, not a computational artifact. **Conclusion: 0.407 (bbox mean) and up to ~0.64 (best real micro-patch found) are consistent with Sathyamangalam's actual, documented dry-thorn/dry-deciduous forest type — the earlier 0.6-0.9 expectation was calibrated to the wrong forest type for this specific reserve, not evidence of a remaining bug.** The cloud-masking fix is confirmed working correctly (masked composite tracks real tree cover in the right direction and magnitude); the absolute ceiling is a real ecological property of this reserve, not a defect.

**2. Rainfall (`image_count: 8` of 31 requested days) — confirmed as a real, current, external CHIRPS data-latency gap, not a query bug.** Probed the exact date boundary directly: single-day `Collection.size` checks for `UCSB-CHG/CHIRPS/DAILY` returned exactly 1 image each for 2026-07-29, 07-30, and 07-31, and exactly 0 for 2026-08-01 and 08-02 — a clean, sharp cutoff, not a scattered/partial gap the way a filtering bug would typically produce. Re-checked several dates further into August (08-10, 08-15, 08-20, 08-22) — all zero, confirming this isn't a narrow missing-week but a hard "nothing published past 2026-07-31 yet" boundary. 2026-07-24 through 2026-07-31 inclusive is exactly 8 calendar days, matching `image_count: 8` precisely — every real day that exists was correctly picked up by the filter, none were missed or double-counted. This is expected, documented behavior for CHIRPS (a blended satellite/rain-gauge product with real weeks-long processing latency before final data is published), not a defect in `filterByDate`/`filterByBounds`.

**3. Forest-loss (`loss_pixel_count: 3350.27`) — converted to real units and time-bound context.** At Hansen's native 30m resolution (the `scale: constant(30)` this stream itself uses): `3350.27 pixels × 900 m²/pixel = 3,015,244 m² ≈ 3.02 km² (≈301.5 hectares)` of flagged tree-cover loss. **Time period**: confirmed from the stream's own constants (`HANSEN_MOST_RECENT_LOSSYEAR_MAX = 23`, `HANSEN_LOSSYEAR_WINDOW = 3` in `src/streams/gee.js`) that `lossYearMin=21, lossYearMax=23` maps to **calendar years 2021, 2022, and 2023** — a real, defined 3-year rolling window, not an unbounded or ambiguous range. For context, also computed the full unfiltered 2001-2023 Hansen loss total for the same bbox: 9083.15 pixels ≈ 8.17 km² (≈817.5 ha) — meaning the 2021-2023 window accounts for **36.9% of all tree-cover loss recorded in the dataset's entire 23-year history**, within just its most recent 3 years. Against the bbox's own approximate forested area (using the 20.46% mean tree-cover figure: ~528.9 km² of the ~2585 km² bbox), the 2021-2023 loss represents **~0.57% of forested area lost in 3 years**, and the all-time figure **~1.55% of forested area lost over 23 years** — both are real, interpretable, and not obviously anomalous figures for a landscape under real, documented pressure from *Prosopis juliflora*/*Lantana camara* invasion and human-wildlife conflict (per the same management plan text).

All three corrected values are now independently cross-checked against real external evidence (Hansen tree-cover data, the management plan's own vegetation-type documentation, direct CHIRPS date-boundary probes, and unit-converted/time-bound Hansen loss figures) rather than accepted on the strength of a 200 response alone. **Stage 6 is done.**

**Coordination gap (unresolved)**: the master prompt directs checking the separate `senna-regrowth-verification` project's docs before building GEE export logic, to avoid duplicating its work. That project **does not exist anywhere on this machine** — checked via Spotlight's indexed search (`mdfind`), targeted checks of `~/Projects`, `~/Documents`, `~/Desktop`, and home directory, and a full-disk `find` (which completed with no matches). **Decision (Vishnu, 2026-08-22): proceed with Stage 6's GEE work regardless, scoped narrowly to what this stage itself needs, and flag this gap rather than block on it.** If `senna-regrowth-verification` turns out to have existing GEE export code once located, reconcile this stage's implementation against it rather than assuming no overlap.

### Fourth GEE layer added: SRTM elevation/slope (2026-08-23)

Built as the template for two more planned layers (ERA5-Land, Landsat) — the metadata pattern established here is meant to be copied, not just the numbers. `buildElevationExpression` in `src/lib/gee-expression.js`; wired into `src/streams/gee.js`'s `COMPUTATIONS` array as `kind: "elevation"`; registry entry `gee-srtm-elevation` in `sources/gee.json`.

**Dataset**: `USGS/SRTMGL1_003` (SRTM 30m, band `elevation`), chosen over `COPERNICUS/DEM/GLO30` — SRTM is a single static `ee.Image` (no mosaic step), and 11.5°N is well inside SRTM's ±60° coverage; GLO30 is an `ee.ImageCollection` of tiles needing `.mosaic()` first, more complexity with no accuracy benefit here. No date dimension, no cloud masking — the simplest possible GEE layer, which is why it went first.

**Reducer output key naming was verified live, not guessed by analogy** (a debug endpoint built, run, and removed before writing the real code): `Reducer.minMax()` on a band named `elevation` returns `elevation_min`/`elevation_max`, but `Reducer.mean()` on that same band returns the **bare band name** `elevation`, not `elevation_mean`. The same pattern holds for `Terrain.slope`'s own fixed output band name `slope`. Guessing the mean case by analogy to minMax's suffix would have been wrong — confirmed by a live `400 Dictionary.get: Dictionary does not contain key: 'elevation_mean'` on the first attempt.

**Live result** (job_run id 82, `rows_written=1, rows_skipped_duplicate=3` — the pre-existing ndvi/forest-loss/rainfall computations correctly deduped, only the new elevation computation ran):
```json
{"elevation_max": 2098, "elevation_mean": 787.09, "elevation_min": 187,
 "slope_max": 79.57, "slope_mean": 11.15, "slope_min": 0}
```
**Idempotency confirmed** (job_run id 83, same day): `rows_written=0`, `rows_skipped_duplicate=4` — a clean re-run with no new writes across all four computations.

**Plausibility check against the task's ~250-1800m expectation band**: mean (787m) and most of the range fall inside it; min (187m) and max (2098m) fall modestly outside both ends. Investigated rather than dismissed: this is a bbox-wide extremum, not the reserve interior — the bbox's own known composition (Bhavanisagar reservoir plain at the low end, real Eastern Ghats peaks at the high end, per Stage 8's management-plan text) plausibly explains both tails without indicating a wrong asset, bbox, reducer, or unit error.

**Cross-checked against an independent historical source, not just the plausibility band**: Nicholson's 1887 *Manual of the Coimbatore District* (already harvested in Stage 5, `historical_passage` id 39, source_id 379, page 32) states, of the Sathyamangalam/Bhavani hill-range taluk specifically: *"its general elevation is between 2,000 and 3,000 feet"* — the full sentence was truncated in the stored 506-character passage; recovered by pulling the raw R2-stored OCR text directly (`raw/sathyamangalam/historical-text/5f/5f861662...`). Converted: 2,000-3,000 ft = **609.6-914.4 m**. The live SRTM mean (787.09m) falls almost exactly inside this independent 1887 figure for the taluk's *general* (i.e. typical/settled) elevation — a real, external agreement, not just an internal plausibility check. The wider SRTM min/max (187-2098m) appropriately captures the full bbox's extremes (reservoir plain to Ghats peaks) that a single "general elevation" prose figure was never describing in the first place — the two figures are consistent, not contradictory, once that distinction is made explicit.

**Metadata defect fix, applied retroactively to all pre-existing rows, not just the new one**: every `observation_layer` row previously stored its computation's context only in prose (this file, or a code comment) — none of the four prior rows (`vegetation-ndvi`, `forest-loss`, `rainfall`, `active-fire-detections`) carried `dataset_version`, `geometry_scope`, `reduction_scale`, or `units` as structured fields. Added as top-level keys in `params_json` (queryable via `json_extract`/`json_set`, confirmed live in this D1 build — no schema migration needed) for the new elevation computation going forward in `gee.js`/`firms.js`, and backfilled onto the 4 existing rows via a one-time `UPDATE ... SET params_json = json_patch(...)` (id 1 needed a follow-up `json_set` fix: `json_patch`'s RFC-7396 merge-patch semantics silently *drop* a key set to JSON `null`, unlike `JSON.stringify` in application code, which keeps it — caught by checking the row after the patch rather than assuming it worked). `geometry_scope` is `"bbox"` on every row, explicitly, not left implied: `reserve.boundary_geojson` is confirmed `NULL` (still blocked on the missing `WDPA_API_KEY`, unchanged from Stage 3).

**Also fixed while here**: `sources/gee.json`'s three pre-existing entries still described the Earth Engine registration 403 as current and called the stream "implemented-but-unverified" — stale since job_run id 76, which already showed live 200s. Corrected all three `notes` fields and added the new `gee-srtm-elevation` entry in the same structure.

### Fifth GEE layer added: ERA5-Land 2m air temperature (2026-08-23)

Second of three planned layers, following the SRTM layer's metadata pattern exactly (`dataset_version`, `geometry_scope`, `reduction_scale`, `units`), plus one new field this layer introduces: `data_latency_days`.

**Latency measured live for both candidate products before choosing, not guessed or skipped** (a temporary debug endpoint, removed after use): per-day `Collection.size` boundary probes against the bbox found `ECMWF/ERA5_LAND/DAILY_AGGR`'s most recent real day was **8 days** behind the probe date (0 images through 7 days back, 1 image from 8 days back on — a clean, sharp cutoff, not scattered) and `ECMWF/ERA5_LAND/HOURLY`'s boundary was between 5 and 7 days back (day 6 only 13 of 24 hours populated, day 7 fully populated with 24) — i.e. HOURLY is real, but only ~1-2 days fresher. **Chose DAILY_AGGR anyway**: the task needs a daily min/mean/max, which DAILY_AGGR already provides as one pre-aggregated image per day; HOURLY would need an extra per-day aggregation step duplicating that work, for a small latency gain judged not worth it — the same "use the pre-aggregated daily product" shape already used for CHIRPS. The query window (`src/streams/gee.js`'s `TEMPERATURE_LATENCY_DAYS=8`, `TEMPERATURE_WINDOW_DAYS=7`) is deliberately built to END at the measured latency boundary, not at "today" — so it never silently reports fewer real days than requested, the CHIRPS incident above not repeated.

**Two real bugs found live, not guessed past**: (1) band names `temperature_2m`/`temperature_2m_min`/`temperature_2m_max` were confirmed correct on the first live probe, but all three are in **Kelvin** — confirmed by a live single-day `reduceRegion.mean` returning `297.43`, which only makes sense as ~24.3°C in Kelvin. Converted explicitly via a `KELVIN_TO_CELSIUS_OFFSET` subtraction rather than stored raw — this project's second unit-conversion trap of this kind, after forest-loss's pixel-count/area confusion. (2) The Kelvin→Celsius subtraction itself first used `Number.subtract({value, value2})`, guessed by analogy to `Image.gte`'s `image1`/`image2` shape — this produced a real, live `400 Parameter 'left' is required and may not be null` on the actual gee stream run (job_run id 85), not caught by an earlier isolated debug check that happened not to exercise this path. Fixed by testing `Number.subtract({left, right})` directly against a trivial `10 - 3 = 7` check before reapplying it — confirmed correct, then the real stream re-run succeeded (job_run id 86).

**Live result** (job_run id 86, `rows_written=1, rows_skipped_duplicate=4`; window 2026-08-08 to 2026-08-15, `image_count=7` confirming the window landed entirely inside available data):
```json
{"image_count": 7, "temperature_mean_c": 23.92, "temperature_min_c": 20.01, "temperature_max_c": 29.26}
```
**Idempotency confirmed** (job_run id 87, same day): `rows_written=0, rows_skipped_duplicate=5` across all five computations now registered.

**Plausibility**: mean 23.92°C sits squarely in the task's expected "low-to-mid 20s" band; min 20.01°C and max 29.26°C are both reasonable for a late-August tropical bbox. No Kelvin leak.

**Cross-checked against an independent historical source**: Stage 8's `extracted_table` has no temperature figures (only Appendices 3/4/5/28 — mammals/birds/butterflies/maps — have been extracted so far; no climate/meteorology appendix is in scope yet). Found instead in Stage 5's already-harvested historical text: the 1915 *Madras District Gazetteers: Coimbatore Vol. II* (`historical_passage` id 1128, source_id 381, page 3778) states *"the mean annual temperature, as determined by the observations of eighteen years, is 77.6°[F], which is lower than that of Madras... April has the highest monthly average (83.3°[F])."* Converted: 77.6°F = **25.33°C** (annual mean), 83.3°F = **28.50°C** (April peak). The live ERA5-Land mean (23.92°C) is within ~1.4°C of this annual figure, and the live max (29.26°C) sits close to the historical April peak — a real, external agreement, given the two figures aren't measuring quite the same thing (this ERA5 window is one week in late August, not an annual mean; the 1915 gazetteer's figure is for the Coimbatore observatory specifically, not the Sathyamangalam bbox — the same passage itself explicitly flags that Coimbatore's readings "are not applicable to the whole district without considerable qualification," a caveat worth repeating rather than glossing over).

**Elevation-averaging limitation, recorded in the registry rather than corrected for**: SRTM measured 187-2098m of relief inside this same bbox; a single bbox-mean temperature averages across the resulting ~12-13°C of lapse-rate difference between the lowest and highest ground, so the stored mean/min/max are genuine bbox-wide reanalysis statistics, not a reserve-representative point temperature. See `sources/gee.json`'s `gee-era5-temperature` entry for the full note.

**`data_latency_days` backfilled onto the pre-existing rainfall (CHIRPS) row** (id 3), converting a previously prose-only finding into a queryable field: `22` — the number of days between the last live re-check date (2026-08-22) and the last real CHIRPS image found (2026-07-31), both already documented above under Stage 6's original rainfall review. Not re-probed for this task, per that section's own "no silent caps" convention of trusting the last directly measured value rather than assuming it's stale. The other three pre-existing rows (`vegetation-ndvi`, `forest-loss`, `active-fire-detections`) do NOT yet carry this field — only the rainfall backfill was in this task's scope; they'll pick it up automatically on their next real (non-deduped) run under the code paths now in place.

### Sixth and last GEE layer added: Landsat 1984-onward annual NDVI time series (2026-08-23)

Third and last of the three planned GEE layers, and structurally different from every prior computation in this stream: it writes **one `observation_layer` row per year**, not one row per run. New builder `buildLandsatAnnualNdviExpression` in `src/lib/gee-expression.js`; new dedicated per-year loop in `src/streams/gee.js` (after the existing `COMPUTATIONS` loop, since a single-hash-per-run shape doesn't fit a series); new registry entry `gee-landsat-ndvi` in `sources/gee.json`.

**All four traps flagged in the task brief were probed live before writing any real code, using a temporary `/api/debug/gee` route (added to `src/index.js`, removed after use — same pattern as the SRTM/temperature layers' now-removed debug endpoints; confirmed by `git diff` showing zero net change to `src/index.js`).**

**TRAP 1 (scaling factors) — confirmed against Google's own dataset catalog pages for all four sensors, not just trusted from the task brief, AND confirmed empirically against raw pixel values:**
```
$ curl https://developers.google.com/earth-engine/datasets/catalog/LANDSAT_LC08_C02_T1_L2 (and LT05/LE07 equivalents)
SR_B1-SR_B7 band table: Scale=2.75e-05, Offset=-0.2, identical across LT05/LE07/LC08/LC09.
```
Live raw-pixel probe (unscaled `Image.reduceRegion` minMax over the bbox, Landsat 8, Jan-Mar 2024 window):
```json
{"QA_PIXEL_max": 55052, "QA_PIXEL_min": 21762, "SR_B4_max": 52860, "SR_B4_min": 91, "SR_B5_max": 54604, "SR_B5_min": 6482, "image_count": 4}
```
Raw `SR_B4`/`SR_B5` values (91-54604) only make sense as scaled 16-bit surface reflectance once `multiply(0.0000275).add(-0.2)` is applied (a value of ~52860 becomes ~1.25, plausible near-saturated reflectance; 91 becomes ~-0.197, plausible for water/deep shadow) — confirmed, not assumed.

**TRAP 2 (cloud masking) — QA_PIXEL bitmask confirmed per-sensor from Google's own catalog pages (bits 1/3/4 identical across all four sensors; bit 2 differs — "Unused" on TM/ETM+, real Cirrus on OLI/OLI-2), and a masked-vs-unmasked comparison run live to prove the mask has real bite, per the task's explicit requirement:**
```
Unmasked full-year 2024 Landsat 8 NDVI over bbox: {"image_count": 22, "nd": 0.554607254679729}
Masked (QA_PIXEL bits 1/3/4, same 22 scenes) NDVI:  {"image_count": 22, "nd": 0.568645351687568}
```
A separate, sharper check on a single Jul-Sep 2024 (monsoon) scene found **99.9993% of its pixels flagged cloudy** by the same mask logic (`{"QA_PIXEL": 0.999992729996285, "scene_count": 4}`) — confirming the mask has large, real bite on genuinely cloud-contaminated scenes; it simply doesn't need to do much work on an already-mostly-clear full-year 2024 composite, which is why the full-year unmasked-vs-masked delta above is modest (+0.014) rather than dramatic like the earlier Sentinel-2 finding (0.109→0.407 — that window was specifically the monsoon month, not a full year).

**TRAP 3 (cross-sensor band numbering) — confirmed from Google's own per-sensor catalog band tables, not assumed identical to Sentinel-2's B4/B8 used elsewhere in this file:**
- TM (Landsat 5) / ETM+ (Landsat 7): `SR_B3` = red (0.63-0.69µm), `SR_B4` = near-infrared (0.77-0.90µm).
- OLI (Landsat 8) / OLI-2 (Landsat 9): `SR_B4` = red (0.636-0.673µm), `SR_B5` = near-infrared (0.851-0.879µm).

Each sensor family gets its own explicit `{redBand, nirBand}` entry in `LANDSAT_SENSORS`/`LANDSAT_SENSOR_PRIORITY`; NDVI is computed with the correct pair per sensor rather than one hardcoded mapping. **Cross-sensor comparability is NOT silently assumed**: exactly one sensor is used per year (never a blend — see the priority-order finding below), and every stored row's `params_json.sensor` records which one, per the task's explicit instruction. No cross-sensor radiometric adjustment is applied (see the 2013-transition finding below for why this matters).

**TRAP 4 (SLC-off) — decision: INCLUDE post-2003 Landsat 7, confirmed safe via a live per-pixel valid-image-count check, not assumed:**
```
2005 (8 real ETM+ scenes for this bbox, full year): SR_B3 median-composite per-pixel valid-image count:
{"SR_B3_max": 8, "SR_B3_min": 2, "mean_count": 6.9744039213011062}
```
Every pixel location in the bbox retains **at least 2 of 8** scenes' worth of valid (non-gap) data after compositing — the scan-line gaps shift position scene-to-scene (each overpass is a different striping pattern), so a multi-scene median composite fills them in almost everywhere. **Caveat, recorded rather than solved**: this protection depends on having enough scenes in a year; a year with very few real ETM+ scenes (this bbox's actual per-year JFM counts for ETM+ ranged 1-5) would have thinner protection — not re-verified per-year, since the min-2-of-8 finding was judged sufficient evidence SLC-off inclusion is sound in general for this bbox, not that every single year is individually immune.

**Compositing season, chosen after live-probing seasonal cloud cover AND scene density across the WHOLE archive, not just one recent year:**
```
2023 seasonal cloud-flag fraction (Landsat 8, QA_PIXEL bits 1/3/4):
  Jan-Mar: 37.6% cloud, 6 scenes | Apr-Jun: 25.3% cloud, 5 scenes | Jul-Sep: 57.1% cloud, 5 scenes | Oct-Dec: 62.4% cloud, 6 scenes
```
Apr-Jun looked clearest in this single 2023 check — but a spread-check of scene COUNTS across sensors/years found Apr-Jun collapses in the older archive:
```
LE07 2005: Jan-Mar=4 scenes, Apr-Jun=1 scene   |   LE07 2010: Jan-Mar=4, Apr-Jun=1   |   LT05 1995: Jan-Mar=3, Apr-Jun=1
```
**Decision: fixed Jan 1 - Mar 31 (JFM) every year** — Tamil Nadu's dry season between the Oct-Dec northeast monsoon and the Jun-Sep southwest monsoon, and far more consistently sampled across all four sensor generations than the marginally-clearer-in-one-recent-year Apr-Jun window.

**Sensor selection per year — exactly one sensor, never blended, checked in priority order (OLI-2 > OLI > ETM+ > TM), first with a real scene wins.** An empty collection makes GEE itself throw `HTTP 400 Image.select: Band pattern 'SR_B3' was applied to an Image with no bands` at the `reduceRegion` step (confirmed live, e.g. probing 1984) — this specific message is treated as "this sensor has zero scenes this year," not a real error, and the next (older) sensor in priority order is tried.

**Live run (job_run id 88)**:
```
status=partial, rows_written=36, rows_skipped_duplicate=5
errors_json: 6 entries, all "no Landsat scenes found for any sensor in the Jan-Mar window (real archive gap for this bbox, not an error)" — years 1984, 1985, 1986, 1987, 1989, 1996
```
`status=partial` reads as "failed" only because the pre-existing status rule is `rows_written>0 && errors.length>0 => partial` — these 6 "errors" are genuine, expected archive gaps, not defects; confirmed live (`Collection.size` probes) that Landsat 5 has zero real scenes over this bbox for 1984, 1985, 1986, 1987, and 1989 specifically (1984-87 predates this exact path/row being tasked; 1989's gap has no obvious single cause and wasn't investigated further, per the task's "don't gap-fill, just record it" instruction). Per this task's explicit instruction, these 6 years get **no `observation_layer` row at all** — not a fabricated, interpolated, or zero-value row.

**Idempotency confirmed PER YEAR, not just per run** (job_run id 89, immediate re-run): `rows_written=0`, `rows_skipped_duplicate=41` — exactly the 36 real Landsat years (each deduped via its own per-year `request_hash`, confirming per-year idempotency genuinely works, not just per-stream-run) plus the 5 pre-existing non-Landsat computations. **One real, minor inefficiency found and recorded rather than silently fixed**: the 6 real-gap years have no `source` row to dedupe against (nothing was ever written for them), so a re-run re-probes all 4 sensors for each of them again — confirmed live, `errors_json` on job_run id 89 lists the identical 6 years with identical messages. This wastes ~24 GEE calls per re-run but is not incorrect: it can never produce a duplicate row, only ever the same accurate "still no data" outcome. Left as-is rather than adding a new "known-empty year" cache, since this task's scope was the series itself, not optimizing repeated-failure cost; worth revisiting if the stream's cron cadence makes this a real, recurring API-quota concern.

**A real off-by-one bug found and fixed in `lastCompletedLandsatYear()` immediately after the above, on self-review before reporting done**: the function computed `year - 2` when run before April (should be `year - 1`) and `year - 1` from April onward (should be the current `year` itself, since that year's own Jan-Mar window has by then genuinely completed) — it was uniformly one year too conservative, silently excluding the single most-recently-completed year from the series regardless of when the stream ran. On this session's actual run date (2026-08-23, August, month index 7 ≥ 3) this meant 2026 — a real, available, fully-elapsed JFM year — was being skipped entirely. Fixed the boundary arithmetic, restarted `wrangler dev`, and re-ran: **job_run id 90**, `rows_written=1, rows_skipped_duplicate=41` — exactly the newly-unlocked 2026 year, with the 36 pre-existing years and 5 non-Landsat computations correctly deduping. New row: `{"dataset_version":"LANDSAT/LC09/C02/T1_L2","year":2026,"sensor":"OLI-2 (Landsat 9)","result":{"result":{"image_count_after_mask":6,"image_count_before_mask":6,"nd":0.5634672002365286}}}`. **Final idempotency re-run** (job_run id 91, all 37 real years + 5 non-Landsat computations now on file): `rows_written=0`, `rows_skipped_duplicate=42` — the same 6 gap-year errors, nothing new or unexpected. `SELECT kind, COUNT(*) FROM observation_layer GROUP BY kind` confirms `vegetation-ndvi-landsat-annual` at exactly **37**.

**Full year-by-year series** (37 rows, `SELECT date_from, params_json FROM observation_layer WHERE kind='vegetation-ndvi-landsat-annual' ORDER BY date_from`, values rounded to 4dp here for readability — raw values are stored unrounded):

| Year | Sensor | NDVI | Images (after/before mask) |
|---|---|---|---|
| 1988 | TM | 0.5394 | 1/1 |
| 1990 | TM | 0.3559 | 3/3 |
| 1991 | TM | 0.3473 | 3/3 |
| 1992 | TM | 0.3417 | 2/2 |
| 1993 | TM | 0.4196 | 4/4 |
| 1994 | TM | 0.4987 | 2/2 |
| 1995 | TM | 0.5053 | 3/3 |
| 1997 | TM | 0.4249 | 4/4 |
| 1998 | TM | 0.5621 | 1/1 |
| 1999 | TM | 0.5542 | 1/1 |
| 2000 | ETM+ | 0.5736 | 1/1 |
| 2001 | ETM+ | 0.5354 | 2/2 |
| 2002 | ETM+ | 0.4395 | 1/1 |
| 2003 | ETM+ | 0.3710 | 2/2 |
| 2004 | ETM+ | 0.3624 | 2/2 |
| 2005 | ETM+ | 0.4556 | 4/4 |
| 2006 | ETM+ | 0.4709 | 2/2 |
| 2007 | ETM+ | 0.4621 | 5/5 |
| 2008 | ETM+ | 0.6046 | 5/5 |
| 2009 | ETM+ | 0.4754 | 3/3 |
| 2010 | ETM+ | 0.5418 | 4/4 |
| 2011 | TM | 0.5562 | 3/3 |
| 2012 | ETM+ | 0.4484 | 4/4 |
| 2013 | ETM+ | 0.4992 | 2/2 |
| 2014 | OLI | 0.4703 | 5/5 |
| 2015 | OLI | 0.5917 | 4/4 |
| 2016 | OLI | 0.5675 | 5/5 |
| 2017 | OLI | 0.4443 | 6/6 |
| 2018 | OLI | 0.5468 | 6/6 |
| 2019 | OLI | 0.5112 | 6/6 |
| 2020 | OLI | 0.5814 | 5/5 |
| 2021 | OLI | 0.5503 | 5/5 |
| 2022 | OLI-2 | 0.5962 | 5/5 |
| 2023 | OLI-2 | 0.6178 | 4/4 |
| 2024 | OLI-2 | 0.5764 | 5/5 |
| 2025 | OLI-2 | 0.5842 | 5/5 |
| 2026 | OLI-2 | 0.5635 | 6/6 |

Missing (real archive gaps, zero scenes for this bbox, confirmed live, not gap-filled): **1984, 1985, 1986, 1987, 1989, 1996.**

**2011 used TM instead of ETM+, investigated rather than assumed a fallback-logic bug**: `LANDSAT_SENSOR_PRIORITY` tries ETM+ before TM, so TM winning in 2011 looked at first like it might indicate the priority order was being ignored. Live-checked directly: `Collection.size` for `LANDSAT/LE07/C02/T1_L2` filtered to this bbox and Jan-Mar 2011 returned **0** — Landsat 7 genuinely had zero scenes here that specific window (plausibly reduced tasking priority as Landsat 8's 2013 launch approached), confirmed the fallback to TM is correct behavior, not a bug.

**Sanity check 1 — recent-year plausibility against Sentinel-2's 0.407 bbox mean**: every Landsat year from 1988-2026 falls between 0.34 and 0.62 — all comfortably inside "same broad territory," none at the flagged 0.1 or 0.8 danger zone. Landsat's JFM dry-season values run somewhat higher than Sentinel-2's July-August (monsoon-transition) 0.407, which is directionally expected (dry-thorn/dry-deciduous vegetation is typically greener right after the Oct-Dec monsoon than mid-transition into the next one), not a discrepancy.

**Sanity check 2 — no year outside [0, 0.9]**: confirmed, min 0.3417 (1992), max 0.6178 (2023), both well inside bounds. No defect.

**Sanity check 3 — 2013 and 2003 step-change check, done explicitly rather than assumed either way**:
- **2003 (SLC-off onset)**: pre-2003 (TM+ETM+, 1988-2002, 13 years) mean **0.4690**; post-2003 ETM+-only years (2003-2013, 10 years) mean **0.4691** — statistically indistinguishable, matching the per-pixel valid-image-count finding above that SLC-off does not bias this bbox's annual composite. **No discontinuity at 2003.**
- **2013→2014 (OLI onset)**: single-year delta is small (0.4992 → 0.4703, -0.029) and smaller than several other adjacent-year swings in the series (e.g. 2016→2017 is -0.123) — not a sharp one-year cliff. **However**, looking at full sensor-family means rather than just the boundary year (37-year series, including 2026): TM years (11 years, 1988-2011) average **0.4641**, ETM+ years (13 years, 2000-2013) average **0.4800**, OLI/OLI-2 years (13 years, 2014-2026) average **0.5540** — a real, systematic ~0.07-0.09 upward shift once OLI takes over (2014 onward), consistent with Landsat's own well-documented TM/ETM+-vs-OLI radiometric differences, not treated here as an ecological signal. **Verdict: no sharp one-year step at 2013 itself, but a genuine multi-year sensor-family-level shift from 2014 onward that any downstream analysis must treat as a sensor artifact, not real vegetation change** — exactly why `sensor` is stored per row rather than presenting this as one uniform instrument's series.

**Sanity check 4 — cross-check against Hansen's own 2021-2023 concentrated forest loss (36.9% of all-time loss, per this file's earlier Stage 6 section)**: 2018-2020 (pre-loss-window) NDVI mean **0.5465**; 2021-2023 (the loss window itself) NDVI mean **0.5881** — **NDVI is HIGHER during the loss window, not lower.** Per this task's explicit instruction, this real disagreement is reported plainly rather than explained away. Plausible (not confirmed) contributing factors, none independently verified this session: (a) Hansen's `loss` band captures ANY stand-replacing disturbance (fire, plantation harvest, natural disturbance — already noted in this file's forest-loss section), not necessarily a loss of the dry-thorn/scrub vegetation this bbox's own NDVI is otherwise dominated by, so a loss event in one land-cover type needn't move a bbox-wide mean measured in a different season; (b) the Jan-Mar compositing window is 2-5 months after each calendar year's loss events are typically detected (Hansen's loss year attribution isn't sub-annual), so JFM 2021's composite could partly precede within-2021 loss; (c) a genuinely wetter 2021-2023 JFM season could raise NDVI independent of forest condition — CHIRPS only has a rolling 30-day window in this project currently, not a historical archive, so this cannot be checked from data already on hand. **This stands as a real, unresolved disagreement between the two layers — the FIRMS-shaped cross-check hole already on record in this file gets a sibling here, not a substitute for one.**

**Cleanup**: the temporary `/api/debug/gee` route added to `src/index.js` for this task's live probing was fully removed — confirmed via `git diff --stat src/index.js` producing no output, i.e. net zero change to that file versus the pre-task commit.

## Stage 7 — Media and journalism

**Done for 3 of 5 candidate sources, with the other 2 explicitly deferred (not silently skipped) — see below.**

### What was built

New shared library `src/lib/rss.js` (mirrors `crawler.js`'s division of labor from Stage 4: `fetchRssFeed` owns fetch/robots/hash/dedupe/R2/parse, each stream supplies its own relevance filter and row-writing — same split as `fetchListingPage` + each Stage 4 stream's own PDF loop) with a small regex-based RSS `<item>` parser (no DOM/XML parser is available in a Worker; every feed here is well-formed WordPress/CMS RSS, so this is enough, same judgment call as Stage 5's `fuzzy-match.js`). New shared helper `src/lib/news-event.js` (find-or-create for `news_event`, deduping by `(reserve_id, url)` since a news item has no content hash the way a PDF does but its URL is a stable publisher-assigned identifier). Three streams: `mongabay-india`, `thehindu-tn`, `toi-coimbatore`.

**Per the master prompt's explicit Stage 7 rule**: never store full article text — every stream writes only headline/outlet/date/URL/a ≤25-word summary. The summary is derived from the feed's own `<description>` teaser (never a fetch of the actual article page), truncated to 25 words by `truncateToWords`, which strips HTML tags first — found live during testing that TOI's `<description>` is a pure `<a><img/></a>` thumbnail-link wrapper with no text content at all; stripped-then-empty correctly yields `null`, not raw markup.

### Sources investigated and explicitly ruled out before writing any code

A dedicated research pass (multiple live-verification agents, not memory) checked robots.txt and real feed availability for every source type the master prompt names, before writing any stream:

- **GDELT DOC 2.0 API**: keyless, no robots.txt restriction on `api.gdeltproject.org` at all — but every request from this environment's egress IP (re-tested directly, not just via the research agent) returns a durable `HTTP 429` ("limit requests to one every 5 seconds"), including retries spaced well past 5 seconds. This is the same shape as Stage 3's Overpass 406 finding — very likely a shared/exhausted dev-environment egress IP, not a real per-request bug — but unlike Overpass, GDELT's actual JSON response schema could not be captured at all from here, so no stream code was written against a guessed contract. `sources/gdelt.json` registers the source with an empty `sources` array and this reasoning; `src/streams/gdelt.js` does not exist yet. **Needs re-verification from the deployed Worker's real egress IP** before writing this stream — if it still 429s from Cloudflare's edge, that would be a genuine (different) finding worth revisiting.
- **Google News RSS**: technically returns 200 with real, well-formed RSS (confirmed live, including the Tamil-language `hl=ta&gl=IN&ceid=IN:ta` edition) — but ruled out entirely, not deferred: `news.google.com/robots.txt` disallows `/rss/search` for the general wildcard user-agent group (not just named AI bots) and *additionally* explicitly disallows `ClaudeBot`/`Claude-Web`/`Anthropic-ai`/`GPTBot`/etc. by name with no exception, and the feed's own channel metadata restricts use to "a personal feed reader for personal, non-commercial use." Unlike The Hindu's case below, there is no separate wildcard-group allowance here — this is an unambiguous, structurally different case from Overpass/Wikidata's Stage-3-style "disallow probably targets search engines, not API clients" reasoning, so it was not overridden. No stream code was written.
- **Dinamalar**: no discoverable RSS feed at any common path (`/feed`, `/rss`, `/rss.xml` all 301-redirect to the homepage) — deferred as blocked (no feed found), not a robots issue.
- **Dinamani**: no true RSS feed either, but does have a live `news_sitemap.xml` (Google News-style sitemap with a `news:` namespace, real title/URL/pubdate/keywords per article) — a genuine alternative to standard RSS, not yet implemented as a stream since it needs its own small parser (different tag shape than RSS `<item>`) and wasn't in scope for this pass. Flagged as a real, viable follow-up rather than a dead end.
- **Vikatan**: robots.txt explicitly disallows `ClaudeBot`/`Anthropic-ai`/`GPTBot`/PerplexityBot and ~15 other named AI/scraper bots site-wide, with no wildcard-group carve-out the way The Hindu has — ruled out on the same unambiguous grounds as Google News RSS, before even checking whether its `/stories.rss` feed technically works (it does, but the robots policy already settles this). No stream code was written.

### A real, structural robots.txt distinction found and relied on (The Hindu)

`thehindu.com/robots.txt` has two separate groups: a wildcard `User-agent: *` group (many specific path disallows, none covering `/feeder/`, and which explicitly lists `thehindu.com/feeder/default.rss` as a **Sitemap** entry — a deliberate machine-readability signal) and a completely separate named-bot group (`GPTBot`, `ClaudeBot`, `Claude-Web`, `Anthropic-ai`, `PerplexityBot`, `CCBot`, and others) with `Disallow: /` site-wide. This project's bot identifies via `buildUserAgent()` as `SathyamangalamRecordHarvestBot/0.1 (+mailto:...)` — a literal string that doesn't match any token in the named-bot group, so per robots.txt's own most-specific-group-wins matching rule (already implemented in `src/lib/robots.js` before this stage, unchanged), this bot correctly falls under the wildcard group, which allows `/feeder/`. This is a real, structural policy distinction the site operator wrote deliberately (bulk AI-training crawlers vs. everything else), not a loophole being exploited — `checkRobotsAllowed()` still runs on every fetch, so this reasoning is enforced in code and would immediately break the stream if The Hindu ever moved `/feeder/` under the wildcard group's own disallows. Documented in full in `sources/thehindu-tn.json`.

### Two real bugs found and fixed during live testing

1. **`source.kind` CHECK constraint violation.** All three new registries were initially written with `"kind": "rss"` — Stage 0's schema constrains `source.kind` to `('api', 'pdf', 'html', 'archive')` only (verified: no prior stage ever needed a 5th category). Every prior "GET a fixed, machine-readable endpoint" case (Wikidata SPARQL, Overpass, GBIF, etc.) was classified `'api'`, and an RSS feed fits that same shape (a structured, machine-consumable response) far better than `'html'` (reserved for a human-facing listing page that happens to contain links, per Stage 4's convention) — fixed to `'api'` in all three registries. Caught immediately by a real `D1_ERROR: CHECK constraint failed` on first live run, not a silent misclassification.
2. **Relevance-filter false positive.** `mongabay-india.js`'s first draft included the bare term `"tiger reserve"` — live testing surfaced a real match against a Kaziranga Tiger Reserve story (a different reserve entirely) on the very first run. Fixed by dropping every standalone generic term (`"tiger reserve"`, bare `"erode"`) from `mongabay-india.js`/`thehindu-tn.js`'s filters in favor of terms specific enough to this reserve/region that a false positive would be surprising. **Decision (Vishnu, 2026-08-22)**: `toi-coimbatore.js` deliberately keeps broader wildlife-conflict terms (`"tiger"`, `"elephant corridor"`) despite the same feed matching a real Gudalur-area (different reserve) tiger story live during testing — regional human-wildlife-conflict news is plausibly relevant to landscape-level conservation tracking, and `news_event` rows are raw material for human review, not auto-published fact, so recall was favored over precision specifically for this one stream.

### Live verification evidence (2026-08-22, local `wrangler dev`, real network calls)

| Stream | job_run status | rows_written | Notes |
|---|---|---|---|
| mongabay-india | `success` | 0 | Feed fetched live (real WordPress RSS, 200), correctly 0 matches after the relevance-filter fix removed the Kaziranga false positive |
| thehindu-tn | `success` | 0 | Feed fetched live (real RSS, pre-scoped to Tamil Nadu, 200), genuinely 0 reserve-relevant stories in the current feed |
| toi-coimbatore | `success` | 1 | Real, live, on-topic match ("Tiger kills one more coffee plantation worker near Gudalur") — `summary_25w` correctly `null` since TOI's description field is a pure image-link HTML wrapper with no text content, not a bug |

**Idempotent re-run confirmed**: re-running all three against the same calendar date correctly hit the source-level (not just news_event-level) dedup path — `rowsSkippedDuplicate: 1`, `rowsWritten: 0`, no re-fetch, verified via a queue-processing-time drop from ~4.2s to ~22ms on the repeat run.

**R2 raw-before-parse confirmed**: all three feeds' raw XML bytes saved to R2 before parsing, per rule #1.

**Provenance**: 1 `news_event` row (toi-coimbatore), 0 orphaned.

**Sensitive-species coarsening**: not applicable — Stage 7 writes only to `news_event`, never `occurrence`/coordinates.

### Blockers and follow-ups (for Vishnu)

1. **GDELT needs re-verification from the deployed Worker's real egress IP** — this dev environment's IP is durably 429'd; cannot confirm the real JSON response schema until re-tested from production or a different environment.
2. **Google News RSS and Vikatan are ruled out on robots.txt/terms grounds, not deferred pending a credential** — would need an explicit policy decision to revisit (e.g. contacting either publisher for permission), not just a technical fix.
3. **Dinamalar has no discoverable feed at all** — would need a different technique entirely (e.g. HTML scraping of a listing page, same shape as Stage 4) if this outlet's coverage is judged valuable enough to pursue.
4. **Dinamani's `news_sitemap.xml` is a viable, not-yet-built alternative** to standard RSS — real live data confirmed, just needs its own small parser for the `news:` namespace tag shape.

## Stage 8 — Management-plan extraction

**Done for 4 of 29 appendices (3 species-list checklists, fully; 1 maps appendix, 6/11 pages — the other 5 hit a real, documented library limitation, not a bug).** The manual-download blocker from the previous entry was resolved: Vishnu downloaded the 391-page management plan PDF from Academia.edu and placed it at `harvest-engine/TAMILNADU_FOREST_DEPARTMENT_MANAGEMENT_P.pdf`. Confirmed via `pdfinfo`: 391 pages, PDF 1.7, 15.86MB, title page text extracted live matches "Management Plan for Sathyamangalam Wildlife Sanctuary: (2010:2020)".

### How the manual-download step fits rule #1 (raw before parse)

Unlike every other stage, this one has no live HTTP fetch to save raw-before-parse. The PDF's sha256 (`8cc0c8ad44c18caa179ac627414180c2e73bb6c5de866882358c5145ec579989`) was computed locally, then uploaded once via `wrangler r2 object put --local` at the exact content-addressed key `src/lib/raw-storage.js`'s `saveRaw()` would have produced (`raw/sathyamangalam/management-plan/8c/<hash>`) — round-trip verified byte-identical via `shasum -a 256` before and after upload. The stream (`src/streams/management-plan.js`) reads this pre-existing R2 object rather than fetching anything; `sources/management-plan.json` documents the exact upload command and hash so this step is reproducible, not a one-off hack.

### What was built

New migration `migrations/0002_stage8_extracted_table.sql`: a new `extracted_table` table, since nothing existing fit — `document`/`legal_instrument` are whole-file provenance records, `historical_passage` is unstructured prose, `occurrence` requires lat/lon, `claim` is one field per row. Appendix row shapes vary (species checklists, maps) so row data is `row_json`, not a fixed column set. Every row is written `review_status='pending'` — the code has no path that ever sets a different value — per the master prompt's explicit instruction to flag every extracted table for human review rather than auto-publish.

New shared library `src/lib/extracted-table.js`: `insertExtractedTable` (plain insert helper, same shape as `historical-passage.js` — no find-or-create dedup by content, since a row's identity is its position within its appendix) and `parseNumberedListAppendix`, a text-pattern row splitter for the "N. <fields>" numbered-list convention shared by the mammal/avifauna/butterfly appendices.

New shared library `src/lib/vision-extract.js`: wraps a Workers AI vision-model call for one embedded page image. Uses `@cf/meta/llama-3.2-11b-vision-instruct` (confirmed a real, current model via `wrangler ai models schema` — not guessed).

New stream `src/streams/management-plan.js`: creates one `source` + `document` row for the whole plan (idempotent — checks for an existing `source` row by `r2_key` first), then per appendix in `sources/management-plan.json`'s `appendix_map`, runs either the text-pattern parser (species-list appendices) or the vision-model pass (maps appendix).

### A real, non-obvious infrastructure requirement: adding the `[ai]` binding

No `[ai]` binding existed in `wrangler.toml` — Stage 4 scoped out an OCR pipeline needing one and explicitly deferred the decision to Vishnu rather than adding it speculatively. This stage's own definition-of-done ("vision-model pass for scanned pages/maps") makes that decision for real now, so the binding was added: `[ai]\nbinding = "AI"\nremote = true`. The `remote = true` flag matters and was verified deliberately, not assumed — local `wrangler dev --local` reports `env.AI ... Mode: not supported` (confirmed live: a bare local run cannot execute AI bindings at all), and Cloudflare's own dev-time warning states "AI bindings always access remote resources." Running `wrangler dev` without `--local` (or `--remote`) and this one binding's own `remote = true` flag produces the correct mixed mode — confirmed live via the bindings table `wrangler dev` prints at startup: `env.DB/RAW_BUCKET/HARVEST_QUEUE ... local`, `env.AI ... remote`. This keeps D1/R2/Queues exactly as isolated and local as every prior stage's evidence was gathered against, while only the AI binding (which cannot run any other way) reaches Cloudflare's real inference service.

### A real one-time gate: Llama 3.2 Vision's license acceptance

The first live vision-model call failed with `5016: Prior to using this model, you must submit the prompt 'agree'` — a genuine Meta Community License + Acceptable Use Policy gate on this specific model, not a bug or a network issue (its error message is worded like one, which cost real debugging time — see below). Paused and asked Vishnu whether to submit that acceptance on this Cloudflare account's behalf, given it's a real legal acknowledgment (including a representation about EU domicile), not a purely technical step. **Decision (Vishnu, 2026-08-22): submit it now.** Done via a temporary debug route calling `env.AI.run(model, {prompt: "agree"})` — confirmed the top-level `prompt` field is what this specific gate checks, not a `messages`-wrapped equivalent (tried first, silently didn't count as acceptance) — received `5016: Thank you for agreeing to this model's terms. You may now use the model.` in response, then verified a real plain-text call succeeded immediately after. The debug route was removed from `src/index.js` immediately after use; it never reached a commit.

### A real bug found and fixed: `@jsquash/png`'s self-fetching Wasm init

Every vision-model call failed with `Network connection lost` even after the license was accepted — genuinely misleading, since the actual failure (confirmed by adding a temporary debug route that called `describeImageWithVisionModel` directly and let the real error surface, rather than my code's own catch-and-rewrap) was inside `@jsquash/png`'s `encode()` → `__wbg_init`, not the AI call at all. Root cause: `@jsquash/png`'s bundled `init()` defaults to `fetch(new URL('squoosh_png_bg.wasm', import.meta.url))` when no module is passed in — a self-fetch with no working network path inside a Worker. **Fixed** by importing the `.wasm` file directly (`import pngWasmModule from "@jsquash/png/codec/pkg/squoosh_png_bg.wasm"` — wrangler's esbuild-based bundler compiles this to a real `WebAssembly.Module` at build time, the standard documented way to ship Wasm in a Worker) and calling jsquash's own exported `init(pngWasmModule)` once before the first `encode()` call. Verified live immediately after: a real map image (1184×1582, RGB) correctly encoded to a 2.33MB PNG and round-tripped through a real vision-model call.

### Live verification evidence (2026-08-22/23, mixed local/remote `wrangler dev`, real remote Workers AI calls)

| Appendix | Method | job_run(s) | Rows | Notes |
|---|---|---|---|---|
| 3 (mammals) | text-pattern | included in runs 67/69/70/71 | 36 | 35/36 high-confidence; 1 row (Sl.No 21) has a page-break footer line bled into its text, correctly flagged `confidence=0.5` rather than silently accepted |
| 4 (avifauna) | text-pattern | included in runs 67/69/70/71 | 207 | 207/207 high-confidence — this appendix has no scientific-name column, confirmed live and handled without forcing a false split |
| 5 (butterflies) | text-pattern | included in runs 67/69/70/71 | 86 | 81/86 high-confidence; 5 flagged low-confidence for real, inspectable reasons (a non-binomial name, page-break text bleed ×2, an unusual capitalization, and one genuine "Unidentified -" entry) — not silently guessed |
| 28 (maps) | vision-model | run 67 (partial, 0/6 — license+Wasm bugs, both fixed same session), run 69 (partial, 3/6 — pages 354-356), run 70 (partial, 3 more — pages 357-359, completing all 6), run 71 (idempotent re-run, 0 new) | 6 | Pages 349-353 (5 of 11 candidate map pages) never produce a row — see limitation below. Pages 354-359 each independently verified: the model's free-text description matches that exact page's real table-of-contents entry (water quality → land capability → land irrigability → soil productivity → crops grown → soils, in the same order the TOC lists them) |

**A real transient failure correctly handled, not hidden**: run 69's job_run recorded `3040: Capacity temporarily exceeded, please try again` for page 357 and `3040: Unknown error` for pages 358-359 — Workers AI's own shared inference capacity, the same class of finding as Stage 2's Semantic Scholar/CORE rate limits. **A real dedup bug found and fixed during this**: the stream's first version deduped the maps appendix the same coarse way as species-list appendices (skip the whole appendix if it has any rows at all) — this would have permanently stopped retrying pages 357-359 after run 69's partial success, treating "3 of 6 done" as "this appendix is finished" forever. Fixed by adding page-level dedup specifically to `extractMapAppendix` (checks `DISTINCT page_from` for that appendix, not just "does any row exist") before run 70, which correctly picked up exactly the 3 missing pages and left 354-356 untouched.

**Idempotency proof (run 71, full clean re-run)**: `rows_written=0`, `rows_skipped_duplicate=9` (3 species-list appendices + 6 already-done map pages, all correctly skipped), `errors_json` containing only the 5 already-known JPX failures — a real, clean idempotent re-run, not a lucky first pass.

**R2 raw-before-parse confirmed two ways**: (1) the source PDF itself, round-trip byte-verified before this stage's code ever ran (see above); (2) the 6 derived map-page PNGs the vision model actually saw, saved to R2 before being reported as input to the model (`raw/sathyamangalam/management-plan/derived/appendix-28-page-<N>-img-0.png`) — spot-checked by downloading one back out (`wrangler r2 object get --local`) and confirming it's a real, valid 1184×1582 RGBA PNG via `file`, not a corrupt or partial write.

**Provenance**: 335 `extracted_table` rows (36+207+86+6), 1 `document` row, 0 orphaned — every row's `document_id`/`source_id` resolves correctly, spot-checked via join.

**Sensitive-species coarsening**: not applicable — Stage 8 writes only to `extracted_table`, never `occurrence`/coordinates. (The mammal/bird/butterfly checklists are text lists with no coordinates attached — a future stage could geocode them against `place`, but that's out of this stage's scope.)

### A known, real limitation — not a silent gap

**5 of the 11 candidate map pages (349-353) never produce a row, on any run.** `unpdf`'s bundled PDF.js build cannot decode these specific embedded images — every attempt logs `JpxError: OpenJPEG failed to initialize` (confirmed live, reproduced identically on every one of 4 separate runs this session). These appear to be JPEG2000-encoded images, a format `unpdf`'s Workers-compatible build doesn't support decoding. This is the same class of finding as Stage 4's OCR-pipeline scoping (`@napi-rs/canvas` being a native dependency Workers can't load) — a real library gap, not something to route around by guessing at the image content. **Action needed from Vishnu, if these 5 pages' content (Appendix 28's Administrative Range Map, Satellite image, Forest type map, Density map, and Administrative Beat map — the first 5 entries in that appendix's own table of contents) is judged valuable enough to pursue**: either find a Workers-compatible JPEG2000 decoder (none identified in this session's research), or extract just those 5 pages as standalone image files (e.g. via a desktop PDF tool) and feed them through the same vision-model pipeline as a one-off, similar in spirit to this stage's own manual-upload pattern for the source PDF itself.

### Appendices not yet attempted

Only 4 of the plan's 29 appendices were scoped for this pass (3 species-list checklists as the clearest "species lists" example the master prompt names, plus the maps appendix for the vision-model requirement) — not a silent gap, but a deliberate first slice. The remaining 25 (water sources, avifauna's Appendix 2, village/beat inventories, control forms 1-18, the budget statement, staff/vehicle/building lists, etc. — see the table of contents captured live in this session, PDF pages 15-21) are real, viable candidates for a follow-up pass using the same two extraction paths already built here. `sources/management-plan.json`'s `appendix_map` is where a future increment adds them — each new entry just needs its real PDF page range (found the same way this session's 4 were: scanning extracted text for `APPENDIX - N` markers) and, for species-list-shaped ones, a `columns` array.

### Blockers and follow-ups (for Vishnu)

1. **5 map pages (349-353) need a Workers-compatible JPEG2000 decoder or a manual per-page image extraction** — see limitation above.
2. **25 of 29 appendices are still unattempted** — a real, scoped-down first pass, not a hidden gap. See above for how to extend `appendix_map`.
3. **Every one of the 335 rows written is `review_status='pending'`** — per the master prompt's explicit rule for this stage, nothing here has been treated as verified fact. A human review pass (updating `review_status` to `confirmed`/`rejected`) is a distinct, not-yet-built step — this stage only produces the raw material for that review, same posture as Stage 5's un-annotated `historical_passage` rows.
