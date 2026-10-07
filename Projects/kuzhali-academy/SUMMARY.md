---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: Kuzhali Academy website

## 1. What this project is
- **Goal:** A public website for Kuzhali Tuition Centre and Kuzhali Academy in Erode, Tamil Nadu.
- **Who it is for / client:** Kuzhali Tuition Centre & Academy, Erode (students and parents looking for tuition and courses).
- **Why it exists:** To show courses, take enquiries over WhatsApp and phone, and give the centre an online presence.

## 2. Status now (as of 2026-10-06)
- No work since 2026-08-23. No trace of Kuzhali in personal mail, calendar or Keep (checked Oct 2026).
- Static site (plain HTML, CSS, JS). No build step, no backend, no database. It is a "digital brochure", not a web app.
- New look done: indigo + orange + gold palette (was teal + orange). All CSS, 17 illustrations, favicon and inline SVGs recoloured.
- Live on Cloudflare Pages: https://kuzhali-academy.pages.dev (deployed by direct upload with Wrangler).
- Code on GitHub: vishnuvarthan18/kuzhali-academy (public, main branch).
- Not done: real domain, confirmed phone numbers, full street address, real testimonials and results, real fees (cards say "Fees on call").
- Not done: Cloudflare Pages is not connected to the GitHub repo yet, so deploys are manual.

## 3. Next steps
1. Confirm the correct phone numbers (flyers show three different ones) and update the 4 places in the code.
2. Add the full street address and a Google Maps link.
3. Decide the real domain (e.g. kuzhaliacademy.in) and add it as a custom domain in Cloudflare Pages; update canonical, og tags, robots.txt, sitemap.xml.
4. Connect Cloudflare Pages to the GitHub repo for auto-deploy on push.
5. Add real testimonials and top-scorer results when available.
6. Test the WhatsApp button on a real phone.
7. Decide if the repo should stay public or become private.

## 4. Decisions
- 2026-08-23 — Bold redesign: indigo #4338CA brand, vivid orange #FF5A1F accent, gold #FFB800, lavender hero — Vishnu wanted a better colour and look and feel. #decision
- 2026-08-23 — Host on Cloudflare Pages (project kuzhali-academy) — Wrangler was already installed and logged in. #decision
- 2026-08-23 — Put the code in a GitHub repo (vishnuvarthan18/kuzhali-academy) — Vishnu asked to push it. #decision
- (date not sure) — No testimonials on the site until real ones exist — inventing student quotes or marks would be dishonest. #decision
- (date not sure) — Fees shown as "Fees on call" until real figures are ready. #decision

## 5. Timeline
- 2026-08-23 — Repo created on GitHub and code pushed (public, main).
- 2026-08-23 — Deployed to Cloudflare Pages at kuzhali-academy.pages.dev.
- 2026-08-23 — UI redesign: teal to indigo/orange/gold; fixed a leftover teal focus glow; deepened lavender hero so illustrations stay visible; added ?v=2 cache-busting to CSS links.
- 2026-08-23 — Site run locally at http://localhost:5500 with python http.server; launch config saved in .claude/launch.json.
- (before 2026-08-23) — Site first built (folder ~/Downloads/kuzhali-academy) (not sure when).

## 6. Key facts
- **People:** [[People/Vishnu]] — builder and owner of the work
- **Companies:** Kuzhali Tuition Centre & Kuzhali Academy, Erode (client)
- **Tools:** [[Tools/Cloudflare]], [[Tools/Wrangler]], [[Tools/GitHub]], [[Tools/VS Code]], [[Tools/WhatsApp]], [[Tools/Claude Code]]
- **Links / repos / servers / file paths:**
  - Live: https://kuzhali-academy.pages.dev
  - Repo: github.com/vishnuvarthan18/kuzhali-academy
  - Local folder: ~/Downloads/kuzhali-academy (personal Mac)
  - Deploy: `wrangler pages deploy .` from the folder
- **Tech:** index.html; css/tokens.css (colours), base.css, layout.css, components.css; js/main.js (sticky nav, scroll reveal, filter tabs, WhatsApp forms, WHATSAPP_NUMBER); assets: 17 unDraw illustrations, 6 Fluent Emoji icons (MIT); fonts Raleway + Inter (Google Fonts); tested 280px to 1920px.
- **Related:** [[Projects/vidivu/SUMMARY]] (Vishnu's studio)
- **Dev sessions:** personal-mac__Downloads-kuzhali-academy__2026-08-23_e056ecff (archived: Projects/kuzhali-academy/claude-code/personal-mac__Downloads-kuzhali-academy__2026-08-23_e056ecff.md)

## 7. Files and documents
- `README.md` — how to run, what to change, pre-launch checklist, deploy options, licences (in the repo; copy in Raw/mac-personal/Downloads/kuzhali-academy/)
- `robots.txt`, `sitemap.xml` — still use placeholder domain kuzhaliacademy.in

## 8. Open questions and problems
- Which phone numbers are correct? Flyers show three different numbers.
- What is the real domain?
- What is the full street address?
- Is the project still active? No work since 2026-08-23.
- Should the GitHub repo be private?
- Raster assets (favicon PNGs, og-image) may still have old colours (not sure).

## 9. All chats in this project
- Run locally, redesign, deploy to Cloudflare, push to GitHub (archived: Projects/kuzhali-academy/claude-code/personal-mac__Downloads-kuzhali-academy__2026-08-23_e056ecff.md) — 2026-08-23
