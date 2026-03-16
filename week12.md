好的！这份笔记内容是关于 **SQLite 资料库 + Session 登入系统**，我来整理：

---

# 课堂笔记：资料库与 Session 管理

---

## 一、SQLite 基础用法

### 基本操作流程

```python
import sqlite3

conn = sqlite3.connect("example.db")  # 建立连线
cursor = conn.cursor()                # 取得游标（Cursor）

# 建立表格
cursor.execute("CREATE TABLE IF NOT EXISTS users (id, name, email)")

# 新增资料
cursor.execute("INSERT INTO users VALUES (1, 'Alice', 'alice@gmail.com')")
conn.commit()   # ⚠️ 一定要 commit，资料才会真正写入

# 查询资料
cursor.execute("SELECT * FROM users")
for row in cursor.fetchall():
    print(row)
```

> **游标（Cursor）** = 资料库中的「指标」，指向目前操作的位置

### CRUD 四大操作

| 操作 | SQL 指令 | 说明 |
|------|----------|------|
| 新增 | `INSERT` | 加入一笔资料 |
| 查询 | `SELECT` | 读取资料 |
| 修改 | `UPDATE` | 更新资料 |
| 删除 | `DELETE` | 删除资料 |

> 💡 不懂 SQL 语法？直接用自然语言告诉 AI，让它帮你翻译成 SQL 指令即可。

---

## 二、SQLAlchemy（进阶资料库套件）

SQLAlchemy 是一个**统合所有资料库**的套件，支援 SQLite、MySQL、PostgreSQL 等，语法统一，只需修改连线字串即可切换资料库。

### 与原生 SQLite 的比较

| | 原生 sqlite3 | SQLAlchemy |
|--|-------------|------------|
| 需要写 SQL | ✅ 需要 | ❌ 不需要 |
| 支援多种资料库 | ❌ 仅 SQLite | ✅ 全部支援 |
| 物件导向操作 | ❌ | ✅ |
| 对熟悉 SQL 的人 | 较直觉 | 多包一层，学习成本 |

### 基本写法

```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import Session

# 定义资料表（用 Class 表示）
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    email = Column(String)

# 新增资料
with Session(engine) as session:
    session.add(User(name="Alice", email="alice@gmail.com"))
    session.commit()

# 查询资料
users = session.query(User).all()
```

---

## 三、FastAPI + SQLite 部落格系统

将上週的部落格系统改用**真实资料库**储存贴文：

### 重要改变

- 贴文储存在 SQLite 档案中，**关闭伺服器后资料不会消失**
- 使用 SQLAlchemy 定义 `Post` 资料模型
- 使用 `SessionMaker` 建立资料库连线

### 路由逻辑

```
GET  /           → 列出所有贴文（从资料库读取）
GET  /new        → 显示新增贴文表单
POST /new        → 接收表单资料，新增贴文到资料库，导回首页
GET  /posts/{id} → 查询并显示特定贴文
```

---

## 四、Session 登入机制

### 什么是 Session？

Session 是伺服器用来「记住」使用者登入状态的机制。HTTP 本身是**无状态**的（每次请求都是独立的），Session 让伺服器能跨请求记住「你是谁」。

### 在 FastAPI 中使用 Session

```python
from starlette.middleware.sessions import SessionMiddleware

app.add_middleware(
    SessionMiddleware,
    secret_key="你的超级乱的密钥字串"  # ⚠️ 绝对不能外泄
)
```

> ⚠️ `secret_key` 必须是一串复杂的随机字串，这是加密用的密钥，外泄会导致网站被入侵。

### 登入 / 登出流程

```python
# 登入：验证成功后，将使用者资讯存入 Session
request.session["user"] = username

# 检查是否已登入
if "user" not in request.session:
    # 导回登入页面

# 登出：清除 Session
request.session.clear()
```

### 完整登入系统流程

```
注册（Sign Up）
  → 输入帐号密码
  → 检查帐号是否已存在
  → 将帐号 + 密码（建议做雜湊）存入资料库

登入（Login）
  → 输入帐号密码
  → 查询资料库是否有此帐号
  → 比对密码（或雜湊值）
  → 成功 → 写入 Session → 允许操作
  → 失败 → 回传错误

登出（Logout）
  → 清除 Session

受保护功能（如发文）
  → 先检查 Session 中是否有登入状态
  → 没有 → 拒绝操作
```

### 密码安全提醒

良心网站不储存明文密码，储存**雜湊值**：

```python
import hashlib
hashed = hashlib.sha256(password.encode()).hexdigest()
```

> 本次课堂范例为简化直接储存明文密码，实际开发应做雜湊处理。

---

## 五、让别人连上你的伺服器

### 开发模式 vs 执行模式

| 模式 | 指令 | 可否外部连线 |
|------|------|-------------|
| 开发模式 | `fastapi dev` | ❌ 仅本机 |
| 执行模式 | `fastapi run` | ✅ 可外部连线 |

### 内网连线（同校/同网路）

1. 执行 `fastapi run`（非 dev 模式）
2. 查询本机 IP：`ifconfig` 或 `ip addr`
3. 其他电脑输入 `http://192.168.x.x:8000` 即可连线

> `192.168` 开头的是**内网 IP**，仅限同一网路内使用。

### 外网连线方案

| 方案 | 优点 | 缺点 |
|------|------|------|
| **ngrok** | 免费、快速测试 | 流量小、不能固定网址、只能测试用 |
| **申请公开 IP**（如中华电信） | 稳定 | 需额外付费申请 |
| **云端主机**（如 Vercel、Railway） | 免费方案多、稳定 | 需学习部署流程 |

### ngrok 使用方式

```bash
# 安装（Mac）
brew install ngrok

# 注册帐号后取得 authtoken，设定一次
ngrok config add-authtoken 你的token

# 将本机 8000 port 暴露到网路
ngrok http 8000
```

执行后会得到一个临时公开网址（如 `https://xxxx.ngrok.io`），任何人都可访问，但关闭后失效。

---

## 六、本週作业

在 **Blog + SignUp** 系统中加入「**删除贴文**」功能：

**基本要求：**
- 贴文旁加上删除按钮（如叉叉）
- 点击后确认，确认后删除该贴文

**进阶要求（加分）：**
- 只有**贴文的作者**才能删除自己的贴文
- 其他使用者无法删除他人贴文（权限检查）

**使用 AI 的注意事项：**
- 可以使用 ChatGPT / Claude 辅助
- 但必须告诉老师：用了哪个 AI、哪些部分是 AI 写的、哪些是自己修改的
- 建议使用支援多档案的编辑器（如 Cursor）让 AI 看到完整专案，避免 AI 因资讯不足而乱写

---

## 七、老师补充观念

**Pydantic 输入验证：**
FastAPI 内建支援 Pydantic，可以在定义资料模型时自动验证输入格式，输入错误时会自动抛出错误，不需要自己写检查逻辑。

**前后端语言分工再次强调：**

| 层次 | 语言 | 位置 |
|------|------|------|
| 前端 | HTML / CSS / JavaScript | 浏览器 |
| 后端 | Python（FastAPI） | 伺服器 |

前后端沟通方式：超连结（GET）、表单（POST），未来会学到 **WebSocket**（双向即时连线）与 **Fetch API**（下週预告）。

---
