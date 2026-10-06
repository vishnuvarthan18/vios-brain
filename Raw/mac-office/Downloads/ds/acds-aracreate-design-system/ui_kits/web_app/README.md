# UI kit — araCreate web app

**This one is a demonstration, not a recreation.** The other three kits rebuild
something that exists — aracreate.group, the Academy's course marketing, the
ten-slide deck. No app screens, wireframes or product copy were provided for a
signed-in araCreate product, so nothing here is copied from anywhere.

It exists because 24 application components with no screen behind them are a
guess. `guidelines/decisions.md` makes the argument better than this file can:
four gates passed for an entire build while the inverse surface silently rendered
step markers at 1.08:1, and it only surfaced when a real page put a real
component on a dark band. **A system nobody has built with is a guess.**

So the domain is deliberately thin — jobs, clients, drawings, across the three
real verticals — and every figure is either a brand fact (`300+`, `3+`) or
obvious filler (`00`, `Client name`, `Team member`).

## Screens

| Screen | File | Exercises |
| --- | --- | --- |
| Sign in | `SignInScreen.jsx` | split auth layout on graphite, live email validation, the negative logo variant |
| Overview | `DashboardScreen.jsx` | KPI tiles with direction arrows, chart shell with bars and a segmented range switch, compact table, steps |
| Jobs | `JobsScreen.jsx` | data table with selection, sorting, row actions, bulk bar, filter tags, and the loading and empty states — switchable from the toolbar |
| Settings | `SettingsScreen.jsx` | tabbed form: profile, preferences (segmented, number, slider, switches), team table, danger zone |

The shell is the same on every screen: side nav collapsing to a 64px rail, top
bar, avatar dropdown, and a page banner that can be dismissed.

## What it proves about phase one

Two switches in the top bar, both worth trying:

- **Dark / Light** sets `data-ac-theme` on `<html>`. Every screen re-themes and
  no component was written twice. Watch the sign-in screen's dark band — inside
  the dark theme it flips to light, which is the one structural change the theme
  needed.
- **Compact / Comfortable** sets `data-ac-density` on the app wrapper. The jobs
  table tightens; the 44px target floor holds on touch.

## What is deliberately missing

Real authentication, real data, and a real product name. The nav counts and job
names are filler. If araCreate has an actual internal tool, point me at its code
or screens and this becomes a recreation instead — which is the more useful thing
for it to be.
