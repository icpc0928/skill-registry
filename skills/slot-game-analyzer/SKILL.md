# 老虎機遊戲分析器 — Skill

> **用途**：分析老虎機遊戲程式碼，自動生成完整的技術規格文件。
> **使用方式**：把這份文件 + 遊戲程式碼路徑交給 AI，自動產出技術規格文件。

---

## 指令

你是一位資深老虎機遊戲數學家 / 技術分析師。請分析指定的老虎機遊戲程式碼，生成完整的技術規格文件。

### Step 1：掃描遊戲程式碼

識別遊戲的程式語言和結構：

```bash
# 列出遊戲目錄結構
find {遊戲路徑} -type f -name "*.java" -o -name "*.ts" -o -name "*.js" -o -name "*.py" -o -name "*.cs" | sort
```

### Step 2：識別核心元件

掃描程式碼，找出以下核心元件：

| 元件 | 關鍵字 / 檔案名 | 提取資訊 |
|------|----------------|---------|
| **盤面設定** | `reel`, `grid`, `rows`, `cols`, `reelStrips` | 軸數、行數、盤面結構 |
| **符號定義** | `symbol`, `enum`, `WILD`, `SCATTER`, `BONUS` | 符號表（ID、名稱、類型） |
| **賠率表** | `paytable`, `payout`, `payLine`, `multiplier` | 各符號的賠率 |
| **計算方式** | `payline`, `ways`, `cluster`, `megaways` | 線（Lines）/ 路（Ways）/ 消除（Cluster） |
| **特色玩法** | `feature`, `bonus`, `freeSpin`, `freeGame`, `respin` | 免費遊戲、Bonus 觸發條件 |
| **RTP 設定** | `rtp`, `returnRate`, `theoreticalReturn` | 理論回報率 |
| **波動度** | `volatility`, `variance`, `hitRate` | 波動度等級 |
| **Reel Strip** | `reelStrip`, `reelBand`, `strip` | 每軸符號排列（權重表） |
| **Wild 規則** | `wild`, `substitute`, `expand`, `sticky` | Wild 類型和行為 |
| **Scatter 規則** | `scatter`, `trigger`, `freeSpinTrigger` | Scatter 觸發條件 |
| **倍率系統** | `multiplier`, `progressiveMultiplier` | 倍率機制 |
| **最大贏分** | `maxWin`, `maxPayout`, `winCap` | 最大贏分上限 |
| **連消/級聯** | `cascade`, `tumble`, `avalanche`, `collapse` | 消除後重填機制 |
| **投注設定** | `bet`, `betMultiplier`, `coinValue`, `baseBet` | 投注倍數 |

### Step 3：分析盤面結構

#### 3.1 基本盤面

```
確認：
- 軸數（reels）
- 每軸行數（rows）— 是否固定或動態（如 Megaways）
- 是否有附加軸（top reel / extra reel）
- 計算方式（Lines / Ways / Cluster）
- 掉落方向（有無 cascade/tumble）
```

#### 3.2 Reel Strips 分析

```
提取每軸的符號排列：
- Base Game 的 reel strips
- Free Spin 的 reel strips（通常不同）
- 各符號在每軸出現的次數 → 計算出現機率
```

### Step 4：分析符號系統

#### 4.1 符號分類

| 類別 | 說明 |
|------|------|
| **低價值** | 通常是 10, J, Q, K, A 等撲克牌符號 |
| **中價值** | 主題相關的普通符號 |
| **高價值** | 主題核心符號，賠率較高 |
| **Wild** | 百搭符號，替代其他符號（注意：通常不替代 Scatter/Bonus） |
| **Scatter** | 觸發免費遊戲或 Bonus（不需要在 payline 上） |
| **Bonus** | 觸發特殊功能（如選擇遊戲、轉盤等） |
| **特殊** | 如 Multiplier Wild、Expanding Wild、Sticky Wild 等 |

#### 4.2 多格符號（Mega Symbols）

如果有多格符號，提取：
- 哪些符號可以是多格
- 尺寸權重（1x1, 2x2, 3x3, 4x4）
- 在哪些軸上出現

#### 4.3 賠率表

提取完整的賠率表：

```
符號 | 3個 | 4個 | 5個 | 6個（如有）
-----|-----|-----|-----|-----
```

注意：
- 賠率是 × 總投注 還是 × 線注
- 是否有 2 個就中獎的符號

### Step 5：分析特色玩法

#### 5.1 Free Spin / 免費遊戲

```
- 觸發條件（幾個 Scatter？在哪些軸？）
- 獲得幾次免費旋轉
- 是否可以重新觸發（retrigger）
- Free Spin 中有無額外加成（如倍率遞增、額外 Wild）
- Free Spin 的 reel strips 是否不同
```

#### 5.2 Bonus 遊戲

```
- 觸發條件
- 玩法類型（選擇、轉盤、收集等）
- 獎勵範圍
```

#### 5.3 其他特色

```
- Cascade / Tumble（連消）
- Multiplier 系統（倍率怎麼增長、何時重置）
- Buy Feature（花錢直接進免費遊戲）
- Gamble（贏分翻倍賭博）
- Random Features（隨機觸發的特殊功能）
- Progressive Jackpot（累積獎金）
```

### Step 6：分析數學模型

#### 6.1 RTP（理論回報率）

```
- Base Game RTP
- Free Spin RTP
- 總 RTP
- 是否有多個 RTP 設定（可調整）
```

#### 6.2 Hit Rate（中獎率）

```
- Base Game 中獎率
- 任何贏分的頻率
- Free Spin 觸發頻率
```

#### 6.3 波動度

```
根據以下判斷波動度等級（低/中/高/極高）：
- 最大贏分倍數
- 賠率分佈（高價值符號 vs 低價值符號差距）
- Free Spin 頻率和潛力
- Multiplier 上限
```

#### 6.4 最大贏分

```
- 理論最大贏分（× 投注）
- 達成條件（什麼情況下能拿到最大贏分）
```

### Step 7：生成技術規格文件

使用 `templates/TECH_SPEC.md` 的格式，生成完整文件。

輸出到遊戲目錄：`{game_path}/{GameName}_TechSpec.md`

---

## 輸出規範

### 文件結構

1. **遊戲概述** — 名稱、主題、基本參數
2. **盤面結構** — 軸數、行數、盤面圖示
3. **符號定義** — 符號表、賠率表
4. **計算方式** — Lines / Ways / Cluster 的具體計算邏輯
5. **特色玩法** — 每個 Feature 的觸發條件、流程、獎勵
6. **數學模型** — RTP、波動度、最大贏分
7. **Reel Strips** — 每軸的符號排列（附出現次數統計）
8. **狀態機** — 遊戲流程圖（Base → Feature → Settle）

### 語言

- 技術術語用英文（如 Wild, Scatter, RTP, Ways）
- 說明用中文
- 表格清晰、數據精確

---

## 品質檢查

生成完成後確認：

- [ ] 所有符號都有列出且賠率完整
- [ ] Wild 規則清楚（替代哪些、不替代哪些、出現位置）
- [ ] Free Spin 觸發條件和規則完整
- [ ] 所有特殊功能都有描述
- [ ] 數學模型數據與程式碼一致
- [ ] Reel Strips 數據正確（與程式碼比對）
- [ ] 最大贏分計算合理

---

## 使用方式

```
請讀取 slot-game-analyzer/SKILL.md，
然後分析我的老虎機遊戲 {遊戲路徑}，生成完整的技術規格文件。
```

AI 會自動掃描程式碼 → 識別遊戲機制 → 提取數學參數 → 生成 TechSpec.md。
