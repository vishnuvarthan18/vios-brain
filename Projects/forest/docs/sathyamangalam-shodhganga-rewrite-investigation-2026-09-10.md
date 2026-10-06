# Shodhganga rewrite — investigation, 10 Sep 2026

Live-probed from Vishnu's Mac (real Terminal, not sandboxed — the device-bridge shell has its own narrower network allowlist and couldn't reach this host at all).

## Findings, endpoint by endpoint

| Endpoint | Result |
|---|---|
| `/` (site root) | Times out (~20s), no response. Confirms prior finding: intermittently unreachable at the connection level. |
| `/oai/request?verb=Identify` | HTTP 404, DSpace "Error: Document Not Found" — dead, structural. Confirmed again. |
| `/robots.txt` | 200, but content is a stale DSpace default template (`Sitemap: http://localhost:8080/...`) — not customized, gives no real guidance. |
| `/open-search/description.xml` | **200**, real OpenSearch descriptor, confirms `ShortName: DSpace`, query param is `searchTerms`. |
| `/simple-search?query=...` | **200** (when it responds — see below), real HTML search-results page from DSpace 5.3, confirms the search UI works server-side. |
| `/feed/rss_2.0/site` | **200** (site-wide RSS feed exists) but this is the whole repository's recent-items feed, not query-filtered. |
| `/open-search/search?query=...&format=rss` | Times out. Endpoint likely not wired to the OpenSearch servlet on this deployment. |
| `/open-search/discover?...` and bare `/open-search?...` | HTTP 400 — wrong path/params, not a working route. |
| `/feed/rss_2.0/search?query=...` | HTTP 404 — not a real route. |
| `/json/discovery/search/objects?query=...` | Times out — the JSON discovery API (used by the site's own JS for autocomplete) may exist but is unreliably reachable. |

## Conclusion

This DSpace 5.3 instance is **real and partially alive**, but flaky at the connection level — even working endpoints (`simple-search`) time out roughly half the time in these probes. There is no working RSS/Atom/JSON search API on this deployment (the ones that should exist per DSpace defaults 400 or time out) — only the human-facing `simple-search` HTML page reliably returns real content when the host responds at all.

**The only viable path forward: scrape `simple-search?query=<term>` HTML**, the same pattern already used successfully for `ntca`/`wii` (regex-based extraction from real captured HTML, see `extractPdfLinks`), NOT a rewrite to a clean structured API as originally hoped. This is a downgrade in elegance from what the current OAI-PMH code assumes, but it's what's actually there.

## Implementation plan (not yet built)

1. **Fetch layer**: reuse `fetchWithRetry` (already retries 5xx per this week's fix) but this host's failure mode is a *connection timeout*, not a 5xx — confirm `fetchWithRetry`/`fetch-timeout.js` already retries on timeout, not just bad status codes. If not, that's a prerequisite fix.
2. **One request per query term** (not a paginated crawl — OAI's resumptionToken pagination goes away entirely; `simple-search` is single-page-of-results per call, DSpace paginates via `&start=N`).
3. **HTML extraction**: parse each result-page `<div class="artifact-description">` (or whatever the real captured HTML shows — need one real full-page capture saved as a fixture first, same as `ntca.raw.html`/`wii.raw.html`) for: thesis title, handle/permalink (the DSpace item URL, e.g. `/handle/10603/xxxxx`), and byline/date if present in the search snippet.
4. **Dedup**: by handle URL, same pattern as `findExistingSource`.
5. **Save raw HTML per query to R2 first** (rule #1, nothing parsed before it's saved raw), matching every other stream.
6. **Tests**: capture one real `simple-search` HTML response (when the host cooperates) as a fixture, write the extraction test against it — same rigor as `ntca.raw.html`/`wii.raw.html` this week.
7. **Rate limiting / retry budget**: given the ~50% timeout rate observed, this stream needs a generous per-run retry allowance and should NOT be treated as broken if a single run comes up empty — that's expected host behavior, not a bug. Worth a comment in the stream file explaining this explicitly so it isn't re-investigated as a regression later.

## Not done here
No code was written. This session's device-bridge shell can't reach this host directly (narrower network allowlist than the actual Mac's Terminal), so implementation and fixture-capture need to happen from a session that can — either a future coding session run directly on the Mac terminal, or a dev agent invocation that has a working path to capture the raw HTML for a fixture.

## Open question for Vishnu
Given the confirmed ~50% timeout rate on this host even when reachable, is it worth the implementation effort for what will likely be a low-yield, unreliable stream? Real theses about Sathyamangalam on Shodhganga are probably rare anyway (it's a PhD-thesis repository, not general literature) — worth weighing against openalex/Crossref which may cover more relevant ground more reliably.
