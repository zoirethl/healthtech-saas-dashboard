# HealthTech SaaS Operations Dashboard
### Looker Studio · Google Sheets · Python · BigQuery-ready

**[→ View Live Dashboard](https://lookerstudio.google.com/reporting/40c8c19d-504c-4348-865a-825ffa1f0a1a)**

---

## Overview

A four-page operational and executive dashboard built for a HealthTech SaaS company, analyzing team performance, implementation efficiency, customer profitability, and revenue health across 80+ client accounts (2024–2025).

The dataset was reconstructed from real operational data using simulated values to preserve confidentiality. All metrics, relationships, and business logic reflect actual SaaS operations.

---

## Business Context

The company provides a SaaS platform to healthcare organizations (clinics, hospitals, university medical centers, physician groups) across the US. Their Customer Success team handles both implementation (onboarding) and ongoing post-launch support for each account.

**The core operational challenge:** implementation is billed as a one-time cost, while post-launch support is covered by the monthly subscription. Without visibility into whether subscriptions actually cover support costs, the business has no way to identify unprofitable accounts or optimize pricing.

This dashboard was built to answer:
- Are we implementing accounts on time and under budget?
- Are our subscription prices covering the cost of post-launch support?
- Which accounts are profitable long-term, and which are eroding our margin?
- Is the team improving their efficiency over time?
- What does our revenue health look like — churn, expansion, NRR?

---

## Dashboard Structure

### Page 1 — Implementation Cost Analysis
Tracks estimated vs actual implementation costs per account, cost efficiency percentage, and budget status across all client accounts.

**Key metrics:**
- Estimated vs Actual Implementation Cost (side-by-side bar chart)
- Cost Efficiency % — colored by budget status (on budget / over budget / in progress)
- Cost Variance in USD
- Avg actual cost segmented by account size (Small / Medium / Large)

**Calculated fields:**
- `Cost Efficiency %` = `ROUND((Actual Cost / Estimated Cost) * 100, 1)`
- `Budget Status` = CASE logic classifying accounts as On Budget, Over Budget, or In Progress
- `Cost Variance` = `Estimated Cost - Actual Cost`

---

### Page 2 — Implementation Time Benchmarks
Separates implementation-phase hours from post-launch support hours — a critical distinction since these map to different revenue streams.

**Key metrics:**
- Avg implementation hours by account size vs estimated benchmark
- Phase breakdown per account (Implementation % vs Supporting %)
- In-progress accounts table with % Progress and conditional formatting
- Support ROI % — whether subscriptions cover the cost of supporting each account
- Support Intensity — avg support hours per month per account
- Net Customer Value — total subscription revenue minus total cost to serve

**Calculated fields:**
- `Support ROI %` = `Total Subscription Revenue / Support Cost`
- `% Progress` = `ROUND(SUM of Time / Estimated Implementation, 3)` (formatted as %)
- `Support Start Date` derived from RawData using `MINIFS` on earliest Phase 3 task date
- `Months in Support` = `DATEDIF(Support Start Date, TODAY(), "M")`
- `Total Subscription Revenue` = `Subscription × Months in Support`

---

### Page 3 — Executive / SaaS Metrics
Executive-level revenue health dashboard covering ARR, NRR, LTV, CAC, and upsell performance.

**Key metrics:**
- ARR Actual vs ARR Adjusted (with expansion MRR)
- Monthly Churn Rate vs 2% SaaS benchmark
- LTV and Avg CAC by account size
- NRR trend — monthly line chart with 1.0 reference line
- Expansion MRR vs Churned MRR by month (stacked bar)
- Upsell revenue by month, broken down by upsell type
- Top 5 and Bottom 5 accounts by Net Customer Value

**Data engineering:**
- Churn modeled with two dates: `Churn Confirmed Date` (when churn was known) and `Churn Effective Date` (when contract ends) — reflecting real committed churn dynamics
- Upsells sheet with 24 expansion events across 2025 (Seat Expansion, Module Add-on, Tier Upgrade)
- NRR calculated monthly using SUMPRODUCT formulas across three sheets (Accounts, Upsells, RawData)
- Monthly Churn Rate = `Churned MRR / ARR Actual`

**NRR formula structure (per month):**
- Starting MRR: active accounts at start of month × subscription
- Churned MRR: accounts whose Churn Effective Date falls in that month
- Expansion MRR: sum of (New MRR - Previous MRR) for upsells in that month
- NRR % = `(Starting MRR - Churned MRR + Expansion MRR) / Starting MRR`

---

### Page 4 — Team Performance
Individual and team-level analysis of the Customer Success team across Implementation Engineers and Customer Success Managers.

**Key metrics:**
- Total hours logged per team member (Client vs Internal split)
- Top tasks by hours per person — identifies bottlenecks and high-effort task types
- Individual trend over time — monthly avg hours per task to detect efficiency improvement
- Team comparison — each person's avg vs team average by role
- Hours by phase (Phase 1 / 2 / 3) — workload distribution across the pipeline
- Internal vs client hours per person — flags time allocation inefficiencies

**Data source:** RawData with `Is Internal Task` field classifying Internal Meetings vs client-billable work.

---

## Data Sources & Architecture

```
RawData (task-level logs)
    │
    ├── JOIN Member ──► CX Team (salaries, hourly rates)
    │
    └── JOIN Project ──► Accounts (account-level summaries)
                              │
                              └── Upsells (expansion events)

SaaS Metrics (sheet)    ──► ARR, NRR, LTV, Churn Rate
NRR SaaS Metrics (sheet) ──► Monthly NRR table (12 rows × 5 columns)
```

**Blends used in Looker Studio:**
- `RawData + CX Team` — join on Member, enables Task Cost calculation
- `RawData + CX Team + Accounts` — three-way blend, enables Support ROI at task level

---

## Technical Stack

| Tool | Use |
|---|---|
| Looker Studio | Dashboard, visualizations, calculated fields |
| Google Sheets | Data source, calculated columns, NRR table |
| SUMPRODUCT | Monthly NRR cohort calculations |
| MINIFS | Deriving Support Start Date from raw task logs |
| DATEDIF | Months in Support per account |
| IFERROR + DATEVALUE | Handling sparse date fields safely |

---

## Key Business Insights Surfaced

- **13 accounts identified as committed churn** — all with Support ROI < 1.0 and negative Net Customer Value. Price sensitivity was the primary churn reason (62% of churned accounts).
- **Small accounts are systematically unprofitable post-launch** — avg Support ROI of 0.6-0.8 vs 1.4-2.0 for Large accounts. Subscription pricing for Small accounts does not cover support costs.
- **NRR stabilized above 100% from April 2025 onward** — driven by upsell program targeting the 10 highest-ROI accounts.
- **Implementation efficiency varies significantly by account size** — Large accounts take 2.1x longer than estimated on average, while Small accounts come in under estimate.

---

## Dataset

The dataset covers:
- **120 client accounts** across 5 US regions
- **9 team members** (2 Customer Success Managers, 2 Support Specialist, 5 Implementation Engineers)
- **2 years of task-level time logs** (2024–2025)
- **24 upsell events** across 2025
- **13 churn events** with confirmed and effective dates

All account names are anonymized using City + Number + Facility Type format. Financial figures are simulated but calibrated to realistic HealthTech SaaS benchmarks.

---

## Author

**Zoireth Liendo** · Data & Revenue Operations Analyst  
[Portfolio](https://zoirethl.github.io) · [GitHub](https://github.com/zoirethl)
