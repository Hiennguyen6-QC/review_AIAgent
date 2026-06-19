---
description: Quy trình thực thi rà soát và đánh giá chất lượng Test Case, bắt buộc tuân thủ qua 3 bước phân tích nghiêm ngặt.
---

# QUY TRÌNH THỰC THI (REVIEW_TCS_FLOW)

## 1. YÊU CẦU ĐẦU VÀO
- **Requirement Analysis:** Lấy từ `Output_QC/1. Analysis/<site_folder>/<module_folder>/<task_name>/`
- **Original Spec:** Lấy từ `https://docs.sota-his.com/docs/business/usecases`
- **Test Cases:** File/Danh sách TC từ `Output_QC/2. Test case/<site_folder>/<module_folder>/` (Định dạng .md)
- **Skill:** Load `.agents/skills/3. Review_TCs_Skill/Tester 3_Review_TCs_Skill.md`.

## 2. CÁC BƯỚC THỰC THI BẮT BUỘC (EXECUTION STEPS)

### BƯỚC 1: CONTEXT ALIGNMENT & RTM INITIALIZATION
1. Tóm tắt nhanh tính năng và mục tiêu bao phủ (High-Level Coverage Goal).
2. Tự xây dựng Requirement Traceability Matrix (RTM) trong bộ nhớ: Map từng gạch đầu dòng Requirement với Test Case ID tương ứng.

### BƯỚC 2: COMPREHENSIVE DEFECT ANALYSIS (RÀ SOÁT ĐA CHIỀU)
Đưa toàn bộ Test Case qua 5 bộ lọc (Category A, B, C, D, E) được định nghĩa trong `Tester 3_Review_TCs_Skill.md`. 
- Ghi nhận mọi lỗi sai vào Master Defect List.
- Nếu phát hiện Requirement trong RTM không có TC -> Ghi vào Missing Scenarios.

### BƯỚC 3: SELF-EVALUATION (TỰ ĐÁNH GIÁ CHẤT LƯỢNG)
- Chấm điểm chất lượng theo thang 50 điểm dựa trên 5 tiêu chí của Quality Score.

## 3. ĐỊNH DẠNG ĐẦU RA (OUTPUT FORMAT)
- LƯU BÁO CÁO TẠI: `Output_QC/3. Review/<TicketID>/TCs_Review_<TicketID>.md`
- Ngôn ngữ: Tiếng Việt cho nội dung mô tả, Tiếng Anh cho thuật ngữ kỹ thuật.
- Cấu trúc BẮT BUỘC:

```markdown
# 📊 BÁO CÁO REVIEW TEST CASE: [Ticket-ID]

## 1. ĐÁNH GIÁ TỔNG QUAN (AUDIT SUMMARY)
- **Tổng số TC đã review:** [Số lượng]
- **Trạng thái:** [🟢 Đạt / 🟡 Cần cập nhật / 🔴 Phải làm lại]
- **Ma Trận Bao Phủ (RTM):** [Bao nhiêu % Requirement đã được cover]
- **Tự Đánh Giá Chất Lượng (Quality Score):** `[   ]/50`
  - *Coverage Completeness:* ` /10`
  - *Test Case Clarity & Quality:* ` /10`
  - *Structural Integrity:* ` /10`
  - *Risk Alignment & Prioritization:* ` /10`
  - *Optimization & Efficiency:* ` /10`
- **Nhận xét & Điểm yếu chí mạng:** [Đánh giá ngắn gọn, chỉ ra lỗi nghiêm trọng nhất]

## 2. 🔴 CÁC KỊCH BẢN BỊ THIẾU (MISSING SCENARIOS)
*(Requirement có trong Spec/RTM hoặc Edge Cases nhưng chưa có Test Case)*
1. **[Tên luồng thiếu]**: [Lý do/Rủi ro]

## 3. 🟡 DANH SÁCH LỖI (MASTER DEFECT LIST)
*(Các lỗi về Logic, Tính Rõ ràng, Data, Cấu trúc, Dư thừa)*

| Defect ID | TC ID | Loại Lỗi (Category) | Mức Độ (Severity) | Lỗ Hổng Hiện Tại | Đề Xuất Sửa Đổi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| D-01 | ... | ... | ... | ... | ... |

## 4. 🟢 ĐIỂM SÁNG (STRENGTHS)
- [Liệt kê các điểm làm tốt]

## 5. 🎯 HÀNH ĐỘNG TIẾP THEO (NEXT STEPS)
- [Yêu cầu sửa chữa/cập nhật]
```
