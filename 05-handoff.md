# Handoff — Session mới

**Paste nguyên block dưới vào session Claude mới để tiếp tục từ điểm dừng**

---

## 📋 CONTEXT PROMPT CHO SESSION MỚI

```
Tôi là Nam, chủ TLXN (Trợ Lý Xây Nhà) — công ty quản lý xây nhà dân dụng tại VN.
Tôi đang triển khai AI Agent cho công ty, mục tiêu scale 100 → 1000 dự án/năm (2026-2030).

Tôi đã làm việc với Claude ở session trước và chốt:
- Plan triển khai AI Agent v3 final (18 task Phase 0, Phase 1 QLDA, Phase 2 Sales, Phase 3)
- Review schema HBSS hiện tại (8 tables Lark Base) → tìm 10 anti-patterns
- Thiết kế Data Model v2: Master Base (8 tables) + Per-project Template (17 tables) + Warehouse (28 entities)
- Setup xong Lark MCP kết nối workspace

Toàn bộ tài liệu lưu ở: C:\Github\tlxn-ai-agent\
- 00-INDEX.md — mục lục
- 01-plan-v3.md — plan triển khai agent
- 02-hbss-review.md — review HBSS hiện tại
- 03-data-model-v2.md — data model v2 chi tiết
- 04-mcp-setup.md — config MCP Lark + credentials

Hãy đọc toàn bộ 5 file trên trước khi trả lời tôi. Đặc biệt 03-data-model-v2.md vì đó là blueprint.

Lark MCP đã hoạt động. App token của file HBSS mẫu là: Ju2dbkBEzaPX6KskHQSlnI3Mgvb

Chiến lược tôi đã chốt:
- Moat = vận hành mượt nhờ AI (không bán data)
- Data nội bộ (không share ra ngoài), nhưng dùng làm training data cho AI chuyên ngành
- Team no-code (không ai biết SQL)
- Dashboard khách hàng đã có trong HBSS
- 1 khách = 1 dự án (nếu > 1 thì tạo file riêng)
- Payment sẽ move từ bộ phận tài chính vào HBSS
- Track tên nhân viên cụ thể (không chỉ Vai trò)
- Scorecard Đơn vị/Đối tác đo MONTHLY để đo hài lòng khách
- Nhiều loại hợp đồng/dự án (TLXN, thiết kế, thi công, NTP...)

Bước tiếp theo tôi muốn đi là: [CHỌN 1 TRONG 4 OPTIONS BÊN DƯỚI]
```

---

## 🎯 4 OPTIONS cho bước tiếp (Nam chọn 1)

### [A] Vẽ ERD chi tiết
Entity-Relationship Diagram cho Warehouse schema. Text-based (mermaid hoặc ASCII).
→ Phù hợp khi muốn **hình dung tổng thể** trước khi vào chi tiết.

### [B] Detailed spec 4 table critical
Deep-dive field-by-field cho: `Thông tin Dự án`, `Theo dõi Thanh toán`, `Quản lý Hợp đồng`, `Chấm điểm đối tác`.
Bao gồm: constraint, index, default value, validation rule, ví dụ data.
→ Phù hợp khi muốn **bắt tay build ngay**.

### [C] Kế hoạch setup Master Base trên Lark
Step-by-step bấm gì trên Lark UI để tạo Master Base + 8 table master + seed data.
Có screenshot guide nếu tôi cần.
→ Phù hợp khi muốn **làm ngay không cần thêm decision**.

### [D] Chuyển sang Phase 2 (Sales)
Đồng bộ data model cho cả Sales → QLDA. Review CRM hiện tại nếu có, thiết kế SKU, validation gate 25 mục bàn giao.
→ Phù hợp khi muốn **có bức tranh tổng** trước khi build.

---

## ⚠️ Những điều cần nhớ cho session mới

1. **MCP Lark đang dùng credential lộ chat** — cần rotate trước Phase 1 production
2. **App Secret**: lưu ở local `.env` (gitignored). Cần rotate vì đã paste vào chat transcript.
3. **App ID**: `cli_a959369f73f8deea` (app đứng tên Nam, scope readonly đủ)
4. **File template HBSS**: `Ju2dbkBEzaPX6KskHQSlnI3Mgvb`
5. Nếu chuyển máy / Claude Desktop mới → copy `04-mcp-setup.md` để setup lại

---

## 🧠 Các câu hỏi chưa trả lời từ Plan v3 (có thể cần trong session mới)

1. Thứ tự Phase: tuần tự (A) hay song song sub-module (B)?
2. Ngân sách LLM/tháng?
3. Judge trong Eval Framework: Nam / QLDA lead / auto-LLM?
4. Kill switch trigger: tay hay auto-rule?
5. Transparency với khách: công khai "chat AI" hay ngầm?
6. DR target: RPO + RTO?
7. Vault chọn gì?

Nam có thể trả lời dần trong sessions tiếp theo, không cần chốt ngay bây giờ.
