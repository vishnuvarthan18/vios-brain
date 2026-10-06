# W2D — Design System

> Read this before styling any screen.
> The tokens live in `tailwind.config.js`. The components live in
> `app/_components/ui/`. This file explains **when to reach for which**, and
> which rules are decisions rather than preferences.

Created 2026-08-01 as a description of patterns. **Rewritten 2026-08-21, when
the system became real** — see "What changed" at the bottom for why that
distinction matters.

---

## The one rule

**Compose from `app/_components/ui`. Never write a raw hex value, and never
hand-roll a control that already exists.**

If something you need is missing, add it to `ui/` — a one-off in a screen is
exactly how the drift this system replaced got started. The 2026-08-01 version
of this file opened with "never invent a new color" and nothing enforced it; by
2026-08-21 the codebase had about a dozen near-duplicate greys and browns
(`#201a19` beside `#1d1b1a`, `#534341` beside `#56413e`, `#857371` beside
`#8a716c`, `#d8c2bf` beside `#ddbfb9`). None of those were decisions.

```tsx
import { Button, Card, EmptyState, Screen, ScreenHeader } from '../_components/ui';
```

---

## Tokens (`tailwind.config.js`)

### Colour — semantic roles, not shades

A screen says what a colour is FOR, not which brown it is. That is what makes a
correction a one-line change, and it is what makes dark mode a token swap rather
than a rewrite.

| Token | Use |
|---|---|
| `brand` / `brand-strong` / `brand-tint` / `brand-wash` | Brand red; pressed state; badge/pill background; faint wash for callouts |
| `canvas` | The page |
| `surface` | Anything raised onto the page — cards, sheets, inputs, rows |
| `inset` | A field that is deliberately not editable; a thumbnail well |
| `sunken` | A placeholder, or a pressed ghost button |
| `ink` / `ink-secondary` / `ink-muted` / `ink-inverse` | Text, descending emphasis. **Three levels, not six** |
| `line` / `line-strong` | Hairline dividers; a border doing real work |
| `danger` + `danger-tint` | Errors, destructive actions, rejected, expired |
| `success` + `success-tint` | Verified, approved, live |
| `warning` + `warning-tint` | Needs attention but not broken — in review, a role that disagrees with its category |
| `external-whatsapp` | **Reserved.** See the rule below |

### Typography — semantic sizes with line heights baked in

`display` · `title` · `heading` · `subhead` · `body` · `label` · `caption` ·
`micro`

Line heights are part of the token because **React Native does not inherit
line-height**. A size without one renders at the platform default, and dense
prose (the legal screens, listing descriptions) ends up cramped.

Tailwind's own `text-sm`/`text-base` still exist — the theme is extended, not
replaced — but nothing in `app/` should use them.

### Radii, named by what they wrap

`field` (12px) · `card` (16px) · `sheet` (24px) · `pill`

A 12px input next to a 16px input is the sort of half-millimetre inconsistency
that reads as "unfinished" without anyone being able to say why.

### Spacing

Tailwind's 4px scale, unchanged — it was never the problem. Two additions:
`gutter` (20px, the screen's horizontal padding) and `touch` (48px, Android's
minimum tap target).

---

## Components

### Layout

| Component | Use |
|---|---|
| `Screen` | Every screen's outermost element. `edges` defaults to top-only, which is right inside the tab bar; use `['top','bottom']` for a pushed screen or one with a pinned footer |
| `ScreenHeader` | Back button, title, optional trailing. `brand` renders the title in red — top-level tabs only |
| `ScreenFooter` | The pinned action at the bottom of a screen whose whole purpose is one decision |
| `Sheet` + `SheetOptions` | **The** modal pattern. There is no second one |

### Controls

| Component | Use |
|---|---|
| `Button` | `primary` (one per screen) · `secondary` · `ghost` · `danger` · `whatsapp`. `loading` disables and swaps the label for a spinner |
| `Field` | Label + control + hint + error, in that order, always |
| `TextField` / `SelectField` / `ReadOnlyField` | A text input; the row that opens a picker; a value being SHOWN rather than asked for |
| `Checkbox` | Box and label are ONE tap target |
| `Chip` | Filter chips and segmented tabs, with an optional count |

`ReadOnlyField` versus a disabled `TextField` is a real distinction: the first
is a value you are being shown, the second is a control you cannot use.
Rendering a derived value (a role, a verified phone number) as a greyed-out
input invites a tap that does nothing.

### Content

| Component | Use |
|---|---|
| `Card` | A **bounded item** — a listing, a catalog entry. Stacks with gaps |
| `RowGroup` + `Row` | **Ongoing state** — settings, interests, your own posts, notifications. Flush rows divided by hairlines |
| `InfoRow` | A label/value pair in a read-only detail list |
| `Badge` + `BadgeRow` | Pills. See the badge rule below |
| `Avatar` | Initials on brand tint. Businesses have no logo field, so this IS the avatar, not a placeholder for one |
| `SectionLabel` · `Divider` · `Hint` | A group's signpost; a hairline; the sentence explaining a control |

### States

| Component | Use |
|---|---|
| `LoadingState` | Full-screen spinner, with an optional label for a wait that is not self-explanatory |
| `EmptyState` | Icon, headline, one explaining line, at most one action |
| `ErrorState` | The same shape, danger-tinted, with a retry |
| `InlineMessage` | A message belonging to a place on the screen — `error`/`success`/`info`/`caution` |
| `Skeleton` | For a wait whose shape is already known, so the layout does not jump |

`EmptyState`'s `title` says what is not there; `body` says what to do about it.
Those are different sentences, which is why they are two props.

### `COLOR` (`ui/colors.ts`)

Three things cannot read a class: `@expo/vector-icons` (`color` string prop),
`ActivityIndicator`, and React Navigation's style objects. `COLOR` is the ONE
sanctioned place for a raw value, and it mirrors `tailwind.config.js` exactly.
**If a token changes there, change it here** — a mismatch shows up as an icon
that is a slightly different red from the text beside it.

---

## Rules that are decisions, not preferences

### 1. One badge language

Badges are `brand` (red on tint) for anything **descriptive** — role, post type,
category, match label. The other tones exist only where a badge reports a
**state**: `success` for verified/live, `danger` for rejected/expired/missing,
`warning` for in-review. A user should never have to learn that a red pill means
one thing here and another there.

### 2. Cards for items, rows for state

Bounded things you could hold — a listing, a catalog entry — are `Card`s and
stack with gaps. Ongoing state — your interests, your posts, notifications,
settings — is a `RowGroup` of flush `Row`s.

This is the Facebook-feed versus WhatsApp-chat-list distinction. It is
deliberate, it carries meaning, and standardising on one for both would flatten
it.

### 3. WhatsApp green is reserved

`external-whatsapp` is for the literal "open WhatsApp" action and nothing else.
Its job is to read as *you are leaving the app*, and it only does that job while
it is not also the generic success colour. The token is named `external` so
reaching for it as a success green feels wrong at the call site.

### 4. Empty, loading and error states are designed, not defaults

An empty state is the **first** thing a new account sees on every feed. A blank
screen with one grey line reads as "nothing here and nothing to do". Every one
of them gets an icon, a headline, an explanation, and — where there is something
to do — one action.

Where an empty state has more than one cause, it gets more than one version.
"No listings exist yet" and "your filters match nothing" need opposite advice,
and the Available feed distinguishes three.

### 5. Say what an action costs before it is tapped

Both reveal buttons say that a reveal uses one of today's allowance and that the
number then stays visible (§6, D4.3). The requirement response says that your
own contact goes to the poster (§4). The post button says the post is reviewed
before it appears (§15). None of that is decoration: a surprise after a tap is
the difference between a rule and a bug, from the user's side.

### 6. The two screens that get extra care

- **The reveal card** (`listing/[id].tsx`). §6 calls this "the highest-trust
  moment in the app". Avatar, §6/§15's full trust context as badges, the number
  as the largest thing on the card, and a line saying where it came from and
  that W2D is not part of what happens next.
- **The public profile** (`p/[slug].tsx`). §6a makes this the only screen a
  non-user ever sees — it is doing the job of a website for a business that
  probably does not have one. It has to look credible to a stranger in about two
  seconds.

---

## Dark mode

**Done:** every colour is a semantic role and no screen hardcodes a hex, so a
dark theme is a token redefinition rather than a rewrite.

**Not done:** the app does not render a dark theme and no component carries
`dark:` variants. A dark theme needs to be *looked at* on a device — contrast on
photo-heavy cards, the reveal screen's trust cues, WhatsApp green on a dark
surface — and a half-converted dark mode, where some screens flip and others
stay white, is worse than none. `app.json` pins `userInterfaceStyle: "light"`.

The follow-up: add a `dark` block redefining the same token names, switch to
`automatic`, and walk the screens on a device. No screen should need structural
change.

---

## What changed on 2026-08-21, and why it is worth knowing

The 2026-08-01 version of this file described patterns and mapped them onto
Facebook/Instagram/WhatsApp precedents. That description was good and most of it
survives above. What it did not have was **anything enforcing it**:
`tailwind.config.js` had an empty `theme.extend`, so every screen hardcoded its
own values, and the file's own rules were followed unevenly.

Concretely, before the revamp:

- buttons ranged 40–56px tall with four different pressed states and three
  different disabled treatments;
- three different input heights, two different focus treatments (one of which
  was none), and error text in a different position per screen;
- five hand-picked status colour pairs in My Listings, none from the palette;
- the bottom-sheet picker was reimplemented five times;
- `getInitials` existed in two copies — this file flagged that in 2026-08-01 and
  asked for it to be promoted "the next time either file is touched";
- loading was a bare spinner, empty was one grey sentence, and an error looked
  identical to an empty feed;
- the tab bar signalled the active tab with colour alone, which is the one
  signal a colour-blind user does not get;
- `otp.tsx` carried a padlock emoji and the words "Secure verification powered
  by Wedding2day Trust" — a trust brand that does not exist, which is the
  opposite of trustworthy.

All of that is gone. The point of writing it down is that none of it was a
decision anyone made; it was what happens when a system is a document instead of
code.
