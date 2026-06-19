# TEST CASE OUTPUT TEMPLATE (Mini HIS) — Horizontal Table Format

You must strictly follow this Markdown TABLE format for EVERY test case generation.
Output uses `<br>` for line breaks within table cells.

---

## FILE HEADER (Required at top of every TC file)

```markdown
# TEST CASES: [TestCaseID] — [Feature Name] ([UI/Function])

> **Module**: [Module ID] — [Module Name]
> **Feature**: [Feature ID]. [Feature Name]
> **UC IDs**: [UC-001, UC-002, ...]
> **UC Map**: UC-001 = [Tên UC 1], UC-002 = [Tên UC 2]
> **Date**: [YYYY-MM-DD]
> **Source**: [Primary sources used]
> **Note**: [Any important clarification or QC-confirmed override]
```

---

## #1. OVERALL SUMMARY

```markdown
### Priority Summary (Overall)
| Priority | UI | Functional | Total |
|---|---|---|---|
| P0 | 0 | 0 | 0 |
| P1 | 0 | 0 | 0 |
| P2 | 0 | 0 | 0 |
| P3 | 0 | 0 | 0 |
| P4 | 0 | 0 | 0 |
| **Total** | **0** | **0** | **0** |
```

> ℹ️ Overall Summary uses 3 data columns (UI / Functional / Total) for a cross-cutting view. It is the ONLY summary table in the output.

---

## #2. UI TCs

TC table grouped by sub-headers:


- `#### 2.1. Screen Initialization & Default State`
- `#### 2.2. Static & Responsive UI`
- `#### 2.3. Component States (Hover, Focus, Disabled, Error)`

---

## #3. FUNCTION TCs

TC table grouped by sub-headers.

Sub-headers đặt tên theo **feature area (WHAT)**, không phải test type (HOW).
- Phân tích màn hình → liệt kê feature areas → mỗi area là 1 sub-header
- Case của từng TC là mô tả ngắn gọn *what exactly is being verified* — không có prefix, không có type tag

| Loại màn hình | Ví dụ sub-headers |
|---|---|
| List screen | `Filter & Search` / `Hiển thị dữ liệu bảng` / `Phân trang` / `Quyền truy cập` / `Xử lý lỗi` |
| Create/Edit screen | `Form nhập liệu` / `Logic nghiệp vụ` / `Submit & Result` / `Quyền truy cập` / `Xử lý lỗi` |
| Mixed screen | Kết hợp tùy thuộc feature areas thực tế |

`#### Performance` — chỉ thêm nếu Performance Self-Assessment Gate (Rule 22 trong Skill) trả về REQUIRED. Nếu bỏ, ghi rõ lý do trong file header.

---

## TABLE FORMAT (11 Columns — MANDATORY)

Each sub-header section contains ONE table with this exact structure:

```markdown
| TC_ID | Section | Scenario | Case | Pre-condition | Test Steps | Test Data | Expected Result | Priority | Notes | Source |
|---|---|---|---|---|---|---|---|---|---|---|
| [ID] | [Section] | [Scenario] | [Case] | [Pre-condition] | [Steps] | [Data] | [Expected] | [P0-P4] | [Notes] | [Source] |
```

### Column Definitions:

| # | Column | Content | Format Rules |
|---|---|---|---|
| 1 | TC_ID | Unique identifier | Format: `[ModuleID]_[UC_ID]_[PREFIX]_[L1].[L2].[L3]` |
| 2 | Section | Feature area being tested (WHAT) | Free name — short Vietnamese/English. e.g., `Filter & Search`, `Default State`, `Phân trang` |
| 3 | Scenario | Specific business context | NOT generic ("Main Flow"). Must describe the exact scenario |
| 4 | Case | Short, specific description of what is being verified | Action verb + component + property. No category prefix. |
| 5 | Pre-condition | Setup state before test | Concise, reusable conditions |
| 6 | Test Steps | Atomic actions (1 step = 1 action) | Numbered: `1. Action`<br>`2. Action` |
| 7 | Test Data | Explicit values used | `Field = Value` format. Use `—` if none |
| 8 | Expected Result | Observable, specific outcomes | Numbered: `1. Result`<br>`2. Result` |
| 9 | Priority | P0 / P1 / P2 / P3 / P4 | P0=critical/security, P1=main flow, P2=validation, P3=GUI/edge, P4=deferred. Full definition: Skill.md Rule 4. |
| 10 | Notes | Optional metadata | Risk, automation info, `[Confirming]` flag, compliance tag |
| 11 | Source | Traceability tag — reference only, read when needed | See Source format below. **Always last column.** |

### Line Breaks Within Cells:

Use `` `<br>` `` to separate multiple lines within a single cell:

```markdown
| M1_UC002_FC_1.1.1 | Tạo mới BN | Tạo mới BN — BHYT đúng tuyến | Lưu BN mới với BHYT hợp lệ | User role `reception staff`, mở màn hình "Tiếp đón BN" | 1. Nhập [Họ tên] = "Nguyễn Văn A"`<br>`2. Chọn [Giới tính] = "Nam"`<br>`3. Nhập [Mã thẻ BHYT] = "DN4010012345678"`<br>`4. Click [Lưu] | Họ tên = Nguyễn Văn A`<br>`Mã thẻ BHYT = DN4010012345678 | 1. Hiển thị toast "Lưu thành công"`<br>`2. Mã BN được sinh tự động`<br>`3. BN xuất hiện trong danh sách tiếp nhận | P1 | — | Spec (UC-002) 2026-05-20 |
```

---

## TC_ID NAMING CONVENTION

Format: `[ModuleID]_[UC_ID]_[PREFIX]_[L1].[L2].[L3]`

| Component | Mô tả | Ví dụ |
|---|---|---|
| ModuleID | Từ QC Function List | `M1`, `M2`, `M10` |
| UC_ID | Từ BA Spec. Sub-UC dùng underscore (không dùng dot) | `UC189`, `UC299`, `UC301_1` |
| PREFIX | `UI` cho UI TCs, `FC` cho Function TCs | `UI`, `FC` |
| L1 | Sequence number của UC trong file, bắt đầu từ 1. Không liên quan module number. | UC đầu tiên = 1, UC thứ hai = 2 |
| L2 | Sequence number của Section trong UC, bắt đầu từ 1. **Không mang semantic** — chỉ là thứ tự. | Section đầu = 1, Section thứ hai = 2 |
| L3 | Sequence number của TC trong Section, bắt đầu từ 1. Reset khi sang Section mới. | 1, 2, 3... |

> ⚠️ L1 không phải module number: `M10_UC189_FC_10.1.1` là SAI. Đúng: `M10_UC189_FC_1.1.1`
> ⚠️ L2 không encode ý nghĩa section. Section "Filter & Search" và "Form nhập liệu" đều có thể là L2=1 nếu là section đầu tiên của UC đó. Ý nghĩa đọc từ cột Section, không đọc từ con số.
> ⚠️ Migration: File v2 hiện có có thể dùng L1=1 cho mọi UC (convention cũ). Convention mới áp dụng từ file mới tiếp theo. Không retroactively đổi TC_IDs trong file cũ.

**Ví dụ cho file có UC189 (List screen) và UC299 (Create modal):**
- `M10_UC189_FC_1.1.5` — UC189 (L1=1), Section 1 "Filter & Search", TC #5
- `M10_UC189_FC_1.2.2` — UC189 (L1=1), Section 2 "Hiển thị dữ liệu bảng", TC #2
- `M10_UC189_FC_1.3.1` — UC189 (L1=1), Section 3 "Phân trang", TC #1
- `M10_UC299_FC_2.1.3` — UC299 (L1=2), Section 1 "Form nhập liệu", TC #3
- `M10_UC299_FC_2.2.1` — UC299 (L1=2), Section 2 "Submit & Result", TC #1

---

## NAMING CONVENTIONS (CRITICAL)

- UI Components (button, textbox, dropdown, **column header**, icon): Use square brackets `[ ]`
  - Column headers are UI components — they MUST use `[]`. Applies to ALL TC fields (Scenario, Case, Steps, Expected Result, Pre-condition, Notes).
  - e.g., `Click [Lưu]`, `Enter [Mã Bệnh Nhân]`, `cột [Thao tác]`, `cột [Ngày hiệu lực]`, `cột [Lương cơ sở (VNĐ)]`
  - ❌ Wrong: `cột Thao tác`, `cột Trạng thái`  ✅ Correct: `cột [Thao tác]`, `cột [Trạng thái]`
- Screen/Popup/Modal: Use double quotes `" "` (e.g., `Màn hình "Chi tiết Bệnh nhân"`)
- Messages/Errors: Use double quotes `" "` (e.g., `Hiển thị lỗi "Sai định dạng"`)

---

## SOURCE (TRACEABILITY) FORMAT

| Source Type | Format | Purpose |
|---|---|---|
| From BA Spec | `Spec (UC-002) 2026-05-20` | Track which spec version TC was written against |
| QC confirmed | `QC confirmed QnA #1` | Behavior confirmed by QC observation |
| AI suggestion | `AI Suggestion QnA #6` | Not yet BA confirmed — may change |
| Regulatory | `Domain knowledge (QĐ 7603 Điều 22)` | Healthcare regulation reference |
| From prototype | `BA Demo (observed)` | Observed on BA interactive prototype |

---

## SCENARIO NAMING EXAMPLES

| Good (Specific business context) | Bad (Generic) |
|---|---|
| Tạo mới BN — BHYT đúng tuyến | Main Flow |
| Chọn BN cũ từ popup → form auto-fill | Happy path |
| Thẻ BHYT hết hạn — submit form | Validation |
| Double-click [Lưu] khi mạng chậm | Edge Case |
| User không có role NV Tiếp đón truy cập URL | Permission test |

---

## [Confirming] HANDLING

For items pending BA confirmation — still write TC fully. Only mark specific Expected Result points with `[Confirming]` prefix:

```
Expected Result:
1. Hiển thị toast thông báo lưu thành công
2. [Confirming] Khối BHYT collapse khi đổi Đối tượng sang Viện phí
3. Data trên form không bị mất
```
