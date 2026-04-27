# Genie Code ML Demo Script

A four-stage demo showing how Genie Code output for ML experiments improves at each step — from ungoverned notebooks to Nokia-standard ML workflows with centralized skills via MCP.

### Configure these values for your environment

| Variable | Description | Example |
|----------|-------------|---------|
| `{catalog}` | Your Unity Catalog name | `nokia_catalog` |
| `{schema}` | Your schema name | `workshop` |
| `{username}` | Your Databricks username | `sanjeev.kumar` |

> Replace `{catalog}`, `{schema}`, and `{username}` throughout this script with your actual values.

---

## Prerequisites

Before running the demo:

1. Session 1 has been completed — gold tables exist in `{catalog}.{schema}`
2. At minimum, `gold_account_summary` or equivalent gold customer table is available
3. A Python notebook exists (or create one) in your workspace to run Genie Code against
4. Genie Code is open in **Agent mode** inside the notebook

---

## Stage 1: Baseline (No Skills, No Instructions, No MCP)

**Goal:** Show that Genie Code scaffolds a functional ML notebook but without Nokia standards — random experiment names, missing logging, no deployment pattern.

### Setup

- No skills uploaded to the workspace
- No `.assistant_ml_instructions.md` file
- No MCP connections enabled in Genie Code

### Prompts

Open Genie Code in Agent mode inside a Python notebook and run these prompts:

**Prompt 1a:**

> Build me a churn prediction model using the gold customer table at `/Volumes/{catalog}/{schema}/raw_data/` — use XGBoost, track it with MLflow

**Prompt 1b:**

> Add evaluation metrics and register the model

### What to observe

Point out what's **missing** from the generated code:

- [ ] No Nokia experiment naming convention (`nokia_churn_propensity_xgboost`)
- [ ] No required params logged (`feature_version`, `data_version`, `gold_table`)
- [ ] No required tags (`team`, `use_case`, `data_source`)
- [ ] No artifact structure (`plots/`, `data/` folders)
- [ ] Missing key metrics (AUC-ROC, F1, log_loss)
- [ ] No feature importance plot
- [ ] No Unity Catalog model registry — registers to workspace registry or not at all
- [ ] No serving endpoint
- [ ] No train/test split logging
- [ ] Inconsistent feature naming

> **Talking point:** "Genie Code writes working ML code, but without standards every experiment looks different. We can't govern, compare, or reproduce runs across the team."

### Cleanup before Stage 2

Delete any MLflow experiments and registered models created in Stage 1 from the Experiments and Models UI.

---

## Stage 2: Add Skills

**Goal:** Show that uploading ML skills to the workspace immediately produces Nokia-standard experiment code.

### Setup

Upload the three skill files from `skills/ml/` to your workspace. The repo structure matches the workspace structure exactly — copy as-is:

```
skills/ml/              (repo)        →   Workspace/.assistant/skills/   (workspace)
  mlflow-experiment.md                      mlflow-experiment.md
  feature-engineering.md                    feature-engineering.md
  model-evaluation.md                       model-evaluation.md
```

Or upload to user level at `/Users/{username}/.assistant/skills/` for a personal demo.

> **Note:** The file name (e.g. `mlflow-experiment.md`) becomes the `@tag` name in Genie Code prompts — `@mlflow-experiment` tells Genie Code to load and apply that skill.

### Prompts

Start a **new** Genie Code session (to clear context from Stage 1):

**Prompt 2a:**

> Build me a churn prediction model using the gold customer table at `{catalog}.{schema}.gold_account_summary` — use XGBoost and apply @mlflow-experiment and @feature-engineering as standards

**Prompt 2b:**

> Add evaluation, register the model, and create a serving endpoint — apply @model-evaluation

### What to observe

Compare side-by-side with Stage 1 output:

- [x] Experiment named `/Users/{username}/nokia_churn_propensity_xgboost`
- [x] All required params logged (`feature_version`, `data_version`, `gold_table`, etc.)
- [x] Nokia tags set (`team`, `use_case`, `data_source`, `environment`)
- [x] Artifact structure: `plots/`, `data/` with feature schema
- [x] AUC-ROC, F1, precision, recall, log_loss all logged
- [x] Feature importance plot saved
- [x] ROC curve and confusion matrix saved
- [x] Model registered to Unity Catalog: `nokia_catalog.workshop.nokia_churn_propensity_xgboost`
- [x] Champion/challenger aliases set
- [x] Serving endpoint named `nokia-churn-propensity-xgboost-endpoint`
- [x] Inference table enabled

> **Talking point:** "Same prompt, Nokia-standard output. The skills encode our experiment template once and every run follows it."

### Cleanup before Stage 3

Delete experiments/models from Stage 2 if desired. Stage 3 prompts are the same — the difference is the skills are applied automatically.

---

## Stage 3: Add Instructions

**Goal:** Show that ML instructions make skills automatic — no `@mention` needed in prompts.

### Setup

Keep the skills from Stage 2 in place. Add a user-level ML instructions file:

1. Copy `local_deployment/instructions_to_use/user_ml_instructions_skills.md`
2. Upload as `/Users/{username}/.assistant_ml_instructions.md` in your workspace

### Prompts

Start a **new** Genie Code session:

**Prompt 3a:**

> Build me a churn prediction model using the gold customer table at `{catalog}.{schema}.gold_account_summary` — use XGBoost

**Prompt 3b:**

> Add evaluation, register the model, and create a serving endpoint

### What to observe

- [x] **No `@skill` mentions needed** — instructions route ML work to skills automatically
- [x] All the same Nokia standards applied as Stage 2
- [x] Instructions act as a routing layer: "for any ML experiment, apply these skills"

> **Talking point:** "With instructions in place, every data scientist gets Nokia-standard ML code by default. No one has to remember which skills to apply."

### Cleanup before Stage 4

Delete experiments/models from Stage 3 if desired.

---

## Stage 4: MCP (Centralized Standards from GitHub)

**Goal:** Show that ML skills can be served from a central GitHub repo via MCP — no local skill files needed in the workspace.

### Setup

1. **Delete the local skills** from the workspace:
   - Remove `Workspace/.assistant/skills/mlflow-experiment.md`
   - Remove `Workspace/.assistant/skills/feature-engineering.md`
   - Remove `Workspace/.assistant/skills/model-evaluation.md`

2. **Add the MCP connection** to Genie Code:
   - Open Genie Code settings
   - Add MCP server → select `genie-code-skills-mcp` connection
   - Enable these tools: `get_file_contents`, `search_code`

3. **Switch instructions to MCP version**:
   - Replace with `local_deployment/instructions_to_use/user_ml_instructions_mcp.md`
   - Upload as `/Users/{username}/.assistant_ml_instructions.md`

### Prompts

Start a **new** Genie Code session:

**Prompt 4a:**

> Build me a churn prediction model using the gold customer table at `{catalog}.{schema}.gold_account_summary` — use XGBoost

**Prompt 4b:**

> Add evaluation, register the model, and create a serving endpoint

**Prompt 4c (bonus):**

> Now build a second experiment using LightGBM on the same gold table and compare it against the XGBoost run

### What to observe

- [x] Genie Code **fetches skills from GitHub** via MCP (you'll see it call `get_file_contents`)
- [x] Same Nokia standards applied — identical quality to Stages 2 and 3
- [x] **No local skill files needed** in the workspace
- [x] ML standards are version-controlled in GitHub

> **Talking point:** "The Nokia ML team maintains experiment standards in a single GitHub repo. Every workspace, every data scientist, every experiment gets the same template — automatically."

---

## Summary

| Stage | Skills | Instructions | MCP | Result |
|-------|--------|-------------|-----|--------|
| 1. Baseline | -- | -- | -- | Functional but ungoverned ML code |
| 2. Skills | Workspace | -- | -- | Nokia-standard experiments (manual `@skill`) |
| 3. Skills + Instructions | Workspace | User-level | -- | Nokia-standard (automatic, no `@` needed) |
| 4. MCP | GitHub | User-level (MCP) | GitHub MCP | Nokia-standard (centralized, version-controlled) |

---

## Full Cleanup (After Demo)

```python
import mlflow
from mlflow import MlflowClient

client = MlflowClient()
mlflow.set_registry_uri("databricks-uc")

# Delete experiments
for exp_name in [
    "/Users/{username}/nokia_churn_propensity_xgboost",
    "/Users/{username}/nokia_churn_propensity_lightgbm"
]:
    exp = mlflow.get_experiment_by_name(exp_name)
    if exp:
        mlflow.delete_experiment(exp.experiment_id)

# Delete serving endpoints via UI or API
# Delete registered models via Catalog Explorer > Models
```

---

## Tips

- **Side-by-side comparison:** Keep Stage 1 notebook open while running Stage 2 to highlight the difference
- **MLflow UI:** Show the Experiments UI after each stage — the difference in run structure is immediately visible
- **Live edit:** Edit a skill in GitHub and show the next Genie Code session picks it up via MCP
- **Bonus prompt:** Ask Genie Code to compare two runs — it can read MLflow metrics and summarize which model performed better

---

## Prompt Reference Bank

A curated set of prompts for hands-on use during the workshop. Participants can pick up from any step depending on their progress.

> Replace `{catalog}`, `{schema}`, `{username}`, `{exp_name_1}`, `{exp_name_2}` with your actual values.

---

### 1. Exploratory Data Analysis (EDA)

**Prompt A1 — Quick EDA (non-expert friendly)**

> I'm a non-expert in ML. Using the table `{catalog}.{schema}.gold_account_summary`, please:
> - Show the schema and basic stats for all columns.
> - Identify which column looks like the churn label and confirm with me.
> - Do basic EDA: class balance for the label, missing values per column, and 3–4 useful visualizations.
> - Generate a clear notebook with Python code and Markdown explanations. Ask before running cells.

**Prompt A2 — Focused EDA with known label**

> Using table `{catalog}.{schema}.gold_account_summary` where the target is `churn_label` (0/1), do EDA:
> - Show feature distributions and correlations with `churn_label`.
> - Highlight obvious data quality issues.
> - Suggest 5–10 potentially useful features to focus on.
> - Use Python and Spark, and structure it as reusable notebook cells.

---

### 2. Build a First MLflow Model

**Prompt B1 — "You choose the algorithm" baseline**

> Build a baseline binary classification model to predict `churn_label` using the table `{catalog}.{schema}.gold_account_summary`.
> - You choose a simple, robust algorithm (for example logistic regression or tree-based).
> - Split into train/validation/test.
> - Use MLflow to track the experiment, parameters, metrics (AUC, accuracy, F1), and the model artifact.
> - Write clear comments so a non-ML person can follow what's happening.

**Prompt B2 — Specific algorithm (XGBoost)**

> Build me a churn prediction model using the gold customer table `{catalog}.{schema}.gold_account_summary`.
> - Target column: `churn_label`.
> - Use XGBoost.
> - Include sensible default hyperparameters and a train/validation split.
> - Use MLflow to log parameters, metrics (AUC, precision, recall), plots if possible, and the trained model.

---

### 3. Evaluation, Registration, and Serving

**Prompt C1 — Evaluation + registration + serving**

> Starting from the churn model you just trained:
> - Add MLflow code to evaluate the model on a held-out test set and log metrics and plots.
> - Register the best run as a model in Unity Catalog under `{catalog}.ml.churn_model`.
> - Create or update a Databricks Model Serving endpoint named `churn-model-endpoint` pointing at the latest version.
> - Generate all the code cells needed, with short explanations in Markdown.

**Prompt C2 — Promote model to staging**

> For the registered model `{catalog}.ml.churn_model`:
> - Find the best version by AUC from the latest experiment.
> - Transition that version to the Staging stage, and include code that could later move it to Production once approved.
> - Log what you changed with MLflow tags or a comment.

---

### 4. Compare Multiple Experiments / Models

**Prompt D1 — Train 2 models and compare**

> Using `{catalog}.{schema}.gold_account_summary` and target `churn_label`:
> - Create experiment 1: model using XGBoost.
> - Create experiment 2: model using LightGBM.
> - Use the same train/validation/test split for fair comparison.
> - Log both experiments with MLflow, then create a comparison cell that reads both runs, shows a table comparing AUC, accuracy, F1, and training time, and prints a recommendation on which model to use and why.

**Prompt D2 — Compare existing runs by name**

> In MLflow, I already have experiments `{exp_name_1}` and `{exp_name_2}` for churn prediction.
> - Write code to load their runs from MLflow.
> - Compare metrics (AUC, F1, log loss) across all runs.
> - Print the top 3 runs overall and indicate which model + parameters each used.
> - Make this a reusable function so I can plug in different experiment names.

---

### 5. Batch Inference and Online Scoring

**Prompt E1 — Batch scoring with a serving endpoint**

> Using the serving endpoint `churn-model-endpoint` and source table `{catalog}.{schema}.gold_scoring_input`:
> - Generate code to send batch requests to the endpoint (or use vectorized scoring if that's easier).
> - Write the predictions to a new table `{catalog}.{schema}.churn_predictions` with columns: all original keys + `churn_score` + `prediction_timestamp`.
> - Include basic error handling and log the scoring job with MLflow if possible.

**Prompt E2 — Offline scoring using the registered model**

> Using the registered model `{catalog}.ml.churn_model@Production`:
> - Load the model from MLflow in a notebook.
> - Score all rows from `{catalog}.{schema}.gold_scoring_input`.
> - Save the results to `{catalog}.{schema}.churn_predictions_offline` with prediction probabilities and a binary prediction.
> - Explain in comments how to swap in a different model later.

---

### 6. Monitoring and Experiment Organization (Optional)

**Prompt F1 — Organize experiments and tags (non-expert friendly)**

> I'm not an ML expert. Please:
> - Create example MLflow code that sets up a clear experiment naming scheme for churn models.
> - Use tags like `model_family`, `data_version`, `owner`, and `use_case`.
> - Show how to search for "best churn model on latest data" using the MLflow client.

**Prompt F2 — Add simple drift checks**

> Extend the churn prediction notebook to include basic data drift checks:
> - Compare feature distributions between the original training set and the latest data in `{catalog}.{schema}.gold_account_summary`.
> - Log summary statistics and any drift metrics you compute to MLflow under a "monitoring" run.
> - Keep the code as simple and commented as possible.
