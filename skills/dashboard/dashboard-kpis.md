---
name: dashboard-kpis
description: Apply Nokia standard executive KPI definitions, counter widget patterns, and metric SQL for Lakeview dashboards. Use when adding KPI tiles or defining metrics for an exec-facing dashboard. Covers standard Nokia KPIs, SQL patterns, thresholds, and period-over-period comparison.
---

# Executive KPI Standards

Apply these standards when defining KPI tiles and metric datasets for exec-facing Nokia dashboards.

## Nokia Standard Executive KPIs

Every Nokia exec dashboard MUST include these four KPIs as the top row:

| KPI Name | Widget Name | Description |
|----------|-------------|-------------|
| Total Active Customers | `kpi_total_customers` | Count of active accounts in the period |
| Churn Rate % | `kpi_churn_rate` | % of customers who churned in the period |
| Revenue at Risk | `kpi_revenue_at_risk` | Total account balance held by high-risk customers |
| High-Risk Customer Count | `kpi_high_risk_count` | Customers with churn probability > 0.7 |

## KPI SQL Patterns

### Total Active Customers

```sql
-- Dataset: ds_kpi_total_customers
-- Description: Count of active customer accounts
SELECT
  COUNT(DISTINCT customer_id) AS total_customers
FROM {catalog}.{schema}.gold_account_summary
WHERE account_status = 'active'
```

### Churn Rate %

```sql
-- Dataset: ds_kpi_churn_rate
-- Description: Percentage of customers churned in current period
SELECT
  ROUND(
    100.0 * SUM(CASE WHEN churned = 1 THEN 1 ELSE 0 END)
    / NULLIF(COUNT(DISTINCT customer_id), 0),
    2
  ) AS churn_rate_pct
FROM {catalog}.{schema}.gold_account_summary
```

### Revenue at Risk

```sql
-- Dataset: ds_kpi_revenue_at_risk
-- Description: Total account balance held by customers with churn_score > 0.7
SELECT
  ROUND(SUM(account_balance), 2) AS revenue_at_risk
FROM {catalog}.{schema}.gold_account_summary
WHERE churn_score > 0.7
```

> **Option A (with ML predictions from Session 2):** Join to `{catalog}.{schema}.churn_predictions` on `customer_id` and use `churn_score` from the prediction table.
>
> **Option B (standalone, gold table only):** Use `churn_risk_flag` or derived `income_tier` as a proxy for risk if no prediction score is available.

### High-Risk Customer Count

```sql
-- Dataset: ds_kpi_high_risk_count
-- Description: Number of customers with churn score above threshold
SELECT
  COUNT(DISTINCT customer_id) AS high_risk_count
FROM {catalog}.{schema}.gold_account_summary
WHERE churn_score > 0.7
```

## Period-over-Period Comparison

Every KPI tile MUST show a period-over-period delta. Use this SQL pattern:

```sql
-- Dataset: ds_kpi_churn_rate_with_delta
-- Description: Current vs prior period churn rate for KPI comparison
WITH current_period AS (
  SELECT
    ROUND(100.0 * SUM(CASE WHEN churned = 1 THEN 1 ELSE 0 END)
      / NULLIF(COUNT(*), 0), 2) AS churn_rate_pct
  FROM {catalog}.{schema}.gold_account_summary
  WHERE period = DATE_TRUNC('month', CURRENT_DATE())
),
prior_period AS (
  SELECT
    ROUND(100.0 * SUM(CASE WHEN churned = 1 THEN 1 ELSE 0 END)
      / NULLIF(COUNT(*), 0), 2) AS churn_rate_pct
  FROM {catalog}.{schema}.gold_account_summary
  WHERE period = DATE_TRUNC('month', CURRENT_DATE() - INTERVAL 1 MONTH)
)
SELECT
  c.churn_rate_pct AS current_value,
  p.churn_rate_pct AS prior_value,
  ROUND(c.churn_rate_pct - p.churn_rate_pct, 2) AS delta
FROM current_period c, prior_period p
```

## KPI Counter Widget Configuration

When creating a KPI counter widget, always set:

| Property | Value |
|----------|-------|
| Title | Human-readable metric name (e.g. `Churn Rate %`) |
| Value field | The primary metric column |
| Comparison field | The `delta` column (period-over-period) |
| Color — positive delta (bad for churn) | Red |
| Color — negative delta (good for churn) | Green |
| Number format | `%` for rates, `$` or `€` for monetary values |

## SQL Safety Rules

- Always use `NULLIF(COUNT(*), 0)` in denominators to prevent divide-by-zero
- Always use `ROUND(..., 2)` for rates and monetary values
- Always filter to `account_status = 'active'` unless explicitly showing churned customers
- Always alias computed columns with descriptive names, never `col1` or unnamed expressions

## KPI Checklist

- [ ] All 4 standard Nokia KPIs present in top row
- [ ] Every KPI has a period-over-period delta
- [ ] SQL uses `NULLIF` in all denominators
- [ ] SQL uses `ROUND()` for all rates and monetary values
- [ ] Counter widgets have color thresholds set
- [ ] Dataset SQL includes a description comment
