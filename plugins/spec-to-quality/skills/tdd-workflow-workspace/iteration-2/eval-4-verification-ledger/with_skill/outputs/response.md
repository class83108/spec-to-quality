好的，feature 檔已確認，我們進入 TDD 流程。按照流程規定，在寫任何測試之前，必須先完成 **Phase 0: Verification Ledger（Mock 邊界審查）**。

讓我根據你描述的 spec.md 中 5 個 SHALL 語句來規劃。

---

## Phase 0: Verification Ledger — 翻譯功能

### Step 1: SHALL 清單

根據 spec 描述，推斷 5 個 SHALL 語句如下：

| # | SHALL 語句 |
|---|-----------|
| S1 | SHALL 使用 `xai_sdk.Client` 呼叫 API 進行翻譯 |
| S2 | SHALL 從 `response.usage` 提取 token 數量 |
| S3 | SHALL 將原文翻譯為目標語言並回傳翻譯結果 |
| S4 | SHALL 在 API 呼叫失敗時拋出適當的錯誤 |
| S5 | SHALL 回傳包含翻譯文字與 token 使用量的結構化結果 |

（如果我推斷的 SHALL 語句與實際 spec 不符，請糾正我。）

### Step 2: Mock 邊界規劃

| SHALL | 驗證方式 | 等級 | Mock 邊界 | 驗證目標 |
|-------|---------|------|-----------|---------|
| S1 | Unit test | 等級 2 | mock `xai_sdk.Client` — 寫 Fake 回傳固定 response | 驗證我們的程式碼正確地建構 Client 並呼叫翻譯 API |
| S2 | Unit test | 等級 2 | 同上，Fake response 包含 `.usage` 屬性 | 驗證我們的程式碼正確從 response.usage 提取 token 數量 |
| S3 | Unit test | 等級 2 | 同上，Fake response 包含翻譯後文字 | 驗證我們的程式碼正確回傳翻譯結果 |
| S4 | Unit test | 等級 2 | Fake Client 拋出 exception | 驗證我們的程式碼正確處理 API 失敗情境 |
| S5 | Unit test | 無需 mock | 無（驗證回傳型別） | 驗證回傳結構包含翻譯文字與 token 使用量欄位 |

### Step 3: Mock 邊界檢查

**Checkpoint A — mock 遮掉的是外部服務還是自己的邏輯？**

- `xai_sdk.Client` 是外部 SDK，mock 切在 Client 層級是正確的。我們 mock 的是「xai API 的回應行為」，不是自己的翻譯邏輯。
- 不會 mock 我們自己組裝 prompt、解析 response 的邏輯——這些是要被測試驗證的核心邏輯。

**Checkpoint B — 跨元件資料流形狀一致性？**

- Fake response 的形狀必須與真實 `xai_sdk` 的 response 結構一致（包含 `.usage.prompt_tokens`、`.usage.completion_tokens` 等屬性）。建議用自定義 Fake class 而非 `MagicMock()`，這樣 Fake 的屬性結構是明確的，如果 SDK 升版改了 response 結構，Fake 不會靜默通過。

**Mock 實作方式：** 自定義 `FakeXaiClient` 和 `FakeResponse`，不使用無限制的 `MagicMock()`。如果要用 `MagicMock`，必須加 `spec=xai_sdk.Client`。

### Step 4: Verification Ledger

```markdown
# Verification Ledger — 翻譯功能

## Unit Test 覆蓋
- S1: SHALL 使用 xai_sdk.Client 呼叫 API → ✅ mock 邊界：FakeXaiClient 回傳固定 response，驗證呼叫邏輯正確建構 request
- S2: SHALL 從 response.usage 提取 token 數量 → ✅ mock 邊界：FakeResponse 包含 .usage 屬性，驗證提取邏輯正確讀取 token 數
- S3: SHALL 將原文翻譯為目標語言並回傳翻譯結果 → ✅ mock 邊界：FakeResponse 包含翻譯文字，驗證回傳值正確
- S4: SHALL 在 API 呼叫失敗時拋出適當的錯誤 → ✅ mock 邊界：FakeXaiClient 拋出 exception，驗證錯誤處理邏輯
- S5: SHALL 回傳包含翻譯文字與 token 使用量的結構化結果 → ✅ 無需 mock，驗證回傳型別與欄位完整性

## 需要整合測試
（本次無等級 3 項目）

## 明確不測（附理由）
（無）
```

---

有兩個問題想確認：

1. **Ledger 內容是否正確？** 我推斷的 5 個 SHALL 語句對嗎？mock 邊界和分級是否合理？
2. **要將 ledger 寫入 `verification.md` 檔案嗎？**（如果同意，我會建在 openspec 對應的 spec 目錄下）

請確認後我再進入 Phase 1: Red（撰寫測試）。
