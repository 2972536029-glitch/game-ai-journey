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
  - [x] 完成 API 设计课程作业（api-design 仓库）

---

## Session 4 (2026-07-21) — API 设计作业（Go 语言）

### 作业任务完成情况

远程仓库 `api-design` 已更新了购物车和订单的基础代码（含幂等中间件）。我在此基础上重新实现了三个作业任务：

**Task 1 — DELETE /cart/items/{sku}**
- [x] store 层（`cart/store.go`）：新增 `RemoveItem` 函数
  - 先加锁 `mu.Lock()` 保证并发安全
  - 从 `store[sessionID]` 取出该用户的商品列表
  - 遍历过滤：`item.SKU != sku` 时才保留（不等于目标 SKU 的留下 → 等于的就被删掉了）
  - 过滤结果写回 store → 删除完成
  - `copy()` 深拷贝一份再返回，防止外部拿到引用后误改 store 内部数据
- [x] handler 层（`cart/handler.go`）：新增 `RemoveItemHandler`
  - `r.PathValue("sku")` 从 URL 路径取 SKU
  - `getSessionID(r)` 从 Header 取用户身份
  - 调 `RemoveItem` → 计算总价 → 返回 JSON
- [x] main.go：注册路由 `DELETE /cart/items/{sku}`
- [x] 幂等设计：SKU 不存在时过滤结果不变，直接返回当前购物车，不报错

**Task 2 — 订单单元测试**
- [x] 新建 `orders/handler_test.go`，包含 3 个测试用例：
  - 订单存在 → 200 ✅：先存订单，模拟 GET 请求，验证状态码和 order_id
  - 订单不存在 → 404 ✅：查不存在的 ID，验证返回 404
  - 同一幂等键重复 POST → 相同 order_id ✅：第一次请求后清空 store，第二次用相同幂等键，验证中间件从缓存返回相同结果

**Task 3 — 优惠券系统**
- [x] 新建 `orders/coupon.go`：
  - `Coupon` 结构体：Code（券码）、Discount（折扣率）、MaxUses（上限）、UsedCount（已用次数）
  - 预置两张券：SAVE10（9折，限1次）、SAVE20（8折，限5次）
  - `UseCoupon(code)`：查字典 → 检查存在 → 检查次数 → `UsedCount++` 扣减
  - `ReleaseCoupon(code)`：支付失败时 `UsedCount--` 回滚
- [x] `orders/handler.go` 集成：
  - 解析请求体 `{"coupon_code":"SAVE10"}`
  - 支付前调 `UseCoupon` 扣次数 + 算折后价
  - 支付失败时调 `ReleaseCoupon` 回滚（保证幂等键重试不重复扣减）

### 个性化知识点回顾（基于学习过程中的疑问）

**1. 深拷贝（deep copy）vs 浅拷贝**
- 困惑："最后一步没懂，为什么要复制一份再返回？"
- 解答：`store[sessionID] = filtered` 已经把数据存进 store 了。直接 `return filtered` 的话，外部拿到的是 store 内部数据的引用，改了会影响 store。用 `copy(result, filtered)` 创建独立副本再返回，外部怎么改都不影响 store。
- 类比：**把箱子里的玩具直接递给你 vs 买个一模一样的玩具给你**

**2. 删除逻辑的理解**
- 困惑："为什么不等于 SKU 就放进新切片？我不是要删除吗？"
- 解答：遍历时 `item.SKU != sku` 才保留 → 等于 SKU 的那个不放进去 → 它就被"丢掉了"。这是**保留不等于的，丢弃等于的**，结果就是删掉了目标 SKU。

**3. Handler 层与 Store 层的分工**
- 困惑："第一步是什么层，这两层有什么意义吗？"
- 解答：
  - **Store 层（数据层）**：管数据的存、取、删。未来换数据库只改这里，handler 不用动。
  - **Handler 层（HTTP 层）**：管请求解析和响应。收到 URL 和 Header，调 store 层拿数据，返回 JSON。
  - 类似"仓库管理员 vs 前台接待员"或"后端 vs 前端的类比中的后端内部细化"

**4. 幂等键的原理**
- 困惑："你这里的幂等键不是随机生成的，是你定义成这个名字是吗？"
- 解答：对，测试里直接写死 `"idem-key-1"`。幂等键由客户端生成（如 UUID），同一操作重试时传相同值，后端中间件检测到已缓存就直接返回，不再执行 handler。

**5. Go 的 error 返回机制**
- 困惑："func UseCoupon(code string) error 这一句的 error 什么意义？"
- 解答：Go 的约定——函数返回 `(结果, error)`，`err == nil` 表示成功，`err != nil` 表示失败，错误信息里说明原因。调用方用 `if err != nil` 判断。

**6. 优惠券查字典 vs 扣减**
- 困惑："c, ok := coupons[code] 是指调用使用一次优惠券吗？"
- 解答：不是，这只是查字典（检查存在 + 获取信息）。真正的扣减是下一行的 `c.UsedCount++`。

**7. 支付失败的判断**
- 困惑："最后一个函数系统怎么判断支付失败了？"
- 解答：`MockCharge(total)` 返回 `(result, err)`。`err != nil` → 支付失败。模拟函数内部用 `rand.Float64()` 随机决定成功（80%）或失败（20%）。

**8. 三层架构设计**
- Store 层（数据管理）→ Handler 层（HTTP 处理）→ main.go（路由注册）
- 路由注册就像"公司门牌"，指定每个 URL 该找谁处理

### 分支管理
- [x] 创建分支 `yuyao-homework`，推送至远程 GitLab
- [ ] 提交 Merge Request

### 测试结果
- 全部 14 个测试通过 ✅（cart 11 个 + orders 3 个）
- 运行命令：`go test ./... -v -count=1`
- 可视化报告：`.\run_tests.ps1`
- 互动菜单：`.\interactive_test.ps1`
