# insight-to-quality

一套把 discovery、system design、system mapping、feature slicing、spec clarification、TDD、review 串成同一條鏈的開發 skill。

重點不是產生很多文件，而是讓人與 AI 在正確抽象層上合作：

- 先把系統輪廓講清楚
- 再把關鍵設計決策與 trade-off 講清楚
- 再把軟體結構、ownership、seams 講清楚
- 再切 feature slice，往 spec、TDD、review 推進

---

## 主流程

1. `discovery`
2. `system-design`
3. `system-map`
4. `feature-slice`
5. `spec-clarification`
6. `tdd-ready-check`
7. `gherkin-extraction` 或直接 `tdd-workflow`
8. `design-review`

這不是 rigid waterfall。
如果問題屬於更上游的層，就往上回推，不要用 local code 硬補。

---

## 三份上游文檔

### `discovery.md`

回答：

- 我們在解什麼問題
- 第一版系統要支撐什麼
- top-level API / interaction shape 是什麼
- baseline flow / high-level design 大致長什麼樣
- goals / non-goals / constraints / NFRs 是什麼

這一層的重點是：先把第一版系統輪廓勾出來。

### `system-design.md`

回答：

- 在目前 baseline 下，最重要的幾個設計決策是什麼
- 為什麼這樣選，不是那樣選
- alternatives 是什麼
- trade-offs / risks 是什麼
- 哪些題目值得下一步 deep dive

這一層的重點是：把「為什麼這樣做」講清楚。

### `SYSTEM_MAP.md`

回答：

- 系統裡有哪些主要 responsibility units
- 哪些核心 state / entity / workflow truth 由誰擁有
- 哪些 seams / handoffs 是關鍵邊界
- 改某件事時應該先看哪裡

這一層的重點是：把已知輪廓與決策，展開成可開發、可維護的軟體結構地圖。

---

## Feature 到 Implementation

### `feature-slice`

把 feature idea 對齊：

- `discovery.md`
- `system-design.md`
- `SYSTEM_MAP.md`

然後切出第一個可實作 slice，建立：

- `docs/features/<feature-slug>/work-card.md`

### `spec-clarification`

只補這個 slice 真正缺的清晰度：

- `surface`
- `contract`
- `behavior`

### `tdd-ready-check`

判斷：

- 現在能不能安全進 TDD
- 主要風險是什麼
- 該用哪個 test strategy
- 要走 Gherkin 還是 direct TDD

### `gherkin-extraction`

只在 acceptance-worthy behavior 值得保護時使用。

### `tdd-workflow`

依選好的策略執行：

- Red
- Green
- Refactor

### `design-review`

檢查：

- slice 是否仍對齊
- seams / ownership 是否 drift
- test strategy 是否真的被遵守
- 是否需要回寫上游文檔

---

## Shared Anchor

feature implementation 的共享協調文件是：

- `docs/features/<feature-slug>/work-card.md`

後續 skill 應盡量沿用這份 work card，而不是散落成許多無關筆記。

---

## 三份 Reference

- `references/design-decision-mindset.md`
  - 給 `discovery` / `system-design`
- `references/architect-mindset.md`
  - 給 `system-map`
- `references/implementation-mindset.md`
  - 給 `spec-clarification` 之後的執行鏈

---

## 核心原則

- 不要在 system shape 還不清楚時直接進 code
- 不要在 key decisions 還沒說清楚時直接切結構
- 不要在 ownership / seams 還不清楚時直接寫 spec 或測試
- 不要假設每個 slice 都需要同一種 test level
- 不要把上游問題藏進 local implementation

這套 plugin 的目的，是讓 discovery、design、structure、slice、spec、test、review 彼此連得起來，而不是各自獨立。 
