# 掃碼留言牆

手機掃 QR Code -> 輸入一句話 -> 即時顯示在展示頁面上，不需登入。

- `index.html`：展示牆頁面（放大螢幕上），顯示 QR Code + 即時留言
- `input.html`：手機掃碼後打開的輸入頁面
- `admin.html`：管理頁面，可建立房間、取得網址、刪除留言（需管理員登入）
- `firebase-config.js`：Firebase 專案設定（需自行填入）
- `database.rules.json`：資料庫安全規則，貼到 Firebase Console 用

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

> 注意：這組 config 裡的 apiKey 不是機密金鑰，可以放心公開在前端程式碼裡；真正的存取控制是靠 Realtime Database 規則。詳見下方「關於 GitHub 的金鑰外洩警告」。

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

回到 **Realtime Database -> 規則**，貼上 `database.rules.json` 的內容，把當中**三處** `PASTE_ADMIN_UID_HERE` 都換成剛剛複製的 UID，然後發布。

這組規則的效果：

| 動作 | 誰可以做 |
| --- | --- |
| 讀取留言（展示牆） | 任何人 |
| 新增一則留言 | 任何人（但內容必須是 1~100 字的文字） |
| 修改或刪除留言 | 只有管理員 |
| 清空整個房間 | 只有管理員 |
| 讀取／建立／刪除房間 | 只有管理員 |

比起全開放規則，這同時擋掉了「陌生人把你的留言全部刪光」以及「灌入超長內容或奇怪欄位」。

### 4-3. 使用管理頁

打開 `https://.../admin.html`，用管理員帳號登入後即可：

**建立房間**：輸入房間代號（英數字、`-`、`_`，最多 40 字）和活動名稱，按建立。每個房間的留言完全獨立，適合一場活動一個房間。

**取得網址**：每個房間會列出兩個網址，各有複製按鈕——

- **展示牆**：投影或大螢幕上要開的頁面，會自動顯示對應的 QR Code
- **手機輸入**：QR Code 實際指向的頁面，通常不用手動分享

**管理留言**：按「管理留言」載入該房間的留言，可逐則刪除，或用「清空這個房間」一次清掉。

**測試工具**：留言區最下方可以一次產生指定筆數（預設 100）的假留言，用來在正式活動前確認展示牆的排版效果。產生的是真實資料，會寫進目前選取的房間，用完記得清空——建議開一個專用的測試房間再用。

刪除房間、清空留言和產生測試留言都需要**按兩次**才會執行，避免誤觸；5 秒內沒有再按就會自動取消。

> 預設房間 `main` 不需要建立就能使用（展示牆網址不帶參數時就是它）。若要在管理頁操作沒有建立過的房間，用最下方「直接管理其他代號」輸入即可。

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

- 單則訊息長度上限 100 字（前端擋一次，資料庫規則再擋一次）
- 每個裝置有 5 秒的送出冷卻時間（存在 localStorage）
- 留言一律以純文字輸出，避免有人輸入程式碼造成 XSS

如果活動規模大、需要更嚴謹的防灌水或髒話過濾，可以再加強。

## 關於 GitHub 的金鑰外洩警告

GitHub 的 secret scanning 會對 `firebase-config.js` 裡的 `apiKey` 發出 **Google API Key** 警告。**這是誤判，不要輪替（rotate）這把金鑰**——換掉只會讓網站連不上 Firebase，擋不到任何人。

原因：Firebase 網頁版的 `apiKey` 是**公開的專案識別碼，不是密碼**。瀏覽器必須帶著它才能連到你的專案，所以它一定會出現在前端程式碼裡；就算從 repo 移除，打開網頁按「檢視原始碼」照樣看得到。GitHub 只認得 `AIza...` 這個格式就報警，分不出是哪一種 Google 金鑰。

真正保護資料的是 **Realtime Database 規則**（第 4 節）。Realtime Database 的存取完全不經過這把金鑰——帶不帶金鑰都一樣受規則管轄。就算有人拿這把金鑰註冊了帳號，規則比對的是 `auth.uid === 管理員UID`，所以他們只能跟一般訪客一樣送出留言，**不能刪除留言，也看不到房間清單**。

### 該做的事

1. **關閉公開註冊**（重要）：Firebase Console -> **Authentication -> 設定 -> 使用者動作**，取消勾選 **啟用建立（註冊）**。

   只需要一個管理員帳號，建好之後就該關掉；否則任何人都能用這把公開金鑰在你的專案裡註冊帳號、灌爆使用者清單。

2. **限制金鑰來源網域**（縱深防禦）：Google Cloud Console -> **API 和服務 -> 憑證** -> 點 `Browser key (auto created by Firebase)` -> **應用程式限制** 選 **HTTP 參照網址**，加入你的 GitHub Pages 網域，例如 `你的帳號.github.io/*`。

   這保護的是 Authentication 等 Google API，對 Realtime Database 無效（它本來就不看這把金鑰）。

3. **關閉 GitHub 警告**：repo 的 **Security -> Secret scanning**，把該警告關閉，原因選 **False positive**。

### 怎麼自己驗證公開註冊是否已關閉

**最可靠也最安全的方法是直接看 Console**：Authentication -> 設定 -> 使用者動作，確認「啟用建立（註冊）」沒有被勾選。

若要用指令再確認一次：

```bash
KEY="你的 apiKey"
curl -s -X POST "https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=$KEY" \
  -H 'Content-Type: application/json' \
  -H 'Referer: https://你的帳號.github.io/' \
  -d '{"email":"你已註冊的管理員Email","password":"aVeryLongPassword123!","returnSecureToken":true}'
```

- `ADMIN_ONLY_OPERATION` -> 公開註冊已關閉（正確）
- `EMAIL_EXISTS` 或成功回傳 -> 公開註冊還開著，請回到上面第 1 步

⚠️ **兩個容易踩到的坑**：

1. **密碼一定要夠長（6 字以上）**。Firebase 會先檢查密碼格式、再檢查註冊是否開放，所以短密碼一律回 `WEAK_PASSWORD`，**不管註冊有沒有關閉**——拿它來判斷會得到錯誤結論。
2. **Email 要填你已經註冊過的那個管理員帳號**。若填一個沒用過的 Email，而註冊其實還開著，這個請求就會**真的幫你建立一個帳號**。用已存在的 Email 就不會有這個副作用。

如果金鑰已設定來源限制，記得像上面那樣帶 `Referer`，否則會先被來源限制擋掉而看不到真正的結果。
