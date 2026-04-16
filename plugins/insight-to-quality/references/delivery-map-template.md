# Delivery Map Template

> 介於 `SYSTEM_MAP` 與 `spec-xxx reports / finding cards / .feature` 之間的中層交付地圖。

## 使用方式

- 本文件是交付狀態的「跨 finding 全局視圖」，不是執行日誌。
- 每一列應可追到：`Gx -> Dx/APx -> Boundary/Seam -> finding-id -> feature`。
- 詳細執行證據在 `docs/spec-backlog/{finding-id}.md`、對應 `spec-xxx report` 主表列與測試結果；本檔只放索引與治理狀態。

## 狀態定義

- `planned`: 已識別交付切片，但尚未進入當前迭代
- `active`: 至少一個對應 finding 為 `in-progress`
- `stabilizing`: 主要實作完成，正在補齊整合/E2E 或治理項
- `done`: 交付完成，無阻塞 gap
- `archive-candidate`: 已完成且符合歸檔建議條件，等待治理確認

## Delivery Rows

| Delivery ID | Scope (Boundary/Journey) | Serves (Gx) | Related (Dx/APx) | Active Findings | Spec Reports | Feature Files | Test Coverage (U/I/E) | Deferred E2E (Owner/Target) | Status | Last Verified |
|---|---|---|---|---|---|---|---|---|---|---|
| DLV-001 | boundary:InterviewSession | G2,G3 | D1,AP-2 | behavior-002 | behaviors:behavior-002 / surface:surface-002 | tests/features/behaviors/behavior-002.feature | U:yes I:yes E:deferred | alice / 2026-05-01 | active | 2026-04-16 |

## Navigation Shortcuts

- 從 `SYSTEM_MAP` 跳入：每個 boundary/seam 可標註對應 `Delivery ID`。
- 從 `Delivery ID` 跳入：
  - `docs/spec-backlog/{finding-id}.md`
  - `docs/contracts/contracts-report.md` / `docs/surface/surface-report.md` / `docs/behaviors/behavior-report.md`（對應列）
  - 主要 `.feature`

## 更新節奏

- `spec-contract` / `spec-surface` / `spec-behavior` 發現或重排切片時：新增或更新對應 delivery row
- `spec-to-gherkin` 完成時：更新 Feature Files / Test Coverage 欄位
- `design-review` 通過時：更新 Status / Last Verified
- `docs-governance` 稀疏巡檢：修正 stale link、狀態漂移、archive-candidate 註記
