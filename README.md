# 🧩 Skill Registry

AI Agent 通用 Skill 倉庫 — Markdown-first，不綁定任何平台。

## 什麼是 Skill？

Skill 是一份結構化的指令文件，讓 AI Agent 能快速完成特定任務。  
每個 Skill 包含 `SKILL.md`（主文件）+ 模板/腳本，AI 讀取後即可執行。

## 📦 Skill 目錄

| Skill | 說明 | 版本 | 標籤 |
|-------|------|------|------|
| [commit](./skills/commit/) | 分析 git diff，自動生成 Conventional Commit message 並執行 commit | 1.0.0 | `git` `commit` `conventional-commit` |
| [qa-framework](./skills/qa-framework/) | 自動為任何專案生成完整 QA 測試框架 | 1.0.0 | `qa` `testing` `automation` |
| [api-docs-generator](./skills/api-docs-generator/) | 掃描後端程式碼，自動生成完整 API 文件 | 1.0.0 | `api` `documentation` `backend` |
| [docker-deploy](./skills/docker-deploy/) | 自動為前後端專案生成 Docker 部署方案 | 1.0.0 | `docker` `deployment` `devops` |
| [slot-game-analyzer](./skills/slot-game-analyzer/) | 分析老虎機程式碼，生成技術規格文件 | 1.0.0 | `slot` `game` `igaming` |
| [slot-flowchart-generator](./skills/slot-flowchart-generator/) | 分析老虎機程式碼，生成遊戲流程圖（Mermaid） | 1.0.0 | `slot` `flowchart` `mermaid` |

## 🚀 使用方式

### 方式 1：直接給 AI 讀

```
請讀取 https://raw.githubusercontent.com/{user}/skill-registry/main/skills/qa-framework/SKILL.md
然後為我的專案生成 QA 測試框架。
```

### 方式 2：Clone 到本地

```bash
git clone https://github.com/{user}/skill-registry.git
```

然後讓 AI 讀取 `skills/{skill-name}/SKILL.md`。

### 方式 3：只下載單一 Skill

```bash
# 下載特定 skill 資料夾
svn export https://github.com/{user}/skill-registry/trunk/skills/qa-framework
```

## 📁 Skill 結構

每個 Skill 遵循統一格式：

```
skills/{skill-name}/
├── SKILL.md          # 主文件（AI 讀這個）
├── skill.json        # 元資料（名稱、版本、作者、標籤）
├── UPDATE.md         # 增量更新指引（可選）
├── templates/        # 模板檔案（可選）
├── scripts/          # 腳本（可選）
└── examples/         # 範例（可選）
```

### skill.json 格式

```json
{
  "name": "skill-name",
  "version": "1.0.0",
  "description": "Skill 說明",
  "author": "作者",
  "tags": ["tag1", "tag2"],
  "platforms": ["any"],
  "license": "MIT",
  "usage": "一句話說明怎麼用"
}
```

## 🤝 貢獻 Skill

1. Fork 此 repo
2. 在 `skills/` 下建立你的 Skill 資料夾
3. 確保包含 `SKILL.md` 和 `skill.json`
4. 更新 README.md 的 Skill 目錄表
5. 發 PR

詳見 [CONTRIBUTING.md](./CONTRIBUTING.md)

## 📜 License

MIT
