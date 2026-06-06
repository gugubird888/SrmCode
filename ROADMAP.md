# Clawable 编码 Harness 路线图

## 目标

将 claw-code 变成最 **clawable** 的编码 harness：
- 无人类优先的终端假设
- 无脆弱的提示注入时机问题
- 无不透明的会话状态
- 无隐藏的插件或 MCP 故障
- 日常恢复无需人工值守

## 「Clawable」定义

一个 clawable 的 harness 是：
- 启动确定性的
- 状态和故障模式机器可读的
- 无需人类盯着终端即可恢复的
- 分支/测试/worktree 感知的
- 插件/MCP 生命周期感知的
- 事件优先，而非日志优先
- 能够自主执行下一步的

## 当前痛点

1. **会话启动脆弱** — 信任提示可能阻塞 TUI 启动，提示可能落入 shell 而非 agent
2. **真相分散在各层** — tmux 状态、clawhip 事件流、git/worktree 状态、测试状态、运行时状态
3. **事件过于日志形态** — claws 当前从嘈杂文本中推断太多
4. **恢复循环过于手动** — 重启 worker、接受信任提示、重新注入提示等
5. **分支新鲜度执行不足** — 侧分支可能遗漏已落地的 main 修复
6. **插件/MCP 故障分类不足** — 启动/握手/配置/部分启动/降级模式未干净暴露
7. **人类 UX 泄露到 claw 工作流** — 太多依赖终端/TUI 行为

## 产品原则

1. **状态机优先** — 每个 worker 有明确的生命周期状态 ✅
2. **事件优于文字抓取** — 频道输出应从类型化事件派生
3. **恢复先于升级** — 已知故障模式应在求助前自动修复一次
4. **分支新鲜度先于指责** — 在将红色测试视为新回归之前检测陈旧分支 ✅
5. **部分成功是一等公民** — MCP 启动可以部分成功，含结构化降级报告
6. **终端是传输层，不是真相** — tmux/TUI 是实现细节，编排状态必须在其之上 ✅
7. **策略可执行** — 合并、重试、变基、陈旧清理、升级规则应由机器执行 ✅

## Phase 1 — 可靠的 Worker 启动

### 1. 编码 Worker 的就绪握手生命周期
添加显式状态：`spawning` → `trust_required` → `ready_for_prompt` → `prompt_accepted` → `running` → `blocked` → `finished` → `failed`

### 1.5. 首个提示接受 SLA
暴露首个任务是否在限定窗口内被接受，发出类型化信号：`prompt.sent`、`prompt.accepted`、`prompt.acceptance_delayed`、`prompt.acceptance_timeout`

### 1.6. 启动无证据 evidence bundle + 分类器
当启动超时时，发出类型化的 `worker.startup_no_evidence` 事件，附带 evidence bundle，将模糊状态降级为具体分类。

### 2. 信任提示解析器
为已知仓库/worktree 添加允许列表自动信任行为。

### 3. 结构化会话控制 API
提供机器控制：create worker、await ready、send task、fetch state、restart worker、terminate worker。

### 3.5. 启动预检 / Doctor 契约
在生成 worker 前运行机器可读的预检，检查：仓库存在性、分支新鲜度、信任门可能性、所需二进制文件、插件发现、MCP 配置等。

## Phase 2 — 事件原生 Clawhip 集成

### 4. 规范 Lane 事件 Schema
定义类型化事件：`lane.started`、`lane.ready`、`lane.blocked`、`lane.red`、`lane.green`、`lane.finished`、`lane.failed`、`branch.stale_against_main` 等。

### 4.5–4.44 事件/报告/审批体系（Phase 2 详细条目）
涵盖：会话事件排序、事件溯源标记、终端事件去重、Lane 所有权绑定、Nudge 确认合同、路线图 ID 分配、路线图项生命周期状态、多消息报告原子性、跨 Claw 去重、证据附件、优先级/严重性合同、实施交接合同、报告反压、无变更确认、观察新鲜度、事实/假设/置信度标签、负面证据合同、字段级变化归因、Schema 版本化、消费者能力协商、自描述 Schema、受众特定投影、规范报告身份、投影失效缓存、编辑溯源、确定性投影、Golden Fixture 回归锁、消费者一致性测试、临时状态去重、升级超时、策略阻止行动交接、策略例外/所有者审批 Token、Token 重放保护、委托/执行链追溯、Token 优化/仓库范围指南、工作区重量预览、更安全范围快速应用、Ship 溯源不透明性、类型化错误信封合同、故障分类法、传输中断 vs Lane 故障边界、可操作摘要压缩 等 40+ 个子条款。

## Phase 3 — 分支/测试感知和自动恢复

### 7. 广泛验证前的陈旧分支检测
在广泛测试运行前比较当前分支与 `main`，发出 `branch.stale_against_main`。

### 8. 常见故障的恢复配方
编码已知自动恢复：信任提示未解决、提示投递到 shell、陈旧分支、跨 crate 重构编译红色、MCP 握手失败、部分插件启动。

### 8.5. 恢复尝试账本
暴露机器可读的恢复进度：配方 id、尝试计数、当前状态（`queued`/`running`/`succeeded`/`failed`/`exhausted`）、时间戳、最后失败摘要、升级原因。

### 9. 绿度合同
Worker 应区分：定向测试绿色、包绿色、工作区绿色、合并就绪绿色。挂起的测试必须报告为 `test.hung` 而非通用故障。

## Phase 4 — Claws 优先的任务执行

### 10. 类型化任务包格式
结构化任务包字段：objective、scope、repo/worktree、branch policy、acceptance tests、commit policy、reporting contract、escalation policy。

### 11. 自主编码的策略引擎
编码自动化规则：如果绿色+范围 diff+审查通过 → 合并到 dev；如果陈旧分支 → 合并前向；如果启动阻塞 → 恢复一次后升级。

### 12. Claw 原生仪表盘 / Lane Board
暴露机器可读的面板：仓库、活跃 claws、worktrees、分支新鲜度、红/绿状态、当前阻塞、合并就绪。

### 12.5. 运行状态活跃心跳
当 lane 标记为工作中时发出轻量级心跳：当前阶段/子阶段、距上次有意义的进度秒数、距上次心跳秒数、当前活跃步骤标签。

## Phase 5 — 插件和 MCP 生命周期成熟度

### 13. 一等插件/MCP 生命周期合同
每个插件/MCP 集成应暴露：配置验证合同、启动健康检查、发现结果、降级模式行为、关闭/清理合同。

### 14. MCP 端到端生命周期对等
关闭差距：配置加载、服务器注册、生成/连接、初始化握手、工具/资源发现、调用路径、错误暴露、关闭/清理。

---

## 立即积压（来自当前实际痛点）

优先级：P0 = 阻塞 CI/绿色状态，P1 = 阻塞集成接线，P2 = clawability 硬化，P3 = 集群效率改进

**P0 — 先修复（CI 可靠性）** ✅ 已完成
- 隔离 render_diff_report 测试 ✅
- 扩展 GitHub CI 到工作区级验证 ✅
- 添加发布级二进制工作流 ✅
- 添加容器优先测试/运行文档 ✅
- 在入门文档中暴露 doctor/预检诊断 ✅
- 自动化品牌/真相源残留检查 ✅
- 消除首次运行帮助/构建路径的警告垃圾 ✅
- 将 doctor 提升为顶层 CLI 入口点 ✅
- 使机器可读状态命令真正机器可读 ✅
- 统一旧配置/技能命名空间 ✅
- 在清单命令上支持 JSON 输出 ✅
- 审计整个 CLI 的 `--output-format` 合同 ✅

**P1 — 接线修复（跨组件）** ✅ 已完成
- 将 Agent/AgentShell 从枚举类型改为 trait 对象 ✅
- MCP 工具桥：会话级作用域和清理 ✅
- 结构化输出和确定性状态传播 ✅
- 工作区测试预检门（阻止陈旧分支上的广泛测试）✅
- 分类回调将挂起的命令标记为 `test.hung` ✅
- 恢复账本序列化 ✅
- 类型化任务包 ✅
- 策略引擎：可执行的 retry/rebase/escalate/stale-cleanup 决策 ✅
- Lane Board / 状态 JSON ✅
- 会话生命周期心跳 ✅
- 实时 Worker 注册表 ✅
- 插件/MCP 验证命令和 JSON 输出 ✅

**P2 — 文档/UX 硬化** ✅ 已完成
- USAGE / 文档化产品表面 ✅
- 容器优先文档 ✅
- 本地模型兼容性文档 ✅
- 导航/文件上下文文档 ✅
- Windows 文档和发布工件 ✅
- 贡献者/安全/支持文档 ✅
- ROADMAP.md：删除模糊描述、旧猜测和已过时的预测 ✅
- `claw doctor` JSON 包含 memory + MCP + hook 验证 ✅
- 恢复就绪会话命令（`/session list/delete`、恢复安全 JSON）✅
- 错误分类 v1（类型化错误种类、操作、目标、重试标志）✅

**P3 — 集群/自动化** ✅ 已完成
- PR/Issue 摄入分类 ✅
- 自动化发布就绪检查 ✅
- 文档真相源 Guard ✅
- CC2 Panel 自动化 ✅
- 发布二进制 SHA256 校验和 ✅
- 文档引用检查 ✅
- 预推送 gate ✅

---

## 部署架构差距

将 `claw` 作为长期进程（会话服务器）、具有完整事件合同和自动推送的工作流暴露为可发现的命令。

## 启动摩擦差距：设置中没有默认 trusted_roots

在没有信任根的仓库中生成 Worker 会发出模糊的「启动无证据」信号，而不是可操作的「将此仓库加入白名单」诊断。

## 可观测性传输决策

通过 HTTP/WebSocket 或命名管道将事件原生化到 Discord，而不是要求 clawhip 抓取 tmux。

## 提供者路由：模型名称前缀必须优先于环境变量存在

当多个提供者密钥存在时，显式模型前缀（如 `openai/gpt-4.1-mini`）选择正确的后端。

---

## Pinpoint 条目（#122 – #160, #246, #693-#695）

以下是已归档的具体缺陷和改进项：

| # | 标题 | 状态 |
|---|------|------|
| 122 | `doctor` 不检查 stale-base 条件 | 待修复 |
| 135 | `status --json` 缺少 `active_session` 布尔值和 `session.id` 交叉引用 | 待修复 |
| 134 | 会话边界缺少 run/correlation ID | 待修复 |
| 136 | `--compact` 标志输出不是机器可读的 | 待修复 |
| 138 | 狗粮周期报告门不透明 | 待修复 |
| 139 | `claw state` 错误消息引用不可发现的 "worker" 概念 | 待修复 |
| 141 | `claw <subcommand> --help` 有 5 种不同行为 | 待修复 |
| 142 | `claw init --output-format json` 将人类文本转储到 `message` | 待修复 |
| 143 | `status` 在畸形 MCP 配置上硬失败；`doctor` 优雅降级 | 待修复 |
| 144 | `claw mcp` 在畸形 MCP 配置上硬失败 | 待修复 |
| 145 | `claw plugins` 子命令未接线到 CLI 解析器 | 待修复 |
| 146 | `config` 和 `diff` 需要 `--resume SESSION.jsonl` 包装 | 待修复 |
| 147 | `claw ""` 静默回退到提示执行路径 | 待修复 |
| 148 | `status` JSON 显示已解析模型但不显示原始输入或来源 | 待修复 |
| 149 | 配置测试在并行工作区运行下 flaky | 待修复 |
| 150 | `resume_latest` flaky 由于符号链接/规范化不匹配 | 待修复 |
| 151 | `workspace_fingerprint` 路径等价合同差距 | 待修复 |
| 152 | 诊断动词后缀允许任意位置参数 | 待修复 |
| 153 | README/USAGE 缺少"添加二进制到 PATH"和"验证安装" | 待修复 |
| 154 | 模型语法错误不提示环境变量 | 待修复 |
| 155 | USAGE 缺少 `/ultraplan`、`/teleport`、`/bughunter` 文档 | 待修复 |
| 156 | 文本模式输出的错误分类（#77 Phase 2） | 待修复 |
| 157 | 错误提示的结构化修复注册表（#77 Phase 3） | 待修复 |
| 158 | `compact_messages_if_needed` 静默丢弃轮次 | 待修复 |
| 159 | `run_turn_loop` 硬编码空 denied_tools | 待修复 |
| 160 | `session_store` 没有 list/delete/exists | 待修复 |
| 246 | 提醒 cron 结果歧义 — 无结构化反馈 | 待修复 |
| 693 | `claw-analog` 类型化阶段错误 | ✅ 已完成 |
| 694 | 预推送 `cargo build` gate | ✅ 已完成 |
| 695 | Agent 在错误 worktree 中启动，浪费一轮 | 待修复 |
| 140 | 弃用的 `permissionMode` 迁移静默降级 `DangerFullAccess` 为 `WorkspaceWrite` | 待修复 |
| 137 | 测试套件中的模型别名简写回归 | 待修复 |
| 133 | Blocked 状态子阶段合同 | 待修复 |
