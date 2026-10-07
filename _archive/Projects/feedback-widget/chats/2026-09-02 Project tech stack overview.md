---
tags: chat
project: "[[Projects/feedback-widget/SUMMARY]]"
date: 2026-09-02
source: Claude office account (Cowork)
session: cse_01EFh21E6jd7wxSiwGWTZLah
---
# Project tech stack overview

- Project: [[Projects/feedback-widget/SUMMARY]]
- Date: 2026-09-02
- Full chat, word for word (tool steps shortened).

---

**Vishnu:** we need to start workin on this project soo 

so tell me what ar ethe tech stack what ar the thigns will ne ther in this porjec front end backe dned DB 

explain me the flow what user will how the clint will set it up 

all in simple words not thec words

**Claude:**   _[tool: Projects — claude/PROJECT-INDEX.md]_

**Claude:**   _[tool: Projects — claude/BUILD-SPEC.md]_

**Claude:**   _[tool: Projects — claude/widget-build-brief.md]_

**Claude:**   _[tool: ToolSearch — select:TaskCreate,TaskUpdate]_

**Claude:**   _[tool: TaskCreate — Pull the settled tech decisions, schema, API and flow out of BUILD-SPEC.md, widget-build-brief.md an]_

**Claude:**   _[tool: TaskCreate — Four pieces of the system in non-technical words, plus the client setup flow, the tester flow and th]_

**Claude:**   _[tool: TaskCreate — Publish the explainer as an artifact so Vishnu and the team can come back to it and send it on]_

**Claude:**   _[tool: TaskCreate — Verify the stack names, hosting choices, screen states, copy and setup steps match BUILD-SPEC.md exa]_

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


**Claude:** Read through the spec. Here's the picture, in plain words.

**Claude:**   _[tool: Write — /home/claude/halle-widget-explainer.html]_

**Claude:**   _[tool: Artifact — /home/claude/halle-widget-explainer.html]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:** No page then — here it is in chat.

---

## What we are making

Old German testers walk through B. Halle's website. If something looks wrong, they **point at it** and pick **one of five plain sentences**. If nothing looks wrong, they press **one button that says so**.

That second part is the whole reason we build instead of buy. No tool on the market can record "I looked at this page and it was fine." Without it, a clean page and a page nobody opened look identical.

---

## The tech stack — four separate pieces

**1. The widget — what testers see**

The dark "Tell us about this page" button in the corner of the client's site. The only part a tester ever touches.

- **Plain JavaScript.** No React, no Vue, no Tailwind.
- Why: it must be under 15KB (about one small photo) and must sit inside someone else's website without the two fighting. So it renders in a **sealed bubble** (Shadow DOM) — Webflow's styling can't reach in, ours can't leak out.
- Two tiny helpers: `@medv/finder` (1.5KB, works out what was clicked) and `modern-screenshot` (10KB, takes the picture — Phase 2 only).
- **Lives on a CDN**, not Webflow. Webflow refuses to host `.js` files.

**2. The API — the go-between**

A small program between the widget and the database. Nothing touches the database directly.

- **Cloudflare Workers.** No server to rent or look after. Runs in whichever data centre is nearest the tester, so it's fast in Germany. Free up to 100,000 requests a day — we'll use a fraction.
- Two jobs: **tells the widget what to say** (every word comes from here, nothing hardcoded — so the same widget file works for the next client), and **writes a report down** when one arrives.

**3. The database — the memory**

- **Postgres, hosted in an EU region** (Supabase or Neon). German client, German law — data doesn't leave Europe.
- Why Postgres: the main screen is one question — *every page crossed with every tester, and the state of each* — and this kind of database answers that in a single query.
- Seven lists inside: organisations, staff, projects, pages (49), testers, assignments (who checks what), reports.
- **One rule:** every single row is stamped with which organisation it belongs to, from day one. ~5 extra days now, saves ~20 if we ever sell this.

**4. The dashboard — what your team sees**

- **React or SvelteKit.** A private site only araCreate staff log into. Testers never see it and have no account at all.
- Four screens: coverage grid (the main one), report list, pages, testers. Plus a CSV download — that download is what you hand the client as proof.

**Joining later in Phase 2:** Cloudflare R2 for screenshot storage (10GB free), and Resend or Postmark for the weekly reminder emails.

---

## How the client sets it up — 6 steps, 5 of them yours

1. **You create the project** in the dashboard → it gives you a key like `pk_live_a1b2c3`. This key is meant to be public, safe in website source code.
2. **You add the 49 pages** — an address plus a friendly name each.
3. **You create the testers** — each gets a label like "Tester 07" and a secret code. No real names needed. No email needed until Phase 2.
4. **You press "generate assignments"** — every page goes to 3 different testers (49 × 3 = 147 assignments, so about 7–8 pages each with 20 testers). Home, Contact and 404 go to everyone. *Why three: one person misses roughly a third of the problems on a page they're staring at.*
5. **Someone pastes one line into Webflow** → Site settings → Custom code → Footer → Publish. That's the entire install, and it covers all 49 pages including CMS product pages.
6. **You email each tester their own link** — `halle-dev.webflow.io/?t=<their code>`. Nothing else.

No plugin, no app, no download for anyone.

---

## What a tester actually does

1. Opens the link. Sees the **normal** B. Halle site, plus one dark button bottom-right.
2. The `?t=` code in the link is how we know who they are. As they browse, we quietly add it to every link so it follows them. **We never save it on their computer** — no cookie, no browser storage. German §25 TDDDG says putting *anything* on a visitor's device needs consent even when it isn't personal data. Keeping it in the address bar means the question never comes up, so no cookie banner.
3. **Nothing wrong?** Button → "This page looked fine". One tap, recorded, done.
4. **Something wrong?** They read: *"We are testing this website — not you. If something looked wrong, that is the website's fault, not yours."* This is the best-supported element in the entire design — across 1,204 people, the age gap in tech use ran through **anxiety and confidence**, not age.
5. **They point.** Whatever's under the mouse gets a box drawn round it. Clicking picks the thing instead of following the link. *On iPad there's no hovering, so first tap highlights and asks "is that right?", second tap confirms — older users mis-tap constantly.*
6. **One question, five sentences.** One tap moves it along, no Submit button to hunt for:
   - The writing was too small or too faint to read
   - I could not find what I was looking for
   - Something looked broken or out of place
   - I clicked something and it did not work
   - I did not understand the words
   - *Something else — I will describe it*

   Five different **activities** — reading, searching, looking, doing, understanding — not five kinds of fault. **Order is shuffled every session and recorded**, because older people tend to pick whatever's first; a fixed order would bend the data permanently.
7. **Optional typing.** Most skip it. Fine — a typical voluntary comment is ~61 characters, so nothing depends on it.
8. **"Thank you."** Anything else on this page? Yes → back to pointing, same session. No → close. Bar also shows "Page 4 of 8 checked".

Five states total: `idle → pointing → question → detail → sent`, with one loop back from *sent* to *pointing*.

---

## Where the data goes

**Tester points** → **API checks it** → **written into Postgres forever** → **your team reads the coverage grid** → **client gets the CSV**

Each report carries: which page, which tester, which sentence, anything typed, **the text of the thing they clicked** (e.g. `Extinction ratio 1 × 10⁻⁵` on the Glan-Thompson page), screen position, device, browser, seconds taken, and which order they saw the options in.

We store the clicked element's *text*, not just its address on the page, because Webflow renames things whenever a designer touches a style. The words are what let us find that element again later.

---

## Rules that don't bend

- **Never break the client's site.** Bad key, dead API, JS off → widget just doesn't appear. No error, no console noise.
- **Never show a tester an error.** Failed send still shows "thank you" while we retry. A lost report is our problem, not theirs.
- **Reports are written once** — never edited, never deleted. It's the client's sign-off evidence.
- **No words live in the widget.** Every string comes from the API.
- **Nothing on the tester's device.** Ever.
- **WCAG 2.1 AA properly** — 16px minimum text, 56px tap targets, keyboard-only completion, visible focus, 4.5:1 contrast. Not one of the eleven competitors claims any accessibility conformance.

---

## Order of work

- **Phase 1 (now, 2–3 weeks):** everything above minus screenshots and emails. ~5–6 days for the minimum useful version, 8–13 for all of Phase 1.
- **Phase 2 (once testing runs):** screenshots (shown to tester first with a "don't include it" option), weekly reminders, 90-day auto-delete.
- **Phase 3 (only if we sell it):** signup, billing, multi-client, white-label. Schema already supports it.

---

## Do these before any code

1. **404 check — five minutes.** Publish with the widget, visit a nonexistent URL, look for the script tag in DevTools. The 404 is one of the 49.
2. **Ask Jakob** whether their Webflow plan allows custom code.
3. **Decide who the testers are** — nothing can be assigned without the list.
4. Write the invitation email + ~20 minutes of onboarding.
5. Eyeball the homepage — a scan found what may be a 40-tile product grid behind the visible six.

---

I did build this as a page before you declined it — want it as a file instead, or shall I just keep it in chat?

**Vishnu:** wow this is super cool litterly super pan but one thisn we need to provife all the featies that in hte market these type of widgest have so do areceh what are the featues they have in that what we have correny and oters we need to finalize

**Claude:**   _[tool: Projects — claude/feature-parity-analysis.md]_

**Claude:**   _[tool: Projects — claude/market-and-build-vs-buy.md]_

**Claude:** Existing parity analysis is solid but let me pressure-test it with fresh research across the market.

**Claude:**   _[tool: TaskCreate — Fan out across BugHerd, Marker.io, Ybug, Userback, Usersnap, Pastel, Ruttl, Atarim, zipBoard and cur]_

**Claude:**   _[tool: TaskCreate — Testpad, TestMonitor, Qase, TestRail, Maze, Lyssna, UserTesting — participant-facing flows, assignme]_

**Claude:**   _[tool: TaskCreate — Every market feature marked: already in our spec / deliberately excluded with the reason / real gap ]_

**Claude:**   _[tool: TaskCreate — Separate tester-facing calls (where the research constrains us) from team-facing calls (where adding]_

**Claude:**   _[tool: Agent — Research visual feedback widget features]_

**Claude:**   _[tool: Agent — Research UAT and testing platform features]_

**Claude:**   _[tool: Bash — cd /root/.claude/projects/-home-claude/2b61ea88-cb3e-5d01-95a0-adfe085498ec/tool-results/ && python3]_

**Claude:**   _[tool: Bash — cd /root/.claude/projects/-home-claude/2b61ea88-cb3e-5d01-95a0-adfe085498ec/tool-results/ && python3]_

**Claude:**   _[tool: Bash — grep -n "^#\|^---$" research-widgets.md | head -80]_

**Claude:**   _[tool: Bash — grep -n "^#\|^### " research-uat.md | head -60]_

**Claude:**   _[tool: Bash — sed -n '270,470p' research-widgets.md]_

**Claude:**   _[tool: Bash — sed -n '212,300p' research-uat.md]_

**Claude:** Research is back — 18 tools, and it corrects three things in our own docs. Let me give you the findings, then the decisions I need from you.

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:** # Three corrections to our own docs

**1. "No tool records positive confirmation" — too strong.** No *feedback widget* does. But three proofing tools do, and one is very close to our design:

- **Atarim** gives guests two buttons in the page's bottom bar, no account needed: **"This is approved"** and **"I'm done"**. Approval lives on the page itself. Its weaknesses: **no central approvals screen**, and the control is **hidden on mobile**.
- **Pastel** has a green "Finish Review" button — and crucially it works with **zero comments**, so a reviewer can approve without complaining. But approvals **reset every time a new version is made**.
- **Filestage** has one-click approval stamps and an "Awaiting my review" sidebar.

Our differentiator survives, but it's now **"positive confirmation *plus* a coverage grid *plus* reminders"**, not positive confirmation alone.

**2. "Not one of 12 tools sends reminders" — wrong.** **Filestage does**: due dates plus automatic email reminders to reviewers so decisions arrive on time. PractiTest emails testers when a test is *ready to run* (a start notification, not a nudge). Still true: no feedback widget and no test-management tool nudges unfinished testers. Gap holds, narrower.

**3. Marker.io shipped a "Waiting for Approval" status on 6 Feb 2026**, renameable to "Client Review" / "Needs QA". It's per-issue, team-set, and guests **cannot change statuses** — so not reporter-facing. But the market is moving toward this. Argument for shipping sooner.

---

# Two things became table stakes in the last 12 months

| New baseline | Who has it |
|---|---|
| **MCP server** | All five widgets shipped one within ~12 months. Now a checkbox, not a differentiator. |
| **AI auto-title / auto-categorisation** | Four of five. Ybug's total absence of AI is now conspicuous. Usersnap's runs on **Anthropic models via AWS Bedrock**. |

---

# The ledger

**Where we already match or beat the market**

Script install · floating launcher · element pinning (BugHerd's strongest feature, and **Marker.io still doesn't have it** — 105 votes, "planned") · auto screenshot · auto environment capture · anonymous no-account reporting · custom metadata · CSV export · EU data residency · GDPR + DPA · works on staging.

Two places we're genuinely ahead: **BugHerd has no EU residency at all** (AU/UK/US transfers — for a German client that's disqualifying), and **Marker.io forces every reporter to type an email address**.

**Deliberately out — research confirms the call**

Session replay · screen recording (permission dialog) · console/network logs · drawing tools · kanban/priorities/workflows · two-way Jira sync · NPS/CSAT/surveys · conditional logic · duplicate merge · AI sentiment · SSO · white-label · mobile SDK · browser extension (we own the Webflow footer).

One vindication: **Usersnap's own mobile SDK docs admit "screen capturing features and annotation toolbar… not supported at the moment."** Even the vendors can't make capture work everywhere.

**Confirmed uncontested ground**

Nothing in the market targets elderly or low-digital-literacy testers — I searched specifically. The evidence base is academic, not product. And **Fable is the only one of 18 tools with a WCAG conformance claim covering the tester-facing product** (WCAG 2.1 A + AA via EN 301 549 V3.2.1, report on request, updated July 2026). Twelve of thirteen UAT tools publish nothing. Four of five widgets claim nothing.

---

# The real gaps — two are ideas we should steal

**Testpad has a fifth state: "query".** One keypress, distinct from fail — *"I'm not sure / something felt odd."* We currently have only `page_ok` and `problem`. The researcher's read: this is the most common thing an elderly tester actually wants to say, and every other tool forces it into "fail" or into silence.

**PractiTest has an N/A escape hatch** — mark a step "didn't apply" and finish the run without failing it. Without an explicit version, testers either guess, skip silently, or stop.

Plus: **Qase load-balances assignments by estimated effort, not case count** — our generator does even *count*, so someone could get the homepage plus seven heavy product pages. Cheap fix.

Four things need your call before the agent starts.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** My fault — I wrote those questions in tech language. Let me redo them properly.

First, two things I should answer, not ask:

**The language button — got it, that's clear.** The Halle site has a language switcher. When a visitor switches the site to German, the widget's words switch to German too. When they switch to English, the widget goes back to English. That's easy for us because all the widget's words already come from our server, not from inside the widget. I'll add it as a requirement: the widget watches the site's language and matches it. We just need to write the German words as well as the English ones.

**On dropping "this page looked fine"** — you may well be right, but let me show you what it does first, because it's the reason we decided to build instead of buying a tool. One small story:

> Tester 7 is given 8 pages. She looks at all 8. Six were good. Two had problems. She reports the two problems.
>
> Next week you open the dashboard. You see 2 reports from her.
>
> But you cannot tell which of these happened:
> - She checked all 8 pages, and 6 were genuinely fine
> - She only ever opened 2 pages and never looked at the other 6
>
> **Both look exactly the same.** Two reports, silence on the rest.

With the "fine" button she taps it 6 times, and now you know: 8 checked, 6 good, 2 problems. And at the end you can tell Jakob *"all 49 pages were checked by 3 people each"* — with proof. That sentence is what he is actually buying.

Without it, the widget still collects bugs perfectly well. You just can't ever prove the site was fully checked, and the coverage grid becomes mostly grey.

So — your call, now that you know what it's for:

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Decisions logged. Three of those have knock-on effects you'll want to know about — let me write the changes into the project first.

**Claude:**   _[tool: Bash — cat > /home/claude/market-feature-ledger.md << 'MDEOF' # Market Feature Ledger  **Researched:** 2 Se]_

**Claude:**   _[tool: Bash — cat > /home/claude/scope-decisions.md << 'MDEOF' # Scope Decisions — 2 September 2026  **Decided by:]_

**Claude:**   _[tool: Bash — cat > /home/claude/PROJECT-INDEX.md << 'MDEOF' # B. Halle Feedback Widget — Project Index  **Last up]_

**Claude:**   _[tool: Projects — claude/market-feature-ledger.md]_

**Claude:**   _[tool: Projects — claude/scope-decisions.md]_

**Claude:**   _[tool: Projects — claude/PROJECT-INDEX.md]_

**Claude:**   _[tool: TaskUpdate]_

**Claude:** Saved. Three documents in the project now: the market research, your four decisions, and an updated index so whoever builds this reads the current scope and not the old one.

**Two things in our old docs were wrong.** Don't repeat these to Jakob:

- We said no tool can record "this page was fine". Not true — **Atarim** has exactly that, a button guests can press with no account. **Pastel** and **Filestage** too. What's actually unique to us is pairing it with page-by-tester tracking.
- We said no tool sends reminders to testers. **Filestage does.** Still true that no *website feedback widget* does.

**Three side-effects of dropping the "fine" button** — worth knowing before you talk to Jakob again:

1. **The "Page 4 of 8 checked" counter is gone.** It counted good pages. Nothing left to count.
2. **The main dashboard screen changes meaning.** It was going to show green / amber / grey. Now it's only amber (problems) or grey — and grey means *either* "she checked it and it was fine" *or* "she never opened it". You can't tell which. So it's a **problem list**, not proof of coverage. I've renamed it in the docs so nobody accidentally sells it as proof.
3. **Your pitch to Jakob changes.** It was "we can prove all 49 pages were checked." It's now "you can trust the data, and you don't need a cookie banner." If he asks why not just buy Ybug for €47/month, those two are your answer — Ybug can't shuffle the five options (which is needed or the results are biased), and it writes to the tester's computer, which forces a cookie banner on his site.

**Timing barely moves.** You cut about 1.5 days (the fine button, the counter, the emails) and added about 1.5 days (language switching, German words, grouping reports). Still 2–3 weeks for Phase 1. Phase 2 got much smaller — just screenshots now.

**One thing I couldn't finish:** the language switcher. Before anyone builds it, we need to know *how* the Halle site switches language — Webflow does it a few different ways, and it changes how the widget detects it. That's a ten-minute look at the live site.

Want me to go check that now?

**Vishnu:** super save all

**Claude:**   _[tool: Bash — ls -la *.md *.html 2>/dev/null && echo "---" && du -sh research-widgets.md research-uat.md halle-wid]_

**Claude:**   _[tool: Bash — cd /home/claude  # Header for the widget research cat > _hdr1.md << 'EOF' # Raw Research — Visual Fe]_

**Claude:**   _[tool: Projects — claude/research-raw-feedback-widgets.md]_

**Claude:**   _[tool: Projects — claude/research-raw-uat-platforms.md]_

**Claude:** Updating the explainer first — it still describes the old scope, and saving a version that contradicts your decisions would be worse than not saving it.

**Claude:**   _[tool: Bash — python3 -c " s=open('halle-widget-explainer.html').read() i=s.index('<div class=\"page\">') open('_h]_

**Claude:**   _[tool: Bash — cat > _body.html << 'HTMLEOF' <div class="page">    <header class="masthead">     <p class="kicker">]_

**Claude:**   _[tool: Bash — grep -n "page looked fine\|Page 4 of 8\|Resend\|Postmark\|coverage grid" halle-widget-explainer.html]_

**Claude:**   _[tool: Bash — python3 - << 'PYEOF' p = 'PROJECT-INDEX.md' s = open(p).read()  old = "| `market-feature-ledger.md` ]_

**Claude:**   _[tool: Projects — claude/widget-explainer-plain-words.html]_

**Claude:**   _[tool: Projects — claude/PROJECT-INDEX.md]_

**Claude:**   _[tool: SendUserFile]_

**Claude:** All saved to the project. Six documents now:

| Document | What it is |
|---|---|
| **PROJECT-INDEX.md** | Updated — your four decisions are in the settled table, old claims corrected, and the open items rewritten |
| **scope-decisions.md** | Your four decisions, every side-effect, and exactly what changes in the build |
| **market-feature-ledger.md** | The market scan — 18 tools, what we have, what we skip, where the whole category is weak |
| **research-raw-feedback-widgets.md** | Full evidence: 5 widgets, ~70 features, every source link |
| **research-raw-uat-platforms.md** | Full evidence: 13 testing platforms against 10 things that matter |
| **widget-explainer-plain-words.html** | The plain-words explainer, rewritten to match your new scope |

The explainer is attached above too, so you can open it now.

Two things worth flagging before you close this:

**BUILD-SPEC.md is now out of date in four places.** I left it alone rather than rewriting it, because it's a long document and I didn't want to change the build instructions without you seeing what changed first. Right now whoever builds this has to read BUILD-SPEC *and* scope-decisions together and know that the second one wins. Cleaner to fold them into one document — say the word and I'll do it.

**The language switcher is still the one thing blocking work.** Nobody can build the German/English switching until we know how the Halle site does it. Ten minutes to check.

**Vishnu:** ok now i need to prepare a one page to sned to the clinet whey we have shifted from the first plan teh form to htis in simple words her is not a tch guy in on epage just give here i will copy past

**Claude:** Here it is — ready to copy.

---

## Why we changed the feedback form

*B. Halle website testing · araCreate · September 2026*

### What we planned first

Testers would find a problem, then sort it into a category. The form had 26 sections and 227 tick-boxes.

### What you told us

The testers would not understand the words. You were right — and when we looked into it properly, we found the wording was the smaller half of the problem.

### What we found

The real problem was the job itself. We were asking testers to do something that trained professionals cannot do reliably.

- Trained usability experts watched the same recordings and agreed on only **5–65%** of what the problem actually was. If experts cannot agree on which box a problem belongs in, a volunteer in their seventies cannot either.
- We checked whether a better-built form would fix this. In one study, researchers gave everyone the same structured reporting format. Agreement went from **30.9% to 30.9%** — no change at all. The only thing that improved it was using **fewer and broader** categories (31% → 57%).
- Older adults find it harder to choose from a list specifically when the options **look similar to each other**. With 227 boxes and overlapping wording, one single problem would have been ticked four times.
- Older adults are most comfortable with about **five** choices. Younger adults are comfortable with about ten.

Put together: the long form would have produced a lot of data we could not trust.

### What we do instead

The tester points at the thing that looked wrong, and answers one question with five plain sentences:

> The writing was too small or too faint to read
> I could not find what I was looking for
> Something looked broken or out of place
> I clicked something and it did not work
> I did not understand the words

These are five different **activities** — reading, finding, looking, doing, understanding — so they do not overlap. Afterwards they can type a sentence if they want to. That is the whole thing.

The first thing they read is:

> **We are testing this website — not you.**
> If something looked wrong, that is the website's fault, not yours.

A study of 1,204 people found that the difference in how older people use technology comes from **confidence and anxiety**, not from age. Those two sentences will do more for the results than any feature we could build.

### What you do not lose

**Your categories are unchanged.** The sorting still happens — our team does it, working from what the tester reported. Your final report looks the same as planned.

You also get more than a tick-box could ever give: **the exact element the tester pointed at**, its text, an automatic screenshot, and the page and device they were using.

### Three things we chose not to do

- **No severity ratings.** Experts disagree with each other 24–30% of the time on severity, and severity does not predict how often a problem actually happens.
- **No "why do you think this happened".** People give confident answers that are invented, and we cannot tell those apart from the real ones.
- **No screen recording.** The browser shows a permission pop-up that cannot be removed or restyled. For this group of testers, that is where the session ends.

### In one line

We made the tester's job smaller so that the answers are worth having, and moved the sorting to our side.

---

Saved a copy to the project as `claude/client-note-why-we-changed-the-form.md` so you can find it again.

Want it as a Word file or PDF to attach instead of pasting into an email?

**Vishnu:** I WILL UPLODE THE FIRST PROTOTGYPE WAIT

**Vishnu:** SL

**Claude:** Waiting.

One correction while you get it: I said I'd saved the client note to the project but hadn't actually done it. Doing that now.

**Claude:**   _[tool: Bash — cat > /home/claude/client-note.md << 'MDEOF' # Why we changed the feedback form  *Client-facing note]_

**Claude:**   _[tool: Projects — claude/client-note-why-we-changed-the-form.md]_

**Claude:** Saved properly now — `claude/client-note-why-we-changed-the-form.md`.

Your "SL" came through as just those two letters, so I'll leave it unless you meant something by it.

Go ahead and upload the prototype whenever you're ready.

**Vishnu:** so this was the first plan now we move to the pointer we need to explain him

**Claude:** Now I can see it properly — and this makes the explanation much easier, because I can point at the actual screens instead of talking in the abstract.

Here's the note, rewritten around what Jakob has already seen:

---

## Why we are changing the feedback tool

*B. Halle website testing · araCreate · September 2026*

The first version is well built. The problem is not the design — it is what the design asks the tester to do.

### What the first version asks of a tester

Three screens, and four decisions, before anything is recorded:

1. **"Which section of the page has the problem?"** — choose one from a list of nine, which is longer than the window, so they must scroll to see the rest.
2. **"What's the issue in Product Details?"** — tick as many as apply from *Filter not working, Wrong material listed, Missing products, Pagination broken*.
3. **"Would you like to share a screenshot?"** — Yes or No.
4. **Submit.**

### Where this breaks for our testers

**They cannot name the section.** *Hero & Product Image*, *Range Specifications*, *Retardation Graph*, *General Specifications* — these are our names for parts of the page. The tester sees a picture, a table, a graph. There is no way for them to know which of our names belongs to which part of what they are looking at. And they have to scroll a list to find out.

**They cannot diagnose the cause.** *Filter not working* and *Pagination broken* ask the tester to tell us **what went wrong technically**. They know something did not work. They do not know whether the filter is broken or whether they used it wrongly. Most people in that position guess — or close the box.

**"Select all that apply" makes it worse.** When the options overlap, one single problem gets ticked three or four times. In the data that looks like three or four problems.

**The screenshot is a question, so most will answer no.** We then have a complaint with no picture attached.

We also checked whether a better-organised form would fix this. In one study researchers gave every evaluator the same structured reporting format, and agreement went from **30.9% to 30.9%** — no change at all. Trained usability experts watching identical recordings agree on only **5–65%** of what the problem was. If professionals cannot agree on which box a problem belongs in, our testers cannot either.

### What we do instead

The tester **points at the thing**. No naming, no scrolling, no list.

The page draws a box around whatever is under their mouse. They click it. We record exactly which part of the page it was **and the words that were in it** — so we get *"the price row on the Glan-Thompson page"*, which is far more precise than *"Product Details"*.

Then one question, five plain sentences, one tap:

> The writing was too small or too faint to read
> I could not find what I was looking for
> Something looked broken or out of place
> I clicked something and it did not work
> I did not understand the words

Afterwards they may type a sentence, or skip it. The screenshot is taken automatically — we show it to them and they can say "don't include it".

**One tap instead of four decisions — and it tells us more, not less.**

### Two practical gains

**Your categories are unchanged.** The sorting still happens; our team does it from what the tester reported. Your final report looks the same as planned.

**It works on all 49 pages with no extra work.** The first version needs its own section list for every kind of page — that was 26 sections and 227 tick-boxes to write and keep updated. The pointer needs none of it, and it will work on any page you add later.

### Three things we chose not to ask

- **No severity ratings.** Experts disagree with each other 24–30% of the time on severity, and it does not predict how often a problem actually happens.
- **No "why do you think this happened".** People give confident answers that are invented, and we cannot tell those from the real ones.
- **No screen recording.** The browser shows a permission pop-up that cannot be removed. For these testers, that is where the session ends.

---

Saving this over the previous version now.

**Claude:**   _[tool: Bash — cat > /home/claude/client-note.md << 'MDEOF' # Why we are changing the feedback tool  *Client-facing]_

**Claude:**   _[tool: Bash — mkdir -p /home/claude/prototype-v1 && cp /root/.claude/uploads/2b61ea88-cb3e-5d01-95a0-adfe085498ec/]_

**Claude:**   _[tool: Projects — claude/client-note-why-we-changed-the-form.md]_

**Claude:** Saved as v2, replacing the old one. I also added a short **internal-only** section at the bottom of the saved copy — things worth knowing but that would read as criticism if you sent them:

- The launcher says "Feedback & **Bug** Report" — jargon for this audience, and yellow-on-white almost certainly fails contrast.
- Step 1's list is already **taller than its box** — the ninth item is cut off mid-row. With 26 sections across all the page types, that gets much worse.
- The prototype is Figma Sites, so it shows the *flow* but none of the hard parts — pointing at elements, screenshot capture, staying separate from Webflow's styling, the tester token.
- The biggest data loss: it records **which named section**, never **which element**. "Product Details" versus "the price row on the Glan-Thompson page".

That internal part is clearly marked "do not send" in the file.

I'd also suggest one thing for the email: **paste your four screenshots into it**, in order, above the note. Jakob will follow "three screens, four decisions" much faster if he's looking at them.

Want me to build a side-by-side page — his four screens on the left, our flow on the right — so he can see the difference rather than read about it?

**Vishnu:** make this small and no need as document

**Claude:** ---

Hi Jakob,

A short note on why we are changing the feedback tool.

The first version asks the tester to do two things they cannot do. First, name the section — *Hero & Product Image*, *Range Specifications*, *Retardation Graph*. Those are our names for parts of the page; the tester just sees a picture, a table, a graph. Second, say what is technically wrong — *filter not working*, *pagination broken*. They know something did not work. They do not know why, and nor should they.

This is not a wording problem. Trained usability experts watching the same recording agree on only 5–65% of what the problem was. If professionals cannot sort problems into boxes reliably, our testers certainly cannot.

So we changed the job. The tester **points at the thing that looked wrong**, and picks one of five plain sentences — one tap, no lists, no scrolling.

We get more from it, not less: the exact element and its text, rather than a section name. Your categories are unchanged — our team does the sorting. And it works on all 49 pages with no per-page setup.

---

About 180 words. Paste your four screenshots above it and it'll be obvious.

**Vishnu:** no no this such mall we need to epxlin him jut cut of some unwnated thisng form this 


Why we are changing the feedback tool
B. Halle website testing · araCreate · September 2026
The first version is well built. The problem is not the design — it is what the design asks the tester to do.
What the first version asks of a tester
Three screens, and four decisions, before anything is recorded:

1. "Which section of the page has the problem?" — choose one from a list of nine, which is longer than the window, so they must scroll to see the rest.
2. "What's the issue in Product Details?" — tick as many as apply from Filter not working, Wrong material listed, Missing products, Pagination broken.
3. "Would you like to share a screenshot?" — Yes or No.
4. Submit.

Where this breaks for our testers
They cannot name the section. Hero & Product Image, Range Specifications, Retardation Graph, General Specifications — these are our names for parts of the page. The tester sees a picture, a table, a graph. There is no way for them to know which of our names belongs to which part of what they are looking at. And they have to scroll a list to find out.
They cannot diagnose the cause. Filter not working and Pagination broken ask the tester to tell us what went wrong technically. They know something did not work. They do not know whether the filter is broken or whether they used it wrongly. Most people in that position guess — or close the box.
"Select all that apply" makes it worse. When the options overlap, one single problem gets ticked three or four times. In the data that looks like three or four problems.
The screenshot is a question, so most will answer no. We then have a complaint with no picture attached.
We also checked whether a better-organised form would fix this. In one study researchers gave every evaluator the same structured reporting format, and agreement went from 30.9% to 30.9% — no change at all. Trained usability experts watching identical recordings agree on only 5–65% of what the problem was. If professionals cannot agree on which box a problem belongs in, our testers cannot either.
What we do instead
The tester points at the thing. No naming, no scrolling, no list.
The page draws a box around whatever is under their mouse. They click it. We record exactly which part of the page it was and the words that were in it — so we get "the price row on the Glan-Thompson page", which is far more precise than "Product Details".
Then one question, five plain sentences, one tap:
The writing was too small or too faint to read
I could not find what I was looking for
Something looked broken or out of place
I clicked something and it did not work
I did not understand the words
Afterwards they may type a sentence, or skip it. The screenshot is taken automatically — we show it to them and they can say "don't include it".
One tap instead of four decisions — and it tells us more, not less.
Two practical gains
Your categories are unchanged. The sorting still happens; our team does it from what the tester reported. Your final report looks the same as planned.
It works on all 49 pages with no extra work. The first version needs its own section list for every kind of page — that was 26 sections and 227 tick-boxes to write and keep updated. The pointer needs none of it, and it will work on any page you add later.
Three things we chose not to ask

* No severity ratings. Experts disagree with each other 24–30% of the time on severity, and it does not predict how often a problem actually happens.
* No "why do you think this happened". People give confident answers that are invented, and we cannot tell those from the real ones.
* No screen recording. The browser shows a permission pop-up that cannot be removed. For these testers, that is where the session ends.


and add like only we came to know whner we shoed the first veriosn the contner porobmle how users will undersntnat

**Claude:** ---

## Why we are changing the feedback tool

*B. Halle website testing · araCreate · September 2026*

The first version is well built. Showing it is what taught us the problem — and the problem is not the design, it is what the design asks the tester to do.

### How we found this out

We only saw it once the first version was in front of people. The wording was the first thing to surface: the testers would not understand the words. When we looked into why, we found the wording was the smaller half of the problem.

### What the first version asks of a tester

Three screens and four decisions before anything is recorded:

1. **"Which section of the page has the problem?"** — one from a list of nine, longer than the window, so they must scroll to see the rest.
2. **"What's the issue in Product Details?"** — tick as many as apply from *Filter not working, Wrong material listed, Missing products, Pagination broken*.
3. **"Would you like to share a screenshot?"** — Yes or No.
4. **Submit.**

### Where this breaks for our testers

**They cannot name the section.** *Hero & Product Image*, *Range Specifications*, *Retardation Graph* — these are our names for parts of the page. The tester sees a picture, a table, a graph, and has no way to know which of our names belongs to which.

**They cannot diagnose the cause.** *Filter not working* and *Pagination broken* ask the tester what went wrong technically. They know something did not work. They do not know whether the filter is broken or whether they used it wrongly. Most people in that position guess, or close the box.

**"Select all that apply" makes it worse.** When the options overlap, one problem gets ticked three or four times. In the data that looks like three or four problems.

**The screenshot is a question, so most will answer no** — leaving us a complaint with no picture.

### Better wording alone would not have fixed it

In one study, researchers gave every evaluator the same structured reporting format. Agreement went from **30.9% to 30.9%** — no change at all. And trained usability experts watching identical recordings agree on only **5–65%** of what the problem was. If professionals cannot reliably sort problems into boxes, our testers cannot either.

### What we do instead

The tester **points at the thing**. No naming, no scrolling, no list.

The page draws a box around whatever is under their mouse. They click it, and we record exactly which part of the page it was **and the words in it** — so we get *"the price row on the Glan-Thompson page"* instead of *"Product Details"*.

Then one question, five plain sentences, one tap:

> The writing was too small or too faint to read
> I could not find what I was looking for
> Something looked broken or out of place
> I clicked something and it did not work
> I did not understand the words

Afterwards they can type a sentence, or skip it. The screenshot is taken automatically, and we show it to them so they can say "don't include it".

**One tap instead of four decisions — and it tells us more, not less.**

### Two practical gains

**Your categories are unchanged.** The sorting still happens; our team does it from what the tester reported. Your final report looks the same as planned.

**It works on all 49 pages with no extra work.** The first version needs its own section list for every kind of page — 26 sections and 227 tick-boxes to write and keep updated. The pointer needs none of that, and it will work on any page you add later.

### Three things we chose not to ask

- **Severity ratings** — experts disagree with each other 24–30% of the time, and severity does not predict how often a problem happens.
- **"Why do you think this happened"** — people give confident answers that are invented, and we cannot tell those from the real ones.
- **Screen recording** — the browser shows a permission pop-up that cannot be removed. For these testers, that is where the session ends.

---

Updated the saved copy in the project too.

**Vishnu:** dont use ths — make and make it humanied

**Claude:** ---

## Why we are changing the feedback tool

B. Halle website testing, araCreate, September 2026

The first version was built well. What we learned from putting it in front of people is that the difficulty is not in how it looks, but in what it asks the tester to do.

### What we learned from the first version

We only really saw the problem once people were sitting in front of it. The wording came up first: the testers would not understand the words. That was the right thing to notice, and when we went looking into why, we found the wording was only part of it.

### What the first version asks of a tester

Before anything is recorded, the tester goes through three screens and makes four decisions.

First, "which section of the page has the problem?" They pick one from a list of nine, and the list is taller than the box it sits in, so they have to scroll to see the rest.

Then, "what's the issue in Product Details?" They tick as many as apply from Filter not working, Wrong material listed, Missing products, Pagination broken.

Then, "would you like to share a screenshot?" Yes or no.

Then submit.

### Where it breaks down

The section names are ours, not theirs. Hero & Product Image, Range Specifications, Retardation Graph. Someone looking at the page sees a picture, a table, and a graph. They have no way of matching our names to what is in front of them, and asking them to scroll through a list to work it out makes it harder rather than easier.

The second question is harder still, because it asks them to say what went wrong technically. Someone can tell you that the filter did nothing when they clicked it. They cannot tell you whether the filter is broken or whether they used it the wrong way, and it isn't reasonable to expect them to. In practice, people in that position either guess or close the box.

Allowing several ticks at once makes this worse rather than better. When the choices overlap, one problem ends up ticked three or four times, and afterwards it reads like three or four separate problems.

The screenshot being optional sounds polite, but most people will click "no, skip", and then we are left with a complaint and no picture of it.

### Better wording on its own would not have solved it

We checked. In one study, researchers gave every evaluator the same structured reporting format to work from, and the level of agreement between them went from 30.9% to 30.9%. It made no difference at all. Separately, trained usability experts watching the same recordings agree on only 5 to 65% of what the problem actually was. If people who do this professionally cannot sort problems into categories reliably, we should not be asking our testers to.

### What we are doing instead

The tester points at the thing that looked wrong. There is nothing to name and nothing to scroll through.

As they move the mouse, the page draws a box around whatever is underneath it. They click, and we record which part of the page it was along with the words that were in it. So instead of "Product Details" we get "the price row on the Glan-Thompson page".

After that there is one question with five plain sentences, and picking one is all it takes:

> The writing was too small or too faint to read
> I could not find what I was looking for
> Something looked broken or out of place
> I clicked something and it did not work
> I did not understand the words

They can add a sentence of their own if they want to, or skip it. The screenshot happens on its own, and we show it to them first in case they would rather it wasn't included.

So it is one tap instead of four decisions, and we come away knowing more about the problem rather than less.

### Two things worth mentioning

Your categories have not changed. The sorting still happens, we just do it on our side using what the tester reported, so the report you receive at the end looks the same as planned.

It also works across all 49 pages without extra setup. The first version needs its own list of sections for each kind of page, which came to 26 sections and 227 tick boxes to write and then keep up to date as the site changes. The pointer approach needs none of that, and it will work on any page you add later.

### A few things we decided not to ask

We are not asking testers to rate how serious a problem is. Experts disagree with each other about severity roughly a quarter to a third of the time, and how serious something seems turns out not to predict how often it actually happens.

We are not asking why they think it happened either. People answer that question confidently even when they are guessing, and there is no way for us to tell the guesses apart from the real explanations.

And we are not recording their screen. The browser puts up a permission pop-up every time, we cannot remove it or restyle it, and for this group of testers that pop-up is where the session ends.

**Vishnu:** in htis verion lot of text

**Claude:** ---

## Why we are changing the feedback tool

B. Halle website testing, araCreate, September 2026

The first version was built well. What we learned from putting it in front of people is that the difficulty is not how it looks, but what it asks the tester to do.

### What we learned

The wording came up first: the testers would not understand the words. That was the right thing to notice. When we looked into why, we found the wording was only part of it.

### What the first version asks

Three screens and four decisions before anything is recorded. Pick the section from a list of nine that has to be scrolled. Tick what kind of problem it is. Say whether you want to attach a screenshot. Submit.

Two of those are things our testers cannot really do.

The section names are ours, not theirs. Hero & Product Image, Range Specifications, Retardation Graph. Someone looking at the page sees a picture, a table and a graph, and has no way of matching our names to what is in front of them.

The second question asks what went wrong technically. Someone can tell you the filter did nothing when they clicked it. They cannot tell you whether it is broken or whether they used it the wrong way, and it isn't fair to ask. In practice people guess, or they close the box.

Better wording would not have fixed this. Trained usability experts watching the same recordings agree on only 5 to 65% of what the problem actually was. If people who do this professionally cannot sort problems into categories reliably, we should not be asking our testers to.

### What we are doing instead

The tester points at the thing that looked wrong. As they move the mouse, the page draws a box around whatever is underneath it. They click, and we record which part of the page it was along with the words in it, so we get "the price row on the Glan-Thompson page" rather than "Product Details".

Then one question with five plain sentences, and picking one is all it takes:

> The writing was too small or too faint to read
> I could not find what I was looking for
> Something looked broken or out of place
> I clicked something and it did not work
> I did not understand the words

They can add a sentence of their own if they want to, or skip it. The screenshot happens on its own.

One tap instead of four decisions, and we come away knowing more about the problem rather than less.

Your categories have not changed. The sorting still happens, we just do it on our side from what the tester reported. It also works across all 49 pages without extra setup, where the first version needed its own list of sections for every kind of page.

---

Roughly 400 words. Say the word and I'll save this version over the one in the project.

**Vishnu:** Then one question with five plain sentences, and picking one is all it takes:
The writing was too small or too faint to read
I could not find what I was looking for
Something looked broken or out of place
I clicked something and it did not work
I did not understand the words
They can add a sentence of their own if they want to, or skip it. The screenshot happens on its own.
One tap instead of four decisions, and we come away knowing more about the problem rather than less.
Your categories have not changed. The sorting still happens, we just do it on our side from what the tester reported. It also works across all 49 pages without extra setup, where the first version needed its own list of sections for every kind of page.

THIS IS PART WE NEED TO CHANGE 

IN FIRST VERIOSNA LOS WE CAN TAKE THE PAGE THE WE RECOD THAT SILENTY

**Claude:** Here's that closing part rewritten around your point:

---

Then one question with five plain sentences, and picking one is all it takes:

> The writing was too small or too faint to read
> I could not find what I was looking for
> Something looked broken or out of place
> I clicked something and it did not work
> I did not understand the words

They can add a sentence of their own if they want to, or skip it. The screenshot happens on its own.

The bigger change is in what we stopped asking. The software already knows which page the tester is on, and now it also knows exactly which part of that page they pointed at. Both get recorded quietly in the background. The first version was asking the tester to tell us things we could see perfectly well for ourselves, and to do it using our names for them.

So the tester answers one question instead of four, and we come away with more detail rather than less. There is also no list of sections to write and keep updated for each kind of page.

Your categories have not changed. The sorting still happens, we just do it on our side from what the tester reported.

---

I read your note as: the page was always something we could capture by ourselves and record silently, so asking the tester which section was never needed. If you meant something else, tell me and I'll redo it.

**Vishnu:** NO WE NEED TO TELL THE CLINET IN THE NEW IDEWA WE ILL BE CAPTURIN HT EXACT POINT AND EXACT ELEMT AND ALOS WE WILL LET USERS TELL WHAT IS WRING INTED OF GIVE THE LSIT ANS ASKING THWM TO CHOOSE

**Claude:** Got it. Here's that part:

---

Two things change in the new version.

**First, we capture the exact spot ourselves.** When the tester clicks, we record the precise position on the page and the exact element they pointed at, along with the words inside it. Nobody has to name anything or find it in a list. Instead of "Product Details" we get "the price row on the Glan-Thompson page".

**Second, we stop asking them to work out what is wrong.** The old version gave them a list of technical faults and asked them to pick the right one. The new one simply asks them to say what happened to them, in ordinary words:

> The writing was too small or too faint to read
> I could not find what I was looking for
> Something looked broken or out of place
> I clicked something and it did not work
> I did not understand the words

These are things a person can honestly say about their own experience, rather than judgements about what is broken underneath. And if none of them fits, there is a box where they describe it in their own words instead.

So the tester answers one question rather than four, and we come away with more detail rather than less. Your categories have not changed either. The sorting still happens, we just do it on our side from what the tester reported.

---

One thing worth being careful about: we do still show five options, so I have not said "no list". What changed is what the list asks of them. If Jakob sees the widget later and it has five choices in it, this wording still holds up.

**Vishnu:** save all memory

**Claude:**   _[tool: Bash — cat > /home/claude/client-note.md << 'MDEOF' # Why we are changing the feedback tool  *Client-facing]_

**Claude:**   _[tool: Projects — claude/client-note-why-we-changed-the-form.md]_

**Claude:**   _[tool: Projects — claude/prototype-v1-00-launcher.png]_

**Claude:** PNGs can't go into project docs — text only. Writing the prototype down instead so the detail survives without the images.

**Claude:**   _[tool: Bash — cat > /home/claude/prototype-v1-record.md << 'MDEOF' # Prototype v1 — full record  **What this is:**]_

**Claude:**   _[tool: Projects — claude/prototype-v1-record.md]_

**Claude:**   _[tool: Bash — python3 - << 'PYEOF' p = 'PROJECT-INDEX.md' s = open(p).read()  old = "| `widget-explainer-plain-wor]_

**Claude:**   _[tool: Projects — claude/PROJECT-INDEX.md]_

**Claude:** Everything's in the project. Eight documents now hold the whole picture:

**New this round**

- **client-note-why-we-changed-the-form.md** — the final approved wording, with the rules noted at the top (no em dashes, plain sentences, short) so nobody rewrites it wrong later. At the bottom, marked *do not send*, I kept the arguments we deliberately left out — the 30.9% study, the 4.8-options finding, why no severity ratings, why no screen recording. If Jakob pushes back, they're there.
- **prototype-v1-record.md** — every screen and label from your prototype written out, including the nine section names, the four checkboxes, and what's wrong with each. Also what was worth keeping from it: the step indicator, the modal placement, the navy header.

**Updated**

- **PROJECT-INDEX.md** — points at both new docs, and records that prototype v1 existed and was shown to the client, since that's now part of the story of how we got here.

One thing to know: **the project only accepts text, so the four screenshots could not be saved there.** That's why I transcribed them instead. Keep the image files somewhere yourself — they're the most persuasive part of the email to Jakob.
