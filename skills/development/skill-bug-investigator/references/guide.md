# Bug Investigator 调查指南

> Bug Investigator `v0.1.0` 工程故障调查参考手册

本文档是 Bug Investigator 的扩展参考资料。

`SKILL.md` 定义技能的核心行为、触发条件、能力边界和约束；本文档提供更详细的工程调查方法、故障模式、证据模型与诊断策略。

本文档的目标不是为每一种 Bug 提供固定答案，而是帮助 Agent 在面对不同软件系统时建立稳定的调查思路。

---

# 1. 故障调查的基本模型

软件故障调查的核心不是寻找“一个看起来可能的原因”，而是建立能够解释实际现象的因果链。

一个完整的故障通常可以拆分为：

```text
系统状态
   ↓
触发条件
   ↓
根因暴露
   ↓
直接故障
   ↓
次生故障
   ↓
用户可见症状
```

例如：

```text
8766 已被其他进程监听
        ↓
Backend 尝试绑定 8766
        ↓
bind() 失败
        ↓
出现 EADDRINUSE
        ↓
Backend 退出
        ↓
Gateway 无法连接 Backend
        ↓
UI 显示“服务不可用”
```

在这个案例中：

| 类型 | 内容 |
|---|---|
| 根因 | 目标端口已经被其他进程占用 |
| 触发条件 | Backend 启动并尝试绑定该端口 |
| 直接故障 | 端口绑定失败 |
| 次生故障 | Backend 退出、Gateway 连接失败 |
| 症状 | UI 显示服务不可用 |

调查时应尽量沿着这条链向上追踪，而不是停留在用户最先看到的症状。

---

# 2. 五个需要区分的概念

## 2.1 症状

症状是用户或系统能够观察到的异常表现。

例如：

- 页面打不开；
- API 返回 500；
- 应用启动失败；
- UI 卡住；
- 登录失败；
- 数据没有显示；
- 进程异常退出。

症状用于确定调查入口，但通常不能直接作为根因。

---

## 2.2 触发条件

触发条件是使潜在问题真正暴露出来的事件或状态。

例如：

- 第二个服务启动；
- 用户点击登录；
- 某个请求携带特定参数；
- 网络断开；
- Token 过期；
- 数据量超过某个阈值。

触发条件可能不是系统设计缺陷本身。

---

## 2.3 根因

根因是能够解释故障为什么发生的核心原因。

一个合格的根因应该能够解释：

1. 为什么出现当前症状；
2. 为什么在当前条件下发生；
3. 为什么相关组件产生了观察到的行为。

---

## 2.4 影响因素

影响因素不是根因，但可能：

- 增加故障发生概率；
- 扩大故障影响；
- 增加调查难度；
- 使系统更难恢复。

例如：

> 日志没有记录实际加载的端口。

这可能不是端口冲突的根因，但会显著增加调查难度。

---

## 2.5 次生故障

一个故障可能继续触发其他故障。

例如：

```text
Database 无法连接
        ↓
Backend 初始化失败
        ↓
Backend 退出
        ↓
Gateway 请求失败
        ↓
Frontend 显示错误
```

调查过程中必须避免把后面的次生故障误认为最初根因。

---

# 3. 调查的核心循环

推荐使用以下循环：

```text
观察
 ↓
收集证据
 ↓
提出假设
 ↓
预测结果
 ↓
执行验证
 ↓
比较结果
 ↓
保留 / 降级 / 排除假设
 ↓
继续调查
```

这个过程可以重复多轮。

关键原则：

> 每一轮调查都应该减少不确定性。

如果一个操作不能：

- 提供新的证据；
- 验证某个假设；
- 排除某个候选原因；
- 缩小故障范围；

那么它可能不是当前最值得执行的调查动作。

---

# 4. 从“现象”开始，而不是从“答案”开始

面对：

```text
Backend 启动失败
```

不要立即假设：

```text
端口冲突
```

应该先建立基础事实：

```text
Backend 是否真的启动过？
        ↓
启动到了哪个阶段？
        ↓
退出码是什么？
        ↓
最后一个有效日志是什么？
        ↓
是否存在 Stack Trace？
        ↓
失败发生在哪个模块？
```

然后再根据证据提出候选原因。

这样可以降低确认偏误。

---

# 5. 故障边界定位

复杂系统中最有效的调查方法之一，是找到：

> **正确行为从哪里开始变成错误行为。**

例如：

```text
Browser
   ↓
Frontend
   ↓
HTTP Request
   ↓
Gateway
   ↓
Backend
   ↓
Database
```

如果确认：

```text
Frontend → Gateway    正常
Gateway → Backend     正常
Backend → Database    失败
```

那么调查范围就可以优先收缩到：

```text
Backend
Database
Connection Configuration
Network
Authentication
Driver
```

而不是继续分析整个 Frontend。

---

# 6. 二分调查

当调用链较长时，可以采用类似二分查找的策略。

例如：

```text
A → B → C → D → E → F
```

不要一定从 A 开始逐层阅读全部实现。

可以先确认：

```text
C 的输入是否正确？
D 的输出是否正确？
```

如果：

```text
C 正常
D 异常
```

那么问题范围迅速缩小到：

```text
C → D
```

这种方法尤其适合：

- 长调用链；
- 多服务系统；
- 数据处理 Pipeline；
- Agent Runtime；
- IPC；
- API Gateway；
- 消息队列。

---

# 7. 证据模型

不同证据的证明力不同。

调查过程中可以将证据粗略分为四类。

## 7.1 直接证据

能够直接观察到故障发生。

例如：

```text
Error: listen EADDRINUSE: address already in use :::8766
```

并且：

```text
8766 当前已经存在监听进程
```

这种证据具有较高价值。

---

## 7.2 代码证据

源代码能够证明系统实际执行逻辑。

例如：

```text
应用读取 PORT
↓
默认值为 8766
↓
server.listen(port)
```

代码证据能够帮助建立执行路径。

但需要注意：

> 仓库中的代码不一定等于当前 Runtime 实际运行的代码版本。

---

## 7.3 配置证据

配置文件能够证明项目存在某项配置。

但还需要进一步判断：

> 这个配置是否真的被当前 Runtime 加载？

例如：

```text
.env
PORT=8766
```

只能证明文件中写了这个值。

还不能自动证明当前进程实际使用了它。

---

## 7.4 推断证据

根据已有信息形成的工程推断。

例如：

> 这个依赖版本可能与当前 Node.js Runtime 不兼容。

这是合理假设，但在验证之前仍然属于推断。

---

# 8. 证据矩阵

当存在多个候选原因时，可以建立内部证据矩阵。

例如：

| 候选原因 | 支持证据 | 反对证据 | 缺失证据 | 当前判断 |
|---|---|---|---|---|
| 端口冲突 | EADDRINUSE；8766 已监听 | 无 | 进程归属 | 高可能 |
| 配置错误 | 存在多个端口配置 | Runtime 使用值未知 | 实际配置 | 可能 |
| Runtime Bug | 暂无 | 其他端口可正常启动 | Runtime Trace | 低 |

证据矩阵不一定需要完整输出给用户。

它主要用于防止 Agent：

> 找到第一个看起来合理的原因就停止调查。

---

# 9. 假设必须可验证

好的假设应该产生一个可以验证的预测。

例如：

## 假设

Backend 因为 8766 已被占用而无法启动。

## 预测

Backend 启动之前应该已经存在进程监听 8766。

## 验证

只读检查：

```text
Port 8766
→ Listener
→ PID
→ Process
```

## 判断

如果：

```text
端口已占用
+
Backend 配置为 8766
+
启动日志出现 EADDRINUSE
```

那么该假设得到强支持。

---

# 10. 反证优先

只寻找支持假设的证据容易产生确认偏误。

因此对于重要假设，应主动问：

> 什么证据出现时，这个假设就是错的？

例如：

假设：

```text
数据库无法连接是因为密码错误。
```

反证可能是：

```text
使用相同凭据的独立连接测试成功。
```

出现反证后，应降低该假设的置信度，而不是继续强行解释。

---

# 11. 控制变量

如果条件允许，可以通过控制变量验证因果关系。

例如：

```text
条件 A：
PORT=8766
→ 启动失败

条件 B：
PORT=8767
→ 启动成功
```

如果其他条件保持一致，那么：

```text
端口相关问题
```

的置信度会显著提高。

但必须注意：

> 验证操作本身不应产生不可接受的副作用。

---

# 12. 启动失败调查

推荐顺序：

```text
启动命令
   ↓
Launcher
   ↓
Runtime
   ↓
Entry Point
   ↓
配置加载
   ↓
依赖初始化
   ↓
资源初始化
   ↓
端口 / Socket
   ↓
Database
   ↓
External Service
```

重点检查：

- 命令是否正确；
- 工作目录是否正确；
- Runtime 是否正确；
- Entry Point 是否存在；
- 环境变量是否加载；
- 配置是否解析成功；
- 依赖是否完整；
- Native Dependency 是否兼容；
- 端口是否可用；
- 数据库是否可连接；
- 外部服务是否可访问；
- 进程退出码；
- 最后一个有效日志。

优先寻找：

> **最早出现的有效失败点。**

后续大量错误可能只是连锁反应。

---

# 13. API / HTTP 故障调查

推荐链路：

```text
Client
  ↓
Request Construction
  ↓
Network
  ↓
Router
  ↓
Middleware
  ↓
Authentication
  ↓
Handler
  ↓
Service
  ↓
Database / External API
  ↓
Serialization
  ↓
Response
```

需要区分：

```text
请求根本没有发出
```

```text
请求发出但没有到服务器
```

```text
服务器收到请求但路由错误
```

```text
Handler 执行失败
```

```text
下游依赖失败
```

```text
响应正常但 Client 解析错误
```

HTTP 状态码本身通常不是根因。

---

# 14. HTTP 500 调查

不要因为看到：

```text
500 Internal Server Error
```

就只分析 HTTP 层。

优先寻找：

```text
500 对应的原始 Exception
```

然后追踪：

```text
Exception
   ↓
Function
   ↓
Service
   ↓
Dependency / Data / Configuration
```

500 是结果。

真正需要调查的是：

> 哪个内部操作失败导致服务器最终返回 500。

---

# 15. 端口冲突调查

推荐检查：

```text
应用准备绑定哪个端口？
        ↓
端口当前是否被监听？
        ↓
监听者是谁？
        ↓
是否属于当前项目？
        ↓
是否是重复实例？
        ↓
是否是遗留进程？
        ↓
为什么两个服务会使用同一端口？
```

不要默认：

```text
占用端口的进程 = 应该被杀掉的进程
```

它可能是：

- 系统服务；
- 用户其他项目；
- 当前项目正常运行实例；
- IDE；
- Docker；
- 数据库；
- 其他应用。

诊断优先于终止进程。

---

# 16. 配置问题调查

软件项目可能同时存在：

```text
Default Config
.env
.env.local
.env.production
Config File
CLI Arguments
Environment Variables
Runtime Injection
User Settings
Remote Configuration
```

调查配置问题时，需要确定：

```text
有哪些配置来源？
        ↓
优先级是什么？
        ↓
当前环境属于哪一种？
        ↓
Runtime 实际读取了什么？
```

不要因为某个文件存在，就假设它已经生效。

---

# 17. 环境变量调查

例如代码：

```text
PORT = process.env.PORT || 8766
```

需要确认：

```text
process.env.PORT
```

在实际 Runtime 中是什么。

不能只读取：

```text
.env
```

然后直接得出结论。

还应关注：

- Shell；
- IDE；
- Desktop App；
- Docker；
- CI；
- System Service；

是否注入了不同变量。

---

# 18. 依赖故障调查

推荐检查：

```text
Manifest
   ↓
Lock File
   ↓
实际安装版本
   ↓
Runtime
   ↓
Platform
   ↓
Architecture
   ↓
Native Dependencies
```

需要区分：

```text
依赖存在
```

和：

```text
依赖可以在当前 Runtime 正常运行
```

典型问题包括：

- Node.js 版本不兼容；
- Python ABI 不兼容；
- Native Module 架构不匹配；
- Lock 文件失效；
- Peer Dependency 冲突；
- API Breaking Change；
- 系统动态库缺失。

---

# 19. 数据库故障调查

推荐检查：

```text
Application
   ↓
Driver
   ↓
Connection Configuration
   ↓
Network
   ↓
Authentication
   ↓
Database Server
   ↓
Schema
   ↓
Query
```

需要区分：

- 无法连接；
- 认证失败；
- 数据库不存在；
- Schema 不一致；
- Migration 未执行；
- Query 错误；
- Driver 不兼容；
- 数据损坏。

默认不要通过修改真实数据来验证假设。

---

# 20. Migration 问题

如果故障涉及 Migration，检查：

- 当前 Schema；
- Migration History；
- 待执行 Migration；
- Migration 顺序；
- 应用代码预期 Schema；
- 环境之间的差异。

不要直接假设：

```text
Migration 文件存在
```

等于：

```text
生产数据库已经执行。
```

---

# 21. 构建失败调查

构建失败通常可以分类为：

```text
Source Error
Type Error
Dependency Error
Configuration Error
Toolchain Error
Runtime Error
Platform Error
Environment Error
```

如果一次 Build 输出几十个错误：

> 优先寻找最早的关键错误。

例如第一个配置加载失败可能导致后面几十个模块全部报错。

不要把每个次生错误都当成独立根因。

---

# 22. UI 故障调查

推荐链路：

```text
User Action
   ↓
Event Handler
   ↓
State
   ↓
Business Logic
   ↓
Request
   ↓
Response
   ↓
State Update
   ↓
Render
```

典型问题可能位于：

- Event 没有触发；
- State 没有更新；
- 请求没有发送；
- API 返回错误；
- 返回数据格式变化；
- State 更新正确但 Render 条件错误。

不要默认“页面有问题”就一定是前端代码问题。

---

# 23. IPC 故障调查

桌面应用常见：

```text
Renderer
   ↓
IPC Client
   ↓
IPC Transport
   ↓
Main Process
   ↓
Runtime / Service
```

需要检查：

- Channel 名称；
- Payload；
- Handler 注册；
- 生命周期；
- 超时；
- Process 状态；
- 序列化；
- 错误传播。

尤其需要区分：

```text
IPC 调用失败
```

和：

```text
IPC 成功，但下游 Runtime 已经失败
```

---

# 24. Runtime / Worker 故障

对于包含独立 Runtime、Worker 或 Agent Runtime 的系统，可以追踪：

```text
Host
  ↓
Runtime Launcher
  ↓
Worker
  ↓
Tool
  ↓
Subprocess
  ↓
Operating System
```

重点关注：

- 谁创建进程；
- 谁拥有进程；
- 谁负责回收；
- 父子关系；
- Crash 是否传播；
- Worker 退出是否影响 Host；
- Runtime 是否可以恢复；
- Tool 是否越过预期边界。

不要仅凭最终 UI 表现判断是哪一个进程发生了问题。

---

# 25. 日志分析

日志调查不要只搜索：

```text
ERROR
```

还应建立时间顺序。

推荐：

```text
最后一个正常事件
        ↓
第一个异常事件
        ↓
后续错误
```

其中：

> **第一个异常事件**

通常比最后出现的大量错误更有价值。

同时注意：

- Timestamp；
- Process ID；
- Thread；
- Request ID；
- Session ID；
- Trace ID；

是否能够帮助关联事件。

---

# 26. Stack Trace 分析

Stack Trace 应用于寻找：

```text
异常发生位置
调用路径
异常传播路径
```

不要只解释最后一行错误。

重点关注：

- 最初抛出异常的位置；
- 用户代码与第三方库边界；
- 是否被包装；
- 是否被重新抛出；
- Root Exception；
- Cause Chain。

例如：

```text
UI Error
 ↓
API Error
 ↓
Service Error
 ↓
Database Driver Error
 ↓
Connection Refused
```

真正值得继续调查的通常是最底层有效 Cause。

---

# 27. 间歇性故障

如果 Bug 不是每次发生，应优先调查变化条件。

例如：

- Timing；
- Concurrency；
- Network；
- Cache；
- Race Condition；
- Resource Usage；
- Data Shape；
- Process Lifecycle；
- External Service。

建立：

```text
成功场景
vs
失败场景
```

然后寻找差异。

---

# 28. 回归问题

如果用户表示：

> 之前正常，现在坏了。

优先调查变化。

例如：

```text
Git Diff
Dependency Change
Configuration Change
Runtime Upgrade
Deployment Change
Database Migration
Environment Change
External API Change
```

问题出现时间本身就是重要证据。

---

# 29. 最近变更并不等于根因

虽然 Regression 经常和最近修改有关，但不要直接假设：

```text
最近修改 = 根因
```

正确方式是：

```text
变化
 ↓
提出假设
 ↓
验证该变化是否能够解释故障
```

---

# 30. 版本问题

涉及版本时，应区分：

```text
项目声明版本
实际安装版本
实际 Runtime 版本
生产部署版本
文档描述版本
```

它们可能完全不同。

如果只能确认：

```text
package.json 要求 X 版本
```

不能直接写成：

```text
生产环境正在运行 X 版本。
```

---

# 31. 权限问题

权限故障常见于：

- 文件；
- 目录；
- Socket；
- 网络；
- Process；
- Database；
- API；
- Cloud Resource。

调查：

```text
谁在执行？
 ↓
访问什么资源？
 ↓
需要什么权限？
 ↓
实际拥有什么权限？
 ↓
失败发生在哪一步？
```

不要通过直接扩大系统权限来验证问题。

---

# 32. 文件路径问题

重点检查：

- Absolute Path；
- Relative Path；
- Current Working Directory；
- Platform Path Separator；
- Packaging Path；
- Resource Path；
- Temporary Directory。

尤其桌面打包应用中：

```text
开发环境路径
```

和：

```text
安装包运行路径
```

可能完全不同。

---

# 33. 跨平台故障

如果：

```text
macOS 正常
Windows 失败
```

或反过来，应重点检查：

- Path；
- Shell；
- Process API；
- Signals；
- File Permissions；
- Encoding；
- Native Dependencies；
- Packaging；
- Environment Variables；
- Line Endings；
- Platform-specific Code。

不要假设一个平台上的验证可以完全证明另一个平台正常。

---

# 34. 超时问题

Timeout 是结果，不一定是原因。

可能来源：

```text
请求根本没有到达
服务处理过慢
数据库阻塞
外部 API 卡住
Worker 死锁
事件循环阻塞
网络异常
Timeout 配置过短
```

首先确认：

> 时间到底消耗在哪一段。

---

# 35. 性能问题

如果问题是：

> 为什么越来越慢？

先定义可测量指标：

- Response Time；
- CPU；
- Memory；
- I/O；
- Query Time；
- Startup Time；
- Throughput。

避免使用：

```text
感觉很慢
```

作为唯一证据。

---

# 36. 内存问题

调查：

```text
Memory Usage
   ↓
增长趋势
   ↓
生命周期
   ↓
对象 / Cache / Buffer
   ↓
释放行为
```

需要区分：

- 正常缓存；
- 瞬时峰值；
- Memory Leak；
- 大对象；
- Worker 未释放；
- 子进程残留。

---

# 37. 并发与竞态

如果 Bug：

- 偶尔出现；
- 难以复现；
- 与速度有关；
- Debug 模式下消失；

需要考虑：

```text
Race Condition
Ordering
Lock
Shared State
TOCTOU
Async Lifecycle
```

不要因为重新运行一次成功就认为问题已经消失。

---

# 38. 外部服务问题

调用外部服务时，区分：

```text
本地请求构造错误
网络失败
DNS
TLS
Authentication
Rate Limit
Remote Error
Remote Schema Change
Timeout
```

外部服务返回错误并不自动意味着：

> 外部服务本身存在 Bug。

也可能是当前应用请求错误。

---

# 39. 故障复现

稳定复现是高价值证据。

好的复现条件应该尽可能明确：

```text
Environment
Version
Precondition
Input
Action
Expected
Actual
```

例如：

```text
环境：
Windows 11

Runtime：
Node.js 22.x

前置条件：
8766 已被另一个服务监听

操作：
启动 Backend

预期：
Backend 正常启动

实际：
出现 EADDRINUSE 并退出
```

---

# 40. 最小复现

复杂项目中，可以尝试缩小到：

> 能够保留故障的最小条件。

例如：

```text
完整桌面应用
 ↓
Backend
 ↓
Server Startup
 ↓
Port Binding
```

如果只运行 Port Binding 就能够稳定复现，那么大量 UI 和业务逻辑可以暂时排除。

---

# 41. 对照实验

有条件时比较：

```text
Working Case
vs
Failing Case
```

例如：

| 条件 | 正常 | 异常 |
|---|---|---|
| Node | 22 | 20 |
| Config | A | A |
| Code | Same | Same |
| OS | Same | Same |

此时 Runtime Version 就值得优先调查。

---

# 42. 不要过度调查

如果已经存在：

```text
直接错误
+
对应 Runtime 状态
+
代码执行路径
+
可复现验证
```

足以确认根因，就不需要继续扫描整个仓库寻找更多“可能问题”。

Bug Investigator 的目标是：

> 解决当前调查问题。

不是：

> 顺便审计整个项目。

---

# 43. 调查停止条件

满足以下条件之一时可以停止：

## 根因已确认

已有直接证据。

## 高可能原因已建立

当前环境无法完成最后验证，但证据已经高度一致。

## 当前证据耗尽

继续调查需要用户或外部环境提供信息。

## 下一步具有明显副作用

不能安全继续。

## 调查范围开始偏离问题

继续分析不会提高当前根因判断质量。

---

# 44. 未知也是有效结果

例如：

```text
已确认：
应用在连接数据库时退出。

已排除：
端口冲突。
配置文件格式正常。

未知：
数据库服务器当时的状态。

缺失证据：
数据库服务端日志。

结论：
当前无法确认最终根因。

根因状态：未知。
```

这比：

```text
应该是数据库网络不稳定。
```

更加可靠。

---

# 45. 调查结果如何移交修复阶段

一个好的 Bug Investigator 输出应该让后续 Coding Agent 不必重新调查整个问题。

建议移交信息包括：

```text
故障
根因状态
根因
关键证据
相关文件
相关模块
执行路径
修复约束
建议修复方向
潜在回归风险
验证方式
```

例如：

```text
故障：
Backend 无法启动。

根因状态：
已确认。

根因：
8766 已被另一个非当前项目进程占用。

关键证据：
Backend 实际配置为 8766；
启动前 8766 已存在 Listener；
Backend 启动日志出现 EADDRINUSE。

修复约束：
不得终止无关用户进程。

建议方向：
改进启动前端口检查或端口配置策略。

验证：
在端口占用和空闲两种情况下分别执行启动测试。
```

---

# 46. 推荐诊断报告模板

```markdown
# 故障调查报告

## 故障概述

描述需要调查的问题。

## 预期行为

描述正常情况下应该发生什么。

## 实际行为

描述当前观察到的异常。

## 复现条件

如果已知，记录复现条件。

## 关键证据

列出影响结论的重要证据。

## 故障链路

描述从触发到最终症状的因果链。

## 候选原因

### 候选 A

支持证据：

反对证据：

缺失证据：

当前判断：

### 候选 B

支持证据：

反对证据：

缺失证据：

当前判断：

## 根因判断

状态：

- 已确认
- 高可能
- 可能
- 未知

根因：

判断依据：

## 影响因素

如存在则说明。

## 已排除项

记录重要的已排除原因。

## 未知信息

说明当前仍无法确认的信息。

## 建议下一步

给出最小合理行动。

## 修复验证建议

说明后续修复完成后应该如何验证。
```

---

# 47. 快速调查清单

面对新的 Bug 时，可以快速检查：

```text
□ 用户看到的症状是什么？
□ 正常行为是什么？
□ 是否可以复现？
□ 第一个有效错误在哪里？
□ 故障发生在哪个组件？
□ 故障边界在哪里？
□ 有哪些直接证据？
□ 当前有哪些候选原因？
□ 每个候选原因是否可以验证？
□ 是否主动寻找了反证？
□ 是否把症状误认为根因？
□ 是否存在次生故障？
□ 是否存在多个原因？
□ 当前结论属于哪个证据等级？
□ 是否还有关键未知信息？
□ 下一步是否真的能减少不确定性？
□ 调查操作是否具有副作用？
□ 是否已经达到停止条件？
```

---

# 48. Bug Investigator 的最终目标

Bug Investigator 不追求：

> 每一次调查都必须给出一个确定答案。

它追求的是：

> 每一个结论都应该与当前证据强度相匹配。

因此：

```text
证据充分
→ 明确确认

证据较强
→ 高可能

证据有限
→ 保持可能

证据不足
→ 明确未知
```

一个可靠的软件工程 Agent，除了需要知道如何解决问题，也必须知道：

> **什么时候自己还不知道答案。**

Bug Investigator 的价值，就在于让 Agent 在真正修改代码之前，先完成一次有边界、有证据、可验证的工程故障调查。

---

## 参考原则

> **先调查，再修复。**

> **先证据，再结论。**

> **症状不等于根因。**

> **假设必须可以验证。**

> **主动寻找反证。**

> **优先缩小故障范围。**

> **允许结论为未知。**

> **调查过程本身不应该制造新的故障。**