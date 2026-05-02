# Netlify 部署指南

## 📦 部署到 https://cht-retirement-jiang.netlify.app/

### 方法 1：透過 Netlify 網頁介面上傳

1. **登入 Netlify**
   - 前往 [https://app.netlify.com/](https://app.netlify.com/)
   - 使用您的帳號登入

2. **找到您的網站**
   - 在 Sites 列表中找到 `cht-retirement-jiang`
   - 點擊進入網站管理頁面

3. **上傳新版本**
   - 點選 **Deploys** 標籤
   - 將以下檔案/資料夾**拖曳到上傳區域**：
     - `index.html`（主網站）
     - `admin.html`（管理後台）
     - `cht-design-system.css`
     - `assets/` 資料夾（包含所有圖片）
   
4. **等待部署完成**
   - Netlify 會自動處理並部署
   - 通常 1-2 分鐘內完成

5. **測試網站**
   - 前台：https://cht-retirement-jiang.netlify.app/
   - 後台：https://cht-retirement-jiang.netlify.app/admin.html

---

### 方法 2：透過 Netlify CLI（命令列工具）

如果您熟悉命令列：

```bash
# 安裝 Netlify CLI（只需執行一次）
npm install -g netlify-cli

# 登入 Netlify
netlify login

# 在專案資料夾中部署
netlify deploy --prod
```

---

## 📋 需要上傳的檔案清單

```
你的專案資料夾/
├── index.html              ← 前台（退休歡送網站）
├── admin.html              ← 後台（管理介面）
├── cht-design-system.css   ← 樣式檔
└── assets/                 ← 圖片資料夾
    ├── logo_horizontal.png
    ├── mascot_flower.png
    ├── innovation_circle_icon.png
    ├── trust_circle_icon.png
    └── ... (其他圖片)
```

---

## 🔗 部署後的網址

- **前台（報名與留言）**：https://cht-retirement-jiang.netlify.app/
- **後台（管理介面）**：https://cht-retirement-jiang.netlify.app/admin.html

---

## ⚙️ 額外設定（選填）

### 設定 Google Sheets 連結

在 `admin.html` 中找到這行：

```javascript
const GOOGLE_SHEETS_URL = 'YOUR_GOOGLE_SHEETS_URL_HERE';
```

改為您的 Google Sheets 網址：

```javascript
const GOOGLE_SHEETS_URL = 'https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/edit';
```

這樣後台的「開啟 Google Sheets」按鈕就能直接連到您的試算表。

---

## ✅ 部署後檢查清單

- [ ] 前台頁面可以正常開啟
- [ ] 後台頁面可以正常開啟
- [ ] 圖片和樣式正常顯示
- [ ] 測試報名功能（填寫一筆資料）
- [ ] 檢查 Google Sheets 是否收到資料
- [ ] 測試留言功能
- [ ] 後台可以看到統計數據

---

## 🆘 常見問題

### Q: 上傳後圖片不顯示？
A: 確認 `assets/` 資料夾有一起上傳

### Q: CSS 樣式跑掉？
A: 確認 `cht-design-system.css` 有上傳到根目錄

### Q: 後台無法讀取資料？
A: 
1. 檢查 Google Apps Script 是否已部署
2. 確認網址設定正確
3. 檢查 Google Sheets 權限設定

---

## 📞 需要協助？

如有任何問題，請聯繫：
- Email: argus911@cht.com.tw
- 電話: 087200372

---

**祝部署順利！** 🎉
