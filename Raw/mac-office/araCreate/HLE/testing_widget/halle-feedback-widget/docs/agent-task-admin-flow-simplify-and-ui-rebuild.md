# Agent task — simplify tester flow, rebuild admin UI properly

Vishnu's decisions, 9 September (evening pass, after the live end-to-end
test). These are product/UX decisions from the person who owns the
product — treat them as settled, not open for debate. This is a bigger
piece of work than the earlier consolidated fixes doc
(`claude/agent-task-consolidated-open-items.md`) — do that one first if
not already done, since item 1 there (screenshot capture) blocks actual
testing of anything built here.

**This task overrides one earlier decision on record**, so read this
before touching tester/page code:

> Previously: testers are created and then individually assigned to
> specific pages (`admin-v2-spec.md`, the tester-page assignment feature
> in M5). **This is now reversed.** See §2 below.

---

## 1. Simplify tester creation — name only, no email

Creating a tester in admin currently asks for more than it needs.
**Fix:** the tester-creation form should ask for a **name only**. Remove
the email field entirely from tester creation (check whether email is
used anywhere downstream — e.g. notifications — before removing the
column outright; if it's unused elsewhere, remove it from the schema
too, otherwise just stop requiring/showing it in the form).

## 2. Remove page assignment — a tester's link works for every page

**Reversed decision, see above.** Testers should no longer be assigned to
specific pages one at a time. Once a tester is created and their link
exists, that one link should let them report on **any page of the
site**, not just pages an admin has assigned them.

**What to do:**
- Remove the tester-to-page assignment UI and flow from admin.
- Remove whatever server-side check currently restricts a tester's
  report to only their assigned page(s) (if one exists) — a tester's
  token should be valid for reporting on any page.
- Check `docs/admin-v2-spec.md` and `docs/widget-v2-spec.md` for exactly
  where this assignment logic lives before removing it, and update
  those specs to reflect the new, simpler rule.
- Reports still need to record which exact page they came from (this
  doesn't change — see the pages note below) — only the assignment
  *restriction* goes away.

## 3. Add delete/remove for a tester

There is currently no way to remove a tester from admin. Add one:
a delete action on a tester (with a confirmation step, since this is
destructive) that removes the tester and revokes their link. Decide
and confirm with Claude/Vishnu what happens to that tester's existing
reports (most likely: reports stay, just the tester's own token/link
stops working) before implementing — don't guess silently on this one,
ask if genuinely unclear.

## 4. Rebuild the report list as a proper dashboard

The current report grid is not good enough for real use. Rebuild it as
a proper dashboard view — think in terms of what someone actually needs
when triaging incoming bug reports: an overview of counts/status at a
glance, not just a flat table. Keep the existing template-grouping
concept from `admin-v2-spec.md §6` (Home, Product Category, Product
Detail, Contact, 404) as a starting point, but the visual treatment
needs a real redesign, not a bare grid.

## 5. Rebuild the whole admin UI as a proper web app

The admin panel currently doesn't look or feel like a real product.
Rebuild it with:
- A proper **left-side navigation** (sections: reports/dashboard,
  testers, pages, whatever else exists — settle the exact nav structure
  as part of this work).
- Consistent layout, spacing, and visual polish throughout — every
  screen should feel like one coherent app, not a set of separate pages
  bolted together.

## 6. Apply the brand design system properly and consistently

The brand design system was already handed over
(`claude/halle-design-system-draft.md` — colors, type, spacing, radius,
component specs, all sourced from real Figma screens) and was applied
once already to both UIs. Vishnu's feedback now is that the result still
looks messy. Go back through the design system doc and apply it properly
and consistently across every screen of the rebuilt admin UI (and
double check the widget too) — colors, spacing scale, typography, and
component styling should all match it exactly, not approximately.

## 7. Think through the whole flow from the user's side before building

Before implementing the above, go through the entire tester-facing and
admin-facing flow end to end, from the point of view of each type of
person who touches it (a non-technical tester like Vishnu, an admin
triaging reports, B. Halle staff), across realistic scenarios — not just
the one happy path. Write down what you find (edge cases, confusing
steps, anything that doesn't make sense from that person's side) before
starting the rebuild, so the new UI actually solves real usability
problems rather than just looking different. Where you find a real
UX problem that isn't already covered by items 1–6 above, flag it and
ask rather than silently deciding.

---

## Note on "pages" (not a task, just context)

Vishnu confirmed: the "pages" list in admin still matters — every report
still needs to record which real page it came from, so reports can be
browsed by page. What's changing here is only the assignment
*restriction* (item 2) — testers are no longer locked to specific pages.
The admin's page list itself currently has 3 placeholder pages and still
needs the real 49 page URLs from Vishnu's own Webflow access — that's a
data task for Vishnu, not something for the agent to build.

---

## Report back

This is a substantial rebuild — batch it as its own milestone, separate
from the consolidated fixes doc. Report your plan for the UI rebuild
(item 7's findings, and a rough shape of items 4–6) back before writing
a large amount of code, so Vishnu and Claude can sanity-check the
direction early rather than after it's all built. Do not commit until
each piece is verified working. Do not push — that always needs
Vishnu's explicit word at the time.
