---
name: feature-engineering
description: Apply Nokia standard feature engineering patterns for ML experiments built on gold layer tables. Use when scaffolding feature preparation code in an ML notebook. Covers feature naming, derivation from gold tables, train/test split, encoding, and scaling standards.
---

# Feature Engineering Standards

Apply these standards when building feature sets from gold layer tables for ML experiments.

## Reading from Gold Tables

Always read features from gold layer tables using the catalog fully qualified name:

```python
gold_df = spark.table("nokia_catalog.workshop.gold_customer_churn").toPandas()
```

Never read from bronze or silver directly for ML experiments. Gold tables are the single source of truth for ML features.

## Feature Naming

All feature column names MUST use `lowercase_snake_case`:

| Rule | Good | Bad |
|------|------|-----|
| Snake case | `account_balance` | `AccountBalance`, `accountBalance` |
| Descriptive | `avg_monthly_transactions` | `amt`, `x1` |
| Layer prefix for derived | `feat_churn_risk_score` | `churn` |

Engineered/derived features MUST be prefixed with `feat_`.

## Standard Feature Categories

Organize features into these categories in the notebook:

```python
# --- Numerical features ---
numerical_features = [
    "account_balance",
    "avg_monthly_transactions",
    "credit_score",
    "account_age_days"
]

# --- Categorical features ---
categorical_features = [
    "region",
    "income_tier",
    "credit_tier",
    "product_type",
    "account_type"
]

# --- Derived / engineered features ---
derived_features = [
    "feat_balance_to_income_ratio",
    "feat_transaction_trend",
    "feat_days_since_last_activity"
]
```

## Standard Derived Features for Churn/Propensity

When building churn or propensity models from the financial gold tables, always derive:

```python
# Balance to income ratio
df["feat_balance_to_income_ratio"] = (
    df["account_balance"] / df["avg_monthly_transactions"].replace(0, 1)
)

# Transaction trend (positive = growing, negative = declining)
df["feat_transaction_trend"] = (
    df["recent_month_transactions"] - df["prior_month_transactions"]
)

# Account tenure in years
df["feat_account_tenure_years"] = df["account_age_days"] / 365.0

# High value flag
df["feat_is_high_value"] = (df["account_balance"] > df["account_balance"].quantile(0.75)).astype(int)
```

## Train/Test Split Standards

Always use a **fixed random seed of 42** and an **80/20 split** unless business requirements specify otherwise:

```python
from sklearn.model_selection import train_test_split

X = df[numerical_features + categorical_features + derived_features]
y = df["churned"]  # target column

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    stratify=y  # always stratify for classification
)
```

Log the split to MLflow:
```python
mlflow.log_params({
    "train_size": X_train.shape[0],
    "test_size": X_test.shape[0],
    "test_ratio": 0.2,
    "stratified": True
})
```

## Encoding Standards

Use `OrdinalEncoder` for tree-based models, `OneHotEncoder` for linear models:

```python
from sklearn.preprocessing import OrdinalEncoder, OneHotEncoder
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer

# For tree-based models (XGBoost, Random Forest)
preprocessor = ColumnTransformer(transformers=[
    ("num", "passthrough", numerical_features + derived_features),
    ("cat", OrdinalEncoder(handle_unknown="use_encoded_value", unknown_value=-1), categorical_features)
])
```

## Scaling Standards

Apply `StandardScaler` to numerical features for linear models only. Never scale for tree-based models:

```python
from sklearn.preprocessing import StandardScaler

# Linear models only
preprocessor = ColumnTransformer(transformers=[
    ("num", StandardScaler(), numerical_features + derived_features),
    ("cat", OneHotEncoder(handle_unknown="ignore", sparse_output=False), categorical_features)
])
```

## Missing Value Handling

```python
# Numerical: fill with median
df[numerical_features] = df[numerical_features].fillna(df[numerical_features].median())

# Categorical: fill with "UNKNOWN"
df[categorical_features] = df[categorical_features].fillna("UNKNOWN")
```

Log missing value counts before imputation:
```python
missing_counts = df[numerical_features + categorical_features].isnull().sum().to_dict()
mlflow.log_dict(missing_counts, "data/missing_values_pre_imputation.json")
```

## Feature Schema Logging

Always log the feature schema as an artifact:

```python
feature_schema = {
    "numerical": numerical_features,
    "categorical": categorical_features,
    "derived": derived_features,
    "target": "churned",
    "total_features": len(numerical_features + categorical_features + derived_features)
}
mlflow.log_dict(feature_schema, "data/feature_schema.json")
```
