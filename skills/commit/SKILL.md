---
name: commit
description: 用 Conventional Commit 格式自動生成 commit message 並執行 commit
---

幫我 commit 目前所有的修改。

先用 `git diff` 和 `git status` 看改了什麼，再決定 commit message。

commit message 規則：
1. 用 Conventional Commit 格式開頭：`feat:` / `fix:` / `refactor:` / `perf:` / `docs:` / `test:` / `chore:` / `ci:`
2. 第一行不超過 50 個字元，用繁體中文描述
3. 如果改動比較多或複雜，空一行後加 body 說明為什麼這樣改
4. 最後執行 `git add .` 然後 `git commit`
