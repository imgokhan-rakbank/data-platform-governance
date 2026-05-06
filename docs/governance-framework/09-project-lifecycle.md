# Data Project Lifecycle

This document defines the end-to-end lifecycle that every RAKBANK data project must follow, from initial
business idea through to production go-live and stabilisation. It is the sequencing backbone that connects the
governance pillars (data standards, DQ, recon, gates, UAT, PT) into a single ordered delivery model.

Every stage has explicit **inputs**, **outputs / evidence**, and **exit criteria (Definition of Done)**. No
stage can be considered complete without its exit criteria being met and recorded in Jira.

---

## Lifecycle Overview

```
Stage 0   Data Product Definition         ─┐
Stage 1   Data Mapping                     │  Pre-Engineering
Stage 2   Source Feasibility & Inventory   │  (before first line of code)
Stage 3   Data + Physical Design          ─┘

Stage 4   Data Engineering Build          ─┐  Engineering Build
          (DDL Change Control)            ─┘  (G0 gate)

Stage 5   DQ Validation (Recon Gate)      ─── G2 Silver gate

Stage 6   Logical Mart Review             ─── G3 Gold gate
          (Data Acceptance Gate)

Stage 7   Semantic / Report Build         ─┐
          (Consumption Gate)               │  G4 Semantic gate
Stage 8   UAT (Usage Validation Gate)      │
Stage 9   Production Readiness            ─┘

Stage 10  Go-live / Hypercare             ─── Post-production
```

### Gate Mapping

| Lifecycle Stage | Governance Gate | Document |
|----------------|----------------|----------|
| Stage 0–3 | Pre-requisites for G0 | [04-release-gates.md § Pre-G0](04-release-gates.md) |
| Stage 4 | G0 – Dev Complete | [04-release-gates.md § G0](04-release-gates.md) |
| Stage 4 (ingest) | G1 – Bronze Gate | [04-release-gates.md § G1](04-release-gates.md) |
| Stage 5 | G2 – Silver Gate | [04-release-gates.md § G2](04-release-gates.md) |
| Stage 6 | G3 – Gold Gate | [04-release-gates.md § G3](04-release-gates.md) |
| Stage 7–9 | G4 – Semantic Gate | [04-release-gates.md § G4](04-release-gates.md) |
| Stage 10 | Post-go-live / BAU | This document § Stage 10 |

---

## Stage 0 — Data Product Definition

**Purpose:** Align scope, business value, and acceptance boundaries. Make use cases, consumers, and
SLAs/OLAs explicit before any technical work starts.

### Inputs

- Use case / regulatory requirement / MI ask (documented in Jira epic)

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| **Data Product Brief** | Decisions supported; primary KPIs; consumers (named individuals/teams); refresh SLA; critical business dimensions; named Data Owner |
| SLAs/OLAs | Load SLA, query response SLA, data freshness SLA |
| Quality & Recon Approach | High-level statement of how quality will be validated and reconciled against source |

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | Business Owner named and formally confirmed |
| 2 | Primary KPIs named (even if not fully specified yet) |
| 3 | Source systems list produced |
| 4 | In-scope entities and subject areas defined at high level |
| 5 | SLAs/OLAs explicitly defined and agreed with consuming teams |
| 6 | Quality and reconciliation approach defined at high level |

> **Hard rule:** No engineering work begins until Stage 0 exit criteria are met and the Jira epic is in
> "Stage 0 – Done" status.

---

## Stage 1 — Data Mapping

**Purpose:** Translate business meaning into data logic. Remove ambiguity before modelling or engineering
work starts. Prevent later "semantic drift" and BI-layer logic invention.

### Inputs

- Approved Data Product Brief (Stage 0)
- Source system list (Stage 0)

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| Source→target mapping | Column-level mapping from source systems to Bronze/Silver/Gold |
| KPI definitions | Each KPI written in precise business language with formula |
| Transformation rules | Business rules applied at each layer transition |
| Reconciliation approach | Agreed method and tolerance for validating each layer transition |

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | KPI definitions written in business language and approved by Business Owner |
| 2 | Source→target mapping reviewed and signed off by Data Product Owner + Data Architecture |
| 3 | Reconciliation logic explicitly agreed (not left as "TBD") |
| 4 | Transformation rules peer-reviewed by Data Analyst for business correctness |

---

## Stage 2 — Source Feasibility & Inventory Gate

**Purpose:** Validate scope and physical realities before modelling or DDL work. Prevent mid-sprint
discoveries of missing history, latency constraints, or unavailable source columns.

### Inputs

- Mapping draft (Stage 1)
- Access to source systems or existing production DDLs

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| Authoritative Table Inventory | Confirmed list of in-scope Silver and Gold tables with row counts and schema snapshots |
| Confirmed in-scope tables list | Subject areas and table counts reconciled against production reality |
| Known constraints log | Latency issues, history gaps, code table problems, access limitations |

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | In-scope table list reconciled with production reality (not estimates) |
| 2 | Any discrepancy between "assumed scope" and "actual scope" is formally resolved or risk-accepted |
| 3 | All source access confirmed (no blocked systems) |
| 4 | Constraints log reviewed by Platform Lead |

---

## Stage 3 — Data + Physical Design Gate

**Purpose:** Ensure the physical model is fully buildable and enforceable — not "80% complete". No DDL
is approved until it passes the schema standards checklist.

### Inputs

- Signed mapping (Stage 1)
- Confirmed table inventory (Stage 2)

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| Logical model updates | Entity-relationship model with grain, conformance, and key decisions documented |
| Physical model | Physical model with keys, datatypes, and naming standards applied |
| Modeler-approved DDL | `CREATE TABLE` / `ALTER TABLE` scripts for all in-scope tables, reviewed and signed off by Data Architect |
| Schema standards checklist | Completed checklist (see below) for every in-scope table |

### Schema Standards Checklist (per table)

Every in-scope table must pass **all** of the following before DDL is approved:

| Check | Requirement |
|-------|-------------|
| Primary Key | PK defined in the logical model **and** implemented in the DDL |
| Foreign Keys | FKs defined where applicable and implemented — or explicitly waived with documented rationale |
| Datatypes | All columns have specific datatypes (no "all-string" default schemas) |
| Naming convention | snake_case applied; boolean prefix `is_`/`has_`; timestamp suffix `_at`/`_dt` |
| Surrogate key policy | Surrogate key applied where required — or explicitly stated as not needed with rationale |
| Partition strategy | Partition column defined and justified (or explicitly not partitioned) |
| Null constraints | Mandatory columns marked `NOT NULL`; nullable columns documented |

> **Hard rule:** "No DDL is approved until it passes the schema standards checklist." DDL execution in
> any environment is only permitted using modeler-reviewed, checklist-passed DDL scripts.

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | Schema standards checklist completed and passed for every in-scope table |
| 2 | DDL reviewed and signed off by Data Architect |
| 3 | Logical model updated to reflect final physical design |
| 4 | DDL stored in version control with a pull request review |

---

## Stage 4 — Data Engineering Build + DDL Change Control Gate

**Purpose:** Build pipelines only against approved schemas. Prevent uncontrolled schema drift.

*This stage is the primary trigger for the **G0 – Dev Complete** governance gate.*

### Inputs

- Modeler-approved DDL (Stage 3)
- Versioned mapping and transformation rules (Stage 1)

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| Bronze → Silver → Gold pipelines | Implemented, tested pipelines following [coding standards](01-data-standards.md) |
| Data tests | Row count checks, null constraint checks, business key uniqueness checks |
| Deployment scripts | Environment promotion plan with rollback procedure |
| G0 gate artefacts | See [04-release-gates.md § G0](04-release-gates.md) |

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | Pipelines run end-to-end in Dev environment with repeatable, deterministic outputs |
| 2 | DDL executed **only** using modeler-reviewed DDL from Stage 3 |
| 3 | All G0 gate artefacts complete (peer review, CI green, schema registered) |
| 4 | No ad-hoc schema changes applied outside the approved DDL scripts |

---

## Stage 5 — Data Quality Validation (Reconciliation Gate)

**Purpose:** Prove data correctness — not just that the pipeline runs without errors.

*This stage maps to the **G2 – Silver Gate** and is the first formal DQ validation checkpoint.*

### Inputs

- Pipeline outputs from Stage 4
- Reconciliation rules agreed in Stage 1

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| DQ scorecard | Scores for completeness, consistency, validity, timeliness (from Informatica) |
| Reconciliation results | Results from `platform_ops.recon_results` vs sources |
| Exception register | Any DQ exceptions with named Business Owner acceptance and review expiry |

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | DQ score ≥ 90 (Bronze → Silver threshold) per Informatica |
| 2 | Reconciliation within agreed tolerances — or exceptions explicitly accepted by Data Owner |
| 3 | Key integrity checks pass: PK uniqueness; FK referential integrity where implemented |
| 4 | All G2 gate artefacts complete (see [04-release-gates.md § G2](04-release-gates.md)) |

---

## Stage 6 — Logical Mart Review & Sign-off (Data Acceptance Gate)

**Purpose:** Business signs off on the **data product**, not visuals. Confirms semantic correctness and
decision fitness before any BI or API consumption layer is built.

*This stage maps to the **G3 – Gold Gate**.*

### Inputs

- DQ scorecard (Stage 5)
- Data dictionary and metric definitions
- Lineage artefacts (logical → physical → source, from Informatica)
- Production-representative datasets (or a justified, accepted proxy)

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| Business sign-off for mart readiness | Formal Jira approval by Data Owner confirming "data is fit for purpose" |
| Decision coverage matrix | Maps each KPI to the specific business or regulatory decisions it supports |
| Data lineage attestation | KPI → Gold table → Silver table → source system, verified in Informatica |
| Exception register | Open data quality or recon exceptions with: named owner, rationale, review expiry date |

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | Business Owner signs off on dataset correctness and KPI definitions |
| 2 | Each KPI mapped to at least one explicit decision scenario in the decision coverage matrix |
| 3 | Lineage verified end-to-end with no semantic logic introduced after the model (no BI-layer business rules) |
| 4 | Data validated on production-representative volumes (or proxy formally accepted) |
| 5 | All quality/recon exceptions have named owners with rationale and review expiry |
| 6 | Dataset is defensible to Audit, Risk, or Regulator where applicable |
| 7 | All G3 gate artefacts complete (see [04-release-gates.md § G3](04-release-gates.md)) |

---

## Stage 7 — Semantic / Report Build (Consumption Gate)

**Purpose:** Build the semantic and BI layer without redefining business logic. Enforce access control,
performance validation, and consumption governance.

*This stage is part of the **G4 – Semantic Gate** prerequisites.*

### Inputs

- Signed Gold mart (Stage 6)
- Approved KPI definitions and lineage (Stage 6)

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| Semantic model | Power BI semantic model (or equivalent), measures, calculated columns |
| RLS documentation | Row-level security persona definitions; test results for positive and negative RLS scenarios |
| Semantic parity checklist | Verification that Gold logic = BI measure definitions (no divergence) |
| Semantic versioning register | Version history; backward-compatibility impact statement |
| Consuming asset register | Named owners for all reports, extracts, and APIs consuming from the Semantic layer |

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | No KPI logic exists **only** in BI — proved via semantic parity check against Gold |
| 2 | All measures trace directly to Gold/mart logic (no additional transformation in BI) |
| 3 | RLS defined using business personas (not report-level logic) |
| 4 | RLS tested for positive scenarios (authorised user sees correct data) AND negative scenarios (unauthorised user is blocked, including join leakage) |
| 5 | Semantic model performance validated for expected concurrency and data volume |
| 6 | Versioning and backward-compatibility impact documented and signed off |
| 7 | All consuming reports, extracts, and APIs have named owners recorded |

---

## Stage 8 — UAT (Usage Validation Gate)

**Purpose:** Validate **decision correctness and usability** — not just visual accuracy. UAT confirms
that business users can make the intended decisions using this data, without needing alternative
spreadsheets or shadow data sources.

*Full UAT process: see [05-uat.md](05-uat.md). This section documents the Stage 8 exit criteria.*

### Inputs

- Semantic layer ready (Stage 7)
- Business test scenarios (authored in [05-uat.md § 5.4](05-uat.md))
- Near-final production-representative datasets

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| UAT results | Test results mapped to business scenarios (logged in Jira) |
| Defect and exception log | All defects with business-assigned severity |
| Pre-go-live recon results | Reconciliation re-run on near-final datasets immediately before go-live |
| Formal decision-level acceptance | Business Owner sign-off statement |

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | UAT completed using **scenario-based testing**: normal operating cases + boundary/edge cases |
| 2 | **Negative tests executed**: invalid dimension combinations, access violations, edge date handling |
| 3 | Reconciliation checks **re-run** on pre-go-live data with no unexplained drift from Stage 5 results |
| 4 | Business Owner provides formal sign-off: *"I would take the intended decision using this data without alternate spreadsheets."* |
| 5 | Accepted variance thresholds explicitly declared and documented for post-go-live monitoring |
| 6 | Named ownership agreed for post-go-live data quality issues |
| 7 | All Critical and High UAT defects resolved; Medium/Low have documented Business Owner risk acceptance |

---

## Stage 9 — Production Readiness (Run/Operate Gate)

**Purpose:** Ensure the data product is fully operable before go-live. Confirm monitoring, alerting,
access control, and support structures are in place.

*This stage is part of the **G4 – Semantic Gate** prerequisites.*

### Inputs

- UAT sign-off (Stage 8)
- PT sign-off (see [06-performance-testing.md](06-performance-testing.md))

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| Monitoring & alerting | Databricks workflow alerts, DQ failure alerts, recon failure alerts configured and tested |
| Ops runbook | Support model, escalation path, common incident response procedures |
| Access controls | Unity Catalog grants, RLS, column masking verified in production |

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | SLA monitoring configured (pipeline duration, query response time) |
| 2 | Alerting tested end-to-end (alert fires to correct on-call channel) |
| 3 | Ops runbook published in Git and linked from the G4 gate Jira ticket |
| 4 | Production access policies verified (no over-permissioning) |
| 5 | Support ownership confirmed: named on-call Data Engineer and Data Steward |
| 6 | All G4 gate artefacts complete (see [04-release-gates.md § G4](04-release-gates.md)) |

---

## Stage 10 — Go-live / Hypercare (Stabilisation Gate)

**Purpose:** Stabilise the data product in production and transition to BAU (Business As Usual)
operations. Hypercare is a defined window during which the project team remains on heightened
support readiness.

### Inputs

- Production readiness sign-off (Stage 9)
- G4 gate approved

### Outputs / Evidence

| Artefact | Description |
|----------|-------------|
| Hypercare logs | Daily summary of alerts, incidents, data quality events during the hypercare window |
| Post-go-live report | Summary of hypercare; open issues; performance vs baseline SLAs |
| Knowledge transfer (KT) record | Evidence that BAU support team has been briefed |

### Hypercare Window

| Dataset Tier | Hypercare Duration |
|-------------|--------------------|
| Tier 1 (regulatory/financial) | Minimum 10 business days |
| Tier 2 (management reporting) | Minimum 5 business days |
| Tier 3 (analytics/exploratory) | Minimum 3 business days |

### Exit Criteria (Definition of Done)

| # | Criterion |
|---|-----------|
| 1 | BAU ownership formally accepted by named Data Engineer and Data Steward |
| 2 | No open Critical or High incidents from hypercare period |
| 3 | Post-go-live report published and reviewed by Platform Lead |
| 4 | KT completed and signed off by BAU support lead |
| 5 | Hypercare logs archived and linked from the project Jira epic |

---

## Lifecycle Governance Roles

| Stage | Primary Owner | Required Sign-offs |
|-------|-------------|-------------------|
| Stage 0 | Data Owner | Business Owner, Data Governance Officer |
| Stage 1 | Data Steward + Data Analyst | Data Product Owner, Data Architecture |
| Stage 2 | Data Engineer | Platform Lead |
| Stage 3 | Data Architect | Platform Lead |
| Stage 4 | Data Engineer | Tech Lead (G0) |
| Stage 5 | Data Steward | Platform Lead (G2) |
| Stage 6 | Data Owner | Data Owner + Data Analyst (G3) |
| Stage 7 | Data Engineer | Platform Lead |
| Stage 8 | Business Owner | Business Owner (UAT sign-off) |
| Stage 9 | Platform Lead | Platform Lead + Business Owner (G4) |
| Stage 10 | Data Engineer | Platform Lead |
