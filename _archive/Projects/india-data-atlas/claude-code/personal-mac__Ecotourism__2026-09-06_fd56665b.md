**Vishnu** (2026-09-06T01:37): In this repo, run a local dry run of the seed-list harvest branch against local D1/R2 stand-ins (no real Cloudflare needed) to verify scripts/normalize_seed_list.py actually produces reserve rows before we touch real infra.

Steps:

cd harvest-engine && source .venv/bin/activate (create the venv and pip install -r requirements.txt if it doesn't exist yet).
Start tests/d1_stand_in.py on http://127.0.0.1:8787 and tests/r2_http_stand_in.py on http://127.0.0.1:9000 (see tests/README.md for exact invocation — they're plain Python HTTP servers, run each in the background).
Export env vars pointing at the stand-ins: D1_HTTP_API_BASE_URL=http://127.0.0.1:8787, D1_API_TOKEN=dummy, R2_ENDPOINT_URL=http://127.0.0.1:9000, R2_ACCESS_KEY_ID=dummy, R2_SECRET_ACCESS_KEY=dummy, R2_BUCKET=dry-run.
Run python scripts/apply_migrations.py to load the schema into the local D1 stand-in.
Run the real spiders against real live sources: scrapy crawl seed_list -a source=ntca-tiger-reserves and scrapy crawl seed_list -a source=wii-gazette-notifications (this fetches real pages over the network — needs internet access, which this repo's own tests/run_branch_test.py pattern relies on for every other branch).
Run python scripts/normalize_seed_list.py and capture its printed summary (reserves_created, reserves_updated, reserves_matched_cross_source, records_skipped_unmapped_type, records_skipped_unparseable).
Query the local D1 stand-in (SELECT COUNT(*), type FROM reserve GROUP BY type and spot-check a handful of rows, especially centroid_lat/centroid_lon — note that this script doesn't set those, only apply_migrations's schema does; NTCA has no lat/lon columns, so they'll be NULL until a later branch backfills them, and export_geojson.py's WHERE clause will drop rows with NULL centroid — flag this explicitly).
Run python scripts/export_geojson.py against the same local D1 and report how many features exports/reserves.geojson actually contains (expect 0 or very few, given point 7 — this is the thing to confirm, not assume).
Report back: reserve counts by type, any skipped/unmatched records logged (especially "no unambiguous type suffix" ones — list a few example names), and the final GeoJSON feature count with an explanation of why it's zero/nonzero.

**Vishnu** (2026-09-06T01:37): <command-message>workflow-authoring</command-message>
<command-name>workflow-authoring</command-name>
<skill-format>true</skill-format>

**Vishnu** (2026-09-06T01:37): # Workflow authoring reference

A workflow structures work across many agents — to be comprehensive (decompose and cover in parallel), to be confident (independent perspectives and adversarial checks before committing), or to take on scale one context can't hold (migrations, audits, broad sweeps). The script is where you encode that structure: what fans out, what verifies, what synthesizes.

When you do call it, the right move is often **hybrid**: scout inline first (list the files, find the channels, scope the diff) to discover the work-list, then call Workflow to pipeline over it. You don't need to know the shape before the *task* — only before the *orchestration step*.

Common single-phase workflows you can chain across turns:
- **Understand** — parallel readers over relevant subsystems → structured map
- **Design** — judge panel of N independent approaches → scored synthesis
- **Review** — dimensions → find → adversarially verify (the review-changes example)
- **Research** — multi-modal sweep → deep-read → synthesize
- **Migrate** — discover sites → transform each (worktree isolation) → verify

For larger work, run several in sequence — read each result before deciding the next phase. You stay in the loop; each workflow is one well-scoped fan-out.

**Ultracode.** When a system-reminder confirms ultracode is on, that opt-in is standing: author and run a workflow for every substantive task by default. The goal is the most exhaustive, correct answer you can produce — token cost is not a constraint. For multi-phase work (understand → design → implement → review), that often means several workflows in sequence — one per phase — so you stay in the loop between them. The quality patterns below (adversarial verify, multi-modal sweep, completeness critic, loop-until-dry) are the tools; pick what fits the task. Lean toward orchestrating with workflows and adversarially verifying your findings — unless the work is trivial or already verified. Solo only on conversational turns or trivial mechanical edits. When a reminder says ultracode is off, revert to the opt-in rule in the Workflow tool description.

Pass the script inline via `script` — do not Write it to a file first. Every invocation automatically persists its script to a file under the session directory and returns the path in the tool result. To iterate on a workflow, edit that file with Write/Edit and re-invoke Workflow with `{scriptPath: "<path>"}` instead of resending the full script.

Every script must begin with `export const meta = {...}`:
  export const meta = {
    name: 'find-flaky-tests',
    description: 'Find flaky tests and propose fixes',   // one-line, shown in permission dialog
    phases: [                                            // one entry per phase() call
      { title: 'Scan', detail: 'grep test logs for retries' },
      { title: 'Fix', detail: 'one agent per flaky test' },
    ],
  }
  // script body starts here — use agent()/parallel()/pipeline()/phase()/log()
  phase('Scan')
  const flaky = await agent('grep CI logs for retry markers', {schema: FLAKY_SCHEMA})
  ...

The `meta` object must be a PURE LITERAL — no variables, function calls, spreads, or template interpolation. Required fields: `name`, `description`. Optional: `whenToUse` (shown in the workflow list), `phases`. Use the SAME phase titles in meta.phases as in phase() calls — titles are matched exactly; a phase() call with no matching meta entry just gets its own progress group. Add `model` to a phase entry when that phase uses a specific model override.

Script body hooks:
- agent(prompt: string, opts?: {label?: string, phase?: string, schema?: object, model?: string, effort?: string, isolation?: 'worktree', agentType?: string}): Promise<any> — spawn a subagent. Without schema, returns its final text as a string. With schema (a JSON Schema), the subagent is forced to call a StructuredOutput tool and agent() returns the validated object — no parsing needed. Returns null if the user skips the agent mid-run or the subagent dies on a terminal API error after retries (filter with .filter(Boolean)). opts.label overrides the display label. opts.phase explicitly assigns this agent to a progress group (use this inside pipeline()/parallel() stages to avoid races on the global phase() state — same phase string → same group box). opts.model overrides the model for this agent call. Default to omitting it — the agent inherits the main-loop model (the resolved session model), which is almost always correct. Only set it when you're highly confident a different tier fits the task; when unsure, omit. opts.effort overrides the reasoning effort for this agent call ('low' | 'medium' | 'high' | 'xhigh' | 'max') — omit to inherit the session effort; use 'low' for cheap mechanical stages and higher tiers only for the hardest verify/judge stages. opts.isolation: 'worktree' runs the agent in a fresh git worktree — EXPENSIVE (~200-500ms setup + disk per agent), use ONLY when agents mutate files in parallel and would otherwise conflict; the worktree is auto-removed if unchanged. opts.agentType uses a custom subagent type (e.g. 'general-purpose', 'code-reviewer') instead of the default workflow subagent — resolved from the same registry as the Agent tool; composes with schema (the custom agent's system prompt gets a StructuredOutput instruction appended).
- pipeline(items, stage1, stage2, ...): Promise<any[]> — run each item through all stages independently, NO barrier between stages. Item A can be in stage 3 while item B is still in stage 1. This is the DEFAULT for multi-stage work. Wall-clock = slowest single-item chain, not sum-of-slowest-per-stage. Every stage callback receives (prevResult, originalItem, index) — use originalItem/index in later stages to label work without threading context through stage 1's return value. A stage that throws drops that item to `null` and skips its remaining stages.
- parallel(thunks: Array<() => Promise<any>>): Promise<any[]> — run tasks concurrently. This is a BARRIER: awaits all thunks before returning. A thunk that throws (or whose agent errors) resolves to `null` in the result array — the call itself never rejects, so `.filter(Boolean)` before using the results. Use ONLY when you genuinely need all results together.
- log(message: string): void — emit a progress message to the user (shown as a narrator line above the progress tree)
- phase(title: string): void — start a new phase; subsequent agent() calls are grouped under this title in the progress display
- args: any — the value passed as Workflow's `args` input, verbatim (undefined if not provided). Pass arrays/objects as actual JSON values in the tool call, NOT as a JSON-encoded string — `args: ["a.ts", "b.ts"]`, not `args: "[\"a.ts\", ...]"` (a stringified list reaches the script as one string, so `args.filter`/`args.map` throw). Use this to parameterize named workflows — e.g. pass a research question, target path, or config object directly instead of via a side-channel file.
- budget: {total: number|null, spent(): number, remaining(): number} — the turn's token target from the user's "+500k"-style directive. `budget.total` is null if no target was set. `budget.spent()` returns output tokens spent this turn across the main loop and all workflows — the pool is shared, not per-workflow. `budget.remaining()` returns `max(0, total - spent())`, or `Infinity` if no target. The target is a HARD ceiling, not advisory: once `spent()` reaches `total`, further `agent()` calls throw. Use for dynamic loops: `while (budget.total && budget.remaining() > 50_000) { ... }`, or static scaling: `const FLEET = budget.total ? Math.floor(budget.total / 100_000) : 5`.
- workflow(nameOrRef: string | {scriptPath: string}, args?: any): Promise<any> — run another workflow inline as a sub-step and return whatever it returns. Pass a name to invoke a saved workflow (same registry as {name: "..."}), or {scriptPath} to run a script file you Wrote earlier. The child shares this run's concurrency cap, agent counter, abort signal, and token budget — its agents appear under a "▸ name" group in /workflows and its tokens count toward budget.spent(). The args param becomes the child's `args` global. Nesting is one level only: workflow() inside a child throws. Throws on unknown name / unreadable scriptPath / child syntax error; catch to handle gracefully.

Subagents are told their final text IS the return value (not a human-facing message), so they return raw data. For structured output, use the schema option — validation happens at the tool-call layer so the model retries on mismatch.

Workflow agents can reach all session-connected MCP tools via ToolSearch — schemas load on demand per agent. Caveat: interactively-authenticated MCP servers (e.g. claude.ai) may be absent in headless/cron runs.

Scripts are plain JavaScript, NOT TypeScript — type annotations (`: string[]`), interfaces, and generics fail to parse. The script body runs in an async context — use await directly. Standard JS built-ins (JSON, Math, Array, etc.) are available — EXCEPT `Date.now()`/`Math.random()`/argless `new Date()`, which throw (they would break resume); pass timestamps in via `args`, stamp results after the workflow returns, and for randomness vary the agent prompt/label by index. No filesystem or Node.js API access.

DEFAULT TO pipeline(). Only reach for a barrier (parallel between stages) when you genuinely need ALL prior-stage results together.

A barrier is correct ONLY when stage N needs cross-item context from all of stage N-1:
- Dedup/merge across the full result set before expensive downstream work
- Early-exit if the total count is zero ("0 bugs found → skip verification entirely")
- Stage N's prompt references "the other findings" for comparison

A barrier is NOT justified by:
- "I need to flatten/map/filter first" — do it inside a pipeline stage: pipeline(items, stageA, r => transform([r]).flat(), stageB)
- "The stages are conceptually separate" — that's what pipeline() models. Separate stages ≠ synchronized stages.
- "It's cleaner code" — barrier latency is real. If 5 finders run and the slowest takes 3× the fastest, a barrier wastes 2/3 of the fast finders' idle time.

Smell test: if you wrote
  const a = await parallel(...)
  const b = transform(a)        // flatten, map, filter — no cross-item dependency
  const c = await parallel(b.map(...))
that middle transform doesn't need the barrier. Rewrite as a pipeline with the transform inside a stage. When in doubt: pipeline.

Concurrent agent() calls are capped at min(16, available CPUs - 2) per workflow — excess calls queue and run as slots free up. You can still pass 100 items to parallel()/pipeline() and they all complete; only ~10 run at any moment. Total agent count across a workflow's lifetime is capped at 1000 — a runaway-loop backstop set far above any real workflow. A single parallel()/pipeline() call accepts at most 4096 items; passing more is an explicit error, not a silent truncation.

When a barrier IS correct — dedup across all findings before expensive verification:
  const all = await parallel(DIMENSIONS.map(d => () => agent(d.prompt, {schema: FINDINGS_SCHEMA})))
  const deduped = dedupeByFileAndLine(all.filter(Boolean).flatMap(r => r.findings))  // <-- genuinely needs ALL at once
  const verified = await parallel(deduped.map(f => () => agent(verifyPrompt(f), {schema: VERDICT_SCHEMA})))

Loop-until-count pattern — accumulate to a target:
  const bugs = []
  while (bugs.length < 10) {
    const result = await agent("Find bugs in this codebase.", {schema: BUGS_SCHEMA})
    bugs.push(...result.bugs)
    log(`${bugs.length}/10 found`)
  }

Loop-until-budget pattern — scale depth to the user's "+500k" directive. Guard on budget.total: with no target set, remaining() is Infinity and the loop would run straight to the 1000-agent cap.
  const bugs = []
  while (budget.total && budget.remaining() > 50_000) {
    const result = await agent("Find bugs in this codebase.", {schema: BUGS_SCHEMA})
    bugs.push(...result.bugs)
    log(`${bugs.length} found, ${Math.round(budget.remaining()/1000)}k remaining`)
  }

Composing patterns — exhaustive review (find → dedup vs seen → diverse-lens panel → loop-until-dry):
  const seen = new Set(), confirmed = []
  let dry = 0
  while (dry < 2) {                                              // loop-until-dry
    const found = (await parallel(FINDERS.map(f => () =>          // barrier: collect all finders this round
      agent(f.prompt, {phase: 'Find', schema: BUGS})))).filter(Boolean).flatMap(r => r.bugs)
    const fresh = found.filter(b => !seen.has(key(b)))           // dedup vs ALL seen — plain code, not an agent
    if (!fresh.length) { dry++; continue }
    dry = 0; fresh.forEach(b => seen.add(key(b)))
    const judged = await parallel(fresh.map(b => () =>           // every fresh bug judged concurrently...
      parallel(['correctness','security','repro'].map(lens => () =>   // ...each by 3 distinct lenses
        agent(`Judge "${b.desc}" via the ${lens} lens — real?`, {phase: 'Verify', schema: VERDICT})))
        .then(vs => ({ b, real: vs.filter(Boolean).filter(v => v.real).length >= 2 }))))
    confirmed.push(...judged.filter(v => v.real).map(v => v.b))
  }
  return confirmed
  // dedup vs `seen`, NOT `confirmed` — else judge-rejected findings reappear every round and it never converges.

Quality patterns — common shapes; pick by task and compose freely:
- Adversarial verify: spawn N independent skeptics per finding, each prompted to REFUTE. Kill if ≥majority refute. Prevents plausible-but-wrong findings from surviving.
    const votes = await parallel(Array.from({length: 3}, () => () =>
      agent(`Try to refute: ${claim}. Default to refuted=true if uncertain.`, {schema: VERDICT})))
    const survives = votes.filter(Boolean).filter(v => !v.refuted).length >= 2
- Perspective-diverse verify: when a finding can fail in more than one way, give each verifier a distinct lens (correctness, security, perf, does-it-reproduce) instead of N identical refuters — diversity catches failure modes redundancy can't.
- Judge panel: generate N independent attempts from different angles (e.g. MVP-first, risk-first, user-first), score with parallel judges, synthesize from the winner while grafting the best ideas from runners-up. Beats one-attempt-iterated when the solution space is wide.
- Loop-until-dry: for unknown-size discovery (bugs, issues, edge cases), keep spawning finders until K consecutive rounds return nothing new. Simple counters (while count < N) miss the tail.
- Multi-modal sweep: parallel agents each searching a different way (by-container, by-content, by-entity, by-time). Each is blind to what the others surface; useful when one search angle won't find everything.
- Completeness critic: a final agent that asks "what's missing — modality not run, claim unverified, source unread?" What it finds becomes the next round of work.
- No silent caps: if a workflow bounds coverage (top-N, no-retry, sampling), `log()` what was dropped — silent truncation reads as "covered everything" when it didn't.

Scale to what the user asked for. "find any bugs" → a few finders, single-vote verify. "thoroughly audit this" or "be comprehensive" → larger finder pool, 3–5 vote adversarial pass, synthesis stage. When unsure, lean toward thoroughness for research/review/audit requests and toward brevity for quick checks.

These patterns aren't exhaustive — compose novel harnesses when the task calls for it (tournament brackets, self-repair loops, staged escalation, whatever fits).

Use this tool for multi-step orchestration where control flow should be deterministic (loops, conditionals, fan-out) rather than model-driven.

## Resume

The tool result includes a runId. To resume after a pause, kill, or script edit, relaunch with Workflow({scriptPath, resumeFromRunId}) — the longest unchanged prefix of agent() calls returns cached results instantly; the first edited/new call and everything after it runs live. Same script + same args → 100% cache hit. Before diagnosing why a completed workflow returned an empty or unexpected result, Read <transcriptDir>/journal.jsonl — it records each agent's actual return value; do not assume cached results are non-empty. Date.now()/Math.random()/new Date() are unavailable in scripts (they would break this) — stamp results after the workflow returns, or pass timestamps via args. Fallback when no journal is available: Read agent-<id>.jsonl files in the transcript directory and hand-author a continuation script.

**Claude** (2026-09-06T01:37): I'll start by exploring the repo structure to understand what actually exists before running anything.

**Claude** (2026-09-06T01:37): _[tool: Bash]_

**Claude** (2026-09-06T01:37): _[tool: Bash]_

**Claude** (2026-09-06T01:37): _[tool: Bash]_

**Claude** (2026-09-06T01:37): _[tool: Bash]_

**Claude** (2026-09-06T01:38): _[tool: Bash]_

**Claude** (2026-09-06T01:38): _[tool: Bash]_

**Claude** (2026-09-06T01:38): _[tool: Bash]_

**Claude** (2026-09-06T01:38): _[tool: Bash]_

**Claude** (2026-09-06T01:38): _[tool: Bash]_

**Claude** (2026-09-06T01:38): _[tool: Bash]_

**Claude** (2026-09-06T01:38): _[tool: Bash]_

**Claude** (2026-09-06T01:38): _[tool: Bash]_

**Claude** (2026-09-06T01:38): _[tool: Bash]_

**Claude** (2026-09-06T01:38): _[tool: Bash]_

**Claude** (2026-09-06T01:39): _[tool: Bash]_

**Claude** (2026-09-06T01:39): _[tool: Bash]_

**Claude** (2026-09-06T01:39): _[tool: Bash]_

**Claude** (2026-09-06T01:39): _[tool: Bash]_

**Claude** (2026-09-06T01:39): _[tool: Bash]_

**Claude** (2026-09-06T01:39): _[tool: Bash]_

**Claude** (2026-09-06T01:39): Dependencies installed (scrapy 2.18.0). Now starting the stand-in servers.

**Claude** (2026-09-06T01:39): _[tool: Bash]_

**Claude** (2026-09-06T01:39): _[tool: Bash]_

**Claude** (2026-09-06T01:39): _[tool: Bash]_

**Claude** (2026-09-06T01:39): _[tool: Bash]_

**Claude** (2026-09-06T01:40): Both stand-ins are up. Launching a static-analysis workflow in the background while I run the pipeline.

**Claude** (2026-09-06T01:41): _[tool: Workflow]_

**Claude** (2026-09-06T01:41): Now applying migrations to the local D1 stand-in.

**Claude** (2026-09-06T01:41): _[tool: Bash]_

**Claude** (2026-09-06T01:41): _[tool: Bash]_

**Claude** (2026-09-06T01:41): Schema is in. Now running the real NTCA spider against the live source.

**Claude** (2026-09-06T01:41): _[tool: Bash]_

**Claude** (2026-09-06T01:41): NTCA fetched cleanly (HTTP 200, robots.txt allowed). Now the WII crawl — 35 state pages, rate-limited.

**Claude** (2026-09-06T01:41): _[tool: Bash]_

**Claude** (2026-09-06T01:41): _[tool: Bash]_

**Claude** (2026-09-06T01:42): _[tool: Bash]_

**Claude** (2026-09-06T01:42): _[tool: Bash]_

**Claude** (2026-09-06T01:43): _[tool: Bash]_

**Claude** (2026-09-06T01:43): All 35 WII state pages fetched (HTTP 200), zero parser failures. Now the normalizer.

**Claude** (2026-09-06T01:43): _[tool: Bash]_

**Vishnu** (2026-09-06T01:43): [Request interrupted by user]

**Vishnu** (2026-09-06T01:44): go ahed

**Claude** (2026-09-06T01:44): _[tool: Bash]_

**Claude** (2026-09-06T01:44): _[tool: Bash]_

**Claude** (2026-09-06T01:45): _[tool: Bash]_

**Claude** (2026-09-06T01:45): _[tool: Bash]_

**Claude** (2026-09-06T01:45): _[tool: Bash]_

**Claude** (2026-09-06T01:45): That query over-matched — every WII record writes a medium-confidence fact, not just cross-source ones. Let me reconstruct the 36 matches exactly by replaying the normalizer's logic offline.

**Claude** (2026-09-06T01:45): _[tool: Bash]_

**Claude** (2026-09-06T01:46): _[tool: Bash]_

**Claude** (2026-09-06T01:46): _[tool: Bash]_

**Claude** (2026-09-06T01:47): _[tool: Bash]_

**Claude** (2026-09-06T01:47): _[tool: Bash]_

**Claude** (2026-09-06T01:47): _[tool: Bash]_

**Claude** (2026-09-06T01:47): _[tool: Bash]_

**Claude** (2026-09-06T01:47): _[tool: Bash]_

**Claude** (2026-09-06T01:48): _[tool: Bash]_

**Claude** (2026-09-06T01:49): Dry run complete, end to end. Here's what happened.

## Setup

Created `harvest-engine/.venv` (Python 3.12.3, scrapy 2.18.0 — the `async def start()` API in the spider is correct for this version), started both stand-ins, applied all 5 migrations cleanly. Env pointed at `127.0.0.1:8787` / `127.0.0.1:9000`, bucket `dry-run`.

Both live crawls succeeded against the real sources — no fixtures:

| Spider | Pages | HTTP | Records parsed | Parser failures |
|---|---|---|---|---|
| `ntca-tiger-reserves` | 1 | 200 | 59 | 0 |
| `wii-gazette-notifications` | 35 | 200 ×35 | 586 | 0 |

(WII's `robots.txt` returned 403, which Scrapy treats as allow-all; NTCA's returned 200 and permitted the path. The `user_agent_override` for NTCA is doing its job — the page came back real, not blocked.)

## normalize_seed_list.py summary

```
records_seen:                    645     (59 NTCA + 586 WII)
records_skipped_unmapped_type:    16
records_skipped_unparseable:       0
reserves_created:                593
reserves_updated:                  0
reserves_matched_cross_source:    36
facts_written:                   729
```

**It works.** 593 reserve rows and 717 `reserve_fact` rows landed. The arithmetic closes exactly: 59 + (586 − 16 skipped − 36 matched) = 593.

## Reserve counts by type

| type | count |
|---|---|
| `wildlife_sanctuary` | 459 |
| `national_park` | 75 |
| `tiger_reserve` | 59 |
| `conservation_reserve` | **0** |
| `community_reserve` | **0** |
| `biosphere_reserve` | **0** |

NTCA rows are rich (`area_total/core/buffer_sq_km`, `established_year`); all 534 WII-created rows are identity-only — name/type/state and nothing else. `reserves_updated: 0` is expected on a fresh DB.

## GeoJSON: 0 features — confirmed, not assumed

```
wrote 0 reserve features to /Users/vishnuvarthanv/Ecotourism/exports/reserves.geojson
```

Your point 7 is exactly right, and the numbers back it: **593 of 593 rows have NULL `centroid_lat` and NULL `centroid_lon`** (also 593 NULL `boundary_geojson`, 593 NULL `bbox_*`). `export_geojson.py`'s `WHERE centroid_lat IS NOT NULL AND centroid_lon IS NOT NULL` therefore matches zero rows.

I ran a control to prove the WHERE clause is the sole cause rather than something upstream: temporarily set a centroid on `id=1` (Bandipur) → export wrote **1 feature** with correct `[lon, lat]` ordering and full properties; reverted to NULL → back to **0**. The export machinery is fine; it has nothing to draw.

Root cause is structural, and worth flagging beyond just this branch: **no production code in the repo ever writes `centroid_lat`/`centroid_lon`.** Grepping the whole tree, the only writes are [tests/fixtures.py:89](harvest-engine/tests/fixtures.py#L89) (test-only seeding) — every other hit is a read. The NTCA parser's record dict has no lat/lon key at all, and neither does WII's. Knock-on effect: migration 0003's bbox backfill is also a no-op here, so `run_api_spider.py` (GBIF/eBird/iNaturalist) would find zero queryable reserves too. The map frontend stays empty until a geocoding/boundary branch backfills centroids.

## The 16 skipped "no unambiguous type suffix" records

All from WII, all logged by name. They fall into four groups:

**Genuinely ambiguous `CR` — the documented, intended skip (7):**
`'Basur Amruth CR'` (Karnataka), `'Hornbill CR'` (Karnataka), `'Jayamangali Blackbuck CR'` (Karnataka), `'Sundhamata CR \xa0\xa0\xa0'` (Rajasthan), `'BhorkadaCR'` (Maharashtra), `'Kokkare Bellur ComReserve'` (Karnataka), `'Kadalundi Vallikkunnu Com R'` (Kerala)

**Known source-side HTML bug — cell contains a filename (2):**
`'AghansnhiniLTMCR.pdf'`, `'BanakapurCR.pdf'` (both Karnataka; called out in the source YAML already)

**Suffix present but not at end-of-string, so `$` never matches (2):**
`'Mollem NP (Bhagwan Mahavir)'` (Goa), `'Simbalbara WLS (Now NP)'` (Himachal Pradesh)

**Camel-case names with no space before the suffix — `\b` fails between two word characters (4):**
`'SanjayGandhiNP'` (Maharashtra), `'KaranjasoholWLS'` (Maharashtra), `'NaigaonPeacockWLS'` (Maharashtra), plus `'Chintamani Kar Bird Sanctuary'` (West Bengal — "Sanctuary" spelled out; only `wildlife sanctuary` is in `TYPE_SUFFIX_MAP`)

Only the first group is the deliberate never-guess behaviour. The last two groups (6 records) are unambiguous protected areas being dropped on a formatting technicality — Sanjay Gandhi NP is not a judgment call. `'Chandrataal'` (Himachal) rounds out the 16 with no suffix at all.

One cosmetic thing: the log line hardcodes `(see module docstring re: bare 'CR')` for every skip, so `'SanjayGandhiNP'` gets blamed on CR ambiguity it has nothing to do with. That will mislead whoever triages this list.

## Two things I'd flag before you touch real infra

**1. `reserves_matched_cross_source: 36` is overstated — 8 of those aren't cross-source.** I replayed the normalizer's logic offline against the stored R2 bodies (reproducing 586/534/36/16 exactly) to split them: **28 are the intended NTCA↔WII merges** and work beautifully — Bandipur carries NTCA's area/years/PDFs at `confidence='high'` *and* WII's gazette PDF at `medium` under a different `source_id`, exactly as the docstring promises.

The other **8 are WII-internal collisions**. `_find_reserve` keys on `(LOWER(name), state)` and ignores type, so a National Park and a co-named Wildlife Sanctuary in the same state — legally distinct protected areas — collapse into one row, and the second one's type is silently discarded:

```
'Gir WLS'                      → merged into Gir            [national_park]
'Sri Venkateswara WLS'         → merged into Sri Venkateswara
'Marine (Gulf of Kachchh) WLS' → merged into Marine (Gulf of Kachchh)
'Kalesar WLS' / 'Bhitarkanika WLS' / 'Simlipal WLS'
'Indira Gandhi (Annamalai) WLS' / 'Galathea Bay WLS'
```

**2. 12 gazette PDFs were silently overwritten.** `facts_written: 729` vs 717 actual rows — the counter counts writes, not rows. All 12 are cases where two WII rows from the *same* state page hit the same `(reserve_id, field_name, source_id)` key, so `_upsert_fact` takes its UPDATE branch and the second URL replaces the first. Gir's row now points at `.../GirWLS` — the Gir NP notification is gone. Same for Valmiki, Periyar, Sariska, Buxa (each an NTCA tiger reserve matched by both an NP and a WLS row), plus the 8 above. The `UNIQUE (reserve_id, field_name, source_id)` constraint is doing what it says; the problem is that one source *row* here is a whole state page containing many distinct PAs.

Neither is a blocker for landing reserve rows — the script does what it's supposed to. Both are data-fidelity bugs that get harder to unwind once real D1 has the rows.

Minor: NTCA now serves 59 data rows, not the 58 the YAML recorded on 2026-08-27 — Ratapani, Madhav (2025) and Guru Ghasidas–Tamor Pingla have been added upstream. Worth refreshing that comment.

## State of things

Repo is clean — `git status` is unchanged from where you started (`exports/*.geojson` and `.venv/` are both gitignored). Test artifacts are at `…/scratchpad/dryrun/` (`d1.sqlite3` + `r2/`).

Both stand-ins are still listening on 8787/9000 if you want to query further — `pkill -f 'stand_in.py'` when you're done.

I also kicked off a parallel static audit of the normalize chain as a cross-check; it hasn't reported back yet. I'll pass along anything it surfaces that the empirical run missed.

**Vishnu** (2026-09-06T01:52): In harvest-engine/scripts/normalize_seed_list.py, fix three bugs found during a local dry-run against real NTCA/WII data (593 reserves created, but with data-fidelity issues) before this touches production D1.

Bug 1 — type-suffix matching drops real reserves. TYPE_SUFFIX_MAP's regexes use \b...$ which fails on camelCase names (SanjayGandhiNP, KaranjasoholWLS, NaigaonPeacockWLS) since \b requires a word boundary between two word characters, and on suffixes that aren't at the end of the string (Mollem NP (Bhagwan Mahavir), Simbalbara WLS (Now NP)). Also missing: a case for the fully-spelled-out "Sanctuary" with no suffix (Chintamani Kar Bird Sanctuary). Fix the matching so these resolve to their correct type instead of being skipped as "unmapped" — likely needs case-insensitive suffix detection that tolerates no-space-before-suffix and suffixes followed by trailing parenthetical text, plus a sanctuary$ pattern. Keep the genuinely-ambiguous bare "CR" skip as-is — don't make that one guess.

Bug 2 — cross-source reserve matching ignores type, so a National Park and a same-named Wildlife Sanctuary in the same state (e.g. "Gir WLS" vs the NTCA "Gir" tiger reserve, "Sri Venkateswara WLS", "Bhitarkanika WLS", "Simlipal WLS", "Kalesar WLS", "Indira Gandhi (Annamalai) WLS", "Galathea Bay WLS", "Marine (Gulf of Kachchh) WLS") incorrectly collapse into one reserve row, silently discarding the second one's distinct identity. _find_reserve / _upsert_reserve's matching key is (LOWER(name), state) only — decide whether to key on (LOWER(name), state, type) instead (so a same-named NP and WLS become two distinct reserve rows, which is the legally correct outcome — they're different protected areas), and update the WII cross-reference logic in run_wii() accordingly, since it currently deliberately does the opposite (match by name+state alone, ignoring type, to find the NTCA tiger reserve to attach to). You'll need to preserve the intended cross-source merge (WII row matching an NTCA tiger reserve by bare name) while fixing the unintended WII-internal collision (two different WII rows of different types matching each other). One approach: only cross-reference-match against a reserve whose type is tiger_reserve; require exact type match otherwise.

Bug 3 — _upsert_fact overwrites instead of preserving both values when two different source records (e.g. two different WII rows on the same state page, one NP one WLS, that get merged into the same reserve by bug 2) hit the same (reserve_id, field_name, source_id) key — the second gazette_notification_pdf_url silently replaces the first, losing real PDF links (observed for Gir, Valmiki, Periyar, Sariska, Buxa). This will likely resolve itself once bug 2 stops merging distinct reserves, but double check: if two genuinely-the-same-reserve records still produce two different values for the same field_name+source_id, decide whether reserve_fact needs a per-record disambiguator in its key, or whether logging a warning on overwrite (rather than silently replacing) is sufficient here.

Bug 4 (minor) — the skip-log line in run_wii() hardcodes "needs manual review (see module docstring re: bare 'CR')" for every unmapped-type skip, which is wrong for the camelCase/mid-string cases from bug 1. Make the log message generic, or only reference "CR" when the actual unmatched text contains "CR".

After fixing, re-run the same local dry-run this repo's tests/README.md describes (tests/d1_stand_in.py + tests/r2_http_stand_in.py, fresh DB, real live NTCA/WII crawls, then normalize_seed_list.py) and report the new summary stats plus confirm: reserves_created should now be higher (the 6 previously-dropped camelCase/mid-string names should land), reserves_matched_cross_source should drop to ~28 (only real NTCA↔WII merges), and spot-check that Gir now has both its NP and WLS identity preserved as separate rows (or confirm your chosen fix), each with its own gazette PDF intact.

**Claude** (2026-09-06T01:54): I'll fix all four bugs. First, let me re-read the exact regions I need to change.

**Claude** (2026-09-06T01:54): _[tool: Read]_

**Claude** (2026-09-06T01:55): Let me check the real WII name data before designing the regexes, so I'm matching against what the sources actually emit.

**Claude** (2026-09-06T01:55): _[tool: Bash]_

**Claude** (2026-09-06T01:55): _[tool: Bash]_

**Claude** (2026-09-06T01:56): Now I have the full picture of the real data. Making the fixes.

**Claude** (2026-09-06T01:57): _[tool: Edit]_

**Claude** (2026-09-06T01:57): _[tool: Edit]_

**Claude** (2026-09-06T01:57): Now bug 2 — the type-aware matching key:

**Claude** (2026-09-06T01:57): _[tool: Edit]_

**Claude** (2026-09-06T01:57): _[tool: Edit]_

**Claude** (2026-09-06T01:58): _[tool: Edit]_

**Claude** (2026-09-06T01:58): Now the `run_wii()` cross-reference logic and the bug 4 log message:

**Claude** (2026-09-06T01:58): _[tool: Edit]_

**Claude** (2026-09-06T01:58): Now updating the module docstring so it describes the new matching rules:

**Claude** (2026-09-06T01:58): _[tool: Edit]_

**Claude** (2026-09-06T01:58): _[tool: Edit]_

**Claude** (2026-09-06T01:58): Before re-running the full pipeline, let me unit-test the new splitter against all 586 real WII names and diff it against the old behaviour.

**Claude** (2026-09-06T01:58): _[tool: Bash]_

**Claude** (2026-09-06T01:59): All 22 pass. Now diffing old vs new behaviour across all 586 real WII names to catch unintended changes:

**Claude** (2026-09-06T01:59): _[tool: Bash]_

**Claude** (2026-09-06T01:59): Zero regressions, exactly the 6 target names recovered. Let me review the full diff and syntax-check before re-running.

**Claude** (2026-09-06T01:59): _[tool: Bash]_

**Claude** (2026-09-06T01:59): Now the fresh dry run — new DB, new R2, real live crawls.

**Claude** (2026-09-06T01:59): _[tool: Bash]_

**Claude** (2026-09-06T01:59): Both old stand-ins stopped as intended (exit 144 = my `pkill`). Starting fresh ones:

**Claude** (2026-09-06T01:59): _[tool: Bash]_

**Claude** (2026-09-06T01:59): _[tool: Bash]_

**Claude** (2026-09-06T02:00): _[tool: Bash]_

**Claude** (2026-09-06T02:00): _[tool: Bash]_

**Claude** (2026-09-06T02:00): _[tool: Bash]_

**Claude** (2026-09-06T02:00): _[tool: ToolSearch]_

**Claude** (2026-09-06T02:00): _[tool: Bash]_

**Claude** (2026-09-06T02:01): _[tool: TaskStop]_

**Claude** (2026-09-06T02:01): _[tool: Bash]_

**Claude** (2026-09-06T02:01): Both crawls clean again (36 source rows, 0 failures). Running the fixed normalizer:

**Claude** (2026-09-06T02:01): _[tool: Bash]_

**Claude** (2026-09-06T02:01): All four targets hit. Verifying the outcomes:

**Claude** (2026-09-06T02:02): _[tool: Bash]_

**Claude** (2026-09-06T02:02): _[tool: Bash]_

**Claude** (2026-09-06T02:02): Let me investigate one anomaly — Simlipal's NP got a type-qualified slug, meaning something else already held `simlipal`:

**Claude** (2026-09-06T02:02): _[tool: Bash]_

**Claude** (2026-09-06T02:02): Found a real pre-existing issue while verifying. Let me quantify it:

**Claude** (2026-09-06T02:02): _[tool: Bash]_

**Claude** (2026-09-06T02:03): _[tool: Bash]_

**Claude** (2026-09-06T02:03): _[tool: Bash]_

**Claude** (2026-09-06T02:03): _[tool: Edit]_

**Claude** (2026-09-06T02:03): _[tool: Bash]_

**Claude** (2026-09-06T02:04): All four bugs fixed and verified against a fresh live re-run. Here's the outcome.

## Summary stats: before → after

| stat | before | after | |
|---|---|---|---|
| `records_seen` | 645 | 645 | unchanged (59 NTCA + 586 WII) |
| `records_skipped_unmapped_type` | 16 | **10** | −6, exactly the 6 targets |
| `records_skipped_unparseable` | 0 | 0 | |
| `reserves_created` | 593 | **607** | +14 |
| `reserves_updated` | 0 | 0 | |
| `reserves_matched_cross_source` | 36 | **28** | −8, only real NTCA↔WII merges |
| `facts_written` | 729 | 735 | |
| actual `reserve_fact` rows | 717 | **735** | writes now == rows: zero overwrites |

The +14 decomposes exactly: **+6** recovered by bug 1, **+8** un-collapsed by bug 2. By type: `wildlife_sanctuary` 459→471, `national_park` 75→77, `tiger_reserve` 59→59.

## Bug 1 — suffix matching

Replaced the hand-written `\b…$` regexes with two builders: `_abbrev_suffix()` for abbreviations and `_phrase_suffix()` for spelled-out types, plus a trailing-parenthetical lift in `split_wii_name_and_type`. All 6 previously-dropped names now land:

```
'Mollem NP (Bhagwan Mahavir)'   -> 'Mollem (Bhagwan Mahavir)'  national_park
'Simbalbara WLS (Now NP)'       -> 'Simbalbara (Now NP)'       wildlife_sanctuary
'SanjayGandhiNP'                -> 'SanjayGandhi'              national_park
'KaranjasoholWLS'               -> 'Karanjasohol'              wildlife_sanctuary
'NaigaonPeacockWLS'             -> 'NaigaonPeacock'            wildlife_sanctuary
'Chintamani Kar Bird Sanctuary' -> 'Chintamani Kar Bird'       wildlife_sanctuary
```

The camelCase branch only accepts an abbreviation written in **capitals** after a lowercase letter. That matters: a naive `np$` would mistype any name merely ending in those letters. I added regression guards for exactly that — `'Some Placenp'`, `'Chambr'`, `'Kanha Tiger Reserve'` all correctly stay unmatched.

Before running anything I diffed old vs new across all 586 real WII names: **6 newly typed, 0 regressions, 0 type changes, 0 bare-name changes.** The bare `CR` skip is untouched — the 10 remaining skips are the 4 bare `CR`s, `BhorkadaCR`, the 2 source-side `.pdf` filename cells, `ComReserve` / `Com R`, and `Chandrataal` (no suffix at all).

One case needed a judgment call. `'Simbalbara WLS (Now NP)'` has a parenthetical asserting a *different* type than its suffix. I kept the suffix's type (`wildlife_sanctuary`) and log a warning rather than acting on "(Now NP)" — deciding whether a reclassification took effect needs the gazette PDF, not a table cell. I also kept the parenthetical in the stored name rather than discarding it, so nothing the source wrote is lost. That does leave the name as `'Simbalbara (Now NP)'`, which is ugly; flagging it so you can decide.

## Bug 2 — type-aware identity

Reserve identity is now `(LOWER(name), state, type)`. `_find_reserve()` takes a type; the cross-source merge moved to a dedicated `_find_tiger_reserve()` restricted to `type='tiger_reserve'` — that restriction is the whole fix, keeping the intended NTCA↔WII merge while stopping two WII rows from matching each other.

All 8 collapsed pairs are now separate rows. **Gir, confirmed as you asked:**

```
id 147  gir                       Gir  national_park       .../gujrat/GirNP
id 152  gir-wildlife-sanctuary    Gir  wildlife_sanctuary  .../gujrat/GirWLS
```

Both identities preserved, each with its own gazette PDF intact. Same for Bhitarkanika, Galathea Bay, Indira Gandhi (Annamalai), Kalesar, Marine (Gulf of Kachchh), Simlipal, Sri Venkateswara.

I also extended `_unique_slug()` to try the type before the state, so the second row reads `gir-wildlife-sanctuary` rather than the meaningless `gir-gujarat`. And I guarded the `_upsert_reserve` UPDATE branch with `if fields:` — the WII path passes `{}`, and with type-aware matching it can now reach that branch, which would have emitted `UPDATE reserve SET  WHERE id = ?` and crashed.

## Bug 3 — it did *not* fully resolve itself

Worth being explicit, since you asked me to double check: 4 of the 12 overwrites survive bug 2's fix. Valmiki, Periyar, Sariska and Buxa each have **both** an NP and a WLS row on the same state page, and both legitimately cross-match the same NTCA tiger reserve — same `reserve_id`, same `field_name`, same `source_id`, two different PDFs.

I fixed it structurally rather than by warning: on a cross-source match the field name is qualified with the WII row's own type. Result for Valmiki:

```
gazette_notification_pdf_url                     -> ntca.gov.in/.../Valmiki.pdf   (high)
gazette_notification_pdf_url_national_park       -> v1.wii.gov.in/.../ValmikiNP   (medium)
gazette_notification_pdf_url_wildlife_sanctuary  -> v1.wii.gov.in/.../ValmikiWLS  (medium)
```

Three real facts, three rows, no schema change needed. That's more accurate than the old field name anyway — the WII PDF documents a constituent PA, not the tiger reserve's own notification. I added the warning too, as defence in depth: `_upsert_fact` now logs loudly when an overwrite would *change* a value (a same-value rewrite is normal idempotency and stays quiet). **The re-run produced zero such warnings**, and `facts_written` 735 == `reserve_fact` rows 735 confirms it.

One downstream note: anything reading `field_name = 'gazette_notification_pdf_url'` won't see the two qualified variants.

## Bug 4 — log message

Now picks the reason from the actual name. `Chandrataal`, `ComReserve`, `Com R` report `no recognized type suffix`; only names actually carrying `CR` get the conservation-vs-community explanation.

## Verification

- **Provenance**: 0 rows violating the `source_id`/`retrieved_at`/`confidence` rule.
- **Idempotency**: all 36 source rows marked normalized; a second `normalize_seed_list.py` run is a clean all-zeros no-op.
- **Crawls**: NTCA 1 page HTTP 200, WII 35 pages HTTP 200, 0 parser failures, both fresh.
- **GeoJSON**: still **0 features** — unchanged and expected. All 607 rows still have NULL centroids; that's the separate backfill branch, untouched by these fixes.

## One thing I found but did not fix

While verifying I noticed Simlipal's NP got a type-qualified slug, which meant something already held `simlipal`. It's the NTCA tiger reserve — filed under state **`'Orissa'`**, while WII's label is `'Odisha'`. NTCA's state names don't match WII's for 9 tiger reserves:

| NTCA writes | should be | TRs affected |
|---|---|---|
| `Chattisgarh` | Chhattisgarh | 3 |
| `Madhy Pradesh` | Madhya Pradesh | 3 |
| `Orissa` | Odisha | 1 |
| `Telangana` | — WII has no Telangana page at all | 2 |

The cross-source merge can never fire for those 9, and `reserve.state` is internally inconsistent (`Orissa` and `Odisha` both present), which will bite any per-state grouping in the frontend. This predates today's changes — it's why Simlipal showed up in the original run's "WII-internal collision" list. Fixing it needs a state-alias map, which is a data decision I'd rather you make than have me slip in under a bug-fix. Happy to do it as a follow-up.

Separately: the background static-audit workflow I mentioned last time died without reporting — its agents stopped writing partway and it never returned. Nothing was lost, since the direct measurements above cover the same ground more conclusively, but I didn't want to leave that dangling.

Stand-ins are still up on 8787/9000 against `…/scratchpad/dryrun2/` if you want to poke at the new data; `pkill -f stand_in.py` to stop them. `git status` is unchanged from where you started.

**Vishnu** (2026-09-06T02:08): In harvest-engine/scripts/normalize_seed_list.py, add a state-name canonicalization step before any reserve lookup/match/insert uses the state field. NTCA and WII disagree on spelling for some states — confirmed real cases: NTCA writes "Chattisgarh" (should be "Chhattisgarh", 3 tiger reserves affected), "Madhy Pradesh" (should be "Madhya Pradesh", 3 affected), "Orissa" (should be "Odisha", 1 affected — Simlipal). Add a STATE_ALIASES dict mapping known-bad NTCA spellings to WII's canonical spelling (WII's spellings match the standard official names, e.g. "Odisha", "Chhattisgarh"), and apply it to every state value immediately after extracting it from a parsed record, in both run_ntca() and run_wii(), before it's used in _find_reserve, _upsert_reserve, or stored. Do NOT invent a fix for Telangana (WII simply has no Telangana source page — that's a missing source, not a spelling mismatch, and the 2 Telangana tiger reserves will keep failing to cross-merge until a Telangana source is added separately). After the fix, re-run the same local dry-run against fresh D1/R2 stand-ins and confirm: Simlipal's WII "Orissa" and NTCA "Odisha" and now merge instead of colliding under two spellings, reserves_matched_cross_source increases by 1 (Simlipal) plus however many of the 6 Chhattisgarh/Madhya Pradesh tiger reserves also have a matching WII row on the correctly-spelled state page, and report the final state list in D1 to confirm no reserve row still has "Chattisgarh", "Madhy Pradesh", or "Orissa" as its stored state value.

**Claude** (2026-09-06T02:08): Let me first enumerate the exact NTCA spellings against WII's canonical labels, so the alias map covers what's actually there.

**Claude** (2026-09-06T02:08): _[tool: Bash]_

**Claude** (2026-09-06T02:08): NTCA is inconsistent with *itself* — it uses both spellings of all three states. Implementing:

**Claude** (2026-09-06T02:08): _[tool: Edit]_

**Claude** (2026-09-06T02:09): _[tool: Edit]_

**Claude** (2026-09-06T02:09): Now applying it at both extraction points:

**Claude** (2026-09-06T02:09): _[tool: Edit]_

**Claude** (2026-09-06T02:09): _[tool: Edit]_

**Claude** (2026-09-06T02:09): _[tool: Edit]_

**Claude** (2026-09-06T02:09): Let me unit-test the canonicalizer, then check no other code path touches `state` before canonicalization:

**Claude** (2026-09-06T02:09): _[tool: Bash]_

**Claude** (2026-09-06T02:09): Every lookup/insert is downstream of a `canonical_state()` call. Now the fresh dry run.

**Claude** (2026-09-06T02:09): _[tool: Bash]_

**Claude** (2026-09-06T02:09): _[tool: Bash]_

**Claude** (2026-09-06T02:09): _[tool: Bash]_

**Claude** (2026-09-06T02:10): _[tool: Bash]_

**Claude** (2026-09-06T02:10): _[tool: Bash]_

**Claude** (2026-09-06T02:11): _[tool: Bash]_

**Claude** (2026-09-06T02:11): +5 cross-source merges (28 → 33). Let me identify exactly which, and verify the state list.

**Claude** (2026-09-06T02:11): _[tool: Bash]_

**Claude** (2026-09-06T02:12): _[tool: Bash]_

**Claude** (2026-09-06T02:12): _[tool: Bash]_

**Claude** (2026-09-06T02:12): The counter says +5 but facts show +4 — one merge must have had no PDF link. Let me replay to find it.

**Claude** (2026-09-06T02:12): _[tool: Bash]_

**Claude** (2026-09-06T02:13): All 5 accounted for — `Bandhavgarh NP` matched but has no PDF link, which is why the fact-table diff showed only 4. Running the final verification battery:

**Claude** (2026-09-06T02:13): _[tool: Bash]_

**Claude** (2026-09-06T02:13): _[tool: Bash]_

**Claude** (2026-09-06T02:14): Done. State canonicalization added and verified against a fresh live re-run.

## The change

`STATE_ALIASES` maps the three confirmed bad spellings onto WII's canonical form, and `canonical_state()` applies it — collapsing whitespace, matching case-insensitively, and passing unknown states through unchanged rather than dropping or guessing at them:

```python
STATE_ALIASES = {
    "chattisgarh":   "Chhattisgarh",
    "madhy pradesh": "Madhya Pradesh",
    "orissa":        "Odisha",
}
```

Applied at the point of extraction in both methods, so every downstream use — `_find_reserve`, `_find_tiger_reserve`, `_upsert_reserve`, and the stored `reserve.state` — sees one spelling:

- [normalize_seed_list.py](harvest-engine/scripts/normalize_seed_list.py) `run_ntca()`: `name, state = record["name"], canonical_state(record["state"])`
- `run_wii()`: `state = canonical_state(record.get("state") or state_label)`

I grepped every `state` reference in both methods to confirm no lookup or insert sits upstream of those two calls. Telangana is deliberately absent, with a comment explaining why aliasing it would fabricate a merge.

One thing worth knowing that wasn't in the brief: **NTCA is inconsistent with itself**, not just with WII. Its live table carries *both* spellings of all three states — `Chattisgarh` (3) alongside `Chhattisgarh` (1), `Madhy Pradesh` (3) alongside `Madhya Pradesh` (6), `Orissa` (1) alongside `Odisha` (1). So this also heals a split *within* the NTCA data that would have shown up as one state appearing twice in any per-state grouping:

| state | tiger reserves, now under one spelling |
|---|---|
| Chhattisgarh | 4 (was 3 + 1) |
| Madhya Pradesh | 9 (was 3 + 6) |
| Odisha | 2 (was 1 + 1) |

## Re-run results

| stat | before | after |
|---|---|---|
| `records_seen` | 645 | 645 |
| `records_skipped_unmapped_type` | 10 | 10 |
| `records_skipped_unparseable` | 0 | 0 |
| `reserves_created` | 607 | **602** (−5) |
| `reserves_updated` | 0 | 0 |
| `reserves_matched_cross_source` | 28 | **33** (+5) |
| `facts_written` | 735 | 735 |

**The +5, enumerated by replaying the matcher with and without aliasing:**

```
'Indravati NP'    -> Indravati   [Chhattisgarh]    gazette PDF: yes
'Achanakmar WLS'  -> Achanakmar  [Chhattisgarh]    gazette PDF: yes
'Bandhavgarh NP'  -> Bandhavgarh [Madhya Pradesh]  gazette PDF: no
'Simlipal NP'     -> Simlipal    [Odisha]          gazette PDF: yes
'Simlipal WLS'    -> Simlipal    [Odisha]          gazette PDF: yes
```

Simlipal +1 as you predicted, plus 4 from the Chhattisgarh/Madhya Pradesh pages. Zero previously-working merges were lost. Only 2 of the 6 Chhattisgarh/MP tiger reserves gained a merge because WII's Madhya Pradesh page consolidates the whole state into one combined PDF rather than per-PA rows (its own YAML warns about this), so there's almost nothing there to match against — `Bandhavgarh NP` is the sole MP hit, and it carries no PDF link, which is why it's invisible in the fact table and the fact-level diff showed +4 against the counter's +5.

**Simlipal, confirmed merged** — one row where there were three (a `tiger_reserve` under "Orissa" plus an NP and a WLS under "Odisha"):

```
id 9  simlipal  Simlipal  tiger_reserve  Odisha  2750.0 sq km  est. 1973
  gazette_notification_pdf_url                     high    ntca.gov.in/.../Notification_Similipal.pdf
  gazette_notification_pdf_url_national_park       medium  v1.wii.gov.in/.../SimlipalNP
  gazette_notification_pdf_url_wildlife_sanctuary  medium  v1.wii.gov.in/.../simlipalWLS
  brief_note_pdf_url                               high    ntca.gov.in/.../simlipal.pdf
  tr_notification_year                             high    2007
```

All three sources' PDFs preserved side by side.

## Final state list in D1

**Zero rows** carry `Chattisgarh`, `Madhy Pradesh`, or `Orissa` — verified by direct query. 36 distinct states: WII's 35 canonical labels plus Telangana.

```
Andaman and Nicobar Islands 105   Jharkhand        12   Odisha          20
Andhra Pradesh               28   Karnataka        30   Puducherry       1
Arunachal Pradesh            14   Kerala           22   Punjab          12
Assam                        20   Lakshadweep       1   Rajasthan       31
Bihar                        11   Madhya Pradesh    9   Sikkim           8
Chandigarh                    2   Maharashtra      45   Tamil Nadu      28
Chhattisgarh                 16   Manipur           3   Telangana        2
Dadra and Nagar Haveli        1   Meghalaya         5   Tripura          6
Daman and Diu                 1   Mizoram           9   Uttar Pradesh   26
Delhi                         1   Nagaland          4   Uttarakhand     12
Goa                           7   Jammu and Kashmir 18  West Bengal     19
Gujarat                      27   Himachal Pradesh 36
Haryana                      10
```

Telangana's 2 tiger reserves (`Kawal`, `Amrabad`) remain unmerged, exactly as intended — WII has no Telangana page, so there is nothing to merge them with until that source is added.

## Other checks

- 602 reserves: 469 `wildlife_sanctuary`, 74 `national_park`, 59 `tiger_reserve`.
- `facts_written` 735 == `reserve_fact` rows 735 — still zero overwrites, no warnings fired.
- Name+state groups holding multiple rows: 8 → 7, the drop being Simlipal's NP/WLS pair folding into its tiger reserve. The rest are the legitimate type-split pairs (Gir, Bhitarkanika, etc.).
- Provenance: 0 violations. Idempotency: second run is a clean all-zeros no-op.
- GeoJSON still 0 features — untouched by this change, still waiting on the centroid backfill branch.
- Crawls fresh and clean: NTCA 1 page / WII 35 pages, all HTTP 200, 0 parser failures.

`git status` unchanged from where you started. Stand-ins are up on 8787/9000 against `…/scratchpad/dryrun3/`; `pkill -f stand_in.py` to stop them.

**Vishnu** (2026-09-06T02:35): Add a source-level cooldown/freshness check to this repo's harvest pipeline so scheduled runs don't redundantly refetch sources that haven't gone stale, and reduce the cron schedule to match how often each source type actually changes.

In .github/workflows/harvest.yml: change the seed-list job's cron from daily (17 3 * * *) to monthly (e.g. 17 3 1 * *, first of the month). Keep workflow_dispatch on all jobs. Leave species-data/corridors/hydrology as workflow_dispatch-only for now (unchanged) — when those graduate to scheduled runs later, they should also be monthly, not daily.
Add a --force CLI flag (or an env var, e.g. FORCE_REFETCH=1) to scripts/run_seed_list_harvest.py that bypasses the cooldown check below. Wire the seed-list job's workflow_dispatch trigger to accept an optional boolean input force_refetch (default false) and pass it through as that flag/env var only when true, so manual runs can still force a real refetch on demand.
Add a cooldown check before each spider invocation: query the source table for MAX(retrieved_at) where name = <source_name>, and skip calling scrapy crawl for that source (log "skipped — last fetched <date>, within cooldown") if the most recent successful fetch is younger than that source's cooldown window (30 days for ntca-tiger-reserves and wii-gazette-notifications; keep this configurable per-source, e.g. a small dict in run_seed_list_harvest.py, since threats/species sources should likely use a shorter window like 7 days once they're wired into this same pattern). Skip the check entirely when the force flag from step 2 is set.
Make sure this doesn't fight the existing SHA-256 idempotency in RawStoragePipeline — that's a separate, lower-level protection (content-level dedup within a fetch) and should stay as-is; this cooldown is a fetch-level gate that runs before a spider is even invoked.
Test locally against the existing D1/R2 stand-ins (same setup as the recent dry runs): run once normally, confirm a source row lands with a retrieved_at; run run_seed_list_harvest.py again immediately without force and confirm it logs skips for both sources with no new HTTP requests made (check via request count or the D1 source table row count staying flat); then run with the force flag and confirm it does refetch.
Report the cooldown windows chosen, the modified cron schedule, and the three-run test result (normal run fetches, immediate re-run skips, forced re-run fetches again).

**Claude** (2026-09-06T02:35): Let me read the two files I'll be changing.

**Claude** (2026-09-06T02:35): _[tool: Bash]_

**Claude** (2026-09-06T02:35): _[tool: Bash]_

**Claude** (2026-09-06T02:36): _[tool: Bash]_

**Claude** (2026-09-06T02:36): Now writing the cooldown-aware orchestrator:

**Claude** (2026-09-06T02:37): _[tool: Write]_

**Claude** (2026-09-06T02:37): Now the workflow — cron, dispatch input, and passing the flag through:

**Claude** (2026-09-06T02:37): _[tool: Edit]_

**Claude** (2026-09-06T02:37): _[tool: Edit]_

**Claude** (2026-09-06T02:37): Updating the now-stale "daily cron" comments on the dispatch-only jobs:

**Claude** (2026-09-06T02:38): _[tool: Edit]_

**Claude** (2026-09-06T02:38): _[tool: Edit]_

**Claude** (2026-09-06T02:38): Let me validate the YAML and unit-test the cooldown logic before the live runs:

**Claude** (2026-09-06T02:38): _[tool: Bash]_

**Claude** (2026-09-06T02:38): _[tool: Bash]_

**Claude** (2026-09-06T02:38): All helpers pass. Now the live three-run integration test against fresh stand-ins.

**Claude** (2026-09-06T02:38): _[tool: Bash]_

**Claude** (2026-09-06T02:39): _[tool: Bash]_

**Claude** (2026-09-06T02:39): _[tool: Bash]_

**Claude** (2026-09-06T02:39): _[tool: Bash]_

**Claude** (2026-09-06T02:39): _[tool: Bash]_

**Claude** (2026-09-06T02:40): Run 1 fetched. Capturing baseline state, then the immediate re-run:

**Claude** (2026-09-06T02:40): _[tool: Bash]_

**Claude** (2026-09-06T02:41): Run 2 skipped both sources with zero outbound requests. Now the forced runs — testing both the `--force` flag and the `FORCE_REFETCH` env var that CI actually uses:

**Claude** (2026-09-06T02:41): _[tool: Bash]_

**Claude** (2026-09-06T02:41): _[tool: Bash]_

**Claude** (2026-09-06T02:43): Source rows went 36 → 71 → 106, so the SHA-256 dedup only caught 1 of 36 pages. Let me find out why the WII pages hash differently each fetch.

**Claude** (2026-09-06T02:44): _[tool: Bash]_

**Claude** (2026-09-06T02:44): Root cause found. Let me tally the final state and check repo hygiene.

**Claude** (2026-09-06T02:44): _[tool: Bash]_

**Claude** (2026-09-06T02:45): Done. Both files changed, tested against the stand-ins across four runs.

## Cron schedule

`17 3 * * *` (daily) → **`17 3 1 * *`** — 03:17 UTC on the 1st of each month. `workflow_dispatch` kept on every job; `species-data`, `corridors` and `hydrology` stay dispatch-only and unchanged. I did update their stale "not the daily cron" comments, and recorded the monthly-not-daily intent (plus the ~7-day cooldown those observation feeds will want) on the `species-data` job where it'll be read when they graduate.

## Cooldown windows

A per-source dict in [run_seed_list_harvest.py](harvest-engine/scripts/run_seed_list_harvest.py), with a conservative default so a new source that's added without an entry under-fetches rather than hammers:

```python
SOURCE_COOLDOWN_DAYS = {
    "ntca-tiger-reserves":       30,
    "wii-gazette-notifications": 30,
    "parivesh":                  30,
}
DEFAULT_COOLDOWN_DAYS = 30
```

30 days for all three: they publish legal notifications, and the underlying facts move at gazette-notification pace — NTCA's table gained 3 reserves in the two years to 2026-09. I documented the 7-day intent for threats/species in the dict's comment but deliberately did **not** add entries for sources this script doesn't run, since dead config for an unfetched source misleads.

The gate queries `SELECT MAX(retrieved_at) FROM source WHERE name = ?` before each spider invocation, and **fails open** — if D1 is unreachable or the env vars are unset, it logs and fetches, mirroring `RawStoragePipeline._hash_exists`. A cooldown check that can't run must never be the reason a scheduled harvest silently does nothing.

`--force` and `FORCE_REFETCH=1` both bypass it. The workflow wires the manual input through as `FORCE_REFETCH: ${{ inputs.force_refetch && '1' || '' }}` — empty on scheduled runs, where the `inputs` context is null.

## Three-run test

| run | invocation | scrapy invoked | outbound HTTP | `source` rows |
|---|---|---|---|---|
| 1 | normal, cold DB | 3 sources | 38 | 0 → **36** |
| 2 | normal, immediate | **only parivesh** | **0** | 36 → **36** (flat) |
| 3 | `--force` | 3 sources | 38 | 36 → 71 |
| 4 | `FORCE_REFETCH=1` | 3 sources | 38 | 71 → 106 |

Run 2's output, verbatim:

```
--- ntca-tiger-reserves: skipped — last fetched 2026-09-06T02:39:23.657357+00:00, within cooldown (30d) ---
--- wii-gazette-notifications: skipped — last fetched 2026-09-06T02:40:27.621611+00:00, within cooldown (30d) ---
  ntca-tiger-reserves: SKIPPED
  wii-gazette-notifications: SKIPPED
  (2 skipped by cooldown — re-run with --force or FORCE_REFETCH=1 to refetch anyway)
```

Zero outbound requests and a flat row count confirm the skip is a real gate, not just a quieter log. I added run 4 because the env-var path is what CI actually exercises, and it behaves identically to the flag.

`parivesh` is invoked on every run by design: it fails at startup on the missing `DATA_GOV_IN_API_KEY`, so it never writes a `source` row, so `MAX(retrieved_at)` is NULL and it reads as never-fetched. A failed fetch correctly does not start a cooldown.

## One thing the test exposed

The forced runs added **+35 rows each, not +36** — the SHA-256 dedup caught NTCA every time but never fired for any of the 35 WII pages. I diffed two stored bodies of the same URL to find out why:

```
same URL stored 3 times with 3 distinct hashes
body sizes: 89746 vs 89746 chars
differing lines: 4
  -..."theme_token":"03ORzXfmEV7EP5lgxELxVLjD31-qUuB7tyKkIXktFqY"...
  +..."theme_token":"erIOdGsxMr99TPNJQqoitxPSaGWDPWXT-GCHtLHl6qY"...
  -...name="form_build_id" value="fo...
  +...name="form_build_id" value="fo...
```

WII runs Drupal, which stamps a per-request `theme_token` and `form_build_id` into every page. Four lines of session nonce in an otherwise byte-identical 89,746-character document, so the content hash differs on every fetch and the dedup is structurally defeated for that source.

That makes this change more load-bearing than it looks. Under the old daily cron, WII alone would have written 35 fresh `source` rows and 35 R2 objects **per day** — ~12,775 rows a year for a document set that doesn't change — and `normalize_seed_list.py` would have re-normalized all 35 every day, since each new row arrives with `normalized_at` NULL. Monthly cron plus the 30-day cooldown takes that to ~420 rows a year, and the cooldown is what enforces it, since the hash layer can't. I left `RawStoragePipeline` alone as you asked; it's still doing its job for NTCA.

Worth a separate look at some point: normalizing away those Drupal nonces before hashing would let the content dedup work for WII too. That's a pipeline change with its own trade-offs (you'd be hashing a modified body, not the raw one), so I didn't fold it into this.

Also unchanged but noted: `run_source` reports `OK` from the subprocess exit code, and `scrapy crawl` exits 0 even on a `start_error` — which is why `parivesh: OK` appears above despite the spider failing outright. Pre-existing, not something this change touched, but the summary is optimistic in that one spot.

Stand-ins are still up on 8787/9000 against `…/scratchpad/dryrun4/`. `pkill -f stand_in.py` to stop them.