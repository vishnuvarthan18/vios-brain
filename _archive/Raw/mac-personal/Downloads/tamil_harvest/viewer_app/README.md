# data_clean/ viewer

A local, read-only browser for the cleaned corpus produced by `clean_data.py`.
Standard library only — no `pip install`, no venv, no server config.

## Run it

    python3 viewer_app/app.py

First run builds `viewer_app/index.sqlite` from `data_clean/*.jsonl` (~10s, ~290 MB),
then serves <http://127.0.0.1:8000> and opens a browser tab. Later runs reuse the
index and start instantly. Ctrl-C to stop.

Options: `--port N`, `--no-browser`, `--quiet`, `--rebuild` (force a fresh index).
The index is rebuilt automatically whenever a file in `data_clean/` is newer than it.

## Pages

- **Overview** — totals, text vs metadata-only, the Tamil/Latin script split, cleaning
  flag counts, and a per-source table. Every number links into a filtered result list.
- **Browse** — full-text search over title, text and metadata, with filters for source,
  kind, flag state and script; 50 results a page.
- **Record** — full text, source, title, URL, which field the text came from, the
  `data_clean/` file and line it sits on, and the raw JSON.

Records flagged by the cleaner (`html`, `short`, `error_text`) keep an orange edge in
the result list and get a banner on their own page explaining the flag, so problem
records are visible without leaving the app.

## Searching Tamil

SQLite's `unicode61` tokenizer treats combining marks as separators, which would split
`தமிழ்` into `தம` + `ழ` and make `படம்` and `பாடம்` collide. The index declares
script-specific marks as token characters instead, so Tamil words stay whole and
distinct. Generic Latin diacritics are still folded, so `café` finds `cafe`.

Every term is a prefix search — `தமிழ`  finds `தமிழர்`, `தமிழ்நாடு`. Use
`"double quotes"` for an exact phrase.

## Scope

Reads `data_clean/` and writes only inside `viewer_app/`. Binds to 127.0.0.1 —
local development only, not for hosting. `index.sqlite` is gitignored; it is
derived data and rebuilds in seconds.
