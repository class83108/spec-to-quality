# tdd-workflow eval-4 Summary — Verification Ledger (iteration-3)

## 變更內容

SKILL.md Step 4 修改：建檔時只記錄缺口區段（需要整合測試 + 明確不測），Unit Test 覆蓋不建檔。
若缺口區段皆為空，提示通常不需要建檔。

## with_skill 結果：10/10

| Assertion | Pass | 說明 |
|-----------|------|------|
| A1 Phase 0 在 Red 前觸發 | ✅ | |
| A2 SHALL 清單提取 | ✅ | 推斷 5 個並請使用者確認 |
| A3 Mock 邊界規劃 | ✅ | |
| A4 Checkpoint A | ✅ | |
| A5 Checkpoint B | ✅ | 建議用自定義 Fake 固定形狀 |
| A6 Ledger 內容展示 | ✅ | 完整三區 |
| A7 詢問內容是否正確 | ✅ | |
| A8 詢問是否建檔 | ✅ | 說明可建在 openspec/ 對應目錄 |
| A9 缺口為空時提示不需建檔 | ✅ | **新 assertion** — 「兩個缺口區段都是空的，通常不需要建檔」 |
| A10 等待確認才進 Red | ✅ | |

## without_skill 結果：⚠️ 未跑

同 iteration-2，subagent 繼承 parent system context 問題，需在新 conversation 中重跑。

## 歷次比較

| | iteration-1 (7 assertions) | iteration-2 (9 assertions) | iteration-3 (10 assertions) |
|---|---|---|---|
| with_skill | 7/7 | 9/9 | 10/10 |
| 詢問內容 (A7) | N/A | ✅ | ✅ |
| 詢問建檔 (A8) | N/A | ✅ | ✅ |
| 缺口為空提示 (A9) | N/A | N/A | ✅ |

## 結論

`verification.md` 的定位已清楚：**整合測試缺口追蹤文件，不是 mock 邊界備忘錄**。
Agent 在無缺口時主動提示不需建檔，有缺口時才建檔並只記錄相關區段。
