# Plan: get more species, places, and everything else — 9 Sep 2026

Plain-language plan. Not a biology or tech document — this is about *what's missing and why*, and what closes each gap.

## The three biggest gaps, in order of size

### 1. Places — the biggest gap by far
You have 46 places right now. The realistic target, based on what's actually out there (villages, hills, streams, temples, forest checkpoints), is 800–2,000. That's not a small miss — it's the single biggest hole in the whole project.

Why: the tool that pulls place names (OpenStreetMap) works, but it can't be properly matched to real village records because two of the official Indian government data connections were never set up — one needs a signup you have to do yourself (a free government data account), the other needs a different free registration (a protected-areas database). Neither is a coding problem. Both are "Vishnu goes and signs up for something" problems.

**What closes this:** you get those two accounts/keys. That single step is worth more than everything else in this plan combined.

### 2. Species — partially working, but missing depth
1,320 species-level entries is real, but it's mostly just "this animal/plant exists here." What's missing:
- No official conservation-status check (is it endangered, threatened, etc.) hooked up for most entries.
- Some individual data connections that would add more species are broken (one is silently failing, one is paused because it hit a paid limit, one is permanently blocked by the source website itself).

**What closes this:** fixing those individual broken connections (dev work, already identified) plus a decision from you on whether one blocked source is worth paying for.

### 3. History, government records, and news — mixed, several silently broken
Several of the pieces meant to pull in old documents, government notices, and news mentions are running but producing nothing — same "says it's working, isn't" problem as the species gaps above. A few are permanently stuck because the source website itself is down or blocks automated access — nothing to be done there, we just mark them as "not available" instead of pretending.

**What closes this:** dev work fixing the broken ones, and accepting a few sources are permanently out of reach.

## What this means for you, specifically

Only two things on this list require you personally, and they're both just signups, not decisions with tradeoffs:
1. Get a free data.gov.in account (Indian government open data).
2. Get a free Protected Planet / WDPA account (world protected-areas database).

Everything else is dev work I can hand off once you say go.

## Suggested order

1. You get the two accounts — this unlocks the single biggest gap (places) and costs you maybe 20 minutes total.
2. Dev builds the "alarm system" from the last plan first, so we stop finding these gaps by accident.
3. Dev fixes the known broken data connections, species and history included, one group at a time.
4. Re-check the numbers after a few weeks and see how much of the gap actually closed — some of it (oral history, unpublished records, original photos) can never be filled by any tool, no matter what we fix.
