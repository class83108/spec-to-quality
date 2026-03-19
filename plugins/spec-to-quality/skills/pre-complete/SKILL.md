---
name: pre-complete
description: >
  完成宣告前的強制驗證 checklist。在說「完成」、commit、或建立 PR 之前，
  必須跑過所有驗證命令並確認輸出。防止「說完成但其實沒跑過驗證」的情況。
  當即將完成一個 feature、要 commit、要建立 PR、或使用者說「差不多了」時觸發。
  Do NOT use for: 開發過程中的中間測試（那是 tdd-workflow 的一部分）、或單純想跑一下測試看狀態。
---

# Pre-Complete Verification（完成前驗證）

你即將宣告工作完成。在說出「完成」、建議 commit、或建立 PR 之前，你必須完成以下驗證。

## 核心原則

**證據先於結論。** 不要說「應該沒問題」——跑過驗證、看到輸出、然後才能宣告。

## 驗證命令

參照專案 CLAUDE.md 的 Commands 區段取得正確的測試、lint、type check 命令。
不要假設任何特定的套件管理工具或命令格式。

## Checklist

按順序執行，每一步都要展示實際輸出：

### 1. 測試

使用專案 CLAUDE.md 中定義的測試命令。

- 必須全部 PASS
- 如果有 FAIL → 停止，進入 debugging skill

### 2. Lint & Format

使用專案 CLAUDE.md 中定義的 lint/format 命令。

- 必須零 error
- 如果有 error → 修正後重跑測試確認還是綠的

### 3. Type Check

使用專案 CLAUDE.md 中定義的 type check 命令。

- 必須零 error（warning 可接受）
- 如果有 error → 修正後重跑測試確認還是綠的

### 4. 變更確認

```bash
git diff --stat
git status
```

- 列出所有變更的檔案
- 確認沒有不該 commit 的檔案（.env, credentials, 暫存檔）
- 確認沒有遺漏該 commit 的檔案

### 5. OpenSpec 狀態（如果有使用）

- 當前 change 的 tasks 是否都標記完成？
- 如果有未完成的 task，是故意留的還是遺漏？

## 輸出

全部通過後，以表格呈現：

| 驗證項目 | 狀態 | 備註 |
|---------|------|------|
| 測試 | PASS (N tests) | |
| Lint | PASS | |
| Format | PASS | |
| Type check | PASS | N warnings (如果有) |
| 變更檔案 | N files | 已確認無敏感檔案 |
| OpenSpec | 全部完成 / N 項未完成 | |

然後才可以建議 commit 或建立 PR。

## Examples

### Example 1: 正常完成流程

使用者說：「應該差不多了，可以 commit 了」

1. 跑測試 → 25 tests passed → 展示輸出
2. 跑 lint → 0 errors → 展示輸出
3. 跑 type check → 0 errors, 2 warnings → 展示輸出
4. git diff --stat → 列出 5 個檔案 → 確認無敏感檔案
5. OpenSpec → 3/3 tasks 完成
6. 呈現總結表格 → 「全部通過，可以 commit」

### Example 2: 驗證過程中發現問題

跑 lint 時發現 2 個 error。

正確行為：修正 → 重跑測試（不是只重跑 lint）→ 重跑 lint → 重跑 type check → 全部從頭走一次。因為修正可能引入新問題。

### Example 3: 使用者想跳過驗證

使用者說：「不用跑了，直接 commit」

正確行為：提醒使用者跳過驗證的風險，但尊重使用者的決定。如果使用者堅持，執行 commit 但在 commit message 中不宣稱「所有測試通過」。

## 如果驗證失敗

- 修正問題
- **從頭重跑整個 checklist**（不是只跑失敗的那項）
- 因為修正可能引入新問題
