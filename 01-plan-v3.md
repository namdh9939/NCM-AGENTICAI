# AI Agent Implementation Plan v3 — TLXN

**Version**: v3 FINAL
**Date**: 2026-04-18
**Review rounds**: 3 (AI analysis + technical reviewer + Nam technical feedback)

---

## 0. NGUYÊN TẮC NỀN TẢNG

1. **Thiết kế cho tương lai ngay cả khi chỉ có 1 tenant** — `tenant_id` từ ngày 1, tránh refactor khi mở franchise
2. **Agent không quyết định dựa trên cache** — dữ liệu decision-critical luôn read-through từ source of truth
3. **Hard constraint vi phạm 1 lần = rollback** — không có chuyện "95% không leak là ok"
4. **RAG không phải là nguồn duy nhất cho quyết định có hậu quả ra ngoài** — hybrid: hardcode rule + RAG + LLM
5. **Kill switch phải có từ agent đầu tiên**, không đợi Phase 3
6. **Validation Gate 2 chiều ở MỌI handover giữa agent** — không chỉ Sales→QLDA
7. **Mọi action ra ngoài phải idempotent** — retry không tạo 2 tin, 2 record trùng
8. **Test 3 lớp trước khi chạm khách hàng thật**: offline eval → shadow mode → pilot

---

## PHASE 0 — NỀN TẢNG CHUNG (18 task, bắt buộc xong trước Phase 1)

### 0.A — Kiến trúc & Dữ liệu

| # | Việc | Output |
|---|---|---|
| 0.1 | Kiến trúc tổng thể: Orchestrator + Sub-agents + **Message protocol chuẩn giữa agent (A2A Hợp đồng liên quan)** | Sơ đồ hệ thống + message schema |
| 0.2 | Tech stack: n8n + Lark + Zalo + model provider | Tech stack confirmed |
| 0.3 | **Schema dùng chung** — bắt buộc có `tenant_id`, `version`, `updated_at`, `audit_by` từ ngày 1 | Data schema v1 multi-tenant |
| 0.4 | Nguyên tắc SOP agent-ready (IF/THEN, trigger, decision point, SLA) | SOP template chuẩn |
| 0.5 | Escalation matrix toàn công ty (ai → ai, điều kiện nào) | Escalation matrix |

### 0.B — An toàn & Kiểm soát

| # | Việc | Output |
|---|---|---|
| 0.6 | **Access Control & Security**: RBAC cho từng agent, API Vai trò n8n → Lark/HBSS, **content-level guard chống prompt injection** (system prompt isolation, output filter, PII redactor) | Security policy v1 |
| 0.7 | **Audit Trail & Logging**: trace mọi quyết định (input → prompt → retrieval → output → action), cost + latency log | Observability stack |
| 0.8 | **HITL Policy matrix**: việc nào auto / việc nào approve. Threshold cho tin ra ngoài, tài chính, sửa data khách. Tách hard constraint vs soft metric | HITL policy v1 |
| 0.12 | **Kill Switch 3 tầng**: tắt 1 agent / 1 nhóm / toàn bộ. Định nghĩa fallback human = ai, SLA nhận việc bao lâu | Kill switch spec |
| 0.13 | **Transparency & Legal**: khách có biết đang chat AI, điều khoản AI trong hợp đồng TLXN, policy retention PII | Disclosure + compliance v1 |

### 0.C — Chất lượng & Đo lường

| # | Việc | Output |
|---|---|---|
| 0.9 | **Eval Framework**: golden test set từ lịch sử, offline eval trước khi chạm khách. Judge là ai | Eval harness + golden set v1 |
| 0.10 | **Baseline KPI (before state)**: đo hiện trạng — thời gian xử lý lead, % lead rơi, thời gian nghiệm thu chậm, số lỗi sót/tháng | Baseline report |
| 0.11 | **Cost model & Model routing**: ngân sách LLM/tháng, routing Haiku → Sonnet → Opus theo độ khó task. Budget ceiling/agent/ngày | Cost policy |

### 0.D — Tầng kỹ thuật nền

| # | Việc | Output |
|---|---|---|
| 0.14 | **Resilience & Queue Architecture**: retry + exponential backoff + jitter, circuit breaker per upstream, idempotency key, DLQ + alert, rate limit nội bộ, async queue trước n8n | Resilience spec |
| 0.15 | **State Sync & Source-of-Truth policy**: webhook + reconciliation job 15 phút, hierarchy data (decision-critical = read-through, enrichment = cache, conversation = session), optimistic lock `version`, HMAC verify webhook | State sync spec |
| 0.16 | **Disaster Recovery & Backup**: backup HBSS + CRM + audit log + agent memory (RPO/RTO định lượng), playbook khôi phục, backup test định kỳ | DR plan |
| 0.17 | **Secrets Management**: vault cho token, rotation schedule, audit truy cập, alert khi leak. KHÔNG commit secret vào Git | Secrets policy |
| 0.18 | **Agent Registry & Onboarding**: đăng ký agent mới, canary deploy (5% → 50% → 100%), rollback cơ chế, compat test | Agent lifecycle spec |

### Thứ tự triển khai Phase 0 đề xuất

**Đợt 1 (tuần 1–2) — bắt buộc trước khi code agent đầu tiên:**
`0.1 → 0.3 → 0.6 → 0.7 → 0.14 → 0.15 → 0.17`

**Đợt 2 (tuần 3–4) — policy & chất lượng, song song cuối đợt 1:**
`0.4 → 0.5 → 0.8 → 0.9 → 0.10 → 0.11 → 0.12`

**Đợt 3 (tuần 5+) — scale readiness:**
`0.2 final → 0.13 → 0.16 → 0.18`

---

## PHASE 1 — BỘ PHẬN QLDA

### 1A — Mapping hiện trạng
| # | Việc | Output |
|---|---|---|
| 1A.1 | Event list toàn bộ vòng đời dự án TLXN | Event list |
| 1A.2 | Mỗi event: ai làm, data đâu, auto/manual | As-is map |
| 1A.3 | Điểm rơi / rủi ro hiện tại | Gap list |
| 1A.4 | **Đo baseline KPI cho từng event** (liên kết 0.10) | QLDA baseline |

### 1B — Thiết kế
| # | Việc | Output |
|---|---|---|
| 1B.1 | **Schema HBSS** đủ cho agent đọc/ghi **+ webhook outbound + version field + audit field** | HBSS schema v2 (xem 03-data-model-v2.md) |
| 1B.2 | SOP agent-ready cho toàn bộ event QLDA | SOP QLDA hoàn chỉnh |
| 1B.3 | Agent backlog + ưu tiên | Agent backlog QLDA |
| 1B.4 | **Validation Gate**: mỗi handover có Hợp đồng liên quan 2 chiều, log reject vào Audit Trail | Gate spec |
| 1B.5 | **Knowledge hybrid**: Decision critical → hardcode rule; Tra cứu mềm → RAG từ SOP markdown; Soạn tin/tóm tắt → LLM+few-shot | Knowledge architecture |
| 1B.6 | **Prompt & SOP version control**: version tag, rollback cơ chế | Versioning spec |

### 1C — Build & Test
| # | Việc | Output |
|---|---|---|
| 1C.1 | Build từng agent. **Kill switch + Audit Trail + Idempotency + Read-through từ agent đầu** | Agent chạy được |
| 1C.2 | **Test 3 lớp**: (a) Offline eval golden set → pass mới qua (b) Shadow mode (song song người, không gửi ra) → (c) Pilot 1–3 dự án thật khi shadow đạt ngưỡng | Test report 3 lớp |
| 1C.3 | **Go/No-Go**: Hard constraint (vi phạm 1 lần = dừng) vs Soft metric (gắn baseline 1A.4) | Go-live criteria |
| 1C.4 | Điều chỉnh sau test | Version ổn định |
| 1C.5 | Rollout toàn dự án | Go-live QLDA |
| 1C.6 | **Change management**: training PM/CA/QA, SOP vận hành agent, xử lý bypass | Training + adoption dashboard |

---

## PHASE 2 — BỘ PHẬN SALES

**Thứ tự khuyến nghị**: Option B — sub-module Phase 1 + Phase 2 song song (quick win, validate framework Phase 0 sớm).

### 2A — Mapping
| # | Việc | Output |
|---|---|---|
| 2A.1 | Event list lead journey | Event list Sales |
| 2A.2 | Mỗi event: ai, data, auto/manual | As-is map |
| 2A.3 | Điểm rơi lead | Gap list |
| 2A.4 | **Baseline KPI Sales**: time-to-first-touch, conversion rate, % lead rơi | Sales baseline |

### 2B — Thiết kế
| # | Việc | Output |
|---|---|---|
| 2B.1 | **Schema CRM** + webhook + version + audit | CRM schema |
| 2B.2 | SKU definition: scope, điều kiện chốt, downgrade flow | SKU hoàn chỉnh |
| 2B.3 | SOP agent-ready cho hành trình Sales | SOP Sales |
| 2B.4 | Agent backlog Sales | Agent backlog |
| 2B.5 | **Validation Gate Sales → QLDA**: 25 mục kiểm đủ (HĐ, CCCD, ngân sách…) | Gate spec |
| 2B.6 | **Prompt injection guard mạnh** cho Zalo/FB chat (tiếp xúc khách trực tiếp) | Guard spec |
| 2B.7 | Knowledge hybrid Sales | Knowledge Sales |

### 2C — Build & Test
Mirror 1C với data Sales.

---

## PHASE 3 — TÍCH HỢP & SCALE

| # | Việc | Output |
|---|---|---|
| 3.1 | Orchestrator chung (CEO Agent) — message protocol đã chuẩn từ 0.1 nên chỉ là kết nối | CEO Agent hoạt động |
| 3.2 | Master dashboard + live Audit Trail + cost dashboard + DLQ monitor | Master dashboard |
| 3.3 | **Franchise activation**: `tenant_id` đã có từ 0.3, chỉ là kích hoạt ĐN/HN | Franchise system |
| 3.4 | **Continuous eval + drift detection**: golden set mở rộng, alert khi accuracy tụt | Monitoring |
| 3.5 | **Incident playbook**: agent spam/sai — ai trigger kill switch, ai báo khách, post-mortem template | Playbook |
| 3.6 | **Quarterly audit**: access control review, secrets rotation, PII leak test, red team prompt injection | Audit checklist |

---

## PHỤ LỤC A — CÂU HỎI NAM CẦN CHỐT

1. Thứ tự: tuần tự (Option A) hay song song sub-module (Option B)?
2. Ngân sách LLM/tháng?
3. Judge trong Eval Framework: Nam / QLDA lead / auto-LLM?
4. Kill switch trigger: Nam tự bấm, hay auto-rule (> 10 tin/phút = auto-kill)?
5. Transparency với khách: công khai "chat AI" hay ngầm?
6. DR target: RPO (mất data tối đa bao nhiêu lâu) + RTO (time to recover)?
7. Vault chọn gì: 1Password / HashiCorp Vault / AWS Secrets / local .env?

---

## PHỤ LỤC B — GLOSSARY

| Thuật ngữ | Ý nghĩa |
|---|---|
| Validation Gate | Cổng 2 chiều giữa 2 agent. Bên gửi xác nhận đủ, bên nhận xác nhận accept. Log reject |
| Kill Switch 3 tầng | Tắt 1 agent / 1 nhóm / toàn hệ. Kèm định nghĩa fallback human |
| Hybrid Knowledge | Hardcode rule (decision critical) + RAG (tra cứu) + LLM (tóm tắt/soạn) |
| Hard Constraint | Ràng buộc tuyệt đối, vi phạm 1 lần = rollback. Vd: không leak PII |
| Soft Metric | Mục tiêu định lượng có ngưỡng chấp nhận (vd: accuracy ≥ 90%) |
| Test 3 lớp | Offline eval (golden set) → Shadow mode → Pilot |
| Read-through | Đọc từ source, không cache. Dùng cho mọi quyết định có hậu quả |
| Idempotency key | Key duy nhất mỗi action ra ngoài. Retry không tạo trùng |
| Circuit Breaker per upstream | Mỗi service (Zalo/Lark/LLM) có breaker riêng |
| Reconciliation Job | Job định kỳ scan diff source vs agent state. Bắt webhook miss |
| Optimistic Lock | Dùng `version` field để detect race giữa agent và human writer |
| Canary Deploy | Agent mới: 5% → 50% → 100% traffic. Rollback nhanh |
| Shadow Mode | Agent chạy song song người, output KHÔNG gửi ra — chỉ so sánh |
| DLQ | Queue chứa request fail sau N retry. Cần owner + SLA |
| HMAC verify | Ký số webhook để chống giả mạo |
