# Google Apps Script CORS 完整解決方案

## ❌ 問題

錯誤訊息：
```
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

## 🔍 原因

Google Apps Script 從 2020 年起，**不支援直接從網頁前端呼叫**。必須透過特定方式處理。

---

## ✅ 解決方案：使用 JSONP 或代理伺服器

由於 Google Apps Script 的限制，我們需要修改程式碼使用不同的方法。

### 方案 A：修改 Apps Script 使用 JSONP（推薦）

這是最簡單的解決方案。

#### 1. 修改 Google Apps Script

**將您的 Apps Script 程式碼改成以下版本**：

```javascript
function doGet(e) {
  try {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const registrations = getSheetData('報名資料');
    const messages = getSheetData('祝福留言');
    const giftVotes = getGiftVotes();
    
    const result = {
      success: true,
      data: {
        registrations: registrations,
        messages: messages,
        giftVotes: giftVotes
      }
    };
    
    // 支援 JSONP
    const callback = e.parameter.callback || 'callback';
    const jsonString = JSON.stringify(result);
    
    return ContentService
      .createTextOutput(callback + '(' + jsonString + ')')
      .setMimeType(ContentService.MimeType.JAVASCRIPT);
      
  } catch (error) {
    const errorResult = {
      success: false,
      message: error.toString()
    };
    const callback = e.parameter.callback || 'callback';
    return ContentService
      .createTextOutput(callback + '(' + JSON.stringify(errorResult) + ')')
      .setMimeType(ContentService.MimeType.JAVASCRIPT);
  }
}

function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const action = data.action;
    
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    
    if (action === 'submitRegistration') {
      const sheet = ss.getSheetByName('報名資料');
      sheet.appendRow([
        new Date(),
        data.name,
        data.dept,
        data.phone,
        data.email,
        data.mealType,
        data.carpool,
        data.notes || '',
        data.gift || '',
        data.giftSuggestion || ''
      ]);
      
      if (data.gift) {
        updateGiftVotes(data.gift);
      }
      
      return createResponse({ success: true, message: '報名成功' });
      
    } else if (action === 'submitMessage') {
      const sheet = ss.getSheetByName('祝福留言');
      const filesStr = data.files ? data.files.join(', ') : '';
      sheet.appendRow([
        new Date(),
        data.author,
        data.content,
        filesStr
      ]);
      
      return createResponse({ success: true, message: '留言成功' });
    }
    
    return createResponse({ success: false, message: '未知的操作' });
    
  } catch (error) {
    return createResponse({ 
      success: false, 
      message: error.toString() 
    });
  }
}

function createResponse(data) {
  return ContentService
    .createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}

function getSheetData(sheetName) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(sheetName);
  
  if (!sheet) {
    return [];
  }
  
  const data = sheet.getDataRange().getValues();
  
  if (data.length <= 1) {
    return [];
  }
  
  const headers = data[0];
  const rows = data.slice(1);
  
  return rows.map(function(row) {
    const obj = {};
    headers.forEach(function(header, index) {
      obj[header] = row[index];
    });
    return obj;
  });
}

function getGiftVotes() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName('禮品統計');
  
  if (!sheet) {
    return {};
  }
  
  const data = sheet.getDataRange().getValues();
  
  const votes = {};
  for (let i = 1; i < data.length; i++) {
    const giftName = data[i][0];
    const count = data[i][1] || 0;
    if (giftName) {
      votes[giftName] = count;
    }
  }
  
  return votes;
}

function updateGiftVotes(giftName) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName('禮品統計');
  
  if (!sheet) {
    return;
  }
  
  const data = sheet.getDataRange().getValues();
  
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] === giftName) {
      const currentCount = data[i][1] || 0;
      sheet.getRange(i + 1, 2).setValue(currentCount + 1);
      return;
    }
  }
  
  sheet.appendRow([giftName, 1]);
}
```

#### 2. 重新部署

- 儲存程式碼
- 點選「部署」→「管理部署作業」
- 編輯 → 選擇「新版本」→ 部署
- **確認「具有存取權的使用者」設為「任何人」**

#### 3. 測試

在瀏覽器中開啟（加上 `?callback=test`）：
```
https://script.google.com/macros/s/YOUR_ID/exec?callback=test
```

應該會看到：
```javascript
test({"success":true,"data":{...}})
```

---

## 📝 接下來

完成 Apps Script 修改後，我會提供更新的 HTML 檔案來配合 JSONP。

**請先完成上述 Apps Script 的修改並重新部署，然後告訴我結果！**
