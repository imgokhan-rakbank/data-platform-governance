# RAKBANK Data Platform Governance

This repository contains the end-to-end governance framework for the RAKBANK data platform.

**Stack:** Databricks on Azure | Medallion Architecture (Landing → Bronze → Silver → Gold → Semantic) |
Informatica IDGC | Unity Catalog

---

## Documentation

| Document | Description |
|----------|-------------|
| [00 – Overview](docs/governance-framework/00-overview.md) | Architecture layer reference, tooling map, design principles |
| [01 – Data Standards](docs/governance-framework/01-data-standards.md) | Naming conventions, metadata standards, coding & pipeline standards |
| [02 – Data Quality](docs/governance-framework/02-data-quality.md) | DQ engine (Informatica), profiling, scoring thresholds, ownership |
| [03 – Reconciliation](docs/governance-framework/03-reconciliation.md) | Recon architecture, checks per layer transition, failure response |
| [04 – Release Gates](docs/governance-framework/04-release-gates.md) | G0–G4 gate model, artefact checklists, hotfix path |
| [05 – UAT](docs/governance-framework/05-uat.md) | UAT scope, entry criteria, test case authoring, sign-off |
| [06 – Performance Testing](docs/governance-framework/06-performance-testing.md) | PT scope, entry criteria, scenarios & thresholds, sign-off |
| [07 – Operating Model](docs/governance-framework/07-operating-model.md) | RACI, role descriptions, governance council, escalation path |
| [08 – Rollout Plan & Metrics](docs/governance-framework/08-rollout-plan.md) | Phased rollout, deliverables, success KPIs, tier classification |
| [09 – Data Project Lifecycle](docs/governance-framework/09-project-lifecycle.md) | Full 11-stage lifecycle (Stage 0–10): inputs, outputs, exit criteria per stage |

---

## Quick Reference: Data Project Lifecycle → Gate Mapping

| Lifecycle Stage | Gate | Key Exit Criterion |
|----------------|------|--------------------|
| Stage 0 – Data Product Definition | Pre-G0 | Signed Data Product Brief; Business Owner named |
| Stage 1 – Data Mapping | Pre-G0 | Source→target mapping approved by Business Owner + Architecture |
| Stage 2 – Source Feasibility | Pre-G0 | Table inventory reconciled with production reality |
| Stage 3 – Physical Design | Pre-G0 | Schema standards checklist passed; DDL signed off |
| Stage 4 – Engineering Build | G0 | CI green; modeler-approved DDL only |
| Stage 4 (ingest) | G1 | Informatica catalog entry + profiling complete |
| Stage 5 – DQ Validation | G2 | DQ ≥ 90; recon PASS |
| Stage 6 – Logical Mart Review | G3 | Business sign-off; decision coverage matrix; lineage attested |
| Stage 7–9 – Semantic + UAT + Prod Readiness | G4 | UAT + PT sign-off; DQ ≥ 99; RLS tested |
| Stage 10 – Go-live / Hypercare | Post-G4 | BAU handover; no open Critical/High incidents |

## Quick Reference: Layer Promotion Requirements

| Promotion | Min DQ Score | Gate | Key Blocker |
|-----------|-------------|------|-------------|
| Landing → Bronze | — | G1 | Informatica catalog entry + profiling run complete |
| Bronze → Silver | 90 | G2 | All 6 DQ dimensions authored + recon PASS |
| Silver → Gold | 95 | G3 | Business glossary linked + Data Analyst review + decision coverage matrix |
| Gold → Semantic | 99 | G4 | UAT sign-off + PT sign-off (parallel tracks) + semantic parity check |