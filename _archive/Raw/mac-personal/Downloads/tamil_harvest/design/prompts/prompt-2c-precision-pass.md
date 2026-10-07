# PROMPT 2c — Precision pass on the existing components

Do this together with Prompt 2 (Parts A and B). Goal: the current components must match the real objects closely, not just look "in style". Work in `website_live/css/components.css`, `textures.css`, `materials.css`, `styleguide.html`.

## Method (for every component below)
1. Open the reference photos in `design/references/<surface>/` (study only) and `design/specs/palm_leaf_visual_notes.md`. Pick 3 to 5 real photos per object.
2. Write measured proportions into `DESIGN_SYSTEM.md` under "Precision spec": what you measured, from which file id, and whether it is an estimate. Do not invent numbers; if photos disagree, give a range.
3. Rebuild the component to those numbers. Then make a side-by-side sheet `_checks/precision-<component>.png`: real reference photo (crop, study only, NOT shipped) next to your render, with the differences listed in one line each.
4. Keep every text area readable (contrast rules unchanged).

## Components and what must become precise
- Leaf strip: ratio about 9:1 (real Tirukkural scan 2081x233). Two holes at about 30% and 70% of the length, set in from the ends; hole size, rim and cord shadow measured. Slightly uneven rounded ends, small nicks, colour from ~#C98A3F centre to ~#A86A2C edges. Text: 5 to 7 tight lines of round letters, a small left-margin note, a red catalogue number at one end (on a paper tag). Text stops before the holes. Real leaves are cut clean at both short ends and vary in length; add 3 size variants.
- Leaf bundle: leaves seen edge-on as a dark block (~#4A3426) with fine horizontal lines that differ in tone; wooden boards slightly longer than the leaves; thread wraps (vertical) plus an X crossing; small paper labels. Measure how many wraps and where they sit. Boards show grain and wear.
- Stylus (ezhuthani) and knife: add as components: thin iron point in a wooden handle, with correct proportions.
- Stone slab: rough-cut edge, chisel marks, weathering. Carved letter depth and angle matched to the Tamil-Brahmi cave bed photos (Arittapatti, Mangulam): letters are shallow, wide-strokes, on a smoothed strip of rock. Add a "cave bed" variant.
- Copper plate: real grants are several thin plates strung on a ring, with a royal seal fixed on the ring. Build a 3-plate set that fans open, ring through holes at the left edge, seal on the ring. Text engraved in lines, not embossed. Patina in grooves first.
- Seal: cast copper seal with a raised emblem in a circle (use the Chola, Pandya or Pallava emblem from the identity kit later).
- Coin: real Chola, Pandya and Chera coins are small, thick, uneven, sometimes punch-marked. Add "punch-marked" and "Roman denarius" variants. Uneven edge, off-centre strike.
- Sherd: irregular polygon edges, thickness edge visible, graffiti marks scratched after firing. Add a "black-and-red ware" variant.
- Ring: signet ring with a bezel, band thickness, side view and top view.
- Tamil text on every object: use the correct historical script for the object (Tamil-Brahmi on stone and pottery, Vatteluttu on copper plates, modern round Tamil on palm leaf) using the project's own fonts. Mark sample text as placeholder.

## Precision of interaction
- Physical rules: leaf flexes but does not fold; copper does not bend; stone does not move; coin and sherd can tilt only a few degrees. The animation for each object must obey this.
- Shadows: one light direction for the whole site (top-left, soft). Every object's shadow must match its thickness (a leaf is thin, a slab is thick).
- Scale: keep a consistent real-size feel. A coin is smaller than a leaf, a plate is about leaf-sized, a slab is the biggest.

## Acceptance checks
1. `DESIGN_SYSTEM.md` has a measured "Precision spec" table for every component with source ids and "estimate" flags.
2. A `_checks/precision-*.png` sheet for each component with the difference notes.
3. No reference photo is shipped: run `git status` and show no files from `design/references/` are tracked.
4. Contrast table rerun for any changed text-on-object pair.
5. Reduced motion, keyboard, file:// and console-error checks as in Prompt 2.
6. Report what you could not measure and what is still a guess.
