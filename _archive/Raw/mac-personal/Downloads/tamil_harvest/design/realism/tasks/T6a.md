# T6a - Lighting: stone by raking light and oil lamp (Prompt 4 section 8, T6)

Read `design/realism/AGENT_BRIEF.md` first (rules, tools, start/finish steps).

Owner text: "T6 Lighting: raking light and oil-lamp light reveal carved letters on stone and copper; tests from
section 6." Section 6: "render at 5 light angles; carved letters must change in shading with angle (normal-map test):
the shading difference between the lit and shadowed edge must be above a set threshold; a flat painted texture must FAIL."

This task: stone. Build `design/realism/t6/stone_light.html` (file://): the tuned stone material
(`materials/stone.params.json` from T4-stone, else spec colour), carved Tamil letters (real Tamil text via the Semmozhi
font; sample text flagged unverified), a raking-light control (azimuth + low elevation, keyboard accessible) and an oil
lamp (warm, seeded flicker; none under reduced motion). Expose `window.__render(params)` with the rig schema and
register a lighting candidate (url) + page. Pass the lighting test at elevation 35 AND report it at a raking 15 deg.
Visual scores must not get worse than T4's. Commit.
