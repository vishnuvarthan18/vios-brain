# Deep research brief — comparable conservation-atlas projects worldwide

For Vishnu to run in a separate Claude Cowork session (different account). Purpose: understand the landscape before continuing to build the Sathyamangalam Atlas, per reviewer's instruction to research first. Output will be brought back to this project and reconciled here — so the report must stand alone and be clean enough to hand off.

## Depth and format requirements — read before starting

- This needs to be **very deep, not a surface pass** — real sources, real names, real numbers, not generic claims. If something can't be verified, say so explicitly rather than guessing.
- **Break the work into clearly separated tasks, done strictly in order.** Task 1 (the full list) must be done first and completely before Task 2's analysis is attempted — you cannot explain how this category of project works without first having gathered the category. Do not blend tasks into one undifferentiated narrative.
- **Deliver as a single clear, well-structured report** with headers per section and a sources list under each — this report will be handed back as-is to another Claude session for reconciliation, so it must be self-contained and readable without the original conversation.
- Use tables wherever comparing multiple projects/items across attributes (scale, team size, funding model, status, etc.) rather than long prose lists.

## Context to paste in

I'm building a landscape-scale conservation "atlas" for Sathyamangalam Tiger Reserve/Wildlife Sanctuary (Tamil Nadu, India) — combining species occurrence data, a place-name gazetteer, legal/government instruments, historical document mining, and literature into one site, with a feature to surface disputed/conflicting figures (e.g. competing area or population estimates) rather than silently picking one. Already found: Atlas of Living Australia, NBN Atlas (UK), GBIF, Minnesota Biodiversity Atlas, India Biodiversity Portal — don't re-find these, go deeper and wider.

## What I need, strictly in this order

### Task 1 (do first) — A full worldwide classified list
Find every website/project like this that exists, anywhere in the world, for **any** subject in this space — not just tigers or forests. Include forest/landscape documentation, tiger/wildlife-specific projects, bird atlases, indigenous/tribal community documentation projects, marine, botanical, anything that follows this general pattern of "document a place or population comprehensively, from many source types, on one site." Cast a wide net first — the category must be fully gathered before it can be analyzed.

**Organize the list hierarchically: by continent, then country, then state/region where applicable** (not a flat list). Within each, present as a table classifying each project by: scale (national/regional/single-site), team size/type (institution/small team/solo), scope (occurrence-only vs multi-domain like ours), subject (forest/tiger/bird/tribal/marine/other), and whether it's still active.

### Task 2 (only after Task 1 is complete) — How this type of project actually works
Using the full list from Task 1 as the evidence base, explain the underlying model for this category of project: typical funding sources, team structure and size, data pipeline architecture, who maintains them long-term, and their typical lifecycle from launch to maturity or abandonment. This section should draw on and cite specific examples from Task 1's list, not speak generically.

### Task 3 — Deep research on how these projects "win and earn"
What makes a project in this category succeed — adoption, credibility, longevity — and specifically how they sustain themselves financially: grants, government backing, institutional hosting, donations, membership, or other models. Ground-truth examples from the Task 1 list, not generic assumptions.

### Task 4 — Four follow-up angles
- Any project — any domain, not just conservation — that has solved the "disputed facts / conflicting claims side-by-side" UX/data problem well (Wikipedia dispute mechanisms, Our World in Data, fact-check sites, legal case-law databases with conflicting precedent).
- Darwin Core and other biodiversity data standards — is adoption realistic for a project this size, real migration cost, lighter-weight alternatives smaller projects use.
- Realistic ground-truth timelines for solo-builder or 2-3 person teams to reach something publishable, not vendor marketing claims.
- Flag anything found that directly contradicts or validates the current approach.

## Deliverable format

A single written report, clearly sectioned as Task 1 / Task 2 / Task 3 / Task 4 exactly as above, in that order, each with its own sources list. Task 1 must include the full continent → country → state classified comparison table before anything else in the report. No mixing sections together — this needs to be clean enough to hand back for direct reconciliation against the existing bug list without further cleanup.

## What happens after
Vishnu brings the finished report back to this project (or this conversation) so findings get folded into `sathyamangalam/reviewer-findings-and-bug-list-2026-08-25.md` and reconciled against what's already logged there.
