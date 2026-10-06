---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: ac-iso-27001-isms

## 1. What this project is
- ISO/IEC 27001:2022 certification (ISMS) for [[Companies/araCreate India]] (ACI), an IT services and software company in Erode / Puducherry.
- [[People/Vishnu]] is the ISMS Coordinator, UX Designer and Security Lead. He is the single point of contact for the auditor.
- Work covers: closing Stage 1 audit findings, organising all ISMS documents in [[Tools/Google Drive]], getting ready for the Stage 2 audit, and planning yearly ISMS work after certification.
- Side topic: Vishnu's own career path from Internal Auditor to Lead Auditor.

## 2. Status now (as of 2026-10-06)
- Stage 1 audit done on 14 May 2026. Two minor NCs (GR/01, GR/02) were closed; the auditor accepted the Corrective Action Report (CAR) in early June 2026.
- Stage 2 certification audit took place in June 2026 (confirmed in office notes). The result is not recorded (not sure if ACI is certified).
- Drive folder clean-up (ACI-ISMS, folders 0-Master to 7-archive) was mostly done; some old folders were still pending.
- Last chat: 16 June 2026. No chat activity since then.
- Status set to "active" because post-certification work (risk review, internal audit) is planned for 2026 (not sure — may be paused or done).

## 3. Next steps
- Confirm the Stage 2 audit result and any new findings.
- Fix the SoA reference-document column (each control points to the next control's procedure).
- Fix SoA conflict for A.5.6 and A.5.7 (applicable vs not applicable on two sheets).
- Add Board of Directors to the org chart (OBS-01).
- Keep one primary risk register (three overlap now); retire duplicate Context and Interested Parties files.
- Dispose of 40+ legacy equipment items with certificates ([[People/Kishor Arjunan]]).
- Deliver training TRN-002 to TRN-005 ([[People/Pradeepa]] / Vishnu).
- 31 Jul 2026: Management Review Meeting #2 ([[People/Navaneethan Kandaraj]]) — not sure if held.
- 30 Sep 2026: quarterly risk review (Kishor + Vishnu) — not sure if done.
- 15 Dec 2026: first annual internal audit (all 93 controls).
- Dec 2026: compliance (legal) register review.
- 15 Jul 2027: Surveillance audit 1. 14 Jul 2028: recertification.

## 4. Decisions
- 2026-05 (not sure) — Mark A.8.30 (outsourced development) in the SoA with proper justification, since ACI does no outsourced development — to close NC GR/02 #decision
- 2026-05 (not sure) — Add version control to the legal and regulatory register — to close NC GR/01 #decision
- 2026-06-02 — Treat OBS-01 (Board missing from org chart) as non-blocking; submit CAR without waiting for it — it is an observation, not an NC #decision
- 2026-06-02 — Edit Drive files in place with [[Tools/Claude in Chrome]], not download/re-upload — Vishnu's preference, faster #decision
- 2026-06-02 — Use short, simple, formal emails to the auditor #decision
- 2026-06-13 — Organise Drive by document type (Policies, Procedures, Records, Evidence) with a Master Register mapping clauses, not by clause folders — ISO documents cross many clauses, clause nesting causes duplicates #decision
- 2026-06-13 — Use type-prefixed IDs: POL-xxx, PRO-xxx, REC-xxx, EVD-xxx, MAS-xxx #decision
- 2026-06-13 — Build a fresh ACI-ISMS folder (0-Master, 1-Clauses, 2-Policies, 3-Procedures, 4-Annex-A, 5-Evidence, 6-Audit, later 7-archive) and move old files into it #decision
- 2026-06-15 — Change-request-form stays in 3-Procedures; technical evidence goes in 5-Evidence #decision
- 2026-06-16 — 90 of 93 Annex A controls applicable; A.8.8, A.8.11, A.8.30 excluded per SoA (not sure — A.8.8 and A.8.11 exclusion looks unusual) #decision
- 2026-06-16 — Keep the Interested Parties Matrix version with Annex A references; retire the older Context of the Organization file #decision
- 2026-06-16 — Vishnu marks document status in the checklist himself (green/red); Claude does not update the file #decision


## 5. Timeline
- 2026-03-20 — ISMS policies approved (date from the master document list).
- 2026-03 — Vishnu got ISO 27001 Internal Auditor certificate from [[Companies/QHSE Solutions]].
- 2026-05-14 — Stage 1 audit by [[People/James Jaganathan]]. Findings: GR/01 (Cl 4.1, legal register missing), GR/02 (Cl 6.1.3, A.8.30 no justification), OBS-01 (Cl 5.1, Board missing in org chart).
- 2026-05-27 — Management Review Meeting verified the corrective actions.
- 2026-06-02 — NC evidence folders cleaned (ACI-NC-001, ACI-NC-002, Common-Evidence, OBS-01). CAR sent to James Jaganathan and [[People/Sreedhar M K]]. CAR accepted. Reply sent asking for Stage 2 schedule.
- 2026-06-12 — Stage 2 audit said to be in 2 days. Made a prep checklist and the Word guide ACI-Stage2-Audit-Readiness-Guide.docx.
- 2026-06-13 — Designed folder structure + Master Register. Built new ACI-ISMS Drive folder and moved files from old clause 4, 5, 6, 8 folders.
- 2026-06-15 — Stage 2 readiness checklist (Excel, 4 sheets) from ISO vendor meeting notes. Folder-by-folder audit-ready checklist. Post-cert plan and 2026–2028 audit roadmap.
- 2026-06-16 — Folder-mapped document checklist + Annex A evidence checklist (Excel, 3 sheets). Reviewed many ISMS docs. Found SoA reference column shift. Lead Auditor career report and demand forecast. Apps Script for bulk Drive-to-PDF export.
- 2026-06 — Stage 2 certification audit held (exact date and result unknown).

## 6. Key facts
- Company: [[Companies/araCreate India]] (AraCreate India Private Limited, ACI). Part of [[Companies/araCreate Group]] (not sure).
- Standard: ISO/IEC 27001:2022, 93 Annex A controls (families ORG / PPL / PHY / TECH).
- People:
  - [[People/Vishnu]] — Vishnuvarthan Venkatapathy, ISMS Coordinator, UX Designer, Security Lead.
  - [[People/Navaneethan Kandaraj]] — Managing Director, ISMS Owner, approves policies, chairs MRM.
  - [[People/James Jaganathan]] — external auditor (Stage 1 and Stage 2).
  - [[People/Sreedhar M K]] — copied on CAR email, from audit side (not sure of role).
  - [[People/Pradeepa]] — HR, training and awareness owner.
  - [[People/Kishor Arjunan]] — "Kishor", Software Engineer / IT and infrastructure, risk review (not sure same as existing Kishore page).
  - [[People/Aravinth Panch]] — "Aravinth Panch", Director (not sure same person).
  - [[People/Joshirahul Vadivel]] — APM ("Rahul").
  - [[People/Vasanthapriya]] — Director / Finance.
  - [[People/Gobinath]] — Software Developer.
  - [[People/Gowthami]] — Moderator.
  - [[People/Shiva]] — independent internal auditor.
  - [[People/Ariv]] — copied on auditor email (role not sure).
- Other companies: external ISO vendor / certification body (name not sure); [[Companies/QHSE Solutions]] (Vishnu's auditor course); [[Companies/PECB]] (Lead Auditor path).
- Tools: [[Tools/Google Drive]] (all ISMS files), [[Tools/Google Sheets]], [[Tools/Claude in Chrome]] (edited Drive files and moved folders), [[Tools/Gmail]] (auditor emails), [[Tools/Google Apps Script]] (PDF export), [[Tools/Claude]] (all work done in Claude chat project "ac-iso-27001-isms").
- Drive: ISMS root folder in My Drive > iso > iso-27001-prj > ACI-ISMS. Folders: 0-Master, 1-Clauses (clause-4 to clause-10), 2-Policies, 3-Procedures, 4-Annex-A, 5-Evidence, 6-Audit, 7-archive.
- ACI policies on file: Child Safeguarding, ISMS, Social Media Content, Travel Expense. ISMS policies approved 2026-03-20.
- No GitHub repos or servers.


## 7. Files and documents
- ACI-Stage2-Audit-Readiness-Guide.docx — formal Stage 2 prep guide (5 sections + common mistakes table).
- Master Register (Excel) — sheets: Master Register, NC-CAPA Tracker, Folder Map.
- Stage 2 readiness tracker (Excel) — sheets: How to Use, Master Tracker, Annex A 2022 Controls, Pre-Submission Sweep.
- Folder-mapped checklist (Excel) — sheets: Folders & Documents, Annex A Evidence, Action Before Stage 2.
- Lead Auditor career report (Word, 10 sections) + 2026–2031 demand forecast.
- In Drive: statement-of-applicability-soa.xlsx (SoA v1.1, official), ISMS Scope Statement (MAS-001), Context of the Organization (2 versions), Interested Parties Matrix (2 versions), Interfaces & Dependencies register, 3 risk registers, Information Security Objectives, Business Process Framework, Process Interaction Matrix, User Access Matrix, Competence Matrix, Communication Procedure (ACI-PRO-002), change-request-form, master-list.xlsx, org-chart.docx, audit roadmap 2026–2028, NC folders ACI-NC-001 / ACI-NC-002.
- Apps Script for bulk-exporting Drive docs to PDF (in chat).

## 8. Open questions and problems
- Stage 2 audit exact date and result — not in chats (held June 2026).
- Is ACI now certified? Certificate number / date?
- SoA reference-document column shifted by one row — fixed?
- A.5.6 / A.5.7 applicability conflict — fixed?
- Exclusion list: chats say A.8.8, A.8.11, A.8.30 excluded, but GR/02 fix and checklist treat A.8.30 and A.8.11 as applicable — which is right?
- Org chart with Board (OBS-01) — done?
- Three overlapping risk registers — merged?
- Scope doc ID mismatch (MAS-001 vs ISMS-DOC-04.3) — fixed?
- Competence Matrix: empty reviewer column, past-due dates.
- Old Drive folders still to move (clause-9, clause-10, policies, NC folders, MRM records, manual, resources-evidence, process-matrix).
- MRM #2 (31 Jul) and risk review (30 Sep) — done?
- Name of the certification body / ISO vendor.
- Did Vishnu start the Lead Auditor course?

## 9. All chats in this project
- Index: [[Projects/ac-iso-27001-isms/chats/INDEX]]
- ISO 27001 audit non-conformities clearance — 2026-06-02
- Preparing for second audit checklist — 2026-06-12
- ISO documentation structure and organization — 2026-06-13
- ISO audit preparation checklist — 2026-06-15
- Organizing drive file structure — 2026-06-15
- ACI-ISMS audit checklist structure — 2026-06-16
- Evidence requirements — 2026-06-13
- Move project to viOS — 2026-10-02 (viOS meta chat)

Links: [[Projects/ac-iso-27001-isms/STATE]] · [[Projects/ac-iso-27001-isms/LOG]]

