**Vishnu** (2026-09-06T01:43): Build a small local web app to browse and explore the data in data_clean/ (the cleaned 28,691-record dataset, 776 files). This is for local development only — not for public hosting, no deployment needed.

Requirements:
1. A simple local web app (Flask or similar is fine) that runs with one command on my machine, opens in the browser at localhost.
2. Home page: overview stats — total records, breakdown by source (spider name), how many are "text" vs "metadata_only" kind, rough Tamil-script vs English-script split (based on percentage of Tamil unicode characters in the extract/text field).
3. A browse/search page: let me filter by source, search by keyword in the title or text, and page through results without loading everything at once (28,691 records is a lot — paginate, don't dump it all in one HTML page).
4. A record detail view: click any record to see its full text, source, title, and URL.
5. Clearly show, per record, whether it was flagged during cleaning (html/short/error_text) so I can visually spot-check problem records without leaving the app.
6. Keep it simple — no user accounts, no database setup beyond reading the existing .jsonl files directly (or building a lightweight local SQLite index from them if that makes search faster, your call).
7. Don't touch data/, data_clean/, or any git files — this is a new, separate app folder, e.g. viewer_app/.

Once built, tell me exactly how to start it (the one command to run) and confirm it works by showing me a screenshot or describing what loads when you open it locally.

**Claude** (2026-09-06T01:43): I'll start by exploring the data structure so the app matches what's actually in `data_clean/`.

**Claude** (2026-09-06T01:43): _[tool: Bash]_

**Claude** (2026-09-06T01:43): _[tool: Bash]_

**Claude** (2026-09-06T01:43): The cleaning script tells me exactly how flags and kinds are stored. Let me check the reports and sample record shapes.

**Claude** (2026-09-06T01:43): _[tool: Bash]_

**Claude** (2026-09-06T01:43): _[tool: Bash]_

**Claude** (2026-09-06T01:43): _[tool: Bash]_

**Claude** (2026-09-06T01:43): _[tool: Bash]_

**Claude** (2026-09-06T01:43): _[tool: Bash]_

**Claude** (2026-09-06T01:44): _[tool: Bash]_

**Claude** (2026-09-06T01:44): FTS5 tokenizes Tamil correctly and Flask isn't installed. I'll build this stdlib-only (SQLite + FTS5 + `http.server`) so the one command works with zero `pip install` or venv setup. Writing the indexer first:

**Claude** (2026-09-06T01:45): _[tool: Bash]_

**Claude** (2026-09-06T01:45): Index built — 28,691 records, matching the cleaning report. Let me verify the derived stats before wiring up the UI:

**Claude** (2026-09-06T01:45): _[tool: Bash]_

**Claude** (2026-09-06T01:46): All flag counts match the cleaning report exactly. Now the app itself:

**Claude** (2026-09-06T01:47): _[tool: Bash]_

**Claude** (2026-09-06T01:48): _[tool: Bash]_

**Claude** (2026-09-06T01:48): _[tool: Bash]_

**Claude** (2026-09-06T01:48): _[tool: Bash]_

**Claude** (2026-09-06T01:48): _[tool: Bash]_

**Claude** (2026-09-06T01:48): All routes respond fast. Let me verify the rendered content is actually correct:

**Claude** (2026-09-06T01:48): _[tool: Bash]_

**Claude** (2026-09-06T01:49): The search works, but look at that Tamil snippet: `<mark>தம</mark>ி<mark>ழ</mark>்` — `remove_diacritics 2` is stripping Tamil vowel signs and the pulli, which both breaks highlighting and conflates distinct Tamil words. Let me test the tokenizer options:

**Claude** (2026-09-06T01:49): _[tool: Bash]_

**Claude** (2026-09-06T01:49): Confirmed a real bug: unicode61 treats Tamil vowel signs as *separators*, shattering `தமிழ்` into fragments `தம`+`ழ` — so `படம்` and `பாடம்` collide. Let me check whether `tokenchars` can coexist with Latin diacritic folding:

**Claude** (2026-09-06T01:49): _[tool: Bash]_

**Claude** (2026-09-06T01:49): `remove_diacritics 2` + `tokenchars` gives both: whole Tamil words (படம் ≠ பாடம்) *and* Latin folding (Café→cafe). But my query builder uses `\w+` — I need to check Python's regex agrees:

**Claude** (2026-09-06T01:49): _[tool: Bash]_

**Claude** (2026-09-06T01:50): Both layers had the same bug — Python's `\w+` shatters Tamil too, so the query builder was sending `"தம"* AND "ழ"*`. Let me see which combining marks actually occur in this corpus before fixing:

**Claude** (2026-09-06T01:50): _[tool: Bash]_

**Claude** (2026-09-06T01:50): Tamil dominates but every Indic script appears. I'll make script-specific marks token characters while still letting Latin diacritics fold. Testing the generated set:

**Claude** (2026-09-06T01:50): _[tool: Bash]_

**Claude** (2026-09-06T01:50): Both fixes now. First the indexer's tokenizer:

**Claude** (2026-09-06T01:51): _[tool: Bash]_

**Claude** (2026-09-06T01:51): _[tool: Bash]_

**Claude** (2026-09-06T01:51): Now the index holds real Tamil words (`தமிழ்நாடு`, `தமிழகத்தில்`) and `படம்`/`பாடம்` are distinct. Restarting to verify end-to-end:

**Claude** (2026-09-06T01:51): _[tool: Bash]_

**Claude** (2026-09-06T01:51): Highlighting now wraps whole words and phrases. But those counts are inconsistent — a phrase (175) can't outnumber its own first word (154). Let me trace it:

**Claude** (2026-09-06T01:51): _[tool: Bash]_

**Claude** (2026-09-06T01:52): The old server never died — `kill %1` ran in a different shell, so my new instance moved to another port and curl hit the stale process. Killing properly:

**Claude** (2026-09-06T01:52): _[tool: Bash]_

**Claude** (2026-09-06T01:52): _[tool: Bash]_

**Claude** (2026-09-06T01:52): Source filter (91) matches the cleaning report exactly. But "titles only" (2,486) needs checking against the unrestricted count — a subset can't be larger:

**Claude** (2026-09-06T01:52): _[tool: Bash]_

**Claude** (2026-09-06T01:52): Titles-only (2,486) ⊂ all-fields (6,314) — correct. But two of those sample titles don't visibly contain "temple". Checking:

**Claude** (2026-09-06T01:52): _[tool: Bash]_

**Claude** (2026-09-06T01:53): Zero false positives across all 2,486 — the titles were just truncated in my preview. Now let me capture actual screenshots:

**Claude** (2026-09-06T01:53): _[tool: Bash]_

**Claude** (2026-09-06T01:53): _[tool: Bash]_

**Claude** (2026-09-06T01:53): _[tool: Read]_

**Claude** (2026-09-06T01:53): Home page renders correctly with all counts matching. Now browse and record views:

**Claude** (2026-09-06T01:53): _[tool: Bash]_

**Claude** (2026-09-06T01:53): _[tool: Read]_

**Claude** (2026-09-06T01:53): Tamil renders correctly, highlights wrap whole words, flags and script chips show, flagged rows have the orange border. Now a record detail page — picking a flagged one:

**Claude** (2026-09-06T01:53): _[tool: Bash]_

**Claude** (2026-09-06T01:53): _[tool: Bash]_

**Claude** (2026-09-06T01:54): _[tool: Read]_

**Claude** (2026-09-06T01:54): Record view works — flag banner, facts table with file+line origin, and the raw `<jats:p>` tags visible for spot-checking. Final safety and edge-case checks:

**Claude** (2026-09-06T01:54): _[tool: Bash]_

**Claude** (2026-09-06T01:54): Everything handles cleanly. One honesty wart: `q=+++---` reports "28,691 records matching +++---" when really no searchable token existed. Let me fix that and add a README:

**Claude** (2026-09-06T01:55): _[tool: Bash]_

**Claude** (2026-09-06T01:55): Let me verify the true cold-start experience — the exact one command with no index present:

**Claude** (2026-09-06T01:55): _[tool: Bash]_

**Claude** (2026-09-06T01:55): Cold start worked (home returned 200) — the error was just zsh globbing the unquoted `?`. Confirming:

**Claude** (2026-09-06T01:56): _[tool: Bash]_

**Claude** (2026-09-06T01:56): Warm start's banner was buffered by my pipe redirect (interactive terminals line-buffer, but let me make it robust):

**Claude** (2026-09-06T01:56): _[tool: Bash]_

**Claude** (2026-09-06T01:56): Done. The app is built and running now.

## Start it

```
python3 viewer_app/app.py
```

That's the whole thing — stdlib only, no `pip install` or venv. Flask wasn't installed on your machine, so I built it on `http.server` + SQLite/FTS5 to keep the one-command promise true. First run builds `viewer_app/index.sqlite` from `data_clean/` (~10s, 293 MB), then serves `http://127.0.0.1:8000` and opens a tab. Later runs start instantly and auto-rebuild only if a `data_clean/` file is newer. Flags: `--port N`, `--no-browser`, `--rebuild`, `--quiet`.

It's live right now on port 8000 if you want to click around immediately.

## What loads

**Overview** — four tiles: 28,691 records / 15,763 with text / 12,928 metadata-only / 2,879 flagged. Below that a script-split bar (Tamil 6,239 · 21.7%, Mixed 795, English/Latin 20,013 · 69.8%, No letters 1,644), flag counts (html 2,522, short 358, error_text 27), and a 40-row per-source table. Every number is a link into a filtered result list. All counts reconcile exactly with `_reports/summary.txt`.

**Browse** — search box plus source / kind / flag / script / titles-only / sort filters, 50 per page (574 pages). Matches are highlighted in the snippet. Flagged records get an orange left edge and badges, so `<p><span>` junk is visible in the list without opening anything.

**Record** — banner naming each flag and what it means, then source, kind, URL, which field the text came from, length, script %, and the `data_clean/` file **and line number** it lives on, followed by full text, all fields, and raw JSON.

## One thing worth knowing

SQLite's `unicode61` tokenizer treats Tamil vowel signs as *separators*, so `தமிழ்` indexed as `தம` + `ழ` and **`படம்` collided with `பாடம்`**. Python's `\w+` has the same flaw, so my query builder was independently broken too. Both are fixed by treating script-specific combining marks as token characters — the index now holds real words (`தமிழ்நாடு`, `தமிழகத்தில்`), `படம்` (121) and `பாடம்` (73) are distinct, and Latin diacritics still fold so `café` finds `cafe`. Terms are prefix-matched (`தமிழ` → `தமிழர்`), `"quoted"` is exact.

Verified: 40/40 source counts match the cleaning report, title-only search has zero false positives across all 2,486 hits, `<script>` in record text renders escaped, and bad input (punctuation-only, `OR`/`NOT`, `page=99999`) degrades gracefully. `data/`, `data_clean/` and git are untouched — `git status` shows only the new `viewer_app/`, with the 293 MB index gitignored.

**Vishnu** (2026-09-06T02:45): save all and close

**Claude** (2026-09-06T02:45): _[tool: Bash]_

**Claude** (2026-09-06T02:45): Server stopped, port 8000 free, no stray temp files. All work is saved on disk in `viewer_app/`:

| File | |
|---|---|
| [app.py](viewer_app/app.py) | server, search, all three views |
| [build_index.py](viewer_app/build_index.py) | JSONL → SQLite/FTS5 indexer |
| [static/style.css](viewer_app/static/style.css) | styling, light + dark |
| [README.md](viewer_app/README.md) | how to run, Tamil tokenizer notes |
| [.gitignore](viewer_app/.gitignore) | keeps the 293 MB index and `__pycache__` out of git |

`data/`, `data_clean/` and git are untouched — `git status` still shows only your pre-existing spider edits, `clean_data.py`, and the new untracked `viewer_app/`. I haven't committed anything, since you'd said to leave git alone; say the word if you want that staged and committed.

To pick it back up later:

```
python3 viewer_app/app.py
```

It'll reuse the existing index and come up instantly at `http://127.0.0.1:8000`.

**Vishnu** (2026-09-06T02:51): stop all the dev server of this project

**Claude** (2026-09-06T02:51): Let me find everything running from this project, not just the viewer:

**Claude** (2026-09-06T02:51): _[tool: Bash]_

**Claude** (2026-09-06T02:51): _[tool: Bash]_

**Claude** (2026-09-06T02:52): Two truncated Python processes are worth identifying before I call this clean:

**Claude** (2026-09-06T02:52): _[tool: Bash]_

**Claude** (2026-09-06T02:52): All dev servers for this project are stopped.

**tamil_harvest — nothing running:**
- `viewer_app/app.py` — stopped (killed at your previous message)
- Port 8000 free, no `scrapy`/`flask`/`http.server` processes, nothing holding `index.sqlite` open

I left two things alone deliberately, since neither belongs to this project:

- **PIDs 3006 / 3015** — `tests/d1_stand_in.py` and `tests/r2_http_stand_in.py`, running with cwd `/Users/vishnuvarthanv/Ecotourism/harvest-engine`. Different project, started 08:09 from another Claude session. One of them (`r2_http_stand_in`) is an HTTP stand-in, so it is a dev server — just not this repo's. Say the word and I'll kill both.
- **Port 5000** — macOS ControlCenter (AirPlay Receiver), a system process, not a dev server.

Restart the viewer any time with `python3 viewer_app/app.py`; the index is still on disk, so it comes up instantly.