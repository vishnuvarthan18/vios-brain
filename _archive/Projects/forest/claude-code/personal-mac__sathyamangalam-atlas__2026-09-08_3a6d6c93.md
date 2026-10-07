**Vishnu** (2026-09-08T16:38): Do a per-stream coverage audit of the harvest engine before any fixes ship.
For each of the ~28 source streams in harvest-engine/src/streams/ (or
sources/*.json), report:

1. Status: healthy / blocked on missing API key / blocked by policy
   (robots.txt, quota, terms of service) / silently broken (runs, reports
   success, but produces wrong or zero real output) / never verified.
2. For "silently broken": show the actual evidence — e.g. compare what
   the source's real API response contains against what the parser
   extracts, field by field, the way the BHL FullTitle/Title mismatch
   was found.
3. Last real data written (not last "success" report) — same method used
   to catch the dedup freeze: compare row timestamps to report history,
   don't trust the status field alone.
4. What it would take to unblock (which API key, which decision, which
   code fix) if not healthy.

Do NOT fix anything, do NOT deploy anything, do NOT touch main or dev.
Read-only investigation. Output a table, one row per stream, plus a
one-line summary: how many of the 28 are actually healthy right now.

Write the full report to
sathyamangalam/harvest-engine-stream-audit-2026-09-09.md.

**Claude** (2026-09-08T16:38): _[tool: Agent]_

**Claude** (2026-09-08T16:38): I've launched a read-only audit agent to inspect all harvest-engine streams — checking code, source configs, env keys, policy blockers, and cross-verifying "success" claims against actual data evidence. It's working in the background and will write the report to `sathyamangalam/harvest-engine-stream-audit-2026-09-09.md`. I'll let you know when it's done.

**Vishnu** (2026-09-08T16:45): <task-notification>
<task-id>a38f7b76909a8b18a</task-id>
<tool-use-id>toolu_01B3rFUi7dn6BaeqwfFM5a7N</tool-use-id>
<output-file>/private/tmp/claude-501/-Users-vishnuvarthanv-sathyamangalam-atlas/3a6d6c93-99f0-4685-8ad0-574b80997165/tasks/a38f7b76909a8b18a.output</output-file>
<status>completed</status>
<summary>Agent "Harvest engine stream coverage audit" finished</summary>
<note>A task-notification fires each time this agent stops with no live background children of its own. The user can send it another message and resume it, so the same task-id may notify more than once.</note>
<result>The audit is complete and written to `/Users/vishnuvarthanv/sathyamangalam-atlas/sathyamangalam/harvest-engine-stream-audit-2026-09-09.md`.

## Summary

I audited all 29 registered streams (`harvest-engine/src/streams/index.js`) plus 2 unregistered one-off diagnostic scripts, cross-checking source code, `sources/*.json`, and — critically — the **live production D1 database** via read-only `wrangler d1 execute --remote` queries (this session was already Cloudflare-authenticated) and `wrangler secret list` (names only).

**Key findings:**

1. **The brief's premise about missing API keys was stale.** `wrangler.toml`'s comments claim BHL/WDPA/DATA_GOV_IN keys are "NOT yet obtained," but live `wrangler secret list` shows only `WDPA_API_KEY` is actually missing — BHL, DATA_GOV_IN, EBIRD, FIRMS, GEE, and IUCN keys are all set in production.

2. **11 of 29 streams are silently broken** — reporting "success" while `rows_written = 0` on every single run in their entire history (or frozen after one early write): gbif, core, bhl, wikidata, lgd, ntca, wii, historical-text, mongabay-india, toi-coimbatore, thehindu-tn. Most striking: `lgd`'s own source config documents 129 confirmed-matching upstream records, yet the database has never received one.

3. **5 streams are honestly blocked** (visible `failed` status, not masked): openalex (HTTP 429), shodhganga and forests-tn (origin site's robots.txt itself unreachable, 522), wdpa (missing key), management-plan (missing prerequisite PDF in R2).

4. **13 of 29 streams are genuinely healthy**, several verified with real recent writes (unpaywall, gee, firms wrote data today; census/bhuvan are single-document sources working exactly as designed).

5. Test suite (`harvest-engine/test/*.test.js`) doesn't cover the write-vs-dedup interaction where all 11 silent failures occur — passing tests would not have caught any of this.</result>
<usage><subagent_tokens>99540</subagent_tokens><tool_uses>31</tool_uses><duration_ms>436219</duration_ms></usage>
</task-notification>

**Claude** (2026-09-08T16:45): The audit's done and written to `sathyamangalam/harvest-engine-stream-audit-2026-09-09.md`.

**Headline: 13 of 29 streams are actually healthy.**

Key findings:
- **11 streams are silently broken** — reporting "success" while `rows_written = 0` for their entire run history: gbif, core, bhl, wikidata, lgd, ntca, wii, historical-text, mongabay-india, toi-coimbatore, thehindu-tn. Notably `lgd`'s own source config lists 129 confirmed upstream matches, yet the DB has never received one.
- **5 streams are honestly blocked** (visible failures, not masked): openalex (429 rate limit), shodhganga & forests-tn (source site itself down/522), wdpa (missing API key), management-plan (missing prerequisite PDF).
- **The brief's premise about missing keys was stale** — `wrangler.toml` comments claim BHL/WDPA/DATA_GOV_IN keys aren't obtained, but live `wrangler secret list` shows only `WDPA_API_KEY` is actually missing.
- 13 streams verified healthy, several with real writes as recent as today (unpaywall, gee, firms).
- The existing test suite doesn't cover the dedup/write-freeze interaction where all 11 silent failures live — passing tests wouldn't have caught this.

Full per-stream table with evidence, last-real-write timestamps, and unblock steps is in the report file.