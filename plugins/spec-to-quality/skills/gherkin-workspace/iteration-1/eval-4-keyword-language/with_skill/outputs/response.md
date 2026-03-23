已撰寫 `tests/features/file_upload.feature`，結構如下：

**Rule 1: 只接受有效的圖片檔案格式與大小**
- ✅ Happy path: 成功上傳 JPG / PNG 圖片（各一個 scenario）
- ✅ Input boundary: 超過 5MB、不支援格式（GIF）、空檔案

**Rule 2: 系統應妥善處理 S3 上傳失敗**
- ✅ Error path: S3 服務無法連線

---

請確認幾點是否需要調整：

1. **邊界值 5MB 本身** — 要不要加一個「剛好 5MB 應該成功」的 scenario？
2. **錯誤訊息的具體內容** — 目前只描述「應說明 X」，如果有明確的 error code 或 message 格式可以更具體
3. **其他格式測試** — 除了 GIF，要不要加 PDF 或 exe 等明顯非圖片格式？
