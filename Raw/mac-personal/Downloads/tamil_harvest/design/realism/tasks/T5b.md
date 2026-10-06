# T5b - Physics: copper plate set on its ring

Read `design/realism/AGENT_BRIEF.md` first (rules, tools, start/finish steps).

Section 6 test: "Pendulum (copper plates on ring): measured period within 3% of 2*pi*sqrt(L/g) for small angles;
amplitude decays monotonically; no energy gain." Build a scene with N plates (spec copper_plate: plates_in_set,
density, thickness estimate, damping) hanging on one ring, plates colliding with each other (layer or plane contact),
the ring as a fixed torus path. Test each plate's small-angle period against its own L_eq, decay, no energy gain,
no plate passing through another, determinism, frame-rate independence; add known-bad variants. Demo page
`design/realism/t5/copper_set.html` (file://, reduced motion = no swing). Register, commit.
