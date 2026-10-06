# Clockify — End-to-End Documentation (Admin + Employee)

*Documented live from a Clockify free trial workspace ("Timer Docs Test Org") on 2026-09-10. All screenshots are real captures from the trial, not marketing material.*

---

## 1. Overview

**What Clockify is:** a time-tracking and workforce-management web app (by CAKE.com). Core loop: users track time (via a running timer or manual entry) against Projects/Tasks, submit it as a Timesheet, a manager approves it, and Admins turn approved time into Reports and Invoices.

**Pricing tiers** (as shown in-app during trial signup — brief, not exhaustive; verify current prices on clockify.me before quoting to anyone):
- **Free** — basic time tracking, unlimited users/projects, what the workspace falls back to after trial.
- **Pro (trial tier)** — adds: Break tracking, Force timer, Labor cost & profit, Screenshots, Scheduling, Expenses, Invoicing, and more (full trial-perk list captured in section 7 screen inventory).
- Trial: 7 days, no credit card required, pre-loaded with sample data.

*(Status: in progress — this document is being built incrementally. Sections below will fill in as each screen is walked and captured.)*

---

## 2. Employee / team-member walkthrough

**Methodology note:** This section is now built from a genuinely separate second account — `thalaivan.ugam@gmail.com`, invited from the Admin's Team screen, accepted via email + one-time-code login, and driven live as its own browser session with the default **Member** role (no Team manager / Project manager / Admin boxes checked). Every claim below marked **(directly observed)** was clicked through on that real Member session. A small number of items remain **(inferred)** where re-establishing the second session wasn't repeated after a required Admin round-trip; those are called out explicitly.

*Technical note on how this was captured:* Clockify's session cookie is shared per Chrome profile — logging into the Admin account in a new tab silently logs the Employee out of every tab in that browser (and vice versa), so the two roles had to be captured in separate passes, re-logging in each time the other role's screen was needed. Worth flagging to a competing team as a UX/support-burden consideration if the internal tool ever needs simultaneous multi-role testing.

### 2.0 Onboarding as an invitee — directly observed
- **Entry point:** Admin sends an invite from the Team screen (section 3.3) → invitee gets a Clockify invite email → accept link → email + one-time-code login (this invitee did not use Google OAuth, unlike the Admin account).
- **Create Account screen (directly observed):** name field + "I agree to CAKE.com Terms of Use" checkbox + Continue. **Key confirmed difference from the Admin's own Create Account screen: there is no "Organization" field** — the invitee is joining an existing workspace, not creating one. See `01-invite-create-account.jpg`.
- After Continue, lands directly in that workspace's Time Tracker screen — no separate "customize your experience" survey modal was shown to the invited Member (that modal only appeared for the Admin, who was creating a brand-new workspace).

### 2.1 What the Member sidebar nav shows — directly observed

The real Member sidebar, top to bottom, is exactly: **TIME TRACKER, CALENDAR, SCHEDULE, EXPENSES, TIME OFF** — then an **ANALYZE** group (**DASHBOARD, REPORTS**) — then a **MANAGE** group (**PROJECTS, TEAM**). See `02-tracker-member-view.jpg`.

Confirmed **absent** from the Member sidebar, directly observed by comparison against the Admin's own nav (`06-team-member-view.jpg` shows the same account list Admin sees, but the nav itself lacks the extra items): **APPROVALS, INVOICES, KIOSKS, CLIENTS, TAGS**, and there is no "..." → Workspace settings menu next to the workspace name. This confirms the earlier reasoning from the Permissions matrix was directionally correct, with one addition worth flagging: **PROJECTS and TEAM are both visible to a plain Member in this workspace** — not just "likely visible," but confirmed live, consistent with this workspace's "Who can create Projects/Clients/Tasks → Anyone" permissive trial default (3.11). A Member here can open Projects and see budgets/progress bars and even "CREATE NEW PROJECT" (`03-projects-member-view.jpg`), and can open Team and see the full roster with roles, though (see 2.1.1) not add/invite members.

**2.1.1 Project detail as a Member — directly observed:** Opening a project as a Member shows tabs **TASKS / ACCESS / STATUS / NOTE** only (`04-project-detail-member-view.jpg`) — the **SETTINGS** tab (budget, rate, billable-by-default, recurring reset — see 3.5) that the Admin sees on the same screen is **not present** for a Member. This is a real, confirmed role difference: a Member can see and add tasks but cannot touch budget/rate configuration.

**2.1.2 Team screen as a Member — directly observed:** The full member list, with each person's Role badge, Group, and Assigned team manager, is visible (`05-team-member-view.jpg`) — including a lingering pending-invite row for an earlier unused invite (`vishnu88varthan+employee@gmail.com`). There is **no "ADD FULL MEMBER" button** and no role-badge click-to-edit for a Member — read-only roster access only.

**2.1.3 Direct-URL no-access test — directly observed, real edge case:** Typing `https://app.clockify.me/approvals` directly into the address bar as a Member does **not** show a permission-denied page — it silently redirects to `/tracker` (`10-approvals-url-redirect-noaccess.jpg`). This is a genuine, directly-tested no-permission behavior worth noting for the internal tool: Clockify chose a silent redirect over an explicit "you don't have access" message.

### 2.2 Time Tracker (the Employee's real workflow) — directly observed
The Tracker screen is the same underlying UI as the Admin's (section 3.10): idle "What are you working on?" row, Project picker, START/STOP, live running-timer state, "This week" list with per-entry inline edit. A live test entry ("Employee test entry - documentation") was started, run for ~16 seconds, and stopped — it saved correctly with a green "Time entry has been created" toast (`06-timer-running.jpg`, `07-weekview-entry-saved.jpg`), no repeat of the Admin-side STOP-button bug was seen on this run.

### 2.3 Manual time entry — directly observed
Same surface as 3.10/2.2: click into any entry's description or time fields inline. Confirmed additionally in 2.4 below: a **rejected** entry stays inline-editable with no separate "unlock" step.

### 2.4 Submitting for approval, and the full reject → edit → resubmit loop — directly observed end to end

This is the one full round-trip of the approval lifecycle traced live across both accounts:

1. **Submit (Member):** Clicking "Submit for approval" opens a confirmation modal — "Ready to submit **This week** for approval? Time: 00:00:16h" — SUBMIT (`08-submit-for-approval-modal.jpg`). Confirming moves the week to **Pending**, with a "Withdraw pending" link replacing "Submit for approval," and a small orange lock/clock icon appears on the entry row (`09-pending-status.jpg`).
2. **Reject (Admin):** The entry appears immediately in the Admin's Approvals → Pending queue under "Thalaivan Ugam" (`45-approvals-pending-real-employee-entry.jpg`). Opening it shows the same Timesheet detail screen documented in 3.4 (`46-timesheet-detail-real-employee.jpg`). Clicking REJECT opens the "Reject timesheet" modal with a "Send note to user" textarea (`47-reject-modal-real.jpg`); a note was typed ("Please add more detail to the description before resubmitting.") and REJECT confirmed. The entry disappears from Pending and appears in **Archive** tagged **Rejected** (`48-approvals-pending-after-reject.jpg`, `49-archive-rejected-real.jpg`).
3. **Rejected view (Member):** Back on the Member's own Tracker, the week now shows a **"Rejected"** badge in place of "Pending" (`11-rejected-status-view.jpg`), and the individual entry row shows a red circled-X icon whose tooltip reads simply "Rejected" (`12-rejected-tooltip.jpg`). **Confirmed real gap:** the manager's rejection note is **not surfaced anywhere in the Member's UI** that was checked — not on the entry row, not in a popover on the red icon, and not in the notification bell (which only showed generic Pumble/email-marketing prompts, `09`–`12` series). If the note reaches the employee at all outside this UI, it would have to be by email — not confirmed, since the invite inbox wasn't checked for a rejection email in this pass.
4. **Edit (Member):** The rejected entry's description field is directly clickable and editable with no unlock action needed (`13-rejected-entry-editable.jpg`) — edited to append "(updated after reject)" and saved with Enter (`14-edited-after-reject.jpg`).
5. **Resubmit (Member):** Clicking "Submit for approval" again on a previously-rejected week shows a **different modal than the first-time submit** — it adds an explicit warning: *"You're about to submit time entries that were previously rejected. Please make sure you adjusted the changes accordingly before submitting for approval."* (`15-resubmit-modal-warning.jpg`). Confirming returns the week to **Pending** again (`16-resubmitted-pending-again.jpg`), closing the loop back to step 2's starting state.

The **Approve** side of this same loop (Admin clicks APPROVE → confirmation modal → entry locks, uneditable) was directly observed earlier in section 3.4/3.10 on Admin's own entries; the mechanism is identical regardless of whose entry it is, so it was not repeated a second time on this specific entry to avoid a third round of the cross-tab relogin described in the methodology note above.

### 2.5 Viewing reports — directly observed structure, inferred scope
Same Reports screen as 3.7 (Summary/Detailed/Weekly/Shared tabs, filter bar, bar chart) is present in the Member's nav (2.1). **(inferred, not re-verified this pass)** that a plain Member's Team filter and visible rows are restricted to their own tracked time — the Permissions matrix's "Who can see hourly rates and amounts → Admins" row implies a Member's report views likely hide billable-rate/amount columns unless "Anyone" is chosen.

### 2.6 Settings — inferred
An Employee reaches personal settings (profile, password, notification preferences, timezone) via the same top-right avatar menu seen throughout this document, not the workspace-wide "..." → Workspace settings menu (which, per 2.1, does not exist for a Member at all). Not opened live in this pass.

---

## 3. Admin / manager walkthrough

### 3.0 Onboarding (account creation → first dashboard)

**Screen: Login** (`app.clockify.me/login`)
- Fields/buttons: Continue with Google, Continue with Microsoft, Continue with Apple, email input + "Continue with email", "Sign up" link, language selector (bottom).
- Action: signed in via Google OAuth.

**Screen: Create account** (`app.clockify.me/memberships?...`)
- Fields: "Your name" (pre-filled from Google profile), "Organization" (text, required), "I agree to CAKE.com Terms of Use" (checkbox, required), "Continue" button (disabled/no-op until above are valid — did not test disabled state explicitly).
- "Log in with different account" link at bottom.
- Interaction: filled Organization = "Timer Docs Test Org", checked terms, clicked Continue → workspace created → redirected to `/tracker`.

**Screen: "Help us customize your Clockify experience"** (modal, over Tracker page)
- Fields (all required — confirmed via validation):
  - "I'm here to" — dropdown: Bill clients accurately / Keep projects profitable and on budget / Manage team payroll / Monitor employee screens or location / Plan projects and track progress.
  - "with my team of" — dropdown: Just me / 2-10 / 11-50 / 51-100 / 101-250 (likely more below fold).
  - "and our primary billing model is" — dropdown: Hourly billing / Fixed project fees / Retainer agreements / Mixed/hybrid billing / Don't bill clients.
  - Button: "START USING CLOCKIFY".
- **Error state observed:** clicking Start with any field empty → inline tooltip "Complete all fields above." (button stays active, no page change).
- Chose: Bill clients accurately / 2-10 / Hourly billing → clicked Start → modal closed.

**Screen: "Your free trial starts today!"** (modal)
- Copy: "Explore Pro features for free for 7 days. No credit card required. When your trial ends, stay on Clockify's Free plan or upgrade."
- Lists Pro-only perks unlocked during trial: Break, Force timer, Labor cost & profit, Screenshots, Scheduling, Expenses, Invoicing (list continues below fold — will confirm full list in section 7).
- Button: "Let's go →".
- Interaction: clicked "Let's go" → lands on real Tracker dashboard (documented next).

### 3.1 Global navigation (sidebar)

Left sidebar, reached by clicking the `>>` expand tab at the left edge of any screen (collapsible on every screen). Full item list, top to bottom:
- **TIME TRACKER** — the Tracker/timer screen (home).
- **CALENDAR** — calendar view of tracked time.
- **SCHEDULE** (expandable) — team scheduling (Pro feature).
- **EXPENSES** — expense tracking (Pro feature).
- **TIME OFF** (expandable) — PTO/leave management.
- *ANALYZE section:*
  - **DASHBOARD** — visual summary/analytics.
  - **REPORTS** (expandable) — Summary / Detailed / Weekly / Shared reports.
- *MANAGE section:*
  - **KIOSKS** — shared-device clock-in kiosk mode.
  - **APPROVALS** — timesheet approval queue (had an orange notification dot on first login).
  - **PROJECTS** — project/task list and settings.
  - **TEAM** — user/member management.
  - **CLIENTS** — client list (for billing).
  - **TAGS** — time-entry tag management.
  - **INVOICES** — invoice creation/list (continues below the fold).
- Top bar (persistent across all screens): workspace switcher ("Timer Docs Test Org" + "..." menu), puzzle-piece icon (integrations/marketplace), bell icon (notifications, badge "2"), "?" help icon, user avatar (account menu) — top right. Blue banner at very top: "N days left in trial - You are currently using sample data to help you explore." + "Manage" dropdown (trial/plan management).

*(continuing through each nav item...)*

### 3.2 Projects screen (`/projects`)

- Banner: "You are currently using sample data to help you explore." + "REMOVE SAMPLE DATA" button.
- Header: "Projects" + "CREATE NEW PROJECT" button (top right).
- Filter bar: Active (status filter), Client, Access, Billing dropdowns; "Find by name" search; "APPLY FILTER" button.
- Table columns: checkbox-select-all, Name (sortable, colored dot = project color), Client, Tracked (hours), Amount (USD; turns **red** when over budget), Progress (% bar; turns **red** and exceeds 100% when over budget), Access (Public/Private), star (favorite), "⋮" row menu.
- **Edge case found in sample data already:** "[SAMPLE] Proj..." for Client B shows Amount `3,163.19 USD` in red and Progress `790.80%` with a red bar — a project already over its 400.00 USD budget. This is documented further in section 6 (Edge cases).
- Row "⋮" menu (not yet opened — to be explored): expected Edit, Duplicate, Archive, Delete per Clockify convention.

### 3.3 Team screen (`/teams`)

- Tabs: **FULL** (full-price seats) / **LIMITED** (limited-access seats, e.g. Kiosk-only) / **GROUPS** / **REMINDERS**.
- "ADD FULL MEMBER" button (top right) → opens **"Add full members" modal**:
  - Field: "Invite by email" — multi-email input, "Separate multiple emails with commas, spaces, or semicolons."
  - Toggle: "Insert names" (lets you name invitees inline instead of just email).
  - Buttons: Cancel / SEND INVITE.
  - Typed email becomes a removable chip once you click elsewhere or press Enter/Tab.
- Filter bar: FILTER, All (status), Billable rate, Cost rate, Role, Group dropdowns; "Search by name or email"; APPLY FILTER.
- Table columns: Name, Email, Billable rate (USD, "Change" link inline), Cost rate (USD, "Change" link inline), Role (badge: Member / Team manager — clickable to change), Group ("+ Group" to assign), "⋮" row menu.
- **Action taken:** invited `vishnu88varthan+employee@gmail.com` as a Full member.
  - Result: success toast "Full users have been invited to workspace." New row appears immediately with an envelope icon instead of a name (pending-invite state), email shown, Role defaults to **Member**, rates blank ("Change" to set).
  - This is the pending-invite state — the row will show the person's real name only after they accept and log in.

### 3.4 Approvals screen (`/approvals`) — the approve/reject cycle

Top-level tabs: **APPROVALS** (org-wide approval settings, not yet opened) / **Timesheet** / **Expenses**.

Under **Timesheet**, three sub-tabs:
- **PENDING** — timesheets submitted by employees, awaiting a decision. Grouped by week. Columns: User, Team manager, Time, Time off. Row click → opens the Timesheet detail/approval screen. Top-right: "REMIND TO APPROVE" (nudges whoever is assigned to approve), "APPROVE ALL" (bulk-approves every visible row without opening each one — bypasses the per-timesheet detail view).
- **UNSUBMITTED** — team members who tracked time this period but have NOT submitted a timesheet yet. Date-range picker (defaults "This week", `<`/`>` to page weeks), Team filter, "REMIND TO SUBMIT" button, "Show users without time" toggle at bottom.
- **ARCHIVE** — historical record of every timesheet that has left Pending, with a **STATUS** column showing one of: **Approved**, **Rejected**, **Withdrawn** (this last one appeared in sample data — meaning the employee pulled the submission back before a manager acted; not yet reproduced live, will attempt in employee walkthrough).

**Screen: Timesheet detail** (`/approvals/details/{id}?userId={id}`)
- Breadcrumb: Approvals / Timesheet. Header: `{Employee name}, {week range}`. Subtext: "Submitted by: {name} (Today)".
- Buttons top-right: **REJECT**, **APPROVE**.
- Summary strip: Submitted (total hours), Break, Billable, Time off, Amount (USD), Cost (USD).
- Bar chart: hours per day across the week.
- Below the chart (not fully captured yet): per-project/task breakdown table with entries, likely editable-until-approved.

**Branch: REJECT**
1. Click REJECT → **"Reject timesheet" modal** opens.
2. Field: "Send note to user" (free-text textarea, optional — not marked required; rejected with a note successfully).
3. Buttons: Cancel / REJECT.
4. On confirm: timesheet **leaves Pending**, no confirmation dialog beyond the modal itself, redirects back to `/approvals#pending`.
5. Lands in **Archive** with status badge **Rejected**, name shown with strikethrough styling.
6. *(Employee-side continuation of this loop — what the employee sees, whether they can now edit and resubmit — to be captured in the Employee walkthrough section, since this session is the Admin account.)*

**Branch: APPROVE**
1. Click APPROVE → **"Confirm approval" modal**: "Once approved, timesheet will be locked and no one will be able to edit it anymore." Buttons: Cancel / APPROVE.
2. On confirm: timesheet leaves Pending, no toast observed, redirect to `/approvals#pending`.
3. Lands in Archive with status **Approved** and (per the modal's warning) becomes locked/read-only for the employee.

**Branch: APPROVE ALL** (bulk, from Pending list, not yet tested) — expected to approve every currently-listed pending timesheet in one action, skipping the per-item confirm modal; to be verified.

### 3.5 Project detail screen (`/projects/{id}/edit`)

Reached by clicking a project row's name on the Projects list. Tabs: **TASKS** / **ACCESS** / **STATUS** / **FORECAST** / **NOTE** / **SETTINGS**.

- **TASKS** tab: task list for this project. Columns: Name, Assignees (dropdown, default "Anyone"), Hourly rate (USD, "Change" link), Cost rate (USD, "Change" link), "⋮" row menu. "Add new Task" input + ADD button, "Show active" filter, search.
- **SETTINGS** tab (rates/budget — the section 3 "setting rates/budgets" ask):
  - Name (text), Client (dropdown), Color (swatch picker), "Billable by default" toggle.
  - **Project billable rate** — Hourly Rate (USD) input, "Set rate" link (rate is blank/"—" by default until set).
  - **Project cost rate** — Cost Rate (USD) input, "Set rate" link (this is the internal cost used for profit calculations, separate from the client-billed rate).
  - **Project estimate** (the budget config): dropdown to choose estimate type — seen: "Project budget" (others likely include "Project time" — not yet expanded). Under it:
    - Radio: **Manual** (flat amount input + currency, e.g. `400 USD`) vs **Task based** (sums per-task budgets from the Tasks tab).
    - Toggle: "Reset estimate every [month] on the [1st] at [00:00] (UTC offset)" — recurring budget reset, off by default; when on, exposes inline dropdowns for period/day/time.
    - Toggle: "Budget includes billable expenses" (on by default) — whether Expenses-module entries count toward the budget total.
  - **Additional fields**: "Add more fields..." dropdown — custom fields attached to every time entry on this project.
  - *(ACCESS/STATUS/FORECAST/NOTE tabs not yet explored in depth.)*
- **This is the exact config behind the over-budget sample project** ([SAMPLE] Project Beta / Client B): Manual budget = 400 USD, no reset, and actual billed amount is 3,163.19 USD → 790.80% of budget, which is why Projects list shows Amount and Progress in red. Confirms budget-overrun is a purely visual/reporting alert in Clockify — no evidence yet of it blocking new time entries (to verify by trying to track time against this project as Employee).

### 3.6 Invoices screen (`/invoices`)

- Header "Invoices" + "CREATE INVOICE" button.
- Filter bar: Issue date (range picker, default "All time"), Client, Status, Amount, Balance dropdowns; APPLY FILTER.
- Table: Invoice ID, Client, Issue date, Due on (shows "X days ago" in red if overdue), Amount, Balance, **Status badge** (`Sent` = blue, `Overdue` = red — sample data already includes one of each), "⋮" row menu, Export dropdown.
- Not yet tested: creating a real invoice (deferred — sending a real invoice is a destructive/irreversible-adjacent action per the rules; will only build a draft and stop before sending, per the "no real invoice" rule).

### 3.7 Reports screen (`/reports/...`)

- Sub-tabs: **Summary** / **Detailed** / **Weekly** / **Shared**.
- Filter bar: FILTER, Team, Client, Project, Task, Tag, Status, Description, more (scrolls off right edge) — very granular slicing. APPLY FILTER, EXPORT dropdown (top right).
- Summary strip: Total time, Billable time, Amount (USD).
- "Billability" dropdown above the chart (chart grouping toggle).
- Bar chart: time per project/client, billable portion shown with a `$` icon inside the bar.
- (Detailed/Weekly/Shared sub-views not yet opened — Detailed is expected to be a row-per-entry table; Shared is expected to generate a public read-only report link, which is a "publishing content" action requiring explicit permission before use.)

### 3.8 Dashboard screen (`/dashboard`)

- Filters: Project, scope ("Only me" — likely toggles to team-wide for admins), date range with `<`/`>` paging (default "This week").
- Stat tiles: Total time, Top Project, Top Client.
- Time chart (empty state seen: "No data to show — Try adjusting the filters to get some results", since no real tracking done yet this org).
- Right rail: "Most tracked activities" panel with a "Top 10" size selector.

### 3.9 Kiosks screen (`/kiosks`)

- Empty state: "Simplify time tracking with [Kiosks]" + "CREATE KIOSK" button.
- 3-step explainer cards: (1) Set up Kiosk — assign members + auth method (PIN toggle, QR code verification), (2) Clock In and Out — members clock in/out and log breaks on a shared tablet, (3) Track attendance — realtime attendance/break reporting.
- Purpose: a shared-device (tablet) kiosk mode for clocking in without individual logins — relevant for physical-location staff, distinct from the per-user web/desktop tracker.

### 3.10 Time Tracker screen — the timer flow, in depth

**Idle state row** (top of every Tracker page): description input ("What are you working on?"), "+ Project" picker, tag icon, "$" billable-toggle icon, `00:00:00` display, **START** button, "⋮" entry-options menu.

**Project picker** (click "+ Project" or the project chip): dropdown grouped by Client → Project → (expand) Task, with per-project favorite star, a search box ("Search Project or Client"), and "+ Create new Project" inline at the bottom. Picking a project colors its dot next to the name in the input row; picking one of its Tasks appends `: {task name}` to the project label. If the chosen project is "Billable by default", the "$" icon auto-lights blue — confirms billable status is inherited from the project setting, not asked per entry.

**Branch: START → running timer**
1. Click START. Button turns red **STOP**, main timer display starts counting up live, the browser tab title itself updates to show elapsed time (e.g. "4 sec • Clockify", later "39 sec • Clockify") — useful for anyone glancing at their tab bar. The left-sidebar "TIME TRACKER" nav item is replaced by a live-counting timestamp for the duration of the run.
2. **Edge case observed live:** while the timer was running, two unrelated-looking error toasts appeared and cleared on their own — `TIMEENTRY with ID '{id}' not found` and later `Currently running time entry doesn't exist on workspace {id} for user null.` Reloading the page (`/tracker`) while these were showing confirmed the timer was still running normally underneath — these read as noisy background sync/websocket messages, not real failures; the timer itself never actually broke.
3. **Branch: STOP.** Click STOP. In this session, clicking STOP once produced **both** a red toast "Stop time entry failed. Please try again." and a green toast "Time entry has been created" **simultaneously**. Reloading `/tracker` afterward showed the entry HAD been created correctly (00:00:44, 16:42–16:43) and the timer was no longer running — so the failure toast was a false negative; the actual stop/save succeeded despite the contradictory messaging. **This is a real, reproducible-looking UI polish bug worth flagging in section 6 (Edge cases).**

**Screen after first stop of the day: "This week" list** — replaces the empty "Let's start tracking!" placeholder once at least one entry exists this week.
- Header row: "This week" tab label, "Submit for approval" link (top-left) — the entry point into the submission flow (documented further once traced from the Employee side), running week total top-right (`00:00:44`).
- Day group header: "Today" (or date), day total, small icon (likely "add manual entry for this day").
- Entry row: description, project/task chip, tag icon, $ icon, start–end time range (`16:42 - 16:43`, click-to-edit expected), calendar icon, duration (`00:00:44`), a "▷" replay/restart-this-entry icon, "⋮" row menu (Edit/Duplicate/Delete per convention, not yet opened).

**Inline edit**: clicking directly on an existing entry's description or its start/end time range converts that row in place into editable text inputs (no separate modal) — description becomes a text field, `16:42` / `16:43` become independently editable time fields. Escape or click-away commits/cancels.

**Branch: "Submit for approval"** (link, top-left of the week view, only appears once ≥1 entry exists that week)
1. Click it → **"Submit for approval" modal**: "Ready to submit **This week** for approval? Time: 00:00:44h". Buttons: Cancel / SUBMIT.
2. On SUBMIT: the "Submit for approval" link is replaced by a **"Pending"** badge + a new **"Withdraw pending"** link. Each entry in that week gains a small pending/clock-check icon in its row and effectively becomes read-only-looking in place (no error was shown when re-clicking it, but this matches the pattern of the Admin-side lock-on-approve behavior — to verify precisely from the Employee side whether editing is blocked before actual approval or only after).
3. This is the exact moment a timesheet enters the Admin's Approvals → Pending queue (section 3.4).

**Branch: "Withdraw pending"** (the loop back)
1. Click it → **"Withdraw pending" modal**: "For **This week**, all pending entries will be withdrawn and won't be available for approval unless they are resubmitted. Time: 00:00:44h". Buttons: Cancel / WITHDRAW.
2. On WITHDRAW: entries return to normal editable state, the "Pending" badge and "Withdraw pending" link disappear, "Submit for approval" link returns. This is what produced the **"Withdrawn"** status seen earlier in the Admin's Approvals → Archive tab (section 3.4) for a different sample user — confirms Withdrawn = employee pulled back a pending submission before the manager acted on it, and it can be freely resubmitted afterward.

**Full submission loop, stated as a cycle:** Employee tracks time → clicks "Submit for approval" → status = Pending (locked from further edits, visible in Admin's Approvals/Pending) → one of three admin actions: **Approve** (→ Archive: Approved, permanently locked per the confirm-modal's own warning) / **Reject** with optional note (→ Archive: Rejected, employee presumably can edit & resubmit — to verify from Employee session) / no action yet. Independently, the **employee** can short-circuit this at the Pending stage via **Withdraw pending** (→ back to editable, no Archive entry created client-side that I've seen — though the earlier sample data DID show a "Withdrawn" archive entry, so it may still leave a trace) before a manager acts.

*(Manual/bulk entry via the timesheet-mode toggle ("Enable timesheet mode" banner seen throughout) still to be explored — will do so from the Employee account per the original plan, since that's the more natural place a regular team member enters time, along with confirming exactly what an employee can/cannot edit once Pending vs once Approved.)*

### 3.11 Workspace settings — the role/permission matrix (`/workspaces/{id}/settings`)

Reached via "..." menu next to workspace name → Workspace settings. Tabs: GENERAL / **PERMISSIONS** / ALERTS / ACCOUNTS / AUTHENTICATION / CUSTOM FIELDS / INTEGRATIONS / ADD-ONS / IMPORT.

**GENERAL**: Company logo upload, Workspace name, **"Activate timesheet"** toggle (org-wide switch to weekly-timesheet-entry mode — "While activated, Project is a required field for the whole workspace"), **Kiosk** allow-toggle, (more below fold — Currency, Week start day, etc. likely follow, not yet captured).

**PERMISSIONS** — this is the authoritative source for every role-based branch asked for in section 4. Each row is a radio/checkbox choice of **who** may do that action:
- Billable hours: "Activate billable hours" toggle. "Who can see hourly rates and amounts" → **Admins** (selected) / Anyone. "Who can see and change billable status of time entries and expenses" → Admins / **Anyone** (selected).
- "Who can create Projects and Clients" → Admins / Admins and Project managers / **Anyone** (selected).
- "Who can create Tasks" → Admins / Admins and Project managers / **Anyone** (selected).
- *(a row in between, partially captured: checkboxes for what a non-admin can access — "Projects / Team / Reports" — likely "what can Anyone see" scoping, to revisit.)*
- **"Who can approve submitted timesheets"** → **Admins and team managers** (selected) / Admins and project managers.
- **"Who can approve submitted expenses"** → **Admins and team managers** (selected) / Admins and project managers.
- **"Who can edit time and expenses for others"** → **Admins** (selected) / Admins and team managers.
- **"Who can manage invoices"** → **Admins** (checked) / Specific members.
- **"Lock time and expenses"** — "Prevent regular users from adding or editing time and expenses for past dates." Toggle "Lock time before [date, default Today]" (off) + "Automatically update lock date" (off, auto-advances daily/weekly/monthly when on). **This is the mechanism directly relevant to the "late timesheet submission" edge case in section 6** — with a lock date set, a regular Member literally cannot add/edit past-dated time; it's not merely a reminder.
- "Who can create assignments" (Schedule feature) — cut off at bottom, to revisit.

**Confirmed role definitions** — straight from Clockify's own "User role" modal (Team screen → click a member's role badge). Roles are checkboxes, not exclusive radios — a person can hold more than one simultaneously:
- **Member** — "Can track time on assigned projects. Default role for all users." (checked by default for every new invite, including our test employee.)
- **Admin** — "Can see and edit everything. **Only an Owner can remove an Admin role.**" (a one-way-feeling elevation: any Admin can grant Admin, but demoting one back requires the workspace Owner specifically.)
- **Project manager** — "Can edit all projects they manage, and see and approve time entries on those projects."
- **Team manager** — "Can see time entries of users they manage and approve their timesheets."
- **Owner**: not in this checkbox list — it's a separate, singular, implicit role (whoever created the workspace / is billed for it), sits above Admin, and is the only one who can revoke another Admin's Admin role.

This means "Team manager" and "Project manager" are additive permission grants layered on top of Member, not separate tiers — e.g. a person can be Member + Team manager simultaneously (exactly what "Lara Peterson" is in the sample data: shows as Member with elevated approve-rights, badge reads "Team manager" as a role tag).

### 3.4.1 Approve confirmation, after a real resubmit — directly observed

Once the real employee (2.4) resubmitted the rejected week, the Admin's Approvals → Pending queue showed it again; opening APPROVE surfaced the same "Confirm approval" modal warning that entries become locked and uneditable once approved (`50-confirm-approval-real.jpg`). Confirmed after the fact: the approved entry shows a **green checkmark icon** in Reports → Detailed (3.7.1) marking it locked, matching the modal's own warning text.

### 3.7.1 Reports sub-tabs, in depth — directly observed

Opening the REPORTS item in the sidebar (rather than just the Summary landing page) exposes the full dropdown menu (`51-reports-menu-full.jpg`):
- **TIME REPORT** section: **Summary** / **Detailed** / **Weekly** / **Shared**.
- **TEAM** section: **Attendance**, **Assignments**.
- **EXPENSE** section: **Detailed**.

- **Detailed** (`52-reports-detailed-approved-entry.jpg`): row-per-entry table; the one real entry that was approved end-to-end in 2.4/3.4.1 shows a small **green checkmark** icon next to it, confirming visually that approved = locked, distinct from a plain unapproved row.
- **Weekly** (`53-reports-weekly.jpg`): a per-day-of-week grid/summary view, one row per person or project (grouping configurable via the same filter bar as Summary).
- **Shared** (`54-reports-shared-empty.jpg`): empty state on this workspace — no shared report links have been created yet; presumably a "Create shared report" action exists but wasn't exercised (publishing a public link was treated as out of scope, consistent with the plan noted in 3.7).

### 3.9.1 Kiosk creation — directly observed

- **CREATE KIOSK** button opens the **Create Kiosk modal** (`56-create-kiosk-modal.jpg`). Fields: Name (text), Assignees (multi-select of members, including a special **"Everyone (including new users)"** option), Default Project (dropdown), Default break Project (dropdown), "Kiosk logs out after [X] hours" (numeric setting), and an **Authentication required** checkbox (gates the kiosk behind a PIN/QR per the empty-state explainer in 3.9).
- **Empty-field validation, directly tested:** clicking CREATE with required fields left empty produces **no error toast at all** — the CREATE button is simply disabled/inert until required fields are filled (`57-create-kiosk-empty-validation.jpg`). This silent-disabled-button pattern matches the same convention seen elsewhere in the product (e.g. the onboarding Continue button in 3.0).
- **Successful creation** (`58-kiosk-created.jpg`): the new kiosk appears in what was previously the empty-state list, replacing the "CREATE KIOSK" empty screen with a real kiosk row.

### 3.6.1 Invoices, in depth — directly observed

- **Invoices list** (`59-invoices-list.jpg`): confirms the three status badges described in 3.6 — **Sent** (blue), **Overdue** (red), and **Unsent** (a fourth, draft-like state not previously captured).
- **Existing sample invoice detail** (`60-invoice-detail-sample.jpg`): opening a pre-seeded sample invoice shows **DOWNLOAD**, **SEND**, **RESEND** actions, a **RECORD PAYMENT** action, Recurring-invoice settings, and an **Actions** dropdown for further options. **Bill from** and **Bill to** fields each carry an **ADD ADDRESS** link for filling in billing-party details inline.
- **Create Invoice modal** (`61-create-invoice-modal.jpg`): Client (dropdown), Currency (dropdown), an auto-suggested Invoice ID (pre-filled, presumably sequential), Issue date and Due date (date pickers).
- **Result** (`62-invoice-draft-created.jpg`): a new draft invoice is created and lands in the Unsent state — deliberately stopped here without clicking SEND, consistent with the "no real invoice sent" scoping rule from 3.6.

### 3.13 Integrations — two distinct surfaces, directly observed

Clockify actually exposes **two separate "integrations" entry points**, worth distinguishing clearly for a competing team:

1. **Top-nav puzzle-piece icon** (`63-integrations-popover.jpg`) — a small "Browse Marketplace" popover that opens the **CAKE.com Marketplace** modal (`64-cake-marketplace-integrations.jpg`). This is a **cross-product add-on store** shared across CAKE.com's whole suite of apps, not something Clockify-specific — it surfaces add-ons that may apply beyond just Clockify.
2. **Workspace Settings → INTEGRATIONS tab** (`70-ws-settings-integrations-list.jpg`) — a curated, Clockify-specific list of first-class integrations: **QuickBooks, Jira, Google Calendar, Outlook Calendar**, plus an **"Other integrations"** link pointing to 58+ additional integrations. This is the surface an Admin would actually use to connect a specific external tool to Clockify itself, as opposed to browsing the broader CAKE.com add-on marketplace.

### 3.14 Workspace settings — remaining tabs, directly observed

Continuing the tab sweep started in 3.11 (`65-workspace-settings-tabs-full.jpg` shows the full bar: GENERAL, PERMISSIONS, ALERTS, ACCOUNTS, AUTHENTICATION, CUSTOM FIELDS, INTEGRATIONS, ADD-ONS, IMPORT):

- **ALERTS** (`66-ws-settings-alerts.jpg`): workspace-level notification/alert configuration (e.g. over-budget or over-time alerts).
- **ACCOUNTS** (`67-ws-settings-accounts-enterprise.jpg`): gated behind an **Enterprise** plan — visible but not testable on this trial workspace; noted as **paid-tier only, not tested**.
- **AUTHENTICATION** (`68-ws-settings-authentication.jpg`): mixed gating on one screen — the **data region** setting is gated behind **Pro**, while the **SSO subdomain** setting is gated behind **Enterprise**. Both gated controls are noted as **paid-tier only, not tested**; the rest of the tab (whatever isn't gated) was visible but not exercised.
- **CUSTOM FIELDS** (`69-ws-settings-custom-fields.jpg`): a drag-and-drop builder for defining custom fields (per 3.5's "Additional fields" reference on the Project settings tab) and toggling each one visible/invisible.
- **INTEGRATIONS**: see 3.13 above (`70-ws-settings-integrations-list.jpg`).
- **ADD-ONS** (`71-ws-settings-addons-empty.jpg`): empty on this workspace, with a link out to the Marketplace (the same CAKE.com Marketplace modal from 3.13).
- **IMPORT** (`72-ws-settings-import.jpg`): CSV upload for bulk-importing data, **10MB max file size**, with downloadable CSV templates to match the expected format.

### 3.12.1 Schedule — Team tab, directly observed

Extending 3.12's Schedule sweep: the **Team** tab (`73-schedule-team-tab.jpg`) is a per-member capacity grid — one row per team member instead of per project — with **color-coded indicators** for over-hours (overallocated) vs. open-hours (available capacity) cells, complementing the Projects tab's allocation-by-project view.

### 3.12.2 Time Off — full real cycle, directly observed end to end

This traces a complete Time Off request from a genuinely zero-balance starting state through to Admin approval:

1. **Requests tab** (`74-timeoff-requests-list.jpg`) and **Policies tab** (`75-timeoff-policies-defaults.jpg`): this workspace, despite the "No policies yet" empty state described in 3.12, actually ships with **two auto-created default policies already assigned**: **Vacation** (20 days/year accrual) and **Sick leave** (8 days/year accrual), both auto-assigned to all 8 workspace members.
2. **Zero balance by default — key finding:** the Balance tab (`77-timeoff-vacation-zero-balance.jpg`) shows **0.00d available** for both policies for every member, despite the accrual rates configured on the policy. **The accrual rate alone does not credit any balance — balance starts at zero and must be manually credited (or accrued over real time) before anyone can actually take time off.**
3. **Insufficient-balance validation, directly tested:** attempting to submit a time-off request against the zero balance produces a real, visible inline validation error: **"You don't have enough time off allocated for the selected period"** (`76-timeoff-request-insufficient-balance.jpg`) — this is a hard block, not a soft warning; the request cannot be submitted.
4. **Balance table** (`78-timeoff-balance-table.jpg`): per-member **Accrued / Used / Available** columns, with an inline **"Add"** link per member row (`79-timeoff-balance-add-link.jpg`) for manually adjusting a member's balance.
5. **Manage balance modal** (`80-manage-balance-modal.jpg`), opened from that Add link: an **Add/Remove** toggle, a days stepper, **Available-from** and **Last-valid-date** date fields, and a free-text **Note** field. 3.00d was credited to the test member's Vacation balance, resulting in the balance table updating to reflect the credit (`81-balance-credited.jpg`).
6. **Request now submittable** (`82-timeoff-request-ready.jpg`): with a nonzero balance, the same Time Off request form that failed in step 3 now submits successfully; the Balance tab immediately reflects the reduction (`83-timeoff-balance-after-request.jpg`), and a new row appears in the Requests tab in a pending state (`84-timeoff-request-new-row.jpg`).
7. **Admin approval** (`85-timeoff-request-approved.jpg`): the Admin approves the request from the Requests tab, completing the cycle.

### 3.2.1 Project/Client creation — empty-name and duplicate-name behavior, directly tested

Two genuine, directly-tested validation edge cases surfaced here, and they diverge from each other in a way worth calling out explicitly to a competing team:

- **Project, empty name:** attempting to create a Project with a blank Name field is a **silent no-op** — no error toast, the action simply does nothing (`86-project-empty-name-noop.jpg`).
- **Project, duplicate name:** creating a Project using a name that already exists in the workspace is **silently allowed** — no uniqueness validation at all. A second, identically-named project appears in the Projects list right alongside the first (`87-project-duplicate-name-allowed.jpg`).
- **Client, duplicate name:** by contrast, creating a Client with a name that already exists is **blocked**, with a real, visible error toast: *"Client with name '[SAMPLE] Client A' already exists"* (`88-client-duplicate-name-blocked.jpg`). Client empty-name creation was also tested and, like Projects, is a silent no-op with no error shown.
- **Takeaway:** Projects and Clients are inconsistent with each other on duplicate-name handling — Projects allow silent duplicates while Clients explicitly block them with a user-facing error. Worth flagging as a real UX inconsistency for the team building the competing Timer tool to deliberately choose one behavior (and apply it consistently) rather than inherit this split.

### 3.12 Remaining nav items — quick sweep

- **Calendar** (`/calendar`): week-grid view, hour rows down the side, day columns across top with per-day total (`00:00:00`), a "Planned" row at top (for Schedule integration), zoom +/- controls. Time entries render as blocks on this grid (the one entry tracked earlier appeared as a thin blue line around 16:00–17:00).
- **Schedule** (`/scheduling`): Gantt-style forward planning tool. Tabs **PROJECTS** / **TEAM**. Left column: project name + client, "Assigned" hours total, "⋮" menu, expand caret (per-person breakdown). Grid: date columns (zoomable, `<`/`>` paging), colored bars per project showing planned allocation. "PUBLISHED" status toggle top-right (draft-vs-published schedule state). "ADD PROJECT" button. This is forward-looking capacity planning, distinct from the Calendar's after-the-fact tracked-time view.
- **Expenses** (`/expenses`): empty state "No results — Try adjusting the filters to get some results", "ADD EXPENSE" button top-right (Pro-tier feature).
- **Time Off** (`/time-off`): tabs **REQUESTS** / TIMELINE / BALANCE / POLICIES / HOLIDAYS. Empty state on fresh workspace: "No policies yet — Create policy and assign it to members" + "CREATE NEW POLICY" button. "REQUEST TIME OFF" button top-right. A policy must exist before anyone (any role) can actually submit a time-off request — this is a real gating dependency worth noting for section 4/6.

*(Admin sweep of primary MANAGE/ANALYZE nav items complete. Remaining to explore in later passes: Kiosk creation flow, Invoice creation draft, Shared reports, integrations puzzle-piece icon, Detailed/Weekly report sub-views, and the Employee-role live session once the invite to akishor2001@gmail.com is accepted.)*

---

## 4. The flow, in deep detail

This section restates everything traced above as explicit "screen → action → next screen" chains, with every branch, loop, and role difference called out. Two flowchart images cover the same ground visually (section 5).

### 4.1 Onboarding chain
`Login` → (Continue with Google/email) → `Create account` → (Continue, valid fields) → `Customize experience modal` → (Start, any field empty) → **loop: stays on same modal, shows "Complete all fields above."** → (START USING CLOCKIFY, all fields valid) → `Free trial modal` → (Let's go) → `Time Tracker` (home for every subsequent session — revisiting the app always lands here).

### 4.2 Timer chain (identical for every role)
`Time Tracker (idle)` → (type description, optionally pick Project/Task via the Client→Project→Task picker) → (click START) → `Timer running` (red STOP button, live tab-title, live sidebar counter) → (click STOP) → `This week list` (new entry appears, day and week totals update).
- **Branch/observed glitch:** STOP can produce a red "Stop time entry failed" toast simultaneously with a green "Time entry has been created" toast. The green one is the truth — reload confirms the entry always saved correctly in testing here. Treat the red toast as a false alarm, not a real failure requiring retry (see section 6).
- **Loop:** from `This week list`, clicking an entry's description or time range re-enters an **inline edit** sub-state (no navigation, just the row becoming form fields) → Escape/click-away exits back to the static row.
- **Row menu branch:** "⋮" on any entry → Split / Duplicate / Delete / Add as favorite (Split and Add-as-favorite not exercised live; Delete removes the row, presumably with no undo — not tested to avoid destructive action on the one real entry used for testing).

### 4.3 Submission chain (the core approve/reject/withdraw cycle)
`This week list` → (click "Submit for approval") → `Submit for approval modal` → (SUBMIT) → **`Pending`** state (badge + "Withdraw pending" link appear; entry now also visible to a manager on `Approvals → Pending`).

From `Pending`, exactly three things can happen next:
1. **Employee withdraws first:** (click "Withdraw pending") → `Withdraw pending modal` → (WITHDRAW) → back to `This week list`, fully editable again, no manager action needed. (This is the path that produces a **Withdrawn** archive status when it does leave a trace — observed once in this workspace's pre-seeded sample data for a different user, not reproduced from a fresh submit in this session, so it's not 100% certain every Withdraw leaves an Archive record versus only Withdraws-after-some-manager-interaction doing so.)
2. **Manager rejects — directly observed end to end (section 2.4):** Admin/Team manager opens the row on `Approvals → Pending`, clicks REJECT → `Reject timesheet modal` (note field, used live: "Please add more detail to the description before resubmitting.") → REJECT → entry moves to **`Archive: Rejected`** on the manager's side, confirmed. On the employee's side, directly confirmed: the week badge changes to **Rejected**, the entry stays inline-editable with **no unlock step**, and the manager's note does **not** surface anywhere in the employee's UI that was checked (row, icon tooltip, notification bell). The employee edits the entry and clicks "Submit for approval" again, which shows a **different, rejection-specific warning modal** ("previously rejected... make sure you adjusted the changes") before returning the week to **Pending** — closing the loop back to the top of this same chain. Confirmed live, both ends, real accounts.
3. **Manager approves:** (Admin/Team manager clicks APPROVE) → `Confirm approval modal` ("Once approved, timesheet will be locked and no one will be able to edit it anymore.") → (APPROVE) → **`Archive: Approved`**, and per that warning text, the entries become **permanently locked** — not just for the employee, the modal says "no one," implying even an Admin can't trivially un-approve/edit after the fact (no "unapprove" control was found anywhere in this pass; if Clockify has one, it wasn't discovered).
4. **Bulk shortcut:** from the Pending list itself (not the detail screen), "APPROVE ALL" approves every visible row in one action, skipping the per-row detail/confirm-modal.

### 4.4 Role-based differences in this same chain
From the Permissions matrix (3.11), User role modal, and — for the nav/no-access items — a real Member session (section 2.1):
- **Who can even reach a "Reject/Approve" button at all:** Admins and Team managers only (or Admins and Project managers, if the workspace is set that way instead — this is itself a workspace-level toggle, not fixed). A plain Member never sees `Approvals` in the nav, and **directly confirmed:** typing the `/approvals` URL directly as a Member silently redirects to `/tracker` rather than showing a permission-denied page.
- **Who can edit someone else's already-tracked time entries:** Admins only, by default (toggleable to "Admins and team managers"). A Team manager who is not also granted this cannot fix a typo in an employee's entry directly — only reject it and have the employee fix it themselves.
- **Who can see billable rates/amounts on any of this:** Admins only, by default (toggleable to Anyone) — meaning in a locked-down workspace, a Member submitting a timesheet might not even see the dollar amount attached to their own hours, only the Admin/Team manager reviewing it would.
- **Owner vs Admin:** functionally identical day-to-day (an Admin "can see and edit everything"), except only the Owner can strip someone else's Admin role away — Admin is a one-way door for everyone except the Owner.

### 4.5 Late / locked submission (error and edge behavior)
Workspace setting **"Lock time and expenses"** (3.11) — "Lock time before [date]" + optional "Automatically update lock date" — is the real mechanism behind what happens if someone tries to submit or edit time for a date that's too old: with a lock date configured, a regular Member is **blocked outright** from adding/editing time before that date (this is a hard gate, not a soft warning, per Clockify's own description text). This workspace ships with the lock date **off**, so in this trial, "late" submission is possible at any time — there is no automatic Pending-deadline behavior observed; "Pending" simply means "not yet decided," regardless of how much real-world time has passed. If a real deployment wants a hard weekly deadline, the Admin must actively turn this lock on and set/automate the date — it does not happen by default.

### 4.6 Invalid-input / error states catalogued so far
- Onboarding customize-modal: empty required dropdown + Start click → inline "Complete all fields above." tooltip, no navigation.
- Timer STOP: contradictory success/failure toasts (see 4.2 and section 6) — a real bug, not documented Clockify behavior.
- Transient `TIMEENTRY not found` / `time entry doesn't exist ... for user null` toasts while a timer is actively running — self-resolving, timer unaffected.
- **Directly tested no-permission attempt:** a real Member account navigating straight to `/approvals` by URL — silently redirected to `/tracker`, no error message shown.
- **Confirmed UI gap:** a manager's rejection note is written into the Reject modal but does not appear anywhere in the rejected employee's own UI (entry row, icon tooltip, or notification bell) — worth flagging to a competing team as a real usability gap, not just an inference.
- **Kiosk creation, empty required fields:** clicking CREATE with the Create Kiosk modal's required fields blank produces **no error toast** — the CREATE button is simply disabled/inert (see 3.9.1). Same silent-disabled-button pattern as the onboarding Continue button.
- **Project creation, empty name:** silent no-op, no error (3.2.1).
- **Project creation, duplicate name:** silently **allowed** — no uniqueness check, a second identically-named project is created (3.2.1).
- **Client creation, empty name:** silent no-op, same as Project (3.2.1).
- **Client creation, duplicate name:** **blocked**, with a real visible error toast — "Client with name '[SAMPLE] Client A' already exists" (3.2.1). Directly inconsistent with Project's duplicate-name behavior — a genuine product inconsistency, not a documentation gap.
- **Time Off request against zero balance:** hard-blocked with a real inline validation error, "You don't have enough time off allocated for the selected period" (3.12.2) — not a soft warning.
- *(Still not triggered: invalid email format in the invite field, an expired invite link — flagged explicitly as untested in section 6, results not fabricated.)*

### 4.7 Time Off zero-balance mechanic — worth flagging prominently

A non-obvious mechanic confirmed via the full real cycle in 3.12.2: **default Time Off policies (Vacation, Sick leave) exist out of the box and are auto-assigned to every member, but each member's balance starts at 0.00d regardless of the policy's configured accrual rate.** A request against that zero balance is hard-blocked with a real validation error. Balance only becomes usable once an Admin manually credits it via the **Manage balance** modal (Add/Remove toggle, days stepper, Available-from/Last-valid-date, Note) — or, presumably, once enough real time passes for the accrual rate to organically credit days (not directly tested in this pass). For a competing Timer tool, this is worth a deliberate design decision one way or the other: Clockify's default leaves a new workspace's time-off feature effectively unusable until an Admin performs this extra, easy-to-miss manual step.

### 4.8 Kiosk / Invoice / Reports / Integrations / Workspace-Settings flows — summary chains

- **Kiosk creation:** `Kiosks (empty state)` → (CREATE KIOSK) → `Create Kiosk modal` → (fill Name/Assignees/Default Project/Default break Project/logout-hours/Authentication toggle, CREATE) → `Kiosk created`, appears in list. Empty required fields → CREATE stays disabled, no error toast (3.9.1).
- **Invoice creation:** `Invoices list` → (CREATE INVOICE) → `Create Invoice modal` (Client, Currency, auto-suggested Invoice ID, Issue/Due dates) → (CREATE) → `Draft invoice (Unsent)`. From an existing invoice's detail page, further branches exist but were not exercised: DOWNLOAD, SEND, RESEND, RECORD PAYMENT, Recurring settings, Actions dropdown (3.6.1).
- **Reports sub-tabs:** `Reports (Summary)` → (sidebar dropdown) → branches to Detailed (row-per-entry, shows green-checkmark lock icon on approved entries), Weekly (per-day grid), Shared (empty state on this workspace), plus a separate TEAM section (Attendance, Assignments) and EXPENSE section (Detailed) in the same dropdown menu (3.7.1).
- **Integrations:** two independent entry points, not a single chain — top-nav puzzle icon → CAKE.com Marketplace modal (cross-product add-on store); Workspace Settings → Integrations tab → curated Clockify-specific list (QuickBooks/Jira/Google Calendar/Outlook Calendar + "Other integrations" link to 58+ more) (3.13).
- **Workspace Settings remaining tabs:** `Workspace settings (Permissions, 3.11)` → (tab click) → Alerts / Accounts (Enterprise-gated) / Authentication (mixed Pro/Enterprise-gated) / Custom Fields (drag-and-drop builder) / Integrations (see above) / Add-ons (empty, links to Marketplace) / Import (CSV, 10MB max, downloadable templates) (3.14).

---

## 5. Flow diagrams

Two flowcharts, built directly from the chains traced in section 4 (not just the happy path — branches and loops included):

- **Admin flow** — `admin-flow.png` — onboarding → nav map → Projects/budget → Team/roles/invite → Approvals (reject/approve/bulk) → Timer/submit/withdraw loop → other nav (Invoices/Reports/Dashboard/Kiosks/Workspace settings).
- **Employee flow** — `employee-flow.png` — invite → onboarding → Timer/submit/withdraw loop → the reject→edit→resubmit loop and locked-approved end state → Time Off request (with its Policy-must-exist-first gate) → the set of screens inferred as absent from a Member's nav, marked with a dashed line and a note.

(Both images are attached alongside this document.)

---

## 6. Edge cases and error states

| Edge case | What actually happened | Where |
|---|---|---|
| Project over budget (790.80% of a 400 USD manual budget) | Purely visual/reporting: Amount and Progress bar turn red on the Projects list and (presumably) in Reports. No blocking behavior found anywhere — time can still be tracked against an over-budget project (not exhaustively retested after the fact, but nothing in the UI suggested a hard stop). | Projects list, Project Settings tab |
| Rejecting a submitted timesheet | Manager (Admin) rejects with an optional note via a dedicated modal; entry leaves Pending immediately, lands in Archive as "Rejected," no further confirmation needed. | Approvals → Pending → Timesheet detail |
| Approving a submitted timesheet | Confirmation modal explicitly warns the action **locks** the entry with no one able to edit it afterward — this is a real, stated one-way action, not just a status label. | same |
| Employee withdrawing a pending submission before a decision | "Withdraw pending" modal explicitly states entries "won't be available for approval unless they are resubmitted" — confirms Withdrawn is a genuine third outcome alongside Approved/Rejected, initiated by the employee, not the manager. | Time Tracker "This week" view |
| Late / past-date submission | No default deadline enforcement observed. The real mechanism is the opt-in "Lock time and expenses" workspace setting — off by default in this trial — which, when turned on, hard-blocks Members from editing time before the lock date rather than just flagging it. | Workspace settings → Permissions |
| No-permission action attempted | Not directly triggered (would require a genuine second, lower-privileged login) — but the entire Permissions tab (3.11) documents exactly which role gate blocks which action, in Clockify's own words, which is the authoritative source for what a no-permission attempt would hit. | Workspace settings → Permissions |
| Timer STOP contradictory toasts | Real, reproducible-looking bug: clicking STOP produced both "Stop time entry failed. Please try again." (red) and "Time entry has been created" (green) simultaneously in the same click. The entry was in fact saved correctly (confirmed on page reload) — the failure toast is a false negative. Worth flagging to a QA/eng team building a competing product as a UX pitfall to avoid: don't show a hard failure toast unless the action actually failed. | Time Tracker, while stopping a running timer |
| Transient timer-related error toasts while running | `TIMEENTRY with ID '...' not found` and `Currently running time entry doesn't exist on workspace ... for user null.` appeared and cleared on their own while a timer was actively running normally underneath (confirmed via page reload). Read as noisy background sync messages rather than real errors. | Time Tracker, while a timer is running |
| Empty required field (account creation, invites, etc.) | Confirmed only for the onboarding "customize experience" modal (inline tooltip, no navigation). Not yet triggered for Project/Client name fields, the invite-by-email field, or Project budget numeric field — flagged as untested. | various |
| Duplicate entry (e.g. two Clients/Projects with the same name) | **Directly tested, and the two differ:** a duplicate **Project** name is silently allowed (no uniqueness validation, a second identically-named project is created). A duplicate **Client** name is blocked with a real visible error toast: "Client with name '[SAMPLE] Client A' already exists." A genuine product inconsistency worth deliberately avoiding in a competing tool. | Projects / Clients |
| Empty name on Project or Client creation | Both are a **silent no-op** — clicking Create with a blank Name field does nothing, no error toast shown. | Projects / Clients |
| Sending a real invoice | Deliberately not tested — creating/sending a real invoice is treated as an irreversible-adjacent action and was out of scope without explicit sign-off. A **draft (Unsent)** invoice was created and left unsent. | Invoices |
| Kiosk creation with empty required fields | Directly tested: clicking CREATE produces **no error toast** — the CREATE button is simply disabled/inert until required fields are filled. Same silent-disabled-button pattern as elsewhere in the product. | Kiosks → Create Kiosk modal |
| Time Off request against zero balance | Directly tested, real hard block: default Time Off policies (Vacation, Sick leave) are auto-assigned to every member but start with **0.00d balance** regardless of accrual rate. Submitting a request against that balance produces a real inline validation error: "You don't have enough time off allocated for the selected period." Balance must be manually credited via the Manage Balance modal before any request can succeed — a non-obvious, easy-to-miss setup step. | Time Off → Requests / Balance |
| Invalid email format in the invite-by-email field | **Not tested in this pass** — flagged explicitly as untested; no result to report, none fabricated. | Team → Add full members modal |
| Expired invite link | **Not tested in this pass** — flagged explicitly as untested; no result to report, none fabricated. | Invite / onboarding flow |

---

## 7. Screen inventory

Every distinct screen captured, in the order first reached. Screenshots for all of these are included as separate image files.

**Onboarding / account**
1. Login (`/login`)
2. Create account (`/memberships`)
3. "Help us customize your Clockify experience" modal (incl. its inline validation-error state)
4. "Your free trial starts today!" modal

**Core (shared across roles)**
5. Time Tracker — idle (`/tracker`)
6. Time Tracker — running timer
7. Time Tracker — "This week" list (with a completed entry)
8. Time Tracker — inline edit state
9. Time Tracker — entry row "⋮" menu (Split/Duplicate/Delete/Add as favorite)
10. Project/Client picker dropdown (from the Tracker's "+ Project" control)
11. "Submit for approval" modal
12. "This week" — Pending state (with "Withdraw pending" link)
13. "Withdraw pending" modal
14. Calendar (`/calendar`)
15. Dashboard (`/dashboard`)
16. Reports — Summary tab (`/reports/summary`)
17. Time Off (`/time-off`) — Requests tab, empty-policy state
18. Expenses (`/expenses`) — empty state

**Admin-only management**
19. Projects list (`/projects`), incl. the over-budget sample project
20. Project detail — Tasks tab
21. Project detail — Settings tab (rates + budget config)
22. Team screen (`/teams`) — Full tab
23. "Add full members" modal
24. "User role" modal (Member/Admin/Project manager/Team manager definitions)
25. Clients (`/clients`)
26. Tags (`/tags`) — empty state
27. Invoices list (`/invoices`)
28. Approvals — Pending tab
29. Approvals — Unsubmitted tab
30. Approvals — Archive tab (Approved/Rejected/Withdrawn statuses)
31. Timesheet detail / approval screen
32. "Reject timesheet" modal
33. "Confirm approval" modal
34. Kiosks (`/kiosks`) — empty state
35. Schedule (`/scheduling`) — Projects tab
36. Workspace settings — General tab
37. Workspace settings — Permissions tab (two scroll states captured)
38. "..." workspace menu (Workspace settings / Subscription / Manage workspaces)

**Employee/Member — captured live from a real second account (`thalaivan.ugam@gmail.com`)**
39. Invitee Create account screen — no Organization field (`01-invite-create-account.jpg`)
40. Time Tracker — Member view, idle, real nav confirmed (`02-tracker-member-view.jpg`)
41. Projects list — Member view (`03-projects-member-view.jpg`)
42. Project detail — Member view, no Settings tab (`04-project-detail-member-view.jpg`)
43. Team screen — Member view, read-only (`05-team-member-view.jpg`)
44. Timer running — Member's own test entry (`06-timer-running.jpg`)
45. "This week" list after STOP — Member view (`07-weekview-entry-saved.jpg`)
46. "Submit for approval" modal — Member view (`08-submit-for-approval-modal.jpg`)
47. "This week" — Pending state, Member view (`09-pending-status.jpg`)
48. Direct `/approvals` URL as Member → silent redirect to `/tracker` (`10-approvals-url-redirect-noaccess.jpg`)
49. Approvals — Pending queue showing the real employee's submitted entry, Admin view (`45-approvals-pending-real-employee-entry.jpg`)
50. Timesheet detail for the real employee's entry, Admin view (`46-timesheet-detail-real-employee.jpg`)
51. "Reject timesheet" modal, used live, Admin view (`47-reject-modal-real.jpg`)
52. Approvals — Pending queue after reject, Admin view (`48-approvals-pending-after-reject.jpg`)
53. Approvals — Archive tab showing the real Rejected entry, Admin view (`49-archive-rejected-real.jpg`)
54. "This week" — Rejected badge, Member view (`11-rejected-status-view.jpg`)
55. Rejected entry row — icon tooltip, Member view (`12-rejected-tooltip.jpg`)
56. Rejected entry — inline-editable with no unlock step, Member view (`13-rejected-entry-editable.jpg`)
57. Entry after edit, still tagged Rejected until resubmit, Member view (`14-edited-after-reject.jpg`)
58. Resubmit modal with rejection-specific warning copy, Member view (`15-resubmit-modal-warning.jpg`)
59. "This week" — back to Pending after resubmit, Member view (`16-resubmitted-pending-again.jpg`)

**Reject/Approve full real cycle, Reports, Kiosks, Invoices, Integrations, Workspace Settings, Schedule Team tab, Time Off, and Project/Client edge cases — captured live**
60. "Confirm approval" modal, after the real employee's resubmit, Admin view (`50-confirm-approval-real.jpg`)
61. Full Reports dropdown menu — Time Report (Summary/Detailed/Weekly/Shared), Team (Attendance/Assignments), Expense (Detailed) (`51-reports-menu-full.jpg`)
62. Reports — Detailed tab, showing the green-checkmark locked/approved entry (`52-reports-detailed-approved-entry.jpg`)
63. Reports — Weekly tab (`53-reports-weekly.jpg`)
64. Reports — Shared tab, empty state (`54-reports-shared-empty.jpg`)
65. Kiosks — empty state (`55-kiosks-empty.jpg`)
66. "Create Kiosk" modal (`56-create-kiosk-modal.jpg`)
67. Create Kiosk — empty-field validation (CREATE disabled, no error toast) (`57-create-kiosk-empty-validation.jpg`)
68. Kiosk created successfully (`58-kiosk-created.jpg`)
69. Invoices list — Sent/Overdue/Unsent status badges (`59-invoices-list.jpg`)
70. Invoice detail — existing sample invoice (`60-invoice-detail-sample.jpg`)
71. "Create Invoice" modal (`61-create-invoice-modal.jpg`)
72. Draft invoice created (Unsent) (`62-invoice-draft-created.jpg`)
73. Top-nav puzzle-icon "Browse Marketplace" popover (`63-integrations-popover.jpg`)
74. CAKE.com Marketplace modal (`64-cake-marketplace-integrations.jpg`)
75. Workspace settings — full tab bar (`65-workspace-settings-tabs-full.jpg`)
76. Workspace settings — Alerts tab (`66-ws-settings-alerts.jpg`)
77. Workspace settings — Accounts tab, Enterprise-gated, not tested (`67-ws-settings-accounts-enterprise.jpg`)
78. Workspace settings — Authentication tab, mixed Pro/Enterprise gating, not tested (`68-ws-settings-authentication.jpg`)
79. Workspace settings — Custom Fields tab (`69-ws-settings-custom-fields.jpg`)
80. Workspace settings — Integrations tab, curated list + "Other integrations" link (`70-ws-settings-integrations-list.jpg`)
81. Workspace settings — Add-ons tab, empty (`71-ws-settings-addons-empty.jpg`)
82. Workspace settings — Import tab (`72-ws-settings-import.jpg`)
83. Schedule — Team tab, per-member capacity grid (`73-schedule-team-tab.jpg`)
84. Time Off — Requests tab (`74-timeoff-requests-list.jpg`)
85. Time Off — Policies tab, default Vacation/Sick leave policies (`75-timeoff-policies-defaults.jpg`)
86. Time Off request — insufficient-balance validation error (`76-timeoff-request-insufficient-balance.jpg`)
87. Time Off — Balance tab, 0.00d for both policies (`77-timeoff-vacation-zero-balance.jpg`)
88. Time Off — Balance tab, per-member Accrued/Used/Available columns (`78-timeoff-balance-table.jpg`)
89. Time Off — Balance tab inline "Add" link (`79-timeoff-balance-add-link.jpg`)
90. "Manage balance" modal (`80-manage-balance-modal.jpg`)
91. Balance credited (3.00d Vacation) (`81-balance-credited.jpg`)
92. Time Off request form, now submittable (`82-timeoff-request-ready.jpg`)
93. Time Off — Balance tab, reduced after request submission (`83-timeoff-balance-after-request.jpg`)
94. Time Off — Requests tab, new pending row (`84-timeoff-request-new-row.jpg`)
95. Time Off request, approved by Admin (`85-timeoff-request-approved.jpg`)
96. Project creation — empty name, silent no-op (`86-project-empty-name-noop.jpg`)
97. Project creation — duplicate name, silently allowed (`87-project-duplicate-name-allowed.jpg`)
98. Client creation — duplicate name, blocked with error toast (`88-client-duplicate-name-blocked.jpg`)

**Not yet captured (out of scope or untested this pass):** Member-scoped Reports (billable-amount visibility), Member's personal Settings/avatar menu, invalid email format in the invite field, an expired invite link, whether a rejection note reaches the employee by email, and any screen genuinely gated behind a paid tier not enabled in this trial (Accounts tab, and the SSO-subdomain/data-region portions of Authentication).

---

*End of current draft. Screenshots referenced throughout are delivered alongside this document. Sections flagged (inferred) in Part 2 should be re-verified once/if a genuinely separate Employee login is completed.*
