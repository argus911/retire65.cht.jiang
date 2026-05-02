# Google Apps Script 程式碼（修正版）

請完整複製以下程式碼，貼到您的 Google Apps Script 編輯器中：

```javascript
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
      
    } else if (action === 'getData') {
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
    }
    
    return createResponse({ success: false, message: '未知的操作' });
    
  } catch (error) {
    return createResponse({ 
      success: false, 
      message: error.toString() 
    });
  }
}

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

function createResponse(data) {
  return ContentService
    .createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## 🔧 重新設定步驟

1. **開啟您的 Google Apps Script**（在試算表：擴充功能 → Apps Script）
2. **刪除所有現有程式碼**
3. **完整複製上方程式碼**（從 `function doPost` 到最後）
4. **貼上到編輯器**
5. **點選「儲存」**（💾 圖示）
6. **重新部署**：
   - 點選右上角「部署」→「管理部署作業」
   - 點選鉛筆圖示 ✏️ 編輯
   - 在「版本」下拉選單選擇「新版本」
   - 點選「部署」
7. **完成！**

---

## ⚠️ 常見問題

如果還是有錯誤：
- 確認三個工作表名稱完全正確：`報名資料`、`祝福留言`、`禮品統計`
- 確認每個工作表都有標題列（第一列）
- 確認「禮品統計」工作表有預設的三個禮品名稱

需要我提供更詳細的協助嗎？
