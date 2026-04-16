# Surface Report Template

> 用途：定義 `docs/surface/surface-report.md` 的固定結構，供 `spec-surface` skill 在 Design/Verify 兩種模式下輸出一致報告。
> 語言：繁體中文（技術名詞可保留英文）。

## File Structure

`surface-report.md` 必須固定包含以下區段，順序不可調換：

1. `# Surface Report`
2. `## Baseline`
3. `## Interface Boundaries Table`

---

## Baseline Section Spec

### Header 欄位（必填）

```md
## Baseline

- Date: YYYY-MM-DD
- Source Docs: goals.md, dominant-ops.md, SYSTEM_MAP.md
- Scope: <本次掃描範圍摘要>
- Summary:
  - total_boundaries: <number>
  - draft: <number>
  - ready: <number>
  - in-progress: <number>
  - done: <number>
  - blocked: <number>
```

---

## Interface Boundaries Table

### Design mode（粗粒度，不定義 schema 或路由）

欄位名稱固定如下：

| finding-id | boundary-area | scope-note | serves | related | status | order | order_reason |
|---|---|---|---|---|---|---|---|
| surface-001 | 認證與 session 管理 | 外部使用者需要建立與結束 session | G1 | D2 | draft | — | — |
| surface-002 | 面試進行中的狀態回報 | 外部使用者需要接收即時回應 | G2,G3 | D1 | ready | 1 | R1 依賴優先：session 建立是 G2 所有操作的前置 |
| surface-003 | 面試結果查詢 | 外部使用者需要取得歷史評分 | G4 | D3 | draft | — | — |

### Verify mode（細化後補入欄位）

Verify mode 針對選定 boundary-area 細化後，在原列補入以下欄位：

| finding-id | boundary-area | slice | serves | related | boundary | priority | status | order | order_reason | type | tests_path | card | notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| surface-002 | 面試進行中的狀態回報 | interface:面試進行中:response-stream | G2,G3 | D1 | InterviewSession | D1 | ready | 1 | R1 依賴優先：session 建立是 G2 前置 | skeleton | tests/features/contracts/ | docs/spec-backlog/surface-002.md | - |

欄位規則：
- `finding-id`：`surface-xxx`，穩定後不重用。
- `boundary-area`：Design mode 命名的粗粒度區域，Verify mode 不更改此欄。
- `slice`：Verify mode 細化後填入 `interface:{boundary-area}:{operation}`；Design mode 可留空。
- `status`：僅能為 `draft` / `ready` / `in-progress` / `done` / `blocked`。
- `order`：只對 `ready` 填整數（1..n），其他狀態可用 `—`。
- `order_reason`：只對 `ready` 必填，一句話說明排序依據（需可追溯）。
- `type` 固定為 `skeleton`。
- `tests_path` 固定為 `tests/features/contracts/`。
- `notes`：若為 infrastructure concern，標註 `escalate to SYSTEM_MAP`；若為 tech stack concern，標註 `dependency block: wait for system-map decision`。

---

## Update Rules

- Design mode：建立或重寫 `Baseline` 與主表（粗粒度盤點所有邊界區域）。
- Verify mode：更新主表對應列（補 `slice`/`boundary`/`status`/`order`/`card` 等欄位），不需 append iteration 區塊。
- 若發現 infrastructure concern（transport/queue/capacity/topology），在 `notes` 記錄 `escalate to SYSTEM_MAP`，不得新增 finding card。
- 若發現 tech stack concern（具體技術選擇），在 `notes` 記錄 `dependency block: wait for system-map decision`，不得新增 finding card。
