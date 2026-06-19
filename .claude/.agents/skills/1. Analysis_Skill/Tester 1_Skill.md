---
name: Hybrid_RequirementAnalysis_Skill
---

# Hybrid Requirement Analysis Skill

## Purpose
Analyze scattered requirement artifacts using severe QA analytical thinking to assess structural risks, identify production failure points, and produce actionable test structures without auto-failing the documents. Focus strictly on coverage, not just summarizing text. 
Ensure rigorous validation of UI constraints and 3C criteria (Completeness, Clarity, Conflict).

## Required Parameters
- `mode`: **QUICK** (for simple bug fixes, small UI changes) OR **DEEP** (for complex, unclear, or high-risk features). Must be identified based on the task prompt. If omitted, default to DEEP.

## 3-Phase Execution

### Phase 1 — Ingest, Scope Isolation & Strict QA Mindset Inference
1. **MANDATORY CONTEXT**: Always start by reading these files — in order — before ANY other action:
   1. `.claude/rules/domain_knowledge.md` — project terminology, BHYT Business Rules Checklist, tech stack, testing strategy
   2. `.claude/rules/ui_ux_standards.md` — UI standards, validation rules, TC writing standards
   3. **BA Common Rules (live)**: Navigate to `https://docs.sota-his.com/docs/business/usecases/common` and read ALL CMR entries (CMR-001 to end). These are BA-authored cross-cutting rules applying to ALL screens — input length limits, blocking behavior, pagination layout, soft-delete, audit log, etc. BA owns and updates this page; read it fresh each session. Apply CMRs with same weight as domain_knowledge.md rules.
      - **Auto-login (HTTP Basic Auth)**: Read `BA_PORTAL_USER` and `BA_PORTAL_PASS` from `.env.test`. The portal uses HTTP Basic Authentication — **NOT** a login form. Construct the URL with embedded credentials: `https://<user>:<pass_encoded>@docs.sota-his.com/...`. URL-encode special characters in the password before embedding (e.g., `#` → `%23`, `@` → `%40`). Example: password `9fouBA8wL!#7!Lrm` → embed as `9fouBA8wL!%237!Lrm`. If credentials are empty in `.env.test` → ask user to fill them in first. Do NOT ask user to login manually if credentials are present.
      - **[MANDATORY] CMR Stale-Data Guard (Rule L)**: DO NOT use CMR rules from conversation history, prior session context, or memory — even if CMR was read earlier in the same thread. Re-read CMR fresh whenever ANY of these triggers fire:
        - User mentions "BA vừa update CMR", "common rule mới", or "CMR v2.X"
        - Session gap > 1 day since last CMR read
        - Any CMR-based conflict analysis is about to be written
        If the portal is inaccessible → **flag immediately and stop**. Do NOT proceed with CMR-based analysis using cached/remembered rules. Reason: BA updates CMR frequently (e.g., v2.1 → v2.4 within 2 days); stale CMR = entire conflict analysis is built on wrong basis and must be redone.
2. **[MANDATORY] Previous Output Review (When creating a new version)**:
   - BEFORE starting analysis, check the corresponding output folder (`Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/`).
   - IF previous output versions exist (v1, v2...):
     a. READ ALL previous outputs (QnA_Report, TestCase_Material, QnA_BA) in full.
     b. IDENTIFY: confirmed QC Answers, points user agreed/rejected, preferred style/format/depth.
     c. CARRY FORWARD: Retain all confirmed QC Answers (do NOT re-ask). Only re-analyze points that are still unclear or have new information available.
     d. IF user did NOT request a change in approach → maintain the same structure/depth as the previous version.
     e. **[REMOVAL GATE — CRITICAL — Applies to BOTH QnA_BA AND QnA_Report]**:
        This is a **Product project** — BA spec is incomplete and evolving. QnA files serve as the bridge to get BA to update their documentation. A question can ONLY be removed from QnA_BA or QnA_Report if:
        - BA Status = `Đã bổ sung` (BA has officially updated spec)
        - OR BA Answer is filled with official confirmation
        
        **QC confirming behavior ≠ permission to remove.** Even if QC observed and confirmed the behavior (from screenshot, demo, testing), the question MUST stay if BA has NOT yet documented it in spec.
        
        **Downstream impact on TestCase_Material:**
        - **[CRITICAL] SPEC IS THE ONLY SOURCE OF TRUTH FOR TEST CRITERIA.** TestCase_Material Expected Results MUST remain `[Confirming]` until BA officially updates spec — regardless of who confirmed the behavior (QC, dev, Figma, demo).
        - QC Answer, dev behavior, Figma design = **observation/input** → helps BA decide what to write in spec. They do NOT replace spec as test criteria.
        - Even if QC leader, dev lead, or anyone else confirms behavior via chat → TestCase_Material keeps `[Confirming]` + notes the observation as reference.
        - When BA officially answers or spec is updated → THEN replace `[Confirming]` with concrete expected result.
        - Mark scenarios dependent on unconfirmed spec with `[Confirming — QC observed: <brief description>. Pending BA spec update (QnA BA#X)]` in Expected Result.
        - **Anti-pattern (NEVER DO)**: ❌ "QC confirmed → remove [Confirming]". ❌ "Dev is doing it this way → confirmed by dev behavior". ❌ "Figma shows X → replace [Confirming] with X". All of these are WRONG — spec is spec, observation is observation.

        - **[SOURCE-TAG TAXONOMY — MANDATORY for non-spec test criteria]**:
          When TestCase_Material Expected Results / Business Rules need to reference authority that is NOT directly from BA spec, AI MUST categorize the source using one of these tags (no longer use generic `[inferred]` or `[QC view]`):
          
          | Tag | When to use | Example |
          |---|---|---|
          | `[Regulatory: <law/regulation reference>]` | Behavior is governed by Vietnamese law/regulation/MOH decision (QĐ 7603/QĐ-BYT, Thông tư 56/2017/TT-BYT, Luật BHYT, etc.). This is the **strongest non-spec source** — auditable, citable, default-applies even if spec is silent. | `[Regulatory: QĐ 7603/BYT Điều 22]` for Cấp cứu auto-set Đúng tuyến |
          | `[Standard UX: <pattern source>]` | Behavior follows industry-standard UX pattern (Ant Design defaults, REST best practice, common HIS pattern). Use only when pattern is universally expected and spec is silent. | `[Standard UX: Ant Design Form validation]` for inline error display |
          | `[Confirming — pending BA#X]` | Behavior cannot be inferred from regulation or standard pattern; needs explicit BA spec. **MUST** include the BA#X cross-reference to QnA_BA. | `[Confirming — pending BA#14]` for ambiguous business rule |
          
          **Tags can combine** — e.g., a regulatory rule that BA has not yet documented:
          `[Regulatory: QĐ 7603/BYT Điều 22]` + `[Confirming — pending BA#14]` (BA "Coming soon")
          
          **Anti-pattern (NEVER DO)**:
          - ❌ `[inferred]` (no source)
          - ❌ `[QC view]` (QC opinion ≠ test criteria)
          - ❌ `[inferred - regulatory]` (use the explicit `[Regulatory: ...]` tag with citation)
          - ❌ Confirming without BA#X reference (must point to a QnA_BA STT for traceability)
          
          **[AUTO-QnA RULE — NO USER CONFIRMATION NEEDED]**: Whenever AI writes a new `[Confirming — pending BA#X]` tag anywhere in TestCase_Material (BR, Scenario, Data Dictionary, Section 5), AI MUST **automatically** in the same output:
          1. **Pre-creation check**: Read QnA_BA → get actual last STT. Next BA#X = last STT + 1. QnA_BA is the only source of truth — never derive STT from QnA_Report count, memory, or assumption. If gap found between QnA_BA and QnA_Report → flag before proceeding.
          2. Assign BA#X per step 1
          3. Create the corresponding QnA entry in **QnA_BA** (full table row with all columns)
          4. Create the matching entry in **QnA_Report** (with `[BA#X]` cross-reference tag, under appropriate category)
          5. Add the corresponding entry to **TestCase_Material Section 5.1** (Requirement Gaps) with `→ **QnA: BA#X**` bridge tag — per Risk ↔ QnA Bridge Rule
          6. Never write `[Confirming — pending BA: <description>]` without a number — always `[Confirming — pending BA#X]`
          
          **Writing QnA for BA**: Use full names for UI components (e.g., panel "Lịch sử sử dụng thẻ BHYT", not "P4"). BA does not know internal abbreviations.
          
          This is automatic workflow — do NOT ask user for permission. Every `[Confirming]` without BA#X is a rule violation.
          
          **Audit trail**: When a `[Confirming — pending BA#X]` is finally answered by BA, the tag is replaced with concrete expected result + a footnote `(Resolved by BA#X on YYYY-MM-DD)`. Regulatory tags remain even after BA confirms — they document the legal basis.
        
     f. **[QnA ORDERING RULE]**: V1 questions keep their original STT order. V2 NEW questions append after V1. Never reorder or renumber — EXCEPT when user confirms an entry is an AI error (no spec/CMR basis, or spec already covers it). In that case: delete from both QnA_BA and QnA_Report atomically, renumber subsequent STTs, update all BA#X references in TestCase_Material. AI must NOT self-delete — only user can trigger this.
   - IF NO previous output exists → skip this step, perform fresh analysis.
3. **[MANDATORY] Reference Output Review (Format Calibration)**:
   - BEFORE writing any output, AI MUST read at least 1 complete output set already existing in project to calibrate format, depth, and style.
   - **Default reference**: `Output_QC/1. Analysis/01_User site/M1_Dang ky kham benh/01_M1_F1_01TiepdonBN/` — read 3 files (latest version):
     - `QnA_Report_01_M1_F1_01TiepdonBN_v2.md`
     - `TestCase_Material_01_M1_F1_01TiepdonBN_v2.md`
     - `QnA_BA_01_M1_F1_01TiepdonBN_v2.md`
   - **Purpose**: Not just template structure compliance, but also matching:
     - Depth of QA Assessment, AI Suggestion (how detailed, how much reasoning)
     - Source-Tag usage and BA#X cross-reference style
     - QnA_BA 14-column table formatting
     - Level of business logic decomposition in TestCase_Material
     - Readiness Score evaluation depth
   - **ALSO check**: When looking up previous version to calibrate format, the folder structure is `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/`. Mapping rule:
     - `01_M0_...` → `01_User site/M0_Dashboard/`
     - `01_M1_...` → `01_User site/M1_Dang ky kham benh/`
     - `01_M2_...` → `01_User site/M2_Kham benh/`
     - `01_M3_...` → `01_User site/M3_Vien phi va BHYT/`
     - `01_M4_...` → `01_User site/M4_Quan ly kho duoc/`
     - `02_M10_...` → `02_Amin site/M10_Cau hinh danh muc/`
   - **If reference output does not exist** (first time in project): skip this step, rely on templates only.
   - **If user says reference output needs improvement**: follow user guidance on what to change.
4. **PRIMARY SPEC SOURCE — MANDATORY**: Navigate to the BA web portal **`https://docs.sota-his.com/docs/business/usecases`** using the browser tool and read the relevant Use Cases and Business Rules. **Do NOT read from the local `BA/` folder** — it is deprecated.

   **[MANDATORY] BA Portal Login Protocol:**
   - **Portal uses HTTP Basic Authentication** — NOT a login form. Always embed credentials directly in the URL.
   - **Step 1 — Read credentials from `.env.test`**: Get `BA_PORTAL_USER` and `BA_PORTAL_PASS`. URL-encode special characters in the password (`#` → `%23`, `@` → `%40`, `!` stays as `!`).
   - **Step 2 — Navigate with embedded credentials**: `https://<BA_PORTAL_USER>:<encoded_pass>@docs.sota-his.com/docs/business/usecases`. Example: `https://sotatek:9fouBA8wL!%237!Lrm@docs.sota-his.com/docs/business/usecases`.
   - **Step 3 — If ERR_INVALID_AUTH_CREDENTIALS**: Credentials in `.env.test` may have changed. Ask user to update `.env.test` with the correct password.
   - **NEVER ask user to login manually** if credentials are present in `.env.test`.
   - **NEVER fallback to Figma/local BA folder** solely because portal auth failed — fix the credentials first.

   **[MANDATORY] URL Recovery Rule — When a URL in the QC-Function list / WBS is wrong or returns 403/404:**
   AI **MUST** be flexible — do NOT stop analysis just because a URL fails. Recovery procedure (in order):
   1. **Try direct URL** from QC-Function list / WBS first.
   2. **If it fails (403, 404, or points to wrong UC)** → do NOT immediately conclude the portal is inaccessible. Continue with:
      a. Navigate to **portal root** `https://docs.sota-his.com/docs/business/usecases` → confirm whether the portal is accessible
      b. Navigate to the **module index** (e.g., `/m10-quan-tri`, `/m1-dang-ky-kham-benh`) → list all UCs in the module
      c. **Search/scan** the page for a link containing the required UC ID (e.g., search for "UC-189", "UC-299" via `document.querySelectorAll('a')` or page snapshot)
      d. Click the correct link → read spec content
   3. **If the URL in QC-Function list / WBS is wrong** (404 or points to wrong UC ID) → AI MUST:
      - Document clearly in output: `URL discrepancy detected: WBS/QC-Function ghi <wrong-url>, đúng URL là <correct-url>`
      - Flag to user to fix the source file (create a QnA Note or document in report)
   4. **ONLY STOP when**:
      - Portal root also returns 403/login required (entire portal inaccessible)
      - Navigated full module but found no matching UC ID (UC does not exist yet)
      - NOT when a single specific URL fails but the portal is still live

   **Anti-pattern (NEVER DO)**:
   - ❌ WBS URL returns 403 → STOP immediately, conclude "portal inaccessible"
   - ❌ Self-fallback to Figma + inference when only the URL path is wrong
   - ❌ Trust URLs in WBS/QC-Function list 100% — source files can be outdated/contain typos

5. **[MANDATORY — BLOCKING GATE] BA DESIGN DEMO / FIGMA — UI Verification**:

   > ⛔ **HARD BLOCK**: Tester 1 CANNOT proceed to Phase 1.5 or Phase 2 without completing step 5. No exceptions. If neither BA Demo nor Figma is accessible → STOP and ask user. Do NOT skip or defer to "I'll reference Figma later."

   **Purpose**: QC MUST see the actual UI before analysis. Analysis based on spec text without viewing the UI = lacks a real foundation; easy to miss fields, layout, and behavior.

   **Execution (in priority order)**:
   1. **BA Demo** (`BA_DEMO_URL` in `.env.test`) — highest priority.
      - Navigate to the URL, log in using credentials from `.env.test` (staff account for Reception/Doctor, admin account for Admin screens).
      - Open the exact screen being analyzed.
   2. **Figma** (file key from `domain_knowledge.md` or link from WBS/QC-Function list) — if BA Demo does NOT have the relevant screen.
   3. **Ask user** — if neither source is accessible → **STOP IMMEDIATELY**, ask user to provide a design reference before continuing.

   **[MANDATORY] UI Observation Checklist** — MUST be completed before proceeding to Phase 2:
   - [ ] Opened the correct screen relevant to the task being analyzed
   - [ ] Listed ALL field labels, button labels, placeholder text, column headers (exact names as shown on UI)
   - [ ] Noted overall layout: how many sections/panels? Display order?
   - [ ] Noted all interactive elements: dropdown options, checkbox/radio states, expandable panels
   - [ ] Noted the source used: "BA Demo" or "Figma (BA Demo not available)"

   **Field Name Authority**: Tên field từ BA Demo/Figma là **authoritative source** — override spec text và WBS naming nếu có mâu thuẫn. Nếu tên khác nhau giữa sources → ghi vào **Naming Discrepancies table** trong QnA_Report.

   **Read-only access rules apply** (see `browser_safety_rules.md`).
6. After the BA portal and demo are accessible, also read any other supplementary URLs (Figma, etc.) and scan `Document refer/` for supporting references.
7. **[MANDATORY] WBS Reference Lookup**:
   - Open the latest WBS file in `BA/` (pattern: `WBS*.csv`)
   - Find the row(s) matching the Module/Function being analyzed
   - Read the **"Tài liệu tham khảo"** columns: **Biểu mẫu** (forms/templates) and **Thông tư** (regulations/circulars)
   - These references define the **legal and regulatory context** for the function. AI MUST use them to:
     - Understand which government regulations apply (e.g., QĐ 7603/QĐ-BYT, Thông tư 56/2017/TT-BYT)
     - Identify required data fields mandated by regulations (even if UI doesn't show them)
     - Cross-check UI fields against regulatory requirements → flag missing fields
   - Also read the **"Note nghiệp vụ cho QA/QC"** column for BA's business notes
   - Also read the **"Note"** column for additional context about the function
8. **Scope Isolation**: Explicitly define the boundary of the ticket.
9. **Context Sufficiency Check**: Is the context TRULY missing after cross-referencing ALL provided sources? If NOT → list missing information before proceeding.
10. **[Bug-Driven Enhancement]**: If historical bug files exist, analyze them to identify "Hotspots" and recurring Root Causes.
11. **[MANDATORY] Pre-Analysis Clarification Gate**:
   BEFORE proceeding to Phase 2, agent MUST:
   - List ALL items classified as UNKNOWN or INFERRED from Phase 1.
   - For each critical item (scope, new business rule, revert behavior): add to QnA file with status PENDING.
   - WAIT for user confirmation on critical items BEFORE writing the Analysis Detail.
12. **[MANDATORY] Tool Call Safety Reminder**: Before using Bash or PowerShell (i.e., any tool other than Read / Grep / Edit / Write), AI MUST explain "Why + What" to the user per Rule 7 of `.ai_assistant_rules.md`. Execute ONLY after the user acknowledges. For bulk renumber or file operations, prefer Read + Edit tools (1–5 changes) or PowerShell loop (>5 changes) — see `.claude/rules/execution_platform_notes.md` for platform-specific constraints.

### Phase 1.5 — Spec Consistency Check (MANDATORY — runs before Phase 2)

**Purpose**: Audit each UC for internal contradictions before extracting BRs or scenarios. Treat the spec as an artifact to be audited, not just a data source to extract from.

**Execution**: Read each UC in scope linearly — start to finish, every section — with the sole goal of detecting contradictions. Do NOT extract BRs or scenarios yet.

**Checklist (apply to each UC in scope):**
- [ ] **Postcondition vs Main Flow**: Does the postcondition contradict what the main flow actually does?
- [ ] **Purpose vs Preconditions**: Does the stated purpose conflict with its own preconditions?
- [ ] **Alt Flow coverage**: Have all Alt Flows been read end-to-end? Is any Alt Flow referenced in the main flow but missing from the Alt Flow list?
- [ ] **BR ID cross-UC**: Are BR IDs within this UC group unique? Flag any duplicate BR IDs across UCs in the same function scope.
- [ ] **Toast/message verb mismatch**: Do toast messages use verbs matching the action? (e.g., "thêm" appearing in an Edit UC = copy-paste remnant)
- [ ] **Modal/screen title remnants**: Do modal titles contain words from a different context? (e.g., "mới" in an Edit modal)
- [ ] **Exception Flow copy-paste**: Does any Exception Flow text appear copied from a different UC without adaptation?

**Output**:
- Conflicts found → add each as a QnA item immediately, before proceeding to Phase 2.
- No conflicts found → log `Phase 1.5: No conflicts detected.` and proceed.
- Never skip this phase regardless of spec length or apparent simplicity.

### Phase 2 — Analyze & Extract
Apply the appropriate depth based on `mode`:

0. **Business & User Thinking**
   - Identify the core business goal and healthcare impact (e.g., BHYT billing, clinical decisions).
   - Identify primary user roles (Doctors, Pharmacists, Receptionists, Admins).
   - **[MANDATORY] Section 1B — Business Context & Valid Test Data Guide**: If the function involves financial parameters, regulatory compliance, or cross-module impact → AI MUST produce Section 1B in TestCase_Material explaining:
     - Business meaning of each field (why this value, not just what it is)
     - Legal basis / regulation reference (Nghị định, Luật, Thông tư)
     - Valid test data with real-world values (so QC tests with correct data)
     - Invalid test data (for negative cases)
     - Impact analysis: what breaks downstream if config is wrong
     - Skip Section 1B ONLY for simple CRUD catalogs with no business logic.

1. **3C Filtering:** First, evaluate the artifact for Completeness, Clarity, and Conflict.

2. **Business Rules Extraction:** Convert ALL explicit/implicit rules into strict **IF-THEN format**. Pay special attention to medical compliance rules (e.g., BHYT coverage limits, required ICD-10 codes).

3. **Medical & PII Preconditions**: Explicitly extract system preconditions. Crucially, analyze Role-Based Access Controls (RBAC) and Patient Identifiable Information (PII) exposure risks.

4. **State Coverage Analysis**: Actively evaluate the requirement against all UI/Data states: *Default, Empty, Has Data, Loading, Disabled, Error, Success, and Timeout*. If the spec misses definitions for these states (e.g., no empty state design), log them as QnA.

4A. **[MANDATORY] UI Components Table Cross-Check & Component Type Appropriateness Review**

   **Purpose**: Systematic verification that (a) every field in the BA spec's UI Components table is covered in output, and (b) BA's chosen component types are UX-optimal.

   **Step 1 — Spec table extract**: From the UC in scope, locate the UI Components table (usually titled "Các thành phần giao diện" or "UI Components"). Extract every row: `#id | Field name | Component Type | Mandatory | Constraint`.

   **Step 2 — Row-by-row cross-check vs Data Dictionary** (Section 2 of TestCase_Material):

   - **[MANDATORY] Section 2 structure**: Data Dictionary MUST be split into sub-sections per UC ID — 1 bảng per UC (e.g., `### 2.1 UC-D03 — View Mode`, `### 2.2 UC-D04 — Edit Mode`). Fields xuất hiện ở nhiều UC → lặp lại ở từng sub-section, ghi rõ constraint khác biệt nếu có. Không dùng cột `Scope` để gộp chung — agent tiếp theo (Tester 2) phải biết field thuộc UC nào mà không cần suy luận.
   - For EACH spec table row: confirm the field exists in the output's Data Dictionary sub-section tương ứng với UC đó.
   - If a field is in spec but NOT in output → flag as **missing field** → add to correct sub-section + create QnA if behavior is unclear.
   - If a constraint (max length, format) is in spec but NOT in output → add to constraint column.

   **Step 3 — Component type appropriateness review**:

   Với TỪNG field, hỏi: *"Component type BA chọn có phù hợp với cách người dùng tương tác với data này không?"*

   Đánh giá theo thứ tự:
   1. **CMR trước** → có CMR nào quy định component type cho context này không? → cite CMR-XXX.
   2. **BA Demo / Figma** → component thực tế trên màn hình là gì? Nếu khác spec → Naming Discrepancy.
   3. **Standard UX (Ant Design)** → component nào phù hợp nhất cho loại data và cách dùng này?

   Nếu BA's choice khác kết quả đánh giá → tạo QnA (Rule D+J). Figma = evidence, không phải reasoning (Rule A).

5. **Test Scenarios (NOT detailed test cases):** Define Functional, Negative, and Edge scenarios.
   - **Scenario Table Columns (6 columns)**: `Scenario ID | Category | Scenario Description | Expected Result | Priority | Source`. NO "Linked BR" column — the Source column already traces back to BRs via Source-Tag Taxonomy (e.g., `[from spec]`, `[Confirming — pending BA#X]`).
   - **Scenario Granularity**: Ensure scenarios are atomic (1 Scenario = 1 specific behavior/validation).
   - Segregate scenarios into **GUI Scenarios** and **Function Scenarios**. Group Function Scenarios logically (Main Flow, Validation, Error Handling, Permission, Edge Cases).
   - **Deep Healthcare State Analysis**: For clinical workflows, explicitly challenge the underlying logic:
     - **Concurrency & Multi-user Conflicts**: What happens when multiple roles interact with the same patient record simultaneously?
     - **Integration Fallbacks**: How does the system behave when external APIs (LIS, PACS, BHYT) time out or return errors?
     - **Auditability**: Is the creation/modification of medical records properly logged?
   - **MUST Include**: At least 1 Negative Case and 1 Edge Case.
   - **[MANDATORY] Risk-Based Case Filtering**: Before outputting a case, ask: "Does this case carry a real risk? Does its logic duplicate a previous case?" If it fails → do NOT output it; merge or remove to keep the test set lean.
   - **[MANDATORY] Expected Result Completeness Check**: Each scenario must specify a concrete output (new DB state? redirect target? toast message?). Do NOT use vague phrases like "System works normally".
   - **[MANDATORY] SCREEN-WIDE ACTION COVERAGE**: For EVERY interactive element on the screen (button, panel, expandable section, clickable row, tab), AI MUST be able to answer 3 questions:
     1. How many **states** does this element have? (empty, has data, loading, expanded, collapsed...)
     2. What **data** does each state display? (columns, rows, summary text...)
     3. Where does clicking/interacting **lead**? (navigate, open modal, load data into form...)
     If ANY question cannot be answered → create a QnA immediately. Do NOT only verify the default state (e.g., "Chưa chọn bệnh nhân") then skip the other states. QC mindset: "I will test ALL actions on this screen — where does each action lead?"

6. **Flow Consistency Check**: Ensure all flows are logically connected (e.g. Check-in -> Examination -> Pharmacy -> Billing).

7. **Data Validation Thinking**: Identify critical data points. Validate data lifecycle. Detect risks of inconsistent or stale data (especially for medical records).

8. **[MANDATORY] Cross-State & Data Conflict Analysis**: For every form/screen, AI MUST think through:
   - **State Transition**: When a key field changes value (e.g., Đối tượng BHYT → Dịch vụ), what happens to related fields? Are they hidden, disabled, cleared, or preserved?
   - **Revert State**: When a state-changing action is UNDONE (e.g., untick checkbox, switch back to original value), does the system revert dependent fields to their previous values? Or do they stay changed? This is a commonly missed scenario — for EVERY "set" scenario, consider the corresponding "unset/revert" scenario.
   - **Data Source Conflict**: When multiple data sources exist (manual input, CCCD scan, BHYT card scan, existing DB record), which source takes priority? Does new data overwrite old data? Is there a confirmation prompt?
   - **Conditional Field Dependencies**: When checkbox/radio changes (e.g., tick [Cấp cứu], [Chuyển tuyến]), do dependent fields auto-update (mức hưởng, loại tuyến)? Are new required fields revealed?
   - **Validity Window**: For date-bounded data (BHYT card, licenses, certificates), check BOTH directions: expired (end date < today) AND not-yet-valid (start date > today). Both are distinct scenarios requiring separate handling.
   - **Duplicate/Ambiguity Scenarios**: When identifying data is missing or non-unique (e.g., no CCCD, duplicate names), how does the system disambiguate?

9. **[MANDATORY] UI vs Regulation Gap Check**: Cross-reference the UI fields visible on screen (from Figma/spec/screenshot) against:
   - Regulatory requirements from WBS "Tài liệu tham khảo" column (Thông tư, Quyết định)
   - BHYT Business Rules Checklist in `domain_knowledge.md`
   - Standard HIS data requirements for the workflow
   
   If a regulation requires data that the UI has NO field for → flag as **missing field** in QnA with high priority. Example: "Chuyển tuyến requires Mã BV chuyển đến but UI has no such field."

10. **[MANDATORY] BHYT Deep Scenario Analysis** (when function involves BHYT):
   Apply the BHYT Business Rules Checklist from `domain_knowledge.md`. For each applicable rule, ask:
   - Does the spec describe this scenario?
   - Does the UI support this scenario (fields, validation, auto-calculation)?
   - If not → raise as QnA question with specific regulatory reference.
   
   Key scenarios to ALWAYS check:
   - Card expiry/invalidity → save behavior
   - Tuyến determination → auto-calculate mức hưởng
   - Cấp cứu override → always đúng tuyến regardless of ĐKKCB
   - Chuyển tuyến → required fields (Mã BV, Giấy chuyển tuyến)
   - Đối tượng switching mid-form → data handling

11. **Requirement Gaps & Impact (DEEP only):** Detail the Issue, its Impact, and the Suggested Question.

12. **QnA Trigger Conditions**: AI MUST raise a question if there is a missing business rule, undefined flow outcome, missing UI behavior (e.g., missing Empty/Loading states), data validation gap, or Security/PII risk.
   - **[MANDATORY] Self-Resolve Decision Tree** (REPLACES old Self-Resolve Gate — use this tree consistently with Risk ↔ QnA Bridge in step 15):
     ```
     PRE-CLASSIFICATION SCAN (MANDATORY — run this BEFORE entering the issue-type tree below):
     Before deciding "create QnA or not", scan IN ORDER:
       1. Spec postcondition / alt flow / exception of THIS UC → does it imply an answer?
       2. Adjacent UCs in the same function group → is there an analogous pattern already defined?
       3. Domain Knowledge catalog (domain_knowledge.md BHYT rules) → is the rule already covered?
       4. Standard UX catalog (Ant Design defaults, REST best practices) → is the pattern universally obvious?

     → If ANY step yields an answer → SKIP QnA, write [N/A] + source of the answer
     → Only when ALL 4 steps produce no answer → proceed to the issue-type tree below

     ⚠️ Danger keyword: "không thấy mô tả rõ ràng" → MUST run Pre-Classification Scan
        before concluding spec is missing.

     Issue type?
     ├─ Spec gap (BA spec missing/unclear/conflicting)
     │  → ALWAYS create QnA (no exception)
     │
     ├─ Regulatory requirement (law/MOH decision applies)
     │  → Tag [Regulatory: <citation>] in TestCase_Material
     │  → ALSO create QnA if BA hasn't acknowledged the regulation in spec
     │     (combine tags: [Regulatory: ...] + [Confirming — pending BA#X])
     │
     ├─ Standard UX universally accepted (no project-specific reason to deviate)
     │  → Tag [Standard UX: <pattern source>] in TestCase_Material, AI Status = `Tự suy luận` (AI-suggested)
     │  → SKIP QnA (don't ask BA about Ant Design defaults, REST 4xx handling, etc.)
     │
     └─ Ambiguous (could be standard but spec/demo shows variation)
        → Create QnA with AI Suggestion proposing standard pattern
     ```
     **Anti-pattern**: ❌ Skipping QnA for spec gaps just because AI can guess. ❌ Asking BA about every Ant Design default.
   - **[MANDATORY] AI Suggestion**: For EACH question, AI MUST simultaneously suggest an answer based on:
     - Domain knowledge (HIS, BHYT, medical workflow, Quyết định 7603/QĐ-BYT)
     - Project context (tech stack NestJS + Ant Design, existing patterns)
     - Common UX patterns (standard form behavior, modal, search)
     - Include **brief reasoning**: explain WHY that suggestion is made
   - **[MANDATORY] Plain Business Language Gate — run before finalizing ANY QnA entry**: Litmus test: *"Would a non-technical BA understand this?"* If NO → rewrite before output.
     - **FORBIDDEN**: Regex notation (`[0-9]+`, `\w+`, `\d{1,3}`), HTTP/API terms ("POST request", "PATCH payload", "endpoint", "response body"), database/code terms ("nullable", "foreign key", "schema", "null constraint"), any technical syntax.
     - **REQUIRED**: Describe behavior from the end-user's perspective on the UI, use exact button/field/screen names as shown in Figma/BA Demo, natural Vietnamese sentences.
     - **Violation → correct examples**:
       - ❌ "Regex `[0-9]{1,3}` validate số nguyên 1–3 chữ số" → ✅ "Ô nhập chỉ chấp nhận số nguyên từ 0 đến 999; nếu nhập chữ cái hoặc ký tự đặc biệt, hệ thống xử lý thế nào?"
       - ❌ "POST /api/config thiếu field x → 400 Bad Request" → ✅ "Khi nhấn [Lưu] mà chưa nhập trường X, hệ thống hiển thị thông báo lỗi gì?"
       - ❌ "nullable FK — có thể để trống nếu config chưa liên kết" → ✅ "Trường này có bắt buộc nhập không, hay được để trống?"
       - ❌ "debounce 300ms trên search input" → ✅ "Hệ thống tìm kiếm ngay khi người dùng nhập hay phải nhấn nút Tìm kiếm?"
     - Applies to **all 3 columns**: `Q&A` (question), `AI Suggestion` (suggested answer), `QA Assessment` (risk description).
   - **[MANDATORY] QnA Entry Content Quality Gate — run after Plain Business Language Gate, before finalizing each entry**:

     **Rule M — Pre-Escalation Check**: Before creating ANY new QnA entry (and its `[Confirming — pending BA#X]` tag), run this 3-point check IN ORDER:
     1. Does the spec postcondition / alt flow / example of THIS UC answer the question?
     2. Does an existing BA#X in QnA_BA already cover this?
     3. Does Standard UX / Ant Design default / Regulatory self-resolve this?
     → If ANY check yields an answer → **DO NOT create QnA**. Write concrete behavior + cite source (`[Standard UX: ...]` / `[Regulatory: ...]` / spec section).
     → Only when ALL 3 fail → proceed.
     Also check before writing follow-up QnA: if a parent BA#X already answered this fully (including BA saying "not needed" or "no") → **do not create follow-up**. BA's decision (including "not needed") IS the spec.

     **Rule A — AI Suggestion Reasoning: NO Figma as reasoning**:
     - Valid reasoning sources: CMR-XXX rule number, Standard UX: Ant Design [component], Business logic, Regulatory citation
     - Figma = **evidence only** (confirms which component is on UI) — NOT the reason for a suggestion
     - ❌ "nên dùng InputNumber vì Figma dùng InputNumber" → ✅ "Ant Design InputNumber block ký tự không hợp lệ tại component level — phù hợp field tài chính để tránh nhập nhầm chữ"
     - ❌ "CMR-033 áp dụng vì Figma không rõ" → ✅ "CMR-033 định nghĩa Textbox là component cho search — khác Input ở chỗ có built-in clear (×) button"

     **Rule B — Q&A column must contain questions, not action-requests**:
     - Every item in the Q&A column MUST be a real question for BA to answer
     - Action-requests ("BA cần cập nhật UC-XXX", "BA cần bổ sung...") are instructions, not questions → move to **Note** column; remove entirely if BA will naturally do this when they answer
     - **Pipe character `|` in Q&A cell is FORBIDDEN** (breaks Markdown table structure). If an action-request unavoidably contains `|` → move the entire item to Note column
     - ❌ Q: "BA cần cập nhật UC-299/UC-301.1 Alt Flow cho click-outside case." → ✅ Note: "Nếu có, BA cần cập nhật UC-299/UC-301.1 Alt Flow."

     **Rule C — Sub-items MUST use `<br>` line breaks (no exceptions)**:
     - When Q&A has multiple sub-items (1)/(2)/(3), MUST separate with `<br>` in table cell
     - Format: `(1) Câu hỏi thứ nhất?<br>(2) Câu hỏi thứ hai?`
     - MD033 linter warning about `<br>` is **expected** — do not "fix" it
     - Applies to QnA_BA table cells AND QnA_Report markdown list format

     **Rule D+J — UX Component QnA: CMR-First Fallback Chain**:
     When writing AI Suggestion about component type (Input vs InputNumber, Select searchable vs non-searchable, DatePicker, etc.):
     1. **Check CMR first** → cite CMR-XXX if applicable
     2. **If no CMR** → "Standard UX: Ant Design [component] — [behavioral reason why this fits the use case]"
     3. **If neither** → "Chưa có CMR — đề xuất follow Ant Design default [specific behavior]. BA confirm có CMR riêng không?"
     Never write "Standard UX" without naming the source (Ant Design / project convention / etc.)

     **Rule I — Ant Design Component Props Check**:
     When AI Suggestion recommends an Ant Design component, verify whether its **default behavior fits the use case**. If not → state the required prop override explicitly:
     | Component | Default behavior that often conflicts | Override to state |
     |---|---|---|
     | InputNumber | Shows +/- step buttons (controls=true) | `controls: false` when step buttons are not needed |
     | Select | Searchable (showSearch=true) | `showSearch={false}` when ≤7 static options |
     | DatePicker | Allows all past dates | `disabledDate` prop when past dates must be blocked |
     | Modal | Closes when user clicks outside (maskClosable=true) | `maskClosable={false}` when form has unsaved state |

     **Rule G — QA Assessment brevity (1-2 sentences max)**:
     - Format: "[Field/screen] đang khác [CMR-XXX / Spec rule] — [cụ thể điểm khác là gì]."
     - DO NOT retell the full conflict history, list all options, or describe background context
     - ❌ Assessment 5+ sentences describing spec origin, data examples, all possible approaches, then finally the conflict
     - ✅ "Toast success 'Thêm mới' đang khác CMR-011 — BA#10 confirm không có từ 'mới', CMR-011 template yêu cầu có."

     **Rule H — Q&A brevity when common rule is already clear**:
     Before finalizing Q&A wording, check: has the AI Suggestion already proposed a clear CMR/Standard UX rule, and is that rule reasonable?
     - If YES → Q&A = one sentence: "Update theo [CMR-XXX] không?" (add at most 1 sub-item for an exception)
     - DO NOT re-list the conflict already described in QA Assessment
     - ❌ Q&A that re-asks every issue already covered in the Assessment as separate numbered sub-questions when they all resolve with one BA decision

     **Rule E — Follow-up QnA must not re-open closed decisions**:
     Before writing AI Suggestion for any follow-up QnA (BA#X.1, BA#X.2, or a new row derived from a parent):
     - Check if a parent BA#X already has an answer
     - If YES → Suggestion MUST **align** with the parent decision, not contradict or raise concern about it
     - ❌ BA#3 confirmed "Mức hưởng KCB không hiển thị trên list" → follow-up suggestion: "Không nên hiển thị vì 6 cột đã đủ" (raises the concern BA already settled)
     - ✅ Follow-up suggestion: "Không có cột 'Mức hưởng KCB' trên list — per BA#3 confirmed. Ask only the specific remaining open detail."

     **Rule N — Split by Ant Design component type**:
     When one candidate QnA row covers multiple fields of **different** Ant Design component types:
     1. List all fields in the question
     2. Group by component type (InputNumber / DatePicker / Select / Input / Textbox / RangePicker)
     3. If > 1 component type → **split into N rows (1 per type)** — each has a different BA decision
     - ✅ InputNumber (step buttons, controls prop) ≠ DatePicker (format, disabledDate) → split
     - ❌ 2 InputNumber fields in the same modal → 1 row (same BA decision)

   - **[MANDATORY] Dual Status**: Each question MUST have both statuses. **Display values use Vietnamese** (per project memory `feedback_skill_language.md`); English names below are skill-internal references only.
     - **BA Status** (VI display | EN internal) — initial snapshot when QnA is created:
       - `Chưa có spec` | `No spec` — spec completely missing this point
       - `Spec chưa rõ` | `Spec unclear` — spec mentions it but is ambiguous
       - `Đã có spec` | `Spec exists` — spec covers it clearly
       - `Đã bổ sung` | `Spec updated` — BA updated spec after the question
       - `Chưa triển khai` | `Coming soon` — feature not yet implemented in current phase
     - **AI Status** (VI display | EN internal) — initial snapshot when QnA is created:
       - `Đã rõ` | `Clear` — AI clearly understands the behavior
       - `Rõ 1 phần` | `Partial` — AI understands part of it, ambiguity remains
       - `Chưa rõ` | `Unclear` — AI cannot infer
       - `Tự suy luận` | `AI-suggested` — AI provides a suggestion based on domain/Standard UX/Regulatory knowledge (do NOT use "Inferred" to avoid collision with deprecated source tag `[inferred]`)
       - `Chưa triển khai` | `Coming soon` — confirmed feature not yet built — skip detailed TCs
     - **New BA Status** — filled after AI QC verifies the BA portal (during BA Answer Sync workflow). This column does NOT overwrite the original BA Status (historical). Values:
       - `AI QC: spec đã update - done` — spec on portal has been updated, no further questions needed (Type A, Type D)
       - `AI QC: spec đã update - cần QnA thêm` — spec updated but gap remains, follow-up QnA created (Type B partial)
     - **New AI Status** — filled at the same time as New BA Status. Reuses existing AI Status vocabulary:
       - `Đã rõ` — fully clear after BA answer + portal verification
       - `Đã rõ 1 phần` — follow-up QnA still exists (BA#X.1) after portal verification
   - **[MANDATORY] Question Quality Gate — Prioritize Business Logic over Generic UX**:
     - Questions about BHYT rules, data integrity, regulatory compliance, cross-state behavior → HIGH VALUE
     - Questions about generic UX (search debounce, reset button, confirmation dialog) → LOW VALUE unless spec explicitly contradicts standard patterns
     - AI MUST ensure at least 50% of questions are Business Logic / Data Handling category
     - If AI finds itself asking mostly UX questions → STOP and re-analyze using BHYT Checklist and Cross-State Analysis

13. **Prioritization**: Identify Top 3 critical scenarios.

14. **Failure Simulation**: Describe 2–3 realistic production failures (e.g., BHYT XML rejection, lost clinical notes, integration timeouts).

15. **[MANDATORY] Risk ↔ QnA Bridge Rule**:
    Risks in TestCase_Material Section 5 MUST be bidirectionally cross-referenced with QnA_BA.
    
    **Outbound rule (Risk → QnA) — DIFFERENTIATED BY SECTION:**
    
    | Section | Nature | Allowed Outbound Tags | Reasoning |
    |---|---|---|---|
    | **5.1 Requirement Gaps** | Spec is silent or incomplete | `→ **QnA: BA#X**` (MANDATORY — no exception) | Spec gap is by definition unanswerable without BA. Cannot be resolved by Regulatory/Standard UX alone — even when external source guides AI Suggestion, BA must still confirm acceptance for the spec. |
    | **5.2 System & Integration Risks** | QC-identified risk based on domain knowledge | `→ **QnA: BA#X**` OR `→ [Regulatory: ...]` OR `→ [Standard UX: ...]` | Risk may be fully resolved by law/standard pattern when spec is silent (e.g., REST PATCH partial-update is universal). When ambiguity exists, escalate to BA. |
    | **5.3 Failure Simulations** | Production-failure scenarios | `→ **QnA: BA#X**` OR `→ [Standard UX: ...]` | Failure handling for generic infrastructure issues (network timeout, retry) → Standard UX. Failure handling involving business decisions (rollback, conflict resolution, data ownership) → BA confirm. **Never `[Regulatory: ...]` alone** — regulations rarely prescribe failure-handling UX. |
    
    **Tag combination is allowed**: `[Regulatory: QĐ 7603 Điều 22] + → **QnA: BA#X**` when law mandates the rule but BA spec must still document it.
    
    Append `→ **QnA: BA#X**` (or multiple BA#X if risk maps to several questions) at the end of each section-5 entry.
    
    **Inbound rule (QnA → Risk):**
    - When a QnA question originates from a risk analysis (not from spec gap), the QnA_BA Note column MUST start with a standardized `[Source: ...]` prefix tag, followed by the human-readable description. Format: `[Source: <tag>] <description>`.
    - **Source tag taxonomy** (machine-grep-able):
      | Origin | Prefix Tag | Example |
      |---|---|---|
      | Section 5.1 Gap | `[Source: Gap-5.1]` | `[Source: Gap-5.1] V2 NEW — gap về thẻ BHYT validity` |
      | Section 5.2 Concurrency Risk | `[Source: Risk-5.2-Concurrency]` | `[Source: Risk-5.2-Concurrency] V2 NEW — Risk từ section 5.2` |
      | Section 5.2 Integration Risk | `[Source: Risk-5.2-Integration]` | — |
      | Section 5.2 Data Integrity Risk | `[Source: Risk-5.2-DataIntegrity]` | — |
      | Section 5.2 Performance Risk | `[Source: Risk-5.2-Performance]` | — |
      | Section 5.2 Field Dependency Risk | `[Source: Risk-5.2-FieldDependency]` | — |
      | Section 5.3 Failure Simulation N | `[Source: FailureSim-5.3-N]` | `[Source: FailureSim-5.3-3] Concurrent edit conflict` |
      | Pure spec gap (no risk origin) | `[Source: Spec-gap]` | `[Source: Spec-gap] Field 22 max length` |
    - This lets future AI/QC `grep "[Source: Risk-5.2-Concurrency]"` to find: "When BA answers BA#X, which risk in section 5 needs re-evaluation?"
    
    **Why this matters:** When BA finally answers a QnA, AI/QC must be able to:
    1. Look up BA#X → find the answer
    2. Trace back via `[Source: ...]` prefix to which risk(s) in section 5 are resolved
    3. Update Expected Results in scenarios linked to those risks
    4. Replace `[Confirming — pending BA#X]` with concrete assertion
    
    Without bidirectional traceability, BA answers get lost → risks remain "open" forever → spec drift.

### Phase 3 — Separated Reporting
1. Output results using the THREE templates located in `.claude/.agents/skills/1. Analysis_Skill/template/`:
   * **Report 1:** `QnA_Report_<TestCaseID>.md` (Based on `Slack_QnA_Template.md`). Extract only the "Clarification List" for Slack.
     - **Naming**: `<TestCaseID>` only — no Vietnamese function name suffix. Example: `QnA_Report_01_M1_F1_01TiepdonBN.md`. Versioned: `QnA_Report_01_M1_F1_01TiepdonBN_v2.md`
     - **IF DEEP:** You MUST group the questions into exactly 5 categories: *Business logic, Data handling, UI/UX, System behavior, Security & PII*.
     - **MANDATORY:** All Q&A entries in QnA_Report MUST have a sequential number `[N]` (resets per file, never per category). Every entry gets both `[N]` AND `[BA#X]` (or `[N/A]`). Format: `- [1] [BA#3] ...` / `- [2] [N/A] ...`. No entry skips `[N]` — including self-resolved, Security "no concern", and section notes.
     - **MANDATORY — Two entry formats depending on tag type (DO NOT mix them):**
       - **`[BA#X]` entries** (question exists in QnA_BA): Use **compact 2-line format**. Full detail (QA Assessment, AI Suggestion, Question, QC Answer, BA Answer) lives in QnA_BA — do NOT repeat it in QnA_Report. QnA_Report entry is a status pointer only:
         ```
         - [N] `[BA#X]` **[BA Status: ... | AI Status: ...]** **Issue**: [one-line issue title]
           → [key resolution in 1 sentence, or "Pending BA confirmation" if unresolved] *(Full detail: QnA_BA #X)*
         ```
       - **`[N/A]` entries** (self-resolved, NOT in QnA_BA): Use **full format** — QnA_Report is the only record for these entries:
         ```
         - [N] `[N/A]` **[BA Status: Đã có spec | AI Status: Đã rõ]** **Issue**: [title]
           - **QA Assessment**: [context + why self-resolved]
           - **AI Suggestion**: N/A — đã rõ. [source: Standard UX: Ant Design / Regulatory: ... / spec postcondition UC-XX]
           - **Priority**: Pn
           - **Question**: N/A (self-resolved)
           - **QC Answer**: [if applicable]
           - **BA Answer**: (N/A — not raised to BA)
         ```
       **Rationale**: `[BA#X]` full detail is already captured in QnA_BA which Tester 2 reads as CRITICAL source. Repeating it in QnA_Report wastes tokens and creates sync drift risk. `[N/A]` has no QnA_BA entry, so full format is required here.
     - **[MANDATORY] CROSS-REFERENCE TAG `[BA#X]`**: Every question in QnA_Report MUST have the tag `[BA#X]` immediately after its sequence number, corresponding to the STT of that question in QnA_BA. Format: `- [N] [BA#X] **[BA Status...]**`. If the question is self-resolved (not in QnA_BA) → write `[N/A]`. Question order in QnA_Report follows Category grouping and MUST NOT be reordered to match QnA_BA STT order.
   * **Report 2:** `TestCase_Material_<TestCaseID>.md` (Based on `TestCase_Material_Template.md`). Contains the IF-THEN rules, Scenario Coverage, Advanced QA Insights, and the Readiness Score.
     - **Naming**: `<TestCaseID>` only. Example: `TestCase_Material_01_M1_F1_01TiepdonBN.md`. Versioned: `TestCase_Material_01_M1_F1_01TiepdonBN_v2.md`
   * **Report 3:** `QnA_BA_<TestCaseID>.md` (Based on `QnA_BA_Template.md`). BA-facing Q&A tracker mapped by Module/Function/UC ID.
     - **Naming**: `<TestCaseID>` only — from `BA/QC-Function list M*.md`. Example: `QnA_BA_01_M1_F1_01TiepdonBN.md`. Versioned: `QnA_BA_01_M1_F1_01TiepdonBN_v2.md`
     - **Content source**: Same questions from Report 1 (QnA_Report), but restructured into table format.
     - **[MANDATORY] QnA_BA ENTRY GATE — Only questions that genuinely need BA to answer go into QnA_BA.**
       QnA_BA is not a log of everything AI considered. Before adding a row, confirm: *"Can this be resolved by reading the spec, an adjacent UC, or applying Standard UX/Regulatory knowledge?"*
       - YES → Do NOT add to QnA_BA. Record as `[N/A]` in QnA_Report only with reasoning.
       - NO → Add to QnA_BA.
       This gate is the downstream enforcement of the Self-Resolve Decision Tree in Phase 2 Step 12. If an entry was accidentally added to QnA_BA and QC later confirms it should not have been raised → **physically delete the row** from QnA_BA (do not convert to N/A status — N/A rows live in QnA_Report only, never in QnA_BA) and renumber remaining STTs. In QnA_Report: convert the corresponding `[BA#X]` entry to `[N/A]` format — **KEEP `[N]` number unchanged**.

       **[MANDATORY] Question types that MUST NOT be added to QnA_BA (use QnA_Report only if a note is needed):**
       | Question type | Reason not to add to QnA_BA | Handling |
       |---|---|---|
       | Default sort/order already documented in Postcondition/Precondition of the same UC | Spec already covers it → self-resolve | `[N/A]` QnA_Report, cite source postcondition |
       | Behavior governed by Ant Design / common browser defaults (e.g., Tab order, focus ring, Enter to submit) | Standard UX — BA has no decision to make | `[N/A]` QnA_Report, cite `[Standard UX: Ant Design]` |
       | Question concerns only cosmetic UX with no business impact (e.g., placeholder text when label is already clear) | BA has no decision to make | `[N/A]` QnA_Report if a note is needed |
       | Question is fully duplicate in logic to another existing QnA_BA row | Redundant | Merge into the original row and add a note |
       | Default sort is obvious from UI (e.g., time-based lists always sort newest first per Standard UX HIS) | Standard UX HIS pattern | `[N/A]` QnA_Report |

       **Anti-pattern (NEVER DO):** ❌ Ask BA about behavior that is already clearly visible in the demo. ❌ Ask BA about behavior that is identical across any HIS system (audit log append-only, search debounce, pagination layout).

     - **[MANDATORY] QnA_BA question order — Group by UC/issue, NOT by order of discovery:**
       - Questions sharing the same UC ID or business issue MUST be placed adjacent to each other in the QnA_BA file.
       - Reason: BA reads and answers by context — related questions placed together reduce context switching and allow simultaneous answers.
       - **Procedure**: After collecting all QnA candidates for a function → **group by UC ID** before assigning STT and writing to file. STT is assigned AFTER sorting is complete — not in order of discovery.
       - **Exception**: V2+ questions are appended at the end of the file (rule `[QnA ORDERING RULE]` Phase 1 step 1.4f) — but within a new V2 batch, questions must still be grouped by UC before appending.
       - **Correct example**: Questions for UC-303.1 (validation Lưu) → BA#1, BA#2, BA#3 adjacent. Questions for UC-303.2 (modal Đóng) → BA#4, BA#5 adjacent. Do NOT intersperse BA#1 UC-303.1, BA#2 UC-303.2, BA#3 UC-303.1.
     - **Table columns (EXACT ORDER)**: `STT | Module | Function/Screen | UC ID | QA Assessment | Q&A | AI Suggestion | QC Answer | AI Status | Priority | BA Status | Assign | BA Answer | New BA Status | New AI Status | Note`
     - **[MANDATORY] STT IS MASTER NUMBERING**: STT in QnA_BA is the official sequence number (master ID) used to cross-reference with QnA_Report via the `[BA#X]` tag. When BA answers by STT, QC/AI traces back to QnA_Report using this tag. DO NOT change STT after publishing — only append new questions at the end.
     - **QA Assessment column**: Copy từ QnA Report — phần context + risk + blocked TC. Giải thích TẠI SAO cần hỏi.
     - **[MANDATORY] QA Assessment ↔ Q&A Consistency Gate**: Before finalizing each QnA entry, verify:
       1. Count the number of **distinct issues/risks** stated in QA Assessment.
       2. Count the number of **distinct questions** in the Q&A column.
       3. If **QA Assessment states N issues but Q&A asks fewer than N** → FAIL. Choose one of:
          - **Split**: Each issue = 1 separate row in QnA_BA (recommended when issues belong to different UCs or have different assignees).
          - **Ask all**: Combine all issues into one clearly multi-part question (only when all issues share the same UC, same assignee, and are directly related).
       4. AI Suggestion MUST provide a recommendation for ALL issues stated in QA Assessment — no issue may be left without a suggestion.

       **[MANDATORY] Rule F — Split condition: "different BA decision", NOT "different issue"** (lesson learned 2026-06-15):
       - **Correct split trigger**: The answer to one question cannot resolve the other → they require **separate BA decisions**
       - **Wrong split trigger**: Two issues exist in the same Assessment but one BA answer covers both → this is `<br>` sub-items in 1 row, not a split
       - **Test before deciding**: "If BA answers this single question, does it resolve ALL issues in Assessment?" → YES = 1 row (use `<br>` for sub-items if needed); NO = split into separate rows
       - **Component type rule (Rule N)**: 2 fields with **different Ant Design component types** in the same question = automatic split (e.g., InputNumber question ≠ DatePicker question — they are different BA decisions)
       - ✅ InputNumber props (controls, step) + DatePicker props (format, disabledDate) → split — different component = different decision
       - ❌ 2 InputNumber fields in the same modal → 1 row — same component type = same decision

       **Violation example**: QA Assessment states 2 risks: (A) whether the [Lưu] button is hidden or disabled when the form is empty, (B) the Edit popup title has a redundant word "mới". But Q&A only asks about (A). → FAIL — must split into BA#X asking (A) and BA#X+1 asking (B), or combine into one clear question: "Is the [Lưu] button hidden or disabled? And does the Edit popup title display correctly?"

       **Anti-pattern**: ❌ QA Assessment is long and states 3 issues but Q&A asks only 1 vague covering question. ❌ AI Suggestion provides a recommendation for only 1 issue while QA Assessment states 2.
     - **AI Suggestion column**: Gợi ý câu trả lời + "(Reasoning: ...)" — BA chỉ cần confirm/reject/adjust.
     - **QC Answer column**: Để trống ban đầu. Khi user trả lời qua chat → điền vào đây.
     - **BA Answer column**: Để trống ban đầu. Khi BA confirm chính thức → điền vào đây.
     - **New BA Status**: Để trống ban đầu. Điền trong BA Answer Sync workflow sau khi AI QC verify BA portal. Giữ nguyên cột BA Status gốc (historical). Values: `AI QC: spec đã update - done` / `AI QC: spec đã update - cần QnA thêm`. (Type D = same as Type A → `done`).
       - **[HARD RULE] Khi tạo QnA mới (bất kể workflow nào)**: New BA Status = `—`. KHÔNG được điền bất kỳ giá trị nào khác.
       - **[HARD RULE] Self-resolved entries (AI tự resolve từ spec/CMR/Standard UX)**: BA Answer = `—`, New BA Status = `—`. Ghi explanation vào cột **Note** thay thế. KHÔNG điền BA Answer chỉ vì AI tìm được answer — BA Answer chỉ điền khi BA người thật trả lời.
     - **New AI Status**: Điền cùng lúc với New BA Status. Values: `Đã rõ` / `Đã rõ 1 phần`.
       - **[HARD RULE] Khi tạo QnA mới (bất kể workflow nào)**: New AI Status = `—`. KHÔNG được điền bất kỳ giá trị nào khác.
     - **AI Status**: Default = `Chưa rõ` (Unclear). Update khi có answer: `Đã rõ` (Clear) / `Rõ 1 phần` (Partial) / `Tự suy luận` (AI-suggested). **Không dùng "Inferred"** — đã rename thành `Tự suy luận` / `AI-suggested` để tránh collision với deprecated source tag `[inferred]`.
     - **BA Status**: Đánh giá spec hiện tại: `Chưa có spec` (No spec) / `Spec chưa rõ` (Spec unclear) / `Đã có spec` (Spec exists). Update khi BA bổ sung: `Đã bổ sung` (Spec updated).
     - **Assign** (cột trong table, sau BA Status):
       **[MANDATORY LOOKUP — DO NOT write generic "BA"]**
       1. Find the latest WBS file in the `BA/` folder (pattern: `WBS*.csv` or `WBS*.xlsx`)
       2. Open the WBS file → find the row corresponding to the Module/Function being analyzed
       3. Read the **PIC** column under the **"Tiến độ BA"** group (NOT PIC DEV BE or PIC DEV FE)
       4. Write the **specific person's name** (e.g., "Xi Ka", "Hoang Doan") in the metadata header
       5. If WBS cannot be found or PIC cannot be determined → write `[WBS not found]` and ask user
       - **Example**: M1.F1 → WBS row "Tìm kiếm bệnh nhân" → PIC (BA) column = "Xi Ka" → Assign = "Xi Ka"
     - **Priority assignment — site-specific (MANDATORY)**:
     
       **Admin site (M10, M11, M12 — catalog config):**
       | Level | Khi nào |
       |---|---|
       | P1 | Spec sai → function **module khác** (ngoài M10/M11/M12) bị sai theo |
       | P2 | Spec conflict internal, ảnh hưởng design TC nhưng không cross-module |
       | P3 | Detail trong Admin module — cần biết nhưng không block execution |
       | P4 | Cosmetic / typo đã fix / nice-to-have |
       
       **User site (M1, M2, M3, M4, M8 — staff workflow):**
       | Level | Khi nào |
       |---|---|
       | P1 | Dữ liệu bệnh nhân sai/mất, tính tiền/BHYT sai, main flow không thực hiện được, hoặc cross-module impact |
       | P2 | Main flow có workaround + business rule BHYT/tuyến/đối tượng chưa rõ + spec conflict ảnh hưởng TC design |
       | P3 | Detail UX không block workflow (search, filter, pagination, toast wording) |
       | P4 | Cosmetic / typo đã fix / non-functional |
     - **UC ID mapping**: Use QC-Function list to identify which UC ID each question relates to. If cross-module, list primary UC and note related UCs.

2. **Calculate Readiness Score**: Evaluate the analysis using the ASTQB/ISTQB-based Readiness Score matrix (out of 100 points):
   - **Logic Coverage (30 pts)**: Are all IF-THEN branches complete? (-10 pts per missing branch).
   - **Explicit Data (25 pts)**: Are data inputs concrete and preconditions clear? (-15 pts if vague).
   - **Risk & Edge Cases (25 pts)**: Are there integration/medical edge cases? (-25 pts if zero edge cases).
   - **QnA Status (20 pts)**: Are all critical questions answered? 
   - **QnA Flagging**: If Medical/BHYT standards are missing, flag as "Future Risk" (Warning) in QnA rather than vetoing.
   - Output the detailed score calculation at the bottom of the `TestCase_Material.md` file.

3. Save ALL THREE reports locally to `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/`.

   **Folder mapping rule** (from task_name prefix → site + module):
   | task_name prefix | site_folder | module_folder |
   |---|---|---|
   | `01_M0_...` | `01_User site` | `M0_Dashboard` |
   | `01_M1_...` | `01_User site` | `M1_Dang ky kham benh` |
   | `01_M2_...` | `01_User site` | `M2_Kham benh` |
   | `01_M3_...` | `01_User site` | `M3_Vien phi va BHYT` |
   | `01_M4_...` | `01_User site` | `M4_Quan ly kho duoc` |
   | `02_M10_...` | `02_Amin site` | `M10_Cau hinh danh muc` |

4. **[MANDATORY] Post-Write Structural Validation**: After saving/editing any output file, verify the following structural markers using **tool evidence** — mental simulation is NOT accepted:

   **Structural checks (all files):**
   - [ ] All required section headers present (per template structure — no section accidentally deleted or merged)
   - [ ] No unfilled placeholders: `"..."`, `"xxx"`, `"[TBD]"` remaining in the file
   - [ ] No empty THEN clause: `[inferred - pending]` without a corresponding `(QnA #X)` reference
   - [ ] No [N/A] entry with `BA Status = "Chưa có spec"` (contradiction — see [N/A FORMAT RULE])

   **[MANDATORY] Consistency tool-checks (run with Grep — required before reporting Done):**

   | Check | Tool command | Pass condition |
   | --- | --- | --- |
   | STT gap in QnA_BA | Grep `^\| \d+` on QnA_BA file → extract all STT numbers → verify sequential, no gap | No missing STT numbers |
   | Duplicate BA# references | Grep `pending BA#` on TestCase_Material → list all BA# cited → verify each maps to a real QnA_BA STT | 0 phantom BA# references |
   | Stale `[Confirming]` | Grep `\[Confirming — pending BA#` on TestCase_Material → for each hit, check if that BA# is now RESOLVED in QnA_BA | 0 resolved BA# still showing as Confirming |
   | Section 5.1 completeness | Grep `→ \*\*QnA: BA#` on TestCase_Material Section 5.1 entries → verify every gap entry has a QnA bridge | 0 Section 5.1 entries without `→ QnA: BA#X` |

   Report evidence inline: e.g., "Grep stale Confirming: 0 hits → Pass". Only report Done when all checks pass.

### **MANDATORY Output Quality Checklist**
- Are all scenarios testable?
- Are edge cases meaningful in a healthcare context?
- Are business rules consistent with BHYT logic?
- **[QnA PLAIN LANGUAGE CHECK]**: Scan all Q&A, AI Suggestion, QA Assessment entries — does any entry contain Regex, HTTP methods, API terms, database terms, or code notation? If YES → rewrite before saving. Final litmus test: *"Would a non-technical BA understand this?"*
- **[Rule K — Entry Validity Pre-flight]**: Before saving QnA_BA, scan ALL entries and **physically delete** any row that meets either condition:
  1. **Conflict self-resolved**: a BA Answer or follow-up note says "resolved" but the entry was never closed, OR the conflict was resolved by CMR update / adjacent UC / Standard UX after the entry was created
  2. **Follow-up with no open question**: parent BA#X already answered fully (including BA saying "not needed" or "no") — no remaining detail is unresolved
  Deleted rows must be **physically removed** (not marked N/A or "Withdrawn") and remaining STTs renumbered. N/A exists only in QnA_Report; QnA_BA must contain no waste rows. For each deleted QnA_BA row: convert its corresponding entry in QnA_Report to `[N/A]` format — **KEEP `[N]` number unchanged**.
- **[QC PERSPECTIVE]**: All Expected Results and Business Rules MUST be written from QC's observable perspective (what user sees on screen: toast messages, list updates, UI state changes). Do NOT write API response fields, HTTP methods, or backend internals — QC does not test API directly. If API testing is needed, it will be a separate test suite.
- **[MVP PLACEHOLDER RULE]**: For features confirmed as "Coming soon" / placeholder in MVP: just state "chưa có action gì" or "placeholder, không cần test chi tiết". Do NOT invent toast messages or disabled states that don't exist.
- **[PRINT/EXPORT COVERAGE]**: Any print/export scenario (PDF, Excel) MUST include: (1) verify file format, (2) verify content of each column/field matches input data, (3) verify layout/template completeness (all sections present).
- **[POPUP/MODAL COVERAGE]**: GUI Scenarios MUST cover ALL popups and modals on the screen — not just the main layout. Each popup needs: layout verification, data display, open/close behavior.
- **[NAMING FROM DESIGN]**: All field names, screen names, button labels in QnA/reports MUST match exactly what appears in Figma design. Do NOT paraphrase or use spec/WBS naming if it differs from design. If Figma shows "Số thẻ BHYT", write "Số thẻ BHYT" — not "Mã BHYT" or "Thẻ bảo hiểm". Note discrepancies between design and spec as observations.
- **[FIELD TYPE CONVENTION — Dropdown vs Combobox]**: When classifying field types in the Component & Data Dictionary:
  - **Dropdown** = fixed list of options, NO search/filter capability. User can only scroll and select. Examples: Loại tuyến (4 fixed options), Khu vực (4 fixed options), Giới tính.
  - **Combobox** = dropdown WITH search/filter capability. User can type to search within the list. Examples: Dân tộc (54 options, searchable), Nơi ĐKKB (thousands of facilities, searchable), Tỉnh/TP, Quận/Huyện, Phường/Xã.
  - **Decision rule**: If the option list is small and fixed (≤10 items, no search needed) → Dropdown. If the list is large or dynamic (loaded from API with many items) AND has search functionality → Combobox.
  - When in doubt, check the BA Demo: if the field has a search input inside the dropdown → Combobox. If it's a simple select → Dropdown.
- **[ACTOR NAME FROM BA PORTAL]**: Role/actor names in Security scenarios and Preconditions MUST use the exact name from the BA Portal's "Actor" field (e.g., "NV Tiếp đón", "Bác sĩ"). Do NOT translate to English (e.g., "reception staff") or use generic terms.
- **[SOURCE ENFORCEMENT]**: Mọi BR và Scenario có được map tới nguồn tài liệu cụ thể không? Nếu trống = FAIL.
- **[CONSISTENCY GATE]**: QnA decisions (RESOLVED) có khớp với nội dung Analysis Detail không? Nếu mâu thuẫn = FAIL.
- **[AI-SUGGESTED GATE]**: Các điểm AI tự suy luận (AI Status = `Tự suy luận` / AI-suggested) đã có QnA/note tương ứng chưa? Source tag tương ứng đã chính xác (`[Regulatory: ...]` / `[Standard UX: ...]` / `[Confirming — pending BA#X]`)?
- **[FIELD COUNT VERIFICATION]**: After extracting fields from BA Demo/Figma, AI MUST count total visible fields on screen and compare with total fields listed in output. If mismatch → re-check the screen. Also verify that section header ranges (e.g., "Fields 8–20") match the actual first and last field numbers in that section's table.
- **[BR ASSERTION RULE]**: Every Business Rule THEN clause MUST be a testable assertion (observable behavior), NOT a question or placeholder text. If the actual behavior is unknown/pending, write the AI Suggestion as the THEN clause and append the appropriate source tag from the **Source-Tag Taxonomy** (see Phase 1 step 1.5e): `[Regulatory: ...]`, `[Standard UX: ...]`, or `[Confirming — pending BA#X]`. Combine tags when applicable (e.g., regulation that BA hasn't documented). Never leave a THEN clause as `[inferred]`, `[QC view]`, or `[inferred - pending]` without a concrete assertion + categorized source tag.
- **[NAMING DISCREPANCY TABLE]**: When field names, button labels, or screen names differ between Spec (BA Portal) and Design (BA Demo/Figma), AI MUST create a "Naming Discrepancies" section in QnA_Report with columns: `Item | Spec name | Demo name | Recommendation`. This ensures Tester 2 uses the correct authoritative names.
- **[QnA ↔ SCENARIO CROSS-REFERENCE]**: Every QnA question that blocks a test scenario MUST include `Blocked TC: SC-XXX` in the QA Assessment. Conversely, every scenario whose Expected Result depends on a pending QnA answer MUST reference `(QnA #X)` in its Expected Result column. This creates bidirectional traceability between questions and test coverage.
- **[RISK ↔ QnA CROSS-REFERENCE — MANDATORY]**: Every entry in TestCase_Material Section 5 (Requirement Gaps / System & Integration Risks / Failure Simulations) MUST end with either:
  - `→ **QnA: BA#X**` (for risks pending BA confirmation — single or multiple BA#X), OR
  - `→ [Regulatory: ...]` / `→ [Standard UX: ...]` (for risks fully resolved by external source — no BA confirm needed)
  
  **Section-specific enforcement:**
  - **5.1 Requirement Gaps** → `→ **QnA: BA#X**` is MANDATORY (no exception). Gaps are spec-silent by definition — only BA can resolve.
  - **5.2 System & Integration Risks** → `→ **QnA: BA#X**` OR `→ [Regulatory: ...]` OR `→ [Standard UX: ...]`. External source alone is acceptable when pattern is unambiguous.
  - **5.3 Failure Simulations** → `→ **QnA: BA#X**` OR `→ [Standard UX: ...]`. Never `[Regulatory: ...]` alone (regulations don't prescribe failure-handling UX).
  
  Conversely, every QnA entry in QnA_BA that originates from risk analysis MUST cite the source in Note column using **standardized prefix tag** (for grep-ability):
  - `[Source: Gap-5.1]` — for Requirement Gap entries (section 5.1)
  - `[Source: Risk-5.2-Concurrency]` / `[Source: Risk-5.2-Integration]` / `[Source: Risk-5.2-DataIntegrity]` / `[Source: Risk-5.2-Performance]` / `[Source: Risk-5.2-FieldDependency]` — for System & Integration Risks (section 5.2)
  - `[Source: FailureSim-5.3-N]` — for Failure Simulations (section 5.3, N = sequence number)
  - `[Source: Spec-gap]` — for QnA originating from spec ambiguity (NOT from section 5)
  
  Format: prepend the prefix tag at the START of Note column, then existing notes after.
  Example: `[Source: Risk-5.2-Concurrency] V2 NEW — Risk từ TestCase_Material section 5.2`
  
  **Verification check before finalizing TestCase_Material**: scan section 5 — if any entry has no `→ QnA: BA#X` and no `→ [Regulatory/Standard UX]` tag → FAIL, must fix before output. Additionally verify section-specific constraints (5.1 must always have BA#X; 5.3 must never have [Regulatory] alone).

- **[[N/A] FORMAT RULE — Self-Resolved Entries]**: When a question is marked self-resolved (not raised to BA), the entry in **QnA_Report only** MUST have exactly:
  - Tag in place of `[BA#X]` = `[N/A]` — self-resolved entries are NOT added to QnA_BA (they would occupy STT slots and confuse BA)
  - `BA Status` = `Đã có spec` ← NEVER "Chưa có spec" or "Spec chưa rõ" — those signal an open gap
  - `AI Status` = `Đã rõ`
  - `Question` = `N/A (self-resolved)`
  - `AI Suggestion` = `N/A — đã rõ. [source: <reason for self-resolve, e.g. Standard UX: Ant Design / Regulatory: QĐ7603 / spec postcondition of UC-XX>]`
  - Reference example: M1_F1 QnA_Report entries 24/25
  - Anti-pattern: ❌ Creating a row in QnA_BA for a self-resolved question — this wastes STT slots. ❌ `[N/A]` entry with BA Status = "Chưa có spec" — semantic contradiction (self-resolved ≠ spec gap).

- **[BA STATUS CONSISTENCY CHECK — Pre-Submit Gate]**:
  Before saving any output file, verify all three conditions:
  □ All `[N/A]` entries → BA Status = `Đã có spec`?
  □ All active QnA entries with BA#X → BA Status = `Chưa có spec` or `Spec chưa rõ`?
  □ No entry where `[N/A]` coexists with a BA Status implying "still a gap"?
  If any check fails → fix before saving output.

---

## ⚠️ File Handling & Incremental Save Rules (MANDATORY)

### 1. Version File Handling Rule
- **IDENTIFY THE LATEST VERSION FIRST** — read all versions to determine which file is the latest (e.g., `_v2.md`, `_v3.md`).
- **ONLY work with the LATEST version** — never edit an older file.
- If the latest version is unclear → ASK user first; do not guess or overwrite.

### 2. Incremental Save Discipline
- After each completed logical unit (e.g., finishing 1 section of an analysis report), AI MUST persist results immediately by updating the file.
- Do NOT wait until the end of the task to save. If the thread dies mid-task, the user still has a baseline to resume from.
- When an out-of-scope finding is discovered (OOS, observations) → update the findings file immediately.

---

## ⚠️ Runtime: QnA Answer Handling (When user answers QnA mid-conversation)

### Distinguishing QC Answer vs BA Answer — ZERO AMBIGUITY RULE:

**Only fill BA Answer column when:**
1. User provides an input file that already has BA Answer column populated (BA filled it)
2. User explicitly states in chat: "BA has answered", "this is from BA", or forwards a BA message

**All other cases** — user answers via chat, says "tentatively agree", "per Figma", "dev is doing it this way", "waiting for BA to confirm" — **ALL go to QC Answer column**.

### If source cannot be determined → ASK USER BEFORE PROCEEDING:
- Ask: "Is this answer from you (QC) or from BA? I need to know which column to fill."
- NEVER guess and fill. Cost of asking = 0. Cost of wrong column = rework entire file.

### Pre-Update Checklist (MANDATORY before every QnA file edit):
1. **Who is answering?** → User chat = QC. BA forward = BA.
2. **Correct column?** → QC Answer or BA Answer?
3. **Does AI Status need updating?** → QC answered clearly = Clear/Partial. Still unclear = keep as-is.
4. **BA Answer left empty?** → MUST remain empty if BA has not answered.

### When filling QC Answer, always perform all 3 steps:
1. Fill content → **QC Answer** column
2. Update **AI Status** → `Clear` / `Partial` (based on clarity)
3. Keep **BA Answer** = empty

### Anti-patterns (NEVER DO):
- ❌ Summarize QC answer and put it in BA Answer
- ❌ User says "tentatively agree" → fill BA Answer (WRONG — this is QC)
- ❌ User references Figma/demo → write "BA confirmed" (WRONG — QC self-referenced)
- ❌ Fill both columns simultaneously when only QC answered
- ✅ All answers from chat → QC Answer. BA Answer stays empty until BA officially replies.

---

## ⚠️ Runtime: Feedback Classification Gate (MANDATORY when QC answers multiple QnA at once)

When the user provides answers to multiple QnA entries simultaneously, AI MUST follow this order — NO EXCEPTIONS:

### Step FC-1 — Classify ALL entries BEFORE syncing any single one:

| Type | Signal from QC | Required action |
|---|---|---|
| **A — Keep/Pending BA** | "Tạm đồng ý, chờ BA" / "QC observe thấy vậy" | Sync QC Answer. Update AI Status → `Rõ 1 phần`. Keep in QnA_BA. BA Status unchanged. |
| **B — AI Suggestion wrong** | "Đáp án đúng là X, không phải Y" | Update AI Suggestion + QC Answer. Flag TestCase_Material BR/SC impact. |
| **C — Should not have been asked** | "AI không nên raise câu này" / "đây là Standard UX hiển nhiên" | **QnA_BA**: physically delete row, renumber STT at end of batch. **QnA_Report**: convert entry to `[N/A]` format + BA Status = `Đã có spec` — **KEEP `[N]` number unchanged. DO NOT delete entry.** |
| **D — QC Fully Resolved** | QC provides complete, certain answer | AI Status = `Đã rõ`. Keep in QnA_BA. |

### Step FC-2 — Report plan to user BEFORE executing:
- List each QnA entry → Type → specific action
- Flag renumber impact: list the actual QnA_BA STT numbers being removed, then state "QnA_BA STT will renumber. QnA_Report [N] numbers stay fixed — no re-indexing."
- Flag which Type B entries require TestCase_Material BR/Scenario updates

### Step FC-3 — Wait for user confirm → DO NOT execute before confirm received

### Step FC-4 — Execute as 1 single batch:
QnA_BA + QnA_Report + TestCase_Material + QnA_BA STT renumber (if Type C exists — QnA_Report `[N]` unchanged)
Use TEMP tag renumber pattern to avoid collision (see `execution_platform_notes.md`).

### Anti-patterns (NEVER DO):
- ❌ Sync each entry sequentially as it is read (no classification, no plan)
- ❌ Delete entries then renumber in a separate pass (2 rounds = wasted tool calls)
- ❌ Skip classification because "only 1–2 entries" — gate is mandatory regardless of count
- ❌ Treat Type C deletions as minor edits and skip the plan-confirm step
- ❌ Renumber `[N]` in QnA_Report when Type C entries are removed — `[N]` is a permanent record ID, never re-indexed
- ❌ Delete a `[N]` entry from QnA_Report for Type C — always convert to `[N/A]` format, never delete the entry

---

## ⚠️ Runtime: QC Answer Sync Rule (MANDATORY when QC answers via chat)

When QC Answer is filled in QnA_BA, AI **MUST** consider syncing back to TestCase_Material — BUT unlike BA Answer Sync, QC Answer is **observation only** and MUST NOT override `[Confirming — pending BA#X]` placeholders. Spec remains the source of truth (see [[feedback-spec-is-source-of-truth]]).

### Trigger:
- User answers QnA question BA#X via chat, AI has just updated QC Answer column
- User fixes/corrects an AI assumption via chat ("không phải vậy, đúng ra là...")

### Sync workflow (5 steps, EXECUTE ALL — do not skip):

**Step 1: Update QnA_BA** *(already done when filling QC Answer column)*
- Fill QC Answer column
- Update AI Status → `Đã rõ` / `Rõ 1 phần` (based on clarity)
- **BA Answer & BA Status STAY UNCHANGED** (no official BA confirmation yet)
- Note column: append `(QC answered YYYY-MM-DD)` for audit trail

**Step 2: Sync QnA_Report**
- Find the matching `[BA#X]` compact pointer entry in QnA_Report
- Update the **status badge** on line 1: `[BA Status: ... | AI Status: ...]` → reflect new AI Status
- Update the **resolution summary** on line 2 (the `→ ...` line): summarize QC's confirmed behavior in 1 sentence. Do NOT copy the full QC Answer text — full detail stays in QnA_BA only.
- **[HARD RULE] KHÔNG được expand 2-line compact entry thành multi-line.** Cấu trúc LUÔN là 2 dòng. Thêm bullet `- **QC Answer**: ...` / `- **Confirmed**: ...` vào entry = SAI.
- Update the Status Summary section if counts change (e.g., increase "Đã rõ qua QC", decrease "Chưa rõ")
- Do NOT change BA Status (remains `Chưa có spec` / `Spec chưa rõ` until BA officially updates spec)

**Step 3: TestCase_Material — `[Confirming]` MUST BE PRESERVED** *(critical rule)*
- Do NOT remove `[Confirming — pending BA#X]` just because QC has answered
- Do NOT rewrite Expected Result into a concrete assertion based on QC Answer
- Reason: QC Answer = observation only; only an official BA spec update removes [Confirming]
- Typical anti-pattern: ❌ "QC confirmed in chat → remove [Confirming]" — this is spec drift

**Step 4: Refine AI Suggestion (optional, safe)**
- If QC Answer provides new insight that improves AI Suggestion accuracy → MAY refine wording of the AI Suggestion in the corresponding BR/Scenario
- Do NOT change assertion structure or remove `[Confirming]` tag
- Append footnote next to AI Suggestion: `(QC observation YYYY-MM-DD: <summary>)`
- Example: BR-30 IF Cấp cứu = true THEN Loại tuyến auto-set "Đúng tuyến" `[Regulatory: QĐ 7603 Điều 22] + [Confirming — pending BA#14]` → QC in chat: "Confirmed disable + auto recalc mức hưởng" → Update AI Suggestion to add "+ disable + recalculate mức hưởng (QC observation 2026-06-01)" — `[Confirming]` tag STAYS

**Step 5: New gaps/risks detection**
- Does the QC Answer generate any follow-up questions or new risks?
- Common example: QC says "Tham khảo design Figma" → AI needs to check Figma → may discover additional fields/behaviors not yet mentioned → create new QnA BA#X+N
- If new gap/risk found → append to QnA_BA + section 5 TestCase_Material per Risk ↔ QnA Bridge Rule (Phase 2 step 15)

### Pre-completion verification:
After completing the sync, AI MUST report to user:
```
QC Answer Sync Report — BA#X (YYYY-MM-DD):
✓ QnA_BA updated (QC Answer + AI Status only — BA Answer/BA Status untouched)
✓ QnA_Report synced (matching by [BA#X] tag)
✓ TestCase_Material: [Confirming — pending BA#X] PRESERVED (no spec drift)
ℹ AI Suggestion refined in: <list of BR/SC IDs> (or "none — QC confirmed AI's existing suggestion")
⚠ New follow-up QnA from QC answer: <list of new BA#X+N> (or "none")
```

### Anti-patterns (NEVER DO):
- ❌ Remove `[Confirming — pending BA#X]` because QC confirmed → spec drift, conflicts with [[feedback-spec-is-source-of-truth]]
- ❌ Move QC Answer content to BA Answer column → wrong column, audit trail broken
- ❌ Update BA Status to `Đã bổ sung` when only QC has answered → BA has not updated spec; cannot claim spec exists
- ❌ Skip step 5 (new gaps detection) → follow-up questions get missed
- ❌ Bulk replace `[Confirming]` across the entire TestCase_Material file → automatic spec drift, very hard to undo

---

## ⚠️ Runtime: BA Answer Re-Analysis Protocol (MANDATORY when user provides a BA answer file)

**Trigger**: User provides a file of BA answers (Excel, CSV, or copy-paste) + requests re-analysis / output update.

**Distinguish from BA Answer Sync Rule**: Sync Rule = after AI already knows the BA answer, syncing back to files. Re-Analysis Protocol = BEFORE that — read raw BA answers + verify portal + classify + present plan. Once this protocol is complete and user confirms → trigger BA Answer Sync Rule.

### Step 1 — Read the BA Answer file
- Read the specified Excel/CSV file (user provides path + sheet + row range)
- For each row: extract `STT`, `Q&A`, `BA Answer`, `BA update spec status`
- If `BA update spec status` ≠ "Đã update" → note it; will not verify portal for this entry

### Step 2 — Verify BA Portal + Full Spec Re-Read (1 pass per UC — MANDATORY)

**Principle**: Read each UC exactly once. In that same read, perform 2 tasks in parallel:

**2A — Verify BA Answers**: For each BA Answer in the Excel file related to this UC:
1. Cross-verify: what BA Excel states vs. what actually exists on the portal
2. Record: (a) spec is accurate as BA stated, (b) spec updated but gap remains, (c) spec update not yet visible

**2B — Full Spec Delta Scan**: In that same UC read, also look for:
- **New BR**: BA added a rule not yet in current output
- **Changed BR**: BA modified a rule — current output has it but it is wrong/outdated
- **New/renamed UC**: new UC appeared or existing UC was renamed (e.g., UC-301.2 → UC-411)
- **New component**: new field/button/filter appeared in spec not yet covered in output
- **Changed flow**: main flow or alt flow was modified

**2B-UI — UI Components Table sweep** (run in the same UC read, no extra navigation):
- Re-read the spec's UI Components table for this UC.
- For EACH row: does the current output's Data Dictionary match? Missing field? Missing constraint?
- For EACH row: does BA's component type still make sense after BA's answers? (apply Step 4A logic — question: *"Component type BA chọn có phù hợp với cách người dùng tương tác không?"*)
- If new gap found → flag as new `[Confirming — pending BA#X]` + auto-create QnA (AUTO-QnA RULE).
- If component type is now confirmed correct by BA Answer → update Data Dictionary; no QnA needed.

**Order**: Read each UC in scope sequentially — 1 navigation per UC, no revisits.

**Anti-pattern (NEVER DO)**:
- ❌ Navigate to each UC separately for verification (Step 2A) then navigate again for delta scan (Step 2B) — wastes tool calls
- ❌ Only read UCs related to BA answers — BA may have updated other parts not mentioned in the Excel

### Step 3 — Classify each entry

After reading Excel + verifying portal, classify each BA#:

| Type | Condition | New BA Status | New AI Status |
|---|---|---|---|
| **A** | BA confirms AI suggestion is correct, spec already updated on portal | `AI QC: spec đã update - done` | `Đã rõ` |
| **B clear** | BA provides new info, spec already updated on portal, no gap remaining | `AI QC: spec đã update - done` | `Đã rõ` |
| **B partial** | BA answers part clearly, portal still has gap or contradiction → follow-up QnA needed | `AI QC: spec đã update - cần QnA thêm` | `Đã rõ 1 phần` |
| **D** | Spec was already fixed on portal before (no additional BA confirmation needed) | `AI QC: spec đã update - done` | `Đã rõ` |

**[CRITICAL] BA's negative/reasoned answer = CLOSED — NOT B partial:**
- If BA explains WHY X is not needed ("field này không hiển thị vì admin có thể xem qua popup UC-411") → this is **Type A/D**, not B partial.
- "BA did not confirm further" ≠ "gap still exists". BA's decision (including the decision "not to implement") IS the spec.
- B partial is only used when BA answers part of the original question and leaves the rest open — NOT when BA has fully explained the reasoning.
- **Anti-pattern**: ❌ BA says "this column is not needed" → AI does not see the column on the portal → classify B partial → create follow-up "confirm if column has been added" — this re-asks what BA already decided.

### Step 4 — Identify follow-up QnA (for Type B partial)

For each entry classified as B partial, apply the **Self-Resolve Decision Tree** (Phase 2 Step 12) first — check spec examples, Figma, Standard UX. Only create a follow-up QnA when no source exists to self-resolve.

**[MANDATORY] Route follow-up QnA to the correct file:**
- UX gap (placeholder text, label wording, display format) → add to **QnA_Report** for QC to self-verify; do NOT add to QnA_BA
- Business rule gap / undocumented spec → add to **QnA_BA**

**[MANDATORY] Follow-up QnA MUST ask something different from the original:**
- Do NOT re-ask what BA already answered — even if AI has not yet seen it on the portal
- Focus on the specific part still open after reading the BA Answer

**Numbering**: `BA#X.1`, `BA#X.2`... (decimal suffix = derived from parent BA#X)

**Required content in new QnA entries**:
- `Note` column: `[Follow-up từ BA#X] <lý do cần hỏi thêm>` — must clearly reference the parent BA#X
- `BA Status` = `Spec chưa rõ` (information exists but gap remains)
- `AI Status` = `Chưa rõ` (pending)
- `Q&A`: a specific question, different from the original BA#X

**Insert position in file**:
- QnA_BA: append immediately after row BA#X (not at end of file)
- QnA_Report: add to the same category as BA#X, tag `[BA#X.1]`

**[MANDATORY] If wrong, delete — leave no trace:**
- If a QnA was created but is later determined unnecessary (QC finds it or AI re-evaluates) → **physically delete the row from QnA_BA** and renumber subsequent STTs. In QnA_Report: convert the corresponding entry to `[N/A]` format — **KEEP `[N]` number unchanged**.
- Do NOT convert to "Withdrawn" or "N/A" status in QnA_BA — N/A exists only in QnA_Report.
- Reason: QnA_BA is a document sent to BA; it should not contain waste or abandoned questions.

### Step 5 — Present classification plan, wait for confirmation

Format for presenting to user — 2 parts:

**Part 1 — BA Answers summary:**
```
SUMMARY
BA#   | Type | New BA Status                        | New AI Status   | New QnA
BA#1  | B    | spec đã update - cần QnA thêm       | Đã rõ 1 phần   | ✓ BA#1.1
BA#2  | A    | spec đã update - done                | Đã rõ           | —
...
```

**Part 2 — Full spec re-read findings (from Step 2B):**
```
SPEC RE-READ DELTA (ngoài BA answers)
- [BR mới] UC-XXX BR-X: <mô tả>
- [UC rename] UC-301.2 → UC-411: <impact>
- [Component mới] <tên field>: <chưa có trong output>
- (Không có delta) → ghi "No additional changes detected"
```

→ **STOP after presenting both parts. Wait for user confirmation before executing any file edits.**

### Step 6 — Execute (after user confirmation)

Execute in a single pass — in order:
1. **QnA_BA**: fill `BA Answer` + `New BA Status` + `New AI Status` + Note for all entries; insert follow-up QnA (BA#X.1) immediately after the parent row; update Priority per site-specific framework
2. **QnA_Report**: sync BA Answer + status summary; add follow-up QnA entries with tag `[BA#X.1]`
3. **TestCase_Material**: for each BA# with a spec update → replace `[Confirming — pending BA#X]` with a concrete assertion + footnote `(Resolved by BA#X on YYYY-MM-DD)`; update BRs/Scenarios per delta from Step 2B; add newly discovered components/BRs
4. **Verify**: run Phase 3 Step 4 Post-Write Structural Validation (all structural checks + 4 consistency tool-checks). Report evidence inline. Only report Done when all checks pass.

**Do NOT execute steps separately** — all 3 files must be consistent after a single pass.

---

## ⚠️ Runtime: BA Answer Sync Rule (subroutine — called from Re-Analysis Protocol Step 6, OR when user pastes a single BA answer directly in chat)

When BA Answer column in QnA_BA is filled → AI **MUST** sync back to TestCase_Material to prevent spec drift.

**Do NOT use this rule when the user provides a BA answer file** — use the **BA Answer Re-Analysis Protocol** above instead. BA Answer Sync Rule applies only to:
- Step 6 of Re-Analysis Protocol (after user confirms the plan), OR
- User pastes a single BA answer directly in chat: "BA đã trả lời câu BA#X: ..."

### Sync workflow (4 steps, EXECUTE ALL — do not skip):

**Step 1: Update QnA_BA**
- Fill BA Answer column
- Update `New BA Status` → `AI QC: spec đã update - done` or `AI QC: spec đã update - cần QnA thêm`
- Update `New AI Status` → `Đã rõ` or `Đã rõ 1 phần`
- **Original BA Status and AI Status DO NOT change** — preserved as historical snapshot
- Note column: append `(BA answered YYYY-MM-DD)`

**Step 2: Sync QnA_Report**
- Find the matching `[BA#X]` compact pointer entry in QnA_Report
- Update the **status badge** on line 1: `[BA Status: ... | AI Status: ...]` → `Đã bổ sung` / `Đã rõ`
- Update the **resolution summary** on line 2 (the `→ ...` line): replace "Pending BA confirmation" with 1-sentence summary of what BA confirmed. Full BA Answer stays in QnA_BA only.
- **[HARD RULE] KHÔNG được expand 2-line compact entry thành multi-line dù BA đã trả lời đầy đủ.** Cấu trúc LUÔN là 2 dòng — dù entry đã Resolved hay vẫn Pending. Expand = tạo duplicate content với QnA_BA + phá format. Violation ví dụ: thêm bullet `- **BA Answer**: ...` / `- **Confirmed**: ...` / `- **Scope giảm**: ...` vào một `[BA#X]` entry = SAI.
- Update the Status Summary section if needed (decrease "Chưa có spec" count, increase "Đã bổ sung" count)

**Step 3: Sync TestCase_Material — Source Tag Replacement** (this is the step most often forgotten)
- `grep "[Confirming — pending BA#X]"` across the entire TestCase_Material file
- For EACH match:
  - Replace the placeholder with a concrete expected result from the BA Answer
  - Append footnote: `(Resolved by BA#X on YYYY-MM-DD)`
  - **Note**: If the tag is combined (`[Regulatory: ...] + [Confirming — pending BA#X]`) → keep the `[Regulatory: ...]` tag (it documents the legal basis and remains applicable after BA confirms); only remove the `+ [Confirming — pending BA#X]` part and replace the content.

**Step 4: Sync TestCase_Material — Section 5 Re-evaluation**
- `grep "→ **QnA: BA#X**"` in section 5 (Gaps / Risks / Failure Sim)
- For each matching entry: mark it `(Resolved BA#X — see TC update on YYYY-MM-DD)` and verify the corresponding BR/Scenario has been updated
- If BA answer changes expected behavior → flag affected TC scenarios for rework

### Pre-completion verification:
After completing the sync, AI MUST report to user:
```
BA Answer Sync Report — BA#X (YYYY-MM-DD):
✓ QnA_BA updated (BA Answer + BA Status: Đã bổ sung)
✓ QnA_Report synced (matching entry by tag [BA#X])
✓ TestCase_Material:
  - X scenarios với [Confirming — pending BA#X] → resolved (list IDs)
  - Y BR rules updated
  - Section 5.X risk entry marked resolved
⚠ Action needed (if any): <list>
```

### Anti-patterns (NEVER DO):
- ❌ Update QnA_BA only → forget TestCase_Material → silent drift
- ❌ Replace `[Regulatory: ...]` tag when BA confirms (regulatory tags stay forever as legal documentation)
- ❌ Skip footnote `(Resolved by BA#X on YYYY-MM-DD)` (audit trail is mandatory)
- ❌ Bulk replace without re-reading context → may incorrectly overwrite related scenarios
