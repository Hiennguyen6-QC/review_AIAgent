---
description: Quy trình thực thi test case 5 phase, cross-check 3 nguồn primary (Spec + Analysis + TCs), tuân thủ Mode + Execution Scope, có checkpoint resume và evidence retention.
---

# EXECUTE TEST WORKFLOW (EXECUTE_TEST_FLOW)

## 1. YÊU CẦU ĐẦU VÀO (Inputs)

| # | Nguồn | Đường dẫn | Bắt buộc |
| :---: | :--- | :--- | :---: |
| 1 | **Original Spec** | `https://docs.sota-his.com/docs/business/usecases` (BA Web Portal) | ✅ Primary |
| 2 | **Requirement Analysis** | `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/TestCase_Material.md` (hoặc versioned `_v2.md`) | ✅ Primary |
| 3 | **Test Cases** | `Output_QC/2. Test case/<site_folder>/<module_folder>/[TestCaseID]_TCs.md` | ✅ Primary |
| 4 | **Q&A clarifications** | `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/QnA_Report.md` | ⚠️ Supplementary |
| 5 | **System Exploration** | `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/Explore CMS V*.md` (nếu có) | ⚠️ Supplementary |
| 6 | **TC Review report** | `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/TCs_Review_<TicketID>.md` | ⚠️ Supplementary |
| 7 | **Bug history** | `Document refer/Bug list*.csv` | ⚠️ Supplementary |

**Folder mapping rule** (từ task_name/TicketID prefix → site + module):
| prefix | site_folder | module_folder |
|---|---|---|
| `01_M0_...` | `01_User site` | `M0_Dashboard` |
| `01_M1_...` | `01_User site` | `M1_Dang ky kham benh` |
| `01_M2_...` | `01_User site` | `M2_Kham benh` |
| `01_M3_...` | `01_User site` | `M3_Vien phi va BHYT` |
| `01_M4_...` | `01_User site` | `M4_Quan ly kho duoc` |
| `02_M10_...` | `02_Amin site` | `M10_Cau hinh danh muc` |

**Skill:** Load `.agents/skills/4. Execute_test_Skill/SKILL.md`
**Template:** `.agents/skills/4. Execute_test_Skill/template/Execution_Result_Template.md`

---

## 2. PARAMETER GATES (Trước khi bắt đầu Phase 1)

Tester 4 PHẢI confirm 2 parameter sau từ user. Nếu user không cung cấp trong prompt → ASK USER trước khi run.

| Parameter | Giá trị hợp lệ | Default | Hành vi nếu missing |
| :--- | :--- | :--- | :--- |
| **Mode** | `NORMAL (Full)` / `LIGHT (Smoke)` | (none) | STOP — Ask user "Run NORMAL (Full) hay LIGHT (Smoke)?" |
| **Execution Scope** | `READ-ONLY` / `SANDBOX` / `PAIRED` | `READ-ONLY` | Notify user "Mặc định READ-ONLY, CRUD TCs sẽ skip. Confirm hoặc đổi?" |

Optional parameter từ user:
- **Build version / commit hash** — nếu không cung cấp → để trống trong report (không tự verify)
- **Platform list** — nếu không nêu → mặc định Chrome only

---

## 3. CÁC BƯỚC THỰC THI BẮT BUỘC (5 PHASES)

### PHASE 1 — Context Ingestion & Cross-Check

1. **Liệt kê files** dùng tool `Glob` với các pattern sau (KHÔNG dùng `list_dir` — không tồn tại):
   - `Document refer/**/*`
   - `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/*`
   - `Output_QC/2. Test case/<site_folder>/<module_folder>/*<TicketID>*`
   - `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/*`
2. **Đọc Spec gốc** từ BA Web Portal — nếu không thể đọc, yêu cầu user cung cấp.
3. **Đọc Analysis** từ `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/TestCase_Material.md` (chọn version mới nhất nếu có nhiều version, vd: `_V2.md` > `_V1.md`).
4. **Đọc TCs** từ `Output_QC/2. Test case/<site_folder>/<module_folder>/[TicketID]_TCs.md`. Nếu có `_Matrix.md` → đọc luôn, mỗi Pattern = 1 nhóm TC cần execute cho tất cả targets listed.
5. **Đọc Supplementary files** nếu có: QnA, Exploration files (ưu tiên version cao nhất `V*.md`), TC Review, Bug history → highlight hotspot cần đặc biệt chú ý.
6. **Self-check** trước khi sang Phase 2:
   - Đã đọc đủ 3 nguồn primary (Spec + Analysis + TCs)?
   - Có conflict giữa các nguồn không? Nếu có → raise clarification, KHÔNG tự assume.
   - Domain context (BHYT rules, PACS/LIS integration, concurrent patient edits) đã ghi nhớ?
   - Mode + Execution Scope đã confirmed từ user?

> Project rules (`domain_knowledge.md`, `qa_testing_standards.md`, `ui_ux_standards.md`, `browser_safety_rules.md`) nằm tại `.agents/rules/`. Chỉ recall các điểm áp dụng cho ticket này.

**Output Phase 1:** Tóm tắt ngắn `<task_name>`, tổng số TC sẽ chạy, danh sách hotspot cần đặc biệt chú ý, danh sách CRUD TCs sẽ skip nếu Scope=READ-ONLY.

---

### PHASE 2 — Pre-Execution Validation (Blocker Gate)

Trước khi chạy bất kỳ TC nào, validate:

| # | Check item | Pass criteria | Nếu Fail |
| :---: | :--- | :--- | :--- |
| 1 | Test environment URL hoạt động | HTTP 200, page load < 10s | STOP |
| 2 | Test account hợp lệ | Login thành công, đúng role | STOP |
| 3 | Test data tồn tại | PatientID, BHYT_Card, VisitID được liệt kê trong Analysis có thể access | STOP |
| 4 | Output folder tồn tại / writable | `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/`, `evidence/` | Tạo folder |
| 5 | Checkpoint file (nếu có) | Đọc `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/.checkpoint_<TicketID>.json` để resume nếu run trước đó interrupt | Bắt đầu fresh nếu không có |

**STOP CONDITION:** Nếu environment/account/data không sẵn sàng → STOP execution, ghi blocker chi tiết vào báo cáo, return cho user.

> Browser safety là **runtime gate** (check trước mỗi action trong Phase 3), không phải pre-execution gate. Quy tắc đầy đủ tại `.agents/rules/browser_safety_rules.md`.

---

### PHASE 3 — Risk-Based Sequential Execution

**Thứ tự thực thi:** **High Priority → Medium → Low; trong cùng priority thì TC ID tăng dần** (theo SKILL Rule 11 — Execution Efficiency). Lý do: bắt critical bugs sớm, tránh waste effort nếu env crash giữa chừng.

**Áp dụng 13 mandatory rules trong `4.Execute_test_Skill/SKILL.md` cho từng TC, theo Mode đã chọn (NORMAL Full / LIGHT Smoke).**

**Quy trình cho mỗi TC:**

1. **Đọc TC** — Title, TC Type, Priority, Pre-condition, Steps, Expected.
2. **Scope check** — Nếu TC Type = `CRUD` và Scope = `READ-ONLY` → mark ⚪ Not Executed, ghi note `Skipped do scope READ-ONLY`, sang TC tiếp theo.
3. **Setup pre-condition** — đảm bảo state hệ thống đúng (account login, data tồn tại, page khởi đầu đúng).
4. **Execute step-by-step** với Atomic execution (1 step = 1 action = 1 verify):
   - Thực hiện action bằng tool an toàn (`browser_click`, `browser_type`, `browser_navigate`).
   - Trước mỗi click button có chữ giống write action (`Lưu`, `Submit`, `Xóa`, `Confirm`, `Xác nhận`) → áp ZERO-GUESS:
     - Scope `READ-ONLY` → STOP, mark TC blocked
     - Scope `SANDBOX` → proceed
     - Scope `PAIRED` → STOP, ask user click manually
   - Screenshot bằng `browser_take_screenshot`, naming `Ticket_<TicketID>_<TCID>_<seq>.png`.
   - Log ra `Log Ticket_<TicketID>.md` với format: `[YYYY-MM-DD HH:mm:ss] TC-XXX | Step N | Action | Response`.
5. **Validate Expected vs Actual** — áp dụng Source Priority (SKILL section "Source Priority").
6. **Layer Verification theo TC Type** (SKILL Rule 5.1 — TC-type aware):
   - `Pure UI` / `Cosmetic` → UI only
   - `Read` / `Display` / `Search` / `List` → UI + Network response (qua `browser_network_requests`)
   - `Validation` / `Error` → UI + Network response
   - `CRUD` / `Mutation` → UI + Network response + Persistence (F5 / reopen) + Side effects
   - `Navigation` → UI + (Network response nếu API involved)
7. **Risk-based Observation** (Rule 3, NORMAL only / LIGHT critical hotspots only) — UI glitch, delay, tag overlap, watermark flicker, Safari rendering nếu test multi-platform.
8. **Edge & Negative probing** (Rule 4, NORMAL only) — nếu TC là Functional, thử thêm 1-2 edge case lân cận.
9. **Kết luận status:**
   - 🟢 **Pass**: applicable layers pass + không có suspicious behavior
   - 🔴 **Fail**: ít nhất 1 applicable layer fail HOẶC actual behavior khác expected
   - 🟡 **Blocked**: không thể chạy (dependency/env issue giữa chừng)
   - ⚪ **Not Executed**: skipped có lý do (scope=READ-ONLY với CRUD, hoặc skip có lý do khác)
10. **Ghi vào Master Execution Table** (Section B của template).
11. **Checkpoint persistence** — Sau mỗi 10 TC hoặc mỗi Critical/High Severity bug detected, save checkpoint:
    ```
    Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/.checkpoint_<TicketID>.json
    {
      "last_executed_tc": "TC-050",
      "results_so_far": [...],
      "evidence_index": [...],
      "timestamp": "ISO"
    }
    ```
    → Cho phép resume nếu context overflow hoặc interruption.

**Network Mutation Guard (theo browser_safety_rules.md):**
- Sau action quan trọng → check `browser_network_requests`, nếu thấy POST/PUT/DELETE không mong đợi trong scope `READ-ONLY` → STOP IMMEDIATELY, screenshot, báo user.
- Nếu confirmation dialog xuất hiện → luôn chọn `Cancel/No/Discard` (trừ scope `SANDBOX` với owned-data).

---

### PHASE 4 — Defect Investigation

Cho mỗi TC có status 🔴 Fail hoặc abnormal behavior phát hiện được:

1. **Reproduce** — Thử lại ≥3 lần để xác định reproducibility:
   - Consistent (≥4/5) → định lượng "5/5 (Consistent)"
   - Intermittent (2-3/5) → ghi "3/5 (Intermittent)"
   - Cannot reproduce (0-1/5) → vẫn ghi log + observation, không tự kết luận "random"
2. **Isolate variable** — Đổi 1 biến tại 1 thời điểm (account, browser, data) để xác định scope.
3. **Severity vs Priority Evaluation** (Rule 8.1) — Đánh giá ĐỘC LẬP 2 chiều, không gộp. Tham khảo 2 bảng riêng trong SKILL.
4. **Suspected Root Cause** — Dùng kinh nghiệm domain (BHYT limits, legacy HIS logic) để đoán nguyên nhân.
5. **Impact Area** — Xác định: Web/Admin, platform (Chrome/Safari), role variant.
6. **Multi-format Evidence collection** (theo SKILL Rule 13):
   - Step screenshots (`*.png`)
   - Network HAR khi có API failure (`*_network.har`)
   - Console log txt khi có JS error (`*_console.txt`)
   - Video MP4 cho intermittent (`*_video.mp4`)
   - Cross-browser screenshot nếu platform-specific (`*_chrome.png`, `*_safari.png`)
7. **Ghi vào Bug Detail Table** với Bug ID format `<TicketID>-BUG-NN` (vd: `SUN-1200-BUG-01`).
8. **Escalation check** (Rule 12) — Nếu match điều kiện escalate (data corruption / security / payment / critical block / unknown intermittent) → đánh dấu trong Section D.4 + thông báo user trong chat ngay.

---

### PHASE 5 — Report Generation & Cleanup

1. **Render báo cáo chính** theo template `Execution_Result_Template.md`:
   - Section A — Header & Summary (totals, Pass Rate, Coverage, Release Risk theo rubric explicit)
   - Section B — Master Execution Table
   - Section C — Bug Detail Table (chỉ Fail, Bug ID format `<TicketID>-BUG-NN`)
   - Section D — Risk & Escalation (incl. D.6 Top 3 findings)
   - Section E — Quality Score `/50` với justification
2. **Save báo cáo** tại: `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/1. Test_Result_Summary_<TicketID>.md`
3. **Save execution log** tại: `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/4. Log_Test_<TicketID>.md`
4. **Verify evidence** đã được lưu đầy đủ tại `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/evidence/` theo retention policy (SKILL Rule 13):
   - Pass TC → giữ 1 final-state screenshot (+ initial nếu relevant)
   - Fail TC → giữ tất cả step screenshots + HAR + console + video nếu intermittent
   - Blocked → 1 screenshot tại điểm block
5. **Archive policy** — Cuối execution, kiểm tra evidence các ticket > 90 ngày:
   - Zip thành `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/evidence/_archive/Ticket_<TicketID>_<YYYYMMDD>.zip`
   - **KHÔNG xóa** original folder/files — user sẽ review & xóa thủ công sau khi work ổn định
   - Ghi summary archive action vào báo cáo (nếu có)
6. **Final Quality Score** — Self-score 5 tiêu chí trong Section E. Nếu tổng < 40/50 → quay lại Phase 3 hoặc Phase 4 investigate thêm trước khi finalize.
7. **Cleanup checkpoint** — Nếu execution hoàn tất 100% TCs → xóa file `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/.checkpoint_<TicketID>.json` (không cần resume nữa). Nếu interrupted → giữ lại để resume lần sau.
8. **Tóm tắt cho user trên chat** (Tiếng Việt):
   - Mode đã dùng (NORMAL / LIGHT) + Scope (READ-ONLY / SANDBOX / PAIRED)
   - Total / Pass / Fail / Blocked / Not Executed
   - Pass Rate + Execution Coverage
   - **Top 3 critical findings** (highest severity bugs hoặc top 3 hidden risks nếu 0 Fail)
   - Production Release Risk assessment (🟢 / 🟡 / 🔴) với rubric reference
   - Quality Score `/50`
   - Link tới báo cáo chính
   - Bất kỳ escalation nào cần báo gấp (Section D.4)

---

## 4. ĐỊNH DẠNG ĐẦU RA (OUTPUT STRUCTURE)

```
Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/
├── 1. Test_Result_Summary_<TicketID>.md       ← Báo cáo chính (bảng .MD)
├── 2. Mapped_Result_Detail_<TicketID>.csv     ← Kết quả map chi tiết (.CSV đã được map)
├── 3. Out_of_scope_findings_<TicketID>.md      ← Phát hiện lỗi ngoài scope (nếu có)
├── 4. Log_Test_<TicketID>.md                  ← Execution log timestamp
├── .checkpoint_<TicketID>.json                ← Resume support (xóa khi xong)
└── evidence/
    ├── Ticket_<TicketID>_<TCID>_01.png
    ├── Ticket_<TicketID>_<TCID>_02.png
    ├── Ticket_<TicketID>_<TCID>_network.har
    ├── Ticket_<TicketID>_<TCID>_console.txt
    ├── Ticket_<TicketID>_<TCID>_video.mp4
    ├── Ticket_<TicketID>_<TCID>_safari.png
    └── _archive/
        └── Ticket_<OldTicketID>_<YYYYMMDD>.zip
```

**Report files (final deliverables) are also saved to `Report_QC/4.Execute test/<site_folder>/<module_folder>/`:**
- Copy `1. Test_Result_Summary_<TicketID>.md` → `Report_QC/4.Execute test/<site_folder>/<module_folder>/`

**Naming convention strict:**
- Test result file: `1. Test_Result_Summary_<TicketID>.md`
- Mapped result file: `2. Mapped_Result_Detail_<TicketID>.csv`
- Log file: `4. Log_Test_<TicketID>.md`
- Evidence file: `Ticket_<TicketID>_<TestCaseID>_<seq|tag>.<ext>`

---

## 5. COMMUNICATION STANDARDS

- **Ngôn ngữ output**: Tiếng Việt cho mô tả + giữ EN cho technical term, message text (vd: `Save Completed`).
- **Atomic Steps trong Steps to Reproduce**: 1 step = 1 action.
- **Explicit Data**: Luôn ghi cụ thể (`PatientID=BN001`, `BHYT=HS4010112345678`), không dùng "valid data".
- **Tone**: Chuyên nghiệp, trực tiếp, không robotic. Format Slack/Teams-friendly cho summary chat.

---

## 6. SAFETY GATES (Tổng hợp)

| Gate | Khi nào trigger | Hành động |
| :--- | :--- | :--- |
| **Mode Gate** (Section 2) | User không nêu Mode trong prompt | STOP → Ask "NORMAL hay LIGHT?" |
| **Scope Gate** (Section 2) | User không nêu Execution Scope | Default READ-ONLY → notify user |
| **Blocker Gate** (Phase 2) | Env/account/data issue | STOP → Ghi blocker → Return user |
| **Browser Safety Gate** (Phase 3) | Trước mỗi action có khả năng write | ZERO-GUESS theo `browser_safety_rules.md` |
| **Conflict Gate** (Phase 1, 3) | Spec ≠ Analysis ≠ TC | Raise clarification, không assume |
| **Network Mutation Guard** (Phase 3) | POST/PUT/DELETE không mong đợi | STOP IMMEDIATELY → screenshot → báo user |
| **Quality Score Gate** (Phase 5) | Trước khi save report | Score < 40/50 → quay lại investigate |
| **Escalation Gate** (Phase 4) | Match Rule 12 condition | Mark Section D.4 + báo user ngay trong chat |
