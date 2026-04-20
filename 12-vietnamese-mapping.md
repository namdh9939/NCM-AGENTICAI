# Bản đồ Việt hóa Dữ liệu (Vietnamese Data Mapping) — TLXN HBSS v2

Bản tài liệu này cung cấp danh sách đối chiếu tên bảng và tên cột từ tiếng Anh sang tiếng Việt để chuẩn hóa giao diện cho nhân sự sử dụng trực tiếp trên Lark Base.

---

## 1. LAYER 1 — MASTER BASE (Dữ liệu dùng chung)

| Tên bảng (EN) | Tên bảng (VI) | Mô tả |
|---|---|---|
| `customers_master` | **Danh mục khách hàng** | Thông tin khách hàng toàn công ty |
| `staff_master` | **Danh mục nhân sự** | Danh sách nhân viên và vai trò |
| `vendors_master` | **Danh mục đối tác** | Nhà thầu, nhà cung cấp, đội nhóm thi công |
| `categories_master` | **Danh mục hạng mục** | 44 hạng mục thi công chuẩn |
| `work_library` | **Thư viện đầu việc** | Checklist chi tiết cho từng hạng mục |
| `sample_products` | **Thư viện mẫu sản phẩm** | Hình ảnh, mẫu sản phẩm hoàn thiện |
| `contract_types_master`| **Loại hợp đồng mẫu** | Các mẫu HĐ TLXN, Thiết kế, Thi công... |

---

## 2. LAYER 2 — HBSS v2 PROJECT TEMPLATE (Bảng dự án)

Dưới đây là chi tiết việt hóa cho 18 bảng trong file dự án HBSS v2.

### 2.1 Thông tin Dự án (`project_info`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `project_code` | Mã dự án |
| `project_name` | Tên dự án |
| `customer` | Khách hàng |
| `site_address` | Địa chỉ công trình |
| `district` | Quận/Huyện |
| `city` | Tỉnh/Thành phố |
| `land_area_m2` | Diện tích đất (m2) |
| `building_area_m2` | Diện tích xây dựng (m2) |
| `floors_count` | Số tầng |
| `basement_count` | Số tầng hầm |
| `building_type` | Loại hình kiến trúc |
| `architecture_style` | Phong cách thiết kế |
| `total_budget_vnd` | Tổng ngân sách (VNĐ) |
| `package_sku` | Gói dịch vụ |
| `contract_sign_date` | Ngày ký hợp đồng |
| `construction_start_date` | Ngày khởi công (dự kiến) |
| `construction_start_actual` | Ngày khởi công (thực tế) |
| `handover_planned_date` | Ngày bàn giao (dự kiến) |
| `handover_actual_date` | Ngày bàn giao (thực tế) |
| `status` | Trạng thái dự án |
| `lark_chat_customer_id` | ID Chat Khách hàng |
| `lark_chat_internal_id` | ID Chat Nội bộ |

### 2.2 Phân công nhân sự (`project_assignments`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `staff` | Nhân sự |
| `role` | Vai trò trong dự án |
| `assigned_from` | Ngày bắt đầu nhận task |
| `assigned_to` | Ngày kết thúc task |
| `notes` | Ghi chú |

### 2.3 Quản lý các bên liên quan (`project_stakeholders`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `vendor` | Đơn vị/Đối tác |
| `category` | Hạng mục phụ trách |
| `role_in_project` | Vai trò tham gia |
| `contract` | Hợp đồng liên quan |
| `status` | Trạng thái hợp tác |
| `joined_date` | Ngày tham gia |
| `end_date` | Ngày rời đi/Kết thúc |

### 2.4 Tiến độ dự án (`project_milestones`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `milestone_name` | Tên mốc tiến độ |
| `category` | Hạng mục chính |
| `planned_start` | Ngày bắt đầu (Kế hoạch) |
| `planned_end` | Ngày kết thúc (Kế hoạch) |
| `actual_start` | Ngày bắt đầu (Thực tế) |
| `actual_end` | Ngày kết thúc (Thực tế) |
| `extension_date` | Ngày gia hạn |
| `progress_percent` | % Hoàn thành |
| `is_critical` | Mốc trọng yếu? |
| `owner_role` | Bộ phận phụ trách |
| `owner_staff` | Người phụ trách |
| `parent_milestone` | Mốc cha |

### 2.5 Checklist công việc chi tiết (`project_tasks`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `task_code` | Mã công việc |
| `task_name` | Tên công việc/Checklist |
| `work_library_ref` | Tham chiếu thư viện |
| `milestone` | Thuộc mốc tiến độ |
| `parent_task` | Công việc cha |
| `owner_role` | Bộ phận phụ trách |
| `owner_staff` | Người phụ trách |
| `planned_start` | Bắt đầu dự kiến |
| `planned_end` | Kết thúc dự kiến |
| `actual_completion` | Thực tế hoàn thành |
| `status` | Trạng thái |
| `report_attachment` | File báo cáo |

### 2.6 Nhật ký công trình (`project_daily_logs`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `log_date` | Ngày nhật ký |
| `reporter_name` | Người báo cáo |
| `vendor` | Đơn vị thi công |
| `work_today` | Nội dung đã làm |
| `work_tomorrow` | Dự kiến ngày mai |
| `issues_proposals` | Vấn đề & Kiến nghị |
| `workers_count` | Số lượng nhân công |
| `delay_risk` | Rủi ro chậm trễ |
| `daily_status` | Tình trạng hiện diện |
| `site_photos` | Hình ảnh hiện trường |

### 2.7 Báo cáo nghiệm thu nội bộ (`project_task_reports`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `report_date` | Ngày báo cáo |
| `category` | Hạng mục |
| `work_item` | Đầu việc nghiệm thu |
| `completion_percent` | % Đã làm xong |
| `quality` | Đánh giá chất lượng |
| `confirmed_complete` | Xác nhận hoàn thành |
| `photos_ok` | Ảnh đạt chuẩn |
| `photos_issues` | Ảnh lỗi/cần sửa |

### 2.8 Quản lý vấn đề/Sự cố (`project_issues`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `created_date` | Ngày phát sinh |
| `description` | Mô tả vấn đề |
| `related_areas` | Lĩnh vực ảnh hưởng |
| `related_category` | Thuộc hạng mục |
| `media` | Hình ảnh/Video minh chứng |
| `deadline` | Hạn xử lý |
| `status` | Trạng thái xử lý |
| `resolution_action` | Biện pháp giải quyết |
| `assignee` | Người chịu trách nhiệm |

### 2.9 Báo cáo tuần (`project_weekly_reports`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `report_week` | Tuần báo cáo |
| `milestone` | Mốc tiến độ liên quan |
| `progress_assessment` | Đánh giá tiến độ |
| `design_compliance` | Tuân thủ thiết kế |
| `legal_note` | Ghi chú pháp lý |
| `quality_note` | Ghi chú chất lượng |
| `reviewer` | Người duyệt báo cáo |

### 2.10 Chấm điểm đối tác (`vendor_scorecards`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `month_year` | Tháng/Năm chấm điểm |
| `vendor` | Đối tác được được chấm |
| `category` | Hạng mục thực hiện |
| `score_cooperation` | Điểm phối hợp |
| `score_report_quality` | Điểm chất lượng báo cáo |
| `score_response_time` | Điểm phản hồi |
| `score_cost_control` | Điểm kiểm soát chi phí |
| `score_supervision` | Điểm giám sát |
| `score_consulting` | Điểm tư vấn |
| `score_vendor_support` | Điểm hỗ trợ |
| `total_score` | Tổng điểm trung bình |

### 2.11 Quản lý Hợp đồng (`project_contracts`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `contract_id` | Mã hợp đồng |
| `contract_type` | Loại hợp đồng |
| `title` | Tên hợp đồng |
| `party_a` | Bên A (Chủ đầu tư) |
| `party_b` | Bên B (Nhà thầu/TLXN) |
| `sign_date` | Ngày ký kết |
| `total_value_vnd` | Giá trị hợp đồng (VNĐ) |
| `scanned_doc` | File bản scan |
| `status` | Tình trạng hợp đồng |

### 2.12 Báo giá & HSNL (`project_quotes`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `quote_date` | Ngày báo giá |
| `category` | Hạng mục báo giá |
| `vendor` | Đơn vị báo giá |
| `amount_vnd` | Số tiền (VNĐ) |
| `quote_doc` | File báo giá |
| `selected` | Được chọn? |

### 2.13 Dự toán hạng mục (`project_budget_items`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `category` | Hạng mục dự toán |
| `planned_amount_vnd` | Ngân sách dự kiến |
| `status` | Trạng thái phê duyệt |

### 2.14 Theo dõi Thanh toán (`project_payments`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `payment_date` | Ngày thanh toán |
| `contract` | Thuộc hợp đồng |
| `payment_type` | Loại thanh toán |
| `installment_no` | Đợt thanh toán số |
| `payee` | Người thụ hưởng |
| `amount_actual_vnd` | Số tiền chi thực tế |
| `payment_method` | Phương thức thanh toán |
| `receipt_doc` | Chứng từ chi |
| `approved_by` | Người phê duyệt |

### 2.15 Quản lý Phát sinh (`project_change_orders`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `requested_date` | Ngày yêu cầu |
| `requested_by` | Bên yêu cầu |
| `description` | Nội dung phát sinh |
| `cost_impact_vnd` | Chi phí thay đổi (+/-) |
| `schedule_impact_days` | Số ngày ảnh hưởng (+/-) |
| `approval_status` | Trạng thái phê duyệt |
| `related_contract` | Hợp đồng gốc |

### 2.16 Kho tài liệu dự án (`project_documents`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `upload_date` | Ngày tải lên |
| `uploaded_by` | Người tải lên |
| `category` | Loại tài liệu |
| `title` | Tên tài liệu |
| `file` | File đính kèm |

### 2.17 Công tác Nghiệm thu (`project_inspections`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `inspection_date` | Ngày nghiệm thu |
| `category` | Hạng mục nghiệm thu |
| `milestone` | Mốc tiến độ liên quan |
| `inspector_internal` | Người nghiệm thu nội bộ |
| `result` | Kết quả nghiệm thu |
| `issues_found` | Vấn đề tồn đọng |
| `signed_doc` | Biên bản có chữ ký |

### 2.18 Cấu hình Base (`project_settings`)
| Field (EN) | Tên cột (VI) |
|---|---|
| `setting_key` | Khóa cấu hình |
| `setting_value` | Giá trị |
| `notes` | Ghi chú |

---
**Ghi chú:** Khi thực hiện thay đổi, ta sẽ giữ nguyên ID hoặc Key kỹ thuật bên dưới (nếu có thể) nhưng hiển thị Name là tiếng Việt để đảm bảo AI Agent vẫn có thể mapping dữ liệu chính xác qua API.
