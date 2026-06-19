# QnA for BA — [Test Case ID] [Function Name]

> **Module**: [Module Name]
> **Feature**: [Feature Name]
> **Generated from**: QnA_Report of analysis task
> **Date**: [YYYY-MM-DD]
> **Source**: QC-Function list + WBS

---

## Hướng dẫn BA

> **BA chỉ cần điền:** cột **BA Answer**, sau đó cập nhật **New BA Status** và **New AI Status**.

### AI Status

| Value | Meaning |
| :--- | :--- |
| **Đã rõ** | AI hiểu đủ để viết TC |
| **Rõ 1 phần** | Cần thêm detail |
| **Chưa rõ** | Chưa hiểu, cần clarify |
| **Tự suy luận** | Chưa được BA confirm |
| **Coming soon** | Chưa làm ở phase hiện tại |

### BA Status

| Value | Meaning |
| :--- | :--- |
| **Chưa có spec** | Spec chưa có cho điểm này |
| **Spec chưa rõ** | Có nhưng mô tả không rõ |
| **Đã có spec** | Đầy đủ, không cần hỏi thêm |
| **Đã bổ sung** | BA đã update sau khi nhận câu hỏi |
| **Chưa triển khai** | Chưa implement ở phase hiện tại |

---

## QnA Table

> **STT là Master Numbering**: STT trong file này là ID chính thức. File QnA_Report sử dụng tag `[BA#X]` để cross-reference về STT này. Khi BA trả lời theo STT, trace ngược QnA_Report bằng tag `[BA#X]`. KHÔNG thay đổi STT sau khi publish — chỉ append câu hỏi mới ở cuối.

| STT | Module | Function/Screen | UC ID | QA Assessment | Q&A | AI Suggestion | QC Answer | AI Status | Priority | BA Status | Assign | BA Answer | New BA Status | New AI Status | Note |
| :---: | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :---: | :---: | :---: | :--- | :--- | :--- | :--- | :--- |
| 1 | [Module] | [Function/Screen] | [UC-XXX] | [1-2 câu: "[field] khác [CMR-XXX] — [delta cụ thể]"] | [Câu hỏi thực sự. Sub-items dùng `<br>`. Không có `\|`.] | [CMR-XXX / Standard UX: Ant Design / Business logic làm reasoning. Nếu Ant Design component: nêu props override nếu default behavior không phù hợp.] | | Chưa rõ | P1 | Chưa có spec | [PIC từ WBS] | | | | |
| 2 | | | | | | | | | P2 | | | | | | |

---

## Mapping Rules (for Tester 1)

1. **Module & Function/Screen**: Map từ QC-Function list `.md` — dùng Screen name và Feature name
2. **UC ID**: Lấy từ QC-Function list — column UC ID tương ứng với sub-function mà câu hỏi liên quan
3. **Assign** (cột trong table): **[MANDATORY LOOKUP — KHÔNG ĐƯỢC GHI CHUNG "BA"]**
   - Tìm file WBS mới nhất trong `BA/` (pattern: `WBS*.csv`)
   - Mở WBS → tìm row Module/Function tương ứng → đọc cột **PIC** dưới nhóm **"Tiến độ BA"**
   - Ghi **tên người cụ thể** (VD: "Xi Ka"). Nếu không tìm thấy → ghi `[WBS not found]`
   - Lưu ý: WBS có nhiều cột PIC (BA, DEV BE, DEV FE) — chỉ lấy PIC của **BA**
4. **QA Assessment**: Copy từ QnA Report — phần "QA Assessment" hoặc "Risk" + context ngắn gọn
5. **AI Suggestion**: AI PHẢI gợi ý câu trả lời dựa trên:
   - Domain knowledge (HIS, BHYT, medical workflow)
   - Context dự án (tech stack, existing patterns)
   - Common UX patterns (Ant Design, standard form behavior)
   - Kèm **reasoning** ngắn: "(Reasoning: ...vì...)"
6. **Priority Assignment**:
   - **P1**: Block test hoàn toàn — không thể viết TC nếu chưa trả lời
   - **P2**: Block 1 phần — có thể viết TC khác nhưng thiếu coverage
   - **P3**: Cần biết để đảm bảo accuracy nhưng không block execution
   - **P4**: Nice-to-have, observation, hoặc confirmed "để làm sau"
7. **Cross-module questions**: Nếu câu hỏi liên quan nhiều UC → liệt kê UC chính, ghi note các UC liên quan

---

## Update Protocol

### Khi QC (user) trả lời qua chat:
1. Điền nội dung vào cột **QC Answer**
2. Cập nhật **AI Status** → `Clear` (nếu hiểu rõ) hoặc `Partial` (nếu cần thêm)
3. Cập nhật **BA Status**:
   - Nếu QC nói "spec chưa có" → giữ `No spec`
   - Nếu QC nói "theo spec thì..." → đổi `Spec exists`
4. Giữ **BA Answer** trống — chờ BA confirm

### Khi BA trả lời:
1. Điền nội dung vào cột **BA Answer**
2. Cập nhật **BA Status** → `Spec updated` (nếu BA bổ sung spec) hoặc `Spec exists`
3. Cập nhật **AI Status** → `Clear`
4. Nếu "để làm sau" → BA Status giữ `No spec`, Priority hạ P4

### Khi QC confirm đủ rõ:
- Ghi note "Closed" trong cột Note
