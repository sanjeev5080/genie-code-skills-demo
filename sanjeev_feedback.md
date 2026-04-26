# Feedback

---

## Feedback 1 — DEMO_SCRIPT.md Stage 2: Git directory structure should match workspace

**File:** `docs/DEMO_SCRIPT.md` → Stage 2 Setup

**Feedback:** For the Stage 2 skills setup section, the git directory structure should be the same as what is expected to be in the Workspace. This way users can copy the folders directly without renaming.

**Change made:** Restructured `skills/data_eng/` from flat `.md` files to `skill-name/SKILL.md` folders, mirroring the workspace layout exactly.

---

## Feedback 2 — sdp-basics/SKILL.md: Remove deprecated LIVE.table_name syntax

**File:** `skills/data_eng/sdp-basics/SKILL.md` → Table Types table + Joins section

**Feedback:** `LIVE.table_name` (e.g. `FROM LIVE.bronze_articles`) used for referencing tables within the same pipeline is old syntax and no longer recommended. This should be removed.

**Change made:** Removed the `LIVE.table_name` row from the Table Types table and removed the corresponding line from the Joins section. Replaced with fully qualified name guidance.

---

## Feedback 3 — DEMO_SCRIPT.md Stage 2: Clarify that @tag directly refers to the skill file

**File:** `docs/DEMO_SCRIPT.md` → Stage 2, Setup + Prompts sections

**Feedback:** In Stage 2, it should be clearly specified that the `@tag` used in the prompts (e.g. `@sdp-basics`, `@table-governance`, `@pii-management`) directly refers to the uploaded skill folder/file. Users need to understand the connection — the folder name you upload to the workspace becomes the `@tag` name you use in Genie Code prompts.

**Suggested addition:** Add a note in the Setup section explaining: *"The folder name (e.g. `sdp-basics/`) becomes the `@tag` name you use in prompts — `@sdp-basics` tells Genie Code to load and apply that skill's `SKILL.md`."*

---

## Feedback 5 — table-governance/SKILL.md: ALTER TABLE must not be placed inside pipeline SQL files

**File:** `skills/data_eng/table-governance/SKILL.md`

**Error:**
```
[PARSE_SYNTAX_ERROR] Syntax error at or near end of input.
```

**Root Cause:** The skill instructs Genie Code to add `ALTER TABLE` statements for column descriptions and UC tags but gives no guidance on where to place them. Genie Code embeds them inside the pipeline `.sql` file (sometimes in `/* ... */` comment blocks). The SDP pipeline parser fails on these even when commented out.

**Fix applied:**
- Added explicit warning in the `Column Descriptions via ALTER TABLE` section: do NOT place `ALTER TABLE` inside pipeline `.sql` files
- Added same warning to the UC Tags section
- Split the full example into two clearly labelled blocks:
  - **Step 1:** `CREATE OR REFRESH ...` → goes in the pipeline `.sql` file
  - **Step 2:** `ALTER TABLE ...` → run separately in a SQL editor or notebook after the pipeline has created the tables

---

## Feedback 4 — table-governance/SKILL.md: "owner" is a reserved TBLPROPERTIES key

**File:** `skills/data_eng/table-governance/SKILL.md`

**Issue:** `"owner"` is a reserved table property in Databricks — it is automatically set to the current user and cannot be manually specified in `TBLPROPERTIES`. Using it causes:
```
[UNSUPPORTED_FEATURE.SET_TABLE_PROPERTY] The feature is not supported: owner is a reserved table property, it will be set to the current user.
```

**Fix applied:** Renamed `"owner"` to `"team"` in all TBLPROPERTIES definitions and examples. The concept is preserved — just using a non-reserved key name.

---
