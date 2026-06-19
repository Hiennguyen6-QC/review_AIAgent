# Domain Knowledge Base (Mini HIS Project)

**Project Architecture Concept:**
The system is divided into two platforms with three primary actors:

**Actors:**
1. **NV Tiếp viên (Receptionist):** Nhân viên tiếp đón tại trạm y tế — đăng ký bệnh nhân, xử lý số thứ tự.
2. **Bác sĩ (Doctor):** Bác sĩ tại trạm y tế — nhập bệnh án, kê đơn thuốc, chỉ định xét nghiệm.
3. **Admin:** Thường là IT của trạm y tế — cấu hình danh mục, quản lý hệ thống.

**Platforms:**
1. **User Site (Staff Site):** For hospital staff (Receptionists, Doctors). Includes modules: M1 (Reception), M2 (Clinic), M3 (Billing/BHYT), M4 & M8 (Pharmacy).
2. **Admin Site (CMS):** The backend dashboard portal (M10, M11, M12) where administrators configure master catalogs (Service Items, Bed assignments, Payer types, PACS modalities, ICD-10).

**Module Codes (from WBS):**
- M1: Tiếp đón (Reception)
- M2: Phòng khám (Clinic/Examination)
- M3: Viện phí / BHYT (Billing/Insurance)
- M4 & M8: Dược (Pharmacy)
- M10, M11, M12: Quản trị hệ thống (Admin/CMS)

**Project Phase & Maturity:**

- **Current Phase**: MVP (Minimum Viable Product) — giai đoạn đầu, nhiều tính năng chưa hoàn thiện.
- **Design Status**: Figma chỉ mang tính tham khảo form và thông tin chức năng. Màu sắc, icon, font size chưa đồng nhất, chưa chính xác → KHÔNG tạo TC cho visual consistency (color, spacing, font).
- **Spec Status**: Một số tính năng chưa có spec đầy đủ. Nếu QC không thấy spec → bổ sung vào QnA để confirm với BA.
- **Naming Inconsistency**: Tên màn hình/chức năng giữa WBS, Figma, Spec có thể KHÔNG đồng bộ. QC chỉ cần quan tâm chức năng đó LÀM GÌ, không cần block vì tên không khớp.

**Tech Stack:**

- Backend: NestJS + Prisma + PostgreSQL
- Frontend: Next.js + Ant Design (antd)
- Multi-tenant: schema-per-tenant
- Print: HTML → PDF (fixed template form 6556)

**Test Environment URLs:**

- User Site (Reception/Doctor): `https://app.sota-his.com/login`
- Admin Site (Admin/Pharmacy): `https://admin.sota-his.com/login`
- BA Spec Portal: `https://docs.sota-his.com/docs/business/usecases`
- Figma File Key: `TLQ5SbHCXdzDKnRnHCZjiE`

**Terminology & Concepts:**

- **Empty BHYT Data**: Refers to any scenario where the Health Insurance code is missing or unidentifiable. "0 value" is functionally equivalent to "no insurance".
- **R4T**: Ready for Test — DEV đã hoàn thành, sẵn sàng cho QC.
- **Tiếp Viên**: Nhân viên tiếp đón (Receptionist).
- **PK**: Phòng khám (Clinic/Department room).
- **Đối tượng**: Patient payment type — BHYT (insurance) or Dịch vụ (self-pay/fee-for-service).
- **Tuyến**: Referral level — đúng tuyến (correct level), trái tuyến (wrong level), chuyển tuyến (referred).
- **Mức hưởng**: BHYT coverage rate (%) — depends on tuyến, loại khám (cấp cứu, chuyển tuyến, thông thường).
- **ĐKKCB**: Đăng ký khám chữa bệnh ban đầu — the primary healthcare facility registered on the BHYT card.
- **Cổng BHYT**: National Health Insurance Portal — receives XML Checkin/Checkout data.
- **XML 130**: The standard format for pushing insurance claims to the National BHYT Portal.
- **LIS / PACS**: External integrations. LIS is for Laboratory (Xét nghiệm), PACS is for Imaging (Chẩn đoán hình ảnh).
- **PII**: Patient Identifiable Information. Data privacy is strictly enforced.

---

**BHYT Business Rules Checklist (Mandatory for QA Analysis):**

When analyzing any function involving BHYT (health insurance), AI MUST check these scenarios:

1. **Card Validity**:
   - Thẻ BHYT hết hạn → system allows/blocks save? Warning message?
   - Thẻ BHYT chưa có hiệu lực (ngày bắt đầu > today) → behavior?
   - Mã thẻ BHYT sai format → validation message?

2. **Tuyến & Mức hưởng (Referral Level & Coverage Rate)**:
   - BN khám đúng tuyến (ĐKKCB trùng cơ sở hiện tại) → mức hưởng 100% (TYT) hoặc 80% (BV tuyến huyện+)
   - BN khám trái tuyến → mức hưởng giảm (40-60%)
   - BN cấp cứu → LUÔN hưởng đúng tuyến (bất kể ĐKKCB ở đâu) — Luật BHYT Điều 22
   - BN có giấy chuyển tuyến → hưởng đúng tuyến tại cơ sở được chuyển đến
   - Khi tick checkbox [Cấp cứu] hoặc [Chuyển tuyến] → system auto-recalculate mức hưởng?

3. **Chuyển tuyến (Referral)**:
   - Giấy chuyển tuyến → bắt buộc nhập "Mã cơ sở y tế chuyển đến/đi" (để đẩy cổng BHYT)
   - UI có trường nhập Mã BV chuyển tuyến không? Nếu thiếu → flag as missing field
   - Giấy chuyển tuyến hết hạn (>30 ngày hoặc theo quy định) → behavior?

4. **Đối tượng Switching (Patient Type Change)**:
   - BN có BHYT nhưng chọn khám Dịch vụ → thông tin BHYT ẩn/disable hay vẫn hiển thị?
   - BN đang nhập thông tin BHYT → chuyển sang Dịch vụ → data BHYT đã nhập xử lý thế nào?
   - BN Dịch vụ → chuyển sang BHYT giữa chừng → form behavior?

5. **Trẻ em & Đối tượng đặc biệt**:
   - Trẻ < 6 tuổi → miễn phí 100% (không cần thẻ BHYT riêng, dùng thẻ mẹ/cha)
   - Người nghèo, cận nghèo → mức hưởng khác
   - Tuổi tính chính xác theo ngày (không theo năm) — ảnh hưởng quyền lợi

6. **XML Checkin/Checkout (Cổng BHYT)**:
   - Timing: đẩy khi lưu tiếp đón hay action riêng?
   - Error handling: cổng timeout/lỗi → block save hay warning + retry?
   - Data required: mã thẻ, mã ĐKKCB, mã BV, loại tuyến, ngày khám

7. **Data Conflict & Overwrite**:
   - BN cũ (data loaded) + quét CCCD mới → thông tin mới ghi đè hay giữ cũ?
   - BN cũ + quét thẻ BHYT mới (thẻ gia hạn, đổi nơi ĐKKCB) → update hay tạo mới?
   - 2 BN trùng họ tên + địa chỉ, không có CCCD → cách phân biệt?

---

**MVP Scope (In-Scope):**

1. **Reception & Identification**: Enter/scan BHYT/CCCD QR, look up patient info + history, create internal patient profile, link identifiers, fast search, returning-patient flow (PATCH + new visit).
2. **Outpatient Care (Encounter-centric)**: Create encounter/visit, assign department/room/doctor, capture diagnosis + encounter notes, create orders for labs/diagnostics.
3. **Electronic Prescribing (eRx)**: Create prescriptions from encounter, **drug-drug interaction checks** as warnings (by severity), allow override with required reason (audited).
4. **Pharmacy**: Pharmacy approval, dispensing + record stock/dispensed status (transaction level).
5. **Billing & Payments**: Split BHYT vs self-pay per line item, calculate totals per encounter, record payments (cash/transfer).
6. **RBAC & Audit**: Action-level RBAC, immutable (append-only) audit logs. Minimum coverage: login/session, view/edit records, prescribing, pharmacy approve/dispense, payments, printing.
7. **Printing**: Print form 6556 with fixed template (HTML → PDF, no customization in MVP).

**MVP Out-of-Scope (DO NOT TEST):**

- Full BHYT claim submission (submission, responses, automated reconciliation) — MVP chỉ có lookup + settlement preparation snapshots.
- Customizable templates / drag-and-drop designer.
- Inpatient, Emergency, Surgery modules.
- Deep LIS/RIS/PACS integration (only placeholders/adapter patterns).
- Multi-facility user switching.

---

**Personas & Permissions (Full MVP):**

| Actor | Permissions |
|---|---|
| **Tiếp Viên (Receptionist)** | Reception, create encounter, BHYT lookup, printing |
| **Bác sĩ (Doctor)** | Encounters, orders, prescribing |
| **Dược sĩ (Pharmacist)** | Approve & dispense prescriptions |
| **Thu ngân (Cashier/Billing)** | Billing totals, record payments, print receipts |
| **Admin** | Facility config, users, roles, permissions |

---

**Critical Functional Requirements (FR):**

- **BHYT/CCCD lookup**: Target response time < 3s at P95 (depending on upstream).
- **Encounter timeline**: Show complete time-ordered view of orders/prescriptions/payments.
- **Drug interaction checks**: Warnings by severity; record `override_reason`. Allow override but MUST audit.
- **Pharmacy**: Prescription can be approved partially or all-at-once.
- **Billing**: Re-calculate when orders/meds change; store a "pricing snapshot" at time of finalization.

---

**Non-Functional Requirements (NFR):**

- **Multi-tenant isolation**: Facility = tenant. All business data belongs to exactly one tenant. Disallow cross-tenant queries except shared catalogs.
- **Audit immutability**: Append-only audit logs. Record: actor, tenant, time, action, entity, before/after diff, IP/user-agent.
- **Performance**: Encounter-centric queries (load encounter with orders/meds/payments) < 500ms at P95. Avoid N+1; use pagination & cursors.
- **Reliability**: Migrations automated + idempotent. Background jobs use retry/backoff.
- **Security**: Action-level RBAC at API boundary. Mask sensitive fields by role. Never commit secrets.

---

**Risks & Mitigations:**

- **BHYT integration**: SLA/format changes → adapter design, contract tests, caching + circuit breaker.
- **Multi-tenant schema**: Migration complexity → tenant-aware runner + standard tenant schema.
- **Drug interaction data**: Data quality issues → allow warn/override + track false positives.

---

**Testing Strategy:**

- **Outpatient/Inpatient Segregation**: Features or logic affecting patient types MUST segregate test cases for Outpatient and Inpatient. Do NOT merge them for test optimization even if the UI flow appears identical, because the underlying data schemas and financial parsing logic differ significantly.

- **TC Grouping by Screen**: Test cases được gộp theo **Screen** (màn hình), không tách ra nhiều file. Ví dụ: màn Danh sách + Xóa bản ghi trong danh sách đó = cùng 1 TC sheet. Mỗi Screen = 1 file test case duy nhất.

- **MVP Testing Focus**: Ưu tiên test chức năng core (business logic, data integrity). KHÔNG ưu tiên test cosmetic UI (color, font, spacing) vì design chưa finalize.
