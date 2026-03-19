# Gherkin Feature 範例集

以下是不同複雜度的真實 Feature 範例，作為撰寫時的參考。

## 範例 1：簡單功能（無 Rule）

簡單功能可以省略 Rule，直接寫 Scenario。

```gherkin
Feature: 密碼重設
  作為 註冊使用者
  我想要 重設我的密碼
  以便 在忘記密碼時重新取得帳號存取權

  Background:
    Given 系統中存在一個已註冊的使用者

  Scenario: 發送重設密碼信件
    When 使用者輸入註冊的 email 並請求重設密碼
    Then 系統應寄送一封包含重設連結的 email
    And 重設連結應在 24 小時內有效

  Scenario: 使用不存在的 email 請求重設
    When 使用者輸入未註冊的 email 並請求重設密碼
    Then 系統應顯示與成功時相同的提示訊息
    And 不應寄送任何 email
```

## 範例 2：中等複雜度（含 Rule 分組）

業務規則較多時用 Rule 分組相關 Scenario。

```gherkin
Feature: 商品庫存管理
  作為 商店管理員
  我想要 管理商品庫存數量
  以便 確保顧客看到的商品可用狀態是正確的

  Background:
    Given 商品目錄中存在商品 "經典白T"
    And 該商品目前庫存為 10 件

  Rule: 庫存扣減應在訂單成立時執行

    Scenario: 成立訂單時扣減庫存
      When 顧客下單購買 2 件 "經典白T"
      Then 該商品庫存應減少為 8 件

    Scenario: 庫存不足時拒絕訂單
      When 顧客下單購買 15 件 "經典白T"
      Then 系統應拒絕該訂單
      And 庫存數量應維持 10 件不變

  Rule: 取消訂單應歸還庫存

    Scenario: 取消訂單後歸還庫存
      Given 顧客已成功下單購買 3 件 "經典白T"
      When 顧客取消該訂單
      Then 該商品庫存應恢復為 10 件

  Rule: 庫存為零時商品應標記為售罄

    Scenario: 庫存歸零時自動標記售罄
      Given 該商品目前庫存為 1 件
      When 顧客下單購買 1 件 "經典白T"
      Then 該商品應被標記為「售罄」
      And 商品頁面應顯示「目前無庫存」
```

## 範例 3：複雜功能（多 Rule、含 Error paths）

```gherkin
Feature: 電子發票開立
  作為 電商系統
  我想要 在訂單完成付款後自動開立電子發票
  以便 符合稅務法規並提供顧客完整的購買憑證

  Background:
    Given 系統已設定營業人統一編號與發票字軌
    And 存在一筆已付款的訂單

  Rule: 應根據訂單金額正確計算稅額

    Scenario: 含稅金額正確拆分
      Given 訂單含稅總金額為 1050 元
      When 系統開立電子發票
      Then 發票稅額應為 50 元
      And 發票未稅金額應為 1000 元

    Scenario: 免稅商品不計稅額
      Given 訂單中只包含免稅商品
      When 系統開立電子發票
      Then 發票稅額應為 0 元

  Rule: 開立失敗時應有重試機制

    Scenario: 財政部 API 暫時不可用時重試
      Given 財政部電子發票 API 暫時回傳 503
      When 系統嘗試開立電子發票
      Then 系統應在 5 分鐘後自動重試
      And 訂單狀態應標記為「發票待開立」

    Scenario: 重試 3 次仍失敗時通知管理員
      Given 系統已重試開立發票 3 次均失敗
      When 第 3 次重試失敗
      Then 系統應發送告警通知給管理員
      And 訂單狀態應標記為「發票開立異常」

  Rule: 買方要求統一編號時應開立公司發票

    Scenario: 開立含統一編號的公司發票
      Given 買方提供統一編號 "12345678"
      When 系統開立電子發票
      Then 發票應包含買方統一編號
      And 發票類別應為「B2B」
```
