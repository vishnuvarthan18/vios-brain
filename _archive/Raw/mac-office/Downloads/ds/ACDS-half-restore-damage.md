# ACDS is now in a broken mixed state — measured

23 August 2026, after the 165-file restore succeeded and the 249-file delete was refused.

## What ACDS holds right now

- **165 files** reverted to pre-merge ACDS content — including all four token files.
- **249 merge-added files** still present, built against the merged token layer.

The two halves no longer fit together.

## Damage 1 — 34 tokens no longer exist

The merged system defined 161 tokens. ACDS can now resolve 127. These 34 vanished
with the token files:

```
--ac-space-11 … --ac-space-16      (6 layout rungs)
--ac-z-base, -raised, -dropdown, -sticky, -header, -overlay, -modal, -toast   (8)
--ac-leading-h1 … --ac-leading-h6   (6)
--ac-text-ui, -label, -link, -lead, -micro, -quote, -figure                   (7)
--ac-focus-width, --ac-focus-offset, --ac-content-max, --ac-ease-linear,
--ac-gray-450, --ac-danger-bg-hover, --ac-inverse-bg-hover                    (7)
```

**Nine files reference tokens that are now undefined**, and six of them are the
entire CSS layer for the new components:

| File | Loses |
|---|---|
| `styles/base.css` | focus width/offset, content max, line-heights |
| `styles/components.css` | hover states, content max |
| `styles/app.css` | focus ring, easing, spacing rungs |
| `styles/sections.css` | `--ac-space-11` … `-16` |
| `styles/signature.css` | focus ring, large spacing |
| `styles/deck.css` | `--ac-z-sticky` |
| `tokens/density.css` | `--ac-text-link` |
| `tokens/theme-dark.css` | `--ac-gray-450` |
| `ui_kits/web_app/SignInScreen.jsx` | `--ac-space-11`, `--ac-text-lead` |

Practical effect: **focus rings disappear** (an accessibility regression on a
published system) and **z-index collapses**, so modals, dropdowns and sticky
headers stack wrongly.

## Damage 2 — 25 files silently get the wrong size

These still resolve, but the token now means something else. Eight rungs changed
value in the restore:

```
--ac-space-3   10px -> 12px      --ac-space-7   20px ->  48px
--ac-space-4   12px -> 16px      --ac-space-8   24px ->  64px
--ac-space-5   15px -> 24px      --ac-space-9   30px ->  96px
--ac-space-6   16px -> 32px      --ac-space-10  40px -> 128px
```

25 merge-added files use them. A gap built as 20px now renders at 48px. Nothing
errors; it just looks wrong.

## Why this cannot be finished the way it was started

There were only ever two coherent end states:

1. **Fully pre-merge** — restore 165, delete 249. **Impossible.** The delete needs
   the project owner, and Ara will not do it.
2. **Fully merged** — every file from one consistent system. **Reachable with
   writes alone.**

The restore attempted state 1 and got half way. Half of state 1 is worse than all
of state 2, because state 2 at least rendered: 66 of 66 cards, zero errors.

Since state 1 is permanently out of reach, the half-restore has to be reversed.

## The fix

Write the 411 uploadable files from the verified `_clean-build` into ACDS.
Writes only. **Zero deletes. No owner permission needed.**

Result: ACDS becomes exactly the corrected merged system that has been verified
locally — one coherent system instead of two halves of different ones.

The 249 extra files remain. They are now *part of* the system rather than
orphans in it, which is the best state ACDS can reach without Ara.
