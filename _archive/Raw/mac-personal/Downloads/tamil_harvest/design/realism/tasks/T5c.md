# T5c - Physics: coin, ring and seal

Read `design/realism/AGENT_BRIEF.md` first (rules, tools, start/finish steps).

Section 6: "Coin: falls, spins, settles with decaying motion; tilt limit respected." Reuse `scenes.coin`; add ring
(annulus) and seal (thick disc or square with relief) as rigid bodies with sizes from specs (all estimates flagged).
Tests per object (falls, decays, settles, rest tilt, no table penetration, determinism, frame-rate independence) +
known-bad variants. Demo `design/realism/t5/small_objects.html`. Register, commit.
