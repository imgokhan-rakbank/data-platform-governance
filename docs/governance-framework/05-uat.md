# Pillar 5: User Acceptance Testing (UAT)

---

## 5.1 Scope

UAT is **mandatory** for:

- Any dataset reaching the Semantic layer for the first time.
- Any **breaking change** to an existing Semantic dataset (schema change, business logic change, aggregation
  logic change, or access policy change that affects existing consumers).
- Any new downstream report, BI dashboard, or API that consumes from the Semantic layer.

UAT is **not required** for:

- Non-breaking additive changes (adding a new column with no impact on existing columns or logic).
- Infrastructure-only changes with no data transformation impact (cluster resize, storage optimisation).

---

## 5.2 UAT Environment

| Item | Detail |
|------|--------|
| Databricks workspace | Dedicated UAT workspace: `rakbank-uat` |
| Unity Catalog | Separate catalog: `rakbank_uat` (mirrors `rakbank_prod` structure) |
| Data | Anonymised or synthetic data, production-representative in volume and distribution |
| Access | Business users, Data Analysts, Data Stewards; no write access for business users |
| Isolation | Fully isolated from production and PT environments |

**Data Anonymisation Standard:**

- PII columns are masked using Informatica's data masking capability before loading to the UAT catalog.
- Synthetic data generation is used for Restricted-classification datasets where even masked data is not
  permitted outside production.
- The anonymisation process is documented and approved by the Data Privacy Officer before UAT begins.

---

## 5.3 UAT Entry Criteria

All of the following must be true before UAT begins:

| Criterion | Verified By |
|-----------|------------|
| G3 gate is closed (Jira ticket in Done) | Platform Lead |
| UAT environment loaded with current anonymised/synthetic data | Data Engineer |
| UAT Jira epic created with acceptance criteria approved by Business Owner | Data Governance Officer |
| UAT test cases authored and reviewed (see section 5.4) | Data Steward + Data Analyst |
| UAT workspace access granted to all UAT participants | Platform Lead |
| Runbook draft available for UAT participants | Data Engineer |

---

## 5.4 Test Case Authoring

Test cases are co-authored by the **Data Steward and Data Analyst** and stored as sub-tasks under the UAT Jira
epic.

### Mandatory Test Case Categories

| Category | Description | Example |
|----------|-------------|---------|
| Expected aggregates | Verify key metrics match expected values at business-agreed tolerances | Total transaction count for last 30 days = X ± 0.1% |
| Edge cases | Known boundary conditions: zero-value transactions, max-length strings, end-of-month dates | |
| Null handling | Verify nullable columns behave correctly in downstream reports | Null `counterparty_id` must not break report rendering |
| Time boundary correctness | Verify partitioning and time filters return correct windows | Daily close figures for 31 Dec correct |
| Downstream report validation | Key BI reports produce correct outputs against UAT data | |
| Access control validation | Users see only the data their Unity Catalog grants permit | `payments_analyst` group cannot see Restricted columns |
| Business rule validation | Derived columns and calculated fields match business definitions | `net_amount = gross_amount - fee_amount` |

### Test Case Format (Jira sub-task)

Each test case sub-task contains:

- **Given** — pre-conditions and test data setup
- **When** — action the tester performs
- **Then** — expected result
- **Acceptance threshold** — numeric tolerance if applicable
- **Test data reference** — which UAT data partition/date range to use

---

## 5.5 UAT Execution

### Participants

| Role | UAT Responsibility |
|------|-------------------|
| Business User | Execute business-facing test cases; validate report outputs |
| Data Analyst | Execute technical and analytical test cases; validate aggregates and edge cases |
| Data Steward | Observe and assist; answer questions on data definitions |
| Data Engineer | Resolve defects; provide technical support |

### Execution Process

1. UAT participants execute test cases in the UAT Databricks workspace.
2. Each test case result (Pass / Fail / Blocked) is recorded as a comment on the Jira sub-task.
3. Screenshots or query results are attached as evidence.
4. Defects are raised as child Jira tickets under the UAT epic.

### Defect Triage

| Severity | Definition | Impact on G4 |
|----------|-----------|-------------|
| Critical | Data loss, incorrect financial figures, security policy breach | **Blocks G4** — must be resolved and re-tested |
| High | Significant business logic error, major report inaccuracy | **Blocks G4** — must be resolved and re-tested |
| Medium | Minor business logic deviation, cosmetic report issue | May proceed with documented **Business Owner risk acceptance** |
| Low | Cosmetic, non-data issue | Logged; does not block G4 |

---

## 5.6 UAT Sign-Off

UAT sign-off is provided by the **Business Owner** in Jira:

1. All Critical and High defects are resolved and closed.
2. Medium/Low defects are either resolved or have documented risk acceptance.
3. Business Owner adds a formal approval comment to the UAT Jira epic.
4. The UAT Jira epic is moved to **Done**.

UAT sign-off is a **prerequisite for opening G4** alongside PT sign-off (see [06-performance-testing.md](06-performance-testing.md)).
Both must be complete before G4 opens.
