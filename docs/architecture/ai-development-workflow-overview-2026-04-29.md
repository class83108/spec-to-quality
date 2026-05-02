# AI 時代下的專案開發流程總覽

> 討論日期：2026-04-29
> 目的：整理一套不依賴特定工具、可逐步由多個 skill / agent 協作修正的專案開發流程骨架。

---

## 核心觀點

這套流程的重點不是讓 AI 更快寫 code，而是讓人和 AI 在正確的抽象層協作：

- 先定義問題，再定義壓力，再切系統，再細化規格
- 前端、後端、內部資料流一起看，不分開各自腦補
- flow 不等於畫面流程；沒有 UI 的 internal flow 一樣需要設計
- spec 的價值是避免邊界與規則被誤解，不是製造大量文件
- 測試先做風險排序，不追求表面覆蓋率

---

## 我們要管理的幾種 Flow

### 1. User Flow

使用者為了完成某件事所走的步驟。

例：
- 註冊與登入
- 建立訂單
- 送出審核請求

### 2. System Flow

系統內部為了完成某件事，資料、狀態、命令、事件如何流動。

例：
- webhook 進來後驗證、轉換、入庫、觸發後續處理
- 建立訂單後進行庫存保留、付款授權、狀態更新

### 3. Operational Flow

非互動性的背景流程、排程、補償、重試、告警、人工介入流程。

例：
- scheduled job 掃描失敗狀態並重試
- dead letter queue 進入人工處理
- 非同步任務超時後的補償流程

結論：

- 有沒有畫面，不決定它是不是 flow
- 只要有狀態變化、責任交接、失敗風險，就值得被當成 flow 設計

---

## 整體分層流程

### Phase 1. 問題定義

回答：
- 這個系統要解什麼問題
- 第一版一定要做到什麼
- 現階段明確不做什麼
- 哪些失敗最不能接受
- 有哪些約束已經存在

主要產物：
- `goals.md`

建議 skill：
- `insight-to-quality:goals-discovery`

這一層不討論 service 實作細節，最多只記錄 constraints。

---

### Phase 2. 壓力與流程辨識

回答：
- 哪些 operation 最常發生
- 哪些 operation 最貴
- 哪些 operation 最怕錯
- 哪些 flow 是主要價值路徑
- 哪些 flow 是高風險 internal / operational flow

主要產物：
- `dominant-ops.md`
- `docs/flows/README.md`
- `docs/flows/Fxx-<flow-name>.md`

建議 skill：
- `insight-to-quality:dominant-ops`

建議方法：
- event storming
- flow inventory

這一層的目的是找出壓力與 flow，不是做最終技術決策。

---

### Phase 3. 系統切分與架構定形

回答：
- 需要哪些責任單位
- 哪些是後端 service / module
- 哪些是前端互動責任 / state responsibility
- 哪些是 orchestration，哪些是核心 domain logic
- 哪些狀態歸誰擁有
- 哪些邊界需要明確 seam

主要產物：
- `SYSTEM_MAP.md`

建議 skill：
- `insight-to-quality:system-map`

這一層才開始談：
- component map
- boundary map
- architecture decisions
- tech stack decisions

這一層決定的是責任與邊界，不是 class / table 細節。

---

### Phase 4. 高風險規格化

回答：
- 哪些地方需要明確定義外部互動面
- 哪些 handoff 需要 schema / contract 保護
- 哪些規則與狀態轉移最容易被寫錯

主要產物：
- `specs/<feature>/spec.md`
或
- `specs/<feature>/surface.md`
- `specs/<feature>/contracts.md`
- `specs/<feature>/behavior.md`

建議 skills：
- `insight-to-quality:start-feature`
- `insight-to-quality:spec-surface`
- `insight-to-quality:spec-contract`
- `insight-to-quality:spec-behavior`

原則：
- 不是整個系統一起寫滿 spec
- 只先細化高風險 flow、邊界、規則

---

### Phase 5. 驗收規格與測試設計

回答：
- 哪些行為值得寫成 acceptance scenarios
- 哪些規則必須被精準驗證
- 哪些 internal flow 比較適合 contract / integration test

主要產物：
- `features/<feature>/<finding-id>.feature`

建議 skill：
- `insight-to-quality:spec-to-gherkin`

原則：
- Gherkin 只抓高價值 scenario
- 不把所有資料搬運或技術流程都寫成 Gherkin

---

### Phase 6. 實作

回答：
- 具體如何落地到 code
- DB / queue / cache / retry / state library 怎麼選
- 如何讓實作持續追溯到前面的 goals、dominant ops、system map、spec

主要產物：
- production code
- tests
- integration adapters
- migrations / schemas

建議 skill：
- `insight-to-quality:tdd-workflow`

---

### Phase 7. 驗證與回寫

回答：
- 這次實作有沒有破壞既有邊界
- 哪些決策只是暫時方案
- 哪些教訓之後必須讓 AI 與團隊看得到

主要產物更新：
- `SYSTEM_MAP.md`
- `specs/<feature>/notes.md`
- 必要時回寫 `goals.md` / `dominant-ops.md`

建議 skills：
- `insight-to-quality:design-review`
- `insight-to-quality:docs-governance`

---

## 前端、後端、內部資料流要一起看

同一個 feature 至少同時有三個觀點：

- 使用者互動面
- 系統行為面
- 資料與狀態面

設計時要同步對齊：

- 前端要如何讓使用者完成任務
- 後端要如何保證規則與狀態正確
- 內部 flow 要如何在失敗、重試、非同步情況下維持一致性

判斷 service 與 API 是否切得好，不只看後端內部結構，還要看：

- 一個 flow 要不要跨太多 service 才跑得完
- 前端是否需要自己補很多 business rule
- API 回傳形狀是否貼近互動需求
- loading / retry / duplicate submit 是否可自然處理

---

## Spec 的三個視角

不一定要硬拆成三份檔案，但建議保留三個視角。

### Surface

回答：
- 外界怎麼碰到系統
- 有哪些 entry points
- 對外成功與失敗如何表現

適用：
- UI flow
- API endpoint
- webhook entry
- CLI / admin action

### Contract

回答：
- 邊界兩側如何交接資料
- 什麼欄位必填
- 什麼欄位語意不能混
- 哪些 schema / payload 需要穩定保護

適用：
- request / response
- internal DTO / payload
- event schema
- state ownership handoff

### Behavior

回答：
- 哪些規則不能寫錯
- 狀態如何轉移
- 失敗、重試、補償如何定義
- 哪些 business assertions 應作為驗收依據

適用：
- business rules
- state transitions
- orchestration rules
- failure semantics

結論：

- 小型 feature：可用一份 `spec.md`，內部分三個 section
- 複雜 feature：拆成 `surface / contracts / behavior`
- 重要的是三種問題不要混在一起

---

## Event Storming 與 Gherkin 的位置

### Event Storming

適合用來：
- 展開業務事件與狀態變化
- 看出 internal flow 與邊界壓力
- 找出責任切分與非同步/補償問題

不適合直接當最終長期文件；比較適合作為流程分析輸入，再整理進 flow docs 與 system map。

### Gherkin

適合用來：
- 定義核心 user flow 的驗收條件
- 定義高風險 business rules
- 定義關鍵 state transitions 的可觀察結果

不適合用來：
- 覆蓋所有低價值 CRUD
- 表達純技術性的資料搬運流程
- 替代 contract / architecture thinking

結論：

- `event storming + gherkin` 是好骨架
- 但中間還需要 `responsibility mapping` 與 `interaction / contract` 兩層

---

## 何時知道抽象不好

抽象不好的常見訊號：

- goals 太模糊，導致不同人理解完全不同
- event 命名只剩 CRUD，沒有業務語意
- service 邊界在圖上很漂亮，但真實 flow 很卡
- API technically 可用，但前端互動非常痛苦
- 很難回答誰擁有哪個狀態、哪條規則
- scenario 很多，但說不出哪裡風險最高

這類問題通常要往上回修，不是直接補實作。

---

## 何時知道實作有誤

實作有誤的常見訊號：

- 測試要依賴大量 mock 才能成立
- 一改 flow 就牽動很多底層
- contract 常常需要補例外處理
- retry / timeout / async 狀態一進來就變亂
- feature 雖然能跑，但團隊無法一致解釋它怎麼運作

這類問題通常表示設計問題已經落地成 code。

---

## 建議的最小專案文件骨架

```text
/goals.md
/dominant-ops.md
/SYSTEM_MAP.md

/docs/flows/README.md
/docs/flows/F01-<flow-name>.md
/docs/flows/F02-<flow-name>.md

/specs/<feature>/spec.md
或
/specs/<feature>/surface.md
/specs/<feature>/contracts.md
/specs/<feature>/behavior.md
/specs/<feature>/notes.md

/features/<feature>/<finding-id>.feature
```

如果專案剛開始，最重要的三個根文件是：

- `goals.md`
- `dominant-ops.md`
- `SYSTEM_MAP.md`

後面的 `specs/` 與 `features/` 可以隨 feature 漸進長出來。

---

## 建議的 Skill 使用順序

### 專案啟動期

1. `insight-to-quality:goals-discovery`
2. `insight-to-quality:dominant-ops`
3. `insight-to-quality:system-map`

### 單一 Feature 進場

1. `insight-to-quality:start-feature`
2. `insight-to-quality:spec-surface`
3. `insight-to-quality:spec-contract`
4. `insight-to-quality:spec-behavior`
5. `insight-to-quality:spec-to-gherkin`
6. `insight-to-quality:tdd-workflow`
7. `insight-to-quality:design-review`

### 文件維護

1. `insight-to-quality:docs-governance`

---

## 最後的原則

- 先問清楚問題，再進設計
- 先找出壓力，再談技術
- 先定責任與邊界，再談實作細節
- 只對高風險區域做高強度 spec 與測試
- 當 code 很痛苦時，不要只補 code，要先判斷是哪一層抽象壞掉
- 不追求一次把流程定死，而是讓 artifact 支撐持續回修
