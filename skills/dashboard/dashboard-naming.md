---
name: dashboard-naming
description: Apply Nokia standard naming conventions to Lakeview dashboards, datasets, widgets, and filters. Use when creating or modifying any dashboard artifact. This is the baseline dashboard skill — apply it first before any other dashboard skill.
---

# Dashboard Naming Standards

Apply these naming rules to every dashboard artifact. Consistent naming enables discoverability, governance, and reuse across Nokia teams.

## Dashboard Name

Pattern: `Nokia_{Use_Case}_{Audience}`

| Component | Rule | Example |
|-----------|------|---------|
| Prefix | Always `Nokia_` | `Nokia_` |
| Use case | Title_Case, underscore-separated | `Churn_Analysis`, `Revenue_Performance` |
| Audience | Who it's for | `Exec`, `Ops`, `Finance` |

Examples:
- `Nokia_Churn_Analysis_Exec`
- `Nokia_Revenue_Performance_Exec`
- `Nokia_Customer_Retention_Exec`

Never use generic names like `New Dashboard`, `Untitled`, `Dashboard 1`.

## Dataset Naming

Pattern: `ds_{entity}_{grain}`

| Component | Rule | Example |
|-----------|------|---------|
| Prefix | Always `ds_` | `ds_` |
| Entity | What it describes | `customers`, `accounts`, `churn` |
| Grain | Time or aggregation level | `monthly`, `summary`, `by_region` |

Examples:
- `ds_churn_summary`
- `ds_customers_by_region`
- `ds_revenue_monthly`
- `ds_high_risk_customers`

Every dataset MUST have a description comment in the SQL:

```sql
-- Dataset: ds_churn_summary
-- Description: Monthly churn rate and at-risk customer count for exec dashboard
-- Source: {catalog}.{schema}.gold_account_summary
-- Grain: One row per month
```

## Widget Naming

Pattern: `{viz_type}_{metric}_{dimension}`

| Visualization | Prefix |
|---------------|--------|
| Counter / KPI tile | `kpi_` |
| Bar chart | `bar_` |
| Line / trend chart | `trend_` |
| Pie / donut chart | `pie_` |
| Table | `tbl_` |
| Scatter plot | `scatter_` |

Examples:
- `kpi_churn_rate`
- `kpi_revenue_at_risk`
- `bar_churn_by_region`
- `trend_monthly_churn`
- `tbl_top_at_risk_customers`

Never leave widgets named `Chart`, `Counter`, `Table`, or `Visualization 1`.

## Filter Naming

Pattern: `filter_{dimension}`

Examples:
- `filter_time_period`
- `filter_region`
- `filter_segment`
- `filter_product_type`

## Summary Checklist

Before completing any dashboard:

- [ ] Dashboard name follows `Nokia_{Use_Case}_{Audience}` pattern
- [ ] All datasets prefixed with `ds_` and have SQL description comments
- [ ] All widgets named with `{viz_type}_{metric}_{dimension}` pattern
- [ ] All filters named with `filter_{dimension}` pattern
- [ ] No generic placeholder names anywhere
