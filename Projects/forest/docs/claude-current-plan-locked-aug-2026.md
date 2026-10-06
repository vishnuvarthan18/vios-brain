# CURRENT PLAN — LOCKED, 12 August 2026

**This is the live document. Everything else in this project is history.**

**For:** Vishnuvarthan Venkatapathy, Erode
**Supersedes:** `six-track-master-plan.md`, `no-government-track-plan.md`, `government-scientist-master-plan.md`, `scientist-degree-path.md`, `jump-now-90-day-plan.md`, `scientist-money-plan.md`, `forest-life-options-beyond-scientist.md`

---

## Three decisions, locked

**1. No permanent government job.** Not about pay — about the closed circle. Forester (TNFUSRC), ACF (TNPSC Group IA) and Scientist-B (ICFRE/CPCB/TNPCB) are permanently out.

Contract and project work with government-funded institutes **stays in**: WII, IFGTB, FSI, NRSC, SACON, DGRE. These are 1–3 year engagements you can walk away from. Different thing entirely.

**2. Degree: IGNOU M.Sc. Geoinformatics (MSCGI).** Distance mode, alongside the job. M.Sc. Environmental Science is dropped — it only ever existed to satisfy ICFRE's eligibility rule, which no longer applies. Study mode is **part-time, always**; full-time residential (WII Dehradun, NCBS Bangalore) is ruled out.

**3. Intake: January 2027, not July 2026.** Decided 12 Aug 2026. The August deadline is deliberately let go.

---

## What January 2027 means

| | |
|---|---|
| Applications open | ~16 December 2026 |
| Last date | 31 January 2027 (extensions are routine, do not rely on them) |
| Apply at | ignouadmission.samarth.edu.in |
| Semester 1 runs | January – June 2027 |
| Assignments due | 31 March 2027 |
| First exam (TEE) | June 2027 |
| Degree completes | Mid 2029 |

**Cost of waiting: four months.** The degree is identical either way — same fee, same syllabus, same certificate. Nobody can tell which cycle you entered from.

### One thing still to verify

MSCGI's January intake is confirmed by inference, not by an official statement. IGNOU's own MSCGI assignment booklets address both a "July cycle" and a "January cycle" cohort, which only makes sense if both intakes exist. But the official programme page states no admission session at all, and IGNOU occasionally suspends a cycle for a specific programme without announcing it.

**Call and confirm.** Regional Centre Madurai (code 43) covers Erode — 0452-2380387 / 2370733, rcmadurai@ignou.ac.in. Chennai RC (044-24312766, rcchennai@ignou.ac.in) will redirect if a Coimbatore centre now exists.

Ask exactly: *"Is MSCGI open for fresh admission in the January session?"*

Do this in September, not December. If the answer is July-only, you need the months to react.

---

## The degree is the smaller half

**The portfolio is the plan. The degree is paperwork that runs in the background.**

Every organisation on the target list hires by interview, not written exam. What they look at is shipped work. IGNOU on a CV gets you past the eligibility filter; a public repo with real imagery, real analysis and a written-up result is what makes someone reply to your email.

**Choosing January makes this more true, not less.** You now have five months with no coursework at all. If December 2026 arrives with an enrolment form and nothing built, the delay bought nothing. If it arrives with a working repo, the delay was the right call.

---

## Aug – Dec 2026: the five months that justify the wait

This period has no classes, no fees and no deadlines except the ones you set. It is the most valuable stretch in the whole plan.

| Month | Target |
|---|---|
| **August** | Repo created, project question written in the README. Google Earth Engine account active. First Sentinel-2 scene loaded and displayed. WILDLABS account made. |
| **September** | Call the regional centre and confirm the January intake. First working change-detection output over one reserve, however rough. One free IIRS (ISRO) course started. |
| **October** | Extend to two or three adjoining reserves. Begin the write-up. Send one pull request to github.com/techforwildlife. |
| **November** | Project public and documented. Written up as a WILDLABS post. First cold emails go out with the link attached. |
| **December** | **Apply to MSCGI from 16 December.** Portfolio already exists by the time the form is submitted. |

**Set a calendar reminder for 15 December 2026.** Four months is long enough to forget.

---

## The first project

The wedge is still unpicked. Each of these is a real, documented, unsolved gap in your own landscape. All start with free data and no hardware.

### Option 1 — Invasive species mapping from satellite  ← recommended

**The gap:** Senna spectabilis and Lantana have spread across the Nilgiri Biosphere Reserve. Tamil Nadu has cleared hundreds of hectares inside Sathyamangalam and Mudumalai under Madras High Court orders and the state PIPER policy. Everyone counts hectares cleared. **Nobody systematically verifies what grew back.**

**Why you can do it:** Senna's bright yellow flowering canopy is a strong spectral and phenological signal. Sentinel-2 imagery is free. Google Earth Engine needs no hardware.

**First build:** A GEE + Sentinel-2 change-detection notebook mapping flowering signals across two or three adjoining reserves, flagging post-clearance regrowth compartment by compartment. Publish as an open QGIS/GeoPandas workflow with a simple web map.

**Cost:** ₹0. **Can start:** today.

### Option 2 — Bioacoustics

**The gap:** BirdNET was trained overwhelmingly on North American and European species and performs measurably worse in Asia. There is no non-commercial multi-species frog classifier for India. Chainsaw and gunshot acoustic detection has no published deployment inside Indian forests despite being proven abroad.

**First build:** A Western Ghats frog-call or Eastern Ghats bird classifier from Xeno-canto data, later supplemented with your own AudioMoth recordings.

**Cost:** ₹0 to start; AudioMoth later (IdeaWild grants cover this). SACON Anaikatty, an hour away, runs active bioacoustics work.

### Option 3 — Drone, thermal and LiDAR processing

**The gap:** In 2025 the TN Forest Department deployed drones with 48 MP cameras and thermal sensors across 13 territorial forest circles. Only three staff per circle were trained — **as pilots**. No stated plan for who turns that imagery into orthomosaics, canopy models or thermal detections.

**First build:** An open pipeline — OpenDroneMap photogrammetry to orthomosaic and DEM, thermal-anomaly detection, LiDAR canopy-height workflow. Demonstrate on public sample data, then offer it to a forest circle.

**Cost:** ₹0 to start; DGCA Remote Pilot Certificate later.

### Option 4 — Animal movement and conflict prediction

**The gap:** The Coimbatore–Sathyamangalam–Valparai corridor is the most instrumented human-elephant conflict landscape in India. Detection is crowded — NCF runs SMS alerts and beacons, the forest department has AI thermal camera command centres. **Prediction is not.**

**Caveat:** needs partnership with NCF or the forest department for data. Starts with an email, not with code.

### Why Option 1

It is the only one where the data, the tools, the landscape and the unsolved question all sit within reach today, with nobody's permission needed. It is also the strongest grant pilot — Ravi Sankaran, Rufford and Mohamed bin Zayed all fund exactly this shape of work.

Options 2 and 3 stay open for 2027 once one thing is shipped. Option 4 needs a relationship first.

**Do not wait for certainty. Ship one thing, then let the response tell you.**

---

## Money

| Item | Official figure |
|---|---|
| MSCGI total programme fee | **₹32,000** for 80 credits (IGNOU official) |
| Paid as | One semester at a time, roughly ₹8,000 × 4 |
| Registration + development | ₹500 at admission |
| Term-end exam form | ~₹200 per course, each session |
| Lab sessions at Learner Support Centre | Travel only |
| **Realistic all-in** | **₹35,000–45,000 over 2 years** |
| **Spend before January 2027** | **₹0** |

Nothing is paid upfront, and nothing at all is due for the next four months.

**The geoinformatics ladder is nested**, if a semester ever needs deferring:

- PG Certificate (PGCGI) — 6 months, 20 credits, ₹8,000
- PG Diploma (PGDGI) — 1 year, 40 credits, ₹16,000 (PGCGI's four courses are its semester 1)
- M.Sc. (MSCGI) — 2 years, 80 credits, ₹32,000

IGNOU allows up to 4 years to finish a 2-year programme, so pace is flexible once enrolled.

**Every project costs ₹0.** GEE, Sentinel-2, QGIS, GeoPandas, OpenDroneMap, Xeno-canto, GitHub, WILDLABS — all free.

---

## The roadmap

| Phase | When | What |
|---|---|---|
| **0 — Build before enrolling** | Aug – Dec 2026 | No coursework. First portfolio project public by November. Confirm January intake in September. Apply from 16 December. |
| **1 — Ship while enrolled** | Jan 2027 – early 2028 | MSCGI semesters 1–2 in the background. Python geospatial stack (GeoPandas, Rasterio, xarray, GEE). Free IIRS certificates. DGCA Remote Pilot Certificate. Second project. |
| **2 — Get inside** | Mid 2027 – 2028 | Apply to Project Associate-I, Technical Assistant and field GIS roles at WII, IFGTB Coimbatore, FSI, NRSC. Cold-outreach PIs with the portfolio link before ads drop. "Pursuing M.Sc. Geoinformatics" explains why a CS graduate is applying; the repo proves you can do it. |
| **3 — Degree in hand** | Mid 2029 | PA-II, Project Scientist, Technical Officer. Alternative fork: MRV / conservation-tech product roles. |
| **4 — Summit** | 2030 | Field-based in forest and mountain systems, running technology for research and conservation. The non-negotiable the whole plan serves. |

---

## Skills, in priority order

| # | Skill | When |
|---|---|---|
| 1 | Python geospatial + Google Earth Engine (GeoPandas, Rasterio, xarray) | **Now — August 2026** |
| 2 | QGIS + PostGIS/SQL | **Now — August 2026** |
| 3 | ML for ecology — MegaDetector (camera traps), BirdNET (bioacoustics) | Q4 2026 |
| 4 | DGCA Remote Pilot Certificate — few days at an approved RPTO, valid 10 years | 2027 |
| 5 | R for ecology — camtrapR, species distribution models | 2027 |
| 6 | Sensors + GNSS — AudioMoth, LoRaWAN, IoT collars | 2027 |

---

## Three tracks the work unlocks

**Government-funded contract / research** — ₹31–37K/mo + HRA entry. PA-I → PA-II → Project Scientist. WII, ICFRE, FSI, NRSC, state RS centres. Maximum forest time, modest pay, contract renewals.

**Private / conservation tech** — ₹4–8 LPA rising to ₹15–35 LPA. GIS Analyst → Geospatial Data Scientist. Carbon MRV and dMRV firms, Indian and remote international. Portfolio matters more than degree brand.

**Hybrid** — the rare-profile premium. Running a conservation product end to end: sensors → pipeline → dashboard. Very few people in India can run the whole stack.

---

## Why the field grows through 2030

- **Mid 2026:** India's compliance carbon market (CCTS) begins credit trading. Every forest carbon credit needs geospatial verification — paid, recurring MRV work.
- **2026 →:** NISAR operational; NRSC/ISRO building forest and hazard products on it, sustaining government radar-applications hiring.
- **2026 →:** TNFD nature reporting pulls corporate money into biodiversity measurement — bioacoustics, camera traps, eDNA as paid data products.
- **Ongoing:** Himalayan disaster-risk budgets (GLOF, landslide, fire) expanding post-Sikkim 2023, funded by ADB, World Bank, NDMA.
- **Trend:** pure "map maker" GIS roles are being automated. Geospatial + programming + forest domain is getting scarcer and better paid.

---

## Target organisations

**Nearest first:** IFGTB Coimbatore · SACON Anaikatty · Keystone Foundation, Kotagiri (45 min) · TNAU Forest College, Mettupalayam

**Then:** WII Dehradun · FSI Dehradun · GBPNIHE Almora · ICFRE/FRI Dehradun · NRSC/ISRO · DGRE/DRDO · state remote sensing centres

**Portfolio-first employers:** Technology for Wildlife Foundation (Goa) — public GitHub, hires on merged pull requests · NCF Mysore · ATREE Bengaluru · WILDLABS jobs board

Roles: Project Associate I/II, Technical Assistant, JRF. Contractual, 1–3 years, PI-funded, mostly filled by direct interview.

---

## Funding that turns a portfolio into a job

| Scheme | Amount | Note |
|---|---|---|
| **Ravi Sankaran / Inlaks Small Grant** | Up to ₹2 lakh/yr | Best fit. Bachelor's in any subject, under 30, wants unconventional ideas |
| **IdeaWild** | Equipment | AudioMoth, camera trap, laptop |
| **WILDLABS Awards** | USD 10,000 / 50,000 | Open to independent developers of all skill levels |
| **Mohamed bin Zayed** | Up to USD 25,000 | Individuals eligible, single-species, in-situ. Windows 15 Oct 2026, 31 Jan 2027 |
| **Rufford** | GBP 7,000+ | Needs a host organisation — pays orgs, not individuals |

All of these want a pilot to look at. That is what the first project is for. **Note the Mohamed bin Zayed window on 15 October 2026** — inside the five-month build period.

---

## Honest risks

- **The delay only pays off if the portfolio gets built.** Four months with nothing shipped is four months lost, and the argument for waiting evaporates.
- The January intake is inferred, not officially stated. Confirm by phone in September.
- Entry pay in project roles stays low for 2–3 years.
- Contracts are 1–3 years; renewals depend on project funding.
- **IGNOU alone impresses nobody.** The degree is the eligibility paper. The portfolio is the argument.
- No pension, no permanent post, no safety net. That is the price of the open circle — pay it deliberately.

---

## Two inconsistencies to resolve

**1. The "PM background" framing.** The dashboard positions this as "From Product Manager to Forest & Mountain Technologist" and calls PM + geoinformatics the unfair advantage. Earlier project notes describe a small technical studio and ~3 years of tech work, not a product management title. Decide which is honest before it reaches a CV — "ran a technical studio" and "product manager" land very differently, and only one is verifiable.

**2. NET/GATE and the permanent-post question.** The dashboard says the permanent-post tension is "still unresolved; revisit NET/GATE in 2027." That contradicts the no-government decision. NET/GATE do raise JRF pay and open a PhD door, which is a legitimate reason to keep them — but that is not the same as reopening permanent government posts. Keep them separate.

---

## Do this week

1. **Open the GitHub repo and push the first commit.** Even an empty README with the project question stated.
2. Create a Google Earth Engine account and load one Sentinel-2 scene over Sathyamangalam
3. Create a WILDLABS account
4. Start one free IIRS (ISRO) course
5. Put "IGNOU MSCGI applications open" in the calendar for 15 December 2026

No admin. No fees. Nothing due until December.

---

## Sources

- [IGNOU M.Sc. Geoinformatics (MSCGI)](https://www.ignou.ac.in/schools/programme/MSCGI) — eligibility, ₹32,000, 80 credits
- [IGNOU PG Diploma Geoinformatics (PGDGI)](https://www.ignou.ac.in/schools/programme/PGDGI) — ₹16,000, 40 credits
- [IGNOU PG Certificate Geoinformatics (PGCGI)](https://www.ignou.ac.in/schools/programme/PGCGI) — ₹8,000, 20 credits
- [MSCGI Sem-II assignment booklet](https://webservices.ignou.ac.in/assignments/Master-Degree/MSCGI/2025/MSCGI%20Sem-II%20Assignment%20Booklet%20for%20July%202024%20-%20Jan%202025.pdf) — evidence of both July and January cycles
- [IGNOU RC Hyderabad — January 2026 window, 16 Dec to 31 Jan](http://rchyderabad.ignou.ac.in/news/detail/1/IGNOU_Admissions_January_2026___Opened_from_16_12_2025_to_31_01_2026-645)
- [IGNOU regional centre list](https://nmeict.ac.in/wp-content/uploads/2020/05/List-of-Regional-Centres-IGNOU.pdf) — Madurai RC covers Erode (2020 list, verify)
- [WILDLABS jobs](https://wildlabs.net/collection/jobs) · [Getting a Software Job in Conservation](https://wildlabs.net/en/discussion/getting-software-job-conservation)
- [IIRS academic calendar](https://admissions.iirs.gov.in/documents/AcademicCalendar.pdf) — degree programmes need a 4-year B.Sc. or B.E./B.Tech; free short courses are open
- Invasive species, drone, bioacoustics and human-elephant conflict gap details carried forward from `forest-tech-career-context-and-findings.md`

**Reconfirm fees and dates on the Samarth portal at checkout.**
