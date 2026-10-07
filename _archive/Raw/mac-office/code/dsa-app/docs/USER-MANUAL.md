# DreamSpace Academy — User Manual

> **Audience:** Everyone who uses the platform — Super Admins, Makerspace Admins, Hub Coordinators, Trainers, Parents/Guardians, Volunteers and Members.
> **Scope:** Phase 1 (the Makerspace module). Features for Academy, Labs, Events, Competitions, etc. are scaffolded but not yet active.
> **Last updated:** 25 Jun 2026.

This manual explains, in plain language, what every feature does, what each role can do, and what terms like *"Subscribers only"* mean. It is the companion to the technical `CLAUDE.md` (which is for developers).

---

## Table of Contents

1. [What DreamSpace Academy Is](#1-what-dreamspace-academy-is)
2. [The Apps & Where to Find Them](#2-the-apps--where-to-find-them)
3. [Signing In](#3-signing-in)
4. [Roles — What Each One Is and Can Do](#4-roles--what-each-one-is-and-can-do)
5. [Key Concepts & Glossary](#5-key-concepts--glossary)
6. [Feature Guide (by Module)](#6-feature-guide-by-module)
   - 6.1 [Students & Enrollment](#61-students--enrollment)
   - 6.2 [Timetable & Sessions](#62-timetable--sessions)
   - 6.3 [Attendance, Class Work & Outcomes](#63-attendance-class-work--outcomes)
   - 6.4 [Payments & Receipts](#64-payments--receipts)
   - 6.5 [Portfolios](#65-portfolios)
   - 6.6 [Certificates](#66-certificates)
   - 6.7 [Open Sessions](#67-open-sessions)  ← *the "Everyone / Subscribers only" part*
   - 6.8 [Notifications & Announcements](#68-notifications--announcements)
   - 6.9 [Volunteers](#69-volunteers)
   - 6.10 [Reports, Users, Roles & Audit](#610-reports-users-roles--audit)
7. [Profile & Settings](#7-profile--settings)
8. [Public Pages (No Login)](#8-public-pages-no-login)
9. [Privacy, Child Safety & PDPA](#9-privacy-child-safety--pdpa)
10. [For Administrators](#10-for-administrators)
11. [Mobile App Notes & Updates](#11-mobile-app-notes--updates)

---

## 1. What DreamSpace Academy Is

DreamSpace Academy is the platform for a Sri Lankan social enterprise that runs **makerspace hubs** for underserved youth. In Phase 1 there are two hubs — **Hatton** and **Batticaloa** (both Tamil-speaking). It connects to DreamSpace's existing internal staff platform.

The platform helps staff **enrol students, run classes, mark attendance, record student work, manage fees, issue certificates, and run public workshops** — while protecting children's privacy at every step (students are minors, so Sri Lanka's PDPA applies fully).

---

## 2. The Apps & Where to Find Them

| App | Who it's for | Where |
|---|---|---|
| **Web app (PWA)** | Everyone — admins, coordinators, trainers, parents | `app.dreamspace.academy` (installable on desktop & phone) |
| **Mobile app** | Trainers & parents (a slice of the web app) | Android (Play Store) / iOS |
| **Public site** | Anyone, no login | Certificate verify, portfolio showcase, public events & open-session registration |

The web app is the **superset** — every capability lives there first. The mobile app is a tailored subset of the same product.

**Public sub-domains:**
- `verify.dreamspace.academy/DSA-CERT-XXXXXX` — verify a certificate from its QR code.
- `showcase.dreamspace.academy` — public portfolio showcase.
- Public events & open-session browsing/registration are reached from the public home.

---

## 3. Signing In

There is **one login screen** with a method toggle:

### Email & password (staff)
Super Admins, Makerspace Admins, Hub Coordinators and Trainers sign in with **email + password** (Supabase Auth).
- **Forgot password?** → enter your email → you get a reset link → set a new password.
- **First invite:** new staff receive an invite link and set their password on first use.

### Phone OTP (parents)
Parents/guardians can sign in with their **phone number**: they receive a one-time SMS code and enter it. (The phone number is the one captured when their child was enrolled.) Parents can also use email if an account was set up that way.

### Choosing a role (multi-role users)
If your account holds **more than one role** (e.g. you're both a Trainer and a Parent), after signing in you'll see a **"Choose how to continue"** screen. Pick the role you want to act as; you can switch any time from your profile. Each role opens to its own home screen.

---

## 4. Roles — What Each One Is and Can Do

Roles decide **what you can see and do**. A user can hold several roles; their abilities are the **combination** of all of them. Two roles are **hub-scoped** (they only work inside the hub(s) you're assigned to); the rest are platform-wide.

| Role | Scope | In one line | What they can do |
|---|---|---|---|
| **Super Admin** | Platform | Full control | Everything. Created only in the database (max 2), never assignable in the app. |
| **Makerspace Admin** | Platform | Runs the whole makerspace | Manage all hubs, users, students (incl. demographics), sessions, payments, portfolios, certificates, sponsorships, announcements, reports, and the audit log — across every hub. |
| **Hub Coordinator** | **Hub** | Runs one hub day-to-day | Enrol & edit students (incl. names/PII), manage the timetable & sessions, mark/override attendance, log class work, approve portfolios, **issue** certificates, manage payments (mark paid / waive / remind), review volunteers, run open sessions, view reports. **Cannot** see demographic data (admin-only). |
| **Trainer** | **Hub** | Delivers classes | View their sessions, **mark attendance**, mark learning **outcomes**, log **class work** & homework, create portfolio drafts and submit them, mark payments. **Sees students by DSA ID only — never names or personal details.** |
| **Parent / Guardian** | Platform (own children only) | Follows their child | View **their own child's** sessions, attendance, class work, outcomes, payments, portfolios and certificates. Register for open sessions. Ask to volunteer. Read-only — they can't change records. |
| **Volunteer** | Platform | Community helper | Minimal in Phase 1 (notifications). Expands in a later phase. |
| **Member** | Platform | Community member | View and register for **open sessions**; ask to volunteer. |

> **Future roles** (Academy Admin, Lab Coordinator, Event Coordinator, Donor, etc.) exist in the system but have **no abilities yet** — they're switched on when those modules are built. Don't assign them in Phase 1.

**How abilities are decided (three layers, in order):**
1. **Deny override** — an explicit block on a specific action for one person. *Always wins.*
2. **Grant override** — an explicit extra ability given to one person by a Super Admin.
3. **Role baseline** — the default abilities of your role(s).

So a Super Admin can give one trainer a special extra ability (grant), or take one away from a coordinator (deny), without changing anyone else.

**Temporary roles:** a role can be granted with an **expiry date** — it stops working automatically after that date (e.g. a cover coordinator for two weeks).

---

## 5. Key Concepts & Glossary

| Term | Meaning |
|---|---|
| **Hub** | A physical makerspace location (Hatton, Batticaloa). Most data belongs to a hub. |
| **DSA ID** | A student's identifier, e.g. `DSA-047313`. **This is how students are shown everywhere in day-to-day screens** — never by name. The number reveals nothing about hub or year. |
| **Robot avatar** | A unique cartoon robot generated from each DSA ID. Used instead of a student photo (no student photos exist in Phase 1). |
| **Hub-scoped** | Coordinators and Trainers only see/act within the hub(s) they're assigned to. Admins are platform-wide. |
| **Slot** | A recurring class time on the timetable (e.g. "Mon 09:00–11:00, Robotics L1"). Students are assigned to a slot at enrolment. |
| **Session** | A single dated class generated from a slot (e.g. "Mon 30 Jun, 09:00"). Attendance is taken on sessions. |
| **Outcome** | A learning achievement for a level. Marked complete (with proof of work where needed). |
| **Sponsored student** | A student whose fees are covered by a donor — their payments show as *Sponsored*, not unpaid. |
| **Subscriber / Member / Public** | Audience tiers for **open sessions** — see §6.7. |

---

## 6. Feature Guide (by Module)

Staff navigate via the left sidebar (you can switch it between an **icon dock** and a **labeled menu** in Profile → Appearance). You only see the menu items your role allows.

### 6.1 Students & Enrollment
**Menu: Students** · *Coordinators & Admins enrol/edit; Trainers view (DSA ID only).*

- **Enrol a student:** capture name, demographics (gender, district, etc.), **consent**, the **enrollment fee** status, the **agreed monthly fee**, an optional **slot**, and an optional **parent/guardian** link (by phone, so the parent can sign in).
- **Student record:** robot avatar, DSA ID, level, enrolment details, identity card (name/PII — consent-gated), demographics (admin-only), linked parents, and learning **outcomes**.
- **Status:** Active / Inactive / Graduated.
- **Privacy:** Trainers and operational lists show **DSA ID only**. Names and personal details appear only to coordinators/admins with the right permission, and demographic data is **admin-only**.

### 6.2 Timetable & Sessions
**Menu: Timetable & Sessions** · *Coordinators manage; Trainers run.*

- **Timetable** is where you define **recurring weekly slots** (name, day, start/end time) and the hub's **public holidays**.
- **Generate session instances:** turns the weekly slots into real, dated sessions you can run attendance on. Choose how far ahead (1 / 2 / 4 weeks). Public holidays are skipped, and running it again never creates duplicates. (This also runs automatically every Monday, two weeks ahead.)
- **Sessions** lists classes under **Upcoming** / **History** tabs, grouped by day. From here staff can:
  - **Cancel** a session (with a reason) — linked parents are alerted.
  - **Delay** a session to a new time (with a reason) — linked parents are alerted.
  - Add/remove specific students for one session (overrides).
  - Create a one-off **special session**.
- **Parents** see their child's upcoming timetable and get alerts when a class is cancelled or delayed.

### 6.3 Attendance, Class Work & Outcomes
**Menu: Attendance & Class work** · *Trainers mark; Coordinators can correct.*

- **Attendance:** pick a session, mark each student **Present / Absent / Late / Excused** (DSA ID only), add notes, then **Submit**. After submitting, attendance is **locked** — only a Coordinator/Admin can change it, and only with a reason (recorded in the audit log). Re-submitting the same data (e.g. after a phone reconnects) is safe and won't duplicate.
- **Class work:** trainers log what each student worked on. An **"apply to all"** option fills a note for everyone, then you fine-tune individuals. You can also post **group homework** (a note + an optional Google Drive link) — posting it **notifies parents**. No photos or ratings here (work photos live in Portfolios).
- **Outcomes:** per-level learning goals. Mark them complete; deliverable outcomes need a **proof-of-work** link (a photo of the finished work).
- **Parents** can see their child's attendance, class work notes and homework.

### 6.4 Payments & Receipts
**Menu: Payments** · *Coordinators & Admins manage; Trainers can mark paid; Parents view their own.*

- The system **auto-creates a monthly fee record** for every active student on the 1st of each month (Asia/Colombo). It never creates duplicates.
- Filter by **month** and **status**: **Paid, Unpaid, Overdue, Sponsored, Waived**.
- **Mark paid** (records the method — cash / bank transfer / online) → a **receipt** `RCP-XXXXXX` is generated.
- **Waive** a payment (reason required, audit-logged).
- **Send reminders** by SMS to parents of selected unpaid students (max one reminder SMS per person per day).
- **Sponsored** students show as *Sponsored*, not unpaid. The one-time **enrollment fee** is tracked separately from monthly fees.

### 6.5 Portfolios
**Menu: Portfolios** · *Trainers create; Coordinators approve; Parents share.*

The lifecycle: **Draft → Submitted → Published → (Showcased)**.
- A **Trainer** creates a project (title, summary, contributors by DSA ID, photos **of the work only**, and a rich-text story) and **submits** it.
- A **Coordinator** reviews and **approves** (publishes) or returns it.
- Approved projects can be **Showcased** (featured on the public site) or un-showcased.
- **Parents** see only their child's **published** projects and can **Share** a public link with family. *(Parents only get "Share" — they can't edit or showcase.)*
- **Privacy:** public projects credit children by **first name + DSA ID + robot avatar only** — never full names, and never student faces.

### 6.6 Certificates
**Menu: Certificates** · *Admins approve; Coordinators issue.*

- The system **flags** a student when they complete all outcomes in their final level (you can also check a student manually).
- **Approve** (admin) → **Issue** (coordinator) generates a **QR-verifiable** certificate.
- Anyone can scan the QR to verify it at `verify.dreamspace.academy/DSA-CERT-XXXXXX` — the public page shows validity and DSA ID, **never the student's name**.
- Certificates can be **revoked** (with a reason).

### 6.7 Open Sessions
**Menu: Open sessions** · *Coordinators & Admins create/manage; anyone (incl. the public) can register.*

Open sessions are **public-facing workshops/events** people can register for — separate from the regular student timetable. This is where the **"Everyone / Subscribers only"** options live.

#### Who can register? (the `accessType` options)
When you create an open session you pick **"Who can register"**:

| Option | What it means |
|---|---|
| **Everyone** | Anyone can register straight away (or from the public open time, if you set one). |
| **Subscribers get early access** | **Subscribers register first**; the general public has to wait until the public open time. Same session, two start times. |
| **Subscribers only** | **Only subscribers/members can ever register.** The general public is blocked entirely. |

**Who counts as what:**
- **Subscriber** — someone who **follows a DreamSpace subsidiary** (opted in to updates / early access).
- **Member** — someone who holds the **Member** role.
- **Public** — everyone else (including staff registering themselves, and walk-in guests).

*(Subscribers and Members are both treated as the "early" audience. Only the Public waits for the public window.)*

#### Registration windows (optional dates)
- **Subscriber opens at** — when subscribers/members may start registering.
- **Public opens at** — when the general public may start registering.
- **Registration closes at** — a hard deadline; nobody can register after this.

If a window hasn't opened yet, the person is told when it opens; after it closes, registration is shut.

#### Pricing
A session can be **Free** or **Paid**. For paid sessions you set a **Public price** (required) and optionally cheaper **Subscriber** / **Member** prices. If a tier price isn't set, that tier pays the public price. Currency defaults to **LKR**.

#### Seats & waitlist
Set **Max seats** (or leave blank for unlimited). When it's full, new registrants join a **waitlist**. If someone cancels, the **next person on the waitlist is promoted automatically** and notified. You can **raise the max seats after publishing** (e.g. when demand is high) — you just can't set it below the number already registered.

#### Where it's shown (audience flags)
- **Notify our students & parents** — on publish, alerts parents (in-app + push, and SMS/email where enabled).
- **Publish to public website** — lists it on the public site with a no-login registration page.
You can choose either, both, or neither (but pick at least one, or nobody will see it).

#### How the public registers (OTP-verified — anti-abuse)
To stop fake sign-ups, public registration is **two steps**:
1. The person enters their **name** and either a **phone number or email**, and requests a code.
2. They receive a **verification code** (SMS if they gave a phone, email if they gave an email) and enter it to confirm. Only then is the seat booked.

> *Current note:* email verification (Resend) is active now; phone/SMS verification turns on once the notify.lk SMS subscription is in place — no further setup needed in the app.

After registering they see **"You're registered"** or **"You're on the waitlist."** For paid sessions they're told to **pay at the hub**.

#### Managing registrations (staff)
- See sessions under **Upcoming / History**, grouped by day.
- **Registrations** list shows each person, their tier (subscriber/member/guest), waitlist/attended status, and payment status.
- **Mark paid:** record **Cash / Bank transfer / Online** plus an optional reference → status becomes *Paid* (audit-logged).
- **Mark attended** for check-in.
- **Edit** a published session (e.g. raise max seats, change time/price).
- **Cancel** a session (with a reason) — all registrants are notified.

### 6.8 Notifications & Announcements
**Menu: Notifications & Announcements** · *Anyone receives; Coordinators/Admins send announcements.*

Messages reach people through up to four channels:
- **In-app** — always on (can't be turned off).
- **SMS** — via notify.lk; **Tamil by default** for Hatton & Batticaloa.
- **Push** — on the mobile app / installed PWA.
- **Email** — for richer messages.

Each person controls SMS / Push / Email in their settings. **Transactional** messages (payment, session cancel/delay, homework) are immediate; **announcements** are rate-limited to **one SMS per person per day**.

### 6.9 Volunteers
**Menu: Volunteers** · *Parents/Members request; Coordinators/Admins review.*

Parents and Members can **request to volunteer**; coordinators and admins **review** and respond.

### 6.10 Reports, Users, Roles & Audit
*Admins (and some for coordinators).*

- **Reports** — impact and operational summaries; exportable.
- **Users** — create/edit user accounts, assign roles (with optional expiry), and set **permission overrides** (grant/deny a specific ability for one person).
- **Roles** — view role definitions and create custom roles. (Super Admin is locked and never editable/assignable in the app.)
- **Audit log** — an **append-only** record of sensitive actions (enrolments, payments, role changes, certificate issues, attendance overrides, etc.). Nothing here can be edited or deleted.

---

## 7. Profile & Settings

Open your avatar (top-right) → **Profile**:
- **Name** and **profile photo**.
- **Language:** English, தமிழ் (Tamil), සිංහල (Sinhala).
- **Appearance:** Dark mode, and **Sidebar style** — **Dock** (icon strip) or **Menu** (labeled) on desktop.
- **Notifications:** turn SMS / Push / Email on or off (in-app stays on).
- **Delete account:** deactivates your account and anonymises your personal data while preserving the audit trail (irreversible). *Super Admin accounts can't be deleted from the app.*

---

## 8. Public Pages (No Login)

| Page | What it shows |
|---|---|
| **Certificate verify** (`verify.dreamspace.academy/DSA-CERT-…`) | Whether a certificate is valid + the DSA ID. No names, no photos. |
| **Portfolio showcase** (`showcase.dreamspace.academy`) | Featured student projects — first name + DSA ID + robot avatar only. |
| **Public events** | DreamSpace event/blog posts. |
| **Open sessions** | Browse public workshops and register (OTP-verified, no login). |

---

## 9. Privacy, Child Safety & PDPA

This is non-negotiable — students are children.
- **No student photos** anywhere in Phase 1. Robot avatars are used instead.
- **DSA ID only** in trainer, attendance, payment and public views. Names appear only to coordinators/admins with permission.
- **Demographic data** (gender, district, income) is **admin-only** and consent-gated.
- **Consent** is captured at enrolment and tracked.
- **Right to erasure:** a student's/user's personal data can be anonymised on request, while the audit trail is preserved.
- The **audit log** is append-only — a permanent, tamper-resistant record of sensitive actions.

---

## 10. For Administrators

- **Super Admin** is bootstrapped in the database only (max 2) — it can't be created, assigned, edited, or deleted through the app.
- **Granting abilities:** prefer assigning the right **role**. For one-off needs, use a **permission override** (grant or deny a specific ability for one person, with a reason). **Deny always wins** over grant and role.
- **Temporary access:** assign a role or override with an **expiry date**; it switches off automatically.
- **Hub scope:** Coordinators and Trainers must be assigned to a hub; they only see that hub's data. Admins are platform-wide.
- **Automated jobs** run on schedule (all safe to re-run): generate sessions (weekly), create monthly fees (1st of month), expire temporary roles (daily), and retry failed staff-platform time-log pushes.

---

## 11. Mobile App Notes & Updates

- The mobile app is the same product as a slice for **trainers and parents**, with native niceties (offline attendance, push notifications). Parents have their own dashboard; staff get the same role-based menu as on the web.
- **Offline attendance** is queued on the device and synced when back online (no duplicates).
- **App updates:** the app checks a minimum-version policy on launch. If a release contains a change that older apps can't handle, users on an old version see a **"Update required"** screen instead of hitting errors; a less critical update shows a gentle "update available" nudge. Web/installed-PWA users always get the latest automatically.

---

*Questions or corrections? This manual reflects Phase 1 as of the date above. Feature menus you don't see are simply ones your role doesn't include.*
