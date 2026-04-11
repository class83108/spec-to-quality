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

**在開始之前，先讀 `references/implementation-mindset.md` Part 3**，取得 6 類 scenario 的定義、分析起點映射、和 overlap 判斷規則。

## 流程（嚴格遵守）

### Step 0: 讀取 Feature Plan

讀 `docs/feature-plans/{feature-name}.md`（命名與此 OpenSpec change 相同）。

如果找不到對應的 feature plan：
→ 停下來告知：「這個 change 沒有對應的 feature plan。請先執行 ec:feature-planning 建立 feature plan，再進行覆蓋率分析。」

從 feature plan 提取並記錄以下資訊——這是類別 1、2、3 的分析起點：

- **Serves**：Gx 清單與對應的 goal 描述
- **Error Handling Strategy**：Catch boundary、Domain errors、Recovery strategy
- **Anti-patterns**：帶 ID 的清單（AP1、AP2...）與說明
- **Boundary Rules**：觸碰的 boundary 與限制

完成 Step 0 後，才進入 Step 1。

### Step 1: 讀取 OpenSpec 實作細節

讀取相關 OpenSpec change 的具體行為規格——這是類別 4、5、6 的分析起點：

- spec.md 的 SHALL / MUST 語句（類別 5 State mutation 的來源）
- Requirements 的條件邏輯（類別 4 Business rules 的來源）
- schema 定義（類別 6 Output contract 的來源）

同時讀取：
- 專案 CLAUDE.md 中的「Feature Scenario 具體化對應表」（如果有的話）
- 如果專案中已有同類型的 .feature 檔，讀取 1-2 個作為覆蓋率參考基準

### Step 2: 產出覆蓋率分析表

對以下 6 類 scenario 類別逐一判斷，輸出固定格式表格。每類的分析起點和定義見 `implementation-mindset.md` Part 3。

| # | 類別 | 適用 | 具體 Scenario 方向 | Gx | 不適用原因 |
|---|------|------|-------------------|----|-----------|
| 1 | Happy path — 正常流程端到端 | | | | |
| 2 | Error / Failure paths — anti-patterns 觸發、domain error 回傳、外部依賴失敗 | | | | |
| 3 | Boundary & Edge cases — theory limits、空值、超大值、最小輸入 | | | | |
| 4 | Business rules — 條件邏輯、多條件組合、規則例外 | | | | |
| 5 | State mutation — DB/檔案寫入正確性、冪等性、re-run 清理 | | | | |
| 6 | Output contract — 回傳格式符合 schema、boundary 跨越處的 shape match | | | | |

**適用欄**：填「是」或「否」。否必須填不適用原因，不得空白。

完成分析表後，**接著做 Step 3**，不要跳過。

### Step 3: Gx 完整性確認

確認 feature plan 的 Serves 中每個 Gx，在分析表的 Gx 欄中都至少出現一次。

如果某個 Gx 完全沒有出現 → 標記出來，詢問使用者是否遺漏了覆蓋，或這個 goal 在此 change 中確實不需要驗證（附理由）。

### Step 4: 與既有 feature 交叉比對

如果專案中有結構類似的 .feature 檔（例如同一層級的其他模組），列出：
- 那些 feature 覆蓋了哪些類別
- 本次分析結果與它們的差異
- 差異是否合理（有意的差異 vs. 遺漏）

### Step 5: 等待確認

將分析結果呈現給使用者，明確詢問：

1. 是否有遺漏的類別或 scenario？
2. 不適用的判斷是否合理？
3. Gx 的對應是否完整？
4. 與既有 feature 的差異是否可接受？

**在使用者明確確認前，不得開始撰寫 .feature 檔。**

回應的最後一句必須明確說：「確認後，我會使用 `ec:gherkin` 開始撰寫 .feature 檔。撰寫完成後會進入 Verification Ledger（Mock 邊界審查）再開始寫測試。」

### Step 6: 進入 Gherkin 撰寫

確認後，使用 ec:gherkin skill 開始撰寫 .feature 檔。確保分析表中每個「適用」的類別都有對應的 Scenario。

## Examples

### Example 1: 正常流程

使用者說：「開始做覆蓋率分析」

**Step 0**：讀 `docs/feature-plans/task-queue.md`，提取：
- Serves: G2（系統必須支援並行處理最多 5 個任務）
- Error Handling Strategy: catch boundary = boundary only；domain errors = TaskNotFound, QueueFull；infrastructure errors = fail fast
- AP1 (D1): 任務未完成不接新任務、AP2 (D2): 狀態必須落地 DB
- Boundary Rules: 資料只能從 TaskQueue 流向 Worker，不可反向

**Step 1**：讀 OpenSpec spec.md，取得 SHALL 語句（任務狀態必須寫入 DB）和條件邏輯（若 queue 已滿則拒絕）

**Step 2** 分析表：

| # | 類別 | 適用 | 具體 Scenario 方向 | Gx | 不適用原因 |
|---|------|------|-------------------|----|-----------|
| 1 | Happy path | 是 | 5 個任務並行，全部成功完成 | G2 | |
| 2 | Error / Failure paths | 是 | AP1 觸發（QueueFull，回傳結構化 domain error）；AP2 驗證（Worker crash 後狀態已落地）；DB 連線失敗時 fail fast | G2 | |
| 3 | Boundary & Edge cases | 是 | 恰好 5 個並行（theory limit 上限）；只有 1 個任務（最小值） | G2 | |
| 4 | Business rules | 否 | | | spec.md Requirements 無條件分支邏輯 |
| 5 | State mutation | 是 | 任務狀態寫入 DB 後可查詢；重複觸發不產生重複任務 | G2 | |
| 6 | Output contract | 是 | TaskQueue ↔ Worker boundary 的 schema match | G2 | |

**Step 3**：G2 在所有適用類別都有出現，完整。

### Example 2: 找不到 feature plan

Step 0 找不到 `docs/feature-plans/task-queue.md`。

正確行為：「這個 change 沒有對應的 feature plan。請先執行 ec:feature-planning 建立 feature plan，再進行覆蓋率分析。」不要繼續往下做。

### Example 3: 使用者跳過分析直接要求寫 feature

使用者說：「直接幫我寫 .feature 檔，不用分析了」

正確行為：提醒使用者覆蓋率分析的價值，然後讓使用者決定：

1. 使用者堅持跳過 → 尊重決定，直接交給 `ec:gherkin` 撰寫 .feature
2. 使用者接受建議 → 開始 Step 0，走完整流程

## 注意事項

- 「不適用」需要有理由，不能空白
- 如果某個類別不確定是否適用，標記為「待討論」而非直接標否
- 覆蓋率分析是為了確保一致性，不是追求 100% 覆蓋——有意識地跳過比無意識地遺漏好
- Gx 欄可以填多個 ID（一個 scenario 服務多個 goal），用逗號分隔
- Overlap 判斷規則見 `implementation-mindset.md` Part 3
