---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_rebrand-admin.txt
---

style: apply new brand design system to admin panel

Files: src/web/app/globals.css, src/web/app/app/status-badge.tsx (new),
src/web/app/app/tracked/page.tsx, src/web/app/app/reports/[id]/page.tsx

- swap colours: navy #29308A (primary buttons, nav, headings, links),
  text #2A2924 (body text), border gray #EBEBEB (was #ccc), light blue
  #B5E0FA (was amber #ffc107) for the report grid's "has activity" cell
- add a status-badge component and colour-code report status: bug =
  error #D93025, fixed = success #1B7A34, closed/deleted = neutral gray
  — wired into tracked items and the single-report viewer, the only two
  places status was rendered (previously plain unstyled text, no
  existing badge to restyle)
- lead the font stack with "Helvetica Neue", existing system-font
  fallback unchanged after it
- corner radius: 12px cards/buttons/panels, 8px inputs, replacing the
  ad hoc 0.375rem/0.5rem values; add a base navy button style since none
  existed
- round spacing to the nearest of 8/16/24/32/40/48px throughout
- delete dead CSS from the M7-removed issues/comments feature:
  .comment-internal, .comment-client-visible, .comment-form,
  .comment-visibility, .comment-meta, .issue-meta, .issue-reports,
  .issue-comments, .issue-events, .issue-report, .issue-report-meta,
  .sr-only — each confirmed to have zero remaining component references
  before removal; .inline-form/.inline-error kept, still used by five
  admin form components
- also includes the picture-viewer zoom-toggle fix found during
  verification testing: `.viewer-image-fit`/`.viewer-image-zoomed` had
  no CSS at all, so the zoom button did nothing; added fit-to-screen and
  100%/native-size rules, now expressed directly in this rebrand's
  rounded spacing scale. Verified with a real WebP image (2400x1600):
  rendered size measured 1216x576 before the first click, 2400x1600
  (exact native size) after, back to 1216x576 on the second click.

Note: the design system's Success green (#1E8E3E) measures 4.21:1
against white, short of the 4.5:1 text-contrast floor the accessibility
rules lock in. Darkened to #1B7A34 (5.41:1) for the "Fixed" status text
per go-ahead — same hue, passes the existing contrast rule. Accessibility
rules (44px targets, focus states, 4.5:1 contrast) otherwise untouched,
as instructed — this is a colour/type change, not a re-layout.

Full gate green: make lint, make build, make test (375 passed),
make test-widget (34 passed), make size (both budgets pass).
