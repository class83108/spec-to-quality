# spec-to-quality

一套給 Claude Code 用的 Python TDD 工作流 Skills。核心目標是讓需求到程式碼之間的資訊流失盡可能少，同時確保產出的 code 有一定的設計品質。

## 為什麼做這個

用 AI 工具開發的時候，我發現每一段交接都在丟東西：

- 從 spec 到 .feature 檔，AI 會漏掉 edge case 跟 error handling
- 從 .feature 到測試，step definitions 常常沒有完整對應 scenario
- 測試通過了，但 code 的設計很糟——職責混在一起、依賴方向亂跑
- 說「完成了」，但其實沒跑過 lint 或 type check

這些問題單獨看都有工具可以解：OpenSpec 可以管 spec、pytest-bdd 可以綁 feature 跟測試。但問題是**沒有東西能讓這整段流程每次都穩定跑完**。沒人盯的時候，步驟就會被跳過，品質就開始飄。

所以我把這套流程包成 6 個 Claude Code skills，用前置條件串起來，強制按順序走：

```
feature-coverage → gherkin → tdd-workflow → design-review → pre-complete
                                                  ↑
                                            debugging（隨時）
```

這不是那種裝了就能全自動開發的東西。它比較像是一套有主見的開發流程，讓 AI 在幫你寫 code 的時候不要亂跳步驟、不要漏東西。

## Skills 在幹嘛

| Skill | 做什麼 |
|-------|--------|
| **feature-coverage** | 寫 .feature 之前，強制對 6 類 scenario 逐一分析，避免漏掉情境 |
| **gherkin** | 照著覆蓋率分析的結果寫 .feature，遵循 Feature / Rule / Scenario 結構 |
| **tdd-workflow** | 嚴格的 Red → Green → Refactor，紅燈沒確認不能開始寫 code |
| **design-review** | 綠燈之後的設計審查，用提問方式引導思考，不是直接叫你改 |
| **debugging** | 遇到 bug 先收集證據、建假說、驗證，不准猜著改 |
| **pre-complete** | 要說「完成」之前，跑完測試 + lint + type check，拿到實際輸出才算數 |

## 適合什麼情境

- Python 後端，有用 pytest-bdd
- 想讓 TDD 流程穩定下來、不要每次都不一樣
- 希望不只是「測試通過就好」，還要有一定的設計品質

## 不適合什麼

- 前端開發
- 寫個小 script、改 config、快速原型
- 沒在用 pytest 或 Gherkin 的專案

## 安裝

```bash
# 加入 marketplace
/plugin marketplace add class83108/spec-to-quality

# 安裝
/plugin install spec-to-quality@spec-to-quality
```

## 你的專案需要準備什麼

在專案的 `CLAUDE.md` 裡面要有一個 **Commands** 區段，告訴 agent 怎麼跑測試、lint、type check。因為每個專案的工具不同，skills 不會假設你用什麼。

參考 [templates/CLAUDE.md.example](templates/CLAUDE.md.example) 看範例。

### 建議的工具組合

這套 skills 是圍繞這些工具設計的（但沒有硬綁定，你可以換）：

- **OpenSpec** — 需求與變更管理（skills 會讀 spec 內容來輔助覆蓋率分析）
- **uv** — 套件管理
- **pytest + pytest-bdd** — 測試
- **ruff** — lint & format
- **pyright** — 型別檢查

參考 [templates/pyproject.toml.snippet](templates/pyproject.toml.snippet) 看建議設定。

## 選用整合

- **Feature Scenario 具體化對應表**：可以在專案 CLAUDE.md 加一個表，把 6 類通用 scenario 類別對應到你專案的概念（例如「Error paths → Celery task timeout」）
