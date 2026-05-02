# Google Sheets 資料庫設定步驟

## 📋 步驟一：建立 Google 試算表

1. 前往 [Google Sheets](https://sheets.google.com)
2. 使用您的帳號 **argus911@gmail.com** 登入
3. 建立新的試算表，命名為：**蔣惠軫退休歡送會報名資料**
4. 建立三個工作表（Sheet）：
   - **報名資料**
   - **祝福留言**
   - **禮品統計**

---

## 📝 步驟二：設定工作表欄位

### 工作表 1：報名資料

在第一列（標題列）輸入以下欄位：

| A | B | C | D | E | F | G | H | I | J |
|---|---|---|---|---|---|---|---|---|---|
| 時間戳記 | 姓名 | 部門 | 電話 | Email | 餐點 | 共乘 | 備註 | 禮品選擇 | 禮品建議 |

### 工作表 2：祝福留言

在第一列（標題列）輸入以下欄位：

| A | B | C | D |
|---|---|---|---|
| 時間戳記 | 姓名 | 留言內容 | 附件檔名 |

### 工作表 3：禮品統計

在第一列（標題列）輸入以下欄位：

| A | B |
|---|---|
| 禮品名稱 | 票數 |

然後在下方輸入預設禮品：
- A2: `高級溫控手沖壺`，B2: `0`
- A3: `智慧手錶`，B3: `0`
- A4: `平板電腦`，B4: `0`

---

## 🔧 步驟三：建立 Google Apps Script

1. 在試算表中，點選 **擴充功能** → **Apps Script**
2. 刪除預設的 `myFunction()` 程式碼
3. 複製以下完整程式碼並貼上：

```javascript
// 設定允許的來源（CORS）
const ALLOWED_ORIGINS = [
  'https://claude.site',
  'http://localhost',
  'https://localhost'
];

function doPost(e) {
  try {
    // CORS 處理
    const origin = e.parameter.origin || '';
    
    // 解析傳入的 JSON 資料
    const data = JSON.parse(e.postData.contents);
    const action = data.action;
    
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    
    if (action === 'submitRegistration') {
      // 新增報名資料
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
      
      // 更新禮品統計
      if (data.gift) {
        updateGiftVotes(data.gift);
      }
      
      return createResponse({ success: true, message: '報名成功' });
      
    } else if (action === 'submitMessage') {
      // 新增祝福留言
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
      // 讀取所有資料（供管理後台使用）
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
  // 處理 GET 請求（供管理後台讀取資料）
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

// 讀取工作表資料
function getSheetData(sheetName) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(sheetName);
  const data = sheet.getDataRange().getValues();
  
  if (data.length <= 1) return []; // 只有標題列
  
  const headers = data[0];
  const rows = data.slice(1);
  
  return rows.map(row => {
    const obj = {};
    headers.forEach((header, index) => {
      obj[header] = row[index];
    });
    return obj;
  });
}

// 讀取禮品統計
function getGiftVotes() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName('禮品統計');
  const data = sheet.getDataRange().getValues();
  
  const votes = {};
  for (let i = 1; i < data.length; i++) {
    const giftName = data[i][0];
    const count = data[i][1] || 0;
    votes[giftName] = count;
  }
  
  return votes;
}

// 更新禮品票數
function updateGiftVotes(giftName) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName('禮品統計');
  const data = sheet.getDataRange().getValues();
  
  // 尋找該禮品
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] === giftName) {
      const currentCount = data[i][1] || 0;
      sheet.getRange(i + 1, 2).setValue(currentCount + 1);
      return;
    }
  }
  
  // 如果找不到，新增一列
  sheet.appendRow([giftName, 1]);
}

// 建立回應
function createResponse(data) {
  return ContentService
    .createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}
```

4. 點選 **儲存** 圖示（💾），命名為：`退休歡送會API`

---

## 🚀 步驟四：部署為網路應用程式

1. 點選右上角 **部署** → **新增部署作業**
2. 點選齒輪圖示 ⚙️ 選擇類型：**網路應用程式**
3. 設定如下：
   - **說明**：退休歡送會資料 API
   - **執行身分**：我（您的 Email）
   - **具有存取權的使用者**：**任何人**（重要！）
4. 點選 **部署**
5. **重要**：複製 **網路應用程式網址**（類似：`https://script.google.com/macros/s/XXXXX/exec`）
6. 點選 **完成**

---

## 🔑 步驟五：取得部署網址

部署後，您會看到一個網址，格式如下：

```
https://script.google.com/macros/s/AKfycbxXXXXXXXXXXXXXXXXXXXXXX/exec
```

**請複製這個網址，並回傳給我**，我會更新網站程式碼。

---

## ⚠️ 授權提示

第一次執行時，Google 會要求授權：

1. 點選 **查看權限**
2. 選擇您的 Google 帳號（argus911@gmail.com）
3. 點選 **進階** → **前往「退休歡送會API」（不安全）**
4. 點選 **允許**

這是正常的，因為這是您自己建立的腳本。

---

## 📊 完成後

設定完成後：
- ✅ 所有報名資料會自動寫入 Google Sheets
- ✅ 所有祝福留言會自動儲存
- ✅ 禮品統計會自動更新
- ✅ 管理後台可以即時讀取資料
- ✅ 跨裝置、跨瀏覽器都能同步

---

## 📞 如需協助

如果設定過程中遇到問題，請隨時告訴我！

請完成步驟後，將 **部署網址** 提供給我，我會立即更新網站程式碼。
