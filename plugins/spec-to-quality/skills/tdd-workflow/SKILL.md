---
name: tdd-workflow
description: >
  Python 專案的 TDD 紅綠燈流程，整合 Gherkin + pytest-bdd。強制按照 Red → Green → Refactor 順序執行，
  不可跳過任何階段。當使用者要開始實作功能、進入 TDD 的實作階段、或說「開始實作」時觸發。
  Do NOT use for: 寫簡單 script、修 typo、改設定檔、還沒有 .feature 檔時（先用 feature-coverage）、
  或 bug 修復時（用 debugging）。
---

# TDD Workflow (Gherkin + pytest-bdd)

你正在進入 TDD 開發流程。此流程是 rigid 的——必須嚴格按步驟執行，不可跳過或合併步驟。

## 前置條件

- .feature 檔已經存在且經過 feature-coverage skill 的覆蓋率確認
- 如果 .feature 檔不存在，先停下來提醒使用者需要先完成 feature-coverage 分析

## 測試與品質命令

參照專案 CLAUDE.md 的 Commands 區段取得正確的測試、lint、type check 命令。
不要假設任何特定的套件管理工具或命令格式。

## 流程

### Phase 1: Red（寫測試，確認失敗）

1. **建立測試檔**：在專案的測試目錄下建立對應的測試檔
2. **撰寫 step definitions**：
   - 按 scenario 順序組織（對應 .feature 檔結構），不要按 step type 分組
   - 每個 scenario 的 given/when/then 放在一起
3. **執行測試**：使用專案 CLAUDE.md 中定義的測試命令
4. **確認紅燈**：所有新測試必須 FAIL
   - 如果有測試意外 PASS → 停下來，告訴使用者哪些 pass 了，討論是否 scenario 寫錯或已有實作
   - 將完整的失敗輸出展示給使用者
5. **等待使用者確認**：明確說「紅燈確認，要開始實作嗎？」

**在使用者確認紅燈前，不得撰寫任何 production code。**

### Phase 2: Green（最小實作，讓測試通過）

1. **只寫讓測試通過的最小 code** — 不要預先最佳化、不要加額外功能
2. **每完成一個 scenario 的實作就跑一次測試**，讓使用者看到進度
3. **全部通過後展示綠燈**：
   - 顯示完整的通過輸出
   - 明確說「綠燈，所有 N 個 scenario 通過」

### Phase 3: Refactor（品質檢查）

1. **Lint & Format**：使用專案 CLAUDE.md 中定義的 lint/format 命令
2. **Type check**：使用專案 CLAUDE.md 中定義的 type check 命令
3. **如果有錯誤**：修正後重跑測試確認還是綠的
4. **評估是否需要重構**：
   - Function complexity 是否超過 10？
   - 有沒有重複的邏輯可以抽取？
   - 命名是否清晰？
   - 如果需要重構，每次重構後都要重跑測試確認綠燈

### Phase 4: 完成

- 列出本次新增/修改的所有檔案
- 確認測試最終狀態是綠的
- 提醒使用者可以觸發 design-review 做設計品質檢視

## Examples

### Example 1: 正常的 TDD 流程

使用者說：「feature 檔確認了，開始實作使用者註冊」

1. 確認 `user_registration.feature` 存在
2. 建立 `test_user_registration.py`，撰寫 step definitions
3. 跑測試 → 全部 FAIL → 展示紅燈輸出 → 「紅燈確認，要開始實作嗎？」
4. 使用者確認 → 逐 scenario 實作，每完成一個跑一次測試
5. 全部 PASS → 展示綠燈 → 跑 lint + type check → 修正問題 → 重跑測試確認綠
6. 提醒可以做 design-review

### Example 2: 紅燈階段有測試意外通過

跑測試時發現 3 個 FAIL、1 個 PASS。

正確行為：停下來告訴使用者「Scenario X 意外通過了，可能原因：1) 已有實作覆蓋了這個行為 2) scenario 寫得太寬鬆。要怎麼處理？」

### Example 3: Green 階段發現 feature 需要修改

實作過程中發現某個 scenario 的 Given 條件不合理。

正確行為：停下來告訴使用者「我在實作 Scenario X 時發現 Given 條件可能需要調整，建議回到 feature-coverage 討論」，不要自己偷改 .feature 檔。

## 關鍵規則

- **紅綠燈順序不可逆轉**：Red → Green → Refactor，絕對不能跳步
- **不要一次寫完所有 production code** — 可以分 scenario 逐步實作
- **每次測試結果都要展示給使用者**，不要只說「通過了」
- **如果在 Green 階段發現 .feature 需要修改**，停下來告訴使用者，回到 feature-coverage 討論
