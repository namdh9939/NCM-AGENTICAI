# TLXN AI Agent

**Internal planning + data model + execution archive** for the TLXN (Trợ Lý Xây Nhà) AI Agent initiative.

**Owner**: Nam
**Scope**: Quản lý vận hành QLDA + Sales nhờ AI. Scale target 100 → 1000 dự án/năm (2026-2030).

> ⚠️ **Private repo — internal only.** Contains business architecture, schema design, and operational docs. Do not share externally.

---

## Getting started on a new machine

```bash
git clone <this-repo-url> C:\Github\tlxn-ai-agent
cd C:\Github\tlxn-ai-agent
cp .env.example .env
# Paste actual LARK_APP_SECRET into .env (get from Lark Developer Console or 1Password)
```

Then configure Lark MCP in Claude Desktop — see [`04-mcp-setup.md`](./04-mcp-setup.md).

---

## Files

| File | Purpose |
|---|---|
| [`00-INDEX.md`](./00-INDEX.md) | Master index + current Trạng thái + next steps |
| [`01-plan-v3.md`](./01-plan-v3.md) | AI Agent implementation plan v3 (Phase 0-3, 18 foundational Checklist công việc chi tiết) |
| [`02-hbss-review.md`](./02-hbss-review.md) | Review existing HBSS schema + 10 anti-patterns found |
| [`03-data-model-v2.md`](./03-data-model-v2.md) | Data Model v2 blueprint (Master + Template + Warehouse) |
| [`04-mcp-setup.md`](./04-mcp-setup.md) | Lark MCP config + troubleshooting |
| [`05-handoff.md`](./05-handoff.md) | Context prompt to paste into a new Claude session |
| [`06-master-base-setup.md`](./06-master-base-setup.md) | Step-by-step Lark UI guide for Master Base (Layer 1) |
| [`07-master-base-handoff.md`](./07-master-base-handoff.md) | Handoff after Đợt 1 MCP execution — what's done, what Nam needs to do in Đợt 2 |

---

## Current Trạng thái (2026-04-18)

✅ **Planning**: v3 final approved (3 review rounds)
✅ **HBSS review**: 8 tables analyzed, 10 anti-patterns identified
✅ **Data Model v2**: Master Base (8 tables) + Template v2 (17 tables) + Warehouse (28 entities) designed
✅ **Lark MCP**: connected, verified working
✅ **Master Base Đợt 1**: Base + 7 tables created, 74 records seeded via MCP

🟡 **Master Base Đợt 2** (in-progress — Nam's turn):
- Move Base to shared folder
- Add Formula / Lookup fields via UI
- Provide Nhân sự list + Đơn vị/Đối tác dedupe → seed via MCP
- (Danh mục khách hàng schema-only — defer seed until Phase 2 Sales design)

⏭️ **Next**: Phase 2 Sales design (D) after Đợt 2 complete.

---

## Security

- App Secret for Lark was paste into chat transcript during initial setup → **must rotate** before Phase 1 production (see 04-mcp-setup.md)
- All secrets go into `.env` (gitignored). Never commit.
- Repo must stay **private**.

---

## Conventions

- Language: Vietnamese primary, English for technical terms
- Each doc file is self-contained — can be read standalone
- Changes: commit per logical unit, not per file. Message in English.
