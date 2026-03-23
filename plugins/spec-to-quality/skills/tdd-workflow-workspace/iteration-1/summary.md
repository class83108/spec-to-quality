# tdd-workflow eval-4 Summary — Verification Ledger

## 結果

| | with_skill | without_skill | Delta |
|---|---|---|---|
| A1 Phase 0 在 Red 前觸發 | ✅ | ❌ | +1 |
| A2 SHALL 清單提取 | ✅ | ❌ | +1 |
| A3 Mock 邊界規劃 | ✅ | ❌ | +1 |
| A4 Checkpoint A | ✅ | ❌ | +1 |
| A5 Checkpoint B | ✅ | ❌ | +1 |
| A6 Ledger 格式 | ✅ | ❌ | +1 |
| A7 等待確認 | ✅ | ❌ | +1 |
| **Total** | **7/7** | **0/7** | **+7** |

## 分析

Verification Ledger 是全新概念，舊版 CLAUDE.md 完全沒有。without_skill 直接跳到準備實作，沒有任何 mock 邊界審查。

with_skill 產出了完整的 ledger：5 個 SHALL 逐一分析、每個 mock 都標明 Checkpoint A/B、建議使用 Fake 而非 MagicMock、明確等待使用者確認。

**Skill 核心價值**：強制在寫測試前思考「每個 SHALL 由誰驗證」和「mock 邊界畫在哪」。
