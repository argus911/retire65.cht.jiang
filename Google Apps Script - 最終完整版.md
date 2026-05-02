# Google Apps Script - 最終完整版本

## 完整程式碼（請複製全部）

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
    
    // 支援 JSONP 用於讀取資料
    const callback = e.parameter.callback;
    if (callback) {
      const jsonString = JSON.stringify(result);
      return ContentService
        .createTextOutput(callback + '(' + jsonString + ')')
        .setMimeType(ContentService.MimeType.JAVASCRIPT);
    }
    
    // 一般 JSON 回應
    return ContentService
      .createTextOutput(JSON.stringify(result))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    const errorResult = {
      success: false,
      message: error.toString()
    };
    
    const callback = e.parameter.callback;
    if (callback) {
      return ContentService
        .createTextOutput(callback + '(' + JSON.stringify(errorResult) + ')')
        .setMimeType(ContentService.MimeType.JAVASCRIPT);
    }
    
    return ContentService
      .createTextOutput(JSON.stringify(errorResult))
      .setMimeType(ContentService.MimeType.JSON);
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

---

## 設定步驟

1. **複製上方所有程式碼**
2. **貼到 Apps Script 編輯器**（刪除舊的）
3. **儲存**
4. **部署**：
   - 點「部署」→「管理部署作業」
   - 點鉛筆 ✏️ 編輯
   - 選「新版本」
   - **確認「具有存取權的使用者」= 「任何人」**
   - 點「部署」

---

## 測試

在瀏覽器開啟：
```
https://script.google.com/macros/s/YOUR_ID/exec?callback=test
```

應該看到：
```javascript
test({"success":true,"data":{...}})
```

完成後告訴我！
