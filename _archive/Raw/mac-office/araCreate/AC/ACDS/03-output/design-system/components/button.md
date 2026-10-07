# Button

Source: live site only (`.cta-button`), verified directly against `02-process/audit/live-site-components-pixel-exact.md`. The template source has a differently-structured `.button` class that is NOT used on the live site — not carried into this component.

## For future agent
This is the only real button component confirmed on the live production site. Built from pixel-exact CSS extraction, not from a screenshot or guess. One variant (`dtf-menu`) is flagged as unresolved — see Open Questions.

## Anatomy

Single class `.cta-button`, with contextual modifier classes layered on top for spacing/sizing changes. No separate "primary/secondary" button classes exist on the live site.

## Base style

```css
.cta-button {
  color: #555;
  background-color: #F9BF3B;   /* token: (secret removed) */
  padding: 15px 20px;
  font-size: 14px;
  font-weight: 500;
  line-height: 1em;
  transition: transform .5s;
  display: block;
}
```

## States

```css
.cta-button:hover {
  transform: translateY(-5px) scale(1.02);
  box-shadow: 0 4px 9px -3px #0003;
}
```

Lift + scale + shadow-in on hover. No separate `:focus` or `:active` state found in source — likely relies on browser default or inherits `:hover`. Flag for accessibility review (visible focus state for keyboard nav should be confirmed).

## Contextual variants (modifier classes, applied on top of base)

| Modifier class | Context | Override |
|---|---|---|
| `.cta-button.cta-header` | Header CTA | `margin-top: 25px` |
| `.cta-button.form` | Form submit | `margin-top: 0` |
| `.cta-button.newsletter` | Newsletter signup | `padding: 18px 36px` (one instance narrows to `width: 130px; padding: 0 30px; font-size: 12px`) |
| `.cta-button.floating` | Floating/sticky CTA | `width: 100%; display: flex; justify-content/align-items: center` |
| `.cta-button.home-button` | Homepage | `margin-top: 10–20px` (inconsistent across instances — see below) |
| `.cta-button.home-button.hero-page` | Homepage hero | `margin-top: 30–40px` (inconsistent — see below) |
| `.cta-button.dtf-menu` | DTF menu | **Inverted colors:** `background-color: var(--button-colour-gray); color: var(--brand-color-yellow)` — see Open Questions |

## Drift found (not yet resolved)

- `.cta-button.home-button` appears twice in source with different `margin-top` (20px vs 10px) and once adds `font-size: 14px`. Likely accumulated per-page overrides in Webflow's visual editor rather than a deliberate system. Needs a decision: pick one canonical value or treat as genuinely contextual.
- `.cta-button.newsletter` appears three times with three different padding/width values. Same issue.

## Open questions

1. **`.dtf-menu` variant uses gray background + yellow text instead of the standard gold background.** Stakeholder flagged as unresolved (2026-10-06) — not yet decided whether this is an intentional distinct button style (e.g. "menu button" vs "CTA button") or accidental drift. Do not resolve without confirming.
2. No distinct `:focus` state found — confirm actual keyboard-focus behavior before shipping an accessible component spec.
3. Template source has a separate `.button` class (different padding scale: `27px 77px`, different hover: solid blue `#3347A0` background) that does NOT appear on the live site. Worth asking: was `.button` deliberately replaced by `.cta-button`, or is `.button` dead code never fully removed from the template fork?
