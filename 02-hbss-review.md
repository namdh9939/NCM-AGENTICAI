# HBSS Schema Review — File mẫu TLXN

**App token**: `Ju2dbkBEzaPX6KskHQSlnI3Mgvb`
**Workspace**: trolyxaynha029.sg.larksuite.com
**Review date**: 2026-04-18
**Scope**: 8 table, 149 field, 176 + 232 + 4 + 7 + ... records (dự án mẫu)
**File là**: Template dùng clone cho mọi dự án mới (Q5)

---

## 📋 Tổng quan 8 table

| # | Table | Table ID | Fields | Records | Đánh giá |
|---|---|---|---|---|---|
| 1 | 📆 Kế Hoạch & Thông Tin | `tbl5NinGKRDjx0Xr` | **33** | 176 | 🔴 MEGA — tách |
| 2 | ✅ Việc Thiết Kế & Thi Công | `tblxTcF0oni93SVt` | 21 | 232 | 🟡 OK, cleanup |
| 3 | ⚡ Ticket & Đánh Giá Tuần | `tblsCNqDYZLIeVvz` | **32** | 4 | 🔴 MEGA — tách |
| 4 | 💵 Báo Giá & Thanh Toán | `tbltD31qVCD2z50h` | 18 | 7 | 🔴 Sparse, cần redesign |
| 5 | 🗂 Hồ Sơ & Pháp Lý | `tblM8Yd9hjLyk3ha` | 7 | ? | 🟢 OK |
| 6 | 📸 Nhật Ký Dự Án | `tbl7Az3O2Wk9danj` | 17 | ? | 🟡 OK |
| 7 | 🎯 Trợ Lý Báo Cáo | `tblb0C10WJPpuyeK` | 17 | ? | 🟡 OK |
| 8 | 🗄 Thiết Lập | `tblnGieIhehZyL2r` | **24** | 48 | 🔴 MEGA — tách |

---

## 🚨 10 ANTI-PATTERNS

### 🔴 Nhóm A — Kiến trúc

**#1. Polymorphic "Chức năng bảng" pattern** (nặng nhất)

4/8 table dùng SingleSelect `Chức năng bảng` để nhét 2–3 concept vào 1 table:
- Table 1: `Tiến độ` / `Đối tác` / `Thông tin`
- Table 3: `Ticket` / `Báo cáo tuần` / `Scorecard`
- Table 4: `Chi dự án` / `Báo giá` / `Dự kiến chi`
- Table 8: `Thiết lập` / `Kho việc` / `Sản phẩm mẫu`

**Hệ quả**: AI agent phải filter trước mỗi query. Schema mỗi row khác → nhiều field rỗng. Scorecard record có 7 field rating weight trong khi ticket record cùng bảng không dùng → data sparsity.

**#2. Không có entity Dự Án (Project) riêng**

"1 dự án = 1 file" nhưng metadata không được store:
- ❌ Tên CĐT: không có field chuẩn (rải rác trong Thông tin text fields)
- ❌ Địa chỉ công trình: không có
- ❌ Diện tích, số tầng, loại nhà: không có
- ❌ Ngân sách tổng: không có
- ❌ Ngày ký HĐ, khởi công, bàn giao dự kiến: không có structured

**#3. "Page ID" làm project identifier — fragile**

Mỗi record có 4 Lookup: `Tra cứu Page ID`, `Tra cứu PID`, `Page ID (CĐT)`, `PID (CĐT)`. Các ID này là **Lark chat group ID** (`zlw785259066282255149`), không phải ID dự án chính thức.

Rủi ro:
- 1 khách có 2 dự án → cùng chat → không phân biệt được
- Đổi tool chat → mất hết link
- Không query được "Dự án X thuộc khách Y"

### 🟡 Nhóm B — Duplication

**#4. Field `Hạng mục` (44 options) duplicate ở 3 table**

Table 1 (`fld55Q5LJ7`), Table 2 (`fldrIz3FYR`), Table 8 (`fldXFX2v7m`) đều có cùng 44 option. Thêm hạng mục mới = update 3 nơi. Đã thấy drift (Table 2 có `#Hạng mục` subset 4 option).

**#5. Field `Giai đoạn` duplicate**

- Table 1 (Lookup)
- Table 2 (Lookup)
- Table 8 `Giai đoạn thi công` (master, 3 option: *Trước xây / Phần thô / Phần hoàn thiện*)

**#6. Redundant "Tra cứu" fields**

Mọi bảng đều có 4 field tra cứu Page ID/PID về Setup table. 4 × 8 = **32 lookup field** chỉ để reference chat ID.

### 🟢 Nhóm C — Thiếu structure

**#7. Không có entity Khách Hàng (Customer)**
Thông tin khách rải rác trong text field của Table 1 (`Tên đơn vị thi công`, `Chức vụ`, `SĐT`, `Địa chỉ công ty`). Không normalized.

**#8. Không có entity Nhân Viên (Staff)**
`Phụ trách chính` chỉ là SingleSelect Vai trò (PM/AA/CA/Account) — **không biết tên cụ thể**. Không thể đo performance cá nhân.

**#9. Không có entity Nhà Thầu (Vendor)**
"Tên đơn vị thi công" là text tự do → "ABC" và "ABC Co." không match. **Scorecard 7 tiêu chí có sẵn ở Table 3 không thể aggregate cross-project** — đáng tiếc nhất.

**#10. Data payment sparse & sai mục đích**

Table 4 có 18 field nhưng 7 record, `Nội dung` lưu comma-separated Vai trò names chứ không phải transaction thật. Q1 confirm: payment tracking đang ở bộ phận tài chính, chưa dùng HBSS.

---

## 💎 ĐIỂM TÍCH CỰC (giữ nguyên)

1. **44 hạng mục chi tiết** — từ `Tiếp nhận` → `BTCT tầng 1-6` → `Thạch cao/Sơn/PCCC` → `Nghiệm thu`. Đây là gold data, không công ty VN nào có.
2. **36 checklist công việc cụ thể** ở Table 2 — *"Biên bản bàn giao ranh mốc", "Giấy kiểm định đồng hồ ép cọc", "Nhồi cát/đá mi vào khe cừ C"…* — training data cực quý cho AI.
3. **Scorecard 7 tiêu chí có weight** (20/15/15/15/15/10/10 %) — framework tốt, chỉ thiếu entity Đơn vị/Đối tác để aggregate.
4. **Daily log NTP** (Table 6) với số nhân công, tình trạng (cả ngày/sáng/chiều), nguy cơ chậm — operational tracking ngon.
5. **Hierarchy task** ở Table 2 (1.1, 1.2, 3.1…) — structure tốt cho work breakdown.

---

## 🔍 FINDINGS TỪ SAMPLE DATA

### Từ Table 1
Record 1: `Nội dung="Tiếp nhận thông tin"`, `Hạng mục="Tiếp nhận - Xin phép - Khái toán dự án"`, `% thiết kế=36.6%`, `Bắt đầu=13/03/2026`, `Kết thúc=16/03/2026`
→ Dự án mẫu đang ở giai đoạn đầu, chưa khởi công.

### Từ Table 2
- 232 Checklist công việc chi tiết
- Hierarchy đánh số: "1.1 Nhận bàn giao thông tin từ sale", "1.2 Tạo nhóm Zalo, chào khách hàng", "3.1 Mặt bằng công năng"
- Owner: PM / Account / CA / AA — vai trò rõ
- Task "Tạo nhóm Zalo, chào khách hàng" được treat như công việc → communication cũng là structured task

### Từ Table 3
- 4 record: có cả Ticket + Báo cáo tuần + Scorecard (confirm polymorphic)
- Phụ trách field có value "Nhà thầu PCCC" — text lookup, không phải Đơn vị/Đối tác entity

### Từ Table 4
- 7 record sparse, `Nội dung` lưu "Nhà thầu chính,Đội thi công điện nước" (Vai trò names comma-separated)
- Confirm: payment thực đang không ở đây

### Từ Table 8 (Thiết lập)
- Chứa Page ID/PID refs của project mẫu: `zlw785259066282255149` (CĐT chat), `zlw4836794913574064343` (nội bộ)
- Mix Thiết lập + Kho việc + Sản phẩm mẫu trong cùng bảng
- 48 record với image attachments (cho Kho việc/Sản phẩm mẫu)

---

## 📌 KẾT LUẬN REVIEW

**Cấu trúc hiện tại được thiết kế cho con người dễ dùng, không cho AI agent đọc.**

Thành công thôi không đủ để scale 1000 dự án/năm. Cần:
1. **Tách các MEGA table** thành table riêng theo responsibility
2. **Thêm entity thiếu**: Thông tin Dự án, Khách hàng, Nhân sự, Đơn vị/Đối tác, Hợp đồng liên quan, payment, change_order, inspection
3. **Di chuyển master data** ra 1 Base riêng (không per-project)
4. **Chuẩn hóa identifier**: Mã dự án/Mã khách hàng/Mã đối tác thay cho Lark chat ID
5. **Xây Warehouse layer** để cross-project query + training data cho AI

Data model v2 giải quyết toàn bộ → xem `03-data-model-v2.md`.
