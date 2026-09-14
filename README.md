# 掃碼留言牆

手機掃 QR Code -> 輸入一句話 -> 即時顯示在展示頁面上，不需登入。

- `index.html`：展示牆頁面（放大螢幕上），顯示 QR Code + 即時留言
- `input.html`：手機掃碼後打開的輸入頁面
- `admin.html`：管理頁面，可刪除個別留言或清空房間（需管理員登入）
- `firebase-config.js`：Firebase 專案設定（需自行填入）

## 1. 建立 Firebase 專案

1. 前往 https://console.firebase.google.com ，建立新專案（可關閉 Google Analytics）
2. 左側選單 **Build -> Realtime Database** -> 建立資料庫，選一個離你近的地區
3. 規則先暫時用開放版本（第 4 節會換成正式版）：
   ```json
   {
     "rules": {
       "messages": { ".read": true, ".write": true }
     }
   }
   ```
4. 左側選單 **專案設定（齒輪圖示）-> 一般** -> 往下找「你的應用程式」-> 點網頁圖示 `</>` 新增一個 Web App
5. 複製出現的 `firebaseConfig` 物件，貼進本專案的 `firebase-config.js` 取代裡面的內容

> 注意：這組 config 裡的 apiKey 不是機密金鑰，可以放心公開在前端程式碼裡；真正的存取控制是靠 Realtime Database 規則。

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

## 4. 管理頁面與正式的資料庫規則

`admin.html` 可以刪除個別留言或清空整個房間。它用 Firebase 帳號登入，**不是**寫死在前端的密碼——前端密碼任何人按「檢視原始碼」就看得到，擋不住任何人。

### 4-1. 建立管理員帳號

1. Firebase Console 左側 **Build -> Authentication** -> 開始使用
2. **Sign-in method** 分頁 -> 啟用 **電子郵件/密碼**
3. **Users** 分頁 -> **新增使用者**，填入你要用的 Email 和密碼
4. 建好後複製該使用者的 **UID**（列表上那串英數字）

### 4-2. 換上正式規則

回到 **Realtime Database -> 規則**，把 `貼上你的管理員UID` 換成剛剛複製的 UID 後發布：

```json
{
  "rules": {
    "messages": {
      "$room": {
        ".read": true,
        ".write": "auth != null && auth.uid === '貼上你的管理員UID'",
        "$msgId": {
          ".write": "!data.exists() || (auth != null && auth.uid === '貼上你的管理員UID')",
          ".validate": "newData.hasChildren(['text','createdAt'])",
          "text": { ".validate": "newData.isString() && newData.val().length > 0 && newData.val().length <= 100" },
          "createdAt": { ".validate": "newData.isNumber()" },
          "$other": { ".validate": false }
        }
      }
    }
  }
}
```

這組規則的效果：

| 動作 | 誰可以做 |
| --- | --- |
| 讀取留言（展示牆） | 任何人 |
| 新增一則留言 | 任何人（但內容必須是 1~100 字的文字） |
| 修改或刪除留言 | 只有管理員 |
| 清空整個房間 | 只有管理員 |

比起原本的全開放規則，這同時擋掉了「陌生人把你的留言全部刪光」以及「灌入超長內容或奇怪欄位」。

### 4-3. 使用管理頁

打開 `https://.../admin.html`，用管理員帳號登入後即可：

- 切換「房間代號」查看不同場次
- 逐則刪除不當留言
- 「清空這個房間」需要按兩次才會執行，避免誤觸

## 版面：所有留言都看得到

展示牆用多欄磚牆排列，並會自動調整字級讓**全部留言同時出現在畫面上**，不需要捲動：

- 留言少的時候字會放大，把畫面撐滿
- 留言變多就自動縮小、自動增加欄數
- 視窗大小改變時會重新計算

實測（80 則留言時的字級）：

| 螢幕 | 80 則留言的字級 |
| --- | --- |
| 3840×2160 | 39.6px |
| 1920×1080 | 17.6px |
| 1366×768 | 11px |

字級有下限（原始大小的 0.4 倍），所以螢幕偏小又超過一百多則時仍可能塞不下。用大螢幕或投影機效果最好。

## 已內建的簡單防濫用機制

- 單則訊息長度上限 100 字
- 每個裝置有 5 秒的送出冷卻時間（存在 localStorage）
- 展示頁顯示時會做 HTML 跳脫，避免有人輸入程式碼造成 XSS

如果活動規模大、需要更嚴謹的防灌水或髒話過濾，可以再加強。
