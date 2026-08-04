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

---

## Session 5 (2026-07-23) — Xsolla 第二周：应用安全实验（App Security Basics）

### 项目背景
- 仓库：`app-security-basics` — 一个故意留有 5 个安全漏洞的 Go 迷你支付服务
- 任务：修复漏洞，让对应测试全绿，提 MR

### 5 个漏洞修复

**EX01 — JWT 身份认证中间件**（`internal/auth/middleware.go`）
- 问题：starter 不校验签名、不检查算法、不验证过期时间
- 修复：keyFunc 里强制要求 HMAC 算法（防 `alg: none` 和算法混淆攻击），手动检查 `iss == "xsolla"` 和 `exp` 未过期
- 通过 8 个测试

**EX02 — CORS 中间件**（`internal/cors/cors.go`）
- 问题：无脑回 `Access-Control-Allow-Origin: *` + `credentials: true`（浏览器拒绝此组合）
- 修复：Origin 白名单精确匹配，回声请求 origin（不用 `*`），加 `Vary: Origin` 头，处理 OPTIONS 预检请求（204 + Allow-Methods/Headers/Max-Age）
- 通过 6 个测试

**EX03 — XSS 输出编码**（`internal/comments/render.go`）
- 问题：用 `template.HTML(c.Body)` 把用户输入标记为"可信 HTML"，`<script>` 直接执行
- 修复：三步骤顺序——先正则剥掉 inline event handler（`onclick=`、`onerror=` 等），再用 `html.EscapeString` 转义特殊字符，最后把 `\n` 转 `<br>`
- 注意：顺序不能反（先换行后转义会把 `<br>` 也转掉）
- 通过 6 个测试

**EX04 — SQL 注入修复**（`internal/users/repo.go`）
- 问题：`fmt.Sprintf` 字符串拼接 SQL，`' OR '1'='1` 可绕过
- 修复：改为参数化查询 `WHERE email = ?`，数据库驱动自动处理转义
- 通过 5 个测试

**EX05 — HMAC Webhook 签名校验**（`internal/webhook/verify.go`）
- 问题：starter 完全放行，不校验签名和时间戳
- 修复：读取 `X-Signature` 和 `X-Timestamp` 头，用 `hmac.Equal` 常数时间比较签名（防时序攻击），校验时间窗口 ±5 分钟（防重放攻击）
- 通过 9 个测试

### 测试结果
- 全部 5 个包测试通过 ✅
- 运行命令：`go test ./internal/...`

### 小问题汇总

**1. JWT keyFunc 的作用？**
`jwt.Parse` 的第二个参数是一个回调函数。库在解析 token 后会调用它，问"这个 token 说自己用 XX 算法签的，你给不给验签密钥？"。在 keyFunc 里先检查算法是不是 HMAC 家族，不是就拒绝——这是防 `alg: none` 和算法混淆攻击的关键。

**2. 为什么 context 要存用户 ID？**
中间件校验通过后，必须把校验结果（用户是谁）传给下游 handler。Go 里中间件和 handler 之间没有参数传递通道，只能通过 `r.Context()` 传递。每个请求独立携带自己的 context，并发安全。

**3. CORS 预检（OPTIONS）是干什么的？**
浏览器发复杂请求（带自定义头、`application/json` 等）前，先发一个 OPTIONS 问服务器"你允许这些方法和头吗？"。服务器不回应 `Allow-Methods`/`Allow-Headers`，浏览器就不发真实请求。不是服务器强制要的，是浏览器强制规则。

**4. `*` + `credentials: true` 为什么不行？**
规范故意让这个组合不合法——需要用 credentials 就必须显式写明 origin，逼开发者认真想清楚哪些域值得信任。

**5. XSS 处理顺序为什么不能反？**
先转义后换行 → 我们插入的 `<br>` 不被转义（正确）。先换行后转义 → `<br>` 也被转成 `&lt;br&gt;`，换行效果消失。

**6. 为什么用 `hmac.Equal` 而不是 `==`？**
`==` 逐字节比较，第一个字节不匹配就立刻返回 false，攻击者可以通过响应时间猜"我第一个字节猜对了没"，逐字节爆破签名。`hmac.Equal` 不管匹配还是差一个字节，耗时都一样。

### 提交记录
- [x] 创建分支 `homework/yuyao`
- [x] 5 个作业分别 commit
- [x] 推送至 GitLab
- [ ] 在 GitLab 创建 Merge Request

### 待办
- [ ] 在 GitLab 上创建 MR，粘贴 MR 描述

---

## Session 5 补充 (2026-07-29) — App Security 作业 Review：两个工程改进 + 深度讨论

### 背景
- 仓库：`app-security-basics`（延续 Session 5 的 5 个安全作业）
- 触发：对 Session 5 作业的 review 反馈，提了两个"提高点"
- 这次不是修漏洞，而是做**工程化改进**：可观测性 + 性能优化

### 改进一：JWT 401 错误的可定位性（`internal/auth/middleware.go`）

**问题**：原来 7 个失败分支全部返回相同的 `401 unauthorized`，前端、日志都无法区分到底是哪种 401（过期？签名错？issuer 错？算法攻击？），线上排障全靠猜。

**方案**（结构化日志 + 标准 WWW-Authenticate 错误头）：抽出统一的出口函数 `rejectUnauthorized(w, code, reason)`，所有 401 走这一个口子：
- **服务端日志**：`auth: reject 401 code=xxx reason=xxx`，排障时 `grep` 一眼定位
- **响应头**：按 RFC 6750 / OAuth2 设 `WWW-Authenticate: Bearer error="invalid_token"`
- **响应体**：仍统一 `unauthorized`，不向攻击者泄露细节

错误码遵循 OAuth2 规范分两类：
- `invalid_request`：缺 Authorization 头 / 格式不是 Bearer
- `invalid_token`：过期 / 签名错 / 算法错 / issuer 错 / claims 异常（过期单独识别，因为最高频）

改动统计：38 行增，7 行删（删的全是原来重复的 `http.Error`）。

### 改进二：评论渲染性能优化（`internal/comments/render.go`）

**问题**：`RenderComments` 每次调用都 `template.New().Parse()`，而模板是常量字符串，纯属重复解析。每个 HTTP 请求都重新"构建一次框架"，再灌入评论。

**方案**（模板预编译到包级）：用 `template.Must` 把模板提到包级全局变量，程序启动时编译一次，之后所有请求复用同一个模板对象。顺带把转义逻辑抽成 `safeComment()` 函数。

**实测（50 条评论 benchmark）**：

| 指标 | 改前 | 改后 | 提升 |
|------|------|------|------|
| 耗时 | 109.6 μs | 87.1 μs | -20% |
| 内存分配 | 34.0 KB | 19.8 KB | **-42%** |
| 分配次数 | 973 次 | 855 次 | -12% |

### 提交记录
- [x] 两个 commit 分别提交（`EX01(review)` / `EX03(review)`）
- [x] 推送至学校 GitLab，自动合并进已有 MR #15
- [x] 全部测试通过 ✅

---

### 个性化深度讨论（基于追问，价值最高部分）

这次最大的收获不是代码本身，而是通过一连串追问，从"代码"一路上升到"设计哲学"。以下记录讨论链条。

**1. JWT 为什么不把具体错误告诉前端？——信任边界**
- 困惑："前端拿到大类错误，攻击者用 Postman 不也能拿到吗？"
- 关键认知：**HTTP 是开放协议，前端所有"校验/隐藏"对攻击者都是透明的、可绕过的。** 前端的安全责任为零，安全决策永远在后端。
- 三层可见性设计：
  - 终端用户 → 永远 `unauthorized`（统一，不泄露）
  - 前端程序/网关 → 响应头大类 `error=invalid_token`（够前端智能处理）
  - 开发者/运维 → 服务端日志详细 reason（排障）
- **为什么敢给前端大类？** 因为前端"有我们的工作流上下文"（知道 token 机制、refresh 流程），能据此定位问题；攻击者"没有工作流"，大类对他就是噪声。这就是「信任边界」。

**2. 大类错误对攻击者到底有没有用？——安全模糊性**
- 困惑："大类不也泄露了技术栈吗？"
- 诚实结论：泄露"几乎为零"是夸张了，攻击者确实多知道"这系统用了标准 OAuth2"。但具体漏洞（签名？过期？issuer？）全部混在一个大类里，攻击者无法用控制变量法探测，**探测成本极高**。
- 这种"统一响应防枚举"叫 security through obscurity 的合理使用：自动化扫描器（占攻击 90%）靠响应差异找漏洞，大类让它无从下手。

**3. 这套安全未来会不会被算力破解？——密码学底气**
- 困惑："计算机算力到新量级，JWT 这套不就失效了？"
- 核心区分两种安全：
  - **计算安全性**（算出来太贵）→ 算力提升会削弱
  - **信息论安全性**（数学上不可能）→ 永不失效
- HMAC 属于"密钥空间指数增长"：算力涨 1 亿倍 ≈ 密钥只少 27 bit，176bit 密钥依然安全。
- 真正的威胁不是经典算力，是**量子计算机**：Shor 算法秒杀 RSA/ECC，但 Grover 算法对 HMAC 只让强度减半（256bit → 128bit，仍安全）。所以 HMAC-SHA256 量子安全。
- **但实际攻击里算力/数学只占 2%**，98% 是密钥泄露、代码 bug、社会工程——"人祸"。真正的破绽是弱密钥（如测试里 `s3cr3t-do-not-use-in-prod`）。

**4. 模板常驻内存会不会是缺点？——空间换时间**
- 困惑："常驻内存不是对服务器内存的挑战吗？"
- 诚实回答：确实是 trade-off。但实测这个模板常驻只占 **7.8 KB**（占最便宜 VPS 的 0.0015%），换来每次请求省 23KB 临时分配 + 更低 GC 压力。
- 反直觉点：旧代码"用完释放"看似省内存，但高并发时 100 个请求同时解析 → 同时存在 100 份临时对象（2.3MB 峰值），反而比常驻（恒定 7.8KB）更费内存。
- 通用判断准则：
  - 小 + 高频 + 不变 → 常驻内存（模板、正则、配置）
  - 大 + 低频 + 不变 → 按需加载 + 缓存
  - 大 + 高频 + 变化 → 按需加载 + 复杂缓存
  - 大 + 低频 + 变化 → 用完释放

**5. ⭐ 两个改动背后的共同哲学——所有工程设计都是权衡**
- 最重要的一层认知：JWT 题（安全性 ↔ 可用性）和模板题（时间 ↔ 空间），表面不同，底层都是**在两个互相拉扯的力之间找平衡点**。
- 工程没有"绝对正确"，只有"在当前场景下最合适"。给大类还是给细节、常驻还是按需，换场景答案就变。
- 这门课的 5 个作业全是权衡：JWT（安全↔便利）、CORS（开放↔安全）、XSS（功能↔安全）、SQL 注入（灵活↔安全）、HMAC（性能↔完整性）。没有"纯赚"的安全措施，每个都是用代价换收益。
- 金句：**Engineering is not about finding the right answer, it's about finding the best trade-off under constraints.**

### 工具认知补充（本次延伸）
- **ZCode 的手机远程控制（Remote Control）**：原生支持，不用折腾 SSH/Tailscale。桌面端 ZCode 左下角手机图标 → 扫码 → 手机浏览器即可连接当前 workspace。电脑是 runtime，手机是遥控器。
- **ZCode 必须联网**：它是云端 GLM-5.2 API 的客户端，没有本地模型选项。断网 = 无法使用。
- **本地部署 GLM-5.2 是企业级**：744B 参数模型，2-bit 量化最低需 256GB RAM + 24GB 显存，成本数万元起，且降智。个人用云端订阅远比本地部署划算。

- **认知里程碑**：从"理解代码"→"理解安全原理"→"理解密码学边界"→"理解工程权衡"，这是从码农思维到工程师思维的跨越

---

## Session 6 (2026-07-28) — Xsolla 第三周：数据持久化（Data Persistence & Code Organization）

### 项目背景
- 仓库：`data-persistence` — 课件 8 章覆盖 Transaction / Index / Table design / Schema / Query in codes / Join / Other databases
- 三个作业全部用 **Go 语言** 完成（删掉了初始的 Python 版本），对应课件 slide 8 / 14 / homework.md
- 环境：MySQL 9.7（REPEATABLE-READ 默认隔离级别），Go 1.26.5

### 三个作业完成情况

**作业一 — MySQL 事务隔离级别对照实验**（`schema.sql` + `isolation/`）
- [x] 用 Go goroutine + 同步通道（barrier）模拟两个并发 Session，精确控制 T1→T8 时序
- [x] 场景 A：`START TRANSACTION WITH CONSISTENT SNAPSHOT` → 三次查询都是 k=1 ✅
- [x] 场景 B：普通 `START TRANSACTION` → 第一次查询就是 k=2 ✅
- [x] 实测验证两个场景，输出完全符合预期

**作业二 — 电商系统表设计**（`goods_orders.sql`）
- [x] 设计 users / goods / orders 三表，InnoDB + utf8mb4
- [x] goods 1:N orders（外键关联，对应 "goods belonging to it"）
- [x] 字段类型：金额 DECIMAL(10,2)、状态 ENUM、时间 DATETIME
- [x] 测试数据 3 用户 + 4 商品 + 5 订单 + 3 条 JOIN 查询示例

**作业三 — database/sql 实现 CRUD**（`crud/`，6 个 Go 文件）
- [x] Goods / Orders 完整 CRUD，分层组织（models / db / repo / main）
- [x] 事务性下单 `CreateOrder`：`FOR UPDATE` 行锁 + 扣库存 + 写订单原子提交
- [x] 实测：下单 5 个库存 200→195；下单 999999 个库存不足，事务回滚库存不变

### 提交记录
- [x] 创建分支 `yuyao-homework`，提交 16 个文件（1890 行）
- [x] 密码以占位符 `your_password` 提交（不泄露真实密码）
- [x] 推送至学校 GitLab
- [ ] 在 GitLab 创建 Merge Request

### 个性化知识点回顾（基于学习过程中的疑问）

**1. 隔离级别和"快照时机"是两件事**
- 困惑："可重复读不是应该读到旧值吗？为什么场景 B 读到了新值 k=2？"
- 解答：场景 B 依然是完美的可重复读。MySQL 的可重复读靠 **MVCC 快照** 实现，承诺的是"快照建立后别的事务再提交你看不到"，**不承诺**"事务一开始数据是啥样你就只能看到啥样"。
- 类比：方案 A 像一进电影院就戴上耳机（录音是进场那一刻录的）；方案 B 像进场后发呆，等电影演一半才戴耳机（录的是半场内容），但戴上后内容不再变。
- **关键区别**：`WITH CONSISTENT SNAPSHOT` 让快照在事务开启瞬间建立；普通 `START TRANSACTION` 让快照延迟到第一条 SELECT。两者都满足可重复读，差别只在"以哪个时间点为准"。
- **实际用途**：`mysqldump --single-transaction` 用它保证导出整个库时所有表来自同一时间点（一致性备份）。

**2. 金额为什么必须用 DECIMAL，绝不用 FLOAT/DOUBLE？**
- 困惑："差那么一点点小数有啥关系？"
- 解答：浮点数是近似存储，`0.1 + 0.2 = 0.30000000000000004` 不是 0.3。电商一天百万订单，每笔差一丢丢，月底财务对账就对不上。`DECIMAL(10,2)` 是定点数，存的就是精确十进制值，零误差。这是数据库设计第一铁律。
- 替代方案：用 INT 存"分"（299 = 2.99 元）也能规避浮点，但 DECIMAL 更直观。

**3. 状态为什么用 ENUM 而不是 VARCHAR？**
- 解答：ENUM 只存整数索引（1-2 字节），天然约束取值（填错报错），查询快（整数比较）。代价是新增状态要 `ALTER TABLE`。所以：状态稳定（像订单状态）用 ENUM；状态频繁变动用 VARCHAR + 应用层校验。

**4. 主键为什么用自增 id 而不是业务字段（如商品名）？**
- 困惑："slide 13 问 Must there be an id column？"
- 解答：主键要满足①唯一②永不变③短小高效④与业务无关。商品名会重名、会改名，字符串还占空间、索引慢。用"和业务无关的自增整数"（代理键 surrogate key）是业界标准。而且外键要引用主键——如果 goods 用商品名当主键，orders 的 goods_id 就得存一长串名字，改名时所有订单外键都要跟着改，灾难。

**5. 外键到底有什么用？**
- 困惑："需求只提到 goods 和 order，为什么要加 users 表？"
- 解答：需求里的 "user" 本质是"订单属于谁"。直接存用户名字符串（方案A）的问题：用户改名要改所有订单、没法保证是真实用户。用外键关联 users 表（方案B）：用户改名只改一处、外键保证每个订单指向真实用户。这就是数据库范式——不要多处重复存同一份数据。
- **外键的实际价值**：数据库主动拒绝脏数据。比如插一个 `goods_id=999`（不存在的商品）的订单，数据库直接报错 `foreign key constraint fails`，不用靠应用层代码检查。

**6. 一个订单只能买一件商品？这设计够吗？**
- 解答：我们的简化版是 1:N（一个订单关联一个商品），但真实电商购物车要买多种商品，需要三表结构：orders（订单主表）1:N order_items（订单明细）N:1 goods（商品）。每个订单可含多条明细。
- **关键细节**：order_items 里要冗余存 `unit_price`（下单时单价快照），因为商品价格会变，但订单金额不能变。这和作业一的"快照"思想异曲同工——关键时刻定格，后续变化不影响已发生的事实。

**7. 为什么扣库存和写订单要在同一个事务？**
- 困惑：作业三那句 "data inserted should reflect the relationship" 怎么体现？
- 解答：下单同时改两张表（扣 goods 库存 + 插 orders 订单）。不用事务会出"库存扣了但没订单"的不一致（仓库少货却找不到买家）。用事务保证 ACID 的 A（原子性）——要么都成功要么都回滚。这正是用作业一的事务知识保证作业二的表关系一致性。

**8. `FOR UPDATE` 行锁是怎么防超卖的？**
- 困惑："SELECT 加个 FOR UPDATE 有啥用？"
- 解答：并发场景下，两个事务同时读到 stock=1，都以为够，都扣减，结果 stock 变 -1（超卖）。`SELECT ... FOR UPDATE` 给这行加写锁，第二个事务必须等第一个提交后才能读，强制串行执行。这是我们代码里防超卖的核心手段。

**9. Go 代码为什么要分成 6 个文件？**
- 解答：呼应课程标题 "Code Organization"。按职责分层：
  - `models.go` 数据结构层 → `db.go` 基础设施层（连接池）→ `*_repo.go` 数据访问层（Repository 模式，每表一个仓库）→ `main.go` 编排层
  - 每层各司其职：换数据库只改 repo 层、改连接配置只改 db 层、main 只管"做什么"不管"怎么做"。这是 Repository 模式的雏形。

**10. `?` 占位符为什么能防 SQL 注入？**
- 解答：字符串拼接（`fmt.Sprintf`）会被注入——如果用户名是 `'); DROP TABLE goods;--`，拼接后整条 SQL 变成删除表。`?` 占位符让驱动把参数当**纯数据**处理，特殊字符自动转义，彻底杜绝注入。这和 Session 5 的 EX04（SQL 注入修复）是同一个知识点。

**11. `defer rows.Close()` 为什么必须写？**
- 解答：`db.Query` 会从连接池**借走一个连接**，直到 `Close` 才归还。忘了关就泄漏连接，池子迟早被掏空程序卡死。`defer` 保证函数退出前一定关闭（即使 panic 也执行）。同理 `tx` 的 `defer Rollback` 兜底也是这个思路。

**12. `sql.Open` 之后为什么一定要 `db.Ping()`？**
- 解答：`sql.Open` 是惰性的，只验证 DSN 格式对不对，**根本没连数据库**。密码错了或 MySQL 没启动，`Open` 不报错，要等到第一次查询才炸。`Ping()` 强制立即建连接，把问题提前暴露。

### 横向串联：三个作业都在回答"怎么保证数据是对的"
- 作业一 → **读一致性**（隔离级别 + 快照时机）
- 作业二 → **结构一致性**（外键约束 + 合理类型）
- 作业三 → **操作一致性**（事务 + 行锁）
- 贯穿主题：数据完整性是数据库设计的核心追求，索引/Join/其他数据库（Redis/ES/TiDB）都是在不同维度平衡一致性与性能

### 知识点
- Xsolla 课程第三周节奏：先讲 ACID/隔离级别理论 → 自己动手建表 → 用 database/sql 写 CRUD
- 学习日志的"小问题汇总"写法很有用：记录真实疑问 + 自己的语言解答，比抄概念记得牢
- Go 操作数据库的标准库就是 `database/sql` + 驱动（`go-sql-driver/mysql`），`sql.DB` 是连接池管理者而非单个连接

- **当前阶段**：Xsolla 课程第三周作业完成（全 Go 实现），三个作业已推送 GitLab
- **待办**：
  - [ ] 在 GitLab 创建 Merge Request
  - [ ] 预习课件后半部分：Index / Join 性能 / 其他数据库（Redis、ES、TiDB）

---

## Session 7 (2026-07-30) — Xsolla React 前端课程：两讲作业连做（GitHub Profile Finder + React Hooks Playground）

### 课程新主题：前端 React
Session 2-6 都是后端（APIs / Go / Security / Database），今天正式进入前端 React 主题，一天连做两讲作业，技术栈 React 19 + TypeScript + Vite。

### 第一讲作业：GitHub Profile Finder（React 入门）

**项目背景**
- 仓库：`checkout-demo` 下建子项目 `homework/github-finder`（独立 package.json 自治运行，和源码区分开）
- 技术栈：React 19 + TS + Vite + React Compiler + 原生 fetch（沿用 checkout-demo 约定）
- 数据源：GitHub API `https://api.github.com/users/{username}`

**完成情况**
- [x] 必做项全做：受控搜索框 + 提交按钮、头像/姓名/简介/粉丝/仓库数、loading 状态、404 友好提示
- [x] 加分项全做：最近仓库列表（`/users/{username}/repos`）、GitHub 风格卡片、localStorage 最近 5 次搜索、防抖 debounce（500ms）
- [x] UI 重新设计：渐变光晕风（glassmorphism 玻璃拟态 + 渐变背景 + 浮动光斑动画 + 深浅色自动切换）
- [x] 写了 CHANGELOG.md 改动总结

**核心技术点**
- **受控表单**：`<input value={state} onChange={e=>setState(...)}>`，一个输入对应一个 state（React 单向数据流的核心写法）
- **三态模式**：loading / error / success，每个异步请求都要管这三态
- **竞态处理**：useEffect 里用 `cancelled` 标志，连续搜索时丢弃过期响应（旧请求回来时 cancelled 已为 true，直接 return）
- **localStorage 惰性初始化**：`useState(() => JSON.parse(localStorage...))`，只在首次渲染读一次，不每次渲染都解析
- **debounce**：useEffect + useRef 存 timer id + cleanup clearTimeout，输入停顿 500ms 才触发搜索

**提交记录**
- [x] 分支 `yuyao-homework`（一开始误用 `feat/github-finder`，按名字+homework 惯例改名）
- [x] GitLab MR !2，Open / Ready to merge（改名时关掉旧 !1，重开 !2）

### 第二讲作业：React Hooks Playground（三 Task + 概念题）

**项目背景**
- 仓库：`react-hooks` 的 `playground` 文件夹（远程更新 commit 5b87849 新增）
- 权重：概念题 ~40%、Focus Timer ~25%、Live Search ~35%、Concurrent Filter 加分 +10

**完成情况**

| Task | 文件 | 核心实现 |
|------|------|---------|
| **1 · Focus Timer**（必做·热身）| `useCountdown.ts` / `FocusTimer.tsx` | useState(秒数/运行态) + useRef(interval id/最新 onDone) + useEffect(每秒 tick，依赖 `[isRunning]`，cleanup `clearInterval`)；到 0 用独立 effect 触发 onDone；session log 持久化到 localStorage（惰性初始化） |
| **2 · Live Search**（必做·主计分）| `useDebouncedValue.ts` / `LiveSearch.tsx` | 泛型 `<T>` debounce（cleanup `clearTimeout`）；每次 effect 新建 `AbortController`、signal 传给 `searchProducts`、cleanup 里 `abort()`；`AbortError` 吞掉；四态齐全（idle/loading/error/空查询） |
| **3 · Concurrent Filter**（加分·难）| `ConcurrentFilter.tsx` | `useDeferredValue(query)` 只在 concurrent 模式驱动列表；`ResultRow` 包 `React.memo`；`isStale` 控制 dim + "updating…"；naive 模式保持卡顿作对比 |
| 概念题 | `CONCEPT-CHECK.md` | A1-A3 / B1-B3 / C1-C2 全部作答，每题写"为什么" |

**浏览器验证**
- [x] Task 2：搜 `shoe` → 40 条全是 Shoes 分类；搜 `fail` → 报错状态；清空 → 回到提示
- [x] Task 3：concurrent 模式输入 `crimson` → 列表正确过滤、输入框保持最新值

**构建与提交**
- [x] `npm run build` + `npm run typecheck` 通过（playground 没配 ESLint）
- [x] commit `c95c71f`，推送 `yuyao-homework` 分支
- [x] GitLab MR !1，Open / Ready to merge

### 代码深度讲解（2026-07-31 晚）

白天实现了全部作业，晚上从第一讲到第二讲逐个深入过代码。记录真实疑问 + 自己语言的解答。

#### 第一讲 GitHub Profile Finder —— 建立地基

**1. DOM 是什么？为什么要 React？**
- 困惑："你说的 DOM 是什么？"
- 解答：DOM = 浏览器眼里的网页，一棵由 `<html>/<body>/<div>` 组成的树。传统写法要亲自操作 DOM（翻树、找节点、改文字）；**React 接管了这件事**——你只管数据（state），React 帮你算出 DOM 该怎么变。手动挡 vs 自动挡：你只管目的地，换挡 React 来。

**2. 页面是 state 的「照片」**
- 核心认知：state（状态）= 决定页面长什么样的那份数据。数据变 → React 自动重新拍一张照片 → DOM 跟着变。你永远不用操心"去哪个 DOM 改文字"。

**3. 受控组件：输入框是 state 的傀儡**
- 困惑："onchange 不是负责改变用户数据的吗？怎么到你这儿变成监听了？"
- 解答（我纠正后）：`onChange` 和 `useEffect` **都是监听**，只是监听对象不同：
  - `onChange` 监听**用户动作**（贴在输入框上，用户打字才触发）
  - `useEffect([username])` 监听**数据变化**（贴在 state 上，username 变了就触发，不管怎么变的）
- 关键追问："必须用 onChange 改变 state 吗？" → 不必。点历史标签的 `onClick`、代码直接调 `setUsername` 都能改。**改变 state 的真正动作是 `setUsername`，onChange 只是"谁来按开关"的一种手指。**
- 受控的真正含义：输入框被 state 死死锁住，没有自由意志。**删掉 onChange → state 改不了 → React 每次重渲染都把输入框强行拉回原值 → 打不了字。**（实测验证）

#### 第二讲 Task 1 Focus Timer —— useCountdown 逐段拆解

**4. state vs ref —— 第一性判断准则**
- 判断："这个值变了，页面要不要跟着变？" 要变 → useState；不变 → useRef
- 困惑："启动次数"该用哪个？→ useState（因为页面要显示"启动次数：3"，值变了页面要变）
- useCountdown 里：秒数/运行态是 state（屏幕要更新），interval id 是 ref（只是给 cleanup 用，不影响 UI）
- 精确化："useRef 不能显示"这个说法因果反了 → 应该是"**因为不需要显示，所以选 useRef**"。ref 改值不触发渲染，硬显示也只能停在初始值不动。

**5. cleanup 清理函数 —— 防定时器堆积（Task 1 的灵魂）**
- 困惑："删掉 cleanup 会怎样？" → 答对了一半（内存泄漏），漏了更致命的：**定时器堆积导致秒数加速**
- 具体场景：Start → Pause → Start，每次重跑 effect 都开新定时器但不关旧的 → 1 个、2 个、3 个定时器同时跑 → 每秒减 1、减 2、减 3…功能直接崩
- StrictMode 在 dev 模式故意把 effect 跑两遍，就是逼你暴露漏写 cleanup 的 bug

**6. ref 中转 onDone —— 闭包陷阱的解法（整个文件最巧妙处）**
- 困惑："为什么定时器会跟着小明换人而频繁重开？这个中间逻辑在哪？"
- 解答：逻辑桥梁在**依赖数组**（保安查名单）。如果 `onDone` 在依赖数组里，而 onDone 每次渲染都是新对象（父组件重新造的函数），React 就监测到"名单变了"→ 关开定时器。
- 套路：`onDoneRef.current = onDone`（悄悄更新 ref，不停定时器）+ effect 依赖只有 `[isRunning]`（onDone 不进名单）+ 定时器触发时读 `onDoneRef.current()`（永远最新）
- 比喻：**前台通讯录**。小明（onDone）每次被重新招一个新人，去前台更新通讯录；定时器到点不直接找小明，而是翻通讯录拿最新联系方式。
- 追问："为什么 onDone 每次都要更新？" → 因为父组件每次渲染 `function handleDone(){}` 重新执行，造出**全新的函数对象**（哪怕内容一样）。React 只认"是不是同一个对象"，内容一样但对象不同也算"变了"。不更新 ref → 定时器会调到过期旧版本。
- 正统解法对比：`useCallback` 让父组件缓存函数（让函数"不变"）；ref 中转让函数"变着也无所谓"。两种都能解，hook 内部用 ref 中转是更通用的套路。

#### 第二讲 Task 1 FocusTimer.tsx —— 惰性初始化 + localStorage

**7. 惰性初始化：少算废活，不是少渲染**
- `useState(readSessions)`（传函数，无括号）vs `useState(readSessions())`（传值，有括号）
- 困惑："加括号能跑吗？" → 能跑，但每次渲染都白执行一遍 `读 localStorage + JSON.parse + 校验`
- 精确化："渲染更麻烦"要改成"每次渲染多干废活"——**渲染次数不变，变的是每次渲染的工作量**。惰性初始化不减少渲染次数，减少每次渲染的工作量。

**8. 读少写多：state 是主，localStorage 是备份**
- 读：只第一次（惰性初始化，搬进内存后用内存的）
- 写：每次 sessions 变都写（保证备份永远最新）
- 困惑："为什么不能反过来读多写少？" → 写只一次 → 后续变化全存不下来 → 刷新就丢数据（用户专注 3 次刷新只剩 1 次）
- 核心：state 是主，localStorage 是副（备份）。备份必须紧跟主（写多次），主不用反复抄备份（读一次）。

#### 第二讲 Task 2 Live Search —— 防抖 + 竞态

**9. 防抖 = 延迟 + cleanup**
- cleanup 是灵魂：每次 value 变开新 setTimeout，cleanup 清掉旧的，只有最后一次活下来
- 删掉 cleanup 退化成单纯延迟：每次输入都延迟 300ms 触发，请求一个没省

**10. 竞态问题：请求发出顺序 ≠ 返回顺序**
- 困惑："为什么显示 s 的结果而不是 shoe？" → 我说反了，纠正：是 **shoe 先返回（快），s 后返回（慢）**。最慢的 s 最后回来，把正确的 shoe 结果覆盖了。
- 核心：网络请求不是排队，先发的不一定先到（像寄快递）。最后回来的（最慢的）请求会用过时结果覆盖正确结果。
- 我自己想到了两种解法：① 版本号机制（递增标识，回来比大小，过期就丢弃）② kill 掉上一个请求（AbortController）。两个全对，是业界两大主流解法。
- 为什么作业强制 AbortController：版本号只"忽略结果"，请求还在后台跑占资源、还会往已卸载组件塞数据。AbortController **真正掐断请求**，干净利落。

#### 第二讲 Task 3 Concurrent Filter —— 为什么会卡 + 怎么解决

**11. 为什么打字会卡：主线程被渲染霸占**
- 浏览器只有一个主线程，渲染列表（250ms）和处理键盘输入抢同一个工人
- 困惑："每个字停顿 2.5 秒？" → 纠正：是每个字卡 **250ms**（四分之一秒），不是 2.5 秒。2.5 秒是 7 个字累积。用户讨厌的是"每一帧都卡"，不是"总共慢"。

**12. useDeferredValue：设优先级，输入急诊、列表门诊**
- `query`（紧急，输入框用）立刻更新；`deferredQuery`（慢半拍，列表用）等空闲再追
- 列表显示旧数据不是 bug 是好事：宁可晚一点看到（像直播延迟几秒），也要保证打字流畅

**13. 光 defer 不够，必须配 memo（Task 3 的陷阱题）**
- 困惑："memo 是什么？" → 是给组件套的壳，props 没变就跳过重新渲染（门卫检查）
- 为什么光 defer 不够：React 渲染输入框那次，默认会顺带重新渲染所有 250 行（即使内容没变也白渲染）→ defer 白用
- 精妙连锁：defer 让 deferredQuery 慢半拍 → 列表过滤数据暂时不变 → 传给 250 行的 props 暂时不变 → memo 看到没变全跳过 → 渲染瞬间完成 → 打字不卡。**两个必须一起用，缺一不可。**

#### 概念题 CONCEPT-CHECK 拆解（8 题，但知识点 Task 1/2/3 几乎全覆盖）

**14. B1 push 的 bug —— React 靠「身份/地址」判断变化**
- 困惑："push 机制不明白" → 解答：JS 里改变量有两种方式
  - `push`（原地改）：在原本子上加内容，**还是同一个本子、同一地址**
  - `[...prev, item]`（造新的）：抄一本新本子，**新地址**
- React 判断"变没变"用 `===` 比地址，不比内容。push 后地址没变 → React 认不出 → 不渲染（你加了苹果屏幕没反应）
- 追问："state 变化都意味着地址变化吗？" → 不是"意味着"，是"**你必须保证**"。React 契约：改 state 必须给新地址（造新对象），否则 React 探测不到。这就是"不可变(immutable)"原则的根因。
- 例外：基本类型（数字/字符串）天生不可变，没有地址概念，直接比值，不存在 push 的坑。坑只在对象和数组上。
- 这跟 Task 3 的 memo 呼应：memo 判断"跳不跳过"也靠 `===` 比地址。原地改会骗过 memo。

**15. 衍生状态直接算（C1）+ key 重置（C2）—— 你可能不需要 effect**
- C1：总价能从 items 算出来 → 别存 state 也别用 effect 同步，渲染时直接 `items.reduce(...)`。原版会渲染两次（先旧值闪一下，再 effect 更新），叫"闪烁(flicker)"。
- C2：userId 变化时重置 state → 不用 effect 手动清，用 `key={userId}`。key 变了 React 当成全新组件 → 销毁旧的（连同 state）→ 新组件 `useState('')` 重新初始化。**一锅端，所有 state 干净重置，只渲染一次。**
- 共同主题（呼应 B3）：**"你真的需要 effect 吗？"** effect 是常被滥用的工具，很多同步逻辑能直接算或用 key 解决。React 官方有篇《You Might Not Need an Effect》专门讲这个。

- **认知里程碑**：从"照着 README 填 TODO"到"能讲清每行代码的 why"——state vs ref 的判断准则、cleanup 的必要性、闭包陷阱的 ref 解法、不可变原则的根因、并发渲染的 defer+memo 配合。这些是 React 的底层心智模型，不是 API 记忆。
- **当前阶段**：Xsolla React 前端课程两讲作业全部完成并提交，两份 MR 均 Ready to merge；代码已逐个深入过完，概念题全部讲透
- **待办**：
  - [x] 晚上逐个深入过代码（从 useCountdown 的 state vs ref、cleanup 开始）
  - [x] 等老师批改第二讲（MR !1）—— 已批改，3 条 review，全部修复 + 回复
  - [ ] 等老师批改第一讲（MR !2）

---

## Session 7 补充 (2026-08-04) — React Hooks 作业 Review：3 条评审 + 深度原理讨论

### 背景
第二讲作业（react-hooks playground，MR !1）被老师 Lingfan YANG (@lf.yang，项目 Maintainer) review，挑了 3 条刺。逐条修复 + 回复 + 深度讨论，把每条背后的原理吃透了。

### 老师的 3 条 review + 修复

**① FocusTimer.tsx — `useRef` 初值每次渲染白跑**
- 原代码：`useRef<number>(sessions.reduce(...)+1)` —— reduce 每次渲染都算，但只有第一次被采用
- 老师原话：「这里期待的结果是组件初始化设置初始状态，但实际行为是会在视图更新的时候反复跑这个循环。」
- 修复：lazy ref 惯用法 —— `useRef<number | null>(null)` + `if (nextIdRef.current === null)` 才计算
- **核心知识点**：`useRef` 不像 `useState` 支持惰性初始化（传函数），括号里的值每次渲染都求值。要惰性必须手写 `if (ref.current === null)` 这个惯用法。

**② useCountdown.ts — ref 中转异步更新有旧值风险**
- 原代码：`useEffect(() => { onDoneRef.current = onDone }, [onDone])` —— 用 useEffect 同步 ref
- 老师原话：「这样赋值本质来说是异步的，如果下面在 useLayoutEffect 这种同步 hooks 里面可能用到旧值。」并推荐 `useLayoutEffect` / `useInsertionEffect` / `useEffectEvent`
- 修复：改用 React 19 的 `useEffectEvent`（`const onDoneEvent = useEffectEvent(onDone)`）
- **这条最有含金量，展开了整条知识链（见下方深度讨论）**

**③ LiveSearch.tsx — hint 提示区漏了 error 状态**
- 原代码：底部 hint 三元判断只有 空查询/loading/无结果，error 时误显示「No results」
- 老师原话：「没有对 error 状态做处理。」
- 修复：补上 `status === 'error'` 分支，显示「Search failed — please try again.」
- **教训**：这条性质比前两条重——README 明确要求「handle error」，属于「要求写了却没全覆盖」的验收流程漏洞，不是单纯技术瑕疵。反思后把「交付前逐条对照 README 验收」加进了自审流程。

### 深度讨论的知识点（围绕评审②展开）

这次讨论从「为什么 onDone 会变」一路追到「render 为什么必须纯净」，覆盖了 React 的核心心智模型：

**1. DOM 是什么（纠正认知）**
- DOM 不是「预制好的界面」，而是浏览器把 HTML 读懂后**在内存里搭出来的一棵树**，每个标签是一个节点
- 类比：HTML 源码=图纸，DOM=按图纸盖好的房子（可改的真实结构），paint=给房子拍照给用户看
- **DOM 变了 ≠ 屏幕变了**，中间隔着 paint 这一步

**2. React 一次渲染的时间线（理解 effect 时机的基石）**
```
① render(渲染)         → React 算出新虚拟 DOM（只算不写，必须纯净）
② 更新 DOM             → 真改 DOM
③ useLayoutEffect 跑   → 同步，绘制前（阻塞绘制！）
④ 绘制(paint)          → 用户看到画面
⑤ useEffect 跑         → 异步，绘制后（不阻塞）
```
- useEffect 在 ⑤（绘制后），useLayoutEffect 在 ③（绘制前），这个先后是旧值问题的根源

**3. 为什么 onDone 每次渲染都是新函数**
- 组件就是个函数，渲染=重新调用这个函数 → `function handleDone(){}` 这行被执行一次 → 造出一个新函数对象
- 哪怕代码内容一模一样，每次都是「新造的杯子」（新内存地址）
- React 判断依赖变没变用**身份比较**（看地址），不是内容比较 → 每次 onDone 身份都变 → 不能放进依赖数组

**4. 闭包与闭包陷阱**
- **闭包** = 函数出生时「拍下」了周围的数据（像拍照定格）
- **闭包陷阱** = 这个函数后来被调用时，看的是旧照片，不是最新数据
- 在 React 里：effect 里调用的旧函数，看的是它出生时的旧 state → 数据错了

**5. useCountdown 的核心矛盾（为什么需要 ref 中转/useEffectEvent）**
```
onDone 写进依赖数组  → 每次渲染都重跑 effect → 定时器被拆掉重建 → 计时器废
onDone 不写进依赖数组 → 闭包陷阱 → 到 0 时调的是第一次的旧 onDone → 数据错
```
两条死路，所以需要第三条：「不进依赖数组，但调用时拿最新值」——这就是 ref 中转 / useEffectEvent

**6. ref 中转的原理（为什么能绕过矛盾）**
- 思路：把 onDone 存进 ref（盒子），effect 依赖盒子（地址不变）而非 onDone
- 盒子地址永远不变 → 不触发 effect 重跑 ✅；盒子里内容可随时换 → 读到最新值 ✅
- 关键：**ref 的全部价值就是「变了但 React 不知道」**——想被监测用 state，不想被监测才用 ref

**7. ref 中转的缺陷 + useEffectEvent 怎么修**
- ref 中转的更新发生在 useEffect（绘制后），但 useLayoutEffect（绘制前）可能抢先读 → 旧值窗口
- 这是**顺序问题**（读早于写），不是并发问题 → **用锁解决不了**（单线程没冲突，且锁无法改顺序）
- useEffectEvent 的解法：把「更新盒子」从绘制后提前到 render 阶段 → 消除旧值窗口
- **核心：思路一样（用盒子绕过监测），只是更新时机更早、更安全**

**8. 为什么 render 阶段不能直接更新 ref**
- React 铁律：**render 必须纯净（只算不写）**，因为 render 会被随时重跑（StrictMode 故意跑两遍）
- 在 render 里干杂活（更新 ref、发请求、写文件）→ 重跑时副作用重复执行 → 灾难
- useEffectEvent 是 React 官方的「特许通行证」——它内部在 render 阶段碰了 ref，但由 React 兜底保证安全；自己手写就不行

**9. useLayoutEffect 什么时候用**
- 默认用 useEffect（99% 场景）
- 只有当「用户能看到错误中间状态（闪烁/跳动）且有害」时才用 useLayoutEffect
- 典型场景：测量 DOM 位置 + 动态定位（弹窗/tooltip）、滚动同步（聊天列表）、布局防抖（SEO 的 CLS）、防敏感信息闪现（安全）
- **关键认知**：位置不是内存里存好的数据，是浏览器临时算的（layout 引擎）；React 只管「画什么」，不管「元素落在屏幕哪」——所以需要量位置必须用浏览器 API，而量+改必须在绘制前同步完成

**10. 惰性初始化 vs 位置计算（容易混的两个「只算一次」）**
| | 惰性初始化 | 位置计算 |
|---|---|---|
| 算什么 | 不变的初始数据（读 localStorage） | 随环境变的位置（滚动/缩放都变） |
| 算几次 | 只算一次（省性能） | 每次需要时重算（保正确） |
| 用什么 | useState(() => fn) | useLayoutEffect + getBoundingClientRect |
- **判断准则**：「这数据第一次算完之后还会变吗？」不会→惰性初始化；会→每次实时算

### 工程流程上的两个改进

**1. 自审流程（加进 AGENTS.md，做作业自动触发）**
- 做完代码后切换到「严格 code reviewer」视角复审，重点查：性能浪费、React 常见坑、状态分支全覆盖、可读性、要求符合度
- 发现问题先改再交付，不让小问题留到被 review 挑出来
- 配置在全局 `~/.zcode/AGENTS.md`（所有项目生效）+ 项目级 `C:\react-hooks\AGENTS.md`（历史教训表）

**2. 交付前逐条对照 README 验收**
- 评审③暴露的漏洞：要求明文写了「handle error」，但只做了主路径（顶部 status bar），漏了次路径（hint 区）
- 教训固化：交付前逐条对照 README，每个要求在页面上实际验证一遍，不只看主路径

### 涉及的提交和操作
- [x] 修复 3 处代码（commit `62e373e`，推送 yuyao-homework）
- [x] 在 GitLab 逐条回复老师（说明每条怎么改的）+ resolve 所有讨论
- [x] 老师的 review 全部处理完毕，MR !1 等老师二次确认

### 认知里程碑
- 从「能跑就行」到「理解每行代码的设计权衡」：useEffectEvent 不只是 API 记忆，而是「为什么需要它」的整条因果链（onDone 身份变→不能进依赖→闭包陷阱→ref 中转→异步旧值→useEffectEvent）
- 跨学科思维迁移：用「锁/死锁」分析 React 时序问题，虽然结论是「单线程不需要锁」，但思考方向是对的——前端问题也能用操作系统思维分析
- 学会区分两种失误的严重程度：「代码不够优雅」（技术问题，轻）vs「要求写了却没全覆盖」（流程问题，重）

