---
tags: chat
date: 2026-06-15
source: Claude personal account
uuid: 5d1d61e1-9379-4996-bd00-0290e6a83d53
---
# ACI-ISMS audit checklist structure

## Summary
**Conversation Overview**

Vishnuvarthan Venkatapathy (referred to as Vishnu) is the ISMS Coordinator, UX Designer, and Security Lead at AraCreate India Pvt Ltd (ACI), a software/IT services firm based in Erode/Puducherry, India. The primary work throughout this conversation involved building and refining ISO 27001:2022 Stage 2 certification documentation. Claude created two Excel deliverables: a folder-mapped mandatory document checklist (matching ACI's Google Drive structure: 0-Master, 1-Clauses/4–10, 2-Policies, 3-Procedures, 4-Annex-A, 5-Evidence, 6-Audit) and a full Stage 2 evidence checklist with one subfolder per applicable Annex A control (90 of 93 — excluding A.8.8, A.8.11, A.8.30 per the SoA). The Excel workbook has three sheets: Folders & Documents, Annex A Evidence, and Action Before Stage 2. Vishnu confirmed he will mark document status manually in green/red rather than having Claude update the file. Key colleagues mentioned include Navaneethan (Managing Director, ISMS policy approver), Pradeepa (training and awareness owner), Kishor, Aravinth Panch (Director), and Rahul.

Vishnu shared screenshots of numerous ISMS documents for review and placement guidance. Documents reviewed included: the SoA (v1.1, official), ISMS Scope Statement (MAS-001, doc ID ISMS-DOC-04.3 — note: naming convention mismatch flagged), two versions of the Context of the Organization (duplicate flagged — retire the older one), Interested Parties Matrix (two versions — the stronger one with Annex A references retained), Interfaces & Dependencies register, three risk register files (asset-based, control-based per Annex A family ORG/PPL/PHY, and process/department-based — Claude flagged overlap and recommended consolidating to one primary register), Information Security Objectives (multi-tab workbook recommended), Business Process Framework, Process Interaction Matrix, User Access Matrix, Competence Matrix (reviewer column empty, past-due dates flagged), Organizational/People/Physical control risk registers, A.5 control responsibility assignment, and the Communication Procedure (ACI-PRO-002, Cl 7.4, correctly placed in 3-Procedures and classified under A.5 organizational controls). A critical recurring finding is that the SoA reference-document column is systematically shifted — each control cites the next control's procedure — which must be corrected before Stage 2.

The conversation also covered Vishnu's ISO 27001 Internal Auditor certificate (QHSE Solutions, March 2026) and his interest in progressing to Lead Auditor. Claude produced a 10-section Word report assessing his exam readiness, credential requirements (PECB Provisional → Auditor → Lead Auditor ladder with exact experience/audit-hour thresholds), a 6-week study plan emphasizing ISO 19011 and Domain 7 (audit program management) as the critical gaps, course options and costs in India (₹20,000–₹60,000 for accredited providers), salary data by career stage (₹5–8 LPA provisional through ₹25–40 LPA senior), three career paths (corporate ISMS lead, GRC consultant, certification body auditor), and a 4-year timeline. A separate demand forecast covered 2026–2031 using global ISO 27001 certificate growth data (~96,709 certificates in 2024, ~20–25% CAGR), India's cybersecurity skills shortage (1 million+ unfilled roles), and the DPDP Act enforcement timeline as a primary India-specific demand accelerator. The conversation closed with Vishnu asking about bulk-exporting Google Drive documents to PDF, for which Claude provided a Google Apps Script solution and a Google Takeout alternative.

**Tool Knowledge**

Claude used Google Drive MCP to search for and read the SoA file. The search query `title contains 'SoA' or title contains 'Statement of Applicability' or title contains 'Master Register'` successfully located multiple SoA versions; the authoritative file was identified as `statement-of-applicability-soa.xlsx` (fileId: `1nzxYd

## Chat

**Vishnu:** * ACI-ISMS/
   * 0-Master
   * 1-Clauses/
      * 4
      * 5
      * 6
      * 7
      * 8
      * 9
      * 10
   * 2-Policies
   * 3-Procedures
   * 4-Annex-A/
      * A5-organization-controls
      * A6-people-controls
      * A7-physical-controls
      * A8-technical-controls
   * 5-Evidence/
      * Common-Evidence
      * Specific
   * 6-Audit/
      * Stage-1
      * Stage-2
      * first-audit-nc
   * 
this is the structure i am using i need the chcek lsit in the smae type 


like each filder ned to have this documets that need to be the mantody douments for the aduit and that need and in the notes give me what it should cover 


i dont wnat anything other than that

**Claude:** I'll build a checklist mapped to your exact folder structure, listing the mandatory ISO 27001:2022 documents per folder with coverage notes.

_[tool: view]_

_[tool: create_file]_

_[tool: bash_tool]_

_[tool: present_files]_

Checklist mapped to your exact folder tree. Four columns: **Folder · Mandatory Document · Doc ID · Notes (what it must cover)**.

Two things to flag:

- **Annex A folders (4-Annex-A/A5–A8)** hold *evidence per applicable control*, not fixed documents — what's mandatory there is driven entirely by your SoA. I listed one evidence row per control family with the full control range in Notes; populate by your SoA inclusions.
- **The 7 mandatory documented-information items** ISO explicitly requires (scope, policy, risk methodology, SoA, risk treatment plan, objectives, plus the records under clauses 7/9/10) are all present and sit in `0-Master` and `1-Clauses`. Everything in `2-Policies`/`3-Procedures` is control-supporting, not strictly clause-mandated — keep only what your SoA marks applicable.

Doc IDs follow your POL/PRO/REC/EVD/MAS convention.

**Vishnu:** i see the document lsit is soo small and i need the must have lsit of docuemts manidtur for a it frim so plce update me and give 


dont jsut add all the docuemnt add the needed thigns only

**Claude:** I'll verify the current ISO 27001:2022 mandatory documents and the typical control set an IT services firm needs, then rebuild a tighter must-have list.

_[tool: web_search]_

The current list already covers the strictly-mandatory clause documents. What an IT services firm additionally needs are the Annex A items that *become* mandatory because the controls apply to a software/IT business (development, supplier, asset, incident, BC). I'll rebuild the list with that distinction marked clearly, and drop anything that's optional padding.

_[tool: create_file]_

_[tool: bash_tool]_

_[tool: present_files]_

Rebuilt as the must-have set for an IT/software services firm, colour-coded by type.

**Two categories, nothing optional:**
- **M (green) — clause-mandatory.** Required for *any* ISMS regardless of business. ISO 27001:2022 cut the mandatory count vs 2013; these are the full set: scope, policy, SoA, risk methodology/register/treatment plan, objectives, competence + awareness records, operational/monitoring/internal-audit/MRM/CAPA records.
- **A (orange) — mandatory for your firm.** Annex A controls that *apply* to a software/IT services business, so their documents become required once your SoA marks them applicable. The IT-critical ones: **Secure Development Policy** (A.8.25–8.31), **Supplier/Cloud Security Policy** (A.5.19–5.23), **Asset Register** (A.5.9–5.11), **Change Management** (A.8.32), **Access Control** (A.5.15/8.x).

**What I deliberately left out** (was padding in v1): standalone Clear Desk, Mobile/Teleworking, Threat Intelligence, Project Management policies — fold these into the parent policies above rather than maintaining separate documents.

**Confirm against your SoA:** every "A" row is only mandatory if that control is marked applicable. If you've excluded any (e.g. no physical data centre → some A.7 controls), drop the matching row.

**Vishnu:** only this are the needed documents for the second audit i need like fodlder and sub fodler wise also for the evidaces

**Claude:** Quick clarification before I rebuild — the structure changes depending on what you mean:

_[tool: ask_user_input_v0]_

**Vishnu:** Q: What do you want the new output to show?
A: a full docucemnt wiht the strudter and alos the needed files and eviadace in that

Q: For the Annex A evidence — how granular?
A: A subfolder per individual control you marked applicable in the SoA

**Claude:** I need your SoA to know which controls you marked applicable — that determines the Annex A subfolders. Let me find it in your Drive.

_[tool: tool_search]_

_[tool: Google Drive:search_files]_

Several SoA versions exist. The authoritative one is `statement-of-applicability-soa.xlsx` (v1.1, the official document referenced by your Master Register as REC-008/MAS-003, with the A.8.30 exclusion). I'll read it fully to get the exact applicable-control list.

_[tool: Google Drive:read_file_content]_

I have the authoritative SoA. Three controls are excluded: **A.8.30** (outsourced development — NO), and the main matrix marks **A.8.8** (Vulnerability Management) and **A.8.11** (Data Masking) as **Not Applicable**. Everything else is applicable.

One conflict to flag before I build: your two SoA versions disagree on A.8.8 and A.8.11. The main matrix in `statement-of-applicability-soa.xlsx` marks them **Not Applicable** with justification, but the secondary sheet inside the same file and the Master Register list them as Implemented. **This is an audit risk — an auditor will spot the contradiction.** I'll treat the formal applicability column as authoritative (A.8.8, A.8.11, A.8.30 excluded → no evidence subfolder), but you must reconcile this in the SoA itself before Stage 2.

Building the full document now — folder tree with a subfolder per applicable Annex A control, each with its required evidence.

_[tool: view]_

I'll build a Word document with the full folder/subfolder tree and an evidence table per applicable control. Given 90 applicable Annex A controls plus clause folders, this is best as a structured reference doc.

_[tool: bash_tool]_

_[tool: create_file]_

_[tool: bash_tool]_

_[tool: present_files]_

Built from your actual SoA. Full folder tree (`0-Master` → `6-Audit`) with required documents and evidence per folder, plus a subfolder per applicable Annex A control.

**Annex A coverage — built from `statement-of-applicability-soa.xlsx` (v1.1, the official one):**
- **90 of 93 controls applicable**, each gets its own subfolder named by control number, with the specific evidence to drop inside.
- **3 excluded — no folder:** A.8.8 (Vulnerability Mgmt), A.8.11 (Data Masking), A.8.30 (Outsourced Development).

**Four things to fix before Stage 2** (last page of the doc):

1. **SoA contradiction — highest risk.** Your main applicability matrix marks A.8.8 and A.8.11 *Not Applicable*, but the secondary sheet in the same file and the Master Register show them *Implemented*. An auditor will open the SoA first and spot this. Decide one position. Note: for an in-house software dev firm, A.8.8 and A.8.11 are hard to justify as *excluded* — if you host any apps/servers/cloud, vulnerability management almost certainly applies. Verify before excluding.
2. **OBS-01** — add Board to org chart (REC-005).
3. **A.5.4 and A.5.15** still say "need justification" / "need to get document" in the SoA.
4. **Reference-document column is shifted** — each control cites the *next* control's procedure (e.g. A.8.25 points to "Key management", A.8.28 to "Secure system architecture"). Realign so each row cites its own.

Want me to create the empty subfolder skeleton directly in your Drive, or fix the SoA contradiction in-place first?

**Vishnu:** need in excell file

**Claude:** _[tool: create_file]_

_[tool: bash_tool]_

_[tool: present_files]_

Same content as Excel, three sheets:

- **Folders & Documents** — `0-Master` through `6-Audit` with sub-folder, must-have document, Doc ID, and required content per row.
- **Annex A Evidence** — one row per applicable control (90 of 93), grouped by A5/A6/A7/A8 family, with the evidence to place in each control sub-folder. A.8.8, A.8.11, A.8.30 omitted per SoA.
- **Action Before Stage 2** — the four fixes, colour-coded by priority.

The SoA contradiction (A.8.8 / A.8.11) is still the top risk — reconcile it before the audit.

**Vishnu:** perfect i will give you the screen shot of each docuem lsts finalise that hten mark as green

**Claude:** Sounds good — send the screenshots whenever ready.

To make sure I mark things correctly: for each folder you screenshot, I'll compare what's actually present against the must-have list and mark each document green (present/done) or leave it flagged (missing). I'll update the same Excel file.

One quick question so the output matches what you want:

_[tool: ask_user_input_v0]_

**Vishnu:** Q: How should I mark the rows once I check your screenshots?
A: [No preference]

**Claude:** I'll use **Green = present, Red = missing** with a status column — it's the clearest for tracking what's left before Stage 2.

Send the first folder's screenshot whenever you're ready. Tell me which folder it is if it's not obvious from the view, and I'll match it against the checklist and update the file.

**Vishnu:** you no need to kepp the sheet and mark the stuauts i will do it manulayyt

**Claude:** Understood — you'll mark status manually. So this screenshot is for me to confirm against the checklist, not to update the file.

This is your **A.5 Organizational controls** SoA sheet, rows A.5.1–A.5.25 visible. Against the checklist, A.5.1 through A.5.25 are all present and marked **YES / Implemented** — consistent so far.

Two things I notice in this view:
- This SoA marks **A.5.6** and **A.5.7** as Implemented. Your other SoA file (the v1.1 official one) had a secondary sheet marking those as *Excluded*. Minor, but worth a glance for consistency.
- This is a **different/cleaner SoA** than the v1.1 file I read earlier — responsibilities are named people (Navaneethan, Kishor, Vishnu, Rahul) and there's no reference-document column shift. If this is the version you're taking to Stage 2, it looks much tighter.

Send the rest of A.5 (A.5.26–A.5.37) and then A.6/A.7/A.8 when ready — I'll confirm each against the must-have list and flag anything missing or marked NO/Not Applicable.

**Vishnu:** i have fulleymakr it green

**Claude:** Got it — A.5 is fully marked green on your end.

Send the next folder's screenshot (A.6, A.7, or A.8) whenever you're ready and I'll confirm it against the checklist.

**Vishnu:** hey this is soa docuem i will share the ss docoumeht vide no in the master chcecklist soa id done

**Claude:** Understood — A.5 in the SoA is done.

To make sure I track this right: the screenshot you shared was the **SoA** sheet, and you're now marking documents complete against the **master checklist** (the Folders & Documents / Annex A Evidence sheets I built). So "SoA done" = the SoA document itself is finalised and present.

Send the next document/folder screenshot whenever ready and I'll confirm it against the checklist.

**Vishnu:** this is scope docuemnt

**Claude:** This is your **ISMS Scope Document** — checklist item **MAS-001** in `0-Master`.

Against the Cl 4.3 requirements, it covers: organization, document ID (ISMS-DOC-04.3), version, last reviewed, introduction, organizational scope, physical address/location, and defined scope of services. That hits the core boundary elements.

To fully satisfy Cl 4.3, confirm the document also includes (lower in the page, not visible here):
- **Interfaces and dependencies** with other organizations (clients, cloud providers, suppliers)
- **Exclusions** with justification, if any
- **MD approval/sign-off**

If those are present, MAS-001 is complete — mark it green.

One note: Document ID is `ISMS-DOC-04.3`, not the `MAS-001` convention in your Master Register. Decide which scheme is authoritative and keep it consistent across docs, or an auditor may flag the mismatch.

Send the next one whenever ready.

**Vishnu:** whar can be this docume name and where it shoud come

**Claude:** This is your **Context & Issues Analysis** combined with **Interested Parties** — it covers two checklist items at once.

**Folder:** `1-Clauses / 4`
**Document name:** Context of the Organization (or "Context, Issues & Interested Parties Analysis")
**Doc IDs:** REC-001 (Cl 4.1 — internal/external issues) **and** REC-002 (Cl 4.2 — interested parties)

It satisfies both clause 4 records because it has all the right elements: external + internal issues split, impact rating with a defined scale (High/Medium/Low), and an Interested Parties column. Good document.

Two suggestions:

- **Naming:** since it merges 4.1 and 4.2, either title it to cover both ("Context, Issues & Interested Parties") or keep it as REC-001 and maintain interested-party *requirements* separately. An auditor wants to see not just *who* the interested parties are but *what their requirements* are (Cl 4.2 specifically asks for requirements). Right now you list the parties but not their security requirements — consider adding a "Requirements" column or a companion sheet.
- The file is named `context-organization` — fine, but align the in-document ID with your register (REC-001/002) for traceability.

If the interested-party requirements are captured somewhere, mark REC-001 and REC-002 green. If not, REC-002 is partially complete.

Next one whenever ready.

**Vishnu:** then what is thsi

**Claude:** This is a **second, more refined version** of the same document — `context-to-the-organization` vs the earlier `context-organization`.

Same checklist items: `1-Clauses / 4` → **REC-001** (Cl 4.1 issues) + **REC-002** (Cl 4.2 interested parties). It's the same combined Context + Issues + Interested Parties analysis, but this one is better structured — cleaner categories (Political & Geopolitical, Cybersecurity Environment, Information Security, etc.) and explicitly IT-firm relevant (cloud providers, ransomware, GDPR/DPDP Act, SLA pressures).

**The problem: you now have two versions of the same record.** `context-organization` and `context-to-the-organization` both target REC-001/002. An auditor seeing two context documents with no clear "current" marker is a document-control finding (Cl 7.5).

**Fix before Stage 2:**
- Pick this one as authoritative (it's stronger), archive or delete the older one.
- Add the version/ID header this one is missing — it has no Document ID, Version, or review date block like your Scope doc had. Add that for traceability.

Same gap as before: still lists interested *parties* but not their explicit *requirements* (Cl 4.2 wants both). The categories imply requirements, but a dedicated requirements column would close it cleanly.

So: one document, not two. Once you've retired the duplicate and added the header block, mark REC-001/002 green.

**Vishnu:** what is that

**Claude:** This is your **Interfaces & Dependencies** register — and it's exactly the Cl 4.3 element the Scope document was missing.

It maps each process/service to its internal and external dependencies (dev team, AWS/GCP, ISPs, cloud providers, certification body, etc.). This is what defines the *interfaces and dependencies* boundary of the ISMS scope.

**Where it goes:** It supports **Cl 4.3 (Scope)**, so it belongs with **MAS-001** in `0-Master` — either as an annex/tab within the Scope document or as a standalone supporting record in the same folder. It can also be referenced from `1-Clauses / 4` since it's context-related, but its primary job is completing the scope definition.

**Document name:** Interfaces & Dependencies Register (or "ISMS Scope — Dependencies Annex").

This is a good document and it closes the scope gap I flagged earlier. Same housekeeping note as the others: it has no Document ID / version header — add one for traceability.

With this, your scope (MAS-001) is now complete: boundaries + location + services + **interfaces/dependencies**. Mark it green once the version header is added and you've confirmed exclusions + MD sign-off are in the Scope doc.

Next one whenever ready.

**Vishnu:** dont give this mych big asnwerf for each

**Claude:** Got it. Short answers from here.

**Vishnu:** answer for last schreen shotp

**Claude:** **Interfaces & Dependencies Register** — Cl 4.3 support. Goes in `0-Master` with MAS-001 (Scope). Closes the scope gap. Add a version header. Mark green.

**Vishnu:** why it shoube do in that master i need to be in the 4.1 correct

**Claude:** You're right — it can go in `1-Clauses/4`. Both are defensible:

- **4.1/4.2** — it's context/dependencies analysis → fits with REC-001/002.
- **0-Master** — it supports the Cl 4.3 scope boundary.

Put it in `1-Clauses/4` since that's your preference and it sits naturally with the context docs. Just reference it from the Scope doc so the 4.3 link is traceable.

**Vishnu:** no i didint get he point

**Claude:** Clause 4 has parts:

- **4.1** = issues (your context sheet)
- **4.2** = interested parties (same sheet)
- **4.3** = scope, which must state boundaries **including interfaces and dependencies**

Your dependencies sheet is the "interfaces and dependencies" piece. That requirement is written under **4.3 (scope)**, not 4.1. That's the only reason I linked it to the Scope doc.

But the folder location is your call — the standard doesn't dictate folders. Putting it in `1-Clauses/4` is fine because it's all clause-4 material. Just make sure the Scope document (4.3) points to it, so the auditor sees the boundary is defined.

So: **file it in `1-Clauses/4`, reference it from the Scope doc.** Does that clear it up?

**Claude:** **Interested Parties Matrix** — REC-002 (Cl 4.2). Goes in `1-Clauses/4`.

This closes the gap I flagged earlier — it has the *requirements* column plus how you address each. Complete. Mark green.

**Vishnu:** so tell me whihc boht can be merges as sub sheets

**Claude:** In `1-Clauses/4`, merge these three into one workbook as tabs:

- **Tab 1 – Issues** → `context-to-the-organization` (4.1)
- **Tab 2 – Interested Parties** → `interested-parties-matrix` (4.2)
- **Tab 3 – Dependencies** → `dependencies` (4.3 interface/dependency support)

One file = "Context of the Organization" covering 4.1–4.3.

Drop the older `context-organization` (the duplicate). Keep the Scope document (MAS-001) separate in `0-Master` — it's the formal 4.3 statement and just references Tab 3.

**Claude:** Another version of the **Interested Parties Matrix** (REC-002) — this one is better. It adds an explicit "Impact on ISMS" column with Annex A references (A.5.19, A.6, etc.).

You now have two interested-parties files: `interested-parties-matrix` and `interested-party-sheet1`. **Keep this one** (stronger), drop the other. Then merge as Tab 2 in the clause-4 workbook.

**Vishnu:** what is the differnt between both

**Claude:** - **`interested-parties-matrix`** (earlier): columns = Party | Expectations/Requirements | Impact (High/Med/Low) | How addressed. Impact is just a rating.

- **`interested-party-sheet1`** (this one): columns = Party | Expectations/Requirements | **Impact on ISMS (described, with Annex A refs)** | How addressed. Impact is explained and linked to controls (A.5.19, A.6, etc.).

Difference: the second describes *how* each party affects the ISMS and ties it to controls — stronger for audit. Keep this one.

**Claude:** **Risk Assessment & Treatment — Asset-based** (REC-006, Cl 6.1.2). Goes in `1-Clauses/6`.

Asset-by-threat-vulnerability scoring (asset value, threat value, vulnerability value, likelihood). This is your Risk Register. Looks solid.

Confirm it also has: risk score/level, risk owner, and treatment columns (visible columns cut off at "F"). If the treatment plan is in the same sheet, it covers REC-007 too. Mark green once owners + treatment are present.

**Vishnu:** ok wait i will give another two screens short alos

**Claude:** Go ahead.

**Vishnu:** tell me how to organize this

**Claude:** These three are one risk-management package. Organize as a single workbook in `1-Clauses/6`:

**File: Risk Assessment & Treatment (REC-006 / REC-007 / PRO-001)**
- **Tab 1 – Value Guide** → `value-guide` (the scoring methodology/criteria = PRO-001, Cl 6.1.2)
- **Tab 2 – Asset Register** → `assets` (asset inventory with C/I/A, asset value = also feeds A.5.9)
- **Tab 3 – Risk Assessment** → `asset-sheet` (threat × vulnerability × likelihood scoring = REC-006)
- **Tab 4 – Risk Treatment Plan** → treatment/controls/owner columns (REC-007)

Order matters: Guide first (defines the scoring), then Asset, then Assessment, then Treatment — each builds on the prior.

One note: the **Asset Register tab also satisfies A.5.9** (Inventory of Assets). Reference it from `4-Annex-A/A5/A.5.9` rather than duplicating.

If Tab 3 already has the treatment columns, you don't need a separate Tab 4 — just confirm risk owner + treatment decision are in it.

**Vishnu:** i have only three screen short

**Claude:** Then three tabs in one workbook in `1-Clauses/6`:

**File: Risk Assessment & Treatment**
- **Tab 1 – Value Guide** → `value-guide` (methodology/criteria = PRO-001)
- **Tab 2 – Asset Register** → `assets` (inventory + C/I/A = REC-006, also feeds A.5.9)
- **Tab 3 – Risk Assessment** → `asset-sheet` (threat/vulnerability/likelihood scoring = REC-006)

Check Tab 3 has **risk owner** and a **treatment decision/control** column — that's what makes it REC-007 (Risk Treatment Plan) too. If those columns exist, all three of REC-006, REC-007, PRO-001 are covered. If not, you're missing the treatment plan.

**Vishnu:** what is diffrenace between this tow i fee like same

**Claude:** They're not the same — they're two tabs of the **same workbook** (`risk-assessment-treatment` in the Risk-Management folder):

- **Image 1 = Table1 "Asset Register"** — the inventory. Columns: Asset ID, Type, Owner, C/I/A, Asset Value, Rating. One row per asset (35 assets). → REC-006 input + A.5.9.

- **Image 2 = Table2 "Risk Assessment"** — the risk analysis. Columns: Asset, Threat, Threat Value, Vulnerability, Vulnerability Value, Likelihood. Multiple rows per asset (one per threat). → the actual risk register, REC-006.

They look similar because Table2 repeats asset name/value as a reference before scoring threats. But Table1 = *what you own*, Table2 = *what can go wrong with it*.

So they're already correctly organized as tabs. Just confirm Table2 continues right (columns F onward) into a **risk score, risk owner, and treatment** — that's REC-007.

**Vishnu:** what need to be cleanded in this

**Claude:** Cleanup needed in this risk workbook:

1. **Risk owner is generic** — every row says "ACI IT". Real risk owners should be named people/roles (per asset or risk), or an auditor will say risk ownership isn't assigned (Cl 6.1.3 / 5.3).

2. **Confirm the treatment columns exist** (Table2, right of column I) — risk score, treatment decision (treat/accept/transfer/avoid), control selected, residual risk, target date. If missing, REC-007 is incomplete.

3. **Duplicate file** — you also have `risk-assement-and-treatment-asset-sheet` (earlier screenshot, different file). This new `risk-assessment-treatment` workbook supersedes it. Delete the old one.

4. **Value Guide tab** — make sure the `value-guide` is a tab inside this same workbook, not a separate file.

5. **Spelling** — "assement", "integirity", "vulnerabiliity" in titles/headers. Fix before audit.

6. **No Document ID/version header** — add REC-006/007 + version + approval, same as other docs.

Fix owners (#1) and confirm treatment (#2) first — those are the audit-critical ones.

**Vishnu:** you are confustion i will share all what i ahve then tell me how to organize

**Claude:** Understood. Send everything you have, then I'll lay out the full organization in one go.

**Vishnu:** this is what i have fully

**Claude:** Now it's clear. You have **three separate files**, but they're really one risk process. And Image 3 confirms the treatment plan exists (Risk Treatment Options, New Controls, Residual Risk).

**The three files:**
1. `risk-assement-and-treatment-value-guide` → the scoring criteria/methodology
2. `risk-assessment-and-treatment-assets` → asset register (Table1)
3. `risk-assessment-treatment` → risk assessment + treatment (Table2, with residual risk in cols M–U)

**How to organize — merge all into ONE workbook** named **Risk Assessment & Treatment**, in `1-Clauses/6`:

- **Tab 1 – Value Guide** (from file 1) = methodology → **PRO-001**
- **Tab 2 – Asset Register** (from file 2) = inventory → feeds **REC-006** + **A.5.9**
- **Tab 3 – Risk Assessment & Treatment** (from file 3) = scoring + treatment + residual → **REC-006 + REC-007**

One file, three tabs, in the order above. Delete the duplicate `risk-assement-and-treatment-asset-sheet` from earlier.

**Two fixes before marking green:**
1. **Risk Owner = "ACI IT" on every row** → assign named owners. Critical for audit.
2. **Spelling** in titles/headers: "assement", "integirity", "vulnerabiliity".

Once merged + owners fixed, this single file covers REC-006, REC-007, and PRO-001 — three checklist items done.

**Vishnu:** which is Register in the screens hots

**Claude:** The **Register** is **Image 2** — `risk-assessment-and-treatment-assets` (Table1).

That's your **Asset Register**: one row per asset (ACI-AST-001 to 035), with Asset ID, Type, Owner, Location, C/I/A scores, Asset Value, Rating. It's the inventory of what you own.

(Image 1 = scoring guide; Image 3 = risk assessment + treatment with residual risk.)

**Vishnu:** what is this where it will come

**Claude:** **Gap Assessment** — clause-by-clause questionnaire mapping each requirement to a response, lead, and evidence.

This is a cross-cutting readiness/audit-prep document, not a single-clause record. Two options:

- **`0-Master`** — best fit. It indexes the whole ISMS against the standard and points to where each evidence lives. Treat it as a master traceability/readiness record.
- **`6-Audit/Stage-2`** — also valid, since it's your Stage 2 readiness check.

Recommend **`0-Master`** (it's a living traceability map across all clauses), and reference it from `6-Audit/Stage-2` as readiness evidence.

It's very useful — the Evidence column is effectively your audit roadmap. Just confirm every evidence item it names actually exists in the right folder.

**Vishnu:** what is the file name need to give

**Claude:** `ISMS-Gap-Assessment` or `ISMS-Clause-Gap-Assessment`.

With your ID convention: **REC-** prefix (it's a record), e.g. `REC-0XX-ISMS-Gap-Assessment`. Pick the next free REC number in your Master Register.

**Claude:** This is an **evidence map for Clause 5 / Information Security Policy** — it lists what to show an auditor for each leadership requirement, with links to the actual evidence (objectives, onboarding log, risk register, MFA screenshot, etc.).

It's a **traceability/evidence index**, not the policy itself. Despite the filename "5.1 Information Security Policy", this is *not* POL-001 — it's a pointer sheet.

**Where it goes:** `0-Master` alongside the Gap Assessment — both are evidence-mapping documents. Or `1-Clauses/5` if you want it with leadership.

**Name:** something like `Clause-5-Evidence-Index` (REC-). Don't name it "Information Security Policy" — that's misleading; the real policy (POL-001, signed PDF) is a separate document this sheet *links to*.

**Watch out:** this overlaps heavily with your Gap Assessment sheet (both map requirements → evidence). Consider folding it into the Gap Assessment to avoid two competing evidence maps.

**Claude:** **Information Security Objectives** (REC-009, Cl 6.2). Goes in `1-Clauses/6`.

- **Image 1** = the master objectives table (objective, KPI, target, frequency, responsible) — this is the main REC-009.
- **Images 2 & 3** = per-objective detail cards (Objective 1, Objective 2…), adding monitoring method + action if target not met.

**How to organize:** one workbook in `1-Clauses/6` named `Information-Security-Objectives` (REC-009):
- **Tab 1 – Objectives Summary** (Image 1)
- **Tab 2+ – per-objective detail cards** (Images 2, 3, …) — or fold the "Monitoring Method" and "Action if not achieved" columns into the summary table to avoid one tab per objective.

Strong document — it's measurable with targets, owners, and frequency, which is exactly what Cl 6.2 wants. Add the Doc ID/version header and mark green.

**Vishnu:** can i ahve objective 1 adn 2 in smae sheet or seperate

**Claude:** Same sheet — one tab, not separate.

15 objectives = 15 separate tabs is unmanageable and an auditor won't want to click through them. Put them as stacked cards down one sheet, or better, just add the two extra fields as columns to your summary table (Image 1):

Objective | KPI | Target | Frequency | Responsible | **Monitoring Method** | **Action if Target Not Met**

That single table covers everything Cl 6.2 needs, in one view. Drop the per-objective card tabs.

**Vishnu:** what is this

**Claude:** **Business Process & Operational Framework** — your end-to-end business process flow with departmental ownership.

This isn't a strict ISO mandatory document, but it's strong supporting evidence for:
- **Cl 4.4 / 8.1** — how the ISMS integrates with business processes (process flow, departments)
- **A.5.2** — roles and responsibilities (departmental ownership)

**Where it goes:** `1-Clauses/4` (context — it describes how the org operates) or `0-Master` as a supporting reference. Recommend `1-Clauses/4`.

**Name:** `Business-Process-Framework` (REC-) or "Process Interaction Map" — your Gap Assessment already referenced a "Process Interaction Matrix", so this likely *is* that evidence. Align the name.

Good to have, not mandatory. Mark as supporting evidence.

**Vishnu:** what is this

**Claude:** **Process Interaction Matrix** — maps each process (category, name, inputs, activities, outputs/evidence, who it interacts with).

This is the "Process Interaction Matrix" your Gap Assessment referenced. Supporting evidence for **Cl 4.4 / 8.1** (ISMS integration with processes).

**Where it goes:** `1-Clauses/4` — pairs naturally with the Business Process Framework you just showed. They're two views of the same thing:
- Business Process Framework = narrative/flow
- Process Interaction Matrix = the table with inputs/outputs/evidence

**Organize:** put both in one workbook in `1-Clauses/4`, named `Process-Interaction-Matrix` (REC-):
- Tab 1 – Process Framework (the doc)
- Tab 2 – Interaction Matrix (this sheet)

The "Outputs (Audit Evidence)" column is useful — it tells the auditor where each process's evidence lives. Good document.

**Vishnu:** is this alredy done

**Claude:** I can't tell you it's "done" — I've only seen this one screenshot, not what it links to.

The matrix *itself* looks complete: all columns filled, processes mapped, evidence outputs named. As a document, yes.

But "done" for audit means the **Outputs (Audit Evidence)** it names actually exist — e.g. "Risk Register", "Statement of Applicability", "Signed NDAs", "Backup logs", "Audit Report". The matrix is only valid if those exist where it implies.

So: the matrix is built. Mark it green as a document. Verify its referenced evidence exists as you work through the rest of the folders.

**Vishnu:** is

**Claude:** **Organizational Control Risk Register** — control-based risk register mapped to Annex A (A.5 controls), with named owners and treatment actions. This is good — it has the named owners your asset risk register was missing.

This is a **second, control-based risk register**, separate from your asset-based one. Both are valid risk views:
- Asset-based register (earlier) = risk per asset/threat
- This = risk per control area, mapped to Annex A

**Where it goes:** `1-Clauses/6` with the other risk docs, OR as a tab in the merged Risk Assessment & Treatment workbook. It's part of REC-006/007.

**Two notes:**
1. **Owners here are named** (ISMS Manager, HR Head, IT Head). Use this same approach to fix the "ACI IT" generic owners in your asset register.
2. Title typo: "organizaional".

So your full risk picture = asset-based register + this control-based register + value guide. Keep both registers — they complement each other. Merge into the one Risk workbook in `1-Clauses/6` as separate tabs.

**Claude:** This is an **A.5 Control Responsibility assignment** — maps each A.5 control to who owns it.

It's a slice of your roles/responsibilities, not a standalone deliverable. This belongs as part of the **RACI / Roles & Responsibilities Matrix** (REC-004, A.5.2) in `1-Clauses/5`.

**Where it goes:** fold into the Roles & Responsibilities matrix. If you have similar sheets for A.6/A.7/A.8, combine all four into one **Control Responsibility Matrix** covering all 93 controls.

It overlaps with the SoA's "Responsibility" column too — keep them consistent (same owners in both).

So: not a new document — it's control-ownership data that lives in the RACI matrix (REC-004). Mark green once merged there.

**Claude:** **People Control Risk Register** (A.6) — same format as the Organizational (A.5) one, but for people controls. PPL-01 to PPL-09, mapped to A.6 controls with named owners.

This is the **A.6 sibling** of the organizational risk register you showed earlier. You're building one risk register per control family:
- ORG-xx → A.5 (organizational)
- PPL-xx → A.6 (people)
- likely PHY-xx → A.7, TECH-xx → A.8 coming

**Where it goes:** `1-Clauses/6` with the other risk docs. Merge all the control-based registers (ORG, PPL, and the A.7/A.8 ones) into **one workbook** — `Control-Based-Risk-Register` — with a tab per family, or stacked in one sheet.

So your full risk picture:
- Asset-based register (per-asset/threat)
- Control-based register (ORG + PPL + PHY + TECH) ← this
- Value Guide

Keep going — send the A.7 and A.8 ones if they exist, then this set is complete.

**Claude:** **Physical Control Risk Register** (A.7) — PHY-01 to PHY-09, mapped to A.7 controls, named owners. Same format as ORG and PPL.

That's the third of four. So far: ORG (A.5), PPL (A.6), PHY (A.7). Send the A.8 / TECH one if it exists, then the control-based register set is complete.

All go into one workbook in `1-Clauses/6` — `Control-Based-Risk-Register`, tab or section per family.

**Claude:** **User Access Matrix** — who has access to what (laptop, email, Slack, Google Workspace, AWS…) per employee, with role and employment status.

This is evidence for **A.5.15 (Access Control), A.5.18 (Access Rights), A.8.2 (Privileged Access)**. Not a clause record — it's control evidence.

**Where it goes:** `4-Annex-A/A8-technical-controls` (or A5) — drop it in the **A.5.18 / A.5.15** evidence subfolder. It's the proof that access rights are tracked and reviewed.

**Name:** `User-Access-Matrix` (EVD-).

Strong evidence document. One audit tip: pair it with an **access review sign-off** (date + reviewer) — the matrix shows current state, the review proves you check it periodically (A.5.18 requires periodic review).

**Claude:** **Competence Matrix** (REC-010, Cl 7.2 / A.6.3) — required vs current skill level per employee, with gap, training required, target date, and reviewer.

Goes in `1-Clauses/7`.

This is exactly what Cl 7.2 wants — competence tracked with evidence and gap closure. Good document.

Two notes:
- **Reviewer column is empty** — fill it (auditor will check who verified competence).
- Some **target dates have passed** (30-05-2026, 04-06-2026 vs today 15-06-2026). Either mark those trainings complete or update dates, or it looks like overdue actions.

Owner: Pradeepa. Mark green once Reviewer is filled and past-due items are resolved.

**Claude:** **Risk & Opportunity Register — process-based** (Ref 6.1/61/Rev 1), organized by process/department (Top Management here), with severity × occurrence scoring and treatment actions.

This is a **third risk register format** — process/department-based, covering Cl 6.1 *risks and opportunities* (note: it includes opportunities, which the others don't). Goes in `1-Clauses/6`.

This has the proper document header (Ref No, dates, owner, review frequency, next review) the others were missing — good template.

**You now have three risk register styles:**
1. Asset-based (per asset/threat)
2. Control-based (ORG/PPL/PHY/TECH → Annex A)
3. Process-based (this — per department, risks + opportunities)

**Decision point — this is too many.** Three overlapping registers is a document-control and consistency risk at audit. Pick your primary:
- **Process-based (this)** best matches Cl 6.1 because it covers risks *and opportunities* and ties to processes.
- Keep **Asset-based** as the asset-level detail feeding it.
- The **control-based (ORG/PPL/PHY)** largely duplicates — consider whether it adds value or just repeats Annex A.

Recommend: one **primary register** (process-based) + asset register as supporting input. Don't take all three to the auditor unless they clearly serve different, stated purposes.

Two flags on this sheet:
- **HO Address = Allen, TX** — that's not your Erode address. Fix (looks copied from a template).
- **Prepared By / Approved By blank** — fill before audit.

**Vishnu:** what evidace can be give for this 
Labels are used to indicate classification level and handling requirements.	Implemented	Identity management Procedure

**Claude:** That SoA row (A.5.13 Labelling of Information) has a **mismatch** — the reference says "Identity management Procedure", which is wrong. A.5.13 should reference a Labelling procedure. Fix that first.

**Evidence for A.5.13 (Labelling of Information):**

- **Labelling procedure/guideline** — how each classification level (e.g. Public, Internal, Confidential, Restricted) must be marked
- **Screenshots of labelled documents** — file headers/footers or watermarks showing "Confidential", "Internal Use Only", etc.
- **Email classification tags** — subject prefixes or sensitivity labels (e.g. Microsoft/Google sensitivity labels)
- **Labelled physical media/assets** — photos of labelled drives, folders, or asset tags
- **System/folder naming** — Drive folders or repos marked by classification
- **Template headers** — your document templates showing a classification field

Minimum to satisfy an auditor: the labelling procedure + 2–3 real examples of labelled information across formats (a document, an email, a storage location).

Place in `4-Annex-A/A5/A.5.13`. Note it links to A.5.12 (Classification) — labelling implements the classification scheme, so reference them together.

**Vishnu:** Protects information in transit between internal and external parties using secure transfer mechanisms.	Implemented	Authentication information Procedure


what can be evidace for this

**Claude:** Same issue — A.5.14 (Information Transfer) references "Authentication information Procedure", which is wrong. It should reference an Information Transfer procedure. Fix the SoA mapping.

**Evidence for A.5.14 (Information Transfer):**

- **Information Transfer policy/procedure** — rules for sending data internally and to external parties
- **Encryption in transit** — TLS/HTTPS config, VPN setup, SFTP usage screenshots
- **Email security** — TLS enforcement, encrypted email, or sensitivity-label-based blocking
- **Transfer agreements** — NDAs / DPAs with clients and vendors covering data exchange
- **Secure file-sharing evidence** — Google Drive/Workspace sharing controls, password-protected links, expiry settings
- **Approved channels list** — which tools are allowed for transfers (and which are banned)

Minimum for audit: the transfer procedure + proof of an encrypted channel (TLS/VPN/SFTP) + a signed NDA/DPA showing external transfers are contractually controlled.

Place in `4-Annex-A/A5/A.5.14`.

Note: your SoA's reference-document column is systematically shifted — each control points to the *next* control's procedure (A.5.13→Identity, A.5.14→Authentication). Realign the whole column before Stage 2; it's the same defect repeating.

**Vishnu:** A.5.15	Access control	Navaneethan, Kishor & Vishnu	YES	Ensures access to systems and data is granted on a need‑to‑know/least‑privilege basis.	Implemented

**Claude:** **Evidence for A.5.15 (Access Control):**

- **Access Control Policy** — rules for granting/reviewing/revoking access on need-to-know / least-privilege
- **User Access Matrix** — you already have this (`user-access-matrix`); it shows who has access to what
- **Access request/approval records** — sample joiner request with manager/owner sign-off
- **Role-based access (RBAC) definition** — roles mapped to permissions
- **Access review records** — periodic review with reviewer + date (proves least-privilege is enforced over time)
- **System screenshots** — permission settings in key systems (Google Workspace, AWS/GCP, GitHub) showing restricted access

Minimum for audit: Access Control Policy + the User Access Matrix + one access request approval + one periodic access review sign-off.

Place in `4-Annex-A/A5/A.5.15`. Links to A.5.18 (Access Rights) and A.8.2 (Privileged Access) — cross-reference them.

**Vishnu:** A.5.16	Identity management	Kishor & Vishnu	YES	Governs the lifecycle of user identities from creation to removal.	Implemented	Use of secret authentication information Procedure

**Claude:** **Evidence for A.5.16 (Identity Management):**

- **Identity Management procedure** — full lifecycle: create → modify → disable → delete identities (note: the SoA references "Use of secret authentication information" — wrong, that's A.5.17; fix the mapping)
- **User/identity register** — list of all unique user identities tied to real people (no shared/orphan accounts)
- **Joiner records** — sample new-identity creation with approval
- **Mover/leaver records** — identity disabled/removed on exit (links to exit checklist)
- **Unique ID evidence** — proof each user has one unique identifier; shared accounts justified/controlled
- **System screenshots** — user lists in Google Workspace, AWS IAM, GitHub showing active vs disabled identities

Minimum for audit: Identity Management procedure + identity register + one joiner creation + one leaver deactivation showing the lifecycle works end-to-end.

Place in `4-Annex-A/A5/A.5.16`. Distinct from A.5.15 (access *rights*) — A.5.16 is about the *identity* itself. Links to A.5.18 and A.6.5 (post-employment).

**Vishnu:** A.5.20	Addressing information security within supplier agreements	Navaneethan & Vishnu	YES	Security, confidentiality, and compliance requirements are included in supplier contracts.	Implemented	Privacy and protection of personally identifiable information (PII) Procedure

**Claude:** **Evidence for A.5.20 (Addressing Information Security within Supplier Agreements):**

- **Supplier agreement template** — with security, confidentiality, and compliance clauses built in (the SoA reference "PII Procedure" is wrong again — should point to supplier agreements; fix mapping)
- **Signed supplier contracts/NDAs** — sample executed agreements containing the security clauses
- **Standard security clauses** — the specific clauses you require: data protection, confidentiality, breach notification, right to audit, sub-contracting limits, return/deletion of data on termination
- **DPA (Data Processing Agreement)** — where suppliers handle personal data
- **Cloud provider terms** — AWS/GCP/Google Workspace security/DPA terms you've accepted

Minimum for audit: the supplier agreement template showing required security clauses + at least 2 signed supplier agreements/NDAs that actually contain them.

Place in `4-Annex-A/A5/A.5.20`. Links to A.5.19 (Supplier Security) and A.5.21 (ICT Supply Chain) — A.5.19 is the policy/assessment, A.5.20 is the contractual enforcement.

**Vishnu:** 22	A.5.22	Monitoring, review and change management of supplier services	Vishnu	YES	Ensures supplier services remain in line with agreed security requirements over time.	Implemented	Compliance with legal, statutory, regulatory and contractual requirements Procedure

**Claude:** **Evidence for A.5.22 (Monitoring, Review and Change Management of Supplier Services):**

- **Supplier monitoring/review procedure** — how often and how you review suppliers (SoA reference "Compliance with legal… Procedure" is wrong; should point to supplier monitoring — fix mapping)
- **Supplier review records** — completed periodic reviews of key suppliers (AWS/GCP, SaaS vendors) with date, reviewer, outcome
- **Supplier performance/SLA tracking** — evidence you monitor agreed service levels
- **Supplier risk re-assessment** — updated risk assessment when a supplier's service or scope changed
- **Change management for supplier services** — record of a supplier change (new service, scope change, version) being reviewed/approved
- **Issue/escalation log** — any supplier security issues raised and how resolved

Minimum for audit: the monitoring procedure + at least one completed periodic supplier review (with reviewer + date) + one record showing a supplier service change was assessed.

Place in `4-Annex-A/A5/A.5.22`. Links to A.5.19 (security), A.5.20 (agreements), A.5.21 (supply chain) — this is the *ongoing* control over the supplier lifecycle.

**Vishnu:** Information security for use of cloud services	Vishnu & Kishor	YES	Cloud services are used and must be assessed, approved, and monitored for security.	Implemented	Intellectual property rights Procedure

**Claude:** **Evidence for A.5.23 (Information Security for Use of Cloud Services):**

- **Cloud Services Security policy/procedure** — acquisition, use, management, and exit of cloud services (SoA reference "Intellectual property rights Procedure" is wrong; should point to cloud security — fix mapping)
- **Cloud service inventory** — list of cloud services used (AWS, GCP, Google Workspace, GitHub, Figma, etc.)
- **Cloud risk assessment / approval** — assessment + sign-off before adopting each service
- **Shared-responsibility evidence** — documented split of security duties between you and the CSP
- **Cloud config screenshots** — IAM, MFA, encryption, logging, backup settings in AWS/GCP/Workspace
- **CSP security terms / DPA** — accepted provider security and data-processing terms
- **Monitoring evidence** — cloud logs, alerts, or periodic config reviews
- **Exit strategy** — how data is retrieved/deleted if you leave a provider

Minimum for audit: cloud policy + cloud service inventory + one approval/risk assessment for a key service + config screenshots showing MFA/encryption/logging enabled.

Place in `4-Annex-A/A8/A.5.23` (or A5 — keep with the A.5 family). Links to A.5.21 (ICT supply chain), A.8.2/A.8.5 (access/auth on cloud).

**Vishnu:** i have alredy evidace for clasue vise do i alos need to prepare evidace for policy and procudew wise

**Claude:** No — you don't prepare *separate* evidence for policies/procedures. Same evidence, organized once.

Here's the distinction:

- **Policies & Procedures** = the documents that *say what you do* (Access Control Policy, Backup Procedure). These live in `2-Policies` and `3-Procedures`. They are themselves the deliverable — no extra "evidence" needed beyond the approved, versioned document.

- **Evidence** = proof you *actually do it* (screenshots, logs, signed forms, review records). This lives in `4-Annex-A` (per control) or `5-Evidence`.

So per control you have three things, not three sets of evidence:
1. The **policy/procedure** (in 2/3) — says the rule
2. The **control evidence** (in 4-Annex-A) — proves the rule is followed
3. The **SoA** maps which control → which policy + evidence

**You don't duplicate.** If your clause-wise evidence already covers a control, just make sure:
- the relevant **policy/procedure document exists** in folder 2 or 3, and
- the **SoA reference column points to it correctly** (yours is currently mis-mapped — that's the real gap).

So the work left isn't more evidence — it's: confirm each policy/procedure document exists, and fix the SoA mappings so each control cites the right one.

**Vishnu:** what are the things needed for caluse 8

**Claude:** Clause 8 (Operation) has three parts. Evidence needed:

**8.1 — Operational Planning & Control**
- Evidence your security processes run as planned: the Process Interaction Matrix you showed
- Change records (links to A.8.32 change management)
- Outsourced/supplier process control (links to A.5.22)
- Documented operating procedures (A.5.37)

**8.2 — Information Security Risk Assessment**
- Risk assessment performed at planned intervals or on significant change
- Your risk registers (asset/control/process-based) with assessment dates
- Proof it's repeated, not one-off (e.g. "Last reviewed" dates, or a re-assessment triggered by a change)

**8.3 — Information Security Risk Treatment**
- Risk Treatment Plan with controls applied
- Residual risk recorded (your `risk-assessment-treatment` sheet, cols M–U: treatment options, new controls, residual risk)
- Evidence treatments were actually implemented (not just planned)

**In short, Clause 8 reuses your Clause 6 risk documents** — but the key auditor test is **recurrence and triggers**: that risk assessment/treatment happens on a schedule and when things change, not just once for certification.

So for `1-Clauses/8`, place:
- A short **Operational Planning record** (or reference to Process Interaction Matrix)
- The **risk assessment** (dated, with review interval) → 8.2
- The **risk treatment plan + residual risk** → 8.3
- One example of a **change triggering re-assessment** — this is what most firms miss

You likely already have the documents; Clause 8 is mostly about *pointing to them* and proving they operate continuously.

**Vishnu:** is this certificate has any value

**Claude:** Yes — significant value for your audit. This is your **ISO 27001:2022 Internal Auditor certificate** (Vishnuvarthan Venkatapathy, QHSE Solutions, Mar 2026).

It's evidence for two things:

- **Cl 9.2 (Internal Audit)** — proves your internal auditor is *competent* to conduct the internal audit. Auditors check that whoever ran your internal audit was trained/qualified. This is that proof.
- **Cl 7.2 (Competence)** — competence record for an ISMS role.

**Where it goes:**
- Primary: `1-Clauses/9` (with the internal audit plan/reports) as auditor competence evidence
- Also reference from `1-Clauses/7` (competence records) and `4-Annex-A/A6/A.6.3` (training)

One practical point: an internal audit should ideally be conducted by someone **independent of the area audited** (Cl 9.2). Since you (Vishnu) are the ISMS Coordinator/Security Lead *and* the trained internal auditor, an auditor may question independence — you shouldn't audit your own work. If possible, have someone else use your competence, or document how independence was maintained (e.g. you audited areas you don't own).

So: keep it, file it in `1-Clauses/9`, mark it as auditor-competence evidence — but mind the independence point.

**Vishnu:** other than this what is the use

**Claude:** Beyond the audit-evidence use, the practical value:

- **You can run the internal audit in-house** — no need to pay an external consultant for the mandatory Cl 9.2 internal audit. Direct cost saving.
- **Qualifies you to maintain the ISMS year-round** — internal audits aren't one-time; they're required before every surveillance audit (annually). You can do all future cycles yourself.
- **Skill/role credibility** — formal qualification backing your ISMS Coordinator / Security Lead role; useful for the competence matrix and for client/RFP credibility.
- **Lets you pre-audit before Stage 2** — you can self-assess and catch nonconformities (like your SoA mapping issues) before the external auditor does.
- **Transferable** — it's a personal certification tied to you, valid across organizations, not just ACI.

What it does **not** do: it's not the company certification (that's Stage 2), and it doesn't make you an external/lead auditor for other companies — that needs a Lead Auditor course.

**Vishnu:** i am asking after the office

**Claude:** If you mean **after this job / leaving ACI** — the certificate's value to you personally:

- **Portable credential** — it's tied to your name, not the company. You keep it regardless of where you work.
- **Career value** — Internal Auditor ISO 27001 is a recognized line on a CV/LinkedIn; useful for roles in security, compliance, GRC, ISMS coordination, or audit.
- **Foundation for higher certs** — it's the entry rung. Next steps that add more value: **ISO 27001 Lead Auditor** (lets you audit other companies / work for certification bodies) and **Lead Implementer** (lets you build ISMS for clients as a consultant).
- **Consulting/freelance** — combined with the hands-on ACI experience you're getting now, you could advise other small firms on certification.

Honest limit: on its own, a one-day *Internal* Auditor completion certificate from a training provider is a modest credential — it shows awareness, not deep expertise. Its real value comes paired with **actual experience** (which you're building now: running a real ISMS through to certification). That combination — cert + "took a company through ISO 27001 Stage 1→2" — is what's genuinely marketable.

So keep it, but let the project experience be the headline; the certificate supports it.

**Vishnu:** 27001 Lead Auditor how to become this

**Claude:** **ISO 27001 Lead Auditor** — the path:

**1. Take an accredited Lead Auditor course (5 days)**
- Must be **certified by an accreditation body** — look for **Exemplar Global, IRCA (CQI/IRCA), or PECB**. The accreditation is what gives it value; a non-accredited course won't count toward certification.
- Covers: audit principles, planning, conducting, reporting, and closing audits per ISO 19011 + ISO 17021.
- Ends with an exam (usually ~2 hours, written).

**2. Pass the exam** — included in the course.

That gets you the **Lead Auditor training certificate**. To become a *certified/registered* Lead Auditor (the credential that lets you audit for a certification body), you also need:

**3. Audit experience / logged audits**
- Bodies like IRCA require a number of completed audits (audit days logged under supervision) before you move from "Provisional" to full "Lead Auditor" grade.
- Your **Internal Auditor cert + the ACI internal audits you run now** count as foundational experience.

**4. Apply for registration** (optional but valuable)
- Register with IRCA/Exemplar Global as a certified auditor once you meet the experience requirement.

**Practical route for you:**
- You already have Internal Auditor (good base).
- Do a **PECB or IRCA-accredited Lead Auditor course** (online options exist, ~₹30,000–60,000 in India depending on provider).
- Log audit days (start with ACI's internal audits).
- Then register.

**Verify before paying:** confirm the provider is accredited by Exemplar Global / IRCA / PECB — ask for their accreditation number. Many cheap "lead auditor" courses aren't accredited and won't be recognized by certification bodies.

Want me to look up currently available accredited Lead Auditor courses in India?

**Vishnu:** really dose this has any good value in the market what is the demand and how this is going to extend in the future.... i need in exact numbers

**Claude:** I'll search for current market data — demand, salaries, and growth numbers for ISO 27001 Lead Auditors.

_[tool: web_search]_

Let me get the global market growth numbers to answer the "future" part.

_[tool: web_search]_

Here are the exact numbers.

## Salary in India (2026)

The figures vary by source and how "Lead Auditor" is defined, so here's the honest spread:

- **Glassdoor (Feb 2026, 42 samples):** average ₹13,22,500 per year, typical range ₹8,50,000 (25th percentile) to ₹22,18,750 (75th percentile), with top earners up to ₹29,55,000.
- **PayScale (2026):** average base ₹763,024, ranging from ₹206k at the low end to ₹2m at the 90th percentile.
- **Profile-platform benchmarks:** professionals with ISO 27001 Lead Auditor skills earn ₹18 LPA to ₹63 LPA, averaging ₹23.3 LPA, with top-10% earners reporting ₹33 LPA+.

The gap is real: the certificate alone tracks the lower PayScale figure (~₹7.6L); the ₹13–23L numbers reflect people who also have **experience + senior GRC/security roles**. The cert is a multiplier on experience, not a standalone ticket to ₹23L.

## Demand (India, now)

- 204 open ISO 27001 lead auditor jobs listed on Glassdoor India, plus active listings on Naukri/LinkedIn. Companies advertising ₹15–40 LPA for ISO 27001 auditors at the senior end.
- Most listings ask for an IRCA-approved ISMS Lead Auditor course and experience in internal or external audits, often bundled with CISA/CISM/CISSP/ISO 27701/SOC 2/PCI DSS.

## Growth (global, the "future" part)

This is the strongest part of the case:

- ISO 27001 certificates issued worldwide: jumped from 48,671 in 2023 to 96,709 in 2024 — nearly doubling, across 179,877 certified sites.
- The total has reached ~100,000 certificates, growing 20–25% per year, with a compound annual growth rate of about 20.3%.
- It is the fastest-growing segment in the ISO certification market, projected to expand at a CAGR of 8.7% from 2026 to 2034; other analysts put the certification-market CAGR higher at 15.2% from 2026 to 2035, with the market rising from about USD 18.59 billion in 2025 toward USD 74.56 billion by 2035.
- Drivers: rising cyber threats, regulatory compliance requirements, and increased adoption of information security frameworks, plus regulations like the EU's NIS2 and GDPR and cloud expansion.
- IT relevance for you specifically: around 65% of IT service providers had obtained ISO 27001 certification by 2024 — your sector is the core adopter.

## Bottom line

- **Demand: high and rising fast** — certificates nearly doubled in one year; ~20% annual growth shows no sign of slowing. Every certified company needs auditors (internal) and certification bodies need lead auditors (external).
- **Salary: wide range.** Cert-only ≈ ₹7–8L; cert + real audit experience + GRC role = ₹13–23L+; senior/consulting = ₹30L+.
- **Future: one of the most durable cyber-compliance skills.** It's regulation-driven, not hype-driven, so it survives downturns.

**The key caveat (in numbers terms):** the certificate by itself moves you to the ~₹7.6L band. What moves you to ₹15–23L is the combination the job posts demand — Lead Auditor cert **+ logged audits + a second cred (CISA/CISM/ISO 27701)**. You're already building the experience leg at ACI; that's what makes the cert pay off.

Want me to pull current accredited (IRCA/PECB/Exemplar Global) Lead Auditor course options and fees in India?

**Vishnu:** i need a proper deep repoert 

my question is can i cleare the exam and get in the filed of not

**Claude:** _[tool: web_search]_

_[tool: web_search]_

_[tool: web_search]_

Now I have everything needed for an honest, specific assessment based on your profile. Let me write the full report.

_[tool: view]_

_[tool: create_file]_

_[tool: bash_tool]_

_[tool: present_files]_

10-section deep report, written specifically for your profile. Here is the summary:

**Can you clear the exam? Yes.** Your real ISMS experience is a bigger advantage than most candidates have. Study ISO 19011 hardest — that is what the scenario questions test, not ISO 27001 knowledge.

**The honest gap:** passing the exam gives you **Provisional Auditor** immediately, not full Lead Auditor. The full credential needs 5 years experience + 300 logged audit hours. That is 3–4 years from today — normal, not a blocker.

**The report covers:**
1. Your profile mapped against all 7 exam domains — where you are strong and where the gaps are
2. Exact exam format (80 questions, 70% pass, 3 hours, free retake)
3. Credential ladder with exact experience and audit-hour requirements per level
4. Your personal 6-week study plan with hours per week
5. Course options and costs in India (₹20,000–₹60,000)
6. Salary by career stage with exact sources and dates
7. Three career paths — corporate ISMS lead, consultant, certification body auditor
8. Your specific advantages and honest risks
9. 4-year timeline with salary bands
10. Final verdict table

**Single most important action right now:** finish ACI Stage 2 first. That is your proof of real project delivery — which is what makes the exam pass mean something to an employer.

**Vishnu:** what will be the demabd from now to after 5 years

**Claude:** _[tool: web_search]_

_[tool: web_search]_

_[tool: web_search]_

Now I have all the data. Here is the demand picture from 2026 to 2031, in numbers.

---

## Demand: 2026 → 2031

### The global numbers

The world currently needs an additional 4.8 million cybersecurity professionals — a figure that has grown by more than 40% in just two years. The global cybersecurity workforce would need to increase by 87% to meet current demand.

The ISO 27001 certification market is projected to grow from USD 21.42 billion in 2026 to USD 74.56 billion by 2035 at a CAGR of 15.2%. That means the ~100,000 certified companies growing at 20–25% annually will become roughly 250,000–300,000 certified companies by 2031. Every one of them needs an auditor.

### India specifically — the numbers are extreme

India has over 1 million unfilled cybersecurity vacancies — second only to China globally.

India's Global Capability Centers (GCCs) — over 2,100 now operational — are expected to create a million new jobs by 2030, but India already reports a 25–30% shortfall of mid-to-senior cybersecurity specialists.

GRC roles specifically — compliance audits and data protection — will be in high demand as businesses face increased scrutiny from regulators and customers.

### The India-specific accelerator nobody is talking about: DPDP

This is the single biggest demand driver for your field over the next 5 years in India specifically.

India's DPDP Rules were notified in November 2025, and 2026 is the year organizations must get serious about compliance — with full enforcement expected by mid-2027. Penalties for non-compliance can reach ₹250 crore (approximately $30 million USD) for serious violations.

What this means practically: every Indian IT company, every company handling Indian citizen data (including multinationals), will need compliance infrastructure built and audited. ISO 27001 + DPDP compliance overlap heavily — the same skills, same documents, same audit framework. Those who specialize in compliance audits and data protection will be in high demand as businesses face increased scrutiny.

### Year-by-year demand trajectory

| Year | What drives demand | ISO 27001 certificates globally (projected at ~20% growth) | India impact |
|------|------|------|------|
| 2026 | DPDP enforcement kicks in; Stage 2 audit wave | ~116,000 | Mass compliance rush across IT sector |
| 2027 | DPDP full enforcement deadline (May 2027); EU NIS2 ripple to Indian exporters | ~140,000 | GCC expansion + mandatory audits |
| 2028 | Surveillance audit cycle begins for all 2025–2026 certified companies | ~168,000 | Surveillance audits need auditors too |
| 2029 | Re-certification cycle (3-year cert renewal wave) | ~200,000 | Every 2026-certified company needs re-cert |
| 2030 | AI governance regulations expected globally; new ISO 42001 (AI) adoption | ~240,000 | GCCs at peak hiring; 1 million jobs target |
| 2031 | You at full Lead Auditor credential (per the timeline) | ~290,000 | Market at peak, you at peak readiness |

Over 80% of employers expect cybersecurity professionals to become even more important by 2030.

### The supply side — why this stays a seller's market

55% of cybersecurity teams are understaffed today, and 65% of organizations say unfilled positions exist — with 38% reporting it takes 3–6 months just to hire entry-level roles.

Even if training pipelines doubled, demand would still outpace supply.

**The key point for your career:** the shortage is not in generic cybersecurity — it is sharpest at the **mid-to-senior GRC/audit/compliance layer**, which is exactly where ISO 27001 Lead Auditor sits. Technical security roles (pen testing, SOC) have more candidates. Compliance audit roles have fewer — because they require both technical understanding and regulatory knowledge, and the DPDP wave is creating sudden, large demand for exactly that combination in India.

### Bottom line in one number

By 2031, when you would hold the full Lead Auditor credential, the global certified-company count will be roughly **3× today's number** (~290,000 vs ~97,000 now). India's compliance obligations will be at their heaviest enforcement phase. The supply of qualified auditors will still lag demand significantly. That is the window you are building toward.

**Vishnu:** COMMUNICATION PROCEDURE
DOCUMENT
Document Title:
COMMUNICATION PROCEDURE
Document ID:
ACI-PRO-002
Document Type:
PROCEDURE DOCUMENT
Document Version:
v0.1
Document Date:
01.01.2026
Authors:
NAVANEETHAN <navaneethan@aracreate.group>
REVIEW
Reviewed Date:
Version:
v1
Phase:
INITIAL RELEASE
Reviewed By:
APPROVAL
Navaneethan
Managing Director
Approver
Signature | Date
Aravinth Panch
Director
Signature | Date
1 GENERAL
1.1 INDEX
1 GENERAL	2
1.1 INDEX	2
1.2 REVIEW HISTORY	3
1.3 REVISION HISTORY	4
1.4 TERMS & ACRONYMS	5
2 PURPOSE & SCOPE	6
2.1 PURPOSE	6
2.2 SCOPE	6
3 INTERNAL COMMUNICATION	6
4 EXTERNAL COMMUNICATION	6
5 COMMUNICATION MATRIX (CLAUSE 7.4 REQUIREMENT)	6
8 REFERENCES	8
8.1 LINKS	8
8.2 TABLE	8
1.2 REVIEW HISTORY
VERSION
PAGE
DESCRIPTION OF CHANGES
DATE
Table 1. 
1.3 REVISION HISTORY
VERSION
PAGE
DESCRIPTION OF CHANGES
DATE
v0.1
All page
Initial Document
01.01.2026
Table 2. 
1.4 TERMS & ACRONYMS
TERM
ACRONYMS
DESCRIPTION
International Security Management System
ISMS
The International Security Management System (ISMS) refers to an organised framework of policies, procedures, and practices designed to manage and improve the security of an organisation's information, assets, and operations
Table 3. 
2 PURPOSE & SCOPE 
2.1 PURPOSE
To establish a structured framework for internal and external communications relevant to the Information Security Management System (ISMS) in accordance with ISO 27001:2022 Clause 7.4.
2.2 SCOPE
This procedure applies to all communications regarding security policies, incidents, and updates within araCreate and with external stakeholders.
3 INTERNAL COMMUNICATION
Internal security communications are primarily handled through the following channels:
Slack: Used for daily operational security updates, general awareness, and rapid response coordination.
Email: Reserved for formal management-level notifications, policy updates, and critical ISMS documentation.
The Managing Director holds primary responsibility for authorizing internal security communications, with situational delegation to relevant leads when necessary.
4 EXTERNAL COMMUNICATION
External communications regarding araCreate’s security posture or incidents are strictly controlled by Management. No employee is authorized to speak with external parties (clients, regulators, or public) about security matters without explicit approval from the Managing Director or Director.
5 COMMUNICATION MATRIX (CLAUSE 7.4 REQUIREMENT)
What to communicate?
When?
With whom?
Who shall communicate?
Process/Channel
ISMS Policy Updates
Upon Revision
All Employees
Managing Director
Email / Slack
Security Incidents
As Needed
Affected Parties
Management
Email / Formal Call
Security Awareness
As Needed
Staff
ISM Lead
Slack
Regulatory Changes
When applicable
Management / Authorities
Compliance Lead
Email
8 REFERENCES
8.1 LINKS
8.2 TABLE


under wwhic anexsuert i till come 

physical thenocal or what

**Claude:** **A.5.6 — Contact with Special Interest Groups** and primarily **A.5 Organizational controls** — specifically:

- **A.5.1** (Policies for information security) — policy communication
- **A.5.4** (Management responsibilities) — management-directed comms
- **A.5.24/A.5.26** (Incident management) — external incident communication

It is **not** physical (A.7) or technical (A.8). Communication procedures are purely organizational controls.

**Where it goes in your folder structure:**
- Document itself → `3-Procedures` (it's a procedure)
- Evidence of use (Slack logs, emails) → `4-Annex-A/A5/A.5.1` or `4-Annex-A/A5/A.5.24`

Also maps to **Cl 7.4** in your clause folders — reference it from `1-Clauses/7`.

**Vishnu:** i have a list of docmune in the google drive i need to explot i all as pdf in bulk how rto do that

**Claude:** Two options depending on how many files:

---

**Option 1 — Google Apps Script (best for bulk)**

1. Open [script.google.com](https://script.google.com) → New project
2. Paste this code:

```javascript
function exportAllToPDF() {
  const folder = DriveApp.getFolderById('YOUR_FOLDER_ID');
  const files = folder.getFiles();
  const pdfFolder = DriveApp.createFolder('PDF-Exports');
  
  while (files.hasNext()) {
    const file = files.next();
    const mimeType = file.getMimeType();
    
    if (mimeType === 'application/vnd.google-apps.document' ||
        mimeType === 'application/vnd.google-apps.spreadsheet' ||
        mimeType === 'application/vnd.google-apps.presentation') {
      
      const url = `https://docs.google.com/feeds/download/documents/export/Export?id=${file.getId()}&exportFormat=pdf`;
      const response = UrlFetchApp.fetch(url, {
        headers: { Authorization: `Bearer ${ScriptApp.getOAuthToken()}` }
      });
      pdfFolder.createFile(response).setName(file.getName() + '.pdf');
    }
  }
}
```

3. Replace `YOUR_FOLDER_ID` with the folder ID from the Drive URL (the string after `/folders/`)
4. Click **Run** → authorize → done. PDFs appear in a new `PDF-Exports` folder.

---

**Option 2 — Google Takeout (simpler, less control)**

1. Go to [takeout.google.com](https://takeout.google.com)
2. Select **Drive only** → click "All Drive data included" → choose specific folder
3. Under format, select **PDF**
4. Export and download the zip

---

**Recommendation:** use Option 1 if you want PDFs staying in Drive. Use Option 2 if you want to download them locally.

What type of files are they — Google Docs/Sheets, or uploaded Word/Excel files? That changes which method works better.
