---
description: Workflow for generating high-rigor Test Cases in Markdown format for the Mini HIS project.
---

# TEST CASE CREATION WORKFLOW (Mini HIS)

> **Design principle**: This Flow is a **pure orchestrator** — it controls WHEN to do things and WHEN to wait. All quality rules (what makes a good TC, how to structure sections, how to run gates) live exclusively in `Tester 2_Skill.md`. If a rule appears in both places, the Skill is authoritative.

---

## Step 1: Load Skill & Read Local Inputs

**Load the Skill first.** The Skill defines how to interpret every input source.

```
1. Load `.claude/.agents/skills/2. Create_TCs_Skill/Tester 2_Skill.md`
2. Load `.claude/.agents/skills/2. Create_TCs_Skill/QA_Checklist_Cheatsheet.md`
```

Then read **local files only** — do NOT read BA Demo or BA portal yet (those happen in the gates below):
- `.claude/rules/domain_knowledge.md`
- All Tester 1 Analysis files from `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/` — latest version of each: `TestCase_Material*.md`, `QnA_BA_*.md`, `QnA_Report_*.md`

---

## Step 1.5: BA Answer Declaration Gate (MANDATORY — ask before any further action)

If the user prompt does **not** include a clear statement about BA answer status → AI **MUST** ask immediately, before reading anything else:

> *"BA đã trả lời câu hỏi QnA nào chưa?*
> - *Chưa → tôi sẽ tiến hành, các câu hỏi chưa rõ vẫn giữ tag `[Confirming — pending BA#X]`*
> - *Đã trả lời → paste câu trả lời hoặc nội dung file vào đây, tôi sẽ sync trước khi viết TC"*

### Branch A — Chưa có answers:
→ Proceed to Gate 1.6.

### Branch B — Có answers (user paste vào chat):
Run **QnA Sync Pass**:

1. Parse từng answer từ user input → map vào BA# tương ứng trong `QnA_BA_<task>.md`
2. Update `QnA_BA_<task>.md`: điền `BA Status = Đã bổ sung`, `BA Answer = [nội dung]`
3. Classify từng answer theo bảng sau:

| Loại answer | Tester 2 tự xử lý được | Cần Tester 1 re-run |
|---|---|---|
| Confirm behavior đã infer đúng | Có — update `[Confirming]` → expected result cụ thể | Không |
| Chỉ confirm naming / wording | Có | Không |
| Rule mới chưa có trong analysis | Không | **Có** |
| Flow / field thay đổi so với analysis | Không | **Có** |

4. Nếu **tất cả** đều loại "tự xử lý được" → proceed to Gate 1.6
5. Nếu **có bất kỳ** answer nào cần Tester 1 → **STOP**. Report cho user:
   - Liệt kê BA# và lý do cần re-run
   - *"Đề nghị chạy Tester 1 re-run trước để cập nhật analysis. Tester 2 sẽ tiếp tục sau khi có TestCase_Material mới."*
   - **WAIT** — không tiếp tục cho đến khi user xác nhận đã chạy lại Tester 1

---

## Step 1.6: Spec Drift Gate (auto)

Đây là lần đầu tiên AI đọc BA Spec Portal trong workflow này.

1. Điều hướng đến `https://docs.sota-his.com/docs/business/usecases` — đọc các UC liên quan đến task
2. Đọc header của `TestCase_Material_<task>.md` → lấy ngày/version của lần analysis gần nhất
3. So sánh nội dung UC hiện tại với những gì Tester 1 đã ghi nhận:
   - Có field mới / field bị xóa?
   - Rule / flow thay đổi?
   - UC mới được thêm vào scope?
4. **Nếu phát hiện drift** → **STOP**. Report:
   - UC nào thay đổi
   - Tóm tắt nội dung khác biệt
   - *"Spec đã thay đổi sau lần analysis cuối. Đề nghị chạy Tester 1 re-run trước."*
   - **WAIT** — không tiếp tục cho đến khi user xác nhận
5. **Nếu clean** → log `"Spec Drift Gate: clean — checked [date], no changes detected"` và proceed

**Sau khi Gate 1.6 pass:** Đọc BA Demo để verify field names (per `Tester 2_Skill.md → Step 1` fallback priority). Đây là bước cuối của input gathering trước Plan Presentation.

---

## Step 2: Plan Presentation (MANDATORY — present after reading inputs)

Present the execution plan to the user with:

1. **Input Sources** — list all sources with exact file paths/URLs actually read
2. **Domain Knowledge** — which healthcare rules apply (BHYT, tuyến, etc.)
3. **Execution Steps** — ordered steps with expected output per step
4. **Coverage Estimate** — from TestCase_Material: Scenarios count, BR count, expected TC count per UC
5. **Output Expected** — file path, TC_ID naming rule, ONE draft row example
6. **Screen Type** — preliminary classification (WORKFLOW or LIST) with reasoning
7. **Questions** — any unclear points that need user clarification

**WAIT for user to confirm "OK" or provide feedback before proceeding.**

---

## Step 3: Pre-TC Analysis Gates (run in order — each gate feeds the next)

Run ALL gates below **before writing any TC**. Document each decision in working notes.

### Gate 3.1 — Screen Type Classification (ST-1)
- Read `Tester 2_Skill.md → SCREEN TYPE ADAPTIVE RULES → ST-1`
- Classify: WORKFLOW or LIST (or mixed)
- Document the classification in the file header before writing TCs

### Gate 3.2 — Figma Verification (mandatory for List screens)
- This gate is defined in `Tester 2_Skill.md → ST-2 → Figma Verification Gate`
- Run it now, before defining any FC section

### Gate 3.3 — Coverage Contract (Rule 21 in Skill)
- Extract all **Scenarios** from TestCase_Material → build numbered list
- Extract all **Business Rules (BR-XX)** → build numbered list
- This is the baseline every TC must map to at delivery

### Gate 3.4 — Security Scope Check (Step 2 in Skill)
- Evaluate all 6 security dimensions per Step 2 of the Skill
- Any that apply → must have corresponding TCs in Permission & Security section

### Gate 3.5 — Performance Self-Assessment (Rule 22 in Skill)
- Evaluate Q1, Q2, Q3 per Rule 22
- Document decision: REQUIRED or SKIPPED + reason
- If SKIPPED → add to file header: `"3.5 Performance skipped: [reason]"`

### Gate 3.6 — Component Discovery + Library Lookup
- Identify all standard UI components on screen (Search, Grid, Pagination, Dropdown…)
- Read `Refer/test case library.txt` — search ONLY sections for identified components
- Read `Refer/Test Case Tone and Wording Reference` — apply wording patterns to ALL TC fields

---

## Step 4: TC Generation

### Scale Check (run before writing):
- SMALL/MEDIUM: proceed with Markdown template from `template/Markdown_Template.md`
- LARGE (3+ modules or 10+ similar screens with repeated logic): **STOP. Warn user**: *"Feature is too large for individual Markdown TCs. Suggest Test Matrix or Script Generation."* Wait for decision.

### Output Structure:
Follow `Tester 2_Skill.md` for section decomposition and naming. The 3-section structure (Summary + UI TCs + Function TCs) is defined in the Skill — not duplicated here.

---

## Step 5: TC Delivery — Phase Loop

**Deliver in 3 phases. Never deliver all at once.**

### For each phase:
1. Write TCs for the phase
2. Run **EXECUTION READINESS REVIEW** from `Tester 2_Skill.md` — fix all failed checks
3. Save file → report to user:
   - TC count by priority (P0~P4): UI / Functional / Total
   - Top 3 Critical Risks covered
   - File path
   - Any `[Confirming]` items pending BA response
   - **New QnA entries appended to QnA_BA in this phase** (if any — list BA#X and one-line question summary)
4. **WAIT** for user confirmation before starting next phase

### Phase coverage:
| Phase | What to write |
|---|---|
| Phase 1 | Core functionality — CRUD chính, UI load, validation cơ bản ("hits user in the face" if broken) |
| Phase 2 | UI detail, complex validation, error handling |
| Phase 3 | Edge cases, permission/security, performance (if required) |

---

## Step 6: QnA Protocol (Tester 2 discoveries)

When writing TCs and encountering behavior that **cannot be confirmed** from any source (Tester 1 Analysis, existing QnA_BA, BA Demo, Spec):

1. Read `QnA_BA_<task>.md` from `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/` → get the current highest STT
2. Assign the next number (e.g., BA#16)
3. Append a new row to `QnA_BA_<task>.md`:
   - Fill all standard columns (STT, Màn hình, Câu hỏi, Gợi ý, BA Status, BA Answer)
   - In the `[Note]` column: add `from Tester 2`
4. Write `[Confirming — pending BA#16]` in the TC's Expected Result
5. Report all new BA#X entries in the phase delivery summary (Step 5 point 3)

**Scope boundary:**
- Append to `QnA_BA_<task>.md` ONLY — **do NOT modify `QnA_Report_<task>.md`** (owned by Tester 1)
- Apply source-tag rules same as Tester 1: use `[Regulatory: ...]` or `[Standard UX: ...]` first; only create a new BA#X when neither tag covers the ambiguity

---

## Step 7: Versioning & File Output

- Files are saved **progressively** — after each phase in Step 5. Do NOT wait until all phases are done.
- Save to `Output_QC/2. Test case/<site_folder>/<module_folder>/`

  **Folder mapping rule** (from task_name prefix):
  | task_name prefix | site_folder | module_folder |
  |---|---|---|
  | `01_M0_...` | `01_User site` | `M0_Dashboard` |
  | `01_M1_...` | `01_User site` | `M1_Dang ky kham benh` |
  | `01_M2_...` | `01_User site` | `M2_Kham benh` |
  | `01_M3_...` | `01_User site` | `M3_Vien phi va BHYT` |
  | `01_M4_...` | `01_User site` | `M4_Quan ly kho duoc` |
  | `02_M10_...` | `02_Amin site` | `M10_Cau hinh danh muc` |

- **Auto-increment version**: check existing files. If `_v1.md` exists → create `_v2.md`. Do NOT move or delete old files.
- File header MUST include: UC IDs, date, sources, version notes, performance decision, any cross-module impact note.
