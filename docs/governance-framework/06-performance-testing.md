# Pillar 6: Performance Testing (PT)

---

## 6.1 Scope

PT is **mandatory** for:

- Any new dataset or pipeline in the Semantic layer.
- Any change to a Gold or Semantic pipeline that:
  - Alters compute configuration (cluster type, size, auto-scaling settings).
  - Modifies join logic or introduces new joins.
  - Changes data volume by more than **20%** (growth or reduction).
- Any new BI report, dashboard, or API consuming from the Semantic layer.

PT is **not required** for:

- Non-breaking additive schema changes with no transformation logic impact.
- Changes to metadata only (table properties, comments, tags).
- Infrastructure changes that do not affect pipeline execution (e.g. storage account migration with no job change).

---

## 6.2 PT Environment

| Item | Detail |
|------|--------|
| Databricks workspace | Dedicated PT workspace: `rakbank-pt` |
| Unity Catalog | Separate catalog: `rakbank_pt` |
| Data | Production-scale data volume (full production copy or statistically representative sample ≥ 80% of production volume) |
| Cluster | Isolated, dedicated PT cluster — not shared with production or UAT |
| Access | Data Engineers and Data Analysts only |
| Isolation | Fully isolated from production and UAT environments |

---

## 6.3 PT Entry Criteria

All of the following must be true before PT begins:

| Criterion | Verified By |
|-----------|------------|
| G3 gate is closed (Jira ticket in Done) | Platform Lead |
| PT environment loaded with production-scale data | Data Engineer |
| PT Jira epic created | Data Governance Officer |
| Baseline SLAs documented and approved | Platform Lead + Data Owner |
| PT test scenarios defined (see section 6.4) | Data Engineer + Data Analyst |
| PT cluster configured and isolated | Platform Lead |

### Baseline SLA Definition

Before PT runs, the following SLAs must be agreed and documented in the PT Jira epic:

| SLA Parameter | Who Defines It | Example |
|--------------|---------------|---------|
| Pipeline end-to-end job duration | Data Engineer + Platform Lead | ≤ 45 minutes for daily Gold → Semantic run |
| P95 query response time (Semantic layer) | Data Analyst + Platform Lead | < 5 seconds |
| Daily DBU budget per pipeline run | Platform Lead + Finance | ≤ 120 DBUs per daily run |

---

## 6.4 PT Scenarios & Thresholds

All five scenarios are mandatory for every PT execution.

### Scenario 1: Full Pipeline Run

| Metric | Measurement Method | Threshold |
|--------|--------------------|-----------|
| End-to-end job duration (Gold → Semantic) | Databricks job run history | ≤ Agreed SLA × 1.2 |
| Pipeline success rate | Job run status | 100% (no errors) |

### Scenario 2: Concurrent Analyst Query Load

| Metric | Measurement Method | Threshold |
|--------|--------------------|-----------|
| P95 query response time | Databricks SQL query history | < 5 seconds |
| P99 query response time | Databricks SQL query history | < 15 seconds |
| Query error rate | Databricks SQL query history | 0% |

Simulation: minimum **10 concurrent queries** from different user sessions against the Semantic layer.

### Scenario 3: Peak Load Simulation

| Metric | Measurement Method | Threshold |
|--------|--------------------|-----------|
| Job duration at 2× normal data volume | Databricks job run history | ≤ Agreed SLA × 1.5 |
| Error rate at 2× normal data volume | Job run logs | **0%** |
| Memory spill to disk | Databricks cluster metrics | < 10% of total shuffle |

### Scenario 4: DBU Consumption

| Metric | Measurement Method | Threshold |
|--------|--------------------|-----------|
| DBUs consumed per pipeline run | Databricks cluster usage report | Within approved budget ± 15% |
| DBUs consumed per query (Databricks SQL) | Databricks SQL query history | Within approved per-query budget |

### Scenario 5: Unity Catalog Query Concurrency

| Metric | Measurement Method | Threshold |
|--------|--------------------|-----------|
| Query queue depth at peak | Databricks SQL warehouse metrics | < 10 queued queries at peak |
| Time to first byte (TTFB) | Databricks SQL query history | < 2 seconds |

---

## 6.5 PT Execution

### Participants

| Role | PT Responsibility |
|------|-----------------|
| Data Engineer | Execute pipeline-level scenarios (1, 3, 4); capture Databricks job metrics |
| Data Analyst | Execute query-level scenarios (2, 5); validate query response times |
| Platform Lead | Review results; approve or reject PT sign-off |

### Execution Process

1. Data Engineer and Data Analyst execute all five scenarios in the PT Databricks workspace.
2. Evidence is captured for each scenario:
   - **Databricks job run history** screenshots / exports (Scenarios 1, 3, 4)
   - **Databricks SQL query history** exports (Scenarios 2, 5)
   - **Cluster metrics** from Databricks UI (Scenarios 3, 4)
3. Evidence is attached to the PT Jira epic.
4. Results are summarised in a PT Results table (see section 6.6) within the Jira epic.

### Threshold Breach Response

If any threshold is breached:

1. The breach is documented in the PT Jira epic with the measured value vs threshold.
2. A **performance review** is initiated — Data Engineer + Platform Lead triage the root cause.
3. Optimisation options are assessed:
   - Cluster sizing / auto-scaling tuning
   - Query optimisation (Z-ordering, partitioning, bloom filters on Delta tables)
   - Pipeline logic simplification
   - Databricks SQL warehouse sizing
4. PT re-runs after optimisation until all thresholds pass.
5. G4 **cannot open** until all PT thresholds are met.

---

## 6.6 PT Results Summary Template

The following table must be completed and attached to the PT Jira epic:

| Scenario | Metric | Threshold | Measured Value | Status |
|----------|--------|-----------|---------------|--------|
| Full pipeline run | Job duration | ≤ `<agreed_sla>` × 1.2 | `<measured>` | PASS / FAIL |
| Concurrent queries | P95 response time | < 5s | `<measured>` | PASS / FAIL |
| Concurrent queries | P99 response time | < 15s | `<measured>` | PASS / FAIL |
| Peak load (2×) | Job duration | ≤ `<agreed_sla>` × 1.5 | `<measured>` | PASS / FAIL |
| Peak load (2×) | Error rate | 0% | `<measured>` | PASS / FAIL |
| DBU consumption | DBUs per run | Within budget ± 15% | `<measured>` | PASS / FAIL |
| UC concurrency | Queue depth at peak | < 10 | `<measured>` | PASS / FAIL |

---

## 6.7 PT Sign-Off

PT sign-off is provided by the **Platform Lead** in Jira:

1. All PT scenario results are recorded with evidence.
2. All thresholds are met (no open threshold breaches).
3. Platform Lead adds a formal approval comment to the PT Jira epic.
4. The PT Jira epic is moved to **Done**.

PT sign-off runs in **parallel with UAT** (see [05-uat.md](05-uat.md)) — both must be complete before G4 opens.
Starting PT and UAT concurrently after G3 closure minimises the total calendar time to Semantic promotion.
