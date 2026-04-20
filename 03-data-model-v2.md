# Data Model v2 — TLXN HBSS

**Date**: 2026-04-18
**Based on**: Review 8 tables HBSS v1 + Q&A với Nam (6 câu chiến lược)

---

## 🔑 Design decisions từ Q&A

| Q | Nam trả lời | Hệ quả kiến trúc |
|---|---|---|
| Q1 | Payment từ tài chính → move vào HBSS | `Theo dõi Thanh toán` phải first-class entity, + `payment_schedules` dự kiến |
| Q2 | Track tên NV cụ thể | `Nhân sự` entity + `Phân công nhân sự` (N-N Nhân sự × project × Vai trò × period) |
| Q3 | 1 khách = 1 dự án (nếu > tạo file riêng) | `Danh mục khách hàng` vẫn tách nhưng relationship 1-1 đơn giản |
| Q4 | Scorecard **monthly** đo hài lòng KH | `Chấm điểm đối tác` tách khỏi Ticket, trigger monthly, field `month_year` |
| Q5 | File hiện tại = template | **Migration cực đơn giản**: update template → dự án mới clone v2 |
| Q6 | Nhiều loại HĐ (TLXN, Thiết kế, Thi công...) | `Quản lý Hợp đồng` entity với `contract_type` enum |

---

## 🏗️ 3-LAYER ARCHITECTURE

```
LAYER 1 — MASTER BASE (Lark, shared toàn công ty)
  8 table: Danh mục khách hàng, Nhân sự, Danh mục đối tác, Danh mục hạng mục, Thư viện đầu việc,
  Thư viện mẫu sản phẩm, Loại hợp đồng mẫu, Danh mục vật tư
  → Ít thay đổi, reference data

LAYER 2 — PER-PROJECT BASE (Lark template v2)
  17 table: Thông tin Dự án, Tiến độ dự án, Quản lý các bên liên quan, Checklist công việc chi tiết,
  Nhật ký công trình, Báo cáo nghiệm thu nội bộ, Quản lý vấn đề và Sự cố, Báo cáo tuần, Chấm điểm đối tác,
  Quản lý Hợp đồng, Báo giá & HSNL, Dự toán hạng mục, Theo dõi Thanh toán, Quản lý Phát sinh,
  Kho tài liệu dự án, Công tác Nghiệm thu, Cấu hình Base
  → Mỗi dự án 1 file, clone từ template

LAYER 3 — WAREHOUSE (PostgreSQL/Supabase)
  ~28 entity + 8 analytics views
  → Cross-project query, AI agent, dashboard, benchmark
```

---

## LAYER 1 — MASTER BASE (8 tables)

### 1.1 `Danh mục khách hàng`

| Field | Type | Ghi chú |
|---|---|---|
| Mã khách hàng | Text (primary) | Auto `CUS-2026-001` |
| Họ và tên | Text | |
| phone_primary | Phone | |
| phone_secondary | Phone | |
| Email | Email | |
| id_card_number | Text | CCCD (PII — RBAC) |
| date_of_birth | DateTime | |
| address_permanent | Text | |
| occupation | Text | |
| referral_source | SingleSelect | FB / Google / Referral / Khác |
| lark_chat_page_id | Text | Thay "Page ID (CĐT)" rải rác |
| created_at | CreatedTime | |
| Ghi chú | Text | |

### 1.2 `Danh mục nhân sự`

| Field | Type |
|---|---|
| Mã nhân viên | Text (primary) — `STAFF-001` |
| Họ và tên | Text |
| role_primary | SingleSelect (PM / AA / CA / Account / PO / PD / Sale / TK) |
| roles_secondary | MultiSelect (kiêm nhiệm) |
| email_work | Email |
| Số điện thoại | Phone |
| lark_user_id | Text |
| active | Checkbox |
| joined_date | DateTime |
| left_date | DateTime (NULL nếu đang làm) |

### 1.3 `Danh mục đối tác`

| Field | Type | Ghi chú |
|---|---|---|
| Mã đối tác | Text (primary) — `VEN-2026-001` | |
| company_name | Text | |
| vendor_type | MultiSelect (NTP-Thi công / NTP-Thiết kế / NCC-Vật tư / NCC-Thiết bị) | |
| category_specialization | MultiSelect → `Danh mục hạng mục` | |
| contact_person | Text | |
| phone_company | Phone | |
| phone_contact | Phone | |
| address | Text | |
| tax_code | Text | MST |
| bank_info | Text | |
| Trạng thái | SingleSelect (Active / Trial / Blacklisted) | |
| overall_score | Number (lookup từ Chấm điểm đối tác) | **Nơi aggregate scorecard** |
| projects_count | Number (lookup) | |
| Ghi chú | Text | |

### 1.4 `Danh mục hạng mục`

| Field | Type | Ghi chú |
|---|---|---|
| category_name | Text (primary) | 44 hạng mục (Tiếp nhận, BTCT móng, Thạch cao…) |
| phase | SingleSelect | Trước xây / Phần thô / Phần hoàn thiện / Bàn giao |
| display_order | Number | |
| typical_duration_days | Number | |
| active | Checkbox | |

→ **Xóa duplicate 3-way** hiện tại. Các table khác Link về đây.

### 1.5 `Thư viện đầu việc`

| Field | Type | Ghi chú |
|---|---|---|
| Mã đầu việc | Text (primary) — `W-001` | |
| Tên đầu việc | Text | VD: "Biên bản bàn giao ranh mốc" |
| Hạng mục | SingleLink → `Danh mục hạng mục` | |
| phase | SingleSelect | |
| priority | SingleSelect (Mốc quan trọng / Thường) | |
| default_owner_role | SingleSelect | |
| standard_doc | Attachment | Tiêu chuẩn thi công |
| sample_image | Attachment | Sản phẩm mẫu |
| preparation_note | Text | Chuẩn bị thi công |
| during_note | Text | Trong khi thi công |
| after_note | Text | Sau khi thi công |

→ Thay vì duplicate 36+ work items/dự án, dự án Link tới library.

### 1.6 `Thư viện mẫu sản phẩm`

| Field | Type |
|---|---|
| sample_code | Text (primary) |
| Hạng mục | SingleLink → `Danh mục hạng mục` |
| description | Text |
| images | Attachment |

### 1.7 `Loại hợp đồng mẫu` **(NEW)**

| Field | Type | Ghi chú |
|---|---|---|
| type_code | Text (primary) | `CT-TLXN`, `CT-DESIGN`, `CT-MAIN-CONSTRUCTION`, `CT-SUB`, `CT-SUPPLIER` |
| type_name | Text | |
| typical_template_doc | Attachment | Mẫu HĐ chuẩn |
| requires_signature_from | MultiSelect (Khách / TLXN / NTP / NCC) | |
| typical_payment_terms | Text | |

### 1.8 `material_catalog` *(có thể skip v2, làm sau)*

| Field | Type |
|---|---|
| material_code | Text (primary) |
| material_name | Text |
| Hạng mục | SingleSelect (Xi măng / Thép / Gạch / Sơn…) |
| unit | SingleSelect (kg / m² / m³ / viên…) |
| typical_price_vnd | Currency |

---

## LAYER 2 — PER-PROJECT TEMPLATE v2 (17 tables)

### 2.1 `Thông tin Dự án` **(NEW — 1 record/file, CORE)**

| Field | Type | Ghi chú |
|---|---|---|
| Mã dự án | Text (primary) | Unique toàn công ty `PRJ-2026-042` |
| Tên dự án | Text | VD: "Nhà anh A Quận 7" |
| Khách hàng | SingleLink → `Danh mục khách hàng` | |
| Địa chỉ công trình | Text | |
| Quận/Huyện | SingleSelect | Quận/huyện |
| Tỉnh/Thành phố | SingleSelect | TP.HCM / Hà Nội / Đà Nẵng |
| land_area_m2 | Number | |
| building_area_m2 | Number | |
| floors_count | Number | |
| basement_count | Number | |
| building_type | SingleSelect | Phố / Biệt thự / Studio / Shophouse / Mini |
| architecture_style | MultiSelect | Hiện đại / Tân cổ điển / Scandinavian / Industrial |
| Tổng ngân sách (VNĐ) | Currency | Ngân sách dự kiến |
| package_sku | SingleSelect | Gói TLXN khách chọn (liên kết Phase 2) |
| contract_sign_date | DateTime | |
| construction_start_date | DateTime | Dự kiến |
| construction_start_actual | DateTime | Thực tế |
| handover_planned_date | DateTime | Dự kiến |
| handover_actual_date | DateTime | Thực tế |
| Trạng thái | SingleSelect | Pre-Hợp đồng liên quan / Signed / Design / Construction / Handover / Warranty / Closed |
| lark_chat_customer_id | Text | Thay "Page ID (CĐT)" rải rác |
| lark_chat_internal_id | Text | Thay "Page ID" nội bộ |

### 2.2 `Phân công nhân sự` **(NEW)**

| Field | Type |
|---|---|
| assignment_id | Text (primary) |
| Nhân sự | SingleLink → `Danh mục nhân sự` |
| Vai trò | SingleSelect (PM / AA / CA / Account / PO / PD) |
| assigned_from | DateTime |
| assigned_to | DateTime (NULL = đang active) |
| Ghi chú | Text |

### 2.3 `Quản lý các bên liên quan`

| Field | Type |
|---|---|
| stakeholder_id | Text (primary) |
| Đơn vị/Đối tác | SingleLink → `Danh mục đối tác` |
| Hạng mục | SingleLink → `Danh mục hạng mục` |
| role_in_project | SingleSelect (Thi công chính / Phụ / Cung cấp) |
| Hợp đồng liên quan | SingleLink → `Quản lý Hợp đồng` |
| Trạng thái | SingleSelect (Đang làm / Đã xong / Terminate) |
| joined_date | DateTime |
| end_date | DateTime |

### 2.4 `Tiến độ dự án`

Thay cho Table 1 "Chức năng bảng = Tiến độ".

| Field | Type |
|---|---|
| milestone_name | Text (primary) |
| Hạng mục | SingleLink → `Danh mục hạng mục` |
| phase | Lookup từ Hạng mục |
| Bắt đầu dự kiến | DateTime |
| Kết thúc dự kiến | DateTime |
| Bắt đầu thực tế | DateTime |
| Kết thúc thực tế | DateTime |
| extension_date | DateTime |
| % Hoàn thành | Progress |
| is_critical | Checkbox |
| owner_role | SingleSelect |
| owner_staff | SingleLink → `Danh mục nhân sự` |
| parent_milestone | SingleLink (self-ref) |
| Ghi chú | Text |

### 2.5 `Checklist công việc chi tiết`

| Field | Type |
|---|---|
| task_code | Text (primary) — "1.1", "3.2" |
| task_name | Text |
| work_library_ref | SingleLink → `Thư viện đầu việc` (optional) |
| Mốc tiến độ | SingleLink → `Tiến độ dự án` |
| parent_task | SingleLink (self-ref) |
| owner_role | SingleSelect |
| owner_staff | SingleLink → `Danh mục nhân sự` |
| Bắt đầu dự kiến | DateTime |
| Kết thúc dự kiến | DateTime |
| extension_date | DateTime |
| Thực tế hoàn thành | DateTime |
| % Hoàn thành | Progress |
| Trạng thái | SingleSelect (Chưa / Đang / Xong / Treo / Hủy) |
| report_attachment | Attachment |
| Ghi chú | Text |

### 2.6 `Nhật ký công trình`

| Field | Type |
|---|---|
| log_date | DateTime (primary, auto) |
| reporter_name | Text |
| Đơn vị/Đối tác | SingleLink → `Danh mục đối tác` |
| work_today | Text |
| work_tomorrow | Text |
| issues_proposals | Text |
| workers_count | Number |
| delay_risk | SingleSelect (Có / Không) |
| daily_status | SingleSelect (Cả ngày / Sáng nghỉ / Chiều nghỉ) |
| site_photos | Attachment |
| Ghi chú | Text |

### 2.7 `Báo cáo nghiệm thu nội bộ`

| Field | Type |
|---|---|
| report_date | DateTime (primary) |
| Hạng mục | SingleLink → `Danh mục hạng mục` |
| work_item | SingleLink → `Thư viện đầu việc` |
| completion_percent | Progress |
| quality | SingleSelect (Đạt / Cần cải thiện) |
| confirmed_complete | Checkbox |
| photos_ok | Attachment |
| photos_issues | Attachment |
| Ghi chú | Text |

### 2.8 `Quản lý vấn đề và Sự cố` (tách từ Table 3)

| Field | Type |
|---|---|
| issue_id | Text (primary) |
| created_date | CreatedTime |
| creator | SingleLink → `Danh mục nhân sự` |
| description | Text |
| related_areas | MultiSelect (Thiết kế / Pháp lý / Chất lượng / Chi phí / Tiến độ / An toàn / Khác) |
| related_category | SingleLink → `Danh mục hạng mục` |
| media | Attachment |
| deadline | DateTime |
| Trạng thái | SingleSelect (Đang / Xong / Treo) |
| resolution_action | Text |
| resolved_date | DateTime |
| assignee | SingleLink → `Danh mục nhân sự` |
| Ghi chú | Text |

### 2.9 `Báo cáo tuần` (tách từ Table 3)

| Field | Type |
|---|---|
| report_week | DateTime (primary — thứ 2 của tuần) |
| Mốc tiến độ | SingleLink → `Tiến độ dự án` |
| progress_assessment | SingleSelect (Đúng / Chậm / Treo / Vượt) |
| design_compliance | SingleSelect (✅ Đúng / ⚠️ Sai lệch / ❌ Nghiêm trọng) |
| legal_note | Text |
| quality_note | Text |
| reviewer | SingleSelect (QLDA / CĐT) |
| Ghi chú | Text |

### 2.10 `Chấm điểm đối tác` **(MONTHLY cadence)**

| Field | Type | Weight |
|---|---|---|
| scorecard_id | Text (primary) | |
| month_year | DateTime | VD: 04/2026 |
| Đơn vị/Đối tác | SingleLink → `Danh mục đối tác` | |
| Hạng mục | SingleLink → `Danh mục hạng mục` | |
| score_cooperation | Rating 1-5 | 20% |
| score_report_quality | Rating 1-5 | 15% |
| score_response_time | Rating 1-5 | 15% |
| score_cost_control | Rating 1-5 | 15% |
| score_supervision | Rating 1-5 | 15% |
| score_consulting | Rating 1-5 | 10% |
| score_vendor_support | Rating 1-5 | 10% |
| total_score | Formula (weighted sum) | auto |
| improvement_action | Text | |
| reviewer | SingleSelect (PM / QLDA / CĐT) | |

**Rule**: UNIQUE (Đơn vị/Đối tác, month_year, Hạng mục) — tránh chấm 2 lần.

### 2.11 `Quản lý Hợp đồng` **(NEW)**

| Field | Type |
|---|---|
| contract_id | Text (primary) — `CTR-PRJ042-001` |
| contract_type | SingleLink → `Loại hợp đồng mẫu` |
| title | Text |
| party_a | SingleLink → `Danh mục khách hàng` / TLXN |
| party_b | SingleLink → `Danh mục đối tác` / TLXN |
| sign_date | DateTime |
| effective_date | DateTime |
| end_date | DateTime |
| Giá trị hợp đồng (VNĐ) | Currency |
| payment_terms | Text |
| scanned_doc | Attachment |
| Trạng thái | SingleSelect (Draft / Active / Completed / Terminated) |
| related_category | SingleLink → `Danh mục hạng mục` |
| Ghi chú | Text |

### 2.12 `Báo giá & HSNL`

| Field | Type |
|---|---|
| quote_id | Text (primary) |
| quote_date | DateTime |
| Hạng mục | SingleLink → `Danh mục hạng mục` |
| Đơn vị/Đối tác | SingleLink → `Danh mục đối tác` |
| Số tiền (VNĐ) | Currency |
| quote_doc | Attachment |
| capability_docs | Attachment (HSNL, ảnh thi công cũ) |
| selected | Checkbox |
| Ghi chú | Text |

### 2.13 `Dự toán hạng mục`

| Field | Type |
|---|---|
| budget_item_id | Text (primary) |
| Hạng mục | SingleLink → `Danh mục hạng mục` |
| description | Text |
| planned_amount_vnd | Currency |
| proposed_date | DateTime |
| proposal_doc | Attachment |
| Trạng thái | SingleSelect (Chờ / Duyệt / Hủy) |
| Ghi chú | Text |

### 2.14 `Theo dõi Thanh toán` **(MAJOR UPGRADE — Q1 integration)**

| Field | Type | Ghi chú |
|---|---|---|
| payment_id | Text (primary) | |
| Ngày thanh toán | DateTime | |
| Hợp đồng liên quan | SingleLink → `Quản lý Hợp đồng` | **Bắt buộc** |
| payment_type | SingleSelect | Thanh toán HĐ / Phát sinh / Tạm ứng / Hoàn ứng |
| installment_no | Number | Đợt thứ mấy |
| Người thụ hưởng | SingleLink → `Danh mục đối tác` hoặc `Danh mục khách hàng` | |
| amount_contract_vnd | Currency | Theo HĐ |
| Số tiền chi thực tế | Currency | Thực chi |
| amount_variance_vnd | Formula | Chênh lệch auto |
| payment_method | SingleSelect | Chuyển khoản / Tiền mặt / Séc |
| receipt_doc | Attachment | Bill/phiếu chi |
| approved_by | SingleLink → `Danh mục nhân sự` | |
| Ghi chú | Text | |

### 2.15 `Quản lý Phát sinh` **(NEW)**

| Field | Type |
|---|---|
| co_id | Text (primary) |
| requested_date | DateTime |
| requested_by | SingleSelect (Khách / PM / NTP) |
| description | Text |
| affected_categories | MultiSelect → `Danh mục hạng mục` |
| cost_impact_vnd | Currency |
| schedule_impact_days | Number |
| approval_status | SingleSelect (Chờ / Duyệt / Reject) |
| approved_by | SingleLink → `Danh mục nhân sự` |
| approved_date | DateTime |
| supporting_docs | Attachment |
| related_contract | SingleLink → `Quản lý Hợp đồng` |

### 2.16 `Kho tài liệu dự án`

| Field | Type |
|---|---|
| doc_id | Text (primary) |
| upload_date | CreatedTime |
| uploaded_by | SingleLink → `Danh mục nhân sự` |
| Hạng mục | SingleSelect (Pháp lý / Thiết kế / Thi công / HĐ / Phát sinh / Khác) |
| title | Text |
| file | Attachment |
| related_category | SingleLink → `Danh mục hạng mục` |
| related_contract | SingleLink → `Quản lý Hợp đồng` |
| Ghi chú | Text |

### 2.17 `Công tác Nghiệm thu` **(NEW)**

| Field | Type |
|---|---|
| inspection_id | Text (primary) |
| inspection_date | DateTime |
| Hạng mục | SingleLink → `Danh mục hạng mục` |
| Mốc tiến độ | SingleLink → `Tiến độ dự án` |
| inspector_internal | SingleLink → `Danh mục nhân sự` |
| inspector_customer | Checkbox |
| result | SingleSelect (Pass / Conditional / Fail) |
| issues_found | Text |
| photos | Attachment |
| signed_doc | Attachment |
| follow_up_needed | Checkbox |
| follow_up_deadline | DateTime |

### 2.18 `Cấu hình Base`

| Field | Type |
|---|---|
| setting_key | Text (primary) |
| setting_value | Text |
| Ghi chú | Text |

---

## LAYER 3 — WAREHOUSE SCHEMA

### Master entities (8)
`Danh mục khách hàng, Nhân sự, Danh mục đối tác, Danh mục hạng mục, Thư viện đầu việc, Thư viện mẫu sản phẩm, Loại hợp đồng mẫu, Danh mục vật tư`

### Project entities (19)
```
projects (1 row/project)
Phân công nhân sự         (Nhân sự × project × Vai trò)
Quản lý các bên liên quan        (Đơn vị/Đối tác × project)
Tiến độ dự án
Checklist công việc chi tiết
Nhật ký công trình
Báo cáo nghiệm thu nội bộ
Quản lý vấn đề và Sự cố
Báo cáo tuần
Chấm điểm đối tác           (monthly)
Quản lý Hợp đồng
Báo giá & HSNL
Dự toán hạng mục
Theo dõi Thanh toán
payment_schedules           (lịch dự kiến, khác actual)
Quản lý Phát sinh
Kho tài liệu dự án
Công tác Nghiệm thu
Cấu hình Base
```

### Analytics views (materialized)
```
vendor_performance_monthly        (agg scorecard)
vendor_performance_lifetime       (cross-project)
staff_performance_monthly
project_progress_daily
cost_benchmarks_by_category_area  (giá tb theo hạng mục × diện tích)
cost_benchmarks_by_project_type   (theo loại nhà × địa bàn)
issue_resolution_metrics          (avg time to resolve)
payment_cashflow_monthly
```

### Constraints quan trọng
- `projects.customer_id` UNIQUE (Q3: 1 khách = 1 dự án)
- `Theo dõi Thanh toán.contract_id` NOT NULL (mọi payment phải link HĐ)
- `Chấm điểm đối tác` UNIQUE (vendor_id, month_year, category_id)
- `Phân công nhân sự` UNIQUE (project_id, staff_id, Vai trò, valid_period)

---

## 🚀 MIGRATION PLAN

Vì **Q5: file hiện tại = template** → migration cực đơn giản.

### Bước 1 — Setup Master Base (tuần 1)
- Tạo Base Lark mới "Master Data TLXN"
- 8 table master
- Seed data:
  - `Danh mục hạng mục`: import 44 hạng mục
  - `Thư viện đầu việc`: import 36 checklist items
  - `Thư viện mẫu sản phẩm`: import từ Table 8
  - `Loại hợp đồng mẫu`: nhập 5-6 loại HĐ
  - `Nhân sự`: danh sách nhân viên hiện có
  - `Danh mục khách hàng`: seed từ dự án đang chạy
  - `Danh mục đối tác`: dedupe từ "Tên đơn vị thi công" hiện có

### Bước 2 — Tạo template v2 (tuần 2)
- Tạo Base Lark "HBSS Template v2"
- 17 table với schema v2
- Config SingleLink → Master Base
- Test clone → check link hoạt động đúng

### Bước 3 — Dry run 1 dự án (tuần 3)
- Dự án đầu năm 2026 dùng template v2
- PM team dùng → feedback UX
- Fix → template v2.1

### Bước 4 — Rollout (tuần 4+)
- Mọi dự án mới clone template v2
- Dự án cũ giữ nguyên
- Training PM/CA/QA

### Bước 5 — Warehouse setup (tháng 2-3, song song)
- Supabase project
- Schema 27 entity
- n8n webhook → Supabase
- Reconciliation job 15 phút
- Dashboard prototype

### Bước 6 — Backfill chọn lọc (tháng 4+)
- Chọn 10-20 dự án cũ điển hình (đa dạng)
- Import semi-auto vào warehouse
- Có benchmark cho AI agent

---

## 📌 OPEN ITEMS (Phase 2)

1. **SKU definition** — Gói TLXN chưa define, cần Phase 2 Sales chốt
2. **Material catalog** — skip v2, làm khi track vật tư chi tiết
3. **Customer portal integration** — dashboard khách hiện có → mapping field mới
4. **Warehouse field-level spec** — SQL DDL chi tiết (có thể làm doc riêng)
5. **Row-level security** cho portal khách (C3: khách tự xem data dự án họ)

---

## Next actions options

- **[A]** Vẽ ERD chi tiết (text/mermaid)
- **[B]** Detailed spec 3-4 table critical (Thông tin Dự án, Theo dõi Thanh toán, Quản lý Hợp đồng, Chấm điểm đối tác)
- **[C]** Kế hoạch setup Master Base (bấm gì trên Lark UI)
- **[D]** Chuyển sang Phase 2 (Sales) — đồng bộ data model cho cả QLDA + Sales
