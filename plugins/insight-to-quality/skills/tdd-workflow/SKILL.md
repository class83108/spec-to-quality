---
name: ec:tdd-workflow
description: >
  Python 專案的 TDD 紅綠燈流程，整合 Gherkin + pytest-bdd。強制按照 Red → Green → Refactor 順序執行，
  不可跳過任何階段。當使用者要開始實作功能、進入 TDD 的實作階段、或說「開始實作」時觸發。
  Do NOT use for: 寫簡單 script、修 typo、改設定檔、還沒有 .feature 檔時（先用 ec:feature-coverage）、
  或 bug 修復時（用 ec:debugging）。
---

# TDD Workflow (Gherkin + pytest-bdd)

你正在進入 TDD 開發流程。此流程是 rigid 的——必須嚴格按步驟執行，不可跳過或合併步驟。

## 前置條件

- .feature 檔已經存在且經過 ec:feature-coverage skill 的覆蓋率確認
- 如果 .feature 檔不存在，先停下來提醒使用者需要先完成 ec:feature-coverage 分析

## 測試與品質命令

參照專案 CLAUDE.md 的 Commands 區段取得正確的測試、lint、type check 命令。
不要假設任何特定的套件管理工具或命令格式。

## 流程

### Phase 0: Verification Ledger（Mock 邊界審查）

在寫任何測試之前，先對照 spec 的 SHALL 語句，規劃每個 SHALL 的驗證歸屬。

#### Step 0: 讀取 Feature Plan

讀 `docs/feature-plans/{feature-name}.md`，提取並記錄：
- **Error Handling Strategy**（Catch boundary、Domain errors、Recovery strategy）— mock 邊界的切點依據
- **Anti-patterns**（AP1、AP2...）— 哪些失敗模式需要明確模擬
- **Boundary Rules** — 資料流方向，影響 Checkpoint B 的判斷

這份資訊在 Step 2 規劃 mock 邊界時直接使用，不需要再去讀 discovery 原始文件。

#### Step 1: 列出 SHALL 清單

從 OpenSpec 的 spec.md 中提取所有 SHALL / MUST 語句。如果沒有 OpenSpec，從 .feature 的 Then 步驟反推應驗證的行為。

#### Step 2: 規劃 Mock 邊界

先從 feature plan 的 Anti-patterns 出發：這些 anti-patterns 指出哪些失敗模式需要明確模擬——mock 邊界應該能讓這些模式可測試，而不是讓它們被吸收掉看不見。Mock 的切點應與 feature plan 宣告的 Catch boundary 一致。

對每個 SHALL，判斷：

1. **可以 unit test 直接驗證？** → 標記為 unit test，規劃 mock 邊界
2. **需要外部服務才能驗證？** → 分級處理：
   - 等級 1（可本地跑）：testcontainers / in-memory 替代 → unit test
   - 等級 2（不可本地跑，output 可預測）：寫 Fake（非 MagicMock）回傳固定 response → unit test + 標記
   - 等級 3（不可本地跑，行為不可預測）：標記為「需要整合測試」

#### Step 3: Mock 邊界檢查（Checkpoint A + B）

對每個計畫中的 mock：
- **Checkpoint A**：這個 mock 遮掉的是「外部服務的行為」還是「自己的邏輯」？如果是自己的邏輯 → 調整 mock 邊界，mock 應該在外部依賴的最外層切
- **Checkpoint B**：跨元件的資料流（A 產出 → B 消費），A 的 output 形狀 = B 期望的 input 形狀？有驗證嗎？
- 如果使用 `MagicMock`，必須加 `spec=` 參數限制介面；否則優先使用自定義 Fake

#### Step 4: 產出 Verification Ledger

將完整 ledger（四區）展示給使用者，確認 ledger 內容是否正確（mock 邊界、分級是否合理）。

對話展示格式：

```markdown
# Verification Ledger — [功能名稱]

## Error Handling Strategy（從 feature plan 讀取）
- Catch boundary: [boundary only / per-layer / selective]
- Domain errors: [TaskNotFound, QueueFull]
- Infrastructure errors: [fail fast / retry N / degrade to X]

## Unit Test 覆蓋
- SHALL xxx → mock 邊界：mock [外部依賴]，驗證 [自己的邏輯]

## 需要整合測試
- SHALL xxx
  - 原因：[為什麼 unit test 無法驗證]
  - 最低驗證方式：[具體可執行的驗證方法]

## 明確不測（附理由）
- SHALL xxx — [原因，例如 scope out 或 v1 不做]
```

使用者確認後，如果「需要整合測試」或「明確不測」兩區有內容，將缺口寫入 `docs/feature-plans/{feature-name}.md` 的 `## Integration Test Gaps` 區段：

```markdown
## Integration Test Gaps

### 需要整合測試
- SHALL xxx
  - 原因：[為什麼 unit test 無法驗證]
  - 最低驗證方式：[具體可執行的驗證方法]

### 明確不測（附理由）
- SHALL xxx — [原因，例如 scope out 或 v1 不做]
```

如果兩個缺口區段都是空的，`## Integration Test Gaps` 留空，不需要填入內容。

**使用者確認 ledger 且 feature plan 已更新後，才進入 Phase 1。**

### Phase 1: Red（寫測試，確認失敗）

在開始撰寫 step definitions 之前，讀 `references/implementation-mindset.md` Part 2 的 **Naming** 和 **Testability** 兩個維度：step 名稱建立了這個 feature 的語彙，會延伸到 production code；如果撰寫測試時感覺難以隔離或需要 mock 太多東西，這是設計問題的信號，先暫停討論再繼續。

1. **掃描現有 step definitions**：在建立任何測試檔之前，掃描 `step_defs/` 目錄和 `conftest.py`：
   - 列出可複用的 step pattern（Given/When/Then 句子）、fixtures、和 helper functions
   - 標記哪些新 scenario 的步驟可以直接複用，哪些需要新寫
   - **決定放置位置**：新 step definitions 要新開 `step_defs/{feature_name}_steps.py`，還是加入現有檔案？判斷依據：功能相近的步驟集中在一起，避免跨檔案重複定義
   - **注意**：pytest-bdd 中，若兩個 step function 匹配相同 pattern，後定義的會無聲覆蓋前者——必須避免重複 pattern
2. **建立或選定測試檔**：根據上一步的決定，新建或選定目標檔案
3. **撰寫 step definitions**：
   - 按 scenario 順序組織（對應 .feature 檔結構），不要按 step type 分組
   - 每個 scenario 的 given/when/then 放在一起
   - 複用的 step 只寫 import，不重複定義
4. **執行測試**：使用專案 CLAUDE.md 中定義的測試命令
5. **確認紅燈**：所有新測試必須 FAIL
   - 如果有測試意外 PASS → 停下來，告訴使用者哪些 pass 了，討論是否 scenario 寫錯或已有實作
   - 將完整的失敗輸出展示給使用者
6. **等待使用者確認**：明確說「紅燈確認，要開始實作嗎？」

**在使用者確認紅燈前，不得撰寫任何 production code。**

### Phase 2: Green（最小實作，讓測試通過）

在開始實作之前，讀 `references/implementation-mindset.md` Part 2 的 **Single Responsibility**、**Dependency Direction** 和 **Consistency** 三個維度：實作時有意識地選擇，比 design-review 事後發現要好。

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
- 提醒使用者可以觸發 ec:design-review 做設計品質檢視

## Examples

### Example 1: 正常的 TDD 流程

使用者說：「feature 檔確認了，開始實作使用者註冊」

1. 確認 `user_registration.feature` 存在
2. **Verification Ledger**：從 spec 提取 6 個 SHALL，規劃 mock 邊界 → 5 個 unit test 直接驗證，1 個（email 實際寄送）標記需要整合測試 → 展示 ledger 並詢問「內容 OK 嗎？要建 verification.md 嗎？」 → 使用者確認內容、決定是否建檔
3. 建立 `test_user_registration.py`，依照 ledger 的 mock 邊界撰寫 step definitions
4. 跑測試 → 全部 FAIL → 展示紅燈輸出 → 「紅燈確認，要開始實作嗎？」
5. 使用者確認 → 逐 scenario 實作，每完成一個跑一次測試
6. 全部 PASS → 展示綠燈 → 跑 lint + type check → 修正問題 → 重跑測試確認綠
7. 提醒可以做 ec:design-review

### Example 2: 紅燈階段有測試意外通過

跑測試時發現 3 個 FAIL、1 個 PASS。

正確行為：停下來告訴使用者「Scenario X 意外通過了，可能原因：1) 已有實作覆蓋了這個行為 2) scenario 寫得太寬鬆。要怎麼處理？」

### Example 3: Green 階段發現 feature 需要修改

實作過程中發現某個 scenario 的 Given 條件不合理。

正確行為：停下來告訴使用者「我在實作 Scenario X 時發現 Given 條件可能需要調整，建議回到 ec:feature-coverage 討論」，不要自己偷改 .feature 檔。

## 關鍵規則

- **紅綠燈順序不可逆轉**：Red → Green → Refactor，絕對不能跳步
- **不要一次寫完所有 production code** — 可以分 scenario 逐步實作
- **每次測試結果都要展示給使用者**，不要只說「通過了」
- **如果在 Green 階段發現 .feature 需要修改**，停下來告訴使用者，回到 ec:feature-coverage 討論
