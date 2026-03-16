我来帮你整理这份课堂记录！内容是关于 **FastAPI 网页开发**的课程，我来梳理成结构清晰的笔记。

---

# 课堂笔记：FastAPI 网页开发

**课程进度：** 第11週，距结束还剩3週（第15週开始交作业）

---

## 一、课程背景与框架选择

本次课程从 Dino/Oak 框架迁移到 **FastAPI**，主要原因：

- Oak 框架近乎停止维护，社群规模小
- FastAPI 是目前较热门的 Python 网页框架（2018年起），预计短期内不会停止维护
- **AI 辅助优势**：FastAPI 基于 Python，AI（如 ChatGPT、Claude）对 Python 的训练资料最多，写 FastAPI 代码的能力远优于写 Dino/Oak

> 推荐 AI 工具：ChatGPT、**Claude**（老师特别推荐）、Grok、Google AI Studio（Gemini）、DeepSeek（免费但服务器常拥塞）

---

## 二、静态网页服务（Static Files）

- 将静态文件放入 `static/` 资料夹
- 呼叫 `StaticFiles`，指定资料夹路径与 HTML 支援
- 执行方式：进入含 `main.py` 的资料夹，开终端机执行：

```bash
fastapi dev
```

> `dev` 是开发模式，程序修改后会**自动重启**，方便调试。

- 访问根目录 `/` 时，自动导向 `static/index.html`

---

## 三、书籍管理 API（Books API）

- 与之前 Dino 版本逻辑相同，改用 FastAPI 语法
- 路由设计：
  - `GET /` → 回应 Hello 讯息
  - `GET /books` → 列出所有书籍
  - `GET /books/{id}` → 取得特定书籍

---

## 四、表单与登入系统

### 4.1 HTTP 方法基础

| 方法 | 用途 |
|------|------|
| `GET` | 向伺服器请求、显示资料 |
| `POST` | 向伺服器送出（提交）资料 |

### 4.2 表单运作逻辑（重点）

HTML 表单中的关键属性：

```html
<form action="/submit" method="post">
  <input type="text" name="username">
  <input type="password" name="password">
  <button type="submit">登入</button>
</form>
```

- **`action`**：资料送往的路径（必须与后端路由一致）
- **`method`**：使用 GET 或 POST
- **`name`**：欄位名称（后端用此名称接收资料，比 `id` 更关键）

> ⚠️ `action` 路径与后端路由**必须完全对应**，否则后端收不到资料。`id` 属性是给前端 JavaScript 用的，与后端无关。

### 4.3 用 curl 观察 HTTP 请求

```bash
# 基本 GET
curl http://127.0.0.1:8000

# 详细模式（显示完整过程）
curl -v http://127.0.0.1:8000

# POST 表单资料
curl -X POST http://127.0.0.1:8000/submit \
  -d "username=ccc&password=123"
```

---

## 五、注册与登入完整系统

### 系统流程

```
GET /          → 显示 index.html（含登入/注册入口）
GET /login     → 显示登入页面
GET /signup    → 显示注册页面
POST /login    → 接收帐号密码，验证登入
POST /signup   → 接收注册资料，存入记忆体资料库
```

### 密码安全（重要观念）

良心网站**不储存明文密码**，而是储存**密码的雜湊值（Hash）**：

1. 注册时：将密码雜湊后储存
2. 登入时：将输入密码再次雜湊，与储存值比对
3. 使用 SHA-256 等演算法，碰撞机率极低（比宇宙毁灭还低），无需担心

---

## 六、Jinja2 模板引擎

FastAPI 使用 **Jinja2**（非"忍者"，正确读音为 **金甲**）作为模板引擎。

### 为何使用模板？

- 可在 HTML 中嵌入程式逻辑（如循环、变量）
- 支援**模板继承**（Base Template），共用 CSS/版型，避免重复撰写

### 模板继承示例

`base.html`（基础模板）：
```html
<html>
  <head><!-- 共用 CSS --></head>
  <body>
    {% block content %}{% endblock %}
  </body>
</html>
```

`list.html`（继承基础模板）：
```html
{% extends "base.html" %}
{% block content %}
  {% for post in posts %}
    <a href="/posts/{{ post.id }}">{{ post.title }}</a>
  {% endfor %}
{% endblock %}
```

> 好处：CSS 只需写一次，所有页面共用。

---

## 七、部落格系统（Blog）

功能路由：

| 路由 | 方法 | 说明 |
|------|------|------|
| `/` | GET | 列出所有贴文 |
| `/new` | GET | 显示新增贴文表单 |
| `/new` | POST | 接收并储存新贴文 |
| `/posts/{id}` | GET | 显示特定贴文 |

流程：POST 新贴文 → 加入贴文清单 → 导回根目录 → 重新列出所有贴文

---

## 八、非同步（async/await）概念

FastAPI 中函式常加 `async`：

```python
async def handle_login(...):
    ...
```

- **目的**：遇到 I/O 操作（如读档、网路请求）时，不卡住其他请求
- 原理：遇到 I/O 时，暂时让出控制权给其他请求；I/O 完成后，FastAPI 排程系统再将其唤回执行
- 这是一种**协程（Coroutine）机制**，类似作业系统的多工，但在 Python 层次实现，而非 OS 层次

---

## 九、前后端区分

| 层次 | 语言 | 说明 |
|------|------|------|
| 前端 | HTML / CSS / JavaScript | 在浏览器执行 |
| 后端 | Python（FastAPI） | 在伺服器执行 |

- 前后端沟通方式：**超连结（GET）** 或 **表单（POST）**
- 未来进阶：WebSocket（连线导向，双向即时通讯）

> ✅ 用 FastAPI 的好处：Python = 后端，JavaScript = 前端，**不会混淆**。

---

## 十、本週作业

执行并读懂以下三个范例程序（难度递增）：

1. **Login**（登入表单）
2. **Sign Up**（注册系统）
3. **Blog**（部落格系统）

重点：**不是只执行就好**，要能将「操作动作」与「程式码逻辑」对应起来，理解每个 Form Action 触发了哪个路由、传了哪些参数。

---

## 十一、老师的学习建议

- **善用 AI**：让 AI 成为你的「阶梯」而非「墙壁」
  - ✅ 能看懂、能修改 AI 写的代码
  - ❌ 完全依赖 AI、自己写不出来
- 基本语法（循环、条件判断）必须自己会
- 函式库的用法可以交给 AI 记忆
- 课程习题一定要**自己动手做**，不做就学不会

---
