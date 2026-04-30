# Pillar 4: Release Gates

---

## 4.1 Five-Gate Promotion Model

All layer promotions follow a mandatory gate model. No dataset is promoted to a higher layer without the
corresponding gate ticket reaching **Done** status in Jira.

### Gate Summary

| Gate | Transition | Approver(s) |
|------|-----------|-------------|
| **G0 – Dev Complete** | Code merged to `main` | Tech Lead |
| **G1 – Bronze Gate** | Landing → Bronze | Data Engineer + Data Steward |
| **G2 – Silver Gate** | Bronze → Silver | Data Steward + Platform Lead |
| **G3 – Gold Gate** | Silver → Gold | Data Owner + Data Analyst |
| **G4 – Semantic Gate** | Gold → Semantic | Business Owner + Platform Lead |

---

## 4.2 Gate Detail & Required Artefacts

### G0 – Dev Complete

**Transition:** Code merged to `main` branch; pipeline is ready for first execution.

**Required artefacts:**

| Artefact | Responsibility | Verification |
|----------|---------------|-------------|
| Pull request with peer code review approved | Data Engineer | GitHub PR — at least 1 approved review |
| Unit tests passing in CI | Data Engineer | CI pipeline green |
| Pipeline/job name compliant with naming standard | Data Engineer | CI lint check |
| Schema registered in Unity Catalog | Data Engineer | `SHOW TABLES IN <catalog>.<schema>` |
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

**Required artefacts:**

| Artefact | Responsibility | Verification |
|----------|---------------|-------------|
| DQ Score ≥ 95 on latest Informatica run | Informatica | DQ Score in Informatica catalog |
| Business Glossary terms linked to all Gold columns | Data Steward | Informatica glossary linkage |
| Silver → Gold aggregate recon check: PASS | Data Engineer | `platform_ops.recon_results` |
| Data Analyst business logic review sign-off | Data Analyst | Jira ticket comment/approval |
| Data Owner approval | Data Owner | Jira ticket approval |

---

### G4 – Semantic Gate

**Transition:** Gold → Semantic; dataset is published for consumption by BI tools, analysts, and APIs.

**Required artefacts:**

| Artefact | Responsibility | Verification |
|----------|---------------|-------------|
| DQ Score ≥ 99 on latest Informatica run | Informatica | DQ Score in Informatica catalog |
| UAT sign-off (see [05-uat.md](05-uat.md)) | Business Owner | Jira UAT ticket in Done |
| PT sign-off (see [06-performance-testing.md](06-performance-testing.md)) | Platform Lead | Jira PT ticket in Done |
| Gold → Semantic recon check: PASS | Data Engineer | `platform_ops.recon_results` |
| Unity Catalog access policies configured for Semantic schema | Platform Lead | Unity Catalog grants review |
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
