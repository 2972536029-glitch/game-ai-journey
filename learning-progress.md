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

### 第 4 条 review（追加，2026-08-05 下午）—— cancelled 标志：belt and braces

**老师的评审**
> 需要加 cancel，防止任何 state 更新以后的重复更新操作

挂在 LiveSearch.tsx 的 AbortController 那段附近。修复：commit `1477b96`，加了 `let cancelled = false` + cleanup 里 `cancelled = true` + then/catch 开头 `if (cancelled) return`。

**为什么光有 abort 不够（这是这条评审的核心）**
- 我原来的推理（错误的）：用了 abort → 请求取消 → promise reject 成 AbortError → .then 不会跑 → 不需要 cancelled
- 这个推理的隐藏假设：「abort 会立即让 promise reject」
- **实际上 abort 是异步的**：`controller.abort()` 调用之后，promise 不是瞬间 reject 的，要等后续 microtask。在这个时间窗口里，如果请求已经返回、data 已经到了，`.then(data => { setResults(data) })` 会照常执行，显示旧数据——abort 拦不住「已送达的包裹」
- cancelled 标志补的就是这个窗口：它是**同步的**，cleanup 里 `cancelled = true` 瞬间生效，`.then` 开头检查它直接 return，不管 abort 有没有来得及拦

**三个方案的对比（融合了我的旧直觉）**
```
我当初提的方案1（编号拒收）  → 「回来比一比，旧的就拒收」 → 保护「结果回来」那一刻
我当初提的方案2（kill 进程） → 「新请求前杀掉旧的」       → 保护「请求还在路上」
老师要的最终方案             → 方案1的简化（cancelled）+ 方案2（abort）
```
- **我的两个旧直觉，本质就是业界标准方案的两块拼图**：方案1变成 cancelled（拒收），方案2变成 abort（取消）。老师要的 = 我的方案1 + 方案2 合体。
- 我的方案1（版本号）其实比 cancelled 更精细（每个请求知道自己排第几，能处理多请求并发），只是 LiveSearch「一次只有一个活请求」，cancelled 够用，编号是杀鸡用牛刀。

**外卖类比（最好记的版本）**
- abort = 打电话给商家取消订单（拦在路上还没送的）
- cancelled = 门口贴便签「此地址已退租，送来也别留」（拦已送达的）
- 光取消订单不够——骑手可能已经在楼下；贴便签补上这道防线
- **abort 是异步的（打电话要等反应），cancelled 是同步的（贴便签瞬间生效），所以 cancelled 能补 abort 的时序漏洞**

**真正的教训：当时为什么没想到加 cancelled？（比「怎么修」更值得记录）**
- 我当时的心态：「我已经用了 abort，这是 README 要求的标准做法，够了」
- 这是「单一防护就够」的错觉——缺了「万一主防护失效怎么办」的危机感
- 经验丰富的工程师会问「这个防护万一失效怎么办」，我当时只问了「这个防护对不对」
- **工程原则：关键操作不要只靠单一防护，主防护（abort）对的情况下也要设想它失效，加个兜底（cancelled）。** 这叫 belt and braces（腰带 + 背带），裤子不会掉光靠腰带，但加条背带更稳
- 类比：删重要数据 = 软删除（主）+ 备份（兜底）；登录验证 = 密码（主）+ 二次验证（兜底）；异步请求 = abort（主）+ cancelled（兜底）

**复盘：这条暴露的是经验差距，不是知识盲区**
- 我没加 cancelled 不是「不知道有这种写法」，而是「没主动想到要加」——因为我没见过 abort 漏过的真实 case
- 老师补上的是经验，不是知识：见过坑，才会主动防坑
- 固化：以后写任何异步请求，abort + cancelled 是默认搭配，不再只取一个

---

## Session 8 (2026-08-05 ~ 08-06) — Xsolla 第四周：Connecting APIs & Building Dynamic UI（两份作业连做）

### 项目背景
仓库 `connecting-apis-and-building-dynamic-ui`（学校 GitLab）。homework 起始项目是个半成品 Vite + React 应用，调 PokeAPI 抓随机宝可梦，用 `useState + useEffect` 三态模板渲染。两份作业都 build on `src/App.jsx`。

### 两份作业完成情况
- **作业一（宝可梦图鉴）**：完成必做项 + 全部加分项。10 个功能：搜索+自动补全+校验、收藏(localStorage)、上一只/下一只、进化链、主题切换(auto/light/dark)、响应式、搜索框收起、柔和错误气泡。部署到腾讯云 EdgeOne（国内可访问）。
- **作业二（HTTP 请求分析）**：抓 `api.xsolla.com/.../agreements` 的真实 GET 请求，分析 URL/Method/Status/为何无 Query/Payload，Authorization 打码。

### 核心知识点回顾（基于学习过程中的深度追问）

这次最有价值的是一连串"为什么"——从会用到理解 React 设计哲学。

#### 1. 三态模板的 try/catch/finally 怎么分工
- `try`：只成功做的事（setPokemon）
- `catch`：只失败做的事（setError）
- `finally`：两种都要做的（setLoading(false)）
- **核心规律**：finally 不是随便分的第三块，是"不管成败都要做"的语义。把 setLoading 放 try 会漏掉失败的情况，放 catch 会漏掉成功。

#### 2. 三态渲染为什么不能只写 `{pokemon && <卡片/>}`
- 因为 pokemon 是 state，**请求成功后会一直留着旧值**。只判断 pokemon，加载中会显示"上一只的残影"（串台）。
- 必须 `!loading && !error && pokemon` 三条件互斥，保证任何时刻只有一种状态独占屏幕。
- 三态本质是**互斥**的，渲染时要显式排除其他态。

#### 3. useEffect 默认只跑一次的根本原因
- 副作用（fetch/localStorage/addEventListener）需要 DOM 先存在才能安全执行 → 必须在挂载后
- 副作用重复做会出事（死循环/内存泄漏）→ 默认只一次
- `[]` = 初始化用；`[dep]` = 初始化 + dep 变了再跑；不写第二个参数 = 每次渲染都跑（危险）
- 用户的直觉质疑"为什么不能默认不跑、靠依赖触发" → 答案：覆盖不了"一进来就抓数据"的初始化需求

#### 4. fetch 死循环的根因 + useEffect 怎么打破
- 根因：fetch 写在组件函数体里 → 每次渲染都执行 → fetch → setState → 又渲染 → 又 fetch（首尾相接成环）
- 解药：放进 useEffect + `[]`，让它只跑一次，打破"每次渲染都重跑"
- 关键概念纠正："重渲染"≠"effect 重跑"。App 重渲染拿到新 state，但 effect 是否重跑看依赖数组

#### 5. 闭包陷阱（stale closure）
- onKey 函数在 useEffect 里创建，**冻结了创建时的变量值**
- 如果依赖数组不包含用到的变量，监听器会用旧值 → 翻错页/加载中拦不住
- 规则：effect 内部用到的所有外部变量，都要进依赖数组，漏一个就中招
- 我们的方案：用 ref 镜像最新值（`stateRef.current = { loading, pokemon }`），监听器读 ref，依赖写成 `[]` 只注册一次 → 监听器不频繁重注册，又能拿到最新值

#### 6. localStorage 的两个易混概念
- **内存 vs 硬盘**：useState 在内存（刷新就没），localStorage 在硬盘（刷新还在、关浏览器还在）
- **同步但不触发渲染**：`ref.current = x` 和 localStorage 写入都是同步的；但 React 不监听 ref/localStorage 变化，改了不会自动更新界面 → 要更新界面必须 setState
- "不触发渲染"容易被误记成"异步"，但本质不同：异步 = "要等"，ref = "React 不看，永远不更新"。**解药不同：异步靠等，ref 靠换 useState**
- 验证实验：关服务器后页面能打开 → 一度以为是缓存魔法 → 实际是 TaskStop 没杀干净 node 子进程。教训：前端出现"不该有的行为"，先验证环境再怀疑代码

#### 7. 自定义 Hook 不是单例
- 每调一次 useFavorites() 都是独立实例，有自己独立的 state
- "多组件看到相同收藏"靠的是 localStorage 桥梁（各自读写），不是共享实例
- 实验验证：左上角组件自己调 useFavorites，App 也调 → 收藏时左上角不更新（独立），底部 FavoritesBar 接收 prop 则实时更新（共享 App 那份）
- 正确做法：状态提升——只在 App 调一次 Hook，数据通过 prop 往下传

#### 8. useCallback + memo
- useCallback 解决：函数每次渲染都新建，传给子组件时 prop "看起来变了"，让 React.memo 失效
- 三条件缺一无效：函数传给 memo 子组件 + 子组件有 memo + 函数稳定
- 我们项目的 useCallback 大多没用（PokemonCard 没 memo），是代码风格不是性能必需
- memo 有成本（对比 prop），不是越多越好。重组件 + 稳定 prop 才值得用

#### 9. 多级 await + 树遍历（进化链）
- PokeAPI 的进化链要 3 步：pokemon.species.url → species.evolution_chain.url → chain 数据
- 用 while 而不用递归：进化链主要是一条线，while 更简单高效，也更适合（递归适合复杂分叉）
- 已知局限：`evolves_to[0]` 只取第一分支，伊布那种多分支进化只显示一条

### 工程流程上的重大教训：MR 协作规范

这次老师在 MR 反馈里点了 5 条规范，我之前全没做好，属于低级错误。已永久写入 `C:\Users\asus\.zcode\AGENTS.md`。

**老师的要求（每条都对应"review diff 时看什么"）**
1. 不提交 node_modules/.npm_cache/.ai/.claude
2. jsx/tsx 不混用
3. commit/MR title 简短、尽量英文
4. title 用规范前缀：`feat:` 新增 / `fix:` 修 bug / `refactor:` 重构 / `chore:` 杂项
5. 不擅自改既有代码结构（case by case，默认只增不改）

**最核心的教训：提交前必须 review diff**
- "我自己看了"不算数，必须把 diff 完整展示给用户，用户确认后才提交
- diff 会暴露 AI 的 3 个陷阱：偷偷改了没要求的东西、删了不该删的、提交了垃圾文件
- 这次的真实问题：把 `catchOne` 改名成 `fetchOne` 没在注释里说明 why → 补了注释解释每处改动理由
- 类比：diff = 提交前的开箱检查，闭着眼签字可能签下大问题

**修复对照**
| 之前的问题 | 修复后 |
|---|---|
| commit 中文长句 `feat: 完成 Connecting APIs 作业...` | `feat: add pokedex search, favorites, evolution and theme toggle` |
| MR title 同样中文长 | `feat: add pokedex search, favorites, evolution and theme` |
| 改了老师 catchOne 没注释 | 补了 4 处注释解释 why |
| 没 review diff 就提交 | 流程固化：AI 写完 → 展示 diff → 用户确认 → 才提交 |

### 部署上线的折腾（3 个平台的真实坑）

| 平台 | 结果 | 坑 |
|---|---|---|
| Vercel | ❌ | 国内访问超时，被墙 |
| 腾讯云 COS | ❌ | 2024 新政策：默认域名强制加 `Content-Disposition: attachment`，HTML 被当文件下载，必须绑备案域名 |
| EdgeOne Makers | ✅ | 国内可访问，免费永久；但**默认链接带 token，约 3 小时过期**（`eo_time` 时间戳 + cookie Max-Age=10800） |

- EdgeOne 链接过期的本质：token 里编码了签发时间，服务器验证超 3 小时就 401。免费版用短期 token 限制白嫖
- 兜底方案：FEATURES.md 里写了"本地运行"说明，链接失效老师能本地跑

### 认知里程碑
- **从"会用 React"上升到"理解 React 设计哲学"**：搞懂了 useState/useEffect 为什么这么分工、副作用为什么要关进 effect、闭包陷阱的本质
- **"重渲染"和"effect 重跑"是两件事**：这是最容易混淆的点，搞清楚后 useEffect 依赖数组的所有规则都能自洽推导
- **ref 同步但不触发渲染**：这个细节纠正了"ref 异步"的误记，解药是换 useState 而不是"等"
- **AI 代码必须人审**：不看 diff 就提交 = 把责任完全交给 AI，老师点的"低级错误"本质是缺少工程协作意识

### 涉及的提交和操作
- 仓库：`connecting-apis-and-building-dynamic-ui`
- 分支：`yuyao-homework` → `main`
- MR #6：`feat: add pokedex search, favorites, evolution and theme`
- 部署：EdgeOne 项目 `pokemon-homework`（makers-8oztebmh5gq3）
- AGENTS.md 新增"团队协作与提交规范"7 条（永久生效）

---

## Session 9 (2026-08-06) — Xsolla QA 入门实战课 + 课后作业

> 仓库：`school-gitlab.xsolla.dev/bj-xsolla-school/qa-training`（克隆到 `D:\xsolla\qa-training`）
> 课程性质：和前几周的开发课不同，这周是 **QA（质量保证）** 课，重点是"找 bug"的方法论 + 写专业 Bug 单。

### 课程内容（讲义 `slides/学生版.html` 30 页）

| # | 主题 | 核心认知 |
|---|---|---|
| 01 | 为什么需要 QA | 三个真实事故（12306 春运、B 站 2021 崩溃、优惠券被薅千万）的共同点：**功能测过了不代表系统没问题**——崩在容量、边界、职责缝隙 |
| 02 | QA vs Testing | QA = 面向**流程**、**预防**缺陷、贯穿全程；Testing = 面向**产品**、**发现**缺陷、特定阶段。测试是 QA 的一部分 |
| 03 | Bug 的发现与描述 | **严重性**（影响多大，QA 定）≠ **优先级**（多急，产品定）。一份好 Bug 单 = 实际 vs 预期 + 可复现步骤 + 环境 + 截图 + 根因 |
| 04 | 用 AI 提升 QA | AI 列场景/整理 Bug 单/读需求挑歧义；但**4 件事绝不交给 AI**：贴真实数据、判断算不算 Bug、不审就合代码、编测试报告 |
| 05 | 收尾 | 质量是团队的事；AI 是副驾驶，判断和责任始终在人 |

**三个核心思考题的答案**：
1. QA ≠ 测试（范围/目标/阶段都不同）
2. 严重性 vs 优先级是独立的——"很严重但不急修"完全可能（如冷门页面崩溃）
3. 好 Bug 单最关键的：**说清"本该是什么"（预期结果）**，开发才知道改到什么程度

### 课上实战：buggy-shop（游戏充值站）

- 凭直觉找 bug → 拿"测试检查清单"再找一轮，对比**有方法 vs 没方法的覆盖面差距**
- 我从代码挖出 12+ bug，对照清单又挖出 8 个（共 20 个）
- 印象最深的对应关系：
  - **边界值（0/负数）** → 数量输 -3，应付变负
  - **类型也是边界** → 6.60×3 = 19.799999999999997（浮点残留）
  - **文案 vs 行为** → "充值成攻"错别字；说锁定实际没锁
  - **功能之间那条缝** → 摘要显示 total()，支付传 subtotal()，用了券白用

### 课后作业：homework-site（发行商后台，30+ bug）

**任务**：找出最严重的 10 个，按 Bug 单模板写成正式报告提交。

**方法论落地**：老师的"找 bug 8 条提醒"全部命中——
1. 两个页面的数字对不对得上 → 订单数 128 vs 实际 124
2. 翻到最后一页 → 同上
3. 导出文件打开看 → CSV 金额被 Math.round 取整
4. 排序点一下 → 金额按字符串字典序排
5. 筛选边界（同一天）→ 起止同日筛出 0 条
6. 表格混着两种币种 → USD 当 CNY 直接相加
7. 状态流转（取消提现）→ 余额不退回 + 超额仍入库
8. 文案 vs 实际行为 → 手续费规则说"最低¥1"实际算 0；状态选项"退款"不存在；成长率 Infinity%

### 提交的 10 个 Bug（按严重度排序）

| # | Bug | 严重性 | 优先级 |
|---|---|---|---|
| 01 | 概览收入把 USD 当 CNY 相加 | 严重 | P0 |
| 02 | 提现超额仍入库 + 取消不退余额 | 严重 | P0 |
| 03 | 订单数 128 vs 实际 124 | 严重 | P0 |
| 04 | 金额排序按字符串字典序 | 严重 | P1 |
| 05 | 同日筛选结果为空 | 严重 | P1 |
| 06 | 时间固定 UTC（差 8 小时） | 严重 | P1 |
| 07 | 状态筛选「退款」无效 | 一般 | P1 |
| 08 | CSV 金额被四舍五入 | 一般 | P2 |
| 09 | 手续费低于最低 ¥1 | 一般 | P1 |
| 10 | 成长率显示 Infinity% | 轻微 | P2 |

### 工程协作上的新进展

**1. 数据验证不能靠肉眼/想当然**
- 一开始我读代码算出"收入 ¥18970.74"，用户实测打脸——我**漏算了 data.js 里 country/method 字段消耗的随机数**，导致后面所有数据错位
- 教训：数据生成用固定种子 + 多个字段共用一个 rnd()，**漏掉任何一个字段的 rnd() 调用，后续全错位**。必须一字不差照搬源码复算
- 纠正后用 Node 把所有关键数字算准（¥13741.69 / 124 条 / 退款 16 笔等），作为写 Bug 单的铁证

**2. "光看截图说不清"的 Bug 要给证据链**
- Bug 06（时间 UTC）单靠页面截图无法证明——页面上就是一串数字，没标时区
- 解决：在 F12 Console 跑 `new Date(TRANSACTIONS[0].ts).toString()`，输出 `GMT+0800 中国标准时间`，和页面对比形成铁证
- 遇到 Chrome/Edge 安全拦截 `allow pasting`，**这个细节也要写进 Bug 单的复现步骤**（不能只在对话里说）

**3. AI 写的内容用户要逐张核对**
- 我用图像分析核对了全部 18 张截图，发现 1 处我自己的疏漏（Bug 06 文件名引用和实际文件名不一致），当场改 md 引用对齐实际文件

**4. 提交规范的严格遵守**
- 文件名严格按老师要求 `bug-01.md`（不带中文描述，虽然带描述更直观但不符合字面要求）
- 速查表/bug-reports 等辅助文件**不进作业**（只 add 老师规定的 bug-*.md + screenshots/）
- 分支 `HomeWork/yuyao`、commit `feat:` 前缀、MR title `HomeWork: yuyao`，全部对齐规范
- **零改动老师原代码**：28 个暂存文件 100% 在 `homework-submissions/` 下

### GitLab vs GitHub 的坑（新认知）

- `gh` CLI 只管 GitHub，对 GitLab（`school-gitlab.xsolla.dev`）无效
- GitLab 建命令行 MR 需要 `glab` + Personal Access Token，本地没装 glab
- **结论：GitLab MR 最优解是网页点**（一键直达链接 + 内容备好），比装工具+生成 token+给 AI 更快更安全
- token 是账号钥匙，**绝不该交给 AI**——这是 AGENTS.md 合规红线的具体场景

### 涉及的提交和操作
- 仓库：`qa-training`（克隆到 `D:\xsolla\qa-training`）
- 分支：`HomeWork/yuyao`
- commit：`feat: add QA homework - 10 bug reports for publisher dashboard`（28 文件，561 行）
- MR #1：`HomeWork: yuyao`（该仓库第一个 MR）
- 目录结构：`homework-submissions/yuyao/bug-01.md ~ bug-10.md` + `screenshots/`（18 张）

### 知识点沉淀
- **测试检查清单 6 大类**：输入框 8 种 / 按钮 5 种 / 业务规则 / 串起来测 / 换视角 / 最后扫一眼——这套清单以后找 bug 直接套
- **Bug 单判断标准**："别人照着步骤操作，不需要回来问你第二遍"——写完自己按步骤走一遍
- **找 bug 的核心思维**：好奇（输 0 会怎样）+ 破坏欲（想办法搞挂）+ 换位（薅平台的人怎么用）+ 细节控 + 会表达
- **正常路径谁都会走，Bug 都藏在"不该这么用"的地方**——这是这门课的魂

- **当前阶段**：Xsolla QA 入门课完成 + 课后作业已提交 MR
- **待办**：
  - [ ] 等 MR review（老师可能挑刺，按 Session 5/7 补充的经验逐条修复）

---

## Session 6 补充 (2026-08-08) — Data Persistence 作业 Review：8 条评审 + 教科书 vs 工业界的系统性反思

### 背景
第三周的数据持久化作业（MR !16）被老师 s.han review，挑了 8 条。这次 review 的含金量极高——**8 条里有 6 条是同一个主题：教科书写法 vs 工业界做法的分歧**。逐条修复 + 在 MR 回复 + 深度讨论，把每条背后的工程原理吃透了。

**最值得记录的一点**：这次 review 推翻了 Session 6 笔记里我自己之前总结的好几个结论（ENUM 好、外键有用、FOR UPDATE 防超卖）——不是之前的知识错了，而是之前只停留在"教科书层面"，没到"工程层面"。

### 老师的 8 条 review + 修复

**① schema.sql — 原题是三个 session，不是两个**
- 原代码：我把原题简化成了"两个 session 对照"（场景 A/B），还自创了执行顺序
- 老师原话：「再看一下原题，是三个 session，而且即使只说前两个 session，你这个执行顺序也和原题要求的不一样。」
- 修复：按 slide 8 /《MySQL 实战 45 讲》第 8 讲重写为三 session（事务 A/B/C，T1→T10）：A/B 用 `WITH CONSISTENT SNAPSHOT`，C 是普通事务先 update + commit，B 再 update + select，最后 A select。
- **核心知识点（见下方深度讨论 1）**：为什么 A 看到 k=1、B 却看到 k=3？——快照读 vs 当前读。

**② goods_orders.sql — DECIMAL 长度**
- 老师原话：「decimal 每 9 位 4 字节……你这里已经定义了小数点前 8 位，那或者就定义成 9 位，反正占用空间是一样的。不是什么大问题，知道一下就可以了。」
- 修复：`(10,2)` → `(9,2)`。
- **核心知识点（见下方深度讨论 2）**：DECIMAL 的字节是阶跃式增长的，跨过 9 位边界会多 1 字节。

**③ goods_orders.sql — category 别用 VARCHAR，建独立表**
- 老师原话：「category 的可选值是相对固定、稳定的，在每个商品上重复保存这些字符串既浪费空间，性能也不够好。可以像你处理 user 那样，单独建表。」
- 修复：新建 `categories(id, name)` 表，goods 的 `category VARCHAR` → `category_id INT` 引用。
- **核心知识点**：数据规范化（第三范式）——别在多处重复存同一份信息。

**④ goods_orders.sql — state 别用 ENUM，用 TINYINT**
- 老师原话：「为什么用 enum？课上是不是说过状态字段用 tinyint 比较合适？用 enum 的话，新增状态怎么办？修改线上的表结构吗？」
- 修复：`ENUM(...)` → `TINYINT`，Go 用 `type OrderState int8` + 常量映射 + 写入前校验。
- **核心知识点**：这条**直接推翻了 Session 6 笔记第 3 条**（之前我总结"ENUM 省空间有约束，适合状态字段"）。真相见深度讨论 3。

**⑤ goods_orders.sql — 一个订单不该只有一种商品，建关系表**
- 老师原话：「一个订单只能有一种商品吗？应该创建一张关系表，保存 order 和 goods 的多对多关系。」
- 修复：把 goods_id/quantity 从 orders 移出，新建 `order_items(order_id, goods_id, quantity, unit_price)`，`CreateOrder` 改成接收多种商品。
- **核心知识点**：多对多关系必须建关系表；unit_price 存价格快照（**这条 Session 6 笔记第 6 条其实提过**，但当时只停在"知道"，没真正落地到表设计）。

**⑥ goods_orders.sql — 删掉 idx_state_time 索引**
- 老师原话：「state 这种 cardinality 极低的字段，不适合创建索引，即使是多字段索引，也一样。」
- 修复：删掉 `KEY idx_state_time (state, time)`。
- **核心知识点（见下方深度讨论 4）**：索引的价值取决于 cardinality（区分度），低基数列建索引是累赘。

**⑦ goods_orders.sql — 不要定义外键**
- 老师原话：「不要定义外键，对性能和操作，都非常不友好。」
- 修复：删掉两个 `FOREIGN KEY` 约束。
- **核心知识点**：这条**直接推翻了 Session 6 笔记第 5 条**（之前我总结"外键主动拒绝脏数据，有用"）。真相见深度讨论 5。

**⑧ crud/orders_repo.go — 不要用 SELECT ... FOR UPDATE**
- 老师原话：「大多数情况下都不需要 for update，很可能也不允许使用，因为 select for update 容易造成死锁。」
- 修复：去掉 `FOR UPDATE`，改成乐观锁 `UPDATE goods SET stock = stock - ? WHERE id = ? AND stock >= ?`，靠 RowsAffected 判断超卖。
- **核心知识点**：这条**直接推翻了 Session 6 笔记第 8 条**（之前我总结"FOR UPDATE 防超卖是核心手段"）。真相见深度讨论 6。

### 深度讨论的知识点

**1. 快照读 vs 当前读（作业一的灵魂）**
- 困惑："为什么 A 和 B 都用了 `WITH CONSISTENT SNAPSHOT`，A 看到 k=1、B 却看到 k=3？"
- 解答：关键在于**读的方式不同**。A 在 T9 是普通 SELECT（**快照读**），读的是 T1 建的快照，所以是旧值 k=1；B 在 T6 是 UPDATE（**当前读**），当前读不受快照约束，读到 C 已提交的最新值 k=2，+1 = 3。
- **我总结出一条规则**（这条规则让我彻底懂了 REPEATABLE READ）：
  - 我没动过的行 → 从始至终读到的都是快照里那个值（不管别人怎么改）
  - 我自己改过的行 → 在我这个事务内，它就是我改后的值，后续读、后续写都基于这个新值
- **关键认知**：UPDATE/DELETE/`SELECT...FOR UPDATE` 都是当前读，只有普通 SELECT 是快照读。记忆口诀："**写操作 + 加锁的读 = 当前读；普通查 = 快照读**"。
- 一个反直觉的点：B 在 T7 的 SELECT 看到 k=3，是因为**快照读始终能看到本事务内自己做的修改**。所以 B 的快照读看到 3，不是因为快照失效了，而是因为这行是自己改过的。

**2. DECIMAL 存储的字节计算（最实用的细节）**
- 规则：小数点左边的整数部分、右边的小数部分，各自独立按"每 9 位 4 字节"打包，不足 9 位按表递增：
  | 剩余位数 | 字节 |
  |---|---|
  | 1~2 | 1 |
  | 3~4 | 2 |
  | 5~6 | 3 |
  | 7~9 | 4 |
  | 10~18 | 5 |
- 关键：**7、8、9 位都是 4 字节**——这就是老师那句话的本质。
- `(10,2)`：整数 8 位(4字节) + 小数 2 位(1字节) = 5 字节
- `(9,2)`：整数 7 位(4字节) + 小数 2 位(1字节) = 5 字节（一样！）
- 常见误区：很多人以为 `DECIMAL(10,2)` 是"10 位整数 + 2 位小数"。**错！** 是"总共 10 位，其中 2 位小数"，整数部分 = M - D = 8 位。
- **更深一层（我自己追问出来的）**：那为什么不把范围尽量写大？答案——**跨字节边界（9→10 位）字节会阶跃式增长**。10 亿行的流水表，金额从 `(9,2)` 改 `(11,2)` 就多 1GB。而且过大的范围会让脏数据合法混进来（schema 是数据正确性的防线）。

**3. ENUM vs TINYINT：为什么状态字段该用 TINYINT（推翻旧认知）**
- Session 6 笔记里我总结"ENUM 省空间、有约束、查询快，适合状态字段"——**这只对了一半**。
- ENUM 的致命伤：**加新状态要 ALTER TABLE**。线上大表 ALTER 是噩梦（可能锁表几小时）。而状态字段几乎肯定会扩展（以后加"退款中"）。
- TINYINT 的优势：加状态只改应用层常量，表结构纹丝不动。而且 TINYINT 也是 1 字节，ENUM 省不了多少。
- **"TINYINT 不安全，随便一个数字都能存进去"的担心怎么破？**——应用层三道防线：①类型系统限制（自定义 type + 常量）②写入前校验 ③API 层拦截。ENUM 的约束力看似是优点，但工业界更愿意把"状态合法性"放应用层，换取"加状态不改表"的灵活性。
- **哲学**：表结构是"难改"的（线上 ALTER 是噩梦），凡是"可能变化的约束"别焊死在 DDL 里。

**4. 索引与 cardinality（区分度）**
- cardinality = 这个列里"不重复的值有多少种"。state 只有 5 个值 → cardinality 极低。
- 索引的价值取决于"能过滤掉多少数据"。state 查某个值还是要扫 ~20% 数据，索引过滤收益小，反而增加写入时维护索引的开销。
- 联合索引遵循"最左前缀"原则，第一列的 cardinality 决定整体过滤能力。`(state, time)` 第一列 state 极低基数 → 拖垮整个索引。**木桶效应：最短的那块板决定整体。**
- 判断口诀：建索引前问"这列能过滤掉多少数据？" 过滤不到 1% 以下不划算。
- 极端例子：`is_deleted`(0/1) 查 `is_deleted=0` 还是要扫一半数据，建索引是累赘。

**5. 外键：逻辑外键 vs 物理外键（推翻旧认知）**
- Session 6 笔记里我总结"外键主动拒绝脏数据，有用"——**这次被推翻**。
- 关键区分（我复盘后理清的）：
  - **逻辑外键**（user_id、goods_id 这些列，用于 JOIN）→ **必须留**，不然没法关联
  - **物理外键**（FOREIGN KEY 约束，数据库强制检查）→ **工业界几乎不建**
- 物理外键的代价是持续的（每次写入多一次检查 + 锁，运维时清表/删数据/分库都被锁死），收益是偶发的（万一应用层有 bug 兜底）。**用持续的大代价换偶发的小收益，不划算。**
- "应用层 bug 导致脏数据怎么办？"——**修应用层的 bug 就好了**。不能因为"可能在应用层出问题"就给数据库背永久包袱。这是工程权衡。
- **哲学**（这条是我自己总结的，老师没直接说但贯穿始终）：**逻辑关联少不了，但业务约束归应用层，不靠数据库强制。**

**6. 悲观锁 vs 乐观锁 + 死锁的本质（推翻旧认知 + 大开眼界）**
- Session 6 笔记里我总结"FOR UPDATE 给行加写锁，是我们代码里防超卖的核心手段"——**这次被彻底推翻**。
- **死锁是怎么产生的**（这个我之前完全没搞懂）：
  - 死锁 ≠ 库存不够。死锁 = 循环等待。
  - 经典场景：用户 A 买"键盘+鼠标"（先锁键盘再锁鼠标），用户 B 买"鼠标+键盘"（先锁鼠标再锁键盘）→ A 拿键盘等鼠标，B 拿鼠标等键盘 → 互相等，永远等下去 → 死锁。
  - **关键认知**：死锁和"冲突频率高低"无关，只要"多行加锁 + 顺序相反"就可能发生。冲突频率低只是"概率低"，不是"概率为零"。
- **乐观锁为什么不死锁**：每条 UPDATE 只在执行瞬间持锁、执行完立刻释放，从不"持有自己的同时等别人的"，所以不会形成循环等待。死锁的充要条件是"持有-等待"，乐观锁每次只拿一把、拿完就放。
- **乐观锁怎么防超卖**：一条 `UPDATE goods SET stock = stock - ? WHERE id = ? AND stock >= ?`，靠 RowsAffected 判断（0 = 库存不足）。实测：下单 999999 个，条件不成立，RowsAffected=0，事务回滚，库存纹丝不动。
- **乐观锁的缺点**（我追问出来的）：高冲突场景失败率高，需要应用层重试。所以——
- **秒杀场景为什么既不用悲观锁也不用乐观锁**：
  - 悲观锁：单行热点 + 10 万请求串行排队 = 数据库堵死（10万 × 5ms = 500 秒）
  - 乐观锁：10 万人抢 100 个，99000 个 UPDATE 失败要重试
  - 工业界解法：**Redis 在数据库前面预扣库存**——内存操作极快（0.01ms vs 数据库 5ms），单线程天然串行无锁，把 10 万请求筛成 100 条成功的再交给数据库。
- **Redis 为什么快**：①根本原因是**数据在内存**（比硬盘快 10~1000 倍）②单线程无锁开销（内存这么快的前提下，加锁反而成瓶颈）③数据结构简单无 SQL 解析开销。
- **哲学**：数据库不擅长扛"高并发冲突"，所以工业界在数据库前面加缓冲层（Redis/消息队列/限流），把冲突提前消化。这套思想处处可见（秒杀 Redis 预扣、点赞先写 Redis 定时同步、订单消息队列削峰）。

### 工程决策方法论（这次最大的元收获）

这次 8 条 review 表面上是 8 个知识点，底层是同一个东西——**工程决策的本质是"权衡"，没有银弹，只有"匹配场景的方案"**。

**反复出现的模式**：教科书教你"数据库能做什么"，工业界教你"在大规模下该选什么"。
- ENUM：教科书"约束取值" vs 工业界"扩展困难"
- 外键：教科书"保证完整性" vs 工业界"运维噩梦"
- FOR UPDATE：教科书"防超卖" vs 工业界"死锁炸弹"
- 索引：教科书"加速查询" vs 工业界"低基数是累赘"

**一句话总结这次的方法论收获**：
> 数据库只负责两件事——①存数据 ②保证 ACID 事务。剩下的（状态合法性、引用完整性、业务规则）都交给应用层，因为它灵活、可测、易改、不带性能包袱。**逻辑关联（列）少不了；业务约束归应用层，不靠数据库强制。**

### 这次 review 暴露的真正问题（最重要的反思）

这 8 条错，**没有一条是"知识盲区"**——ENUM 的扩展问题、外键的运维代价、FOR UPDATE 的死锁风险，如果我自己想，其实都能想到。那为什么这些错还是出现在了代码里？

**根因：这次代码是 AI 生成的，我没有逐行 review 就提交了。** 问题不在知识缺失，而在流程缺失——**AI 生成 + 人工不 review = 必然带病交付**。

更隐蔽的一层：**AI 写的代码"看起来很专业"**（命名规范、注释齐全、能编译），这种表面专业感会麻痹人，让人觉得"它应该是对的"。但"看起来专业"和"工程最优"是两回事——ENUM 那段代码写得很漂亮，但它就是会在"加状态"时炸。

**应对（已写进 AGENTS.md 全局配置）**：
1. AI 生成的代码，用户必须逐行 review，尤其对字段类型/索引/锁/事务这类工程决策
2. review 时要问"这是教科书写法还是工业界主流？有什么取舍？"
3. 已积累一张"教科书写法 vs 工业界写法"对照表（持续补充）

**老师的角色**：老师不会因为你用了 AI 而扣分，但会因为你"用了 AI 却没 review、交了一堆教科书式的问题"而失望。主动说出来 + 表态会建立 review 习惯，反而把这事变成加分项。**拥抱 AI 而不是逃避，但要把 review 权握在自己手里。**

### 涉及的提交和操作
- 仓库：`data-persistence`（学校 GitLab `bj-xsolla-school/data-persistence`）
- 分支：`yuyao-homework`
- MR：!16「feat: 数据持久化三项作业（全 Go 实现）」
- 代码 commit：`e91ae13 refactor: address review feedback on isolation and shop schema`（9 文件，+669/-528）
- MR 回复：8 条逐条回复 + 1 条汇总总结，全部贴到 GitLab 对应 discussion thread
- 全局 AGENTS.md 更新：新增「工程优先而非教科书写法」「AI 代码必 review」「MR 回复规范」「课程作业总结归档」四节
- 两个 Go module（isolation / crud）均通过 `go vet` + `go build`，并实跑验证（作业一 k=1/k=3、作业三乐观锁防超卖）


---

## Session 10 (2026-08-11) — Xsolla 第五周：Containerization with Docker + CI/CD

> 仓库：`school-gitlab.xsolla.dev/bj-xsolla-school/containerisation-with-docker-and-cicd`（克隆到 `D:\xsolla\containerisation-with-docker-and-cicd`）
> 课程性质：从"写代码"跳到"交付代码"——把一个 Go web 应用**打包成镜像、搭流水线、部署到 K8s**。整条交付链路动手走一遍。
> 分支：`yuyao/homework`，tag `v-yao-yu-20260811`，MR !2

### 项目背景

仓库两部分：
- 根目录：老师演示的 demo（Go web 服务 + Dockerfile + 共享流水线 + K8s 配置）—— **完整参考实现**
- `homework/`：另一个小 Go 应用（监听 9000，`/healthz` 健康检查，运行时读 `index.html` 模板和 `TAG_NAME` 环境变量）—— **要做的作业**

两份任务（各 50 分）：
1. 给 `homework/` 写 Dockerfile，本地 `docker build/run` 通过 `curl /healthz` 验收
2. 在根目录 `.gitlab-ci.yml` 末尾追加 `deploy_homework` job，打 tag 触发部署，首页显示 tag 名

### 知识点

#### Docker / 容器化

- **两阶段构建（multi-stage build）**：builder 阶段（`golang:1.24`，~800MB，带编译器）产出二进制 → COPY 到运行阶段镜像。**核心目的是瘦身 + 隔离构建工具链**（安全只是副产品）。不用两阶段的话，Go 编译器/源码会全被打进最终镜像，体积巨大且全是运行时用不到的垃圾。
- **`CGO_ENABLED=0`**：编纯静态二进制，不依赖 glibc。这是能跑在 `scratch`/`distroless` 这种无 libc 镜像上的前提。不加会动态链接 glibc，scratch 里直接 `not found` 退出。
- **Docker 层缓存优化**：`COPY go.mod` + `RUN go mod download` 单独成层，放在 `COPY . .` 之前。因为 Docker 每条指令一层，**任何层变化会让它及下层全部重建**。把"变化频率低的依赖声明"和"变化频率高的源码"分层，改业务代码时依赖层命中缓存、不用重拉。
- **`EXPOSE` 的真相**：纯文档声明，**不影响端口实际通断**。容器端口通不通完全由 `docker run -p` 决定，`EXPOSE` 只对 `docker run -P`（大写自动映射）和 docker-compose 有参考作用。不写它，`-p 9000:9000` 照样通。—— 这是 Docker 最常见的误解之一。
- **scratch vs distroless**（关键工程取舍，详见下文个人思考）

#### GitLab CI/CD

- **stages / job / pipeline**：流水线按 `stages`（lint→test→build→deploy）顺序跑，每个 stage 里可以有多个 job。job 是最小执行单元，跑在 `image` 指定的容器里。
- **rules 决定 job 何时触发**：`if: '$CI_COMMIT_TAG =~ /正则/'` 让 job 只在 tag 触发且格式匹配时跑。这是"部署由人的动作（打 tag）触发"的实现方式 = **Continuous Delivery**（持续交付，区别于持续部署 CD）。
- **GitLab 预定义变量**：`$CI_COMMIT_TAG`（tag 名）、`$CI_COMMIT_REF_SLUG`（分支或 tag 的 slug，**tag 触发时是 tag 名 slug！**）、`$CI_COMMIT_SHORT_SHA`（commit 短哈希）、`$CI_REGISTRY_IMAGE`（镜像仓库地址）、`$GITLAB_USER_LOGIN`（当前用户名）、`$K8S_NAMESPACE`（项目配的 K8s namespace）。
- **dotenv artifact**：job 运行时把 `KEY=VALUE` 写进文件，声明为 `artifacts.reports.dotenv`，GitLab 会读取并把这些变量**回填给后续 job 和 environment.url**。这是让"运行时才能算出来的值"（如动态 URL）回填到 yaml 静态字段的标准机制。
- **tag-guard**：共享流水线里一个 lint stage 的校验 job，用正则把格式不对的 tag 直接拦红。学生 tag `v-<拼音>-<8位日期>`、老师 tag `v-<数字>`，其他格式报错退出。

#### Kubernetes

- **Deployment + Service**：Deployment 管 Pod 怎么跑（镜像、副本、健康检查、env）；Service 管 Pod 怎么被访问（ClusterIP/LoadBalancer、端口映射）。`homework/deploy/k8s.yaml` 模板里用 `---` 分隔同时声明了这两个资源。
- **`kubectl apply -f` vs `set image`**：apply 是**声明式**（给 yaml，资源不存在就创建、存在就更新），set image 是**命令式**（改已有资源的字段，不存在就 NotFound）。每个学生从零部署自己的 homework，必须用 apply。
- **`kubectl rollout status`**：阻塞轮询，等新 Pod Ready 或超时。Pod Ready 的判定靠 `readinessProbe`（本作业是 `/healthz`）。这是部署的"健康守门员"——镜像拉不下来或应用启动即退，都会让 rollout 超时失败。
- **hw-proxy 反代路由**：一个 nginx（`deploy/hw-proxy.yaml`），把 `http://34.102.5.180/<用户名>/` 按路径转发到 K8s service `hw-<用户名>`。所以资源名必须 `hw-` 前缀、URL 用纯用户名。

### 本次作业的核心工程决策（教科书 vs 工业界）

| 决策点 | 教科书写法 | 我选的写法 | 为什么 |
|:---|:---|:---|:---|
| 运行阶段基础镜像 | `gcr.io/distroless/static-debian12:nonroot`（demo 用的） | `scratch` + `USER 65532:65532` | **网络环境约束**：gcr.io 国内直连不通。scratch 在 docker.io 走 mirror 能拉。65532 是 distroless:nonroot 的标准 UID，等价复刻非 root 效果 |
| 用户名转换 | 假设用户名干净 | `tr '._' '--'` + `hw-` 前缀 | **K8s 资源名只允许小写字母/数字/-（DNS 子域名规范）**，我的 GitLab 用户名 `yao.yu` 有点，必须转。这是 deploy job 的隐藏必选项 |
| rules 正则宽严 | `if: $CI_COMMIT_TAG` 或 `/^v-[a-z]/`（同学 jiefuyang 这么写） | `^v-[a-z][a-z0-9-]*-[0-9]{8}$`（完整匹配） | **双保险**：tag-guard 已挡一遍，rules 再挡一次，防止 `v-数字` 老师 tag 误触发学生部署 |
| 镜像 tag 命名 | 任意 | `$CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA` | **必须跟 build job 对齐**！deploy 拉镜像要和 build 推镜像用完全一样的 tag，否则拉不到。同一 pipeline 里两者看到的变量值一致 |

### Review 反馈 + 复盘

这次作业还没收到老师的 MR review（刚提交），但过程中**自审 + 自我挑刺**收获很大：

1. **tag 命名的乌龙**：第一次打 tag `v-yuyao-20260811`（基于我误以为用户名是 yuyao），部署成功但访问 502。诊断半天才发现 GitLab 用户名是 `yao.yu`（姓在前），`tr` 把点转横杠 → 资源名 `hw-yao-yu`、URL `/yao-yu/`，我给的 `/yuyao/` 是错的。**教训：用 `$GITLAB_USER_LOGIN` 的代码本身没问题（自动适配），错的是我"想当然"地假设了用户名。** 重打 `v-yao-yu-20260811` 跟用户名统一，tag/URL/资源名全一致。

2. **gcr.io 拉不下绕路**：第一次 build 失败 `gcr.io 超时`，挂代理也只解决一半（代理只监听 127.0.0.1，WSL 里的 daemon 通过 host.docker.internal 连不进去 → 拒绝连接）。最终方案：换 scratch + 配 daemon.json 加国内 mirror（`docker.1ms.run` 等）。**这件事的真正教训不是"scratch 更好"——而是"约束下找刚好够用的解"。** scratch 没 CA 证书/时区数据，但这个 app 用不上，所以无功能损失。

3. **build job 的 TLS 偶发失败**：tag 触发的 pipeline 全绿，但分支 push 触发的那条 build job 报 `x509: certificate signed by unknown authority (docker:dind CA)`。这不是我的代码问题（build job 是老师写的共享流水线），是 dind CA 的偶发问题，重跑（重打 tag）就好了。—— 验证了课上那句话"本地能跑 ≠ 别处能跑"，反过来 CI 偶发失败也不一定是代码问题。

4. **自审注释的严格度**：老师要求"每行都要写注释，证明你学会了"。我第一版注释其实够用，但自审发现 3 处偏简略（`stage: deploy`、`environment.name`、rules 正则没拆解），主动补全 + 把"两个差异"扩成"三个差异"对齐讲给用户的内容。**老师在意的就是这种"逐行证明懂"的细节。**

### 个人思考 / 方法论

#### 1. "教科书正确" vs "工程现实约束"——这次最深的体会

demo 用 distroless，是因为 demo 跑在 GitLab CI（Google Cloud）上，能访问 gcr.io。**教科书/课程演示默认环境理想**，但在我的现实约束（中国大陆本地构建）下，distroless 拉不下来。

正确的思考链不是"scratch 更小所以更好"（那是事后找理由，顺序反了），而是：
```
我想用 distroless（老师演示的、更稳妥）
  → 但本地拉不下 gcr.io
    → 要么挂代理（代理监听问题），要么换镜像源
      → 评估 distroless 多出来的功能 app 用不用得上
        → 用不上（不发 HTTPS、不处理时间）
          → 换 scratch，功能无损
```
**真正的工程思维：在约束下找"刚好够用"的解，并诚实承认代价**（scratch 没 CA 证书/时区数据；CI 里其实可以用 distroless，本地妥协不等于 CI 也得妥协）。

#### 2. "自动化验证" 的价值——tag 乌龙的快速定位

如果纯手动部署，tag 写错可能要花几小时排查。但这次靠 deploy job 日志里的两行输出：
```
deployment.apps/hw-yao-yu created      ← 资源名暴露了真实用户名
HW_URL=http://34.102.5.180/yao-yu/     ← URL 跟我假设的不一致
```
**3 秒定位问题**。CI/CD 不只是自动化交付，也是**自动化留下诊断痕迹**。job 日志、rollout status、环境变量回显，都是事后排查的金矿。设计 CI 时要想着"失败了能不能从日志一眼看出原因"。

#### 3. 声明式 vs 命令式的本质差异（apply vs set image）

之前学 Kubernetes 只记了"apply 是建、set image 是改"，这次真正理解了为什么 demo 用 set image、homework 必须用 apply：
- demo 是**单实例、预置好、命令行微调**的部署模型
- homework 是**每人一个、从零创建、模板化批量部署**的部署模型

**两个模型不是语法差异，是部署哲学差异。** apply 配合模板（`homework/deploy/k8s.yaml` + sed 占位符替换）能"一份模板服务全班"，而 set image 做不到。这让我想起数据库的"DDL 焊死 vs 应用层灵活"——K8s 的"set image 焊死单实例 vs apply + 模板支持多实例"是同构问题。

#### 4. "完整交付链路"的全局观

老师说"看懂一个项目的交付链路，先找三个文件：Dockerfile（怎么打包）、.gitlab-ci.yml（怎么测试和发布）、deploy/ 下的 yaml（怎么部署）"。这次三个文件都亲手写/改了一遍：
- Dockerfile → 打包（任务一）
- .gitlab-ci.yml → 流水线（任务二）
- homework/deploy/k8s.yaml → 部署清单（老师给的模板，我 sed 替换）

三块串起来，第一次完整看到"代码 → 镜像 → 流水线 → K8s"的全链路。**比起"会写 Dockerfile"或"会写 CI"，更重要的是理解三块怎么衔接**：镜像 tag 在 build 和 deploy 之间怎么对齐、环境变量名（TAG_NAME vs APP_VERSION）要跟应用代码读的变量一致、访问 URL 怎么被 hw-proxy 路由——**衔接处的细节才是工程问题的多发地带**。

### 涉及的提交和操作

- 仓库：`containerisation-with-docker-and-cicd`（学校 GitLab `bj-xsolla-school/containerisation-with-docker-and-cicd`）
- 分支：`yuyao/homework`
- 两个 commit：
  - `915a57c feat: add homework Dockerfile`（任务一，59 行含注释）
  - `ec26b71 feat: add deploy_homework job`（任务二，60 行含注释）
- tag：`v-yao-yu-20260811`（第一次误打 `v-yuyao-20260811`，发现 GitLab 用户名是 `yao.yu` 后重打）
- Pipeline：#3902（tag 触发，4 job 全绿，用时 1分56秒）
- 部署地址：http://34.102.5.180/yao-yu/（首页显示 tag 名）
- MR：!2「feat: homework — Dockerfile and deploy_homework job」
- 本地环境额外折腾：装 Docker Desktop（下安装包失败重试 5 次）→ 装 WSL2（`wsl --install` 联网失败 → 从 GitHub 下 msixbundle 离线包装）→ 配 daemon.json 国内 mirror（gcr.io 不通绕路 scratch）—— 全程绕了不少网络环境的弯路



## Session 11 (2026-08-12 ~ 08-13) — Xsolla 第六周：Kubernetes Basics（环境搭建 + 首个 K8s 作业 + 概念全图速成）

> 仓库：`school-gitlab.xsolla.dev/bj-xsolla-school/k8s-training`（克隆到 `D:\xsolla\k8s-training`）
> 课程性质：真正进入 K8s——先搞定本地环境（kubectl + kubeconfig），再做 k8s-basics 作业（nginx Pod + 自定义首页），最后把 K8s 全套核心概念用"聊天+类比"方式从零学了一遍。作业 MR !8 已提交待 review。
> 本次特别之处：**作业是"AI 动手、我看着学"模式**——代码由 AI 敲，但事后要求 AI 把每个概念讲透，作业和学习分两步走。

### 项目背景

三段式的一天半：
1. **Setup（08-12）**：装 kubectl v1.34.10（对齐集群 v1.34）、迁移 kubeconfig、配 PATH、连集群验证
2. **作业（08-13 下午）**：k8s-basics 基础 5 条要求 + 2 个加分项（声明式 yaml + ConfigMap 挂载），提交 MR !8
3. **学习（08-13 晚上）**："我还没学呢"——从云的概念到 K8s 全图，全部用类比过一遍，再对照老师 PPT 补 8 个盲区

### 知识点

#### 环境搭建（Setup）的工程要点

- **kubectl 版本对齐**：官方支持窗口是集群版本 ±1 minor。集群 v1.34 → 必须装 v1.33~v1.35（推荐 v1.34.x）。Docker Desktop 自带的 v1.36 超窗，老师课上明确说过要换。用 `stable-1.34.txt` 端点锁定 minor 版本。
- **kubeconfig 三要素**：clusters（总台在哪：server 地址 + CA）、users（我是谁：凭证）、contexts（组合预设：哪个集群+哪个身份+默认 namespace）。文件本质 = 对讲机的通讯录。
- **kubeconfig 的安全隐患排查**：老师的 kubeconfig 是微信发的，最初 KUBECONFIG 环境变量直接指向微信缓存目录（`xwechat_files/...`）——微信清缓存即失效。正确做法：复制到 `~/.kube/config` 标准位置 + 删掉环境变量。
- **PATH 优先级（Windows）**：系统级 PATH 拼在用户级前面，想让新 kubectl 盖过 Docker 的，必须改系统级 PATH 并放在最前。
- **`kubectl auth whoami` + `kubectl get pods` 双验证**：前者验认证（集群认得你），后者验授权（能在 namespace 干活）。学生账号是 namespace 级 serviceaccount，`get nodes` 这类集群级命令被拒是**预期行为**，不是配置坏了。

#### K8s 核心概念（晚上速成，全部用类比打通）

- **云 vs 远程服务器**：远程只说"在哪"（租房），云强调"怎么用"（酒店）：按需开关、按量付费、坏了厂商管、秒级弹性。云三档：裸机（毛坯房）→ 容器平台 GKE（精装房拎包入住）→ Serverless（酒店式公寓）。
- **多地域三理由**：① 就近连（物理延迟：同城 ~5ms、跨国 150-300ms，光速不可商量）② 容灾（OVH 大火案例：数据只存一地=没存）③ 合规（数据不出境法规）。对应信安课的 RTO/RPO、两地三中心、异地多活。
- **镜像/容器/Pod 三套娃**：镜像=预制菜包（只读、可复制、消灭"在我电脑上是好的"）；容器=加热好的活实例；Pod=托盘（豆荚）。**Pod 存在的意义**：① 合租房——多容器共享 IP/存储/生死（sidecar 边车模式）② 通用接口——K8s 不绑死 Docker（USB-C 思想）③ 卷声明的挂载点。
- **数据哲学（本次最值钱）**：镜像永远只读；容器写数据落在"草稿纸"（可写层），容器一删草稿纸撕掉；**想留的数据必须出家门**（卷/ConfigMap/数据库）。无状态（纸杯随便换）vs 有状态（金贵需托管）。原神类比：游戏程序=可重装的容器，账号存档=云端数据库——程序和数据分家是云的第一设计哲学。
- **命令旅程**：kubectl 只是对讲机，每条命令=一条 HTTPS 请求发给 API Server 总台，总台协调 etcd 账本、Scheduler 分房员、kubelet 管家。一切经过总台=K8s 管几千台机器不乱的秘密。
- **Deployment**：长期工单（"保持 3 份 nginx 活着"），核心机制=**调谐循环**（对账→发现偏差→修正）。三超能力：自愈（奶妈复活术）、扩缩容（一条命令）、滚动升级（换厨师不停业）。最少 2 副本（滚动升级需要"换的过程中有人接客"）。
- **Service/Ingress**：Pod IP 随更替变化，Service=固定号码的总机（实时追踪健康 Pod 轮转流量）。三种门：ClusterIP（内线）/NodePort（钉门牌）/LoadBalancer（云上正经正门）。Ingress=按域名/路径分诊的金牌前台。port-forward=调试隧道（依赖本机+VPN+命令不关），生产用 Service。
- **Kustomize/Helm**：前者"一份底稿+便利贴"（base+overlays 防配置漂移），后者"应用商店"（Chart 安装包+values 选项单）。下个 track 的"three ways to ship"路线：kubectl → kustomize → helm。

#### PPT 对照补盲（8 个点）

1. **Labels & Selectors**：K8s 的胶水。Service 不认 Pod 的 IP/名字，只认标签（`app: web`）。下个作业 Deployment+Service 的 yaml 里 `selector` 和 `labels` 必须对上——新人最常见翻车点。
2. **Health Probes**：Liveness（死了→重启）、Readiness（没准备好→暂不转流量，滚动升级无缝的核心）、Startup（慢启动保护）。`1/1` 的分母分子就是就绪探针。
3. **Secret**：ConfigMap 的孪生兄弟，专存密码/token/证书，base64+更严权限。
4. **VM vs Docker vs K8s**：三个抽象层——整台电脑/一个进程/一群进程。
5. **DaemonSet/Job/CronJob**：每机一个（日志agent）/干完即走（迁移备份）/定时任务。
6. **HPA**：机器人看 CPU 自动加减副本，双十一完全体。
7. **三道门安全模型**：Authentication（你是谁）→ Authorization（RBAC：能干啥）→ Admission（改得合规吗）。`get nodes` 被拒=倒在第二道门。
8. **SRE**：把运维当软件工程做，用自动化代替人肉。Xsolla 老师全是 SRE 团队的。

**老师推荐的自学资源（PPT 最后一页）**：在线免费练 K8s —— killercoda.com（浏览器直接玩）；kubectl 速查表 —— k8s.io/docs/reference/kubectl/cheatsheet（下个 track 必用）；深入一本书 ——《Kubernetes in Action》(Marko Luksa)；谷歌免费 SRE 书 —— sre.google/books（真实生产系统怎么跑）；职业认证 —— CNCF: KCNA → CKA/CKAD/CKS。

### 实操踩坑记录（Windows + K8s 专属）

| 坑 | 现象 | 解法 |
|:---|:---|:---|
| **Git Bash MSYS 路径转换** | `kubectl exec ... cat > /usr/share/...` 报 `D:/Git/usr/share/...: No such file`——Git Bash 把容器内 Unix 路径自动改写成 Windows 路径 | `export MSYS_NO_PATHCONV=1` |
| **apply 的 OpenAPI 超时** | `failed to download openapi: context deadline exceeded`——apply 默认先下 OpenAPI schema 做客户端校验，VPN 带宽小扛不住 | `--validate=false`（服务端仍校验）；命令式命令不触发 |
| **官方源单连接限速** | dl.k8s.io 单连接 18KB/s，59MB 要 40 分钟 | aria2c 16 线程 → 3.4MiB/s（**190 倍**），20 秒下完。Google CDN 不限并发，瓶颈是单连接 QoS |
| **HTTPS 克隆中断** | 大仓库克隆 `curl 56 Recv failure: Connection was reset` | 开 VPN 重试成功；备选浅克隆 `--depth=1` |
| **IAB 浏览器自动化 GitLab 富文本编辑器失败** | MR 表单 description 编辑器在 IAB 里渲染不出来，force click 把状态搞乱后 reload 也救不回 | **及时 fallback 手动**——title/分支 GitLab 已自动预填，用户手动贴 description+点 Create。教训：自动化状态搞乱后应更早放弃，别反复 reload 折腾 |

### Review 反馈 + 复盘

**MR !8 已提交，老师 review 还没来（待补充）。** 交付前的自审已经抓到并修掉两个问题：

1. **README 命令记录与实际不一致**：第一版 README 里贴的 HTML 是简化版（缺 `.tag` 底部字样、viewport、字体栈），别人照着跑复现不出截图页面。自审发现后改成与实际执行**逐字一致**的版本。教训：**"命令记录"的价值就在可复现，省几行是反向优化**。
2. **缺 `MSYS_NO_PATHCONV=1`**：实际执行时踩了这个坑，但初版 README 的命令块里没写（只在感想里提）——照抄会失败。补进命令块+注释。和上一条同根源：**记录要如实，不是记"理想版"**。

加分项的一个**主动工程决策**：ConfigMap 挂载的是静态文件，nginx 直接 serve，不经过 shell，所以 `$(hostname)`/`$(cat .../namespace)` 不会展开——加分项 HTML 里 namespace/pod 名是**硬编码**的。权衡：这俩值在本作业固定，硬编码可接受；真要动态需 Downward API + envsubst/nginx sub_filter。**这个"静态 vs 动态"的取舍主动写进了 README 感想**，是加分项最值钱的点。

### 个人思考 / 方法论

#### 1. "先体验再学理论"的学习模式（本次最大特色）

作业做完才发现"我还没学呢"——但反过来，**先跑通了完整流程再学概念，每个概念都有实体可挂靠**：学 Pod 时想起 `my-nginx`，学调谐循环时想起 `kubectl wait`，学数据哲学时想起 ConfigMap 加分项。比纯看 PPT 记得牢得多。（代价：做的时候全靠 AI，自己必须事后补课，不能跳过。）

#### 2. 数据哲学是自己推出来的，不是背的

被问到"原神存档会不会回到镜像里"时，我顺着"镜像只读→容器可写层是草稿纸→草稿纸随容器撕掉"自己推出了"**存数据库（云端服务），不然可写层的数据随服务重启消失**"。这个结论和工业界"无状态计算+外置状态"的架构完全一致——**概念是自己推出来的才算真懂**。

#### 3. 高可用与资源浪费的权衡

"即使浪费一些资源，也要多设副本保证意外不中断"——这就是高可用（HA）的核心思路，再往前一步就是工程权衡：副本数=流量容量+故障余量，不是越多越好（100 副本老板拿账单找你），但最少 2 个（滚动升级需要）。**成本 vs 可靠性的权衡**和数据库课的"教科书 vs 工业界"是同一种思维。

#### 4. AI 动手时代的作业流程

AI 把作业从 2 小时压到 30 分钟，但真正的学习发生在晚上的"诶这个我学过"式追问里。**AI 干活越快，人越要主动补"为什么"层**——否则作业是交了，知识还是 AI 的。（这呼应了 AGENTS.md 里"AI 写的代码必须自己逐行 review"——这次是"AI 做的作业必须自己逐个概念补课"。）

### 涉及的提交和操作

- 仓库：`k8s-training`（学校 GitLab `bj-xsolla-school/k8s-training`，clone 到 `D:\xsolla\k8s-training`）
- 分支：`homework/yao-yu/k8s-basics`
- commit：`7fb73a0 Homework: nginx page by yao-yu`（3 files, +173 行：README.md / pod.yaml / screenshot.png）
- MR：!8「Homework: nginx page by yao-yu」（Open，待 review）
- 本地环境沉淀：`C:\tools\kubectl\kubectl.exe`（v1.34.10，系统级 PATH 首位）；`~/.kube/config`（脱离微信缓存）；用户级 KUBECONFIG 环境变量已删
- 作业页面的那句话：「过去是一个幽灵，虚无缥缈，没什么影响力，只有未来才有分量！」——前半句灰色暗淡、后半句亮色高亮，呼应句意

