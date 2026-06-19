# UI/UX Testing Standards - Project [Mini HIS]

## 1. Project Overview
- **Project name**: Mini HIS (Healthcare system)
- **Primary Language**: Vietnamese
- **Testcase output format**: Markdown table (.md) — 10 fixed columns + Phase conditional

### Language Rules
- Testcase content must be written in Vietnamese.
- **Bilingual Labels**: If a technical term is ambiguous, use the format `Vietnamese (English)` to clarify.
- Exact system messages should keep Vietnamese text.
- Example: `Lưu (Save)` or `Hủy (Cancel)`.

---

## 2. Agent Roles
| Agent | Role | Main Responsibility |
| --- | --- | --- |
| Tester 1-Analyzer | Requirement Analyst | Analyze requirement, apply 3C filters, identify bugs based on HIS historical bugs, create Clarification QnA and Analysis Detail. |
| Tester 2-TCs-creator | Test Case Generator | Apply Atomic thinking, cover Functional/Negative/Edge cases, and generate Markdown table test cases. |

---

## 3. Requirement Analysis Rules

### 3.1. General Rules
- GUI should follow Figma/design first.
- **Field & Screen Naming**: Field names (label), screen names, button text MUST be taken directly from the Figma design — not from spec text, WBS, or AI's own wording. If design shows "Số thẻ BHYT" then use exactly "Số thẻ BHYT", not "Mã thẻ bảo hiểm". If naming differs between Figma and spec, use Figma and note the discrepancy.
- Function logic should follow legacy web / old system if docs/spec do not describe behavior clearly.
- If message list and legacy mismatch: record the mismatch in the report and state which source should be prioritized.

### 3.2. Requirement Quality Report
- Requirement analysis must clearly list: `Missing`, `Ambiguous`, `Conflict`, `Discrepancy`.
- If there is an important assumption, record it explicitly.

---

## 4. Validation And Business Rules

### 4.1. General Validation
- Trim leading/trailing spaces before validating input, unless requirement says otherwise.
- For create/update API payloads, send user-entered text after trimming leading/trailing spaces by default.
- If submit has multiple invalid fields, show all relevant errors together by default (Inline validation).

### 4.2. Message Display Type
- Always identify the display type: `inline message`, `toast message`, `toast success message`, `toast error message`, `alert message`.
- Do not write vague wording like: `message is displayed` or `error is shown`.

### 4.3. API Failure And Network Failure
- Treat backend API response failure and lost-network/no-response as separate testcase scenarios.
- Verify the exact confirmed message behavior, including display type and text if any.

### 4.4. Edit Screen Default Value Rules
- For an Edit screen, GUI testcase should use `Default: loaded from saved data`.
- Do not write `Default: blank` for fields that load saved data.

---

## 5. Testcase Writing Rules

### 5.1. Atomic Steps & Explicit Data
- **Atomic Steps**: 1 Step = 1 Action. Do not combine "Input username and password and click login" into one step.
- **Explicit Data**: Use explicit, hardcoded values for testing (e.g., `PatientCode=BN123`, `RecordID=456`) instead of "Input valid data".

### 5.2. Priority
- Always use `P0 / P1 / P2 / P3 / P4`.

| Level | Khi nào dùng |
|---|---|
| `P0` | Verify Business Rules trực tiếp, high severity risk (data loss, security, core function broken), core refactor verification |
| `P1` | Main flow, critical flow, critical business rule, data integrity, key blocker, cases từ RD/spec gốc |
| `P2` | Important validation, alternative flow, key error handling, QA bổ sung |
| `P3` | Common GUI, cosmetic, rare edge case |
| `P4` | Deferred — not run in current sprint, out-of-scope tạm thời |

### 5.3. Text Format
- Ensure all popup titles, buttons, and fields strictly match the UI design.

### 5.4. Placeholder / Option List / Message Text
- Write the exact placeholder if confirmed.
- For dropdown/radio/checkbox: if values are dynamic, write `loaded from API`.

### 5.5. Default Selected Value
- If a radio/control already has a default selected value when opening the screen/popup, do not write mandatory cases based on leaving it blank unless UI allows clearing it.

---

## 6. List Screen Rules
- For a `List` screen, testcase should cover only list-level logic.
- Actions opening separate popup/screen should cover row selection condition and list-level validation. Do not test deep logic inside the opened popup/screen here.

### 6.1. Concurrency And Network Failure
- For list screens with data-changing action (e.g., Delete), add testcase for:
  - Two tabs / two browsers concurrency.
  - Network/backend error.

---

## 7. Mini HIS Domain Specific Rules (Historical Awareness)
- **Data Synchronization**: Always check if data saved in Reception immediately reflects in Doctor's Examination screen.
- **Concurrent Edits**: Always test behavior when two roles (e.g., two Doctors, or Doctor and Nurse) try to edit the same record simultaneously.
- **Integration Reliability**: Verify UI fallback behavior when external integrations (LIS, PACS, National Pharmacy) are delayed or offline.
