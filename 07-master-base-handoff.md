# Master Base — Đợt 1 Handoff

**Date**: 2026-04-18
**Status**: Đợt 1 xong (execute qua MCP Lark). Đợt 2 = Nam làm UI + cung cấp data.

---

## 🎯 Base đã tạo

**URL**: https://trolyxaynha029.sg.larksuite.com/base/OmLlbadqGaix8tsku1vlqJbhg9K

**App token**: `OmLlbadqGaix8tsku1vlqJbhg9K`

⚠️ **Base hiện ở ROOT của tenant**, không phải folder Nam chọn. Lý do: app Lark (`cli_a959369f73f8deea`) thiếu permission ghi vào folder `GpmYf9kcWlcanUdqeh5lnzYjgNd`. **Nam cần move thủ công qua UI** (xem Đợt 2 bên dưới).

---

## 📊 7 tables đã tạo + data seeded

| Table | Table ID | Records | Status |
|---|---|---|---|
| `categories_master` | `tblJGRWqvc5raSfp` | **44** ✅ | Seeded đầy đủ |
| `contract_types_master` | `tblRYk1I7IeDZ4gK` | **6** ✅ | Seeded đầy đủ |
| `work_library` | `tblE6Sqat6N2i1od` | **24** ✅ | Seeded 24 mốc quan trọng (subset từ HBSS cũ 232 tasks) |
| `staff_master` | `tbleat45Byd4imLa` | 0 | Schema xong, chờ Nam cung cấp data |
| `vendors_master` | `tbljpeiAFBfnERQ1` | 0 | Schema xong, chờ dedupe |
| `sample_products` | `tbl3NCnE5kJi6rtG` | 0 | Schema xong, chờ upload ảnh |
| `customers_master` | `tblpH8UP3Kp7Ksnx` | 0 | Schema-only (defer seed đến sau Phase 2 Sales) |

**Tổng**: 74 records seeded qua API.

---

## ⚠️ Khác biệt so với thiết kế trong `06-master-base-setup.md`

Một số field Lark API không tạo được → phải add qua UI:

### 1. Primary field là Text, không phải Formula code
Thiết kế gốc: primary = `category_code` / `staff_code` / `vendor_code` (Formula auto-gen).

Thực tế: primary = human-readable name (`category_name`, `full_name`, `company_name`...).

**Hệ quả**: Vẫn có field `auto_num` (AutoNumber), Nam cần **thêm Formula field** qua UI để gen code — xem Đợt 2 bước 3.

### 2. Không có field Lookup
`work_library.phase` (lookup từ category) — chưa tạo. Nam add qua UI.

### 3. Không có field Rollup
`vendors_master.overall_score` — sẽ tạo SAU khi Layer 2 Template có `vendor_scorecards`.

### 4. customers_master thiếu field `created_at` (CreatedTime)
API báo lỗi format khi tạo. Nam add qua UI (CreatedTime → auto).

### 5. Base còn 1 **default empty table** (Table 1, id `tblGR85VCYleyNMV`)
Tạo tự động khi create Base, không xóa được qua API. Nam xóa qua UI.

---

## 📋 Đợt 2 — Việc Nam cần làm (UI + cung cấp data)

### Bước 1 — Move Base vào folder shared (5')

1. Mở URL Base bên trên
2. Góc trên phải → biểu tượng ⋯ (More) → **Move to**
3. Chọn folder https://trolyxaynha029.sg.larksuite.com/drive/folder/GpmYf9kcWlcanUdqeh5lnzYjgNd
4. Confirm

### Bước 2 — Xóa default empty table (1')

1. Tab **"Table 1"** (hoặc tên tương tự, rỗng) → chuột phải → **Delete table**
2. Confirm

### Bước 3 — Thêm Formula field cho auto-code (15')

Áp dụng cho 5 table: `staff_master`, `vendors_master`, `customers_master`, `work_library`, `sample_products`.

**Ví dụ với `staff_master`**:
1. Click **+** cuối hàng field → **Formula**
2. Field name: `staff_code`
3. Formula: `CONCATENATE("STAFF-", TEXT({auto_num}, "000"))`
4. Save → kéo field này lên làm column đầu tiên (drag header)
5. Click chuột phải header `staff_code` → **Set as index field** (nếu Lark cho phép đổi primary)

⚠️ Lark có thể KHÔNG cho đổi primary sau khi tạo. Nếu vậy → giữ primary = `full_name`, `staff_code` chỉ là field phụ — chấp nhận được, vì link từ table khác sẽ show tên người (UX tốt hơn mã).

**Công thức cho từng table**:

| Table | Formula |
|---|---|
| `staff_master` | `CONCATENATE("STAFF-", TEXT({auto_num}, "000"))` |
| `vendors_master` | `CONCATENATE("VEN-", TEXT(YEAR(CREATED_TIME()), "0"), "-", TEXT({auto_num}, "000"))` |
| `customers_master` | `CONCATENATE("CUS-", TEXT(YEAR(CREATED_TIME()), "0"), "-", TEXT({auto_num}, "000"))` |
| `work_library` | `CONCATENATE("W-", TEXT({auto_num}, "000"))` |
| `sample_products` | `CONCATENATE("SMP-", TEXT({auto_num}, "000"))` |

### Bước 4 — Thêm Lookup field (5')

**work_library → phase** (lookup qua category):
1. `work_library` → **+** cuối field → **Lookup**
2. Field name: `phase`
3. Source: `category` (SingleLink đã có)
4. Target field: `phase` (từ categories_master)
5. Save

### Bước 5 — Thêm CreatedTime cho customers_master (2')

1. `customers_master` → **+** cuối field → **Created time**
2. Field name: `created_at`
3. Save

### Bước 6 — Cung cấp data cho tôi (Nam làm) → tôi seed đợt 2

Sau khi Nam làm xong bước 1-5, paste cho tôi một trong các format dưới → tôi seed qua MCP:

#### 6.1 Staff list
```
FullName | role_primary | email | phone
Nam Do | BOD | nam.dh9939@gmail.com | 09xx
Minh Nguyen | PM | minh@tlxn.vn | 09xx
...
```

#### 6.2 Vendors dedupe
Nam extract từ HBSS cũ field "Tên đơn vị thi công" → dedupe → gửi:
```
company_name | vendor_type | category_specialization | contact_person | phone
Cty ABC | NTP-Thi công | BTCT móng, BTCT tầng trệt | Anh A | 09xx
...
```

#### 6.3 Sample products
Nam upload ảnh qua UI (vì API khó attach). Hoặc paste text description để tôi seed thô trước.

### Bước 7 — Share Base với team (5')

1. Góc phải → **Share**
2. Add members: team QLDA, Sales, BOD
3. Set role: Edit cho QLDA/BOD/Account, View cho Sales/TK

---

## 🔧 Nếu Nam gặp lỗi

### Lỗi: Permission denied khi move Base
→ Folder đích phải là shared space. Nam phải là **Owner** hoặc **Editor** của folder.

### Lỗi: Formula báo sai syntax
→ Field reference dùng `{field_name}`. Trong Lark UI sẽ có autocomplete khi gõ `{`.

### Lỗi: Lark không cho đổi primary field
→ Bỏ qua. Primary = Text name cũng OK, vì Link từ Layer 2 Template sẽ hiện tên người/công ty (UX tốt hơn mã code thô).

---

## ✅ Khi Nam xong Đợt 2, ping tôi

Paste vào session mới (hoặc tiếp session này):

> "Master Base Đợt 2 xong. Staff list + vendor list đây: [paste data]. Seed tiếp cho tôi."

Sau đó:
- Tôi seed staff, vendors, sample_products
- Verify toàn bộ 7 table
- Chuyển sang **D — Phase 2 Sales design** → bổ sung field `customers_master` trước khi seed customer thật

---

## 📝 Ghi chú kỹ thuật (for future reference)

### MCP Lark tools đã dùng
- `bitable_v1_app_create` — tạo Base
- `bitable_v1_appTable_create` — tạo 7 table với schema
- `bitable_v1_appTableField_list` — extract 44 categories options
- `bitable_v1_appTableRecord_search` — extract 232 work tasks từ HBSS cũ
- `bitable_v1_appTableRecord_create` — seed 74 records

### Field type mapping Lark API
| UI | type | ui_type |
|---|---|---|
| Text | 1 | Text |
| Number | 2 | Number |
| SingleSelect | 3 | SingleSelect |
| MultiSelect | 4 | MultiSelect |
| DateTime | 5 | DateTime |
| Checkbox | 7 | Checkbox |
| Phone | 13 | Phone |
| Attachment | 17 | Attachment |
| SingleLink | 18 | SingleLink (+ `multiple: true` cho MultiLink) |
| CreatedTime | 1001 | CreatedTime |
| AutoNumber | 1005 | AutoNumber |

### Records IDs của 44 categories
Lưu lại để sau này seed tiếp (vd: seed thêm work_library → cần link category record_id):
- Tiếp nhận - Xin phép - Khái toán dự án: `recvh8dU1uc7W0`
- Lựa chọn đơn vị thiết kế: `recvh8dUdufD3Z`
- Concept kiến trúc: `recvh8dUqMVmcZ`
- Thiết kế nội thất: `recvh8dUCQmJMU`
- Triển khai bản vẽ: `recvh8dUPvr3m6`
- Tháo dỡ nhà: `recvh8dV0Cvbho`
- Chọn nhà thầu chính: `recvh8dVd9dXbG`
- Chọn nhà thầu ép cọc: `recvh8dVpoxCAv`
- Bàn giao hồ sơ & Bản vẽ: `recvh8dVB2ih7P`
- Khởi công: `recvh8dVO3oKhD`
- Ép cọc: `recvh8dXw6Wta1`
- Ép cừ C: `recvh8dXV4QdqP`
- BTCT móng: `recvh8dYpCxLpU`
- BTCT hầm: `recvh8dYDaeCso`
- MEP âm sàn: `recvh8dZ296rjC`
- PCCC âm sàn: `recvh8dZrVrGlx`
- BTCT tầng trệt: `recvh8dZOW4crq`
- BTCT tầng lửng: `recvh8e0feqaQL`
- BTCT tầng 1: `recvh8e0tACEVN`
- BTCT tầng 2: `recvh8e0HOAwec`
- BTCT tầng 3: `recvh8e3oYhSKM`
- BTCT tầng 4: `recvh8e3D1G6D9`
- BTCT tầng 5: `recvh8e3RgPRvH`
- BTCT tầng 6: `recvh8e43hMg0C`
- BTCT tầng Sân thượng: `recvh8e4zvQJxQ`
- BTCT tầng Mái: `recvh8e4Pfckgk`
- Sân: `recvh8e57zLfsv`
- Xây: `recvh8e5kwCyLG`
- MEP thô: `recvh8e6MtQnjJ`
- Tô: `recvh8e7bpkhNH`
- PCCC: `recvh8e8MFJH1I`
- Chống thấm: `recvh8e98C5orZ`
- Ốp & lát: `recvh8e9xbcDfM`
- Cán nền: `recvh8e9UT4uDx`
- Thạch cao: `recvh8eakele81`
- Sơn: `recvh8eaBhzsIM`
- Đá: `recvh8eb0H8QMr`
- Cửa cuốn: `recvh8ebikNxju`
- MEP hoàn thiện: `recvh8eciz0c6u`
- Nhôm kính: `recvh8ecuxFPsH`
- Lắp đặt thang máy: `recvh8eevQyPCt`
- Cơ khí: `recvh8eeUTg7mV`
- Hoàn thiện nội thất: `recvh8efiEBS32`
- Nghiệm thu bàn giao: `recvh8efHnA1eM`
