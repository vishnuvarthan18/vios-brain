# Motion audit — Prompt 6, Part C

Every animation on the new design-system site (`website_live/`: styleguide, realism-home, realism-chola,
realism-kural, identity, and the CSS/JS they load), classified and then cut to the minimum.

**Rule applied.** FUNCTIONAL = shows a state change the visitor asked for (open/close the bundle reader, flip a
leaf, expand an identity card, fan the copper set). Everything else is DECORATIVE and was **removed entirely**,
not replaced by a smaller version. Every functional animation was shortened: nothing is longer than 400 ms, and
every single tween is 250 ms or less (the only totals above 250 ms are sequences of short steps: the bundle reader
0.34 s). Nothing new was added. `prefers-reduced-motion: reduce` still sets every final state at once (GSAP
`matchMedia` path) and forces every CSS duration to 0 (`css/tokens.css`).

Out of scope, untouched: the 7 old pages and their own `css/style*.css` / `js/app*.js` (the owner's rule), and the
realism engine's demo pages in `design/realism/t3|t5|t6|t8` (they are measuring rigs, not the site).

Durations "before" are read from the code at `realism-engine` (commit 8e4b12f); "after" from this commit, and
the GSAP ones are also measured by `website_live/_checks/checks_p2.mjs` (results in `_checks/results-*.json`,
`motion` block).

## Kept — FUNCTIONAL (shortened)

| # | animation | where | trigger | before | after |
|---|---|---|---|---|---|
| F1 | Leaf flip, next: top leaf swings off on its hole, next leaf rises | `js/motion.js` `Motion.leafFlip` | Next button, → key, drag left | 0.68 s (lift 0.16, swing 0.52, two-part skew bend, fade, lift shadow, rise 0.45) | **0.24 s** (swing-off 0.22 + rise 0.20, overlapped; bend and lift shadow removed) |
| F2 | Leaf flip, previous | same | Prev button, ← key, drag right | 0.70 s | **0.22 s** |
| F3 | Leaf goes back to rest after a short drag | same (`settle`) | drag released under the threshold | 0.60 s elastic spring | **0.18 s** ease-out, no spring |
| F4 | Bundle reader open: thread off, top board opens, 5 leaves fan, reader shows | `Motion.untieBundle` (reader bundles only) | click / Enter / Space on a `button.leaf-bundle` | 1.14 s (knot morph, DrawSVG unwind of two threads, board back-out bounce, fan stagger 0.06, reader slide-in) | **0.34 s** (thread fade 0.10, board 0.20, fan 0.20 + stagger 0.02, reader fade 0.16; knot morph, thread drawing, bounce and slide removed) |
| F5 | Bundle reader close | same, reversed | Close button, Escape, click again | 1.14 s | **0.34 s** |
| F6 | Copper set fans open / closed around its ring | `Motion.fanPlates` | "Fan out the plates" button | 0.70 s (back-out overshoot, stagger 0.05) | **0.24 s** (0.20 + stagger 0.02, ease-out, pivot = the hole from the T5b rest pose) |
| F7 | Identity card grows into its detail panel | `Motion.flipOpen` (GSAP Flip) | click / Enter on a card | about 0.7 s (Flip 0.55, fields fade 0.30 after 0.30 with a stagger); `checks_identity` measured 640 ms | **0.22 s** (Flip 0.22, fields fade 0.12 after 0.10, no stagger) |
| F8 | Identity detail closes back into its card | same | Close, Escape | 0.45 s | **0.20 s** |
| F9 | Leaf flip, no-GSAP fallback | `css/components.css` `@keyframes leaf-out / leaf-in`, `js/components.js` | same as F1/F2 without GSAP | 350 ms (175 + 175) | **240 ms** (120 + 120), `--dur-flip` |
| F10 | Copper set fan, no-GSAP fallback | `.copper-set__plate` transition | same as F6 without GSAP | 500 ms (`--dur-untie`) | **200 ms** (`--dur-open`) |

Also kept, and **not** animations: dragging a leaf (it follows the pointer — direct manipulation, no timeline);
instant hover states (colour, underline, shadow: no transition); pressed buttons and cards sitting 1 px lower while
pressed (a state, no transition); the stylus cursor over leaves (a still cursor image).

## Removed — DECORATIVE

| # | animation | where | what it did | before |
|---|---|---|---|---|
| D1 | Page-load intro | `Motion.intro`, `.intro*` CSS, styleguide "Play the intro" button | a leaf slid in, holes glinted, the title was inked, then faded | 1.75 s, once per session (`?intro=1` or `<body data-intro>`) |
| D2 | Ink writing | `Motion.inkWrite`, `[data-ink]`, `.ink-line/.ink-copy/.ink-nib` CSS, "Write again" button | an iron nib scratched and inked each line when scrolled into view | 1.15 s per block |
| D3 | Stone carving | `Motion.carveStone`, `[data-carve]`, `.is-carving/.is-deepening/.carve-dust` CSS, "Carve again" button | letters struck one by one with dust, then the groove faded in | about 1.0-1.2 s |
| D4 | Copper sheen following the pointer / device tilt | `Motion.copperSheen` (every `.copper-plate`, `[data-sheen]`, last copper-set plate) | the highlight band chased the cursor (0.45 s quickTo) | continuous while hovering |
| D5 | Ring spring | `Motion.ringSpring` | the plate's ring swung on its hole with an elastic spring on first scroll into view | 1.1 s |
| D6 | Pinned scroll story with parallax | `Motion.scrollStory`, `[data-story]`, `.story.is-pinned`, `.story__layer*` CSS and markup | pinned the section for 270% of a screen, stages slid and scaled, back layers moved at other speeds | scroll-scrubbed |
| D7 | Dust drift (the one ambient loop) | inside D6, `.story__dust` | 14 faint specks drifting forever | 6 s yoyo, infinite |
| D8 | Button and chip press spring | `Motion.micro` (`--press` scale) | pressed controls sprang back elastically | 0.08 s in + 0.55 s out |
| D9 | Card hover lift / coin and sherd tilt (GSAP) | `Motion.micro` (`--card-y`, `--card-r`) | cards rose 4 px, coins and sherds tilted 3° with a spring | 0.35 s in + 0.5 s out |
| D10 | Emblem rise on the identity page | `Motion.rise`, `[data-rise]` | the three hero emblems rose and settled on load | 1.06 s (0.7 + 2 x 0.18 stagger) |
| D11 | Bundle **link** untie before navigating | `js/components.js` `initBundle` (GSAP path and CSS path) | clicking a bundle link played the untie, THEN followed the link | 0.76 s (GSAP) / about 0.6 s (CSS) added before every navigation |
| D12 | Oil-lamp flicker | `js/realism/materials3d.js` rAF loop | re-rendered the WebGL stage every frame while the lamp was on — from page load on realism-chola, whose wall starts lamp-lit | continuous |
| D13 | Hover transitions (colour, underline, background, shadow, filter) | `css/components.css` links, `.button`, `.chip`, `.topnav__link`, object card links, `.leaf-strip__surface`, `.leaf-bundle__title/__art`, `.coin-card__coin`, `.sherd-card__piece`, `button.leaf-bundle`, `.theme-switch__btn`, `.subnav__link`, `.card__title a` | eased every hover change | 150 ms each (`--dur-hover`) |
| D14 | CSS hover lifts and tilts | `.button`, `.chip`, object card links (`translateY(-2px)`), coin and sherd cards (`rotate(-3deg)`), identity `.id-card__btn` | moved the control on hover | 150 ms |
| D15 | Stone slab shadow shift on hover | `.stone-slab::after` (`css/materials.css`, `css/components.css`) | the drop shadow moved as if the slab lifted | 150 ms |
| D16 | CSS untie states | `.leaf-bundle.is-untying / .is-open / .is-reset`, board/stack/thread transitions | the no-GSAP half of D11 | 500 ms + 200 ms |

Removed with them: the GSAP plugins nothing uses any more (ScrollTrigger, DrawSVGPlugin, MorphSVGPlugin,
SplitText, CustomEase — not loaded by any page now; Draggable only where there is a leaf stack; realism-chola
loads no GSAP at all), the `--dur-hover`, `--dur-untie` and `--lift` tokens, the `--leaf-groove` registry colour
(only the ink animation used it), and `Motion.restoreThread`.

## What moves now, in total

| page | what can move | longest |
|---|---|---|
| styleguide.html | F1-F6, F9-F10 | 0.34 s (bundle reader) |
| realism-home.html | F4-F5 (the reader bundle); the five bundle links navigate at once | 0.34 s |
| realism-kural.html | F1-F3, F9 | 0.24 s |
| realism-chola.html | nothing (the lamp is steady; the light sliders re-render on input only) | — |
| identity.html | F7-F8 | 0.22 s |

Nothing plays on its own, on load, on scroll or on hover. There is no loop anywhere.

## Checks after the change (same numbers or better, never worse)

Measured timelines (`_checks/results-*.json`, `motion` block): leaf flip **249 ms** end to end on realism-kural and
260 ms on the styleguide (0.24 s tween + the check's 10 ms polling), bundle reader **0.34 s**, copper-set fan
**0.24 s**, identity card open **238 ms** (was 646), longest tween on any page 0.24 s, ambient loops **0**, a bundle
link animates before navigating: **no**. `Motion.intro/inkWrite/carveStone/copperSheen/ringSpring/scrollStory/micro/rise`
are gone.

| check | before (Part B) | after (Part C) |
|---|---|---|
| T15 `eval/cross_browser.mjs`: pages passing | 13 / 13 | **13 / 13** |
| T15 reduced motion, changed pixels over 2 s | 0.000 on all 13 | **0.000 on all 13** |
| T15 console errors / failed / external requests / axe violations | 0 / 0 / 0 / 0 | **0 / 0 / 0 / 0** |
| T15 Tab reaches every control | styleguide 142/142, home 42/42, chola 41/41, kural 43/43, identity 24/144 (roving, by design) | styleguide **139/139** (the 3 demo buttons of removed animations are gone), home 42/42, chola 41/41, kural 43/43, identity 24/144 |
| T15 focus ring visible (sampled) | 8/8 per site page | **8/8** |
| checks_p2 reduced motion: GSAP busy ticks / elements with a running transition | 0 / 0 on every page | **0 / 0** |
| checks_p2 keyboard in order, missing rings | styleguide 142 & 129, home 28/41 & 22/35, kural 43 & 37, chola 41 & 35; 0 missing | styleguide 139 & 126, the rest identical; **0 missing** |
| checks_p2 overflow / Tamil type / small targets at 360-768-1200 | none | **none** |
| checks_identity keyboard (arrows, Home, End, Enter, Space, Escape, focus return, dialog trap) | all pass | **all pass** |
| identity page weight | 1.402 MB | **1.365 MB** (CustomEase no longer loaded) |
