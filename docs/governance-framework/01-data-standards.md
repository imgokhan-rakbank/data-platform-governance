# Pillar 1: Data Standards

---

## 1.1 Naming Conventions

### Unity Catalog Hierarchy

Pattern: `<catalog>.<domain>_<layer>.<entity>_<version>`

| Segment | Rules | Example |
|---------|-------|---------|
| `catalog` | Environment-scoped: `rakbank_prod`, `rakbank_uat`, `rakbank_pt`, `rakbank_dev` | `rakbank_prod` |
| `domain_layer` | Domain in snake_case + layer suffix: `_landing`, `_bronze`, `_silver`, `_gold`, `_semantic` | `payments_silver` |
| `entity_version` | Entity in snake_case + `_v<N>` (increment on breaking schema change) | `transactions_v1` |

**Full example:** `rakbank_prod.payments_silver.transactions_v1`

### Pipeline / Job Naming

Pattern: `<domain>__<source>_to_<target>__<frequency>`

| Token | Allowed Values | Example |
|-------|---------------|---------|
| `domain` | snake_case domain name | `payments` |
| `source_to_target` | layer or system names | `bronze_to_silver` |
| `frequency` | `hourly`, `daily`, `weekly`, `adhoc` | `daily` |

**Full example:** `payments__bronze_to_silver__daily`

This pattern is enforced via the CI gate (see [04-release-gates.md](04-release-gates.md)).

### Column Naming

| Rule | Pattern | Example |
|------|---------|---------|
| General | `snake_case` | `transaction_amount` |
| Boolean | Prefix `is_` or `has_` | `is_pii`, `has_late_fee` |
| Timestamp | Suffix `_at` (epoch/UTC datetime) or `_dt` (date only) | `created_at`, `value_dt` |
| Foreign key | Suffix `_id` | `customer_id` |
| Partition column | Prefix `part_` when not a natural column | `part_load_dt` |

### Informatica Asset Naming

Informatica IDGC asset names **mirror** the Unity Catalog hierarchy so catalog entries resolve 1-to-1 to Delta tables.

- **Technical asset name:** `<catalog>/<domain>_<layer>/<entity>_<version>` (forward-slash separated in Informatica)
- **Business name:** Free-text, human-readable, set by Data Steward

---

## 1.2 Metadata Standards

Every Delta table registered in Unity Catalog **must** have a corresponding Informatica IDGC catalog entry populated with the following attributes before the table is used downstream.

### Required Informatica Catalog Attributes

| Attribute | Description | Mandatory |
|-----------|-------------|-----------|
| Business name | Human-readable name approved by Data Steward | Yes |
| Description | What the dataset contains; business context | Yes |
| Domain | Owning business domain (e.g. Payments, Cards, Risk) | Yes |
| Data Owner | Named individual accountable for the data | Yes |
| Data Steward | Named individual responsible for DQ and metadata | Yes |
| Data Classification | `Public` / `Internal` / `Confidential` / `Restricted` | Yes |
| Retention Policy | Retention period and disposal rule | Yes |
| Legal Hold Flag | Boolean; set by Legal when litigation hold applies | Yes |
| PII Flag (per column) | Tagged in Informatica; drives Unity Catalog column masking | Yes (per column) |
| Refresh Frequency | Expected load cadence | Yes |
| Source System | Originating system(s) | Yes |

### Data Classification → Unity Catalog Access Policy Mapping

| Classification | Unity Catalog Policy |
|---------------|---------------------|
| Public | Read granted to all authenticated users |
| Internal | Read granted to `rakbank_internal` group |
| Confidential | Read granted to named domain group; column masking on PII columns |
| Restricted | Read granted to named individuals only; full dynamic view with row-level security |

### Business Glossary

- Gold and Semantic layer columns **must** have Business Glossary terms linked in Informatica before promotion to the respective layer.
- Glossary terms are owned by the Data Steward and approved by the Data Owner.
- Terms follow the pattern: `<Domain>.<ConceptName>` (e.g. `Payments.SettlementAmount`).

### Lineage Requirements

End-to-end lineage must be traceable in Informatica from:

```
Source System → Landing → Bronze → Silver → Gold → Semantic
```

Lineage is captured automatically by Informatica scanners for Databricks Delta tables. Data Engineers must ensure
Informatica scanner credentials are configured for every Unity Catalog schema.

---

## 1.3 Coding & Pipeline Standards

### Version Control

- All Databricks notebooks, DLT pipelines, and Databricks Jobs configuration are stored in Git.
- Direct edits in the Databricks production workspace are **prohibited**.
- The main branch is protected; all changes require a pull request with at least one peer review.

### Transformation Standards

- Prefer **Delta Live Tables (DLT)** for layer-to-layer transformations; use DLT expectations for inline DQ checks.
- Where DLT is not applicable, use Databricks Jobs with structured JSON logging to stdout.
- Transformations must be idempotent (safe to re-run without duplicating data).
- Use `MERGE INTO` (not `INSERT OVERWRITE`) for incremental loads to preserve audit history.

### Pipeline Audit Record

Every pipeline **must** emit one audit record per run to the central audit table:

**Table:** `rakbank_prod.platform_ops.pipeline_audit`

| Column | Type | Description |
|--------|------|-------------|
| `run_id` | STRING | Unique pipeline run identifier (UUID) |
| `layer` | STRING | Target layer: `landing`, `bronze`, `silver`, `gold`, `semantic` |
| `domain` | STRING | Owning domain |
| `entity` | STRING | Target entity/table name |
| `start_ts` | TIMESTAMP | Pipeline start timestamp (UTC) |
| `end_ts` | TIMESTAMP | Pipeline end timestamp (UTC) |
| `rows_read` | BIGINT | Rows read from source |
| `rows_written` | BIGINT | Rows written to target |
| `rows_rejected` | BIGINT | Rows rejected by DQ rules |
| `status` | STRING | `SUCCESS`, `FAILED`, `PARTIAL` |

### Schema Management

- Schema changes must be backward-compatible where possible (add columns; do not drop or rename).
- Breaking schema changes require incrementing the entity version (`_v2`, `_v3`) and updating the Informatica catalog entry.
- Schema is registered in Unity Catalog before the G0 gate is closed (see [04-release-gates.md](04-release-gates.md)).
