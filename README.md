# 議題觀察室：部署說明

這個網站分成兩部分：

| 部分 | 放在哪裡 | 檔案 | 負責什麼 |
|---|---|---|---|
| 前端網頁 | GitHub Pages | `index.html` | 讀者看到的網站 |
| 後端 API | Google Apps Script | `Code.gs` | 讀取雲端硬碟文章、存留言、驗證登入 |

讀者用「使用 Google 帳號登入」按鈕登入後才能留言。後端會向 Google 驗證登入憑證，所以留言顯示的 E-mail 是真的，不能偽造。

整個設定大約 20 分鐘，請**照順序**做：先在 Google Cloud 拿到用戶端 ID，再部署 Apps Script，最後上傳到 GitHub。

---

## 步驟 0：先決定 GitHub 網址

GitHub Pages 的網址格式是 `https://<你的 GitHub 帳號>.github.io/<儲存庫名稱>/`。

例如帳號是 `amy-chen`、儲存庫叫 `issues`，網址就是 `https://amy-chen.github.io/issues/`。

步驟 1 會用到「來源」：**只取到 `.github.io` 為止**，也就是 `https://amy-chen.github.io`，不要加後面的路徑和斜線。

---

## 步驟 1：Google Cloud 建立 OAuth 用戶端 ID（免費）

1. 打開 <https://console.cloud.google.com/>，用你的 Google 帳號登入。
2. 上方「選取專案」→「新增專案」，名稱隨意（例如 `issues-site`）→ 建立，並確認已切換到這個專案。
3. 左側選單找「Google Auth Platform」（或搜尋列輸入「OAuth」）。第一次進入會要求設定應用程式：
   - **應用程式名稱**：例如「議題觀察室」（讀者登入時會看到）
   - **使用者支援電子郵件**：你的 E-mail
   - **目標對象（Audience）**：選「**外部**」
   - **聯絡資訊**：你的 E-mail
   - 同意政策 → 建立
4. 到「目標對象（Audience）」頁面，把發布狀態從「測試中」改成「**正式版（發布應用程式）**」。
   - 如果停在「測試中」，只有你手動加入的測試使用者能登入。
   - 本網站只取得 E-mail 和名稱這類基本資料，不需要送 Google 審核。
5. 到「用戶端（Clients）」頁面 →「建立用戶端」：
   - **應用程式類型**：網頁應用程式
   - **名稱**：隨意
   - **已授權的 JavaScript 來源**：
     - `https://<你的帳號>.github.io`
     - 想在自己電腦測試的話，再加 `http://localhost` 和 `http://localhost:8000`
   - **已授權的重新導向 URI**：留空
   - 按「建立」
6. 複製畫面上的「**用戶端 ID**」，格式像 `1234567890-abcdefg.apps.googleusercontent.com`。用戶端密碼用不到，不必理會。

> 設定改完後，Google 有時要等幾分鐘才會生效。如果按登入出現「origin not allowed」，等一下再試。

---

## 步驟 2：部署 Apps Script 後端

1. 打開原本的 Apps Script 專案（或到 <https://script.google.com/> 新增一個）。
2. 用新的 `Code.gs` 整份**覆蓋**原本的內容。
3. 修改檔案最上方的 `CONFIG`：
   ```js
   FOLDER_ID: '你的雲端硬碟資料夾 ID',
   CLIENT_ID: '步驟 1 複製的用戶端 ID',
   ADMINS: ['你的 E-mail（小寫）'],   // 可填多個，用逗號分隔
   ```
4. 專案裡原本的 `Index.html` 可以刪除，網頁改放在 GitHub。左側「服務」裡的 Drive API 也不再需要，可以移除。
5. 右上「部署」→「新增部署作業」→ 齒輪選「網頁應用程式」：
   - **執行身分**：**我**
   - **誰可以存取**：**任何人**（注意是「任何人」，**不是**「任何擁有 Google 帳戶的使用者」）
   - 按「部署」，第一次會要求授權，依畫面允許即可。
6. 複製「網頁應用程式網址」，格式是 `https://script.google.com/macros/s/……/exec`。
7. 用瀏覽器直接打開這個網址，看到 `{"ok":true,"data":"議題觀察室 API 運作中"}` 就代表成功。

> **之後修改 Code.gs 的重要提醒**：要到「部署」→「管理部署作業」→ 點鉛筆 →「版本」選「新版本」→ 部署。
> 這樣網址不會變。如果按「新增部署作業」，會產生新網址，前端就得跟著改。

因為以「我」的身分執行，文章資料夾**不需要**分享給任何人，保持私人即可。讀者只能透過網站讀取資料夾裡的 PDF / PNG / JPG。

---

## 步驟 3：上傳到 GitHub Pages

1. 打開 `index.html`，修改最前面 `<script>` 裡的 `CONFIG`：
   ```js
   API_URL: '步驟 2 複製的 /exec 網址',
   GOOGLE_CLIENT_ID: '步驟 1 複製的用戶端 ID'
   ```
2. 到 <https://github.com/new> 建立儲存庫：
   - 名稱用步驟 0 決定的名稱
   - 選 **Public**（免費方案的 GitHub Pages 需要公開儲存庫）
   - 建立
3. 在儲存庫頁面按「Add file」→「Upload files」，拖入 `index.html`，也可以一起放 `README.md`。**不要上傳 `Code.gs`**，它只放在 Apps Script。按「Commit changes」。
4. 到儲存庫的「Settings」→ 左側「Pages」：
   - Source：**Deploy from a branch**
   - Branch：`main`，資料夾 `/ (root)` → Save
5. 等 1～2 分鐘，重新整理 Pages 設定頁，上方會出現網址，打開就能看到網站。

之後要改網頁，直接在 GitHub 編輯 `index.html` 再 Commit，約一分鐘後生效。

---

## 步驟 4：上線後檢查

- [ ] 首頁有出現文章和議題分類
- [ ] 點文章可以閱讀 PDF，「下載 PDF」可以下載
- [ ] 右上角有「使用 Google 帳號登入」按鈕，登入後顯示你的 E-mail
- [ ] 用管理員帳號登入後，有「投稿」按鈕和摘要的「編輯」
- [ ] 能送出留言，留言顯示 E-mail；可以刪除自己的留言
- [ ] 用另一個非管理員帳號（或無痕視窗）登入，確認看不到「投稿」和「編輯」

---

## 常見問題

**網頁顯示「伺服器回應格式錯誤」**
Apps Script 的存取權不是「任何人」，或網址不是 `/exec` 結尾。請重新檢查步驟 2 的第 5、6 項。

**改了 Code.gs 卻沒生效**
沒有發布新版本。請照步驟 2 最後的提醒，在「管理部署作業」裡選「新版本」。

**登入按鈕沒出現，或出現「origin_mismatch」、「not allowed」**
步驟 1 的「已授權的 JavaScript 來源」要和網址的開頭完全一樣（`https://帳號.github.io`，不含路徑、結尾不要斜線）。改完要等幾分鐘才會生效。

**只有我自己能登入，別人不行**
OAuth 應用程式還在「測試中」，請照步驟 1 的第 4 項改成「正式版」。

**我登入了，卻沒有管理員功能**
`Code.gs` 的 `ADMINS` 要用小寫 E-mail，改完要發布新版本。

**檔案很大打不開**
單檔上限 20MB（`Code.gs` 的 `MAX_BYTES`）。大檔要傳輸比較久，建議壓縮 PDF 後再放進資料夾。

**留言會存在哪裡、能存多少？**
存在 Apps Script 的「指令碼屬性」，總容量約 500KB，大約可放 1000～2000 則短留言。
到 Apps Script「專案設定」→「指令碼屬性」可以看到每一則留言（`c_` 開頭）。

**分享單篇文章**
打開文章後，網址會帶上 `#檔案ID`，直接複製網址分享即可。# taiwan
