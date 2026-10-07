# Prompt 6 — report (stop here; nothing further is queued)

Branch `realism-polish` off `realism-engine` (8e4b12f). Four commits, one per part:
`473a3bf` A · `0f46303` B · `7c7a232` C · and the Part D commit, which also carries this file. The 7 old pages,
`website_live_backup_2026-09-25/` and every engine backup are untouched; everything edited was first copied to
`design/realism/backups/P6/`.

## 1. Screenshot sheet

**`design/realism/polish/SHEET.jpg`** — closed bundle, open bundle, the coin card, every coin card, the copper set
and the fanned copper set, BEFORE (realism-engine) and AFTER (this branch), at 360 / 768 / 1200 px, file:// in the
rig's Chrome for Testing 151, DPR 2. Raw shots: `polish/shots/before|after/` (gitignored), made by
`node design/realism/polish/shots.mjs before|after`; 0 console errors in both runs.

What changed, part by part:

- **A1 Copper ring.** The ring now goes *through* the hole: back half under the plates, front half over them, the
  wire seated on the hole's ring-side edge. Geometry is the T5b `plateSet` scene settled once
  (`tools/build_copper_set.mjs`: 64 s simulated, residual swing 6.1e-5 rad, holes 0.12 mm off a 0.5 mm limit), not
  hand placement. The hole moved to the spec card's measured 0.205 of the length (SHIP photo COP-028 agrees); the old
  6% was by eye. The single `.copper-plate` card had the same fault and was fixed the same way.
- **A2 Coin lettering.** Every coin face is rendered once by the measured stage with the T4-coins material; the legend,
  bead border, inner line, wreath and rim text are one raised height mask (uneven stroke height, worn smooth bright
  tops, dirt at their foot), lit by the material's single top-left light. Gold (1 SHIP photo) and silver (6) swap only
  the colour ramp, measured the T1 way — both low confidence. The 3D coin block uses the same raised legend
  (lighting 9.61 dL\*, passes).
- **A3 Fanned leaves.** They are the site's 2D palm-leaf material now (fibre grain along the leaf, spec-position holes,
  the leaf outline), each with its own grain phase, tone and length.

## 2. Orphan materials

Command (documented in `website_live/DESIGN_SYSTEM.md` → "Material registry"):
`node design/realism/eval/orphan_materials.mjs` — scans the new pages' CSS, inline styles and scripts, `js/` and the
identity kit, and also fails if any generated block (registry, copper-set rest pose, site stage) is stale.

| run | orphans |
|---|---|
| realism-engine as it stood | **435** (`polish/orphans_start.json`) |
| after Part A | 436 (`polish/ORPHANS_BEFORE.md`, every finding listed) |
| after Part B-D | **0**, 3 allowed exceptions, generated blocks 3/3 current → PASS (`polish/ORPHANS_AFTER.md`) |

What the 436 were: 95 material definitions scattered over tokens.css, textures.css and materials.css with no single
source; 246 raw colour copies (242 = one clay tone copied into every identity SVG); 35 cross-material (every seal
drawn in copper tokens, the signet ring in coin gold, a sherd edge in stone, a copper hole in leaf ink); 28 one-off
mixes/gradients (the thread cord, carved and engraved text, sherd edges, hole rims, coin rims); 27 Prompt 1 textures
still under components; 5 stray texture files. **All fixed**, by moving them into or pointing them at the registry
(`design/realism/registry/registry.json` → generated block in `css/tokens.css` + `js/realism/materials.params.js`).
The 3 allowed exceptions are the photo-credit swatches, which show the texture files themselves next to their licence
(reason recorded in the registry). Visible side effects, on purpose: the seal on copper-plate card rings is now seal
bronze like every other seal; the 3D blocks' no-WebGL fallback shows the block's own T4 colours. 2D palette values
were not changed (a T4 ramp is an albedo for a lit shader, not a flat CSS colour); page diffs show 0 changed pixels on
home, Chola, Kural and identity from the refactor.

## 3. Motion audit

**`design/realism/MOTION_AUDIT.md`** — every animation, its class, before/after. Kept and shortened: leaf flip
0.68 s → 0.24 s, drag release 0.6 s spring → 0.18 s, bundle reader 1.14 s → 0.34 s, copper-set fan 0.70 s → 0.24 s,
identity card 646 ms → 238 ms, CSS fallbacks 350 → 240 ms and 500 → 200 ms. Removed entirely: page intro, ink writing,
stone carving, pointer-following copper sheen, ring spring, pinned parallax story and its dust loop, press springs,
hover lifts and tilts, emblem rise, the untie that delayed bundle links, the oil-lamp flicker, every hover transition.
T15 re-run: 13/13 pages pass, reduced motion 0.000 changed pixels on all 13, 0 axe violations, Tab reaches every
control (styleguide 139/139 — the 3 buttons of removed animations are gone), focus rings 8/8 — same or better.

## 4. Palm-leaf gate (`eval/gate.py palm_leaf`, same 7 conditions, same targets)

| condition | before (T3b) | now |
|---|---|---|
| visual scores | 8 of 10 (anisotropy 0.189, grain 17.46°) | **10 of 10 — PASS** (anisotropy diff 0.02, render 0.232 vs photos 0.252; grain 3.56°) |
| lighting: lit vs shadowed letter wall | 2.448 dL\* | **3.721 dL\* vs ≥ 4 — FAIL** (per azimuth 3.11 / 2.95 / 4.16 / 4.30 / 4.09) |
| physics | 5/5 | 5/5 PASS |
| JS added (gzip) | 36 KB | 38.5 KB PASS |
| p95 frame at 4x throttle | 7 ms | 6.6 ms PASS |
| console / network errors | 0 | 0 PASS |
| fallback | 2/2 | 2/2 PASS |

**6 of 7 → keep 2D**, as the rule says. What changed: the measured cause of the old anisotropy was fibre streaks *plus*
directional shading from raking light on the isotropic height noise; the 3D leaf now uses the registry's tuned palm
leaf with the fibre normal's strength x0.6 (1/0.6 ≈ the owner's "1.7x"), and its groove has a sharp edge that casts
shadows into it (new opt-in shader code). At the same 0.144 mm cut that lifts lighting 2.84 → 3.72. A cap of a third of
the leaf (0.157 mm) was set before probing depth; at the cap it reaches 3.96 — still short. About 0.17 mm would pass,
which is over a third of an average leaf and over half of a thin one; not done. Probes: `polish/leaf_probe/r1-r7.json`.

Scoreboard after all four parts: 345 pass / 45 fail of 390 scored (was 340 / 42 of 382). The 3 extra failures are
the site's 2D signet ring, measured for the first time: the capture selector `.ring` had been grabbing the copper
plate's charter ring; it now captures the signet ring, whose Prompt 1 coin-gold paint fails 7 of 8 rows. The ring's
paint did not change.

## 5. NOT verified

- **No real phone.** Every mobile number is a 360 px emulation on a desktop with software GL.
- **No Safari, no iOS, no Firefox.** Only Chrome for Testing 151 (and system Chrome for the page checks) was run.
- **No screen reader.** 0 axe violations is not accessibility; axe cannot read text inside WebGL canvases, and the
  coin faces are now baked canvases (each coin keeps its `role="img"` label).
- **No Tamil speaker** has checked any Tamil on any page; the identity kit has 0 items verified.
- **No real object measured.** The copper ring's size, wire and seal, the coin relief height, its wear and dirt
  amounts, the per-leaf variation, the 0.144 mm leaf cut and the cap rule are estimates. Gold coin colour rests on
  1 photo, silver on 6 (below the 20-photo floor). Colour is from uncalibrated photos.
- **Canvas text is not bit-stable** between browser launches here (1-5 LSB on a few renders); the stage identity check
  reports its control run next to every result.
- Stale check artefacts left in place (never deleted): `website_live/_checks/rec-ink-writing.gif`,
  `rec-stone-carving.gif`, `rec-intro.gif` still show animations that no longer exist.
- The realism engine's own demo pages (`design/realism/t3|t5|t6|t8`) were not motion-audited; the T3 3D leaf page still
  flickers its lamp when turned on (it is a measuring rig, not the site, and the leaf stays 2D on the site).

Waiting for the owner's review before any further work.
