---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: The Regen Room website (theregenroom.co.uk)

## 1. What this project is
- **Goal:** Run and support the theregenroom.co.uk website on [[Tools/Webflow]], with [[Tools/Kartra]] forms and emails attached. Keep it working and within hosting limits.
- **Who it is for / client:** The Regen Room (REGEN, cellular wellness therapies), via araCreate's client [[Companies/Future State]] (FST). Contacts: [[People/Shay Lynch]] and [[People/Meiraj]]; [[People/Shyam]] (araCreate) checks the hours budget. Client talks to Vishnu on [[Tools/WhatsApp]].
- **Why it exists:** The client runs campaigns (like the "Perimenopause Reset Programme") through the site. Vishnu handles the site, forms and email setup.

## 2. Status now (as of 2026-10-06)
- Last site work logged: 2026-09-11. Sept 2026: ~19 h of Vishnu's time on home, about, contact page design, user story mapping and user flow tests.
- Earlier build work (Jul–Aug, Claude Code): nav fixes (07-21/22), Perimenopause Reset Programme page built from Figma and published to staging (07-29, 07-31), video thumbnail, testimonial carousel, Kartra popup, mobile bugs (08-03 to 08-05), SEO/GEO work (08-11).
- Done: Welcome email automation for the Perimenopause Reset Programme form, tested and working.
- Done: Same welcome email sent to the people who applied before the automation existed.
- Done: Webflow bandwidth fix. 8 big images compressed to WebP, site published 2026-09-11 16:14 IST and checked live.
- Not done (parked): 4 big MP4 videos on the Perimenopause Reset Programme page (4.2GB of bandwidth).
- Not done (parked): a smaller batch of images (~1.5GB).

## 3. Next steps
1. Fix the 4 videos: set preload=none, or host them on Vimeo.
2. Compress the smaller image batch (~1.5GB).
3. Watch Webflow bandwidth next month to see the saving is real (projected ~11GB instead of ~30.57GB).
4. Check that new form sign-ups keep getting the welcome email.

## 4. Decisions
- 2026-09-01 — Nothing is sent or saved without Vishnu's OK. #decision
- 2026-09-01 — Everything stays inside Kartra, no Gmail. — Vishnu wanted all mail from Kartra. #decision
- 2026-09-01 — Use a Kartra Sequence ("Perimenopause Reset") that starts when the form is filled, not a separate automation. — Kartra sends email only through sequences. #decision
- 2026-09-01 — Do new sign-ups first, then old applicants. #decision
- 2026-09-01 — Leave the Kartra sender as default (James O Mahoney, hello@theregenroom.co.uk). #decision
- 2026-09-01 — Emails come from Renée's personal voice ("Renée x"). — Client asked; it is a female campaign. #decision
- 2026-09-01 — Do not put 3 names in the signature; instead list the 3 partners in the email body. #decision
- 2026-09-01 — Leave out the one contact who came via a different form ("Lead Magnet - 1"). #decision
- 2026-09-11 — Compress images in place to WebP; park the videos and smaller images for later. #decision
- 2026-07-22 — Claude does not publish to production; Vishnu pushes live himself (staging theregenroom.webflow.io allowed on 07-29). #decision
- 2026-07-22 — Audit and plan first, no changes before approval. Group nav items into dropdowns instead of shrinking text. #decision
- 2026-07-29 — Perimenopause Reset Programme page: reuse nav and footer, native Webflow elements, match Figma section by section. #decision

## 5. Timeline
- 2026-09-11 — Site published 16:14 IST with compressed images; checked live.
- 2026-09-11 — 8 images compressed to WebP (16.8MB to 0.53MB). Saving ~19.3GB per quarter.
- 2026-09-11 — Webflow connector reconnected to the right workspace.
- 2026-09-11 — Found cause: not traffic (363 visitors in 30 days) but file size. 7 "-clean" PNGs re-uploaded ~30x bigger used 18.8GB; 4 MP4s used 4.2GB.
- 2026-09-11 — Webflow warned the site used 50% of its 50GB monthly bandwidth (CMS Hosting plan).
- 2026-09-01 — Welcome email sent to the earlier applicants via the sequence.
- 2026-09-01 — Automation tested; Vishnu confirmed "tested correct and working".
- 2026-09-01 — Final email copy agreed with the client (partner list with names).
- 2026-09-01 — Client asked: did we create a thank-you email for the Perimenopause campaign? Send it to those already registered.
- 2026-08-11 — SEO/GEO work: Search Console, Cloudflare purge, schema, alt text, image compression.
- 2026-08-05 — Mobile bugs fixed (cropped quote photos, banner over Join button); an unwanted push to production, then backup/restore fix.
- 2026-08-03 — Custom video thumbnail, testimonial carousel, Kartra popup form, mobile nav.
- 2026-07-31 — Perimenopause Reset Programme page refined to Figma (nav, video cards, apply section, responsive).
- 2026-07-29 — Perimenopause Reset Programme page (`/perimenopause-reset-programme`) built in Webflow and published to staging; fixed section by section against Figma; marquee added.
- 2026-07-22 — Nav fixes: logo/hamburger alignment, desktop nav regrouped into dropdowns; mobile full-screen menu attempt reverted.
- 2026-07-21 — Webflow MCP re-authorised to reach theregenroom.co.uk.

## 6. Key facts
- **People:**
  - [[People/Vishnu]] — runs the site and setup.
  - [[People/Renée LeBlanc]] — Founder, Elevated Wellness by Renée; emails go out in her voice.
  - [[People/James O'Mahoney]] — REGEN; Kartra sender name.
  - [[People/Shay Lynch]] — Shay Lynch, REGEN (named with James in the email). Likely the client contact (not sure).
  - David — leads Nuvivo (advanced health testing).
- **Companies:** The Regen Room / REGEN; Elevated Wellness by Renée; Nuvivo; [[Companies/Future State]] (not sure).
- **Tools:** [[Tools/Webflow]] (CMS Hosting plan, 50GB/month), [[Tools/Kartra]], [[Tools/Cloudflare]] (not helping, assets come from Webflow CDN), [[Tools/WhatsApp]].
- **Programme:** "Perimenopause Reset Programme" — 3 partners: REGEN (cellular wellness therapies), Elevated Wellness (coaching and behaviour change), Nuvivo (advanced health testing).
- **Kartra setup:** form id 3 "Perimenopause Reset Programme"; sequence "Perimenopause Reset"; step "Welcome Email", sends immediately; subject "Thanks for registering!"; merge tag {first_name}.
- **Links:** theregenroom.co.uk; Webflow staging theregenroom.webflow.io; Figma files in `aracreate/fst/the regen room/` (office Mac).
- **Dev history:** [[Projects/the-regen-room/DEV-LOG]] (Claude Code sessions, office Mac; includes sessions moved from tech-to-me and ac-training). Chats index: INDEX (archived: Projects/the-regen-room/chats/INDEX.md).
- **Related content:** "CEO Circle" brochure (Future State 2026) and AIBF Cork talk slides.
- **Related:** [[Projects/system/SUMMARY]] (463 MB duplicate room images to remove).

## 7. Files and documents
- `webflow-bandwidth-fix-sep-2026.md` — the bandwidth analysis and fix (Claude project doc).
- Final welcome email text — in the 2026-09-01 chat.

## 8. Open questions and problems
- 4 videos still use a lot of bandwidth; not fixed.
- Smaller image batch (~1.5GB) not fixed.
- Mobile menu covers the header when open; mobile dropdowns cannot collapse (from July nav work).
- Perimenopause page: full Figma match not confirmed; real pilot video and final content were placeholders in July.
- Chat recap says all 4 earlier applicants got the email; Vishnu's own words say "the person who filled the form earlier". Exact count not sure.
- Kartra cannot be opened by Claude (blocked in built-in browser); Vishnu must click.

## 9. All chats in this project
- Email replies for applicants (archived: Projects/the-regen-room/chats/2026-09-01 Email replies for applicants.md) — 2026-09-01
- Problem meeting (archived: Projects/the-regen-room/chats/2026-09-11 Problem meeting.md) — 2026-09-11
