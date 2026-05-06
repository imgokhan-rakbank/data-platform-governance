# Pillar 4: Release Gates

---

## 4.0 Pre-Engineering Prerequisites (Stage 0–3)

Before the G0 gate can be opened, the data project must have completed the four pre-engineering stages
defined in the [Data Project Lifecycle](09-project-lifecycle.md). These stages ensure that scope, business
value, physical design, and DDL are fully resolved before the first line of pipeline code is written.

| Lifecycle Stage | Key Exit Criterion | Jira Status Required |
|----------------|-------------------|---------------------|
| **Stage 0 – Data Product Definition** | Signed Data Product Brief; Business Owner named; KPIs and SLAs agreed | Stage 0 ticket → Done |
| **Stage 1 – Data Mapping** | Source→target mapping signed off by Business Owner + Data Architecture | Stage 1 ticket → Done |
| **Stage 2 – Source Feasibility** | In-scope table inventory reconciled with production reality; constraints logged | Stage 2 ticket → Done |
| **Stage 3 – Physical Design** | Schema standards checklist passed; modeler-approved DDL in Git | Stage 3 ticket → Done |

> **Hard rule:** No G0 gate ticket can be opened until all four Stage tickets are in Done status.

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
