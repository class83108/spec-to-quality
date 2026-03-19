# Design Review 檢視維度詳細說明

對每個維度，閱讀相關 code 後判斷是否有問題。不是每個維度都一定有問題——沒問題的標記「OK」即可。

## 1. 職責分離 (Single Responsibility)
- 一個 class/function 是否只做一件事？
- 有沒有 function 同時在做 IO（DB/API）和業務邏輯？
- 改動一個需求時，是否只需要改一個地方？

## 2. 依賴方向 (Dependency Direction)
- 內層（domain/business logic）是否依賴了外層（framework/infrastructure）？
- 有沒有 import 方向違反架構層級？
- Protocol/interface 是否定義在正確的層？

## 3. 命名語意 (Naming)
- function/class/variable 的名稱是否準確反映其行為？
- 有沒有過於泛化的命名（`process_data`, `handle_request`, `utils`）？
- 命名是否與團隊/專案中已有的慣例一致？

## 4. 錯誤處理策略 (Error Handling)
- exception 是在正確的層被 catch 的嗎？
- 是 fail fast（早期檢查）還是 defensive programming（到處 try/except）？
- 錯誤訊息是否提供足夠的 debug 資訊？

## 5. 可測試性 (Testability)
- 如果要為這段 code 補更多測試，容易嗎？
- 有沒有隱藏的依賴（global state, datetime.now(), random）讓測試困難？
- 是否有需要 mock 太多東西才能測試的 function？

## 6. 一致性 (Consistency)
- 與專案中類似功能的實作方式是否一致？
- 有沒有同一件事在不同地方用不同方式做？
- 新 code 是否遵循專案已建立的 pattern？
