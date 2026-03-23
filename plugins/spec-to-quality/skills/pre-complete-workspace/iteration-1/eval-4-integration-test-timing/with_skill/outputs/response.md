我看到 openspec/specs/ 只有 `translation/spec.md`，但 change 目錄有 `conversation-api/spec.md`。這表示 **conversation-api 的主規格尚未建立**，需要同步。

由於這不是一個真實的 git repo（且沒有實際的 Python 程式碼），我無法執行測試、lint、type check。但讓我根據你提供的資訊整理 Pre-Complete Verification 的發現：

---

## ⚠️ Pre-Complete Verification 結果

| 驗證項目 | 狀態 | 備註 |
|---------|------|------|
| 測試 | ⏸️ 無法執行 | 非 git repo / 無 Python code |
| Lint | ⏸️ 無法執行 | 同上 |
| Format | ⏸️ 無法執行 | 同上 |
| Type check | ⏸️ 無法執行 | 同上 |
| 變更檔案 | ⏸️ 無法檢查 | 非 git repo |
| OpenSpec tasks | ✅ 全部完成 | 3/3 tasks 已勾選 |
| **Delta Spec 同步** | ❌ **需要處理** | `conversation-api/spec.md` 未同步到 `openspec/specs/` |
| **整合測試缺口** | ⚠️ **2 項需處理** | 等級 2，conversation 最後一個 spec |

---

## 🚨 需要處理的項目

### 1. Delta Spec 同步（必須）

change 目錄有 `openspec/changes/2026-03-23-conversation/specs/conversation-api/spec.md`，但主規格目錄 `openspec/specs/` 沒有對應的 `conversation-api/spec.md`。

**需要建立：** `openspec/specs/conversation-api/spec.md`

### 2. 整合測試缺口（這是 conversation 模組的最後一個 spec）

verification.md 有 **2 個等級 2 項目**：

| 項目 | 最低驗證方式 | 狀態 |
|------|-------------|------|
| WebRTC offer/answer 交換 | 本地跑 app + 瀏覽器手動測試 | ❓ 未驗證 |
| GeminiHandler Live API session | staging 環境 smoke test | ❓ 未驗證 |

**既然這是 conversation 模組的最後一個 spec**，建議現在補寫整合測試：

1. **WebRTC signaling contract test** — 驗證 offer/answer exchange 流程
2. **GeminiHandler smoke test** — staging 環境驗證 Live API session 建立

---

## 下一步

1. **要現在同步 delta spec 嗎？** 我可以建立 `openspec/specs/conversation-api/spec.md`
2. **整合測試怎麼處理？**
   - A) 現在補寫整合測試
   - B) 先記錄到 issue，之後再補
   - C) 手動測試過了就 commit（需確認有跑過手動測試）

請告訴我你的決定。
