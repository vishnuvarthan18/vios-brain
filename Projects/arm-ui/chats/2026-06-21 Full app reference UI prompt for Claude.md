---
tags: chat
date: 2026-06-21
source: Claude personal account
uuid: b4d38d22-fe30-40c9-91f0-1d1d9e8636fd
---
# Full app reference UI prompt for Claude

## Summary
**Conversation Overview**

The person is building araMetrics, a modular B2B super-app, and asked Claude to provide a complete prompt they could give to Claude Design to generate reference UI mockups for the application. Claude produced a detailed prompt specifying the full design system (Radix Themes, amber accent `#F9BF3B`, Sand gray scale, Poppins font, corporate-minimal aesthetic) and outlined three surfaces to generate: the Core Shell (login, Home variants for regular users and super-admins, profile/settings), the Admin Portal (seven nav sections with role variants for admin vs. operator), and the Calendar Module (Calendar Merger with empty and populated states). The prompt also specified that all screens should cover empty, loading, error, and success states using minimal realistic data.

The person then asked what the prompt would actually do, and Claude clarified that Claude Design produces visual reference mockups rather than production code, and that its output should not be treated as a source of truth alongside the tuned Radix codebase being built in Claude Code as part of Stage 5 of their pipeline. Claude also flagged a practical concern about generating all 15+ screens at once, noting the risk of visual drift across screens and recommending a surface-by-surface approach starting with the Shell. The conversation reflects that the person is working within a defined build pipeline with distinct stages, and that Claude Design output is positioned as exploration and stakeholder reference only.

## Chat

**Vishnu:** give me a full promt ot give ot claude deisn to genreat some refreacce ui for full app

**Claude:** Here's a prompt to give Claude Design to generate reference UI for araMetrics:

---

**Generate reference UI screens for araMetrics — a modular B2B super-app. Web-only, desktop-first.**

**Design system (apply to every screen):**
- Built on Radix Themes
- Amber accent `#F9BF3B` — used ONLY on primary buttons, active nav item, focus rings
- Neutral: Sand gray scale, base `#555555`
- Font: Poppins
- Aesthetic: clean, minimal, corporate-professional. Generous whitespace, restrained borders, no decorative color.

**Architecture: ONE role-aware shell.** Every screen renders inside this frame:
- Persistent left sidebar = app launcher listing only permitted apps
- Global shell-wide search bar (top)
- Role-adaptive Home/dashboard
- Profile/settings
- Collapsible sidebar, reflowing content (responsive-aware, but render desktop)

**Generate these screens as connected flows, grouped by surface:**

**1. Core Shell**
- Login screen (single shared auth)
- Shell empty/loading state
- Regular-user Home — personal summary: their apps, upcoming calendar, quick actions. Sidebar shows only Home, Calendar, Profile.
- Super-admin Home — operational platform-health overview: system status, API traffic, active users, error count. Sidebar adds Admin Portal.
- Profile/settings
- Quiet admin context label shown when super-admin enters Admin Portal

**2. Admin Portal** (super-admin app — invisible to regular users)
Seven nav sections: Overview, Users, Monitoring, Calendar Ops, Logs, Security, Audit Log.
- Overview — platform-health dashboard
- Users — list/table (Phase 1 priority): search, role badges, status, enable/disable, row actions. Include user detail/profile view.
- Monitoring — API traffic + performance + infrastructure
- Calendar Ops — sync operations status
- Logs — application log viewer
- Security — auth/security activity, session list with revoke
- Audit Log — immutable log table (view-only, admin-export button)
Show two role variants where they differ: `admin` (full actions) vs `operator` (role assignment, delete, export disabled/hidden).

**3. Calendar Module** (user-facing)
- Calendar Merger — multiple Google Calendars mirrored and merged into one unified availability view. Include connect-calendar empty state and merged populated view.

**Cover all states per screen:** empty, loading, error, success — not just happy path.

**Use minimal realistic data**, not lorem ipsum. Production-ready fidelity, not throwaway mockup.

---

Two adjustments before you send it:

This is your Stage 5 (Generate) work — your locked pipeline says code-first in Claude Code on tuned Radix, then push to Figma. Claude Design gives you fast visual reference, but its output won't be your tuned Radix codebase. Use it for **exploration/stakeholder reference**, then build the real thing in Claude Code, or you'll have two diverging sources of truth.

If you want, tell it to generate **one surface at a time** (Shell first) rather than all three at once — you'll get tighter, more consistent screens and can correct direction before it propagates errors across 15+ screens.

**Vishnu:** hey what will this promt do

**Claude:** It tells Claude Design to generate visual reference mockups of your araMetrics UI — login, both Home variants, the seven Admin Portal sections, the Calendar Merger, plus empty/loading/error states for each — all styled with your amber/Sand/Poppins system on Radix.

Two things to be clear on:

**It produces pictures, not your build.** Claude Design outputs visual screens for exploration and stakeholder review. It is *not* the tuned Radix codebase you're building in Claude Code (Stage 5). Treat its output as reference, not your source of truth — otherwise you get two diverging systems.

**It's a lot at once.** ~15+ screens across three surfaces in one shot. It'll drift — sidebar widths, spacing, label w
