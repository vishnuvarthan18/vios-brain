# Deep research brief — the execution plan (not the landscape)

For Vishnu to run in a separate Claude Cowork session. **Purpose: everything so far (three rounds, one AI) has researched the past — what exists, how similar projects worked, why they lived or died. This round researches the future — the specific decisions and steps needed to actually start building, over the next 12–24 months, for this exact reserve.** Do not re-survey comparable projects. Do not re-litigate funding models or lifecycle stages — those are settled. This is a planning and decision brief, not a discovery brief.

## Depth and format requirements — read before starting

- This needs to be **decision-grade, not descriptive**. Every task below should end in a specific recommendation, a specific number, or a specific document — not a survey of options.
- Where a decision depends on something unknown (e.g. a reply from GBIF or IFP that hasn't arrived yet), give the **default plan for each possible answer**, not just "wait and see."
- **Verify against primary sources** wherever a claim is checkable (government fee schedules, hosting prices, domain registration costs, software licence terms) — real numbers, not estimates.
- Deliver as a single report, one section per task below, each with its own sources list.

## Context to paste in

Three prior research rounds (all from one AI session, now analysed and filed) have already established: the project's shape (solo/near-solo, aspiring to small-lab or community-governed — the most durable archetype in the corpus); the recommended stack (GeoNature for occurrence/observation, Mukurtu for community/tribal knowledge with cultural-protocol access control, Linked Places Format for the gazetteer, a boring CMS like Drupal for legal instruments and documents); a six-primitive disputed-figures data schema (assertion-as-record, rank not delete, mandatory method field, certainty value, superseded-versions-stay-live, whole-resource versioning); seven demonstrated figure-disputes in the reserve's own public record to build the feature around (area, tiger population, elephant corridors, Western Ghats ESA boundary, sacred groves, and two national-scale ones from WII's own site); a recommended build sequence (scope audit → gazetteer with DOI → legal-instrument layer → disputed-figures layer → historical documents → occurrence aggregation last); and five outreach documents already drafted (IFP, Tamil Nadu Biodiversity Board RTI, GBIF, Keystone Foundation, FES, Forest Department RTI).

None of that needs re-researching. What's missing is everything about actually **doing it**, starting now, as one person in Erode, Tamil Nadu, with no institutional backing yet and no confirmed funding.

## What I need, strictly in this order

### Task 1 — A concrete 90-day plan, week by week
Not a phase table like the prior report's — an actual week-by-week task list for the first 90 days, assuming the outreach emails have been sent and responses are pending. What can be done in parallel with waiting for replies? What is the very first technical artifact to produce (the scope statement + gap audit was recommended — what does that document actually need to contain, and can you draft a template for it)? What's the realistic very-first thing to publish that is "citable by someone else"?

### Task 2 — Real cost of the recommended stack, for this specific build
GeoNature, Mukurtu, and a Drupal instance were recommended, but nobody in the prior research could find published pricing. Find or estimate:
- Realistic hosting cost in India (or wherever cheapest/most reliable) for a small GeoNature + Mukurtu + Drupal deployment at launch scale (a few thousand records, a few hundred documents) — actual numbers from a cloud provider or a shared-hosting comparison, not "it depends."
- Domain registration strategy — given the corpus's repeated lesson that domain loss (GB1900, Arctic Bay Atlas, CRBio) is a leading death mode, what is the actual best practice for registration length, registrar choice, and renewal safeguards for a solo-run project?
- Whether any of GeoNature, Mukurtu or a comparable Drupal setup can realistically be self-hosted on free tiers (e.g. Oracle Cloud Free Tier, a university partnership, GitHub Pages for static parts) at launch scale, and what breaks first as the project grows.
- A realistic estimate of the time cost to stand up all three, for one person with general web/software competence but no prior GeoNature/Mukurtu experience — hours, not hand-waving.

### Task 3 — The legal and consent groundwork that has to happen before any tribal-knowledge data is touched
Prior research flagged this as a live legal matter, not a technical one, but didn't produce an actual action plan. Research and produce:
- What specific permissions, consultations, or agreements are legally or ethically required before any Soliga or Oorali (Irula) community knowledge — including anything sourced from a People's Biodiversity Register — is included in the atlas, even in a protocol-gated form like Mukurtu's.
- Whether Tamil Nadu or India has any existing legal framework, precedent case, or model consent agreement for this kind of digitisation project that could be adapted, rather than drafted from scratch.
- A realistic assessment of how long this groundwork typically takes before a project can proceed, based on any comparable Indian or global precedent found.

### Task 4 — Positioning and the first real audience
Given the project has no institution, no funding, and no team yet:
- Who is actually the first realistic audience or partner for a working prototype — a specific department, journalist, researcher, or organisation in Tamil Nadu or nationally — not a generic "conservationists" answer?
- What would make the Tamil Nadu Forest Department, NTCA, or a state university take this seriously enough to have a conversation, based on how comparable small projects (Kerala Bird Atlas, Goa's bird atlas, Land Conflict Watch) actually got their first institutional foothold?
- Is there a specific grant, fellowship, or competition (Indian or international) with a near-term deadline that this project could realistically apply to at prototype stage, given it has no track record yet?

### Task 5 — What could kill this specific project in year one, and the mitigation for each
Not the generic lifecycle research already done — a specific pre-mortem for this project, this person, this timeline. What are the three or four most likely reasons this particular attempt stalls in its first 12 months, and what is the specific, concrete thing to do now to reduce each risk?

## Deliverable format

One report, Tasks 1–5 in order, each with a sources list. Task 1 must produce an actual week-by-week list, not a description of a plan. Every recommendation should be specific enough to act on immediately without further research.

## What happens after

This comes back to the same project for reconciliation against the existing plan documents and the reviewer's bug list.
