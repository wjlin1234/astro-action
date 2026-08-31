# astro-funnel 專案資料夾

星盤個性輪廓漏斗——公開漏斗頁面 + 後台報告工具 + 共用的 Google Sheet 資料層。這個資料夾可以直接整包上傳到 GitHub，開啟 GitHub Pages 對外公開 `funnel.html`。

## 檔案說明

- **funnel.html** — 漏斗版網頁，放到 GitHub Pages 上，公開給任何人使用的行銷流程。
  - 步驟 1：填生日資料 → 瀏覽器當場算出完整星盤（跟 original_tool.html 同一套引擎：太陽/月亮/上升，也算宮位、相位、中天）→ 立即顯示個性輪廓（36 段正式文案，依算出的星座自動組合）
  - 步驟 2：填目標／時間投入／Email
  - 步驟 3：完成畫面
  - 兩次寫入 Google Sheet：填完步驟 1 送出一次（只寫 9 個欄位到「基本輪廓」分頁，不管有沒有走到步驟 2，純粹做流失追蹤用）；填完步驟 2 才把完整星盤資料（含經緯度/時區/分宮制/中天/文字摘要/JSON）+ 目標/時間投入/Email 一起寫進「完整名單」分頁

- **original_tool.html** — 後台報告工具。打開後預設是「名單畫面」，從漏斗完整名單抓資料列出來（姓名/Email/填表時間/目標/時間投入/處理狀態），名單超過約 10 筆時容器內部會自己出現捲軸、標題列固定在頂端。每一列有「排盤」按鈕，點下去會把這個人的出生資料帶入計算引擎，切到「報告畫面」顯示完整星盤（星體配置/宮位/相位/JSON）。處理狀態可以直接在名單畫面用下拉選單改，選項：待處理／報告製作中／已傳到七天寄信系統／已完成——改了會即時寫回 Google Sheet。報告畫面右上角有「下載星盤資訊 txt」，可以把文字摘要存成檔案下載到本機。

  這支檔案讀寫的是漏斗專用的 Google Sheet（完整名單分頁），跟你原本私下用的舊試算表無關。

- **apps_script_code.gs.txt** — 貼進 Google Sheet 的 Apps Script 程式碼（純文字檔，貼上時不用管 .txt 副檔名）。負責：接收 funnel.html 送來的兩種寫入、讓 original_tool.html 讀取名單、讓 original_tool.html 更新處理狀態。

## 建立 Google Sheet + 部署 Apps Script

**1. 建立試算表跟兩個分頁**
開一份新的 Google Sheet，重新命名（例如「阿杰博士星盤漏斗名單」）。把預設的「工作表1」改名成 `基本輪廓`，再新增一個分頁改名成 `完整名單`（分頁名稱要跟這兩個字完全一樣，程式碼是用名稱去抓分頁的）。

在 `基本輪廓` 分頁的 A1:I1 貼上表頭：
`傳送時間｜姓名｜出生日期｜出生時間｜出生城市｜國家｜太陽星座｜月亮星座｜上升星座`

在 `完整名單` 分頁的 A1:T1 貼上表頭——順序是「你打開試算表直接看的時候，最想先看到的排在最前面」，管理判讀用的欄位在前，技術用的詳細資料（經緯度、JSON 等）在最後：
`處理狀態 (Status)｜盤主姓名 (Name)｜Email｜目標 (Goal)｜時間投入 (Time Commitment)｜傳送時間 (Timestamp)｜出生日期 (Birth Date)｜出生時間 (Birth Time)｜出生城市 (City)｜國家/地區 (Country)｜太陽星座 (Sun Sign)｜月亮星座 (Moon Sign)｜上升星座 (ASC Sign)｜中天星座 (MC Sign)｜緯度 (Latitude)｜經度 (Longitude)｜時區 (Timezone)｜分宮制 (House System)｜星盤文字摘要 (Summary Text)｜JSON 原始數據 (JSON Data)`

**2. 貼上 Apps Script 程式碼**
在試算表上方選單點「擴充功能」→「Apps Script」，把預設的 `function myFunction() {}` 整段刪掉，貼上 `apps_script_code.gs.txt` 的完整內容，按左上角儲存（磁片圖示）。

**3. 部署成網頁應用程式**
右上角「部署」→「新增部署作業」→ 齒輪圖示選「網頁應用程式」。設定：
- 執行身分：**我**
- 誰可以存取：**任何人**

點「部署」。過程中會跳出 Google 要求帳號授權的畫面，選你自己的帳號；如果看到「Google 尚未驗證這個應用程式」的警告，這是正常的（自己寫的私人腳本沒送 Google 審核），點「進階」→「前往〈專案名稱〉(不安全)」→「允許」即可。

**4. 拿到網址**
部署完成後會顯示一個網址，格式類似 `https://script.google.com/macros/s/AKfycb.../exec`。這個網址已經分別填進 `funnel.html` 的 `WEBHOOK_URL` 跟 `original_tool.html` 的 `FUNNEL_API_URL`，兩支檔案接的是同一個網址。

## ⚠️ 上傳 GitHub 前，需要重新部署一次 Apps Script

途中修過一個 bug：Google Sheet 有時候會把「出生時間」欄位自動存成完整日期時間格式（用 1899-12-30 當預設日期），Apps Script 讀回來時如果沒轉換，名單畫面跟排盤結果都會出現一串奇怪的日期字串，且會讓排盤時間跑掉。`apps_script_code.gs.txt` 目前的版本已經修好這個問題，但**如果你的 Apps Script 編輯器裡貼的是舊版程式碼，要重新貼一次最新內容，並且「部署」→「管理部署作業」→ 編輯（鉛筆圖示）→ 版本選「新版本」→ 部署**，這個問題才會真的修好（單純存檔不會讓已部署的網頁應用程式套用新程式碼）。

## 部署到 GitHub Pages

1. 到 GitHub 建一個新的 repository（例如 `astro-funnel`），設定成 Public（GitHub Pages 的免費方案需要 Public repo）。
2. 把這個資料夾裡的檔案上傳上去（`funnel.html`、`original_tool.html`、`README.md`；`apps_script_code.gs.txt` 要不要一起放上去都可以，它只是給你自己參考用，不影響網站運作）。
3. 到 repository 的 Settings → Pages，Source 選 `main` branch（或你上傳的那個分支）、根目錄 `/`，儲存。
4. 等一兩分鐘，會拿到一個 `https://<你的帳號>.github.io/astro-funnel/` 這樣的網址。
5. 因為 GitHub Pages 預設開啟的是 `index.html`，如果想讓網址直接打開就是漏斗頁面，建議把 `funnel.html` 複製一份改名成 `index.html`（或用 GitHub 網頁介面直接把 `funnel.html` rename 成 `index.html`）。`original_tool.html` 維持原名即可，之後你自己用網址 `.../original_tool.html` 開啟後台。

## 技術留意事項（CORS）

`original_tool.html` 讀取名單時是瀏覽器直接發送 GET 請求給 Apps Script 網址，多數情況下 Apps Script 部署成「任何人可存取」後這種簡單 GET 讀取可以正常運作。如果部署好、填了網址之後名單畫面一直顯示「讀取名單失敗」，很可能是瀏覽器擋下跨網域讀取——這種情況下還是可以直接打開 Google Sheet 查看名單，排盤功能改成手動把出生資料貼進報告畫面裡的欄位即可，不影響核心功能，只是少了「自動列表」這個方便介面。

## 建議測試方式

先開啟 `funnel.html`，填一組測試資料走過一次流程，打開瀏覽器開發者工具的 Console，確認 `chart_only` 和 `full_lead` 兩筆資料有正確送出、欄位沒有缺漏，再打開 `original_tool.html` 確認名單畫面能不能抓到剛剛那筆測試資料、排盤跟改狀態是否正常運作。

## 目前狀態總結

- 36 段個性文案：完成
- 行銷文案（標題／導言／銜接／完成畫面）：完成
- 出生時間精準度提醒文案：已加在步驟 1
- Apps Script 網址：已接上（`WEBHOOK_URL` / `FUNNEL_API_URL`）
- 出生時間格式化 bug：已修（**需要重新部署 Apps Script 才會生效，見上方警示**）
- 後台名單超過 10 筆自動捲動＋固定表頭：完成
- GitHub Pages 部署：待你上傳
