# Template — acds-template-app

The signed-in araCreate app. Entry: `App.dc.html`. It replaces the
`ui_kits/web_app/` kit, which held the same four screens.

Companion to `acds-template-web`: that one is the public marketing site, this
one is the product behind the sign-in. They share the token layer and every
component; they share no layout, because a scroll of full-bleed bands and a
fixed application shell are not the same page.

## What is in it

| Part | Where | Exercises |
| --- | --- | --- |
| Sign in | `SignIn.jsx` | split auth layout on graphite, live email validation, negative logo |
| App shell | `App.dc.html` template | `.ac-app` grid, side nav collapsing to a 64px rail, top bar, dismissible banner |
| Overview | `Dashboard.jsx` | KPI tiles with direction arrows, chart shell with bars, segmented range switch, compact table, steps |
| Jobs | `Jobs.jsx` | data table with selection, sorting, row actions, bulk bar, filter tags, drawer, loading and empty states |
| Settings | `Settings.jsx` | tabbed form — profile, preferences (segmented, number, slider, switches), team table, danger zone |

The shell is static markup in the template, so the nav items, brand, banner copy
and bar buttons are all directly editable. The four screens are `<x-import>`ed
`.jsx` files — that is where the interactive component demos live.

## Tweaks

- `showBanner` — drops the maintenance banner and gives the shell the full
  viewport height.
- `startSignedIn` — set `always-out` to land on the sign-in screen, which is
  otherwise skipped so the app is the first thing you see.

Both switches in the top bar work: **Dark / Light** sets `data-ac-theme` on
`<html>` and re-themes every screen; **Compact / Comfortable** sets
`data-ac-density` on the app wrapper and tightens the jobs table, with the 44px
target floor holding.

## Copying it into a consuming project

1. `ds-base.js` — point `base` at the bound design-system folder. It links
   `system.css`, the whole system; `tokens.css` is variables only and would leave
   every `.ac-*` class unstyled.
2. Asset paths — the `../../assets/…` references become `<base>/assets/…`.

## Two things to know about the markup

- **The nav must carry `hidden` when closed under 992px.** Below that width
  `.ac-app__nav` is `position: fixed`, so a nav without `hidden` covers the
  page. The logic class computes `navHidden` from a resize listener, and the
  hamburger switches meaning: rail toggle on wide, open/close on narrow.
- **Nothing here authenticates.** The template supplies the appearance and the
  validation. The domain — jobs, clients, drawings across the three verticals —
  is deliberately thin, and every figure is either a brand fact (`300+`, `3+`)
  or visible filler (`00`, `Client name`, `Team member`).
