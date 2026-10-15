# 國中家教學習管理系統（MVP 第一版）

## 功能
- 老師：切換學生，新增、編輯、刪除課程進度、作業、成績、教材連結、錯題紀錄。
- 學生：帳密登入，只能查閱自己的資料；可以標記自己的作業完成。
- 成績：不同滿分換算百分比的折線圖。
- 手機與電腦響應式頁面。

## 架構
靜態網頁 GitHub Pages + Firebase Authentication (Email/Password) + Cloud Firestore。
教材使用 HTTPS 連結（如 Google Drive 共用連結），本版**不含檔案直接上傳**。

## Firebase 設定
1. 開啟 https://console.firebase.google.com/ 建立專案。
2. 新增 Web App，取得 Firebase config，複製 `firebase-config.example.js` 為 `firebase-config.js` 並填入設定。
3. Authentication → Sign-in method → 啟用 Email/Password。
4. Firestore Database → 建立資料庫，選擇適合地區。將 `firestore.rules` 的**完整內容**貼到 Rules 並發布。不要用公開讀寫測試規則。
5. Authentication → Users → Add user：先建立老師帳號及每個學生的帳號，複製每個帳號的 UID。
6. Firestore → 建立 `users` 集合。文件 ID 必須**完全等於 Authentication 的 UID**。
   - 老師文件範例：`{ "name": "老師", "role": "teacher" }`
   - 學生文件範例：`{ "name": "小明", "role": "student" }`
   - 注意 Firestore 控制台需逐欄新增，不是直接貼 JSON；`name` 和 `role` 都是 string。
7. 登入老師帳號。從右上方下拉選學生，再新增資料。

### 安全注意事項
- `users` 集合只允許從 Firebase 控制台／可信任管理後端編輯，前端不能自行提升為老師。
- Firebase 網頁 config 的 apiKey 是用於辨識 Firebase 專案的公開設定，**不是管理員私鑰**；不要將 service account JSON 或任何私密金鑰上傳 GitHub。
- 所有學生須使用個別密碼。若需自助重設密碼、老師後台建立帳號、批次匯入、作業上傳，需要後續版本加上受保護的管理後端／Cloud Functions。
- 本版教師可以存取所有紀錄；學生 Firestore 查詢依 `studentId == auth.uid` 限制。安全規則必須發布才有保護效果。
- 請確認學生與家長知悉學習資料的蒐集用途；不要記錄不必要的敏感資料。

## 部署 GitHub Pages
1. 建立 GitHub repository，例如 `tutor-portal`。
2. 上傳 `index.html`、`firebase-config.js`（可上傳網頁 config）、`README.md`。`firestore.rules` 是部署到 Firebase 控制台的，不會由 GitHub Pages 自動套用。
3. GitHub repo → Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)` → Save。
4. 網址通常為 `https://你的帳號.github.io/tutor-portal/`。
5. Firebase Authentication → Settings → Authorized domains：加入 `你的帳號.github.io`（只填主機名稱，不含 https:// 與路徑）。
6. 等候 Pages 部署，開啟網站測試老師與學生帳號。

## 驗收清單
- [ ] 老師可登入並切換多位學生。
- [ ] 老師可建立與修改六大模組（圖表由成績自動產生）。
- [ ] 學生 A 無法讀取學生 B 的成績、作業或教材（含直接呼叫 Firestore）。
- [ ] 學生只能修改自己的作業 `done` 欄位。
- [ ] 學生無法修改自己的 `role`，也不能新增成績。
- [ ] 不同裝置重新登入後資料一致。

## 目前限制／下一版
第一版未包含學生帳號建立介面（由 Firebase Console 建立）、教材檔案上傳、忘記密碼、成績圖表科目篩選、批次匯入及即時監聽；資料變更後可切換頁籤或重新整理。學生作業狀態可以操作，但目前沒有老師批改／審核機制。正式大量使用前建議加入測試、備份、資料保留政策與管理員操作紀錄。
