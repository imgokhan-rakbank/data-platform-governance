# Pillar 2: Data Quality (DQ)

---

## 2.1 DQ Engine: Informatica IDGC

Informatica IDGC is the **single DQ authority** for the RAKBANK data platform.

- All DQ rules are **authored, versioned, and executed in Informatica** — not duplicated in ad-hoc notebook logic.
- Informatica publishes a DQ Score per dataset per run, which is used as the gate criterion for layer promotion.
- DQ rule results are surfaced in the Informatica catalog alongside the dataset entry.

### DQ Dimensions

All six dimensions must be covered for every dataset before it is promoted past Silver.

| Dimension | Informatica Rule Type | Blocking Layer Transition |
|-----------|----------------------|--------------------------|
| Completeness | Null/blank checks on mandatory columns | Bronze → Silver |
| Accuracy | Reference data lookups, range checks | Silver → Gold |
| Consistency | Cross-domain join checks, referential integrity | Silver → Gold |
| Timeliness | Arrival SLA check, processing SLA check | Bronze |
| Uniqueness | Duplicate detection on business key | Bronze → Silver |
| Validity | Pattern, format, domain/enum checks | Bronze → Silver |

---

## 2.2 Data Profiling

### Ownership and Tooling

Data profiling is **owned by the Data Steward and executed by Informatica IDGC**.

### Trigger Points

| Trigger | Action |
|---------|--------|
| New dataset landed in Bronze | Automated profiling run initiated by Informatica scanner |
| Schema change detected on an existing Bronze table | Automated re-profile run |
| Data Steward request | Manual profiling run via Informatica UI |

### Profiling Outputs

Informatica captures the following statistics per column:

- Value distribution (frequency histogram)
- Null rate and blank rate
- Cardinality (distinct value count)
- Min / Max / Mean / Median (numeric columns)
- Pattern frequency (string columns)
- Inferred data type vs declared data type

### Steward Sign-Off

No dataset progresses past Silver without:

1. A completed profiling run recorded in Informatica.
2. Data Steward review and sign-off on the profile report in Informatica.
3. DQ rules authored for all applicable dimensions (see section 2.1).

---

## 2.3 DQ Scoring & Thresholds

### Scoring

Informatica publishes a **composite DQ Score (0–100)** per dataset per run, calculated as a weighted average
across all active DQ rule results for the dataset.

### Layer Promotion Thresholds

| Promotion | Minimum DQ Score | Exception Path |
|-----------|-----------------|----------------|
| Bronze → Silver | 90 | Data Owner waiver + logged exception in Jira |
| Silver → Gold | 95 | Data Owner waiver + Risk team sign-off |
| Gold → Semantic | 99 | **No exceptions** for financial or regulatory datasets |

### Consecutive Failure Escalation

Any dataset with **3 consecutive runs** below the applicable threshold is automatically:

1. Flagged with a `dq-review-board` label in Jira.
2. Added to the agenda of the next **DQ Review Board** session.
3. Blocked from further layer promotion until the Review Board resolves the root cause.

---

## 2.4 DQ Ownership & Incident Response

### Ownership Matrix

| Role | DQ Responsibility |
|------|------------------|
| Data Steward | Author and maintain DQ rules in Informatica for their domain |
| Data Analyst | Validate that DQ rules reflect business logic; review rule coverage at G3 |
| Data Owner | Grant DQ score exception waivers (Silver and Gold layers only) |
| Data Engineer | Investigate and fix pipeline-level causes of DQ failures |
| Data Governance Officer | Monitor overall DQ health; administer Informatica IDGC |

### Jira Ticket Creation (Automated via Informatica Webhook)

| Severity | Trigger | SLA to Resolution |
|----------|---------|-------------------|
| Critical | DQ score drops below threshold on a financial/regulatory dataset | 4 hours |
| High | DQ score drops below threshold on a Tier-1 dataset | 24 hours |
| Medium | Individual rule failure rate > 5% on any dataset | 72 hours |
| Low | Individual rule failure rate 1–5% | Next sprint |

### Monthly DQ Health Report

- Exported from Informatica IDGC by the Data Governance Officer.
- Distributed to Domain Leads and EVP of Data.
- Contents: DQ score trends by domain/layer, open exceptions, consecutive failure flags, rule coverage gaps.

---

## 2.5 Standard DQ Checks Applied to All Entities

A baseline set of DQ checks is automatically provisioned for every entity onboarded to the data platform.
These checks are deployed by the CD pipeline when an integration PR is merged
(see [04-release-gates.md § 4.6.1](04-release-gates.md) and [09-project-lifecycle.md § Stage 4a](09-project-lifecycle.md)).
They are authored as Informatica IDGC DQ rules and are in addition to any entity-specific rules.

### Baseline DQ Rule Set

| Rule | Informatica Rule Type | Threshold | Applies To |
|------|-----------------------|-----------|------------|
| **Record count reconciliation** | Row count check against source system | ≤ 0.1 % variance for batch feeds; threshold defined per entity in Data Product Brief for real-time / near-real-time feeds | Landing → Bronze, Bronze → Silver |
| **Business key (BK) uniqueness** | Duplicate detection on the defined BK column(s) | Zero tolerance — no duplicate BK values | Bronze, Silver |
| **Mandatory column null rate** | Null / blank check on every `NOT NULL` column | Zero tolerance | Bronze, Silver |
| **Arrival SLA check** | Data arrival timestamp vs agreed load SLA | Per-entity SLA from Data Product Brief | Landing |

### Provisioning

- The standard rule set is provisioned automatically by the CD pipeline when an Integration PR is merged.
- The entity's PK and BK must be declared in the DDL and mapping artefact before the CD pipeline can
  provision the uniqueness check.
- Thresholds for the record count reconciliation check are read from the Data Product Brief at
  provisioning time. If no threshold is specified, the default of 0.1 % is applied.

### Entity-Specific Rules

In addition to the standard baseline, the Data Steward must author entity-specific DQ rules in
Informatica IDGC covering the remaining applicable DQ dimensions (see § 2.1) before the entity can
progress past the G1 – Bronze Gate.
