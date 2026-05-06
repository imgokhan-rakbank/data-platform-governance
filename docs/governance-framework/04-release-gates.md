# Pillar 4: Release Gates

---

## 4.0 Pre-Engineering Prerequisites (Stages 0–3)

Before the G0 gate can be opened, the data project must have completed the four pre-engineering stages
defined in the [Data Project Lifecycle](09-project-lifecycle.md). Each stage is enforced by a Git PR
gate with CI checks and PR approvals. No stage can proceed until the previous stage's PR is merged.

| Lifecycle Stage | Gate | CI/CD Mechanism | Key Exit Criterion |
|----------------|------|-----------------|-------------------|
| **Stage 0 – Data Product Definition** | Definitions in Informatica IDGC | CI queries Informatica API; PR blocked if entry/DQ missing | Data product + DQ definitions confirmed in Informatica IDGC |
| **Stage 1 – Data Mapping** | Mappings complete, layer changes determined | Mapping PR: CI lint + Data Architect approval | Versioned mapping in Git; bronze/silver changes explicitly scoped |
| **Stage 2 – Source Feasibility** | Source feasibility confirmed | Manual review | In-scope table inventory reconciled with production reality; constraints logged |
| **Stage 3 – Physical Design** | Model PR approved; CD deploys physical entities | Model PR: CI lint + Data Architect approval → CD pipeline executes DDL | Modeler-approved DDL in Git; physical entities created in Databricks via CD |

> **Hard rule:** No G0 gate ticket can be opened until all four Stage PRs are merged.

---

## 4.1 Five-Gate Promotion Model

All layer promotions follow a mandatory gate model. No dataset is promoted to a higher layer without the
corresponding gate ticket reaching **Done** status in Jira.

### Gate Summary

| Gate | Lifecycle Stage | Transition | Approver(s) |
|------|----------------|-----------|-------------|
| **G0 – Dev Complete** | Stage 4 | Code merged to `main` | Tech Lead |
| **G1 – Bronze Gate** | Stage 4 (ingest) | Landing → Bronze | Data Engineer + Data Steward |
| **G2 – Silver Gate** | Stage 5 | Bronze → Silver | Data Steward + Platform Lead |
| **G3 – Gold Gate** | Stage 6 | Silver → Gold | Data Owner + Data Analyst |
| **G4 – Semantic Gate** | Stage 7–9 | Gold → Semantic | Business Owner + Platform Lead |

---

## 4.2 Gate Detail & Required Artefacts

### G0 – Dev Complete

**Transition:** Code merged to `main` branch; pipeline is ready for first execution.

**Lifecycle stage:** Stage 4 — Data Engineering Build.
Pre-requisites: Stages 0–3 must be complete (see section 4.0).

**Required artefacts:**

| Artefact | Responsibility | Verification |
|----------|---------------|-------------|
| Pre-engineering stages (0–3) all in Done | Data Engineer | Jira stage tickets |
| Pull request with peer code review approved | Data Engineer | GitHub PR — at least 1 approved review |
| Unit tests passing in CI | Data Engineer | CI pipeline green |
| Pipeline/job name compliant with naming standard | Data Engineer | CI lint check |
| Schema registered in Unity Catalog using modeler-approved DDL only | Data Engineer | `SHOW TABLES IN <catalog>.<schema>` |
| Schema standards checklist passed (see [01-data-standards.md § 1.4](01-data-standards.md)) | Data Architect | Checklist in Jira Stage 3 ticket |
| Column names compliant with naming standard | Data Engineer | CI lint check or manual review |
| Audit record emission verified | Data Engineer | Test run logs showing write to `pipeline_audit` |

---

### G1 – Bronze Gate

**Transition:** Landing → Bronze; dataset is ingested and stored in structured Delta format.

**Required artefacts:**

| Artefact | Responsibility | Verification |
|----------|---------------|-------------|
| Informatica catalog entry created with all mandatory attributes | Data Steward | Informatica IDGC catalog record |
| Profiling run completed in Informatica | Informatica (automated) + Data Steward | Profiling report in Informatica |
| PII columns tagged in Informatica | Data Steward | Informatica column-level PII flags |
| Unity Catalog column masking policies applied for PII columns | Platform Lead | Unity Catalog policy review |
| Landing → Bronze recon check: PASS | Data Engineer | `platform_ops.recon_results` |
| Source → Landing recon check: PASS | Data Engineer | `platform_ops.recon_results` |
| Data classification set and Unity Catalog access policy configured | Platform Lead | Unity Catalog grants review |

---

### G2 – Silver Gate

**Transition:** Bronze → Silver; dataset is cleansed and deduplicated.

**Required artefacts:**

| Artefact | Responsibility | Verification |
|----------|---------------|-------------|
| DQ rules authored for all 6 dimensions in Informatica | Data Steward | Informatica rule set |
| DQ Score ≥ 90 on latest Informatica run | Informatica | DQ Score in Informatica catalog |
| Profiling sign-off by Data Steward | Data Steward | Informatica sign-off record |
| Bronze → Silver recon check: PASS | Data Engineer | `platform_ops.recon_results` |
| Deduplication volume documented (if row count variance > 0) | Data Engineer | Jira ticket comment |
| Business key uniqueness verified in Silver | Data Engineer | `platform_ops.recon_results` |

---

### G3 – Gold Gate

**Transition:** Silver → Gold; dataset is aggregated and business-ready.

**Lifecycle stage:** Stage 6 — Logical Mart Review & Sign-off (Data Acceptance Gate).

**Required artefacts:**

| Artefact | Responsibility | Verification |
|----------|---------------|-------------|
| DQ Score ≥ 95 on latest Informatica run | Informatica | DQ Score in Informatica catalog |
| Business Glossary terms linked to all Gold columns | Data Steward | Informatica glossary linkage |
| Silver → Gold aggregate recon check: PASS | Data Engineer | `platform_ops.recon_results` |
| Data Analyst business logic review sign-off | Data Analyst | Jira ticket comment/approval |
| Decision coverage matrix: each KPI mapped to ≥ 1 business decision | Data Steward + Data Analyst | Jira attachment |
| Data lineage attestation: KPI → Gold → Silver → source (Informatica) | Data Steward | Informatica lineage view |
| Data validated on production-representative volumes (or accepted proxy) | Data Engineer | Volume comparison in Jira comment |
| Exception register: all open DQ/recon exceptions have named owners and expiry | Data Steward | Exception log in Jira |
| Data Owner approval | Data Owner | Jira ticket approval |

---

### G4 – Semantic Gate

**Transition:** Gold → Semantic; dataset is published for consumption by BI tools, analysts, and APIs.

**Lifecycle stages:** Stage 7 (Semantic Build), Stage 8 (UAT), Stage 9 (Production Readiness).

**Required artefacts:**

| Artefact | Responsibility | Verification |
|----------|---------------|-------------|
| DQ Score ≥ 99 on latest Informatica run | Informatica | DQ Score in Informatica catalog |
| UAT sign-off (see [05-uat.md](05-uat.md)) | Business Owner | Jira UAT ticket in Done |
| PT sign-off (see [06-performance-testing.md](06-performance-testing.md)) | Platform Lead | Jira PT ticket in Done |
| Gold → Semantic recon check: PASS | Data Engineer | `platform_ops.recon_results` |
| Semantic parity checklist: Gold logic = BI measure definitions (no divergence) | Data Analyst | Checklist in Jira |
| RLS positive and negative scenario test results (incl. join leakage test) | Data Engineer | Test evidence in Jira |
| Consuming asset register: named owners for all reports, extracts, APIs | Data Steward | Register in Jira |
| Unity Catalog access policies configured for Semantic schema | Platform Lead | Unity Catalog grants review |
| Monitoring & alerting configured and end-to-end tested | Data Engineer | Alert test evidence in Jira |
| Ops runbook published | Data Engineer | Runbook linked in Jira |
| Business Owner approval | Business Owner | Jira ticket approval |
| Platform Lead approval | Platform Lead | Jira ticket approval |

---

## 4.3 Gate Metadata in Unity Catalog

Gate status is stored as Unity Catalog table properties for auditability and automated gate checks:

```sql
ALTER TABLE rakbank_prod.payments_semantic.transactions_v1
SET TBLPROPERTIES (
  'governance.gate_status'       = 'G4_APPROVED',
  'governance.gate_approved_by'  = 'firstname.lastname@rakbank.ae',
  'governance.gate_approved_at'  = '2026-04-30T12:00:00Z',
  'governance.gate_jira_ticket'  = 'DPG-1234'
);
```

Automated pipeline jobs check `governance.gate_status` before promoting data. If the gate is not approved,
the job fails with a descriptive error message.

---

## 4.4 Hotfix / Emergency Promotion Path

### Eligibility

A hotfix promotion bypasses the normal gate sequence only when:

- A production data incident is actively impacting business operations or regulatory reporting.
- The fix cannot wait for the standard gate cycle.

### Approval

- **Dual approval required:** Business Owner + EVP Data delegate (or nominated Data Governance Officer).
- Both approvals must be recorded in the Jira hotfix ticket before deployment.

### Post-Implementation Requirements

| Requirement | Timeline |
|-------------|----------|
| Post-implementation review | Within 5 business days |
| Retroactive gate checklist completion | Within 10 business days |
| All missing artefacts remediated | Within 10 business days |

### Reporting

All hotfix promotions are flagged in the monthly governance report reviewed by the Data Governance Council.
Repeated hotfixes on the same dataset trigger a root-cause analysis.

---

## 4.5 Post-G4: Go-live / Hypercare (Stage 10)

After G4 is approved and the dataset is live in the Semantic layer, the project team enters a mandatory
**hypercare period** before handing over to BAU operations.

### Hypercare Duration

| Dataset Tier | Minimum Hypercare |
|-------------|-----------------|
| Tier 1 (regulatory/financial) | 10 business days |
| Tier 2 (management reporting) | 5 business days |
| Tier 3 (analytics/exploratory) | 3 business days |

### Hypercare Activities

- Daily review of DQ alerts, recon failures, and pipeline SLA breaches.
- All production incidents triaged within the standard SLA (see [03-reconciliation.md § 3.3](03-reconciliation.md)).
- Any variance from agreed post-go-live tolerance triggers immediate escalation to Data Owner.

### Hypercare Exit (BAU Handover)

| Requirement | Verified By |
|-------------|------------|
| No open Critical or High incidents | Platform Lead |
| Post-go-live report published (hypercare logs, performance vs SLA baseline) | Data Engineer |
| Knowledge transfer to BAU support team completed | Data Engineer + Data Steward |
| BAU ownership formally accepted by named Data Engineer and Data Steward | Platform Lead |

See [09-project-lifecycle.md § Stage 10](09-project-lifecycle.md) for the full exit criteria.

---

## 4.6 CI/CD and Gate Automation: New Report / Data Product / Data Integration

This section defines the end-to-end CI/CD workflow and gate automation model for building a new report, data
product, or data integration. All gates are enforced through Git pull request (PR) reviews, automated CI
checks, and CD pipelines. Approvals happen on PR reviews — not through out-of-band sign-off processes.

### 4.6.1 Standard DQ Checks Applied to All Entities

A baseline set of DQ checks is automatically provisioned for every new entity onboarded to the platform.
These checks are deployed by the CD pipeline when an integration PR is merged.

| Check | Description | Threshold |
|-------|-------------|-----------|
| **Record count reconciliation** | Compare record count in target vs source; raise an alert on variance | ≤ 0.1 % variance for batch feeds; agreed threshold for real-time / near-real-time feeds (set per entity in Data Product Brief) |
| **Business key (BK) uniqueness** | Confirm the defined BK is unique in Bronze and Silver | Zero tolerance — zero duplicate BK values |
| **Mandatory column null rate** | Null check on every column marked `NOT NULL` in the DDL | Zero tolerance |
| **Arrival SLA check** | Confirm data arrived within the agreed load SLA | Per-entity SLA as defined in the Data Product Brief |

These checks are in addition to any entity-specific DQ rules authored in Informatica IDGC
(see [02-data-quality.md § 2.5](02-data-quality.md)).

---

### 4.6.2 End-to-End CI/CD and Gate Workflow

The diagram below shows the phases, gates, and CI/CD automation steps for a new build.

```
┌───────────────────────────────────────────────────────────────────────────────┐
│  PHASE 1 — DATA PRODUCT DEFINITION                                            │
│  • Document data product in Informatica IDGC governance portal                │
│  • Define initial DQ definitions for the data product                         │
│                                                                                │
│  ► GATE 1 (PR check): CI confirms data product exists in Informatica IDGC     │
│    with all mandatory attributes; DQ definitions authored                      │
└───────────────────────────────────────────┬───────────────────────────────────┘
                                            │
┌───────────────────────────────────────────▼───────────────────────────────────┐
│  PHASE 2 — DATA MAPPING (GIT-DRIVEN, VERSIONED)                               │
│  • Produce column-level mappings: source → landing → bronze → silver → gold   │
│  • Determine whether new bronze / silver entities or attributes are needed     │
│  • Commit versioned mapping artefact to Git                                    │
│                                                                                │
│  ► GATE 2 (Mapping PR review): Data Architect approves PR; mappings complete; │
│    required layer changes are explicitly documented (not "TBD")                │
└───────────────────────────────────────────┬───────────────────────────────────┘
                                            │
┌───────────────────────────────────────────▼───────────────────────────────────┐
│  PHASE 3 — DATA MODELING + DDL (PR-DRIVEN, CD-DEPLOYED)                       │
│  • Model new / changed silver entities and attributes                          │
│  • Update bronze→silver and silver→gold mappings                               │
│  • Model gold layer and semantic layer for the report / data product           │
│  • Commit DDL (physical table structures) to Git                               │
│                                                                                │
│  ► GATE 3 (Model PR review): Data Architect approves PR; schema standards      │
│    checklist passed; CI lint checks pass                                       │
│    → On merge: CD pipeline creates / alters physical tables in Databricks      │
└───────────┬───────────────────────────────────────────────────┬───────────────┘
            │                                                   │
            │  (new source entities to onboard)                 │  (layer changes needed)
            │                                                   │
┌───────────▼───────────────────────┐             ┌────────────▼───────────────────────┐
│  PHASE 4a — NEW ENTITY ONBOARDING │             │  PHASE 4b — PIPELINE IMPLEMENTATION │
│  (Integration PR)                 │             │  (Feature Branch PRs)               │
│                                   │             │                                     │
│  • Profile source entity          │             │  Silver pipeline:                   │
│  • Determine PK and BK            │             │  • Implement in feature branch      │
│  • Define entity DQ checks;       │             │  • Raise PR; peer review + CI       │
│    standard checks (§ 4.6.1)      │             │  • Approved → CD deploys pipeline   │
│    auto-applied                   │             │                                     │
│  • Register entity in Informatica │             │  Gold pipeline:                     │
│  • Implement ingest pipeline in   │             │  • Same process as silver           │
│    feature branch (landing+bronze)│             │  • PR additionally requires Data    │
│  • Raise integration PR           │             │    Analyst sign-off on business     │
│                                   │             │    logic correctness                │
│  ► GATE 4a (Integration PR):      │             │                                     │
│    CI checks: profiling available,│             │  ► GATE 4b (Pipeline PR):           │
│    DQ definitions confirmed,      │             │    Required approvals + CI checks   │
│    PK/BK defined in schema        │             │    must pass before merge           │
│    → On merge: CD deploys         │             │    → CD deploys on merge            │
│      ingestion jobs               │             │                                     │
└───────────────────────────────────┘             └─────────────────────────────────────┘
```

---

### 4.6.3 Phase 1 — Data Product Definition Gate

**Trigger:** A new report, data product, or data integration is requested.

**Required actions before raising the Phase 1 PR:**

1. Create the data product entry in the Informatica IDGC governance portal with all mandatory catalog
   attributes (see [01-data-standards.md § 1.2](01-data-standards.md)).
2. Author the initial DQ rule set for the data product in Informatica IDGC, covering at minimum the
   standard checks defined in § 4.6.1.
3. Link the data product to relevant business glossary terms and KPIs.

**Gate artefacts and CI/CD mechanism:**

| Artefact / Check | Automated / Manual | CI/CD Mechanism |
|------------------|--------------------|-----------------|
| Data product record in Informatica IDGC with all mandatory attributes | Automated CI check | CI pipeline queries Informatica API; PR blocked if entry missing or incomplete |
| DQ definitions authored in Informatica IDGC | Manual (Data Steward) + CI confirmation | CI pipeline confirms at least one DQ rule set exists for the asset; PR reviewer verifies coverage |
| Business glossary linkage | Manual (Data Steward) | PR description checklist; reviewer verifies |
| PR approved by Data Steward | Manual | GitHub PR: at least 1 Data Steward approval required before merge |

> **Gate mechanism:** CI pipeline queries the Informatica IDGC API on every PR push. The PR is blocked
> (status check fails) until the data product catalog entry and DQ definitions are confirmed present.

---

### 4.6.4 Phase 2 — Data Mapping Gate (Git-Driven, Versioned)

**Required actions before raising the Mapping PR:**

1. Produce column-level source→target mappings for the full path:
   `source → landing → bronze → silver → gold`.
2. Explicitly determine and document whether any **new entities** or **new attributes** are required
   in the bronze or silver layers (answer must be Y/N with scope — not "TBD").
3. Commit the mapping artefact to the Git repository under
   `/mappings/<domain>/<data-product>-v<N>.yaml` (or `.md`), following the versioning convention.
4. Raise a Mapping PR targeting the integration branch.

**Gate artefacts and CI/CD mechanism:**

| Artefact / Check | Automated / Manual | CI/CD Mechanism |
|------------------|--------------------|-----------------|
| Mapping file committed to Git in `/mappings/<domain>/` | Automated CI check | File-presence and path-convention lint on PR |
| Mapping format and version increment compliant | Automated CI check | Schema validation lint on mapping YAML/MD |
| All target columns mapped (no unmapped columns) | Manual (Data Architect PR review) | PR approval required |
| Bronze / silver change determination is explicit (Y/N + scope) | Manual (Data Architect PR review) | PR approval; "TBD" blocks merge |
| PR approved by Data Architect | Manual | GitHub PR: at least 1 Data Architect approval required before merge |

> **Gate mechanism:** The mapping PR requires a passing CI lint (file path + format) **and** at least
> one Data Architect approval. The change determination field in the mapping document must be resolved
> before the approval is granted.

---

### 4.6.5 Phase 3 — Model PR Gate (CD-Deployed to Databricks)

**Required actions before raising the Model PR:**

1. If silver layer changes are required: produce updated logical and physical models for all new or
   changed entities and attributes.
2. Update the mapping artefact (from Phase 2) for bronze→silver and silver→gold to reflect the new model.
3. Model the gold layer and semantic layer tables for the report or data product.
4. Produce DDL scripts (`CREATE TABLE` / `ALTER TABLE`) for all new or changed physical tables.
   Commit DDL under `/ddl/<catalog>/<schema>/` in Git.
5. Complete the schema standards checklist for every new or changed table.
6. Raise a Model PR including: DDL scripts, updated mappings, and completed schema standards checklist.

**Gate artefacts and CI/CD mechanism:**

| Artefact / Check | Automated / Manual | CI/CD Mechanism |
|------------------|--------------------|-----------------|
| DDL scripts committed under `/ddl/<catalog>/<schema>/` | Automated CI check | File-path lint on PR |
| Schema standards checklist completed and attached to PR | Manual (Data Architect PR review) | PR description checklist; reviewer verifies all items |
| DDL naming convention compliance | Automated CI check | CI DDL lint (snake_case, version suffix, etc.) |
| No ad-hoc DDL executed outside the PR process | Automated (environment policy) | Databricks environment policy blocks manual DDL from non-CD service principals |
| PR approved by Data Architect | Manual | GitHub PR: at least 1 Data Architect approval required before merge |

> **Gate mechanism:** Model PR approved + CI checks pass → CD pipeline is triggered on merge. The CD
> pipeline is the **only** permitted mechanism for creating or altering physical table structures in
> Databricks. Manual DDL execution by individuals is blocked by environment policy.

---

### 4.6.6 Phase 4a — New Entity Onboarding (Integration PR)

*Applicable when new source entities need to be ingested into the data platform for the first time.*

**Required actions before raising the Integration PR:**

1. Run data profiling on the source entity in Informatica IDGC.
2. Determine the **Primary Key (PK)** and **Business Key (BK)** for the entity; record in the mapping
   artefact and DDL.
3. Define entity-level DQ checks in Informatica IDGC. The standard checks in § 4.6.1 are automatically
   provisioned by the CD pipeline; the Data Steward must additionally author any entity-specific rules.
4. Register the entity in Informatica IDGC with profiling results, PK/BK metadata, and DQ definitions.
5. Implement the ingestion pipeline (landing + bronze ingest) in a **feature branch**.
6. Raise an **Integration PR** targeting the integration branch.

**Integration PR automated CI checks (all must pass before human review):**

| PR Check | CI/CD Mechanism |
|----------|-----------------|
| Profiling results available in Informatica IDGC for the entity | CI pipeline queries Informatica API |
| DQ definitions (including standard checks in § 4.6.1) authored in Informatica IDGC | CI pipeline queries Informatica API |
| PK and BK declared in DDL and recorded in the mapping artefact | DDL lint + mapping schema validation |
| DDL committed to Git matching the approved model from Phase 3 | File-presence check + DDL diff against model PR |
| Unit tests covering ingestion logic present and passing | CI test execution |
| Pipeline naming convention compliant | CI naming lint |

> **Gate mechanism:** All automated CI checks must pass before the PR is eligible for human review.
> PR requires at least 1 Data Engineer peer approval and 1 Data Steward confirmation that Informatica
> checks are satisfied. On merge → CD pipeline deploys the ingestion jobs to the target environment.

---

### 4.6.7 Phase 4b — Silver and Gold Pipeline Implementation (Feature Branch PRs)

*Applicable when changes to silver or gold layer pipelines are required.*

#### Silver Pipeline

1. Implement the bronze→silver transformation pipeline in a **feature branch** following the naming
   convention `feature/<ticket-id>-<short-description>` and the [coding standards](01-data-standards.md).
2. Raise a PR targeting the integration branch. PR must satisfy:

| Requirement | CI/CD Mechanism |
|-------------|-----------------|
| Feature branch naming convention compliant | CI branch-name lint |
| Peer code review: ≥ 1 Data Engineer approval | GitHub PR approval |
| Unit tests present and passing in CI | CI test execution |
| Naming convention lint (pipeline/job/column) passing | CI naming lint |
| Mapping alignment: all transformations traceable to the versioned mapping artefact | Manual (Data Engineer + Data Architect review) |

3. On PR approval + CI green → merge → CD pipeline deploys the silver pipeline.

#### Gold Pipeline

Same process as the silver pipeline, with one additional requirement:

| Additional Requirement | CI/CD Mechanism |
|------------------------|-----------------|
| Data Analyst sign-off on business logic correctness | GitHub PR: at least 1 Data Analyst approval |

> **Gate mechanism:** PRs cannot be merged without all required approvals and passing CI checks.
> The CD pipeline is triggered automatically on merge to the integration branch and deploys the
> pipeline to the target Databricks environment.
