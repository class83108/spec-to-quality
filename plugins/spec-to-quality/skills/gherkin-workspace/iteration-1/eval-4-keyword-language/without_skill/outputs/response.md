看起來需要你授權寫入權限。以下是我準備寫入 `tests/features/file_upload.feature` 的內容：

```gherkin
Feature: 使用者檔案上傳
  作為 使用者
  我想要 上傳圖片檔案
  以便 系統能夠儲存並提供存取連結

  Background:
    Given S3 服務已就緒

  Rule: 成功上傳圖片應存至 S3 並回傳 URL

    Scenario Outline: 成功上傳支援的圖片格式
      Given 使用者準備一個 <格式> 圖片檔案
      And 檔案大小為 <大小> MB
      When 使用者上傳該檔案
      Then 系統應將檔案儲存至 S3
      And 系統應回傳該檔案的 S3 URL
      And URL 格式應為有效的 S3 物件路徑

      Examples:
        | 格式 | 大小 |
        | jpg  | 1    |
        | png  | 1    |
        | jpg  | 4.9  |
        | png  | 5    |

  Rule: S3 上傳失敗時應回傳錯誤訊息

    Scenario: S3 服務不可用時回傳錯誤
      Given 使用者準備一個 jpg 圖片檔案
      And S3 服務暫時不可用
      When 使用者上傳該檔案
      Then 系統應回傳上傳失敗錯誤
      And 錯誤訊息應包含 "儲存失敗"

    Scenario: S3 上傳逾時時回傳錯誤
      Given 使用者準備一個 jpg 圖片檔案
      And S3 服務回應逾時
      When 使用者上傳該檔案
      Then 系統應回傳上傳失敗錯誤
      And 錯誤訊息應包含 "上傳逾時"

  Rule: 檔案大小超過 5MB 應拒絕上傳

    Scenario: 檔案大小剛好超過上限
      Given 使用者準備一個 jpg 圖片檔案
      And 檔案大小為 5.1 MB
      When 使用者上傳該檔案
      Then 系統應拒絕上傳
      And 錯誤訊息應包含 "檔案大小超過 5MB 上限"

    Scenario: 檔案大小遠超過上限
      Given 使用者準備一個 png 圖片檔案
      And 檔案大小為 50 MB
      When 使用者上傳該檔案
      Then 系統應拒絕上傳
      And 錯誤訊息應包含 "檔案大小超過 5MB 上限"

  Rule: 不支援的檔案格式應拒絕上傳

    Scenario Outline: 拒絕非圖片格式
      Given 使用者準備一個 <格式> 檔案
      And 檔案大小為 1 MB
      When 使用者上傳該檔案
      Then 系統應拒絕上傳
      And 錯誤訊息應包含 "僅支援 jpg 或 png 格式"

      Examples:
        | 格式 |
        | gif  |
        | pdf  |
        | txt  |
        | exe  |

    Scenario: 拒絕偽裝副檔名的非圖片檔
      Given 使用者準備一個副檔名為 jpg 但內容為 txt 的檔案
      When 使用者上傳該檔案
      Then 系統應拒絕上傳
      And 錯誤訊息應包含 "檔案格式不符"

  Rule: 空檔案應拒絕上傳

    Scenario: 檔案大小為 0 bytes
      Given 使用者準備一個空的 jpg 檔案
      When 使用者上傳該檔案
      Then 系統應拒絕上傳
      And 錯誤訊息應包含 "檔案不可為空"
```

**覆蓋摘要：**

| 類別 | Scenarios |
|------|-----------|
| Happy path | 成功上傳 jpg/png（含邊界 5MB） |
| Error paths | S3 不可用、S3 逾時 |
| Input boundary | 超過 5MB（剛超過 + 遠超過）、不支援格式（gif/pdf/txt/exe）、偽裝副檔名、空檔案 |

請授權寫入後我會建立檔案。
