# Realism plan: ancient Tamil theme, physics, deep analysis (2026-09-26)

Owner direction: site is for an international-level stage. Current design system is about 50-60% done. Look and feel is not real enough yet. Use more physics, deeper analysis, more realism. Theme = ancient Tamil.

## Honest limits
- "100% real" is not reachable in a browser. Goal: convincing at first look, and physically right in motion.
- Real 3D and physics cost weight and speed. Every step must be measured on a real phone. A light fallback (current 2D) stays for slow phones, reduced motion, and no-WebGL.
- Only SHIP-tier photos (public domain, CC0, CC BY) may be used or turned into textures. Textures made from CC BY-SA or STUDY photos may not ship.

## Phase 0: analysis before building (no code changes to the site)
1. Material study per object, written into DESIGN_SYSTEM.md (each value sourced or flagged "estimate"): thickness, stiffness, weight, friction, how it breaks, how it sounds. Palm leaf: flexible along the length, splits along the fibre, curls when dry. Copper: rigid, heavy, rings when tapped. Stone: never moves. Thread: cotton, friction, slack. Sherd: brittle. Coin: spins, rolls, settles.
2. Light study: real objects in real light. Test raking light (low angle) to show carved letters, and oil-lamp (kuthuvilakku) light with warm flicker.
3. Wear and age study: cracks, stains, edge loss by age of the object.
4. Look-and-feel gap list from the owner: what still looks fake (input needed).
5. Script-evolution content: no sourced content exists yet. Needed for a Tamil-Brahmi to Vatteluttu to modern timeline.

## Phase 1: realism spike (one object, then decide)
Prototype only the palm leaf in WebGL (Three.js): PBR material, normal and roughness maps from SHIP photos plus procedural noise, one light (lamp), leaf as a bending mesh with mass-spring or Verlet physics, thread as a rope, two holes with cord. Compare with the current 2D on the same phone: looks, frame rate, file size. Decision gate: adopt 3D for all objects, or only for hero objects, or keep 2D with better shading.
Candidate stack (dev agent must measure sizes and licenses, not assume): Three.js for rendering, a small physics library (cannon-es or Rapier, or Matter.js for 2D) for rigid bodies and constraints, custom Verlet for leaf and thread, GSAP for choreography and Draggable. Must still open by double-click (bundle to one script, no CDN, no ES-module fetch on file://).

## Phase 2: rebuild materials on the chosen stack
- Palm leaf: bends along its length, does not fold; stacked leaves shift and slide; thread has slack and friction; untie is simulated, not scripted.
- Copper plates: heavy swing on the ring, small ring chime, sheen follows light.
- Stone: carved letters revealed by moving lamp light (normal map). Nothing moves.
- Coins: toss, spin, settle, tilt limit. Sherds: tilt only.
- Optional sound layer, muted by default: leaf rustle, stylus scratch, copper chime, chisel. Only CC0 or self-made audio; check every file.
- One light direction rule stays, with a moving lamp as the special case.

## Phase 3: ancient-Tamil theme depth
- Scenes from real content: Sangam poem inked on a leaf; Tamil-Brahmi cave bed; Keeladi sherd; Chola temple wall by lamplight; copper grant with seal; Roman coin from a Muziris/Arikamedu context.
- Five landscapes (Aintinai) as section moods.
- Time control: script morph Tamil-Brahmi to Vatteluttu to modern (needs sourced data).
- Every caption and fact sourced; Tamil speaker review.

## Other work in parallel (do not drop)
A. Reference engine: viewer (no download) and extra-source overnight run for palm leaf, pottery, coins, copper plates; own-photo inbox for rings and seals.
B. Prompt 2d: fix list of what looks fake.
C. Identity page batches 2 and 3, and a Tamil speaker check of CONTENT_TO_VERIFY.md.
D. Pages: home (bundle navigation), Chola section, Thirukkural leaf reader (one Kural per leaf), origins, kings, trade; then old-site pages.
E. Dark mode and page-level components (nav, footer, timeline, map, search).
F. License cleanup: 25 duplicate IDs, bad author fields, STO-070/071 clash, credits page.
G. QA: Safari, Firefox, real phone, screen reader, real frame rate.
H. Git: commit prompts and harvester tools; keep design/references ignored.

## Order
1. Now, in parallel: A (viewer + extra sources), B (owner fix list), Phase 0 study.
2. Phase 1 spike, then the decision gate with the owner.
3. Phase 2, then D pages on the final look.
4. C, E, F, G, H alongside; nothing ships before the fact check and license cleanup.
