這個 eval-workspace 目錄不是 git repository。請問你的專案主目錄在哪裡？或者要我在這裡初始化一個新的 git repo？

關於你提到的 verification.md 內容，我看到了：

**等級 2 項目（需要整合測試）：**
1. **WebRTC offer/answer 交換** - FastRTC signaling 行為無法用 fake 完整模擬，最低驗證方式是本地跑 app + 瀏覽器手動測試
2. **GeminiHandler Live API session** - Gemini Live API 行為不可預測，最低驗證方式是 staging 環境 smoke test

這兩項已在 verification.md 標註需要整合測試，commit message 會註明這是 conversation 模組的最後一個 spec。
