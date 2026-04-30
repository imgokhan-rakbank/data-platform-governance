# Pillar 8: Rollout Plan & Success Metrics

---

## 8.1 Phased Rollout Plan

The framework is delivered in six phases. Phases overlap to allow parallel workstreams.

```
Month:  1    2    3    4    5    6    7    8    9    10
Phase 1 [====]
Phase 2      [=========]
Phase 3           [=========]
Phase 4                [=========]
Phase 5                     [=========]
Phase 6                          [=================]
```

---

### Phase 1 – Foundation (Month 1–2)

**Goal:** Establish the governance baseline; make the framework operational for all new work.

| Deliverable | Owner | Done When |
|-------------|-------|-----------|
| Governance framework published to Git repository | Data Governance Officer | This document set merged to `main` |
| Roles assigned for all active domains | EVP Data | Named individuals confirmed in Jira |
| Naming convention standards communicated and acknowledged | Data Governance Officer | All Data Engineers sign-off record |
| CI gate for pipeline naming convention lint check deployed | Platform Lead | CI pipeline running on all repos |
| Unity Catalog / Informatica IDGC integration audit complete | Platform Lead + DGO | Audit report published; gaps logged in Jira |
| Jira governance workflow templates live (G0–G4, UAT, PT) | Data Governance Officer | Templates active in Jira |
| `platform_ops` schema created in Unity Catalog | Platform Lead | `pipeline_audit` and `recon_results` tables created |

---

### Phase 2 – DQ Baseline (Month 2–4)

**Goal:** Achieve full DQ rule coverage and profiling for all Tier-1 datasets.

**Tier-1 definition:** Datasets that feed regulatory reporting, customer-facing products, or financial close processes.

| Deliverable | Owner | Done When |
|-------------|-------|-----------|
| All Tier-1 datasets profiled in Informatica | Data Steward (per domain) | Profiling runs recorded in Informatica for all Tier-1 |
| DQ rules authored for all 6 dimensions on all Tier-1 Bronze and Silver datasets | Data Steward | Rules active in Informatica; Data Analyst reviewed |
| DQ Score baseline established for all Tier-1 datasets | Data Governance Officer | Baseline recorded in DQ dashboard |
| DQ health dashboard live | Data Governance Officer | Dashboard accessible to Domain Leads and EVP Data |
| Jira webhook from Informatica live (auto-ticket on DQ failure) | Platform Lead | End-to-end tested; first auto-ticket created |
| DQ rule authoring training delivered to all Data Stewards | Data Governance Officer | Attendance recorded |

---

### Phase 3 – Recon Automation (Month 3–5)

**Goal:** Automate recon for all Tier-1 layer transitions; eliminate manual recon.

| Deliverable | Owner | Done When |
|-------------|-------|-----------|
| Recon Databricks Workflows deployed for Source → Landing and Landing → Bronze (all Tier-1) | Data Engineer | Workflows running; results in `recon_results` |
| Recon Databricks Workflows deployed for Bronze → Silver and Silver → Gold (all Tier-1) | Data Engineer | Workflows running; results in `recon_results` |
| Recon Databricks Workflows deployed for Gold → Semantic (all Tier-1) | Data Engineer | Workflows running; results in `recon_results` |
| Recon failure alerts wired to PagerDuty / Teams | Platform Lead | Alert end-to-end tested; test page received |
| Recon results surfaced in Informatica catalog | Data Governance Officer | Linked quality metric visible in Informatica UI |
| Recon failure runbook published | Platform Lead | Runbook in Git; linked from `platform_ops` docs |

---

### Phase 4 – Gates (Month 4–6)

**Goal:** All layer promotions go through the formal G0–G4 gate process.

| Deliverable | Owner | Done When |
|-------------|-------|-----------|
| G0–G4 Jira workflow with mandatory checklists live | Data Governance Officer | Workflows enforced in Jira |
| Unity Catalog gate metadata properties implemented in all pipelines | Data Engineer | `governance.gate_status` set on all Tier-1 tables |
| Automated gate check in pipeline jobs (fail if gate not approved) | Data Engineer | End-to-end tested on 1 pilot domain |
| Training delivered: Data Engineers (G0–G2) | Data Governance Officer | Attendance recorded |
| Training delivered: Data Stewards (G1–G3) | Data Governance Officer | Attendance recorded |
| Training delivered: Data Owners and Analysts (G3–G4) | Data Governance Officer | Attendance recorded |
| First dataset promoted end-to-end through G0–G4 gates | All | Jira gate tickets all in Done |

---

### Phase 5 – UAT & PT (Month 5–7)

**Goal:** UAT and PT environments operational; first full UAT+PT cycle completed for 2 pilot domains.

| Deliverable | Owner | Done When |
|-------------|-------|-----------|
| UAT Databricks workspace (`rakbank-uat`) provisioned | Platform Lead | Workspace live; `rakbank_uat` catalog accessible |
| PT Databricks workspace (`rakbank-pt`) provisioned | Platform Lead | Workspace live; `rakbank_pt` catalog accessible |
| Data anonymisation pipeline for UAT data loading deployed | Data Engineer | Anonymised data in `rakbank_uat` for pilot domains |
| UAT Jira epic template live | Data Governance Officer | Template in Jira |
| PT Jira epic template live | Data Governance Officer | Template in Jira |
| PT Results Summary template published | Data Governance Officer | Template in Git |
| First full UAT cycle completed (2 pilot domains) | Data Analyst + Business User | UAT Jira epics in Done for both pilots |
| First full PT cycle completed (2 pilot domains) | Data Engineer + Data Analyst | PT Jira epics in Done for both pilots |
| UAT + PT lessons learned documented | Data Governance Officer | Retrospective report in Git |

---

### Phase 6 – Full Coverage (Month 7–10)

**Goal:** All pillars applied to Tier-2 and Tier-3 datasets; continuous improvement cycle established.

| Deliverable | Owner | Done When |
|-------------|-------|-----------|
| All Tier-2 datasets profiled and DQ rules authored | Data Steward | Verified in Informatica |
| All Tier-2 datasets on G0–G4 gate process | Data Governance Officer | Jira gate tickets for all Tier-2 |
| Recon automation extended to Tier-2 and Tier-3 | Data Engineer | Workflows running |
| Tier-3 profiling and DQ rule backlog prioritised | Data Governance Officer | Backlog in Jira with owners |
| Continuous improvement: quarterly DQ rule review cycle established | Data Steward | First quarterly review complete |
| Framework v2 planning — lessons learned incorporated | Data Governance Officer | v2 scope agreed in Governance Council |

---

## 8.2 Success Metrics

### Primary KPIs (reviewed at monthly Data Governance Council)

| Metric | Target | Measurement Method | Timeline |
|--------|--------|--------------------|----------|
| DQ Score ≥ 95 on 100% of Tier-1 Gold datasets | 100% of datasets | Informatica DQ dashboard | 6 months |
| Zero undetected recon breaks on financial datasets | 0 breaks | `recon_results` table; incident log | 3 months post Phase 3 |
| All Semantic promotions through full G0–G4 gate | 100% | Jira gate ticket audit | 4 months |
| UAT pass rate on first submission | ≥ 80% | Jira UAT epic metrics | Ongoing |
| P95 query response time on Semantic layer | < 5 seconds | Databricks SQL query history | Ongoing |
| Informatica catalog coverage — Tier-1 datasets | 100% | Informatica catalog completeness report | 3 months |
| Mean time to detect (MTTD) data incidents | −60% vs baseline | Incident log comparison | 6 months |

### Secondary KPIs

| Metric | Target | Timeline |
|--------|--------|----------|
| DQ rule coverage (all 6 dimensions) on Tier-1 Silver datasets | 100% | 4 months |
| Recon automation coverage — Tier-1 layer transitions | 100% | 5 months |
| PII column tagging coverage — Tier-1 Bronze datasets | 100% | 3 months |
| Business Glossary term linkage — Gold and Semantic columns | 100% | 6 months |
| Hotfix promotions as % of total Semantic promotions | < 5% | Ongoing |
| Average gate cycle time (G0 → G4) | < 10 business days | Ongoing |

---

## 8.3 Tier Classification

| Tier | Criteria | Examples |
|------|---------|---------|
| **Tier 1** | Feeds regulatory reporting, customer products, or financial close | Core banking transactions, AML feeds, CBUAE reports |
| **Tier 2** | Feeds internal management reporting or operational dashboards | Branch performance, product profitability |
| **Tier 3** | Analytics and exploratory datasets; no direct regulatory dependency | Marketing segmentation, churn models |
