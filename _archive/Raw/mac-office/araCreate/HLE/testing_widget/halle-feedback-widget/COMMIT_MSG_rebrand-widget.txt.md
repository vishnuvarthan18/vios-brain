---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_rebrand-widget.txt
---

style: apply new brand design system to widget

Files: src/widget/src/styles.ts, src/widget/src/capture.ts,
src/widget/src/marker-pen.ts

- panel/launcher background #13202A -> navy #29308A
- body/secondary text #55686F -> text #2A2924
- borders #d7dfe1 / #eef1f2 -> border gray #EBEBEB
- marker pen stroke and element-box colour #e0362e -> error #D93025
  (capture.ts's burned-in overlay and marker-pen.ts's live canvas both)
- corner radius rounded to 12px (panel, mode-btn, icon-btn, touch-confirm
  bar, .btn) / 8px (textarea) — launcher's 999px pill and small decorative
  overlay radii (outline-box, outline-label, review-image-wrap,
  review-target-box) left as-is, out of the panel/button/input scope
  named for this pass
- font stack now leads with "Helvetica Neue", existing fallback stack
  unchanged after it — no external font load, per agent-rules.md §1.6
- spacing rounded to the nearest of 8/16/24px

No change to font sizes, tap target sizes (44px / 56px for the two
mode-chooser option buttons), or contrast ratios below what was already
locked in — verified: every swapped colour still clears its required
ratio against the surface it sits on (navy/error text and white-on-navy
all exceed 4.5:1). Border gray as a 2px UI border measures ~1.2:1
against white, same order of magnitude as the two border colours it
replaces (~1.1-1.4:1) — a pre-existing condition, not a new regression.

computed-styles.spec.ts asserts only font-size (>=16px) and interactive
target height (>=44/56px) floors, not colour or exact pixel values, so
no expected-value changes were needed there — confirmed by reading the
file before and re-running it green after.
