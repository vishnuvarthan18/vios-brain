# PROMPT 6 — Fix the 3 known bugs, one material system, cut to minimal motion (design-system wrap-up)

This is the last planned pass on the DESIGN SYSTEM itself before work moves to building real pages on top of it.
Do this as ONE reviewed batch, not another long autonomous queue. Stop at the end and report; do not start a new
open-ended task list after this.

Context: `redesign-design-system` branch, then `realism-engine` branch on top of it (T0–T16 already done, see
`design/realism/FINAL.md`). Work on a new branch `realism-polish` off `realism-engine`. Do not touch the 7 old
site pages. Keep `website_live_backup_2026-09-25/` and all engine backups untouched.

## Owner direction (read this before doing anything)
This site is going to an international-level showcase. Look and feel matters more than feature count. Two rules
now apply together:
1. Push visual depth for the hero objects: real geometry, real PBR material, real lighting — not a CSS trick
   pretending to be real.
2. STOP adding animation. Cut to the minimum. Depth work is about correctness, not about more moving parts.

## Part A — Fix the 3 confirmed bugs
1. **Copper ring not threading the hole.** Currently `.copper-set__ring` is hand-placed with CSS percentages and
   z-index 4 puts the whole ring in front of the plate, so it can never look threaded. Wire the already-built and
   tested T5b physics scene ("copper plate set on its ring", period accuracy 0.064% of a 3% target) into the
   `.copper-set` component as its REST POSE: part of the ring renders behind the plate (through the hole), part
   in front. This is a one-time geometry fix, not new animation — the physics computes the correct static
   position; it does not need to keep simulating on screen.
2. **Coin lettering is flat text, not engraving.** `.coin-card__legend` is a DOM `<span>` with one soft shadow.
   Replace it with lettering baked into the coin's material/texture: uneven stroke depth, worn high points on the
   relief, lit from the same single light direction as the rest of the coin's surface. Reuse the T4-coins tuned
   material as the base; do not invent a new coin look.
3. **Leaf-bundle open state doesn't match the closed state.** Closed uses the tuned `--mat-bundle` / `--mat-wood`
   materials (correct). The fanned-open leaves (`.leaf-bundle__fan i`) use a different, older `--mat-leaf` flat
   oval with no grain and no per-leaf variation. Give the fanned leaves the same tuned palm-leaf material as the
   rest of the site, plus small per-leaf colour/grain variation so five leaves don't look identical.

## Part B — One material system, no orphans
1. Build (or finish, if partly there in `materials.params.js`) a single material registry: one source of truth
   per material (palm_leaf, stone, copper_plate, pottery, coins, rings, seals, cotton_thread, wood_board), each
   with its tuned colour/texture parameters from T4.
2. Every component that renders a material — cards, detail views, open/closed states, 3D scenes, the styleguide
   demo blocks — must read from this registry. No component may define its own one-off color, gradient, or
   texture value for a material that already exists in the registry.
3. Write an automated check (`design/realism/eval/orphan_materials.mjs` or similar) that scans the built site's
   CSS/JS for colour or texture values that resemble a registry material but don't reference it, and FAILS if it
   finds one. Run it against the whole site now and report every orphan found — expect more than the 3 already
   caught by the owner. Fix every orphan it finds, using the same rule as Part A.
4. This check must be runnable again later (document the command in `DESIGN_SYSTEM.md`) so this class of bug
   cannot quietly come back as new pages are built.

## Part C — Motion audit: cut to minimal
1. List every animation currently on the site: every GSAP timeline in `js/motion.js` / `js/realism/*`, every CSS
   transition or `@keyframes`. Write the list to `design/realism/MOTION_AUDIT.md`.
2. Classify each as FUNCTIONAL (needed to show a state change the user asked for: open/close the bundle, flip a
   leaf, expand an identity card, the copper set fanning open) or DECORATIVE (ambient dust drift, the page-load
   intro flourish, hover lifts on cards, cursor followers, scroll-linked parallax that isn't load-bearing for
   understanding the content).
3. Remove every DECORATIVE animation entirely. Do not replace it with a smaller version of itself — cut it.
4. Shorten every remaining FUNCTIONAL animation: target under 400ms, most under 250ms. State changes should feel
   immediate, not performed.
5. Reduced-motion and keyboard behaviour must keep working exactly as before (rerun the T15 checks after this
   change and report the same pass numbers or better, never worse).
6. Do not add anything new here. This part only removes and shortens.

## Part D — Reopen the palm-leaf 3D case, properly this time
The earlier 3D leaf failed its gate on 2 of 7 conditions: fibre direction about 1.7x too strong
(anisotropy_diff 0.189 against a 0.15 target) and letters didn't shadow enough under raking light (lighting dL*
2.448 against a 4.0 target). Fix the shader itself — reduce the directional strength of the fibre normal map,
and rework the letter-groove depth so it responds correctly to light angle without literally cutting through the
leaf (the T3 log already ruled that out; find a shape that reads under light without being unrealistically
deep — a shallower groove with a sharper edge profile can shadow more than a shallow rounded one).
Rerun the exact same automatic gate as before (`eval/gate.py`, same 7 conditions, same targets) and report the
result honestly — adopt 3D only if it passes everything this time; otherwise keep 2D and say so, same as before.
This is the one place where deeper 3D work is explicitly wanted, because palm leaf is the most visible object on
the home page and in the Kural reader.

## Rules (carried over, still apply)
Only SHIP-tier photos may be used or turned into textures, with credit. Never delete or overwrite existing files;
back up before editing. File:// double-click must still work, zero console errors, zero external requests except
self-hosted fonts. Contrast 4.5:1 body text stays. Keep Tamil text as real text. One commit per part (A, B, C, D),
clear messages, on `realism-polish`.

## Stop condition and report
When Parts A–D are done, STOP. Do not queue further tasks or start a new open-ended run. Produce:
1. A screenshot sheet: closed vs. open bundle, the coin card, the copper set — before and after — at 360/768/1200.
2. The orphan-materials report: how many found, all fixed or listed with reasons if not.
3. `MOTION_AUDIT.md`: full list, what was cut, what was kept and its new duration.
4. The palm-leaf gate result: pass or fail, same 7 conditions, with numbers.
5. What is NOT verified: real phone, Safari, Firefox, screen reader, Tamil content — same caveats as before.
Wait for the owner to review before any further work begins.
