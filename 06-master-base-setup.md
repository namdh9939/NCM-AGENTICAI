# Master Base Setup — Step-by-step trên Lark

**Date**: 2026-04-18
**Mục đích**: Hướng dẫn click-by-click build Layer 1 Master Base cho TLXN
**Đối tượng**: Nam + team no-code (không cần SQL)
**Scope**: 7 table (skip `material_catalog` v2), trong đó `customers_master` chỉ dựng schema (chưa seed — chờ D xong)

---

## 📋 Kế hoạch tổng quan

| Bước | Table | Seed data? | Thời gian ước tính |
|---|---|---|---|
| 0 | Tạo Base mới | - | 5' |
| 1 | `categories_master` | ✅ 44 hạng mục | 30' |
| 2 | `contract_types_master` | ✅ 5-6 loại | 15' |
| 3 | `staff_master` | ✅ toàn bộ NV | 20' |
| 4 | `work_library` | ✅ 36 items | 45' |
| 5 | `sample_products` | ✅ từ Table 8 cũ | 20' |
| 6 | `vendors_master` | ✅ dedupe từ HBSS cũ | 45' |
| 7 | `customers_master` | 🟡 **SCHEMA ONLY** | 15' |
| 8 | Verification | - | 15' |
| 9 | Permissions | - | 10' |

**Tổng**: ~3.5 tiếng nếu seed data đã chuẩn bị sẵn.

---

## 📌 Quy ước & lưu ý Lark trước khi bắt đầu

### Primary field
Mỗi table Lark có **1 primary field** (cột đầu tiên, không xóa được). Đây là field hiển thị khi được Link từ table khác. Ta sẽ dùng cột `xxx_code` làm primary (vd: `category_name`, `staff_code`, `vendor_code`).

### Auto-code pattern (cho mọi `*_code`)
Lark không có auto-increment native. Dùng combo:
1. Field `auto_num` kiểu **Autonumber** (ẩn)
2. Field `xxx_code` kiểu **Formula** với công thức:
   ```
   CONCATENATE("STAFF-", TEXT({auto_num}, "000"))
   ```
   → Output: `STAFF-001`, `STAFF-002`, …

Cho field có năm: `CONCATENATE("CUS-", TEXT(YEAR(CREATED_TIME()), "0"), "-", TEXT({auto_num}, "000"))` → `CUS-2026-001`.

### SingleLink vs MultiLink
- `SingleLink` = liên kết 1 record ở table khác (vd: vendor → category chính)
- `MultiSelect Link` (Lark gọi là "Link multi") = nhiều records (vd: vendor chuyên nhiều hạng mục)

Khi tạo field type chọn **"Link to other tables"** → chọn table đích → toggle "Allow multiple records" nếu MultiLink.

### UNIQUE constraint
Lark **không có UNIQUE native**. Workaround:
- Field `auto_num` đảm bảo ID unique
- Với business unique (vd: `staff_code` không trùng tên), dùng **Filtered view** check duplicate + training team

### Formula sẽ cần
| Field | Công thức |
|---|---|
| `xxx_code` (no year) | `CONCATENATE("PREFIX-", TEXT({auto_num}, "000"))` |
| `xxx_code` (có năm) | `CONCATENATE("PREFIX-", TEXT(YEAR(CREATED_TIME()), "0"), "-", TEXT({auto_num}, "000"))` |
| `amount_variance_vnd` | `{amount_actual_vnd} - {amount_contract_vnd}` |
| `total_score` (scorecard) | `{score_cooperation}*0.2 + {score_report_quality}*0.15 + ...` |

---

## Bước 0 — Tạo Base mới

1. Vào https://trolyxaynha029.sg.larksuite.com
2. Menu trái → **Bitable** (hoặc **Base**)
3. Click **"+ Create"** → **"Blank base"**
4. Đặt tên: **`TLXN Master Data v2`**
5. Folder: **Shared Space** của TLXN (không để private — team cần access)
6. Click **Create**
7. Đổi tên **Table 1** mặc định thành **`categories_master`** (click chuột phải tab → Rename)

→ Base mới có 1 table empty. Copy **App Token** từ URL: `base/XXXXXXXXX` — lưu vào file `04-mcp-setup.md` dưới mục "Master Base app token".

---

## Bước 1 — `categories_master` (44 hạng mục)

### 1.1 Schema

Xóa toàn bộ default field. Tạo lần lượt (click dấu `+` cuối hàng field):

| # | Field name | Type | Config |
|---|---|---|---|
| 1 | `category_name` | **Text** | Primary (mặc định cột 1) |
| 2 | `phase` | **SingleSelect** | Options: `Trước xây`, `Phần thô`, `Phần hoàn thiện`, `Bàn giao` |
| 3 | `display_order` | **Number** | Integer, không decimal |
| 4 | `typical_duration_days` | **Number** | Integer, cho phép empty |
| 5 | `active` | **Checkbox** | Default: checked |
| 6 | `notes` | **Text** | Long text |

### 1.2 Seed 44 hạng mục

Mở HBSS cũ (`Ju2dbkBEzaPX6KskHQSlnI3Mgvb`) → Table 8 **Thiết Lập** → filter `Chức năng bảng = Thiết lập` → lấy danh sách 44 hạng mục.

**Cách nhanh**: Copy toàn cột category ở HBSS cũ → paste trực tiếp vào `category_name` (Lark hỗ trợ paste multi-row).

**Sau khi paste xong**, điền:
- `phase` cho từng row (Trước xây / Phần thô / …)
- `display_order` (1, 2, 3… theo thứ tự logic vòng đời)
- `active` = true hết

💡 Tạo **view `By phase`** → group by phase → dễ visualize.

### 1.3 Checkpoint
- [ ] 44 row
- [ ] Không row nào trùng `category_name`
- [ ] Mỗi row có `phase` và `display_order`

---

## Bước 2 — `contract_types_master` (5-6 loại)

### 2.1 Schema

| # | Field | Type | Config |
|---|---|---|---|
| 1 | `type_code` | **Text** | Primary |
| 2 | `type_name` | **Text** | |
| 3 | `typical_template_doc` | **Attachment** | |
| 4 | `requires_signature_from` | **MultiSelect** | `Khách`, `TLXN`, `NTP`, `NCC` |
| 5 | `typical_payment_terms` | **Text** | Long text |
| 6 | `active` | **Checkbox** | |

### 2.2 Seed (nhập tay)

| type_code | type_name | requires_signature_from |
|---|---|---|
| `CT-TLXN` | Hợp đồng dịch vụ Trợ Lý Xây Nhà | Khách, TLXN |
| `CT-DESIGN` | Hợp đồng thiết kế | Khách, TLXN |
| `CT-MAIN-CONSTRUCTION` | Hợp đồng thi công chính (nhà thầu chính) | Khách, NTP, TLXN |
| `CT-SUB` | Hợp đồng nhà thầu phụ | TLXN, NTP |
| `CT-SUPPLIER` | Hợp đồng cung ứng vật tư/thiết bị | TLXN, NCC |
| `CT-CONSULT` | Hợp đồng tư vấn (nếu có) | Khách, TLXN |

Upload template HĐ mẫu (nếu có) vào `typical_template_doc`.

---

## Bước 3 — `staff_master`

### 3.1 Schema

| # | Field | Type | Config |
|---|---|---|---|
| 1 | `staff_code` | **Formula** | `CONCATENATE("STAFF-", TEXT({auto_num}, "000"))` — Primary |
| 2 | `auto_num` | **Autonumber** | Ẩn cột này đi (click chuột phải field → Hide) |
| 3 | `full_name` | **Text** | |
| 4 | `role_primary` | **SingleSelect** | `PM`, `AA`, `CA`, `Account`, `PO`, `PD`, `Sale`, `TK`, `QA`, `BOD` |
| 5 | `roles_secondary` | **MultiSelect** | Cùng options với role_primary |
| 6 | `email_work` | **Email** | |
| 7 | `phone` | **Phone** | |
| 8 | `lark_user_id` | **Text** | Điền sau, cần query từ Lark contact |
| 9 | `active` | **Checkbox** | Default: checked |
| 10 | `joined_date` | **Date** | |
| 11 | `left_date` | **Date** | Empty nếu đang làm |
| 12 | `notes` | **Text** | |

### 3.2 Seed

Nhập danh sách NV hiện tại. Mỗi row 1 NV. `staff_code` sẽ auto-generate khi save row.

⚠️ **Lưu ý PII**: `phone`, `email_work` là thông tin nội bộ → sau này set permission ở Bước 9 để chỉ BOD + HR thấy hết, PM/CA chỉ thấy tên + role.

### 3.3 Query `lark_user_id` (optional, làm sau)

Dùng MCP tool `contact_v3_user_batchGetId` với email → trả về `user_id`. Chạy batch sau khi seed xong.

---

## Bước 4 — `work_library` (36 items)

### 4.1 Schema

| # | Field | Type | Config |
|---|---|---|---|
| 1 | `work_item_code` | **Formula** | `CONCATENATE("W-", TEXT({auto_num}, "000"))` — Primary |
| 2 | `auto_num` | **Autonumber** | Hide |
| 3 | `work_name` | **Text** | |
| 4 | `category` | **Link** → `categories_master` | Single |
| 5 | `phase` | **Lookup** | Auto từ `category.phase` |
| 6 | `priority` | **SingleSelect** | `Mốc quan trọng`, `Thường` |
| 7 | `default_owner_role` | **SingleSelect** | Giống role_primary ở staff_master |
| 8 | `standard_doc` | **Attachment** | Tiêu chuẩn thi công |
| 9 | `sample_image` | **Attachment** | |
| 10 | `preparation_note` | **Text** | Long text |
| 11 | `during_note` | **Text** | Long text |
| 12 | `after_note` | **Text** | Long text |
| 13 | `active` | **Checkbox** | |

**Cách tạo Lookup `phase`** (Field #5):
1. Thêm field type **"Lookup"**
2. Target link: chọn field `category`
3. Target field to look up: chọn `phase` từ categories_master
4. Save

### 4.2 Seed 36 items

Mở HBSS cũ → Table 2 **Việc Thiết Kế & Thi Công** → lấy 36 work name ("Biên bản bàn giao ranh mốc", "Tạo nhóm Zalo, chào khách hàng", "Mặt bằng công năng"…).

Copy `work_name` → paste vào `work_name` field. Sau đó cho mỗi row:
- Click field `category` → pick từ dropdown (link to `categories_master`)
- `phase` sẽ tự điền
- Điền `priority`, `default_owner_role`

Attachment (standard_doc, sample_image) + các note text → điền dần khi có thời gian. Không block.

---

## Bước 5 — `sample_products`

### 5.1 Schema

| # | Field | Type |
|---|---|---|
| 1 | `sample_code` | **Formula** `CONCATENATE("SMP-", TEXT({auto_num}, "000"))` |
| 2 | `auto_num` | **Autonumber** (hide) |
| 3 | `category` | **Link** → `categories_master` |
| 4 | `description` | **Text** |
| 5 | `images` | **Attachment** |
| 6 | `supplier_hint` | **Text** (tùy chọn) |
| 7 | `active` | **Checkbox** |

### 5.2 Seed

Mở HBSS cũ → Table 8 **Thiết Lập** → filter `Chức năng bảng = Sản phẩm mẫu` → copy rows.

Paste `description`, link `category`. Upload ảnh từng row (download từ HBSS cũ rồi upload).

💡 Nếu có ~10-20 ảnh thì làm tay OK. Nếu >50 thì viết script MCP batch sau.

---

## Bước 6 — `vendors_master`

### 6.1 Schema

| # | Field | Type | Config |
|---|---|---|---|
| 1 | `vendor_code` | **Formula** `CONCATENATE("VEN-", TEXT(YEAR(CREATED_TIME()), "0"), "-", TEXT({auto_num}, "000"))` | Primary |
| 2 | `auto_num` | **Autonumber** | Hide |
| 3 | `company_name` | **Text** | |
| 4 | `vendor_type` | **MultiSelect** | `NTP-Thi công`, `NTP-Thiết kế`, `NCC-Vật tư`, `NCC-Thiết bị` |
| 5 | `category_specialization` | **Link (multi)** → `categories_master` | |
| 6 | `contact_person` | **Text** | |
| 7 | `phone_company` | **Phone** | |
| 8 | `phone_contact` | **Phone** | |
| 9 | `address` | **Text** | |
| 10 | `tax_code` | **Text** | MST |
| 11 | `bank_info` | **Text** | Long text |
| 12 | `status` | **SingleSelect** | `Active`, `Trial`, `Blacklisted` |
| 13 | `first_worked_date` | **Date** | |
| 14 | `notes` | **Text** | |

⚠️ Field `overall_score` và `projects_count` là **Rollup từ scorecards** — SẼ tạo ở Bước tương lai (sau khi template v2 có table `vendor_scorecards`). Tạm bỏ qua.

### 6.2 Dedupe & seed từ HBSS cũ

1. Mở HBSS cũ → tìm field text **"Tên đơn vị thi công"** trong các table
2. Export list unique → Excel/Sheet
3. **Dedupe manual**: "ABC", "ABC Co.", "Cty ABC" → gộp thành 1 vendor
4. Gán `vendor_type`, `category_specialization` cho từng cái
5. Bulk import vào `vendors_master` (paste multi-row)

💡 Dedupe là việc lần duy nhất tốn thời gian. Sau khi làm xong, mọi dự án mới đều Link về entity chuẩn.

---

## Bước 7 — `customers_master` (SCHEMA ONLY, no seed)

⚠️ **Quan trọng**: CHỈ build schema. **KHÔNG seed data khách hàng cũ** đến khi Phase 2 Sales (D) xong. Phase 2 sẽ thêm field Sales → tránh migrate 2 lần.

### 7.1 Schema

| # | Field | Type | Config |
|---|---|---|---|
| 1 | `customer_code` | **Formula** `CONCATENATE("CUS-", TEXT(YEAR(CREATED_TIME()), "0"), "-", TEXT({auto_num}, "000"))` | Primary |
| 2 | `auto_num` | **Autonumber** | Hide |
| 3 | `full_name` | **Text** | |
| 4 | `phone_primary` | **Phone** | |
| 5 | `phone_secondary` | **Phone** | |
| 6 | `email` | **Email** | |
| 7 | `id_card_number` | **Text** | ⚠️ PII |
| 8 | `date_of_birth` | **Date** | |
| 9 | `address_permanent` | **Text** | |
| 10 | `occupation` | **Text** | |
| 11 | `referral_source` | **SingleSelect** | `Facebook`, `Google`, `Referral`, `Zalo`, `YouTube`, `Event`, `Khác` |
| 12 | `lark_chat_page_id` | **Text** | |
| 13 | `created_at` | **Created Time** | Auto |
| 14 | `notes` | **Text** | |

### 7.2 Seed
**BỎ QUA ở bước này**. Tạo 1 row test dummy (`full_name = "TEST - DELETE ME"`) để verify formula → xóa ngay sau đó.

---

## Bước 8 — Verification checklist

- [ ] Base có 7 table đúng tên
- [ ] `categories_master` có 44 row, mỗi row có phase + display_order
- [ ] `contract_types_master` có 5-6 row
- [ ] `staff_master` có đủ NV hiện tại, auto-code chạy đúng
- [ ] `work_library` có ~36 row, Link `category` hoạt động, Lookup `phase` tự điền
- [ ] `sample_products` có data + ảnh
- [ ] `vendors_master` dedupe xong, Link `category_specialization` hoạt động
- [ ] `customers_master` schema OK, chưa có data thật (chỉ 0 hoặc 1 row test đã xóa)
- [ ] Thử link từ 1 work_library record → categories_master → mở được target

### Test cross-table reference
Ở `work_library`, click vào 1 Link `category` → phải jump sang `categories_master` đúng row. Ở ngược lại, mở 1 row `categories_master` → phải thấy **reverse link** `Linked records in work_library` (Lark tự tạo).

---

## Bước 9 — Permissions & sharing

### 9.1 Share Base

1. Góc phải Base → **Share**
2. **"Add members"** → add team QLDA + Sales + BOD
3. Set role:
   - **BOD / Nam**: Can edit (full admin)
   - **PM / Account / CA**: Can edit
   - **Sale / TK / PO**: Can edit (hoặc View-only tùy role — xem 9.2)
   - **NTP / NCC** (future): Không share Base master, chỉ share per-project file

### 9.2 Hide PII cho non-BOD roles (optional v1, bắt buộc Phase 1)

Lark Base hỗ trợ **field-level permission**:
1. Right-click field `id_card_number` ở `customers_master` → **"Field Permissions"**
2. Set: Only `BOD`, `Account lead`, `QLDA lead` có thể view
3. Lặp tương tự cho `bank_info` ở `vendors_master`, `phone`/`email_work` ở `staff_master`

*Có thể làm sau khi seed xong, trước khi Phase 1 production.*

### 9.3 Backup policy
- Lark Base tự backup (có revision history)
- Export CSV/Excel định kỳ: Menu → Export → All tables
- **Khuyến nghị**: Set cron weekly export → lưu Drive riêng (sẽ setup ở Phase 0.16 DR)

---

## 📎 Appendix — Các lưu ý Lark hay gặp

### A.1 Paste multi-row từ Excel/Sheet
- Select range ở Excel → Ctrl+C
- Ở Lark table, click cell trống đầu cột → Ctrl+V
- Lark auto-detect column → paste. Với SingleSelect, option chưa có thì Lark sẽ báo → confirm "Create new option"

### A.2 Formula không auto-update Autonumber ngay
Khi paste nhiều row cùng lúc, `auto_num` cần 1-2 giây để populate. Formula `xxx_code` sẽ empty tạm thời → refresh (F5) là thấy.

### A.3 Không thể đổi type field sau khi có data
Nếu lỡ tạo `phone` kiểu Text thay vì Phone → phải tạo field mới + migrate data. **Kiểm tra type trước khi paste data.**

### A.4 Link field behavior
- Đổi `category_name` của 1 row ở `categories_master` → **mọi Link ở work_library/vendors tự update** theo (tốt)
- Xóa 1 row ở `categories_master` mà có Link đang reference → Lark báo cảnh báo, **không auto-cascade** (cần fix thủ công)

### A.5 Autonumber bắt đầu từ đâu?
Mặc định từ 1. Muốn bắt đầu từ số khác (vd: `STAFF-100` để phân biệt với test): setting field Autonumber → "Starting value".

### A.6 Field Lookup chỉ đi 1 hop
`work_library.phase` lookup qua `category → phase` (1 hop): OK.
Muốn 2 hop (vd: lookup qua 2 link) → phải tạo thêm intermediate field.

---

## ✅ Khi bước 9 xong

- Master Base đã sẵn sàng làm **source of truth cho 6 entity**
- Customer entity chờ Phase 2 Sales bổ sung field → mới seed
- Next: **Phase 2 Sales design (D)** → review lại customers_master + staff_master
- Sau D: build Template v2 (Layer 2) → clone cho mọi dự án mới
- Song song: setup Warehouse Layer 3 (Supabase) + webhook sync

---

## 🔗 Handoff cho session sau

Khi Nam hoàn thành Base, lưu lại:
- **App Token** của Master Base (ghi vào `04-mcp-setup.md`)
- Table IDs (8 tables) → Lark hiển thị trong URL khi mở từng table
- Screenshot 1 row của mỗi table làm reference

Sau đó mở session mới với Claude: *"Master Base đã xong, đi tiếp D (Phase 2 Sales)"*.
