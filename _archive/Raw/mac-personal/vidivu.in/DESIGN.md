---
version: alpha
name: Vidivu-design-system
description: A motorsport-engineering interface anchored on a near-black canvas with white display headlines in confident UPPERCASE. The brand carries no decorative voltage of its own — its energy comes from full-bleed automotive photography (cars on tracks, driver-cockpit shots, carbon-fiber detail) and the Vidivu signature tricolor stripe (light blue → deep blue → red) used sparingly as a brand mark on logos, dividers, and motorsport chrome. Type stays light to medium weight to feel engineered, never bombastic.

colors:
  primary: "#ffffff"
  ink: "#ffffff"
  body: "#bbbbbb"
  body-strong: "#e6e6e6"
  muted: "#7e7e7e"
  hairline: "#3c3c3c"
  hairline-strong: "#262626"
  canvas: "#000000"
  surface-card: "#1a1a1a"
  surface-elevated: "#262626"
  surface-soft: "#0d0d0d"
  on-primary: "#000000"
  on-dark: "#ffffff"
  signature-1: "#3ba0e0"
  signature-2: "#1c69d4"
  signature-3: "#e22718"
  carbon-gray: "#2b2b2b"
  warning: "#f4b400"
  success: "#0fa336"

typography:
  display-xl:
    fontFamily: "Inter, sans-serif"
    fontSize: 80px
    fontWeight: 800
    lineHeight: 1
    letterSpacing: 0
  display-lg:
    fontFamily: "Inter, sans-serif"
    fontSize: 56px
    fontWeight: 800
    lineHeight: 1.05
    letterSpacing: 0
  display-md:
    fontFamily: "Inter, sans-serif"
    fontSize: 40px
    fontWeight: 800
    lineHeight: 1.1
    letterSpacing: 0
  display-sm:
    fontFamily: "Inter, sans-serif"
    fontSize: 32px
    fontWeight: 800
    lineHeight: 1.15
    letterSpacing: 0
  title-lg:
    fontFamily: "Inter, sans-serif"
    fontSize: 24px
    fontWeight: 700
    lineHeight: 1.3
    letterSpacing: 0
  title-md:
    fontFamily: "Inter, sans-serif"
    fontSize: 20px
    fontWeight: 400
    lineHeight: 1.4
    letterSpacing: 0
  title-sm:
    fontFamily: "Inter, sans-serif"
    fontSize: 18px
    fontWeight: 400
    lineHeight: 1.4
    letterSpacing: 0
  label-uppercase:
    fontFamily: "Inter, sans-serif"
    fontSize: 14px
    fontWeight: 700
    lineHeight: 1.3
    letterSpacing: 1.5px
  body-md:
    fontFamily: "Inter, sans-serif"
    fontSize: 16px
    fontWeight: 300
    lineHeight: 1.5
    letterSpacing: 0
  body-sm:
    fontFamily: "Inter, sans-serif"
    fontSize: 14px
    fontWeight: 300
    lineHeight: 1.5
    letterSpacing: 0
  caption:
    fontFamily: "Inter, sans-serif"
    fontSize: 12px
    fontWeight: 400
    lineHeight: 1.4
    letterSpacing: 0.5px
  button:
    fontFamily: "Inter, sans-serif"
    fontSize: 14px
    fontWeight: 700
    lineHeight: 1
    letterSpacing: 1.5px
  nav-link:
    fontFamily: "Inter, sans-serif"
    fontSize: 14px
    fontWeight: 400
    lineHeight: 1.4
    letterSpacing: 0.5px

rounded:
  none: 0px
  xs: 2px
  sm: 4px
  md: 6px
  full: 9999px

spacing:
  xxs: 4px
  xs: 8px
  sm: 12px
  md: 16px
  lg: 24px
  xl: 40px
  xxl: 64px
  section: 96px

components:
  button-primary:
    backgroundColor: "{colors.canvas}"
    textColor: "{colors.on-dark}"
    typography: "{typography.button}"
    rounded: "{rounded.none}"
    padding: 16px 32px
    height: 48px
  button-outline:
    backgroundColor: transparent
    textColor: "{colors.on-dark}"
    typography: "{typography.button}"
    rounded: "{rounded.none}"
    padding: 16px 32px
    height: 48px
  button-icon:
    backgroundColor: "{colors.surface-card}"
    textColor: "{colors.on-dark}"
    rounded: "{rounded.full}"
    size: 48px
  text-link:
    backgroundColor: transparent
    textColor: "{colors.on-dark}"
    typography: "{typography.label-uppercase}"
  top-nav:
    backgroundColor: "{colors.canvas}"
    textColor: "{colors.on-dark}"
    typography: "{typography.nav-link}"
    height: 64px
  hero-photo-band:
    backgroundColor: "{colors.canvas}"
    textColor: "{colors.on-dark}"
    typography: "{typography.display-xl}"
    padding: 96px
  signature-stripe:
    backgroundColor: transparent
    textColor: "{colors.on-dark}"
    height: 4px
  spec-cell:
    backgroundColor: "{colors.surface-soft}"
    textColor: "{colors.on-dark}"
    typography: "{typography.body-md}"
    rounded: "{rounded.none}"
    padding: 24px
  model-card:
    backgroundColor: "{colors.canvas}"
    textColor: "{colors.on-dark}"
    typography: "{typography.title-lg}"
    rounded: "{rounded.none}"
    padding: 24px
  magazine-card:
    backgroundColor: "{colors.surface-card}"
    textColor: "{colors.on-dark}"
    typography: "{typography.title-md}"
    rounded: "{rounded.none}"
    padding: 24px
  category-tab:
    backgroundColor: transparent
    textColor: "{colors.body}"
    typography: "{typography.label-uppercase}"
    padding: 12px 0
  category-tab-active:
    backgroundColor: transparent
    textColor: "{colors.on-dark}"
    typography: "{typography.label-uppercase}"
    padding: 12px 0
  cta-band-photo:
    backgroundColor: "{colors.canvas}"
    textColor: "{colors.on-dark}"
    typography: "{typography.display-md}"
    padding: 80px
  footer:
    backgroundColor: "{colors.canvas}"
    textColor: "{colors.body}"
    typography: "{typography.body-sm}"
    padding: 64px
---

## Overview

Vidivu's marketing surface is a near-pure black canvas (`{colors.canvas}` — #000) holding white display headlines in **confident UPPERCASE**. The system has no decorative voltage of its own; brand energy comes from **full-bleed automotive photography** — cars cornering at speed, carbon-fiber detail, driver cockpit shots, track pit lanes — placed as edge-to-edge content that fills entire bands. UI chrome around the photography stays minimal: light sans-serif copy, dividers as 1px hairlines (`{colors.hairline}`), all-caps button labels with no fill until hovered.

The **Vidivu signature stripe** — `{colors.signature-1}` (#3ba0e0) → `{colors.signature-2}` (#1c69d4) → `{colors.signature-3}` (#e22718) — appears sparingly as the brand's accent, used on the wordmark, motorsport chrome, and model badges. It is never a CTA color and never used as a background fill — the stripe is exclusively a brand-identity marker.

Type voice runs in two weights: 800 (extrabold) for display + button labels, and 300 (light) for body + secondary copy. The contrast between heavy display and light body is the system's editorial signature.

**Key Characteristics:**
- Near-pure black canvas with white type. The system inverts almost nothing — there is no light-mode marketing surface.
- Display headlines in UPPERCASE at weight 800. Sub-heads stay sentence-case at lighter weight.
- Signature stripe used as 4px brand dividers and motorsport chrome — never as buttons or fills.
- Photography fills entire bands edge-to-edge. Cars are always the visual subject; UI chrome backs off to small white labels overlaid on photography.
- Buttons are flat with `{rounded.none}` (0px) corners and uppercase letterspaced labels. The rectangular silhouette IS the brand.
- Border radius is mostly zero across the system. The only exception is `{rounded.full}` on circular icon buttons.
- Spacing is generous and grid-aligned: `{spacing.section}` (96px) between major bands.

## Colors

### Brand & Accent
- **Primary** (#ffffff): The system's primary type and CTA color.
- **Signature 1** (#3ba0e0): First stop in the signature stripe.
- **Signature 2** (#1c69d4): Middle stop.
- **Signature 3** (#e22718): Third stop — the power-red accent used in the stripe and motorsport callouts.

### Surface
- **Canvas** (#000000): The default page floor across every marketing surface.
- **Surface Soft** (#0d0d0d): Barely-different-from-black, used for spec table cells.
- **Surface Card** (#1a1a1a): Cards, secondary buttons, icon-button backgrounds.
- **Surface Elevated** (#262626): One step lighter, nested cards inside dark bands.
- **Carbon Gray** (#2b2b2b): Technical-spec card surfaces.

### Hairlines & Borders
- **Hairline** (#3c3c3c): 1px divider tone on dark surfaces.
- **Hairline Strong** (#262626): Section dividers, footer border.

### Text
- **On Dark** (#ffffff): Headlines and primary text.
- **Body** (#bbbbbb): Default running text.
- **Body Strong** (#e6e6e6): Emphasized body / lead paragraph.
- **Muted** (#7e7e7e): Footer links, captions.

## Typography

Two weights only: **800** for display, nav labels, button text, and category labels; **300** for body paragraphs and secondary metadata. Never blur the contrast with intermediate weights (400/500).

| Token | Size | Weight | Use |
|---|---|---|---|
| `display-xl` | 80px | 800 | Hero h1 |
| `display-lg` | 56px | 800 | Section heads |
| `display-md` | 40px | 800 | Sub-section heads, model names |
| `display-sm` | 32px | 800 | CTA-band heads |
| `title-lg` | 24px | 700 | Card titles |
| `title-md` | 20px | 400 | Card sub-titles |
| `label-uppercase` | 14px | 700 | Category tabs, inline links, 1.5px tracking |
| `body-md` | 16px | 300 | Default body |
| `body-sm` | 14px | 300 | Footer body, fine print |
| `caption` | 12px | 400 | Photo captions |
| `button` | 14px | 700 | Button labels, uppercase, 1.5px tracking |
| `nav-link` | 14px | 400 | Top-nav items |

UPPERCASE display is the default voice for h1/h2. Font fallback: system stack (`-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif`) with Inter as the primary loaded face.

## Layout

- **Base unit:** 4px. Tokens: `xxs` 4px · `xs` 8px · `sm` 12px · `md` 16px · `lg` 24px · `xl` 40px · `xxl` 64px · `section` 96px.
- **Max content width:** ~1440px, wider than typical SaaS to give photography breathing room.
- **Card grids:** 3-up at desktop, 2-up at tablet, 1-up at mobile.
- **Footer:** 4-column at desktop, 2-up at tablet, 1-up at mobile.

## Shapes

| Token | Value | Use |
|---|---|---|
| `none` | 0px | All buttons, cards, photo containers, spec cells, inputs — dominant radius |
| `full` | 9999px | Circular icon buttons only |

The radius hierarchy is "almost always 0, sometimes circular." Sharp rectangles read as engineered precision; circles read as functional controls. Nothing in between.

## Components

- **top-nav** — 64px black bar, wordmark + swatch at left, horizontal menu, icon cluster right.
- **button-primary / button-outline** — flat, 0px radius, uppercase letterspaced label, 48px height.
- **button-icon** — 48×48 circular, the only non-rectangular button shape.
- **text-link** — inline uppercase label with chevron, no underline.
- **hero-photo-band** — full-width black band with full-bleed photography and left-aligned display-xl headline.
- **model-card** — photo on black, model name in display-md, spec line, text-link.
- **spec-cell** — value in display-sm at top, label-uppercase below.
- **magazine-card** — surface-card background, category label, title-lg, short body.
- **category-tab / category-tab-active** — text-only, active state adds 2px underline.
- **signature-stripe** — 4px horizontal divider carrying the brand tricolor. Used sparingly, never as a fill.
- **cta-band-photo** — pre-footer CTA with full-bleed photography and centered headline.
- **footer** — black, 4-column link list, never inverts.

## Do's and Don'ts

### Do
- Anchor every page with full-bleed automotive photography.
- Use UPPERCASE display headlines. Sentence-case display reads off-brand.
- Pair heavy display (800) with light body (300).
- Reserve the signature stripe for brand-identity moments — never as a button fill or surface.
- Use `rounded.none` by default; `rounded.full` only for circular icon buttons.
- Letter-space all-caps labels at 1.5px.
- Use `spacing.section` (96px) between major editorial bands.

### Don't
- Don't introduce a brand color outside the signature stripe stops.
- Don't bold body type — body stays at 300.
- Don't use rounded buttons. The rectangular silhouette IS the brand.
- Don't put gradient backdrops behind hero type — the photography provides the depth.
- Don't repeat the same surface mode in two consecutive bands.
- Don't use the signature stripe as a button fill.

## Responsive Behavior

| Name | Width | Key Changes |
|---|---|---|
| Mobile | < 768px | Hamburger nav; hero h1 scales 80→48px; grids 1-up; footer 4→1 col |
| Tablet | 768–1024px | Nav tightens; 2-up card grids; spec tables 2-up |
| Desktop | 1024–1440px | Full nav; 3-up grids; spec tables 4-up |
| Wide | > 1440px | Same as desktop, max content 1440px |

## Notes

This spec was adapted from a motorsport-brand design-system analysis and rebuilt as an original Vidivu identity: colors, component names, and the signature stripe stops are Vidivu's own — not copied from any third-party brand's exact values. The homepage in this repository (`app/page.tsx`, `components/`) implements this spec.
