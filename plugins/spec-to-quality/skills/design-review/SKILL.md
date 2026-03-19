---
name: design-review
description: >
  TDD green 之後的設計品質 review，教育導向。檢視程式碼的設計原則、職責分離、耦合方向等
  linter 抓不到的問題，以提問方式引導使用者思考，而非直接修改。
  當使用者完成 TDD green 階段、要做 code review、或說「review 一下」、「檢查設計」時觸發。
  Do NOT use for: 單純看 code 什麼意思、學習語法、尚未完成 TDD green 時（先完成 tdd-workflow）、
  或 bug 修復時（用 debugging）。
---

# Design Review（設計品味檢視）

你正在進入 design review 階段。這個 review 的目的是**教育性的**——發現 linter 抓不到的設計問題，用提問方式引導使用者思考，而不是直接改 code。

## 前置條件

- 測試已經是綠燈（所有測試通過）
- lint + type check 已經通過（參照專案 CLAUDE.md 的 Commands 區段）
- 如果上述條件未滿足，提醒使用者先完成

## 檢視範圍

只 review 本次新增或修改的檔案。用 git diff 或使用者提供的檔案清單確定範圍：

```bash
git diff --name-only HEAD~1  # 或根據實際情況調整
```

## 檢視維度

對以下 6 個維度逐一檢視。詳細的檢查要點見 `references/dimensions.md`。

1. **職責分離** — 一個 class/function 只做一件事？
2. **依賴方向** — 內層是否依賴了外層？
3. **命名語意** — 名稱是否準確反映行為？
4. **錯誤處理策略** — exception 在正確的層被 catch？
5. **可測試性** — 補測試容易嗎？有隱藏依賴嗎？
6. **一致性** — 與專案中類似功能的風格一致？

## 輸出格式

### 問題（如果有的話）

對每個發現的問題，用以下格式呈現：

**[維度] 檔案:行號 — 簡述**

> 目前的寫法：（簡要描述現狀）
>
> 考慮的方向：（提問，不是指令）
>
> 為什麼值得思考：（解釋這個設計原則，教育目的）

### 總結

以表格呈現：

| 維度 | 狀態 | 備註 |
|------|------|------|
| 職責分離 | OK / 有發現 | |
| 依賴方向 | OK / 有發現 | |
| 命名語意 | OK / 有發現 | |
| 錯誤處理 | OK / 有發現 | |
| 可測試性 | OK / 有發現 | |
| 一致性 | OK / 有發現 | |

## Examples

### Example 1: 正常的 design review

使用者完成 TDD green，說：「review 一下」

1. 確認測試綠燈 + lint/type check 通過
2. `git diff --name-only HEAD~1` 確定範圍（例如 3 個檔案）
3. 逐維度檢視，發現：
   - 職責分離：某個 function 同時讀 DB 和做格式轉換
   - 一致性：其他模組用 Protocol，這裡直接 import 具體 class
4. 以提問方式呈現發現 + 總結表格

### Example 2: 沒有設計問題

review 完 6 個維度都 OK。

正確行為：直接呈現全 OK 的表格，不要為了 review 而硬找問題。可以簡短說「設計乾淨，沒有需要討論的點」。

### Example 3: 前置條件未滿足

使用者說「review 一下」但測試還沒跑過。

正確行為：提醒「design review 的前提是測試綠燈 + lint/type check 通過，建議先完成這些再 review」。不要直接開始 review。

## 重要原則

- **提問優於指令**：「這個 function 同時讀 DB 和做轉換，你覺得拆開會不會更好？」而非「把這個拆成兩個 function」
- **解釋為什麼**：每個建議都要附上設計原則的簡短說明
- **不要吹毛求疵**：只提有實質影響的問題，不要為了 review 而 review
- **承認取捨**：有些設計是 trade-off，明確說明兩邊的優缺點讓使用者判斷
- **尊重上下文**：如果是原型/spike，設計標準可以放寬；如果是核心模組，標準要高
