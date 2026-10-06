---
source: office Mac ~/code/dsa-app/docs/DreamSpace-Academy-User-Guide.docx
---




Platform User Guide
Makerspace Module — Hatton & Batticaloa
A complete guide for Super Admins, Makerspace Admins, Hub Coordinators and Trainers
Version 1.0  •  June 2026
Co-Creating the Dreams

1. About this guide
DreamSpace Academy is a platform for running makerspace hubs for young people. This guide explains how staff use the web app (app.dreamspace.academy) and the mobile app to manage roles, users, students, sessions, payments and communications. It is written for the people who operate the platform day to day.
A note on privacy: students are children, so the operational screens (trainer views, attendance, payments) identify students only by their DSA ID and a friendly avatar — never by name or photo. Names appear only in the enrolment record, which is restricted to coordinators and admins.
Who's who — the built-in roles

2. Signing in
Staff accounts use email + password or phone + OTP. Parents typically use phone + OTP.
First-time set-up (email accounts)
An admin or coordinator creates your account and sends you a personal set-password link.
Open the link on your own device. Choose a password (at least 8 characters) and confirm it.
You are signed in and taken to your dashboard.
Note  A set-password / reset link is single-use and expires after about an hour. If it stops working, ask an admin to send a fresh one and open it only once. Don't reuse an old link.
Phone accounts
Enter your registered phone number, then the one-time code (OTP) we text you. No password needed.
Forgot your password
On the sign-in screen choose “Forgot password?”, enter your email, and open the reset link we send you to choose a new password.
Note  If you hold more than one role (e.g. Coordinator and Trainer), after signing in you'll pick which role to use for this session.
3. Roles & permissions
A role is a named bundle of permissions. A permission is a single capability written as “resource:action”, for example attendance:mark (mark attendance) or payment:waive (waive a fee). When you assign a role to a user, they gain that role's permissions.
How permissions are decided
For any action, the platform checks three layers in order:
Deny override — a per-user DENY always wins (blocks the action even if a role would allow it).
Grant override — a per-user GRANT adds a single capability on top of someone's role.
Role baseline — otherwise, the combined permissions of the user's role(s) apply.
Wildcards make roles compact:
“open_session:*” means every action on open sessions (view, manage, register).
“*” means absolutely everything — reserved for Super Admin only.
Platform scope vs hub scope
Platform roles apply everywhere, across all hubs (e.g. Makerspace Admin).
Hub roles are tied to one or more specific hubs; the holder only sees and acts within those hubs (e.g. Hub Coordinator, Trainer). When you assign a hub-scoped role you pick the hub(s).
Creating a custom role (Super Admin)
Custom roles let you fit the platform to your team — for example a “Registrar” who only enrols students, or a “Finance Officer” who only handles payments. Go to Roles → New role.
Key — an internal code in UPPER_SNAKE_CASE (3–40 letters/numbers/underscores), e.g. REGISTRAR. It can't be changed later, so choose carefully. (SUPER_ADMIN is reserved.)
Label — the friendly name shown in the app, e.g. “Registrar”.
Icon — pick one of the supplied icons for the role's badge.
Start from (optional) — copy the permissions of an existing role as a starting point, then adjust.
Scope — choose Platform (all hubs) or Hub (assigned to specific hubs).
Permissions — tick the capabilities. Each resource group has an “All actions” switch (the wildcard, e.g. student:*). Turn it on for full control of that area, or leave it off and tick only the specific actions you want.
Save. The role is available immediately for assignment.
Note  A role with no permissions ticked produces an empty app for its holders. Built-in roles can be re-labelled and their permissions adjusted, but their key and scope are locked. You can Disable a role (keeps it but stops new assignments) or Delete a custom role (removes it from everyone who has it).
Permission reference — what each feature does
Use this when building a role. Tip: ticking a resource's “All actions” switch grants every row in that group.

Suggested role recipes
Practical custom roles you can build. Always include section:view and notification:view so the app and alerts work. Prefer specific actions over wildcards for limited roles, and grant studentname:view only when the job truly needs names.

Note  Need an exception for one person rather than a whole role? A Super Admin can add a per-user GRANT (extra capability) or DENY (block) on the user's page — every override requires a reason and is recorded in the audit log.
4. Creating & managing users (staff)
Go to Users to invite staff and manage their access. Needs user:create / user:edit; assigning roles needs role:assign.
Create a user
Click New user. Enter the full name.
Choose Email or Phone. For email, enter the address; for phone, use the Sri Lanka (+94) number.
Save. For email accounts the app generates a copyable set-password link — send it to the person (e.g. via WhatsApp) so they set their own password on their device. Phone accounts receive a welcome SMS and sign in with OTP.
Give them access (assign roles)
Open the user. Under Roles, pick a role.
If it's a hub-scoped role (Coordinator, Trainer), choose the hub(s) it applies to.
Assign. Repeat to give someone more than one role. Use Revoke to remove a role.
Note  A brand-new user has no roles yet, so the app looks empty for them until you assign one. Super Admin can't be assigned from the app, and you can't deactivate or delete a Super Admin here. Deactivating a user signs them out and blocks sign-in; reactivate to restore.
5. Enrolling & managing students
Go to Students (needs student:create to enrol). Each student gets a unique DSA ID like DSA-047313 — a global number that reveals nothing about hub or year, used everywhere instead of names.
Enrol a student
Click Enrol / New student.
Enter the full name and optional preferred name and date of birth. (Names stay hidden in trainer/attendance/payment views.)
Set the level and assign a timetable slot — this becomes the student's default session roster.
Set the Agreed monthly fee if it differs from the hub's default. Tick Enrolment fee paid if collected up front.
Optionally record demographics (gender, district, income) — these are sensitive and admin-only.
Link a parent/guardian by phone so they can use the app and receive notifications.
Capture PDPA consent (data processing, photo, marketing, impact reporting) and save.
Privacy & avatars
Operational screens show the DSA ID and an avatar only — never names or photos.
Trainers can choose a friendly animal avatar; otherwise a unique robot avatar is generated from the DSA ID.
Sponsorship
If a donor sponsors a student's full fee, the student's monthly fees are marked SPONSORED instead of UNPAID and are not collected.
6. Timetable & sessions
There are four kinds of session. The first three are for enrolled students; open sessions (Section 7) are public workshops.

The weekly timetable
Go to Timetable (needs timetable:manage to edit).
Add a Slot: choose the day, start/end time, level, default trainer and optional capacity.
The system automatically generates the actual session instances every week, two weeks ahead, and skips public holidays.
Add public holidays with “+ Holiday” so no sessions are created on those dates.
Note  Instances generate automatically, but you can also use Generate sessions to create them on demand (e.g. during initial set-up). It never creates duplicates.
One-off and special sessions
One-off — Create one for a makeup or extra class on a single date, then add the students who should attend.
Special (invite-only) — Create one, pick the students, and choose whether to also notify parents by SMS/email (in-app + push are always sent). Each child's parent receives a join request; the child is added to the session only when the parent accepts. Students without a linked parent are listed so you can add them manually.
Cover trainer & editing
If the assigned trainer is away, assign a cover (substitute) trainer to that specific session.
On an existing session instance you can edit the topic/notes; to change the time or the recurring pattern, edit the slot (this affects future instances, not ones already created).
Attendance
Trainers mark attendance per session (needs attendance:mark).
Open the session and mark each student Present, Absent, Late or Excused. Add an optional note (visible to the parent).
Submit & lock. The session is then locked and marked complete.
After locking, only a coordinator with attendance:override can correct a record, and a reason is required (this is recorded).
Note  Trainers can mark attendance offline; it's stored on the device and synced when back online without creating duplicates.
7. Open sessions (public & subscriber workshops)
Open sessions are workshops or events that people register for — they are not tied to enrolled students. They support categories, tags, seat limits with a waitlist, and optional paid tickets. Manage them under Open Sessions (needs open_session:manage).
Special vs open — what's the difference?

Create & publish an open session
New open session. Add a title, description, date/time, and location (or mark it online).
Pick a category and tags, and a seat limit if you want a cap (extra sign-ups go to a waitlist).
Choose the access type: OPEN (anyone, straight away), SUBSCRIBER_EARLY (subscribers register first, public later), or SUBSCRIBER_ONLY (subscribers only). Set the open/close dates if you use windows.
If it's paid, turn on pricing and set the public price (and optional subscriber/member prices). A paid session needs at least a public price before it can be published.
Choose audience options: Notify our students/parents (alerts parents of active students when published) and List publicly (shows it on the public website).
Save as draft, then Publish when ready. Use Cancel (with a reason) to call it off — registrants are notified.
How people register
From the app, a parent or member taps Register. If the session is full they join the waitlist; if someone cancels, the next person on the waitlist is promoted automatically and notified.
Staff can add a walk-in (someone without an account) by entering their name/contact.
For paid sessions, staff mark a registration paid (cash, bank transfer or online) and can mark attendance on the day.
8. Announcements
Send a message to groups of people in your hub (needs announcement:send). Go to Announcements.
Write a title and message.
Choose the target roles — e.g. Parents (this reaches the parents of your active students), Trainers, etc.
Pick extra channels: SMS and/or Email (in-app and push are always sent).
Send. You'll see how many recipients it reached.
Note  To prevent spam, announcement SMS is limited to one per recipient per day — anyone who already received an announcement SMS that day still gets the in-app and push message, and the screen tells you how many SMS were skipped.
9. Payments & receipts
Fees are tracked per student per month. Go to Payments (needs payment:view; collecting needs payment:mark).
How monthly fees are created
On the 1st of each month the system automatically creates a fee record for every active student.
The amount is the student's Agreed monthly fee, or the hub's default fee if none is set.
Students with a full-fee sponsorship are marked SPONSORED and aren't collected.
Payment statuses

Collect, waive, export
Find the fee in the list (shown by DSA ID). Click Mark paid.
Choose the method (cash, bank transfer, online) and optional reference. A receipt number is generated automatically.
To forgive a fee, use Waive and enter a reason (recorded).
Use the month/status filters and Export CSV for reporting; the summary shows expected vs collected.
Note  Reversing a paid fee (payment:reverse) is restricted — usually to admins/senior finance — and every change is recorded in the audit log.
10. Fee reminders
Send reminders for outstanding fees (needs payment:remind).
In Payments, filter to UNPAID / OVERDUE and select the fees to chase.
Click Send reminder and choose SMS and/or Email (in-app and push are always sent).
Reminders go to the student's linked parents/guardians. Any student without a linked parent is listed so you can follow up.
Note  Fee reminders are transactional, so unlike announcements they are not capped at one SMS per day. Add a parent link to a student to make sure reminders reach someone.
11. How notifications reach people

12. Appendix — built-in roles at a glance


DreamSpace Academy — Co-Creating the Dreams.