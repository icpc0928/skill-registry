# API 文件生成器 — Skill

> **用途**：掃描後端專案程式碼，自動生成完整的 API 文件。
> **使用方式**：把這份文件 + 專案路徑交給 AI，自動產出 `API_DOCS.md`。

---

## 指令

你是一位資深後端工程師。請掃描指定的後端專案，生成完整的 API 文件。

### Step 1：偵測專案框架

掃描專案目錄，判斷使用的框架：

| 框架 | 偵測方式 | Controller 路徑 |
|------|---------|----------------|
| **Spring Boot (Java)** | `build.gradle` 或 `pom.xml` 含 `spring-boot` | `**/controller/*.java` |
| **Express (Node.js)** | `package.json` 含 `express` | `**/routes/*.js` 或 `**/controllers/*.js` |
| **Django (Python)** | `manage.py` 存在 | `**/views.py` + `**/urls.py` |
| **FastAPI (Python)** | `requirements.txt` 含 `fastapi` | `**/main.py` 或 `**/routers/*.py` |
| **Gin (Go)** | `go.mod` 含 `gin-gonic` | `**/handlers/*.go` 或 `**/routes/*.go` |
| **Laravel (PHP)** | `artisan` 存在 | `**/Controllers/*.php` + `routes/*.php` |
| **NestJS (Node.js)** | `package.json` 含 `@nestjs/core` | `**/*.controller.ts` |

### Step 2：掃描 API Endpoint

根據框架類型，掃描所有 API endpoint：

#### Spring Boot
```bash
# 找所有 Controller
find . -name "*Controller.java" -path "*/controller/*"

# 掃描 annotation
grep -n "@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping\|@PatchMapping\|@RequestMapping" {file}
```

#### Express
```bash
# 找所有路由定義
grep -rn "router\.\(get\|post\|put\|delete\|patch\)" routes/
```

#### Django
```bash
# 找 URL patterns
grep -rn "path(" urls.py
```

#### FastAPI
```bash
# 找所有路由
grep -rn "@app\.\(get\|post\|put\|delete\|patch\)\|@router\.\(get\|post\|put\|delete\|patch\)" .
```

### Step 3：分析每個 Endpoint

對每個 endpoint，提取：

| 資訊 | 來源 |
|------|------|
| **HTTP Method** | Annotation / 路由定義 |
| **路徑** | Annotation 參數 |
| **說明** | 方法名稱 / 註解 / JavaDoc |
| **Request Body** | DTO class / 參數型別 |
| **Response** | 回傳型別 / ResponseEntity |
| **路徑參數** | `@PathVariable` / `:param` |
| **查詢參數** | `@RequestParam` / `req.query` |
| **權限** | SecurityConfig / middleware / decorator |

### Step 4：分析權限設定

掃描安全設定檔，判斷每個 endpoint 的存取權限：

| 框架 | 安全設定檔 |
|------|-----------|
| Spring Boot | `SecurityConfig.java` → `requestMatchers().permitAll()` / `.hasRole()` |
| Express | middleware（`auth`, `isAdmin`）|
| Django | `@permission_classes` / `IsAuthenticated` |
| FastAPI | `Depends(get_current_user)` |
| NestJS | `@UseGuards()` / `@Roles()` |

分為三層：
- **Public** — 不需要登入
- **User** — 需要用戶登入（Bearer Token）
- **Admin** — 需要管理員權限

### Step 5：分析 Request/Response 格式

讀取 DTO / Model / Schema，提取欄位：

```
欄位名稱 | 型別 | 必填 | 說明 | 驗證規則
```

注意提取：
- `@NotBlank`, `@NotNull`, `@Email`, `@Size` (Java)
- `required`, `type`, `minlength` (JS)
- `Field(...)`, `validator` (Python)

### Step 6：生成文件

使用 `templates/API_DOCS.md` 的格式，生成完整文件。

輸出到專案根目錄：`{project}/API_DOCS.md`

---

## 輸出規範

### 排列順序

1. **概覽**（專案名、技術棧、Base URL）
2. **認證說明**（登入方式、Token 格式）
3. **API 列表**（按模組分組）
4. **每個 API 的詳細說明**
5. **錯誤碼對照表**
6. **資料模型**

### API 詳細說明格式

每個 endpoint 需包含：

```markdown
### {說明}

`{METHOD} {PATH}`

**權限**：Public / User / Admin

**路徑參數**：
| 參數 | 型別 | 說明 |
|------|------|------|

**查詢參數**：
| 參數 | 型別 | 必填 | 預設值 | 說明 |
|------|------|------|--------|------|

**Request Body**：
| 欄位 | 型別 | 必填 | 驗證規則 | 說明 |
|------|------|------|---------|------|

**Response**：
```json
{
  "example": "response"
}
```

**狀態碼**：
| 狀態碼 | 說明 |
|--------|------|
| 200 | 成功 |
| 400 | 參數錯誤 |
| 401 | 未認證 |
```

---

## 進階：自動產生 curl 範例

對每個 endpoint 產生可直接執行的 curl 範例：

```markdown
**curl 範例**：
```bash
curl -s {BASE_URL}/api/products?page=0&size=10
```
```

- Public API → 不帶 Token
- User API → 帶 `Authorization: Bearer {USER_TOKEN}`
- Admin API → 帶 `Authorization: Bearer {ADMIN_TOKEN}`

---

## 品質檢查

生成完成後確認：

- [ ] 每個 Controller / 路由檔案的所有 endpoint 都有列出
- [ ] 權限分層（Public / User / Admin）正確
- [ ] Request Body 的必填欄位和驗證規則完整
- [ ] Response 格式有範例
- [ ] curl 範例可直接執行（URL 用變數佔位）
- [ ] 沒有暴露敏感資訊（密碼、API Key 等）

---

## 使用方式

```
請讀取 api-docs-generator/SKILL.md，
然後掃描我的專案 {專案路徑}，生成完整的 API 文件。
Base URL: {API 的 URL}
```

AI 會自動偵測框架 → 掃描所有 endpoint → 分析權限和參數 → 生成 `API_DOCS.md`。
