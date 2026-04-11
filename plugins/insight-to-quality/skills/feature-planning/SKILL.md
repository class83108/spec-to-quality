---
name: ec:feature-planning
description: >
  Discovery 完成後、OpenSpec 建立前的橋接步驟。讀取 SYSTEM_MAP 的 Current State 與 gap，
  比對已有的 feature plans 和 OpenSpec changes，決定下一個要實作的 feature，引導 error handling
  策略決定，產出 docs/feature-plans/{name}.md，並更新 SYSTEM_MAP Current State。
  當 discovery 完成、使用者要決定下一個 feature 要實作什麼、或說「要開始規劃實作」時觸發。
  Do NOT use for: discovery 尚未完成時（先完成 ec:align-internals 和 ec:align-surface）、
  或已有 feature plan 要直接進入實作時（直接用 ec:feature-coverage）。
---

# Feature Planning

你正在進入 feature planning 階段。這個 skill 的職責是：從 discovery 的 gap 決定要實作什麼，
並在 OpenSpec 建立之前把關鍵設計決定固定下來。

**在開始之前，先讀：**
- `references/architect-mindset.md`（traceability：確認 feature 有 goal 根基）
- `references/implementation-mindset.md`（Part 1：引導 Three Decisions 的框架）

## 前置條件

以下文件必須全部存在才能進入此 skill：
- `goals.md`
- `dominant-ops.md`
- `SYSTEM_MAP.md`
- 至少一份 align report（ec:align-internals 或 ec:align-surface 的產出）

如果任一文件不存在，停止並告知：「feature planning 需要完整的 discovery 產出才能進行。
沒有 discovery 就進入實作，後續每個 skill 都會缺乏依據。請先完成 [缺少的 skill]。」

## 流程

### Step 1: 讀取 Discovery 產出

讀以下文件，提取關鍵資訊：

**goals.md** → 所有 Gx 與其描述

**dominant-ops.md** → 所有 Dx 與其 anti-patterns，記下每個 anti-pattern 的完整 ID（AP1、AP2...）

**SYSTEM_MAP.md** → 特別關注：
- `Current State.Gaps`：目前已知的 gap 清單（有哪些已被 feature plan 部分覆蓋）
- `Current State.In-flight`：目前進行中的工作
- `Boundary Map`：各 boundary 的 contract 狀態

**Align reports** → 提取 contract 缺口（align-internals）和介面缺口（align-surface）

**已有的 feature plans**：掃描 `docs/feature-plans/` 目錄，了解哪些 gap 已被覆蓋或部分覆蓋

### Step 2: 比對已覆蓋的 gap，列出候選清單

將 SYSTEM_MAP 的 Gaps 與已有的 feature plans 交叉比對：

```
SYSTEM_MAP Gaps
    ↓ 比對
docs/feature-plans/ 的已有文件
openspec/changes/ 的已有 change
    ↓
未覆蓋 → 直接列為候選
部分覆蓋 → 列為候選，標注「接續 [已有 feature plan]，第 N 個 feature」
已完全覆蓋 → 排除
```

產出候選清單，每項標注：
- 對應的 SYSTEM_MAP gap 或 align report 缺口
- 服務的 Gx
- 是否為接續既有 feature plan

```
候選 Feature：
1. [feature-name] — 解決 [gap 描述]，服務 G1, G2
2. [feature-name] — 接續 task-queue.md（第 2/3 個），服務 G1
...
```

詢問使用者：「要先規劃哪一個？」

### Step 3: 確認 Traceability 與 SYSTEM_MAP 更新必要性

使用者選定後，確認兩件事：

**Traceability：**
- 這個 feature 服務哪些 Gx？（必須至少對應一個 goal）
- 與哪些 Dx 相關？（決定後續引用哪些 anti-patterns）
- 觸碰哪些 SYSTEM_MAP boundary？

如果 feature 無法對應任何 Gx → 停止：「這個 feature 目前沒有 goal 根基。建議先確認它服務哪個 goal，或將其加入 goals.md，再繼續規劃。」

**SYSTEM_MAP 是否需要更新：**

問使用者：「這個 gap 需要多個 feature 是因為：
A) 對系統結構有新的理解（boundary 需要調整、component 需要拆分）→ 先更新 SYSTEM_MAP 的 Boundary Map 或 Component Map，再繼續
B) 工程上分批實作（系統結構不變，只是分階段做）→ 不更新結構，在 feature plan 和 SYSTEM_MAP Current State 記錄分批資訊」

如果是 A → 引導使用者更新 SYSTEM_MAP 相關區段後，再繼續 Step 4。

### Step 4: 引導 Error Handling Strategy

讀 `implementation-mindset.md` Part 1，逐一引導 Three Decisions。用 discovery 的上下文驅動引導，不是泛泛詢問。

**Decision 1 — Catch Boundary**

結合 SYSTEM_MAP 的 boundary 資訊：
「這個 feature 觸碰了 [boundary A]。出錯時，呼叫方需要知道這個層次的錯誤，還是這個元件應該在內部處理後再回應？」

**Decision 2 — Error Taxonomy**

結合 dominant-ops 的 anti-patterns：
「根據 [Dx] 的 [APx]，[失敗情況] 是需要防範的。這屬於業務預期的 domain error（例如找不到資料、重複建立），還是外部依賴失敗的 infrastructure error（DB 連不到、API timeout）？除了 anti-patterns 提到的，這個 feature 還有哪些預期失敗情況？」

**Decision 3 — Recovery Strategy**

「遇到這些 domain errors 時，回傳結構化錯誤給呼叫方？遇到 infrastructure errors 時，fail fast 讓上層處理，還是有重試或降級策略？」

### Step 5: 提取 Constraints

**Anti-patterns**：從 dominant-ops.md 提取與此 feature 相關的 Dx anti-patterns，帶完整 ID。每條說明「這條限制這個 feature 的什麼行為」。

**Boundary Rules**：從 SYSTEM_MAP.md 的 Boundary Map 提取這個 feature 觸碰的 boundary 規則。說明「這個 feature 不可以跨越哪個 boundary」或「資料只能從哪個方向流」。

### Step 6: 產出 Feature Plan

在 `docs/feature-plans/` 下建立 `{feature-name}.md`，命名用 kebab-case 與後續 OpenSpec change 同名。

```markdown
# Feature Plan — {feature-name}

## Source
- Serves: G1, G3
- SYSTEM_MAP gap: [gap 描述]
- Coverage: 完整覆蓋 / 部分覆蓋（第 N/M 個 feature，接續 [前一個 feature-plan]）

## Error Handling Strategy
- Catch boundary: [boundary only / per-layer / selective: 哪些 exceptions]
- Domain errors: [列表，例如 TaskNotFound, DuplicateTask]
- Infrastructure errors: [fail fast / retry N times / degrade to X]

## Constraints

### Anti-patterns (from dominant-ops)
- AP1 (D1): [anti-pattern 描述] — 對此 feature 的影響：[說明]
- AP2 (D2): [anti-pattern 描述] — 對此 feature 的影響：[說明]

### Boundary Rules (from SYSTEM_MAP)
- [這個 feature 不可以跨越 boundary X，原因：...]
- [資料只能從 A 流向 B，不可反向]

## Integration Test Gaps
<!-- 由 ec:tdd-workflow Verification Ledger 完成後填入，初始為空 -->
<!-- 記錄需要整合測試但暫時不做的缺口，pre-complete 會讀取此區段確認追蹤狀態 -->
```

### Step 7: 更新 SYSTEM_MAP Current State

將 feature plan 的進度反映到 SYSTEM_MAP：

```markdown
## Current State
- **In-flight**: [feature-name]（docs/feature-plans/{feature-name}.md）
- **Gaps**:
  - [gap 描述]：部分覆蓋（第 1/3 個 feature，in-flight）/ 已規劃（docs/feature-plans/...）
```

### Step 8: 確認並交接

展示 feature plan 給使用者確認：
1. Traceability 正確（Gx 對應、gap 描述）
2. Error handling strategy 符合意圖
3. Constraints 沒有遺漏重要的 anti-patterns 或 boundary rules

確認後告知：
「Feature plan 已建立於 `docs/feature-plans/{feature-name}.md`，SYSTEM_MAP Current State 已更新。

接下來：
1. 用 `opsx:apply` 建立 OpenSpec change，在 spec.md body 加上 `**Serves:** G1, G3`
2. design.md 的 Decisions 區段引用此 feature plan 的 Error Handling Strategy
3. OpenSpec 建立完成後進入 `ec:feature-coverage`」

## Examples

### Example 1: 正常流程

使用者說：「discovery 完成了，要開始規劃實作」

1. 讀 discovery 文件，發現 3 個 gap：TaskQueue contract 未實作、Worker boundary 未定義、API surface 缺少狀態查詢
2. 掃描 `docs/feature-plans/`，目前是空的
3. 列出 3 個候選 → 使用者選「TaskQueue 實作」
4. 確認 Serves G1, G3；問是否需要更新 SYSTEM_MAP → 工程分批，不更新結構
5. 引導 Three Decisions：
   - Catch boundary: boundary only
   - Domain errors: TaskNotFound, QueueFull
   - Infrastructure errors: fail fast
6. 提取 AP1（任務未完成不接新任務）、AP2（狀態必須落地）
7. 產出 `docs/feature-plans/task-queue.md`
8. 更新 SYSTEM_MAP Current State
9. 告知下一步

### Example 2: 接續既有 Feature Plan

SYSTEM_MAP Gaps 顯示「TaskQueue 實作」已部分覆蓋（第 1/3 個 feature 完成）。

正確行為：候選清單列「task-queue-worker（接續 task-queue.md，第 2/3 個）」，feature plan 的 Coverage 欄標注「部分覆蓋（第 2/3 個，接續 task-queue.md）」。

### Example 3: 發現需要更新 SYSTEM_MAP 結構

選定 feature 後發現，原本 SYSTEM_MAP 把 A 和 B 畫在同一個 boundary，但實作規劃後發現需要拆成兩個獨立 boundary。

正確行為：停下來告知「這個發現代表對系統結構有新的理解，建議先更新 SYSTEM_MAP 的 Boundary Map（把 Seam A 拆成 Seam A1 和 Seam A2），再繼續 feature planning。這樣後續的 feature plan 和 OpenSpec 才有正確的 boundary 依據。」

### Example 4: Discovery 不完整

只有 goals.md，沒有 SYSTEM_MAP。

正確行為：「feature planning 需要完整的 discovery 產出。目前缺少 SYSTEM_MAP.md，請先完成 ec:system-map、ec:align-internals、ec:align-surface，再回來做 feature planning。」
