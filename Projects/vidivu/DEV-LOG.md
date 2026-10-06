---
tags: project
updated: 2026-10-06
---
# DEV-LOG: Vidivu (vidivu.in)

Coding history from Claude Code sessions on the personal Mac. Business side: [[Projects/vidivu/SUMMARY]]

## What was built
- Vidivu website: Next.js 16 + React 19 + Tailwind CSS v4. Dark "motorsport engineering" design (black canvas, white type, blue-to-red signature stripe). Repo: vishnuvarthan18/vidivu.in- . Local copy: ~/vidivu.in.
- By the Aug 2026 Cowork work the site had 15 pages: Home, Services hub + 6 service pages (Product Engineering, UI/UX Design, MVP Sprints, Cloud & DevOps, API & Systems Integration, Retainers), Industries, Process, Technology, About, Contact, /startups, /smes. SEO: sitemap, robots, per-page metadata, JSON-LD.
- Real logo (vidivu-logo.svg) put in the navbar and footer (was a stripe + text placeholder).
- Separate practice build in ~/Desktop/vidivu.in: a "Halo / USD Halo" stablecoin landing page (React + TypeScript + Vite + Tailwind + lucide-react) from a long spec prompt (not sure if this was for Vidivu or a test).

## Timeline (newest first)
- 2026-08-10 — Ran local dev server again (http://localhost:3000); clean, no errors.
- 2026-08-10 — Shared brand colours from DESIGN.md / globals.css.
- 2026-08-10 — Swapped real vidivu-logo.svg into navbar, then footer (copied asserts/vidivu-logo.svg to public/).
- 2026-08-10 — Cloned github.com/vishnuvarthan18/vidivu.in- into ~/vidivu.in, npm install (359 packages, 0 vulnerabilities), ran Next.js dev server.
- 2026-08-10 — Tried to run the Halo Vite project; node_modules was broken, reinstalled; stopped by Vishnu.
- 2026-08-03 — Built the Halo / USD Halo landing page (Hero with video + brand marquee, Info, Backed By marquee, Use Cases). Build passes; TT Norms Pro font files missing (licensed, must be added by hand).
- 2026-08-02 — Tried /design-sync to Claude Design from an empty folder; nothing to sync.

## Decisions
- 2026-08-03 — Halo page uses only lucide-react for icons, no other UI libraries (from spec). #decision
- 2026-08-10 — Use the real SVG wordmark in navbar and footer via next/image. #decision
- (Aug 2026) — No published pricing; never say solo or part-time; no fabricated proof; WhatsApp Business is the main contact channel; no Tamil script on site (meaning carried in English hero copy). #decision
- (Aug 2026) — Hold Tier-2 city landing pages until there is real capacity. Top nav capped at 6 items. #decision

## Brand colours
- Signature stripe: #3ba0e0 (sky blue), #1c69d4 (royal blue), #e22718 (power red) — accent only, never CTA or background.
- Canvas #000000, primary/text #ffffff, cards #1a1a1a, body text #bbbbbb, muted #7e7e7e, hairline #3c3c3c. 0px radius. Inter 800 display / 300 body.

## State at last session (2026-08-10)
- Site runs locally on port 3000 with the real logo.
- Not deployed on Vercel yet.
- Contact details (WhatsApp, email, founder name/LinkedIn) are still TODO placeholders — a lead cannot reach Vidivu yet.
- Media is generated dummy art; case studies say "coming soon".

## Open items
- Fill real contact details in app/contact and app/about.
- Deploy to Vercel and get real Lighthouse numbers.
- Replace dummy media with real screenshots/photos.
- Publish real metric-led case studies.
- Delete unused build-loop.mp4.
- Halo page: add TT Norms Pro woff2 files (not sure if still needed).

## Session index
- [[Projects/vidivu/claude-code/personal-mac__vidivu-in__2026-08-10_9fc48207]] — 2026-08-10 — run local server again
- [[Projects/vidivu/claude-code/personal-mac__vidivu-in__2026-08-10_1eeeceb9]] — 2026-08-10 — run dev server, check homepage
- [[Projects/vidivu/claude-code/personal-mac__vidivu-in__2026-08-10_c33c21b0]] — 2026-08-10 — clone repo, run, brand colours, logo in navbar + footer
- [[Projects/vidivu/claude-code/personal-mac__Desktop-vidivu-in__2026-08-03_97cdc351]] — 2026-08-03 — build Halo / USD Halo landing page; later run attempt
- [[Projects/vidivu/claude-code/personal-mac__Desktop-vidivu-in__2026-08-02_e088fd8b]] — 2026-08-02 — /design-sync tried in empty folder
