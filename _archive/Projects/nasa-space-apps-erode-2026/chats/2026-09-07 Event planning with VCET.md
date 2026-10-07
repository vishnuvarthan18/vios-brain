---
tags: chat
project: "[[Projects/nasa-space-apps-erode-2026/SUMMARY]]"
date: 2026-09-07
source: Claude office account (Cowork)
session: cse_01MYGcuBUhdtY9vJobphPapZ
---
# Event planning with VCET

- Project: [[Projects/nasa-space-apps-erode-2026/SUMMARY]]
- Date: 2026-09-07
- Full chat, word for word (tool steps shortened).

---

**Vishnu:** collage accepted the invite and we are going to have the event in the collage 

aracreate has the local leavder 
vcet is the collage sponcerin the palave food and more so what we did is like 

VCET is condution the even in collabration wiht aracrete 

and we have not planned anyhtings yes we need to go in detail and plan more and more

**Claude:** Good news on VCET accepting. Before I build out a plan, I need a few basics locked down — everything else follows from these.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: ToolSearch — select:WebSearch,WebFetch,TaskCreate,TaskUpdate]_

**Claude:**   _[tool: WebSearch — NASA Space Apps Challenge 2026 dates October hackathon]_

**Claude:**   _[tool: WebSearch — NASA Space Apps Challenge 2026 local lead handbook host event requirements]_

**Claude:**   _[tool: WebFetch — https://www.spaceappschallenge.org/2026/]_

**Claude:**   _[tool: WebFetch — https://www.spaceappschallenge.org/host-an-event/]_

**Claude:**   _[tool: WebSearch — Space Apps 2026 local event Erode India spaceappschallenge.org]_

**Claude:**   _[tool: WebFetch — https://www.spaceappschallenge.org/resources/participant-faqs/]_

**Claude:**   _[tool: Artifact]_

**Claude:**   _[tool: TaskCreate — Dates, theme, registration, challenge release, submission and judging timeline]_

**Claude:**   _[tool: TaskCreate — Concept, VCET+araCreate split, org structure, 10-week timeline, 48hr run sheet, budget, marketing, j]_

**Claude:**   _[tool: TaskCreate — Publish a live page with timeline, task board, roles, budget and run sheet that Vishnu can share wit]_

**Claude:**   _[tool: TaskCreate — Cross-check every date against official sources, write the plan into the project for future sessions]_

**Claude:**   _[tool: Skill]_

**Vishnu:** Approach this as the design lead at a small studio known for their versatility, giving every client a visual identity pitched at the treatment the task actually calls for. Make deliberate choices about palette, typography, and layout that are specific to this subject, and avoid templated designs.

## Read the request first

Calibrate treatment, not whether to design. A doc deserves the same craft as a landing page - what changes is the treatment that craft is delivered in. Format is not part of this read: author HTML, and publish Markdown only when a loaded skill explicitly instructs it - a Markdown publish keeps its filename as its title and takes almost none of the craft below, and is never a way to save time.

Many requests call for a more utilitarian treatment: a plan, a memo, a demo. Make it polished: include real typographic hierarchy, considered spacing, and a proper palette, but avoid over-designing. Most pages do not need a flashy, gigantic hero. Keep flourishes tasteful and limited.

Some requests call for an editorial treatment: a landing page, a game, an app or tool they'll keep or share.

When unsure: a well-composed page is never the wrong answer; an over-designed visual identity sometimes is.

Fundamentals below apply to everything. The editorial process after that runs only when the read above says so.

## Fundamentals for every artifact

**Honor what's already there** Look for an existing design system first - CLAUDE.md, a tokens or theme file, existing component styles. When one exists, apply it; everything below fills gaps and never overrides. Precedence is always: the user's own words, then the project's existing system, then your choices.

**Ground it in the subject.** If the subject isn't already clear, pin it: one concrete subject, its audience, and the page's single job. The subject's own world - its materials, instruments, vernacular - is where distinctive choices come from. Whatever the treatment, carry at least one detail only this subject would have - its real units and scales, its document conventions, its terms of art - as content, not ornament; it costs a plain page nothing. Build with real content throughout, never lorem.

**Pair typefaces** Typography carries the page even when the page isn't about typography. Google Fonts is the one font host the Artifact CSP admits - link it directly (`<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=...&display=swap">`); a face from anywhere else must be inlined as a @font-face data URI or it falls back silently. Either way, declare a real fallback stack. Keep running text near 65 characters wide; set a type scale and stay on it; give headings `text-wrap: balance`, body text room to breathe, and uppercase labels a touch of letter-spacing.

**Load libraries, don't paste them.** When the page genuinely needs a library - React, a charting or highlighting package - load its UMD build from cdnjs (only the script - a library's stylesheet still has to be inlined) with one pinned `<script src="https://cdnjs.cloudflare.com/ajax/libs/...">` placed before the inline script that uses its global, instead of inlining the library's source or hand-writing a stand-in; the Artifact tool's description lists the few other script hosts the CSP admits. The page's own CSS and JS, its images and its data ship with the page. Most pages need no library at all - reach for one only when it carries real weight.

**Choose neutrals, don't default to them.** A pure mid-grey reads as unconsidered; a grey with a slight hue bias toward the page's accent reads as chosen. Pure white and near-black are fine grounds when they suit the subject - the point is that the neutral was picked, not inherited.

**Design both themes.** The page renders in the viewer's theme, and the viewer has three states, not two: an explicit choice stamps `data-theme="dark"` / `data-theme="light"` on the root element, and the default "system" setting stamps *nothing* - most viewers see the un-stamped document, where only `prefers-color-scheme` separates light from dark. Structure the CSS token-level for all three: the bare `:root` block defines the complete light palette (for a deliberately dark-first design, swap light and dark consistently through this whole pattern); `@media (prefers-color-scheme: dark)` redefines only the tokens, guarded as `:root:not([data-theme="light"])` so an explicit light choice beats a dark OS; `:root[data-theme="dark"]` redefines them again so the toggle also wins in the other direction. Style components through the tokens, never directly inside a media or `[data-theme]` block - a color whose only definition sits behind `[data-theme]` never applies in the un-stamped state, and the page renders one theme's text on the other theme's ground. Two more rules keep each theme resolving as a set: the artifact composites over a ground the viewer paints in *its* theme, so `body` must set an explicit `background` from a token - a transparent body silently borrows the host's ground; and every element that sets a color takes it from the same token set as the surface behind it, never a literal that only works in one theme. Declare every token in the bare `:root` block before any media or `[data-theme]` block redefines it - a color that exists only inside one of those blocks is the classic unreadable-artifact bug. Give the second theme the same care as the first - don't naively invert; keep contrast legible and the accent working on both grounds. A design that deliberately commits to one visual world (a neon arcade screen, a letterpress invitation) may stay single-theme - then skip the media query and stamps entirely but still paint the background and every color explicitly, so the page holds on either host ground; make it a choice, not an omission.

**Let layout do the spacing.** Lay out sibling groups with flex or grid and `gap`, not per-element margins that silently collapse or double. Wide content - tables, code, diagrams - gets `overflow-x: auto` on its own container so the page body never scrolls sideways. Reach for `font-variant-numeric: tabular-nums` wherever digits line up in columns.

**Compose repeated things as one object.** Cards in a row, label/value pairs down a list, badges on siblings: same edges, baselines and inner padding from one to the next, and a recurring element sits in the same place on each. Let content set a container's height and pick a column count the items fill, so nothing stretches over dead space or sits alone in a row. Text that can outgrow its track wraps or scrolls in its own container; clipped text is a bug.

**Not everything is a card.** Border, fill, radius and shadow each say "separate object" - spend them by role, lifting the one thing that needs it, instead of one radius and one shadow stamped on every block, which flattens the hierarchy. Lead with big-number tiles only when those figures are the point of the page.

**Draw charts to the scale.** One scale places marks, ticks and labels, and every label names a value the chart reaches; chart text takes its color from the theme tokens so it reads in both themes; marks, labels and edges stay clear of one another and inside the drawing's bounds - in SVG, leave room in the viewBox for the outermost labels and give every drawn shape an explicit fill.

**Show the page at rest.** Everything meant to be read is visible once the page has loaded, without scrolling to trigger it - that first still frame is what a thumbnail, a shared link, and a skimming reader all get. A section may animate in, but from a visible resting state, never parked at `opacity: 0` waiting on an observer. Size a hero to what it holds, not to the viewport; a `100vh` opener pushes the page itself out of that first frame. A tool or app opens in a realistic working state - the user's real data where it exists, otherwise example rows, a loaded sample, a form someone plausibly filled, plainly marked as examples and never passed off as the user's own figures - so the first look shows what it does; an empty shell waiting for input shows nothing.

**Avoid AI-generated design** AI-generated design currently clusters around a few looks: warm cream (#F4F1EA) with a serif display and terracotta accent; near-black with a lone acid-green or vermilion pop; broadsheet hairline rules with dense columns; a purple-to-blue gradient hero on white; Inter or Space Grotesk as the "safe" face; emoji as section markers; everything centered; `rounded-lg` everywhere; accent bar/rail on rounded cards. Where the user pins down a visual direction, follow it exactly - their words always win, including when they ask for one of these looks. Where nothing is specified, don't spend that freedom on one of these defaults.

**Build cleanly** Be cognizant of overlapping elements, cascade collisions, silent font fallbacks. Close every non-void element, double-quote attributes, give keyboard focus a visible state, respect `prefers-reduced-motion`. For generative or decorative graphics, reach for Canvas or WebGL rather than hand-authoring long SVG path data.

**CSS rules** When writing the CSS, watch your selector specificities. It is easy to generate classes that cancel each other out - a type-based selector like `.section` fighting an element-based one like `.cta` over padding and margins between sections. Structure the cascade so it doesn't silently undo your spacing.

**Writing the copy** Words are design material, not decoration. Write from the user's side of the screen - name things by what people recognize, not how the system is built (a person manages *notifications*, not *webhook config*). Active voice; a control says exactly what happens ("Publish", then a toast that says "Published"). Errors explain what went wrong and how to fix it - no apologies, no vagueness. Specific beats clever.

**Name the page like a product, not a caption.** The `<title>` is the artifact's name in the gallery and the browser tab, and it sets the reader's first impression of care. Give the page a real name: a short noun phrase, typically two to four words, specific to the subject - or, for a page that exists to answer one question, that question itself, which is then the page's name. Stop at the name - a title that carries its own explainer after a dash or colon reads as generated filler. The name must also identify the page among many: in the gallery it sits beside dozens of other artifacts, and a generic category label that could sit on any of them fails as a name just as surely as an appended explainer. When a candidate title pairs the name with a generic word - a greeting, a category, a page-type label - the name is the half to keep; a trim that drops the identity and keeps the generic word produces exactly the title that could sit on any page. And the rule removes explainers, it does not impose brevity: a multi-word title that already reads as one specific name is finished, and shortening it further only makes it generic. The one-sentence publish `description` is where the explanation belongs; the gallery shows it right under the title.

**Structure is information** Structural devices, numbering, eyebrows, dividers, labels, should encode something true about the content, not decorate it. Many generic designs use numbered markers (01 / 02 / 03), but that's only appropriate if the content actually is a sequence - like a real process or a typed timeline where order carries information the reader needs. Question if choices like numbered markers actually make sense before incorporating them.

**When it's a UI, not a document** A dashboard or tool is scanned and operated, not read top-to-bottom, so the craft shifts from typography to information design. Surface the summary before the detail; encode state in form as well as number - a pill, a chip, a severity stripe - so what needs attention reads at a glance. Semantic color (good / warning / critical) is separate from the accent hue and doesn't count as your accent. Give sparklines and charts the same care as type: an area fill, a faint grid, an emphasized endpoint. What's interactive should look interactive.



## Process

Start with what the viewer should be able to do on the page, not only what they will read: if it should take input, keep what people change for whoever opens it next, show live data, or ask Claude something, load the `artifact-capabilities` skill now and design around what it makes available to this user; a page that is only read needs none of that.

Before writing code, sketch a short design plan - a compact token system with color, type, and layout:
- **Color**: describe the palette as 4-6 named hex values.
- **Type**: typefaces for 2+ roles - a characterful display face used with restraint, a complementary body face, and a utility face for captions or data if needed.
- **Layout**: a layout concept in one or two sentences.

Then build, following the plan and deriving every color and type decision from it.

**Write, look once, publish.** Before publishing you may look at the rendered page once - one screenshot of the local file, or the Artifact tool's preview where it offers one - then one pass of edits for what it shows, without a second look. For a page that charts real numbers, take that look rather than skip it, and spend it on the chart. Don't build a test loop around your own file: no repeated screenshots, no pulling the script out to run it through node, no scripts that probe the DOM. That loop spends the session re-checking what a careful write already settled, while the user waits for a link. Then publish, check once any `window.claude` call the preview couldn't run, and stop: the live page is the review surface, and further polish is the user's to ask for. If the user reports something visibly broken - a clipped column, unreadable text, a control that does nothing - fix that and republish once.

## When the request is editorial

The stance shifts: the client has already rejected proposals that felt templated, and is paying for a distinctive point of view. Make opinionated calls, and take one real aesthetic risk where it serves the work.

Review the design plan against the subject before building: if any part of it reads like the generic default you would produce for any similar page, revise that part, and note what you changed and why. Only after you've confirmed the plan's uniqueness do you write the code, following the revised plan exactly.

**Principles** 

- The hero is a thesis: open with the most characteristic thing in the subject's world - headline, image, live demo, interactive moment. 
- Typography carries the personality of the page. Pair the display and body faces deliberately, not the same families you would reach for on any other project, and set a clear type scale with intentional weights, widths, and spacing. Make the type treatment itself a memorable part of the design, not a neutral delivery vehicle for the content. 
- Leverage motion deliberately. Think about where and if animation can serve the subject: a page-load sequence, hover micro-interactions, ambient atmosphere. An orchestrated moment usually lands harder than scattered effects; choose what the direction calls for. However, sometimes less is more, and extra animation contributes to the feeling that the design is AI-generated. 
- Match complexity to the vision. Maximalist directions need elaborate execution; minimal directions need precision in spacing, type, and detail. Elegance is executing the chosen vision well.
- Spend your boldness in one place; keep everything around it quiet. If the accent fights the ground, shift it toward analogous or drop saturation rather than replacing it.


**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: Skill]_

**Vishnu:** Base directory for this skill: /tmp/claude-0/bundled-skills/2.1.263/a1d88e833e1a68ed58fa71f0378b2036/artifact-capabilities

# Artifact runtime capabilities

A published Artifact page can declare **runtime capabilities** — abilities the claude.ai viewer grants the page at open time — by passing `capabilities: {name: config}` to the Artifact tool. The control plane is the authority on valid names and config shapes. Declaration gestures: **omitting** `capabilities` on a redeploy carries the stored declaration forward unchanged (and preserves the artifact's stored contract pin); an **empty object** `{}` is the explicit clear-all; a **non-empty object** is a full-set declaration (anything stored but not restated is revoked). Moving a republished artifact's runtime version is a deliberate gesture — pass `contract: 'latest'` to upgrade, or a specific version to pin or roll back — never a side effect of editing.

**Available capabilities:** `artifact`, `db`, `downloads`, `mcp`, `room`, `sample`, `self` — the complete set of capability names you may declare. Anything not listed is unavailable to this user.

Runtime contract 0.2.41


Capability namespaces live behind `claude.use(name)`: `const db = await claude.use("db")` resolves the capability's namespace, or `null` when this view cannot run it (not served, not granted, or failed to load — indistinguishable by design). Branch on `null` and design for absence. `window.claude` carries only `use`: no `window.claude.db`, `.room`, or `.artifact` member is ever promised, so never read one — render the page without them and light features up when the promise resolves (later, never within your script's first run, and unordered with DOMContentLoaded; `null` after 10 s when no viewer answers). The resolved namespace is frozen and platform-owned: call its functions and keep the reference; never assign to it, `defineProperty` on it, or replace a member (wrap it for your own helpers). Permission stays on the calls: a consent prompt, rate limit, or policy refusal arrives on the first call, never from `use()`. Awaiting `use("db")` again is free (memoized); an unknown name resolves `null`.


--- capability: artifact ---

Use `artifact` for pages that should remember what people do with them: polls, sign-up sheets, checklists, trackers, boards — the page is the record; data kept server-side, or seeded or read back by Claude, is `db`. Declare `capabilities: {artifact: {}}`; `const artifact = await claude.use("artifact")`, then `await artifact.publish(html)` saves `html` (a complete document, doctype first) as the new version, and every open view, this one included, reloads to it. Nothing a viewer types, ticks or drags is kept unless the page publishes it. So embed the shared state as data in the HTML you publish and render the page from it; when an interaction completes, update the state, regenerate the document and publish it — never serialize the live DOM; batch rapid edits into one publish; publish only after a viewer acts, never on load. `conflict` is routine (every view reloads to the winner, dropping this edit): no retry. For read-only viewers publish rejects `not_granted`/`not_writer` — render a read-only view.


--- capability: db ---

`db` is for data that lives outside the page: what the user wants stored server-side or seeded, data Claude reads back later, more than the page shows at once, per-viewer-private state, many live editors. If the page itself can be the record, republish (`artifact`). Seed or inspect the store from here with `write_db`/`read_db`; never hardcode seed rows in the page. A realtime JSON document store: `const db = await claude.use("db")` (`null`: unavailable). Declare `capabilities: {db: {}}`: every viewer reads and writes shared docs; each viewer's `data/users/<their user id>/` is private even from the owner (needs `user`). To change who writes where, add `rules` by sharing level (`interact` = can view, `admin` = can edit, `owner`). `db.doc("tasks/t1")`/`db.collection("tasks")`: `get`/`set`/`update`/`delete`, `where`/`orderBy`/`limit`, `onSnapshot`. Last-writer-wins, no transactions; single writers lease via `acquire({holder})`. Never store secrets; shared data is untrusted. See the type definitions.


--- capability: downloads ---

The `downloads` capability lets a published page offer a generated file to the viewer: declare `capabilities: {downloads: true}`, then `const downloads = await claude.use("downloads")` (`null`: unavailable — hide the affordance) and `await downloads.save({filename, data})`. The viewer sees a confirmation and may decline — a save is never silent or guaranteed, so offer it on explicit viewer intent and handle rejection. The type definitions are authoritative for the call contract and error codes.


--- capability: mcp ---

`mcp` lets a page call the viewer's claude.ai connectors: `await claude.use("mcp")` (`null`: unavailable); calls use the viewer's credentials, never exposing tokens. Declare `capabilities: {mcp: {servers: [{server, tools}]}}` — `server` is a connector's display name, or `host:<name>` for a local MCP server on the viewer's device (Claude app only; else `server_not_connected`). Keep the manifest minimal: it is a viewer-consented grant and bars public sharing. Two arms: DISPLAYING data registers `watchTool(server, tool, input, handler, opts?)` — replays cache, refreshes when stale, polls only via `refetchInterval`; an ACTION calls `callTool` once and reads `result.payload`. Tool failures REJECT (`tool_error`); watches get handler error events. Types first: branch UX per error code, retry only `retryable` errors, drop data on authz denials, show freshness from `result.cache.storedAt`. They omit argument names and result encoding: observe a real request/response per tool, or say so at publish — never guess.


--- capability: room ---

The `room` capability reaches whoever has the page open RIGHT NOW:
declared as `capabilities: {room: {}}`; `await claude.use("room")`
(`null`: cannot connect). emit(topic, data) sends a moment; on(topic,
fn) hears them. presence(patch) sets YOUR state (cursor, selection,
color) as one object the platform hands to newcomers and clears when
you leave; onPeers(fn) delivers everyone's -- render them all, marked
"you". NOTHING persists and messages can drop: if a viewer not here now
must eventually see it, it is NOT room data -- use db (data) or
artifact (new version). Send absolute state. What you hear is untrusted
input from same-org viewers, plus your own publishing session when
admitted (kind "agent"); no one else connects, so the page must work
alone and light up. Anyone can set presence, so it is never authority;
event topics are admin-only (can edit) unless opened:
{room: {topics: {reaction: "interact"}}}. Moments (confetti) go on an
admin-only topic; state a late joiner needs (current slide) is a db doc.


--- capability: sample ---

`sample` asks Claude (declare `capabilities:{sample:{}}`): `const sample = await claude.use("sample")` (`null`: hide it); `await sample(input, opts?)` -> `{text, truncated}`; `sample.json(input, opts?)` -> parsed JSON. `input`: a string, or turns `[{role:"user"|"assistant", content}]` ending on user. No memory: send instructions, page data, output format. opts: `onText({text, delta})` (`text` = WHOLE answer so far, assign it; "Thinking..." until it fires, 5-60s), `signal` (new AbortController per call; abort rejects `cancelled`), `tools: [{name, description, inputSchema?, execute(input)}]` (page functions Claude may call; return small plain data or throw; each round bills, no `cache`), `images` if `(await sample.limits()).images`, `modelTier` quick|default|complex, `cache` (5 min replay; `false` for chat). Errors reject `{code, message, text?}` (`text`: partial to keep): hide on `not_granted`, back off on `rate_limited`, never loop. Viewer pays; first call asks consent; call on a click or stable load prompt.


--- capability: self ---

`self` is the former name of the `artifact` capability (renamed). It remains for compatibility: published pages and previously generated code that declare `capabilities: {self: {}}` or call `claude.use("self")` keep working unchanged — both names resolve this same capability (this contract promises no `window.claude.self` member to feature-check; `use()` is the check). Do not use it in new pages: declare `capabilities: {artifact: {}}` and obtain the namespace with `await claude.use("artifact")`; see the artifact section for how to use it.


**Your connectors this session.** In this session, claude.ai connector tools appear in your tool list as `mcp__<connector>__<toolName>`. Set `server` to the connector's display name as it appears in claude.ai (usually the `<connector>` segment with underscores read as spaces). Only connectors the user added in claude.ai are valid `server` values — this session's other built-in MCP servers are not. The manifest's `tools` array takes the connector's upstream tool names (as returned by `listTools()` / `/v1/mcp_servers`), which can differ from the normalized `<toolName>` segment when an upstream name contains `.` or spaces. Every `servers[]` entry needs a non-empty `tools` array naming the tools the page calls — an empty or omitted `tools` list is refused and never means "all tools"; to publish without connector access, leave `mcp` out of `capabilities` (pass `capabilities: {}` to clear a stored declaration) rather than declaring an empty `servers` list. In hermetic/CI sessions where connectors aren't loaded but `$CLAUDE_CODE_OAUTH_TOKEN` is set, fetch the list via Bash: `curl -H 'anthropic-version: 2023-06-01' -H 'anthropic-beta: mcp-servers-2025-12-04' -H "Authorization: Bearer $CLAUDE_CODE_OAUTH_TOKEN" https://api.anthropic.com/v1/mcp_servers?limit=1000`; in that case use each entry's `display_name` as the `server` value (exact display names are always accepted alongside tool-prefix segments).

**Call contract** (runtime contract 0.2.41). The platform-served `window.claude` type definitions for this contract are extracted under `/tmp/claude-0/bundled-skills/2.1.263/a1d88e833e1a68ed58fa71f0378b2036/artifact-capabilities`: `0.2.41/artifact.d.ts`, `0.2.41/claude.d.ts`, `0.2.41/db.d.ts`, `0.2.41/downloads.d.ts`, `0.2.41/mcp.d.ts`, `0.2.41/room.d.ts`, `0.2.41/sample.d.ts`, `0.2.41/self.d.ts`. Read `/tmp/claude-0/bundled-skills/2.1.263/a1d88e833e1a68ed58fa71f0378b2036/artifact-capabilities/0.2.41/claude.d.ts` (how a page reaches any capability on this contract) and `/tmp/claude-0/bundled-skills/2.1.263/a1d88e833e1a68ed58fa71f0378b2036/artifact-capabilities/0.2.41/mcp.d.ts` before writing any code that calls the `mcp` capability — they are authoritative for this contract version over any remembered API shape. The type definitions cover only the call envelope — they do not tell you a connector tool's argument names or its result encoding. Never publish a page that calls a connector tool without having observed one real request/response pair for that tool in this session; if you cannot safely observe one (for example, the connector is unauthenticated here, or calling the tool would have side effects), say that explicitly to the user at publish time — in your reply, not as a note inside the published page — instead of shipping a guessed shape. Observed response payloads are the user's real data: learn the shape from them, but never embed the observed values in the published page as sample or placeholder data.

**Claude:**   _[tool: Bash — cat /tmp/claude-0/bundled-skills/2.1.263/a1d88e833e1a68ed58fa71f0378b2036/artifact-capabilities/0.2.]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: Write — /tmp/claude-0/-home-claude/fac8c4f8-9319-528b-aa39-0702ebfcad03/scratchpad/erode-mission-plan.html]_

**Claude:**   _[tool: Artifact — /tmp/claude-0/-home-claude/fac8c4f8-9319-528b-aa39-0702ebfcad03/scratchpad/erode-mission-plan.html]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: Projects — claude/space-apps-erode-2026-brief.md]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:** I built the full plan as a live page — **Space Apps Erode Mission Plan**. It's private to you and shareable with VCET when you're ready.

A few things worth knowing before you open it:

**You have 68 days.** Hackathon is 14–15 November, theme is *The Next Frontier*. Challenge statements only drop 28 October, so everything between now and then is recruitment and de-risking — we don't design the problems, NASA does.

**The plan covers:** the araCreate/VCET ownership split (no overlap in the middle), NASA's fixed dates and rules, a six-phase countdown with ~65 tickable tasks, the full 48-hour run sheet with three hard checkpoints, a 62-person crew roster, a ₹4.93 lakh budget sized for 200 participants, and an 11-item risk register.

**Two things I'd flag hardest:**

1. The single most common failure at local events is people registering with *you* but never selecting Erode on spaceappschallenge.org. I've built verification into three places.
2. **Overnight or two long days** — this decides food, security, hostel permissions, the run sheet and your marketing copy. It cannot be reversed in November. Settle it this month with the wardens in the room.

The last section lists nine decisions that are blocking everything else. Answer those and I'll rebuild the plan around your real numbers — the budget, cap and run sheet all move together.

Sources: [Space Apps 2026](https://www.spaceappschallenge.org/2026/) · [Host an event](https://www.spaceappschallenge.org/host-an-event/) · [Participant FAQs](https://www.spaceappschallenge.org/resources/participant-faqs/)

**Vishnu:** make the artifact in simple words now it is too complicated

**Claude:** Rewriting it in plain, short language.

**Claude:**   _[tool: Write — /tmp/claude-0/-home-claude/fac8c4f8-9319-528b-aa39-0702ebfcad03/scratchpad/erode-mission-plan.html]_

**Claude:**   _[tool: Artifact — /tmp/claude-0/-home-claude/fac8c4f8-9319-528b-aa39-0702ebfcad03/scratchpad/erode-mission-plan.html]_

**Claude:** Rewritten in plain English — same link, same plan, much easier to read.

What changed:

- Section names are now normal words: "What this is", "Who does what", "Dates we can't change", "The two event days", "Money", "What can go wrong", "9 things to decide"
- Times are 10:30 am, not 10:30. Stages 1–6, not Phase 00–05.
- Cut the long paragraphs. The opening is now four short lines that explain the whole event.
- Plain terms throughout — "problems" not "challenge statements", "upload" not "submission", "wifi crashes" not "network saturation", "people register but don't turn up" not "no-show rate".
- Checklist items are shorter and say what to actually do.

Your ticks are saved, so nothing you'd already marked is lost.

**Vishnu:** i have lot of conufeesion... i have to manage all because i am the local leader

**Claude:** That feeling makes sense, but I think the plan is scaring you more than the job actually will.

Here's the thing: **being Local Lead does not mean doing everything.** It means one specific job — you are NASA's person in Erode. Nobody else can do these four things:

- Keep the Erode listing correct on NASA's site
- Attend the monthly Local Lead calls
- Send your best teams to NASA for global judging
- Make sure the NASA branding rules are followed

That's it. That's the part that is truly, only yours.

Everything else on that page — food, halls, security, volunteers, printing, wifi, chairs — **is not your work.** VCET agreed to host. Hosting means they carry that. Your job there is to ask for it and check it's happening, not to arrange it.

And of the araCreate side, you should be handing out ownership, not holding it. Mentors, judging, posters, sponsors, the schedule on the day — each of those needs one name that isn't yours. If you're the schedule keeper on 14 November, you will also be with the chief guest, and with the judges, and with the press, and the day will slip.

**Right now, in September, only three things need you personally:**

1. Open spaceappschallenge.org and check the Erode listing — address, capacity, timings, contact. Fifteen minutes.
2. Get one answer from VCET in writing: can people stay overnight, yes or no.
3. Get two names from VCET — the faculty in charge, and the schedule keeper.

Nothing else is due this month. The other 60 items are October and November, and most belong to other people.

Would it help if I made you a much smaller page — just your own jobs, in order, with dates, and a separate list of what you're asking other people to take? You could hand that second list straight to VCET.
