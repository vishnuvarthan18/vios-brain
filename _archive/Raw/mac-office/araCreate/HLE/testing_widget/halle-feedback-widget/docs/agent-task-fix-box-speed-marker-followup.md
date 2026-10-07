# Follow-up — Vishnu's decisions on the flagged items

22 September 2026. Answers `docs/agent-task-fix-box-speed-marker.md`'s final
report (Parts A-D, all complete and tested).

**1. Commit now.** Everything reported is tested and safe (build-and-test
only, nothing live). Go ahead and commit — one concern per commit, as
always. Still do not push or deploy without asking separately.

**2. B2's parallel-clone tradeoff: skip it.** Do not overlap
`inline_pseudo_backgrounds()` with the server attempt. The speed already
measured (Home ~2019ms -> ~1663ms, timeout tightened to 4,000ms) is enough
— not worth the fragile shared-state risk for a small extra gain. Leave
this exactly as you left it in the final report.

**3. Part C's DB/admin wiring: do it now.** Add the `capture_method`
column to `reports` (not buried in the `meta` blob, as your report already
suggested) and wire `post_report_schema` to accept and store the
`captureMethod` field already being sent from the client — otherwise it
keeps being silently dropped. Surfacing it in the admin UI itself is still
not required for this task, just make sure it is actually saved so it can
be queried later. This is needed soon: real bad-screenshot examples are
coming to diagnose the still-open third problem (elements out of place in
the picture itself), and every report between now and then that isn't
recording its capture method is a missed data point.

Do not start anything else beyond these three items. Report back briefly
once committed and once the DB/admin wiring is done and tested.
