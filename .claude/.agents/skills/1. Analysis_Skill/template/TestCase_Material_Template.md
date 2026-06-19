# TestCase Material: [Task Name]
*(Raw material for Test Case Generation Step)*

## 1. Preconditions & Test Data Requirements 🔑
### 1.1 System Preconditions & Roles
- **Preconditions:** [State required before testing, e.g., Patient must be admitted]
- **Permissions/Roles:** [Roles that can access this feature, e.g., Head Nurse, Doctor]

### 1.2 Test Data Requirements
*(Specific data states required to execute the scenarios)*
- [ ] [e.g., "Patient profile with active Health Insurance"]
- [ ] [e.g., "Prescription with out-of-stock medication"]

## 1B. Business Context & Valid Test Data Guide 📖
*(MANDATORY: Explain business meaning so QC understands WHY certain data is correct/incorrect. Critical for regulatory functions like BHYT, billing, pharmacy.)*

**When to include this section:** Always include when the function involves:
- Financial/billing parameters (BHYT rates, pricing, insurance thresholds)
- Regulatory compliance (data must match government regulations)
- Cross-module impact (config here affects calculations elsewhere)
- Domain-specific logic that QC may not intuitively understand

**When to SKIP:** Simple CRUD catalogs with no business logic (e.g., list of room names, staff list).

### Business Overview
*(1-3 sentences: What does this function configure? Why does it matter? What breaks if data is wrong?)*

### Field Business Explanation

| Field | Business Meaning | Legal Basis | Valid Test Data | Invalid Test Data |
|---|---|---|---|---|
| [Field name] | [What it means in business context] | [Regulation/Law if applicable] | [Actual correct value + source] | [Values to test negative cases] |

### Auto-calculation / Derived Values (if any)
*(Explain formulas QC needs to verify. Show calculation with real numbers.)*

### Impact Analysis (if config is wrong)

| Misconfiguration | Consequence | Affected Modules |
|---|---|---|
| [What could go wrong] | [Business impact on patients/billing] | [Which modules break] |

### Reference Data for Test (if applicable)
*(Historical values, regulatory timeline, lookup tables QC can use to create realistic test data)*

## 2. Component & Data Dictionary Matrix 🧩
*(MANDATORY: Analyze UI/API fields to provide data for boundary/negative testing)*
*(Split thành sub-section per UC ID — 1 bảng per UC. Nếu function chỉ có 1 UC → dùng 2.1 duy nhất. Fields xuất hiện ở nhiều UC → lặp lại ở từng sub-section, ghi rõ constraint khác biệt nếu có.)*

### 2.1 [UC-XXX — Tên màn hình / mode]

| # | Field / Component | Type | Mandatory? | Constraints / Validation Rules | Source |
| --- | --- | --- | --- | --- | --- |
| 1 | [e.g., Patient ID] | [e.g., Text] | [Yes/No] | [e.g., Exactly 10 chars, Alphanumeric] | `[from spec]` |
| ... | ... | ... | ... | ... | ... |

### 2.2 [UC-YYY — Tên màn hình / mode]

| # | Field / Component | Type | Mandatory? | Constraints / Validation Rules | Source |
| --- | --- | --- | --- | --- | --- |
| 1 | [Field name] | [Type] | [Yes/No] | [Constraints] | `[from spec]` |
| ... | ... | ... | ... | ... | ... |

## 3. Business Rules (IF-THEN Format & RTM) 📐
*(MANDATORY: Use the **Source-Tag Taxonomy** below for non-spec sources. Never use bare `[inferred]` or `[QC view]`.)*

**Source-Tag Taxonomy:**
- `[from spec]` — directly from BA spec / BA Portal
- `[Regulatory: <law/regulation>]` — Vietnamese law / MOH (e.g., `[Regulatory: QĐ 7603/BYT Điều 22]`, `[Regulatory: TT 56/2017]`)
- `[Standard UX: <pattern>]` — universal UX pattern (e.g., `[Standard UX: Ant Design Form validation]`)
- `[Confirming — pending BA#X]` — needs BA confirmation; MUST cite a QnA_BA STT
- **Combine** when applicable: `[Regulatory: ...] + [Confirming — pending BA#X]`

| BR ID | Rule (IF-THEN Format) | Source | Traceability / Note |
|---|---|---|---|
| BR-01 | IF [Condition] THEN [System Behavior] | `[from spec]` | AC#X |
| BR-02 | IF ... THEN [concrete assertion from AI Suggestion] | `[Confirming — pending BA#14]` | Needs BA confirm |
| BR-03 | IF Cấp cứu = true THEN Loại tuyến auto-set "Đúng tuyến" | `[Regulatory: QĐ 7603/BYT Điều 22]` + `[Confirming — pending BA#14]` | Law mandates; BA spec missing |
| ... | ... | ... | ... |

## 4. Scenario Coverage 🎯
*(Focus heavily on coverage, NOT step-by-step detail. Use the same Source-Tag Taxonomy as Section 3 in the Source column — this replaces the old "Linked BR" column since Source already traces back to BRs.)*

### 4.1 GUI & Function Scenarios
| Scenario ID | Category | Scenario Description | Expected Result | Priority | Source |
|---|---|---|---|---|---|
| SC-01 | Main Flow | [Description] | [Specific output/UI state] | P1 | `[from spec]` |
| SC-02 | Validation | [Description] | [Specific error message] | P2 | `[from spec]` |
| SC-03 | Error Handling| [Description] | [Specific error state/fallback] | P2 | `[Standard UX: REST error handling]` |
| SC-04 | Security | [Description] | [Specific restriction/alert] | P1 | `[from spec]` |
| SC-05 | Edge Case | [Description] | [Specific outcome] | P3 | `[Confirming — pending BA#X]` |

*(Must include Negative & Edge cases)*

## 5. Risk Assessment & Gap Analysis (DEEP Mode Only) ☢️
*(Critical for HIS: Focus on data integrity, concurrency, and system integration)*

**[MANDATORY] Risk ↔ QnA Bridge:** Every entry below MUST end with either:
- `→ **QnA: BA#X**` (for risks pending BA confirmation), OR
- `→ [Regulatory: ...]` / `→ [Standard UX: ...]` (for risks fully resolved by external source).

**Section-specific enforcement:**
- **5.1 Gaps** → `→ **QnA: BA#X**` is MANDATORY (no exception — spec gap requires BA).
- **5.2 Risks** → `→ **QnA: BA#X**` OR `→ [Regulatory: ...]` OR `→ [Standard UX: ...]`.
- **5.3 Failure Sim** → `→ **QnA: BA#X**` OR `→ [Standard UX: ...]` (never `[Regulatory]` alone).

This ensures every risk reaches BA via QnA file (not buried as internal QC notes), and BA answers can be traced back to specific risks.

### 5.1 Requirement Gaps (Missing Logic)
- **Gap 1:** [e.g., Spec mentions deleting prescription but doesn't define billing rollback behavior] → **QnA: BA#02**.
- **Gap 2:** [e.g., Missing max length for Medical History field] → **QnA: BA#33** (proposed 1000 chars in AI Suggestion).

### 5.2 System & Integration Risks
- **Concurrency Risk:** [e.g., Two doctors edit same record simultaneously] → **QnA: BA#30** (Optimistic lock proposal).
- **Integration Risk:** [e.g., Pharmacy module impact when drug code changes] → **QnA: BA#XX**.
- **Data Integrity/Audit Risk:** [e.g., Patient status changes logged in Audit Trail?] → **QnA: BA#XX**.
- **Performance Risk:** [e.g., Search realtime với DB >100k records] → **QnA: BA#29**.

### 5.3 Failure Simulations
1. **[Failure scenario]:** [Description] → Expected: [behavior] → **QnA: BA#XX** (or `→ [Standard UX: ...]` if no BA confirm needed).
2. **[Failure scenario]:** [Description] → Expected: [behavior] → **QnA: BA#XX**.

## 6. Readiness Score Evaluation 📊
*(Calculated by Tester 1 to determine if this analysis is ready for Tester 2)*
- **Logic Coverage (Max 30):** [Score] - [Reason]
- **Component & Data Explicit (Max 25):** [Score] - [Reason]
- **Risk & Edge Cases (Max 25):** [Score] - [Reason]
- **Traceability & QnA Status (Max 20):** [Score] - [Reason]
- **TOTAL SCORE:** [Total]/100
- **Decision:** [PASS to Tester 2 / REJECT - Need more info / WARNING - Missing Medical Specs]
