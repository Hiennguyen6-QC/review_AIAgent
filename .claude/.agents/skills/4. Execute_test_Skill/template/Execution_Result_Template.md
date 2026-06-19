# 📊 BÁO CÁO THỰC THI TEST: [Ticket-ID]

> Mẫu chuẩn cho Tester 4-Executor. Ngôn ngữ: Tiếng Việt cho mô tả + giữ nguyên EN cho thuật ngữ kỹ thuật (ví dụ: `Save`, `Submit`, `BHYT`).

---

## 📌 Section A — Thông tin chung (Header)

### A.1 Metadata

| Trường | Giá trị |
| :--- | :--- |
| **Ticket ID** | [SUN-XXXX] |
| **Tính năng** | [Tên tính năng / Module] |
| **Build / Version** | [User cung cấp — vX.Y.Z hoặc commit hash. Để trống nếu không có] |
| **Environment** | [Dev / Staging / UAT — URL nếu có] |
| **Platform tested** | [Chrome vXXX / Safari vXXX / TV App vXXX] |
| **Mode** | [NORMAL (Full) / LIGHT (Smoke)] |
| **Execution Scope** | [READ-ONLY / SANDBOX / PAIRED] |
| **Executed by (AI)** | Tester 4-Executor |
| **Verified by (Human)** | [QA name — để trống nếu không có pair-testing] |
| **Tester 4 Version** | v1.0.0 |
| **Skill Version** | v1.0.0 |
| **Start time** | [YYYY-MM-DD HH:mm — tz] |
| **End time** | [YYYY-MM-DD HH:mm — tz] |
| **Duration** | [Xh Ymin] |
| **Report generated at** | [ISO timestamp] |

### A.2 Execution Summary

| Metric | Value |
| :--- | :--- |
| **Total TC** | [N] |
| **Pass** | 🟢 [N] |
| **Fail** | 🔴 [N] |
| **Blocked** | 🟡 [N] |
| **Not Executed (Skipped CRUD do READ-ONLY scope)** | ⚪ [N] |
| **Pass Rate** = Pass / (Pass + Fail) × 100% | [X%] |
| **Execution Coverage** = (Pass + Fail) / Total × 100% | [Y%] |
| **Production Release Risk** | [🟢 Thấp / 🟡 Trung bình / 🔴 Cao] |
| **Tổng quan rủi ro** | [1-2 câu nhận định cao cấp về độ rủi ro release] |

### A.3 Rubric — Production Release Risk

| Mức | Điều kiện (đáp ứng ít nhất 1 điều) |
| :--- | :--- |
| 🔴 **Cao** | Có ≥1 Critical severity bug chưa fix; HOẶC ≥1 escalation tại Section D.4 active; HOẶC Pass Rate < 80% |
| 🟡 **Trung bình** | Có ≥1 High severity bug có workaround; HOẶC suspicious behavior chưa root-cause; HOẶC Pass Rate 80-94% |
| 🟢 **Thấp** | All Pass HOẶC chỉ có Low severity bugs; Pass Rate ≥ 95%; không có hidden risk lớn |

---

## 📋 Section B — Bảng tổng kết quả thực thi (Master Execution Table)

> Mỗi dòng = 1 TC đã chạy. Cột `Status` dùng emoji để dễ scan: 🟢 Pass / 🔴 Fail / 🟡 Blocked / ⚪ Not Executed.

| TC ID | Title | Priority | TC Type | Pre-condition | Test Data | Steps Summary | Expected Result | Actual Result | Status | Evidence | Risk / Observation | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- |
| TC-001 | [Tóm tắt mục đích TC] | High | Pure UI | [Điều kiện tiên quyết — vd: đã login với account `qa_user01`] | [Dữ liệu cụ thể — vd: `PatientID=BN001`, `BHYT=HS4010112345678`] | [Tóm tắt các step chính, tối đa 2-3 dòng] | [Kết quả mong đợi cốt lõi] | [Quan sát thực tế trên hệ thống] | 🟢 Pass | [Ticket_SUN-XXXX_TC-001_01.png](Evidence/Ticket_SUN-XXXX_TC-001_01.png) | [Quan sát phụ — vd: load chậm 3s, tag bị lệch 2px] | [Ghi chú khác] |
| TC-002 | ... | High | CRUD | ... | ... | ... | ... | ... | 🔴 Fail | [link] | [Mô tả symptom] | Xem chi tiết tại Bug SUN-XXXX-BUG-01 |
| TC-003 | ... | Medium | Read | ... | ... | ... | ... | ⚠ Cannot test | 🟡 Blocked | — | [Lý do block — vd: API trả 500] | Cần fix dependency trước |
| TC-004 | ... | High | CRUD | ... | ... | ... | ... | — | ⚪ Not Executed | — | Skipped do scope `READ-ONLY` (write action) | Cần PAIRED hoặc SANDBOX để chạy |

**TC Type values**: `Pure UI` / `Read` / `Validation` / `CRUD` / `Navigation`. Cột này map với 4-Layer Verification matrix trong SKILL Rule 5.1.

---

## 🐞 Section C — Chi tiết Bug (Bug Detail Table)

> Chỉ render những TC có `Status = Fail`. Nếu không có Fail → ghi "Không phát hiện defect nào trong execution này."

**Bug ID format**: `<TicketID>-BUG-NN` (vd: `SUN-1200-BUG-01`) để tránh trùng cross-ticket.

| Bug ID | TC ID | Severity | Priority | Bug Summary | Steps to Reproduce | Actual Result | Expected Result | Suspected Root Cause | Impact Area | Reproducibility | Evidence | Jira Ticket |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| SUN-XXXX-BUG-01 | TC-002 | High | High | [Tóm tắt 1 dòng — vd: Lưu đăng ký khám nhưng không sinh STT chờ khám] | 1. Mở màn hình "Đăng ký khám"<br>2. Nhập thông tin BN `PatientID=BN001`<br>3. Chọn phòng khám [Nội tổng hợp]<br>4. Click [Lưu] | Hệ thống lưu thành công nhưng không sinh STT, BN không xuất hiện trong danh sách chờ khám | Lưu thành công + sinh STT + BN hiển thị trong danh sách chờ khám | Logic sinh STT bị skip khi kiểu khám = BHYT (thiếu handle case) | User Site — Module M1 Đăng ký | 5/5 (Consistent) | [link] | [Để trống — sẽ điền sau khi tạo Jira] |
| SUN-XXXX-BUG-02 | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | [link] | |

**Severity vs Priority — Đánh giá ĐỘC LẬP** (xem chi tiết trong SKILL Rule 8.1). Không có mapping cố định.

---

## 🚨 Section D — Risk & Escalation Report

### D.1 Hidden Risks (Rủi ro ẩn)
> Các rủi ro phát hiện được trong quá trình execute nhưng KHÔNG nằm trong scope của TC.

- [Rủi ro 1 — vd: Khi search rỗng, API gọi 3 lần liên tiếp trong 1s → có thể gây overload backend]
- [Rủi ro 2 — vd: Session timeout không được handle, user phải F5 mới biết bị logout]

### D.2 Suspicious Behaviors (Hành vi đáng nghi)
> Behavior khác thường nhưng chưa đủ căn cứ để raise bug chính thức.

- [Quan sát 1 — vd: Đôi khi list video load lại không đầy đủ sau khi delete 1 item, F5 mới đúng. Reproduce 2/10 lần]

### D.3 Intermittent Issues (Lỗi không ổn định)
> Lỗi khó tái hiện, cần thêm investigation.

- [Lỗi 1 — vd: Toast message `保存しました (Save Completed)` thi thoảng hiện 2 lần liên tiếp]

### D.4 Escalation (Cần báo gấp)
> Các issue cần escalate ngay theo Rule 12. Đánh dấu `[X]` nếu match.

- [ ] Production data corruption risk — [mô tả nếu có]
- [ ] Security/privacy concern — [mô tả nếu có]
- [ ] Payment/authentication issue — [mô tả nếu có]
- [ ] Critical flow blocked — [mô tả nếu có]
- [ ] Intermittent issue with unknown root cause — [mô tả nếu có]

### D.5 Regression Concerns (Vùng có nguy cơ regression)
- [Vùng 1 — vd: Logic inheritance giữa Normal Web và R20 Web có thể bị ảnh hưởng do shared component]
- [Vùng 2 — vd: subDub=4 chưa được test trong scope này nhưng dùng chung parser với subDub=3]

### D.6 Top Findings (Bắt buộc render)
> Nếu có ≥1 Fail: liệt kê **Top 3 critical bugs** theo Severity.
> Nếu 0 Fail: liệt kê **Top 3 hidden risks / areas needing more testing**.

1. [Finding 1]
2. [Finding 2]
3. [Finding 3]

---

## ✅ Section E — Quality Score (Final Gate, /50)

> Tester 4 PHẢI tự chấm điểm và justify từng tiêu chí trước khi finalize report. Pattern đồng nhất với Tester 3-Reviewer.

| Tiêu chí | Điểm | Justification (bằng chứng cụ thể) |
| :--- | :---: | :--- |
| **Real Behavior Validation** (Đã validate behavior thật, không chỉ check expected text) | /10 | [Vd: cross-check network response + UI hiển thị + console log clean cho 90% TC] |
| **Suspicious Detection** (Đã detect & log behavior đáng nghi) | /10 | [Vd: phát hiện 3 hidden risks tại Section D.1] |
| **Investigation Depth** (Đã investigate sâu, reproduce ≥3 lần với abnormal) | /10 | [Vd: SUN-XXXX-BUG-01 reproduce 5/5, SUN-XXXX-BUG-02 reproduce 3/10 → mark intermittent] |
| **Layer Coverage** (Verify đủ applicable layers theo TC Type) | /10 | [Vd: CRUD TCs verify đủ 4 layer; Pure UI TCs verify UI layer] |
| **Anti-Shallow Execution** (Apply Bug Hunting, edge probing, không robotic) | /10 | [Vd: thử 5 edge cases ngoài predefined steps, phát hiện 1 bug additional] |
| **TỔNG ĐIỂM** | **/50** | |

**Pass threshold:** Score ≥ 40/50 để finalize.

**Final Gate Status:** [✅ PASSED — Sẵn sàng submit (≥40/50) / ⚠️ FAILED — Cần investigate thêm (<40/50)]

---

## 📎 Phụ lục — Input Sources & Output Locations

**Input đã sử dụng:**
- **Spec gốc:** `https://docs.sota-his.com/docs/business/usecases` (BA Web Portal)
- **Requirement Analysis:** `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/TestCase_Material.md`
- **QnA (nếu có):** `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/QnA_Report.md`
- **QnA BA (nếu có):** `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/QnA_BA_<task_name>.md`
- **Test Cases:** `Output_QC/2. Test case/<site_folder>/<module_folder>/[TestCaseID]_TCs.md`
- **TC Review (nếu có):** `Output_QC/3. Review/<TicketID>/TCs_Review_<TicketID>.md`
- **Bug history (nếu có):** `Document refer/Bug list*.csv`

**Output đã tạo:**
- **Báo cáo này:** `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/1. Test_Result_Summary_<TicketID>.md`
- **Execution Log:** `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/4. Log_Test_<TicketID>.md`
- **Evidence folder:** `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/evidence/`
- **Checkpoint (resume support):** `Output_QC/3. Execute test/<site_folder>/<module_folder>/<TicketID>/.checkpoint_<TicketID>.json`
