# insight-to-quality

一套把「發現問題、切片、釐清、驗證、實作、回寫」串成同一條鏈的開發 skill。

重點不是產生很多文件，而是讓 AI 與人類在正確抽象層上協作：

- 先定義目標
- 先定義全局互動面與主要系統語言
- 再快速找出真正會主導設計的壓力
- 再切出責任模塊與關鍵 seams
- 再對單一 feature slice 補足清晰度
- 必要時只對高風險區塊做 deep dive
- 再依風險選測試策略進 TDD
- 最後檢查是否需要回寫上游假設

---

## 核心主鏈

目前新的主鏈是：

1. `goals-discovery`
2. `design-driver-discovery`
3. `system-map`
4. `feature-slice`
5. `spec-clarification`
6. `targeted deep dive`（必要時）
7. `tdd-ready-check`
8. `gherkin-extraction` 或 `direct TDD`
9. `tdd-workflow`
10. `design-review`

這不是瀑布流程，而是：

- 大方向由抽象往實作走
- 局部問題往正確上一層回推
- 不用每次都重跑全部 discovery

---

## 兩份 Mindset

### Architect Mindset

用來支撐 discovery 與結構設計。

核心原則包括：

- 設計是為了吸收變動
- 抽象是為了保護穩定意義
- 邊界應該跟著變更與故障切
- 責任比技術容器更重要
- 真正的壓力應該主導結構

### Implementation Mindset

用來支撐 pre-TDD、測試策略、實作、review。

核心原則包括：

- 測試前先釐清
- 測試應保護主要風險
- 選擇最低但誠實的測試層
- Red 要暴露真正的不確定性
- Refactor 要消除 drift，而不是只整理 code style
- 當結構假設移動時，要往上回寫

---

## Discovery 層

### 1. `goals-discovery`

回答：

- 系統要解什麼問題
- 一定要做到什麼
- 明確不做什麼
- 哪些失敗不能接受

主要產物：

- `goals.md`

### 2. `design-driver-discovery`

回答：

- 全局主要 actor 與 interaction surface 是什麼
- 哪些 flow 值得優先保護
- 哪些壓力會逼出 architecture decision
- 哪些 operation 只是 symptom，不是 design driver
- 哪些地方只是先標記，後續才值得 deep dive

主要產物：

- `design-driver-discovery.md`

### 3. `system-map`

回答：

- 全局 interaction surface 對應到哪些責任單位
- 主要責任模塊是什麼
- 哪些 seams 是關鍵 handoff
- 核心 state / entity ownership 在哪裡
- 改某件事時先看哪裡

主要產物：

- `SYSTEM_MAP.md`

---

## Pre-TDD 層

### 4. `feature-slice`

把一個 feature idea 切成：

- 哪條 flow 的哪一段
- 支援哪個 goal
- 對應哪個 design driver
- 主要 responsibility unit 與 seams 是什麼

主要產物：

- `docs/features/<feature-slug>/work-card.md`

### 5. `spec-clarification`

補足這個 slice 真正缺的清晰度。

三種視角：

- `surface`：外部互動與可觀察結果
- `contract`：責任交接與 seam semantics
- `behavior`：規則、狀態轉移、正確性

不是每次都三種全跑。
shared work card 是預設容器；只有在 detail 值得長期保留時，才獨立成 `surface.md / contract.md / behavior.md`。
如果發現 ownership、failure semantics、consistency、或 shared model 仍然只能靠猜，這一層要升級成 targeted deep dive，而不是直接進 TDD。

### 6. `targeted deep dive`

不是新瀑布階段，而是條件式機制。

觸發時機：

- boundary 畫不穩
- state / entity ownership 不清楚
- 高壓 flow 的 failure / retry / consistency 不清楚
- 不同 slice 開始長出不一致的 API / model
- `spec-clarification` 後仍然需要猜測才能寫測試或定 seam

輸出：

- 回寫 `design-driver-discovery.md`
- 或補強 `SYSTEM_MAP.md`
- 或新增 durable spec 細節

### 7. `tdd-ready-check`

回答：

- 現在能不能安全進 TDD
- 主要風險是什麼
- 要用什麼測試策略
- 要走 `gherkin-extraction` 還是 `direct TDD`

這裡的重點不是單選一個 test level，而是定義：

- primary protection layer
- supporting automated layers
- manual validation
- deferred coverage

---

## 實作層

### 8. `gherkin-extraction`

不是所有 slice 都要 Gherkin。

它只用在：

- user-visible behavior
- acceptance-worthy business rule
- 重要 cross-boundary outcome

如果 work 是 helper-level 或 lower-level tests 已足夠，則可以 direct TDD。

### 9. `tdd-workflow`

用選好的測試策略執行：

- Red
- Green
- Refactor

重點：

- Red 不一定從 unit 開始
- 可以從 `unit / integration / feature` 任一層開始，取決於主要風險
- Refactor 要檢查重複知識、職責過載、code/test smells
- 如果發現結構假設錯了，就往上回推，不要硬補 code

### 10. `design-review`

不是只看 tests 綠不綠。

它要檢查：

- slice alignment
- seam alignment
- test strategy alignment
- manual validation 是否完成
- deferred coverage 是否還可見
- 是否需要回寫 `spec / SYSTEM_MAP / design-driver-discovery / goals`

---

## Flow 與 Responsibility

這套流程裡有兩個很容易混的概念：

### 分析視角

例如：

- input/output
- validation
- state transition
- recovery

這些是理解系統的方式。

### 責任單位

例如：

- 誰接住工作
- 誰擁有狀態
- 誰推進流程
- 誰負責補救

這些才是 `SYSTEM_MAP` 主要在畫的東西。

不要把分析視角直接當成責任模塊。

---

## 測試策略原則

測試類型不是按習慣選，而是按風險選。

### `unit`

保護：

- 小邏輯
- helper
- deterministic transformation

### `integration`

保護：

- seam / handoff
- persistence
- adapter
- component coordination

### `feature`

保護：

- user-visible behavior
- acceptance-worthy outcome

### `manual validation`

保護：

- 體感延遲
- UI 可理解性
- 媒體品質
- 暫時還不值得全自動的 happy path

很多 slice 會混搭：

- `unit` 保局部規則
- `integration` 保 seam
- `feature` 保 acceptance
- `manual` 補人類判讀

---

## 回推原則

當 implementation 發現問題時，不要一律在 code 層硬補。

### 回 `goals-discovery`

當：

- 系統要做的事情變了

### 回 `design-driver-discovery`

當：

- 真正的設計壓力改變了

### 回 `system-map`

當：

- ownership 或 seam 假設錯了

### 回 `spec-clarification`

當：

- surface / contract / behavior 還不夠清楚

### 回 `tdd-ready-check`

當：

- 原本的測試策略已經不再合理

---

## 目前保留的主要 skill

| Skill | 作用 |
|---|---|
| `goals-discovery` | 問清楚系統必須做什麼 / 不做什麼 |
| `design-driver-discovery` | 找出真正會主導設計的 flow 與壓力 |
| `system-map` | 畫責任模塊、關鍵 seams、變更導航 |
| `feature-slice` | 把 feature idea 切成可實作的 flow slice |
| `spec-clarification` | 補足 slice 缺的 surface / contract / behavior 清晰度 |
| `tdd-ready-check` | 判斷是否 ready for TDD，並決定測試策略 |
| `gherkin-extraction` | 萃取 acceptance-worthy scenarios |
| `tdd-workflow` | 用既定測試策略執行 Red/Green/Refactor |
| `design-review` | 驗證實作是否仍與 slice 與設計對齊 |

---

## 一句話總結

這套 skill flow 的核心不是「讓 AI 幫你寫更多 code」，而是：

**讓人與 AI 在對的抽象層上協作，並在每次實作後知道問題到底應該往哪一層回推。**
