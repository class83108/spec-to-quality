# tdd-workflow eval-4 Summary — Verification Ledger (iteration-2)

## 變更內容

SKILL.md Step 4 修改：從「直接建立 verification.md」改為「展示 ledger → 詢問內容是否正確 + 是否要建檔」。

## with_skill 結果：9/9

| Assertion | Pass | 說明 |
|-----------|------|------|
| A1 Phase 0 在 Red 前觸發 | ✅ | 明確進入 Phase 0 |
| A2 SHALL 清單提取 | ✅ | 列出 5 個 SHALL 表格 |
| A3 Mock 邊界規劃 | ✅ | 每個 SHALL 都有 mock 邊界 |
| A4 Checkpoint A | ✅ | 確認 mock 外部服務而非自己邏輯 |
| A5 Checkpoint B | ✅ | 跨元件資料流形狀有驗證 |
| A6 Ledger 內容展示 | ✅ | 三區分類完整 |
| A7 詢問內容是否正確 | ✅ | **新 assertion** — 明確問「SHALL 語句對嗎？mock 邊界合理嗎？」 |
| A8 詢問是否建檔 | ✅ | **新 assertion** — 明確問「要將 ledger 寫入 verification.md 嗎？」 |
| A9 等待確認才進 Red | ✅ | 「請確認後我再進入 Phase 1: Red」 |

## without_skill 結果：⚠️ 已污染

subagent 繼承 parent conversation 的 system context（含舊版 CLAUDE.md，有 Verification Ledger 步驟）。
結果不可用作 baseline。需要在新 conversation 中重跑。

## 與 iteration-1 比較

| | iteration-1 (7 assertions) | iteration-2 (9 assertions) |
|---|---|---|
| with_skill | 7/7 | 9/9 |
| 新增 A7（問內容） | N/A | ✅ |
| 新增 A8（問建檔） | N/A | ✅ |

## 結論

修改後的 SKILL.md 成功引導 agent：
1. 展示完整 ledger 內容（與 iteration-1 一致）
2. **新行為**：明確分開詢問「內容是否正確」和「是否要建檔」
3. 等使用者確認後才進入 Red

## 待辦

- [ ] 在新 conversation 中重跑 without_skill baseline（先確認全域 CLAUDE.md 已改為舊版）
