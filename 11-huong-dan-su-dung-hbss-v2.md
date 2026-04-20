# Hướng Dẫn Sử Dụng: QUẢN LÝ DỰ ÁN VỚI HBSS V2 🚀

Chào mừng các bạn đến với hệ thống HBSS v2! Nhằm mục tiêu giúp AI có thể tự động đọc báo cáo, tự động nhắc việc và cảnh báo tiến độ thay cho con người, file quản lý dự án nội bộ của chúng ta đã được nâng cấp toàn diện.

**Nguyên tắc "Bàn Tay Sắt" của V2:**
> [!IMPORTANT]  
> Các bạn **TUYỆT ĐỐI KHÔNG ĐƯỢC gõ tay** tên Nhà thầu, tên Khách hàng hay tên các hạng mục công việc. Toàn bộ thông tin này phải được chọn từ cột **Link (biểu tượng móc xích)** liên kết trực tiếp về Master Base của công ty để quản lý chặt chẽ dữ liệu. Nếu chưa có trong danh sách, yêu cầu thư ký hoặc quản trị viên (Admin) cập nhật vào Master Base trước.

Hệ thống được chia rành mạch chức năng theo từng bộ phận để tránh chồng chéo dữ liệu. Dưới đây là sổ tay hướng dẫn cụ thể:

---

## 👷‍♂️ 1. Dành cho Quản lý dự án (PM) & Kỹ sư hiện trường
Trách nhiệm chính: Bám sát tiến độ, nghiệm thu chất lượng và báo cáo vấn đề phát sinh tại công trình.

**Khởi tạo đầu dự án:**
*   **`project_info`**: Nhập đầy đủ thông số ban đầu (Diện tích, số tầng, quy mô, ngày khởi công dự kiến).
*   **`project_milestones`**: Thiết lập các mốc tiến độ lớn (VD: Ép cọc, đổ móng, cất nóc...).

**Công việc hàng ngày (Thực hiện vào mỗi buổi chiều):**
1.  Truy cập bảng **`project_daily_logs` (Nhật ký dự án)**:
    *   Tạo dòng mới (chọn đúng Ngày hiện tại).
    *   Chọn tên Nhà thầu từ danh sách Liên kết (Link).
    *   Ghi ngắn gọn nội dung công việc hôm nay/ngày mai và số lượng nhân công thi công.
    *   *Bắt buộc:* Tải lên 1-3 ảnh thực tế tại công trường.

**Xử lý sự cố & Nghiệm thu:**
*   **Nghiệm thu đạt:** Truy cập bảng **`project_inspections`** -> Chọn mốc tiến độ -> Đánh dấu ✅ Pass và đính kèm biên bản xác nhận.
*   **Vấn đề phát sinh (Lỗi/Chậm tiến độ):** Tạo ngay một Ticket tại bảng **`project_issues`**. Giao trách nhiệm cho cá nhân/nhà thầu liên quan và thiết lập thời hạn (Deadline) xử lý.

**Báo cáo cuối tuần:**
*   Truy cập bảng **`project_weekly_reports`**: Chỉ cần đánh giá trạng thái (Đúng hạn/Chậm) và tính tuân thủ thiết kế (Đúng/Sai lệch). AI sẽ tự động tổng hợp từ Nhật ký dự án để soạn báo cáo chi tiết gửi khách hàng.

---

## 👩‍💼 2. Dành cho Thư ký dự án (CA), Kế toán & Mua hàng
Trách nhiệm chính: Quản lý tính minh bạch tài chính. Mọi khoản thanh toán phải đối chiếu trực tiếp với Hợp đồng gốc.

*   **Mời thầu / Duyệt báo giá:** Tải file báo giá của các nhà thầu vào bảng **`project_quotes`**. Đánh dấu "Selected" cho nhà thầu được chọn.
*   **Ký kết Hợp đồng:** Tạo dòng mới tại bảng **`project_contracts`**. Nhập tổng giá trị (VND), chọn Nhà thầu từ Master Base và tải lên bản scan PDF.
*   **Thực hiện Thanh toán:** 
    *   Tạo dòng mới tại bảng **`project_payments`**. 
    *   **Bắt buộc:** Phải Liên kết (Link) với đúng Hợp đồng tương ứng. Nhập số tiền thực chi và đính kèm bằng chứng thanh toán (UNC/Biên lai). Hệ thống sẽ tự động tính toán dư nợ còn lại của gói thầu.
*   **Xử lý Phát sinh:** Các yêu cầu phát sinh ngoài hợp đồng phải được nhập vào bảng **`project_change_orders`**. Chỉ sau khi được cấp quản lý phê duyệt tại đây mới được tiến hành thanh toán.

---

## 🕵️ 3. Dành cho Ban quản lý & Bộ phận Kiểm soát chất lượng (QA)
*   **Đánh giá Nhà thầu (Định kỳ hàng tháng):** Truy cập bảng **`vendor_scorecards`**. Chọn Nhà thầu và chấm điểm theo thang 1-5 (Thái độ hợp tác, Tiến độ, Chất lượng). Kết quả này là cơ sở để AI đưa ra đề xuất tiếp tục hợp tác hay loại bỏ nhà thầu trong các dự án tương lai.

---

💡 **Mẹo thao tác nhanh trên Lark:**
- Sử dụng các chế độ hiển thị (**Views**) khác nhau như Kanban, Calendar hay Gantt để quản lý trực quan hơn. Ví dụ: Dùng View Kanban cho bảng `project_issues` để theo dõi trạng thái xử lý lỗi.
- Nếu lỡ tay xóa hoặc nhập sai dữ liệu, hãy nhấn **Ctrl + Z** (Win) hoặc **Cmd + Z** (Mac) để hoàn tác ngay lập tức.
