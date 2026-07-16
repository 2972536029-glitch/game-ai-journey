# Game AI Journey - 学习日志

## 基本信息
- **开始日期**：2026-07-13
- **背景**：Xsolla 培训，方向游戏+AI工程（AIGC管线、NPC、反欺诈）
- **编程基础**：Python 浅尝辄止

---

## Session 1 (2026-07-13)
- **已完成**：建立项目仓库 `game-ai-journey`，创建 `learning-progress.md`
- **知识点**：学习日志机制，用于跨对话无缝续接进度
- **当前阶段**：第一阶段（搭环境）

---

## Session 2 (2026-07-14) — Xsolla 第一节课

### 环境搭建（课前准备）
- [x] GitLab 登录 & 设置多因素验证（MFA）
- [x] 安装 Git、Node.js 并验证安装成功
- [x] 下载 VS Code 编辑器，配置 Terminal / zsh shell
- [x] 克隆课程仓库 `apis-and-orders`（`school-gitlab.xsolla.dev`）
- [x] 运行代码，了解项目结构（`orders-api` / `checkout-api` / `slides` / `handouts`）
- [x] Miniconda 26.5.3 + conda 虚拟环境 `game-ai-journey`（Python 3.13）

### 课程内容：APIs & Orders — Journey of a Request

**Module 1 — 请求的生命周期**
- API 定义：Application Programming Interface，是程序间的契约
- HTTP 请求四要素：**Method**（动词）、**Path**（路径）、**Headers**（元数据）、**Body**（数据体）
- HTTP 响应：Status Code + Headers + Body
- 状态码家族：**2xx** 成功 · **3xx** 重定向 · **4xx** 客户端错误 · **5xx** 服务端错误
- JSON 基础：`string` / `number` / `boolean` / `array` / `null` / `object`

**Module 2 — 第一个 Server**
- 服务器本质：**一个监听端口等待连接的程序**
- 原始 Node.js HTTP 服务器（`http.createServer`）
- `localhost:3000` = 本机（127.0.0.1）+ 端口号
- 网络栈四层：应用层 HTTP → 传输层 TCP → 网络层 IP → 物理层

**Module 3 — REST & Orders API**
- REST 原则：资源 = 名词（`/orders`），动词 = HTTP Method
- URL 设计：动词不要放路径里（✅ `DELETE /orders/7` 而非 ❌ `GET /orders/delete?id=7`）
- 用 curl 发送请求测试 API
- 构建了完整的 Orders API（POST/GET）和支付端点

**Module 4 — 错误处理**
- 宁可大声报错也不要静默存脏数据（400 vs 201 for garbage）
- 统一的错误响应信封格式：`{ error: { code, message, details? } }`
- 4xx 决策三步走：①能解析吗？②资源存在吗？③状态允许吗？
- 并发竞态：double-pay race condition（read → await → write 之间状态可能已变）
- 安全：不要在错误信息里泄露堆栈跟踪、数据库 schema、用户枚举

**Module 5 — Node 中的 JSON**
- 三大陷阱：①多余字段不报错（mass assignment）②缺失字段 → `undefined` → `NaN` ③类型自动转换
- 永远在边界做 validate：parse 成功 ≠ 数据正确

**Module 6 — Checkout API 构建**
- 项目的依赖注入模式：`createApp(store)` → 测试用 fake store，后续换成 PostgreSQL
- 安全关键：**服务端定价**（客户端只传 itemId + quantity，价格由服务端查表）
- 模拟支付 provider，402 Payment Required

**Module 7 — 闭环**
- 请求完整旅程：DNS → TCP 三次握手 → TLS → HTTP 请求 → 服务端处理 → HTTP 响应 → 浏览器渲染
- 罗永浩点咖啡 meme 的 API 视角解读：协议不匹配 vs 契约明确

### Git 操作
- [x] 复习常用 Git 命令
- [x] 在 `apis-and-orders` 仓库创建自己的分支（`Homework_xxx`）
- **作业**：在 orders-api 中新增 `DELETE` / `PATCH` / `GET` 分页过滤三个端点 + 加强验证

- **知识点**：Xsolla Code Camp 的课程节奏：先理解核心概念 → instructor 演示 → 自己动手写 → 用 `npm test` 验证
- **当前阶段**：Xsolla 课程第一天完成 + 本地开发环境就绪
- **待办**：
  - [ ] 完成 Xsolla 课后作业（HOMEWORK.md）

---

## Session 3 (2026-07-16) — Xsolla 课后作业完成

### 分支管理
- [x] 重命名分支：`Homework_余尧` → `Homework_yuyao`（不允许中文名）
- [x] 推送新分支到 GitLab，删除远程旧分支

### 课后作业（HOMEWORK.md）完成情况

**A0 — 新增校验规则**（`validate.js`）
- [x] `quantity` ≤ 100 校验
- [x] `amountUsd` 最多 2 位小数（乘以 100 判断是否为整数）
- [x] 拒绝未知字段（白名单模式）
- [x] `validatePatchOrder`：PATCH 部分更新校验（至少提供一个字段、amountUsd 不可编辑、未知字段拦截）
- [x] `validateListQuery`：查询参数校验（limit 1~100、offset ≥ 0、status 枚举）

**A1 — DELETE /orders/:id**（`server.js`）
- [x] pending 订单 → 200，状态变为 cancelled（软删除）
- [x] paid 订单 → 409 `order_not_editable`
- [x] 不存在 → 404 `not_found`

**A2 — PATCH /orders/:id**
- [x] pending 订单可修改 item 和/或 quantity → 200
- [x] 校验不通过 → 400 `invalid_update` + `details[]`
- [x] paid/cancelled 订单 → 409 `order_not_editable`
- [x] amountUsd 不可编辑

**A3 — GET /orders 分页+过滤**
- [x] 替换原有裸数组返回为 `{ data, total, limit, offset }` 结构
- [x] limit 默认 20，offset 默认 0
- [x] 支持 `?status=` 过滤（先过滤后分页）
- [x] 参数非法 → 400 `invalid_query`

**测试**
- [x] 更新原有 GET /orders 测试以适配新返回格式
- [x] 全部 17 个测试通过 ✅

### 知识巩固
- **HTTP 方法详解**：GET（查，幂等）、POST（增，非幂等）、PUT（全量替换）、PATCH（部分更新）、DELETE（删）
- **RESTful 设计**：资源用名词、动词用 HTTP 方法、URL 参数做过滤/分页
- **参数错误（400）** 的含义：客户端传参不合法，服务端拒绝处理
- **JSON 字段含义**：`id` / `item` / `quantity` / `amountUsd` / `status` / `createdAt`
- **测试文件作用**：`orders.test.js` 含 17 个用例，覆盖数据层、校验、路由、并发防护

### Lecture 2 课前准备（新课：API 设计，Go 语言）
- [x] Git — ✅ 2.54.0
- [x] Go 1.22+ — ✅ 已安装 1.26.5（配置 PATH 环境变量）
- [x] VS Code — ✅ 1.129.0
- [x] Postman — ✅ 已安装
- [x] GitLab 连通 — ✅ 可访问
- [x] 克隆 `api-design` 仓库到 `C:\Users\asus\api-design`

### 知识点
- 将 Go 安装路径 `C:\Program Files\Go\bin` 添加到用户 PATH 环境变量
- 400 ≠ 服务器 bug，是客户端参数问题；500 才是服务器故障
- 测试驱动开发（TDD）流程：RED（写失败测试）→ GREEN（实现通过）→ REFACTOR（重构优化）

- **当前阶段**：Xsolla 课程作业完成，准备进入 API 设计课程（Go 语言）
- **待办**：
  - [ ] 学习 Go 语言基础
  - [ ] 完成 API 设计课程（api-design 仓库）
