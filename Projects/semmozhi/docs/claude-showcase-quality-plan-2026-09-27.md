# Showcase-quality plan (2026-09-27)

Owner direction: this site is going to an international-level lineup/showcase. Look and feel is the priority.
Use real web VFX and proper 3D/deep models, not CSS tricks pretending to be real. At the same time: STOP adding
more animation. Keep motion minimal. Spend the effort on correctness and depth, not on more moving parts.

This reverses the earlier "more physics, more animation" direction. Physics stays, but its job changes: use it to
compute correct REST geometry (where a ring sits through a hole, how thread wraps, the natural fan angle of open
leaves), not to add more animated flourishes.

## Bugs found by the owner (confirmed in code, first fixes)
1. Copper ring floats beside the plate instead of threading the hole. Root cause: hand-placed CSS percentages,
   z-index puts 100% of the ring in front of the plate (a threaded ring must render partly behind, partly in
   front). Fix: wire the already-built and tested T5b physics scene (copper plate set on its ring, period
   accurate to 0.064% of target) into the actual `.copper-set` component as its rest pose.
2. Coin lettering is flat DOM text with one soft shadow, not engraving. Fix: bake the legend into the coin's
   material/texture with real relief lighting (die-strike look: uneven stroke depth, worn high points, same
   light direction as the metal).
3. Leaf-bundle: closed state uses the tuned real "bundle" and "wood" materials (looks right). The fanned-open
   leaves use a different, older material (`--mat-leaf` flat oval, one drop shadow, no grain, no per-leaf
   variation) built in a different task and never matched to the closed look. Reads as an unrelated object
   pasted in. Fix: fanned leaves use the same tuned palm-leaf material and per-leaf variation as everywhere else.

These three share one root cause: components were built in separate tasks with separate hand-tuned values and
never reconciled against each other. That is a systemic risk, not three isolated bugs — see "no orphan materials"
below.

## New rules for this phase

### 1. Motion: cut to minimal
- Audit every animation on the site (GSAP timelines in motion.js, CSS transitions). Classify each as
  FUNCTIONAL (needed to show a state change the user asked for: open/close a bundle, flip a leaf, expand a card)
  or DECORATIVE (ambient dust drift, hover lifts, page-load intro flourish, scroll parallax, cursor followers).
- Keep only FUNCTIONAL motion, and shorten it. Target under 400ms for any transition, most under 250ms.
- Cut every DECORATIVE animation. State changes happen, they just don't perform on the way there.
- Reduced-motion and keyboard behaviour must still work exactly as before; a smaller minimal-motion set is easier
  to get right, not harder.

### 2. Visual depth: real, not simulated-real
- Where an object is a showcase "hero" (see scope below), push it to real geometry and real PBR material:
  correct thin-leaf bending profile, correct ring-through-hole depth, correct coin relief, correct stone carving
  depth from an actual normal map, lit with one consistent light rig across the whole site.
- Re-open the palm-leaf 3D gate that failed before (fibre direction ~1.7x too strong, letters didn't shadow
  enough) and fix the shader properly, now that the bar is higher. This time the target is not just passing a
  score — it's holding up next to a real photo at showcase size.
- No new decorative VFX for its own sake (no added particle effects, no new camera moves). Depth work goes into
  making the existing hero objects correct, not into adding more visual noise.

### 3. One material system — no orphans
Build (or finish) a single material registry that every component must read from — cards, detail views, open
states, 3D scenes, all of it. Add an automated check that fails the build if any component defines its own
one-off color, texture, or shading instead of referencing the registry. This is what actually prevents the
"looks injected" bug from recurring, rather than patching each instance by hand as it's found.

### 4. Scope: pick the hero set, freeze the rest
Not everything needs showcase-level polish before a lineup. Recommendation (confirm or correct): the hero set is
what will actually be shown — home page, Chola section, Thirukkural leaf reader, and the four already-3D
materials (stone, copper, coins) plus the palm leaf (since it's the most visible object on the home page and the
reader). Everything else — the Tamil Identity gallery (121 icons), pottery/rings/seals, the reference viewer —
stays at its current, tested-but-not-glamorous quality. Effort is not spread thin across all of it before the
lineup.

### 5. Process: reviewed batches, not another blind overnight run
Given the stakes, this phase should NOT be a 30-task autonomous overnight queue like the last one. Each batch
should be small, produce a side-by-side screenshot (real photo vs. render, closed vs. open, before vs. after),
and stop for the owner to look before the next batch. Slower, but no surprise regressions reach the lineup.

## Immediate next batch (Prompt 6)
1. Fix the 3 confirmed bugs (ring, coin lettering, leaf-bundle fan) using the rules above.
2. Build the material registry + orphan check; run it against the whole site once and report every orphan found
   (there will likely be more than the 3 already caught).
3. Motion audit: list every animation, tag functional/decorative, cut the decorative ones, shorten the rest.
4. Re-open the palm-leaf 3D gate with a corrected shader; report the new score against the same targets as
   before, honestly, pass or fail.
5. Stop. Screenshot sheet for owner review before touching anything else.
