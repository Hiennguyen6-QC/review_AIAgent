---
name: Tester 3_Review_TCs_Skill
description: Kỹ năng phân tích đa chiều và rà soát chất lượng kịch bản kiểm thử (Testcase), dựa trên Requirement Traceability Matrix (RTM) và tư duy Tangibility.
---

# LĂNG KÍNH VÀ TIÊU CHUẨN THỰC THI (TESTER 3_REVIEW_TCS_SKILL)

**1. TƯ DUY CỐT LÕI (MINDSET)**
- **[MANDATORY] Bắt đầu bằng việc đọc context** trước khi review bất kỳ TC nào:
  1. `.claude/rules/domain_knowledge.md` — terminology, BHYT rules, tech stack
  2. **BA Common Rules (live)**: Navigate to `https://docs.sota-his.com/docs/business/usecases/common` và đọc toàn bộ CMR entries. BA owns và update trực tiếp trên portal — đọc fresh mỗi session, không dùng local copy. Dùng CMRs để verify TCs đã tuân thủ đúng không (input limits, pagination, soft-delete, audit).
     - **Auto-login (HTTP Basic Auth)**: Đọc `BA_PORTAL_USER` và `BA_PORTAL_PASS` từ `.env.test`. Portal dùng HTTP Basic Authentication — KHÔNG phải login form. Nhúng credentials trực tiếp vào URL, URL-encode ký tự đặc biệt (`#` → `%23`, `@` → `%40`). Ví dụ: `https://sotatek:9fouBA8wL!%237!Lrm@docs.sota-his.com/docs/business/usecases/common`. Nếu credentials trống → báo user điền vào `.env.test`. KHÔNG yêu cầu user login thủ công.
- **Không giả định (Never assume correctness):** Mọi Test Case đều có lỗi cho đến khi được kiểm chứng.
- **Traceability & RTM:** Mọi Test Case phải liên kết trực tiếp với Requirement. Xây dựng ma trận ngầm (RTM) để quét và phát hiện Requirement nào chưa có Test Case (Coverage Gap).
- **Devil's Advocate:** Nhập vai người kiểm thử máy móc nhất. Đặt câu hỏi: "Nếu tôi nhập y hệt bước này, kết quả có ra đúng như Expected không?"
- **Tangibility (Tính Hữu Hình):** 
  - Step phải là hành động vật lý (Click, Nhập text, Kéo thả).
  - Expected Result phải có dấu hiệu trực quan (Hiển thị màu đỏ, Nút bị mờ, Pop-up hiện lên). Tránh tuyệt đối các từ trừu tượng như "Hoạt động đúng", "Xử lý thành công".
- **[MANDATORY] Figma Wireframe Verification:** Khi review Test Cases, PHẢI cross-check field names, button labels, placeholder text trong TC với Figma design. Quy trình:
  1. **Đọc Figma file key** từ `domain_knowledge.md` (hoặc function definition nếu function có link Figma riêng). File key mặc định: `TLQ5SbHCXdzDKnRnHCZjiE`.
  2. **Dùng Figma MCP tool** (`mcp__figma__get_figma_data`) để lấy data màn hình tương ứng với function đang review.
  3. **Cross-check**: Nếu TC dùng tên trường/button/placeholder khác với Figma → flag as naming error. Figma là **authoritative source** cho tên trường trong MVP.
  4. **Nếu màn hình không tồn tại trên Figma** → ghi note "Figma screen not found" và hỏi user trước khi tiếp tục.
  - **Read-only Figma access only** — không modify bất kỳ gì trong Figma.
- **[MANDATORY] BA Spec Portal Login Protocol** (khi cần cross-check spec):
  1. **Portal dùng HTTP Basic Authentication** — KHÔNG phải login form. Nhúng credentials trực tiếp vào URL.
  2. **Đọc credentials từ `.env.test`**: Lấy `BA_PORTAL_USER` và `BA_PORTAL_PASS`. URL-encode ký tự đặc biệt (`#` → `%23`, `@` → `%40`).
  3. **Navigate với embedded credentials**: `https://<BA_PORTAL_USER>:<encoded_pass>@docs.sota-his.com/docs/business/usecases`. Ví dụ: `https://sotatek:9fouBA8wL!%237!Lrm@docs.sota-his.com/docs/business/usecases`.
  4. **Nếu ERR_INVALID_AUTH_CREDENTIALS**: Credentials có thể đã thay đổi → báo user update `.env.test`.
  5. **KHÔNG yêu cầu user login thủ công** nếu credentials đã có trong `.env.test`.
  6. **KHÔNG fallback** sang Figma/file local chỉ vì auth fail — fix credentials trước.

**2. NGÔN NGỮ ĐẦU RA**
- Sử dụng **Tiếng Việt** cho các câu văn phân tích, nhận xét, hướng dẫn.
- Giữ nguyên **Tiếng Anh** đối với các thuật ngữ chuyên ngành (ví dụ: Test Case, Negative Test, Edge Cases, Boundary, Happy Path, Expected Result, RTM, Severity, CRITICAL).

**3. TIÊU CHUẨN QUÉT LỖI (7 CATEGORY)**
- **Category A (Cấu trúc):** Thiếu Title, Steps, Expected, Data, Priority.
- **Category B (Clarity & Logic):** Step gộp quá nhiều bước, Expected không đo lường được, Data test phi thực tế.
- **Category C (Coverage Gap):** Bỏ sót Requirement (dựa vào RTM). Thiếu Negative cases hoặc Edge cases. (Thiếu Negative là lỗi CRITICAL).
- **Category D (Redundancy):** Các test case trùng lặp logic, có thể gộp để tối ưu.
- **Category E (Priority Risk):** Đặt Priority sai lệch so với mức độ nghiêm trọng của chức năng.
- **Category F (CMR Compliance):** TC thiếu coverage cho CMR clause áp dụng cho function. Xem **Section 5** để biết quy trình kiểm tra chi tiết.
- **Category G (Tester 2 Rule Violation):** TC vi phạm các Embedded Generation Rules hoặc Screen Type Adaptive Rules định nghĩa trong Tester 2 Skill. Xem **Section 6** để biết quy trình kiểm tra chi tiết.

**4. TIÊU CHUẨN ĐÁNH GIÁ ĐIỂM (QUALITY SCORE)**
Hệ thống chấm điểm trên thang 50:
- **Coverage Completeness (/10):** Độ phủ kín RTM, Happy/Negative/Edge cases.
- **Clarity & Quality (/10):** Mức độ Hữu hình của Steps và Expected.
- **Structural Integrity (/10):** Định dạng, không bỏ sót trường thông tin.
- **Risk Alignment (/10):** Sự chính xác của Priority.
- **Optimization (/10):** Không có case thừa thãi.


---

## ⚠️ Quy tắc Xử lý File & Lưu tiến độ (BẮT BUỘC)

### 1. Version File Handling Rule (Xử lý File có nhiều phiên bản)
- **XÁC ĐỊNH FILE MỚI NHẤT TRƯỚC** — đọc tất cả các version để xác định file nào là mới nhất (ví dụ _v2.md, _v3.md).
- **CHỈ làm việc với phiên bản MỚI NHẤT** — tuyệt đối không edit file cũ.
- Nếu chưa rõ version nào là mới nhất → HỎI user trước, không tự đoán hoặc tự ghi đè.

### 2. Incremental Save Discipline (Lưu Tiến độ Tăng dần)
- Sau mỗi **đơn vị logic** hoàn thành (ví dụ: xong 1 phần analysis, 1 cụm testcase, 1 file review), AI MUST persist kết quả ngay bằng cách update file.
- KHÔNG chờ đến cuối task mới lưu. Nếu thread chết giữa chừng, user vẫn có baseline để resume.

---

## Section 5: CMR SYSTEMATIC CHECK (MANDATORY)

### Mục đích
Verify từng Common Rule (CMR) trên BA Portal xem có apply cho function đang review không. Nếu có → kiểm tra TC file đã cover đủ từng clause trong CMR đó chưa.

### Quy trình (bắt buộc chạy mỗi lần review)

**Bước 1 — Lấy CMR list từ BA Portal (LIVE, không dùng cached)**
1. Đọc credentials từ `.env.test` (fields `BA_PORTAL_USER`, `BA_PORTAL_PASS`)
2. Navigate đến `https://<BA_PORTAL_USER>:<encoded_pass>@docs.sota-his.com/docs/business/usecases/common`
3. Đọc toàn bộ danh sách CMR IDs và nội dung. Ghi lại: CMR ID, tên, và từng clause/rule trong CMR đó.
4. Nếu auth fail → báo user update `.env.test`. KHÔNG bỏ qua bước này.

**Bước 2 — Phân loại applicability cho từng CMR**
Với mỗi CMR ID, đánh giá: **CMR này có apply cho function đang review không?**

Tiêu chí applicability:
- CMR về input limits (max length, format) → apply nếu function có input fields
- CMR về pagination → apply nếu function có list screen
- CMR về soft-delete / audit log → apply nếu function có Delete/Edit action
- CMR về date/time validation → apply nếu function có date fields
- CMR về search/filter → apply nếu function có search bar
- CMR về upload/file → apply nếu function có file upload
- Ghi rõ lý do NOT APPLY nếu bỏ qua CMR ID nào

**Bước 3 — Coverage check cho từng APPLICABLE CMR**
Với mỗi CMR đã xác định là applicable:
1. Đọc từng clause/rule nhỏ trong CMR đó
2. Tìm trong TC file: có TC nào cover clause này không?
3. Nếu có → ghi `✓ Covered by [TC_ID]`
4. Nếu thiếu → ghi `✗ MISSING — [mô tả clause]` → đây là **Category F defect**

**Bước 4 — Report kết quả** (xem format ở Section 7, output block `## #5`)

---

## Section 6: TESTER 2 RULE COMPLIANCE CHECK (MANDATORY)

### Mục đích
Verify TC file tuân thủ tất cả các rules được định nghĩa trong Tester 2 Skill. Tester 2 Skill có thể được cập nhật thường xuyên → PHẢI đọc fresh tại mỗi lần run review.

### Quy trình (bắt buộc chạy mỗi lần review)

**Bước 1 — Đọc Tester 2 Skill FRESH**
```
Read: .claude/.agents/skills/2. Create_TCs_Skill/Tester 2_Skill.md
```
Không dùng version cached từ session trước. Mỗi review session phải đọc lại từ đầu.

**Bước 2 — Extract rule list**
Từ Tester 2 Skill, extract toàn bộ:
- **Embedded Generation Rules** (Rules 1–N, số lượng có thể thay đổi theo version)
- **Screen Type Adaptive Rules** (ST-1 through ST-N)
- **Execution Readiness Review checklist** (nếu có)
- **QA Tone & Wording standards** (nếu có)

**Bước 3 — Check từng rule**
Với mỗi rule được extract:
1. Xác định rule có applicable cho function/screen type đang review không
2. Nếu applicable → sample-check TCs trong file có comply không
3. Ghi kết quả: `✓ Compliant` / `✗ Violation: [TC_ID list] — [mô tả vi phạm]` / `N/A — [lý do không apply]`

**Bước 4 — Tổng hợp violations**
Mọi violation → classify là **Category G defect** và đưa vào defect list chính.

**Bước 5 — Report kết quả** (xem format ở Section 7, output block `## #6`)

---

## Section 7: OUTPUT REPORT FORMAT

**Naming convention:** `Review_<FunctionID>.md`
- Ví dụ: `Review_02_M10_F1_DM05_DinhMucBHYT.md`
- Thư mục: `Output_QC/3. Review/<ModuleFolder>/`

**Template bắt buộc:**

```markdown
# REVIEW REPORT — [FunctionID]

**Reviewer:** Tester 3-TCs-Reviewer
**TC File reviewed:** [path/to/TCs_file.md] (version: vX)
**Review date:** [YYYY-MM-DD]
**TC count:** [N] TCs ([UI_count] UI + [FC_count] FC)

---

## #1. CONTEXT ALIGNMENT

### Input Sources
| Source | File/URL | Status |
|---|---|---|
| Requirement Analysis | [path] | Read ✓ |
| Original Spec (BA Portal) | [URL] | Read ✓ |
| Figma | File key: [key], Node: [node] | Verified ✓ |
| QC-Function list | [path] | Read ✓ |

### UC Coverage Map
| UC ID | Tên UC | TC count | Covered? |
|---|---|---|---|
| UC-XXX | ... | N | ✓ / ✗ |

---

## #2. DEFECT LIST

### Category A — Cấu trúc
| # | TC_ID | Mô tả lỗi | Đề xuất sửa |
|---|---|---|---|

### Category B — Clarity & Logic
| # | TC_ID | Mô tả lỗi | Đề xuất sửa |
|---|---|---|---|

### Category C — Coverage Gap
| # | UC / Requirement | TC còn thiếu | Priority đề xuất |
|---|---|---|---|

### Category D — Redundancy
| # | TC_IDs trùng | Lý do | Đề xuất |
|---|---|---|---|

### Category E — Priority Risk
| # | TC_ID | Priority hiện tại | Priority đề xuất | Lý do |
|---|---|---|---|---|

### Category F — CMR Compliance
*(Chi tiết xem #5 bên dưới — đây là summary)*
| # | CMR ID | Clause | TC_ID thiếu | Đề xuất |
|---|---|---|---|---|

### Category G — Tester 2 Rule Violation
*(Chi tiết xem #6 bên dưới — đây là summary)*
| # | Rule ID | Mô tả vi phạm | TC_IDs liên quan | Đề xuất |
|---|---|---|---|---|

---

## #3. QUALITY SCORE

| Tiêu chí | Điểm (/10) | Nhận xét |
|---|---|---|
| Coverage Completeness | X | ... |
| Clarity & Quality | X | ... |
| Structural Integrity | X | ... |
| Risk Alignment | X | ... |
| Optimization | X | ... |
| **TỔNG** | **X/50** | |

---

## #4. TỔNG KẾT & KHUYẾN NGHỊ

**Verdict:** PASS / CONDITIONAL PASS / FAIL
**Lý do:**
**Defect summary:** A=[N] B=[N] C=[N] D=[N] E=[N] F=[N] G=[N]
**Blocking defects (phải fix trước khi chạy):** [list]
**Non-blocking defects (có thể fix sau):** [list]

---

## #5. CMR COMPLIANCE CHECK (Chi tiết)

**CMR data source:** [URL navigated]
**CMR version/date:** [date fetched]

| CMR ID | Tên CMR | Applicable? | Lý do | Clauses checked | Coverage |
|---|---|---|---|---|---|
| CMR-XX | ... | Yes/No | ... | N clauses | X/N covered |

### Chi tiết các CMR APPLICABLE

#### CMR-XX: [Tên CMR]
| Clause | Nội dung | TC cover | Status |
|---|---|---|---|
| 1 | ... | TC_ID hoặc — | ✓ / ✗ MISSING |

---

## #6. TESTER 2 RULE COMPLIANCE CHECK (Chi tiết)

**Tester 2 Skill version read:** [date/commit nếu có]
**Total rules extracted:** [N] Embedded + [N] ST rules

### Embedded Generation Rules
| Rule # | Nội dung tóm tắt | Applicable? | Status | Ghi chú |
|---|---|---|---|---|
| Rule 1 | ... | Yes/No | ✓ / ✗ Violation / N/A | ... |

### Screen Type Adaptive Rules
| ST Rule | Applicable screen type | Applicable? | Status | Ghi chú |
|---|---|---|---|---|
| ST-1 | ... | Yes/No | ✓ / ✗ Violation / N/A | ... |

### Execution Readiness Checklist
| Item | Status |
|---|---|
| ... | ✓ / ✗ |
```
