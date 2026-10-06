---
tags: company
aliases: [DSA, DreamSpace, DreamSpace Productions, DSP, DreamSpace Pvt Ltd]
updated: 2026-10-06
---
# DreamSpace Academy
- Non-profit social enterprise in Sri Lanka (Batticaloa, Mullaitivu, Hatton); makerspace hubs for underserved youth.
- Sister org of araCreate; DreamSpace Productions (DSP) is linked. On the group map as "DreamSpace Pvt Ltd".
- Concept report 27 Jul 2025 (authors Aravinth, Rathees, Abilajini).
- 2025 work: social media training, DSP business reviews (with [[Companies/Asia Berlin Forum]]), ABS design, DSP pitch deck.
- Dec 2025 → now: DSA website, template selection, DSA design system (Vishnu organised, Sep 2026), dsa-app.
- dsa-app Phase 1 "Makerspace" (Sprints 1–10): auth + permissions, students, sessions/timetable, attendance, payments/receipts, portfolios + QR certificates, notifications + PWA, open sessions, reporting.
- Stack: Express + Prisma API, React + Vite PWA, Next.js public pages, Capacitor mobile later; pnpm monorepo.
- Hosting: one Dokploy VPS (app, api, verify, showcase subdomains); Supabase for Postgres, staff auth, storage. Main dreamspace.academy is a separate marketing site.
- Go-live: new Mumbai Supabase project; schema via `prisma db push` (no migrations — data-loss risk).
- Latest release: payments self-heal for current month, server-side filters, API refuses prod boot with mock auth; single "Lilac" theme with dark mode (30 Jun 2026).
- Design system: cream base #FDF9F6, purple 700 #6A0BB2, orange 500 #E45B00 for one CTA per block; Poppins only; no gradients, no emoji. Copy: plain, community-first, sentence case.
- Meetings: DSA WEB: WEEKLY (Apr – Jul 2026), DSA x ACI: WEEKLY (from Jul 2026); Vishnu logs it weekly.
- Contacts: [[People/Rathees]], [[People/Kishoth]], [[People/Basith]], [[People/Erosha Rajendran]].
