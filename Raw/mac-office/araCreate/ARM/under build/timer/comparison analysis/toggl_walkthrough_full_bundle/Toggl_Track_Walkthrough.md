# Toggl Track (Toggl 2.0): End-to-End Product Walkthrough

Captured live on **23 Sep 2026** in a real browser on a new organization, using two real accounts on the 30-day Premium trial (no card).

**Accounts used**

- **Owner/Admin:** `vishnu88varthan@gmail.com`. Created the organization "Vishnu88varthan's organization".
- **Member/Employee:** `thalaivan.ugam@gmail.com` ("Thalaivan Ugam"). Invited as Member and signed in with Google.
- **Side accounts:** `vishnu88varthan+member@gmail.com` (first test invite, joined without ever setting a password) and `vishnu88varthan+manager@gmail.com` (Manager invite, never accepted).

Every image in this document is a real screenshot of the live app (`shots/` folder, 143 files). File names are cited as `[NNN]`.

> **Important context:** new sign-ups on toggl.com/track no longer land in classic Toggl Track (`track.toggl.com`). They land in **"Toggl 2.0"** at `focus.toggl.com`, which merges Track (time tracking), Plan (tasks and timeline) and Focus. The login page states: "You're logging in to Toggl – a new tool that brings planning, time tracking, and reporting together. Looking for Toggl Track? Log in here." `[085]`. This document covers what a **new customer actually gets today**. The classic `track.toggl.com` UI was not explored.

---

## 1. Overview

**What it does.** Toggl 2.0 is a time tracker with planning built in:

- **Timer and manual entry:** a live timer, plus calendar, list and weekly timesheet views.
- **Project and client setup:** clients, billable rates, estimates, fixed fees, tags, and task planning with timeline and milestones.
- **Approvals:** weekly timesheet submission and approval.
- **Reporting:** summary, time logs, utilization, workload and profitability, with PDF/CSV/XLSX export and shared reports.
- **Integrations:** Google Calendar and Outlook auto-tracking, Jira and Asana sync, and a universal CSV importer.

**Who it's for.** Agencies, consultancies and internal teams that bill or cost time by client and project, and want managers to sign off on weekly hours.

**Plans seen in the app.** A free plan (up to 5 members after the trial) and Starter / Premium. Premium features carry a ★ in the sidebar and tabs. The trial unlocks "Billable rates, labor cost and profitability; Timeline, milestones and unlimited teammates; Utilization and workload reports" `[010]`. Billing is license-based ("Toggl 2.0 now uses license-based billing") `[078]`. **Time Off / Time accounting** is a separate paid add-on ($2/user/month, its own 30-day trial) `[070]`. It was **not tested**.

**Roles** (from the invite role picker `[045]`):

| Role | Description shown in app |
|---|---|
| Organization Owner | Creator; "The organization needs another admin before ownership can be transferred" `[072]` |
| Tool admin | Can access all data in the workspace |
| Manager | Can manage teams and projects but not users, integrations, or workspace data |
| Member | Can manage their own tasks, planned time, and time tracking |
| Time tracker | Can manage their own planned time and time tracking, but not tasks or workspace structure |
| Guest | An external party with access limited to a specific project |

There is also a separate **Organization access → Admin** toggle on each member's page `[128]`. This document covers **Owner/Admin** and **Member** in depth. Manager, Time tracker and Guest were **not tested live**: the Manager invite `[083]` was never accepted, because that needed a third Google login.

**What each role sees (observed):**

| Area | Owner/Admin | Member |
|---|---|---|
| Sidebar | Timer, Reports, Projects, Tasks, Timeline★, Members, Approvals★, Time off★, Upgrade, Admin settings `[011]` | Timer, Reports, Projects, Tasks, Timeline★, Time off★ only; no Members, Approvals, Admin settings or Upgrade `[099]` |
| Projects | All projects, New project, Billable/Rate/Fixed fee columns `[029]` | Only shared projects, no create, no billing columns `[100]` |
| Reports tabs | Summary, Utilization, Workload, Profitability, Time logs, Time off, Time accounting, Shared `[063]` | Summary, Workload, Time logs, Time accounting, Shared; no money figures `[125]` |
| Timesheet | Plain timesheet (Owner not in any approval setup) `[138]` | Approval week Mon–Sun with status chip and Submit/Withdraw/Resubmit `[103]` |

---

## 2. Employee/Member walkthrough

These are the screens in the order a first-time invited employee hits them. The account is `thalaivan.ugam@gmail.com`, using Google login.

1. **Invite email.** Subject "You have been invited to join Toggl" from support@toggl.com, with an **Accept invitation** button linking to `accounts.toggl.com/invitation?code=…`. It was read live from Gmail. No screenshot, because it's an email and not app UI.
2. **Invitation page.** "Welcome to Toggl 2.0! You have been invited to join the organization Vishnu88varthan's organization". Buttons: **Accept and join**, plus the Terms text. `[088]`
   ![](shots/088_invite_accept_page.jpg)
3. **Set your password** (email/password path). "You'll use this password to log in to all Toggl tools." `[089]`
   ![](shots/089_invite_set_password.jpg)
4. **Login page** (for later sessions). Options: email/password, Google, Apple, Passkey, SSO. `[085]`
   ![](shots/085_login_page_toggl2.jpg)
5. **Terms screen** for a first Google login. "Welcome to Toggl 2.0! Clear plans, powered by time tracking data", with **Agree and continue** / Cancel. `[087]`
   ![](shots/087_google_signup_terms_second_user.jpg)
6. **Forced onboarding, 3 steps.** The invited member still has to complete these: goal, calendar, first project. Even a direct URL to the employer's workspace redirects back here. `[096]`
   ![](shots/096_member_forced_onboarding.jpg)
7. **The member's OWN organization and a trial modal.** Onboarding creates a personal organization ("Thalaivan Ugam's organization", marked Default) and shows its own trial modal. `[097]`
   ![](shots/097_member_own_org_trial_modal.jpg)
8. **Org switcher** (top-left). Lists both organizations, plus Create organization. The member picks the employer's organization. `[098]`
   ![](shots/098_member_org_switcher.jpg)
9. **Timer home** (calendar view) in the employer's organization, with the reduced sidebar. `[099]`
   ![](shots/099_member_timer_home.jpg)
10. **Projects:** only the shared project "Mobile App", with no create button. `[100]`
    ![](shots/100_member_projects_list.jpg)
11. **Timesheet view, NOT SUBMITTED.** Header "Approval week · Sep 21 – 27, 2026", "Approval week runs Mon–Sun. Why?", and **Submit for approval**. `[103]`
    ![](shots/103_member_timesheet_not_submitted.jpg)
12. **Add row.** The row starts as "No task / Without project" with day cells Mon–Sun and a **Save all** button. `[108]`
    ![](shots/108_member_add_row.jpg)
13. **Project picker:** grouped by client, with **no "Create a project"** option. `[109]`
    ![](shots/109_member_project_picker_no_create.jpg)
14. **Hours entered.** `[111]`
    ![](shots/111_member_timesheet_filled_unsaved.jpg)
15. **Submitted, PENDING REVIEW.** The grid is read-only and the button changes to **Withdraw submission**. `[112]`
    ![](shots/112_member_submitted_3h_pending.jpg)
16. **Timer locked while pending.** Tooltip: "Today's timesheet is submitted or approved, so you can't start tracking". `[105]`
    ![](shots/105_member_timer_locked_tooltip.jpg)
17. **Withdraw confirmation.** `[106]`, followed by the result, back to NOT SUBMITTED. `[107]`
    ![](shots/106_member_withdraw_confirm.jpg)
    ![](shots/107_member_withdrawn_not_submitted.jpg)
18. **CHANGES REQUESTED** (after the approver sends it back). Cells are editable again and the button changes to **Resubmit**. `[120]`. Hovering the chip shows the approver's comment. `[121]`
    ![](shots/120_member_changes_requested_resubmit.jpg)
    ![](shots/121_member_approver_comment_tooltip.png)
19. **Notifications panel:** "You're all caught up". No in-app alert arrived for the change request. `[122]`
    ![](shots/122_member_notifications_empty.jpg)
20. **Edited to 7h** `[123]`, then **Resubmitted**. `[124]`
    ![](shots/123_member_edited_7h.jpg)
    ![](shots/124_member_resubmitted_pending.jpg)
21. **APPROVED:** read-only, and the member has no withdraw option. `[139]`
    ![](shots/139_member_timesheet_approved_locked.jpg)
22. **Manual time entry from the timer bar.** Clicking `0:00:00` opens Start / Stop / date fields `[141]`, and the timer shows the duration with a ✓ button. `[142]`
    ![](shots/141_member_manual_entry_popover.jpg)
    ![](shots/142_member_manual_entry_filled.jpg)
23. **Member reports:** Summary showing logged time only. `[125]`
    ![](shots/125_member_reports_summary.jpg)
24. **Profile menu → Log out.** It's the same menu as the Owner's. `[084]`

The Member uses the same timer, list, calendar and focus-mode screens as the Owner. Those are documented with screenshots in §3.

---

## 3. Admin/Owner walkthrough

These are the screens in first-time order, using the Owner account.

**Sign-up and onboarding**

1. **Marketing site.** "Try for free" / "Login", with a banner saying Track and Plan remain available for existing users. `[001]`
   ![](shots/001_marketing_home.jpg)
2. **Terms / Welcome to Toggl Track:** Start tracking / Cancel. `[002]`
   ![](shots/002_terms_welcome.jpg)
3. **App loading shell.** It briefly shows the old Track sidebar, including Clients, Invoices and Integrations, before redirecting to Toggl 2.0. `[003]`
   ![](shots/003_app_loading.jpg)
4. **Onboarding step 1: "What will you mainly use Toggl for?"** Single-select. Continue stays disabled until an option is picked `[004]`, then enables. `[005]`
   ![](shots/004_onboarding_goal_empty.jpg)
   ![](shots/005_onboarding_goal_selected.jpg)
5. **Onboarding step 2: "Log time from your meetings and events"**. Google Calendar / Microsoft Outlook Connect, an Auto-track toggle (on by default) with a tooltip, Back and Continue. `[006]` `[007]`
   ![](shots/006_onboarding_calendar.jpg)
   ![](shots/007_onboarding_autotrack_tooltip.png)
6. **Onboarding step 3: "Create your first project"**. The name is required. `[008]` `[009]`
   ![](shots/008_onboarding_first_project_empty.jpg)
   ![](shots/009_onboarding_first_project_filled.jpg)
7. **Trial modal:** "Your trial starts now … No credit card needed" → Start trial. `[010]`
   ![](shots/010_trial_modal.jpg)

**Timer and manual time entry**

8. **Timer home, calendar view** (week W39), with the Get started checklist. `[011]`
   ![](shots/011_timer_calendar_empty.jpg)
9. **Timer running with no description.** Allowed. Coachmark: "Type what you're focusing on". `[012]`
   ![](shots/012_timer_running_no_description.jpg)
10. **Project picker** `[013]` and **tag creation** ("Create 'design' tag"). `[014]`
    ![](shots/013_timer_project_picker.jpg)
    ![](shots/014_timer_create_tag.jpg)
11. **Running timer with project and tag.** `[015]` **Timer mode selector:** count up, countdown, pomodoro. `[016]` **More menu:** Enter Focus mode, Discard logged time. `[017]`
    ![](shots/015_timer_running_with_project_tag.jpg)
    ![](shots/016_timer_mode_selector.jpg)
    ![](shots/017_timer_more_menu.jpg)
12. **Focus mode:** full-screen timer. `[018]` **Stop:** shows a coachmark for Reports. `[019]`
    ![](shots/018_focus_mode.jpg)
    ![](shots/019_timer_stopped_reports_coachmark.jpg)
13. **List view** `[020]`. The row menu offers Duplicate, Go to project, Copy description and Delete `[021]`. Duplicated entries are grouped `[022]`, and times can be edited inline. `[023]`
    ![](shots/020_timer_list_view.jpg)
    ![](shots/021_entry_row_menu.jpg)
    ![](shots/022_entry_duplicated_grouped.jpg)
    ![](shots/023_entry_inline_time_edit.jpg)
14. **Timesheet view** (manual weekly entry) `[026]`, with an entry saved. `[028]`
    ![](shots/026_timesheet_view.jpg)
    ![](shots/028_timesheet_manual_entry_saved.jpg)
15. **Calendar with entries.** `[090]`
    ![](shots/090_timer_calendar_with_entries.jpg)

**Project and client setup**

16. **Projects list.** `[029]` **New Project modal:** Name, Draft, Client, Privacy, Invite members, More options, Start from template★. `[030]`
    ![](shots/029_projects_list.jpg)
    ![](shots/030_new_project_modal.jpg)
17. **Project page:** Overview, Tasks, Board, Timeline, Dashboard and Members tabs, plus right-side toggles for Recurring, Estimate, Billable and Fixed fee. `[032]`
    ![](shots/032_project_detail_overview.jpg)
18. **Client picker, empty.** `[034]` Inline "Create client" `[035]` shows a "Client created" toast. `[036]`
    ![](shots/034_project_client_picker_empty.jpg)
    ![](shots/035_project_create_client_inline.jpg)
    ![](shots/036_client_created_toast.jpg)
    There is **no standalone Clients page** in Toggl 2.0. Clients are created and assigned from the project page.
19. **Billable on** `[039]`, then the **Add project rate** modal (hourly rate, currency, start date) `[040]`, with the rate saved. `[042]`
    ![](shots/039_project_billable_on.jpg)
    ![](shots/040_add_project_rate_modal.jpg)
    ![](shots/042_project_rate_saved.jpg)
20. **Private → Shared confirmation** `[081]`, then the shared project. `[082]`
    ![](shots/081_share_project_confirm.jpg)
    ![](shots/082_project_shared.jpg)

**Members**

21. **Members list** (Owner only) `[043]`. The **Invite members** modal `[044]` has a role picker. `[045]`
    ![](shots/043_members_list_owner_only.jpg)
    ![](shots/044_invite_members_modal.jpg)
    ![](shots/045_invite_role_options.jpg)
22. **Ready to send** `[048]`. After sending, the new row shows as pending (clock icon). `[049]`
    ![](shots/048_invite_ready.jpg)
    ![](shots/049_invite_sent_pending_no_role.jpg)
23. **Inline role dropdown** `[052]`. The pending member's menu offers Edit and Revoke invitation. `[053]`
    ![](shots/052_member_role_dropdown.jpg)
    ![](shots/053_pending_member_menu_revoke.jpg)
24. **Members list with Manager and Member rows.** `[083]` `[092]`
    ![](shots/092_members_thalaivan_member_role.jpg)
25. **Member detail page:** name, role, Admin toggle, billable rates, labor costs, working hours. `[128]` `[129]`
    ![](shots/128_admin_member_detail_page.jpg)
    ![](shots/129_admin_member_working_hours.jpg)

**Approvals**

26. **Approvals empty state** `[054]`, then the **New timesheet setup** modal: members, approvers, Weekly (Monthly is "Coming soon"), start date, and an email reminder day and time. `[055]` `[057]` `[058]` The setup succeeds. `[059]`
    ![](shots/054_approvals_empty_state.jpg)
    ![](shots/055_new_timesheet_setup_modal.jpg)
    ![](shots/058_timesheet_setup_filled.jpg)
    ![](shots/059_approvals_setup_success.jpg)
27. **Not submitted tab** `[060]`. The ⚙ Manage timesheet setups menu offers Edit setup, Manage members, Discontinue and Delete `[061]`. The row menu offers Approve and View in reports. `[062]`
    ![](shots/060_approvals_not_submitted_tab.jpg)
    ![](shots/061_manage_timesheet_setups_menu.png)
28. **Manage members → Add member.** `[093]` `[094]` `[095]`
    ![](shots/095_timesheet_setup_member_added.jpg)
29. **Pending review row** `[113]`. Its menu offers Approve, Request changes and View in reports. `[114]`
    ![](shots/113_admin_pending_review_row.jpg)
    ![](shots/114_admin_pending_row_menu.jpg)
30. **Request timesheet changes** modal with an optional note `[115]` `[116]`. The timesheet moves to the **Changes requested** tab `[117]`, whose row menu offers Approve, Edit request and View in reports. `[118]`
    ![](shots/116_admin_request_changes_note.jpg)
    ![](shots/117_admin_changes_requested_tab.jpg)
31. **Timesheet review page** `[119]`, then the resubmission at 7h. `[127]`
    ![](shots/119_admin_timesheet_review_detail.jpg)
    ![](shots/127_admin_review_resubmitted_7h.jpg)
32. **Approved** `[130]`, then **Withdraw approval** with confirmation `[131]`, back to Not submitted `[132]`, approved again `[133]`. The timesheet appears on the Approved tab. `[134]`
    ![](shots/130_admin_timesheet_approved.jpg)
    ![](shots/131_admin_withdraw_approval_confirm.jpg)
    ![](shots/134_admin_approved_tab.jpg)

**Reporting**

33. **Summary** `[063]`, **Export menu** (PDF/CSV/XLSX) `[064]`, **Filter menu** `[065]`, **Time logs** `[066]` `[135]`, **Utilization** `[067]`, **Workload** `[068]`, **Profitability** with a missing-data warning `[069]`, **Time accounting** add-on gate `[070]`, **Shared reports**. `[071]`
    ![](shots/063_reports_summary.jpg)
    ![](shots/064_reports_export_menu.jpg)
    ![](shots/065_reports_filter_menu.jpg)
    ![](shots/066_reports_time_logs.jpg)
    ![](shots/067_reports_utilization.jpg)
    ![](shots/068_reports_workload.jpg)
    ![](shots/069_reports_profitability_missing_data.jpg)
    ![](shots/070_reports_time_accounting_addon_gate.jpg)
    ![](shots/071_reports_shared_empty.jpg)

**Admin settings and integrations**

34. **Admin settings → Overview:** organization name, logo, owner, 2FA, AI features. `[072]`
    ![](shots/072_admin_settings_overview.jpg)
35. **Integrations:** Jira and Asana `[074]`. Jira Connect redirects to Atlassian OAuth `[075]`. **Data import:** the universal CSV importer. `[076]`
    ![](shots/074_integrations_jira_asana.jpg)
    ![](shots/075_integration_jira_oauth_login.jpg)
    ![](shots/076_data_import.jpg)
36. **Access** `[077]`, **Billing** `[078]`, **Tags**. `[079]`
    ![](shots/077_settings_access.jpg)
    ![](shots/078_settings_billing.jpg)
    ![](shots/079_settings_tags.jpg)
37. **Profile menu** (Settings, Log out) `[084]`, and the **Owner's own timesheet**, which has no approval chrome. `[138]`
    ![](shots/084_profile_menu_logout.jpg)
    ![](shots/138_owner_timesheet_no_approval_workflow.jpg)

---

## 4. The flow in deep detail

For each screen: how you get in, how you leave, the branches and loops, the validation observed, and role differences. "Observed" means triggered live.

### 4.1 Auth and entry

| Screen | Enter | Exit / branches | Validation and states (observed) | Role diff |
|---|---|---|---|---|
| Marketing `[001]` | toggl.com/track | Try for free → sign-up; Login → login | – | – |
| Login `[085]` | Log out; `/focus/login/` | Email+password → app; Google → Google "Choose an account" (lists every Google account in the browser) → app; Apple / Passkey / SSO; Sign up; Reset password; links to classic Track and Plan logins | Empty submit → "Please enter a password"; "notanemail" → "Please enter a valid email" `[086]`. Wrong password **not tested** (I don't enter passwords). The Google button sometimes did nothing on a second click; the direct URL `/focus/login/google/` always worked | Same for all roles |
| Terms `[002]` `[087]` | First login of a new identity | Start tracking / Agree and continue → onboarding; Cancel → back out | Continuing also opts you into marketing emails (stated in the fine print) | Shown again for each new identity |
| Invitation `[088]` | Link in invite email | Accept and join → Set password (email account) or straight into the app | **If a different account is already logged in, accepting silently swaps the session to the invited identity** ("Hi (secret removed)") without warning. The invite is bound to the email address | Member-only entry |
| Set password `[089]` | After accepting an email-only invite | Save new password → app; Log out (top right) | **The member became Active in the org even though no password was ever set** | – |

### 4.2 Onboarding (every new identity, including invited members)

| Step | Enter | Exit / loops | Validation (observed) |
|---|---|---|---|
| 1 Goal `[004]` `[005]` | After terms | Continue → step 2 | Continue disabled until a card is picked; single-select (picking another card moves the check) |
| 2 Calendar `[006]` | Step 1 | Back → step 1; Continue (no connect needed) → step 3; Connect → Google/Microsoft OAuth (**not tested**, it would link a real calendar) | Auto-track toggle is ON by default |
| 3 First project `[008]` | Step 2 | Back → step 2; Continue → app | Empty or whitespace-only name → Continue stays disabled (no message) |
| Trial modal `[010]` `[097]` | First app load | Start trial → app; Esc also closes it | Trial started with no card |

**Role difference.** An invited Member is also forced through onboarding. It creates a **personal organization**, which becomes their Default, and a first project in it `[096]` `[097]`. They must use the org switcher `[098]` to reach the employer's organization. A direct URL to the employer workspace redirects to `/onboarding` until onboarding is finished.

### 4.3 Timer (all roles)

- **Enter:** sidebar Timer, or right after login. **Leave:** any sidebar item.
- **Views:** Calendar, Split, List and Timesheet toggles; a "5 Days" range; week navigation; a Logged progress bar; "View reports".
- **Start/stop.** An empty description is allowed and saved as "(No description)" `[012]`. Project picker: search, grouping by client, "Create a project" (**Owner only**; the Member has no create option `[109]` `[110]`). Tag picker: search, create inline. Billable is `$`. Mode: count up, countdown, pomodoro `[016]`. More: Focus mode (F) `[018]` and Discard logged time.
- **Manual entry.** Click the `0:00:00` display to get Start / Stop / date `[141]`; the play button turns into ✓ `[142]`. You can also type in timesheet cells `[026]` or edit rows inline in the list.
- **List row loops.** Duplicate (the copy is grouped under a count badge) `[022]`, Continue (play), Go to project, Copy description, Delete (**not clicked**, destructive).
- **Inline edits (observed):** see §6. Invalid text reverts silently; a start after the end is saved as an overnight entry; there is no maximum duration.
- **Member-only locks.** While today's approval week is Pending or Approved, the timer and manual ✓ are disabled, with the tooltip "Today's timesheet is submitted or approved, so you can't start tracking" `[105]` `[143]`.

### 4.4 Projects and clients

| Screen | Enter | Exit / branches | Validation (observed) | Role diff |
|---|---|---|---|---|
| Projects list `[029]` | Sidebar | New project; + ADD PROJECT; row → project page; filters Active / Filters / Group by / Sort by | – | Member: shared projects only, no create, no Billable / Rate / Fixed fee columns `[100]` |
| New Project modal `[030]` | New project | Create project (Enter) → project page; × → list; More options; Start from template★ | Empty name → red "A name is required here." `[031]`. **Duplicate name → allowed** `[033]` | Owner/Admin |
| Project page `[032]` | Row click or create | ‹ back; tabs Overview / Tasks / Board / Timeline / Dashboard / Members; ⋮ menu; Invite | Title renames inline | Member: private project URL → silently bounced to the list `[101]` |
| Client picker `[034]` | "Choose client" on the project page | Pick; Create "X" client (Enter); Clear | Case variant "acme corp" vs "Acme Corp" → allowed `[037]`. **Exact duplicate → generic red toast "Something went wrong, please try again later"** `[038]` | Owner/Admin (Access settings control whether Members can manage clients `[077]`) |
| Billable rate `[040]` | Billable toggle → New rate | Add rate from today; × | "-50" → button stays disabled, no message `[041]`; "abc" not accepted; 75 → "75 USD · Sep 23 2026 – Ongoing" `[042]` | Owner/Admin |
| Privacy | Private / Shared toggle | Shared → confirm "Share project with team?… All assigned admins will be permanently removed" → Cancel / Share with team `[081]` | – | Premium★ |

### 4.5 Members (Owner/Admin only)

- **Members list `[043]`.** People / Teams tabs, Filters, search, and columns Member, Role, Time off, Rate, Cost, Working hours. Clicking a row opens the member detail page `[128]`.
- **Invite modal `[044]`.**
  - Email chips are created on Tab (not Enter). Invalid text stays as raw text `[047]`.
  - The Invite button stays disabled for an invalid email `[046]` or no role picked `[050]`.
  - Re-inviting an existing email shows a red banner: "One of the users you are trying to invite already exists in the Organization." `[051]`
  - Success toast: "Members successfully invited…" `[049]`.
- **Role bug (reproduced 2 of 3 times).** The new row showed **"No role assigned"** even though Member was picked `[049]` `[091]`. The fix is the inline role dropdown `[052]`.
- **Pending row menu:** Edit, Revoke invitation `[053]` (**not clicked**, destructive).
- **Member direct URL** `/members` → silently shows Projects `[102]`.

### 4.6 Approvals: the full live round-trip

**Setup (Owner).**

- Approvals → New timesheet setup `[055]`.
- Submitting empty shows "Select at least one member." and "Select at least one approver." `[056]`.
- Success modal `[059]`.
- Manage via ⚙ `[061]` → Manage members → Add member `[094]`. Even pending invitees can be added.

**State machine (all transitions exercised live, both sides):**

```
NOT SUBMITTED --member: Submit--> PENDING REVIEW
PENDING REVIEW --member: Withdraw (confirm)--> NOT SUBMITTED             [106][107]
PENDING REVIEW --approver: Request changes (+note)--> CHANGES REQUESTED   [115-117][120]
CHANGES REQUESTED --member: edit + Resubmit--> PENDING REVIEW            [123][124]
PENDING REVIEW --approver: Approve--> APPROVED                           [130]
APPROVED --approver: Withdraw approval (confirm)--> NOT SUBMITTED        [131][132]
NOT SUBMITTED --approver: Approve directly--> APPROVED                   [133]
```

**Member side:**

- The chip and the primary button change with the state: Submit for approval → Withdraw submission → Resubmit.
- While a week is Pending or Approved, its grid and the timer are locked.
- The approver's note is only visible by hovering the CHANGES REQUESTED chip `[121]`. No in-app notification arrived `[122]`; emails are "sent" per the dialogs but couldn't be checked in the member's inbox.

**Approver side:**

- Tabs: Pending review / Changes requested / Approved / Not submitted.
- A row opens the Timesheet review page `[119]`, which shows total vs expected hours, billable %, the member's timezone, and one row per entry.
- "Edit expected hours" links to the member detail page.

**Gaps found:**

- An empty 0h week can be submitted `[104]`.
- A future week can be submitted `[140]`.
- Approvers can approve a week that was never submitted `[062]` `[133]`.
- **An admin can edit an approved entry in Reports → Time logs. The approved total changes from 7h to 8h and the status stays Approved, with no warning** `[136]` `[137]`.
- A Tue cell still being edited when the member clicked Submit was **silently lost** (only Mon 3h was submitted) `[112]`.

### 4.7 Reports

- **Owner:** all tabs `[063]`–`[071]`. Filters: Member, Team, Client, Project, Task, Status, Tag, Member Tag, Priority, Billable status, Time log description `[065]`. Export PDF / CSV / XLSX★ `[064]` (**not downloaded**; downloading needs your permission). Time logs rows are editable inline `[135]`.
- **Member:** Summary (logged time only), Workload, Time logs, Time accounting, Shared `[125]`. The direct URL `/reports/profitability` bounces to Summary `[126]`.

### 4.8 Admin settings

- **Enter:** sidebar Admin settings. **Leave:** "Exit settings".
- **Sections:**
  - Organization: Overview, Holiday calendar, Billing, Explore plans.
  - Workspace setup: Access, Audit log★, Tags, Statuses, Required fields★, Custom fields★, PDF reports★.
  - Tracking & rates: Alerts & reminders, Targets★, Billable rates★, Default currency.
  - Connections: Data import, Integrations★.
- **Validation:** an empty organization name shows "Organization name can't be empty." `[073]`. A duplicate tag shows the inline error "A tag with this name already exists" with Save disabled `[080]`.
- **Integrations:** the Jira Connect button goes to Atlassian OAuth (scopes read:jira-work, read:jira-user, manage:jira-webhook, offline_access) `[075]`. I stopped there because there was no third-party account to grant. An unknown `?section=` value falls back to Overview.
- **Member:** direct `/settings` and `/approvals` URLs bounce to Timer (no error screen).

---

## 5. Flowcharts

These are built from §4 and show branches, loops and error self-loops. The Mermaid sources are in `flow/member.mmd` and `flow/admin.mmd`.

### 5.1 Member/Employee

![Member flowchart](flow/member_flowchart.png)

### 5.2 Admin/Owner

![Admin flowchart](flow/admin_flowchart.png)

---

## 6. Edge cases and error states

| # | Area | Action (what was actually done) | Observed result | Evidence |
|---|---|---|---|---|
| 1 | Onboarding | Continue with no goal picked | Button disabled, no message | [004] |
| 2 | Onboarding | Project name empty or whitespace | Continue disabled | [008] |
| 3 | Timer | Start with empty description | Allowed; saved as "(No description)" | [012] |
| 4 | Entry edit | Start time "abc" | Silently reverts to previous value | [023] |
| 5 | Entry edit | Start 5:30 PM after end 4:27 PM | **No error**; saved as an overnight entry of 22h 56m | [024] |
| 6 | Entry edit | Duration "1000:00:00" | **Accepted**, 1000h entry, no cap | [025] |
| 7 | Timesheet | Type "xyz" in a day cell | Shows 0:00:00, no entry created | [027] |
| 8 | Project | Create with empty name | Red "A name is required here." | [031] |
| 9 | Project | Duplicate name "Website Redesign" | **Allowed**; two identical projects | [033] |
| 10 | Client | Case variant "acme corp" | Allowed | [037] |
| 11 | Client | Exact duplicate "Acme Corp" | Generic red toast "Something went wrong, please try again later" | [038] |
| 12 | Rate | Hourly rate "-50" / "abc" | Button disabled / input refused, no message | [041] |
| 13 | Tag | Duplicate "design" in Settings | Inline "A tag with this name already exists", Save disabled | [080] |
| 14 | Org | Clear organization name | "Organization name can't be empty." | [073] |
| 15 | Invite | Invalid email "not-an-email" / "bad@" | Not turned into a chip; Invite disabled; no message | [046] [047] |
| 16 | Invite | No role picked | Invite disabled | [050] |
| 17 | Invite | Re-invite existing email | Red banner "One of the users you are trying to invite already exists in the Organization." | [051] |
| 18 | Invite | Invite with role Member | **Row showed "No role assigned"** (2 of 3 invites) | [049] [091] |
| 19 | Invite | Open invite link while logged in as a different account | **Session silently switched** to the invited identity | [089] |
| 20 | Invite | Accept, never set password | Member still became Active | [092] |
| 21 | Approvals setup | Submit with no member/approver | "Select at least one member." / "Select at least one approver." | [056] |
| 22 | Approvals | Member submits empty 0h week | **Accepted** | [104] |
| 23 | Approvals | Member submits a future week | **Accepted** (then withdrawn) | [140] |
| 24 | Approvals | Submit while a cell is still being edited | Unsaved cell value **silently dropped** | [111] [112] |
| 25 | Approvals | Member starts timer / adds manual entry in a pending or approved week | Blocked; tooltip "Today's timesheet is submitted or approved, so you can't start tracking" | [105] [143] |
| 26 | Approvals | Approver approves a Not-submitted week | Allowed | [062] [133] |
| 27 | Approvals | Admin edits an approved entry in Reports → Time logs | **Saved; approved total 7h→8h; status stays Approved; no notice** | [136] [137] |
| 28 | Approvals | Change request note | Visible to the member only via chip hover; no in-app notification | [121] [122] |
| 29 | Permissions | Member opens private project URL | Silently bounced to Projects list | [101] |
| 30 | Permissions | Member opens `/members` | Projects list shown, no error | (same as [101]) |
| 31 | Permissions | Member opens `/settings`, `/approvals` | Redirected to Timer, no error | [102] |
| 32 | Permissions | Member opens `/reports/profitability` | Redirected to Summary | [126] |
| 33 | Permissions | Member tries to create a project from the picker | No create option | [110] |
| 34 | Login | Empty / invalid email | "Please enter a password" / "Please enter a valid email" | [086] |
| 35 | Onboarding | Invited member opens employer workspace URL before onboarding | Redirected to `/onboarding` | [096] |
| 36 | Settings | Unknown `?section=` param | Falls back to Overview | [072] |
| 37 | Quota | Free plan "up to 5 members after the trial" | **Not testable**: trial allows unlimited; a 6th invite during trial wouldn't hit the cap | [044] |
| 38 | Login | Wrong password | **Not tested**: I don't type passwords | – |
| 39 | Destructive | Delete entry / project, Revoke invite, Discontinue / Delete setup, Discard logged time | **Not tested**: destructive, not approved | [021] [053] [061] |
| 40 | Paid add-on | Time Off / Time accounting module | **Paid add-on, not tested** | [070] |
| 41 | Integrations | Google / Outlook calendar, Jira, Asana OAuth | **Not completed**: would grant third-party access; stopped at the OAuth screen | [006] [075] |
| 42 | Export | PDF / CSV / XLSX download | **Not downloaded** (file download not authorised) | [064] |
| 43 | Roles | Manager, Time tracker, Guest behaviour | **Not tested live**: Manager invite never accepted | [083] |

**Loops exercised live (verified at the end of the run):**

- **Reject → edit → resubmit.** Request changes with a note → member sees CHANGES REQUESTED → edits to 7h → Resubmit → Pending → Approve.
- **Withdraw (member).** Submit → Withdraw → Not submitted. Done twice, once on the future week.
- **Withdraw (approver).** Approve → Withdraw approval → Not submitted → Approve.
- **Error correction.** Bad duration → corrected; 22h overnight entry → corrected via the duration field; empty required fields → filled; wrong invite role → fixed inline.

---

## 7. Screen inventory

Every screenshot captured, in order. `E` marks edge-case / error screens.

### 001: Marketing home

![Marketing home](shots/001_marketing_home.jpg)

### 002: Terms welcome

![Terms welcome](shots/002_terms_welcome.jpg)

### 003: App loading

![App loading](shots/003_app_loading.jpg)

### 004: Onboarding goal empty

![Onboarding goal empty](shots/004_onboarding_goal_empty.jpg)

### 005: Onboarding goal selected

![Onboarding goal selected](shots/005_onboarding_goal_selected.jpg)

### 006: Onboarding calendar

![Onboarding calendar](shots/006_onboarding_calendar.jpg)

### 007: Onboarding autotrack tooltip

![Onboarding autotrack tooltip](shots/007_onboarding_autotrack_tooltip.png)

### 008: Onboarding first project empty

![Onboarding first project empty](shots/008_onboarding_first_project_empty.jpg)

### 009: Onboarding first project filled

![Onboarding first project filled](shots/009_onboarding_first_project_filled.jpg)

### 010: Trial modal

![Trial modal](shots/010_trial_modal.jpg)

### 011: Timer calendar empty

![Timer calendar empty](shots/011_timer_calendar_empty.jpg)

### 012: Timer running no description

![Timer running no description](shots/012_timer_running_no_description.jpg)

### 013: Timer project picker

![Timer project picker](shots/013_timer_project_picker.jpg)

### 014: Timer create tag

![Timer create tag](shots/014_timer_create_tag.jpg)

### 015: Timer running with project tag

![Timer running with project tag](shots/015_timer_running_with_project_tag.jpg)

### 016: Timer mode selector

![Timer mode selector](shots/016_timer_mode_selector.jpg)

### 017: Timer more menu

![Timer more menu](shots/017_timer_more_menu.jpg)

### 018: Focus mode

![Focus mode](shots/018_focus_mode.jpg)

### 019: Timer stopped reports coachmark

![Timer stopped reports coachmark](shots/019_timer_stopped_reports_coachmark.jpg)

### 020: Timer list view

![Timer list view](shots/020_timer_list_view.jpg)

### 021: Entry row menu

![Entry row menu](shots/021_entry_row_menu.jpg)

### 022: Entry duplicated grouped

![Entry duplicated grouped](shots/022_entry_duplicated_grouped.jpg)

### 023: Entry inline time edit

![Entry inline time edit](shots/023_entry_inline_time_edit.jpg)

### 024 (E): Start after end 22h

![Start after end 22h](shots/024_edge_start_after_end_22h.jpg)

### 025 (E): Duration 1000h accepted

![Duration 1000h accepted](shots/025_edge_duration_1000h_accepted.jpg)

### 026: Timesheet view

![Timesheet view](shots/026_timesheet_view.jpg)

### 027 (E): Timesheet invalid xyz

![Timesheet invalid xyz](shots/027_edge_timesheet_invalid_xyz.png)

### 028: Timesheet manual entry saved

![Timesheet manual entry saved](shots/028_timesheet_manual_entry_saved.jpg)

### 029: Projects list

![Projects list](shots/029_projects_list.jpg)

### 030: New project modal

![New project modal](shots/030_new_project_modal.jpg)

### 031 (E): Project name required

![Project name required](shots/031_edge_project_name_required.jpg)

### 032: Project detail overview

![Project detail overview](shots/032_project_detail_overview.jpg)

### 033 (E): Duplicate project allowed

![Duplicate project allowed](shots/033_edge_duplicate_project_allowed.jpg)

### 034: Project client picker empty

![Project client picker empty](shots/034_project_client_picker_empty.jpg)

### 035: Project create client inline

![Project create client inline](shots/035_project_create_client_inline.jpg)

### 036: Client created toast

![Client created toast](shots/036_client_created_toast.jpg)

### 037 (E): Client case variant allowed

![Client case variant allowed](shots/037_edge_client_case_variant_allowed.jpg)

### 038 (E): Duplicate client generic error

![Duplicate client generic error](shots/038_edge_duplicate_client_generic_error.jpg)

### 039: Project billable on

![Project billable on](shots/039_project_billable_on.jpg)

### 040: Add project rate modal

![Add project rate modal](shots/040_add_project_rate_modal.jpg)

### 041 (E): Negative rate blocked

![Negative rate blocked](shots/041_edge_negative_rate_blocked.png)

### 042: Project rate saved

![Project rate saved](shots/042_project_rate_saved.jpg)

### 043: Members list owner only

![Members list owner only](shots/043_members_list_owner_only.jpg)

### 044: Invite members modal

![Invite members modal](shots/044_invite_members_modal.jpg)

### 045: Invite role options

![Invite role options](shots/045_invite_role_options.jpg)

### 046 (E): Invite invalid email disabled

![Invite invalid email disabled](shots/046_edge_invite_invalid_email_disabled.jpg)

### 047 (E): Invite invalid not chipped

![Invite invalid not chipped](shots/047_edge_invite_invalid_not_chipped.png)

### 048: Invite ready

![Invite ready](shots/048_invite_ready.jpg)

### 049: Invite sent pending no role

![Invite sent pending no role](shots/049_invite_sent_pending_no_role.jpg)

### 050 (E): Invite role required

![Invite role required](shots/050_edge_invite_role_required.jpg)

### 051 (E): Invite duplicate email

![Invite duplicate email](shots/051_edge_invite_duplicate_email.jpg)

### 052: Member role dropdown

![Member role dropdown](shots/052_member_role_dropdown.jpg)

### 053: Pending member menu revoke

![Pending member menu revoke](shots/053_pending_member_menu_revoke.jpg)

### 054: Approvals empty state

![Approvals empty state](shots/054_approvals_empty_state.jpg)

### 055: New timesheet setup modal

![New timesheet setup modal](shots/055_new_timesheet_setup_modal.jpg)

### 056 (E): Timesheet setup required fields

![Timesheet setup required fields](shots/056_edge_timesheet_setup_required_fields.jpg)

### 057: Timesheet setup member picker

![Timesheet setup member picker](shots/057_timesheet_setup_member_picker.jpg)

### 058: Timesheet setup filled

![Timesheet setup filled](shots/058_timesheet_setup_filled.jpg)

### 059: Approvals setup success

![Approvals setup success](shots/059_approvals_setup_success.jpg)

### 060: Approvals not submitted tab

![Approvals not submitted tab](shots/060_approvals_not_submitted_tab.jpg)

### 061: Manage timesheet setups menu

![Manage timesheet setups menu](shots/061_manage_timesheet_setups_menu.png)

### 062: Approvals row menu admin

![Approvals row menu admin](shots/062_approvals_row_menu_admin.jpg)

### 063: Reports summary

![Reports summary](shots/063_reports_summary.jpg)

### 064: Reports export menu

![Reports export menu](shots/064_reports_export_menu.jpg)

### 065: Reports filter menu

![Reports filter menu](shots/065_reports_filter_menu.jpg)

### 066: Reports time logs

![Reports time logs](shots/066_reports_time_logs.jpg)

### 067: Reports utilization

![Reports utilization](shots/067_reports_utilization.jpg)

### 068: Reports workload

![Reports workload](shots/068_reports_workload.jpg)

### 069: Reports profitability missing data

![Reports profitability missing data](shots/069_reports_profitability_missing_data.jpg)

### 070: Reports time accounting addon gate

![Reports time accounting addon gate](shots/070_reports_time_accounting_addon_gate.jpg)

### 071: Reports shared empty

![Reports shared empty](shots/071_reports_shared_empty.jpg)

### 072: Admin settings overview

![Admin settings overview](shots/072_admin_settings_overview.jpg)

### 073 (E): Org name empty

![Org name empty](shots/073_edge_org_name_empty.jpg)

### 074: Integrations jira asana

![Integrations jira asana](shots/074_integrations_jira_asana.jpg)

### 075: Integration jira oauth login

![Integration jira oauth login](shots/075_integration_jira_oauth_login.jpg)

### 076: Data import

![Data import](shots/076_data_import.jpg)

### 077: Settings access

![Settings access](shots/077_settings_access.jpg)

### 078: Settings billing

![Settings billing](shots/078_settings_billing.jpg)

### 079: Settings tags

![Settings tags](shots/079_settings_tags.jpg)

### 080 (E): Duplicate tag inline error

![Duplicate tag inline error](shots/080_edge_duplicate_tag_inline_error.jpg)

### 081: Share project confirm

![Share project confirm](shots/081_share_project_confirm.jpg)

### 082: Project shared

![Project shared](shots/082_project_shared.jpg)

### 083: Members three pending

![Members three pending](shots/083_members_three_pending.jpg)

### 084: Profile menu logout

![Profile menu logout](shots/084_profile_menu_logout.jpg)

### 085: Login page toggl2

![Login page toggl2](shots/085_login_page_toggl2.jpg)

### 086 (E): Login invalid email empty password

![Login invalid email empty password](shots/086_edge_login_invalid_email_empty_password.jpg)

### 087: Google signup terms second user

![Google signup terms second user](shots/087_google_signup_terms_second_user.jpg)

### 088: Invite accept page

![Invite accept page](shots/088_invite_accept_page.jpg)

### 089: Invite set password

![Invite set password](shots/089_invite_set_password.jpg)

### 090: Timer calendar with entries

![Timer calendar with entries](shots/090_timer_calendar_with_entries.jpg)

### 091 (E): Invite role not applied again

![Invite role not applied again](shots/091_edge_invite_role_not_applied_again.jpg)

### 092: Members thalaivan member role

![Members thalaivan member role](shots/092_members_thalaivan_member_role.jpg)

### 093: Timesheet setup manage members

![Timesheet setup manage members](shots/093_timesheet_setup_manage_members.jpg)

### 094: Timesheet setup add member picker

![Timesheet setup add member picker](shots/094_timesheet_setup_add_member_picker.jpg)

### 095: Timesheet setup member added

![Timesheet setup member added](shots/095_timesheet_setup_member_added.jpg)

### 096: Member forced onboarding

![Member forced onboarding](shots/096_member_forced_onboarding.jpg)

### 097: Member own org trial modal

![Member own org trial modal](shots/097_member_own_org_trial_modal.jpg)

### 098: Member org switcher

![Member org switcher](shots/098_member_org_switcher.jpg)

### 099: Member timer home

![Member timer home](shots/099_member_timer_home.jpg)

### 100: Member projects list

![Member projects list](shots/100_member_projects_list.jpg)

### 101 (E): Member private project url redirect

![Member private project url redirect](shots/101_edge_member_private_project_url_redirect.jpg)

### 102 (E): Member settings url redirect

![Member settings url redirect](shots/102_edge_member_settings_url_redirect.jpg)

### 103: Member timesheet not submitted

![Member timesheet not submitted](shots/103_member_timesheet_not_submitted.jpg)

### 104 (E): Member empty timesheet submitted

![Member empty timesheet submitted](shots/104_edge_member_empty_timesheet_submitted.jpg)

### 105: Member timer locked tooltip

![Member timer locked tooltip](shots/105_member_timer_locked_tooltip.jpg)

### 106: Member withdraw confirm

![Member withdraw confirm](shots/106_member_withdraw_confirm.jpg)

### 107: Member withdrawn not submitted

![Member withdrawn not submitted](shots/107_member_withdrawn_not_submitted.jpg)

### 108: Member add row

![Member add row](shots/108_member_add_row.jpg)

### 109: Member project picker no create

![Member project picker no create](shots/109_member_project_picker_no_create.jpg)

### 110 (E): Member cannot create project

![Member cannot create project](shots/110_edge_member_cannot_create_project.png)

### 111: Member timesheet filled unsaved

![Member timesheet filled unsaved](shots/111_member_timesheet_filled_unsaved.jpg)

### 112: Member submitted 3h pending

![Member submitted 3h pending](shots/112_member_submitted_3h_pending.jpg)

### 113: Admin pending review row

![Admin pending review row](shots/113_admin_pending_review_row.jpg)

### 114: Admin pending row menu

![Admin pending row menu](shots/114_admin_pending_row_menu.jpg)

### 115: Admin request changes modal

![Admin request changes modal](shots/115_admin_request_changes_modal.jpg)

### 116: Admin request changes note

![Admin request changes note](shots/116_admin_request_changes_note.jpg)

### 117: Admin changes requested tab

![Admin changes requested tab](shots/117_admin_changes_requested_tab.jpg)

### 118: Admin changes requested menu

![Admin changes requested menu](shots/118_admin_changes_requested_menu.jpg)

### 119: Admin timesheet review detail

![Admin timesheet review detail](shots/119_admin_timesheet_review_detail.jpg)

### 120: Member changes requested resubmit

![Member changes requested resubmit](shots/120_member_changes_requested_resubmit.jpg)

### 121: Member approver comment tooltip

![Member approver comment tooltip](shots/121_member_approver_comment_tooltip.png)

### 122: Member notifications empty

![Member notifications empty](shots/122_member_notifications_empty.jpg)

### 123: Member edited 7h

![Member edited 7h](shots/123_member_edited_7h.jpg)

### 124: Member resubmitted pending

![Member resubmitted pending](shots/124_member_resubmitted_pending.jpg)

### 125: Member reports summary

![Member reports summary](shots/125_member_reports_summary.jpg)

### 126 (E): Member profitability url redirect

![Member profitability url redirect](shots/126_edge_member_profitability_url_redirect.jpg)

### 127: Admin review resubmitted 7h

![Admin review resubmitted 7h](shots/127_admin_review_resubmitted_7h.jpg)

### 128: Admin member detail page

![Admin member detail page](shots/128_admin_member_detail_page.jpg)

### 129: Admin member working hours

![Admin member working hours](shots/129_admin_member_working_hours.jpg)

### 130: Admin timesheet approved

![Admin timesheet approved](shots/130_admin_timesheet_approved.jpg)

### 131: Admin withdraw approval confirm

![Admin withdraw approval confirm](shots/131_admin_withdraw_approval_confirm.jpg)

### 132: Admin approval withdrawn not submitted

![Admin approval withdrawn not submitted](shots/132_admin_approval_withdrawn_not_submitted.jpg)

### 133: Admin approved unsubmitted

![Admin approved unsubmitted](shots/133_admin_approved_unsubmitted.jpg)

### 134: Admin approved tab

![Admin approved tab](shots/134_admin_approved_tab.jpg)

### 135: Admin time logs all members

![Admin time logs all members](shots/135_admin_time_logs_all_members.jpg)

### 136 (E): Admin edits approved entry

![Admin edits approved entry](shots/136_edge_admin_edits_approved_entry.jpg)

### 137 (E): Approved total changed 8h

![Approved total changed 8h](shots/137_edge_approved_total_changed_8h.jpg)

### 138: Owner timesheet no approval workflow

![Owner timesheet no approval workflow](shots/138_owner_timesheet_no_approval_workflow.jpg)

### 139: Member timesheet approved locked

![Member timesheet approved locked](shots/139_member_timesheet_approved_locked.jpg)

### 140 (E): Member future week submitted

![Member future week submitted](shots/140_edge_member_future_week_submitted.jpg)

### 141: Member manual entry popover

![Member manual entry popover](shots/141_member_manual_entry_popover.jpg)

### 142: Member manual entry filled

![Member manual entry filled](shots/142_member_manual_entry_filled.jpg)

### 143 (E): Member manual entry blocked approved day

![Member manual entry blocked approved day](shots/143_edge_member_manual_entry_blocked_approved_day.jpg)
