---
name: ec:gherkin
description: >
  產生符合最佳實踐的 Gherkin .feature 功能規格文件。關鍵字使用英文（Feature/Scenario/Given/When/Then），內容描述使用繁體中文，遵循 Feature / Rule / Scenario 結構。
  當使用者提到要建立功能規格、撰寫 .feature 檔、定義驗收條件、描述使用情境、或想用 Gherkin 語法記錄需求時，
  都應觸發此 skill。即使使用者只是說「幫我寫規格」、「整理這個功能的情境」、「定義 acceptance criteria」，
  也應使用此 skill。
  Do NOT use for: 還沒做過覆蓋率分析時（先用 ec:feature-coverage）、要開始實作測試時（用 ec:tdd-workflow）、
  或修改既有 .feature 檔的小幅調整（直接改即可）。
---

# Gherkin Feature 規格撰寫指南

你是一個功能規格撰寫助手。你的任務是透過引導式對話，幫助使用者產出結構清晰、可測試的 Gherkin `.feature` 檔案。

## 工作流程

### 第一步：釐清需求

在撰寫之前，先理解使用者要描述的功能。依序確認以下資訊（已知的可跳過）：

1. **功能名稱**：這個功能叫什麼？（例如：「對話功能」、「上下文壓縮」）
2. **目標角色**：誰在使用這個功能？（例如：使用者、系統管理員、Agent 系統）
3. **核心價值**：使用者透過這個功能想達成什麼目的？
4. **主要情境**：有哪些使用場景？正常流程與異常流程各有哪些？
5. **業務規則**：有沒有需要遵守的規則或限制？這些規則會成為 Rule 的基礎
6. **放置路徑**：確認 .feature 檔應放在哪裡（若專案有明確結構）

如果使用者已經在對話中提供了足夠資訊（例如從 ec:feature-coverage 分析銜接過來），可以減少提問，直接進入撰寫。

### 第二步：撰寫 .feature 檔

根據收集到的資訊撰寫 Feature 檔。遵循以下結構：

```gherkin
Feature: 功能名稱
  作為 [角色]
  我想要 [功能]
  以便 [價值]

  Background:
    # 所有 Scenario 共用的前置條件（選用，有共用設定時才加）
    Given 系統已初始化

  Rule: 業務規則描述

    Scenario: 情境名稱
      Given 前置條件
      When 執行動作
      Then 預期結果
```

### 第三步：確認與調整

產出後主動詢問使用者是否需要調整，例如：
- 是否漏掉了重要情境？
- Rule 的分組是否合理？
- Scenario 的粒度是否適當？

## 語言規範

- **Gherkin 關鍵字一律使用英文**：`Feature`、`Rule`、`Scenario`、`Given`、`When`、`Then`、`And`、`But`、`Background`、`Scenario Outline`、`Examples`
- **內容描述使用繁體中文**：Feature 名稱、Scenario 名稱、步驟描述、Rule 描述等
- 不要使用中文關鍵字（如「功能」、「場景」、「假設」、「當」、「那麼」）— pytest-bdd 不支援，且降低工具相容性

```gherkin
# 正確
Feature: 密碼重設
  Scenario: 發送重設密碼信件
    Given 系統中存在一個已註冊的使用者
    When 使用者輸入註冊的 email 並請求重設密碼
    Then 系統應寄送一封包含重設連結的 email

# 錯誤 — 關鍵字不可用中文
功能: 密碼重設
  場景: 發送重設密碼信件
    假設 系統中存在一個已註冊的使用者
    當 使用者輸入註冊的 email 並請求重設密碼
    那麼 系統應寄送一封包含重設連結的 email
```

## 撰寫原則

### Feature 層級
- 一個 Feature 檔 = 一個獨立的業務領域或功能模組
- Feature 描述使用「作為 / 我想要 / 以便」三段式，清楚說明角色、功能、價值

### Rule 層級
- 每個 Rule 對應一條明確的業務規則
- Rule 的用途是分組——將驗證同一規則的 Scenario 放在一起
- 若 Feature 很簡單（只有 2-3 個 Scenario），可以省略 Rule

### Scenario 層級
- **單一職責**：每個 Scenario 只驗證一個行為，不要塞太多斷言
- **獨立性**：Scenario 之間不應有依賴關係，每個都可以獨立執行
- **可讀性**：使用領域語言，讓非技術人員也能理解
- **可測試性**：每個 Scenario 最終要能轉換成自動化測試案例

### Given / When / Then 原則
- **Given**：描述初始狀態（前置條件），可以有多個，用 `And` 串接
- **When**：描述觸發的動作，盡量只有一個——如果有兩個 When，考慮拆成兩個 Scenario
- **Then**：描述可驗證的預期結果，避免模糊的斷言（「應正常運作」是不好的）

### Background 使用時機
- 當多個 Scenario 有相同的前置條件時使用
- Background 中只放 Given 步驟
- 不要把只有部分 Scenario 需要的前置條件放進 Background

## 常見問題與避免的寫法

### 避免：Scenario 做太多事
```gherkin
# 不好：一個 Scenario 驗證太多行為
Scenario: 完整流程
  Given 使用者登入
  When 使用者建立訂單
  And 使用者付款
  And 系統寄送確認信
  Then 訂單狀態為已完成
```

應拆分為多個獨立 Scenario，各自驗證一個步驟。

### 避免：不可驗證的 Then
```gherkin
# 不好：「正確」太模糊
Then 系統應正確處理請求

# 好：具體可驗證
Then 回應狀態碼應為 200
And 回應內容應包含使用者名稱
```

### 避免：技術實作細節
```gherkin
# 不好：暴露實作
When 發送 POST /api/users 並帶入 JSON body

# 好：使用領域語言
When 使用者提交註冊表單
```

但若 Feature 本身就是描述技術元件（如 API 規格、工具行為），使用技術語言是合理的——重點是使用該功能領域的自然語言。

## Examples

### Example 1: 從 ec:feature-coverage 銜接過來

使用者已完成覆蓋率分析，確認了 5 個適用類別。

正確行為：不需要重新問需求，直接根據分析表的「具體 Scenario 方向」欄位開始撰寫 .feature 檔。每個「適用」的類別至少對應一個 Scenario。

### Example 2: 使用者資訊不足

使用者說：「幫我寫一個通知功能的 feature」

正確行為：依序確認——誰收通知？什麼事件觸發？通知管道（email、push、in-app）？失敗時怎麼辦？收集足夠後再撰寫。

### Example 3: 使用者提供了完整資訊

使用者在對話中已清楚描述了功能需求、角色、情境。

正確行為：跳過提問，直接進入第二步撰寫。產出後仍須走第三步確認。

## 參考範例

參閱 `references/examples.md` 取得完整的真實範例，涵蓋不同複雜度的 Feature 檔案。
