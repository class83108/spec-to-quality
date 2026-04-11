---
name: ec:design-review
description: >
  TDD green 之後的設計品質驗證，分兩部分：（1）核實 OpenSpec 宣告的設計決定有沒有在實作中被遵守；
  （2）結構性檢查——職責分離、依賴方向等 linter 抓不到的問題，以提問方式引導思考。
  當使用者完成 TDD green 階段、要做 code review、或說「review 一下」、「檢查設計」時觸發。
  Do NOT use for: 單純看 code 什麼意思、學習語法、尚未完成 TDD green 時（先完成 ec:tdd-workflow）、
  或 bug 修復時。
---

# Design Review（設計驗證）

你正在進入 design review 階段。**在開始之前，先讀：**
- `references/architect-mindset.md`（用 traceability lens 看實作）
- `references/implementation-mindset.md`（核實標準與結構性檢查要點）

這個 review 分兩部分：

1. **宣告決定核實**：對照 OpenSpec 宣告的錯誤處理策略與 discovery anti-patterns，確認實作有沒有遵守。
   有違反就直接指出，不用提問。
2. **結構性檢查**：職責分離、依賴方向等 linter 抓不到的問題，用提問引導思考，不下指令。

## 前置條件

- 測試已經是綠燈（所有測試通過）
- lint + type check 已經通過（參照專案 CLAUDE.md 的 Commands 區段）
- 如果上述條件未滿足，提醒使用者先完成

## Phase 0: 收集宣告的設計決定

在 review 任何 code 之前：

1. **讀 `docs/feature-plans/{feature-name}.md`**：取得：
   - Error Handling Strategy（Catch boundary、Domain errors、Recovery strategy）
   - Anti-patterns（帶 ID）
   - Boundary Rules
   - 如果找不到 feature plan → 記錄「未宣告」，在第一部分標注所有決定為未宣告

2. **確定 review 範圍**：
   ```bash
   git diff --name-only HEAD~1  # 或根據實際情況調整
   ```

## 第一部分：宣告決定核實

### 錯誤處理策略

對照 OpenSpec 宣告的三個決定（見 `implementation-mindset.md` Part 1）：

| 宣告的決定 | 檢查項目 |
|-----------|---------|
| Catch boundary | 是否只在宣告的層有 try/except？有沒有在不該有的地方出現？ |
| Domain errors | 宣告的 domain error 有沒有被正確 raise？有沒有未宣告的偷偷出現？ |
| Recovery strategy | 實際的 recovery 行為有沒有跟宣告一致？ |

如果沒有宣告 → 標記「未宣告」，提示在 OpenSpec 補上後繼續做第二部分。

### Discovery 對齊（feature plan 存在時）

從 Phase 0 讀取的 feature plan 出發：
- 實作有沒有違反 feature plan 的 Anti-patterns？
- 實作有沒有跨越 feature plan 的 Boundary Rules？
- 如果有 Internals / Surface Alignment Report，標記的缺口在這次實作中有沒有被意外引入？

有發現才提，沒有就標 OK。

## 第二部分：結構性檢查

對以下五個維度逐一檢視，詳細的檢查問題見 `implementation-mindset.md` Part 2。

1. **職責分離** — 一個 class/function 只做一件事？
2. **依賴方向** — 內層是否依賴了外層？
3. **命名語意** — 名稱是否準確反映行為？
4. **可測試性** — 補測試容易嗎？有隱藏依賴嗎？
5. **一致性** — 與專案中類似功能的風格一致？

## 輸出格式

### 第一部分：宣告決定核實

| 宣告的決定 | 宣告內容 | 核實狀態 | 備註 |
|-----------|---------|---------|------|
| Catch boundary | [從 OpenSpec 讀取] | 符合 / 違反 / 未宣告 | |
| Domain errors | [從 OpenSpec 讀取] | 符合 / 違反 / 未宣告 | |
| Recovery strategy | [從 OpenSpec 讀取] | 符合 / 違反 / 未宣告 | |
| Discovery 對齊 | anti-patterns + boundary | 符合 / 有違反 / 不適用 | |

如有違反，在表格下方直接列出：

**違反** `檔案:行號` — 說明（宣告 X，但實作是 Y）

### 第二部分：結構性問題（如果有的話）

**[維度] `檔案:行號` — 簡述**

> 目前的寫法：（簡要描述現狀）
>
> 考慮的方向：（提問，不是指令）
>
> 為什麼值得思考：（解釋這個設計原則）

### 總結

| 維度 | 狀態 | 備註 |
|------|------|------|
| 錯誤處理策略 | 符合 / 違反 / 未宣告 | |
| Discovery 對齊 | 符合 / 有違反 / 不適用 | |
| 職責分離 | OK / 有發現 | |
| 依賴方向 | OK / 有發現 | |
| 命名語意 | OK / 有發現 | |
| 可測試性 | OK / 有發現 | |
| 一致性 | OK / 有發現 | |

## Examples

### Example 1: 有宣告，發現違反

OpenSpec 宣告：catch boundary = boundary only；infrastructure errors = fail fast

Review 發現：service layer 裡有 `try/except Exception: logger.error(...)` 沒有 re-raise。

正確行為：第一部分表格標記「Catch boundary：違反」，表格下方列出：
「**違反** `service.py:42` — 非 boundary 層 catch 了 Exception 且未 re-raise，違反宣告的 boundary only 策略。」

### Example 2: 沒有宣告

Phase 0 讀 OpenSpec 找不到錯誤處理策略宣告。

正確行為：第一部分三個決定全部標記「未宣告」，提示：「建議在 OpenSpec design decisions 補上錯誤處理策略宣告（見 `implementation-mindset.md` Declaration Format）。」繼續做第二部分。

### Example 3: 全部 OK

第一部分全部符合；第二部分 5 個維度都 OK。

正確行為：直接呈現兩個表格。可以說「宣告決定全部符合，結構性設計乾淨。」

### Example 4: 前置條件未滿足

使用者說「review 一下」但測試還沒跑過。

正確行為：提醒「design review 的前提是測試綠燈 + lint/type check 通過，建議先完成這些再 review」。

## 重要原則

- **宣告決定違反 → 直接指出**：不用提問，直接說「宣告 X，但實作是 Y」
- **結構性問題 → 提問優於指令**：「這個 function 同時讀 DB 和做轉換，你覺得拆開會不會更好？」
- **沒有宣告不等於設計錯誤**：標注「未宣告」，讓使用者知道這是需要補上的決定
- **不要吹毛求疵**：只提有實質影響的問題
- **承認取捨**：有意識選擇的 trade-off，即使你不認同，也不是 review 的目標
- **尊重上下文**：原型 / spike 標準可以放寬；核心模組標準要高
