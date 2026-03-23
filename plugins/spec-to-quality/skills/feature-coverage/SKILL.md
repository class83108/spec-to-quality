---
name: ec:feature-coverage
description: >
  寫 .feature 檔之前的覆蓋率分析流程。強制在撰寫 Gherkin scenario 前，先對 6 類 scenario 類別
  逐一判斷「適用 / 不適用（附原因）」並等使用者確認。
  當使用者要開始寫 .feature 檔、定義測試情境、規劃 scenario、或說「覆蓋率分析」時觸發。
  Do NOT use for: 已經有 .feature 檔要開始實作時（用 ec:tdd-workflow）、修 bug 時、或單純討論需求時。
---

# Feature 覆蓋率分析

你正在進入 .feature 檔撰寫的前置步驟。在寫任何 Gherkin scenario 之前，你必須完成覆蓋率分析並取得使用者確認。

## 流程（嚴格遵守）

### Step 1: 收集上下文

1. 如果專案使用 OpenSpec，讀取相關的 change（spec, design decisions, tasks）
2. 讀取專案 CLAUDE.md 中的「Feature Scenario 具體化對應表」（如果有的話）
3. 如果專案中已有同類型的 .feature 檔，讀取 1-2 個作為覆蓋率參考基準

### Step 2: 產出覆蓋率分析表

對以下 6 類 scenario 類別，逐一判斷並輸出表格：

| # | 類別 | 適用？ | 具體 Scenario 方向 | 不適用原因 |
|---|------|--------|-------------------|-----------|
| 1 | **Happy path** — 正常流程端到端 | | | |
| 2 | **Error / Failure paths** — 外部依賴失敗、例外處理、重試策略 | | | |
| 3 | **Input boundary** — 空值、超大值、格式錯誤、邊界值 | | | |
| 4 | **Edge cases** — 單一元素、重複執行、特殊字元、併發情境 | | | |
| 5 | **State mutation** — DB/檔案寫入正確性、冪等性、re-run 清理 | | | |
| 6 | **Output contract** — 回傳格式符合 schema、欄位完整性 | | | |

完成分析表後，**接著做 Step 3 交叉比對**，不要跳過。

### Step 3: 與既有 feature 交叉比對

如果專案中有結構類似的 .feature 檔（例如同一層級的其他模組），列出：
- 那些 feature 覆蓋了哪些類別
- 本次分析結果與它們的差異
- 差異是否合理（有意的差異 vs. 遺漏）

### Step 4: 等待確認

將分析結果呈現給使用者，明確詢問：

1. 是否有遺漏的類別或 scenario？
2. 不適用的判斷是否合理？
3. 與既有 feature 的差異是否可接受？

**在使用者明確確認前，不得開始撰寫 .feature 檔。**

回應的最後一句必須明確說：「確認後，我會使用 `ec:gherkin` 開始撰寫 .feature 檔。撰寫完成後會進入 Verification Ledger（Mock 邊界審查）再開始寫測試。」

### Step 5: 進入 Gherkin 撰寫

確認後，使用 ec:gherkin skill 開始撰寫 .feature 檔。確保分析表中每個「適用」的類別都有對應的 Scenario。

## Examples

### Example 1: 使用者要新增一個 CRUD 功能的 feature

使用者說：「幫我寫使用者註冊的 feature」

1. 收集上下文（openspec, 既有 feature）
2. 產出分析表：
   - Happy path: 適用 — 正常註冊流程
   - Error paths: 適用 — email 已存在、外部驗證服務失敗
   - Input boundary: 適用 — email 格式錯誤、密碼太短、必填欄位空白
   - Edge cases: 適用 — 重複提交、特殊字元 email
   - State mutation: 適用 — User 寫入 DB、重複註冊不應產生多筆
   - Output contract: 適用 — 回傳使用者資料符合 schema
3. 交叉比對既有的 `login.feature`，確認覆蓋類別一致
4. 等使用者確認

### Example 2: 使用者跳過分析直接要求寫 feature

使用者說：「直接幫我寫 .feature 檔，不用分析了」

正確行為：提醒使用者覆蓋率分析的價值，然後讓使用者決定：

1. 使用者堅持跳過 → 尊重決定，直接交給 `ec:gherkin` 撰寫 .feature
2. 使用者接受建議 → 開始 Step 1，走完整流程

### Example 3: 某些類別明顯不適用

使用者要寫一個「純計算」功能的 feature（例如：匯率轉換）

分析表中：
- State mutation: 不適用 — 純函式計算，無 DB 寫入或副作用
- Error paths: 適用 — 外部匯率 API 失敗的 fallback 策略

「不適用」附上理由，使用者可能會補充「其實有 cache，所以 state mutation 要考慮」。

## 注意事項

- 「不適用」需要有理由，不能只寫「不適用」
- 如果某個類別你不確定是否適用，標記為「待討論」而非直接標不適用
- 覆蓋率分析是為了確保一致性，不是為了追求 100% 覆蓋 — 有意識地跳過比無意識地遺漏好
