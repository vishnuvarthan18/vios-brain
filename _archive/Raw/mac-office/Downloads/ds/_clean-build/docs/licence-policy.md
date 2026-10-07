> **Ported verbatim from the araCreate Design System repository, `docs/licence-policy.md`.**
> This is araCreate's own record and has not been rewritten. Path references
> (`src/`, `docs/`, `tests/`, `make` targets) point at that repository, not at
> this project.
>
> **One difference in this project:** Monument Extended IS self-hosted here, from `assets/fonts/`, because the licensed files were supplied directly. That is a licence decision the client made knowingly — see the Type section of `readme.md`.

---

# araCreate Design System — build policy

**Version 1.0 · 19 August 2026 · applies to every component in this system**

This document exists so that anyone joining this project later understands why
the code is written from scratch rather than copied, and does not "helpfully"
paste template code back in.

## The situation

araCreate's live website was built on **Duotint Pro**, a Webflow template bought
under Webflow's Single-Use Licence. That licence permits the template's design
on **one End Product** — one website — and states:

> "You can't extract a single component out of a template and use it outside the
> scope of the End Product."

araCreate Group intends to run several websites (Academy, Meditate, service
landing pages, and more). Some may end up on their own web addresses. Copying
Duotint's components into a shared library that serves all of them is exactly
what the licence forbids.

## The line we work to

### Locked exactly as-is — araCreate's own property

1. **Colours.** Brand yellow `#f9bf3b` and brand grey `#555555` are taken from
   araCreate's own logo files, which predate and are independent of the
   template. Charcoal `#2e2e2e`, canvas `#f6f6f6`, black and white likewise.
2. **Typefaces.** Poppins and Red Hat Mono, both open-licence (SIL Open Font
   License), free for unlimited commercial use on unlimited sites. No
   restriction whatsoever.
3. **The logo, graphics and photography.**
4. **All copy and content.**
5. **Sizes, spacing values and proportions** — these are measurements, not
   creative expression.

### Rebuilt from scratch — the template author's property

1. **HTML structure.** Every component's markup is authored fresh.
2. **CSS.** No rule is copied from `aracreate-template.webflow.css` or
   `aracreate.webflow.shared.css`.
3. **Class names.** Ours are prefixed `ac-` — `ac-button`, `ac-card`,
   `ac-nav`. Duotint's names (`cta-button`, `colour-01-light`,
   `block-mouse-focus-on-current-page`) do not appear anywhere in this system.
4. **JavaScript behaviour.** Written fresh, and with no jQuery dependency.

### Also removed

Two variables in the live site still hold Duotint's own navy palette
(`--button-colour-gray-light: #2f354599` and `--photo-overlay: #2e419e`). These
are replaced with araCreate values. Webflow's factory defaults — the blue focus
ring `#3898ec`, the blue link `#0082f3`, the pink error box `#ffdede` — are
likewise replaced.

## Why this is sound

A design's *visual character* — a yellow button with a dark label, a card with a
20px corner radius — is not what the licence protects. The licence protects the
template author's **code**. Reimplementing an appearance in original code is
ordinary practice; extracting the author's stylesheet and shipping it on five
sites is not.

Two facts strengthen the position materially:

1. **araCreate had already replaced Duotint's palette.** The template ships mint
   green, teal and navy. None of those colours appear on aracreate.group. The
   visual identity being locked into this system is araCreate's, not Duotint's.
2. **The brand colours are provable from an independent source** — the logo SVG
   files, which contain `#555555` and `#f9bf3b` and nothing else.

## What stays as it is

**aracreate.group itself remains the licensed End Product.** Its current
Duotint-based code is untouched and continues to run under the existing licence.
Nothing in this policy requires rebuilding the live site. When araCreate chooses
to move the main site onto this design system, that is an improvement, not an
obligation.

## Not legal advice

This is an engineering policy written to keep the project on the right side of a
licence, not a legal opinion. If araCreate ever sells, sublicenses or
white-labels this design system to third parties, that is a different question
and warrants a lawyer's view.

---

*Reference: https://webflow.com/templates/template-licenses*
