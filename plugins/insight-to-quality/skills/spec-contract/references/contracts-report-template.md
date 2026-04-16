# Contracts Report Template

> 用途：定義 `docs/contracts/contracts-report.md` 的固定結構，供 `spec-contract` skill 在 Design/Verify 兩種模式下輸出一致報告。
> 語言：繁體中文（技術名詞可保留英文）。

## File Structure

`contracts-report.md` 必須固定包含以下區段，順序不可調換：

1. `# Contracts Report`
2. `## Baseline`
3. `## Handoff Contracts Table`

---

## Baseline Section Spec

### Header 欄位（必填）

```md
## Baseline

- Date: YYYY-MM-DD
- Source Docs: goals.md, dominant-ops.md, SYSTEM_MAP.md
- Scope: <本次掃描範圍摘要>
- Summary:
  - total_handoffs: <number>
  - draft: <number>
  - ready: <number>
  - in-progress: <number>
  - done: <number>
  - blocked: <number>
```

### Handoff Contracts Table（必填）

欄位名稱固定如下：

| handoff | from | to | related Dx/APx | status | order | order_reason | finding-id | type | tests_path | card | notes |
|---|---|---|---|---|---|---|---|---|---|---|---|
| handoff:Pipeline→StageRunner | Pipeline | StageRunner | DOM-2, AP-1 | ready | 1 | R1 依賴優先：此交接點為下游三個 handoff 前置。 | contract-001 | skeleton | tests/features/contracts/ | docs/spec-backlog/contract-001.md | - |
| handoff:StageRunner→Store | StageRunner | Store | DOM-3 | draft | - | - | contract-002 | skeleton | tests/features/contracts/ | - | wait for goal clarification |

欄位規則：
- `handoff`: 使用穩定名稱，建議 `handoff:{from}→{to}`。
- `status`: 僅能為 `draft` / `ready` / `in-progress` / `done` / `blocked`。
- `order`: 只對 `ready` 填整數（1..n），其他狀態可用 `-`。
- `order_reason`: 只對 `ready` 必填，一句話說明為何排在此順位（需可追溯）。
- `type` 固定為 `skeleton`。
- `tests_path` 固定為 `tests/features/contracts/`。
- `notes`: 補充限制或升級資訊；若為 infrastructure concern，標註 `escalate to SYSTEM_MAP`。

---

## Update Rules

- Design mode：建立或重寫 `Baseline` 與主表（全盤盤點）。
- Verify mode：更新主表對應列（status/order/order_reason/card/notes），不需 append iteration 區塊。
- 若發現 infrastructure concern（queue/transport/capacity/topology），在 `notes` 記錄 `escalate to SYSTEM_MAP`，不得新增 finding card。
