# Harvest (getharvest.com) — click-by-click teardown

Observed live on 23 Sep 2026 in a real Harvest trial account ("Acme Test Co", `acmetestco.harvestapp.com`). Every screenshot comes from the live app. Images are in `shots/` and the flowcharts are in `flowcharts/`.

**Test accounts**

| Role | Person | How created |
|---|---|---|
| Owner (Administrator) | Vishnuvarthan Venkatapathy, `vishnu88varthan@gmail.com` | Google sign-up (the user clicked "Create my account") |
| Member (Employee) | Thalaivan Ugam, `thalaivan.ugam@gmail.com` | Invited by Owner with the Member profile; the user accepted with Google |
| (pending) | Maya Member, `vishnu88varthan+member@gmail.com` | Invited; never accepted, and left pending |

**Test data created:** clients *Example Client* (seeded) and *Globex Corp* (with contact Hank Scorpio); projects *Example Project* (seeded) and *[WR-01] Website Redesign*; task *QA Testing*; time entries; 3 Owner expenses and 1 Member expense; draft invoice #2 (**not sent**); estimate #1 (draft).

**Constraints hit, and how they were handled**

- **One login per browser.** Harvest keeps one session per browser, so Owner and Member could not stay signed in at the same time. Workaround: Sign out, then "Sign in with Google" and pick the other account from Google's account chooser. The user approved this. It took 3 swaps for the approval round-trip.
- **Sending invoices is blocked on the trial.** Harvest requires a Stripe connection or a paid plan first. Sending was also off-limits by instruction, so it was not tested.
- **Not done:** Reimbursement payouts (they need bank or card details), Close account (destructive), Bulk delete (destructive), and a wrong-password sign-in (it would mean typing a password).
- **No paid-tier gating.** The trial unlocks every Enterprise feature for 30 days. Features badged **[Enterprise]** would be missing on lower paid tiers; that could not be observed and is called out in the text.

---

## 1. Overview

**What it is.** Harvest is a time-tracking and billing tool for service businesses. People log time (with a timer or by duration) and expenses against client projects. Managers approve timesheets, then turn approved billable time and expenses into invoices. Budgets, profitability and utilization reporting come out of the same data. Estimates can be converted to invoices.

**Who it is for.** Agencies, consultancies, studios and freelancers who bill clients by the hour or by fixed fee. Onboarding asks for team size (Just me to 100+) and industry (Architecture, Consulting, Design, Legal, Software Development and others).

**Main objects.** Client → Contact; Project (Time & Materials / Fixed Fee / Non-Billable; billable-rate strategy; budget) → Task assignments + Team assignments; Time entry; Expense (category, billable, reimbursable, receipt); Timesheet approval (per person per period); Invoice (draft → sent → paid); Estimate (draft → sent → accepted/declined); Reports.

**Roles (permission profiles), from the invite screen**

| Profile | What Harvest says it can do |
|---|---|
| **Member** (default, 4 of 50 permissions) | Track time and expenses on assigned projects. View and log only their own time and expenses. |
| Project Manager | Manage projects, clients and tasks. View and edit time and expenses for their team. No rates, invoices, people or account settings. |
| People Admin | Manage people and all time entries and expenses. No rates, invoices or account settings. |
| Accounting | Manage invoices, estimates, expenses and clients. View rates and reports. No people or account settings. |
| Executive Manager | Manage time, expenses, projects, people, invoices and reports. View rates. No account settings or billing. |
| Administrator | Full access, including account settings, billing and all permissions. The **Owner** is an Administrator who also appears as "Account Owner". |

Each profile is a preset over **50 checkboxes in 11 groups**, and the Owner can adjust them individually:

- Time (8)
- Expenses (6)
- Projects (5)
- Clients & Tasks (4)
- People (4)
- Rates (6)
- Invoices (5)
- Estimates (2)
- Reports (5)
- Approvals (1)
- Account (4)

Only the two ends of this range were walked: **Owner/Administrator** and **Member**.

---

## 2. Employee / Member walkthrough (in first-time order)

| # | Screen | Screenshot |
|---|---|---|
| 1 | **Invite email → accept.** Email "*Vishnuvarthan Venkatapathy has invited you to join Harvest*" with a Join link that expires in 14 days. The user accepted it with Google (the agent may not create accounts). | *(email not screenshotted; see the Owner-side invite, [25c](shots/25c_invite_filled.jpg))* |
| 2 | **Timesheet – Day (landing page).** Reduced sidebar: Timesheet, Expenses, Projects, Reports, Customize, Bulk actions. No trial banner, no Team/Clients/Invoices/Approvals/Settings. | ![](shots/M01_member_timesheet_day.jpg) |
| 3 | **Timesheet – Week, empty.** | ![](shots/M08_member_week_empty.jpg) |
| 4 | **Add row modal.** The project list shows **only assigned projects**. | ![](shots/M09_member_add_row_modal.jpg) ![](shots/M09b_member_project_list_only_assigned.jpg) |
| 5 | **Week with hours.** Some cells are locked; see §4. | ![](shots/M10_member_week_locked_days.jpg) ![](shots/M10b_locked_tooltip.png) ![](shots/M10c_member_week_hours.jpg) |
| 6 | **Submit week.** Inline confirm, then "Pending approval" pill. | ![](shots/M08b_submit_confirm_inline.jpg) ![](shots/M08c_empty_week_submitted_pending.jpg) ![](shots/M08d_resubmit_menu.jpg) |
| 7 | **Expenses (empty).** Only the "All expenses" tab. | ![](shots/M04_member_expenses_empty.jpg) |
| 8 | **Expense form.** | ![](shots/M11_member_expense_form.jpg) ![](shots/M11b_member_expense_invalid_date.jpg) ![](shots/M11c_member_expense_saved.jpg) |
| 9 | **Review before submit** (`/daily/review`). | ![](shots/M12_member_review_before_submit.jpg) |
| 10 | **Submitted / pending** (expenses and week). | ![](shots/M12b_member_submitted_pending.jpg) ![](shots/M13_member_week_pending.jpg) |
| 11 | **After Owner feedback:** edit, add a note, resubmit. | ![](shots/62_member_after_reject_pending.jpg) ![](shots/62b_member_add_note_popover.jpg) ![](shots/62c_member_resubmit_confirm.jpg) ![](shots/62d_member_resubmit_review.jpg) ![](shots/62e_member_resubmitted.jpg) |
| 12 | **Approved:** week and expenses locked. | ![](shots/67_member_week_approved_locked.jpg) ![](shots/67c_member_expenses_approved_locked.jpg) |
| 13 | **Notifications.** | ![](shots/67b_member_notifications_approved.jpg) |
| 14 | **Projects (empty state)** with an "assigned projects" link. | ![](shots/M02_member_projects_empty.jpg) |
| 15 | **My profile:** Assigned projects. | ![](shots/M03_member_profile_assigned_projects.jpg) |
| 16 | **My profile:** Basic info (and its validation). | ![](shots/M06_member_profile_basic_info.jpg) ![](shots/M06b_member_profile_empty_name.jpg) |
| 17 | **My profile:** Permissions (read-only). | ![](shots/M07_member_permissions_readonly.jpg) |
| 18 | **Reports:** Time (empty) and Activity log. | ![](shots/M05_member_reports_empty.jpg) ![](shots/M14_member_activity_log.jpg) |
| 19 | **Bulk actions** (delete-only cards). | ![](shots/M15_member_bulk_actions.jpg) |
| 20 | **Blocked pages** (direct URLs). | ![](shots/M20_member_blocked_team_404.jpg) ![](shots/M21_member_projects_new_403.jpg) ![](shots/M22_member_invoice_404.jpg) ![](shots/M23_member_people_permissions_404.jpg) |

---

## 3. Admin / Owner walkthrough (in first-time order)

**Sign-up and onboarding**

| # | Screen | Screenshot |
|---|---|---|
| 1 | Sign-up page | ![](shots/00_signup.jpg) |
| 2 | Google completion: "You're almost there!" | ![](shots/01_complete_signup_google.jpg) |
| 3–7 | Welcome wizard: team size → industry → invite → referral/marketing → goal | ![](shots/02_welcome_team_size.jpg) ![](shots/02b_welcome_team_size_selected.jpg) ![](shots/03_welcome_industry.jpg) ![](shots/04_welcome_invite.jpg) ![](shots/04b_welcome_invite_invalid_email.jpg) ![](shots/05_welcome_marketing.jpg) ![](shots/06_welcome_goal.jpg) ![](shots/06b_welcome_goal_selected.jpg) |
| 8 | Billing-cycle modal → "You're all set" (Enterprise trial) | ![](shots/07_billing_cycle_modal.jpg) ![](shots/08_trial_welcome_modal.jpg) |
| 9 | Getting Started checklist and seeded demo invoice #1 | ![](shots/09_getting_started_invoice_editor.jpg) |

**Timesheet**

| # | Screen | Screenshot |
|---|---|---|
| 10 | Timesheet – Day (empty) | ![](shots/10_timesheet_day_empty.jpg) |

**Clients**

| # | Screen | Screenshot |
|---|---|---|
| 11 | Clients list | ![](shots/11_clients_list.jpg) |
| 12 | New client form (and its validation) | ![](shots/12_client_new.jpg) ![](shots/12b_client_new_empty_name.jpg) ![](shots/12c_client_duplicate_name.jpg) ![](shots/12d_client_tax_not_number.jpg) |
| 13 | Clients after save; Edit client | ![](shots/13_clients_list_after_add.jpg) ![](shots/14_client_edit.jpg) |
| 14 | New contact (and its validation) | ![](shots/15_contact_new.jpg) ![](shots/15b_contact_empty.jpg) ![](shots/15c_contact_invalid_email.jpg) ![](shots/16_clients_with_contact.jpg) |
| 15 | Clients menus: New ▾, Actions, Import/Export | ![](shots/17_clients_new_dropdown.jpg) ![](shots/17b_clients_actions_menu.jpg) ![](shots/17c_clients_import_export.jpg) |

**Tasks**

| # | Screen | Screenshot |
|---|---|---|
| 16 | Tasks list | ![](shots/18_tasks_list.jpg) |
| 17 | Inline new task (and its validation); row Actions | ![](shots/19_task_new_inline.jpg) ![](shots/19b_task_duplicate_name.jpg) ![](shots/19c_task_negative_rate_saved.jpg) ![](shots/19d_task_row_actions.jpg) |

**Projects**

| # | Screen | Screenshot |
|---|---|---|
| 18 | Projects list | ![](shots/20_projects_list.jpg) |
| 19 | New project form (4 scroll positions) | ![](shots/21_project_new_top.jpg) ![](shots/21b_project_new_type_budget.jpg) ![](shots/21c_project_new_tasks_team.jpg) ![](shots/21d_project_new_invoice_values.jpg) |
| 20 | Client picker and date picker | ![](shots/21e_project_client_dropdown.jpg) ![](shots/21f_project_datepicker.jpg) ![](shots/21g_project_end_before_start.jpg) |
| 21 | Project detail; Actions menu | ![](shots/22_project_detail.jpg) ![](shots/22b_project_actions_menu.jpg) |
| 22 | "Heads up!" modal; duplicate project name | ![](shots/23_project_heads_up_no_rate.jpg) ![](shots/23b_project_duplicate_name.jpg) |

**Team and invites**

| # | Screen | Screenshot |
|---|---|---|
| 23 | Team | ![](shots/24_team_members.jpg) |
| 24 | Invite person (step 1) | ![](shots/25_invite_person_form.jpg) ![](shots/25b_invite_duplicate_email.jpg) ![](shots/25c_invite_filled.jpg) |
| 25 | Permissions (step 2) | ![](shots/26_invite_permissions_member.jpg) ![](shots/26b_permissions_expanded.jpg) ![](shots/26c_permission_profiles.jpg) |
| 26 | Assign projects (step 3) → Person profile | ![](shots/27_assign_projects_picker.jpg) ![](shots/27b_assign_projects_selected.jpg) ![](shots/28_person_assigned_projects.jpg) |
| 27 | Team with a pending invite | ![](shots/44_team_member_pending.jpg) |

**Time tracking**

| # | Screen | Screenshot |
|---|---|---|
| 28 | New time entry, timer, and Edit entry modals | ![](shots/29_time_new_entry_modal.jpg) ![](shots/29b_time_duration_abc.jpg) ![](shots/29c_timer_running.jpg) ![](shots/29d_time_edit_modal.jpg) ![](shots/29e_time_25h_accepted.jpg) ![](shots/29f_time_project_picker.jpg) |
| 29 | Week view; submit menu; Teammates switcher | ![](shots/30_timesheet_week.jpg) ![](shots/30b_week_invalid_cell_ignored.jpg) ![](shots/30c_submit_menu.jpg) ![](shots/30d_teammates_switcher.jpg) |
| 30 | Calendar view | ![](shots/31_timesheet_calendar.jpg) |

**Expenses**

| # | Screen | Screenshot |
|---|---|---|
| 31 | Expenses (empty); Track expenses ▾ | ![](shots/32_expenses_empty.jpg) ![](shots/32b_track_expenses_menu.jpg) |
| 32 | Expense form: categories, negative amount, edit, mileage | ![](shots/33_expense_new_form.jpg) ![](shots/33b_expense_category_list.jpg) ![](shots/33c_expense_negative_saved.jpg) ![](shots/33d_expense_edit_inline.jpg) ![](shots/33e_expense_mileage_fields.jpg) |
| 33 | Expenses list; Configure > Categories; Reimbursements | ![](shots/34_expenses_list.jpg) ![](shots/35_expense_categories_config.jpg) ![](shots/36_reimbursements_overview.jpg) |

**Invoices and estimates**

| # | Screen | Screenshot |
|---|---|---|
| 34 | Invoices overview | ![](shots/37_invoices_overview_popup.jpg) ![](shots/37b_invoices_overview.jpg) |
| 35 | New invoice builder | ![](shots/38_invoice_new_choose_client.jpg) ![](shots/38b_invoice_client_dropdown.jpg) ![](shots/38c_invoice_unapproved_warning.jpg) ![](shots/38d_invoice_line_items.jpg) ![](shots/38e_invoice_appearance_panel.jpg) ![](shots/38f_invoice_duplicate_number.jpg) |
| 36 | Invoice detail (Draft #2) | ![](shots/39_invoice_draft_detail.jpg) ![](shots/39b_invoice_more_actions.jpg) ![](shots/39c_invoice_send_step_locked.jpg) ![](shots/39d_invoice_get_paid_gate.jpg) ![](shots/39e_record_payment_disabled.jpg) |
| 37 | Uninvoiced report | ![](shots/40_report_uninvoiced.jpg) |
| 38 | Estimates: list, new estimate, validation | ![](shots/41_estimates_empty.jpg) ![](shots/42_estimate_new.jpg) ![](shots/42b_estimate_empty_client.jpg) ![](shots/42c_estimate_price_abc.jpg) |
| 39 | Estimate detail: accept → revert loop | ![](shots/43_estimate_draft_detail.jpg) ![](shots/43b_estimate_actions_draft.jpg) ![](shots/43c_estimate_accepted.jpg) ![](shots/43d_estimate_actions_accepted.jpg) ![](shots/43e_estimate_revert_confirm.jpg) ![](shots/43f_estimate_reverted_draft.jpg) |

**Approvals**

| # | Screen | Screenshot |
|---|---|---|
| 40 | Approvals (empty) and its filters | ![](shots/45_approvals_empty.jpg) ![](shots/45b_approvals_status_filter.jpg) ![](shots/45c_approvals_group_by.jpg) ![](shots/45d_approve_visible_empty.jpg) |
| 41 | Approvals with Member's submission; person detail | ![](shots/60_owner_approvals_pending_thalaivan.jpg) ![](shots/60b_owner_approvals_person_detail.jpg) ![](shots/60c_owner_approvals_timesheet_details.jpg) |
| 42 | Send email (reject / feedback) | ![](shots/61_owner_send_email_modal_empty.jpg) ![](shots/61b_owner_send_email_filled.jpg) ![](shots/61c_owner_email_sent_still_pending.jpg) |
| 43 | Approve / Undo / Approved filter | ![](shots/63_owner_approvals_resubmitted_7h.jpg) ![](shots/63b_owner_approved_undo.jpg) ![](shots/63c_owner_approvals_status_approved.jpg) ![](shots/65_owner_reapproved_undo_link.jpg) ![](shots/65b_owner_undo_approval_withdrawn.jpg) |
| 44 | Member's week seen by Owner; Withdraw approval | ![](shots/64_owner_views_member_week_approved_locked.jpg) ![](shots/64b_owner_withdraw_confirm.jpg) ![](shots/64c_owner_withdrawn_pending.jpg) |
| 45 | Notifications | ![](shots/66_owner_notifications.jpg) |

**Reports and settings**

| # | Screen | Screenshot |
|---|---|---|
| 46 | Reports: Time, New report ▾, Profitability, Activity log, Contractor, Detailed time | ![](shots/46_reports_time.jpg) ![](shots/46b_new_report_menu.jpg) ![](shots/46c_reports_profitability.jpg) ![](shots/46d_reports_activity_log.jpg) ![](shots/46e_reports_contractor.jpg) ![](shots/46f_detailed_time_report.jpg) |
| 47 | Settings: Billing, Preferences, Modules | ![](shots/47_settings_billing.jpg) ![](shots/47b_settings_preferences.jpg) ![](shots/47c_settings_modules.jpg) |

**Sign out**

| # | Screen | Screenshot |
|---|---|---|
| 48 | Sign out → Sign in page (and its validation) | ![](shots/50_signed_out_sign_in.jpg) ![](shots/50b_sign_in_empty_submit.jpg) |

---

## 4. The flow in deep detail

Format for each screen:

- **In:** how you get there.
- **Out:** how you leave.
- **Branches / loops:** what can happen on the screen.
- **Errors:** validation and error states actually seen.
- **Role:** what differs between Owner and Member.

Error texts are quoted exactly. "Native" means the browser's HTML5 `required` bubble ("Please fill out this field."), not an app message.

### 4.1 Sign-up and onboarding (Owner only)

- **Sign-up** (`getharvest.com/signup`)
  - **In:** marketing site.
  - **Fields:** "Sign up with Google", or First/Last/Company/Work email/Password → "Start your free trial". "No credit card required."
  - **Out:** Google → `id.getharvest.com/harvest/complete_sign_up` ("You're almost there!"). Name and email are prefilled from Google; only **Company name** is typed. "Create my account" means agreeing to the Terms of Service.
  - Not exercised by the agent (account creation is off-limits).
- **Welcome wizard** (`/welcome/*`), step by step:
  1. `team_size`: Just me / 2 / 3–9 / 10–24 / 25–49 / 50–99 / 100+. **Next stays disabled until something is picked.**
  2. `industry`: 13 options, with "Skip step and continue".
  3. `team_invites`: email field + "Add another"; "Invite teammates" or "Skip step and continue". **The Skip link disappears once text is typed**, so the field must be cleared to skip.
  4. `marketing`: "How did you hear about Harvest?" + an "Email me about product updates…" checkbox, pre-ticked (the agent unticked it).
  5. `what_to_do_first`: Invoice my clients / Keep projects profitable / Pay my team / Balance workload / Track time for myself → "Set up Harvest".
  - **Errors:** invite field "not-an-email" → native bubble "*Please include an '@' in the email address. 'not-an-email' is missing an '@'.*"
- **After the wizard (goal = invoicing)**
  - Lands in the invoice editor with a **seeded demo invoice #1** for a seeded "Example Client", plus a seeded "Example Project" and tasks.
  - **Billing-cycle modal:** Yearly $14 or Monthly $17.50 per seat per month, "You won't be charged until 22/10/2026". It has an **X**, then the "You're all set — Enterprise unlocked" modal.
  - **Getting Started bar** on every page: 1 Create first invoice ✓, 2 Add a client, 3 Connect Stripe. It advanced as steps were done.

### 4.2 Global shell

- **Owner sidebar:**
  - Timer, Recent, More (⋯).
  - Track: Timesheet, Expenses. Organize: Team, Clients, Projects, Tasks. Bill: Invoices, Estimates. Review: Approvals, Reports.
  - Customize, Integrations, Settings, Upgrade, profile menu (My profile, My time report, Notifications, Refer a friend, Help, Sign out), bell.
  - Top trial banner "30 days left… Upgrade now". "Harvest AI" floating button. An AI prompt box ("What would you like to do with your …?") on most list pages.
- **Member sidebar:** Timesheet, Expenses, Projects, Reports, Customize, Bulk actions, profile, bell. **No trial banner, no Upgrade.**
- **Sessions:** one per browser. Signing in as another person in another tab replaces the session in all tabs.

### 4.3 Clients (Owner; the Member gets 404)

- **List** (`/clients`)
  - **In:** sidebar.
  - **Actions:** "+ New client" split button (▾ New contact); **Actions** ▾ (Archive clients / Reactivate clients / View all bulk actions); **Import/Export** ▾ (Import clients from CSV, Import contacts from CSV, Export clients, Export contacts); filter box; per row **Edit** and **+ Add contact**. Contacts are listed under their client.
- **New client** (`/clients/new`)
  - **Fields:** Client name*, Address, Preferred currency (Account default USD), Invoice due date (Upon receipt …), Tax % + "Enable second tax", Discount %. Save client / Cancel. "← Back to Clients".
  - **Errors:**
    - Empty name → native required.
    - Existing name → server re-render, red banner "*We're sorry, but there's a problem • Name has already been taken*", field red, **Dismiss**.
    - Tax "abc" → "*Tax is not a number*"; the field resets to 0.00.
    - **Discount 150 accepted** (no upper bound).
  - **Out:** Save → list + toast "*Globex Corp has been added.*"
- **Edit client** (`/clients/:id/edit`)
  - Same form, plus a side panel "Remove '…' from your account" (blue) and "Archive '…'" (yellow). Neither was clicked.
  - Save → "*… has been updated.*"
- **New contact** (`/clients/:id/contacts/new`)
  - Note on page: "*No email is sent when adding a contact.*"
  - **Fields:** First*, Last*, Email, Invoice email default, Title, Office, Mobile, Fax.
  - **Errors:** empty → native required on first and last name. "hank@globex" → banner "*Email is not valid*".
  - **Out:** Save → "*Hank Scorpio has been added.*"

### 4.4 Tasks (Owner; the Member gets 404)

- **List** (`/tasks`)
  - Two sections: **Common tasks** ("automatically added to all new projects") and **Other tasks** (added manually). Seeded: Business Development, Design, Marketing, Programming, Project Management, and Vacation (other).
  - Row **Actions**: Edit / Add to all projects / Archive / Delete. Header: New task / Export / Actions.
- **Inline new/edit task**
  - **Fields:** Task name, Default billable rate, Assigned people (Edit), "billable by default", "common task", "Add this task to all existing projects". Save task / Cancel. Enter submits.
  - **Errors:**
    - Existing name "Design" → red toast "*Could not create task. Name has already been taken by an active or archived task.*" **The form closes and the input is lost.**
    - **Rate −5 accepted** and shown as −$5.00.
  - **Success toast:** "*QA Testing has been added. We'll start assigning it to all active projects now…*"

### 4.5 Projects

- **List** (`/projects?filter=active`)
  - **Owner:** New project, Actions, Import, Export, search; "Active projects (n)" ▾; Filter by client / manager; columns Budget / Spent / Remaining / Costs; row Actions.
  - **Member:** "Active projects (0)" with the empty state "*…once an Administrator or Manager makes that report available to you… review your assigned projects.*" Even the assigned project is not listed.
- **New project** (`/projects/new`; the Member gets **403 "Permission denied"**)
  - **Sections:** Client* (searchable, "+ New client"), Project name*, Project code (hint "Last project code: WR-01"), Dates, Tags, Billing currency, Notes (admins only), Permissions (report visible to admins/managers or to everyone).
  - **Project type** tabs:
    - Time & Materials: billable rate by No rate / Project / Person / Task; Budget by No budget / Total fees / Fees per task / Total hours / Hours per task / Hours per person; "resets every month"; email alert at 80%.
    - Fixed Fee.
    - Non-Billable.
  - **Other sections:** Tasks table (assigned, billable, "Add a task…"), Team (cost rate, manages), Invoice values (due date, PO, tax, discount).
  - **Branches:**
    - No billable rate → modal "**Heads up!** *Some settings might cause trouble: • Your project doesn't have a billable rate.*" with [Edit project] [Save project anyway].
    - Leaving with unsaved input → browser "Leave site?" dialog.
  - **Errors:**
    - No client → native required.
    - End date before start (the picker allows it) → custom bubble "*Start date must come before end date*".
    - Duplicate name under the same client → inline "*This project name is already used on another project.*" After "Save anyway" → banner "*Drats! We weren't able to save your changes.*"
  - **Out:** Save → detail page.
- **Project detail** (`/projects/:id`; **the Member gets 404 even when assigned**)
  - Header "[WR-01] Website Redesign", type badge, date range "(5 weeks left)", Edit project, **Actions** (Pin / Duplicate / Archive / Delete).
  - Charts: Project progress / Hours per week. Cards: Total hours, Budget remaining, Internal costs, Invoiced, Uninvoiced. Tabs: Tasks / Team / Invoices.

### 4.6 Team and invites (Owner; the Member gets 404)

- **Team** (`/team`)
  - Tabs Members / Assignments; Invite person, Actions, Import, Export; week selector; capacity summary.
  - Rows show hours, utilization and capacity. A pending invite shows "*Hasn't signed in yet. Resend invitation*".
- **Invite step 1** (`/people/new`)
  - **Fields:** First*, Last*, Work email*, Employee ID, Start date, Employee/Contractor, Roles, Departments, Capacity (0–60, default 35), Default billable rate, Cost rate → "Invite and continue".
  - **Errors:** empty → native required ×3. Existing email → "*Email address vishnu88varthan@gmail.com has already been taken.*"
  - **Success:** toast "*Success! We've emailed … an invitation to join your team.*" **The email goes out at step 1**, even if the rest of the wizard is skipped.
- **Step 2 — "What can Maya do in Harvest?"** Profile radios plus 50 checkboxes (see §1). Save permissions and continue / Skip.
- **Step 3 — "What project is … working on?"** Multi-select grouped by client, "Select all", "Also assign to all future projects", "Manages this project" → Assign projects / Skip. Toast "*… is ready to go!*"
- **Person profile** (`/people/:id/...`): tabs Basic info, Rates, Assigned projects, Assigned people, Permissions, Security. The Member's own profile adds Integrations and Notifications.

### 4.7 Timesheet (both roles)

- **Views:** Day `/time`, Week `/time/week/Y/M/D/userId`, Calendar `/time/calendar?timeframe=`.
  - Common controls: date nav, "+ Track time", AI prompt, "Copy from previous day/last week (projects only)" ▾, "**Submit week for approval**" ▾ (Submit day [disabled] / Submit custom range).
  - **Owner-only:** **Teammates** ▾ (search, employee list) opens a teammate's timesheet with the banner "*Thalaivan Ugam's timesheet — Changes will save to Thalaivan's timesheet. Resume editing your own timesheet*".
- **New/Edit entry modal**
  - **Fields:** Project (grouped, searchable, sort toggle), Task, Notes, Duration. Buttons: "Start timer" (when duration is empty) or "Save entry"; "Pull in a calendar event". Edit mode adds Update entry / Change date / Delete.
  - **Errors:**
    - "abc" duration → silently treated as empty, so a **timer starts** ("*Timer started.*").
    - **25:00 in one day accepted.**
  - **Loop:** timer running (sidebar clock) → Stop → Edit.
- **Week grid**
  - Cells autosave ("Saving…", "Saved.", "Last saved at …"). Row notes icon → "Add a note (optional)" popover. "+ Add row" modal.
  - **Errors:** "xyz" → silently dropped. "1.5" → 1:30.
- **Locks**
  - Cells for days already on an invoice, even a **draft**, are disabled, with a row lock and the tooltip "*Only an Administrator can unlock this entry*". Observed on the Member's Tue/Wed after the Owner drafted invoice #2 for 22–23 Sep.
  - An approved week is fully locked.
- **Submit (Member)**
  - Click → inline "*Submit this week's timesheet for approval? [Yes, submit timesheet] [Cancel]*" → toast "*Timesheet has been submitted for approval.*" and blue pill **Pending approval**. The button becomes "Resubmit week for approval".
  - **An empty week can be submitted.** The Member has **no withdraw/unsubmit option.**
  - Editing is still allowed while pending; new entries are tagged "Adjustment record" on the review page.

### 4.8 Expenses (both roles)

- **List** (`/expenses`)
  - **Owner tabs:** All expenses / Reimbursements (NEW) / Configure. **Member:** All expenses only.
  - "+ Track expenses" ▾: Create expenses from files (AI receipts) / Enter expenses manually. Owner also has Teammates ▾.
  - Grouped by week, with a "Submit for approval" link.
- **Inline form**
  - **Fields:** Date, Project, Category (Entertainment / Lodging / Meals / Mileage / Other / Transportation), Notes, Receipt file, Amount, "billable" (ticked by default), "I need reimbursing". Save expense / Cancel. **The form stays open after saving.**
  - Mileage swaps Amount for miles: 12 mi → $8.40 at $0.70/mi.
  - **Errors:**
    - Empty → native required on project, category and amount.
    - **−20 accepted** and saved as −$20.00.
    - Date 31/02/2026 → custom bubble "*Is not a valid date*".
  - **Edit:** inline form with Update expense / Cancel / Delete.
- **Configure > Categories:** Edit / Archive / Delete per row; Delete is disabled for categories in use. Sub-tabs Settings / Members / Payments.
- **Reimbursements:** wizard Overview → Payment method → Add members. **Stopped at the Overview**, because the next step needs bank or card details.

### 4.9 Approvals — the two-account loop (tested live)

- **Screen** (`/approvals?approval_status=submitted&group=user&period_start&period_end`; the Member gets 404)
  - Banner "*Time and expenses are still editable*", with Add auto-lock / Apply one-time manual lock.
  - Week selector; Status ▾ (Unsubmitted / Pending Approval / Approved); Group by ▾ (Person / Project / Client); filters.
  - KPI cards; rows with [Send email] [Approve]; footer "Approve visible timesheets and expenses".
  - Clicking a name opens the person detail: Time / Expenses tabs, List/Table, "View timesheet details".
- **There is no "Reject" button.** Rejecting in practice means **Send email**. The modal "*Send an email to Thalaivan Ugam — Add your comments… We'll send an email with your comments and a link to their Detailed Time and Detailed Expense reports.*" keeps [Send email] disabled until text is entered. Sending shows "*Email successfully sent.*" and **the status stays Pending**.

**Live sequence**

| # | Who | Action | Observed result |
|---|---|---|---|
| 1 | Member | Submitted an empty week | Pending approval |
| 2 | Member | Added Mon 3h and Thu 2h, plus a $30 Transportation expense (reimbursable); submitted from Expenses → `/daily/review` → Submit | Pending |
| 3 | Owner | Approvals row showed 5:00 / $30; sent the email "REJECTED (test): Thursday 2h is wrong…" | Status unchanged |
| 4 | Member | Changed Thu 2 → 4 and added a Monday note → "Resubmit week" → confirm → review → Submit | Toast "*Timesheet has been submitted for approval.*" |
| 5 | Owner | Row now 7:00 → **Approve** | Toast "*Approved.*"; row greys out with an **Undo** link |
| 6 | Owner | **Undo** | "*Approval withdrawn.*"; back to Pending |
| 7 | Owner | Approved again → Status: Approved shows [Withdraw] | — |
| 8 | Owner | Opened the Member's week: pill Approved, every day locked, "Withdraw approval" → "*This will unlock the entire timesheet. [Yes, unlock timesheet]*" | Back to **Pending approval** (not to draft), editable |
| 9 | Owner | Approved again (final state) | — |
| 10 | Member | Opened week and expenses | Pill **Approved**, all locked, no submit button; notifications show 3× "*Your timesheet was approved*". No notification for the feedback email or the withdrawals. |
| 11 | Owner | Opened notifications | 3× "*Thalaivan Ugam submitted a timesheet*", 1× "*…requested an expense reimbursement 30.0 USD · Taxi to client office*" |

- **Edge case:** "Approve visible timesheets and expenses" clicked with **zero** items → toast "*Approved.*"

### 4.10 Invoices (Owner; the Member gets 404)

- **Overview** (`/invoices`)
  - Tabs Overview / Recurring / Retainers / Uninvoiced / Configure. KPIs Total open and Total paid; 2026 bar chart; Open / All lists with Client / Project / period / Columns filters.
  - Seeded Draft #1 ($100, "Not sent yet") counts toward "Total open".
- **New invoice** (`/invoices/new_invoice`)
  1. **Choose client.** The dropdown groups clients into "Uninvoiced time & expenses" (Globex 4h $454) and "Nothing to invoice yet", plus "+ New client". **Continue** and **Save invoice draft** stay disabled until a client is chosen.
  2. **Type:** From time & expenses / From scratch; "Make recurring" toggle; project checklist with hours/expenses; currency.
     - Yellow, non-blocking warning: "*Some selected projects have unapproved or unsubmitted hours or expenses*". Expanding it shows "*Website Redesign — has unsubmitted hours, unsubmitted expenses*" with [Review approvals] [Disable warnings].
  3. **Auto line items:** Service, Design 4.00 × $100; Product, Meals $45.60; Product, Mileage $8.40. The right panel becomes **Appearance** (show columns, Templates, Look & feel).
  - **Errors:** Invoice ID "1" (used by the demo) → "*Number The invoice number '1' has already been assigned.*"
  - **Out:** Save → "*Invoice 2 has been created.*"
- **Invoice detail** (`/invoices/:id`)
  - Status Draft, DRAFT stamp; Record payment is **disabled** (tooltip "*Send or mark this invoice as sent before recording a payment.*"); View history.
  - **More actions:** Duplicate, Create recurring, Preview, Export as PDF, Export as UBL, Print, Delete.
  - **Stepper:** 1 Review invoice (Edit) → 2 Get paid (Credit Card / ACH / US Bank Transfer) → 3 **Send invoice (locked)**.
  - "Save and continue" → **gate modal** "*Action required — Choose how to get paid: Free or trialing plans require Stripe to be connected before invoices can be sent…*" [Upgrade your plan] [Connect to Stripe] "Don't continue with invoice".
  - **Sending is not possible on this trial, and was not attempted.**
- **Side effect:** once the draft existed, the Uninvoiced report dropped Globex to $0 and the Member's invoiced days became locked.

### 4.11 Estimates (Owner; the Member gets 404)

- **List** (`/estimates`): tabs Open / All estimates / Configure.
- **New estimate**
  - **Fields:** ID (auto), PO, Issue date, Subject, **Estimate for\*** (client), Tax (link), Discount, Currency, line items, Notes.
  - **Errors:** no client → native required. Unit price "abc" → silently becomes 0.00. A blank line item is dropped on save.
- **Detail**
  - Send estimate, Copy estimate link, Edit estimate, Actions (Draft: Mark as sent / Duplicate / Delete), Preview / PDF / Print, "Estimate worth", [Accepted] [Declined].
  - **Loop tested:** Accepted → stamp ACCEPTED plus a **New invoice** button; Actions becomes Duplicate / **Revert status** / Delete → confirm "*…convert the estimate from accepted status back to draft status?*" → Draft.
  - "Send estimate" was not clicked, because it emails the client.

### 4.12 Reports

- **Owner:** Time (Clients/Projects/Tasks/Teammates, billable donut, Save report, Export, Detailed report), Profitability (warns "*…missing rates. Add missing rates*"), Activity log (full audit with value diffs), Contractor (empty state), Invoicing (Invoiced / Payments received / Receivables / Uninvoiced), Saved reports. "+ New report" ▾: Custom report / Detailed time / Detailed expense.
- **Member:** Time, Activity log (own actions only) and Saved reports only. Profitability and Uninvoiced return 404; `/reports/detailed` loads.

### 4.13 Settings (Owner; the Member gets 404)

- **Billing:** subscription, payment, docs, and a "**Close account**" link (not touched).
- **Preferences:** read-only list plus "Edit preferences". Notable items: Timesheet deadline Fri 5pm, **Auto-lock** / **Auto-submit** (NEW), Time entry notes [Enterprise], Manager self-approval (Disabled), "Deleting invoice line items returns time to Uninvoiced".
- **Modules:** checklist; Timesheet approval and Activity log carry an **[Enterprise]** badge.
- **Other pages:** Sign in security, Import/Export, Bulk actions, Harvest AI (BETA).

### 4.14 Sign in / sign out

- **Sign out** → `id.getharvest.com/harvest/sign_in` with "*You have been signed out.*"
- **Sign-in options:** Google / email + password / SAML SSO / Forgot password.
- **Errors:** empty email and password → **no client-side check**; server says "*Incorrect email or password.*"
- **Google path:** Google "Choose an account" → back into the account.

### 4.15 Member direct-URL permission matrix (tested with the Member's session)

| Result | URLs |
|---|---|
| **404 "We can't find the page…"** | `/team`, `/clients`, `/clients/new`, `/tasks`, `/projects/:id` (even assigned), `/projects/:id/edit`, `/invoices`, `/invoices/:id`, `/invoices/new_invoice`, `/estimates`, `/estimates/:id`, `/approvals`, `/reports/uninvoiced`, `/reports/profitability`, `/company/account`, `/company/settings/preferences`, `/company/settings/integrations`, `/people/<owner>/edit`, `/people/new`, `/expenses/categories`, `/reimbursements` |
| **403 "Permission denied"** | `/projects/new` |
| **Silent redirect** | `/time/week/.../<ownerId>` → own week; `/overview` → `/time/week` |
| **200 allowed** | `/projects` (empty), `/expenses`, `/reports`, `/reports/activity`, `/reports/detailed`, `/company/settings/bulk_actions` |

---

## 5. Flowcharts

Built from §4. Branches, error self-loops and state loops are drawn as edges.

**Owner / Administrator** — [`flowcharts/flowchart_owner.png`](flowcharts/flowchart_owner.png)

![Owner flow](flowcharts/flowchart_owner.png)

**Member / Employee** — [`flowcharts/flowchart_member.png`](flowcharts/flowchart_member.png)

![Member flow](flowcharts/flowchart_member.png)

The Graphviz sources are included (`owner.dot`, `member.dot`) so they can be edited.

---

## 6. Edge cases and error states (observed live)

| # | Area | Action (what was tried) | Observed result | Kind | Shot |
|---|---|---|---|---|---|
| 1 | Onboarding invite | Email "not-an-email" | Native: "Please include an '@'… missing an '@'." | Native | 04b |
| 2 | Onboarding | Next with nothing chosen | Next disabled | Disabled CTA | 02 |
| 3 | New client | Empty name | Native "Please fill out this field." | Native | 12b |
| 4 | New client | Duplicate "Example Client" | Banner "Name has already been taken" | Server | 12c |
| 5 | New client | Tax "abc" | "Tax is not a number"; field reset to 0.00 | Server | 12d |
| 6 | New client | Discount 150% | **Accepted and saved** | Missing check | 14 |
| 7 | New contact | Empty first/last | Native required | Native | 15b |
| 8 | New contact | Email "hank@globex" | "Email is not valid" | Server | 15c |
| 9 | New task | Duplicate "Design" | Toast "Could not create task. Name has already been taken by an active or archived task."; **input lost** | Server | 19b |
| 10 | New task | Rate −5 | **Accepted**, shows −$5.00 | Missing check | 19c |
| 11 | New project | No client | Native required | Native | — |
| 12 | New project | End date before start | Picker allows it; submit blocked: "Start date must come before end date" | Client custom | 21g |
| 13 | New project | No billable rate | "Heads up!" warning modal (skippable) | Soft warning | 23 |
| 14 | New project | Duplicate name, same client | Inline + "Drats! We weren't able to save your changes." | Server | 23b |
| 15 | New project | Navigate away with input | Browser "Leave site?" | Guard | — |
| 16 | Invite | Empty fields | Native required ×3 | Native | — |
| 17 | Invite | Owner's email | "Email address … has already been taken." | Server | 25b |
| 18 | Time entry | Duration "abc" | **Silently ignored → timer started** | Silent | 29b/29c |
| 19 | Time entry | 25:00 in one day | **Accepted** | Missing check | 29e |
| 20 | Week grid | "xyz" in a cell | **Silently dropped** | Silent | 30b |
| 21 | Week grid | "1.5" | Converted to 1:30 | Normalisation | 30b |
| 22 | Week grid (Member) | Edit a day covered by a draft invoice | Cell disabled; "Only an Administrator can unlock this entry" | Lock | M10b |
| 23 | Submit (Member) | Submit an **empty** week | **Allowed** → Pending approval | Missing check | M08c |
| 24 | Submit (Member) | Look for withdraw | **None exists** (only resubmit / custom range) | Design gap | M08d |
| 25 | Expense | Empty save | Native required on project, category, amount | Native | — |
| 26 | Expense | Amount −20 | **Accepted** (−$20.00) | Missing check | 33c |
| 27 | Expense (Member) | Date 31/02/2026 | "Is not a valid date" | Client custom | M11b |
| 28 | Expense category | Delete an in-use category | Delete button disabled | Disabled CTA | 35 |
| 29 | Approvals | Approve visible with **0** items | Toast "Approved." (no-op) | Misleading | 45d |
| 30 | Approvals | Send email with empty text | Send disabled | Disabled CTA | 61 |
| 31 | Approvals | "Reject" | No reject control; email only, status unchanged | Design gap | 61c |
| 32 | Approvals | Approve → Undo | "Approval withdrawn." → Pending | Loop | 65b |
| 33 | Approvals | Withdraw an approved week | Confirm "This will unlock the entire timesheet." → Pending (not draft) | Loop | 64b/64c |
| 34 | Approved week (Member) | Try to edit | All cells locked; Track time / Add row disabled | Lock | 67 |
| 35 | Invoice | Continue without a client | Disabled | Disabled CTA | 38 |
| 36 | Invoice | Duplicate invoice # "1" | "Number The invoice number '1' has already been assigned." | Server | 38f |
| 37 | Invoice | Unsubmitted time selected | Yellow warning (non-blocking) | Soft warning | 38c |
| 38 | Invoice | Record payment on a Draft | Disabled; "Send or mark this invoice as sent before recording a payment." | Disabled CTA | 39e |
| 39 | Invoice | Go to Send on trial | **Gate:** Stripe or paid plan required | Plan gate | 39d |
| 40 | Estimate | No client | Native required | Native | 42b |
| 41 | Estimate | Unit price "abc" | Silently becomes 0.00 | Silent | 42c |
| 42 | Estimate | Accepted → Revert | Confirm modal → Draft | Loop | 43e/43f |
| 43 | Member profile | Empty first name | Native required | Native | M06b |
| 44 | Member permissions | Try to edit own permissions | Read-only; "Only an Administrator can edit your permissions." | Permission | M07 |
| 45 | Member direct URLs | 26 Owner-only URLs | 404 for most, 403 for `/projects/new`, silent redirect for the Owner's timesheet (§4.15) | Permission | M20–M23 |
| 46 | Sign in | Empty email and password | Server "Incorrect email or password." (no field hints) | Server | 50b |

**Not tested, and why**

| Item | Reason |
|---|---|
| Actually **sending** an invoice or estimate | Blocked on trial by the Stripe/paid-plan gate; also excluded by instruction (it would email the client). |
| Record payment / overpayment / "insufficient balance" | Needs a sent invoice (see above). |
| Reimbursement payout | Needs bank or card details. |
| Stripe connect, Upgrade / checkout | Financial; outside scope. |
| Delete client / project / task / invoice / estimate / person; Archive; Close account; Bulk delete | Destructive; not done without explicit sign-off. |
| Wrong-password sign-in, password reset | Would mean typing a password. |
| CSV import (clients / contacts / people) | Needs a file upload; not exercised. |
| Features on lower paid tiers (e.g. whether Timesheet approval disappears) | **Paid-tier only, not testable.** The trial unlocks Enterprise; approvals, Activity log and required notes carry an "Enterprise" badge. |
| Other profiles (Project Manager, People Admin, Accounting, Executive Manager) | Only the Member and Administrator ends were walked. |

---

## 7. Screen inventory

Numbered list of every distinct screen visited. Where a screen has several states, the representative shot is listed first.

1. Sign-up — `shots/00_signup.jpg`
2. Complete sign-up (Google) — `shots/01_complete_signup_google.jpg`
3. Welcome: team size — `shots/02_welcome_team_size.jpg`
4. Welcome: industry — `shots/03_welcome_industry.jpg`
5. Welcome: invite teammates — `shots/04_welcome_invite.jpg`
6. Welcome: marketing — `shots/05_welcome_marketing.jpg`
7. Welcome: goal — `shots/06_welcome_goal.jpg`
8. Billing-cycle modal — `shots/07_billing_cycle_modal.jpg`
9. "You're all set" modal — `shots/08_trial_welcome_modal.jpg`
10. Getting-started invoice editor — `shots/09_getting_started_invoice_editor.jpg`
11. Timesheet Day (Owner) — `shots/10_timesheet_day_empty.jpg`
12. Clients list — `shots/11_clients_list.jpg`
13. New client — `shots/12_client_new.jpg`
14. Edit client — `shots/14_client_edit.jpg`
15. New contact — `shots/15_contact_new.jpg`
16. Clients menus — `shots/17b_clients_actions_menu.jpg`
17. Tasks list — `shots/18_tasks_list.jpg`
18. Task inline form — `shots/19_task_new_inline.jpg`
19. Projects list — `shots/20_projects_list.jpg`
20. New project — `shots/21_project_new_top.jpg`
21. Project detail — `shots/22_project_detail.jpg`
22. "Heads up!" modal — `shots/23_project_heads_up_no_rate.jpg`
23. Team — `shots/24_team_members.jpg`
24. Invite person — `shots/25_invite_person_form.jpg`
25. Invite: permissions — `shots/26_invite_permissions_member.jpg`
26. Invite: assign projects — `shots/27_assign_projects_picker.jpg`
27. Person profile (assigned projects) — `shots/28_person_assigned_projects.jpg`
28. New time entry modal — `shots/29_time_new_entry_modal.jpg`
29. Timer running — `shots/29c_timer_running.jpg`
30. Edit time entry modal — `shots/29d_time_edit_modal.jpg`
31. Timesheet Week (Owner) — `shots/30_timesheet_week.jpg`
32. Teammates switcher — `shots/30d_teammates_switcher.jpg`
33. Timesheet Calendar — `shots/31_timesheet_calendar.jpg`
34. Expenses (Owner) — `shots/32_expenses_empty.jpg`
35. Expense form — `shots/33_expense_new_form.jpg`
36. Expenses list — `shots/34_expenses_list.jpg`
37. Expense categories config — `shots/35_expense_categories_config.jpg`
38. Reimbursements overview — `shots/36_reimbursements_overview.jpg`
39. Invoices overview — `shots/37b_invoices_overview.jpg`
40. New invoice builder — `shots/38d_invoice_line_items.jpg`
41. Invoice detail (Draft) — `shots/39_invoice_draft_detail.jpg`
42. Get-paid gate modal — `shots/39d_invoice_get_paid_gate.jpg`
43. Uninvoiced report — `shots/40_report_uninvoiced.jpg`
44. Estimates list — `shots/41_estimates_empty.jpg`
45. New estimate — `shots/42_estimate_new.jpg`
46. Estimate detail — `shots/43_estimate_draft_detail.jpg`
47. Estimate revert confirm — `shots/43e_estimate_revert_confirm.jpg`
48. Approvals (list) — `shots/60_owner_approvals_pending_thalaivan.jpg`
49. Approvals person detail — `shots/60c_owner_approvals_timesheet_details.jpg`
50. Send email modal — `shots/61b_owner_send_email_filled.jpg`
51. Approvals: Approved filter — `shots/63c_owner_approvals_status_approved.jpg`
52. Member's week seen by Owner — `shots/64_owner_views_member_week_approved_locked.jpg`
53. Withdraw confirm — `shots/64b_owner_withdraw_confirm.jpg`
54. Owner notifications — `shots/66_owner_notifications.jpg`
55. Reports: Time — `shots/46_reports_time.jpg`
56. Reports: Profitability — `shots/46c_reports_profitability.jpg`
57. Reports: Activity log — `shots/46d_reports_activity_log.jpg`
58. Reports: Contractor — `shots/46e_reports_contractor.jpg`
59. Detailed time report — `shots/46f_detailed_time_report.jpg`
60. Settings: Billing — `shots/47_settings_billing.jpg`
61. Settings: Preferences — `shots/47b_settings_preferences.jpg`
62. Settings: Modules — `shots/47c_settings_modules.jpg`
63. Sign in page — `shots/50_signed_out_sign_in.jpg`
64. Member: Timesheet Day — `shots/M01_member_timesheet_day.jpg`
65. Member: Projects — `shots/M02_member_projects_empty.jpg`
66. Member: profile (assigned projects) — `shots/M03_member_profile_assigned_projects.jpg`
67. Member: Expenses — `shots/M04_member_expenses_empty.jpg`
68. Member: Reports — `shots/M05_member_reports_empty.jpg`
69. Member: profile basic info — `shots/M06_member_profile_basic_info.jpg`
70. Member: permissions (read-only) — `shots/M07_member_permissions_readonly.jpg`
71. Member: Timesheet Week — `shots/M08_member_week_empty.jpg`
72. Member: Add row modal — `shots/M09_member_add_row_modal.jpg`
73. Member: expense form — `shots/M11_member_expense_form.jpg`
74. Member: review before submit — `shots/M12_member_review_before_submit.jpg`
75. Member: notes popover — `shots/62b_member_add_note_popover.jpg`
76. Member: Activity log — `shots/M14_member_activity_log.jpg`
77. Member: Bulk actions — `shots/M15_member_bulk_actions.jpg`
78. Member: approved week — `shots/67_member_week_approved_locked.jpg`
79. Member: notifications — `shots/67b_member_notifications_approved.jpg`
80. 404 page — `shots/M20_member_blocked_team_404.jpg`
81. 403 page — `shots/M21_member_projects_new_403.jpg`

All 168 captured images (every state, including error states) are in `shots/`. The filename prefix matches the order above: `00–50` are Owner screens, `60–67` are the round-trip, and `M##` are Member screens.

**Final account state:** Thalaivan's week of 21–27 Sep is **Approved** (7h + $30). Invoice #2 is a draft and was never sent. Estimate #1 is a draft. Maya's invite is still pending. Nothing was deleted.
