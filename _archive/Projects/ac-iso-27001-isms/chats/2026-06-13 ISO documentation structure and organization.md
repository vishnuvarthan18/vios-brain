---
tags: chat
date: 2026-06-13
source: Claude personal account
uuid: 047bf420-9dd6-4b06-a591-1f3f6dd74a4d
---
# ISO documentation structure and organization

## Summary
**Conversation Overview**

The person is working on ISO/IEC 27001:2022 ISMS implementation for AraCreate India Pvt Ltd (ACI), an IT services and software development company. Key personnel mentioned include Vishnu (ISMS Coordinator, UX Designer and Security Lead), Navaneethan Kandaraj (MD/Approver), Pradeepa (HR/training owner), and James Jaganathan (external auditor). The conversation progressed through three major phases: designing the ideal ISMS folder structure, auditing the existing Google Drive ISMS document set, and actively reorganizing Drive files into a clean structure using browser automation.

On folder structure, the person initially wanted clause-by-clause nesting but accepted Claude's recommendation to organize by document type (Policies, Procedures, Records, Evidence) with a Master Register mapping clauses as columns — the rationale being that ISO 27001 documents are cross-cutting and nesting forces duplication. The agreed ID convention is type-prefixed sequential numbering: POL-xxx, PRO-xxx, REC-xxx, EVD-xxx, MAS-xxx. Claude produced a complete Master Register Excel file with three sheets (Master Register, NC-CAPA Tracker, Folder Map) pre-populated with IT-company sample data covering all clauses 4–10 and Annex A controls. When the person connected Google Drive, Claude read the existing ISMS files and identified critical gaps: Clause 7 folder entirely missing, SoA status unclear, duplicate/version-controlled documents (mr-v1/v2/v3, two legal registers), misfiled documents across clause folders, and inconsistent document ID conventions.

The active Drive reorganization created a fresh ACI-ISMS folder with seven top-level folders (0-Master, 1-Clauses, 2-Policies, 3-Procedures, 4-Annex-A, 5-Evidence, 6-Audit) and 17 subfolders including the previously missing clause-7. File moves completed so far: all old clause-4 contents (6 files) into new clause-4; old clause-5 contents fully re-sorted across clause-5, clause-6, clause-7, clause-9, clause-10, and A5-organization-control; old clause-6 contents moved with change-management-form correctly re-sorted to clause-8; old clause-8 contents split between new clause-8 and A8-technical-controls. Still pending: old clause-9, clause-10 contents; policies folder; NC folders (ACI-NC-001, ACI-NC-002); MRM records; manual folder; resources-evidence; process-matrix; and master-list.xlsx into 0-Master.

**Tool Knowledge**

For Google Drive navigation via Claude in Chrome, the Move dialog's "Suggested" folder list ordering is non-deterministic and changes between sessions — never assume a folder is at a fixed row position. Always take a screenshot after the dialog loads fully before clicking any row. The reliable navigation pattern is: open move dialog → double-click a known folder in Suggested to enter it → wait 4+ seconds for full load (screenshot to confirm "Loading…" is gone) → then click the target row. Drilling through All Locations works via: All locations → double-click My Drive → find "iso" folder → chevron to enter → iso-27001-prj → chevron → ACI-ISMS → chevron → target subfolder → Move button on the highlighted row.

The Drive SPA (single-page app) does not refresh the source folder view after a move completes — files appear to still be present even after a successful move. The only reliable verification method is to navigate away and reload the source folder URL directly. Successful moves show a toast confirmation ("X has been moved from Y to Z") when the dialog closes; if the dialog closes without a toast, the move likely failed. The selection toolbar (with the move-to icon at approximately x=411, y=123) takes 1–2 seconds to render after clicking a file — clicking the move icon immediately after file selection causes it to hit the filter bar behind the toolbar instead, opening a "Modified" dropdown. The fix is to wait 1 second after file selection, then click the move icon. Cmd+A selects all items in a folder view; Cmd+click toggles individual item selection for multi-select with exclusions. The New Folder dialog input field requires clicking at approximately (783, 362) and using Cmd+A then Delete to clear

## Chat

**Vishnu:** heyy i am going to mainatrain the whole iso realted thiss in a file so please me me a finla structure 


like 

clasue wise first then ofter that policy producres then ill evidace 

i need a cleare thing 

there shoube be a poper palce to maiantal all the things from mrm revioer nc all end to end

**Claude:** Here's a complete folder structure for end-to-end ISO 27001:2022 ISMS maintenance.

```
ACI-ISMS/
│
├── 00-Master-Index/
│   ├── ISMS-Document-Register.xlsx        (master list: doc ID, version, owner, date, status)
│   ├── Version-Control-Log.xlsx
│   └── ISMS-Scope-Statement.docx
│
├── 01-Clause-Wise/                         (ISO 27001 main clauses 4–10)
│   ├── C4-Context/
│   │   ├── 4.1-Internal-External-Issues.docx
│   │   ├── 4.2-Interested-Parties.docx
│   │   ├── 4.3-ISMS-Scope.docx
│   │   └── 4.4-ISMS-Process.docx
│   ├── C5-Leadership/
│   │   ├── 5.1-Leadership-Commitment.docx
│   │   ├── 5.2-Information-Security-Policy.docx
│   │   ├── 5.3-Roles-Responsibilities-Org-Chart.docx
│   ├── C6-Planning/
│   │   ├── 6.1.2-Risk-Assessment.xlsx
│   │   ├── 6.1.3-Risk-Treatment-Plan.xlsx
│   │   ├── 6.1.3-Statement-of-Applicability.xlsx
│   │   └── 6.2-Security-Objectives.docx
│   ├── C7-Support/
│   │   ├── 7.1-Resources.docx
│   │   ├── 7.2-Competence-Training.docx
│   │   ├── 7.3-Awareness.docx
│   │   ├── 7.4-Communication.docx
│   │   └── 7.5-Documented-Info-Control.docx
│   ├── C8-Operation/
│   │   ├── 8.1-Operational-Planning.docx
│   │   ├── 8.2-Risk-Assessment-Records.xlsx
│   │   └── 8.3-Risk-Treatment-Records.xlsx
│   ├── C9-Performance/
│   │   ├── 9.1-Monitoring-Measurement.xlsx
│   │   ├── 9.2-Internal-Audit/
│   │   └── 9.3-Management-Review-MRM/
│   └── C10-Improvement/
│       ├── 10.1-NC-Corrective-Action/
│       └── 10.2-Continual-Improvement.docx
│
├── 02-Policies-Procedures/
│   ├── Policies/
│   │   ├── Information-Security-Policy.docx
│   │   ├── Access-Control-Policy.docx
│   │   ├── Asset-Management-Policy.docx
│   │   ├── Cryptography-Policy.docx
│   │   ├── Supplier-Security-Policy.docx
│   │   ├── Incident-Management-Policy.docx
│   │   ├── BCP-Policy.docx
│   │   └── ... (one per applicable Annex A theme)
│   └── Procedures/
│       ├── Risk-Management-Procedure.docx
│       ├── Access-Provisioning-Procedure.docx
│       ├── Incident-Response-Procedure.docx
│       ├── Backup-Restore-Procedure.docx
│       ├── Change-Management-Procedure.docx
│       └── Internal-Audit-Procedure.docx
│
├── 03-Annex-A-Controls/                    (A.5–A.8, evidence per applicable control)
│   ├── A5-Organizational/
│   ├── A6-People/
│   ├── A7-Physical/
│   └── A8-Technological/
│       └── A.8.30-Outsourced-Development/   (your SoA-justified control)
│
├── 04-Evidence/
│   ├── Common-Evidence/                     (shared across NCs/clauses — no duplication)
│   │   ├── MRM-Records/
│   │   ├── Training-Attendance/
│   │   ├── Internal-Audit-Reports/
│   │   └── Risk-Assessment-Snapshots/
│   ├── Clause-Specific/
│   └── Control-Specific/
│
├── 05-Audit/
│   ├── Stage-1/
│   │   ├── Audit-Report.pdf
│   │   ├── NC-Closure/
│   │   │   ├── ACI-NC-001/                  (GR/01 — Clause 4.1)
│   │   │   ├── ACI-NC-002/                  (GR/02 — Clause 6.1.3 / A.8.30)
│   │   │   └── OBS-01/                      (Clause 5.1 — Board on org chart, non-blocking)
│   │   └── Correspondence/
│   ├── Stage-2/
│   └── Surveillance/                        (future annual audits)
│
└── 06-Registers-Logs/
    ├── Risk-Register.xlsx
    ├── Asset-Register.xlsx
    ├── Legal-Regulatory-Register.xlsx       (with version control / revision history)
    ├── NC-CAPA-Tracker.xlsx                 (all NCs end-to-end: ID, clause, root cause, action, owner, status, closure date)
    ├── Incident-Log.xlsx
    ├── Training-Register.xlsx
    └── Continual-Improvement-Log.xlsx
```

**Where each thing lives end-to-end:**

- **MRM** → master copy in `04-Evidence/Common-Evidence/MRM-Records/`, linked (not duplicated) from `01-Clause-Wise/C9-Performance/9.3-Management-Review-MRM/`. Reference all relevant NCs inside the MRM doc.
- **NCs** → tracked centrally in `06-Registers-Logs/NC-CAPA-Tracker.xlsx`; closure evidence per NC under `05-Audit/Stage-1/NC-Closure/ACI-NC-xxx/`.
- **Reviews / internal audits** → `Common-Evidence/Internal-Audit-Reports/`, surfaced under `C9.2`.
- **Policies vs Procedures** → kept in `02`, separate from clause docs so they're reusable; reference them from clause and Annex A files rather than copying.

**Two rules to enforce as you build:**

1. **One source of truth per artifact.** Shared evidence (MRM, training, audit records) lives once in `Common-Evidence`; everything else points to it. No copies in NC subfolders.
2. **Version control on every record** — doc ID, version number, revision history. The master register in `00` is the index; sweep `[Date]` placeholders and stale dates before any submission.

Want this as an actual folder tree created in your Google Drive, or a starter `ISMS-Document-Register.xlsx` with the rows pre-filled?

**Vishnu:** of a proper more engneered foramt to make it more simple and tell me jsut as a note what are the thigns need to be in that

**Claude:** Simpler structure — six top-level folders, flat where possible.

```
ACI-ISMS/
├── 0-Master/          → register, scope, version log
├── 1-Clauses/         → C4 to C10 (the ISO 27001 requirements)
├── 2-Policies/        → policies + procedures
├── 3-Controls/        → Annex A (A.5–A.8) evidence
├── 4-Evidence/        → Common-Evidence (MRM, training, audits) + specific
├── 5-Audit/           → Stage-1, Stage-2, NC-Closure, Surveillance
└── 6-Registers/       → all .xlsx trackers (risk, asset, NC, legal, incident)
```

**What must exist in each — note form:**

**0-Master**
- ISMS Document Register (every doc: ID, version, owner, date, status) — the index for everything
- ISMS Scope Statement
- Version Control Log

**1-Clauses** (one doc per sub-clause)
- C4: internal/external issues, interested parties, scope, ISMS process
- C5: leadership commitment, **Information Security Policy**, roles + org chart (Board must appear — OBS-01)
- C6: risk assessment, risk treatment plan, **SoA**, security objectives
- C7: resources, training/competence, awareness, communication, documented-info control
- C8: operational planning, risk assessment + treatment records
- C9: monitoring/measurement, internal audit, **MRM**
- C10: NC + corrective action, continual improvement

**2-Policies**
- Policies: one per applicable Annex A theme (Access Control, Asset, Crypto, Supplier, Incident, BCP…)
- Procedures: the "how" (Risk Mgmt, Access Provisioning, Incident Response, Backup, Change Mgmt, Internal Audit)

**3-Controls**
- Evidence per applicable Annex A control, grouped A5/A6/A7/A8
- Each control here must trace back to a "Yes" in the SoA (A.8.30 = your outsourced-dev justification)

**4-Evidence**
- `Common-Evidence/` — MRM records, training attendance, internal audit reports, risk snapshots. **Store once here; everything else links to it.**
- `Specific/` — anything tied to a single clause or control

**5-Audit**
- Stage-1: audit report, NC-Closure (`ACI-NC-001`, `ACI-NC-002`, `OBS-01`), correspondence
- Stage-2: (awaiting James's schedule)
- Surveillance: future annual audits

**6-Registers** (all spreadsheets, one source of truth each)
- Risk Register
- Asset Register
- Legal & Regulatory Register (with revision history)
- NC-CAPA Tracker — every NC end-to-end: ID, clause, root cause, action, owner, status, closure date
- Incident Log
- Training Register

**Three rules that keep it clean:**
1. Shared evidence lives once in `Common-Evidence` — never duplicate into NC folders.
2. Every record carries doc ID + version + revision history; the Master register indexes all of it.
3. MRM and training docs must reference **all** relevant NCs, not just one. Sweep `[Date]` placeholders before any submission.

Want me to create this skeleton in Google Drive, or generate the `ISMS-Document-Register.xlsx` pre-filled with these rows?

**Vishnu:** no i need to connect all to be one it need to be like a  tree fomr top eo ened 
but only the short cut to be there 

that need to be seperate alos

**Claude:** Want this as a single master file where the tree is laid out top-to-bottom, and each line is a shortcut (clickable link) to the actual Drive folder/doc — kept separate from the real files?

One clarifying question before I build it:

**Where should this master tree live — as a Google Sheet, a Google Doc, or just a folder of shortcuts in Drive?**

- **Google Sheet** → best if you want columns (link, owner, version, status) beside each node. Sortable, filterable.
- **Google Doc** → best if you want a clean visual tree (indented, collapsible-looking) that reads like a map.
- **Shortcut folder in Drive** → Drive's native shortcut feature; mirrors the tree as actual clickable shortcut items, real files stay elsewhere.

**Vishnu:** no now i am just talking about the file structere

**Claude:** Got it — just the structure. One master tree, top to bottom, where every node is a shortcut to the real thing, and the actual files live separately.

```
ACI-ISMS-MASTER/                        ← shortcuts only, this is your map
│
├── ▸ 0-Master
│   ├── → ISMS-Document-Register
│   ├── → ISMS-Scope-Statement
│   └── → Version-Control-Log
│
├── ▸ 1-Clauses
│   ├── → C4-Context
│   ├── → C5-Leadership
│   ├── → C6-Planning
│   ├── → C7-Support
│   ├── → C8-Operation
│   ├── → C9-Performance
│   └── → C10-Improvement
│
├── ▸ 2-Policies
│   ├── → Policies
│   └── → Procedures
│
├── ▸ 3-Controls
│   ├── → A5-Organizational
│   ├── → A6-People
│   ├── → A7-Physical
│   └── → A8-Technological
│
├── ▸ 4-Evidence
│   ├── → Common-Evidence
│   └── → Specific
│
├── ▸ 5-Audit
│   ├── → Stage-1
│   ├── → Stage-2
│   └── → Surveillance
│
└── ▸ 6-Registers
    ├── → Risk-Register
    ├── → Asset-Register
    ├── → Legal-Register
    ├── → NC-CAPA-Tracker
    ├── → Incident-Log
    └── → Training-Register
```

**The two-layer split:**

- `ACI-ISMS-MASTER/` = the tree above. Contains **only shortcuts** (`→`). Nothing real lives here. This is the single entry point you navigate from.
- `ACI-ISMS-FILES/` = the real repository. All actual folders and documents live here, organized the same way. You rarely open this directly — you reach everything through the master tree.

So: `→` = shortcut (in MASTER), `▸` = a shortcut to a folder. Every leaf points into `ACI-ISMS-FILES/`.

This gives you one top-to-end map, clickable end to end, with the real files cleanly separated so the structure never gets messy.

Want me to build both layers in Drive — the real `ACI-ISMS-FILES/` tree and the `ACI-ISMS-MASTER/` shortcut mirror pointing into it?

**Vishnu:** i am asking to map to end to end there ill be policy producre for each clais i need like that

**Claude:** Now I understand — you want the tree to show the *real relationships* end-to-end: each clause → its policies/procedures → its controls → its evidence, all nested under the clause itself. Not separate top-level folders, but everything that belongs to a clause sitting *under* that clause.

```
ACI-ISMS/
│
├── 0-Master
│   ├── ISMS-Document-Register
│   ├── ISMS-Scope
│   └── Version-Log
│
├── C4-Context
│   ├── Policy/Procedure  → (Context & Scope definition doc)
│   ├── Records           → 4.1 Issues, 4.2 Interested Parties, 4.3 Scope
│   └── Evidence          → (shortcut → Common-Evidence if shared)
│
├── C5-Leadership
│   ├── Policy            → Information-Security-Policy
│   ├── Procedure         → Roles & Responsibilities
│   ├── Records           → Org-Chart (Board incl. — OBS-01)
│   └── Evidence          → Leadership commitment proof
│
├── C6-Planning
│   ├── Policy            → Risk-Management-Policy
│   ├── Procedure         → Risk-Assessment-Procedure
│   ├── Records           → Risk-Assessment, Risk-Treatment-Plan, SoA, Objectives
│   └── Evidence          → A.8.30 justification (NC-002 link)
│
├── C7-Support
│   ├── Policy            → Competence & Awareness Policy
│   ├── Procedure         → Training-Procedure, Doc-Control-Procedure
│   ├── Records           → Training-Register, Competence-Matrix
│   └── Evidence          → shortcut → Common-Evidence/Training-Attendance
│
├── C8-Operation
│   ├── Policy            → (operational security policies)
│   ├── Procedure         → Change-Mgmt, Backup, Access-Provisioning
│   ├── Records           → Risk-Assessment + Treatment records
│   └── Evidence          → operational logs
│
├── C9-Performance
│   ├── Policy            → Monitoring & Measurement Policy
│   ├── Procedure         → Internal-Audit-Procedure
│   ├── Records           → Metrics, Internal-Audit-Reports, MRM
│   └── Evidence          → shortcut → Common-Evidence/MRM + Audit-Reports
│
├── C10-Improvement
│   ├── Policy            → Corrective-Action Policy
│   ├── Procedure         → NC-CAPA-Procedure
│   ├── Records           → NC-CAPA-Tracker
│   └── Evidence          → NC-Closure (ACI-NC-001, ACI-NC-002, OBS-01)
│
├── Annex-A-Controls
│   ├── A5-Organizational   → policies/procedures/evidence per control
│   ├── A6-People
│   ├── A7-Physical
│   └── A8-Technological     → A.8.30-Outsourced-Dev (links to C6 SoA + NC-002)
│
├── Common-Evidence          ← shared, stored once, everything links here
│   ├── MRM-Records
│   ├── Training-Attendance
│   ├── Internal-Audit-Reports
│   └── Risk-Snapshots
│
└── Audit
    ├── Stage-1  → Report, NC-Closure, Correspondence
    ├── Stage-2
    └── Surveillance
```

**The mapping logic — how it connects end-to-end:**

- Each **clause** is a self-contained unit with the same four children: `Policy → Procedure → Records → Evidence`. Open one clause and you see everything that satisfies it, in order.
- **Annex A controls** stay separate (they're a different axis — themes, not requirements), but each control **links back** to the clause that drives it. A.8.30 → points to C6 (SoA) and C10 (NC-002).
- **Common-Evidence** is the one exception to nesting: shared records (MRM, training, audits) live once here, and clauses reach them by shortcut. Prevents duplication.
- **Audit** stays separate because it spans all clauses — NC-Closure links into whichever clause each NC belongs to.

**One decision to make:** a single policy often satisfies *multiple* clauses (e.g. the InfoSec Policy touches C5, C6, C7). Two ways to handle it:

- **Store the policy once** in a `Policies/` home, and put a **shortcut** under each clause it serves. Cleaner, no duplication — recommended.
- **Nest the real file** under its primary clause and shortcut from the others.

Recommend the first. Want me to build this clause-mapped tree in Drive with the shortcut links wired in?

**Vishnu:** is this a good way or like it sneed to be like this in producres all the profdufed in policy all the policy whi is good way to maintain

**Claude:** Two valid models. Here's the call:

**Recommended: group by document type, map by reference — not by clause.**

```
ACI-ISMS/
├── 0-Master           → Register, Scope, Version-Log
├── 1-Policies         → ALL policies together
├── 2-Procedures       → ALL procedures together
├── 3-Records          → ALL records/registers together
├── 4-Evidence         → Common-Evidence + specific
├── 5-Annex-A          → control evidence
└── 6-Audit            → Stage-1, Stage-2, NC-Closure
```

The clause mapping does **not** live in the folder tree. It lives as a **column in your Master Register**:

```
Doc-ID │ Document Name              │ Type      │ Clause(s)      │ Annex-A      │ Version │ Owner
P-01   │ Information Security Policy │ Policy    │ 5.2, 6.1, 7   │ A.5.1        │ 2.0     │ Nava
PR-04  │ Internal Audit Procedure    │ Procedure │ 9.2           │ A.5.35       │ 1.1     │ Vishnu
R-02   │ Statement of Applicability  │ Record    │ 6.1.3         │ all          │ 3.0     │ Vishnu
```

**Why this is the better way:**

- **One policy serves many clauses.** InfoSec Policy touches 5.2, 6.1, 7.3, and a dozen Annex A controls. If you nest folders by clause, that one file has to be duplicated or shortcut into 15 places — and the moment you update it, you're chasing copies. Grouping by type means it lives **once**.
- **Auditors read by clause, but ISMS is maintained by type.** James audits "show me clause 9.2 evidence" — you answer that from the Register's clause column, which points to the real file. You don't reorganize your whole repo around how he happens to traverse it.
- **The Register is the map.** Filter by clause → see every doc satisfying it. Filter by type → see all policies. Filter by Annex-A → control coverage. One spreadsheet replaces the entire nested-folder problem.

**When clause-nesting (your previous tree) would be right:** only if each clause had its *own dedicated* policy/procedure with zero overlap. ISO 27001 doesn't work that way — documents are cross-cutting. So nesting forces duplication.

**Bottom line:** folders by type (flat, clean, no duplication) + Master Register with a Clause column doing the end-to-end mapping. That's how maintained ISMS repos are actually run.

Want me to build the type-grouped tree and generate the Master Register `.xlsx` with the `Doc-ID / Type / Clause / Annex-A / Version / Owner / Status` columns ready to fill?

**Vishnu:** yes we usually use like pol-001 this is how our id need to be 


so first me a total master document where shoule have all the vlasuse policy peroducre and all end to end not to miss a single palve enevn vidacne 


aslo give me some data like we are a it company keep that in mind

**Claude:** I'll build the complete Master Register as a spreadsheet — every clause, policy, procedure, record, Annex A control, and evidence item, end-to-end, with your `POL-001` ID convention and IT-company sample data filled in.

_[tool: view]_

Building the master register now — IT company context, `POL-001` style IDs, end-to-end from clause to evidence.

_[tool: create_file]_

_[tool: bash_tool]_

_[tool: bash_tool]_

_[tool: present_files]_

Done. Three sheets, end-to-end, nothing missed:

**Sheet 1 — Master Register.** The full end-to-end map. Clause 4 → 10, each with its policies, procedures, records, and evidence underneath. Every row has: `Doc ID | Name | Type | Clause | Annex A | Owner | Version | Status | Location`. `POL-001` convention applied. IT-company sample data filled in (SDLC procedure, access provisioning, change/backup, A.8.30 outsourced dev, etc.). Your real NCs are wired in — NC-001 → REC-003, NC-002 → REC-008/EVD-002, OBS-01 → REC-005.

**Sheet 2 — NC-CAPA Tracker.** All three findings end-to-end: ID, source, clause, root cause, action, owner, status, closure date.

**Sheet 3 — Folder Map.** The tree, so the file structure and the register stay in sync.

**What the register gives you that folders can't:** filter the Clause column → every doc satisfying that clause appears, regardless of which folder it physically sits in. One policy serving five clauses shows up under all five without being duplicated.

**Three things to know about the sample data:**
- IDs, versions, and owners (Vishnu / Nava / Pradeepa / Technical / Ariv) are placeholders — adjust to your actuals.
- I mapped Annex A controls to the most common IT-company set. Your **SoA is the source of truth** — if a control is "No" there, drop its row here.
- Document statuses are guesses except the two closed NCs. Sweep them against reality.

Want me to add the **Asset Register** and **Incident Log** as two more sheets, or push this structure into your Google Drive?

**Vishnu:** ok lets do the whole sheet wiht all the poepr cahange here it slef tell me what are thethings you need to do it full corect a audit ready docuemnt

**Claude:** To make this a truly audit-ready master document — not a sample — I need your real data. Here's exactly what I need, grouped so you can answer fast.

**1. SoA — the single most important input**
The SoA decides which Annex A controls are in scope. Everything in the register's control column must match it. Either:
- Share/upload your SoA (or its Drive link), **or**
- Tell me which of the 93 Annex A controls are marked "Applicable = Yes."

Without this I'm guessing at controls.

**2. Your actual document inventory**
For every policy, procedure, and record that already exists, I need:
- Real **Doc ID** (your existing numbering, if any) — else I assign POL/PRO/REC sequentially
- Real **document name** (as titled in Drive)
- **Owner** (real person per doc)
- **Version** and **approval date**
- **Status** (Approved / Draft / In Review)

Fastest path: point me at the Drive folder and I'll pull the actual file list.

**3. People & roles**
Confirm the real names behind each function:
- ISMS Coordinator (Vishnu — confirmed)
- MD / Approver (Navaneethan — confirmed)
- Who owns: HR/training, IT/infrastructure, software development, legal/compliance?
(I used Pradeepa / Technical / Ariv as placeholders — give me real names + titles.)

**4. Dates that must be real**
- Stage-1 audit date
- NC closure dates (I have 2026-05-20 as placeholder)
- Last MRM date, last internal audit date
- Current SoA version + date

**5. Company facts for the header**
- Legal entity name and certification scope statement (the exact scope wording)
- Certification body name (and confirm auditor = James Jaganathan)
- ISMS effective date / Stage-1 date

---

**The single fastest way to do this right:** give me access to the Drive folder holding your current ISMS docs. I'll read the real file list, versions, and owners, then populate every row from source instead of you typing it all.

**Pick one:**

_[tool: ask_user_input_v0]_

**Vishnu:** Q: How do you want to feed me the real data so I can finish the audit-ready register?
A: Connect my Google Drive — pull the real files

**Claude:** I'll pull your real ISMS files from Drive. Let me load the Drive tools and find the folder.

_[tool: tool_search]_

Let me find your ISMS folder. Searching for the evidence/audit folder structure.

_[tool: Google Drive:search_files]_

Found your real ISMS files. Let me pull the rest of the inventory and read the key documents — scope, roles, legal register, and the NC folders.

_[tool: Google Drive:search_files]_

The root has mixed content. The ISMS files I need are the specific ones identified. Let me read the key source documents — scope, roles framework, roles matrix, and legal register — to populate real data.

_[tool: Google Drive:read_file_content]_

**Vishnu:** i will five you link for rthe file then do the preocsea and you bettwe be a top levler auditer adn tell me what is missinf and wirng we can make corecr for audit ready

**Claude:** Understood. Send me the links and I'll go through them as an auditor would — checking each against the ISO 27001:2022 requirement it's meant to satisfy, flagging what's missing, wrong, or audit-risky, and telling you exactly what to fix.

Before you send them, here's what I already caught in the **Scope Document** (ISMS-DOC-04.3 v1.0) from reading it just now — so you see the level I'll work at:

**Findings on the Scope Document:**

1. **Doc ID convention mismatch.** This uses `ISMS-DOC-04.3`, but you told me your standard is `POL-001` style. Pick one and apply it everywhere — an auditor notices inconsistent IDs immediately. If `ISMS-DOC-[clause]` is your real convention, the register should follow that, not `POL-xxx`.

2. **SoA version pinned to 1.0 — verify this is still true.** Section 5 says all controls are evaluated against "Statement of Applicability Version 1.0." But NC-002 (GR/02) required you to add the A.8.30 justification to the SoA. If you edited the SoA to close that NC, its version should have incremented (1.1 or 2.0). A scope doc pointing at "v1.0" while the live SoA is a later version is a **traceability gap** — fix the version reference here.

3. **Role title inconsistency — audit risk.** Here Vishnu is "UX Designer & Security Lead." In memory/other docs the function is "ISMS Coordinator." Auditors check that the person named in records matches the role defined in the Roles Matrix. Confirm one title and make every document agree.

4. **"Last Reviewed: May 14, 2026" with no next-review date and no revision history table.** Clause 7.5.3 expects documented info to show review status. Add a revision history block (version, date, author, change description) — you flagged this yourself as a recurring gap.

5. **Exclusions wording is slightly risky.** "There are no specific exclusions" is fine, but pairing it with "all controls evaluated for relevance" is good — just make sure the SoA actually marks every excluded control with a justification. Stage-2 auditors test this directly.

**Now send the links.** For each file tell me, if you can, what clause/control it's meant to satisfy. I'll produce a finding list per document — **Missing / Wrong / Audit-risk / OK** — then fold the corrections into the master register so the whole thing is Stage-2 ready.

Which files first — SoA, Roles Matrix, Legal Register, MRM, or the NC closure packs?

**Vishnu:** https://drive.google.com/drive/folders/1KxTnmIgEwoYmPLRL27sekamxjA5fUaZ8?usp=drive_link

this is the whole iso file we have is this only

**Claude:** _[tool: Google Drive:search_files]_

This folder (`1-input`, `2-output`, `3-pm`, `4-media`) looks like a project workspace, not your ISMS document set. The real ISMS files I found earlier sit in a different location. Let me check the subfolders to confirm what's actually here, then give you the verdict.

_[tool: Google Drive:search_files]_

This is the real ISMS set. There's a `policies` folder, a `manual`, a `master-list.xlsx`, process matrix, resources evidence, plus `iso-audit`, `first-audit-nc`, `documents`, and `audit-report`. Let me go deeper into the document folders and read the master list to see your actual inventory.

_[tool: Google Drive:search_files]_

Good structure — clause folders 4,5,6,8,9,10 and Annex A control folders (org/people/physical/technical). I notice **clause-7 is missing**. Let me read the master-list.xlsx (your existing inventory) and check the clause-7 gap plus the policies folders.

_[tool: Google Drive:read_file_content]_

This is a comprehensive policy list — 80+ policies and procedures. The data is messy (two columns jammed together), but the content is substantial. Let me read the manual folder and the clause-7 question, then check the process matrix, which usually holds the clause-by-clause mapping.

_[tool: Google Drive:search_files]_

_[tool: Google Drive:read_file_content]_

I have a clear picture now. Here's my auditor's read on what you have — and the gaps that need fixing before Stage 2.

**Verdict: No, this isn't all you need — but you have more than you think. The problem is organization and traceability, not missing content.**

## What you have (confirmed real)

- **Scope document** (ISMS-DOC-04.3 v1.0) — solid, defines services, boundaries, location
- **Clause folders** 4, 5, 6, 8, 9, 10 under `iso-audit`
- **Annex A control folders** — organization / people / physical / technical (all four themes)
- **80+ policies and procedures** listed in master-list.xlsx
- **Process Interaction Matrix** (Clause 4.4) — genuinely good, maps 20 processes end-to-end
- **Roles & Process Framework** + **Roles Matrix** (Clause 5.3)
- **Legal Compliance Evaluation Record** (two versions)
- **NC folders** — ACI-NC-001, ACI-NC-002, common-evidence
- **MRM** (mr-v1/v2/v3), resources evidence, audit-report folder

## Gaps an auditor will catch — fix these

**1. Clause 7 folder is missing entirely.** You have clause-4, 5, 6, 8, 9, 10 — **no clause-7 (Support)**. Clause 7 covers resources, competence, awareness, communication, and documented information (7.5). This is a mandatory clause. Either the evidence is misfiled elsewhere or it doesn't exist. **This is a likely Stage-2 nonconformity.** Create clause-7 and populate: competence records, training attendance, awareness evidence, communication log, documented-info control procedure.

**2. No Statement of Applicability found in the clause-6 folder.** The SoA is the single most-checked document in the entire audit. I see policies and a risk approach, but no confirmed SoA file. If it exists, point me to it. If not — **critical blocker.** It must list all 93 Annex A controls with applicable Yes/No + justification (and the A.8.30 justification from NC-002).

**3. Massive policy duplication / no version control.** The master-list has the same policy themes repeated (multiple "business continuity" policies, two incident management policies, IPR policy listed twice). The list is also a jumbled two-column dump with typos ("encription", "segration", "informat5ion"). An auditor reading this sees an uncontrolled document set. **Each policy needs: one canonical version, a doc ID, version number, owner, approval date.**

**4. No master register tying clause → policy → procedure → evidence.** You have the pieces in separate folders but nothing maps them together. When James says "show me how you satisfy 9.2," you need one row that points to the procedure + the evidence. That's exactly the register I'm building.

**5. Version sprawl on key records.** mr-v1/v2/v3 (three MRM versions), two legal registers, "Untitled spreadsheet," "table.xlsx" — unclear which is authoritative. Auditors flag ambiguous document control under 7.5.3. Mark one as current, archive the rest.

**6. Clause 7.5 documented-information procedure.** "control of documentation procedure" is in the list, but with the version chaos above, I need to verify it's actually enforced.

## What I need to finish the audit-ready register

I can read the Google native files directly. The `.xlsx` files (master-list, process matrix, resources evidence) I can read via the content tool as I just did. To build the real register, let me now read:

- **clause-7 contents** — confirm it truly doesn't exist
- **the SoA** — find it or confirm it's missing
- **clause-4/5/6 actual file contents** — to capture real doc names, versions, owners

Want me to keep pulling — read every clause folder and the policies subfolders, find or rule out the SoA and clause-7, then build the complete register with real file names, real IDs, and a **Findings column** marking each row OK / Missing / Fix? That gives you the audit-ready document and the gap list in one file.

**Vishnu:** no lets now focus on first arrage the gooel drive first then we can move to this register work please do ath connect to the google chomre claude

**Claude:** I can't drive Chrome to reorganize your Drive — but I don't need to. The Google Drive tools I'm already connected to can move and rename files directly, which is faster and less error-prone than browser clicking.

Before I touch anything, one hard rule on my side: **I won't delete, or change sharing/permissions on, any file.** Moving and renaming is fine. Archiving duplicates means moving them to an `_Archive` folder, not deleting them — you delete only after you've confirmed.

Here's the cleanup plan I'd execute. Confirm the target structure and I'll start moving real files into it:

**Proposed clean structure (inside your ISO root):**
```
ACI-ISMS/
├── 0-Master          → master-list, scope, register, version log
├── 1-Clauses
│   ├── clause-4 ... clause-10   (CREATE clause-7 — currently missing)
├── 2-Policies        → canonical policies (POL-xxx), dedup
├── 3-Procedures      → procedures (PRO-xxx)
├── 4-Annex-A         → organization / people / physical / technical
├── 5-Evidence        → common-evidence + resources evidence
├── 6-Audit           → audit-report, first-audit-nc (NC-001/002/OBS-01)
└── _Archive          → mr-v1, mr-v2, old legal register, Untitled, table.xlsx
```

Two things I need you to decide before I move anything:

_[tool: ask_user_input_v0]_

**Vishnu:** Q: Where should I build the clean structure?
A: Create a fresh ACI-ISMS folder and move files into it

Q: How do I handle duplicates (mr-v1/v2, two legal registers, Untitled/table.xlsx)?
A: leave them and in final give me the peport i will cleare that manulay

**Claude:** Clear. Fresh `ACI-ISMS` folder, real files moved in, duplicates left untouched (I'll list them in a report for you to clean manually).

One blocker on my side I have to flag before moving anything: **the Google Drive tools I'm connected to can search, read, copy, and get metadata — but they don't expose a "move file" or "create folder" operation.** I can *copy* files (which duplicates them) but cannot relocate originals or make new folders through this connector.

So reorganizing the actual Drive needs Claude in Chrome — driving the browser to create folders and drag files. Let me connect that.

_[tool: tool_search]_

Chrome tools loaded. Let me find your connected browser. Per the connection step, I need you to pick which browser to use.

_[tool: Claude in Chrome:list_connected_browsers]_

One browser connected. Confirm which to use:

_[tool: ask_user_input_v0]_

**Vishnu:** Q: Which Chrome browser should I use to reorganize your Drive?
A: Open a confirmation screen in every connected Chrome extension and let me select the right one there.

**Claude:** _[tool: Claude in Chrome:switch_browser]_

Only Browser 1 is connected — the confirmation broadcast found no others. So that's the one we'll use. Let me select it.

_[tool: Claude in Chrome:select_browser]_

Connected. Now let me open your ISO root folder in Drive to start the reorganization.

_[tool: Claude in Chrome:navigate]_

_[tool: tool_search]_

_[tool: Claude in Chrome:tabs_context_mcp]_

Tab ready. Before I start clicking through Drive, a quick reality check on the most efficient path — because doing this entirely by browser drag-and-drop across 80+ files and a dozen folders is slow and error-prone.

Let me open your Drive root so I can see the current state and start creating the clean structure.

_[tool: Claude in Chrome:navigate]_

_[tool: Claude in Chrome:browser_batch]_

The browser action was blocked — looks like the permission prompt in Chrome was declined, or the extension needs you to grant access for this site.

Before we go further, let me be straight about whether the browser route is even the right call here, because I don't want to burn your time on a slow, fragile process.

**Honest assessment of doing this by browser:**

Reorganizing 80+ files plus a dozen folders by driving Chrome means hundreds of individual drag-drop and right-click-move actions, each with a screenshot to verify. It's slow, and one misfire can drop a file in the wrong folder. For a one-time structural cleanup, that's a lot of risk for the payoff.

**The faster, safer path — and it uses the Drive tools I already have working:**

The Google Drive connector *can* copy files into new locations. What it's missing is "create folder." But here's the thing — **you can create the empty folder skeleton in 2 minutes by hand** (right-click → New folder, twelve times), and then **I populate them** by copying each real file into the right folder via the Drive API. No browser, no dragging, no permission fights. Copies preserve originals, so nothing is at risk, and your duplicates stay put for the manual report you wanted.

So the division of labor:
- **You:** create the empty `ACI-ISMS` folder tree (I'll give you the exact folder names to make).
- **Me:** copy every real file into its correct folder via Drive tools, then hand you the cleanup report.

This is genuinely faster than me clicking through Chrome, and far less error-prone.

**Want to go that way?** If yes, I'll give you the precise folder list to create now. If you'd rather I push through with Chrome regardless, re-grant the extension permission on the Drive tab and tell me — I'll resume the browser route.

Which way?

**Vishnu:** i cant undersant what you asking

**Claude:** Let me say it simply.

I tried to control your Chrome browser to move files, but it got blocked — Chrome refused permission.

So I'm giving you two choices:

**Choice A — You make the empty folders, I fill them**
- You go into Google Drive and create empty folders yourself (right-click → New folder). Takes 2 minutes.
- I then copy your files into the correct folders automatically.
- Safe, fast, nothing gets lost.

**Choice B — I keep trying with Chrome**
- You re-allow the Chrome permission that got blocked.
- I do everything by controlling your browser.
- Slower and more likely to make mistakes.

I recommend **Choice A**.

One question so I know what's actually possible right now:

_[tool: ask_user_input_v0]_

**Vishnu:** Q: Which way do you want to do the Drive cleanup?
A: Choice B — keep using Chrome, I'll allow permission

**Claude:** Choice B it is. Before I retry, here's what you need to do on your side:

In Chrome, the Claude extension needs permission to act on the Drive tab. When I take the next action, a permission prompt will appear (or the extension icon will need a click to allow access on `drive.google.com`). **Click Allow / Connect.**

Once you've done that, tell me "go" and I'll resume — open your ISO root folder, screenshot the current state, and start building the clean `ACI-ISMS` structure.

**Vishnu:** i have allowed

**Claude:** _[tool: Claude in Chrome:browser_batch]_

Browser is working now. I can see your `iso-27001-prj` folder with the four subfolders (1-input, 2-output, 3-pm, 4-media) and the stray "Untitled" spreadsheet.

I'll start by creating the clean `ACI-ISMS` folder here. Let me create the first folder via the New button.

_[tool: Claude in Chrome:browser_batch]_

The New menu is open. Let me click "New folder" and name it `ACI-ISMS`.

_[tool: Claude in Chrome:browser_batch]_

The dialog opened but the text didn't land in the field — the "New folder" box still shows the placeholder "Untitled folder." I need to click into the text field first, then type. Let me fix that.

_[tool: Claude in Chrome:browser_batch]_

"ACI-ISMS" is in the field. Creating it now.

_[tool: Claude in Chrome:browser_batch]_

"Working…" is showing — the folder is being created. Let me refresh the view to confirm it appeared.

_[tool: Claude in Chrome:browser_batch]_

`ACI-ISMS` folder is created and visible. Now let me open it and build the subfolder structure inside. Double-click to enter.

_[tool: Claude in Chrome:browser_batch]_

Inside the empty `ACI-ISMS` folder. Now I'll create the seven top-level subfolders. I'll use the keyboard shortcut for New folder (in Drive it's pressing the New button → New folder). Let me create the first one, `0-Master`, then repeat for the rest.

I'll batch each folder creation. Starting with `0-Master`:

_[tool: Claude in Chrome:browser_batch]_

The flow works reliably now. "0-Master" is typed. I'll create it and then repeat the exact same sequence for the remaining six folders. Let me create this one and chain the next.

_[tool: Claude in Chrome:browser_batch]_

Both `0-Master` and `1-Clauses` created. The sequence is solid. `1-Clauses` is currently selected (highlighted) — I'll click empty space first to deselect, then create the remaining five: `2-Policies`, `3-Procedures`, `4-Annex-A`, `5-Evidence`, `6-Audit`.

_[tool: Claude in Chrome:browser_batch]_

Four folders now: 0-Master, 1-Clauses, 2-Policies, 3-Procedures. Chaining works perfectly. Let me create the last three: `4-Annex-A`, `5-Evidence`, `6-Audit`.

_[tool: Claude in Chrome:browser_batch]_

All seven top-level folders created: `0-Master`, `1-Clauses`, `2-Policies`, `3-Procedures`, `4-Annex-A`, `5-Evidence`, `6-Audit`.

Next I need to build the subfolders inside `1-Clauses` (clause-4 through clause-10, **including the missing clause-7**) and inside `4-Annex-A` (organization / people / physical / technical). Then move your existing files in.

Before I keep going deep, let me confirm the subfolder plan so I don't have to redo it:

**Inside `1-Clauses`:** clause-4, clause-5, clause-6, **clause-7** (new), clause-8, clause-9, clause-10

**Inside `4-Annex-A`:** A5-Organizational, A6-People, A7-Physical, A8-Technological

**Inside `5-Evidence`:** Common-Evidence, Specific

**Inside `6-Audit`:** Stage-1, Stage-2, first-audit-nc

That's 17 more subfolders. Want me to create all of them now in one pass, then start moving your existing files (clause folders, policies, NC packs, scope doc, etc.) into the right places? Or pause here so you can eyeball the top level first?

**Vishnu:** do all

**Claude:** I'll create all 17 subfolders, then move your existing files in. Starting with the `1-Clauses` subfolders. Let me open that folder first.

_[tool: Claude in Chrome:browser_batch]_

Inside `1-Clauses`. Creating the seven clause subfolders. I'll batch the first four (clause-4, 5, 6, 7) then the rest.

_[tool: Claude in Chrome:browser_batch]_

clause-4, 5, 6, 7 done (clause-7 now exists — the gap is filled). Creating clause-8, 9, 10.

_[tool: Claude in Chrome:browser_batch]_

All seven clause folders done: clause-4 through clause-10. Now to `4-Annex-A` for its four control theme folders. Let me navigate back up and into 4-Annex-A.

_[tool: Claude in Chrome:browser_batch]_

Inside `4-Annex-A`. Creating the four control theme folders, matching your existing naming (organization-control, people-controls, physical-controls, technical-controls).

_[tool: Claude in Chrome:browser_batch]_

All four Annex A theme folders created. Now `5-Evidence` subfolders (Common-Evidence, Specific) and `6-Audit` subfolders (Stage-1, Stage-2, first-audit-nc). Navigating to 5-Evidence.

_[tool: Claude in Chrome:browser_batch]_

Still in 4-Annex-A. Let me click the ACI-ISMS breadcrumb to go up, then into 5-Evidence.

_[tool: Claude in Chrome:browser_batch]_

Inside `5-Evidence`. Creating Common-Evidence and Specific.

_[tool: Claude in Chrome:browser_batch]_

The folder shows empty after refresh — that's odd. The "Drop files here" empty state suggests the two folders may not have been created, or the page hasn't reloaded the contents. Let me reload once more and check.

_[tool: Claude in Chrome:browser_batch]_

`5-Evidence` is genuinely empty — the Common-Evidence and Specific folders didn't get created. Looking back at that batch: after the F5 reload, the page state changed and the icon positions shifted (the earlier screenshot showed the toolbar had moved), so the clicks likely landed in the wrong place and the dialogs never opened correctly.

Let me create the two folders cleanly, one at a time, verifying each.

_[tool: Claude in Chrome:browser_batch]_

I see the issue — the menu layout shifted. "New folder" is at y=77 here (not y=75 as before, close enough), but the menu items are at different positions than my earlier batch assumed. The menu is open correctly now. Let me click New folder and create Common-Evidence.

_[tool: Claude in Chrome:browser_batch]_

"Common-Evidence" is in the field. The earlier batch failed because after F5 the New-button menu positions shifted. I'll create this, then do Specific in a separate verified step (no F5 mid-batch this time).

_[tool: Claude in Chrome:browser_batch]_

Common-Evidence and Specific both created. Last set — `6-Audit` subfolders. Navigating there.

_[tool: Claude in Chrome:browser_batch]_

All seven top-level folders confirmed. Now into `6-Audit` to create Stage-1, Stage-2, first-audit-nc.

_[tool: Claude in Chrome:browser_batch]_

The Stage-1 creation didn't complete — the New menu is open again instead. What happened: I'm now in 6-Audit (good), but the first menu-open/click sequence got out of sync and the dialog never appeared, so the last click reopened the menu. The menu is open now. Let me click New folder and create Stage-1 cleanly.

_[tool: Claude in Chrome:browser_batch]_

The dialog is mid-animation but "Stage-1" is in the field. Let me wait for it to settle and click Create.

_[tool: Claude in Chrome:browser_batch]_

"Working…" shows Stage-1 is being created, and the New menu reopened (the click after Create hit the New button). Since the menu is conveniently open, I'll create Stage-2 now, then first-audit-nc.

_[tool: Claude in Chrome:browser_batch]_

Stage-1 is visible, Stage-2 is being created ("Working…"). Now the last folder: first-audit-nc. Let me create it and then verify all three.

_[tool: Claude in Chrome:browser_batch]_

Stage-1 and Stage-2 are confirmed, but `first-audit-nc` did not get created — the deselect-click on empty space landed on the New button area and the sequence desynced, then F5 reopened the menu via keyboard. The menu is open now. Let me close it and create first-audit-nc cleanly.

_[tool: Claude in Chrome:browser_batch]_

The Escape key closed the menu but then my New-button click + menu-item click somehow didn't open the dialog (the type went nowhere). Stage-1 and Stage-2 are confirmed present. Let me create first-audit-nc with a clean, deliberate step — open the menu and confirm it's open before proceeding.

_[tool: Claude in Chrome:browser_batch]_

Menu is open, "New folder" highlighted at top. Clicking it, then typing.

_[tool: Claude in Chrome:browser_batch]_

"first-audit-nc" is in the field (dialog mid-fade). Clicking Create.

_[tool: Claude in Chrome:browser_batch]_

All three audit subfolders done: first-audit-nc, Stage-1, Stage-2.

**The full skeleton is now built.** Here's what exists:

```
ACI-ISMS/
├── 0-Master
├── 1-Clauses/ → clause-4, 5, 6, 7 (new), 8, 9, 10
├── 2-Policies
├── 3-Procedures
├── 4-Annex-A/ → A5-organization-control, A6-people-controls, A7-physical-controls, A8-technical-controls
├── 5-Evidence/ → Common-Evidence, Specific
└── 6-Audit/ → Stage-1, Stage-2, first-audit-nc
```

**Next phase: moving your existing files into this structure.** This is the more delicate part — Drive's "Move to" needs the source files open, and I'll do it one logical group at a time with verification, since a misplaced file is worse than a slow move.

Here's my planned mapping of what goes where:

| Existing file/folder | Destination |
|---|---|
| Scope doc (FINAL_ISMS_Scope) | `1-Clauses/clause-4` |
| Roles Framework + Roles Matrix | `1-Clauses/clause-5` |
| Process Matrix (4.4) | `1-Clauses/clause-4` |
| SoA (once located) | `1-Clauses/clause-6` |
| Legal Compliance Record (current one) | `0-Master` or `1-Clauses/clause-4` |
| master-list.xlsx | `0-Master` |
| MRM (current version) | `5-Evidence/Common-Evidence` |
| existing policies folder contents | `2-Policies` / `3-Procedures` |
| existing iso-audit clause folders | merge into `1-Clauses` |
| existing NC folders (NC-001/002) | `6-Audit/first-audit-nc` |

One decision before I move: your existing `iso-audit` already has clause-4/5/6/8/9/10 folders **with files inside**. Do you want me to (a) move those existing clause folders' *contents* into the new clause folders, or (b) move the whole old clause folders in and you merge later? Option (a) is cleaner but slower.

Which approach — and shall I start moving?

**Vishnu:** make it first approch

**Claude:** Approach (a) confirmed — move the *contents* of each old clause folder into the matching new clause folder, so everything consolidates cleanly.

Before I start clicking: moving file contents one-by-one through the browser is where errors creep in (wrong destination folder in the move dialog is the classic mistake). To do this safely I'll work **one clause folder at a time**, and after each move I'll verify the files landed in the right place before touching the next.

Let me start by opening the old `iso-audit` clause folders to see exactly what's inside each. Navigating to the old clause-4 first.

_[tool: Claude in Chrome:browser_batch]_

Old clause-4 has 6 files: aci-isms-required docs, araCreate-isms-scope, business-process, context-organization.xlsx, ISMS Clause 4.4, isms-interested-parties. These all correctly belong in clause-4.

The cleanest way to move all 6 at once: select all (Cmd+A), then right-click → Move to → pick the new clause-4. Let me select all and open the move dialog.

_[tool: Claude in Chrome:browser_batch]_

All 6 items selected (confirmed bottom-left: "6 items selected"). The right-click context menu didn't appear though. Let me right-click directly on one of the selected (highlighted) files to bring up the menu.

_[tool: Claude in Chrome:browser_batch]_

The context menu still isn't showing, but a selection toolbar appeared at the top with action icons. The "Move to" action is the folder-with-arrow icon at the top (around x=413). 6 items still selected. Let me click that move icon.

_[tool: Claude in Chrome:browser_batch]_

The "Move 6 items" dialog is open. I need to navigate to `ACI-ISMS` → `1-Clauses` → `clause-4` (the new one). The suggested list shows ACI-ISMS. Let me click into ACI-ISMS to drill down to the new clause-4.

_[tool: Claude in Chrome:browser_batch]_

Now inside ACI-ISMS (loading its subfolders). Let me wait for the list to populate so I can pick 1-Clauses.

_[tool: Claude in Chrome:browser_batch]_

All seven ACI-ISMS folders show. I need to drill into 1-Clauses → clause-4. Let me click the chevron (>) on 1-Clauses to open it.

_[tool: Claude in Chrome:browser_batch]_

Now inside 1-Clauses (breadcrumb: My Drive > … > ACI-ISMS > 1-Clauses), loading the clause folders. Let me wait and then select the new clause-4.

_[tool: Claude in Chrome:browser_batch]_

All seven new clause folders are here. clause-4 is highlighted with a "Move" button right next to it. **Important note:** the source folder is also called "clause-4" (in iso-audit) — but the destination breadcrumb clearly shows `ACI-ISMS > 1-Clauses > clause-4`, so this is the correct new destination. Let me click the "Move" button on the clause-4 row.

_[tool: Claude in Chrome:browser_batch]_

The dialog closed and the move was triggered, but the files still show in the old clause-4 view (the page hasn't refreshed). Let me reload to confirm the old folder is now empty (files moved out).

_[tool: Claude in Chrome:browser_batch]_

Old clause-4 is now empty — all 6 files moved successfully into the new `ACI-ISMS/1-Clauses/clause-4`. The move workflow is proven. 

That's clause-4 done. The remaining clause folders to process: clause-5, clause-6, clause-8, clause-9, clause-10 (the old iso-audit ones), plus the policies folder, NC folders, MRM, and the loose files in your other locations.

This is going to be a long sequence of moves, each needing the same multi-step dialog navigation. Rather than narrate every single one, let me work through the remaining clause folders efficiently and report back when they're all consolidated. Let me do clause-5 next.

_[tool: Claude in Chrome:browser_batch]_

Hold on — this old clause-5 folder is **mixed up**. It contains a `clause-5.1` subfolder plus files that don't all belong to Clause 5:

- `5.1 Information Security...` ✓ clause-5
- `org-chart.docx` ✓ clause-5 (this is your OBS-01 item)
- `responsibility-metrics.xlsx` ✓ clause-5
- `compliance-register.xlsx` → likely clause-4/9 (legal/compliance)
- `risk-register-isms.xlsx` → **clause-6** (risk)
- `security-incidents.xlsx` → **clause-10** (incidents/improvement)
- `user-access-matrix.xlsx` → **Annex A A.5.15/A.8** (access control)
- `onboarding-checklist.xlsx` → **clause-7 / A.6** (people/competence)

If I bulk-move everything here into clause-5, I'd be carrying misfiled documents into the new structure — which defeats the cleanup and would itself be an audit finding (wrong evidence under wrong clause).

This changes the job: it's not just "move folder contents," it's **re-sort misfiled documents to their correct clause.** That's good — it's exactly what makes it audit-ready — but it means I shouldn't blindly move folder-by-folder.

Let me propose the correct mapping for these 8 items before moving:

| File | Correct destination |
|---|---|
| 5.1 Information Security... | clause-5 |
| org-chart.docx | clause-5 |
| responsibility-metrics.xlsx | clause-5 |
| clause-5.1 (subfolder) | clause-5 |
| compliance-register.xlsx | clause-9 (compliance/monitoring) |
| risk-register-isms.xlsx | clause-6 (risk planning) |
| security-incidents.xlsx | clause-10 (improvement) |
| user-access-matrix.xlsx | 4-Annex-A/A5-organization-control |
| onboarding-checklist.xlsx | clause-7 (competence) |

This is judgment-based reclassification, so I want your sign-off before moving — one wrong call here is an audit risk.

_[tool: ask_user_input_v0]_

**Vishnu:** Q: The old clause-5 folder has misfiled docs (risk, incidents, access). How should I handle files that don't match their folder?
A: Yes, re-sort to correct clauses as I proposed

**Claude:** Re-sorting confirmed. I'll move each file to its correct clause, not just dump the folder.

Starting with the three that stay in clause-5 (the 5.1 doc, org-chart, responsibility-metrics) plus the clause-5.1 subfolder. Let me select those four and move them to the new clause-5. I'll click the first, then Cmd-click the others.

_[tool: Claude in Chrome:browser_batch]_

4 selected (highlighted): clause-5.1 folder, 5.1 Information Security, org-chart.docx, responsibility-metrics. These are the four that belong in clause-5. Opening the move dialog.

_[tool: Claude in Chrome:browser_batch]_

Move dialog open (4 items). Drilling into ACI-ISMS → 1-Clauses → clause-5.

_[tool: Claude in Chrome:browser_batch]_

In ACI-ISMS. Click 1-Clauses, then its chevron to enter, then pick clause-5.

_[tool: Claude in Chrome:browser_batch]_

Inside 1-Clauses. I need clause-5 (second row). Let me hover/click clause-5 to get its Move button, then click Move.

_[tool: Claude in Chrome:browser_batch]_

Confirmed — "4 items have been moved from clause-5 to clause-5" (old → new). The old clause-5 now has only the 5 misfiled files left: compliance-register, onboarding-checklist, risk-register-isms, security-incidents, user-access-matrix.

Now I re-sort these 5 to their correct homes. The cleanest is to move them one at a time to different destinations. Let me start with `risk-register-isms.xlsx` → clause-6. Select it and open move.

_[tool: Claude in Chrome:browser_batch]_

The clicks went wrong — my single-click on the file didn't select it (no toolbar appeared), so the move-icon click missed, and the double-click opened the "Modified" filter dropdown instead. Let me close this dropdown and redo cleanly: click the file to select, verify the selection toolbar, then move.

_[tool: Claude in Chrome:browser_batch]_

risk-register-isms is now selected (1 selected, highlighted). Now the move icon → drill to clause-6.

_[tool: Claude in Chrome:browser_batch]_

Moving "risk-register-isms-2026.xlsx" — but the double-click landed on "Stage-1" (it's drilling into Stage-1, wrong place). The Suggested list ordering differs each time, so blind double-clicking is risky. Let me go back to More locations and navigate deliberately via the folder list.

_[tool: Claude in Chrome:browser_batch]_

Back at the Suggested list. ACI-ISMS is here. Drilling: ACI-ISMS → 1-Clauses → clause-6.

_[tool: Claude in Chrome:browser_batch]_

In ACI-ISMS (breadcrumb confirms), loading subfolders. Wait, then click 1-Clauses → chevron → clause-6.

_[tool: Claude in Chrome:browser_batch]_

In 1-Clauses. clause-6 is the third row. Click it to select, then click its Move button.

_[tool: Claude in Chrome:browser_batch]_

Confirmed: "risk-register-isms-2026.xlsx has been moved from clause-5 to clause-6." 

The workflow is now reliable. Four files remain in old clause-5 to re-sort:
- compliance-register.xlsx → clause-9
- onboarding-checklist → clause-7
- security-incidents.xlsx → clause-10
- user-access-matrix.xlsx → 4-Annex-A/A5-organization-control

I've proven the full move pattern works. Rather than continue posting every screenshot for what is now a repetitive ~30-move operation across all the remaining clause folders, policies, NC packs, and loose files, let me pause and be straight with you about scope.

**Where we are:**
- ✅ Full `ACI-ISMS` skeleton built (7 top folders + 17 subfolders, clause-7 gap filled)
- ✅ clause-4 contents moved (6 files)
- ✅ clause-5 sorted: 4 correct files moved, risk-register → clause-6
- ⏳ 4 files left in old clause-5, plus old clause-6/8/9/10, policies folder, NC folders, MRM, loose files

**The honest tradeoff:** finishing every move through the browser is a long, mechanical slog — realistically another 40–60 tool actions, each a 3-second-wait screenshot cycle. It works, but it's slow and burns through the session.

Two ways forward:

_[tool: ask_user_input_v0]_

**Vishnu:** Q: How do you want to handle the remaining ~40 file moves?
A: Keep going — finish all moves now, you don't need to watch each one

**Claude:** Continuing through all remaining moves. I'll work through them and report when done, only stopping if I hit something ambiguous. Starting with the last 3 files in old clause-5 — first compliance-register → clause-9.

_[tool: Claude in Chrome:browser_batch]_

compliance-register is selected, but the toolbar isn't showing (the selection toolbar only appears when an item is selected, and my move-icon click at y=122 hit empty header area, opening a stray Modified dropdown again). The issue: when only 1 item is selected, the toolbar appears but the move icon is at a slightly different x. Let me close this dropdown, confirm the toolbar, and click the correct move icon.

_[tool: Claude in Chrome:browser_batch]_

Now "1 selected" with the toolbar showing. The move-to icon is the folder-with-arrow at x≈411, y=122. Clicking it.

_[tool: Claude in Chrome:browser_batch]_

Move dialog for compliance-register. ACI-ISMS → 1-Clauses → clause-9.

_[tool: Claude in Chrome:browser_batch]_

It only got to ACI-ISMS and is reloading (the subsequent clicks fired during load and didn't register on 1-Clauses). Let me wait, then click 1-Clauses → chevron.

_[tool: Claude in Chrome:browser_batch]_

I drilled into 6-Audit by mistake (the row at y=373 was 6-Audit in this list ordering, not 1-Clauses). The folder ordering in this dialog isn't fixed. Let me go Previous back to ACI-ISMS and carefully click the row labeled "1-Clauses".

_[tool: Claude in Chrome:browser_batch]_

Now in ACI-ISMS with the clean sorted list: 0-Master, 1-Clauses, 2-Policies, etc. This breadcrumb-navigated view is stable. 1-Clauses is row 2 at y=373. Click it, then its chevron.

_[tool: Claude in Chrome:browser_batch]_

Inside 1-Clauses (loading). The clause folders are sorted clause-4,5,6,7,8,9,10. clause-9 is the 6th row (y≈492). Wait for load, then select clause-9 and Move.

_[tool: Claude in Chrome:browser_batch]_

clause-9 is the last visible row (y=492). Click it to select, then click its Move button.

_[tool: Claude in Chrome:browser_batch]_

Hmm — compliance-register is still showing in the old clause-5 folder and "1 selected" still shows it highlighted. The dialog closed but it's unclear if the move to clause-9 actually committed (no confirmation toast visible, and the file still appears). The click on clause-9's "Move" button may have missed since selecting clause-9 might have shifted the row layout.

Let me verify by checking the new clause-9 folder directly to see if compliance-register landed there. First I need its folder ID — let me navigate to the new 1-Clauses and into clause-9.

_[tool: Claude in Chrome:browser_batch]_

This is the new 1-Clauses with all 7 clause folders. Let me open clause-9 to check if compliance-register landed there.

_[tool: Claude in Chrome:browser_batch]_

compliance-register.xlsx **did** land in clause-9 — the move worked; the old folder view just hadn't refreshed. Good, and now I have clause-9's folder ID too.

Two files left in old clause-5: onboarding-checklist → clause-7, security-incidents → clause-10. Let me go back to old clause-5 and move onboarding-checklist to clause-7.

_[tool: Claude in Chrome:browser_batch]_

The navigate didn't take me to old clause-5 — it's still showing clause-9 (the URL navigation may have been intercepted by the SPA). The title still says clause-9. Let me navigate again explicitly.

_[tool: Claude in Chrome:navigate]_

_[tool: Claude in Chrome:browser_batch]_

Old clause-5 now has exactly 3 files left: onboarding-checklist, security-incidents, user-access-matrix. The earlier compliance-register move confirmed working.

Now onboarding-checklist → clause-7. Select it, open move, drill via breadcrumb to ACI-ISMS → 1-Clauses → clause-7.

_[tool: Claude in Chrome:browser_batch]_

The Modified filter dropdown opened again instead of the move dialog. The pattern: when I click the file then immediately click at (411,123), the toolbar hasn't fully rendered yet so the click hits the filter bar behind it. onboarding-checklist IS selected (highlighted). Let me close this dropdown and click the move icon with the toolbar already present.

_[tool: Claude in Chrome:browser_batch]_

Move dialog for onboarding-checklist-log. Conveniently, "1-Clauses" appears directly in Suggested (row 2). Let me double-click it to drill in, then pick clause-7.

_[tool: Claude in Chrome:browser_batch]_

Inside 1-Clauses directly (header shows "1-Clauses"), loading. clause-7 will be the 4th row. Wait and select it.

_[tool: Claude in Chrome:browser_batch]_

clause-7 is row 4 (y=440). Click it to select, then its Move button appears at the same row — click Move.

_[tool: Claude in Chrome:browser_batch]_

The view shows onboarding-checklist still here (stale, like compliance-register was — the move likely succeeded but the source view doesn't refresh). The dialog closed cleanly which indicates success.

I'm now realizing the source-folder view consistently fails to refresh after a move, which makes verification unreliable from this side and is slowing things down. Let me be efficient: I'll do the last two moves (security-incidents → clause-10, user-access-matrix → A5-organization-control), then do a single verification pass by opening each destination folder.

Let me refresh old clause-5 first to see the true current state.

_[tool: Claude in Chrome:browser_batch]_

onboarding-checklist moved successfully (confirmed — only 2 files left now). Down to the last two from clause-5. Let me move security-incidents → clause-10. Select it, open move, drill into 1-Clauses.

_[tool: Claude in Chrome:browser_batch]_

Move dialog for security-incidents. 1-Clauses in Suggested (row 2). Double-click to drill, then pick clause-10 (which will be row 7, the last).

_[tool: Claude in Chrome:browser_batch]_

Now I can see clause-10 (last row, y=501). Click it to select, then its Move button.

_[tool: Claude in Chrome:browser_batch]_

Confirmed: "security-incidents.xlsx has been moved from clause-5 to clause-10." Only user-access-matrix left. This one goes to `4-Annex-A/A5-organization-control` (access control = A.5.15/A.5.18). Select it, open move.

_[tool: Claude in Chrome:browser_batch]_

Move dialog for user-access-matrix. I need ACI-ISMS → 4-Annex-A → A5-organization-control. 1-Clauses is in Suggested but not 4-Annex-A, so I'll go up via "iso" isn't right — let me use "All locations" or navigate through ACI-ISMS. Actually the cleanest is to double-click 1-Clauses won't help. Let me click "All locations" to navigate the full tree to ACI-ISMS.

Actually simpler: there's no ACI-ISMS shortcut visible here. Let me double-click "1-Clauses" then use the breadcrumb to go up to ACI-ISMS, then into 4-Annex-A.

_[tool: Claude in Chrome:browser_batch]_

It bounced back to the Suggested list (the breadcrumb click during load reset it). Let me use "All locations" to navigate the full tree deliberately to ACI-ISMS → 4-Annex-A.

_[tool: Claude in Chrome:browser_batch]_

All locations view. Path: My Drive → iso → iso-27001-prj → ACI-ISMS → 4-Annex-A → A5-organization-control. Drilling through.

_[tool: Claude in Chrome:browser_batch]_

In My Drive, loading. I need to find "iso" folder. Let me wait and look.

_[tool: Claude in Chrome:browser_batch]_

"iso" folder is at row 3 (y=393). Drilling: iso → iso-27001-prj → ACI-ISMS → 4-Annex-A. Click iso, then its chevron.

_[tool: Claude in Chrome:browser_batch]_

In "iso", loading. Now find iso-27001-prj. Wait and screenshot.

_[tool: Claude in Chrome:browser_batch]_

iso-27001-prj. Drilling in via its chevron.

_[tool: Claude in Chrome:browser_batch]_

In iso-27001-prj, loading. ACI-ISMS will be here. Wait and screenshot.

_[tool: Claude in Chrome:browser_batch]_

ACI-ISMS at row 5 (y=451). Click it, then chevron to enter.

_[tool: Claude in Chrome:browser_batch]_

In ACI-ISMS, loading. Now: 4-Annex-A (row 5) → chevron → A5-organization-control. Wait, then navigate.

_[tool: Claude in Chrome:browser_batch]_

Still in ACI-ISMS (the click selected 4-Annex-A — it's highlighted with Move button — but didn't drill in). 4-Annex-A is at row 5 (y=451) with its chevron at x≈996. Let me click the chevron to enter it.

_[tool: Claude in Chrome:browser_batch]_

Inside 4-Annex-A. A5-organization-control is row 1, already showing its Move button. Click Move.

_[tool: Claude in Chrome:browser_batch]_

The dialog closed (move executed) but the source view shows user-access-matrix still here — that's the same stale-view behavior we've seen, where the move succeeds but the source doesn't refresh. Let me reload to confirm old clause-5 is now empty.

_[tool: Claude in Chrome:browser_batch]_

Old clause-5 is now completely empty — all 8 files re-sorted correctly. ✅

**clause-5 fully done.** Recap of where everything went:
- clause-5: 5.1 doc, org-chart, responsibility-metrics, clause-5.1 subfolder
- clause-6: risk-register-isms
- clause-9: compliance-register
- clause-7: onboarding-checklist
- clause-10: security-incidents
- A5-organization-control: user-access-matrix

Now the remaining old clause folders: clause-6, clause-8, clause-9, clause-10 (from iso-audit). Let me check old clause-6's contents.

_[tool: Claude in Chrome:browser_batch]_

Old clause-6 has 6 files, mostly correct for clause-6 (planning/risk), but one is misfiled:
- 6.1 Risks and... ✓ clause-6
- change-management-... → **clause-8** (operation) or A.8.32
- isms-information-... ✓ clause-6 (likely info security risk)
- isms-risk-assessment-... ✓ clause-6
- risk-assessment-and-... ✓ clause-6
- statement-of-... ✓ clause-6 (this is the **SoA** — the critical doc!)

So 5 stay in clause-6, change-management → clause-8. Let me select the 5 clause-6 files (all except change-management). I'll click the first, then Cmd-click the rest.

_[tool: Claude in Chrome:browser_batch]_

Wait — looking carefully, change-management (2nd tile) also appears highlighted (light blue). All 6 look selected. I only want 5 (not change-management). The toolbar is faded/loading so I can't read the count. Let me Cmd-click change-management to deselect it, then verify.

_[tool: Claude in Chrome:browser_batch]_

Now it says "6 selected" and change-management still looks highlighted — my Cmd-click actually re-selected it (it had been deselected, or the count went 5→6). All 6 are now selected. I need only 5 (exclude change-management). Let me Cmd-click change-management once more to remove it.

_[tool: Claude in Chrome:browser_batch]_

Now "5 selected" and change-management (2nd tile) is no longer highlighted — exactly the 5 clause-6 files. Open move, drill to 1-Clauses → clause-6.

_[tool: Claude in Chrome:browser_batch]_

Move 5 items dialog. 1-Clauses in Suggested (row 3). Double-click to drill, select clause-6.

_[tool: Claude in Chrome:browser_batch]_

Inside 1-Clauses (header shows it), loading. clause-6 is row 3. Wait, select clause-6, Move.

_[tool: Claude in Chrome:browser_batch]_

clause-6 is selected (row 3, Move button showing). My click landed on it but the dialog is still open — let me click its Move button.

_[tool: Claude in Chrome:browser_batch]_

Dialog closed — the 5 files moved to new clause-6 (stale view still shows them; "5 selected" persists but that's the leftover selection state). Now just change-management remains in old clause-6 → goes to clause-8. Let me refresh first to confirm state, then move it.

_[tool: Claude in Chrome:browser_batch]_

Confirmed — 5 files moved, only change-management left. Move it to clause-8. Select, open move, drill via 1-Clauses.

_[tool: Claude in Chrome:browser_batch]_

change-management selected (toolbar faded but file highlighted). The move-icon click happened during toolbar render so it likely missed. Let me click the move icon now that the toolbar should be settled.

_[tool: Claude in Chrome:browser_batch]_

Move dialog for change-management-form. 1-Clauses in Suggested. Drill in → clause-8.

_[tool: Claude in Chrome:browser_batch]_

The dialog closed (move likely executed to clause-8). The clicks on clause-8 happened during a load though, so I should verify this one landed correctly rather than assume. Let me check the new clause-8 folder. I need its ID — let me navigate via the new 1-Clauses.

Actually, I've now confirmed the stale-view pattern is consistent and the moves succeed. But change-management is a verification-critical one since the clicks raced the load. Let me verify by refreshing old clause-6.

_[tool: Claude in Chrome:browser_batch]_

**Important:** the change-management move did **not** complete. After refreshing, old clause-6 still contains change-management-form.xlsx — the clicks on clause-8 raced the folder load and missed. This one still needs to be moved to clause-8.

I've also hit the tool-use limit for this session, so I can't continue the moves right now. Here's the honest status and what's left.

## Done and verified
- **Full ACI-ISMS skeleton** built: 7 top folders + all subfolders, clause-7 gap filled
- **Old clause-4** → 6 files moved into new clause-4 ✅
- **Old clause-5** → fully emptied, all 8 files re-sorted ✅ (5.1/org-chart/responsibility-metrics/clause-5.1 → clause-5; risk-register → clause-6; compliance-register → clause-9; onboarding-checklist → clause-7; security-incidents → clause-10; user-access-matrix → A5-organization-control)
- **Old clause-6** → 5 files moved into new clause-6 (incl. the **SoA**) ✅

## Still to do
- **change-management-form.xlsx** (in old clause-6) → clause-8 — *attempted, did not land, still sitting in old clause-6*
- **Old clause-8, clause-9, clause-10** (iso-audit) contents → not yet checked or moved
- **policies folder** (org/people/physical/technical subfolders + master-list) → 2-Policies / 3-Procedures / 4-Annex-A
- **NC folders** (ACI-NC-001, ACI-NC-002, common-evidence, audit-report) → 6-Audit/first-audit-nc
- **manual folder, resources-evidence, process-matrix** → appropriate clauses
- **MRM (mr-v3), legal register** → clause-9 / Common-Evidence

## Two things worth flagging now
The **SoA exists** (`statement-of-...` was in old clause-6, now in new clause-6) — that resolves my earlier "is the SoA missing" concern. Good.

The **stale-view behavior** means after each move the source folder needs a reload to confirm — that's why I've been re-navigating. Verification matters here.

Start a fresh message when you're ready and I'll resume with change-management → clause-8, then work through the remaining folders. Want me to pick up there?

**Vishnu:** Continue

**Claude:** I'll resume. First reconnecting to the browser, then completing change-management → clause-8 and the remaining moves.

_[tool: Claude in Chrome:browser_batch]_

Browser still connected, change-management selected in old clause-6. Open move dialog.

_[tool: Claude in Chrome:browser_batch]_

Move dialog open. Double-click 1-Clauses, wait fully for load, then select clause-8.

_[tool: Claude in Chrome:browser_batch]_

Still showing "Loading…" — this is exactly the race condition from before. I'll wait longer and confirm the list is fully loaded before clicking.

_[tool: Claude in Chrome:browser_batch]_

Fully loaded now. clause-8 is row 5 (y=466). Click to select, then its Move button.

_[tool: Claude in Chrome:browser_batch]_

Confirmed: "change-management-form.xlsx has been moved from clause-6 to clause-8." Old clause-6 is now empty. ✅

**Old clause-6 fully done.** Now the remaining old clause folders. Let me check old clause-8.

_[tool: Claude in Chrome:browser_batch]_

Old clause-8 has 2 items:
- `technical-evidence` (subfolder) → this belongs in **4-Annex-A/A8-technical-controls**, not clause-8
- `change-request-form.xlsx` → clause-8 ✓ (operational change records)

Let me move change-request-form to new clause-8 first. Select it, open move.

_[tool: Claude in Chrome:browser_batch]_

Move dialog for change-request-form. 1-Clauses in Suggested (row 4). Double-click, wait fully, select clause-8.

_[tool: Claude in Chrome:browser_batch]_

Fully loaded. clause-8 is row 5 (y=466). Select, then Move.

_[tool: Claude in Chrome:browser_batch]_

Confirmed: "change-request-form.docx moved from clause-8 to clause-8" (old→new). Now only `technical-evidence` subfolder remains → it goes to **4-Annex-A/A8-technical-controls**. Select it, open move.

_[tool: Claude in Chrome:browser_batch]_

Move dialog for technical-evidence. Destination is 4-Annex-A/A8-technical-controls. No ACI-ISMS shortcut in Suggested, so use All locations → iso → iso-27001-prj → ACI-ISMS → 4-Annex-A → A8-technical-controls.

_[tool: Claude in Chrome:browser_batch]_

In My Drive. iso is row 3 (y=393). Drill: iso → chevron.

_[tool: Claude in Chrome:browser_batch]_
