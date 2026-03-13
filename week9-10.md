**Deno 2.0 版本變更、EJS 模板引擎、Session 會話機制、以及 HTML5 各項前端 API (Canvas, SVG, Media, Web API)** 的教學內容。


---

### 一、 技術重點整理

#### 1. Deno 版本重大變更 (1.x 轉 2.x)

* **JSR 平台取代 Deno X**：Deno 2.0 建議不再使用舊有的 `deno.land/x/` 網址引用模組，而是改用 **JSR** (JavaScript Registry)。舊網址在瀏覽器中可能被隱藏，但程式碼抓取暫時仍可運行。
* **建議操作**：之後引用 Oak 等模組時，語法會逐漸轉向 JSR 標準。

#### 2. 模板引擎 (Template Engine - EJS)

* **目的**：不只是用「模板字串」嵌入變數，而是讓 HTML 檔案中能撰寫 JavaScript 邏輯（如 `if` 判斷、`forEach` 迴圈）。
* **Layout 概念**：將網頁切分為 `header.ejs` (標頭)、`footer.ejs` (頁尾) 與 `main.ejs` (主體)，方便統一管理全站風格。
* **注意**：老師提到目前部分 Deno 模板模組因版本升級可能出現快取或連線問題，需注意環境配置。

#### 3. Session (會話) 機制

* **作用**：讓原本「無狀態」的 HTTP 協議能「記住」使用者。
* **登入流程**：
1. 檢查帳號密碼。
2. 若正確，在 `ctx.state.session` 中設定 User 資料。
3. 權限控管：後端判斷是否有 Session 資料，決定是否顯示「新增貼文」功能。


* **與 Cookie 的關係**：Session 通常依賴 Cookie 在瀏覽器儲存一個 ID，伺服器再根據 ID 找回對應的資料。

#### 4. 前端繪圖與多媒體

* **Canvas vs SVG**：
* **Canvas**：點陣繪圖，使用 JavaScript 程式碼一筆一畫寫出來（適合動態遊戲、特效）。
* **SVG**：向量標記，使用類似 HTML 的標籤定義形狀（適合圖示、簡單圖表，放大不失真）。


* **Video/Audio**：使用 `controls` 屬性開啟播放控制條。老師提醒在直播環境播放可能有版權封鎖風險。

#### 5. Web API 功能

* **Geolocation**：手機定位較準（GPS），電腦定位靠 IP 較不準。
* **Web Storage**：
* **LocalStorage**：永久儲存（除非清除瀏覽器），上限約 5MB。
* **SessionStorage**：分頁關閉即消失。


* **Web Workers**：讓耗時的計算（如大數據處理）在背景執行，不卡住網頁介面。

---

### 二、 網頁美化與框架：Bootstrap

* **核心價值**：響應式設計 (Responsive Web Design, RWD)，讓網頁在手機與電腦都有良好排版。
* **Grid System**：將網頁寬度分為 **12 欄**，透過 `col-sm-4` 等類別快速切換版面。
* **使用方式**：建議直接使用 **CDN** (如 JSDelivr) 引用，不需下載安裝。

---

### 三、 期末專案與 GitHub 展示建議

#### 1. 專案繳交與評分

* **截止日**：6 月 8 日 23:59 前。
* **展示方式**：強烈建議 **現場展示**（第 17、18 週），分數通常較高且能解釋實作細節。
* **誠實交代**：需在 `README.md` 標註哪些是原創、哪些參考 AI 或 YouTube 教學。

#### 2. GitHub 展示技巧

* **GitHub Pages**：雖然 GitHub Pages 只能架設**靜態**網頁（HTML/CSS/JS），不支援 Deno 後端，但你仍可用它來展示作品的前端介面。
* **多作業整合**：在 GitHub 根目錄建立一個 `index.html` 或 `README.md`，做成超連結清單指向各週作業（如 `Homework01/hello.html`）。

#### 3. 專案主題建議

* **一頁式商店**：含簡單下單功能。
* **AI 應用**：呼叫 API 製作簡易對話機器人。
* **不建議做遊戲**：除非你有很強的美術或引擎底子，否則網頁開發應專注在「資訊流與互動」而非複雜動畫。

---

