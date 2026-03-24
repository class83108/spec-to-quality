# TDD Workflow — Phase 0: Verification Ledger

好的，`.feature` 檔已確認，進入 **Verification Ledger（Mock 邊界審查）** 階段。

---

## Step 1: SHALL 清單（從 spec.md 推斷）

根據你提供的資訊，spec.md 中有 5 個 SHALL 語句。以下是我根據翻譯功能常見規格推斷的內容，請確認是否符合你的 spec：

1. **SHALL 使用 `xai_sdk.Client` 呼叫翻譯 API**
2. **SHALL 從 `response.usage` 提取 token 數量**（輸入 token + 輸出 token）
3. **SHALL 將翻譯結果回傳為結構化資料**（包含翻譯文字與 token 統計）
4. **SHALL 在 API 呼叫失敗時拋出明確的例外**（或回傳 error 狀態）
5. **SHALL 支援指定目標語言作為參數**

> 如果推斷的 SHALL 語句與你的 spec.md 不符，請告知實際內容，我會重新規劃。

---

## Step 2 & 3: Mock 邊界規劃

| SHALL | 驗證方式 | Mock 邊界 | 等級 |
|-------|---------|-----------|------|
| SHALL 使用 `xai_sdk.Client` 呼叫翻譯 API | Unit test | mock `xai_sdk.Client`（最外層，SDK 進入點） | 等級 2 |
| SHALL 從 `response.usage` 提取 token 數量 | Unit test | mock `xai_sdk.Client`，回傳含 `usage` 欄位的 Fake response | 等級 2 |
| SHALL 將翻譯結果回傳為結構化資料 | Unit test | mock `xai_sdk.Client`，驗證回傳的 Pydantic model 欄位完整性 | Unit test |
| SHALL 在 API 呼叫失敗時拋出明確的例外 | Unit test | mock `xai_sdk.Client` 拋出例外，驗證包裝後的例外型別 | Unit test |
| SHALL 支援指定目標語言作為參數 | Unit test | mock `xai_sdk.Client`，驗證呼叫參數中的 `target_language` 正確傳入 | Unit test |

**Checkpoint A**（mock 遮掉的是外部還是自己的邏輯？）

- 所有 mock 都切在 `xai_sdk.Client` 這一層，屬於外部 SDK，不是自己的邏輯。

**Checkpoint B**（跨元件資料流的形狀是否一致？）

- `xai_sdk.Client` 回傳的 `response` 物件形狀（特別是 `response.usage` 的欄位）需要和我們解析邏輯的期望一致。使用自定義 Fake（非 `MagicMock()`）來固定 `response.usage` 結構，確保 output 形狀可控。

---

## Step 4: Verification Ledger — 翻譯功能

```markdown
# Verification Ledger — 翻譯功能

## Unit Test 覆蓋
- SHALL 使用 `xai_sdk.Client` 呼叫翻譯 API
  → mock 邊界：mock `xai_sdk.Client`（等級 2 Fake），驗證 client 被正確呼叫
- SHALL 從 `response.usage` 提取 token 數量
  → mock 邊界：Fake response 含 `usage.input_tokens` / `usage.output_tokens`，驗證解析邏輯正確
- SHALL 將翻譯結果回傳為結構化資料
  → mock 邊界：mock client，驗證回傳 Pydantic model 的欄位完整性
- SHALL 在 API 呼叫失敗時拋出明確的例外
  → mock 邊界：mock client.translate 拋出 SDK 例外，驗證包裝後型別正確
- SHALL 支援指定目標語言作為參數
  → mock 邊界：mock client，驗證呼叫時 `target_language` 參數正確傳遞

## 需要整合測試
（無）

## 明確不測（附理由）
（無）
```

---

## 兩個問題請確認：

**1. Ledger 內容是否正確？**
- mock 邊界（全部切在 `xai_sdk.Client` 最外層）是否合理？
- SHALL 語句的推斷是否符合你的 spec.md？如有不符請告知實際內容

**2. 是否要建立 `verification.md`？**
- 目前兩個缺口區段都是空的（無需整合測試、無明確不測），通常不需要建檔
- 如果你仍希望留存記錄，我可以在 `openspec/` 對應目錄下建立 `verification.md`

---

請確認以上內容後，我們才進入 **Phase 1: Red**（撰寫 step definitions，確認全部失敗）。
