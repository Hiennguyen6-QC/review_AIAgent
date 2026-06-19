# QA & Testing Standards - Project [Mini HIS]

Đây là bộ quy chuẩn nghiệp vụ dành cho mọi hoạt động kiểm thử trong dự án này.

## 1. Tư duy Kiểm thử (QA Mindset & Rigor)

### Nguyên tắc "Nguồn tin Hợp nhất" (Multi-source Synthesis):
- **Luôn đối chiếu chéo (Cross-check)**: Không bao giờ tin duy nhất vào một nguồn tin. Phải tổng hợp thông tin từ cả **Bản phân tích (Requirement Analysis)** và **Tài liệu gốc (Original Spec)** để đảm bảo tính toàn vẹn của logic.
- **Verification Gate**: Agent sau có trách nhiệm kiểm tra và verify lại kết quả của Agent trước đó.

### Tiêu chuẩn "QA chuyên nghiệp":
- **Atomic Thinking (Tư duy Nguyên tử)**: Mọi hành động, mọi bước mô tả phải được chia nhỏ đến mức tối giản (**1 Hành động = 1 Bước**).
- **Explicit Data (Dữ liệu cụ thể)**: Tuyệt đối không dùng dữ liệu giả định mơ hồ. Luôn sử dụng dữ liệu thực tế (VD: Mã thẻ BHYT hợp lệ, ID Bệnh nhân cụ thể).
- **Historical Awareness (Tư duy Phòng ngừa)**: Luôn soi xét các lỗi UI/Behavior kinh điển (dựa trên lịch sử lỗi HIS): *Dữ liệu BHYT có bị sync trễ không? Có bị mất dữ liệu khi 2 bác sĩ cùng sửa 1 bệnh án không? LIS/PACS trả kết quả lỗi thì UI xử lý thế nào?*

### 1.3. Analysis Coverage Discipline (Kỷ luật bao quát)

**Checklist bắt buộc TRƯỚC KHI finalize bảng test cases:**

1. **Default State Check**: Nếu ticket có state/value changes → mỗi case test non-default state PHẢI xem xét case verify behavior khi quay về default state (nếu applicable cho ticket)
2. **Cross-Reference Check**: So sánh page-specific cases với common TCs → Nếu common TC đã cover → KHÔNG tạo duplicate ở page-specific
3. **Scope Completeness**: Mỗi filter/feature type → scan ALL pages/components áp dụng → Không được loại item nào mà không có lý do tường minh từ spec
4. **Pattern Uniformity**: Các cases cùng nhóm PHẢI có cùng structure (steps, verify, expected) → Nếu case A có verify step mà case B không có → flag inconsistency

**Anti-pattern cần tránh:**
- Tự loại page/component vì "out of scope" mà không có evidence từ spec
- Tạo case cho 1 variant mà quên variant khác cùng behavior
- Cover "set value" mà quên "revert to default" (khi ticket có state changes)
- Cases cùng nhóm nhưng structure khác nhau mà không có lý do

## 2. Bộ lọc Phân tích dựa trên lỗi (Bug-Driven Analysis Filter)

### Nguyên tắc "Học từ lịch sử":
- TRƯỚC KHI phân tích spec mới, AI phải chủ động tìm kiếm tệp lịch sử lỗi (Bug list/Bug history) liên quan đến tính năng đó nếu có (thường nằm trong `Document refer/` hoặc patterns như `Bug list*.csv`).
- Nếu không tìm thấy tệp lỗi, phải ghi rõ trong phần báo cáo là: `No bug history found for this task`.

### Checklist bắt buộc (nếu có Bug List):
- [ ] **Hotspot identification:** Đã xác định các vùng "nhạy cảm" thường xuyên xảy ra lỗi trong quá khứ chưa?
- [ ] **Root Cause Mapping:** Đã đối chiếu Spec mới với danh sách Root Cause của các bug cũ để đảm bảo không lặp lại sai lầm cũ chưa?
- [ ] **Error Guessing:** Đã thực hiện kỹ thuật đoán lỗi dựa trên các "vết xe đổ" cũ chưa? (Ví dụ: Dự án hay lỗi ở Safari, lỗi logic inheritance, v.v.)
- [ ] **Regression risk:** Đã liệt kê các vùng có rủi ro bị ảnh hưởng ngược (regression) dựa trên các fix trước đó chưa?

## 3. Quy chuẩn Lưu trữ & Đặt tên Output (Storage & Naming Standards)

### 2.1. Đối với Agent Phân tích yêu cầu (Tester 1-Analyzer):
- **Thư mục lưu trữ**: `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/`
  - Ví dụ: `Output_QC/1. Analysis/01_User site/M1_Dang ky kham benh/01_M1_F1_01TiepdonBN/`
- **Quy tắc đặt tên file**:
    - File làm rõ yêu cầu (QnA nội bộ): `QnA_Report.md` (hoặc versioned: `QnA_Report_<task_name>_v2.md`)
    - File nguyên liệu Test Case: `TestCase_Material.md` (hoặc versioned: `TestCase_Material_<task_name>_v2.md`)
    - File QnA gửi BA: `QnA_BA_<task_name>.md`

### 2.2. Đối với Agent Tạo Test Case (Create TCs):
- **Thư mục lưu trữ**: `Output_QC/2. Test case/<site_folder>/<module_folder>/`
- **Quy tắc đặt tên file**:
    - File danh sách Test Case: `[TestCaseID]_TCs.md` (theo ID trong QC-Function list)
    - File này phải đảm bảo đủ 10 cột cố định (+ Phase conditional) theo định nghĩa trong Skill template.

### 2.3. TC Grouping Strategy (Gộp Test Case theo Screen)
- **1 Screen = 1 TC file** — Gộp tất cả sub-function của cùng 1 Screen vào 1 file test case duy nhất.
- **Bao gồm cả action trên list**: Ví dụ màn "Danh sách BN" sẽ gộp cả: xem danh sách, tìm kiếm, filter tab, XÓA bản ghi — tất cả trong cùng 1 file.
- **KHÔNG tách file** theo từng sub-function riêng lẻ (trừ khi Screen quá lớn >50 TCs thì mới xem xét split).
- **Đặt tên file theo TestCase ID** trong QC-Function list: `[TestCaseID]_TCs.md`
  - Ví dụ: `01_M1_F2_01DsDangKy_TCs.md` chứa tất cả TC cho Screen "Danh sách BN đã tiếp nhận"
- **Lý do**: Dễ follow khi test, giảm số file, mapping 1:1 với Screen trên hệ thống.

## 4. Quy tắc Xử lý File & Lưu tiến độ (BẮT BUỘC)

### 4.1. Version File Handling Rule (Xử lý File có nhiều phiên bản)
- **XÁC ĐỊNH FILE MỚI NHẤT TRƯỚC** — đọc tất cả các version để xác định file nào là mới nhất (ví dụ `_v2.md`, `_v3.md`).
- **CHỈ làm việc với phiên bản MỚI NHẤT** — tuyệt đối không edit file cũ.
- Nếu chưa rõ version nào là mới nhất → HỎI user trước, không tự đoán hoặc tự ghi đè.
- **Script/Generator output** → phải target vào file mới nhất, không phải file gốc.

### 4.3. Tool-Based Self-Check Gate (MANDATORY — "Done" requires evidence)

**"Đã review" và "Done" chỉ hợp lệ khi có tool evidence. Mental simulation không được tính.**

Sau khi hoàn thành TC file (tạo mới hoặc fix), bắt buộc chạy các checks sau bằng tool trước khi báo Done:

| Check | Tool | Pass condition |
|---|---|---|
| Blank lines trong table body | Grep `^\s*$` → đọc line numbers → xác nhận không line nào nằm giữa `\| --- \|` và `\n---\n` | 0 blank lines in table body |
| Row count khớp Summary | Grep `^\| M` đếm data rows → so với `## #1 OVERALL SUMMARY` | Count match |
| Không còn section header trong table | Grep `^#{2,4}\s` → xác nhận không kết quả nào nằm trong table range | 0 results in table body |

Quy trình: chạy checks → nếu fail → fix → chạy lại → pass → mới báo Done.

### 4.4. Skill Execution Pre-flight (TC Generation)

Trước khi generate bất kỳ TC file nào, **bắt buộc theo thứ tự**:

1. Read Skill file → xác nhận cấu trúc sections `## #1` / `## #2` / `## #3`
2. Read reference TC file (VD: `DinhMucBHYT_TCs_v3.md`) lines 1–60 → xác nhận skeleton bằng mắt
3. **Write skeleton trước** (header + section shells + table headers, chưa có data rows)
4. Xác nhận skeleton đúng → mới điền content

Không Write toàn bộ file trong 1 lần mà không verify skeleton. Sai skeleton = phải rebuild toàn bộ.


- Sau mỗi **đơn vị logic** hoàn thành (ví dụ: xong 1 module test, 1 cụm testcase, 1 section của analysis), AI MUST persist kết quả ngay — không chờ đến cuối task.
- KHÔNG chờ đến cuối task mới lưu. Nếu thread chết giữa chừng, user vẫn có baseline để resume.
- Khi phát hiện finding ngoài scope (OOS, observations) → cập nhật file findings ngay tại thời điểm đó.
