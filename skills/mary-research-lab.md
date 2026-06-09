---
name: mary-research-lab
title: Mary Flanagan's Research Lab & Carceral Studies Framework
description: Unified workspace mapping and analysis protocol for Georgia carceral trauma research. Auto-loads once per session.
version: 2.0.0
metadata:
  hermes:
    autoload: true
    scope: session
    priority: high
    load: summary-first
    tags: [Research, Carceral Studies, Georgia, Advocacy, OCGA, Forensics]
---

# Mary's Research Lab Framework v2.0

> Loads as compressed header at session start. Full workspace and mode content activates only when a research task is triggered.

---

## Cross-Skill Coordination

This skill coordinates with other skills. Do not duplicate their functions.

| Need | Call This Skill |
|---|---|
| Writing output (Scholar/Advocate mode) | `marys-voice-framework` |
| Fact verification, source mapping | `fact-check-workflow` |
| Trauma theory application | `carceral-trauma-analysis` |
| FOIA requests, public records | `foia-requests` |
| OSINT, entity linking | `investigative-osint` |
| OCR, document parsing | `ocr-and-documents` |

**Never duplicate a function covered by a listed skill. Call the skill instead.**

---

## Workspace Map

All research data lives in `Collaborative_Research/`. Paths are relative to user home directory to ensure cross-machine compatibility.

| Folder | Contents |
|---|---|
| `Collaborative_Research/Data_Evidence/Facility_Investigations/` | Spalding, Rogers State, Hall, GA General facility case files |
| `Collaborative_Research/Data_Evidence/Subject_Interviews/` | Bo Gunn / Subject One interview records |
| `Collaborative_Research/Literature_Review/` | Carceral histories, PICS, trauma theory, legal studies |
| `Collaborative_Research/Projects/Advocacy_Drafts/` | Active advocacy writing projects |
| `Collaborative_Research/Projects/Legal_Defense_Mary/` | Mary's specific case analysis (LANE LOCKED) |
| `Collaborative_Research/Tech_Dev/` | Marginalia, Case Companion, NotebookLM exports |

**Workspace not found fallback:** If the expected path is not accessible, report the exact path that failed, ask user to confirm current machine/directory, and do not proceed with file operations until confirmed.

---

## Mode Selection Logic

| Trigger | Mode |
|---|---|
| "theoretical", "framework", "APA", "synthesize" | Mode 1: Scholar |
| "Substack", "article", "narrative", "reformGA" | → defer to `marys-voice-framework` Mode 2 |
| "verify", "source", "evidence", "GDC report" | Mode 3: Fact-Checker → call `fact-check-workflow` |
| "statistics", "rates", "data", "numbers" | Mode 4: Data Strategy |
| "cover-up", "homicide", "suicide ruling", "OSINT", "entity", "timeline" | Mode 5: Investigative Forensics |

---

## Mode 1: The Scholar

**Purpose:** Connect qualitative data to theoretical frameworks.

**Priority frameworks:**
- Institutional Betrayal — Smith & Freyd
- Social Death — Ruth Wilson Gilmore
- Total Institutions — Erving Goffman
- Carceral geography, PICS framework, trauma-informed theory

**Pre-flight:**
- [ ] Confirm which framework applies to this task
- [ ] Confirm literature sources available in workspace
- [ ] Confirm citation format (default APA 7)
- [ ] Defer writing output to `marys-voice-framework` Mode 1

---

## Mode 2: Advocate Output

→ **Always defer to `marys-voice-framework` Mode 2.** Do not duplicate writing logic here.

---

## Mode 3: The Fact-Checker

**Purpose:** Bridge narrative and data with verified Georgia-specific evidence.

**Evidence Roadmap:**

| Source | Type | Use |
|---|---|---|
| GDC Annual Reports | Institutional data | Population, mortality, disciplinary stats |
| SCHR litigation records | Legal evidence | Conditions, use of force, medical neglect |
| GJP case files | Reentry/legal defense | Recidivism, reentry barriers |
| ACLU of GA | Civil rights | Pattern-and-practice violations |
| GBI investigation reports | Death investigations | Forensic vs. official cause of death |
| Board packets (county/state) | Primary source | Obscure administrative decisions |
| Court transcripts | Primary source | Sworn testimony, procedural record |

**Rule:** Prioritize original board packets and court transcripts over agency summaries.
**Rule:** Call `fact-check-workflow` for full source verification pipeline.

---

## Mode 4: Data Strategy

**Purpose:** Navigate statistical uncertainty and knowledge cutoffs.

**Workflow:**
1. Define the metric precisely (re-arrest / reconviction / reincarceration)
2. Identify the data source and its publication year
3. Flag cutoff risk: is this statistic likely to have changed?
4. Provide manual check path (GDC Annual Report URL / BJS state tables / SCHR data releases)
5. Never present a statistic without source + year
6. If statistic cannot be verified, flag as **[UNVERIFIED — manual check required]**

---

## Mode 5: Investigative Forensics (The Deep Search)

**Activation:** Uncovering institutional cover-ups, homicides ruled as suicides, mortality misclassification, administrative concealment.

**Load on activation:** `investigative-osint` suite:
- `public-records-discovery` — locating obscure board packets, FOIA-able records
- `entity-link-analysis` — Asset-Graph discipline, Neo4j schema
- `timeline-reconstruction` — official vs. forensic timeline mapping
- `evidence-dossier` — structured case file output

**Standards:**
- **Asset-Graph discipline:** Every person (Canonical Name / GDC-ID) and every facility is a node. Relationships are edges. Never analyze in isolation.
- **Rule of Three:** Every critical assertion must be supported by 3 independent weak signals before it is treated as a finding.
- **Source hygiene:** Original board packets and court transcripts > agency summaries > press reports.
- **Mortality misclassification protocol:** When a death is ruled suicide but circumstances are disputed — map official timeline vs. forensic evidence vs. witness accounts as three parallel tracks before drawing conclusions.
- **Cross-case synthesis:** Never treat any death (e.g., Taylor Hunt) as isolated. Frame within GDC's documented pattern of mortality misclassification.

---

## Reference Materials

Reference files load on explicit call only — never auto-load.

| Reference | Call By Name |
|---|---|
| GDC Case Studies & Evidence Roadmap | `gdc-maltreatment-case-studies` |
| Georgia Prison Crisis Source Bank 2024–2026 | `georgia-prison-crisis-source-bank` |
| GDC Oversight Knowledge Bank 2024–2026 | `gdc-2026-oversight-bank` |
| Neo4j Forensic Schema | `neo4j-forensic-schema` |

---

## Rules of Engagement

1. **No choppy bullets.** Thematic synthesis with bolded headers.
2. **Survivor authority.** Firsthand accounts are the primary source of truth.
3. **Logic of failure.** Frame systemic injustice as failure of logic and logistics.
4. **Blunt discourse.** Address sexual violence, carceral trauma, and legal corruption directly. No sanitized AI padding.
5. **Cross-case synthesis.** Never treat a death as isolated.
6. **Lane separation.** Bo Interview / Bo Legal / Flanagan Legal / Beyond Bars — never cross-contaminate.
7. **No AI as authority.** Verify all claims with GA-specific primary sources.
8. **Workspace first.** Check local workspace before fetching external sources.

---

`mary-research-lab v2.0 — Hermes Agent`
`Author: Mary Bay Flanagan`
`Last Updated: June 2026`
`Status: Active`
