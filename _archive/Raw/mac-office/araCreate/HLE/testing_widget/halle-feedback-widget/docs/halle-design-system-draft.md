# Halle Website — Standard Design Values (Draft for Approval)

Source: real data pulled from 3 Figma "final-design" screens (Home, Contact,
Polarizers product page). Read-only — nothing changed in Figma or Webflow.

This is **not built anywhere yet [on the Webflow site]**. This is the
proposed final list for that site. **It has, however, now been applied to
this project's own UIs — see the status block below.**

---

## 1. Colors

| Name | Value | Use |
|---|---|---|
| Navy (Primary) | `#29308A` | Buttons, headings, nav links, section backgrounds |
| Text (Neutral Dark) | `#2A2924` | Body text — replaces `#555` and `rgba(34,34,34,0.7)`, which should not be used anymore |
| White | `#FFFFFF` | Backgrounds, text on navy |
| Border Gray | `#EBEBEB` | Input field borders |
| Light Blue | `#B5E0FA` | Official brand color — confirmed on the client's own branding page |
| Pale Blue | `#D3EDFC` | Official brand color — confirmed on the client's own branding page |

`#787747` was checked against the client's official branding page and does
not appear anywhere — stays removed as accidental drift, replaced with
standard navy wherever it shows up.

| Status | Value | Use |
|---|---|---|
| Success | `#1E8E3E` | Confirmation messages, success states |
| Error | `#D93025` | Form errors, warnings |

(These 2 status colors did not exist in the Figma pages checked — added as
standard, accessible red/green since none were found. **Note: in this
project's admin panel, Success had to be darkened to `#1B7A34` — see status
block below.**)

---

## 2. Typography

Font: **Helvetica Neue** — 4 weights: Light, Regular, Medium, Bold.

| Style name | Size | Weight | Use |
|---|---|---|---|
| Page Title | 42px | Bold | Main H1 per page |
| Section Title | 26px | Bold | Section headings |
| Sub-heading | 24px | Medium | Card/block titles |
| Nav / Label | 22px | Medium | Nav menu, labels |
| Body Large | 20px | Regular | Main paragraph text |
| Body | 18px | Regular | Secondary text |
| Small Text | 16px | Regular | Fine print, footnotes |
| Micro | 12px | Regular | Only if truly needed — confirm with team, otherwise remove |

Rule: no custom/odd sizes. Round to the nearest size above.

## 3. Line height

- **Tight (1.1x)** — headings/titles.
- **Normal (1.4x-1.5x)** — paragraphs/body text.

## 4. Icon sizes

18px (small inline), 24px (detail), 32px (feature/content), 36px (button
icons). Outline style only, consistent stroke weight.

## 5. Spacing scale

8 · 16 · 24 · 32 · 40 · 48 · 64 · 80 (px). Nothing in between.

## 6. Corner rounding

4px (small tags/badges), 8px (input fields), 12px (cards/buttons/panels),
full round (pills/circular buttons).

## 7. Letter spacing

Normal (0) for body; Tight (-0.2px) for large headings only.

## 8. Screen size / grid

1440px page width, 80px side margin, 1280px content width. No tablet/mobile
frames found yet — needs confirming with the designer.

## 9. Component specs

- Small button: 40px height, 20px/8px padding, 6px radius.
- Large CTA button: 50px height, 20px padding, 10px radius.
- Input field: 56px height, 12px radius, 1px `#EBEBEB` border, 24px icon
  left, text starts ~60px from left.
- Search bar: 48px height, 8px radius, 1px navy border.
- Product/thumbnail card: ~190x153px, 1px white border, 4px radius.
- New token: (secret removed) Gray** `rgba(143,143,143,0.8)` for input
  placeholder text.

## 10. Still missing

Component states (hover/clicked/disabled/loading), motion timing (agreed:
keep small and subtle, nothing flashy), tablet/mobile grid (not designed
yet).

---

# APPLYING THIS TO THE FEEDBACK WIDGET AND ADMIN PANEL

Written 9 September 2026, after Vishnu asked for this design system to be
applied to both UIs in this project (separate from the Webflow site work
above, which is not this project's concern).

## STATUS: DONE, VERIFIED, NOT COMMITTED (9 September 2026)

Both phases built and gate-checked. Nothing committed or pushed yet —
two draft commit messages are ready and waiting on Vishnu's go-ahead:
`COMMIT_MSG_rebrand-admin.txt` and `COMMIT_MSG_rebrand-widget.txt`.

**Admin panel** — `globals.css` rebranded with Navy/Text/Border Gray/Light
Blue, Helvetica Neue leading the font stack, 12px/8px radius, 8/16/24/32/40/48
spacing. New `status-badge.tsx` component (didn't exist before), wired into
both places status renders: Bug=Error, Fixed=Success, Closed/Deleted=neutral
gray. Dead M4-era issues/comments CSS deleted after confirming zero
remaining references per class — `.inline-form`/`.inline-error` correctly
kept (still live; shared a section header with the truly-dead rules, caught
before it was wrongly deleted too).

**Widget** — `styles.ts`/`capture.ts`/`marker-pen.ts` colors swapped 1:1 per
the mapping below, Helvetica Neue added first, radius rounded to 12px/8px,
spacing rounded to 8/16/24. Decorative-only radii (e.g. the outline box) left
alone, out of scope. `computed-styles.spec.ts` needed no changes — it only
asserts font-size/target-height floors, not colors or exact pixel values.

**Contrast fix, decided with Vishnu during the build:** the design system's
Success green `#1E8E3E` measured 4.21:1 against white — short of the 4.5:1
text floor already locked in for this project. Vishnu chose darkening it to
**`#1B7A34`** (5.41:1), used for "Fixed" status text specifically. If this
green is ever reused elsewhere (e.g. the future Webflow site), it needs the
same contrast check against whatever background it sits on there — `#1E8E3E`
as originally specified only clears 4.5:1 on some backgrounds, not all.

**Gate:** lint/build/test (375)/test-widget (34)/size — all green. Visually
verified both UIs via Playwright screenshots (login, queue, tracked-items
with the new status badges, strings form, and the widget's full
launcher→review→marker-draw→sent flow). Shadow DOM isolation still holds
against the hostile test host page.

## What each codebase looked like before (checked directly, not assumed)

**Widget** (`src/widget/src/styles.ts`, `capture.ts`, `marker-pen.ts`) — had
its own consistent look, just not this design system's colors: dark navy
panel (`#13202A`), white text on it, body text gray (`#55686F`), borders
(`#d7dfe1`, `#eef1f2`), red accent (`#e0362e`) for the element box and
marker pen, system font stack.

**Admin panel** (`src/web/app/globals.css`, 397 lines) — plain, unstyled:
system font, black-on-white, gray borders (`#ccc`), one amber highlight
(`#ffc107`) for the report grid, pale yellow/green backgrounds for old
internal vs. client-visible comments. No brand color anywhere.

## Rules that apply only to the widget (already locked in, not up for
## debate — `docs/agent-rules.md`) — all respected in the build above

- No external font loaded — Helvetica Neue via the OS's own copy only
  (Mac/iOS have it; Windows/Android/Linux fall back to the existing system
  font stack).
- 16px minimum font size, 44px minimum tap target (56px for the widget's
  own big option buttons), 4.5:1 contrast — unchanged, still enforced by
  `computed-styles.spec.ts`.
- Zero new dependencies, size budget respected.

## Final token mapping (as built)

**Widget:**

| Design system token | Old widget value | Now |
|---|---|---|
| Navy `#29308A` | `#13202A` (panel/launcher background) | Navy |
| White `#FFFFFF` | already white | No change |
| Text gray `#55686F` | body/secondary text | Text `#2A2924` |
| Border `#d7dfe1` / `#eef1f2` | input/panel borders | Border Gray `#EBEBEB` |
| Radius | 16px (panel), 8-10px (buttons/inputs) | 12px (panels/buttons) / 8px (inputs) |
| Font stack | system-ui stack | `"Helvetica Neue"` first, same fallback after |
| Marker pen / element box | red `#e0362e` | Error `#D93025` |
| Spacing | ad hoc pixel values | Rounded to nearest of 8/16/24 |

**Admin panel:**

| Design system token | Old admin value | Now |
|---|---|---|
| Navy `#29308A` | none (plain black text/links) | Primary buttons, nav, headings |
| Text `#2A2924` | default black | Body text |
| Border Gray `#EBEBEB` | `#ccc` | Table/card/input borders |
| Success `#1B7A34` (darkened) / Error `#D93025` | none | Status badges: Bug=Error, Fixed=Success, Closed/Deleted=neutral gray |
| Grid "has activity" highlight | amber `#ffc107` | Light Blue `#B5E0FA` |
| Radius | `0.375rem`/`0.5rem` ad hoc | 12px (cards/buttons) / 8px (inputs) |
| Font | system-ui stack | `"Helvetica Neue"` first |
| Dead comment/issue CSS | present | Removed |
