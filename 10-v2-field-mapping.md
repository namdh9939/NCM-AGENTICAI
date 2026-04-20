# Checklist Cấu hình Cột (Field Mapping) cho HBSS v2

Do giới hạn về API không tự tạo được cột Link (liên kết chéo) và cột Options (lựa chọn thả xuống), tất cả các cột này hiện đang đứng tạm ở dạng **Text (Văn bản)**. Bạn chỉ cần *Double-click* vào tên cột và chuyển Type theo đúng bảng dưới đây.

**Mẹo làm nhanh:** Ưu tiên làm các cột Link trước để dữ liệu thông nhau, các cột Dropdown (Single/Multi Select) cứ từ từ điền Options dần trong quá trình dùng.

---

## 1. Các bảng nền tảng (Base Info)

### Bảng `Thông tin Dự án`
- [ ] `Khách hàng` ➡️ Đổi sang **Link** ➜ Chọn Master Base ➜ Bảng `Danh mục khách hàng`
- [ ] Các cột Dropdown (Đổi sang SingleSelect/MultiSelect): `Quận/Huyện`, `Tỉnh/Thành phố`, `building_type`, `architecture_style`, `package_sku`, `Trạng thái`

### Bảng `Phân công nhân sự`
- [ ] `Nhân sự` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục nhân sự`
- [ ] `Vai trò` ➡️ Dropdown

### Bảng `Quản lý các bên liên quan`
- [ ] `Đơn vị/Đối tác` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục đối tác`
- [ ] `Hạng mục` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] `Hợp đồng liên quan` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ Bảng `Quản lý Hợp đồng`
- [ ] Các cột Dropdown: `role_in_project`, `Trạng thái`

---

## 2. Kế hoạch & Công việc (Planning & Tasks)

### Bảng `Tiến độ dự án`
- [ ] `Hạng mục` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] `owner_staff` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục nhân sự`
- [ ] `parent_milestone` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ Bảng `Tiến độ dự án`
- [ ] `owner_role` ➡️ Dropdown

### Bảng `Checklist công việc chi tiết`
- [ ] `work_library_ref` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Thư viện đầu việc`
- [ ] `owner_staff` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục nhân sự`
- [ ] `Mốc tiến độ` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `Tiến độ dự án`
- [ ] `parent_task` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `Checklist công việc chi tiết`
- [ ] Các cột Dropdown: `owner_role`, `Trạng thái`

---

## 3. Báo cáo & Nhật ký (Reporting & Logs)

### Bảng `Nhật ký công trình`
- [ ] `Đơn vị/Đối tác` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục đối tác`
- [ ] Các cột Dropdown: `delay_risk`, `daily_status`

### Bảng `Báo cáo nghiệm thu nội bộ`
- [ ] `Hạng mục` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] `work_item` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Thư viện đầu việc`
- [ ] `quality` ➡️ Dropdown

### Bảng `Quản lý vấn đề và Sự cố`
- [ ] `creator` & `assignee` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục nhân sự`
- [ ] `related_category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] Các cột Dropdown: `related_areas`, `Trạng thái`

### Bảng `Báo cáo tuần`
- [ ] `Mốc tiến độ` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `Tiến độ dự án`
- [ ] Các cột Dropdown: `progress_assessment`, `design_compliance`, `reviewer`

### Bảng `Chấm điểm đối tác`
- [ ] `Đơn vị/Đối tác` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục đối tác`
- [ ] `Hạng mục` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] `reviewer` ➡️ Dropdown

---

## 4. Tài chính & Hợp đồng (Finance & Legal)

### Bảng `Quản lý Hợp đồng`
- [ ] `contract_type` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Loại hợp đồng mẫu`
- [ ] `party_a` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục khách hàng`
- [ ] `party_b` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục đối tác`
- [ ] `related_category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] `Trạng thái` ➡️ Dropdown

### Bảng `Báo giá & HSNL`
- [ ] `Hạng mục` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] `Đơn vị/Đối tác` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục đối tác`

### Bảng `Dự toán hạng mục`
- [ ] `Hạng mục` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] `Trạng thái` ➡️ Dropdown

### Bảng `Theo dõi Thanh toán`
- [ ] `Hợp đồng liên quan` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `Quản lý Hợp đồng`
- [ ] `Người thụ hưởng` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục đối tác` (có thể custom tùy bạn nếu trả cho khách)
- [ ] Các cột Dropdown: `payment_type`, `payment_method`

### Bảng `Quản lý Phát sinh` (Phát sinh)
- [ ] `affected_categories` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] `approved_by` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục nhân sự`
- [ ] `related_contract` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `Quản lý Hợp đồng`
- [ ] Các cột Dropdown: `requested_by`, `approval_status`

---

## 5. Khác (Misc)

### Bảng `Kho tài liệu dự án`
- [ ] `uploaded_by` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục nhân sự`
- [ ] `related_category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] `related_contract` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `Quản lý Hợp đồng`
- [ ] `Hạng mục` ➡️ Dropdown

### Bảng `Công tác Nghiệm thu` (Nghiệm thu)
- [ ] `Hạng mục` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục hạng mục`
- [ ] `inspector_internal` ➡️ Đổi sang **Link** ➜ Master Base ➜ `Danh mục nhân sự`
- [ ] `Mốc tiến độ` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `Tiến độ dự án`
- [ ] `result` ➡️ Dropdown

### Bảng `Cấu hình Base`
- (Không cần chỉnh gì)
