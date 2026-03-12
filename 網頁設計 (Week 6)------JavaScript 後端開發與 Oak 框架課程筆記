# JavaScript 後端開發與 Oak 框架課程筆記

## 一、課程概述

本課程開始進入後端開發領域,採用 Deno 執行環境和 Oak 框架。課程特色是前後端交錯教學,與過往一學期專注前端或後端的方式不同。這種教學方式有助於學生更好地理解全端開發的完整流程。

---

## 二、Deno 環境與非同步操作

### 2.1 同步與非同步

Deno 提供兩種版本的檔案操作方法:

#### 同步版本 (Sync)

```javascript
// 同步讀取檔案
let content = Deno.readTextFileSync("file.txt");
```

#### 非同步版本 (Async)

```javascript
// 非同步讀取檔案 - 需要使用 await
let content = await Deno.readTextFile("file.txt");
```

> **重要概念**: 
> - 非同步版本是 Deno 的主流和推薦做法
> - 非同步操作速度更快,不需要等待
> - 涉及硬碟 I/O 等耗時操作時必須使用 `await`
> - 背後機制涉及 Generator,非常複雜,暫不深入討論

### 2.2 執行 Deno 程式的注意事項

```bash
# 必須加上 -A 參數,開啟所有權限
deno run -A your_script.ts

# -A 包含:
# --allow-net (網路存取)
# --allow-read (檔案讀取)
# --allow-write (檔案寫入)
# --allow-env (環境變數存取)
```

---

## 三、Oak 框架基礎

### 3.1 什麼是 Oak

Oak 是 Deno 的 Web 框架,類似於 Node.js 的 Express。它提供了簡潔的 API 來建立 Web 伺服器。

### 3.2 第一個 Oak 程式 (01-basic)

#### 完整程式碼

```javascript
// 從遠端 URL 引入 Oak 模組
import { Application } from "https://deno.land/x/oak/mod.ts";

// 創建應用實例
const app = new Application();

// 定義中介軟體(Middleware)
app.use((ctx) => {
    ctx.response.body = "Hello 你好";
});

// 啟動伺服器
console.log('Start at http://127.0.0.1:8000');
await app.listen({ port: 8000 });
```

#### 程式碼解析

**1. 引入模組**
```javascript
import { Application } from "https://deno.land/x/oak/mod.ts";
```
- 直接從 URL 引入模組(Deno 特色)
- `mod.ts` 是 TypeScript 檔案
- TypeScript = JavaScript + 型態系統

**2. 創建應用實例**
```javascript
const app = new Application();
```
- 使用 `new` 關鍵字創建 Application 類別的實例

**3. 定義回應邏輯**
```javascript
app.use((ctx) => {
    ctx.response.body = "Hello 你好";
});
```
- `ctx` = Context(上下文物件)
- `ctx.response.body` 設定回應內容

**4. 啟動伺服器**
```javascript
await app.listen({ port: 8000 });
```
- 監聽 8000 埠號
- 必須使用 `await` 因為這是非同步操作

### 3.3 執行與測試

#### 執行命令

```bash
deno run -A 01-basic.js
```

#### 測試方式

**方法一**: 瀏覽器輸入
```
http://127.0.0.1:8000
```

**方法二**: 使用 localhost(等同於 127.0.0.1)
```
http://localhost:8000
```

**Ctrl+點擊**: 在 VS Code 或終端機中,按 Ctrl(Mac 為 Cmd)加點擊 URL 可直接開啟瀏覽器

---

## 四、網路基礎概念

### 4.1 IP 位址與 localhost

#### 127.0.0.1 的意義

- **特殊 IP 位址**: 代表本機(自己的電腦)
- **TCP/IP 標準**: 固定用來代表本地主機
- **別名**: localhost

```bash
# ping 測試
ping google.com
# 顯示 Google 的 IP,例如: 142.250.185.46

ping 127.0.0.1
# 顯示本機回應
```

### 4.2 埠號(Port)

#### 基本概念

- **範圍**: 0 ~ 65535
- **已預約埠號**(0-1023):
  - 80: HTTP(網頁伺服器)
  - 443: HTTPS(加密網頁)
  - 25: SMTP(郵件傳送)
  - 21: FTP(檔案傳輸)

#### 開發常用埠號

- **1024 以上**: 開發時建議使用
- **常見選擇**: 
  - 3000(Node.js/Express 預設)
  - 8000(本課程使用)
  - 8001, 8002...(避免衝突)

#### 埠號衝突

```bash
# 如果 8000 已被佔用
$ deno run -A program.js
# 錯誤: Address already in use

# 解決方法:
# 1. 關閉佔用該埠號的程式(Ctrl+C)
# 2. 使用其他埠號
```

### 4.3 URL 結構

```
http://127.0.0.1:8000/path/to/resource?param=value#section
└─┬─┘ └────┬────┘└┬──┘└──────┬──────┘└─────┬─────┘└───┬──┘
協定    主機位址   埠號    路徑          查詢參數      錨點

```

**各部分說明**:
- **協定**: http, https, ftp 等
- **主機**: IP 或域名
- **埠號**: 預設 80(HTTP)或 443(HTTPS)可省略
- **路徑**: 資源位置
- **查詢參數**: ?key=value&key2=value2
- **錨點**: 頁面內定位(#後的內容不會傳送到伺服器)

---

## 五、進階 Oak 應用

### 5.1 回應 HTML 內容 (oak-html)

```javascript
import { Application } from "https://deno.land/x/oak/mod.ts";

const app = new Application();

app.use((ctx) => {
    ctx.response.body = `
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>我的網站</title>
    </head>
    <body>
        <h1>歡迎來到我的網站</h1>
        <p>這是一個完整的 HTML 頁面</p>
        <a href="https://github.com">GitHub</a>
    </body>
    </html>
    `;
});

await app.listen({ port: 8000 });
```

#### 樣板字串(Template Literals)

```javascript
// 一般字串
let str1 = "單行字串";
let str2 = '單行字串';

// 樣板字串(多行 + 變數插值)
let name = "張三";
let html = `
    <h1>你好,${name}</h1>
    <p>這是多行字串</p>
`;
```

**樣板字串特點**:
- 使用反引號(`` ` ``)包圍
- 支援多行
- 支援變數插值(`${variable}`)
- 鍵盤位置:數字 1 鍵左邊

### 5.2 路徑處理 (oak-info)

```javascript
import { Application } from "https://deno.land/x/oak/mod.ts";

const app = new Application();

app.use((ctx) => {
    const pathname = ctx.request.url.pathname;
  
    ctx.response.body = `
        <h1>請求資訊</h1>
        <p>路徑: ${pathname}</p>
        <p>完整 URL: ${ctx.request.url}</p>
        <p>方法: ${ctx.request.method}</p>
    `;
});

await app.listen({ port: 8000 });
```

#### 可存取的請求資訊

```javascript
// Context 物件結構
ctx = {
    request: {
        url: {
            pathname: "/path/to/resource",  // 路徑
            search: "?key=value",            // 查詢字串
            hash: "#section"                 // 錨點
        },
        method: "GET",                       // HTTP 方法
        headers: {                           // 請求標頭
            "user-agent": "...",
            "accept": "text/html",
            // ...更多標頭
        }
    },
    response: {
        body: "",                            // 回應內容
        headers: new Headers(),              // 回應標頭
        status: 200                          // 狀態碼
    }
}
```

### 5.3 條件路由 (oak-me)

```javascript
import { Application } from "https://deno.land/x/oak/mod.ts";

const app = new Application();

app.use((ctx) => {
    const pathname = ctx.request.url.pathname;
  
    if (pathname === "/") {
        // 根路徑:顯示所有資訊
        ctx.response.body = `
            <h1>自我介紹</h1>
            <ul>
                <li><a href="/name">姓名</a></li>
                <li><a href="/age">年齡</a></li>
                <li><a href="/gender">性別</a></li>
            </ul>
        `;
    } 
    else if (pathname === "/name") {
        ctx.response.body = "姓名: 陳鍾誠";
    } 
    else if (pathname === "/age") {
        ctx.response.body = "年齡: 55";
    } 
    else if (pathname === "/gender") {
        ctx.response.body = "性別: 男";
    } 
    else {
        ctx.response.body = "找不到頁面";
    }
});

await app.listen({ port: 8000 });
```

---

## 六、今日作業:自我介紹網站

### 6.1 作業要求

**基本要求**:
1. 建立一個自我介紹網站
2. 至少包含三個欄位:姓名、年齡、性別
3. 根路徑(`/`)顯示所有連結
4. 各子路徑顯示對應資訊

**進階挑戰**(選做):
1. 使用 CSS 美化頁面
2. 添加更多個人資訊(興趣、專長、聯絡方式等)
3. 實作共用模板(見下節)

### 6.2 範例程式碼

```javascript
import { Application } from "https://deno.land/x/oak/mod.ts";

const app = new Application();

app.use((ctx) => {
    const pathname = ctx.request.url.pathname;
  
    // 根路徑
    if (pathname === "/") {
        ctx.response.body = `
            <!DOCTYPE html>
            <html>
            <head>
                <meta charset="UTF-8">
                <title>自我介紹</title>
                <style>
                    body { font-family: Arial, sans-serif; }
                    ul { list-style-type: none; }
                    a { text-decoration: none; color: blue; }
                </style>
            </head>
            <body>
                <h1>歡迎來到我的自我介紹網站</h1>
                <ul>
                    <li><a href="/name">📛 姓名</a></li>
                    <li><a href="/age">🎂 年齡</a></li>
                    <li><a href="/gender">⚧ 性別</a></li>
                </ul>
            </body>
            </html>
        `;
    }
    // 姓名
    else if (pathname === "/name") {
        ctx.response.body = `
            <!DOCTYPE html>
            <html>
            <head><meta charset="UTF-8"><title>姓名</title></head>
            <body>
                <h1>姓名</h1>
                <p>我的姓名是:王小明</p>
                <a href="/">返回首頁</a>
            </body>
            </html>
        `;
    }
    // 年齡
    else if (pathname === "/age") {
        ctx.response.body = `
            <!DOCTYPE html>
            <html>
            <head><meta charset="UTF-8"><title>年齡</title></head>
            <body>
                <h1>年齡</h1>
                <p>我的年齡是:20 歲</p>
                <a href="/">返回首頁</a>
            </body>
            </html>
        `;
    }
    // 性別
    else if (pathname === "/gender") {
        ctx.response.body = `
            <!DOCTYPE html>
            <html>
            <head><meta charset="UTF-8"><title>性別</title></head>
            <body>
                <h1>性別</h1>
                <p>我的性別是:男</p>
                <a href="/">返回首頁</a>
            </body>
            </html>
        `;
    }
    // 404 找不到頁面
    else {
        ctx.response.status = 404;
        ctx.response.body = `
            <!DOCTYPE html>
            <html>
            <head><meta charset="UTF-8"><title>404</title></head>
            <body>
                <h1>404 - 找不到頁面</h1>
                <a href="/">返回首頁</a>
            </body>
            </html>
        `;
    }
});

console.log("伺服器啟動於 http://127.0.0.1:8000");
await app.listen({ port: 8000 });
```

### 6.3 執行與測試

```bash
# 1. 確保已更新專案
git pull

# 2. 執行程式
deno run -A your_homework.js

# 3. 開啟瀏覽器測試
# http://127.0.0.1:8000
# http://127.0.0.1:8000/name
# http://127.0.0.1:8000/age
# http://127.0.0.1:8000/gender
```

---

## 七、共用模板技巧

### 7.1 問題:重複的 HTML 結構

每個頁面都有相同的標頭和樣式,造成程式碼冗長:

```javascript
// 每個頁面都重複這些程式碼
ctx.response.body = `
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>...</title>
        <style>...</style>
    </head>
    <body>
        <!-- 只有這裡不同 -->
    </body>
    </html>
`;
```

### 7.2 解決方案:提取共用模板函數

```javascript
import { Application } from "https://deno.land/x/oak/mod.ts";

const app = new Application();

// 共用模板函數
function layout(title, body) {
    return `
        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="UTF-8">
            <title>${title}</title>
            <style>
                body {
                    font-family: Arial, sans-serif;
                    max-width: 800px;
                    margin: 0 auto;
                    padding: 20px;
                    background-color: #f0f0f0;
                }
                h1 { color: #333; }
                a {
                    color: #0066cc;
                    text-decoration: none;
                }
                a:hover { text-decoration: underline; }
            </style>
        </head>
        <body>
            ${body}
            <hr>
            <p><a href="/">返回首頁</a></p>
        </body>
        </html>
    `;
}

app.use((ctx) => {
    const pathname = ctx.request.url.pathname;
  
    if (pathname === "/") {
        ctx.response.body = layout("首頁", `
            <h1>自我介紹</h1>
            <ul>
                <li><a href="/name">姓名</a></li>
                <li><a href="/age">年齡</a></li>
                <li><a href="/gender">性別</a></li>
            </ul>
        `);
    }
    else if (pathname === "/name") {
        ctx.response.body = layout("姓名", `
            <h1>姓名</h1>
            <p>王小明</p>
        `);
    }
    else if (pathname === "/age") {
        ctx.response.body = layout("年齡", `
            <h1>年齡</h1>
            <p>20 歲</p>
        `);
    }
    else if (pathname === "/gender") {
        ctx.response.body = layout("性別", `
            <h1>性別</h1>
            <p>男</p>
        `);
    }
    else {
        ctx.response.body = layout("404", `
            <h1>找不到頁面</h1>
        `);
    }
});

await app.listen({ port: 8000 });
```

### 7.3 共用模板的優點

1. **減少重複程式碼**: 只需維護一份模板
2. **易於修改**: 修改樣式只需改一個地方
3. **一致性**: 所有頁面自動套用相同樣式
4. **可擴展**: 容易添加新功能(如導航列、頁尾)

---

## 八、靜態檔案服務 (Static File Serving)

### 8.1 靜態網站範例 (static-1)

```javascript
import { Application } from "https://deno.land/x/oak/mod.ts";
import { send } from "https://deno.land/x/oak/send.ts";

const app = new Application();

// 提供 public 資料夾中的靜態檔案
app.use(async (ctx) => {
    await send(ctx, ctx.request.url.pathname, {
        root: `${Deno.cwd()}/public`,
        index: "index.html",  // 預設檔案
    });
});

await app.listen({ port: 8000 });
```

### 8.2 資料夾結構

```
project/
├── server.js
└── public/
    ├── index.html
    ├── about.html
    ├── style.css
    └── images/
        └── photo.jpg
```

### 8.3 index.html 預設行為

當訪問資料夾路徑時(不指定檔案名),自動載入 `index.html`:

```
http://localhost:8000/         → public/index.html
http://localhost:8000/images/  → public/images/index.html(如果存在)
```

### 8.4 完整路徑存取

```
http://localhost:8000/about.html        → public/about.html
http://localhost:8000/style.css         → public/style.css
http://localhost:8000/images/photo.jpg  → public/images/photo.jpg
```

---

## 九、雲端部署

### 9.1 SSH 連線

```bash
# 連線到雲端伺服器
ssh username@your-server-ip

# 輸入密碼
# 課程密碼: CSIE3
```

### 9.2 埠號規定

**重要**: 每位學生只能使用特定埠號避免衝突

```
埠號格式: 8XXX
XXX = 學號末三碼

例如:
學號 1234567 → 使用 8567
學號 1234001 → 使用 8001
```

### 9.3 部署步驟

```bash
# 1. 連線到伺服器
ssh student@server-ip

# 2. 建立專案資料夾(使用學號末三碼)
mkdir 8567
cd 8567

# 3. 複製專案檔案(從本機上傳)
# 或使用 git clone

# 4. 執行伺服器(記得修改埠號!)
deno run -A server.js
```

### 9.4 HTTPS 與網域名稱(選修)

#### SSL 憑證

- **免費憑證**: Let's Encrypt
- **有效期**: 3 個月
- **自動更新**: 需設定自動更新機制

```bash
# 使用 Certbot 申請憑證
sudo certbot certonly --standalone -d yourdomain.com
```

#### 網域名稱

**付費服務**:
- GoDaddy, Namecheap 等
- 費用約 300-600 元/年

**免費服務**(有限制):
- Freenom
- 通常是小國家的頂級域名(.tk, .ml 等)
- 第一年免費,之後可能收費

---

## 十、HTTP 標頭與請求資訊

### 10.1 查看請求標頭 (oak-info)

```javascript
app.use((ctx) => {
    // 取得所有標頭
    const headers = ctx.request.headers;
  
    let headerInfo = "<h2>請求標頭</h2><ul>";
    for (const [key, value] of headers.entries()) {
        headerInfo += `<li><strong>${key}</strong>: ${value}</li>`;
    }
    headerInfo += "</ul>";
  
    ctx.response.body = `
        <h1>請求資訊</h1>
        ${headerInfo}
    `;
});
```

### 10.2 常見請求標頭

| 標頭名稱 | 說明 | 範例值 |
|---------|------|--------|
| `Accept` | 可接受的內容類型 | `text/html,application/xhtml+xml` |
| `Accept-Encoding` | 支援的壓縮格式 | `gzip, deflate, br` |
| `Accept-Language` | 偏好的語言 | `zh-TW,zh;q=0.9,en;q=0.8` |
| `User-Agent` | 瀏覽器識別 | `Mozilla/5.0 (Windows NT 10.0...)` |
| `Cookie` | 儲存的 Cookie | `session=abc123; user=john` |
| `Referer` | 來源頁面 | `https://google.com` |

### 10.3 請求方法(HTTP Methods)

```javascript
app.use((ctx) => {
    const method = ctx.request.method;
  
    switch(method) {
        case "GET":
            ctx.response.body = "這是 GET 請求";
            break;
        case "POST":
            ctx.response.body = "這是 POST 請求";
            break;
        case "PUT":
            ctx.response.body = "這是 PUT 請求";
            break;
        case "DELETE":
            ctx.response.body = "這是 DELETE 請求";
            break;
        default:
            ctx.response.body = `未知的方法: ${method}`;
    }
});
```

---

## 十一、終端機基本操作

### 11.1 關閉執行中的程式

**方法一**: 鍵盤快捷鍵
```bash
Ctrl + C  # 中斷程式執行
```

**方法二**: 關閉終端視窗
- 點擊垃圾桶圖示
- 或直接關閉終端視窗

### 11.2 檢查埠號佔用

```bash
# Windows
netstat -ano | findstr :8000

# Mac/Linux
lsof -i :8000

# 釋放埠號:關閉佔用該埠號的程式
```

---

## 十二、除錯技巧

### 12.1 常見錯誤

#### 錯誤 1: Address already in use

```
錯誤訊息: error: Uncaught (in promise) AddrInUse: Address already in use (os error 48)
```

**原因**: 埠號已被其他程式佔用

**解決方法**:
1. 關閉佔用該埠號的程式(Ctrl+C)
2. 更換埠號

#### 錯誤 2: Permission denied

```
錯誤訊息: error: Uncaught PermissionDenied: Requires net access
```

**原因**: 未加上 `-A` 參數

**解決方法**:
```bash
deno run -A your_script.js
```

#### 錯誤 3: Module not found

```
錯誤訊息: error: Module not found "file:///path/to/module"
```

**原因**: 檔案路徑錯誤或檔案不存在

**解決方法**:
1. 檢查檔案路徑是否正確
2. 確認檔案是否存在
3. 檢查大小寫(Linux/Mac 區分大小寫)

### 12.2 伺服器端除錯

```javascript
app.use((ctx) => {
    // 在終端機印出請求資訊
    console.log(`${ctx.request.method} ${ctx.request.url.pathname}`);
  
    // 印出更詳細的資訊
    console.log("Headers:", ctx.request.headers);
  
    // 你的處理邏輯
    ctx.response.body = "OK";
});
```

---

## 十三、作業提交規範

### 13.1 檔案組織

```
homework-5/
├── README.md      # 說明文件
├── server.js      # 主程式
└── public/        # 靜態檔案(如果有)
    └── ...
```

### 13.2 README.md 範本

```markdown
# 作業五:自我介紹網站

學號: 12345678
姓名: 王小明

## 功能說明

1. 根路徑 (`/`): 顯示所有資訊連結
2. `/name`: 顯示姓名
3. `/age`: 顯示年齡
4. `/gender`: 顯示性別

## 執行方式

```bash
deno run -A server.js
```

然後開啟瀏覽器訪問 http://localhost:8000

## 額外功能(如果有)

- 使用 CSS 美化
- 共用模板
- 其他...

## AI 使用說明(如果有使用)

使用 ChatGPT 協助:
- 美化 CSS 樣式
- 除錯

[ChatGPT 對話連結](https://chat.openai.com/share/xxx)
```

### 13.3 提交方式

```bash
# 1. 確保在專案目錄
cd your-project-folder

# 2. 加入所有檔案
git add .

# 3. 提交變更
git commit -m "完成作業五:自我介紹網站"

# 4. 推送到遠端
git push origin main
```

---

## 十四、學習資源

### 14.1 Deno 官方文件

- [Deno 官網](https://deno.land/)
- [Deno 手冊](https://deno.land/manual)
- [Deno 標準函式庫](https://deno.land/std)

### 14.2 Oak 框架

- [Oak GitHub](https://github.com/oakserver/oak)
- [Oak 文件](https://oakserver.github.io/oak/)

### 14.3 其他資源

- [MDN Web Docs](https://developer.mozilla.org/): Web 開發百科
- [JavaScript.info](https://javascript.info/): JavaScript 教學
- [HTTP 狀態碼](https://httpstatuses.com/): HTTP 狀態碼查詢

---

## 十五、下週預告

### 15.1 課程內容

- 表單處理(POST 請求)
- 資料驗證
- Session 與 Cookie
- 簡易部落格系統

### 15.2 學習建議

1. **完成本週作業**: 紮實練習是進步的關鍵
2. **實驗不同設計**: 嘗試不同的 CSS 樣式和佈局
3. **閱讀範例程式碼**: 理解每一行的作用
4. **提問與討論**: 遇到問題及時提出

---

## 十六、總結

本堂課介紹了:

1. **Deno 基礎**: 執行環境、非同步操作、權限管理
2. **Oak 框架**: 建立 Web 伺服器、處理請求與回應
3. **HTTP 基礎**: URL 結構、埠號、請求標頭
4. **路由處理**: 根據路徑回應不同內容
5. **模板技巧**: 使用函數提取共用 HTML 結構
6. **靜態檔案**: 提供 HTML、CSS、圖片等靜態資源
7. **雲端部署**: SSH 連線、埠號管理、上傳專案

透過實作自我介紹網站,將理論應用到實際專案中。記住,後端開發的核心是**處理請求、產生回應**,其他都是建立在這個基礎之上的擴展。

---

**祝學習順利!有問題隨時提問。**
