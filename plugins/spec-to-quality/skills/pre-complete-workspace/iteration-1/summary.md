# pre-complete eval-4 Summary — 整合測試時機判斷

## 結果

| | with_skill | without_skill | Delta |
|---|---|---|---|
| A1 標準 checklist | ❌* | ❌ | 0 |
| A2 Delta spec 同步提醒 | ✅ | ❌ | +1 |
| A3 讀取 verification ledger | ✅ | ✅ | 0 |
| A4 整合測試時機判斷 | ✅ | ❌ | +1 |
| A5 具體建議 | ✅ | ❌ | +1 |
| A6 總結表格 | ✅ | ❌ | +1 |
| **Total** | **4/6** | **1/6** | **+3** |

*A1 兩者都未通過是因為 eval workspace 不是完整專案（非 git repo、無 pyproject.toml），屬於環境限制。

## 分析

with_skill 的核心新功能全部觸發：
- **Delta spec 同步**：主動發現 change 目錄有 spec.md 但主規格沒有，提醒需要同步
- **整合測試時機**：正確判斷「最後一個 spec + 有等級 2 項目」→ 主動建議現在補寫
- **具體建議**：提出 WebRTC signaling contract test 和 GeminiHandler smoke test
- **結構化輸出**：用表格呈現驗證狀態和需處理項目

without_skill 只是被動列出 verification.md 的內容，沒有任何主動判斷或建議。

**Skill 核心價值**：把 pre-complete 從「被動 checklist」升級為「主動判斷 + 建議」。
