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
