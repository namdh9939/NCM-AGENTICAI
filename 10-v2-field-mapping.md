# Checklist Cấu hình Cột (Field Mapping) cho HBSS v2

Do giới hạn về API không tự tạo được cột Link (liên kết chéo) và cột Options (lựa chọn thả xuống), tất cả các cột này hiện đang đứng tạm ở dạng **Text (Văn bản)**. Bạn chỉ cần *Double-click* vào tên cột và chuyển Type theo đúng bảng dưới đây.

**Mẹo làm nhanh:** Ưu tiên làm các cột Link trước để dữ liệu thông nhau, các cột Dropdown (Single/Multi Select) cứ từ từ điền Options dần trong quá trình dùng.

---

## 1. Các bảng nền tảng (Base Info)

### Bảng `project_info`
- [ ] `customer` ➡️ Đổi sang **Link** ➜ Chọn Master Base ➜ Bảng `customers_master`
- [ ] Các cột Dropdown (Đổi sang SingleSelect/MultiSelect): `district`, `city`, `building_type`, `architecture_style`, `package_sku`, `status`

### Bảng `project_assignments`
- [ ] `staff` ➡️ Đổi sang **Link** ➜ Master Base ➜ `staff_master`
- [ ] `role` ➡️ Dropdown

### Bảng `project_stakeholders`
- [ ] `vendor` ➡️ Đổi sang **Link** ➜ Master Base ➜ `vendors_master`
- [ ] `category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] `contract` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ Bảng `project_contracts`
- [ ] Các cột Dropdown: `role_in_project`, `status`

---

## 2. Kế hoạch & Công việc (Planning & Tasks)

### Bảng `project_milestones`
- [ ] `category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] `owner_staff` ➡️ Đổi sang **Link** ➜ Master Base ➜ `staff_master`
- [ ] `parent_milestone` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ Bảng `project_milestones`
- [ ] `owner_role` ➡️ Dropdown

### Bảng `project_tasks`
- [ ] `work_library_ref` ➡️ Đổi sang **Link** ➜ Master Base ➜ `work_library`
- [ ] `owner_staff` ➡️ Đổi sang **Link** ➜ Master Base ➜ `staff_master`
- [ ] `milestone` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `project_milestones`
- [ ] `parent_task` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `project_tasks`
- [ ] Các cột Dropdown: `owner_role`, `status`

---

## 3. Báo cáo & Nhật ký (Reporting & Logs)

### Bảng `project_daily_logs`
- [ ] `vendor` ➡️ Đổi sang **Link** ➜ Master Base ➜ `vendors_master`
- [ ] Các cột Dropdown: `delay_risk`, `daily_status`

### Bảng `project_task_reports`
- [ ] `category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] `work_item` ➡️ Đổi sang **Link** ➜ Master Base ➜ `work_library`
- [ ] `quality` ➡️ Dropdown

### Bảng `project_issues`
- [ ] `creator` & `assignee` ➡️ Đổi sang **Link** ➜ Master Base ➜ `staff_master`
- [ ] `related_category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] Các cột Dropdown: `related_areas`, `status`

### Bảng `project_weekly_reports`
- [ ] `milestone` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `project_milestones`
- [ ] Các cột Dropdown: `progress_assessment`, `design_compliance`, `reviewer`

### Bảng `vendor_scorecards`
- [ ] `vendor` ➡️ Đổi sang **Link** ➜ Master Base ➜ `vendors_master`
- [ ] `category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] `reviewer` ➡️ Dropdown

---

## 4. Tài chính & Hợp đồng (Finance & Legal)

### Bảng `project_contracts`
- [ ] `contract_type` ➡️ Đổi sang **Link** ➜ Master Base ➜ `contract_types_master`
- [ ] `party_a` ➡️ Đổi sang **Link** ➜ Master Base ➜ `customers_master`
- [ ] `party_b` ➡️ Đổi sang **Link** ➜ Master Base ➜ `vendors_master`
- [ ] `related_category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] `status` ➡️ Dropdown

### Bảng `project_quotes`
- [ ] `category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] `vendor` ➡️ Đổi sang **Link** ➜ Master Base ➜ `vendors_master`

### Bảng `project_budget_items`
- [ ] `category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] `status` ➡️ Dropdown

### Bảng `project_payments`
- [ ] `contract` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `project_contracts`
- [ ] `payee` ➡️ Đổi sang **Link** ➜ Master Base ➜ `vendors_master` (có thể custom tùy bạn nếu trả cho khách)
- [ ] Các cột Dropdown: `payment_type`, `payment_method`

### Bảng `project_change_orders` (Phát sinh)
- [ ] `affected_categories` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] `approved_by` ➡️ Đổi sang **Link** ➜ Master Base ➜ `staff_master`
- [ ] `related_contract` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `project_contracts`
- [ ] Các cột Dropdown: `requested_by`, `approval_status`

---

## 5. Khác (Misc)

### Bảng `project_documents`
- [ ] `uploaded_by` ➡️ Đổi sang **Link** ➜ Master Base ➜ `staff_master`
- [ ] `related_category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] `related_contract` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `project_contracts`
- [ ] `category` ➡️ Dropdown

### Bảng `project_inspections` (Nghiệm thu)
- [ ] `category` ➡️ Đổi sang **Link** ➜ Master Base ➜ `categories_master`
- [ ] `inspector_internal` ➡️ Đổi sang **Link** ➜ Master Base ➜ `staff_master`
- [ ] `milestone` ➡️ Đổi sang **Link** ➜ TRONG FILE NÀY ➜ `project_milestones`
- [ ] `result` ➡️ Dropdown

### Bảng `project_settings`
- (Không cần chỉnh gì)
