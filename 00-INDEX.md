# TLXN AI Agent — Project Archive

**Mục đích**: Triển khai AI Agent cho QLDA + Sales/NCM của TLXN (Trợ Lý Xây Nhà).
**Owner**: Nam
**Last updated**: 2026-04-18
**Session reference**: Claude Desktop, initial planning session

---

## Chiến lược tổng

- **Moat**: Vận hành mượt nhờ AI (không phải bán data)
- **Scale target**: 2026: 100 dự án → 2030: 1000 dự án/năm
- **Secondary moat**: Dataset xây dựng dân dụng VN duy nhất (training data + thought leadership)
- **Market**: Thuần nội bộ + franchise về sau (Đà Nẵng, Hà Nội)

---

## Files trong folder này

| File | Nội dung | Dùng khi nào |
|---|---|---|
| `00-INDEX.md` | File này — mục lục + status | Mở trước |
| `01-plan-v3.md` | AI Agent implementation plan v3 final (Phase 0-3, 18 task nền tảng) | Tham chiếu roadmap triển khai |
| `02-hbss-review.md` | Review schema HBSS hiện tại, 10 anti-patterns, findings | Hiểu vấn đề data hiện tại |
| `03-data-model-v2.md` | Data Model v2: Master Base + Template v2 + Warehouse schema | Blueprint để build |
| `04-mcp-setup.md` | Lark MCP config, credentials, troubleshooting | Khi cần config lại MCP |
| `05-handoff.md` | Context compact để paste vào session Claude mới | Mở session mới với Claude |
| `06-master-base-setup.md` | Step-by-step build Layer 1 Master Base trên Lark (7 table) | Khi đi thực thi setup Base |
| `07-master-base-handoff.md` | Handoff sau Đợt 1 execute qua MCP: Base URL, 7 table IDs, việc Nam cần làm Đợt 2 | Sau khi Claude execute qua MCP |
| `08-dot-2-huong-dan-tung-buoc.md` | Hướng dẫn Đợt 2 click-by-click cho Nam (no-tech) | Khi Nam ngồi làm Master Base trên Lark |

---

## Status hiện tại (2026-04-18)

### ✅ Đã làm xong
- Plan v3 triển khai agent (qua 3 vòng review: AI + reviewer kỹ thuật + Nam)
- Review schema 8 table HBSS hiện tại → tìm 10 anti-patterns
- Setup Lark MCP kết nối workspace trolyxaynha029
- Thiết kế Data Model v2 (Master Base + Template v2 + Warehouse)

### 🟡 Đang chờ quyết định
1. Model pricing decision — Nam chốt budget LLM/tháng
2. Eval Framework judge — ai làm judge (Nam, QLDA lead, auto-LLM)
3. Transparency với khách — công khai "chat AI" hay ngầm
4. DR target — RPO/RTO
5. Vault chọn gì (1Password / Vault / AWS Secrets / local .env)

### ⏭️ Roadmap tiếp theo (2026-04-18 chốt)
- **[C] Đợt 1** ✅ Execute qua MCP xong: Base + 7 tables + 74 records seeded (xem `07-master-base-handoff.md`)
- **[C] Đợt 2** 🟡 Đang chờ Nam: move Base, thêm Formula/Lookup, cung cấp staff+vendor list
- **[D]** Phase 2 Sales design — đi sau khi Đợt 2 xong
- **[A]** ERD JIT khi build Layer 2 Template
- **[B]** Detailed spec JIT khi build từng table ở Layer 2

---

## ⚠️ Security notes

- App Secret Lark **cli_a959369f73f8deea** đã được paste vào chat (transcript)
- Cần **rotate** trước khi đi vào Phase 1 production
- Xem chi tiết ở `04-mcp-setup.md`
