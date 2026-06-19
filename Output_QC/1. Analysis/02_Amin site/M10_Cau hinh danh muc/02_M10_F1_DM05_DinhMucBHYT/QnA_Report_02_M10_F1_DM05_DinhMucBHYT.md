# Q&A Report: 02_M10_F1_DM05_DinhMucBHYT Cấu hình định mức BHYT

🚨 *Clarification List for PO/Dev (Ready to copy-paste to Slack)* 🚨

> **Date**: 2026-06-03 (updated 2026-06-16 — CMR v2.7 re-analysis)
> **Mode**: DEEP
> **Assign (BA PIC)**: Trang
> **Spec source**: BA Portal (UC-189, UC-299, UC-301, UC-411, UC-302) — v1.2 (2026-06-04)
> **CMR Version**: v2.7 (2026-06-16)

> **Cross-reference**: Mỗi câu hỏi có tag `[BA#X]` tương ứng với STT trong file `QnA_BA`. Dùng để mapping khi BA trả lời. Thứ tự trong file này nhóm theo Category, không theo STT của QnA_BA. Câu hỏi self-resolved (không có trong QnA_BA) ghi `[N/A]`.

---

### Business Logic:

- [1] `[BA#1]` **[BA Status: Đã bổ sung | AI Status: Rõ 1 phần]** **Issue**: Search field range trên màn Danh sách (UC-189) — không nói rõ search match theo column nào
  → Searchbox match Lương cơ sở only (BR-189.2, debounce 300ms); Ngày hiệu lực là Date Picker filter riêng (BR-189.3). *(Full detail: QnA_BA #1)*
- [2] `[N/A]` **[BA Status: Đã có spec | AI Status: Đã rõ]** ~~Issue: Sort logic — AI raised unnecessarily~~

  - **QC Note (self-check)**: Sort logic đã được mô tả trong spec Postcondition Success của UC-189: "ưu tiên Đang áp dụng lên đầu, sau đó Ngày hiệu lực giảm dần". Đây là behavior từ spec, không phải gap. Admin page ít data nên không cần sort manual. **AI không nên raise câu này cho BA.** QC verify SC-09 theo spec postcondition.
  - **AI Suggestion**: Sort cố định per spec postcondition. Không có sort manual. *(Đã rõ từ spec)*
  - **Priority**: ~~P2~~ → **P4** (hạ vì QC đã resolved, không cần BA)
  - **QC Answer**: Đồng ý AI Suggestion. Sort per spec postcondition. Admin page ít data, không cần sort manual. AI không nên raise.
  - **BA Answer**:
- [3] `[N/A]` **[BA Status: Đã có spec | AI Status: Đã rõ]** ~~Issue: Race condition Edit + cron — AI raised unnecessarily~~

  - **QC Note (self-check)**: Logic này hệ thống phải handle theo quy tắc đã define. Spec UC-302 Exception đã có message tương tự ("Cấu hình này đã được áp dụng hoặc hết hiệu lực. Không thể xóa.") — cùng pattern, UC-301 phải follow. **AI không nên raise cho BA** — BA không cần ghi thêm. QC cần tạo test case để verify logic race condition này.
  - **AI Suggestion**: Backend reject lúc Save + message phù hợp. *(Implied từ UC-302 Exception)*
  - **Priority**: ~~P2~~ → **P4** (QC self-verify)
  - **QC Answer**: Logic hệ thống phải handle. Implied từ UC-302 Exception. AI không nên raise — QC tự verify test case.
  - **BA Answer**:

### Data Handling:

- [4] `[BA#3]` **[BA Status: Đã bổ sung | AI Status: Rõ 1 phần]** **Issue**: Field "Mức hưởng KCB Tuyến Xã" có ở popup Create/Edit nhưng không có column trên bảng list
  → BA confirmed: field không hiển thị trên list (default 100%), admin xem qua popup Xem chi tiết (UC-411) hoặc Chỉnh sửa — đây là quyết định design. *(Full detail: QnA_BA #3)*
  - **Status**: Resolved by BA#3 on 2026-06-04 ✓
- [5] `[BA#4]` **[BA Status: Đã bổ sung | AI Status: Rõ 1 phần]** **Issue**: Header column "Miễn lũy kế (Năm)" inconsistent với data "6 Tháng" và rule nghiệp vụ
  → BA đã sửa header thành "Miễn lũy kế (5 Năm)"; "(5 Năm)" là context period, data cell hiển thị số nguyên kèm đơn vị (VD: "6 Tháng"). *(Full detail: QnA_BA #4)*
- [6] `[BA#7]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: Validation field "Mức hưởng KCB Tuyến Xã" — cho phép decimal hay chỉ integer?
  → Integer only, range 0–100 (BR-301.5/BR-299.3). Error: "Giá trị phải từ 0 đến 100 và là số nguyên." *(Full detail: QnA_BA #7)*
- [7] `[BA#8]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: Field "Lũy kế miễn cùng chi trả (5 năm)" — không có max value
  → Max = 100 (BR-299.4/BR-301.6), không phải 60 như AI Suggestion. Error: "Giá trị phải từ 0 đến 100 và là số nguyên." *(Full detail: QnA_BA #8)*

### UI/UX:

- [8] `[BA#2]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: Filter "Trạng thái" — default value khi mở màn hình
  → Default = "Tất cả" (UC-189 UI Component #4). *(Full detail: QnA_BA #2)*
- [9] `[BA#6]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: Modal Xem chi tiết (UC-411) — Read-only behavior chưa rõ chi tiết
  → Tất cả fields readonly (copy được), button [Lưu] disabled (BR-411.2), footer chỉ có [Đóng], không có hint message. *(Full detail: QnA_BA #6)*
- [10] `[BA#9]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: Confirm discard khi đóng popup Create/Edit có data chưa lưu
  → Click Hủy/X/Esc/click outside → đóng luôn, không cảnh báo (UC-299/UC-301 Alt Flow 5c). Lưu ý: có conflict với CMR-043 v2.7 — xem BA#19. *(Full detail: QnA_BA #9)*
- [11] `[BA#10]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: Toast message wording — không rõ text chính xác cho 3 case success
  → (1) "Thêm cấu hình định mức BHYT thành công." (2) "Cập nhật cấu hình định mức BHYT thành công." (3) "Xóa cấu hình định mức BHYT thành công." *(Full detail: QnA_BA #10)*
- [12] `[N/A]` **[BA Status: Đã có spec | AI Status: Đã rõ]** ~~Issue: Empty state sau khi xóa record cuối cùng — AI raised unnecessarily~~

  - **QC Note (self-check)**: UX phải thế — list empty → hiển thị empty state + button Thêm mới luôn available. **AI không nên raise câu hỏi này cho BA.** QC verify SC-58 theo pattern standard.
  - **AI Suggestion**: Empty state giống UC-189 Alt Flow 2a + button Thêm mới luôn hiển thị. *(Standard UX)*
  - **Priority**: ~~P3~~ → **P4** (QC self-verify)
  - **QC Answer**: UX phải thế. AI không nên raise. QC tự verify.
  - **BA Answer**:

### System Behavior:

- [13] `[BA#5]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: UC-189 Action Buttons #4 có 2 reference mâu thuẫn (internal conflict)
  → UC-411 là đúng; UC-303 trong cột Ghi chú là copy-paste error, đã xóa. UC-301.2 đổi tên thành UC-411. *(Full detail: QnA_BA #5)*

### Security & PII:

- [N/A] Không có Security/PII concern cho màn cấu hình định mức BHYT — data là tham số hệ thống (Lương cơ sở, %, số tháng), không chứa PII bệnh nhân. RBAC đã được spec quy định: chỉ Admin/Quản trị hệ thống có quyền truy cập (NFR-4 UC-189).

---

### Spec Conflicts & Gaps (phát hiện 2026-06-04 khi đọc spec gốc):

- [14] `[BA#11]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: UC-299 conflict — Postcondition + BR-305.6 ghi cứng "Chưa áp dụng", bỏ qua case Ngày hiệu lực ≤ today
  → Portal v1.2 Postcondition đã fix: Ngày ≤ today → "Đang áp dụng" + cấu hình cũ → "Hết hiệu lực"; Ngày > today → "Chưa áp dụng". *(Full detail: QnA_BA #11)*
- [15] `[BA#12]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: UC-411 Purpose sai — ghi "chưa áp dụng" nhưng đúng phải là "Đang áp dụng" + "Hết hiệu lực"
  → UC-411 Purpose đã fix: view cấu hình trạng thái "Đang áp dụng" và "Hết hiệu lực". *(Full detail: QnA_BA #12)*
- [16] `[BA#13]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: BR-305.3 conflict — UC-299 và UC-301 dùng cùng ID cho 2 rule khác nhau
  → Portal v1.2 đã renumber toàn bộ: BR-189.x / BR-299.x / BR-301.x / BR-411.x / BR-302.x. Không còn duplicate ID. *(Full detail: QnA_BA #13)*
- [17] `[N/A]` **[BA Status: Đã có spec | AI Status: Đã rõ]** ~~Issue: UC-301 Alt Flow 5b — Save không có thay đổi~~

  - **QC Note (self-check)**: Spec Alt Flow 5b đã ghi rõ *"vẫn lưu và hiển thị thông báo thành công"* — behavior đúng với AI Suggestion. **AI không cần raise câu hỏi này cho BA** vì spec đã có. QC tự verify behavior này khi execute test case SC-15 (Edit happy path). Không cần thêm SC riêng.
  - **AI Suggestion**: Hệ thống vẫn gọi API + toast success + refresh list (spec đã confirm). *(Self-resolved)*
  - **Priority**: ~~P2~~ → **P4** (QC self-verify)
  - **QC Answer**: Đồng ý spec đã có Alt Flow 5b. Không cần escalate BA. QC tự verify khi execute SC-15.
  - **BA Answer**:
- [18] `[BA#14]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: Format tiền — spec dùng dấu chấm `2.340.000` (chuẩn VN) nhưng analysis output dùng dấu phẩy `2,340,000`
  → Dấu chấm "." là đúng (BR-301.8). Tester 2 phải dùng `2.340.000` trong toàn bộ TC. *(Full detail: QnA_BA #14)*
- [19] `[BA#15]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: UC-301 Exception Step 6 toast error dùng text của Create (UC-299), không phải Edit
  → Toast error Edit đã fix: "Không thể cập nhật cấu hình mức hưởng BHYT. Vui lòng thử lại sau." *(Full detail: QnA_BA #15)*
- [20] `[BA#16]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: UC-189 Action Button #6 ghi UC-305 (sai — không tồn tại)
  → Nút [+ Thêm mới] mở UC-299 (đúng). UC-305 là typo, đã fix trong portal v1.2. *(Full detail: QnA_BA #16)*
- [21] `[BA#17]` **[BA Status: Đã bổ sung | AI Status: Đã rõ]** **Issue**: UC-301 modal title ghi "Chỉnh sửa cấu hình định mức BHYT **mới**" — chữ "mới" thừa
  → Modal title đúng: "Chỉnh sửa cấu hình định mức BHYT" (bỏ "mới"). Đã fix trong portal v1.2. *(Full detail: QnA_BA #17)*

---

### Follow-up Questions (phát sinh 2026-06-12 sau khi BA trả lời):

- [22] `[BA#18]` **[BA Status: Spec chưa rõ | AI Status: Rõ 1 phần]** **Issue**: [From QnA BA#1] Placeholder của searchbox "Cấu hình định mức BHYT" không phản ánh đúng search behavior đã confirmed

  - **QA Assessment**: BA đã xác nhận search chỉ match Lương cơ sở (BR-189.2). Nhưng placeholder trong spec UC-189 vẫn ghi "Nhập từ khóa, tên viết tắt tìm kiếm..." — user nhìn vào không biết cần nhập số lương. UX inconsistency giữa placeholder text và search behavior.
  - **AI Suggestion**: Placeholder đề xuất: "Nhập lương cơ sở để tìm kiếm..." hoặc ngắn gọn "Tìm theo lương cơ sở...". *(Reasoning: Placeholder = hint cho user về kiểu input cần nhập — không nên là generic copy từ component khác)*
  - **Priority**: P3
  - **Question**: [From QnA BA#1] Placeholder của searchbox UC-189 cần được cập nhật cho phản ánh đúng behavior (chỉ search theo Lương cơ sở). BA xác nhận và update spec/design với placeholder text phù hợp: "Nhập lương cơ sở..." hoặc tương tự.
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#18]`
- [23] `[BA#31]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: Bản ghi "Cấu hình định mức BHYT" có bị chặn xóa theo CMR-032 khi đang được tham chiếu bởi module khác không?

  - **QA Assessment**: CMR-032 định nghĩa "blocked delete": khi một record đã được sử dụng bởi hệ thống (ví dụ: đợt khám M2, thanh toán M3 đã tính dựa trên config này), hệ thống hiển thị toast error *"Không thể xóa dữ liệu này. Dữ liệu này đã được sử dụng trên hệ thống, không thể xóa."* và giữ nguyên record. Spec UC-302 hiện chỉ mô tả happy path (xóa thành công) với điều kiện là record ở trạng thái "Chưa áp dụng" — không đề cập đến trường hợp record bị tham chiếu từ M2/M3.
  - **AI Suggestion**: Cấu hình định mức BHYT ảnh hưởng trực tiếp đến tính toán mức hưởng trong lần khám và thanh toán — có thể hệ thống đã lưu lại thông tin cấu hình này vào lịch sử khám/thanh toán. Nếu đã được lưu lại: khi xóa config đó, hệ thống phải chặn xóa và hiển thị thông báo lỗi theo CMR-032: "Dữ liệu này đã được sử dụng trên hệ thống, không thể xóa." — BA cần bổ sung trường hợp này vào spec UC-302 (Exception Flow). Nếu không lưu lại: không cần TC này. *(Không thể tự xác định — BA/DEV mới biết hệ thống có lưu lại hay không)*
  - **Priority**: P2
  - **Question**: Bản ghi "Cấu hình định mức BHYT" có được tham chiếu (lưu lại trong lịch sử khám/thanh toán) bởi module khác (M2 — Phòng khám, M3 — Viện phí/BHYT) không? Nếu có, khi xóa một config record đang được tham chiếu, hệ thống có chặn xóa và hiển thị toast error theo CMR-032 ("Dữ liệu này đã được sử dụng trên hệ thống, không thể xóa") không?
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#31]`
  - **Note**: *Trước đây tag [BA#19] trong file này — renumbered BA#31 để không conflict với QnA_BA BA#19 (CMR-043 dirty form — topic khác)*

---

### CMR v2.7 Conflicts & UX Component Types (phát sinh 2026-06-16 sau re-analysis CMR v2.7)

- [24] `[BA#19]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận — CRITICAL]** **Issue**: CMR-043 (v2.7, 2026-06-16) vs BA#9 — Conflict về dirty form warning khi đóng modal

  - **QA Assessment**: CMR-043 v2.7 (cập nhật 2026-06-16) yêu cầu: khi user đóng modal có thay đổi chưa lưu, phải hiện popup với title "Dữ liệu chưa được lưu", content "Thay đổi chưa được lưu. Bạn có chắc chắn muốn đóng?", nút rời đi = **[Đóng]**, nút ở lại = **[Tiếp tục sửa]**. BA#9 (trả lời 2026-06-04) xác nhận đóng ngay không cần confirm. **CMR-043 v2.7 cập nhật SAU khi BA#9 được trả lời** — conflict nghiêm trọng. Block TC SC liên quan discard/close behavior.
  - **AI Suggestion**: Nên áp dụng CMR-043 v2.7 — đây là standard cross-site, data loss trên financial config = incident. Popup "Dữ liệu chưa được lưu" với nút [Đóng] (rời đi) / [Tiếp tục sửa] (ở lại) là UX protection cần thiết. BA#9 cần re-confirm.
  - **Priority**: P2
  - **Question**:
    - (1) CMR-043 (v2.7, 2026-06-16) có áp dụng cho modal Thêm mới / Chỉnh sửa Định mức BHYT không?
    - (2) Nếu có: khi user click [Hủy] / dấu X / Esc / click outside khi form đã có thay đổi → hiện popup "Dữ liệu chưa được lưu" với nút **[Đóng]** (rời đi) / **[Tiếp tục sửa]** (ở lại)?
    - (3) Nếu không áp dụng: BA ghi rõ exception trong spec và re-confirm BA#9 do CMR-043 v2.7 cập nhật sau đó.
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#19]`
- [25] `[BA#20]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-007 — Click outside modal khi form dirty không đóng modal

  - **QA Assessment**: CMR-007 (v2.1) quy định click ngoài vùng modal KHÔNG đóng nếu form có thay đổi chưa lưu. BA#9 xác nhận click outside đóng ngay (không phân biệt dirty/clean). Liên quan đến BA#19 (CMR-043). Block TC click-outside behavior.
  - **AI Suggestion**: CMR-007 định nghĩa trigger (click outside không đóng modal khi form dirty); CMR-043 định nghĩa response (hiện popup confirm). Hai rules độc lập nhưng phải consistent — nếu BA#19 confirm áp dụng CMR-043 thì CMR-007 cũng phải áp dụng.
  - **Priority**: P2
  - **Question**:
    - (1) CMR-007 có áp dụng cho modal Định mức BHYT không?
    - (2) Nếu có: click outside khi form dirty → không đóng hay hiện popup CMR-043? Khi form chưa thay đổi → click outside vẫn đóng?
  - **Note**: Nếu BA#19 confirm áp dụng CMR-043, BA cần cập nhật UC-299/UC-301 Alt Flow cho cả hai trigger (Hủy/X/Esc và click outside).
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm. Liên quan BA#19.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#20]`
- [26] `[BA#21]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-009 v2.7 + CMR-063 — Delete dialog có 3 điểm conflict với spec UC-302

  - **QA Assessment**: CMR-009 v2.7 đã có bảng đối chiếu với entry UC-302 rõ ràng (tên đối tượng = "cấu hình định mức BHYT", toast = "Xóa cấu hình định mức BHYT thành công"). Chuẩn CMR-009 v2.7: title = "Xác nhận xóa", content bắt buộc có "Hành động này không thể hoàn tác.", nút thực hiện = **[Xóa]**, nút đóng = **[Đóng]** (CMR-063). Spec UC-302 hiện ghi: title = "Xóa cấu hình định mức BHYT" (khác), content không có câu "Hành động này không thể hoàn tác.", nút confirm = "Xác nhận" (khác), nút cancel = "Hủy" (khác). **3 điểm conflict**. Block TC SC-13.
  - **AI Suggestion**: Theo CMR-009 v2.7 + CMR-063: title = "Xác nhận xóa", nút thực hiện = "Xóa", nút đóng = "Đóng". Đề xuất giữ nội dung nghiệp vụ riêng UC-302 + thêm "Hành động này không thể hoàn tác.": "Bạn có chắc chắn muốn xóa cấu hình định mức BHYT này không? Sau khi xóa, hệ thống vẫn sẽ tiếp tục áp dụng mức định mức hiện tại cho đến khi có cấu hình mới. Hành động này không thể hoàn tác."
  - **Priority**: P2
  - **Question**:
    - (1) Title confirm dialog xóa là "Xác nhận xóa" (CMR-009 v2.7) hay "Xóa cấu hình định mức BHYT" (spec hiện tại)?
    - (2) Content dialog: có thêm câu "Hành động này không thể hoàn tác." không? Nội dung nghiệp vụ riêng UC-302 có giữ lại không?
    - (3) Tên nút: "Xóa" (CMR-009 v2.7) hay "Xác nhận" (spec hiện tại)? Nút đóng: "Đóng" (CMR-063) hay "Hủy" (spec hiện tại)?
  - **Note**: CMR-009 v2.7 đã có bảng đối chiếu UC-302 (tên đối tượng = "cấu hình định mức BHYT") — BA cần update spec UC-302 cho đúng chuẩn.
  - **QC Answer**: Gợi ý theo common rule, không giữ nội dung nghiệp vụ riêng vì BA chưa follow common. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#21]`
- [27] `[BA#22]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-011 toast template vs BA#10 confirmed text — từ "mới" có hay không?

  - **QA Assessment**: CMR-011 (v2.1) template: "Thêm mới [tên đối tượng] thành công" (có "mới"). BA#10 (2026-06-04) confirm text: "Thêm cấu hình định mức BHYT thành công." (không "mới"). BA#10 trả lời trước khi CMR-011 v2.1 cập nhật. Block TC SC-34.
  - **AI Suggestion**: Giữ BA#10 đã confirm: "Thêm cấu hình định mức BHYT thành công." (không "mới"). Từ "mới" trong CMR-011 template là pattern cho object noun — BA#10 là quyết định có chủ đích, không phải bỏ sót. Nếu CMR-011 v2.1 thay đổi intent → BA cần cập nhật mapping table exception.
  - **Priority**: P3
  - **Question**:
    - Toast success khi Thêm mới: "Thêm cấu hình định mức BHYT thành công." (BA#10, không có "mới") hay "Thêm mới cấu hình định mức BHYT thành công." (CMR-011 template v2.1)? BA#10 cần re-confirm do CMR-011 cập nhật sau ngày BA trả lời.
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#22]`
- [28] `[BA#23]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-015 v2.4 — format LCS đã nhất quán, chỉ còn câu hỏi về suffix VNĐ

  - **QA Assessment**: CMR-015 v2.4 xác nhận format integer + dấu chấm — nhất quán với BA#14 confirm "2.340.000". Conflict decimal tự giải quyết. Vấn đề còn lại: spec chưa nói rõ suffix "VNĐ" xuất hiện ở đâu — list header đã có "(VNĐ)" nhưng data cell và modal auto-calc chưa rõ. **Ảnh hưởng: SC-02, SC-63.**
  - **AI Suggestion**: Ô nhập liệu trong popup (Lương cơ sở, Quy đổi) hiển thị chữ "VNĐ" cố định ở bên phải ô — người dùng thấy rõ đơn vị mà không cần tự nhập. Cột "Lương cơ sở (VNĐ)" trên list đã có đơn vị trong header → data cell chỉ hiển thị số thuần "2.340.000" (không lặp lại "VNĐ"). *(Standard UX: Ant Design — ô nhập số tài chính thường có nhãn đơn vị cố định bên phải)*
  - **Priority**: P2
  - **Question**: Cột "Lương cơ sở (VNĐ)" trên list và số "Quy đổi" trong modal có hiển thị suffix "VNĐ" ngay sau số không: "2.340.000 VNĐ" hay chỉ "2.340.000"?
  - **QC Answer**: Đồng ý AI Suggestion (sau khi sửa cách diễn đạt). Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#23]`
- [29] `[BA#24]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-021 format error message — có cần prefix tên trường không?

  - **QA Assessment**: CMR-021 (v2.1) format chuẩn: "[Tên trường] không hợp lệ. Giá trị phải từ [min] đến [max]." Spec BA#7/BA#8 confirm: "Giá trị phải từ 0 đến 100 và là số nguyên." (không có tên trường, có thêm "và là số nguyên"). Block TC SC-26, SC-27, SC-28.
  - **AI Suggestion**: Giữ spec BA#7/BA#8 đã confirm: ngắn gọn, phù hợp form nhỏ. Khi field label đã hiển thị ngay cạnh input, lặp tên trường trong error message là redundant — người dùng đã biết mình đang nhập trường nào. BA có thể ghi exception vào CMR-021 mapping.
  - **Priority**: P2
  - **Question**:
    - (1) Error message khi trường "Mức hưởng KCB" vượt range: "Giá trị phải từ 0 đến 100 và là số nguyên." (BA#7) hay "Mức hưởng KCB đúng tuyến tại Tuyến Xã không hợp lệ. Giá trị phải từ 0 đến 100." (CMR-021)?
    - (2) UC-specific message có được override CMR-021 template không — hay phải follow template chuẩn?
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#24]`
- [30] `[BA#25]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-029 v2.4 — text empty search state UC-189 không khớp chuẩn mới

  - **QA Assessment**: CMR-029 v2.4 bỏ bảng mapping per-screen, đặt text cố định toàn hệ thống = "Không tìm thấy kết quả". Spec UC-189 hiện ghi "Không tìm thấy cấu hình phù hợp." — không khớp. **Block TC SC-12.**
  - **AI Suggestion**: Spec đang dùng text cũ không khớp CMR-029 v2.4. Text chuẩn mới = "Không tìm thấy kết quả" (universal, không còn per-screen). BA cần update spec UC-189 empty search state.
  - **Priority**: P2
  - **Question**: Text khi tìm kiếm không có kết quả trên UC-189: "Không tìm thấy cấu hình phù hợp." (spec hiện tại) hay "Không tìm thấy kết quả" (CMR-029 v2.4)?
  - **Note**: BA cần sửa text trong spec UC-189 cho khớp CMR-029 v2.4.
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#25]`
- [31] `[BA#26]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-010 v2.4 — text empty list state UC-189 không khớp chuẩn mới

  - **QA Assessment**: CMR-010 v2.4 đơn giản hóa: chỉ cần text "Không có dữ liệu", không còn yêu cầu icon. Spec UC-189 hiện ghi "Chưa có cấu hình định mức BHYT nào" — không khớp. **Block TC SC-11.**
  - **AI Suggestion**: Spec đang dùng text cũ không khớp CMR-010 v2.4. Text chuẩn mới = "Không có dữ liệu" (universal, không icon). BA cần update spec UC-189 empty list state.
  - **Priority**: P3
  - **Question**: Text khi list UC-189 không có record nào: "Chưa có cấu hình định mức BHYT nào" (spec hiện tại) hay "Không có dữ liệu" (CMR-010 v2.4)?
  - **Note**: BA cần sửa text trong spec UC-189 cho khớp CMR-010 v2.4. Không còn cần icon per CMR-010 v2.4.
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#26]`
- [32] `[BA#27]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-026/027 page size selector — spec không mention nhưng CMR yêu cầu

  - **QA Assessment**: CMR-026/027 (v2.1) yêu cầu pagination có dropdown thay đổi page size 10/20/50/100 trên tất cả list screen. Spec BR-191.8 chỉ ghi "10 dòng/trang" cố định. Block TC pagination.
  - **AI Suggestion**: Áp dụng CMR-026/027, options 10/20/50/100. CMR-026/027 áp dụng tất cả list screen — spec BR-191.8 cần align với common rule. BA bổ sung vào BR-191.8 và UC-189 UI Components.
  - **Priority**: P2
  - **Question**:
    - (1) Pagination UC-189 có hỗ trợ thay đổi số dòng/trang (10/20/50/100) theo CMR-026/027 không? Hay BR-191.8 "10 dòng/trang" là cố định?
  - **Note**: Nếu có page size selector → BA bổ sung vào spec BR-191.8.
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#27]`
- [33] `[BA#28]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: UX Component — Các field số trong modal chưa rõ component type và format behavior

  - **QA Assessment**: Spec chỉ ghi "Input Number" chung chung. Ant Design `InputNumber` và `Input` có behavior khác nhau: InputNumber tự block ký tự chữ/ký tự đặc biệt, có step buttons (+/-), tự format dấu chấm. Nếu spec không confirm → TC không biết expected behavior khi nhập chữ vào field số. Cũng liên quan CMR-003: field bắt buộc (*) phải có marker.
  - **AI Suggestion**: Dùng **Ant Design InputNumber** per Standard UX: InputNumber block ngay ký tự không hợp lệ tại component level — tốt hơn cho field tài chính, người dùng không thể nhập nhầm chữ vào LCS. CMR-003 áp dụng: field required phải có dấu `*` đỏ.
  - **Priority**: P3
  - **Question — Numeric fields:**
    - (1) Các field số ("Mức lương cơ sở", "Tỷ lệ 1 lần miễn", "Lũy kế miễn", "Mức hưởng KCB Tuyến Xã") dùng Ant Design **InputNumber** hay Input thông thường?
    - (2) Khi nhập ký tự chữ → bị block ngay hay chờ blur/submit mới báo lỗi?
    - (3) Step buttons (+/-) có hiển thị không?
    - (4) Theo CMR-003: các field bắt buộc có dấu `*` đỏ bên label không?
  - **Note**: Suffix đơn vị (VNĐ / % / Tháng LCS) — BA confirm dùng nhãn đơn vị cố định bên phải ô hay text label đặt bên ngoài ô nhập liệu.
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#28]`
- [33b] `[N/A]` **[BA Status: Đã có spec | AI Status: Đã rõ]** ~~Issue: UX Component — Field ngày tháng trong modal chưa rõ component type và behavior~~

  - **QC Note (self-check 2026-06-16)**: Spec UC-301 UI Components #3 đã mô tả rõ: DatePicker, format `dd/mm/yyyy`, disabledDate block ngày quá khứ. **AI không nên raise câu hỏi này cho BA** — đây là lỗi của AI (không đọc kỹ spec trước khi tạo entry). Entry này chưa bao giờ tồn tại trong QnA_BA (không có STT 31 tương ứng). QC verify SC-20/SC-21 theo spec UC-301 UI Components #3.
  - **Status**: `[Đã rõ — Self-resolved: UC-301 UI Components #3 đã mô tả DatePicker, format dd/mm/yyyy, disabledDate ngày quá khứ]`
- [34] `[BA#29]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: UX Component — List screen controls chưa rõ type; searchbox không đúng CMR-033; required fields chưa rõ CMR-003

  - **QA Assessment**: Spec UC-189 mô tả searchbox là "Input (text)" nhưng theo CMR-033 (v2.1) searchbox phải là **Textbox** với placeholder chuẩn "Tìm kiếm theo [tiêu chí]…". Spec hiện tại dùng placeholder không theo CMR-033 (đã hỏi BA#18). Ngoài ra filter "Trạng thái" và "Ngày hiệu lực" cũng chưa được mô tả rõ component type. Nếu component type khác → TC về keyboard input behavior, placeholder, clear button sẽ khác — ảnh hưởng SC-01 (searchbox), SC-10 (filter behavior).
  - **AI Suggestion**:
    - **Searchbox**: Sửa từ "Input (text)" → **Textbox** (per CMR-033). CMR-033 định nghĩa rõ Textbox là component cho search input, không phải generic Input — khác nhau ở có/không có clear (×) button built-in.
    - **Filter "Trạng thái"**: Dùng **Select Dropdown non-searchable** vì chỉ có 4 options cố định. Lý do (Standard UX: Ant Design Select): dropdown non-searchable phù hợp khi options ≤ 7 và cố định; searchable chỉ cần khi options > 7 hoặc dynamic từ API.
    - **Filter "Ngày hiệu lực"**: Dùng **DatePicker** single date. Lý do (Standard UX): admin tra cứu theo 1 ngày cụ thể, không cần range.
    - **CMR-003**: BA cần xác nhận field bắt buộc trên list/filter có marker `*` đỏ không (thường không cần với filter, nhưng cần confirm).
  - **Priority**: P2
  - **Question — Nhóm 1 (Searchbox):**
    - (1) Spec ghi searchbox là "Input (text)" — theo CMR-033 có cần đổi thành **Textbox** không? CMR-033 yêu cầu gì khác so với "Input (text)" thông thường?
    - (2) Placeholder chính thức là gì? Đề xuất: "Tìm kiếm theo lương cơ sở…" (per CMR-033 format "Tìm kiếm theo [tiêu chí]…").
  - **Question — Nhóm 2 (Filter controls):**
    - (3) Filter "Trạng thái" dùng **Select non-searchable** với 4 options: Tất cả / Đang áp dụng / Chưa áp dụng / Hết hiệu lực?
    - (4) Filter "Ngày hiệu lực" dùng **DatePicker** single date hay RangePicker?
    - (5) BA bổ sung mô tả component type đầy đủ vào UC-189 UI Components.
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#29]`
- [35] `[BA#30]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: UX — Keyboard input cho DatePicker "Ngày hiệu lực áp dụng" chưa rõ behavior

  - **QA Assessment**: Spec UC-299/UC-301 đã ghi rõ DatePicker và disabledDate (block ngày quá khứ). Tuy nhiên chưa xác nhận behavior khi user gõ tay ngày bằng keyboard — Ant Design DatePicker cho phép nhập tay nhưng validate behavior khác nhau tùy config. Ảnh hưởng TC SC-20 (boundary date = today), SC-21 (past date bằng keyboard).
  - **AI Suggestion**: Ant Design DatePicker default: cho phép nhập tay, khi blur validate — nếu ngày không hợp lệ → clear field. Error message per CMR-018: "Ngày hiệu lực không hợp lệ" (inline). disabledDate đã có per BR-299/BR-301 → calendar block ngày quá khứ; keyboard bypass cần validate riêng.
  - **Priority**: P3
  - **Question**: "Ngày hiệu lực áp dụng" có cho phép nhập tay bằng bàn phím không? Nếu user gõ ngày quá khứ bằng keyboard → validate khi nào (blur / submit)? Error message khi gõ ngày không hợp lệ là gì?
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#30]`
- [36] `[BA#32]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-063 (v2.6+) — Tên nút hủy trong modal Thêm mới / Chỉnh sửa phải là "Đóng"

  - **QA Assessment**: CMR-063 (v2.6+) quy định toàn hệ thống: nút tắt popup / hủy hành động phải ghi "Đóng". Spec UC-299 và UC-301 hiện ghi nút = "Hủy" bên cạnh nút "Lưu". Inconsistency ngay trong cùng feature: UC-411 View Detail đã dùng "Đóng" đúng, nhưng UC-299/UC-301 vẫn ghi "Hủy". Block TC SC liên quan nút đóng modal.
  - **AI Suggestion**: Đổi thành "Đóng" theo CMR-063. Spec UC-411 trong cùng feature này đã dùng đúng → UC-299/UC-301 là inconsistency cần fix. BA update spec footer buttons cho UC-299 và UC-301.
  - **Priority**: P2
  - **Question**: Nút bên cạnh "Lưu" trong modal Thêm mới (UC-299) và Chỉnh sửa (UC-301): tên chính thức là "Hủy" (spec hiện tại) hay "Đóng" (CMR-063 v2.6+)? BA cần sửa cho nhất quán với UC-411 (View Detail đã dùng "Đóng" đúng).
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#32]`
- [37] `[BA#33]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-059 (v2.5+) — Breadcrumb behavior chưa được mô tả trong spec

  - **QA Assessment**: CMR-059 (v2.5+) quy định breadcrumb cho tất cả màn hình: segment root = không click được, segment giữa = click được, segment cuối = plain text (Danh sách/Thêm mới/Chỉnh sửa/Chi tiết), cắt ngắn khi >4 segments. Spec UC-189/UC-299/UC-301/UC-411 không mô tả breadcrumb structure hoặc behavior click. Ảnh hưởng TC SC liên quan đến navigation.
  - **AI Suggestion**: CMR-059 áp dụng toàn hệ thống. Breadcrumb đề xuất cho Thêm mới: "Quản trị > Cài đặt tham số > Cấu hình định mức BHYT > Thêm mới". Segment "Cấu hình định mức BHYT" click → về UC-189. Segment cuối = plain text, không click. BA bổ sung vào spec.
  - **Priority**: P3
  - **Question**:
    - (1) Màn Thêm mới / Chỉnh sửa / Chi tiết của Định mức BHYT có breadcrumb không? Nếu có, cấu trúc là gì?
    - (2) Segment giữa (ví dụ: "Cấu hình định mức BHYT") click được và navigate về UC-189 không?
    - (3) Segment cuối (tên màn active) có click được không — theo CMR-059 là không click?
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#33]`
- [38] `[BA#34]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-060 (v2.6+) — Cột "Thao tác" trong spec UC-189 ở đầu (trái), CMR-060 yêu cầu cuối (phải)

  - **QA Assessment**: CMR-060 (v2.6+) quy định cột "Thao tác" luôn ở cột ngoài cùng bên phải trên tất cả list screen. Spec UC-189 liệt kê columns: Thao tác → Ngày hiệu lực → Lương cơ sở → Miễn cùng chi trả → Miễn lũy kế → Trạng thái (Thao tác ở đầu — bên trái nhất). **Conflict hoàn toàn ngược với CMR-060**. Block TC SC-01 (column order verification).
  - **AI Suggestion**: Đưa cột "Thao tác" về cuối cùng bên phải theo CMR-060. Thứ tự đề xuất: Ngày hiệu lực → Lương cơ sở (VNĐ) → Miễn cùng chi trả (1 lần) → Miễn lũy kế (5 Năm) → Trạng thái → Thao tác. BA update spec UC-189 Table Structure và UI Components.
  - **Priority**: P2
  - **Question**: Cột "Thao tác" trong bảng danh sách UC-189 ở vị trí nào: đầu tiên bên trái (spec hiện tại) hay cuối cùng bên phải (CMR-060 v2.6+)? BA cần cập nhật spec UC-189 cho đúng CMR-060.
  - **QC Answer**: Tạm đồng ý với AI Suggestion. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#34]`

---

### CMR v2.7 Gap Fixes (phát sinh 2026-06-18 sau cross-check TC v4)

- [40] `[BA#35]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-009 v2.7 + CMR-063 — Spec UC-302 delete dialog chưa cập nhật theo common rule (3 điểm conflict)

  - **QA Assessment**: CMR-009 v2.7 quy định cấu trúc confirm dialog xóa cố định toàn hệ thống: (1) title = "Xác nhận xóa", (2) content bắt buộc có câu "Hành động này không thể hoàn tác.", (3) nút thực hiện = "Xóa", (4) nút đóng = "Đóng" (CMR-063 v2.7). Spec UC-302 hiện ghi: title không tách biệt, content không có câu "không thể hoàn tác", nút = "Xác nhận xóa" / "Hủy". Phát hiện khi cross-check TC M10_UC302_UI_1.1.1 vs CMR v2.7. Lưu ý: QnA_BA BA#21 đã hỏi phần title/nút — BA#35 bổ sung câu hỏi về nội dung cụ thể để BA update spec đầy đủ.
  - **AI Suggestion**: Áp dụng đầy đủ CMR-009 v2.7 + CMR-063: title "Xác nhận xóa", content "Bạn có chắc chắn muốn xóa cấu hình định mức BHYT này không? Hành động này không thể hoàn tác.", nút [Xóa] / [Đóng]. BA cập nhật UC-302 Main Flow Step 3 và UI Components footer buttons.
  - **Priority**: P2
  - **Question**: BA cần cập nhật UC-302 theo CMR-009 v2.7 + CMR-063: (1) Title = "Xác nhận xóa". (2) Nội dung gồm câu "Hành động này không thể hoàn tác." — nội dung nghiệp vụ riêng của UC-302 có giữ lại cùng không? (3) Nút thực hiện = "Xóa", nút đóng = "Đóng".
  - **QC Answer**: Đề xuất theo common rule, không giữ nội dung nghiệp vụ riêng vì BA chưa follow common. Chờ BA confirm.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#35]`
  - **Note**: *Liên quan BA#21 (đã hỏi title/nút). BA#35 bổ sung câu hỏi về câu "không thể hoàn tác" và content detail.*

- [41] `[BA#36]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-026/027 — Hành vi reset về trang 1 khi đổi rows-per-page từ trang ≠ 1 chưa được mô tả trong spec UC-189

  - **QA Assessment**: CMR-026/027 v2.1 quy định: khi đổi rows-per-page → system luôn reset về trang 1 (không giữ nguyên trang hiện tại). Spec BR-191.8 chỉ ghi "10 dòng/trang" cố định — chưa mô tả behavior reset khi page size thay đổi từ trang bất kỳ. Phát hiện khi thêm TC M10_UC189_FC_1.3.19 (scenario đang ở trang 2, đổi rows/page → verify reset về trang 1). Lưu ý: BA#27 đã hỏi về việc có dropdown page size không — BA#36 bổ sung hành vi reset trang cụ thể.
  - **AI Suggestion**: Bổ sung vào BR-191.8 hoặc UC-189 Pagination spec: "Khi thay đổi số dòng/trang, hệ thống reset về trang 1 bất kể đang ở trang nào."
  - **Priority**: P2
  - **Question**: Khi user đang ở trang 2 (hoặc bất kỳ trang nào ≠ 1) rồi thay đổi "Dòng trên trang" → pagination reset về trang 1 (CMR-027) hay giữ nguyên trang hiện tại? BA cần bổ sung hành vi này vào BR-191.8 / UC-189 Pagination spec.
  - **QC Answer**: Theo common rule, reset về trang 1. Chờ BA confirm và bổ sung vào spec.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#36]`
  - **Note**: *Liên quan BA#27 (đã hỏi về dropdown page size). BA#36 bổ sung behavior reset trang 1 khi đổi page size.*

- [42] `[BA#37]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-010 v2.4 — Text empty state DB trống trong spec UC-189 không khớp common rule

  - **QA Assessment**: CMR-010 v2.4 quy định text empty state khi DB trống = "Không có dữ liệu" (cố định toàn hệ thống, không cần icon). Spec UC-189 Alt Flow 2a hiện ghi "Chưa có cấu hình định mức BHYT nào" — text tùy chỉnh per-screen, không align CMR-010. Phát hiện khi cross-check TC M10_UC189_UI_1.2.6 vs CMR v2.7. Lưu ý: BA#26 đã hỏi cùng issue — entry này là bổ sung để BA confirm hành động cụ thể cần thực hiện trong spec.
  - **AI Suggestion**: Cập nhật UC-189 Alt Flow 2a: thay "Chưa có cấu hình định mức BHYT nào" → "Không có dữ liệu". Xóa icon (CMR-010 v2.4 không yêu cầu icon). Text cố định giúp đồng nhất toàn hệ thống, BA không cần duy trì per-screen mapping.
  - **Priority**: P3
  - **Question**: BA cần cập nhật UC-189 Alt Flow 2a: text empty state khi DB trống đổi thành "Không có dữ liệu" (CMR-010 v2.4), xóa icon nếu có. Text hiện tại "Chưa có cấu hình định mức BHYT nào" là per-screen text cũ chưa align common rule.
  - **QC Answer**: Theo common rule, "Không có dữ liệu". Chờ BA confirm và cập nhật spec.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#37]`
  - **Note**: *Liên quan BA#26. BA#37 bổ sung hành động cụ thể BA cần làm trong spec.*

- [43] `[BA#38]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-033 — Spec UC-189 chưa ghi maxlength 255 cho [Searchbox]

  - **QA Assessment**: CMR-033 v2.7 quy định Textbox tìm kiếm trên tất cả list screen có maxlength = 255 ký tự, block silently tại giới hạn (không cần error message). Spec UC-189 UI Components #1 (searchbox) không ghi maxlength. Phát hiện khi thêm TC M10_UC189_FC_1.1.20 (verify ký tự thứ 256 bị block khi [Searchbox] đã đạt 255 ký tự). Lưu ý: BA#29 đã hỏi về component type — BA#38 bổ sung maxlength spec cụ thể.
  - **AI Suggestion**: BA bổ sung vào UC-189 UI Components #1: "[Searchbox] maxlength = 255 ký tự. Khi đạt 255 ký tự, block ký tự tiếp theo (không nhận thêm), không hiển thị error message."
  - **Priority**: P2
  - **Question**: [Searchbox] trên màn danh sách UC-189 có maxlength = 255 ký tự theo CMR-033 không? Khi đạt 255 ký tự, ký tự thứ 256 bị block và không có error message? BA cần bổ sung vào UC-189 UI Components #1 (Textbox tìm kiếm).
  - **QC Answer**: Theo CMR-033, maxlength 255 block silently. Chờ BA confirm và bổ sung vào spec.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#38]`
  - **Note**: *Liên quan BA#29 (component type searchbox). BA#38 bổ sung maxlength behavior.*

- [44] `[BA#39]` **[BA Status: Chưa trả lời | AI Status: Cần xác nhận]** **Issue**: CMR-059 v2.5+ — Breadcrumb trên màn danh sách UC-189 chưa được mô tả trong spec

  - **QA Assessment**: CMR-059 v2.5+ quy định breadcrumb cho tất cả full-page screen: segment đầu không click, segment giữa click được → navigate về màn cha, segment cuối là plain text (tên màn hiện tại) không click. Spec UC-189 (màn danh sách) không mô tả cấu trúc breadcrumb hoặc behavior từng segment. Phát hiện khi thêm TC M10_UC189_FC_1.5.4 (verify click segment giữa navigate đúng). Lưu ý: BA#33 đã hỏi breadcrumb cho UC-299/301/411 — BA#39 bổ sung cho chính màn UC-189 (danh sách).
  - **AI Suggestion**: CMR-059 áp dụng bao gồm cả màn danh sách UC-189. Cấu trúc đề xuất: "Quản trị > Danh mục > Cấu hình định mức BHYT". Segment "Danh mục" click → navigate về màn cha (danh sách module Danh mục). Segment cuối "Cấu hình định mức BHYT" là plain text, không click. BA bổ sung vào UC-189 UI Components.
  - **Priority**: P2
  - **Question**: Màn danh sách UC-189 ("Cấu hình định mức BHYT") có breadcrumb không? Nếu có: (1) Cấu trúc các segment là gì? (2) Segment giữa click được và navigate về màn nào? (3) Segment cuối không click được? BA bổ sung breadcrumb structure vào UC-189 UI Components.
  - **QC Answer**: Theo CMR-059, segment giữa clickable, segment cuối không clickable. Chờ BA confirm cấu trúc breadcrumb cho UC-189.
  - **BA Answer**:
  - **Status**: `[Confirming — pending BA#39]`
  - **Note**: *Liên quan BA#33 (breadcrumb UC-299/301/411). BA#39 bổ sung breadcrumb cho màn danh sách UC-189.*
- [39] ~~`[BA#35]`~~ **[BA Status: N/A | AI Status: Self-resolved]** **Issue**: Cột "Miễn lũy kế (5 Năm)" — data cell hiển thị số thuần hay kèm đơn vị?

  - **Resolution**: AI error — câu hỏi không cần gửi BA. Spec UC-189 ghi rõ data example = "6 Tháng" (kèm đơn vị). Design (screenshot 2026-06-16) cũng thể hiện "6 Tháng" trong cell. User đã confirm: **data cell = "6 Tháng"**. Tên cột "(5 Năm)" là context period — không thay thế đơn vị trong cell. Xóa khỏi QnA_BA. Không hỏi lại.
  - **Status**: `[N/A — self-resolved 2026-06-16. Data cell = "6 Tháng". Đừng tạo lại câu hỏi này.]`

| Metric                                                                | Count                                                                                  |
| --------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Total questions (v2 original)                                         | 13                                                                                     |
| Total questions (updated 2026-06-13 — incl. existing + 2 follow-ups) | 23                                                                                     |
| Total questions (updated 2026-06-15 — CMR v2.4 re-analysis)          | 35                                                                                     |
| Total questions (updated 2026-06-16 — CMR v2.7 re-analysis)          | **38** (39 − 1 AI error removed: BA#35)                                          |
| Total questions (updated 2026-06-18 — CMR v2.7 TC gap fixes)         | **43** (+5: BA#35–39 cho TC v4 CMR gaps)                                               |
| BA Status: Đã bổ sung (BA answered 2026-06-04)                     | 17 (BA#1–BA#17)                                                                       |
| BA Status: Chưa trả lời (follow-up + new)                          | 21 (BA#18–BA#34 trừ self-resolved + BA#35–BA#39)                                      |
| BA Status: Self-resolved (N/A)                                        | 6 ([2], [3], [12], [17], [33b], [39])                                                 |
| AI Status: Đã rõ (fully resolved)                                  | 16 (BA#1–BA#17 excl. BA#18)                                                           |
| AI Status: Rõ 1 phần (follow-up pending)                            | 1 (BA#18 — placeholder UX)                                                            |
| AI Status: Cần xác nhận (pending — new)                            | 17 (BA#19–BA#34 trừ self-resolved + BA#35–BA#39)                                      |
| AI Status: P4 self-resolved (không cần BA)                          | 6 (Sort [2], Race condition [3], Empty state [12], Alt Flow 5b [17], DatePicker [33b], Lũy kế cell format [39]) |
| P1 (Block test)                                                       | 1 (BA#30)                                                                              |
| P2 (Partial block)                                                    | 12 (BA#19, BA#20, BA#21, BA#23, BA#24, BA#25, BA#27, BA#29, BA#35, BA#36, BA#38, BA#39) |
| P3 (Cần biết, không block)                                         | 5 (BA#18, BA#22, BA#26, BA#28, BA#37)                                                  |
| P4 (Đã resolved, không cần BA)                                    | 6 (Sort, Race condition, Empty state, Alt Flow 5b, DatePicker, Lũy kế cell format)    |

**Note (updated 2026-06-15 — CMR v2.4 re-analysis):**

- BA Trang đã trả lời chính thức 17 QnA qua file Excel (sheet "AI output - admin site"), tất cả "Đã update" (2026-06-04).
- Portal spec cũng update lên v1.2 (2026-06-04) — portal là authoritative source.
- 16 entries resolved hoàn toàn. BA#18 là follow-up mới (P3 — placeholder UX inconsistency).
- BA#3, BA#4 đóng hoàn toàn: BA's decision = spec.
- Key correction: Lũy kế max = **100** (không phải 60 như AI Suggestion cũ); UC-301.2 đổi tên thành **UC-411**.
- **2026-06-15**: CMR v2.4 re-analysis phát sinh 12 mục mới (BA#19–BA#30). Một mục cũ BA#19 (CMR-032) renumbered thành BA#31.
- BA#28 ban đầu split thành 2 câu riêng biệt: BA#28 (numeric fields) và BA#31 (date field) — tuy nhiên BA#31 (DatePicker) là AI error vì UC-301 UI Components #3 đã mô tả rõ. BA#31 self-resolved, renumbered: CMR-032 blocked delete = BA#31, Lũy kế cell format = BA#35.
- CMR v2.4 changes ảnh hưởng BA#23 (decimal tự resolved), BA#25 (text cố định "Không tìm thấy kết quả"), BA#26 (text cố định "Không có dữ liệu", không còn icon).
- **CRITICAL**: BA#19 (CMR-043 dirty form conflict) — CMR-043 v2.1 cập nhật 2026-06-13 sau khi BA#9 đã trả lời (2026-06-04). BA cần re-confirm BA#9.

---

## Naming Discrepancies (cần Tester 2 lưu ý)

| Item                                 | Spec (BA Portal)                                                                     | Analysis Output v2                                                    | Recommendation                                                                                                |
| ------------------------------------ | ------------------------------------------------------------------------------------ | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Field "Tỷ lệ miễn cùng chi trả" | List: "Miễn cùng chi trả (1 lần)" / Popup: "Tỷ lệ 1 lần miễn cùng chi trả" | Đúng                                                                | Dùng tên theo screen: List = "Miễn cùng chi trả (1 lần)", Popup = "Tỷ lệ 1 lần miễn cùng chi trả" |
| Header "Lũy kế"                    | v1.2:**"Miễn lũy kế (5 Năm)"** (Resolved by BA#4 on 2026-06-04)            | Output dùng tên cũ                                                 | **Sửa thành "Miễn lũy kế (5 Năm)"**; data cell format pending BA#20                               |
| Modal Edit title                     | UC-301: "Chỉnh sửa cấu hình định mức BHYT" (no "mới") — Resolved by BA#17   | Ghi đúng (bỏ "mới")                                               | ✓ Confirmed by BA                                                                                            |
| **Format tiền**               | **v1.2 BR-301.8: dấu chấm "." — Resolved by BA#14**                         | **Output dùng `2,340,000` (dấu phẩy — chuẩn US) — SAI** | **Tester 2 phải dùng dấu chấm `2.340.000` trong toàn bộ TC**                                    |
| UC-301.2 → UC-411                   | v1.2: UC đổi tên thành **UC-411** — Resolved by BA#5                      | Output dùng UC-411                                                   | **Tester 2 dùng UC-411 trong mọi tham chiếu**                                                        |
