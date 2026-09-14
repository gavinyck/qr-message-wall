# 掃碼留言牆

手機掃 QR Code -> 輸入一句話 -> 即時顯示在展示頁面上，不需登入。

- `index.html`：展示牆頁面（放大螢幕上），顯示 QR Code + 即時留言
- `input.html`：手機掃碼後打開的輸入頁面
- `firebase-config.js`：Firebase 專案設定（需自行填入）

## 1. 建立 Firebase 專案

1. 前往 https://console.firebase.google.com ，建立新專案（可關閉 Google Analytics）
2. 左側選單 **Build -> Realtime Database** -> 建立資料庫，選一個離你近的地區
3. 「規則」分頁貼上以下內容並發布（因為不需登入，開放讀寫，但僅限這個小型互動用途）：
   ```json
   {
     "rules": {
       "messages": {
         ".read": true,
         ".write": true
       }
     }
   }
   ```
4. 左側選單 **專案設定（齒輪圖示）-> 一般** -> 往下找「你的應用程式」-> 點網頁圖示 `</>` 新增一個 Web App
5. 複製出現的 `firebaseConfig` 物件，貼進本專案的 `firebase-config.js` 取代裡面的內容

> 注意：這組 config 裡的 apiKey 不是機密金鑰，可以放心公開在前端程式碼裡；真正的存取控制是靠上面的 Realtime Database 規則。

## 2. 放到 GitHub Pages

```bash
cd post_board
git init
git add .
git commit -m "Init QR message wall"
git branch -M main
git remote add origin https://github.com/<你的帳號>/<repo名>.git
git push -u origin main
```

接著到 GitHub repo -> **Settings -> Pages** -> Source 選 `main` branch / `/ (root)` -> Save。
幾分鐘後就會有網址：`https://<你的帳號>.github.io/<repo名>/`

## 3. 使用方式

- 展示頁面打開：`https://.../index.html`（可加 `?room=活動代號` 區分不同場次，例如 `?room=party1`）
- 該頁面會顯示 QR Code，內容自動指向對應的 `input.html?room=活動代號`
- 手機掃碼 -> 輸入文字 -> 送出，展示頁面上會即時跳出新留言

## 已內建的簡單防濫用機制

- 單則訊息長度上限 100 字
- 每個裝置有 5 秒的送出冷卻時間（存在 localStorage）
- 展示頁顯示時會做 HTML 跳脫，避免有人輸入程式碼造成 XSS

如果活動規模大、需要更嚴謹的防灌水或髒話過濾，可以再加強。
