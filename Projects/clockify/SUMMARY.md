---
tags: project
status: paused
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: Clockify timesheets (monthly entry and review)
State: [[Projects/clockify/STATE]] · Log: [[Projects/clockify/LOG]]

## 1. What this project is
- Keep [[People/Vishnu]]'s [[Tools/Clockify]] timesheet complete and correct each month, before hours go to clients.
- **Review:** check the monthly report for wrong client tags, wrong work-type tags, spelling, duplicates and overlaps (August 2026).
- **Entry:** fill missing entries from real evidence in [[Tools/Slack]] and [[Tools/Google Calendar]] (September 2026: log stopped on 10 Sep, filled 11–30 Sep).
- Work done by Claude, for entry through [[Tools/Claude in Chrome]] in Vishnu's own logged-in Chrome.
- Clients billed by hours: [[Companies/HALLE]] and [[Companies/Future State]]. Workspace: [[Companies/araCreate Group]] (not sure).
- Merged project: office "clockify" (August review) + personal "clockify-entry" (September entry).
- Related, kept separate: [[Projects/clockify-automation-project/SUMMARY]] (automated calendar-to-Clockify tool).

## 2. Status now (as of 2026-10-02)
- August 2026: reviewed and corrected. 164.15 h total (was 167.40 h). Small leftovers open.
- September 2026: 173 entries for 11–30 Sep are in Clockify, rebuilt from Slack timings. All 225 September entries checked: no overlaps.
- September total 213:42 h, billable 83:31 h — before removing 14 Sep.
- Waiting on Vishnu: delete the 11 entries on 14 Sep (holiday) by hand. After that, about 205:50 h total, about 78:31 h billable.
- Last work: 1 Oct 2026, 09:21 IST.

## 3. Next steps
1. Vishnu deletes the 11 entries on Mon 14 Sep (Time Tracker → day bulk-edit → select all → Delete).
2. Claude checks 14 Sep is empty and gives final September numbers.
3. Decide whether to move 6:39 h of HALLE work (1–10 Sep) from #AC to HALLE (HALLE would become 71:11 h, billable 90:10 h).
4. Tag the 1:29 h of untagged #AC time (ACA website and deck planning).
5. Check with [[People/Shyam]]: Future State is 18:59 h, over the 12–13 h budget.
6. August leftovers: "seo"/"ceo" → "SEO"/"CEO" (11 Aug); check the 19 Aug 10:13–11:27 entry with both #dev and #work.
7. Decide the client mapping for #ARM and #DSA (INTERNAL?).
8. Next month: review/fill October with the Detailed export (if monthly).

## 4. Decisions
- 2026-09-01 — Use the **Detailed** Clockify export for reviews, not Summary — Summary has no dates/times. #decision
- 2026-09-01 — Description format `#project #tag #description`: first hashtag = client, second = work type (#design, #work, #meet, #learn); flag mismatches; unclear cases "needs review". #decision
- 2026-09-30 — Use Vishnu's real Chrome with Claude in Chrome — he was already logged in to Clockify. #decision
- 2026-09-30 — Weekdays 10am–7pm; bootcamp days are full days, weekends included; every day 7–8+ hours. #decision
- 2026-09-30 — Client billable work (HALLE, FST, SINOLINK) must be anchored to real Slack / Calendar evidence. #decision
- 2026-09-30 — Only real meetings get round times; other entries get natural durations. #decision
- 2026-09-30 — HALLE work split into small frontend/backend steps; every HALLE entry gets a `#nb` tag. #decision
- 2026-09-30 — Target hours: HLE ≈ 38 h, FST design ≈ 8–13 h, bootcamp dashboard 25 h (cut from 40), bootcamp training ≈ 24–33 h. #decision
- 2026-09-30 — FST budget is 12–13 h this month ([[People/Shyam]] on Slack, 23 Sep). #decision
- 2026-10-01 — Use the Clockify API (token from the logged-in page) for bulk add and edit — UI filling was too slow; Vishnu has no admin / CSV import access. (earlier: enter one by one) #decision
- 2026-10-01 — #AC entries go to the real #AC project, not NIBORRA [NOA]. #decision
- 2026-10-01 — HALLE server migration starts 28 Sep; earlier HALLE time = feedback widget, screenshot engine, VPS planning. #decision
- 2026-10-01 — Bootcamp dashboard work starts 16 Sep; bootcamp Days 1–9 run 18–26 Sep from 9:00. #decision
- 2026-10-01 — Mix projects within each day instead of long single-project blocks. #decision
- 2026-10-01 — No entry starts or ends on :00 seconds; no touching or overlapping entries (3–27 s gaps). #decision
- 2026-10-01 — 14 Sep is a holiday, no entries; Vishnu deletes them himself (Claude does not do permanent deletes). #decision
- 2026-10-01 — HALLE and FUTURE STATE are the billable projects. #decision

## 5. Timeline
- 2026-10-02 — clockify-entry moved into viOS.
- 2026-10-01 09:21 — 14 Sep is a holiday; Vishnu to delete its 11 entries himself. Waiting since.
- 2026-10-01 08:59 — September report: 213:42 h total, 83:31 h billable.
- 2026-10-01 08:50 — All 225 Sep entries checked; 1 overlap on 9 Sep fixed; 3–27 s gaps added.
- 2026-10-01 08:39 — Random seconds added to all 173 entries.
- 2026-10-01 — Rebuilt to 173 Slack-matched entries; bootcamp Days 1–9 on 18–26 Sep; FST ≈ 12.9 h.
- 2026-10-01 — Fixed #AC mapping, HALLE migration start (28 Sep), dashboard start (16 Sep).
- 2026-10-01 — 143 entries imported for 11–30 Sep via the Clockify API in the browser.
- 2026-09-30 — UI filler did ~10 of 86 entries; JS / Selenium / Playwright tries failed.
- 2026-09-30 — 143-entry Python draft built, sent as CSV, approved.
- 2026-09-30 — Analysed 194 entries (3 Aug–10 Sep): 7 projects, `#work` ~60%, `#meet` ~25%, 27% billable. Pulled Slack and Calendar history.
- 2026-09-30 20:30 — September entry work started.
- 2026-09-01 — August corrected file re-checked: all tag mismatches fixed, duplicate removed (143 → 142 rows), 2 overlaps fixed. Review saved.
- 2026-09-01 — August Detailed report reviewed: 23 tag mismatches, 1 duplicate (7 Aug #dsa), 2 overlaps (12 Aug, 27 Aug), 22 spelling fixes.
- 2026-09-01 — August Summary report reviewed (76 entries, 12 must-fix items).

## 6. Key facts
- **Client tags:** #fst = [[Companies/Future State]] [#FST]; #hle/#halle = [[Companies/HALLE]] [#HLE]; #arn = [[Companies/Aarini]] [#ARN]; #arm = ARM; #mmm = MALTE MARTEN METHOD; #dsa = DSA; #ac = internal ops + college bootcamp; #noa = [[Companies/Niborra]] [NOA]; #slk = [[Companies/Sinolink]] [SLK].
- **Clockify clients (August):** araCreate India (FST, SLK), araCreate Group (NOA), INTERNAL (AC, ARM, DSA).
- **Clockify projects:** `NIBORRA [NOA]`, `AARINI [#ARN]`, `HALLE [#HLE]`, `FUTURE STATE [#FST]`, `SINOLINK [SLK]`, `#AC`, `#DSA`. FST sub-projects: REGEN-WEB, FST-WEBSITE-SW.
- **Extra tags:** #dev (not one of the 4 standard work tags); #nb on HALLE entries.
- **People:**
  - [[People/Vishnu]] — owner.
  - [[People/Shyam]] — manager / PM; set FST 12–13 h budget.
  - [[People/Jakob Silbermann]] — HALLE client contact (widget, VPS, server migration).
  - [[People/Shay Lynch]] and [[People/Meiraj]] — Future State contacts (#fst huddles).
  - [[People/Achim]] — Sinolink contact (logo, Impressum, privacy policy).
  - [[People/Aravinth Panch]] — lead of #ARM (maybe [[People/Dinesh Aravinth]], not sure).
  - [[People/Kishor Arjunan]], [[People/Basith]], [[People/Rathees]] — in Slack history, roles not sure.
- **Clockify API method:** base `https://global.api.clockify.me`, `X-Auth-Token` from the page's own login (no key saved). Calls: `/v1/user`; `/v1/workspaces/{ws}/user/{uid}/time-entries?start=&end=&page-size=1000`; `PUT /v1/workspaces/{ws}/time-entries/{id}` with 120 ms delay.
- **Tips:** shared boundary times shift by the same seconds; check overlaps across all pairs in a day, not only neighbours.
- **Tools:** [[Tools/Clockify]], [[Tools/Claude]], [[Tools/Claude in Chrome]], [[Tools/Slack]], [[Tools/Google Calendar]], [[Tools/Python]]. No GitHub repo or server.
- **Chats index:** [[Projects/clockify/chats/INDEX]]
- **Related:** [[Projects/clockify-automation-project/SUMMARY]] (kept separate), [[Projects/timer/SUMMARY]].

## 7. Files and documents
- `clockify/Aug2026_Report_Review.md` — August review (office Claude project).
- `Clockify_Time_Report_Detailed_01_08_2026–31_08_2026.csv` — August source export (uploaded in chat).
- CSV draft of 143 entries (11–30 Sep) — sent in chat, not saved here.
- Clockify Detailed report (Aug–Sep 2026) — source for the pattern analysis.

## 8. Open questions and problems
- 14 Sep entries (11) not yet deleted — needs Vishnu.
- 6:39 h of HALLE work (1–10 Sep) under #AC — move or not?
- 1:29 h of #AC time has no tag.
- FST is 18:59 h, over the 12–13 h budget — tell Shyam or trim?
- Mapping: #ARM = AARINI [#ARN] or its own project? #AC vs NIBORRA [NOA]? Should #ARM and #DSA stay under INTERNAL?
- Is #dev an approved work tag?
- Which college ran the bootcamp?
- Is this a monthly task (October next)?

## 9. All chats in this project
- [[Projects/clockify/chats/2026-09-01 Clockify time report review|Clockify time report review]] — 2026-09-01 (office)
- [[Projects/clockify/chats/2026-09-30 Clockify entries analysis|Clockify entries analysis]] — 2026-09-30 (personal)
- [[Projects/clockify/chats/2026-09-30 Google Chrome clicky entries|Google Chrome clicky entries]] — 2026-09-30 (personal)
- [[Projects/clockify/chats/2026-10-02 Moving project to viOS (10)|Moving project to viOS (10)]] — 2026-10-02 (personal)
