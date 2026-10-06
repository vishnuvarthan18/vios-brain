# Sathyamangalam Atlas — Locked Scope (6 Sep 2026)

This replaces informal understanding with a single reference. Every future build, prompt, and coding-agent instruction should be checked against this document. If a change conflicts with this, stop and re-confirm with Vishnu before proceeding.

## The one-sentence aim

A complete, detailed reference for everything in the Sathyamangalam landscape — every plant, tree, bird, mammal, insect, snake/reptile, and every hill, river, village and place — each with a full record, not just a count, sourced and honestly labeled where data is thin or missing.

## What "done" means for one record — the standard every category must meet

**Not this:** "2,664 species in the database" or "93 places found."
**This:** every individual plant/animal/place has its own page, and that page is filled in as completely as real data allows.

## Content model — what a complete record contains, per category

### Plants & trees
- Basic facts: common name (English + Tamil), scientific name, family, description
- Forest type / zone it belongs to (of the 5 Champion & Seth zones in the management plan: dry thorn, dry mixed deciduous, sub-tropical hill, semi-evergreen, riparian)
- Flowering/fruiting season
- Traditional/local use — medicinal, edible, sacred, or tribal use (from the management plan's herb/tree appendices and any already-documented ethnobotanical sources — no new fieldwork required to start)
- Threats — clearing, invasive-species pressure (esp. Senna spectabilis displacing native flora), over-harvesting
- Where it's been recorded (locations)
- Photos, ideally from this reserve (iNaturalist CC-licensed)
- Sources for every fact
- Confidence/completeness label (see below)

### Animals — mammals, birds, insects, snakes/reptiles
- Basic facts: common name (English + Tamil), scientific name, family, conservation status (IUCN, WPA schedule), description
- Where it's actually been recorded here — real sighting locations and dates, not "present in India"
- Population numbers over time — not a single static count. Track the trend with sources for each data point (example already in hand: tigers 8-10 in 2009 → 112 in 2024, elephants 350-450 in 2009 → 651 recently)
- Photos, ideally from this reserve
- Relationships to other species — what it eats, what eats it, which habitats/plants it depends on. Build toward a real ecosystem web, not isolated entries
- Threats specific to it, named and sourced — not generic. (Poaching for tigers/sandalwood-linked species, electrocution and corridor loss for elephants, vulture population collapse, roadkill, etc.)
- Behavior/season notes — migratory patterns, breeding season, nocturnal/diurnal
- Local/tribal names — Sholaga/Irula/Kurumba names, wherever already documented in existing sources (do not require new fieldwork to populate this field; leave blank with a note if nothing documented yet)
- Sources for every fact
- Confidence/completeness label

### Places — hills, rivers, villages, streams, temples, etc.
- Basic facts: name (English + Tamil + historical spelling variants), type (village/hill/river/temple/etc.), coordinates
- What's been recorded there — every species sighting, document, or historical passage linked to this specific location. Every place becomes a hub, not just a dot on a map
- History of the name — old spellings, name changes over time, colonial-era mentions (the Sittimungulum/Sathyamangalam variant work already done is the model)
- Administrative details — forest range/beat/division, village population, distance to nearest town, census data where it exists
- Elevation and terrain, nearby water sources
- Sources for every fact
- Confidence/completeness label

## Honesty about gaps — a standing design rule, not optional

Every record must show how complete/confident its data is, not silently present blanks as if they were checked-and-empty. Examples: "last verified 2010," "only 1 sighting recorded," "no photo available yet," "no local name documented." This matches the project's original design principle (the `claim` table for disputed figures) and extends it to completeness generally — a page should never imply more certainty than the data supports.

## Organization

Species are split by type into separate browsable sections: Birds, Mammals, Insects, Snakes/Reptiles, Trees, Plants (and any other natural category the data supports — fungi, fish, amphibians appeared in earlier counts and need their own section too). Not one mixed list.

## Explicitly OUT of this scope (for now — separate future decision, not part of this atlas)

- Community/cultural fieldwork of the Keystone Foundation Archives kind (oral history, ethnobotany collected first-hand, community radio, village relationship programs). This atlas only uses ethnobotanical/cultural facts that are *already documented* in an existing source (management plan, papers, gazetteers) — it does not do new fieldwork or interviews. A cultural archive is a separate, later idea if pursued at all.
- Anything requiring new field visits, physical archive trips (Gass Forest Museum, Tamil Nadu Archives), or the Field Director relationship — those remain tracked in `harvest-plan.md`'s Phase 9 but are not gating this data-completeness work.

## What this changes about current priorities

The recent work (species browser as one flat list, place count going 89→93) was aimed at showing *that data exists*, not building the actual record depth described above. That work isn't wasted — the underlying data and the sync pipeline are real building blocks — but the actual deliverable is per-record detail pages meeting the standard above, not summary counts or flat searchable lists.

## Immediate blockers to this scope, inherited from ongoing work

1. Gazetteer stuck at 93 of 800-2,000 places — most "place hub" pages can't exist yet because most places aren't found.
2. Species aren't split by category in the database/export yet — same table for birds, plants, insects.
3. No occurrence-record page/export exists — the "where it's been seen" requirement has no delivery mechanism yet.
4. Population-over-time tracking has no dedicated structure yet — today's data model has current snapshots, not tracked trends with per-point sourcing (closest existing precedent: the empty `claim` table, designed for exactly this kind of competing/evolving figure).
5. No photo-attachment pipeline exists yet, despite iNaturalist already being a harvested source with CC-licensed photos.
6. Relationship data (what eats what, species-to-habitat links) doesn't exist as a data structure anywhere in the current schema.

## Next step

Turn this into an actual build plan: a concrete database schema change (or additions) that can hold all of the above, then a page-template design that works across all categories, before writing any more collection or display code. This should be scoped as its own planning pass, not squeezed into a quick prompt.
