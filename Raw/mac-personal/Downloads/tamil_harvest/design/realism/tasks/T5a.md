# T5a - Physics: leaf, bundle and thread (Prompt 4 section 8, T5)

Read `design/realism/AGENT_BRIEF.md` first (rules, tools, start/finish steps).

Owner text: "T5 Physics per object with the tests above (leaf, bundle and thread, copper plate set on its ring, coin,
sherd, ring, seal, stone)." Tests are section 6 (already in `eval/physics_tests.mjs`).

This task: the site-scale leaf, the bundle (stack of leaves between two wooden boards, threaded cord through both
holes, knot) and the thread. Build `physics/scenes.js` builders for the bundle with sizes from specs (palm_leaf and
wood_board geometry from the Glasgow catalogue, thickness/mass estimates) - add, do not break existing builders.
Add tests for the new scenes to `eval/physics_tests.mjs` with a known-bad variant each (the harness pattern is there).
All reference scenes must pass; performance: measure ms per step for the bundle in Node and in the page (target: the
page stays under 20 ms p95 at 4x throttle). Wire the bundle into the T3 page if T3b adopted 3D, else into a demo page
`design/realism/t5/bundle.html` (2D render of the simulated state is fine). Register pages/candidates. Commit.
