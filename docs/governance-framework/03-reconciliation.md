# Pillar 3: Reconciliation (Recon)

---

## 3.1 Recon Architecture

### Overview

Recon jobs run as **Databricks Workflows** and are triggered automatically at the completion of each
layer-to-layer load job. They are implemented as independent, idempotent jobs that can be re-run safely.

### Results Storage

Recon results are written to the central Delta table:

**Table:** `rakbank_prod.platform_ops.recon_results`

| Column | Type | Description |
|--------|------|-------------|
| `recon_id` | STRING | Unique recon run identifier (UUID) |
| `pipeline_run_id` | STRING | Foreign key to `pipeline_audit.run_id` |
| `transition` | STRING | e.g. `landing_to_bronze`, `bronze_to_silver` |
| `domain` | STRING | Owning domain |
| `entity` | STRING | Target entity/table name |
| `check_name` | STRING | Name of the specific recon check |
| `source_value` | DOUBLE | Measured value at the source layer |
| `target_value` | DOUBLE | Measured value at the target layer |
| `variance_pct` | DOUBLE | Percentage variance `((target - source) / source) * 100` |
| `tolerance_pct` | DOUBLE | Allowed tolerance threshold |
| `status` | STRING | `PASS`, `FAIL`, `WARN` |
| `run_ts` | TIMESTAMP | Recon execution timestamp (UTC) |
| `notes` | STRING | Human-readable detail for failures |

### Informatica Integration

Recon results are surfaced in the Informatica IDGC catalog as a linked quality metric against the target dataset,
giving a unified view of lineage + DQ + recon in one place.

---

## 3.2 Mandatory Recon Checks per Layer Transition

### Source → Landing

| Check | Method | Tolerance |
|-------|--------|-----------|
| Row count vs source API/DB | Compare source system row count to Landing file/table count | **0%** — zero variance allowed |
| File completeness | Verify all expected files/partitions arrived | 0% |

### Landing → Bronze

| Check | Method | Tolerance |
|-------|--------|-----------|
| Row count | Landing row count vs Bronze Delta row count | **0%** |
| Byte size delta | Landing file size vs Bronze Delta size (within expected compression ratio) | **0% raw row count** |
| Schema match | All Landing columns present in Bronze schema | 0% |

### Bronze → Silver

| Check | Method | Tolerance |
|-------|--------|-----------|
| Row count (post-dedup) | Bronze row count vs Silver row count; delta documented as dedup volume | **Documented only** — no numeric threshold; variance must be explained |
| Business key uniqueness | Distinct business keys in Silver = total Silver rows | 0% duplicates on business key |
| Null rate on mandatory columns | Null rate on NOT NULL columns in Silver | 0% |

### Silver → Gold

| Check | Method | Tolerance |
|-------|--------|-----------|
| Aggregate recon — financial sums | Sum of key financial metrics (e.g. `transaction_amount`) in Silver vs Gold | **0.001%** |
| Aggregate recon — record counts by dimension | Count by key dimension (e.g. by `product_type`) in Silver vs Gold | **0.001%** |
| Business key coverage | All Silver business keys present in Gold (where applicable) | 0% unexplained drop |

### Gold → Semantic

| Check | Method | Tolerance |
|-------|--------|-----------|
| Aggregate recon vs Gold | All key metrics summed in Gold match Semantic | **0%** |
| Column-level hash (critical tables) | SHA-256 hash of row-ordered critical columns for small tables (< 1M rows) | 0% — exact match |
| Access policy check | Verify Unity Catalog policies are applied before data is queryable | Must pass |

---

## 3.3 Recon Failure Response

### Automatic Actions (no human intervention required)

1. **Downstream layer jobs are halted automatically** via Databricks Workflow dependency configuration.
   The failed recon job sets a status that downstream jobs check before proceeding.
2. **Alert fires within 5 minutes** to:
   - On-call Data Engineer (PagerDuty / Teams channel)
   - Platform Lead (Teams channel)
3. The recon failure is recorded in `platform_ops.recon_results` with `status = 'FAIL'` and a `notes` field
   describing the specific check and variance.
4. A Jira ticket is auto-created with severity mapped to the layer:

| Layer Transition | Default Severity |
|-----------------|-----------------|
| Source → Landing | Critical |
| Landing → Bronze | Critical |
| Bronze → Silver | High |
| Silver → Gold | Critical (financial domains) / High (others) |
| Gold → Semantic | Critical |

### Manual Override Process

No manual override of a recon failure is permitted without:

1. **Dual approval**: Platform Lead + Data Owner, both recorded in the Jira ticket.
2. A documented root-cause explanation attached to the Jira ticket.
3. The override is flagged in the monthly governance report reviewed by the Data Governance Council.

### Recon Failure SLA

| Severity | Time to Acknowledge | Time to Resolve or Escalate |
|----------|--------------------|-----------------------------|
| Critical | 15 minutes | 2 hours |
| High | 30 minutes | 8 hours |
| Medium | 2 hours | 24 hours |
