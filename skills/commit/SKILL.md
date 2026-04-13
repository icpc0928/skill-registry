# Git Commit 生成器 — Skill

> **用途**：分析目前所有 staged/unstaged 修改，自動生成符合 Conventional Commit 格式的 commit message，並執行 commit。
> **使用方式**：在任何有 git 的專案中交給 AI 執行，AI 會分析 diff 後決定最合適的 commit message。

---

## 指令

你是一位資深工程師。請分析目前專案的 git 修改，生成符合規範的 commit message 並執行 commit。

---

## Step 1：分析修改內容

依序執行以下指令，全面了解改動範圍：

```bash
# 查看 staged 和 unstaged 的所有修改
git diff

# 查看已 staged 的修改
git diff --staged

# 查看修改狀態總覽
git status
```

閱讀 diff 時，重點關注：
- **新增了什麼功能？** → `feat:`
- **修復了什麼 Bug？** → `fix:`
- **重構或改善了什麼？** → `refactor:` / `perf:`
- **改了設定或文件？** → `chore:` / `docs:`
- **改了測試？** → `test:`
- **改了 CI/CD？** → `ci:`

---

## Step 2：決定 Commit Type

| Type | 使用情境 | 範例 |
|------|---------|------|
| `feat` | 新功能 | `feat: 新增用戶登入功能` |
| `fix` | 修復 Bug | `fix: 修正金額計算錯誤` |
| `refactor` | 重構（不影響行為）| `refactor: 拆分 UserService 邏輯` |
| `perf` | 效能改善 | `perf: 優化查詢使用索引` |
| `docs` | 文件更新 | `docs: 更新 API 說明` |
| `test` | 測試相關 | `test: 新增登入單元測試` |
| `chore` | 雜務（依賴、設定）| `chore: 升級 Spring Boot 3.2` |
| `ci` | CI/CD 設定 | `ci: 新增 GitHub Actions 流程` |
| `style` | 格式調整（不影響邏輯）| `style: 統一縮排格式` |
| `revert` | 撤銷 commit | `revert: revert feat: 新增用戶登入功能` |

---

## Step 3：撰寫 Commit Message

### 規則

**Subject（第一行）**
- 格式：`<type>(<scope>): <description>`
- `scope` 可選，填入影響的模組（如 `auth`、`api`、`ui`）
- 描述用**動詞開頭**（新增、修正、移除、優化、更新）
- **不超過 50 個字元**
- 不以句號結尾

**Body（選填）**
- 改動比較多或複雜時加上 body
- 說明「為什麼這樣改」，而不是「改了什麼」（diff 已經說明了 what）
- 每行不超過 72 個字元
- 與 subject 空一行

**Footer（選填）**
- 關聯 issue：`Closes #123`
- Breaking change：`BREAKING CHANGE: 說明影響`

### 範例

**簡單改動（只寫 subject）**
```
feat(auth): 新增 JWT 登入驗證
```

**複雜改動（加 body）**
```
refactor(payment): 拆分付款流程為獨立 Service

原本付款邏輯散落在 Controller 各處，難以維護。
將付款、退款、查詢邏輯集中到 PaymentService，
Controller 只負責 HTTP 層的處理。
```

**多個改動（使用 body 列舉）**
```
feat(slot): 新增免費遊戲與 Scatter 觸發機制

- 新增 FreeGame 模組，支援 10 次免費旋轉
- Scatter 出現 3 個以上觸發免費遊戲
- 免費遊戲中 Wild 倍率翻倍
```

---

## Step 4：Stage 並執行 Commit

```bash
# Stage 所有修改
git add .

# 執行 commit（subject 只有一行時）
git commit -m "feat: 新增用戶登入功能"

# 執行 commit（需要 body 時，用 heredoc）
git commit -m "$(cat <<'EOF'
refactor(payment): 拆分付款流程為獨立 Service

原本付款邏輯散落在 Controller 各處，難以維護。
將付款、退款、查詢邏輯集中到 PaymentService。
EOF
)"
```

---

## 判斷要不要加 body 的原則

| 情況 | 要不要加 body |
|------|-------------|
| 改動單純，一行說得清楚 | ❌ 不需要 |
| 改了 3 個以上檔案，各有不同目的 | ✅ 用列點說明 |
| 有不明顯的設計決策或 tradeoff | ✅ 說明原因 |
| 修 Bug 但原因不直觀 | ✅ 說明為什麼這樣修 |
| 重構但邏輯沒變 | ✅ 說明重構目的 |

---

## 使用方式

```
請讀取 commit/SKILL.md，然後幫我 commit 目前所有的修改。
```

或直接說：
```
幫我 commit，照 conventional commit 格式。
```
