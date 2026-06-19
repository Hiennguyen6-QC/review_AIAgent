# Q&A Report: [Task Name]

🚨 *Clarification List for PO/Dev (Ready to copy-paste to Slack)* 🚨

> **Date**: [YYYY-MM-DD]
> **Mode**: [QUICK/DEEP]
> **Assign (BA PIC)**: [Tên PIC từ WBS]

> **Cross-reference**: Mỗi câu hỏi có tag `[BA#X]` tương ứng với STT trong file `QnA_BA`. Dùng để mapping khi BA trả lời. Thứ tự trong file này nhóm theo Category, không theo STT của QnA_BA. Câu hỏi self-resolved (không có trong QnA_BA) ghi `[N/A]`.

*(If QUICK: List only blockers. If DEEP: Group into exactly 5 categories as specified below)*

---

### Business Logic:

- [1] `[BA#1]` **[BA Status: Chưa có spec | AI Status: Chưa rõ]** **Issue**: [one-line issue title]
  → Pending BA confirmation *(Full detail: QnA_BA #1)*

- [2] `[N/A]` **[BA Status: Đã có spec | AI Status: Đã rõ]** **Issue**: [one-line issue title — self-resolved]
  - **QA Assessment**: [Context + tại sao self-resolved]
  - **AI Suggestion**: N/A — đã rõ. [source: Standard UX: Ant Design / Regulatory: ... / spec postcondition UC-XX]
  - **Priority**: P4
  - **Question**: N/A (self-resolved)
  - **QC Answer**: [if applicable]
  - **BA Answer**: (N/A — not raised to BA)

### Data Handling:

- [3] `[BA#2]` **[BA Status: Spec chưa rõ | AI Status: Rõ 1 phần]** **Issue**: [one-line issue title]
  → [key resolution in 1 sentence, or "Pending BA confirmation"] *(Full detail: QnA_BA #2)*

### UI/UX:

- [4] `[BA#3]` **[BA Status: Chưa có spec | AI Status: Chưa rõ]** **Issue**: [one-line issue title]
  → Pending BA confirmation *(Full detail: QnA_BA #3)*

### System Behavior:

- [5] ...

### Security & PII:

- [6] ...

---

## Status Summary

| Metric | Count |
|---|---|
| Total questions | |
| BA Status: No spec | |
| BA Status: Spec unclear | |
| AI Status: Clear | |
| AI Status: Unclear/Partial | |
| P1 (Block test) | |
| P2 (Partial block) | |

---

## Column Definitions (for AI reference)

- **BA Status**: `No spec` (hoàn toàn chưa có) | `Spec unclear` (có nhưng thiếu detail) | `Spec exists` (đã đủ) | `Spec updated` (BA đã bổ sung) | `Coming soon` (feature chưa implement ở phase hiện tại)
- **AI Status**: `Đã rõ` (hiểu rõ) | `Rõ 1 phần` (hiểu 1 phần) | `Chưa rõ` (chưa hiểu) | `Tự suy luận` (AI tự suy luận từ CMR/Standard UX/Regulatory — chưa BA confirm) | `Chưa triển khai` (xác nhận chưa làm ở phase hiện tại — skip TC chi tiết)
- **QA Assessment**: Context + risk + blocked TC — giải thích TẠI SAO cần hỏi câu này
- **AI Suggestion**: Gợi ý dựa trên domain knowledge + reasoning. BA chỉ cần confirm/reject/adjust.
- **QC Answer**: User (QC) trả lời qua chat — behavior đã observe hoặc confirm từ stakeholder, nhưng spec chưa ghi rõ
- **BA Answer**: BA chính thức bổ sung — có thể update vào spec sau
