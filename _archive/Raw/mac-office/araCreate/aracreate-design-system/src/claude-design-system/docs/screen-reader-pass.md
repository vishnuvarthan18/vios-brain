# SCREEN-READER PASS

A script for the one check nothing automated can do.

**Why this file and not a gate.** `tests/checks.html` catches a missing
accessible name, a broken `aria-describedby`, a skipped heading level. It cannot
tell you that a label is confusing, that an order is illogical, or that a screen
reader announces something technically correct and practically useless. That
needs a person. It is the source repository's own outstanding item too, recorded
in `decisions.md`: *"A pass with a real screen reader… That needs a person and
VoiceOver."*

**Time needed: about forty minutes.** Do it once, write what you find at the
bottom of this file, and it stops being outstanding.

---

## Setup

- **macOS**: VoiceOver, `Cmd + F5`. Rotor is `Ctrl + Option + U`.
- **Windows**: NVDA, free. Elements list is `Insert + F7`.
- **Use the keyboard only.** If you touch the mouse you are testing something
  else.

Open each page below and work through its checks. **Write down what you hear**,
not whether it worked — "announced as 'button'" is a finding; "fine" is not.

---

## 1 · `templates/web/Web.dc.html` → Contact — the form path

The richest form in the system. (Retargeted twice on 1 Sep 2026: this pass first
named the Academy kit, then the website kit; both are gone. The template's
contact band carries the labelled fields, the required marks, the hint and the
live-validation pattern. The kit's modal, toast, dropdown and tooltip demos went
with it — test those on the component cards in `components/feedback/` and
`components/overlays/`.)

- [ ] Tab from the top. Does the skip link come first, and does it work?
- [ ] Every field: is the **question** announced, or just the type? A field
      announced as "edit text" has lost its label.
- [ ] Type a bad email into the Email field. Is the error announced **when it
      appears**, or only when you tab back? The error is wired with
      `aria-describedby`; that only helps if focus returns.
- [ ] The hint under Email — is it read, and is it read *once*?
- [ ] The `ChoiceGroup`: is each option's required state announced?
- [ ] The vertical `Select`: does it open with the native picker?
- [ ] Submit. Is the success toast announced? It is `aria-live="polite"`, so it
      should wait its turn rather than interrupt — does it ever get read at all?

**The known risk here** is the toast. Polite regions are routinely missed when
focus moves at the same moment.

## 2 · `templates/app/App.dc.html` — the app path

(Retargeted 1 Sep 2026: this pass named `ui_kits/web_app/index.html`, now
`acds-template-app`. Same four screens, same shell.)

- [ ] Sign in with the keyboard only. Reachable? Is the error announced?
- [ ] In the shell: does the side nav announce as a navigation landmark, and is
      the current item announced as current?
- [ ] Collapse the nav to the rail. **Are the labels still announced?** They are
      hidden visually and kept in the DOM specifically for this. If they are not
      read, that decision failed and needs `.ac-sr-only` instead.
- [ ] Tab to the table. Is it announced with its caption and its row and column
      count?
- [ ] Select a row with Space. **Is the selection announced?** The row carries
      `aria-selected`; whether that is spoken in a `<table>` varies by reader,
      and this is the check most likely to fail.
- [ ] Once rows are selected the toolbar is replaced by the bulk bar. Is that
      change announced? It is `role="status"`.
- [ ] Open a row's drawer. Does focus move into it? Does Escape return focus to
      the button you came from?
- [ ] Settings → Preferences. Do the switches announce as switches, on and off?
- [ ] Settings → Danger zone. Does "Delete workspace" announce its consequence,
      or only its label? The consequence is in a sibling paragraph and is
      probably **not** associated with the button.

**Two known risks**: the rail labels, and the danger-zone buttons.

## 3 · `components/forms/app-inputs.card.html` — the new controls

Every one of these was written this week and none has been heard.

- [ ] `SegmentedControl`: announced as a radio group with a position — "2 of 3"?
- [ ] `NumberInput`: do the stepper buttons announce as "Increase" / "Decrease",
      and is the new value announced after pressing one?
- [ ] `Slider`: is the value announced as you arrow, and does it carry a unit?
- [ ] `Combobox`: typing filters the list. **Is the number of matches
      announced?** There is no live region for it — this is a probable finding.
- [ ] `DatePicker`: arrow onto a day. Is the full date read? Is "today"
      distinguishable from "selected" **by ear**? Visually one is an outline and
      one is a fill; a reader gets `aria-selected` for the selection and
      **nothing at all** for today.
- [ ] `FileUpload`: is the drop zone announced as a file input? Is the failed
      row's error associated with that row, or read as loose text?

**Three known risks**: combobox match count, "today" in the date picker, upload
error association.

## 4 · `templates/deck/Deck.dc.html` — the deck

- [ ] Is each slide announced in order?
- [ ] The slide number comes from a **CSS counter**, so it is generated content.
      Is it read? Almost certainly not — which may be fine, or may mean slide
      numbers should be real text.
- [ ] Are the chevrons silent? They should be — they are decoration.

---

## Findings

Write them here. Date each one. A finding with no measurement or quote is not a
finding — say what you heard.

```
Date        Page                Heard                              Verdict
----------  ------------------  ---------------------------------  --------
(empty)
```

---

## What to do with what you find

- **A wrong or missing announcement** is a component bug. Fix it in the
  component, not in the page that surfaced it — that is what fixed the inverse
  surface once instead of six times.
- **A confusing label** is a copy bug. `guidance.md` has the voice rules;
  microcopy is where a voice actually lives.
- **A silence that should be silence** is a pass. Write that down too, so the
  next person does not re-test it.
