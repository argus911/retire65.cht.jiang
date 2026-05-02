# Google Apps Script - 修正「javascript is not defined」錯誤

## ❌ 錯誤原因

您的 Apps Script 程式碼第 1 行有錯誤。這通常是因為：
1. 複製貼上時格式跑掉
2. 多餘的註解或文字
3. 程式碼前面有隱藏字元

---

## ✅ 解決方法

### 步驟 1：完全清空 Apps Script

1. 開啟 Google Apps Script 編輯器
2. **全選** (Ctrl+A / Cmd+A)
3. **刪除** 所有內容
4. 確認編輯器**完全空白**

### 步驟 2：複製以下程式碼（完整版）

**重要：從第一個 `function` 開始複製，不要複製前面的任何文字**

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
    
    const callback = e.parameter.callback;
    if (callback) {
      return ContentService
        .createTextOutput(callback + '(' + JSON.stringify(result) + ')')
        .setMimeType(ContentService.MimeType.JAVASCRIPT);
    }
    
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
      
      return ContentService
        .createTextOutput(JSON.stringify({ success: true, message: '報名成功' }))
        .setMimeType(ContentService.MimeType.JSON);
      
    } else if (action === 'submitMessage') {
      const sheet = ss.getSheetByName('祝福留言');
      const filesStr = data.files ? data.files.join(', ') : '';
      sheet.appendRow([
        new Date(),
        data.author,
        data.content,
        filesStr
      ]);
      
      return ContentService
        .createTextOutput(JSON.stringify({ success: true, message: '留言成功' }))
        .setMimeType(ContentService.MimeType.JSON);
    }
    
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, message: '未知的操作' }))
      .setMimeType(ContentService.MimeType.JSON);
    
  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, message: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
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

### 步驟 3：貼上並儲存

1. **貼上**以上程式碼到**完全空白**的編輯器
2. 點選 **儲存** (💾)
3. 確認沒有任何錯誤訊息

### 步驟 4：重新部署

1. 點選 **部署** → **管理部署作業**
2. 點鉛筆 ✏️ **編輯**
3. 選擇 **新版本**
4. **確認「具有存取權的使用者」= 「任何人」**
5. 點選 **部署**

### 步驟 5：測試

在瀏覽器開啟：
```
https://script.google.com/macros/s/AKfycbzqA-9dRvDdibt6veSpMGQagAi0pEvKd5xXcuzCKwIkMeJgmUAR2JbIH1KfjV-cYZWtyQ/exec?callback=test
```

應該看到：
```javascript
test({"success":true,"data":{...}})
```

---

## 完成後告訴我結果！

如果還是有錯誤，請告訴我：
1. 完整的錯誤訊息
2. 錯誤在第幾行
3. 可能的話，截圖整個 Apps Script 編輯器
