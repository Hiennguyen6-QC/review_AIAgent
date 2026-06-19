# Browser Automation Safety Rules

## Read-Only Access Policy (ABSOLUTE - NO EXCEPTIONS)

Khi truy cập vào bất kỳ hệ thống nào qua browser (CMS, Admin, Web), AI PHẢI tuân thủ:

---

### 1. KHÔNG ĐƯỢC LÀM (Forbidden Actions):
- **KHÔNG** click nút Save / Lưu / Submit / Confirm / Xác nhận khi action đó GHI DATA vào hệ thống
- **KHÔNG** tạo mới (Create / Add / Thêm / Tạo mới) bất kỳ entity nào
- **KHÔNG** chỉnh sửa và LƯU (Edit + Save) bất kỳ data nào
- **KHÔNG** xóa (Delete / Xóa) bất kỳ data nào
- **KHÔNG** thay đổi trạng thái (Publish / Unpublish / Activate / Deactivate)
- **KHÔNG** upload file lên hệ thống
- **KHÔNG** drag-and-drop để reorder items (có thể auto-save thứ tự)
- **KHÔNG** sử dụng keyboard shortcut Ctrl+S, Ctrl+Enter, hoặc Enter trên form fields (có thể trigger submit)

---

### 2. ĐƯỢC PHÉP LÀM (Allowed Actions):
- Navigate / mở các màn hình, menu
- View / xem danh sách, chi tiết
- Search / tìm kiếm (đây là mục đích chính của đa số testcase đọc)
- Open popup / dialog để kiểm tra UI, fields, layout
- Fill input fields để test behavior (nhưng KHÔNG submit)
- Click Cancel / Hủy / Close / Đóng để đóng popup
- Take screenshot để làm evidence
- Đọc DOM, console logs, network requests
- Pagination / sorting / filtering trên list screens

---

### 3. Pre-Click Verification (MANDATORY):
Trước khi click BẤT KỲ button/link nào, AI PHẢI:
1. Đọc text/label của element (snapshot)
2. Xác nhận element KHÔNG phải action ghi data
3. **Nếu không chắc chắn → DỪNG NGAY, hỏi user confirm. KHÔNG ĐƯỢC suy đoán.**
4. Đặc biệt cẩn thận với nút có 2 chức năng (vd: "Save & Close", "Submit & Continue")

**ZERO-GUESS PRINCIPLE:** Nếu AI có BẤT KỲ nghi ngờ nào (dù chỉ 1%) về việc action có an toàn hay không → PHẢI DỪNG và hỏi user. Không bao giờ được "đoán" rằng action đó an toàn. Sai lầm do đoán = incident. Hỏi user = zero risk.

---

### 4. Keyboard Safety:
- **KHÔNG** nhấn Enter khi cursor đang ở trong form input (có thể submit form)
- **KHÔNG** nhấn Ctrl+S / Cmd+S (có thể trigger save)
- **KHÔNG** nhấn Ctrl+Enter (có thể submit)
- **CHỈ ĐƯỢC** dùng: Tab (chuyển field), Escape (đóng popup), Arrow keys (navigation)
- Khi cần test input behavior: dùng `browser_type` tool, KHÔNG dùng `browser_press_key` với Enter/Return

---

### 5. JavaScript Execution Safety:
- `browser_evaluate` CHỈ ĐƯỢC dùng để ĐỌC data (querySelector, getComputedStyle, localStorage.getItem)
- **KHÔNG ĐƯỢC** dùng `browser_evaluate` để:
  - Gọi fetch/XMLHttpRequest với method POST/PUT/DELETE
  - Trigger click events trên buttons
  - Modify DOM content
  - Dispatch form submit events
  - Gọi bất kỳ function nào có side-effect

---

### 6. Network Mutation Guard:
- Sau mỗi action quan trọng, kiểm tra `browser_network_requests` 
- Nếu phát hiện request POST/PUT/DELETE không mong muốn → BÁO NGAY cho user
- Chỉ chấp nhận: GET requests (read-only)
- Exception: POST cho search/filter API là OK (vì nhiều CMS dùng POST cho search)

---

### 7. Confirmation Dialog Handling:
- Nếu xuất hiện dialog/popup xác nhận BẤT KỲ:
  - Luôn chọn **Cancel / No / Không / Hủy / Discard / Leave**
  - KHÔNG BAO GIỜ chọn OK / Yes / Confirm / Có / Lưu khi dialog liên quan đến write action
- Nếu xuất hiện "unsaved changes" warning → chọn **"Discard" / "Leave without saving"**
- Nếu không rõ dialog đang hỏi gì → DỪNG LẠI, screenshot, hỏi user

---

### 8. Quy tắc cho Create/Edit screens:
- Được phép MỞ màn hình Create/Edit để kiểm tra UI
- Được phép NHẬP dữ liệu vào fields để test validation/behavior
- **PHẢI đóng bằng Cancel/Close** — KHÔNG BAO GIỜ nhấn Save/Submit
- Nếu có unsaved changes warning → chọn "Discard" / "Leave without saving"
- **Timeout rule**: Không ở lại màn hình Edit quá lâu — một số CMS có auto-save

---

### 9. Accident Recovery Protocol:
Nếu LỠ trigger một write action (do bug, misclick, hoặc unexpected behavior):
1. **DỪNG NGAY** — không thực hiện thêm bất kỳ action nào
2. **Screenshot** trạng thái hiện tại
3. **BÁO USER NGAY LẬP TỨC** với thông tin:
   - Action nào đã xảy ra
   - Trên màn hình nào
   - Data nào có thể bị ảnh hưởng
4. **CHỜ user chỉ đạo** trước khi tiếp tục

---

### 10. Scope Limitation:
- CHỈ truy cập các màn hình liên quan đến task đang thực hiện
- KHÔNG explore random các phần khác của hệ thống nếu không cần thiết
- KHÔNG truy cập Settings / Configuration / User Management sections
- KHÔNG mở Admin/System pages trừ khi user yêu cầu cụ thể

---

### 11. ZERO-GUESS PRINCIPLE (OVERRIDES ALL OTHER RULES):
**Đây là rule ưu tiên cao nhất — override mọi rule khác nếu có mâu thuẫn.**

- Nếu AI **không chắc chắn 100%** rằng action tiếp theo là an toàn (read-only) → **DỪNG NGAY VÀ HỎI USER**
- KHÔNG ĐƯỢC suy đoán, giả định, hoặc "nghĩ rằng có lẽ OK"
- KHÔNG ĐƯỢC dựa vào kinh nghiệm từ hệ thống khác để đoán behavior của hệ thống này
- Mỗi hệ thống có behavior riêng — button cùng tên có thể làm việc khác nhau
- **Cost of asking = 0. Cost of wrong guess = incident.**

Ví dụ PHẢI dừng hỏi:
- Button text không rõ ràng (vd: "OK", "Tiếp tục", "Xử lý")
- Không biết click link này sẽ navigate hay trigger action
- Popup xuất hiện bất ngờ không nằm trong flow dự kiến
- Element không có label rõ ràng
- Hệ thống redirect đến URL không mong đợi

---
- Đây là môi trường test CHUNG, data ảnh hưởng đến team khác
- Mục đích truy cập: verify behavior, check flow, collect test data IDs
- AI chỉ cần QUAN SÁT, không cần THAY ĐỔI hệ thống
- Dự án Y tế (Healthcare) yêu cầu bảo mật dữ liệu bệnh nhân cực kỳ cao — mọi thay đổi không được phép đều là incident
