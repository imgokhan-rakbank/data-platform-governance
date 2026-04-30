# RAKBANK Data Platform Governance Framework — Overview

**Stack:** Databricks on Azure | Medallion Architecture | Informatica IDGC | Unity Catalog

---

## Purpose

This framework defines the end-to-end governance model for RAKBANK's data platform. It covers data standards,
data quality, reconciliation, release gates, user acceptance testing (UAT), and performance testing (PT) across
the full medallion architecture lifecycle.

---

## Medallion Architecture Layer Reference

| Layer | Purpose | Databricks Zone | Informatica Role | Unity Catalog Scope |
|-------|---------|-----------------|-----------------|---------------------|
| **Landing** | Raw ingest, immutable source copy | Azure Data Lake (ADLS) | Scan, catalog, classify | External location registered |
| **Bronze** | Raw structured, schema-on-read, full history | Delta table | Auto-profile, tag PII | Bronze schema per domain |
| **Silver** | Cleansed, deduplicated, conformed | Delta table | DQ rules applied, lineage tracked | Silver schema per domain |
| **Gold** | Aggregated, business-ready | Delta table | Business glossary linked, DQ score | Gold schema per domain |
| **Semantic** | Consumption-ready, labelled for BI/API | Delta table / Lakehouse Federation | Published to business catalog | Semantic schema, access-controlled |

---

## Framework Pillars

| # | Pillar | Document |
|---|--------|----------|
| 1 | Data Standards | [01-data-standards.md](01-data-standards.md) |
| 2 | Data Quality | [02-data-quality.md](02-data-quality.md) |
| 3 | Reconciliation | [03-reconciliation.md](03-reconciliation.md) |
| 4 | Release Gates | [04-release-gates.md](04-release-gates.md) |
| 5 | User Acceptance Testing | [05-uat.md](05-uat.md) |
| 6 | Performance Testing | [06-performance-testing.md](06-performance-testing.md) |
| 7 | Operating Model | [07-operating-model.md](07-operating-model.md) |
| 8 | Rollout Plan & Metrics | [08-rollout-plan.md](08-rollout-plan.md) |

---

## Tooling Responsibility Map

| Capability | Tool | Owner |
|-----------|------|-------|
| Data Catalog & Lineage | Informatica IDGC | Data Governance Officer |
| Data Profiling | Informatica IDGC | Data Steward |
| DQ Rules & Scoring | Informatica IDGC | Data Steward |
| Metadata Store | Unity Catalog | Platform Lead |
| Access Control (RLS/CLS) | Unity Catalog | Platform Lead |
| Pipeline Orchestration | Databricks Workflows | Data Engineer |
| Recon Results Store | Delta (`platform_ops` schema) | Platform Lead |
| Gate & Defect Tracking | Jira | Data Governance Officer |
| PT Evidence | Databricks Query History + Jira | Data Engineer |

---

## Key Design Principles

1. **Informatica IDGC is the single DQ authority** — no parallel DQ logic in ad-hoc notebooks.
2. **Unity Catalog governs access** (row/column security, PII enforcement) based on classifications set in Informatica.
3. **PT runs in parallel with UAT** — both are required before G4 (Semantic promotion), not sequential.
4. **Data Analysts** are formal participants in G3 review, UAT authoring/execution, and PT execution.
5. **Databricks `platform_ops` schema** is the operational observability store (audit records, recon results).
6. **All code is version-controlled** — direct production edits are prohibited.
