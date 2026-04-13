---
name: api-conventions
description: 寫 API endpoint 時自動套用團隊規範。在寫 RESTful API、Express route、或任何後端 endpoint 時使用。
---

這個專案的 API 規範，寫任何 endpoint 時都要遵守：

## 1. URL 命名規則

用 **kebab-case**（全小寫，連字號分隔）。動詞用 HTTP method 表達，不要放在 URL 裡。

- ✅ 正確：`GET /api/user-profiles`
- ❌ 錯誤：`GET /api/getUserProfiles`
- ✅ 正確：`DELETE /api/orders/:id`
- ❌ 錯誤：`POST /api/deleteOrder`

## 2. 統一回傳格式

所有 endpoint 回傳以下結構：

```json
{
  "data": null,
  "error": null,
  "meta": {}
}
```

- **成功**：`data` 放資料，`error` 為 `null`
- **失敗**：`data` 為 `null`，`error` 放錯誤訊息字串
- **分頁**：`meta` 放 `{ "page": 1, "pageSize": 20, "total": 100 }`

## 3. HTTP Status Code

用標準 HTTP status code，不要自己發明 error code：

| Code | 情境 |
|------|------|
| `200` | 成功 |
| `201` | 新增成功 |
| `204` | 刪除成功（無回傳內容）|
| `400` | Bad Request：參數格式錯誤或缺少必填欄位 |
| `401` | Unauthorized：未登入或 Token 無效 |
| `403` | Forbidden：無權限 |
| `404` | Not Found：資源不存在 |
| `409` | Conflict：資料衝突（如重複 email）|
| `422` | Unprocessable Entity：格式正確但業務邏輯驗證失敗 |
| `500` | Internal Server Error：後端錯誤 |

## 4. Input Validation

所有 endpoint 都要有 input validation，用 **zod** 或 **joi**：

```typescript
// zod 範例
const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(1).max(100),
})
```

validation 失敗回傳 `400`，`error` 欄位放驗證錯誤訊息。

## 5. RESTful 動詞對應

| 操作 | Method | 路徑 |
|------|--------|------|
| 取得列表 | `GET` | `/api/resources` |
| 取得單筆 | `GET` | `/api/resources/:id` |
| 新增 | `POST` | `/api/resources` |
| 完整更新 | `PUT` | `/api/resources/:id` |
| 部分更新 | `PATCH` | `/api/resources/:id` |
| 刪除 | `DELETE` | `/api/resources/:id` |

## 6. 其他規範

- **版本前綴**：統一加 `/api/v1/`（若專案有版本控管）
- **分頁參數**：用 `?page=1&pageSize=20`，不用 `offset/limit`
- **時間格式**：統一用 ISO 8601（`2026-04-13T10:00:00Z`）
- **ID 格式**：統一用 UUID 或數字，不混用
