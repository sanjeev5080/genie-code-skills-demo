---
name: dashboard-layout
description: Apply Nokia standard Lakeview dashboard layout, filter placement, section structure, and standard slices for exec dashboards. Use when arranging widgets, adding filters, or structuring sections in a dashboard.
---

# Dashboard Layout Standards

Apply these standards when arranging widgets, filters, and sections in Nokia exec dashboards.

## Standard Page Layout

Every Nokia exec dashboard MUST follow this top-to-bottom layout order:

```
┌─────────────────────────────────────────────────┐
│  FILTER BAR (full width)                        │
├──────────┬──────────┬──────────┬────────────────┤
│ kpi_     │ kpi_     │ kpi_     │ kpi_           │
│ total_   │ churn_   │ revenue_ │ high_risk_     │
│ customers│ rate     │ at_risk  │ count          │
├──────────┴──────────┴──────────┴────────────────┤
│  SECTION HEADER: Churn Trends                   │
├─────────────────────────┬───────────────────────┤
│  trend_monthly_churn    │  bar_churn_by_region  │
│  (full left half)       │  (full right half)    │
├─────────────────────────┴───────────────────────┤
│  SECTION HEADER: Customer Segments              │
├─────────────────────────┬───────────────────────┤
│  bar_churn_by_segment   │  pie_churn_by_product │
├─────────────────────────┴───────────────────────┤
│  SECTION HEADER: At-Risk Detail                 │
├─────────────────────────────────────────────────┤
│  tbl_top_at_risk_customers (full width)         │
└─────────────────────────────────────────────────┘
```

## Filter Bar

The filter bar MUST be the first row on every dashboard and MUST include these three filters as a minimum:

| Filter | Widget Name | Type |
|--------|-------------|------|
| Time Period | `filter_time_period` | Date range picker |
| Region | `filter_region` | Multi-select dropdown |
| Customer Segment | `filter_segment` | Multi-select dropdown |

Optional additional filters (add if relevant):
- `filter_product_type` — Product type multi-select
- `filter_income_tier` — Income tier multi-select
- `filter_risk_threshold` — Churn score threshold slider

All filters must be connected to all datasets on the page.

## Section Headers

Use section headers to separate logical groups of visualizations. Required sections:

| Section | Content |
|---------|---------|
| **Churn Trends** | Time series and regional breakdown |
| **Customer Segments** | Segment and product breakdowns |
| **At-Risk Detail** | High-risk customer table and drill-downs |

## Standard Slices

Every Nokia exec dashboard MUST include these standard cross-cuts (one chart each):

| Slice | Chart Type | Widget Name |
|-------|------------|-------------|
| By Region | Horizontal bar | `bar_churn_by_region` |
| By Product Type | Horizontal bar or pie | `bar_churn_by_product` |
| By Income Tier | Horizontal bar | `bar_churn_by_income_tier` |
| Over Time | Line chart | `trend_monthly_churn` |

These slices must always be present — they are the Nokia standard "exec view" of any churn or retention metric.

## At-Risk Customer Table

Every exec dashboard MUST include a detail table as the last section:

```sql
-- Dataset: ds_top_at_risk_customers
-- Description: Top 50 highest-risk customers ranked by churn score × account balance
SELECT
  customer_id,
  region,
  income_tier,
  credit_tier,
  ROUND(account_balance, 2)   AS account_balance,
  ROUND(churn_score * 100, 1) AS churn_score_pct,
  ROUND(account_balance * churn_score, 2) AS revenue_at_risk
FROM {catalog}.{schema}.gold_account_summary
WHERE churn_score > 0.5
ORDER BY revenue_at_risk DESC
LIMIT 50
```

Table columns to display (in order): `customer_id`, `region`, `income_tier`, `account_balance`, `churn_score_pct`, `revenue_at_risk`

## Color Standards

| Metric direction | Color |
|-----------------|-------|
| Churn increasing (bad) | Red `#E53935` |
| Churn decreasing (good) | Green `#43A047` |
| Neutral / informational | Blue `#1E88E5` |
| Secondary / supporting | Grey `#757575` |

Apply consistent colors across all charts showing the same metric — never use random palette colors.

## Widget Sizing Guidelines

| Widget type | Recommended width |
|-------------|-------------------|
| KPI counter | 1/4 page (4 per row) |
| Trend / line chart | 1/2 page |
| Bar chart | 1/2 page |
| Full-width table | Full page |
| Pie / donut | 1/2 page |

## Layout Checklist

- [ ] Filter bar is the first row with at least 3 standard filters
- [ ] All filters connected to all datasets
- [ ] KPI row (4 tiles) immediately below filter bar
- [ ] Section headers present between logical groups
- [ ] All 4 standard slices present (region, product, income tier, time trend)
- [ ] At-risk detail table as last section
- [ ] Consistent colors applied across charts
