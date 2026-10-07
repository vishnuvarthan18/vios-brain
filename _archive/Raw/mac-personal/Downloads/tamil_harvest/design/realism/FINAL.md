# Realism Engine - final report (T16)

**Targets are missed: 42 of 382 scored rows FAIL** (340 pass, plus 2 rows that cannot be scored at all).
31 of the 42 failures are the *site 2D baseline* - the deliberate "before" measurement of the pages as they
stood before this engine ran, kept in the scoreboard so every improvement has a number next to it. **11 failures
are in work this engine produced**, and every one of them is listed below with its reason. Nothing in this report
has been checked by a human, by a Tamil reader, or against a real object; see "NOT verified" at the end.

Generated 2026-09-27 (run started 04:11:36 IST) on branch `realism-engine`.
Scoreboard: `design/realism/eval/scoreboard.md` / `.json`, written 2026-09-27T04:19:30+05:30 by
`node design/realism/eval/run_all.mjs` (453 s, exit 0, evaluator self-check **OK**).

---

## 1. Status of every task

33 tasks: **31 done, 1 skipped, 1 (this one) running**. No task is blocked.

| id | status | commit | one line |
|---|---|---|---|
| T0 Setup and self-test | done | 1d33258 | fixed rig; identical pixels + scores over 2 launches |
| T1 Material spec cards | done | 064c99b | 7 photo cards (rings 16 / seals 16 photos = low); thread + board estimates only; 74 low-confidence values |
| T2 Evaluators + self-tests | done | 0dfb099 | all self-checks behave as expected; 2D baseline 32/92 rows fail |
| T3a Palm leaf in real 3D: build | done | f9835fe | plain WebGL2, one classic script, 35.9 KB gzip of a 300 KB budget |
| T3b Palm leaf 3D: automatic gate | done | df2d9e5 | gate FAILS 2 of 7 -> keep 2D; 2D leaf shading raised to 10/10 visual rows |
| T4-palm_leaf | done | 96b97b7 | tuned 10/10 rows, loss 10.43 -> 2.76 |
| T4-stone | done | 8f1581e | tuned 6/6 rows, loss 2.11; lighting 21.8 dL* |
| T4-copper_plate | done | f288ac6 | tuned 7/7 rows, loss 2.74 -> 1.76; lighting 7.39 dL* |
| T4-pottery | done | 1fe2a16 | tuned 7/8 rows, loss 16.17 -> 4.24 |
| T4-coins | done | 3b00e1a | tuned 8/8 rows, loss 4.63 -> 2.98; lighting 15.42 dL* |
| T4-rings | done | 48d4dad | tuned 5/8 rows, loss 27.27 -> 9.50; 3 rows FAIL and are reported |
| T4-seals | done | fa03a34 | tuned 7/8 rows, loss 25.31 -> 4.44; lighting 26.38 dL* |
| T4-thread_board | **skipped** | - | **no reference photos exist** (0 for cotton_thread, 0 for wood_board), so there is no colour or texture spec to tune against and visual tuning is impossible. Both cards hold physical estimates only (10 estimated values). Physics for both is built and tested (T5a). |
| T5a Physics: leaf, bundle, thread | done | a67abb3 | bundle + thread scenes, tests, demo page; all rows as expected |
| T5b Physics: copper plate set on its ring | done | 854ee99 | 21 plates; worst period error 0.064% of a 3% target; holes 0.19 mm off the 0.5 mm seat limit |
| T5c Physics: coin, ring, seal | done | 36b6bb2 | 60 new rows, all as expected |
| T5d Physics: sherd and stone | done | 80240ee | sherd scene + site-scale stone; all rows as expected |
| T6a Lighting: stone, raking + oil lamp | done | d3a1bb3 | lighting PASS (el 35 dL 20.39; raking el 15 dL 24.39); painted control correctly FAILS |
| T6b Lighting: copper plate | done | bb29d95 | lighting PASS (6.35 / 9.63 dL); visual 7/7; painted control FAILS |
| T7 Wear and age system | done | 9fdf079 | one age 0..1 drives cracks, stains, edge loss, darkening; default 0.25; 11/11 wear rows pass |
| T8 Optional sound layer | done | f82dcef | 4 synthesized sounds, muted by default, no AudioContext on load; 24/24 checks |
| T9 Styleguide rebuild | done | e2214ec | 3D copper + stone next to the 2D "before"; 25/25 checks |
| T9b Integrate gate + tuned materials | done | d914bde | 8 gates run: adopt 3D for stone, copper, coins; keep 2D for leaf, pottery, rings, seals |
| T10 Components + dark mode + contrast | done | 0f7312f | 182/182 token pairs and 44/44 rendered pairs pass light + dark; worst 4.78:1 day, 5.68:1 night |
| T11a Home page | done | db5e1ad | palm-leaf bundle as navigation; perf p95 18.7 ms, compat 7/7 |
| T11b Chola section page | done | eb6ba63 | stone wall by lamplight, 3D stone; perf p95 18.7 ms with the lamp |
| T11c Thirukkural leaf reader | done | 5d503e6 | 10 kural quoted verbatim from this repo's Project Madurai file; glosses unverified |
| T12a Identity batch 2 | done | 1e8607b | +54 items (42 -> 96); 96/96 SVG checks |
| T12b Identity batch 3 | done | bf1f64d | +25 items (96 -> 121); 121/121 SVG checks; page 1.385 MB of a 1.5 MB budget |
| T13 Reference viewer | done | f438222 | 2,412 rows, 896 SHIP / 1,516 STUDY, refs.db opened read-only; 18/18 checks |
| T14 Licence cleanup report | done | fc1a83e | 271 findings over 267 files; **7 blocking** (SHIP CC BY rows with no author) |
| T15 Cross-browser / a11y / keyboard | done | f737fd1 | 13 of 13 pages: 0 console errors, 0 axe violations, reduced motion 0.000 changed pixels - **in one browser only** |
| T16 Final report | this run | - | this file |

---

## 2. Full scoreboard summary

382 scored rows: **340 PASS, 42 FAIL**. By category:

| category | pass | fail |
|---|---|---|
| physics | 128 | 0 |
| visual | 98 | 35 |
| compat | 84 | 0 |
| lighting | 7 | 7 |
| perf | 12 | 0 |
| wear | 11 | 0 |

Per material (best subject shown; "visual" = colour dE2000 median / p95):

| material | visual (dE median / p95) | visual rows | physics rows | site perf p95 |
|---|---|---|---|---|
| palm_leaf | 3.77 / 6.84 (T3b 2D leaf, no text) | 10/10 | 16/16 | 17.7 ms |
| stone | 6.00 / 11.26 (T4 tuned stage) | 6/6 | 5/5 | 17.7 ms |
| copper_plate | 2.63 / 5.76 (T4 tuned stage) | 7/7 | 25/25 | 17.7 ms |
| pottery | 7.34 / 11.25 (T4 tuned stage) | 7/8 | 23/23 | 17.7 ms |
| coins | 2.81 / 10.15 (T4 tuned stage) | 8/8 | 8/8 | 17.7 ms |
| rings | 11.65 / 13.87 (T4 tuned stage) | 5/8 | 8/8 | 17.7 ms |
| seals | 5.75 / 7.36 (T4 tuned stage) | 7/8 | 22/22 | 17.7 ms |
| cotton_thread | **not scorable** (no reference photos) | - | 16/16 | 17.7 ms |
| wood_board | **not scorable** (no reference photos) | - | 5/5 | 17.7 ms |

Physics is 128 of 128 rows, every reference scene behaving as expected and every known-bad variant failing as it
should. **The physics engine is not wired into the website**: the scoreboard's physics rows are engine reference
scenes and the four T5 demo pages, not the shipped pages.

### Every failing row

**(a) Site 2D baseline - the "before" number, 31 rows (26 visual + 5 lighting).** These are the pages as they were before this engine
ran, kept on purpose so the improvement is measurable. They are not regressions.

| material | test | value | target |
|---|---|---|---|
| coins | colour_dE2000_median | 28.37 | < 6.0 |
| coins | colour_dE2000_p95 | 37.37 | < 12.0 |
| coins | colour_mean_in_photo_range | [68.1, 3.5, 46.1] | outside the photo range |
| coins | spectral_slope_diff | 2.14 | < 0.3 |
| coins | anisotropy_diff | 0.239 | < 0.15 |
| coins | local_contrast_ratio | 0.427 | 0.8..1.25 |
| coins | grain_direction_diff_deg | 57.93 | < 15.0 |
| copper_plate | colour_dE2000_median | 23.41 | < 6.0 |
| copper_plate | colour_dE2000_p95 | 26.09 | < 12.0 |
| copper_plate | colour_mean_in_photo_range | [67.8, 12.4, 25.5] | outside |
| copper_plate | spectral_slope_diff | 0.658 | < 0.3 |
| copper_plate | local_contrast_ratio | 0.485 | 0.8..1.25 |
| palm_leaf | texture_measurable | false | true |
| pottery | spectral_slope_diff | 0.339 | < 0.3 |
| pottery | grain_direction_diff_deg | 30.39 | < 15.0 |
| rings | render_measurable | false | true |
| seals | colour_dE2000_median | 6.92 | < 6.0 |
| seals | spectral_slope_diff | 2.38 | < 0.3 |
| seals | anisotropy_diff | 0.330 | < 0.15 |
| seals | local_contrast_ratio | 0.551 | 0.8..1.25 |
| seals | grain_direction_diff_deg | 36.45 | < 15.0 |
| seals | aspect_ratio | 1.008 | 1.009..2.465 |
| stone | colour_dE2000_median | 6.27 | < 6.0 |
| stone | spectral_slope_diff | 0.336 | < 0.3 |
| stone | anisotropy_diff | 0.180 | < 0.15 |
| stone | local_contrast_ratio | 0.388 | 0.8..1.25 |
| + 5 lighting rows | shading_changes_with_light_angle | "no light input: fixed CSS shadows" | >= 4 dL* over 5 angles |

(the 5 baseline lighting rows are stone, copper_plate, coins, seals, palm_leaf: flat CSS has no light input at all,
which is the whole reason the 3D stack exists.)

**(b) This engine's own work - 11 rows.** Each one is reported rather than tuned away; the reason is a logged
DECISION in every case.

| material | subject | test | value | target | why it stands |
|---|---|---|---|---|---|
| palm_leaf | T3 3D leaf | anisotropy_diff | 0.189 | < 0.15 | the shader's fibre streaks are ~1.7x too directional (spec 0.2518, render 0.44). 73.1% of real palm-leaf photos pass this target, so it is a real miss. The gate therefore chose **2D**. |
| palm_leaf | T3 3D leaf | grain_direction_diff_deg | 17.46 | < 15.0 | same cause |
| palm_leaf | T3a 3D leaf page | lighting dL* | 2.448 | >= 4 | 4 dL* needs cutting more than half way through a 0.47 mm leaf. Palm-leaf letters are read because they are rubbed with soot, not because they cast shadows. Deepening to pass would invent a scribe who cut through the leaf. |
| palm_leaf | T4 tuned stage, carved | lighting dL* | 2.841 | >= 4 | same, after the tuner (better than T3a's 2.448, still failing) |
| pottery | T4 tuned stage | colour_dE2000_median | 7.34 | < 6.0 | only 23.3% of real pottery photos pass this target |
| rings | T4 tuned stage | colour_dE2000_median | 11.65 | < 6.0 | **0 of 16 real ring photos pass this target.** 16 photos is below the 20-photo floor; the colour card is low confidence. |
| rings | T4 tuned stage | colour_dE2000_p95 | 13.87 | < 12.0 | same, 0/16 real photos pass |
| rings | T4 tuned stage | aspect_ratio | 1.000 | 1.101..2.304 | `rig/stage.js` forces aspect = 1 for the ring shape; a ring lying face-on *is* a circle, and no edge roughness reaches 1.101 before the render stops being measurable (5 values probed). |
| seals | T4 tuned stage | aspect_ratio | 1.005 | 1.009..2.465 | same for the disc shape; every edge_rough that reaches 1.009 breaks grain_direction instead (7 values probed) |
| stone | T6a page render | colour_dE2000_median | 6.46 | < 6.0 | the T6a probe params; the later T4 tuned stone reaches 6.00 and passes |
| stone | T6a page render | colour_dE2000_p95 | 13.2 | < 12.0 | same |

---

## 3. Gate results (Prompt 4 section 7)

`design/realism/eval/gates/*.json`. A candidate is adopted only if it passes **every** condition
(visual rows, physics, JS budget, frame time, console/network errors, fallback).

| object | decision | why |
|---|---|---|
| **stone** | **adopt 3D** | 6/6 visual rows; lighting 21.80 dL*; tuner loss 2.11 |
| **copper_plate** | **adopt 3D** | 7/7 visual rows; lighting 7.39 dL*; dE median 2.63 |
| **coins** | **adopt 3D** | 8/8 visual rows; lighting 15.42 dL*; dE median 2.81 |
| palm_leaf (3D bundle) | keep 2D | 8/10 visual rows + lighting 2.448 dL* |
| palm_leaf_tuned | keep 2D | 10/10 visual rows but lighting 2.841 dL* - so the site keeps the 2D leaf, which the T3b work raised from 7/10 to 10/10 rows itself |
| pottery | keep 2D | colour dE2000 median 7.34 |
| rings | keep 2D | 5/8 visual rows |
| seals | keep 2D | aspect_ratio 1.005; the 2D seal was re-coloured from the tuned ramp instead (dE median 24.11 -> 6.92) |

Three of eight objects earned 3D. Five did not, and the pages say so on the page itself.

Other gates: perf 12/12 (every page p95 under the 20 ms target at 4x CPU throttle), compat 84/84 rows
(0 console errors, 0 failed `file://` requests, renders with WebGL off and at quality=low), wear 11/11,
contrast 182/182 token pairs and 44/44 rendered pairs in both light and dark, cross-browser 13/13 pages
(one browser only - see below).

---

## 4. All commits on `realism-engine`

`git log --oneline redesign-design-system..HEAD` - 41 commits, oldest last:

```
f0f6f02 Realism T15: state files
f737fd1 Realism T15: cross-browser, a11y, keyboard, reduced motion, file://
fc1a83e Realism T14: license cleanup report
f438222 Realism T13: reference viewer
95eb820 Realism T12b: state files
bf1f64d Realism T12b: Tamil Identity batch 3 (25 new items, 96 -> 121)
1e8607b Realism T12a: Tamil Identity batch 2 (54 new items, 42 -> 96)
37e6325 Realism T11c: state files
5d503e6 Realism T11c: website_live/realism-kural.html (Thirukkural leaf reader)
7c65ef8 Realism T11b: state files
eb6ba63 Realism T11b: website_live/realism-chola.html (stone wall by lamplight)
9fed53b Realism T11a: state files
db5e1ad Realism T11a: website_live/realism-home.html (bundle as navigation)
a6451a2 Realism T9b: state files
d914bde Realism T9b: gate run for all 8 candidates (3 adopt 3D, 5 keep 2D)
fa03a34 Realism T4-seals: 7/8 rows, loss 25.31 -> 4.44
48d4dad Realism T4-rings: 5/8 rows, loss 27.27 -> 9.50
3b00e1a Realism T4-coins: 8/8 rows, loss 4.63 -> 2.98
1fe2a16 Realism T4-pottery: 7/8 rows, loss 16.17 -> 4.24
f288ac6 Realism T4-copper_plate: 7/7 rows, loss 2.74 -> 1.76
8f1581e Realism T4-stone: 6/6 rows, loss 2.11
96b97b7 Realism T4-palm_leaf: auto-tuner + leaf 10/10 rows, loss 2.76
df2d9e5 Realism T3b: palm leaf gate FAILS 2 of 7 -> keep 2D
f9835fe Realism T3a: palm leaf in real 3D (plain WebGL2, 35.9 KB gzip)
31b6f5c Realism supervisor: read HTTP 429 as waits, not crashes; requeue 18; add T9b
bd5cb87 Realism T10: state files
0f7312f Realism T10: components + dark mode + contrast tables
e2214ec Realism T9: styleguide rebuilt on the measured stack
f82dcef Realism T8: optional sound layer, muted by default
9fdf079 Realism T7: wear and age system (one age 0..1)
bb29d95 Realism T6b: copper plate under raking light and an oil lamp
d3a1bb3 Realism T6a: stone under raking light and an oil lamp
80240ee Realism T5d: sherd and stone physics
36b6bb2 Realism T5c: coin, ring and seal physics
854ee99 Realism T5b: copper plate set on its ring (21 plates)
a67abb3 Realism T5a: bundle + thread physics scenes, tests, demo page
602a1f8 Realism supervisor: denials block only after a normal end
45f35b0 Realism supervisor: run_forever.sh, agent brief, 29 task files, queue
0dfb099 Realism T2: evaluators, reference physics engine, scoreboard
064c99b Realism T1: material spec cards from the reference library
1d33258 Realism T0: fixed test rig, measuring library, state tools, self-test
```

(T16's own commit is not in this list; it is the one that adds this file.)

---

## 5. Decisions

**105 decisions** are recorded in full, with options, choice and reason, in `design/realism/DECISIONS.md`
(2026-09-26 08:35 to 2026-09-27 04:09). They are not repeated here; the ones that most change what a reader sees:

- **No 3D library.** three.js ships only ES-module builds since r160 and every bundler is denied here, so the 3D
  leaf and the site stage are plain WebGL2 reusing the rig's shader (~30-36 KB gzip instead of ~165 KB).
- **Blocked counts as resolved** for task ordering: a later task works around a blocked dependency rather than
  stalling the queue.
- **Physical constraints beat score.** The tuner was pinned to the material's own physics for palm leaf, stone and
  pottery (metal = 0, grain angle within the spec card's own range) rather than letting it buy points with a 26%
  metallic leaf. Copper, coins, rings and seals *are* metal, so metal searched freely there.
- **Failures are reported, not tuned away.** The palm-leaf lighting row, the ring and seal aspect rows and the
  ring colour rows were all left failing with their physical reason instead of being forced to pass.
- **Sound starts muted with no persistence**; no AudioContext before a user gesture.
- **Dark mode rebinds page roles only.** Material colours are measured from photos and do not change with the theme.
- **Tamil text is quoted only from this repo's own data**, with the file cited on the page; everything else is
  flagged unverified.
- **Reference photos are read-only.** No photo of any tier was copied into `website_live/`, and no thumbnails were
  written into `design/references/`.

---

## 6. Low-confidence numbers

**74 low-confidence values** are listed material by material in `design/realism/specs/README.md`, each with its
range, sample count and reason. The important ones:

- **Photo counts below the 20-photo floor:** rings 16 used, seals 16 used. Their colour and texture cards are low
  confidence by construction, and rings fails 3 scored rows.
- **cotton_thread and wood_board have 0 reference photos.** No colour or texture card exists; 10 physical values
  are estimates (cord diameter 1.5 mm, fibre density 1.54, packing 0.6, linear mass 1.6 g/m, max stretch 2%,
  friction 0.5; board thickness 8 mm, density 0.65 g/cm3, friction 0.4, restitution 0.3). T4-thread_board was
  skipped for this reason.
- **Colour comes from uncalibrated photos** with unknown white balance. Colour confidence is never "high" on any
  card. `between_photo_sd` is partly camera, not material.
- **No scale bars were detected in any photo**, so every texture size is relative, not millimetres. The T13
  viewer's ruler reports image pixels for the same reason.
- **Estimated geometry, each logged and flagged:** the charter ring (centreline radius 90 mm, wire radius 2.0 mm),
  seal thickness 8 mm and relief 0.5 mm, sherd long axis 8 cm, coin legend relief 0.20 mm, the copper groove
  1.2 mm, the stone groove 5 mm, the coin flan wobble +-8%, ring inner diameter 18 mm / mass 8 g / restitution 0.4,
  seal diameter 30 mm / density 8.7 g/cm3.
- **Wear amounts are all estimates:** crack frequency 22/unit, line half-width 0.05 at age 1, stain frequency 2.3,
  edge loss 2% of the long side at age 1, 18% L* darkening at age 1. No source measures crack density or stain
  area on these objects.
- **Targets stricter than reality.** `eval/self_test/real_photo_pass_rates.json` measures how often a *real photo*
  passes each target (leave-one-out). Several targets are stricter than photo-to-photo variation: spectral slope
  is passed by only **13-35%** of real photos, copper colour dE median by 22.7%, pottery by 23.3%, stone by 27.4%,
  seals by 25.0%, and **ring colour by 0 of 16**. The owner's targets were **not** changed; the pass rate is
  printed next to every visual row instead.
- **Tuned, not robust:** the 2D leaf's spectral slope is sensitive to one `baseFrequency` value (1.15 -> -1.45,
  1.20 -> -1.20). It is a tuned number.
- **One scoreboard row moves with page length:** the 2D pottery capture lands on different device pixels when a
  page grows, which moved `spectral_slope_diff` between 0.291 and 0.303 against a 0.3 target.

---

## 7. Licences

`design/reference_engine/NEEDS_CLEANING.csv` - **271 findings over 267 distinct files**, generated read-only
(`LICENSES.csv` and `refs.db` shasums identical before and after).

- **7 blocking rows: SEA-002, SEA-003, SEA-020, PAL-145, PAL-149, PAL-164, BRO-383** are SHIP / CC BY 4.0 with
  **no author at all**, and CC BY requires attribution. They must not go on the website until a name is recovered.
- 179 author_credit problems in total (96 placeholder, 70 empty, 8 boilerplate paragraphs, 5 holding a URL).
- 25 duplicate ids over 53 rows; 11 of those groups mix SHIP and STUDY rows, so a lookup by id alone can return
  the wrong tier and licence.
- 1 empty licence (RIN-032), 8 licences outside the CC rules (5 GODL-India, 3 "No restrictions"), 2 ported strings.
  All are tiered STUDY, so none can ship today.
- STO-070 / STO-071 are one file on disk (macOS case-insensitive paths); one row is a phantom.
- Tools installed for this engine are listed with their licences in `design/realism/LICENSES_TOOLS.md`
  (axe-core 4.12.1 MPL-2.0 and opencv-python-headless are dev-only and are never shipped).
- **No reference photo of any tier was copied into `website_live/`.** No sound file, font, image or library was
  used without a recorded licence.

---

## 8. NOT verified

Everything in this list is unchecked. Nothing below should be treated as a result.

**Devices and browsers**
- **No real phone.** All "mobile" numbers are a 360x740 DPR 2 emulation on a desktop GPU with software rendering
  (SwiftShader). No touch input, no phone GPU, no thermal behaviour, no real network.
- **No Safari, no iOS, no Firefox.** Exactly one browser is installed - the pinned Chrome for Testing 151. WebKit
  and Firefox are not in the cache and were not downloaded. `CROSS_BROWSER.md` says so; nothing in this repo is
  evidence about any other engine.
- No real screen reader. **0 axe violations is not the same as accessible** - axe cannot read text drawn inside
  the WebGL canvases at all, and the canvases carry Tamil letters.
- No high-refresh or low-end hardware, no colour-managed display, no printed output.

**The objects themselves**
- **No real object has been measured by hand.** Every physical number is either from a named published source
  (listed in `specs/sources/physical_sources.json`) or a logged estimate. No palm leaf, copper plate, coin, ring,
  seal, sherd or stone was weighed, measured with calipers or photographed with a scale bar for this work.
- **No scale bar exists anywhere in the reference library**, so no texture size is in millimetres.
- Colour is from uncalibrated photos with unknown white balance and no colour chart.
- The physics is not validated against a filmed real object: it is validated against analytic solutions
  (pendulum period, catenary), determinism, frame-rate independence and known-bad controls.

**Tamil content**
- **No Tamil speaker has checked anything.** Every Tamil line, every English gloss, every transliteration, every
  date and every name on every page is unverified.
- The 10 Thirukkural couplets are quoted character-for-character from this repo's Project Madurai file, but the
  **English meanings are my own wording** and the ISO-15919 romanisation is **rule-based, generated, not
  hand-checked** (it reproduces the 3 hand transliterations already in the styleguide, which is not proof).
- The Purananuru attribution of the home page's Tamil line is **not confirmed** by the file it was quoted from.
- Every other Tamil line on the Chola page is my own plain wording, not a transcription.
- **121 identity items, 0 rows marked verified:true.** 81 carry an unverified period, which the page hides.
  The named monuments are schematics of the building *type*, not measured proportions.
- `design/CONTENT_TO_VERIFY.md` is the list a Tamil reader should work through.

**Licences**
- Every licence, author and date is **what the source API recorded**. None has been checked by a human.
  A row absent from `NEEDS_CLEANING.csv` is **not** thereby verified.
- The 7 blocking CC BY rows above are the known hard stop before anything ships.

**Scope**
- The physics engine is **not wired into the website pages**; it runs in four demo pages and in the test harness.
- The three pages (home, Chola, Kural) carry **sample content**, not a finished site.
- The word "real" appears in this project only next to a score. Nothing here has been judged real by a person.
