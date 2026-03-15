# Web 后端开发oak串接资料库

> 涵盖：Router 路由管理、Blog 系统建立、SQLite 资料库实作

---

## 一、Router 路由管理

### 1.1 为什么需要 Router

用 `if/else` 判断路径虽然可行，但路由一多就变得难以维护：

```javascript
// ❌ if/else 写法：混乱难维护
app.use((ctx) => {
    const path = ctx.request.url.pathname;
    const method = ctx.request.method;
    if (path === "/users" && method === "GET")  { ... }
    if (path === "/users" && method === "POST") { ... }
    if (path === "/books" && method === "GET")  { ... }
    // 越来越长...
});

// ✅ Router 写法：一目了然
router.get("/users",  (ctx) => { ... });
router.post("/users", (ctx) => { ... });
router.get("/books",  (ctx) => { ... });
```

### 1.2 基本路由写法

```typescript
import { Application, Router } from "https://deno.land/x/oak/mod.ts";

const app = new Application();
const router = new Router();

router.get("/",        (ctx) => { ctx.response.body = "首页"; });
router.get("/about",   (ctx) => { ctx.response.body = "关于"; });
router.post("/submit", (ctx) => { ctx.response.body = "收到"; });

app.use(router.routes());
await app.listen({ port: 8000 });
```

### 1.3 参数路由

在路径里用 `:参数名` 捕获动态值：

```typescript
router.get("/posts/:id", (ctx) => {
    const id = ctx.params.id;  // 从 URL 取出参数
    // 访问 /posts/42 → id = "42"
    ctx.response.body = `第 ${id} 篇文章`;
});

// 多个参数
router.get("/school/:schoolId/class/:classId", (ctx) => {
    ctx.params.schoolId  // → "3"
    ctx.params.classId   // → "7"
});
```

> **注意**：`ctx.params.id` 永远是字符串，需要和数字比对时要用 `Number()` 转换。

### 1.4 五种 HTTP 方法对应 CRUD

| 方法 | 用途 | SQL 对应 | JS 对应 |
|---|---|---|---|
| `GET` | 读取 Read | `SELECT` | `.find()` `.filter()` |
| `POST` | 新增 Create | `INSERT INTO` | `.push()` |
| `PUT` | 修改 Update | `UPDATE SET` | `array[i] = {...}` |
| `DELETE` | 删除 Delete | `DELETE WHERE` | `.splice()` |

### 1.5 ctx 物件结构

```javascript
ctx = {
    request: {
        url: {
            pathname: "/posts/1",   // 路径
            search:   "?page=2",    // 查询字串
        },
        method:  "GET",             // HTTP 方法
        headers: { ... },           // 请求标头
    },
    response: {
        body:    "",                // 回传内容
        status:  200,               // 状态码
        headers: new Headers(),     // 回传标头
    }
}
```

**关键原则**：`request` 只读（浏览器传来的），`response` 可写（我们决定回传什么）。

### 1.6 多个 Router 分档管理

```typescript
// postsRouter.ts
const postsRouter = new Router();
postsRouter.get("/posts",     (ctx) => { ... });
postsRouter.get("/posts/:id", (ctx) => { ... });
export { postsRouter };

// main.ts
app.use(postsRouter.routes());
app.use(booksRouter.routes());  // 多个都可以挂上去
```

---

## 二、Blog 系统建立

### 2.1 五条核心路由

| 路由 | 功能 | 说明 |
|---|---|---|
| `GET /` | 列表页 | 显示所有贴文 |
| `GET /new` | 表单页 | 显示空白新增表单 |
| `POST /posts` | 建立贴文 | 接收表单，存入资料库 |
| `GET /posts/:id` | 单篇页 | 显示特定一篇贴文 |
| `GET /posts/:id/edit` | 编辑页 | 显示编辑表单 |

### 2.2 layout 模板函数

避免每个页面重复写相同的 HTML 结构（DRY 原则）：

```typescript
function layout(title: string, body: string): string {
    return `
        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="UTF-8">
            <title>${title}</title>
            <style>
                body { font-family: Arial; max-width: 800px; margin: 0 auto; padding: 20px; }
                h1   { color: #333; }
                a    { color: #0066cc; text-decoration: none; }
            </style>
        </head>
        <body>
            ${body}
            <hr>
            <a href="/">← 返回首页</a>
        </body>
        </html>
    `;
}

// 使用：只传入变动的部分
ctx.response.body = layout("首页", `<h1>所有贴文</h1>`);
```

> **概念**：`layout()` 和 `constructor` 是同一个思想——固定结构 + 变动参数 → 产生结果。

### 2.3 表单与路由的对应关系

```html
<!-- GET /new 回传这张表单 -->
<form method="POST" action="/posts">
<!--         ↑ 送出方式       ↑ 送到哪个路径 -->
    <input name="title" />
    <textarea name="content"></textarea>
    <button type="submit">送出</button>
</form>
```

```typescript
// action="/posts" + method="POST"
// 对应 Router 里的：
router.post("/posts", async (ctx) => {
    const form = await ctx.request.body().form();
    const title   = form.get("title");    // 对应 name="title"
    const content = form.get("content");  // 对应 name="content"
    // ... 存入资料库
});
```

**关键**：表单的 `action` 路径必须和 Router 的路径完全对应，`method` 也必须一致。

### 2.4 PRG 模式（Post / Redirect / Get）

```
❌ 不用 Redirect：
   POST /posts → 回传 HTML
   用户重新整理 → 浏览器问「要重送表单吗？」→ 重复新增！

✅ 用 Redirect：
   POST /posts → 存好资料 → redirect → GET /
   用户重新整理 → 重新 GET / → 安全，不会重复新增
```

```typescript
router.post("/posts", async (ctx) => {
    // ... 存入资料库
    ctx.response.redirect("/");  // 存好后跳回首页
});
```

### 2.5 `<pre>` 标签保留换行

```html
<!-- 没有 pre：换行消失 -->
<p>${post.content}</p>   <!-- "第一行\n第二行" → 显示成一行 -->

<!-- 有 pre：保留格式 -->
<pre>${post.content}</pre>  <!-- 正确显示换行 -->
```

### 2.6 完整 Blog 程序

```typescript
import { Application, Router } from "https://deno.land/x/oak/mod.ts";
import { DB } from "https://deno.land/x/sqlite/mod.ts";

const app = new Application();
const router = new Router();
const db = new DB("blog.db");

// 建立资料表
db.execute(`
    CREATE TABLE IF NOT EXISTS posts (
        id         INTEGER PRIMARY KEY AUTOINCREMENT,
        title      TEXT    NOT NULL,
        content    TEXT    NOT NULL,
        created_at TEXT    DEFAULT CURRENT_TIMESTAMP
    )
`);

// GET / 列表页
router.get("/", (ctx) => {
    const posts = Array.from(db.query("SELECT id, title, created_at FROM posts"));
    const listHtml = posts.map(([id, title, createdAt]) =>
        `<li><a href="/posts/${id}">${title}</a> <small>${createdAt}</small></li>`
    ).join("");
    ctx.response.body = layout("首页", `<h1>所有贴文</h1><ul>${listHtml}</ul><a href="/new">＋ 新增</a>`);
});

// GET /new 表单页
router.get("/new", (ctx) => {
    ctx.response.body = layout("新增贴文", `
        <h1>新增贴文</h1>
        <form method="POST" action="/posts">
            标题：<input name="title" /><br><br>
            内容：<br><textarea name="content" rows="6" cols="40"></textarea><br><br>
            <button type="submit">送出</button>
        </form>
    `);
});

// POST /posts 建立贴文
router.post("/posts", async (ctx) => {
    const form = await ctx.request.body().form();
    db.query(
        "INSERT INTO posts (title, content) VALUES (?, ?)",
        [form.get("title"), form.get("content")]
    );
    ctx.response.redirect("/");
});

// GET /posts/:id 单篇页
router.get("/posts/:id", (ctx) => {
    const id   = Number(ctx.params.id);
    const rows = Array.from(db.query("SELECT * FROM posts WHERE id = ?", [id]));
    if (rows.length === 0) {
        ctx.response.status = 404;
        ctx.response.body = layout("404", "<h1>找不到这篇文章</h1>");
        return;
    }
    const [, title, content, createdAt] = rows[0];
    ctx.response.body = layout(String(title), `
        <h1>${title}</h1>
        <small>${createdAt}</small>
        <hr>
        <pre>${content}</pre>
    `);
});

app.use(router.routes());
console.log("Blog 启动在 http://localhost:8000");
await app.listen({ port: 8000 });
```

---

## 三、SQLite 资料库实作

### 3.1 SQLite 的特点

| 特性 | 说明 |
|---|---|
| 无需启动服务器 | 直接读写 `.db` 文件，不像 MySQL 要先启动服务 |
| 单一文件 | `blog.db` 就是整个资料库，方便携带备份 |
| 轻量简单 | 适合个人项目、教学、原型开发 |
| 标准 SQL | 学会后可直接转用 MySQL / PostgreSQL |

### 3.2 四条核心 SQL 指令

```sql
-- 1. 建立资料表（IF NOT EXISTS：已存在就跳过）
CREATE TABLE IF NOT EXISTS posts (
    id         INTEGER PRIMARY KEY AUTOINCREMENT,
    title      TEXT    NOT NULL,
    content    TEXT    NOT NULL,
    created_at TEXT    DEFAULT CURRENT_TIMESTAMP
);

-- 2. 新增资料（对应 POST /posts）
INSERT INTO posts (title, content)
VALUES ('标题', '内容');
-- id 和 created_at 不用填，资料库自动处理

-- 3. 读取全部（对应 GET /）
SELECT * FROM posts;

-- 4. 读取单笔（对应 GET /posts/:id）
SELECT * FROM posts WHERE id = 1;
```

### 3.3 自动化栏位

```sql
id INTEGER PRIMARY KEY AUTOINCREMENT
-- 每新增一笔自动编号：1, 2, 3...
-- 等同于 JS 的 nextId++，但资料库帮你管

created_at TEXT DEFAULT CURRENT_TIMESTAMP
-- 自动记录存入时间：'2024-03-15 14:30:00'
-- 等同于 JS 的 new Date()，但不用手动传入
```

### 3.4 在 Deno 中使用 SQLite

```typescript
import { DB } from "https://deno.land/x/sqlite/mod.ts";

const db = new DB("blog.db");  // 打开或建立 .db 文件

// 执行不需要回传值的指令（建表、INSERT、UPDATE、DELETE）
db.execute(`CREATE TABLE IF NOT EXISTS posts (...)`);
db.query("INSERT INTO posts (title, content) VALUES (?, ?)", ["标题", "内容"]);
//                                                   ↑ ? 是占位符，防止 SQL 注入

// 执行需要读取结果的指令（SELECT）
const rows = db.query("SELECT * FROM posts");
const posts = Array.from(rows);  // Iterator → 阵列
// posts = [[1, "标题", "内容", "2024-03-15"], [2, ...], ...]
```

### 3.5 占位符 `?` 的重要性

```typescript
// ❌ 危险：直接拼接字串（SQL 注入漏洞）
db.query(`SELECT * FROM posts WHERE id = ${id}`);
// 如果 id = "1; DROP TABLE posts"，整张表会被删掉！

// ✅ 安全：使用占位符
db.query("SELECT * FROM posts WHERE id = ?", [id]);
// 资料库会把参数当成纯数据处理，不会被当成 SQL 指令执行
```

### 3.6 阵列 vs 资料库对照

| 操作 | 内存阵列（教学过渡） | SQLite（实际使用） |
|---|---|---|
| 新增 | `posts.push(newPost)` | `db.query("INSERT INTO ...")` |
| 读全部 | `posts` | `db.query("SELECT * FROM posts")` |
| 读单笔 | `posts.find(p => p.id === id)` | `db.query("SELECT * WHERE id = ?", [id])` |
| 修改 | `posts[i] = {...old, ...new}` | `db.query("UPDATE posts SET ... WHERE id = ?")` |
| 删除 | `posts.splice(i, 1)` | `db.query("DELETE FROM posts WHERE id = ?")` |
| 重启后 | 数据消失 ❌ | 数据永久保存 ✅ |

---

## 四、完整知识脉络

```
浏览器按「送出表单」
    ↓
POST /posts（HTTP Method）
    ↓
router.post("/posts", ...)（Oak Router 接收）
    ↓
ctx.request.body().form()（读取表单数据）
    ↓
INSERT INTO posts (title, content) VALUES (?, ?)（SQLite 储存）
    ↓
ctx.response.redirect("/")（PRG 模式，跳回列表）
    ↓
GET /（重新读取列表）
    ↓
SELECT * FROM posts（SQLite 查询）
    ↓
layout() + 模板字串（产生 HTML）
    ↓
浏览器显示更新后的列表
```

每一层各司其职，合在一起就是一个完整的 Web 应用程序。

---

## 五、下一步可以学习

| 现在 | 下一步 |
|---|---|
| 数据存在 SQLite | 换成 PostgreSQL / MySQL（多人共用） |
| HTML 写在字串里 | 模板引擎（EJS、Handlebars） |
| 没有登录功能 | Session / JWT 用户验证 |
| 本机运行 | 部署到 Deno Deploy / VPS |
| 前后端混在一起 | 前后端分离（REST API + React） |
