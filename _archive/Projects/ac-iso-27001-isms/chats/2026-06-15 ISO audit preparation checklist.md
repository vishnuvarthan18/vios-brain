---
tags: chat
date: 2026-06-15
source: Claude personal account
uuid: 91b163df-0c6a-4c00-8bb9-1abb8ee4d976
---
# ISO audit preparation checklist

## Summary
**Conversation overview**

The person is working on ISO/IEC 27001:2022 certification for a company called ACI and is preparing for their Stage 2 audit. They are working with an external ISO vendor and referenced specific open findings from their Stage 1 audit: GR/01 (related to the legal/regulatory register and context documentation), GR/02 (related to Statement of Applicability justification for control A.8.30), and OBS-01 (Board of Directors missing from the org chart). A colleague named Shiva is identified as the independent internal auditor. The MD named Navaneethan is responsible for signing approvals. The auditor conducting Stage 2 is referred to as James.

The person asked Claude to first analyze meeting notes from their ISO vendor and produce a comprehensive Stage 2 readiness checklist, then to convert that checklist into a more effective end-to-end format, and finally to present it as a simple tickable checklist they could work through directly. Claude produced an Excel workbook with four sheets (How to Use, Master Tracker, Annex A 2022 Controls, and Pre-Submission Sweep) and then rendered the full checklist inline in tickable format organized by clause (4 through 10), a dedicated section for the 11 new ISO 27001:2022 Annex A controls plus 8.2 and 8.3, and a final pre-submission sweep section.

Key audit priorities flagged throughout include: the internal audit must be completed and dated before NC closure to demonstrate the ISMS operates in practice; backup evidence must include restore-test records not just backup logs; the 11 new 2022 transition controls (5.7, 5.23, 5.30, 7.4, 8.9, 8.10, 8.11, 8.12, 8.16, 8.23, 8.28) require fresh evidence as auditors focus on these during transition audits; MRM minutes and training attendance records must reference all relevant NCs not just one; and a hard copy of the NC correction report must be prepared. The person communicates in a brief, informal style with frequent typos and prefers direct, structured, actionable output rather than explanatory prose.

## Chat

**Vishnu:** I will uplode some meeting notes that are the things my iso vendor sad and you also gave a depp analys nad tell me the whole things need to chckerlsit before second audit

**Claude:** These are your handwritten Stage 2 prep notes from the ISO vendor. Here's the consolidated checklist, organized by clause, with my read on what each item means for ACI's Stage 2 readiness.

## Stage 2 scope (what the auditor will do)
- Stage 2 is the full controls-and-evidence audit. Auditor will pull roughly **20 random controls** from your 93 Annex A controls and demand evidence for each. You cannot predict which 20, so every control with a "applicable" status in your SoA needs a ready evidence artifact.
- Internal audit must be completed **before** NC closing — the vendor flagged that you need a documented internal NC raised and closed cycle to demonstrate the ISMS actually runs, not just exists on paper.

## Clause-by-clause checklist

**Clause 4 (Context)**
- Add documentation for internal/external issues — this connects to your open GR/01 finding. The register needs to be a controlled document, not a draft.
- Interested parties list must be documented and current.

**Clause 5 (Leadership)**
- Roles, Responsibilities & Authority (RRA) document — update it.
- Objectives finding — leadership-set objectives must trace to evidence. (Note: this is also where OBS-01, Board of Directors on the org chart, lives — close it here while you're in this clause.)

**Clause 6 (Planning)**
- Risk identification, risk treatment, and the risk assessment document — all three must be consistent with each other and with the SoA. This is your GR/02 area (Control A.8.30 justification). Note these down for final discussion with the auditor.

**Clause 7 (Support)**
- New-employee onboarding records.
- Training plan with plan → feedback → attendance evidence. Show last cycle vs. now to demonstrate continual improvement.
- Evidence that all staff have security-awareness knowledge (completion records).

**Clause 8 (Operation)** — *highest-risk area, most items*
- Master list of procedures and documents (Clauses 4–10 process docs).
- Access control, password control evidence.
- Backup managed and tested — need test restore evidence, not just backup logs.
- Remote work policy + evidence.
- Supplier management records.
- Incident management records.
- The **11 new 2022 controls** specifically called out — have fresh evidence for each: `5.7, 5.23, 5.30, 7.4, 8.9, 8.10, 8.11, 8.12, 8.16, 8.23, 8.28`.
- Vulnerability + risk assessment (`8.2, 8.3`) — add recent evidence (recent scan/assessment dated close to audit).

**Clause 9 (Performance evaluation)**
- Objectives update with monitoring evidence (9.1 metrics).
- Internal audit (9.2) — full Stage 1 + Stage 2 internal audit cycle documented.
- Internal auditor must hold a certificate / training certification (QHSE). Auditor will check competence.
- MRM (Management Review Meeting) — must cover all 10 standard inputs (training, objectives, audit results, customer feedback, NC status). Your notes flag "10 questions" — confirm all 10 MRM inputs are addressed and the MRM references all relevant NCs, not just one.

**Clause 10 (Improvement)**
- NC correction report and guidance — vendor wants a **hard copy** ready.
- Internal audit conducted by someone independent (notes name "Shiva" as the external/independent internal auditor — confirm independence from the area audited).

## Cross-cutting items to sweep before submission
- **Common-Evidence folder**: MRM, training attendance, and internal audit records go here once, referenced by each NC — not duplicated into NC subfolders.
- **MRM and training docs must reference all relevant NCs**, not just one (your recurring error to watch).
- **Sweep all date placeholders** (`[Date]`) and stale dates across every document.
- **Version control** (revision history + version number) on every ISMS record, including the legal register.

## My recommendation
Two items are genuinely audit-blocking if missed: the **internal audit cycle completed before NC closure** (Clause 9.2) and **evidence for the 11 new 2022 controls** (these are the most common Stage 2 failure points for 2022-transition audits). Prioritize those. Everything else is closeable with the document sweep you already have a process for.

Want this as a tracked checklist file (xlsx) with status columns per item, so you can mark off evidence as you stage it in Drive?

**Vishnu:** ok lets make this more effect and poepr check list to maintain full end to end proepr audit ready 

go to ened to end what are the docuemts and each and single thisg need to chcek before the audit

**Claude:** I'll build a complete end-to-end audit-readiness checklist as a tracked spreadsheet — every document and verification item mapped to its clause/control, with status columns you can work through. Let me set it up properly.

_[tool: view]_

Now I'll build the checklist. This will be a multi-sheet workbook: a master tracker organized by clause, a dedicated Annex A 2022-controls evidence sheet, and a pre-submission sweep sheet.

_[tool: create_file]_

_[tool: bash_tool]_

_[tool: present_files]_

Done. Four sheets, all dropdown-tracked.

**Master Tracker** — 35 items across Clauses 4–10, each with the exact document/evidence required, what the auditor checks, priority, and status/owner/Drive-location columns to fill.

**Annex A 2022 Controls** — the 11 new transition controls (`5.7, 5.23, 5.30, 7.4, 8.9, 8.10, 8.11, 8.12, 8.16, 8.23, 8.28`) plus `8.2`/`8.3`, each with specific evidence required.

**Pre-Submission Sweep** — 12 final hygiene checks (date placeholders, version control, NC cross-referencing, hard copy, SoA coverage).

**How to use** — priority key and audit context up front.

**The 10 Blocking items** (do first — these stop Stage 2 progression): Context/legal register (GR/01), risk identification, risk assessment, risk treatment plan, SoA with A.8.30 justification (GR/02), internal auditor competence, backup restore-test evidence, the 11 new 2022 controls, internal audit completed before NC closure, MRM covering all 10 inputs, and Stage 1 NC closure hard copy.

Two items I'd treat as the highest failure risk based on your notes: **backup needs restore-test evidence** (not just backup logs — auditors reject logs alone), and **the internal audit must be dated before NC closure** with an internal NC raised-and-closed to prove the ISMS actually operates.

Want me to drop this into your Drive evidence folder, or pre-fill the Owner/Drive-location columns from what you already have staged?

**Vishnu:** give me the chceklist wise so i can do chcek and give tick

**Claude:** Here's the checklist in tickable format. Work top to bottom; Blocking items first.

## CLAUSE 4 — Context
- ☐ Internal & external issues register — controlled doc *(Blocking, GR/01)*
- ☐ Interested parties register with needs/requirements
- ☐ ISMS scope statement (boundaries, interfaces, exclusions)
- ☐ Legal & regulatory register — with version control *(Blocking, GR/01)*

## CLAUSE 5 — Leadership
- ☐ Information Security Policy — signed by MD, dated, versioned
- ☐ Roles, Responsibilities & Authority (RRA) matrix — updated
- ☐ Org chart including Board of Directors *(OBS-01)*
- ☐ Leadership commitment evidence (MRM minutes, resourcing)

## CLAUSE 6 — Planning
- ☐ Risk identification — risk register (assets, threats, vulnerabilities) *(Blocking)*
- ☐ Risk assessment — completed, with scoring method *(Blocking)*
- ☐ Risk treatment plan — risks mapped to controls + owner + date *(Blocking)*
- ☐ Statement of Applicability — all 93 controls, A.8.30 justified *(Blocking, GR/02)*
- ☐ Information security objectives — measurable, owned, monitored

## CLAUSE 7 — Support
- ☐ Training plan — plan → delivery → feedback → attendance; references all NCs
- ☐ New employee onboarding records (incl. security awareness)
- ☐ Security awareness — 100% staff completion records
- ☐ Internal auditor certificate (QHSE/equivalent) *(Blocking)*
- ☐ Master list of documents & procedures (Clauses 4–10)
- ☐ Communication matrix (who/what/when/how)

## CLAUSE 8 — Operation
- ☐ Operational process docs / procedures
- ☐ Access control policy + user access records (joiner/mover/leaver)
- ☐ Password / authentication control — policy + enforcement evidence
- ☐ Backup — policy + **restore-test evidence** (not just backup logs) *(Blocking)*
- ☐ Remote / teleworking policy + evidence
- ☐ Supplier register + security agreements
- ☐ Incident management procedure + incident log
- ☐ Vulnerability scan / technical assessment — recent, dated near audit
- ☐ **11 new 2022 controls** — see dedicated list below *(Blocking)*

## CLAUSE 9 — Performance
- ☐ Monitoring & measurement (9.1) — objectives tracked with data
- ☐ Internal audit programme — plan + schedule *(Blocking)*
- ☐ Internal audit completed **before** NC closure — report + internal NC raised & closed *(Blocking)*
- ☐ Internal audit independence evidence (Shiva — not auditing own work)
- ☐ MRM minutes — all 10 standard inputs, references **all** relevant NCs *(Blocking)*

## CLAUSE 10 — Improvement
- ☐ NC log + corrective action records (root cause, correction, effectiveness)
- ☐ Stage 1 NC closure (GR/01, GR/02) — **hard copy ready** *(Blocking)*
- ☐ Continual improvement evidence

## ANNEX A — 11 New 2022 Controls + 8.2/8.3
- ☐ 5.7 Threat intelligence — source + how it feeds risk assessment
- ☐ 5.23 Cloud services security — policy + provider assessment
- ☐ 5.30 ICT readiness for BC — BC/DR plan + test evidence
- ☐ 7.4 Physical security monitoring — CCTV/logs/alarms
- ☐ 8.9 Configuration management — baselines + change records
- ☐ 8.10 Information deletion — deletion/disposal evidence
- ☐ 8.11 Data masking — masking/anonymisation (where applicable)
- ☐ 8.12 Data leakage prevention — DLP measures
- ☐ 8.16 Monitoring activities — log monitoring + anomaly detection
- ☐ 8.23 Web filtering — policy + technical enforcement
- ☐ 8.28 Secure coding — standards + review (if dev work)
- ☐ 8.2 Privileged access rights — list + recent review
- ☐ 8.3 Information access restriction — need-to-know evidence

## PRE-SUBMISSION SWEEP (final pass)
- ☐ All `[Date]` placeholders removed
- ☐ Stale/old dates replaced with current
- ☐ Version control on every record (incl. legal register)
- ☐ MRM references all relevant NCs, not just one
- ☐ Training attendance references all relevant NCs
- ☐ Shared evidence in Common-Evidence folder, not duplicated
- ☐ All approvals signed by MD
- ☐ SoA justifications complete for all 93 controls
- ☐ Hard copy of NC correction report prepared
- ☐ Internal audit dated before NC closure
- ☐ Drive folders clean (NC-001, NC-002, Common-Evidence, OBS-01)
- ☐ Every "Applicable" SoA control has evidence ready (20 will be pulled at random)

Replace ☐ with ☑ as you go. Want this as a printable PDF you can tick by hand during the walkthrough?
