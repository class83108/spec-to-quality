# Skill 與 Mindset 對照

> 目的：說明新的 skill 架構主要依賴哪些 mindset 原則，避免每個 skill 都籠統地引用整份 mindset 文件。

---

## 核心原則

不是每個 skill 都平均依賴所有 mindset。

比較合理的做法是：

- 保留 `architect-mindset.md` 與 `implementation-mindset.md` 作為共用底座
- 每個 skill 明確知道自己主要在用哪些原則
- skill 內文優先引用「這次真正會用到的部分」

---

## 上游 Discovery Skills

### `goals-discovery`

主要依賴：

- 穩定抽象層
- 問對問題，再談解法

用途：

- 把使用者描述拉回正確抽象層
- 分辨 goal / non-goal / constraint / open question
- 避免 implementation detail 混入 goals

### `design-driver-discovery`

主要依賴：

- 壓力驅動結構
- Traceability
- 問對問題，再談解法

用途：

- 從 flow 中找出真正的 design pressure
- 判斷哪些只是 symptom，哪些會主導設計
- 讓後續邊界與架構決策有壓力來源可追溯

### `system-map`

主要依賴：

- 邊界品質測試
- 責任與 ownership
- Traceability

用途：

- 判斷 responsibility unit 是否切得合理
- 判斷 seam 是否真的是 seam
- 把 design drivers 轉成可變更、可導航的結構地圖

---

## Pre-TDD Skills

### `feature-slice`

主要依賴：

- 正確切 slice，而不是偷帶實作
- Traceability

用途：

- 把 feature idea 切成可實作的 flow slice
- 對齊 goal / design driver / responsibility unit / seam
- 決定最早缺的 clarification layer

### `spec-clarification`

主要依賴：

- testing 前先釐清
- 把規格放在對的抽象層
- 先承認未知，再補清楚

用途：

- 補足 surface / contract / behavior 中缺的清晰度
- 避免在還模糊時就直接進 Gherkin 或 TDD
- 決定哪些 clarification 只留在 work card，哪些值得獨立成長期 spec

### `tdd-ready-check`

主要依賴：

- 主風險優先
- 最低誠實測試層
- 保留 deferred risk 的可見性

用途：

- 判斷是否已經清楚到可以進 Gherkin 或 TDD
- 決定測試策略是 unit / integration / acceptance 的哪種組合
- 缺清晰度時，往正確上游層回推

---

## 後段 Skills

### `gherkin-extraction`

主要依賴：

- 只抽取值得驗收的行為
- acceptance 不等於所有測試

用途：

- 把已經足夠清楚且值得驗收的行為收斂成 Gherkin
- 不替代 slice / clarification / readiness

### `tdd-workflow`

主要依賴：

- Red 應暴露真實不確定性
- Green 只做到剛好通過
- Refactor 要消除 drift 與 overload

用途：

- 將 Gherkin 或既定測試策略落成 TDD 工作流
- 決定測試層級與實作邊界
- 發現衝突時回推到正確上游層

### `design-review`

主要依賴：

- Traceability
- 邊界品質測試
- alignment 比綠燈更重要

用途：

- 驗證這次實作是否仍符合 goals / drivers / map
- 找出需要回寫 discovery 或 spec 的地方

---

## 判斷規則

如果某份 mindset 拿掉後，skill 仍然可以完整執行，代表它只是背景參考。

如果拿掉後：

- skill 的判斷邏輯開始模糊
- 不知道什麼該往上回推
- 不知道哪種細節屬於哪層文件

那它就是該 skill 的核心依據。

---

## 簡表

| Skill | 主要依賴 |
|---|---|
| `goals-discovery` | Stable abstraction, Better questioning |
| `design-driver-discovery` | Pressure shapes structure, Traceability |
| `system-map` | Boundary quality, Ownership, Traceability |
| `feature-slice` | Slice the right thing, Traceability |
| `spec-clarification` | Clarify before testing, Keep abstraction honest |
| `tdd-ready-check` | Main risk first, Lowest honest test layer |
| `gherkin-extraction` | Acceptance-worthy behavior only |
| `tdd-workflow` | Real Red, minimal Green, drift-removing Refactor |
| `design-review` | Alignment, boundary integrity, writeback thinking |
