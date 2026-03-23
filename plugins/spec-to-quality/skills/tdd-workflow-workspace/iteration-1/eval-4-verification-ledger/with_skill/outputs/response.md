現在我有了足夠的資訊來進行 **Phase 0: Verification Ledger**。

---

## Phase 0: Verification Ledger（Mock 邊界審查）

### Step 1: SHALL 清單

從 spec.md 提取的 5 個 SHALL 語句：

| # | SHALL 語句 |
|---|-----------|
| 1 | SHALL 使用 xai_sdk.Client 呼叫 Grok API 進行翻譯 |
| 2 | SHALL 從 response.usage 提取 input_tokens 和 output_tokens |
| 3 | SHALL 支援 batch 翻譯（一次傳入多個文字段落） |
| 4 | SHALL 在 API 回傳 429 時自動重試（最多 3 次） |
| 5 | SHALL 將翻譯結果存入 TranslationRecord model |

---

### Step 2 & 3: Mock 邊界規劃 + Checkpoint A/B

| SHALL | 驗證方式 | Mock 邊界 | Checkpoint A | Checkpoint B |
|-------|---------|----------|--------------|--------------|
| **1. 使用 xai_sdk.Client** | Unit test | Mock `xai_sdk.Client` — 這是外部 SDK，mock 在最外層切 | ✅ Mock 的是外部服務（xai_sdk），不是自己邏輯 | N/A |
| **2. 從 response.usage 提取 token** | Unit test | 承上，mock 的 response 需包含 `.usage.input_tokens` 和 `.usage.output_tokens` 形狀 | ✅ Mock 外部回傳的 response 結構 | ✅ 需驗證我們產出的 token 數量 = response.usage 的值 |
| **3. Batch 翻譯** | Unit test | 同上 mock xai_sdk.Client，測試傳入多筆 → 回傳多筆 | ✅ 自己的 batch 邏輯，mock 外部 | ✅ 輸入 N 筆 → 輸出 N 筆對應關係需驗證 |
| **4. 429 重試** | Unit test | Mock xai_sdk.Client 的呼叫，讓它先回 429 再成功（side_effect 控制） | ✅ Mock 外部服務的行為（429 response） | ✅ 驗證重試次數、最終成功/失敗 |
| **5. 存入 TranslationRecord** | Unit test | Mock 儲存層（Repository 或 DB session），驗證被呼叫時傳入正確的 model | ✅ Mock 的是儲存層（外部依賴） | ✅ TranslationRecord 的欄位 = 翻譯結果的欄位 |

**Mock 技術決策**：
- 使用自定義 **Fake** 或 `MagicMock(spec=...)` 來限制介面
- `xai_sdk.Client` → 建立 `FakeXaiClient` 回傳固定 response 結構
- 儲存層 → 定義 `TranslationRepository` Protocol，測試用 in-memory 實作

---

### Step 4: Verification Ledger

```markdown
# Verification Ledger — 翻譯功能

## Unit Test 覆蓋

- SHALL 1 (使用 xai_sdk.Client)
  → ✅ mock 邊界：mock xai_sdk.Client，驗證我們的翻譯 service 正確呼叫 API

- SHALL 2 (從 response.usage 提取 token)
  → ✅ mock 邊界：mock response 包含 usage 結構，驗證提取出正確的 token 數量

- SHALL 3 (batch 翻譯)
  → ✅ mock 邊界：同上，驗證 N 筆輸入 → N 筆輸出

- SHALL 4 (429 重試)
  → ✅ mock 邊界：mock Client 用 side_effect 控制先 429 再成功，驗證重試邏輯

- SHALL 5 (存入 TranslationRecord)
  → ✅ mock 邊界：mock Repository，驗證被呼叫時傳入正確的 TranslationRecord

## 需要整合測試

- 無。所有 SHALL 都可透過 unit test + mock 外部依賴來驗證。
- 若未來需要驗證「實際呼叫 Grok API」，可另開整合測試 scope。

## 明確不測（附理由）

- （無）
```

---

**請確認這個 Verification Ledger**，確認後我會進入 Phase 1: Red（寫測試、確認失敗）。
