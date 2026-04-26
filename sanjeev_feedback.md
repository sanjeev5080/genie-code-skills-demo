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
