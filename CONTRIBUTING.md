# 貢獻指南

感謝你想貢獻 Skill！以下是流程和規範。

## 新增 Skill

### 1. 建立資料夾

```
skills/{your-skill-name}/
├── SKILL.md          # 必須 — AI 讀取的主文件
├── skill.json        # 必須 — 元資料
├── UPDATE.md         # 可選 — 增量更新指引
├── templates/        # 可選 — 模板檔案
├── scripts/          # 可選 — 腳本
└── examples/         # 可選 — 使用範例
```

### 2. SKILL.md 規範

- 開頭說明這個 Skill 做什麼
- 清楚列出 AI 需要執行的步驟
- 定義輸入（AI 需要知道什麼）和輸出（會生成什麼）
- 使用 Markdown 格式，任何 AI 都能讀

### 3. skill.json 必填欄位

| 欄位 | 說明 |
|------|------|
| `name` | Skill 名稱（英文、小寫、用 `-` 連接） |
| `version` | 語義化版本 (semver) |
| `description` | 一句話說明 |
| `author` | 作者名稱 |
| `tags` | 標籤陣列 |
| `platforms` | 支援的平台（`any` 表示通用） |

### 4. 更新 README.md

在 Skill 目錄表新增一行。

### 5. 發 Pull Request

- 標題格式：`[New Skill] skill-name`
- 說明你的 Skill 做什麼、怎麼用

## 更新 Skill

- 修改內容 + 更新 `skill.json` 的 `version`
- PR 標題格式：`[Update] skill-name v1.1.0`

## 命名規則

- 全小寫英文
- 用 `-` 連接單字
- 簡短明確（如 `qa-framework`、`stock-tracker`、`deploy-docker`）

## 品質標準

- [ ] SKILL.md 能讓 AI 獨立完成任務，不需要額外說明
- [ ] 有明確的輸入和輸出定義
- [ ] 不包含敏感資訊（API key、密碼等）
- [ ] 不包含惡意指令
