# Google Apps Script CORS 問題修正

## 問題現象

後台顯示錯誤：`Failed to fetch`

這是因為 Google Apps Script 的 CORS（跨來源資源共用）設定問題。

---

## ✅ 解決方案

### 步驟 1：檢查 Apps Script 部署設定

1. **開啟 Google Apps Script**
   - 在您的 Google Sheets：擴充功能 → Apps Script

2. **點選右上角「部署」→「管理部署作業」**

3. **檢查設定**：
   - ✅ **具有存取權的使用者**：必須選擇 **「任何人」**
   - ❌ 如果是「僅限我自己」，會導致 CORS 錯誤

4. **如果設定錯誤**：
   - 點選鉛筆圖示 ✏️ 編輯
   - 改為「任何人」
   - 選擇「新版本」
   - 點選「部署」

---

### 步驟 2：確認 Apps Script 程式碼

確認您的 Apps Script 中有 `doGet` 函數（用於讀取資料）：

```javascript
function doGet(e) {
  try {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const registrations = getSheetData('報名資料');
    const messages = getSheetData('祝福留言');
    const giftVotes = getGiftVotes();
    
    return createResponse({
      success: true,
      data: {
        registrations: registrations,
        messages: messages,
        giftVotes: giftVotes
      }
    });
  } catch (error) {
    return createResponse({ 
      success: false, 
      message: error.toString() 
    });
  }
}
```

---

### 步驟 3：測試 Apps Script 網址

1. **複製您的部署網址**：
   ```
   https://script.google.com/macros/s/AKfycbzqA-9dRvDdibt6veSpMGQagAi0pEvKd5xXcuzCKwIkMeJgmUAR2JbIH1KfjV-cYZWtyQ/exec
   ```

2. **在瀏覽器中直接開啟這個網址**

3. **應該看到 JSON 格式的回應**：
   ```json
   {
     "success": true,
     "data": {
       "registrations": [],
       "messages": [],
       "giftVotes": {}
     }
   }
   ```

4. **如果看到錯誤或要求登入**：
   - 表示權限設定錯誤
   - 回到步驟 1 重新設定

---

### 步驟 4：重新部署網站

修正 Apps Script 後：

1. **重新下載更新後的檔案**：
   - `index.html`
   - `admin.html`

2. **上傳到 Netlify**（拖放上傳）

3. **測試後台**：
   - 開啟 https://cht-retirement-jiang.netlify.app/admin.html
   - 應該能正常載入資料

---

## 🔍 除錯技巧

### 在瀏覽器中檢查錯誤

1. 按 **F12** 開啟開發者工具
2. 切換到 **Console** 標籤
3. 重新整理頁面
4. 查看詳細的錯誤訊息

### 常見錯誤與解決方法

| 錯誤訊息 | 原因 | 解決方法 |
|---------|------|---------|
| `Failed to fetch` | CORS 問題 | 確認 Apps Script 權限設定為「任何人」 |
| `Redirect` | 需要授權 | 用無痕模式開啟 Apps Script 網址並授權 |
| `404 Not Found` | 網址錯誤 | 確認 GOOGLE_SCRIPT_URL 正確 |
| `Script not deployed` | 未部署 | 重新部署 Apps Script |

---

## 📞 還是無法解決？

請提供以下資訊：

1. 在瀏覽器直接開啟 Apps Script 網址時看到什麼？
2. F12 Console 中的完整錯誤訊息
3. Apps Script 部署設定的截圖

我會協助您進一步診斷問題！
