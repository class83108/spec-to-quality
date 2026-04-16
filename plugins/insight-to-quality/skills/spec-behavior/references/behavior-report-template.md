# Behavior Report Template

> 用途：定義 `docs/behaviors/behavior-report.md` 的固定結構，供 `spec-behavior` skill 在 Design/Verify 兩種模式下輸出一致報告。
> 語言：繁體中文（技術名詞可保留英文）。

## File Structure

`behavior-report.md` 必須固定包含以下區段，順序不可調換：

1. `# Behavior Report`
2. `## Baseline`
3. `## Behavior Slices Table`

---

## Baseline Section Spec

### Header 欄位（必填）

```md
## Baseline

- Date: YYYY-MM-DD
- Source Docs: goals.md, dominant-ops.md, SYSTEM_MAP.md, docs/contracts/contracts-report.md, docs/surface/surface-report.md
- Scope: <本次掃描範圍摘要>
- Summary:
  - total_goals: <number>
  - total_slices: <number>
  - draft: <number>
  - ready: <number>
  - in-progress: <number>
  - done: <number>
  - blocked: <number>
  - skeleton_deps_unresolved: <number>
```

---

## Behavior Slices Table

### Design mode（粗粒度，不定義業務規則或 state machine）

欄位名稱固定如下：

| finding-id | Gx | flow-note | skeleton_deps | status | order | order_reason |
|---|---|---|---|---|---|---|
| behavior-001 | G1 | 使用者完成認證，系統建立 session | surface-001 | draft | — | — |
| behavior-002 | G2 | 使用者在面試中提交回答，系統即時評分 | surface-002,contract-001 | ready | 1 | R1 Goal criticality：G2 核心互動流程，前置依賴 surface-002 已完成 |
| behavior-003 | G4 | 使用者查詢歷史面試結果 | surface-003 | draft | — | — |

### Verify mode（細化後補入欄位）

Verify mode 針對選定切片細化後，在原列補入以下欄位：

| finding-id | Gx | slice | flow-note | serves | related | boundary | skeleton_deps | priority | status | order | order_reason | type | tests_path | card | notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| behavior-002 | G2 | behavior:G2:submit-answer | 使用者提交回答，系統評分並回傳即時結果 | G2 | D1 | InterviewSession | surface-002,contract-001 | D1 | ready | 1 | R1 Goal criticality：G2 核心互動流程 | feature | tests/features/behaviors/ | docs/spec-backlog/behavior-002.md | - |

欄位規則：
- `finding-id`：`behavior-xxx`，穩定後不重用。
- `Gx`：來自 goals.md 的目標編號，是此切片的主要追蹤錨點。
- `slice`：Verify mode 細化後填入 `behavior:{Gx}:{flow-name}`；Design mode 可留空。
- `flow-note`：一句話描述使用者動作與系統應達成的結果；Design mode 使用，Verify mode 可保留或更新。
- `skeleton_deps`：依賴的 skeleton finding-id，以逗號分隔（如 `surface-001,contract-002`）；僅列必要依賴，無依賴可填 `-`。
- `status`：僅能為 `draft` / `ready` / `in-progress` / `done` / `blocked`。
- `order`：只對 `ready` 填整數（1..n），其他狀態可用 `—`。
- `order_reason`：只對 `ready` 必填，一句話說明排序依據（需可追溯）。
- `type` 固定為 `feature`。
- `tests_path` 固定為 `tests/features/behaviors/`。
- `notes`：若為 schema concern，標註 `dependency block: skeleton not done`；若為 infrastructure concern，標註 `escalate to SYSTEM_MAP`。

---

## Update Rules

- Design mode：建立或重寫 `Baseline` 與主表（粗粒度盤點所有 Gx 行為切片）。
- Verify mode：更新主表對應列（補 `slice`/`boundary`/`status`/`order`/`card` 等欄位），不需 append iteration 區塊。
- 若發現 schema concern（skeleton 未完成），在 `notes` 記錄 `dependency block: skeleton not done`，不得建立 feature finding card。
- 若發現 infrastructure concern（transport/queue/capacity/topology），在 `notes` 記錄 `escalate to SYSTEM_MAP`，不得新增 finding card。
