# QA Checklist Cheatsheet — Tester 2 (Mini HIS)

> **Dành cho:** Junior QC (1–2 năm kinh nghiệm) + AI Agent Tester 2
> **Nguồn tham chiếu:** `Output_QC/2. Test case/02_Amin site/M10_Cau hinh danh muc/02_M10_F1_DM05_DinhMucBHYT_TCs_v2.md` (đã qua review)
> **Cập nhật:** 2026-06-10

---

## 1. COLUMN QUALITY — Scenario & Case

Đây là hai cột bị sai nhiều nhất. Fix trước khi làm bất cứ việc gì khác.

### 1.1 Cột Scenario — Nhãn ngắn, KHÔNG có outcome

Scenario = nhãn phân loại tình huống. Người đọc cần biết **bối cảnh**, không cần biết kết quả.

| Loại Scenario | Pattern | Ví dụ ĐÚNG | Ví dụ SAI |
|---|---|---|---|
| Main Flow | Verb ngắn | `Create thành công` | `Create thành công → toast hiển thị` |
| Validation | `Validate [tên field đầy đủ theo UI]` | `Validate [Lương cơ sở (VNĐ)]` | `[LCS] = 0 → lỗi validation` |
| Error Handling | `API error khi [action]` | `API error khi tạo mới` | `Backend lỗi → toast error` |
| Permission | `Phân quyền — [mô tả]` | `Phân quyền — ẩn icon [Xóa]` | `User không có quyền xóa → icon bị ẩn` |
| Security | `Bảo mật — [loại]` | `Bảo mật — XSS injection` | `Nhập script vào field → không execute` |
| Edge Case | `Xung đột đồng thời — [context ngắn]` | `Xung đột đồng thời — 2 admin edit cùng record` | `2 admin edit cùng → conflict` |
| Performance | `Hiệu năng — [component] [behavior]` | `Hiệu năng — [Quy đổi Tỷ lệ] cập nhật realtime` | `Response time < 500ms sau nhập` |
| Audit | `Audit log — [action] record` | `Audit log — xóa record` | `Audit log ghi lại action xóa` |

**Dấu hiệu Scenario đang sai:**
- Có ký tự `→` trong cột Scenario
- Có số, %, đơn vị `ms`, tên giá trị test data
- Dài hơn ~50 ký tự
- Có chữ "auto-calc", "recalc", "pre-fill", "no-change" (dev jargon)

### 1.2 Cột Case — `Verify [component] [property]`

Case = mô tả cụ thể *cái gì đang được verify*. Không có prefix, không có `→`.

| Ví dụ ĐÚNG | Ví dụ SAI |
|---|---|
| `Verify [Lưu] button kích hoạt sau nhập đủ required fields` | `Click [Lưu] → lưu thành công` |
| `Verify [Trạng thái] badge hiển thị "Đang áp dụng" màu xanh` | `Badge xanh khi trạng thái đang áp dụng` |
| `Verify [LCS] = 0 báo validation error` | `[LCS] = 0 → lỗi` |
| `Verify [Tỷ lệ] > 100 không cho nhập hoặc báo lỗi` | `Nhập Tỷ lệ > 100 → lỗi validation` |
| `Verify toast "Xóa thành công" hiển thị sau confirm` | `Xóa record thành công → toast hiển thị` |

**Quy tắc nhớ nhanh:** Bắt đầu bằng `Verify`, sau đó `[component]`, sau đó property/condition. Không bao giờ kết thúc Case bằng outcome sau `→`.

---

## 2. MÀN DANH SÁCH (List Screen)

### Cần test những gì?

**Filter & Search — quan trọng nhất của List screen**

Với mỗi bộ lọc/ô tìm kiếm:
- [ ] Nhập từ khóa hợp lệ → hiện kết quả đúng
- [ ] Từ khóa partial match (nhập 1 từ trong chuỗi dài)
- [ ] Từ khóa không tồn tại → hiện empty state có message
- [ ] Xóa từ khóa / reset filter → danh sách về trạng thái ban đầu
- [ ] Debounce: hệ thống không gọi API sau mỗi keystroke — chỉ sau khi dừng gõ

Với filter nhiều điều kiện:
- [ ] AND logic: filter A + filter B cùng lúc → chỉ hiện record thỏa cả hai
- [ ] 3-way AND: filter A + B + C → record phải thỏa cả ba
- [ ] Hai filter độc lập: thay đổi filter A không reset giá trị filter B

**Hiển thị dữ liệu bảng**

- [ ] Mỗi cột hiển thị đúng format (số dùng dấu `.` ngăn cách nghìn — VD: `2.340.000`, không phải `2340000`)
- [ ] Cột trạng thái: màu badge đúng từng giá trị ("Đang áp dụng" = xanh, "Hết hiệu lực" = đỏ, "Chưa áp dụng" = xám)
- [ ] Record "Đang áp dụng" luôn hiển thị đầu tiên (pinned) — còn lại sort theo ngày giảm dần
- [ ] Cột icon/action: icon đúng nhóm (Edit chỉ cho record chưa/đang áp dụng, View cho tất cả, Delete chỉ cho "Chưa áp dụng")
- [ ] Cột tính toán (nếu có): giá trị hiển thị khớp với công thức spec

**Phân trang (Pagination)**

- [ ] Default: 10 record/trang
- [ ] Số trang > 8: hiển thị dấu `...` (ellipsis) — antd pagination chuẩn
- [ ] `[Trước]` disabled khi đang ở trang 1, `[Sau]` disabled khi ở trang cuối
- [ ] Trang hiện tại được highlight
- [ ] Nhảy thẳng đến trang bằng ô nhập số
- [ ] Đổi số record/trang → danh sách re-load
- [ ] **Quan trọng:** Khi thay đổi filter → tự động quay về trang 1

**Permission & Security**

- [ ] User không có quyền: menu item bị ẩn hoặc disable
- [ ] User không có quyền **truy cập trực tiếp bằng URL** (direct URL bypass) → redirect về 403 hoặc trang không có quyền
- [ ] (2 TC riêng biệt: menu navigation block + direct URL block)

**Edge Cases đặc trưng của List**

- [ ] Tìm kiếm với ký tự đặc biệt (`%`, `_`, `'`, `<script>`) → không bị SQL injection, hiển thị đúng
- [ ] Record bị xóa bởi admin khác trong khi đang xem danh sách → reload hiện trạng thái mới
- [ ] Cron job / batch process đổi trạng thái record tự động → sau khi chạy, danh sách phản ánh đúng

---

## 3. MÀN THÊM MỚI (Create Modal / Drawer)

### Cần test những gì?

**Main Flow**

- [ ] Nhập đủ dữ liệu hợp lệ → click [Lưu] → toast "Lưu thành công" / "Tạo thành công"
- [ ] Record mới xuất hiện trong danh sách ngay sau khi lưu
- [ ] Nếu có logic tự động (VD: ngày hôm nay → status "Đang áp dụng"; ngày tương lai → "Chưa áp dụng") → verify trạng thái đúng
- [ ] Nếu record mới làm record cũ "hết hiệu lực" → verify record cũ bị cập nhật trạng thái

**Tính toán realtime (nếu có)**

- [ ] Khi nhập giá trị vào field A → field B tự động tính toán ngay (không cần click [Lưu])
- [ ] Đổi field A → field B cập nhật lại
- [ ] Công thức: verify kết quả tính đúng với spec (VD: `Quy đổi Tỷ lệ = [Tỷ lệ] × [LCS] / 100`)
- [ ] Nếu field tính toán phụ thuộc field C: thay đổi field C → cả B và D đều recalculate

**Validation — thứ tự bắt buộc**

```
1. Required fields (submit khi để trống)
   → theo thứ tự field trên form (trên → xuống, trái → phải)

2. Per-field validation — theo thứ tự field trên form:
   a. Invalid range (VD: nhập âm, nhập > max)
   b. Input type filter (nhập chữ vào field số → field tự lọc hoặc báo lỗi)
   c. Boundary values (min hợp lệ, max hợp lệ)

3. Cross-field business rules
   - Unique constraint (VD: ngày hiệu lực không được trùng với record đã tồn tại)
   - Date range (VD: datepicker disable ngày trong quá khứ)
```

⚠️ **Lỗi thường gặp:** Viết Validation TCs không theo thứ tự form — nhảy lung tung giữa các field. Executor phải test từng field từ đầu đến cuối, không nhảy.

**Error Handling**

- [ ] API lỗi khi submit (500, timeout) → modal vẫn mở, dữ liệu đã nhập được giữ nguyên, hiển thị toast error
- [ ] (Không test: API lỗi không được xóa data người dùng đã nhập)

**Security**

- [ ] Nhập XSS payload vào field text (`<script>alert(1)</script>`) → không execute, field tự lọc hoặc báo lỗi

**Edge Cases**

- [ ] Giá trị numeric quá lớn (VD: LCS = 999.999.999) → system handle đúng hoặc có warning

**Performance (chỉ thêm khi có realtime calculation user-triggered)**

- [ ] Nhập giá trị → Quy đổi cập nhật < 500ms (antd input debounce thường 200-300ms)
- [ ] Nhập nhanh nhiều lần liên tiếp → không bị lag hoặc giá trị sai

---

## 4. MÀN CHỈNH SỬA (Edit Modal)

### Khác gì so với Create?

| Điểm khác | Create | Edit |
|---|---|---|
| Pre-condition trong TC | "Form trống" | "Form load dữ liệu từ record" |
| Default field value | Blank | `Default: loaded from saved data` (không viết "blank") |
| Main Flow thêm | — | Verify pre-fill đúng, verify no-change save |
| Edge Cases thêm | — | Xung đột đồng thời |

### Cần test những gì?

**Main Flow**

- [ ] Click [Sửa] → modal mở, tất cả fields đã load đúng dữ liệu của record
- [ ] Chỉnh sửa → [Lưu] → toast thành công, danh sách cập nhật
- [ ] Tính toán realtime khi edit (tương tự Create — khi đổi field A, field B recalculate ngay)
- [ ] Save không đổi gì → vẫn lưu được, không báo lỗi (no-change save)
- [ ] Click [Hủy] → modal đóng, không có gì thay đổi

**Validation**

- Tương tự Create — cùng thứ tự required → per-field → cross-field
- Dùng `Default: loaded from saved data` ở cột Pre-condition, không phải "blank"

**Edge Cases đặc trưng của Edit**

- [ ] **Xung đột đồng thời — status thay đổi trong khi edit:** Record đang ở "Chưa áp dụng", admin A đang sửa, admin B (hoặc tác vụ định kỳ) đổi record về "Đang áp dụng" → admin A submit → hệ thống từ chối hoặc cảnh báo
- [ ] **Xung đột đồng thời — 2 admin edit cùng record:** Admin A và admin B cùng mở modal sửa của 1 record → admin A lưu trước → admin B lưu → last-write-wins hay conflict?
- [ ] **Audit log:** Sau khi lưu thành công → DB/API có ghi before_state và after_state đúng không?

---

## 5. HÀNH ĐỘNG XÓA (Delete Action)

### Cần test những gì?

**Main Flow (3 TCs bắt buộc)**

- [ ] Click icon [Xóa] → popup confirm xuất hiện → click [Xác nhận] → record bị xóa, danh sách refresh, toast "Xóa thành công"
- [ ] Click icon [Xóa] → popup confirm xuất hiện → click [Hủy] → popup đóng, record VẪN còn trong danh sách
- [ ] Xóa record cuối cùng trong danh sách → danh sách trống, hiển thị empty state

**Error Handling**

- [ ] API lỗi khi xóa (500, timeout) → toast error, record vẫn còn trong danh sách

**Permission & Security (2 TCs)**

- [ ] Icon [Xóa] không hiển thị với user không có quyền (RBAC ở UI level)
- [ ] User không có quyền gọi thẳng DELETE API → backend trả 403/422 (RBAC ở API level — tách riêng khỏi TC UI)

**Edge Cases (4 TCs điển hình)**

- [ ] **Xung đột đồng thời — record đã bị xóa bởi admin khác:** Mở popup confirm → admin khác xóa record đó trước → user này click [Xác nhận] → hệ thống báo "not found", UI xử lý gracefully
- [ ] **Xung đột đồng thời — record đổi status trong lúc confirm:** Record là "Chưa áp dụng" → mở confirm dialog → tác vụ định kỳ chuyển nó sang "Đang áp dụng" → user click [Xác nhận] → hệ thống từ chối xóa (rule: không xóa record đang áp dụng)
- [ ] **Xung đột đồng thời — 2 admin xóa cùng record:** Ai xóa trước thành công, người sau nhận lỗi "not found"
- [ ] **Audit log:** Sau khi xóa → DB/API có ghi actor, timestamp, before_state của record đã xóa không?

---

## 6. PHÂN QUYỀN (RBAC) — Luôn cần 2 TCs riêng

Mọi chức năng có giới hạn quyền truy cập đều cần **2 TC tách biệt:**

| TC | Gì đang verify | Cách test |
|---|---|---|
| TC #1 | UI block | User thiếu quyền → menu item ẩn, hoặc button/icon không hiển thị |
| TC #2 | API block (direct URL bypass) | User thiếu quyền → gọi thẳng URL hoặc API → nhận 403/401, không load được data |

❌ **Đừng gộp 2 cái vào 1 TC** — chúng test ở 2 layer khác nhau (UI layer vs API layer).

---

## 7. AUDIT LOG

Khi spec yêu cầu audit log cho action nào → verify qua **DB hoặc API**, không qua UI.

Mỗi audit log TC phải verify:
- `actor` (user nào thực hiện)
- `timestamp` (thời điểm)
- `action` (tên action: CREATE/UPDATE/DELETE)
- `before_state` (dữ liệu trước khi thay đổi)
- `after_state` (dữ liệu sau khi thay đổi — với DELETE: null hoặc deleted flag)

---

## 8. PERFORMANCE — Khi nào cần thêm?

Chỉ thêm Performance TCs khi thỏa **cả 3 điều kiện:**

1. Có realtime calculation được trigger bởi user input (không phải load API thông thường)
2. Calculation phức tạp hoặc gọi server (không phải pure frontend math)
3. Spec/requirement có đề cập SLA hoặc QC thấy risk rõ ràng

**Nếu chỉ là load danh sách, mở modal, gọi API 1 lần** → không cần Performance TCs riêng.

---

## 9. CHECKLIST TỰ KIỂM TRA TRƯỚC KHI SUBMIT

Sau khi viết xong TCs cho mỗi UC, chạy qua checklist này:

**Structural checks:**
- [ ] Tất cả TCs của cùng UC có nằm liền nhau không? Không có TC nào bị displaced xuống cuối file?
- [ ] Thứ tự section trong UC: Main Flow → Validation → Error → Permission & Security → Edge Cases → Performance → Audit?
- [ ] TC_ID monotonically tăng trong mỗi section?

**Scenario column checks:**
- [ ] Scan cột Scenario: có `→` nào không? → rewrite về nhãn ngắn
- [ ] Scenario có chứa số, %, `ms`, test data value không? → strip đi
- [ ] Scenario có dùng "auto-calc", "recalc", "pre-fill", "no-change" không? → đổi sang tiếng Việt phổ thông
- [ ] Scenario nào dài > 50 ký tự? → trim

**Case column checks:**
- [ ] Scan cột Case: có `→` nào không? → rewrite thành `Verify [component] [condition]`
- [ ] Case có chứa prefix category (Positive, Negative, Edge Case)? → xóa prefix đi

**Coverage checks:**
- [ ] Màn danh sách: đã test AND-logic filter chưa (2-way và 3-way)?
- [ ] Màn danh sách: pagination đã test reset-to-page-1 khi filter chưa?
- [ ] Create/Edit: Validation TCs có theo thứ tự field trên form không?
- [ ] Create/Edit: có Audit log TC không (nếu spec yêu cầu)?
- [ ] Delete: có đủ 3 main flow TCs (confirm xóa, hủy xóa, xóa record cuối)?
- [ ] RBAC: có 2 TC riêng (menu/UI block + direct URL bypass)?
- [ ] Xung đột đồng thời: đã bao gồm cho Edit và Delete chưa?

**Format checks:**
- [ ] Số tiền/giá trị VNĐ dùng dấu `.` ngăn cách nghìn (2.340.000, không phải 2340000)?
- [ ] `[Confirming — pending BA#X]` chỉ xuất hiện trong cột Expected Result, không có trong Steps hay Pre-condition?
- [ ] Edit screen có dùng `Default: loaded from saved data` thay vì "blank"?
