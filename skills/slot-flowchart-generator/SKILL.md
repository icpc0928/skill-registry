# 老虎機流程圖生成器 — Skill

> **用途**：分析老虎機遊戲程式碼，自動生成完整的遊戲流程圖（Mermaid 格式）。
> **使用方式**：把這份文件 + 遊戲程式碼路徑交給 AI，自動產出流程圖文件。
> **輸出格式**：Mermaid（可直接在 GitHub、GitLab、Markdown 編輯器中渲染）

---

## 指令

你是一位資深老虎機遊戲架構師。請分析指定的老虎機遊戲程式碼，生成完整的遊戲流程圖。

### Step 1：掃描程式碼，識別遊戲狀態

掃描以下關鍵字來識別遊戲的狀態和流程：

| 元件 | 關鍵字 | 用途 |
|------|--------|------|
| **主流程** | `spin`, `play`, `gameLoop`, `round`, `startRound` | 遊戲主循環 |
| **停輪** | `stop`, `reelStop`, `evaluate`, `calculateWin` | 停輪和計算 |
| **連消** | `cascade`, `tumble`, `avalanche`, `collapse`, `drop` | 消除重填循環 |
| **免費遊戲** | `freeSpin`, `freeGame`, `bonusSpin`, `triggerFree` | Free Spin 觸發和流程 |
| **Bonus** | `bonus`, `bonusGame`, `pickGame`, `wheel` | Bonus 遊戲 |
| **倍率** | `multiplier`, `progressiveMultiplier`, `increaseMultiplier` | 倍率系統 |
| **Jackpot** | `jackpot`, `progressive`, `grandPrize` | 累積獎金 |
| **Gamble** | `gamble`, `doubleUp`, `riskGame` | 賭博翻倍 |
| **Buy Feature** | `buyFeature`, `buyBonus`, `anteBet` | 購買功能 |
| **結算** | `settle`, `payout`, `collect`, `endRound` | 結算贏分 |
| **狀態管理** | `state`, `gameState`, `setState`, `phase` | 狀態機 |
| **Scatter 判定** | `checkScatter`, `scatterCount`, `isScatter` | Scatter 觸發判定 |
| **Wild 處理** | `expandWild`, `stickyWild`, `walkingWild`, `wildTransform` | Wild 特殊行為 |
| **Retrigger** | `retrigger`, `addFreeSpin`, `extraSpin` | 重新觸發 |
| **隨機功能** | `randomFeature`, `mysterySymbol`, `randomWild` | 隨機觸發的特殊功能 |
| **收集機制** | `collect`, `meter`, `fill`, `progress`, `counter` | 收集/計量表 |
| **分裂符號** | `split`, `colossal`, `megaSymbol` | 符號分裂/合併 |

### Step 2：識別流程圖類型

根據遊戲複雜度，需要生成以下流程圖：

| 流程圖 | 說明 | 必要性 |
|--------|------|--------|
| **主遊戲流程** | 從投注到結算的完整流程 | ✅ 必要 |
| **狀態機圖** | 遊戲所有狀態和轉換 | ✅ 必要 |
| **連消流程** | Cascade/Tumble 的迴圈 | 有連消時必要 |
| **Free Spin 流程** | 免費遊戲觸發到結束 | 有 Free Spin 時必要 |
| **Bonus 遊戲流程** | Bonus 的完整流程 | 有 Bonus 時必要 |
| **倍率系統流程** | 倍率怎麼增長和重置 | 有倍率系統時必要 |
| **Jackpot 流程** | 累積獎金觸發和派獎 | 有 Jackpot 時必要 |
| **中獎計算流程** | 從停輪到算出贏分 | ✅ 必要 |
| **符號處理流程** | Wild 展開、Mystery 轉換等 | 有特殊符號處理時必要 |

### Step 3：生成 Mermaid 流程圖

#### 3.1 主遊戲流程

```mermaid
flowchart TD
    START([玩家按下 SPIN]) --> VALIDATE{餘額足夠?}
    VALIDATE -->|否| INSUFFICIENT[提示餘額不足]
    VALIDATE -->|是| DEDUCT[扣除投注金額]
    DEDUCT --> GENERATE[生成盤面結果]
    GENERATE --> DISPLAY[顯示轉動動畫]
    DISPLAY --> STOP[停輪]
    STOP --> EVALUATE[計算中獎]
    
    EVALUATE --> HAS_WIN{有中獎?}
    HAS_WIN -->|否| CHECK_SCATTER{Scatter 觸發?}
    HAS_WIN -->|是| SHOW_WIN[顯示贏分動畫]
    
    SHOW_WIN --> HAS_CASCADE{有連消機制?}
    HAS_CASCADE -->|否| CHECK_SCATTER
    HAS_CASCADE -->|是| CASCADE[消除中獎符號]
    CASCADE --> DROP[掉落新符號]
    DROP --> EVALUATE
    
    CHECK_SCATTER -->|是| FREE_SPIN[進入免費遊戲]
    CHECK_SCATTER -->|否| CHECK_BONUS{Bonus 觸發?}
    
    CHECK_BONUS -->|是| BONUS[進入 Bonus 遊戲]
    CHECK_BONUS -->|否| SETTLE[結算]
    
    FREE_SPIN --> SETTLE
    BONUS --> SETTLE
    
    SETTLE --> ADD_WIN[加入贏分至餘額]
    ADD_WIN --> HAS_GAMBLE{提供 Gamble?}
    HAS_GAMBLE -->|是| GAMBLE_CHOICE{玩家選擇?}
    HAS_GAMBLE -->|否| END_ROUND([回合結束])
    GAMBLE_CHOICE -->|Gamble| GAMBLE[賭博翻倍]
    GAMBLE_CHOICE -->|Collect| END_ROUND
    GAMBLE --> END_ROUND
```

#### 3.2 狀態機圖

```mermaid
stateDiagram-v2
    [*] --> Idle : 遊戲載入
    Idle --> Spinning : 按下 SPIN
    Spinning --> Evaluating : 停輪完成
    Evaluating --> Winning : 有中獎
    Evaluating --> Idle : 無中獎
    Winning --> Cascading : 有連消
    Winning --> Idle : 無連消
    Cascading --> Evaluating : 重新計算
    
    Evaluating --> FreeSpinTrigger : Scatter 觸發
    FreeSpinTrigger --> FreeSpinning : 開始免費遊戲
    FreeSpinning --> FreeEvaluating : 停輪完成
    FreeEvaluating --> FreeWinning : 有中獎
    FreeEvaluating --> FreeSpinCheck : 無中獎
    FreeWinning --> FreeCascading : 有連消
    FreeWinning --> FreeSpinCheck : 無連消
    FreeCascading --> FreeEvaluating : 重新計算
    FreeSpinCheck --> FreeSpinning : 剩餘次數 > 0
    FreeSpinCheck --> Idle : 免費遊戲結束
    
    Evaluating --> BonusGame : Bonus 觸發
    BonusGame --> Idle : Bonus 結束
```

#### 3.3 連消流程

```mermaid
flowchart TD
    SPIN_RESULT[停輪結果] --> EVAL{有中獎?}
    EVAL -->|否| END_CASCADE[連消結束]
    EVAL -->|是| CALC_WIN[計算贏分]
    CALC_WIN --> INC_MULT[倍率 +1]
    INC_MULT --> REMOVE[消除中獎符號]
    REMOVE --> DROP_NEW[掉落新符號]
    DROP_NEW --> EVAL
    
    END_CASCADE --> TOTAL[累計總贏分]
    TOTAL --> RESET_MULT[重置倍率]
```

#### 3.4 Free Spin 流程

```mermaid
flowchart TD
    TRIGGER[Scatter 觸發] --> AWARD[獲得 N 次免費旋轉]
    AWARD --> FS_SPIN[執行免費旋轉]
    FS_SPIN --> FS_STOP[停輪]
    FS_STOP --> FS_EVAL[計算中獎]
    
    FS_EVAL --> FS_WIN{有中獎?}
    FS_WIN -->|是| FS_SHOW[顯示贏分]
    FS_WIN -->|否| RETRIGGER_CHECK
    
    FS_SHOW --> FS_CASCADE{有連消?}
    FS_CASCADE -->|是| FS_REMOVE[消除 + 掉落]
    FS_CASCADE -->|否| RETRIGGER_CHECK
    FS_REMOVE --> FS_EVAL
    
    RETRIGGER_CHECK{Retrigger?}
    RETRIGGER_CHECK -->|是| ADD_SPINS[增加免費次數]
    RETRIGGER_CHECK -->|否| REMAIN_CHECK
    ADD_SPINS --> REMAIN_CHECK
    
    REMAIN_CHECK{剩餘次數 > 0?}
    REMAIN_CHECK -->|是| FS_SPIN
    REMAIN_CHECK -->|否| FS_SETTLE[免費遊戲結算]
    FS_SETTLE --> RETURN[返回主遊戲]
```

#### 3.5 中獎計算流程

```mermaid
flowchart TD
    REEL_RESULT[盤面結果] --> PROCESS_SPECIAL[處理特殊符號]
    
    PROCESS_SPECIAL --> EXPAND_WILD{有 Expanding Wild?}
    EXPAND_WILD -->|是| DO_EXPAND[展開 Wild]
    EXPAND_WILD -->|否| CHECK_MYSTERY
    DO_EXPAND --> CHECK_MYSTERY
    
    CHECK_MYSTERY{有 Mystery 符號?}
    CHECK_MYSTERY -->|是| TRANSFORM[轉換為隨機符號]
    CHECK_MYSTERY -->|否| CALC_START
    TRANSFORM --> CALC_START
    
    CALC_START --> CALC_TYPE{計算方式}
    CALC_TYPE -->|Lines| CALC_LINES[逐線計算]
    CALC_TYPE -->|Ways| CALC_WAYS[Ways 計算]
    CALC_TYPE -->|Cluster| CALC_CLUSTER[Cluster 計算]
    
    CALC_LINES --> APPLY_WILD[套用 Wild 替代]
    CALC_WAYS --> APPLY_WILD
    CALC_CLUSTER --> APPLY_WILD
    
    APPLY_WILD --> APPLY_MULT[套用倍率]
    APPLY_MULT --> SUM_WIN[加總所有中獎]
    SUM_WIN --> CHECK_MAX{超過最大贏分?}
    CHECK_MAX -->|是| CAP_WIN[封頂]
    CHECK_MAX -->|否| RETURN_WIN[回傳贏分]
    CAP_WIN --> RETURN_WIN
```

### Step 4：根據實際程式碼調整

以上是通用模板，需要根據實際程式碼：

1. **移除不存在的流程** — 沒有連消就刪掉 Cascade 相關
2. **新增特殊流程** — 如遊戲有獨特機制（如 Megaways 動態行數、Hold & Spin、Infinity Reels 等）
3. **標注具體數值** — 如 "獲得 10 次免費旋轉"、"倍率從 1x 開始，每次連消 +1"
4. **標注機率/權重** — 在判斷節點標上機率（如 "Scatter 觸發率 ≈ 1/150"）

### Step 5：生成額外圖表（如適用）

#### 5.1 倍率系統圖

```mermaid
flowchart LR
    C1[連消 1] -->|1x| C2[連消 2]
    C2 -->|2x| C3[連消 3]
    C3 -->|3x| C4[連消 4]
    C4 -->|5x| C5[連消 5+]
    C5 -->|10x| MAX[最大倍率]
    
    RESET([回合結束]) -.->|重置為 1x| C1
```

#### 5.2 收集機制圖

```mermaid
flowchart TD
    COLLECT[收集特定符號] --> METER[計量表 +1]
    METER --> FULL{計量表滿?}
    FULL -->|否| CONTINUE[繼續遊戲]
    FULL -->|是| UPGRADE[升級 / 觸發獎勵]
    UPGRADE --> RESET[重置計量表]
```

#### 5.3 Jackpot 流程圖

```mermaid
flowchart TD
    SPIN[每次投注] --> CONTRIBUTE[貢獻至獎金池]
    CONTRIBUTE --> TRIGGER{觸發 Jackpot?}
    TRIGGER -->|否| CONTINUE[繼續遊戲]
    TRIGGER -->|是| JP_LEVEL{哪個等級?}
    JP_LEVEL -->|Mini| MINI[Mini Jackpot]
    JP_LEVEL -->|Minor| MINOR[Minor Jackpot]
    JP_LEVEL -->|Major| MAJOR[Major Jackpot]
    JP_LEVEL -->|Grand| GRAND[Grand Jackpot]
    MINI --> AWARD[派獎]
    MINOR --> AWARD
    MAJOR --> AWARD
    GRAND --> AWARD
    AWARD --> RESET_JP[重置對應獎金池]
```

---

## 輸出規範

### Mermaid 語法注意事項

1. **節點 ID 不能有空格或特殊字元** — 用英文縮寫
2. **中文顯示在方括號/圓括號內** — `NODE[中文說明]`
3. **判斷用菱形** — `{條件?}`
4. **起始/結束用雙圓** — `([文字])`
5. **子流程用雙方括號** — `[[子流程]]`
6. **虛線表示可選路徑** — `-.->` 

### 配色建議

```mermaid
flowchart TD
    classDef trigger fill:#ff6b6b,stroke:#c92a2a,color:#fff
    classDef feature fill:#4dabf7,stroke:#1971c2,color:#fff
    classDef win fill:#51cf66,stroke:#2b8a3e,color:#fff
    classDef normal fill:#f8f9fa,stroke:#495057

    A[普通步驟]:::normal
    B[觸發條件]:::trigger
    C[Feature]:::feature
    D[贏分]:::win
```

---

## 品質檢查

生成完成後確認：

- [ ] 主遊戲流程圖涵蓋從投注到結算的所有步驟
- [ ] 狀態機圖包含所有可能的遊戲狀態
- [ ] 每個 Feature 都有獨立的流程圖
- [ ] 所有判斷節點（菱形）都有「是/否」兩條路徑
- [ ] 連消迴圈有明確的結束條件
- [ ] Free Spin 流程包含 Retrigger 判定
- [ ] 具體數值標注在流程圖中（如倍率、次數）
- [ ] Mermaid 語法正確，可以渲染
- [ ] 沒有孤立的節點（所有節點都連通）

---

## 使用方式

```
請讀取 slot-flowchart-generator/SKILL.md，
然後分析我的老虎機遊戲 {遊戲路徑}，生成完整的遊戲流程圖。
```

AI 會自動掃描程式碼 → 識別遊戲狀態和機制 → 生成 Mermaid 流程圖 → 輸出為 Markdown。

### 搭配使用

建議先跑 `slot-game-analyzer` 生成技術規格，再跑本 Skill 生成流程圖，兩份文件互相補充。
