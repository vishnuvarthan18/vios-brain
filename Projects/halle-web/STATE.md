---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# STATE: Halle website (B. Halle Webflow UI fix)

Full history: [[Projects/halle-web/SUMMARY]] · Log: [[Projects/halle-web/LOG]] · Dev: [[Projects/halle-web/DEV-LOG]]

## Where we are (as of 2026-10-06)
- Last site work: 2026-09-09. Style Guide page built in [[Tools/Webflow]] and published on staging (halle-dev.webflow.io/style-guide) for the dev team.
- Figma has a "Design System" page with tokens, 8 text styles and 8 components.
- Line-height mess fixed: no overrides, one `normal` on the page wrapper.
- Buttons match Figma (Contact Us white/navy, Regular weight, Ionicons arrow, 24px icon, 8px/12px corners).
- Responsive rules table (992 / 768 / 480) is on the Style Guide.
- Home hero (Figma SVG curve), hero/logos split and commemorative card done on staging (July).
- Font is still a stand-in, not Helvetica Neue. Other live pages not fixed yet.
- Server work (2026-10-05) now lives in [[Projects/feedback-widget/SUMMARY]].

## Next steps
1. Vishnu does a visual check of the published style guide.
2. Dev team builds hover/focus/disabled states, mobile menu, mobile hero.
3. Swap font to Helvetica Neue (Figma desktop + Webflow font file).
4. Use the guide to fix the other pages (nav, H1s, hidden sections); carry out the July text hierarchy plan.

## Blockers
- Helvetica Neue not available to Claude.
- Team sign-off needed on guessed states.
- Real icons and product photos not received.

## Key places
- Staging: halle-dev.webflow.io/style-guide
- Figma: figma.com/design/A7xoUgwhqye2UmZ7R2Obfa/www.b-halle.de (Design System page node 374-362)
- Docs: `halle-standard-design-values.md`, `halle-webflow-style-guide-progress.md`, `halle-design-system-next-steps.md`
- Brand guide: B-Halle-Optik-Logodesign-Guide-V01.pdf (archived: Raw/mac-personal/Vishnu/projects /creative work/Branding/B-Halle-Optik-Logodesign-Guide-V01.pdf.md)
