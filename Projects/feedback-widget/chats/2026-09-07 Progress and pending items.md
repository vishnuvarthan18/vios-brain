---
tags: chat
project: "[[Projects/feedback-widget/SUMMARY]]"
date: 2026-09-07
source: Claude office account (Cowork)
session: cse_01CCm2u5A9NrbPCBzve9kXVi
---
# Progress and pending items

- Project: [[Projects/feedback-widget/SUMMARY]]
- Date: 2026-09-07
- Full chat, word for word (tool steps shortened).

---

**Vishnu:** hi good morning... where we stopped and what is pending

**Claude:**   _[tool: Projects — claude/PROJECT-INDEX.md]_

**Claude:**   _[tool: Projects]_

**Claude:** Good morning. Here's where things stand on the B. Halle feedback widget.

**Where we stopped**

The design is settled and specified. The 227-checkbox taxonomy is gone from the tester's view (kept internally as a triage list); the widget is now 5 screens, one question, 5 plain-language answers plus "something else", optional free text, automatic screenshots. Build-vs-buy came out as **build**, and after the 18-tool market scan it survives on two hard blockers: nobody randomises answer order, and every bought widget writes to the tester's device (§25 TDDDG problem).

The last substantive session was the four scope decisions of 2 Sept — "this page was fine" dropped, tester language follows the site's own switcher, no emails to testers at all, and the accessibility conformance *statement* declined (the work itself unchanged). Those live in `scope-decisions.md` and **override `BUILD-SPEC.md` in four places** — the spec has not been rewritten to absorb them yet.

**What's pending, in the order it matters**

1. **Three quick live-site checks** — how the site switches language (this blocks the whole language build), whether site-wide Webflow code runs on the 404 page, and the homepage product-grid ambiguity. Maybe 20 minutes total.
2. **Rewrite BUILD-SPEC** so the build agent reads one document instead of a spec plus four overrides.
3. **Two questions for Jakob** — Webflow plan allows custom code, and sign-off on "page opened" logging (it's behavioural logging of identifiable people, so it needs his yes either way).
4. **Commission the German five answers** — half a day of content work, not a translation job.
5. **Tester recruitment and the ~20-minute onboarding** — now load-bearing, since dropping reminder emails means the invitation and onboarding carry everything.
6. **Decide how we know a round is finished** — with no confirmations and no reminders there's no completion signal. Needs answering before testing starts.
7. Then tester list → assignments → Phase 1 ships.

Phase 1 target was testing in 2–3 weeks from 7 Sept, so the clock is running.

Want me to knock out the three live-site checks now, or start on the BUILD-SPEC rewrite?

**Vishnu:** so now what is the plan how thw widget will look and how the user will, do it and how we will track that

**Claude:**   _[tool: Projects — claude/scope-decisions.md]_

**Claude:**   _[tool: Projects — claude/widget-content-FINAL.md]_

**Claude:**   _[tool: Projects — claude/BUILD-SPEC.md]_

**Claude:** Read through the spec plus the four scope decisions that override it. Here's the whole thing end to end, as it now stands.

## 1. What it looks like

A single small dark pill button, bottom-right, fixed on every one of the 49 pages: **"Tell us about this page"**. That's the entire footprint until it's tapped.

Everything after that is one modal, max 450px wide, built for readability rather than looks — 16px minimum text anywhere, 23px headings, 56px-tall tap targets, 2px borders on options (not 1px), a 3px focus ring that's always visible, teal accent `#0E7C86`, no animation beyond a 150ms fade. It lives inside a Shadow DOM so Webflow's global stylesheet can't touch it and it can't leak out. Under 15KB gzipped.

## 2. How the tester does it

They get one email with a personal link — `halle-dev.webflow.io/?t=<their token>`. No account, no login, no password, ever. The token rides in the URL and the widget rewrites the site's internal links to carry it forward as they browse, because we deliberately never write anything to their device (that's the §25 TDDDG decision — it's what keeps a cookie banner off the client's site).

Then, per problem:

**Tap the button.** The launcher hides and a dark bar appears at the bottom: *"Click on the part that did not look right."*

**Point at it.** As they move the mouse, the element under the cursor gets outlined. Click, and it's captured — we intercept in the capture phase so clicking a nav link selects it instead of navigating. On an iPad there's no hover, so the first tap highlights and asks *"You picked this part — is that right?"* and the second confirms. If they can't isolate it, one button: **"It was the whole page."**

**Answer one question.** *"What happened?"* — five options, pick one, and picking it advances immediately (no submit button):

> The writing was too small or too faint to read
> I could not find what I was looking for
> Something looked broken or out of place
> I clicked something and it did not work
> I did not understand the words

Plus *"Something else — I will describe it."* The order is **randomised per session and the order shown is stored**, because older respondents lean on whatever sits first — that's the single feature no tool in the market has and one of the two reasons we're building rather than buying Ybug.

**Optionally type a sentence.** *"What were you trying to do?"* — genuinely optional, always last, Send or Skip-and-send both submit. Design as if half arrive empty.

**Done.** *"Thank you — that really helps."* Then either "something else on this page" (loops back to pointing) or finish.

Two taps minimum, under a minute. And the line that does the most work in the whole design sits at the top: **"We are testing this website — not you."**

One thing to be aware of: **"This page looked fine" is gone** — your call on 2 Sept. So is the "Page 4 of 8 checked" progress line, because with nothing to confirm there's nothing left to count, and showing a problem count as progress would quietly reward finding faults.

## 3. What we capture without asking

The tester types at most one sentence. Everything else is automatic: page URL and type, product/category, **exact click coordinates plus the element's text content** (text, not just a CSS selector — Webflow class names change whenever a designer renames a style), viewport and device, browser, OS, timestamp, seconds taken, which answer order they saw, the language they saw it in, and in Phase 2 a screenshot captured at the moment of reporting. That's where the value actually lives.

## 4. How we track it

Reports are **append-only** — never updated, never deleted, because that record is the client's sign-off evidence.

Staff-only dashboard, four screens:

- **Report Grid** — pages down, testers across. Amber = this tester reported problems here, grey = nothing received. Note the honest limit: grey now means *either* "she checked it and it was fine" *or* "she never opened it", and those are no longer distinguishable. It shows which pages generate complaints and who's active. It is **not** evidence the site was checked, and it shouldn't be presented to Jakob as such.
- **Report list** — newest first, filtered by page / tester / answer. Three testers hitting the same broken table read as **one item marked "3 testers"**, expandable.
- **Pages** — the 49, editable.
- **Testers** — list, create, copy each invitation link.
- **CSV export** — this evidences *problems found*, not coverage achieved.

Triage stays with your team: the old 201-item taxonomy didn't die, it moved. A report arrives as page + click location + one of five answers + screenshot + maybe a sentence, and your team tags it against the existing taxonomy. Jakob's reporting is unchanged.

## What still blocks parts of this

The language behaviour — widget follows the site's own switcher, both string sets served in one config call, re-rendering without throwing a mid-report tester back to the start — **cannot be built until someone spends ten minutes on the live site working out how it switches** (locale subdirectory, query param, or JS). And the German five answers don't exist yet; they need writing against the research, not translating.

The open decision that most affects tracking is **"page opened" logging** — it would split grey into "opened and sent nothing" versus "never opened", costs half a day, and needs Jakob's sign-off because it's behavioural logging of identifiable people.

---

Want me to rebuild the clickable prototype so you can actually tap through it — current wording, randomised options, no "page fine" button, no progress line — as something you could put in front of Jakob? The existing one in the project predates all four scope decisions.

**Vishnu:** here is what we need do do a deep resacch on the compaters we need to buidl the quality and featus like that

**Claude:**   _[tool: Projects — claude/market-feature-ledger.md]_

**Claude:** Before I set this running — the project already has an 18-tool ledger of *which* features exist, but nothing on *how well* they're built. So the new research should be about craft, not presence. Four things to pin down:

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: ToolSearch — select:TaskCreate,TaskUpdate,WebSearch,WebFetch,mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-]_

**Claude:**   _[tool: TaskCreate — Pick the top 3 of the five widgets (proposal: Marker.io, BugHerd, Ybug), confirm Chrome is reachable]_

**Claude:**   _[tool: TaskCreate — Drive the reporter/tester flow hands-on: launcher, element selection, annotation, form fields, requi]_

**Claude:**   _[tool: TaskCreate — Same protocol as competitor 1, so the three are directly comparable screen by screen.]_

**Claude:**   _[tool: TaskCreate — Same protocol as competitors 1 and 2.]_

**Claude:**   _[tool: TaskCreate — For each of the three: Shadow DOM or iframe isolation, element fingerprinting method (selector vs co]_

**Claude:**   _[tool: TaskCreate — Report list and detail view, grouping, filters, screenshot presentation, element re-location, export]_

**Claude:**   _[tool: TaskCreate — Every feature of the three, broken down bit by bit, against our spec: match / deliberately reject / ]_

**Claude:**   _[tool: TaskCreate — Turn the ledger into concrete additions, changes and explicit rejections for BUILD-SPEC, absorbing t]_

**Claude:**   _[tool: TaskCreate — An artifact showing their screens against ours, screen by screen, with the captured screenshots, so ]_

**Claude:**   _[tool: TaskCreate — Re-check every claim that will drive a build decision against a second source or a second hands-on p]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: mcp__claude-in-chrome__tabs_context_mcp]_

**Claude:** Claude in Chrome is offline, so I'll use the in-app browser instead — either satisfies hands-on.

**Claude:**   _[tool: ToolSearch — select:mcp__remote-devices__Claude_Browser__tabs_context,mcp__remote-devices__Claude_Browser__previe]_

**Claude:**   _[tool: mcp__remote-devices__Claude_Browser__preview_start — https://ybug.io/]_

**Claude:**   _[tool: WebSearch — Marker.io live demo try the widget no signup sandbox]_

**Claude:**   _[tool: WebSearch — BugHerd interactive demo try widget without account 2026]_

**Claude:**   _[tool: mcp__remote-devices__Claude_Browser__preview_start — https://www.markerdemo.com/]_

**Claude:**   _[tool: mcp__remote-devices__Claude_Browser__preview_start — https://marker.io/interactive-demo]_

**Claude:**   _[tool: mcp__remote-devices__Claude_Browser__request_access — https://marker.io/interactive-demo]_

**Claude:** Both browsers just dropped out from under me — the Chrome extension isn't connected, and the in-app browser's connection died (and every vendor URL I tried before it did was blocked by policy anyway). So I'll run the documentation pass now, feature by feature, and pick up the hands-on walkthrough when a browser is back.

Going with **Marker.io, BugHerd and Ybug** as the three: Marker.io is the best-built reporter flow in the category, BugHerd is the adoption benchmark and the closest thing to our pointing interaction, and Ybug is our actual buy-alternative (EU-hosted, 19 locales). Say the word if you'd rather swap one for Userback or Usersnap.

**Claude:**   _[tool: Agent — Research Marker.io feature by feature]_

**Claude:**   _[tool: Agent — Research BugHerd feature by feature]_

**Claude:**   _[tool: Agent — Research Ybug feature by feature]_

**Claude:**   _[tool: Bash — cd /root/.claude/projects/-home-claude/7236ea55-ecfb-5235-a38e-a7b458ca6efe/tool-results/ && mkdir -]_

**Claude:**   _[tool: Bash — cat /tmp/claude-0/-home-claude/7236ea55-ecfb-5235-a38e-a7b458ca6efe/scratchpad/research/markerio.md]_

**Claude:**   _[tool: Read — /tmp/claude-0/-home-claude/7236ea55-ecfb-5235-a38e-a7b458ca6efe/scratchpad/research/markerio.md]_

**Claude:**   _[tool: Read — /tmp/claude-0/-home-claude/7236ea55-ecfb-5235-a38e-a7b458ca6efe/scratchpad/research/markerio.md]_

**Claude:**   _[tool: Read — /tmp/claude-0/-home-claude/7236ea55-ecfb-5235-a38e-a7b458ca6efe/scratchpad/research/bugherd.md]_

**Claude:**   _[tool: Read — /tmp/claude-0/-home-claude/7236ea55-ecfb-5235-a38e-a7b458ca6efe/scratchpad/research/bugherd.md]_

**Claude:**   _[tool: Read — /tmp/claude-0/-home-claude/7236ea55-ecfb-5235-a38e-a7b458ca6efe/scratchpad/research/ybug.md]_

**Claude:** Three teardowns are in — dense, well-sourced, and they correct several things in our existing docs. Saving the raw evidence to the project first.

**Claude:**   _[tool: Bash — cd /tmp/claude-0/-home-claude/7236ea55-ecfb-5235-a38e-a7b458ca6efe/scratchpad/research/ && for n in ]_

**Claude:**   _[tool: Projects — claude/research-raw-markerio-teardown.md]_

**Claude:**   _[tool: Projects — claude/research-raw-bugherd-teardown.md]_

**Claude:**   _[tool: Projects — claude/research-raw-ybug-teardown.md]_

**Claude:**   _[tool: Bash — mkdir -p /home/claude/research && cp /tmp/claude-0/-home-claude/7236ea55-ecfb-5235-a38e-a7b458ca6efe]_

**Claude:**   _[tool: Projects — claude/research-raw-markerio-teardown.md]_

**Claude:**   _[tool: Projects — claude/research-raw-bugherd-teardown.md]_

**Claude:**   _[tool: Projects — claude/research-raw-ybug-teardown.md]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: TaskUpdate — Feature-by-feature from vendor docs, help centre, changelog, CSP, DPA. Saved to the project as resea]_

**Claude:**   _[tool: TaskUpdate — Feature-by-feature from vendor docs, string inventory, sub-processor register, CSP. Saved as researc]_

**Claude:**   _[tool: TaskUpdate — Feature-by-feature including the JS API, webhook schema, form field types, DPA. Saved as research-ra]_

**Claude:**   _[tool: TaskUpdate — Done to the limit of documentation: isolation model, CSP, screenshot method, storage posture, failur]_

**Claude:**   _[tool: TaskUpdate — Done from documentation: statuses, lists, grouping/merge, filters, export, sync, targeting for all t]_

**Claude:**   _[tool: TaskCreate — The documented pass leaves a defined list of unknowns that only DevTools and a live widget can answe]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:** Now the ledger that turns all of this into decisions.

**Claude:**   _[tool: Bash — mkdir -p /home/claude/out && cat > /home/claude/out/competitor-teardown-and-final-widget.md <<'ENDOF]_

**Claude:**   _[tool: Projects — claude/competitor-teardown-and-final-widget.md]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:** Now the visual comparison. Since no browser was reachable, I'll reconstruct each competitor's screens from their own documented strings rather than screenshots — and label them as such.

**Claude:**   _[tool: TaskUpdate]_

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


**Claude:**   _[tool: Write — /home/claude/out/widget-vs-market.html]_

**Claude:**   _[tool: Artifact — /home/claude/out/widget-vs-market.html]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: WebFetch — https://support.bugherd.com/en/articles/11430440-send-feedback-in-german-deutsch]_

**Claude:**   _[tool: WebFetch — https://ybug.io/docs/installation/language]_

**Claude:** Two highest-stakes claims re-verified against source, both hold. Updating the project index.

**Claude:**   _[tool: Bash — cat >> /home/claude/out/competitor-teardown-and-final-widget.md <<'ENDOFDOC'  ---  ## 12. VERIFICATI]_

**Claude:**   _[tool: Projects — claude/competitor-teardown-and-final-widget.md]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: Bash — cat > /home/claude/out/PROJECT-INDEX.md <<'ENDOFDOC' # B. Halle Feedback Widget — Project Index  **L]_

**Claude:**   _[tool: Projects — claude/PROJECT-INDEX.md]_

**Claude:**   _[tool: Projects — claude/widget-vs-market.html]_

**Claude:** Deep pass done on Marker.io, BugHerd and Ybug — feature by feature, from vendor documentation, changelogs, CSP pages, DPAs and sub-processor registers. Everything is in the project, and the visual comparison is published as the card above: the tester's journey, six steps across four products, built from each vendor's own default strings rather than screenshots.

**Three findings that change things**

BugHerd's pin — the thing the category is measured against — **only works for people logged into a bugherd.com account, and it needs a third-party cookie.** Anonymous reporters get a separate, older widget. The good experience and the anonymous one are not the same product, and our testers would get the lesser one.

Ybug **cannot record which order it showed the answers, and never could**: the webhook payload has no order, seed or permutation field. Even randomising by hand would leave nowhere to store what the tester saw. Our main build justification, now confirmed at the level of the data contract rather than the feature list.

And our own spec has the hole we were about to criticise BugHerd for. BUILD-SPEC promises WCAG 2.1 AA and keyboard-only reporting, but the pointing step is written as hover-then-click only. **There is no keyboard path to selecting an element.** That's a defect, not a trade-off — fix is to promote the region-list fallback to a permanent alternative reachable by Tab.

**Seven things we had wrong**

Ybug has 28 locales, not 19 (re-verified today), and €47 is its *top* tier annually — the entry price for custom fields at all. Marker.io has no element pinning whatsoever. Ybug stores no positional data at all, not "coordinates only". Ybug does publish an accessibility claim, so the "only Fable" line has to go. BugHerd's transfer scope is "any country" and their hosting region is disclosed nowhere. And duplicate grouping is no longer absent from the market — BugHerd shipped AI similar-task detection in December.

**The best single line for Jakob**, verified verbatim against BugHerd's own published German recipe: their success string reads *"Ihr Feedback wurde gesendet."* and their **error** string reads *"Feedback erfolgreich abgeschickt."* — the failure state tells a German tester their lost report succeeded. Their pin instruction stays in English, and the consent line isn't translated at all.

**What's waiting on you**

Two decisions: whether the launcher shows to everyone or only to token-holders, and whether we publish a storage statement — we write nothing to the device, which is provable and is exactly the document none of the three can produce.

Then the BUILD-SPEC rewrite, which now has to absorb both the four scope decisions and twelve changes from this pass — new bans on external fonts and on demanding any CSP concession, keyboard event scoping, contrast validation on the accent, and the keyboard path above. That's the next job and I've left it queued.

The hands-on walkthrough is still outstanding — all three vendor sites were blocked by policy in the in-app browser and Chrome wasn't connected. Nine questions documentation can't answer are listed, chief among them what each widget actually writes to a tester's device. When you've got Chrome up I can drive it, though the trial accounts would need to be created by you.

**Vishnu:** tell me the compates list

**Claude:** Here's the full competitor set across the project, grouped by how deep we've gone.

**The three we tore down feature by feature (7 Sept)**

- **Marker.io** — best-built screenshot annotation in the category; English-only widget, no pins, email required
- **BugHerd** — the pin benchmark, but the good widget is login-gated behind a third-party cookie
- **Ybug** — our only credible "buy" alternative; German hosting, 28 locales, but no radio groups and no way to record answer order

Full evidence in `research-raw-markerio-teardown.md`, `research-raw-bugherd-teardown.md`, `research-raw-ybug-teardown.md`.

**The other two feedback widgets (breadth scan only)**

- **Userback** — 24 locales but forced in code, custom text breaks translation; shipped native mobile SDKs Apr 2026
- **Usersnap** — closest to us on string control, gates multi-language to Premium; its AI runs on Anthropic models via AWS Bedrock

**The 13 testing and user-research platforms**

Testpad · TestMonitor · Qase · TestRail · PractiTest · TestLodge · Zephyr · Maze · Lyssna · UserTesting · Userlytics · Loop11 · Fable

Those 13 plus the 5 widgets are the "18 tools" the earlier ledger refers to. The ones that actually taught us something: **Testpad** (guest link, no login, nothing to learn — our model), **TestMonitor** (the only proper coverage grid with a grey circle for "no result"), **Qase** (load-balanced assignment), **Lyssna** and **Maze** (both refuse to infer completion from a pageview), and **Fable** (the only platform organised around an excluded population — but it professionalises the testers rather than simplifying the tool).

**Three proofing tools that have the confirmation button we dropped**

- **Atarim** — guest-facing "This is approved", no account needed
- **Pastel** — "Finish Review" works with zero comments
- **Filestage** — approve/needs-changes stamps, and the only tool in the whole scan that sends reminders to unfinished reviewers

These are why the old "nobody has positive confirmation" and "nobody sends reminders" claims had to be corrected.

**Six new entrants, mostly unverified beyond a landing page**

Simple Commenter (~$49 one-time) · Feedbucket (Swedish, $39/mo) · ReviseFlow · Huddlekit · Marklet (publishes an accessibility statement, unusual) · Quackback

**Adjacent threat for later — AI browser and QA agents**

BrowserStack's agent suite · TestMu AI (formerly LambdaTest) · Bug0 · Loop11's AI Browser Agents

Not feedback widgets, but they compete for the same "who checks the site before launch" budget. If this becomes a product, the pitch has to be about what a real 78-year-old could not do, which no agent can simulate.

Want me to deepen any of these? Userback and Usersnap are the obvious candidates if you'd like all five widgets at the same depth as the three.

**Vishnu:** now ther is only the pointer correct 

i wnast to add more like to that 

like this now we only have the pointer and if we clock that a option to enter the bug only that need to be more adance we need to implete more fetur and very simple thigns

**Claude:** Right — today the tester points at the element, then gets one question and an optional sentence. That's it. The toolbar you've shown is the other end of the spectrum: eight tools, freehand drawing, drag-to-place arrows.

Worth saying plainly before we choose: that toolbar is the one thing our research explicitly cut, and Ybug — which ships almost exactly it — documents in its own accessibility page that *"the only exception are our annotation tools which currently require the use of a pointing device."* For a 78-year-old on an iPad, dragging an arrow accurately is the hardest thing in the whole flow. But your instinct that the widget is thin is fair, and there's a lot of capability we can add that needs **no drawing skill at all** — some of it things nobody in the market has.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Dropping the drawing tools — good call, and it keeps the keyboard path and the WCAG position intact. On the other two I'll make the call myself.

**What I'm adding** — four things, none of which asks the tester to learn anything:

1. **Point at more than one thing** in a single report ("And this bit too") — stops three testers filing three near-identical reports about one broken table
2. **"Where did you look for it?"** — a second, optional point, shown *only* after "I could not find what I was looking for". The most useful thing a navigation problem can carry, and nobody in the market collects it
3. **Read it aloud, and bigger text** — a speaker button that reads the question and answers using the browser's own voice (no permission prompt, no network) plus A+/A−
4. **Plain-words confirmation** of what they picked, on desktop too, not just touch

**Rejecting the voice note** for Phase 1, for consistency: it throws a microphone permission dialog, which is the exact reason screen recording was cut in §0. It goes on the Phase 3 list.

And on who it's for — the honest answer is all three, so it sequences: tester additions now, a richer report view for you and Jakob alongside, and the demo carries the dashboard rather than the widget. Let me build it so you can click it.

**Claude:**   _[tool: Write — /home/claude/out/widget-prototype-v2.html]_

**Claude:**   _[tool: Artifact — /home/claude/out/widget-prototype-v2.html]_

**Claude:**   _[tool: Bash — cat > /home/claude/out/widget-v2-additions.md <<'ENDOFDOC' # Widget v2 — Additions to the Tester Flo]_

**Claude:**   _[tool: Projects — claude/widget-v2-additions.md]_

**Claude:**   _[tool: Projects — claude/widget-prototype-v2.html]_

**Claude:**   _[tool: Bash — cd /home/claude/out && python3 - <<'EOF' p='PROJECT-INDEX.md' t=open(p).read()  # 1. status line t=t]_

**Claude:**   _[tool: Projects — claude/PROJECT-INDEX.md]_

**Claude:**   _[tool: Projects — claude/PROJECT-INDEX.md]_

**Claude:** Prototype is live in the card above — click **Start a report** and go through it. It's the real flow on a stand-in B. Halle prism page, and the panel underneath fills up as you tap so you can see what the team receives.

**Yes — v1 was just the pointer, and now it's this:**

Point at something → **"You picked the table of details. Is that the part you meant?"** → **"And this bit too"** if more than one thing was wrong → the five answers, randomised, with a **speaker button that reads them aloud in the order they were actually shown** → and if they pick *I could not find what I was looking for*, one extra optional screen: **"Where did you look for it?"** → optional sentence → done.

There's also **"Choose from a list instead"** in the pointing bar — that's the keyboard path from the teardown, now built rather than promised. Try the second button on the demo to see it; the whole flow completes with no mouse at all.

The minimum path is still two taps and no typing. Everything added sits on a branch the tester chooses to take.

**What I rejected, so you know where the line is:** voice notes. It's genuinely the right idea for this group — speaking produces far more than typing — but it throws a microphone permission dialog, and that's the exact rule that killed screen recording in §0. Breaking it for one feature would cost us the "no permission dialogs anywhere" line. It's on the Phase 3 list.

**Two things worth knowing before you commit:**

The multi-target change needs a schema change — a `report_targets` table, because the single `target_*` columns can't hold three targets plus an expected one. The grouping key stays exactly as it was, deliberately: extra targets enrich a report, they don't split a group.

And the real cost isn't the code, it's the content. Every one of the 49 pages needs its four-to-nine regions named in plain words in both languages — "the big picture", "the table of details" — because those same names get spoken aloud, shown in the confirmation, and offered in the keyboard list. That's the largest single content item in the project now, and it isn't developer time. The build itself is about 2.75 days, which takes Phase 1 to roughly +3 days on the original estimate — still inside the window, no longer comfortably.

One last thing, on "it needs to look more advanced": I'd keep the tester's side looking almost embarrassingly simple, and put the sophistication in the report view instead — several named targets, the expected-versus-actual pair drawn together, an automatic crop of each element. Two taps in, fourteen fields out. That contrast is the argument to Jakob, and the prototype now demonstrates it live rather than describing it.

**Vishnu:** save all for now

**Claude:**   _[tool: Projects]_

**Claude:**   _[tool: TaskUpdate — The change lists are written and saved — scope-decisions.md (4 scope changes), competitor-teardown-a]_

**Claude:** All saved. The project now holds 29 docs — seven of them new from today.

**Written today**

- `competitor-teardown-and-final-widget.md` — the decision document: seven corrections, feature-by-feature comparison, twelve BUILD-SPEC changes, two open decisions, the restated build-vs-buy case, and a verification log
- `widget-v2-additions.md` — the four tester-flow additions, what was rejected, new schema, new strings, acceptance criteria, effort
- `research-raw-markerio-teardown.md` / `research-raw-bugherd-teardown.md` / `research-raw-ybug-teardown.md` — the full sourced evidence behind all of it
- `widget-prototype-v2.html` and `widget-vs-market.html` — both also live as artifacts, so they'll open from your gallery without digging up this conversation

`PROJECT-INDEX.md` is updated to match: new status, the four-document read order, the drawing-tools decision and the four additions in the settled table, the new docs listed, and region-naming added as its own open item.

**Where you'll pick up.** BUILD-SPEC is now overridden by three separate documents and has absorbed none of them, so rewriting it into one document is the top of the queue. Before that, two things are yours to answer — who sees the launcher, and whether we publish the storage statement. And the three ten-minute live-site checks are still outstanding; the language one still blocks the German build.
