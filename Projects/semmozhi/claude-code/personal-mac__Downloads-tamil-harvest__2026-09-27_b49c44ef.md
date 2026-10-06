**Vishnu** (2026-09-27T06:19): Read design/prompts/prompt-6-consistency-minimal-motion.md in the tamil_harvest folder and follow it exactly. This is one reviewed batch, not an open-ended run: do Parts A through D, then stop and report as the file says. Do not start any new task queue after this. Work on branch realism-polish off realism-engine. Do not touch the 7 old pages.

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): Reading the prompt and codebase now; next I'll create the `realism-polish` branch and map out the materials, components and motion code before editing anything.

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:19): _[tool: Bash]_

**Claude** (2026-09-27T06:20): Still mapping the code: found the copper-set, coin-card and leaf-bundle rules in `components.css`; now reading them in full plus the page markup.

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): Reading `js/motion.js` (the GSAP timelines) so I can inventory every animation for Part C.

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): _[tool: Bash]_

**Claude** (2026-09-27T06:20): Now reading the engine side: tuned material params, the palm-leaf gate and the T3 3D leaf, which Parts B and D build on.

**Claude** (2026-09-27T06:21): _[tool: Bash]_

**Claude** (2026-09-27T06:21): _[tool: Bash]_

**Claude** (2026-09-27T06:21): _[tool: Bash]_

**Claude** (2026-09-27T06:21): _[tool: Bash]_

**Claude** (2026-09-27T06:21): _[tool: Bash]_

**Claude** (2026-09-27T06:21): _[tool: Bash]_

**Claude** (2026-09-27T06:21): Now checking how the anisotropy score is measured (`lib/measure.py`), since the T3 notes say the leaf reads as too directional even with the fibre term off.

**Claude** (2026-09-27T06:21): _[tool: Bash]_

**Claude** (2026-09-27T06:21): _[tool: Bash]_

**Claude** (2026-09-27T06:22): _[tool: Bash]_

**Claude** (2026-09-27T06:22): _[tool: Bash]_

**Claude** (2026-09-27T06:22): _[tool: Bash]_

**Claude** (2026-09-27T06:22): _[tool: Bash]_

**Claude** (2026-09-27T06:22): Taking the "before" screenshots next (bundle closed/open, coin card, copper set at 360/768/1200), before any edit.

**Claude** (2026-09-27T06:22): _[tool: Bash]_

**Claude** (2026-09-27T06:22): _[tool: Bash]_

**Claude** (2026-09-27T06:22): _[tool: Bash]_

**Claude** (2026-09-27T06:23): Now the lighting evaluator, which scores the palm-leaf letter shadowing for Part D.

**Claude** (2026-09-27T06:23): _[tool: Bash]_

**Claude** (2026-09-27T06:23): _[tool: Bash]_

**Claude** (2026-09-27T06:25): _[tool: Bash]_

**Claude** (2026-09-27T06:25): _[tool: Bash]_

**Claude** (2026-09-27T06:25): _[tool: Write]_

**Claude** (2026-09-27T06:25): _[tool: Bash]_

**Claude** (2026-09-27T06:26): _[tool: Read]_

**Claude** (2026-09-27T06:26): _[tool: Read]_

**Claude** (2026-09-27T06:26): _[tool: Read]_

**Vishnu** (2026-09-27T06:26): [Image: original 2304x912, displayed at 2000x792. Multiply coordinates by 1.15 to map to original image.]

**Claude** (2026-09-27T06:26): Before screenshots captured (18, 0 console errors). The open-bundle shot clips the fanned leaves, so I'm widening the capture area and re-taking the before set, still with no site edits made.

**Claude** (2026-09-27T06:26): _[tool: Bash]_

**Claude** (2026-09-27T06:27): _[tool: Read]_

**Claude** (2026-09-27T06:27): Now designing the Part B registry, since Part A's fixes should read from it. First comparing the T4-tuned colour ramps with the site's current material tokens.

**Claude** (2026-09-27T06:27): _[tool: Bash]_

**Claude** (2026-09-27T06:29): _[tool: Bash]_

**Claude** (2026-09-27T06:29): _[tool: Bash]_

**Claude** (2026-09-27T06:29): _[tool: Bash]_

**Claude** (2026-09-27T06:29): _[tool: Bash]_

**Claude** (2026-09-27T06:30): _[tool: Bash]_

**Claude** (2026-09-27T06:30): Status: before-shots done, codebase mapped. Starting Part A.1 now — reading the T5b copper-set physics scene to get the ring's rest pose.

**Claude** (2026-09-27T06:30): _[tool: Bash]_

**Claude** (2026-09-27T06:30): _[tool: Bash]_

**Claude** (2026-09-27T06:30): _[tool: Bash]_

**Claude** (2026-09-27T06:32): _[tool: Bash]_

**Claude** (2026-09-27T06:32): _[tool: Bash]_

**Claude** (2026-09-27T06:32): _[tool: Bash]_

**Claude** (2026-09-27T06:32): Checking two SHIP copper-plate photos to see how the ring actually threads the hole before fixing the geometry.

**Claude** (2026-09-27T06:32): _[tool: Read]_

**Claude** (2026-09-27T06:32): _[tool: Read]_

**Claude** (2026-09-27T06:34): _[tool: Bash]_

**Claude** (2026-09-27T06:34): _[tool: Bash]_

**Claude** (2026-09-27T06:36): _[tool: Write]_

**Claude** (2026-09-27T06:36): Backing up every file this batch may touch before editing (the repo rule), then running the generator.

**Claude** (2026-09-27T06:36): _[tool: Bash]_

**Claude** (2026-09-27T06:37): _[tool: Bash]_

**Claude** (2026-09-27T06:37): The set comes to rest at a small non-zero angle (6e-5 rad) rather than exactly 0, so I'll switch the settle test from "angle below 1e-5" to "angle no longer changing".

**Claude** (2026-09-27T06:37): _[tool: Bash]_

**Claude** (2026-09-27T06:37): _[tool: Bash]_

**Claude** (2026-09-27T06:37): Rest pose written: settled after 64 s simulated, holes 0.12 mm off the seat circle (T5b limit 0.5 mm). Pointing the GSAP fan at the new hole pivot, then screenshotting.

**Claude** (2026-09-27T06:37): _[tool: Bash]_

**Claude** (2026-09-27T06:37): _[tool: Bash]_

**Claude** (2026-09-27T06:38): _[tool: Read]_

**Claude** (2026-09-27T06:38): _[tool: Read]_

**Claude** (2026-09-27T06:38): _[tool: Bash]_

**Claude** (2026-09-27T06:38): _[tool: Read]_

**Claude** (2026-09-27T06:38): Ring now reads as a heavy wire threaded through the hole. Hiding the small seam where the two halves meet, then regenerating.

**Claude** (2026-09-27T06:39): _[tool: Bash]_

**Claude** (2026-09-27T06:39): _[tool: Read]_

**Claude** (2026-09-27T06:39): Copper set fixed at 360 and 1200. Checking the sibling single `.copper-plate` card, which draws its ring the same hand-placed way.

**Claude** (2026-09-27T06:39): _[tool: Bash]_

**Claude** (2026-09-27T06:39): _[tool: Read]_

**Claude** (2026-09-27T06:40): _[tool: Bash]_

**Claude** (2026-09-27T06:40): _[tool: Bash]_

**Claude** (2026-09-27T06:40): _[tool: Bash]_

**Claude** (2026-09-27T06:41): _[tool: Bash]_

**Claude** (2026-09-27T06:41): _[tool: Read]_

**Claude** (2026-09-27T06:41): Card ring now threads its hole too; enlarging it slightly to use the card's 2.5 rem ring margin.

**Claude** (2026-09-27T06:41): _[tool: Bash]_

**Claude** (2026-09-27T06:41): _[tool: Read]_

**Claude** (2026-09-27T06:41): _[tool: Bash]_

**Claude** (2026-09-27T06:41): _[tool: Bash]_

**Claude** (2026-09-27T06:42): _[tool: Write]_

**Claude** (2026-09-27T06:42): _[tool: Bash]_

**Claude** (2026-09-27T06:42): Measuring the gold/silver coin ramps from SHIP photos now (the first attempt used a missing `timeout` command).

**Claude** (2026-09-27T06:42): _[tool: Bash]_

**Claude** (2026-09-27T06:42): _[tool: Bash]_

**Claude** (2026-09-27T06:42): _[tool: Bash]_

**Claude** (2026-09-27T06:43): _[tool: Read]_

**Claude** (2026-09-27T06:43): _[tool: Bash]_

**Claude** (2026-09-27T06:43): _[tool: Bash]_

**Claude** (2026-09-27T06:43): Editing `rig/stage.js` (opt-in relief mode + mask override); defaults unchanged.

**Claude** (2026-09-27T06:44): _[tool: Bash]_

**Claude** (2026-09-27T06:44): _[tool: Bash]_

**Claude** (2026-09-27T06:44): The rig page failed to expose `__render` — likely a GLSL or JS error from my edit. Checking the console.

**Claude** (2026-09-27T06:44): _[tool: Bash]_

**Claude** (2026-09-27T06:44): A backtick in my GLSL comment closed the JS template literal. Fixing.

**Claude** (2026-09-27T06:44): _[tool: Bash]_

**Claude** (2026-09-27T06:44): Relief renders. Verifying the change is inert for existing modes: rendering the same params through the backed-up original stage and comparing pixel hashes.

**Claude** (2026-09-27T06:45): _[tool: Bash]_

**Claude** (2026-09-27T06:45): All letter-bearing renders differ, including the pure letter mask — which the edit can't touch. Suspecting the backup copy's relative font path breaks from its folder.

**Claude** (2026-09-27T06:45): _[tool: Bash]_

**Claude** (2026-09-27T06:46): Confirmed: the backup page's `../../../website_live` font URL doesn't resolve from the backup folder. Pointing the harness at a scratchpad copy with an absolute font URL.

**Claude** (2026-09-27T06:46): _[tool: Bash]_

**Claude** (2026-09-27T06:47): Palm-leaf renders now match, but four other lettered materials still differ, even as a bare letter mask. Measuring how large the difference actually is before drawing conclusions.

**Claude** (2026-09-27T06:47): _[tool: Bash]_

**Claude** (2026-09-27T06:47): Only 63 bytes differ, each by 1 LSB, in the canvas text raster — the two pages load the font by different URLs. Making the test fair by running both stages from identical temp copies.

**Claude** (2026-09-27T06:47): _[tool: Bash]_

**Claude** (2026-09-27T06:47): _[tool: Bash]_

**Claude** (2026-09-27T06:48): _[tool: Bash]_

**Claude** (2026-09-27T06:50): _[tool: Bash]_

**Claude** (2026-09-27T06:51): _[tool: Bash]_

**Claude** (2026-09-27T06:53): _[tool: Bash]_

**Claude** (2026-09-27T06:54): Still verifying the shader change is inert (fixing a missing import in the checker).

**Claude** (2026-09-27T06:54): _[tool: Bash]_

**Claude** (2026-09-27T06:57): _[tool: Bash]_

**Claude** (2026-09-27T06:58): _[tool: Write]_

**Claude** (2026-09-27T06:58): _[tool: Bash]_

**Claude** (2026-09-27T06:58): _[tool: Read]_

**Vishnu** (2026-09-27T06:58): [Image: original 2304x912, displayed at 2000x792. Multiply coordinates by 1.15 to map to original image.]

**Claude** (2026-09-27T06:59): _[tool: Bash]_

**Claude** (2026-09-27T06:59): _[tool: Read]_

**Claude** (2026-09-27T07:00): _[tool: Bash]_

**Claude** (2026-09-27T07:00): _[tool: Bash]_

**Claude** (2026-09-27T07:00): _[tool: Read]_

**Claude** (2026-09-27T07:00): _[tool: Read]_

**Claude** (2026-09-27T07:01): _[tool: Bash]_

**Claude** (2026-09-27T07:01): Tuning the coin relief (a comment-matching step failed so nothing was written; rerunning).

**Claude** (2026-09-27T07:01): _[tool: Bash]_

**Claude** (2026-09-27T07:02): _[tool: Read]_

**Claude** (2026-09-27T07:02): _[tool: Read]_

**Claude** (2026-09-27T07:03): _[tool: Bash]_

**Claude** (2026-09-27T07:03): _[tool: Bash]_

**Claude** (2026-09-27T07:03): _[tool: Read]_

**Claude** (2026-09-27T07:04): _[tool: Bash]_

**Claude** (2026-09-27T07:06): _[tool: Bash]_

**Claude** (2026-09-27T07:06): _[tool: Bash]_

**Claude** (2026-09-27T07:06): Adding the low-quality/forced-colours fallbacks for the fan leaves, then running the existing page checks (`checks_p2`) on the two edited pages before committing Part A.

**Claude** (2026-09-27T07:06): _[tool: Bash]_

**Claude** (2026-09-27T07:06): _[tool: Bash]_

**Claude** (2026-09-27T07:07): _[tool: Bash]_

**Claude** (2026-09-27T07:07): _[tool: Bash]_

**Claude** (2026-09-27T07:07): While page checks run, logging Part A's decisions in `DECISIONS.md` (the engine's rule: every choice made without the owner is recorded).

**Claude** (2026-09-27T07:07): _[tool: Bash]_

**Claude** (2026-09-27T07:07): _[tool: Bash]_

**Claude** (2026-09-27T07:07): Decisions logged. Checking on the page checks.

**Claude** (2026-09-27T07:07): _[tool: Bash]_

**Claude** (2026-09-27T07:09): _[tool: Bash]_

**Claude** (2026-09-27T07:09): _[tool: Bash]_

**Claude** (2026-09-27T07:09): _[tool: Bash]_

**Claude** (2026-09-27T07:10): _[tool: Write]_

**Claude** (2026-09-27T07:10): _[tool: Bash]_

**Claude** (2026-09-27T07:10): Registry source extracted (9 materials; `--thread` and `--catalogue-red` stay UI tokens). `checks_p2` on the styleguide finished; reading its results.

**Claude** (2026-09-27T07:10): _[tool: Bash]_

**Claude** (2026-09-27T07:10): _[tool: Bash]_

**Claude** (2026-09-27T07:10): Still +16 KB. The projected ring is an exact ellipse (an affine image of a circle), so I'll emit true SVG elliptical arcs instead of polylines — exact and a few dozen bytes each.

**Claude** (2026-09-27T07:11): _[tool: Bash]_

**Claude** (2026-09-27T07:11): _[tool: Read]_

**Claude** (2026-09-27T07:11): _[tool: Read]_

**Claude** (2026-09-27T07:11): Same look, styleguide now only +2.5 KB over the original. Reading the home-page check results.

**Claude** (2026-09-27T07:11): _[tool: Bash]_

**Claude** (2026-09-27T07:11): Home page checks identical to before (0 errors, no overflow, keyboard counts unchanged). Preparing the Part A commit: checking what's tracked/ignored for backups and screenshots.

**Claude** (2026-09-27T07:11): _[tool: Bash]_

**Claude** (2026-09-27T07:12): Adding the raw screenshot folder to `.gitignore` (7.8 MB of DPR-2 PNGs; only the final sheet will be committed), then committing Part A.

**Claude** (2026-09-27T07:12): _[tool: Bash]_

**Claude** (2026-09-27T07:13): Part A committed (`473a3bf`). Starting Part B proper: writing the orphan checker `design/realism/eval/orphan_materials.mjs` first, so it can measure the site as it stands before any registry fixes.

**Claude** (2026-09-27T07:14): _[tool: Write]_

**Claude** (2026-09-27T07:14): Adding the material metadata (T4 source files, gate files, and the component-to-material class map the cross-material rule uses) to `registry.json`, then running the checker on the post-Part-A site.

**Claude** (2026-09-27T07:14): _[tool: Bash]_

**Claude** (2026-09-27T07:14): The checker runs. Raw count is 1,595, dominated by the identity kit and `tokens.css`; inspecting samples before interpreting (some may be checker mistakes).

**Claude** (2026-09-27T07:14): _[tool: Bash]_

**Claude** (2026-09-27T07:15): _[tool: Bash]_

**Claude** (2026-09-27T07:15): Refined count: 436 findings. Breaking them down by what they actually are before fixing.

**Claude** (2026-09-27T07:15): _[tool: Bash]_

**Claude** (2026-09-27T07:15): _[tool: Bash]_

**Claude** (2026-09-27T07:18): _[tool: Bash]_

**Claude** (2026-09-27T07:19): _[tool: Bash]_

**Claude** (2026-09-27T07:19): _[tool: Bash]_

**Claude** (2026-09-27T07:19): _[tool: Write]_

**Claude** (2026-09-27T07:20): _[tool: Bash]_

**Claude** (2026-09-27T07:20): Now pointing `build_site_materials.mjs` at the registry's `site_3d` entries instead of its own hard-coded list.

**Claude** (2026-09-27T07:20): _[tool: Bash]_

**Claude** (2026-09-27T07:20): _[tool: Bash]_

**Claude** (2026-09-27T07:21): _[tool: Bash]_

**Claude** (2026-09-27T07:21): Registry block generated. Trimming its comments to roles only (provenance stays in `registry.json`) to keep `tokens.css` lean, then moving on to `textures.css`.

**Claude** (2026-09-27T07:21): _[tool: Bash]_

**Claude** (2026-09-27T07:21): _[tool: Bash]_

**Claude** (2026-09-27T07:21): _[tool: Bash]_

**Claude** (2026-09-27T07:21): Now `materials.css`, the largest edit. Reading its current state in full first.

**Claude** (2026-09-27T07:21): _[tool: Bash]_

**Claude** (2026-09-27T07:22): _[tool: Bash]_

**Claude** (2026-09-27T07:22): `materials.css` done (no material mixes left). Now `components.css`.

**Claude** (2026-09-27T07:23): _[tool: Bash]_

**Claude** (2026-09-27T07:23): _[tool: Bash]_

**Claude** (2026-09-27T07:23): Pointing the 3D blocks' no-WebGL fallbacks at the registry's T4 colours for their own material (text on a plate for contrast), in both pages that carry them.

**Claude** (2026-09-27T07:23): _[tool: Bash]_

**Claude** (2026-09-27T07:23): Making `identity_build.py` take its colour fallbacks from the registry (instead of hand-typed hex) and giving the identity line tone a registry name.

**Claude** (2026-09-27T07:24): _[tool: Bash]_

**Claude** (2026-09-27T07:24): _[tool: Bash]_

**Claude** (2026-09-27T07:24): _[tool: Bash]_

**Claude** (2026-09-27T07:24): _[tool: Bash]_

**Claude** (2026-09-27T07:24): _[tool: Bash]_

**Claude** (2026-09-27T07:25): Before-count confirmed with the final checker: **436 orphans** at the Part A commit. Also measuring the untouched `realism-engine` state for the report.

**Claude** (2026-09-27T07:25): _[tool: Bash]_

**Claude** (2026-09-27T07:25): _[tool: Bash]_

**Claude** (2026-09-27T07:26): _[tool: Bash]_

**Claude** (2026-09-27T07:27): _[tool: Bash]_

**Claude** (2026-09-27T07:28): Still 12.6% after alignment, so the first difference isn't the inserted note. Viewing the diff map to see where changes are.

**Claude** (2026-09-27T07:28): _[tool: Bash]_

**Claude** (2026-09-27T07:28): _[tool: Bash]_

**Claude** (2026-09-27T07:29): _[tool: Bash]_

**Claude** (2026-09-27T07:30): Several sections changed more than expected. Looking at the diff maps.

**Claude** (2026-09-27T07:30): _[tool: Bash]_

**Claude** (2026-09-27T07:31): _[tool: Read]_

**Claude** (2026-09-27T07:31): _[tool: Read]_

**Claude** (2026-09-27T07:31): Diffs cover text too, which points to a fractional layout shift from the inserted note rather than paint changes. Re-running with that note hidden in the new copy so layout matches exactly.

**Claude** (2026-09-27T07:31): _[tool: Bash]_

**Claude** (2026-09-27T07:32): Only two sections now differ: `copper` (0.13%, expected: the card rings' seal now in seal bronze, engraved-lip alpha unified) and `materials-3d` (5.5%). Checking the latter.

**Claude** (2026-09-27T07:32): _[tool: Bash]_

**Claude** (2026-09-27T07:32): _[tool: Read]_

**Claude** (2026-09-27T07:32): _[tool: Read]_

**Claude** (2026-09-27T07:33): _[tool: Bash]_

**Claude** (2026-09-27T07:33): One command now covers orphans plus all three generated blocks: PASS. Documenting it in `DESIGN_SYSTEM.md`.

**Claude** (2026-09-27T07:33): _[tool: Bash]_

**Claude** (2026-09-27T07:33): _[tool: Bash]_

**Claude** (2026-09-27T07:33): _[tool: Bash]_

**Claude** (2026-09-27T07:34): The merged-block change broke the section scopes (it also merged the dark-mode `.surface-*` blocks). Restricting the merge to `:root` only, as intended.

**Claude** (2026-09-27T07:34): _[tool: Bash]_

**Claude** (2026-09-27T07:34): Contrast unchanged: the token table is byte-identical to before (0 failing pairs) and the component table passes 182/182. Now the identity kit checks after its regeneration.

**Claude** (2026-09-27T07:34): _[tool: Bash]_

**Claude** (2026-09-27T07:34): _[tool: Bash]_

**Claude** (2026-09-27T07:34): 121/121 identity SVGs pass (largest 7,951 B of the 8,192 B limit). Running the identity page's own checks in the background while I log Part B decisions.

**Claude** (2026-09-27T07:34): _[tool: Bash]_

**Claude** (2026-09-27T07:34): _[tool: Bash]_

**Claude** (2026-09-27T07:34): While those run, a quick load check of all five pages in three modes (normal, `?quality=low`, WebGL disabled) for console errors and failed requests.

**Claude** (2026-09-27T07:35): _[tool: Bash]_

**Claude** (2026-09-27T07:35): _[tool: Bash]_

**Claude** (2026-09-27T07:36): _[tool: Bash]_

**Claude** (2026-09-27T07:36): _[tool: Bash]_

**Claude** (2026-09-27T07:36): _[tool: Bash]_

**Claude** (2026-09-27T07:36): Identity page checks match before, except page weight 1.384 → 1.402 MB (budget 1.5 MB). Committing Part B.

**Claude** (2026-09-27T07:36): _[tool: Bash]_

**Claude** (2026-09-27T07:37): _[tool: Bash]_

**Claude** (2026-09-27T07:37): Part B committed (`0f46303`). Starting Part C, the motion audit. Inventorying every animation on the new pages: GSAP calls, rAF loops, CSS transitions and keyframes.

**Claude** (2026-09-27T07:37): _[tool: Bash]_

**Claude** (2026-09-27T07:37): _[tool: Bash]_

**Claude** (2026-09-27T07:37): _[tool: Bash]_

**Claude** (2026-09-27T07:38): Inventory is clear. Checking three remaining spots (smooth scrolling, the `.lift` utility, the identity card hover) before writing the audit.

**Claude** (2026-09-27T07:38): _[tool: Bash]_

**Claude** (2026-09-27T07:38): _[tool: Bash]_

**Claude** (2026-09-27T07:38): Before cutting anything, reading the styleguide's motion demo section so its markup and text can be updated to match the cuts.

**Claude** (2026-09-27T07:38): _[tool: Bash]_

**Claude** (2026-09-27T07:39): Classification is settled. Before editing, checking what `checks_p2` exercises in motion (it may call removed functions and need updating).

**Claude** (2026-09-27T07:39): _[tool: Bash]_

**Claude** (2026-09-27T07:39): `checks_p2` calls several functions Part C removes (ink, carve, intro, story, ambient loops), so it needs guards that report them as removed. Reading those sections.

**Claude** (2026-09-27T07:39): _[tool: Bash]_

**Claude** (2026-09-27T07:39): _[tool: Bash]_

**Claude** (2026-09-27T07:39): _[tool: Bash]_

**Claude** (2026-09-27T07:40): _[tool: Write]_

**Claude** (2026-09-27T07:40): _[tool: Read]_

**Claude** (2026-09-27T07:40): _[tool: Bash]_

**Claude** (2026-09-27T07:40): Rewriting `js/motion.js` to the four functional timelines now (the file is unchanged since the last commit).

**Claude** (2026-09-27T07:41): _[tool: Write]_

**Claude** (2026-09-27T07:41): _[tool: Bash]_

**Claude** (2026-09-27T07:41): Removing the identity hero's emblem rise (`identity.js`) and the oil lamp's continuous flicker loop (`materials3d.js`; the lamp stays on, steady, as it already was under reduced motion).

**Claude** (2026-09-27T07:41): _[tool: Bash]_

**Claude** (2026-09-27T07:42): Now the CSS: motion tokens trimmed, hover transitions and hover lifts/tilts removed, the untie/ink/carve/intro/story/press-spring styles deleted, the two functional CSS fallbacks shortened.

**Claude** (2026-09-27T07:42): _[tool: Bash]_

**Claude** (2026-09-27T07:42): The tokens part applied; the components part stopped on one pattern (a rule whose only declaration is the transition). Fixing that pattern and re-running the rest.

**Claude** (2026-09-27T07:43): _[tool: Bash]_

**Claude** (2026-09-27T07:43): _[tool: Bash]_

**Claude** (2026-09-27T07:43): _[tool: Bash]_

**Claude** (2026-09-27T07:44): Styleguide updated (no demo buttons for removed animations remain). Removing the now-unused `--leaf-groove` from the registry and regenerating.

**Claude** (2026-09-27T07:44): _[tool: Bash]_

**Claude** (2026-09-27T07:44): Updating `checks_p2.mjs` so it reports removed animations as removed instead of calling them (backed up first), and so it measures the new durations.

**Claude** (2026-09-27T07:44): _[tool: Bash]_

**Claude** (2026-09-27T07:44): Smoke-testing the pages after the cuts (console errors in all three modes), then the full T15 and page checks.

**Claude** (2026-09-27T07:44): _[tool: Bash]_

**Claude** (2026-09-27T07:45): No errors in any mode. Starting the T15 cross-browser/a11y/keyboard/reduced-motion check (≈7 min) in the background, then I'll write `MOTION_AUDIT.md` while it runs.

**Claude** (2026-09-27T07:45): _[tool: Bash]_

**Claude** (2026-09-27T07:45): _[tool: Bash]_

**Claude** (2026-09-27T07:46): While T15 runs, writing `design/realism/MOTION_AUDIT.md` (every animation, class, what was cut, what was kept and its new duration).

**Claude** (2026-09-27T07:46): _[tool: Write]_

**Claude** (2026-09-27T07:46): Audit written. Updating the `DESIGN_SYSTEM.md` motion section to match (it still documents the removed timelines).

**Claude** (2026-09-27T07:46): _[tool: Bash]_

**Claude** (2026-09-27T07:47): _[tool: Bash]_

**Claude** (2026-09-27T07:47): _[tool: Bash]_

**Claude** (2026-09-27T07:47): _[tool: Bash]_

**Claude** (2026-09-27T07:47): _[tool: Bash]_

**Claude** (2026-09-27T07:47): Docs updated. Logging Part C decisions while T15 finishes.

**Claude** (2026-09-27T07:47): _[tool: Bash]_

**Claude** (2026-09-27T07:48): _[tool: Bash]_

**Claude** (2026-09-27T07:48): _[tool: Bash]_

**Claude** (2026-09-27T07:49): _[tool: Bash]_

**Claude** (2026-09-27T07:49): _[tool: Bash]_

**Claude** (2026-09-27T07:49): While those run, starting Part D research: how the T3 leaf's gate renders are produced (visual judge render and lighting render).

**Claude** (2026-09-27T07:49): _[tool: Bash]_

**Claude** (2026-09-27T07:50): _[tool: Write]_

**Claude** (2026-09-27T07:50): _[tool: Bash]_

**Claude** (2026-09-27T07:51): _[tool: Bash]_

**Claude** (2026-09-27T07:51): _[tool: Bash]_

**Claude** (2026-09-27T07:53): _[tool: Bash]_

**Claude** (2026-09-27T07:53): Adding the opt-in sharp-shoulder groove and letter self-shadowing to `rig/stage.js` now.

**Claude** (2026-09-27T07:53): _[tool: Bash]_

**Claude** (2026-09-27T07:54): _[tool: Bash]_

**Claude** (2026-09-27T07:55): _[tool: Bash]_

**Claude** (2026-09-27T07:55): _[tool: Bash]_

**Claude** (2026-09-27T07:55): _[tool: Bash]_

**Claude** (2026-09-27T07:58): _[tool: Bash]_

**Claude** (2026-09-27T07:59): _[tool: Bash]_

**Claude** (2026-09-27T08:00): _[tool: Bash]_

**Claude** (2026-09-27T08:01): _[tool: Bash]_

**Claude** (2026-09-27T08:01): The checker assumes GSAP exists; realism-chola no longer loads it. Guarding that section and re-running chola.

**Claude** (2026-09-27T08:01): _[tool: Bash]_

**Claude** (2026-09-27T08:02): Chola check passes now. Comparing all four pages' `checks_p2` results and the identity check against the committed before-state.

**Claude** (2026-09-27T08:02): _[tool: Bash]_

**Claude** (2026-09-27T08:02): _[tool: Bash]_

**Claude** (2026-09-27T08:02): _[tool: Bash]_

**Claude** (2026-09-27T08:03): _[tool: Bash]_

**Claude** (2026-09-27T08:03): _[tool: Bash]_

**Claude** (2026-09-27T08:06): _[tool: Bash]_

**Claude** (2026-09-27T08:06): Wiring the groove shape and letter shadows into the T3 3D view's own shader and uniforms, so the interactive 3D leaf matches its judge render.

**Claude** (2026-09-27T08:06): _[tool: Bash]_

**Claude** (2026-09-27T08:07): _[tool: Write]_

**Claude** (2026-09-27T08:07): _[tool: Bash]_

**Claude** (2026-09-27T08:07): _[tool: Bash]_

**Claude** (2026-09-27T08:08): While the scoreboard runs, logging the Part D decisions (the depth cap was set before the depth probe; the log says so).

**Claude** (2026-09-27T08:08): _[tool: Bash]_

**Claude** (2026-09-27T08:08): Writing the screenshot-sheet stitcher while the scoreboard finishes.

**Claude** (2026-09-27T08:08): _[tool: Write]_

**Claude** (2026-09-27T08:08): _[tool: Bash]_

**Claude** (2026-09-27T08:08): _[tool: ToolSearch]_

**Claude** (2026-09-27T08:08): _[tool: Bash]_

**Claude** (2026-09-27T08:09): Waiting for the scoreboard run to finish (background watcher armed); the gate reads its output.

**Claude** (2026-09-27T08:09): _[tool: Bash]_

**Claude** (2026-09-27T08:09): _[tool: Bash]_

**Claude** (2026-09-27T08:09): Waiting for the scoreboard run to complete.

**Claude** (2026-09-27T08:09): _[tool: Monitor]_

**Claude** (2026-09-27T08:09): Scoreboard still running; I'll pick up the gate as soon as it lands.

**Claude** (2026-09-27T08:09): _[tool: Bash]_

**Claude** (2026-09-27T08:19): _[tool: Bash]_

**Claude** (2026-09-27T08:20): Scoreboard finished (722 s, exit 0, no step errors). Running the palm-leaf gate — the same `eval/gate.py`, same 7 conditions, same targets.

**Claude** (2026-09-27T08:20): _[tool: Bash]_

**Claude** (2026-09-27T08:20): _[tool: Bash]_

**Claude** (2026-09-27T08:20): _[tool: Bash]_

**Claude** (2026-09-27T08:20): _[tool: Bash]_

**Claude** (2026-09-27T08:20): _[tool: Bash]_

**Claude** (2026-09-27T08:21): _[tool: Bash]_

**Claude** (2026-09-27T08:21): _[tool: Bash]_

**Claude** (2026-09-27T08:21): Updating the T3 README and the design-system gate notes with the gate result.

**Claude** (2026-09-27T08:21): _[tool: Bash]_

**Claude** (2026-09-27T08:22): Taking the "after" screenshots (same script, same widths) and building the before/after sheet.

**Claude** (2026-09-27T08:22): _[tool: Bash]_

**Claude** (2026-09-27T08:22): _[tool: Bash]_

**Claude** (2026-09-27T08:22): _[tool: Read]_

**Claude** (2026-09-27T08:22): _[tool: Read]_

**Claude** (2026-09-27T08:23): _[tool: Bash]_

**Claude** (2026-09-27T08:25): _[tool: Read]_

**Claude** (2026-09-27T08:25): _[tool: Bash]_

**Claude** (2026-09-27T08:27): _[tool: Bash]_

**Claude** (2026-09-27T08:28): _[tool: Bash]_

**Claude** (2026-09-27T08:28): _[tool: Bash]_

**Claude** (2026-09-27T08:28): Self-checks pass (rig self-test and evaluator self-check both OK); scoreboard header: 45 of 390 scored rows fail. Committing Part D.

**Claude** (2026-09-27T08:28): _[tool: Bash]_

**Claude** (2026-09-27T08:29): _[tool: Write]_

**Claude** (2026-09-27T08:29): _[tool: Bash]_

**Claude** (2026-09-27T08:29): _[tool: Bash]_

**Claude** (2026-09-27T08:29): Prompt 6 is finished: Parts A to D are done on `realism-polish`, one commit each, and I've stopped there with no new work queued. The palm leaf stays 2D, because the 3D leaf still fails the gate's lighting condition.

## 1. Screenshot sheet
[SHEET.jpg](design/realism/polish/SHEET.jpg) shows the closed bundle, the open bundle, the coin card, every coin card, and the copper set closed and fanned, before and after, at 360/768/1200 px.
- **Copper ring:** it now goes through the hole. The back half is drawn under the plates and the front half over them. The position comes from the T5b physics scene, run once until it settled; nothing simulates on the page. The hole moved to 0.205 of the plate's length, the spec card's measured value (the old 6% was by eye). The single copper-plate card had the same fault and is fixed too.
- **Coin lettering:** every coin face is now rendered with the T4-coins material. The legend and rim marks are raised, with uneven stroke height, worn bright tops and dirt at their foot, all lit from the same top-left light. Gold and silver change only the colour. Those colours are measured from photos, but gold rests on 1 photo and silver on 6, so confidence is low.
- **Fanned leaves:** they now use the site's palm-leaf material, with holes, fibre grain and small per-leaf differences.

## 2. Orphan materials
- **Found:** 435 on `realism-engine` as it stood. Every one is listed in [ORPHANS_BEFORE.md](design/realism/polish/ORPHANS_BEFORE.md), which is the same check run just after Part A (436). The main groups were:
  - 95 material definitions spread over three CSS files with no single source.
  - 242 copies of one clay colour typed into every identity drawing.
  - Seals drawn in copper colours, and the signet ring in coin gold.
  - One-off gradients, e.g. the thread cord and the carved text.
- **Now:** 0. The only allowed exceptions are the 3 photo-credit swatches, which show the texture files themselves.
- **Check command** (documented in `DESIGN_SYSTEM.md`): `node design/realism/eval/orphan_materials.mjs`. It also fails if any generated file is out of date.
- **Visible changes:** the seal on the copper-plate card rings now uses the seal colours, and the 3D blocks' no-WebGL fallback now shows each block's own tuned colours.

## 3. Motion audit
[MOTION_AUDIT.md](design/realism/MOTION_AUDIT.md) lists every animation, what was cut, and what was kept.
- **Kept, shortened:**

  | Animation | Before | After |
  |---|---|---|
  | Leaf flip | 0.68 s | 0.24 s |
  | Bundle reader | 1.14 s | 0.34 s |
  | Copper-set fan | 0.70 s | 0.24 s |
  | Identity card | 646 ms | 238 ms |
- **Removed:** the intro, ink writing, stone carving, the pointer-following copper sheen, the ring spring, the scroll story and its dust, press springs, hover lifts, the emblem rise, the animation that delayed every bundle link before it navigated, the oil-lamp flicker, and all hover transitions.
- **T15 re-run, same or better:** 13/13 pages pass, reduced motion still shows 0.000 changed pixels, 0 axe violations, and Tab reaches every control. The styleguide now has 139 controls instead of 142, because the three demo buttons for removed animations are gone.

## 4. Palm-leaf gate
Same `eval/gate.py`, same 7 conditions, same targets: **6 of 7 pass, so it stays 2D.**
- **Visual:** 10/10, up from 8/10. The old excess texture direction came from the fibre streaks plus raking light on the leaf's surface noise, not the fibres alone. The fix was to use the tuned palm-leaf material with the fibre normal at 0.6× strength (about your 1.7× estimate).
- **Lighting: 3.72 dL\* against the 4.0 target, still failing** (up from 2.45). I gave the groove a sharp edge that casts a shadow into it, which is your "sharper edge" idea, and kept the same 0.144 mm cut.
- **Depth:** before probing, I capped the cut at a third of the 0.47 mm leaf (0.157 mm). At that cap it reaches 3.96, still short. Passing would need about 0.17 mm, more than half of a thin leaf, so I didn't do it.
- **Scoreboard:** 345 pass, 45 fail (was 340 / 42). The 3 extra failures come from fixing a check, not from any visual change: it had been capturing the copper plate's ring as the "signet ring". It now measures the real signet ring, whose original gold paint fails 7 of its 8 rows.

## 5. Not verified
- No real phone.
- No Safari, iOS or Firefox. Only Chrome for Testing 151 and the system Chrome were run.
- No screen reader. The coin faces are now canvases, each with a text label.
- No Tamil speaker has checked anything.
- Every new size, wear amount and the leaf cut depth is an estimate.
- The old GIF recordings of the removed animations are still in `_checks`.

The full report is in [P6_REPORT.md](design/realism/P6_REPORT.md). I'll wait for your review before starting anything else.