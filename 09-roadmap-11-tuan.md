# Lộ trình 11 Tuần triển khai AI Agent - TLXN

**Bắt đầu:** Tuần 1 (Đã hoàn thành)
**Trạng thái:** Đang triển khai Tuần 2

## Giai đoạn 1: Nền tảng vận hành & Data (Tuần 1 - Tuần 4)

- [x] **Tuần 1: Hoàn tất Master Base**
  - Xóa bảng rỗng
  - Gắn mã tự động 100% cho các bảng (`staff_code`, `vendor_code`,...)
  - Seed dữ liệu thành công (15 nhân sự, 35 nhà thầu)
- [x] **Tuần 2: Thiết kế file mẫu dự án (HBSS V2)**
  - Đã thiết lập thành công 18 bảng (project_info, milestones, daily_logs, contracts, payments...) thông qua API.
  - Các cấu hình Link/Select đang được hoàn thiện bằng tay.
- [ ] **Tuần 3: Đưa thử nghiệm vào dự án thực tế**
  - Copy file mẫu ra 3 dự án test
  - Đưa PM, CA, Account vào chạy thử, kiểm tra lỗi và dòng chảy dữ liệu.
- [ ] **Tuần 4: Xây dựng kho dữ liệu tổng (Supabase)**
  - Chọn cấu trúc database để đồng bộ liên tục 1 chiều từ Lark về Supabase.
  - Làm Source of Truth (nguồn sống) duy nhất cho mọi suy luận bằng AI.

## Giai đoạn 2: An toàn & Tiêu chuẩn AI (Tuần 5 - Tuần 7)

- [ ] **Tuần 5-6: Xây dựng khiên an toàn AI (Guardrails)**
  - Nút tắt khẩn (Kill Switch) và cơ chế gọi người hỗ trợ.
  - Xây Audit Trail: ghi lại mọi lịch sử hành động hệ thống.
  - Cơ chế Access Control & Khóa bảo mật API.
- [ ] **Tuần 7: Hệ thống chấm điểm AI (Eval Framework)**
  - Chuẩn bị bộ câu hỏi mẫu.
  - Trắc nghiệm QA: để test AI trả lời đúng/sai trước khi cấp token cho liên hệ khách.

## Giai đoạn 3: Nghiệp vụ AI vào dự án (Tuần 8 - Tuần 10)

- [ ] **Tuần 8: Chuẩn hóa/Map luồng QLDA**
  - Liệt kê toàn bộ công việc theo vòng đời dự án.
  - Mapping: Ai làm gì, dữ liệu ở đâu, điểm rơi lỗi thường xảy ra.
- [ ] **Tuần 9-10: Thiết kế luồng cho AI (Agent SOP)**
  - Lên kịch bản nếu/thì, kích hoạt/quyết định thế nào ở từng đầu việc.
  - Xây dựng Validation Gate (Cổng kiểm tra chéo) giữa các khâu.

## Giai đoạn 4: Xây dựng & Pilot (Tuần 11+)

- [ ] **Tuần 11+: Build AI & Test 3 Lớp**
  - Offline Eval (Chạy ẩn chấm điểm) -> Shadow Mode (Chạy nền song song người thực) -> Pilot (Áp dụng 1-3 dự án thật).
