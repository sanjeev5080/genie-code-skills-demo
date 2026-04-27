---
name: mlflow-experiment
description: Apply Nokia standard MLflow experiment and run structure for ML projects. Use when creating or scaffolding an ML experiment notebook. Covers experiment naming, run logging, required params/metrics/tags, and artifact structure.
---

# MLflow Experiment Standards

Always structure MLflow experiments and runs following these rules when scaffolding or modifying an ML experiment notebook.

## Experiment Naming

All experiments MUST follow this naming convention:

```
/Users/{username}/nokia_{use_case}_{model_type}
```

| Component | Rule | Example |
|-----------|------|---------|
| Prefix | Always `nokia_` | `nokia_` |
| Use case | snake_case description | `churn_propensity` |
| Model type | algorithm family | `xgboost`, `logistic_regression`, `random_forest` |

Example: `/Users/sanjeev.kumar/nokia_churn_propensity_xgboost`

Never use generic names like `my_experiment`, `test`, or `untitled`.

## Run Naming

Each run MUST be named descriptively:

```python
with mlflow.start_run(run_name=f"{model_type}_v{version}_{date}"):
```

Example: `xgboost_v1_20260427`

## Required Parameters to Log

Every run MUST log these parameters:

```python
mlflow.log_params({
    "model_type": "<algorithm>",
    "feature_version": "<feature_set_version>",
    "data_version": "<gold_table_version_or_date>",
    "train_size": <int>,
    "test_size": <int>,
    "random_seed": 42,
    "target_column": "<target>",
    "gold_table": "<catalog.schema.table_name>"
})
```

## Required Tags

Every run MUST have these tags:

```python
mlflow.set_tags({
    "team": "nokia-data-engineering",
    "use_case": "<churn_propensity|revenue_forecast|...>",
    "data_source": "gold_layer",
    "environment": "dev"
})
```

## Artifact Structure

Every run MUST log artifacts in this structure:

```
artifacts/
  model/              ← MLflow autolog or manual log_model
  plots/
    feature_importance.png
    confusion_matrix.png   (classification only)
    roc_curve.png          (classification only)
    residuals.png          (regression only)
  data/
    feature_schema.json    ← column names, types, transformations applied
    train_test_split.json  ← split dates or indices
```

Log plots using:
```python
mlflow.log_figure(fig, "plots/feature_importance.png")
```

Log schemas using:
```python
mlflow.log_dict(feature_schema, "data/feature_schema.json")
```

## Autologging

Always enable framework autologging at the top of the training cell:

```python
mlflow.sklearn.autolog()    # for scikit-learn
mlflow.xgboost.autolog()    # for XGBoost
mlflow.lightgbm.autolog()   # for LightGBM
```

## Notebook Structure

Every ML experiment notebook MUST follow this cell order:

1. **Setup** — imports, MLflow setup, widget config
2. **Data Loading** — read from gold tables, display schema
3. **Feature Engineering** — apply `@feature-engineering` skill
4. **Model Training** — inside `mlflow.start_run()` block
5. **Evaluation** — apply `@model-evaluation` skill
6. **Registration** — register to Unity Catalog model registry

## Example: Experiment Setup

```python
import mlflow
import mlflow.sklearn
from datetime import date

# Set experiment
mlflow.set_experiment("/Users/{username}/nokia_churn_propensity_xgboost")

# Enable autologging
mlflow.xgboost.autolog()

# Start run
with mlflow.start_run(run_name=f"xgboost_v1_{date.today().strftime('%Y%m%d')}"):

    mlflow.log_params({
        "model_type": "xgboost",
        "feature_version": "v1",
        "data_version": "2026-04-27",
        "train_size": X_train.shape[0],
        "test_size": X_test.shape[0],
        "random_seed": 42,
        "target_column": "churned",
        "gold_table": "nokia_catalog.workshop.gold_customer_churn"
    })

    mlflow.set_tags({
        "team": "nokia-data-engineering",
        "use_case": "churn_propensity",
        "data_source": "gold_layer",
        "environment": "dev"
    })

    # ... training code ...

    mlflow.log_figure(fig_importance, "plots/feature_importance.png")
    mlflow.log_dict(feature_schema, "data/feature_schema.json")
```
