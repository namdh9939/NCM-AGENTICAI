# Lark MCP Setup — TLXN

**Status**: ✅ Working (verified 2026-04-18)

---

## Credentials hiện tại

| Field | Value |
|---|---|
| App ID | `cli_a959369f73f8deea` |
| App Secret | `<REDACTED — see .env locally>` |
| Domain | `https://open.larksuite.com` |
| Workspace | trolyxaynha029.sg.larksuite.com |

**App cũ đã abandon**: `cli_a9593be1a0f8ded1` (Nam không có quyền manage → tạo app mới `cli_a959369f73f8deea` đứng tên Nam)

### ⚠️ SECURITY — PHẢI LÀM

App Secret đã paste vào chat transcript + ghi vào settings.json. **Rotate trước khi Phase 1 production**:
1. Vào [Lark Developer Console](https://open.larksuite.com/app/cli_a959369f73f8deea)
2. Credentials & Basic Info → Regenerate App Secret
3. Update vào `claude_desktop_config.json`
4. Restart Claude Desktop

---

## Scopes đã enable

- ✅ `bitable:app:readonly` — đọc Base (HBSS)
- ✅ `bitable:app` — full access
- ✅ `base:table:read`

**Scopes có sẵn nhưng chưa cần**:
- `docx:document:readonly`
- `drive:drive:readonly`
- `contact:user.base:readonly`
- `im:*` (chat messaging)
- `wiki:*`

→ Khi build Phase 1 agent thật sự, có thể cần bật thêm `bitable:app` (write) cho agent ghi dữ liệu. Tạo app mới riêng cho write thay vì shared.

---

## Config file location

**Đúng file**: `C:\Users\namdh\AppData\Roaming\Claude\claude_desktop_config.json`

(Đây là file Claude Desktop đọc, KHÔNG phải `~/.claude/settings.json` của Claude Code CLI)

Content hiện tại:

```json
{
  "preferences": {
    "coworkScheduledTasksEnabled": false,
    "ccdScheduledTasksEnabled": true,
    "sidebarMode": "epitaxy",
    "coworkWebSearchEnabled": true
  },
  "mcpServers": {
    "lark": {
      "command": "C:\\Program Files\\nodejs\\node.exe",
      "args": [
        "C:\\Users\\namdh\\AppData\\Roaming\\npm\\node_modules\\@larksuiteoapi\\lark-mcp\\dist\\cli.js",
        "mcp",
        "-a", "cli_a959369f73f8deea",
        "-s", "<APP_SECRET_HERE>",
        "-d", "https://open.larksuite.com",
        "-l", "en",
        "-m", "stdio"
      ]
    }
  }
}
```

---

## Dependencies đã cài

- Node.js v24.15.0 (`C:\Program Files\nodejs\`)
- `@larksuiteoapi/lark-mcp` v0.5.1 (installed globally)
  - Path: `C:\Users\namdh\AppData\Roaming\npm\node_modules\@larksuiteoapi\lark-mcp\`

---

## Tools khả dụng (19 total)

| Category | Tools |
|---|---|
| Bitable (Base) | `bitable_v1_app_create`, `bitable_v1_appTable_list`, `bitable_v1_appTable_create`, `bitable_v1_appTableField_list`, `bitable_v1_appTableRecord_search`, `bitable_v1_appTableRecord_create`, `bitable_v1_appTableRecord_update` |
| Docx | `docx_builtin_import`, `docx_builtin_search`, `docx_v1_document_rawContent` |
| Drive | `drive_v1_permissionMember_create` |
| IM (Chat) | `im_v1_chat_create`, `im_v1_chat_list`, `im_v1_chatMembers_get`, `im_v1_message_create`, `im_v1_message_list` |
| Wiki | `wiki_v1_node_search`, `wiki_v2_space_getNode` |
| Contact | `contact_v3_user_batchGetId` |

---

## File HBSS reference

**App Token** (của file HBSS template đang dùng làm template cho mọi dự án):
```
Ju2dbkBEzaPX6KskHQSlnI3Mgvb
```

URL: https://trolyxaynha029.sg.larksuite.com/base/Ju2dbkBEzaPX6KskHQSlnI3Mgvb

8 tables (xem chi tiết ở `02-hbss-review.md`):
| Table | Table ID |
|---|---|
| 📆 Kế Hoạch & Thông Tin | `tbl5NinGKRDjx0Xr` |
| ✅ Việc Thiết Kế & Thi Công | `tblxTcF0oni93SVt` |
| ⚡ Ticket & Đánh Giá Tuần | `tblsCNqDYZLIeVvz` |
| 💵 Báo Giá & Thanh Toán | `tbltD31qVCD2z50h` |
| 🗂 Hồ Sơ & Pháp Lý | `tblM8Yd9hjLyk3ha` |
| 📸 Nhật Ký Dự Án | `tbl7Az3O2Wk9danj` |
| 🎯 Trợ Lý Báo Cáo | `tblb0C10WJPpuyeK` |
| 🗄 Thiết Lập | `tblnGieIhehZyL2r` |

---

## Troubleshooting

### MCP tools không xuất hiện sau khi update config
→ Phải **quit Claude Desktop hoàn toàn** (right-click tray icon → Quit), sau đó mở lại. Reload chat không đủ.

### Error `99991672 Access denied`
→ App thiếu scope hoặc version chưa publish:
1. Vào Developer Console → Permissions → bật scope → Save
2. Version Management → Create Version → Submit
3. (Internal app) Admin Console → Approve

### Không vào được Developer Console (403)
→ Tài khoản login không phải owner app. Hoặc dùng tài khoản đã tạo app, hoặc tạo app mới đứng tên mình.

### MCP pick credential cũ sau khi update config
→ MCP server spawn lúc Claude Desktop khởi động. Đổi config không hot-reload. **Phải restart Claude Desktop**.

### Fallback nếu MCP fail
Có thể gọi Lark API trực tiếp bằng curl:

```bash
# Get tenant access token
curl -X POST "https://open.larksuite.com/open-apis/auth/v3/tenant_access_token/internal" \
  -H "Content-Type: application/json" \
  -d '{"app_id":"cli_a959369f73f8deea","app_secret":"<APP_SECRET_HERE>"}'

# List tables
curl "https://open.larksuite.com/open-apis/bitable/v1/apps/Ju2dbkBEzaPX6KskHQSlnI3Mgvb/tables" \
  -H "Authorization: Bearer <TOKEN>"
```
