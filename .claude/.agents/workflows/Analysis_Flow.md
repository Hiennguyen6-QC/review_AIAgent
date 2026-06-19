---
description: Trigger hybrid requirement review for Mini HIS project (supports online docs, Figma, and local files).
---

# /Analysis_Flow — Hybrid Requirement Analysis Workflow

## Purpose
Trigger the requirement review. Handles both local files and external web/Figma links. It outputs a lightweight, Slack-friendly clarification list instead of a strict blocker report, alongside a pure TestCase Material file for the next AI agent.

## Step 1 — Reconnaissance & Information Gathering (MANDATORY)
0. **[READ SKILL FIRST]**: Before any other action, read `.claude/.agents/skills/1. Analysis_Skill/Tester 1_Skill.md` in full. This defines all protocols and decision rules used in subsequent steps.
1. Note the `<function_id>` from user prompt (e.g., `01_M1_F1`).
2. **[MANDATORY] QC-Function Lookup**: Read `BA/QC-Function list M*.md` → find the row matching `<function_id>` → extract `<task_name>`, UC IDs, and BA portal URLs for this function. This is the authoritative source for task scoping and output folder naming.
3. Activate the `Tester 1-Analyzer` agent.
4. **PRIMARY SPEC SOURCE — MANDATORY**: Navigate to the BA web portal **`https://docs.sota-his.com/docs/business/usecases`** using the browser tool, reading the UC IDs extracted in step 2. This is the **only authoritative source** for BA specs. Do **NOT** read from the local `BA/` folder. Follow the BA Portal Login Protocol and URL Recovery Rule in `Tester 1_Skill.md` Phase 1 step 4.
5. After the BA portal is accessible, also fetch any other supplementary URLs provided (e.g., Figma links) and scan the `Document refer/` directory. Follow the UI Observation Checklist in `Tester 1_Skill.md` Phase 1 step 5.
6. **[MANDATORY GATE — Clarification]**: Run Pre-Analysis Clarification Gate per `Tester 1_Skill.md` Phase 1 step 11. If critical UNKNOWN/INFERRED items exist → STOP, print QnA to user, WAIT for confirmation before Step 2.
7. **[MANDATORY GATE — Spec Consistency Check]**: Run Phase 1.5 per `Tester 1_Skill.md`. Conflicts found → add as QnA entries, report to user. None found → log `Phase 1.5: No conflicts detected.` and proceed.

## Step 2 — Analysis (Only after Phase 1 is cleared)
1. Apply the `Tester 1_Skill` to synthesize the fragmented input files (Docs, CSVs, Figma, APIs).
2. **AUTO-MODE**: Automatically determine the appropriate `mode` (QUICK/DEEP) if not specified by the user based on task complexity.

## Step 3 — Report Generation
1. Parse the synthesized data into the THREE output templates located in `.claude/.agents/skills/1. Analysis_Skill/template/`.
2. Create output directory if it doesn't exist: `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/`

   **Folder mapping rule** (from task_name prefix):
   | task_name prefix | site_folder | module_folder |
   |---|---|---|
   | `01_M0_...` | `01_User site` | `M0_Dashboard` |
   | `01_M1_...` | `01_User site` | `M1_Dang ky kham benh` |
   | `01_M2_...` | `01_User site` | `M2_Kham benh` |
   | `01_M3_...` | `01_User site` | `M3_Vien phi va BHYT` |
   | `01_M4_...` | `01_User site` | `M4_Quan ly kho duoc` |
   | `02_M10_...` | `02_Amin site` | `M10_Cau hinh danh muc` |

3. Save the reports to:
   - `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/QnA_Report_<TestCaseID>.md`
   - `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/TestCase_Material_<TestCaseID>.md`
   - `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/QnA_BA_<TestCaseID>.md`
4. Run Post-Write Structural Validation per Skill Phase 3 Step 4. Fix any failures before reporting done.
5. Display the Slack-friendly Clarification List to the user in chat, so they can immediately copy-paste it to their PO/DEV.
