# Data Model v2 — TLXN HBSS

**Date**: 2026-04-18
**Based on**: Review 8 tables HBSS v1 + Q&A với Nam (6 câu chiến lược)

---

## 🔑 Design decisions từ Q&A

| Q | Nam trả lời | Hệ quả kiến trúc |
|---|---|---|
| Q1 | Payment từ tài chính → move vào HBSS | `payments` phải first-class entity, + `payment_schedules` dự kiến |
| Q2 | Track tên NV cụ thể | `staff` entity + `project_assignments` (N-N staff × project × role × period) |
| Q3 | 1 khách = 1 dự án (nếu > tạo file riêng) | `customers` vẫn tách nhưng relationship 1-1 đơn giản |
| Q4 | Scorecard **monthly** đo hài lòng KH | `vendor_scorecards` tách khỏi Ticket, trigger monthly, field `month_year` |
| Q5 | File hiện tại = template | **Migration cực đơn giản**: update template → dự án mới clone v2 |
| Q6 | Nhiều loại HĐ (TLXN, Thiết kế, Thi công...) | `contracts` entity với `contract_type` enum |

---

## 🏗️ 3-LAYER ARCHITECTURE

```
LAYER 1 — MASTER BASE (Lark, shared toàn công ty)
  8 table: customers, staff, vendors, categories, work_library,
  sample_products, contract_types, materials
  → Ít thay đổi, reference data

LAYER 2 — PER-PROJECT BASE (Lark template v2)
  17 table: project_info, milestones, stakeholders, tasks,
  daily_logs, task_reports, issues, weekly_reports, scorecards,
  contracts, quotes, budget_items, payments, change_orders,
  documents, inspections, settings
  → Mỗi dự án 1 file, clone từ template

LAYER 3 — WAREHOUSE (PostgreSQL/Supabase)
  ~28 entity + 8 analytics views
  → Cross-project query, AI agent, dashboard, benchmark
```

---

## LAYER 1 — MASTER BASE (8 tables)

### 1.1 `customers_master`

| Field | Type | Ghi chú |
|---|---|---|
| customer_code | Text (primary) | Auto `CUS-2026-001` |
| full_name | Text | |
| phone_primary | Phone | |
| phone_secondary | Phone | |
| email | Email | |
| id_card_number | Text | CCCD (PII — RBAC) |
| date_of_birth | DateTime | |
| address_permanent | Text | |
| occupation | Text | |
| referral_source | SingleSelect | FB / Google / Referral / Khác |
| lark_chat_page_id | Text | Thay "Page ID (CĐT)" rải rác |
| created_at | CreatedTime | |
| notes | Text | |

### 1.2 `staff_master`

| Field | Type |
|---|---|
| staff_code | Text (primary) — `STAFF-001` |
| full_name | Text |
| role_primary | SingleSelect (PM / AA / CA / Account / PO / PD / Sale / TK) |
| roles_secondary | MultiSelect (kiêm nhiệm) |
| email_work | Email |
| phone | Phone |
| lark_user_id | Text |
| active | Checkbox |
| joined_date | DateTime |
| left_date | DateTime (NULL nếu đang làm) |

### 1.3 `vendors_master`

| Field | Type | Ghi chú |
|---|---|---|
| vendor_code | Text (primary) — `VEN-2026-001` | |
| company_name | Text | |
| vendor_type | MultiSelect (NTP-Thi công / NTP-Thiết kế / NCC-Vật tư / NCC-Thiết bị) | |
| category_specialization | MultiSelect → `categories_master` | |
| contact_person | Text | |
| phone_company | Phone | |
| phone_contact | Phone | |
| address | Text | |
| tax_code | Text | MST |
| bank_info | Text | |
| status | SingleSelect (Active / Trial / Blacklisted) | |
| overall_score | Number (lookup từ scorecards) | **Nơi aggregate scorecard** |
| projects_count | Number (lookup) | |
| notes | Text | |

### 1.4 `categories_master`

| Field | Type | Ghi chú |
|---|---|---|
| category_name | Text (primary) | 44 hạng mục (Tiếp nhận, BTCT móng, Thạch cao…) |
| phase | SingleSelect | Trước xây / Phần thô / Phần hoàn thiện / Bàn giao |
| display_order | Number | |
| typical_duration_days | Number | |
| active | Checkbox | |

→ **Xóa duplicate 3-way** hiện tại. Các table khác Link về đây.

### 1.5 `work_library`

| Field | Type | Ghi chú |
|---|---|---|
| work_item_code | Text (primary) — `W-001` | |
| work_name | Text | VD: "Biên bản bàn giao ranh mốc" |
| category | SingleLink → `categories_master` | |
| phase | SingleSelect | |
| priority | SingleSelect (Mốc quan trọng / Thường) | |
| default_owner_role | SingleSelect | |
| standard_doc | Attachment | Tiêu chuẩn thi công |
| sample_image | Attachment | Sản phẩm mẫu |
| preparation_note | Text | Chuẩn bị thi công |
| during_note | Text | Trong khi thi công |
| after_note | Text | Sau khi thi công |

→ Thay vì duplicate 36+ work items/dự án, dự án Link tới library.

### 1.6 `sample_products`

| Field | Type |
|---|---|
| sample_code | Text (primary) |
| category | SingleLink → `categories_master` |
| description | Text |
| images | Attachment |

### 1.7 `contract_types_master` **(NEW)**

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
| category | SingleSelect (Xi măng / Thép / Gạch / Sơn…) |
| unit | SingleSelect (kg / m² / m³ / viên…) |
| typical_price_vnd | Currency |

---

## LAYER 2 — PER-PROJECT TEMPLATE v2 (17 tables)

### 2.1 `project_info` **(NEW — 1 record/file, CORE)**

| Field | Type | Ghi chú |
|---|---|---|
| project_code | Text (primary) | Unique toàn công ty `PRJ-2026-042` |
| project_name | Text | VD: "Nhà anh A Quận 7" |
| customer | SingleLink → `customers_master` | |
| site_address | Text | |
| district | SingleSelect | Quận/huyện |
| city | SingleSelect | TP.HCM / Hà Nội / Đà Nẵng |
| land_area_m2 | Number | |
| building_area_m2 | Number | |
| floors_count | Number | |
| basement_count | Number | |
| building_type | SingleSelect | Phố / Biệt thự / Studio / Shophouse / Mini |
| architecture_style | MultiSelect | Hiện đại / Tân cổ điển / Scandinavian / Industrial |
| total_budget_vnd | Currency | Ngân sách dự kiến |
| package_sku | SingleSelect | Gói TLXN khách chọn (liên kết Phase 2) |
| contract_sign_date | DateTime | |
| construction_start_date | DateTime | Dự kiến |
| construction_start_actual | DateTime | Thực tế |
| handover_planned_date | DateTime | Dự kiến |
| handover_actual_date | DateTime | Thực tế |
| status | SingleSelect | Pre-contract / Signed / Design / Construction / Handover / Warranty / Closed |
| lark_chat_customer_id | Text | Thay "Page ID (CĐT)" rải rác |
| lark_chat_internal_id | Text | Thay "Page ID" nội bộ |

### 2.2 `project_assignments` **(NEW)**

| Field | Type |
|---|---|
| assignment_id | Text (primary) |
| staff | SingleLink → `staff_master` |
| role | SingleSelect (PM / AA / CA / Account / PO / PD) |
| assigned_from | DateTime |
| assigned_to | DateTime (NULL = đang active) |
| notes | Text |

### 2.3 `project_stakeholders`

| Field | Type |
|---|---|
| stakeholder_id | Text (primary) |
| vendor | SingleLink → `vendors_master` |
| category | SingleLink → `categories_master` |
| role_in_project | SingleSelect (Thi công chính / Phụ / Cung cấp) |
| contract | SingleLink → `project_contracts` |
| status | SingleSelect (Đang làm / Đã xong / Terminate) |
| joined_date | DateTime |
| end_date | DateTime |

### 2.4 `project_milestones`

Thay cho Table 1 "Chức năng bảng = Tiến độ".

| Field | Type |
|---|---|
| milestone_name | Text (primary) |
| category | SingleLink → `categories_master` |
| phase | Lookup từ category |
| planned_start | DateTime |
| planned_end | DateTime |
| actual_start | DateTime |
| actual_end | DateTime |
| extension_date | DateTime |
| progress_percent | Progress |
| is_critical | Checkbox |
| owner_role | SingleSelect |
| owner_staff | SingleLink → `staff_master` |
| parent_milestone | SingleLink (self-ref) |
| notes | Text |

### 2.5 `project_tasks`

| Field | Type |
|---|---|
| task_code | Text (primary) — "1.1", "3.2" |
| task_name | Text |
| work_library_ref | SingleLink → `work_library` (optional) |
| milestone | SingleLink → `project_milestones` |
| parent_task | SingleLink (self-ref) |
| owner_role | SingleSelect |
| owner_staff | SingleLink → `staff_master` |
| planned_start | DateTime |
| planned_end | DateTime |
| extension_date | DateTime |
| actual_completion | DateTime |
| progress_percent | Progress |
| status | SingleSelect (Chưa / Đang / Xong / Treo / Hủy) |
| report_attachment | Attachment |
| notes | Text |

### 2.6 `project_daily_logs`

| Field | Type |
|---|---|
| log_date | DateTime (primary, auto) |
| reporter_name | Text |
| vendor | SingleLink → `vendors_master` |
| work_today | Text |
| work_tomorrow | Text |
| issues_proposals | Text |
| workers_count | Number |
| delay_risk | SingleSelect (Có / Không) |
| daily_status | SingleSelect (Cả ngày / Sáng nghỉ / Chiều nghỉ) |
| site_photos | Attachment |
| notes | Text |

### 2.7 `project_task_reports`

| Field | Type |
|---|---|
| report_date | DateTime (primary) |
| category | SingleLink → `categories_master` |
| work_item | SingleLink → `work_library` |
| completion_percent | Progress |
| quality | SingleSelect (Đạt / Cần cải thiện) |
| confirmed_complete | Checkbox |
| photos_ok | Attachment |
| photos_issues | Attachment |
| notes | Text |

### 2.8 `project_issues` (tách từ Table 3)

| Field | Type |
|---|---|
| issue_id | Text (primary) |
| created_date | CreatedTime |
| creator | SingleLink → `staff_master` |
| description | Text |
| related_areas | MultiSelect (Thiết kế / Pháp lý / Chất lượng / Chi phí / Tiến độ / An toàn / Khác) |
| related_category | SingleLink → `categories_master` |
| media | Attachment |
| deadline | DateTime |
| status | SingleSelect (Đang / Xong / Treo) |
| resolution_action | Text |
| resolved_date | DateTime |
| assignee | SingleLink → `staff_master` |
| notes | Text |

### 2.9 `project_weekly_reports` (tách từ Table 3)

| Field | Type |
|---|---|
| report_week | DateTime (primary — thứ 2 của tuần) |
| milestone | SingleLink → `project_milestones` |
| progress_assessment | SingleSelect (Đúng / Chậm / Treo / Vượt) |
| design_compliance | SingleSelect (✅ Đúng / ⚠️ Sai lệch / ❌ Nghiêm trọng) |
| legal_note | Text |
| quality_note | Text |
| reviewer | SingleSelect (QLDA / CĐT) |
| notes | Text |

### 2.10 `vendor_scorecards` **(MONTHLY cadence)**

| Field | Type | Weight |
|---|---|---|
| scorecard_id | Text (primary) | |
| month_year | DateTime | VD: 04/2026 |
| vendor | SingleLink → `vendors_master` | |
| category | SingleLink → `categories_master` | |
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

**Rule**: UNIQUE (vendor, month_year, category) — tránh chấm 2 lần.

### 2.11 `project_contracts` **(NEW)**

| Field | Type |
|---|---|
| contract_id | Text (primary) — `CTR-PRJ042-001` |
| contract_type | SingleLink → `contract_types_master` |
| title | Text |
| party_a | SingleLink → `customers_master` / TLXN |
| party_b | SingleLink → `vendors_master` / TLXN |
| sign_date | DateTime |
| effective_date | DateTime |
| end_date | DateTime |
| total_value_vnd | Currency |
| payment_terms | Text |
| scanned_doc | Attachment |
| status | SingleSelect (Draft / Active / Completed / Terminated) |
| related_category | SingleLink → `categories_master` |
| notes | Text |

### 2.12 `project_quotes`

| Field | Type |
|---|---|
| quote_id | Text (primary) |
| quote_date | DateTime |
| category | SingleLink → `categories_master` |
| vendor | SingleLink → `vendors_master` |
| amount_vnd | Currency |
| quote_doc | Attachment |
| capability_docs | Attachment (HSNL, ảnh thi công cũ) |
| selected | Checkbox |
| notes | Text |

### 2.13 `project_budget_items`

| Field | Type |
|---|---|
| budget_item_id | Text (primary) |
| category | SingleLink → `categories_master` |
| description | Text |
| planned_amount_vnd | Currency |
| proposed_date | DateTime |
| proposal_doc | Attachment |
| status | SingleSelect (Chờ / Duyệt / Hủy) |
| notes | Text |

### 2.14 `project_payments` **(MAJOR UPGRADE — Q1 integration)**

| Field | Type | Ghi chú |
|---|---|---|
| payment_id | Text (primary) | |
| payment_date | DateTime | |
| contract | SingleLink → `project_contracts` | **Bắt buộc** |
| payment_type | SingleSelect | Thanh toán HĐ / Phát sinh / Tạm ứng / Hoàn ứng |
| installment_no | Number | Đợt thứ mấy |
| payee | SingleLink → `vendors_master` hoặc `customers_master` | |
| amount_contract_vnd | Currency | Theo HĐ |
| amount_actual_vnd | Currency | Thực chi |
| amount_variance_vnd | Formula | Chênh lệch auto |
| payment_method | SingleSelect | Chuyển khoản / Tiền mặt / Séc |
| receipt_doc | Attachment | Bill/phiếu chi |
| approved_by | SingleLink → `staff_master` | |
| notes | Text | |

### 2.15 `project_change_orders` **(NEW)**

| Field | Type |
|---|---|
| co_id | Text (primary) |
| requested_date | DateTime |
| requested_by | SingleSelect (Khách / PM / NTP) |
| description | Text |
| affected_categories | MultiSelect → `categories_master` |
| cost_impact_vnd | Currency |
| schedule_impact_days | Number |
| approval_status | SingleSelect (Chờ / Duyệt / Reject) |
| approved_by | SingleLink → `staff_master` |
| approved_date | DateTime |
| supporting_docs | Attachment |
| related_contract | SingleLink → `project_contracts` |

### 2.16 `project_documents`

| Field | Type |
|---|---|
| doc_id | Text (primary) |
| upload_date | CreatedTime |
| uploaded_by | SingleLink → `staff_master` |
| category | SingleSelect (Pháp lý / Thiết kế / Thi công / HĐ / Phát sinh / Khác) |
| title | Text |
| file | Attachment |
| related_category | SingleLink → `categories_master` |
| related_contract | SingleLink → `project_contracts` |
| notes | Text |

### 2.17 `project_inspections` **(NEW)**

| Field | Type |
|---|---|
| inspection_id | Text (primary) |
| inspection_date | DateTime |
| category | SingleLink → `categories_master` |
| milestone | SingleLink → `project_milestones` |
| inspector_internal | SingleLink → `staff_master` |
| inspector_customer | Checkbox |
| result | SingleSelect (Pass / Conditional / Fail) |
| issues_found | Text |
| photos | Attachment |
| signed_doc | Attachment |
| follow_up_needed | Checkbox |
| follow_up_deadline | DateTime |

### 2.18 `project_settings`

| Field | Type |
|---|---|
| setting_key | Text (primary) |
| setting_value | Text |
| notes | Text |

---

## LAYER 3 — WAREHOUSE SCHEMA

### Master entities (8)
`customers, staff, vendors, categories, work_library, sample_products, contract_types, materials`

### Project entities (19)
```
projects (1 row/project)
project_assignments         (staff × project × role)
project_stakeholders        (vendor × project)
project_milestones
project_tasks
project_daily_logs
project_task_reports
project_issues
project_weekly_reports
vendor_scorecards           (monthly)
contracts
quotes
budget_items
payments
payment_schedules           (lịch dự kiến, khác actual)
change_orders
documents
inspections
settings
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
- `payments.contract_id` NOT NULL (mọi payment phải link HĐ)
- `vendor_scorecards` UNIQUE (vendor_id, month_year, category_id)
- `project_assignments` UNIQUE (project_id, staff_id, role, valid_period)

---

## 🚀 MIGRATION PLAN

Vì **Q5: file hiện tại = template** → migration cực đơn giản.

### Bước 1 — Setup Master Base (tuần 1)
- Tạo Base Lark mới "Master Data TLXN"
- 8 table master
- Seed data:
  - `categories`: import 44 hạng mục
  - `work_library`: import 36 checklist items
  - `sample_products`: import từ Table 8
  - `contract_types`: nhập 5-6 loại HĐ
  - `staff`: danh sách nhân viên hiện có
  - `customers`: seed từ dự án đang chạy
  - `vendors`: dedupe từ "Tên đơn vị thi công" hiện có

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
- **[B]** Detailed spec 3-4 table critical (project_info, payments, contracts, vendor_scorecards)
- **[C]** Kế hoạch setup Master Base (bấm gì trên Lark UI)
- **[D]** Chuyển sang Phase 2 (Sales) — đồng bộ data model cho cả QLDA + Sales
