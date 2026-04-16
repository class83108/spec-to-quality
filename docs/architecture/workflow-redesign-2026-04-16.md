# Workflow 重新設計討論紀錄

> 討論日期：2026-04-16
> 觸發點：mock-interview 專案在跑完 align-internals 後，spec-backlog 只有契約缺口（I-001–I-003），G1–G9 的功能實作沒有任何 finding card。

---

## 核心洞見：骨架 vs. 肉

現有流程只能產生「骨架」型的工作項目，沒有機制產生「肉」型的工作項目。

| 層次 | 定義 | 目前誰產生 |
|------|------|-----------|
| **骨架** | 資料契約、schema、邊界保護 | spec-contract / spec-surface |
| **肉** | 功能邏輯、使用者流程、業務行為 | spec-behavior |

這在**既有系統**（如 video-dubbing）感受不明顯，因為肉已存在，align 找的是缺少保護的骨架。
在**全新系統**（如 mock-interview）問題就會浮現：align 跑完，只有契約缺口進 spec-backlog，G1–G9 的功能實作無處承接。

---

## 正確的建構順序

來源：LinkedIn 架構師的開發方法：

```
Interface（契約）→ Tests（驗收）→ AI 實作
```

對應到本流程：

```
spec-contract/spec-surface（定義骨架契約）→ 骨架 TDD → spec-behavior（定義功能行為）→ 功能 TDD
```

**骨架必須先建**，因為骨架是肉的生長邊界。契約保護 AI 實作不會走歪——填錯了測試就會打回來。

---

## 開卡片的分工

三個 skill 各自負責一種 finding type，每個 skill 只開一種卡：

| Skill | 開的卡片 type | 掃描維度 |
|-------|-------------|---------|
| spec-contract | skeleton | 系統內部資料流 handoff（Dx 優先） |
| spec-surface | skeleton | 對外介面邊界（API/CLI/event shape） |
| spec-behavior | feature | goals 下的使用者流程（Gx 覆蓋） |

finding card（`docs/spec-backlog/{finding-id}.md`）加 `type/tests_path` 欄位，spec-to-gherkin 讀 `type` + `finding-id` 前綴後自動路由 gherkin guide reference 與測試路徑，不需要人記。

---

## 各 Skill 職責調整

### dominant-ops（擴充輸出）

現有輸出：Dx 清單、Anti-patterns、Theory Limits

**新增輸出：Design Implications**

每個 Dx 補一節，說明壓力對技術特性的要求，但不做最終技術決策。

**Agent 的角色是結構化追問，不是做決策。** Agent 對每個 Dx 掃以下維度清單，挑與該 Dx 特性相關的維度問出來：

| Dx 特性 | 追問維度 | 代表問題 |
|---------|---------|---------|
| 高頻讀取 | 快取策略 | Application cache / Redis / CDN？讀多寫少比例？ |
| 高失敗代價 | 可靠性邊界 | 需要 transaction 保證嗎？idempotency 要求？retry + DLQ？ |
| Real-time 回饋 | 通訊協議 | SSE / WebSocket / long-polling？斷線重連語義？ |
| 並存長短任務 | Queue 策略 | 互動型 vs 批次需要隔離 queue + worker？ |
| 大量資料 | 儲存類型 | 關聯型夠嗎？search / time-series / geospatial 專用？ |
| 跨邊界資料流 | 一致性模型 | 強一致還是最終一致？誰是 source of truth？ |
| 外部依賴 | 容錯策略 | Adapter 模式？fallback？timeout 多少？ |
| 人機協作迴圈 | 狀態持久化 | 中間狀態需要持久化嗎？斷點恢復需求？ |

不是全部問——根據 Dx 特性挑相關維度。使用者回答具體選擇，或「還不確定，system-map 再決定」，兩者都合法。Agent 確保使用者**有意識地過一遍**，不替使用者做技術判斷——技術判斷需要領域知識，那是使用者的責任。

```
DOM-2（human coordination loop）
  壓力：高頻、失敗代價高、使用者等待感強
  技術特性需求：即時回饋機制、互動型任務與長任務需要隔離
  → 具體技術選擇留給 system-map
```

---

### system-map（升級為技術決策層）

現有職責：Component Map、Boundary Map、Change Protocol

**新增職責：**

1. **技術棧決策**：消費 dominant-ops 的 Design Implications，做出具體技術選擇，rationale 明確引用 Dx

   ```
   DOM-2 要求即時回饋 → 選 SSE + ASGI（不用 polling），原因：polling 延遲無法滿足 DOM-2 SLA
   DOM-1 + DOM-3 並存 → 選獨立 queue/worker，原因：避免長任務餓死互動型任務
   ```

2. **架構圖**：Component Map 加上技術標註，讓 Boundary Map 的「driven by Dx」延伸到「therefore 選擇 X 技術」

3. **Architecture Decision 欄位**（Boundary Map 擴充）：

   四行格式，技術人員和 AI 都能一眼讀懂，technology 未決定可暫留空：

   ```
   Seam C：Pipeline 編排 → Stage 執行
     driven by:  D2（human coordination，失敗代價高）
     decision:   互動型任務 / 長任務獨立 queue + worker
     rationale:  D2 response time 不能被 D3 長任務餓死
     technology: Celery — dedicated `interactive` + `batch` queues
   ```

---

### spec-contract（重新定位）

**不是**：找出所有 seam 的所有契約缺口

**是**：在資料流路徑上，根據 Dx 優先序，判斷哪些 handoff 現在需要 schema 保護

- 高壓 Dx 路徑上的 handoff → 優先定義 schema
- 低壓或未來才需要的 handoff → 記錄但不急著形式化

掃描單位：**資料流中的 handoff**（不是所有 seam）

---

### spec-surface（重新定位）

**不是**：檢查 user journey 覆蓋、基礎設施容量

**是**：API-first——在實作之前定義對外介面的形狀

- API endpoints（request/response schema）
- CLI 指令（argument、output format）
- 對外事件格式

掃描單位：**系統對外暴露的介面邊界**

---

### spec-to-gherkin（根據 type 路由 reference）

職責：讀 finding card → 確認 type + finding-id 前綴 → 套對應 gherkin guide reference → 覆蓋率分析（使用者確認）→ 寫 Gherkin

| Finding type | Gherkin 錨點 | 場景重點 | 測試路徑 |
|---|---|---|---|
| **skeleton** | Contract reference | 形狀是否被守住（schema validation、error code、boundary guard） | `tests/features/contracts/` |
| **feature** | Feature reference | 使用者行為是否正確（user journey、business rule、state transition） | `tests/features/behaviors/` |

Finding card 的 `type` 欄位是路由依據，不需要新增 skill。覆蓋率分析的六個類別不變，但骨架的 happy path 錨點是「valid schema → passes」，功能的 happy path 錨點是「使用者完成行為 → 系統正確回應」，兩者根本不同。

---

### spec-behavior（新增）

**位置**：
- Design mode：在骨架盤點（spec-contract/spec-surface Design）完成後即可執行
- Verify mode：僅在該行為切片依賴的 `skeleton_deps` 已完成時執行（不需等全部骨架完成）

**輸入**：goals.md + SYSTEM_MAP（資料流 + 使用者流程）+ contracts/surface reports + behavior-report 主表

**職責**：對照使用者流程與 Gx，找出哪些功能行為還沒有 feature finding card，產出功能型 finding card

**掃描單位**：**goal（Gx）下的使用者流程**（不是 seam）

功能 finding card 不需要重新定義 schema（骨架已定義），只需描述：
- 使用者做了什麼
- 系統應該如何回應
- 引用已有的 schema/contract 作為邊界

**不處理 infrastructure**：infrastructure 決策已在 SYSTEM_MAP 的 Architecture Decision 欄位記錄，實作時自然跟著走，不需要 finding card，也不在 spec-behavior 掃描範圍內。

---

## Infrastructure 的歸屬

Infrastructure 決策（queue 隔離、SSE transport、worker 容量）**不是 finding card**，而是 **SYSTEM_MAP 的架構決策**。

```
dominant-ops 揭露風險（Design Implications）
  → system-map 做出架構決策（含技術選擇與 rationale，記錄在 Architecture Decision 欄位）
  → align / 實作根據 SYSTEM_MAP 走，不需要另外開卡
```

架構圖在 system-map 做好後，實作時自然跟著走。infrastructure 的 NFR 驗證（如「queue 隔離真的防止 starvation」）屬於壓力測試 / chaos testing 範疇，不在 TDD 的骨架或功能測試裡。

如果 align 途中發現某個 infrastructure concern 沒有在 SYSTEM_MAP 捕捉到：

- **不開 finding card**
- 升回 Level 2（Discovery Conflict Triage）
- 更新 SYSTEM_MAP 補上架構決策
- align 繼續

---

## 完整修訂後的流程

```
goals-discovery
  ↓
dominant-ops
  → Dx 清單 + Anti-patterns + Theory Limits
  → Design Implications（壓力 → 技術特性需求）
  ↓
system-map
  → 技術棧決策（引用 Dx Design Implications）
  → 架構圖（含技術標註）
  → Boundary Map（含 Architecture Decision 欄位）
  → Change Protocol
  ↓
spec-contract（內部 handoff schema gaps，Dx 優先）
spec-surface（API-first：對外介面 shape gaps）

  ※ 發現 infrastructure gap → 不開 finding → 回 SYSTEM_MAP 補

  ↓
spec-behavior（Design 可先盤點）
  → goals + SYSTEM_MAP + skeleton reports → 找出未覆蓋的功能行為
  ↓
骨架 TDD / 功能 TDD（可依切片依賴交錯推進）
  → spec-behavior Verify 僅在該切片 `skeleton_deps` 已完成時開 feature card
  ↓
design-review + release-gate
  ↓
docs-governance（同步 delivery-map + index 狀態）
```

---

## Finding Card 分類

| Type | 來源 | 描述 | Gherkin reference | 測試路徑 |
|------|------|------|-------------------|---------|
| **skeleton** | spec-contract / spec-surface | 契約/schema/API shape 缺口 | Contract reference | `tests/features/contracts/` |
| **feature** | spec-behavior | 功能行為/使用者流程缺口 | Feature reference | `tests/features/behaviors/` |
| ~~infrastructure~~ | ~~—~~ | ~~不開卡，直接更新 SYSTEM_MAP~~ | ~~—~~ | ~~—~~ |

### Finding Card 新增欄位

```markdown
| 欄位        | 說明                                             |
|-------------|--------------------------------------------------|
| type        | skeleton / feature                               |
| tests_path  | tests/features/contracts/ 或 tests/features/behaviors/ |
```

`type` + `finding-id` 前綴讓 spec-to-gherkin 自動路由 gherkin guide reference；`tests_path` 讓 .feature 檔案落在正確位置，不靠約定俗成記憶。

### 測試資料夾結構

```
tests/
  features/
    contracts/        ← skeleton finding 的 .feature（契約驗證）
    behaviors/        ← feature finding 的 .feature（行為驗收）
  step_defs/
    contracts/
    behaviors/
```

**分開執行的好處：**
- `pytest tests/features/contracts/` — 快速跑所有契約驗證，適合改 schema 後即時確認
- `pytest tests/features/behaviors/` — 跑所有功能驗收，適合整合階段
- 骨架壞（contracts 紅）vs 功能壞（behaviors 紅）訊號分開，定位問題更快

---

## MVP 迭代情境

MVP 完成後開始迭代（超越重構與優化、新增功能），**不需要從頭跑 discovery**。

Discovery 文件（goals、dominant-ops、SYSTEM_MAP）是**活的文件，不是快照**。正確做法是 **delta update from the earliest impacted layer**：

| 迭代性質 | 最早影響層 | 動作 |
|---------|-----------|------|
| 新功能完全在現有架構內 | spec-backlog | start-feature → spec-behavior → 實作 |
| 需要新 boundary 或 contract | SYSTEM_MAP | 更新 SYSTEM_MAP → align 新邊界 → 實作 |
| 影響 dominant-ops 壓力分佈 | dominant-ops | 更新 dominant-ops → 更新 SYSTEM_MAP → align → 實作 |
| 新增全新能力，goals 要加 Gx | goals | append 新 Gx → 視情況更新 dominant-ops → 往下 delta |

關鍵原則：找到最早被影響的那一層，從那裡往下更新，下游文件跟著 delta。邏輯與 Discovery Conflict Triage 相同，方向換成「主動新增」。

---

## 實作狀態更新（2026-04-16）

| 工作 | 狀態 |
|------|------|
| `spec-contract`：補齊掃描概念、與其他 skill 的分工、Design/Verify 輸出契約 | 已完成 |
| `spec-surface`：補齊 Design mode 主表同步、開卡時機收斂到 Verify | 已完成 |
| `spec-behavior`：建立/校正 skill（切片依賴 `skeleton_deps`、`type: feature`、`tests_path: tests/features/behaviors/`） | 已完成 |
| contracts/surface/behavior report templates 對齊欄位規則與 notes guardrail | 已完成 |
| Contract reference：`references/contract-gherkin-guide.md`（骨架錨點、schema validation 場景結構） | 已完成 |
| Surface reference：`references/surface-gherkin-guide.md`（外部介面形狀、error contract 場景結構） | 已完成 |
| Feature reference：`references/feature-gherkin-guide.md`（使用者行為錨點、business rule 場景結構） | 已完成 |
| `system-map`：技術棧決策與 Architecture Decision（含 AP/Dx 追溯）對齊強化 | 已完成 |
| `dominant-ops`：Design Implications 與 APx/Dx 追溯引導強化 | 已完成 |
| docs-governance：解決 alignment report lifecycle 矛盾（append-only vs. move-to-archive） | 待處理 |
