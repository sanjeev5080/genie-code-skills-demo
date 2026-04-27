---
name: model-evaluation
description: Apply Nokia standard model evaluation, MLflow model registry, and Model Serving endpoint patterns. Use after training an ML model. Covers evaluation metrics by model type, Unity Catalog model registry registration, serving endpoint naming, and inference table standards.
---

# Model Evaluation and Deployment Standards

Apply these standards after training an ML model to evaluate, register, and deploy it.

## Evaluation Metrics by Model Type

### Classification (Churn, Propensity)

Always log these metrics for classification models:

```python
from sklearn.metrics import (
    roc_auc_score, accuracy_score, precision_score,
    recall_score, f1_score, log_loss, confusion_matrix
)
import matplotlib.pyplot as plt
from sklearn.metrics import RocCurveDisplay, ConfusionMatrixDisplay

y_pred = model.predict(X_test)
y_prob = model.predict_proba(X_test)[:, 1]

mlflow.log_metrics({
    "auc_roc":   round(roc_auc_score(y_test, y_prob), 4),
    "accuracy":  round(accuracy_score(y_test, y_pred), 4),
    "precision": round(precision_score(y_test, y_pred), 4),
    "recall":    round(recall_score(y_test, y_pred), 4),
    "f1_score":  round(f1_score(y_test, y_pred), 4),
    "log_loss":  round(log_loss(y_test, y_prob), 4)
})

# ROC curve
fig_roc, ax = plt.subplots()
RocCurveDisplay.from_predictions(y_test, y_prob, ax=ax)
ax.set_title("ROC Curve")
mlflow.log_figure(fig_roc, "plots/roc_curve.png")

# Confusion matrix
fig_cm, ax = plt.subplots()
ConfusionMatrixDisplay.from_predictions(y_test, y_pred, ax=ax)
mlflow.log_figure(fig_cm, "plots/confusion_matrix.png")
```

### Regression

```python
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
import numpy as np

y_pred = model.predict(X_test)

mlflow.log_metrics({
    "rmse": round(np.sqrt(mean_squared_error(y_test, y_pred)), 4),
    "mae":  round(mean_absolute_error(y_test, y_pred), 4),
    "r2":   round(r2_score(y_test, y_pred), 4)
})
```

## Feature Importance Plot

Always log a feature importance plot:

```python
import pandas as pd

# Works for XGBoost, RandomForest, LightGBM
importance_df = pd.DataFrame({
    "feature": X_train.columns,
    "importance": model.feature_importances_
}).sort_values("importance", ascending=True)

fig_imp, ax = plt.subplots(figsize=(8, max(4, len(importance_df) * 0.3)))
ax.barh(importance_df["feature"], importance_df["importance"])
ax.set_title("Feature Importance")
plt.tight_layout()
mlflow.log_figure(fig_imp, "plots/feature_importance.png")
```

## Unity Catalog Model Registry

### Registration

Register models to Unity Catalog — never use the legacy Workspace registry:

```python
import mlflow

# Set UC as registry
mlflow.set_registry_uri("databricks-uc")

model_name = "nokia_catalog.workshop.nokia_{use_case}_{model_type}"
# Example: nokia_catalog.workshop.nokia_churn_propensity_xgboost

mlflow.sklearn.log_model(
    sk_model=model,
    artifact_path="model",
    registered_model_name=model_name,
    input_example=X_test.head(5),
    signature=mlflow.models.infer_signature(X_train, y_pred)
)
```

### Naming Convention

```
nokia_catalog.workshop.nokia_{use_case}_{model_type}
```

| Component | Rule | Example |
|-----------|------|---------|
| Catalog | `nokia_catalog` | `nokia_catalog` |
| Schema | `workshop` | `workshop` |
| Model name | `nokia_{use_case}_{model_type}` | `nokia_churn_propensity_xgboost` |

### Model Aliases (Champion/Challenger)

Use aliases to manage production vs. experimental versions:

```python
from mlflow import MlflowClient

client = MlflowClient()

# Set latest version as challenger
client.set_registered_model_alias(
    name=model_name,
    alias="challenger",
    version=latest_version
)

# Promote to champion after validation
client.set_registered_model_alias(
    name=model_name,
    alias="champion",
    version=latest_version
)
```

Always tag the registered model:
```python
client.set_model_version_tag(model_name, latest_version, "team", "nokia-data-engineering")
client.set_model_version_tag(model_name, latest_version, "use_case", "{use_case}")
client.set_model_version_tag(model_name, latest_version, "validated", "false")
```

## Model Serving Endpoint

### Naming Convention

```
nokia-{use_case}-{model_type}-endpoint
```

Example: `nokia-churn-propensity-xgboost-endpoint`

### Create Endpoint

```python
import requests

WORKSPACE_URL = spark.conf.get("spark.databricks.workspaceUrl")
TOKEN = dbutils.notebook.entry_point.getDbutils().notebook().getContext().apiToken().get()

endpoint_config = {
    "name": "nokia-churn-propensity-xgboost-endpoint",
    "config": {
        "served_models": [{
            "model_name": "nokia_catalog.workshop.nokia_churn_propensity_xgboost",
            "model_version": latest_version,
            "workload_size": "Small",
            "scale_to_zero_enabled": True
        }]
    },
    "tags": [
        {"key": "team", "value": "nokia-data-engineering"},
        {"key": "use_case", "value": "churn_propensity"}
    ]
}

response = requests.post(
    f"https://{WORKSPACE_URL}/api/2.0/serving-endpoints",
    headers={"Authorization": f"Bearer {TOKEN}"},
    json=endpoint_config
)
print(response.json())
```

### Inference Table

Always enable the inference table when creating an endpoint:

```python
endpoint_config["config"]["auto_capture_config"] = {
    "catalog_name": "nokia_catalog",
    "schema_name": "workshop",
    "table_name_prefix": "nokia_churn_propensity_inference"
}
```

This creates: `nokia_catalog.workshop.nokia_churn_propensity_inference_payload`

### Test the Endpoint

```python
import json

test_payload = {
    "inputs": X_test.head(3).to_dict(orient="records")
}

response = requests.post(
    f"https://{WORKSPACE_URL}/serving-endpoints/nokia-churn-propensity-xgboost-endpoint/invocations",
    headers={"Authorization": f"Bearer {TOKEN}", "Content-Type": "application/json"},
    json=test_payload
)
print(json.dumps(response.json(), indent=2))
```

## Evaluation Checklist

Before completing model evaluation:

- [ ] AUC-ROC / RMSE logged to MLflow run
- [ ] Confusion matrix or residual plot saved as artifact
- [ ] Feature importance plot saved as artifact
- [ ] Model registered to Unity Catalog with correct naming
- [ ] Model alias set (`challenger` or `champion`)
- [ ] Model version tags set (`team`, `use_case`, `validated`)
- [ ] Serving endpoint created with Nokia naming convention
- [ ] Inference table enabled on endpoint
- [ ] Endpoint tested with sample payload
