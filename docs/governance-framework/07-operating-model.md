# Pillar 7: Governance Operating Model

---

## 7.1 Roles & Responsibilities

### RACI Summary

| Activity | Data Owner | Data Steward | Data Analyst | Data Engineer | Platform Lead | DG Officer |
|----------|-----------|-------------|-------------|--------------|--------------|------------|
| DQ rule authoring (Informatica) | I | **R/A** | C | I | I | I |
| DQ rule business logic validation | A | C | **R** | I | I | I |
| Data profiling sign-off | I | **R/A** | I | I | I | I |
| G0 gate closure | I | I | I | **R** | **A** | I |
| G1 gate closure | I | **R** | I | **R/A** | I | I |
| G2 gate closure | I | **R** | I | R | **A** | I |
| G3 gate closure | **A** | I | **R** | R | I | I |
| G4 gate closure | **A** | I | I | R | **A** | I |
| UAT test case authoring | C | **R** | **R** | I | I | I |
| UAT execution | C | C | **R** | C | I | I |
| UAT sign-off | **A** | I | I | I | I | I |
| PT execution | I | I | **R** | **R** | I | I |
| PT sign-off | I | I | I | I | **A** | I |
| Unity Catalog access policy | I | I | I | C | **R/A** | I |
| Informatica IDGC administration | I | C | I | I | I | **R/A** |
| Governance reporting | I | I | I | I | C | **R/A** |
| Hotfix dual approval | **A** | I | I | R | **A** | I |

> **R** = Responsible (does the work) | **A** = Accountable (owns the outcome) | **C** = Consulted | **I** = Informed

---

### Role Descriptions

#### Data Owner (Business)

The Data Owner is a senior business stakeholder accountable for a data domain.

**Governance responsibilities:**

- Approves G3 and G4 gate tickets.
- Provides UAT sign-off.
- Grants DQ score exception waivers (Silver and Gold layers only).
- Provides dual approval for hotfix promotions.
- Participates in the monthly Data Governance Council.

---

#### Data Steward (Business / Analytics function)

The Data Steward is responsible for the quality and fitness-for-purpose of data in their domain.

**Governance responsibilities:**

- Authors and maintains DQ rules in Informatica IDGC for their domain.
- Reviews and signs off data profiling outputs.
- Maintains the data dictionary and business glossary entries in Informatica.
- Co-authors UAT test cases with the Data Analyst.
- Maintains Informatica catalog entries (business name, description, classification, PII flags).
- Participates in the weekly Ops Standup and monthly Governance Council.

---

#### Data Analyst

The Data Analyst is the domain expert closest to the business logic nuances of the data.

**Governance responsibilities:**

- Validates that DQ rules authored by the Data Steward accurately reflect business logic.
- Provides business logic review sign-off at G3.
- Co-authors UAT test cases with the Data Steward.
- Executes UAT test cases in the UAT workspace.
- Executes query-level PT scenarios (Scenarios 2 and 5) in the PT workspace.
- Supports the Data Steward in identifying edge cases and boundary conditions.

---

#### Data Engineer

The Data Engineer builds and maintains data pipelines and is responsible for technical delivery.

**Governance responsibilities:**

- Implements pipelines following naming conventions and coding standards (see [01-data-standards.md](01-data-standards.md)).
- Closes G0, G1, and G2 gate artefacts.
- Implements and maintains recon jobs.
- Executes pipeline-level PT scenarios (Scenarios 1, 3, 4).
- Authors ops runbooks for Semantic layer datasets.
- Investigates and fixes pipeline-level causes of DQ failures.
- Responds to recon failure alerts within SLA.

---

#### Platform Lead

The Platform Lead is accountable for the technical health and governance of the data platform.

**Governance responsibilities:**

- Enforces data standards across all pipelines and schemas.
- Approves G2 and G4 gates.
- Governs Unity Catalog access policies (grants, row/column security, dynamic views).
- Reviews and signs off PT results.
- Provides dual approval for hotfix promotions.
- Participates in the weekly Ops Standup and monthly Governance Council.
- Monitors recon alerts and escalates critical failures.

---

#### Data Governance Officer (DGO)

The DGO owns the governance framework and ensures compliance across all domains.

**Governance responsibilities:**

- Owns and maintains this governance framework.
- Administers Informatica IDGC (user access, scanner configuration, rule templates).
- Tracks gate backlog, DQ health, and recon exceptions across all domains.
- Produces the monthly DQ health report and governance report.
- Facilitates the monthly Data Governance Council.
- Trains new Data Engineers and Data Stewards on the governance process.
- Manages the Jira governance workflow templates.

---

## 7.2 Governance Council Structure

### Monthly Data Governance Council

**Cadence:** Monthly (last Thursday of each month)

**Participants:** EVP Data, Domain Leads, Platform Lead, Risk representative, Data Governance Officer

**Standing agenda:**

1. DQ health report (exported from Informatica) — domain-by-domain review
2. Open recon exceptions and hotfix retrospectives
3. Gate backlog review — datasets stuck in a gate
4. Policy changes or standards updates requiring council approval
5. Phased rollout progress (see [08-rollout-plan.md](08-rollout-plan.md))
6. Escalated DQ Review Board items

**Outputs:**

- Action items logged in Jira with owners and due dates.
- Exception approvals recorded in the meeting minutes.
- Updated governance report distributed within 2 business days.

---

### Weekly Ops Standup

**Cadence:** Weekly (every Tuesday, 30 minutes)

**Participants:** Platform Lead, Data Engineers (all domains), Data Stewards (all domains), Data Governance Officer

**Standing agenda:**

1. Open recon failures — status and ETA to resolution
2. Open DQ incidents — status and ETA to resolution
3. Gate blockers — what is preventing a gate from closing
4. Upcoming releases requiring gate activity this week
5. PT / UAT in progress — blockers

---

## 7.3 Escalation Path

```
Data Engineer / Data Analyst
         ↓ (unresolved after SLA)
    Data Steward + Platform Lead
         ↓ (unresolved after 2× SLA)
    Data Owner + Data Governance Officer
         ↓ (regulatory / financial impact)
    EVP Data + Risk representative
```

---

## 7.4 Governance Review Calendar

| Activity | Frequency | Owner |
|----------|-----------|-------|
| DQ rule review per domain | Quarterly | Data Steward |
| Access policy audit (Unity Catalog) | Quarterly | Platform Lead |
| Informatica scanner configuration review | Quarterly | Data Governance Officer |
| Framework review and update | Semi-annually | Data Governance Officer |
| Data retention policy review | Annually | Data Owner + Legal |
| Governance framework training | On-boarding + annually | Data Governance Officer |
