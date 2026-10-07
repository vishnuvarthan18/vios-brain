# PROMPT 7 — Clean the design system: remove dev scratchpad, keep it neat

Branch `realism-clean` off `realism-polish`. Do not touch the 7 old pages. This is a cleanup pass only — no new
features, no new materials, no new animation. If something needs fixing beyond removal/tidying, list it instead
of fixing it here.

## The problem
Engineering QA detail is visible on pages meant to be shown to people, not just in `styleguide.html`:
- `realism-chola.html`: shows "colour dE2000 median / p95 (targets 6.0 / 12.0)" and "chained runs, loss 2.11,
  which the section 7 gate adopted in 3D over the flat CSS stone: colour ramp CIELAB..." directly in the page.
- `realism-kural.html`: shows the same dE2000 numbers plus a "Why 2D" panel explaining the gate's anisotropy and
  lighting failure with raw numbers and file paths (`eval/gates/palm_leaf.json`).
- `styleguide.html`: is currently a before/after comparison scratchpad — every component shown twice (old flat
  vs. tuned), plus paragraphs of measurement notes ("measured on SHIP photos COP-001, -002...", check counts,
  gate tables, T9b/T4 task references).

None of this belongs on a page meant to look finished. It reads as an internal test report, not a design system.

## Two categories — do not mix them up
1. **Engineering/QA commentary — REMOVE from every visible page.** Anything that references: dE2000 numbers,
   gate pass/fail conditions, task IDs (T4, T9b, T15, etc.), file paths like `eval/gates/*.json`, "chained runs",
   loss values, check counts ("32/32 pass"), photo ID citations (COP-001, STO-095, etc.), or "section 7 gate".
   This is real and valuable — it does NOT get deleted, it gets MOVED into `DESIGN_SYSTEM.md` (which already
   holds some of it) so nothing is lost, it's just off the visible page.
2. **Content honesty flags — KEEP on every visible page.** Anything marking a fact as unverified, a sample, or
   a placeholder for the reader's benefit ("sample verse", "unverified", "symbolic, not a portrait", "placeholder
   until the identity kit supplies real emblems"). These are not engineering scratchpad — they are honest labels
   for content accuracy, and stay exactly as they are. Do not remove or water these down.

If you're unsure which category something is, ask: would a visitor with no interest in how the site was built
care about this sentence? If no, it's category 1, move it. If it's telling them the CONTENT (not the code) isn't
verified yet, it's category 2, keep it.

## Part A — `styleguide.html`: stop being a before/after page
1. Remove every before/after comparison block (`.sg-cmp` and similar): no component should be shown twice. Show
   only the final, current version of each component.
2. Remove every paragraph of QA commentary per category 1 above. Move anything worth keeping into
   `DESIGN_SYSTEM.md` under a clear heading (e.g. "Measured realism" already exists there — extend it, don't
   duplicate).
3. What remains should read as a clean design system reference: tokens (colour, type, spacing), each component
   once in its best form, the dark mode toggle, the Tamil-sample-text toggle. Consistent spacing between
   sections; no leftover headings for things that no longer exist (e.g. a "gate table" heading with nothing
   useful under it once the table is removed).
4. Keep the `data-l="en"/"ta"` bilingual structure and all accessibility attributes exactly as they are — this is
   a content/tidiness pass, not a rebuild.

## Part B — `realism-chola.html` and `realism-kural.html`: same rule
1. Remove the dE2000 numbers, the "Why 2D" gate explanation panel, task/file references, and any "section 7
   gate" language from the visible page. A visitor does not need to know the leaf failed an anisotropy test.
2. If the page needs to say anything at all about why the leaf is 2D and the stone is 3D, it should be an
   invisible implementation detail, not visible text — or, if a short honest note is genuinely useful for
   context (e.g. explaining the theme, not the engineering), rewrite it in plain language with zero numbers,
   file paths, or task IDs, and only if it adds something a reader actually wants.
3. Do NOT remove or change any "sample verse", "unverified", or similar content-honesty flag. Check every removal
   against the two-category rule above before deleting it.
4. Check `realism-home.html` too (found only 2 harmless code-comment/placeholder-note mentions, but verify — the
   line "A box the size and shape a real map would take, so the layout can be finished before there is any map
   data" is a developer placeholder sentence, not real content; either remove it or replace it with a plain
   "Map coming soon" if a placeholder box is still shown).
5. Check `identity.html` too even though the earlier scan found nothing — confirm no leftover QA text slipped in
   during batches 2 or 3.

## Part C — general tidiness
1. Consistent spacing and heading levels across all pages touched.
2. No orphaned CSS classes left over from the removed before/after blocks (run a quick check: any class defined
   in CSS but no longer referenced in any HTML, report the list, remove ones that are safely unused).
3. Re-run the orphan-materials check (`node design/realism/eval/orphan_materials.mjs`) after all edits — it must
   still report 0 orphans. Re-run T15's checks (console errors, axe, keyboard, reduced motion) on every page
   touched and report the same or better numbers.

## Rules (carried over)
Back up before editing. File:// double-click must still work. Zero console errors, zero external requests except
self-hosted fonts. Contrast 4.5:1 body text stays. One commit per part.

## Report
List what was removed (by category), what was moved into `DESIGN_SYSTEM.md`, screenshots of the cleaned
`styleguide.html`, `realism-chola.html`, `realism-kural.html` at 360/768/1200, the orphan-check result, and the
re-run T15 numbers. Stop after this and wait for review — no new work queued.
