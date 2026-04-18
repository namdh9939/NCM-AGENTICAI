# Đợt 2 — Hướng dẫn từng bước cho Nam

**Mục đích**: Hoàn thiện Master Base trên Lark để sẵn sàng dùng.
**Đối tượng**: Nam (không cần kỹ thuật).
**Thời gian tổng**: ~1 tiếng click Lark + tùy Nam tự chuẩn bị data staff/vendor.

**Base URL**: https://trolyxaynha029.sg.larksuite.com/base/OmLlbadqGaix8tsku1vlqJbhg9K

---

## 🎯 Checklist tổng quan

Nam tick dần khi làm xong:

- [ ] **Phần 1**: Dọn Base (10 phút) — move, xóa table rỗng, share
- [ ] **Phần 2**: Thêm cột auto-tạo mã cho 5 tables (15 phút)
- [ ] **Phần 3**: Thêm 2 cột phụ (5 phút)
- [ ] **Phần 4**: Gom data staff + vendor (Nam tự làm, tùy thời gian)
- [ ] **Phần 5**: Gửi data cho Claude seed (5 phút)

---

# PHẦN 1 — Dọn Base (10 phút)

## 1.1 Move Base vào folder chung

**Tại sao**: Hiện Base đang nằm ở "My Space" (riêng Nam). Cần chuyển vào folder chung để team thấy được.

**Các bước**:

1. Mở link Base: https://trolyxaynha029.sg.larksuite.com/base/OmLlbadqGaix8tsku1vlqJbhg9K
2. Nhìn góc trên bên phải → bấm icon **⋯** (3 chấm ngang) → chọn **"Move to"** (Di chuyển đến)
3. Trong cửa sổ chọn folder → navigate đến folder:
   `https://trolyxaynha029.sg.larksuite.com/drive/folder/GpmYf9kcWlcanUdqeh5lnzYjgNd`
4. Bấm **"Move"** (Di chuyển)

✅ Xong khi Base đã ở trong folder mong muốn.

---

## 1.2 Xóa "Table 1" rỗng

**Tại sao**: Khi tạo Base, Lark tự tạo 1 table mặc định tên "Table 1" trống — không dùng.

**Các bước**:

1. Ở cạnh trái danh sách tables, tìm tab **"Table 1"** (hoặc tên tương tự, không phải 7 tables chính)
2. Chuột phải vào tab đó → chọn **"Delete table"** (Xóa bảng)
3. Confirm xóa

✅ Xong khi chỉ còn 7 tables: `categories_master`, `contract_types_master`, `staff_master`, `work_library`, `sample_products`, `vendors_master`, `customers_master`.

---

## 1.3 Share Base với team

**Tại sao**: Để PM, Account, CA, Sales dùng được.

**Các bước**:

1. Góc trên phải Base → bấm nút **"Share"** (Chia sẻ)
2. Gõ email/tên từng người trong team
3. Phân quyền:
   - Nam, BOD, QLDA lead: **"Can edit"** (Toàn quyền)
   - PM, Account, CA: **"Can edit"** (Toàn quyền)
   - Sale, TK: **"Can view"** (Chỉ xem)
4. Bấm **"Send"** hoặc **"Share"**

✅ Xong khi mọi NV cần dùng đã được add.

---

# PHẦN 2 — Thêm cột tự-tạo-mã (15-20 phút)

**Mục đích**: Mỗi nhân viên/khách hàng/nhà thầu/sản phẩm sẽ có mã riêng tự sinh (vd: `STAFF-001`, `VEN-2026-015`). Hiện tại chỉ mới có cột đếm số (`auto_num`), cần thêm cột mã đẹp.

⚠️ **Làm đúng 5 lần, mỗi lần giống hệt nhau chỉ đổi công thức.**

## Template làm cho MỖI table:

1. Mở table (bấm tab tên table ở cột trái)
2. Kéo ngang thanh cột sang phải hết mức → bấm dấu **"+"** ở cuối cùng để thêm field mới
3. Trong cửa sổ chọn type field → chọn **"Formula"** (Công thức)
4. **Field name**: điền tên theo bảng bên dưới (vd: `staff_code`)
5. **Formula**: Copy-paste đúng công thức từ bảng bên dưới vào ô **Input formula**
6. Bấm **"Confirm"** hoặc **"Save"**

---

## Bảng công thức (copy đúng, KHÔNG sửa gì)

### 2.1 Table `staff_master`

| Field name | Công thức (copy nguyên) |
|---|---|
| `staff_code` | `CONCATENATE("STAFF-", TEXT({auto_num}, "000"))` |

**Kết quả mong đợi**: Cột mới hiện `STAFF-001`, `STAFF-002`... tự động (nhưng chưa có row nào nên cột trống — không sao).

---

### 2.2 Table `vendors_master`

| Field name | Công thức |
|---|---|
| `vendor_code` | `CONCATENATE("VEN-", TEXT(YEAR(CREATED_TIME()), "0"), "-", TEXT({auto_num}, "000"))` |

**Kết quả**: `VEN-2026-001`, `VEN-2026-002`...

---

### 2.3 Table `customers_master`

| Field name | Công thức |
|---|---|
| `customer_code` | `CONCATENATE("CUS-", TEXT(YEAR(CREATED_TIME()), "0"), "-", TEXT({auto_num}, "000"))` |

**Kết quả**: `CUS-2026-001`...

---

### 2.4 Table `work_library`

| Field name | Công thức |
|---|---|
| `work_item_code` | `CONCATENATE("W-", TEXT({auto_num}, "000"))` |

**Kết quả**: `W-001`, `W-002`... Cột này sẽ điền ngay 24 mã cho 24 rows đã seed.

---

### 2.5 Table `sample_products`

| Field name | Công thức |
|---|---|
| `sample_code` | `CONCATENATE("SMP-", TEXT({auto_num}, "000"))` |

**Kết quả**: `SMP-001`...

---

## ⚠️ Nếu Lark báo lỗi công thức

- **Lỗi "field not found {auto_num}"**: Nghĩa là cột `auto_num` bị đổi tên hoặc xóa. Kiểm tra lại table xem còn cột `auto_num` không. Nếu không → thêm field type **"AutoNumber"** (Số tự tăng) tên `auto_num`.
- **Lỗi "unknown function CONCATENATE"**: Lark dùng tiếng Anh cho function. Nếu giao diện Nam đang tiếng Việt, thử đổi sang English ở Settings → hoặc dùng tên Lark dịch (ít khả năng).

✅ Xong Phần 2 khi cả 5 tables đều có cột mã mới.

---

# PHẦN 3 — Thêm 2 cột phụ (5 phút)

## 3.1 Table `work_library` — thêm cột `phase`

**Tại sao**: Để khi nhìn work item, thấy luôn nó thuộc giai đoạn nào (Trước xây / Phần thô / Hoàn thiện) mà không cần click sang category.

**Các bước**:

1. Mở tab `work_library`
2. Bấm dấu **"+"** cuối hàng field → chọn type **"Lookup"** (Tra cứu)
3. **Field name**: `phase`
4. **Lookup target**:
   - **Lookup through field**: chọn `category` (cột SingleLink đã có)
   - **Lookup value from**: chọn `phase` (từ table categories_master)
5. Bấm **"Confirm"**

**Kết quả**: Mỗi row work_library giờ tự hiển thị giai đoạn (vd: "Khởi công" → phase "Phần thô").

---

## 3.2 Table `customers_master` — thêm cột `created_at`

**Tại sao**: Tracking thời gian tạo record khách — cần cho báo cáo và audit.

**Các bước**:

1. Mở tab `customers_master`
2. Bấm dấu **"+"** cuối hàng field → chọn type **"Created time"** (Thời gian tạo)
3. **Field name**: `created_at`
4. Bấm **"Confirm"**

**Kết quả**: Cột `created_at` tự hiện timestamp khi có record mới.

✅ Xong Phần 3 khi cả 2 cột này đều đã thêm.

---

# PHẦN 4 — Gom data staff + vendor (Nam tự làm offline)

## 4.1 Danh sách nhân viên (Staff)

Nam mở file Excel/Sheet/Word riêng, điền theo format sau rồi gửi tôi:

```
Full Name | Role | Email | Phone | Joined Date
Đỗ Hoài Nam | BOD | nam.dh9939@gmail.com | 09xxxxxxxx | 01/01/2024
Nguyễn Văn A | PM | a@tlxn.vn | 09xxxxxxxx | 15/03/2025
Trần Thị B | CA | b@tlxn.vn | 09xxxxxxxx | 20/06/2025
...
```

**Role available**: `BOD`, `PM`, `AA`, `CA`, `Account`, `PO`, `PD`, `Sale`, `TK`, `QA`

**Lưu ý**:
- Ghi đủ TẤT CẢ NV đang làm (và NV đã nghỉ nếu họ từng quản dự án — cho audit lịch sử)
- Email chính xác (sau này Lark có thể link với Lark account qua email)
- Nếu NV kiêm nhiệm nhiều role, ghi role chính ở cột "Role", các role phụ để ghi chú

---

## 4.2 Danh sách nhà thầu / nhà cung cấp (Vendors)

Đây là phần **tốn nhiều thời gian nhất** vì phải dedupe (gộp trùng lặp). Nhưng làm 1 lần, dùng cả đời công ty.

### Cách làm:

**Bước 1 — Extract danh sách từ HBSS cũ**

Nam mở file HBSS mẫu: https://trolyxaynha029.sg.larksuite.com/base/Ju2dbkBEzaPX6KskHQSlnI3Mgvb

Tìm các tables có field **"Tên đơn vị thi công"** (text tự do) — Table 1, Table 4, Table 6. Export hoặc copy các tên đơn vị thi công ra Sheet/Excel.

Ngoài ra, Nam có thể mở thêm các file HBSS của dự án đã/đang chạy khác để lấy thêm tên nhà thầu.

**Bước 2 — Dedupe (gộp trùng)**

Mở Sheet/Excel, sort alphabet. Những cái giống nhau/gần giống sẽ nằm cạnh nhau, vd:
- "Cty ABC"
- "Công ty ABC"
- "ABC"
- "ABC Co."

→ Chọn 1 tên chuẩn (vd: "Công ty TNHH ABC") để đại diện, xóa các dòng trùng.

**Bước 3 — Điền thông tin đầy đủ**

Sau khi dedupe, bổ sung các cột:

```
company_name | vendor_type | category_specialization | contact_person | phone | status
Công ty TNHH ABC | NTP-Thi công | BTCT móng, BTCT tầng 1 | Anh Minh | 0901234567 | Active
Cửa hàng Xi măng X | NCC-Vật tư | (để trống) | Cô Lan | 0912345678 | Active
...
```

**Column explain**:
- `vendor_type`: chọn 1 hoặc nhiều trong: `NTP-Thi công`, `NTP-Thiết kế`, `NCC-Vật tư`, `NCC-Thiết bị`
- `category_specialization`: tên category (tìm trong file `03-data-model-v2.md` → 44 hạng mục). Để trống nếu vendor phủ rộng.
- `status`: `Active` (đang hợp tác), `Trial` (mới thử), `Blacklisted` (ngừng hợp tác)

💡 **Giảm effort**: Nếu Nam có nhiều vendors (50-100+), có thể làm theo batch:
- Batch 1: 10-20 vendors hay dùng nhất → gửi tôi seed trước
- Batch 2-3: những cái còn lại, làm dần

---

## 4.3 Sample products (để sau cũng được)

Sample products cần upload ảnh → API khó làm. Nam cứ để đó, làm tay qua UI sau cũng được, không gấp.

Nếu muốn làm:
- Mở HBSS cũ → Table 8 → filter `Chức năng bảng = Sản phẩm mẫu`
- Download ảnh
- Upload vào `sample_products` qua UI
- Gắn category cho từng sample

---

# PHẦN 5 — Gửi data cho Claude seed

Khi Nam có data staff + vendor sẵn sàng:

1. Paste data vào chat với Claude (session hiện tại hoặc session mới)
2. Bảo: **"Seed staff + vendors vào Master Base giúp tôi. Data đây: [paste data]"**
3. Claude sẽ:
   - Parse data
   - Seed qua MCP Lark
   - Confirm số lượng đã seed
   - Báo nếu có row nào lỗi (vd: email trùng, role không hợp lệ)

Nếu Nam mở session mới → nhớ paste context từ `05-handoff.md` trước, rồi mới paste data.

---

# ❓ FAQ — Nam hay gặp

### Q1: Làm sai bước 2 (Formula), muốn làm lại?
- Chuột phải cột Formula sai → **"Delete field"**
- Thêm lại theo hướng dẫn.

### Q2: Field Lookup (Phần 3.1) không thấy option chọn `phase`?
- Đảm bảo `work_library` đã có cột Link `category` (đã có rồi, đã seed link)
- Nếu vẫn lỗi, refresh trang (F5) rồi thử lại.

### Q3: Nam chưa có danh sách staff/vendor sẵn, làm sao?
- Phần 1-3 vẫn làm được ngay (không phụ thuộc data).
- Phần 4 làm từ từ trong 1-2 tuần. Khi nào có data, gửi Claude.

### Q4: Nam muốn add field mới không trong plan?
- Cứ add qua UI. Sau đó báo Claude: "Tôi đã add field X ở table Y, update lại doc giúp"
- Claude sẽ update `03-data-model-v2.md` cho đồng bộ.

### Q5: Đến lúc nào chuyển qua Phase 2 Sales (D)?
- Sau khi Nam xong Phần 1-3 (không cần Phần 4 xong hết).
- Customers_master schema OK rồi → Phase 2 Sales sẽ bổ sung field Sales-specific.

---

# 🎯 Khi Nam xong hết Đợt 2

Nhắn Claude: **"Đợt 2 Master Base xong, đi tiếp Phase 2 Sales"**

Claude sẽ:
1. Verify Base qua MCP (đếm records, check fields)
2. Update doc `07-master-base-handoff.md` → mark Đợt 2 completed
3. Bắt đầu thiết kế Phase 2 Sales (event list, SKU, CRM schema, validation gate 25 mục...)

---

## 📝 Ghi chú

File này nằm ở: `C:\Github\tlxn-ai-agent\08-dot-2-huong-dan-tung-buoc.md`

Trên GitHub: https://github.com/namdh9939/NCM-AGENTICAI/blob/main/08-dot-2-huong-dan-tung-buoc.md

Nam có thể xem trên điện thoại/tablet lúc ngồi làm Lark.
