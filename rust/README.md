# 🦞 Claw Code — Rust 实现

Claw Code CLI agent harness 的高性能 Rust 重写。为速度、安全性和原生工具执行而构建。

任务导向指南（含可复制示例）见 [`../USAGE.md`](../USAGE.md)。

## 快速开始

```bash
# 查看可用命令
cd rust/
cargo run -p rusty-claude-cli -- --help

# 构建工作区
cargo build --workspace

# 运行交互式 REPL
cargo run -p rusty-claude-cli -- --model claude-opus-4-7

# 一次性提示词
cargo run -p rusty-claude-cli -- prompt "explain this codebase"

# 自动化 JSON 输出
cargo run -p rusty-claude-cli -- --output-format json prompt "summarize src/main.rs"
```

## 配置

设置 API 凭证：

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
# 或使用代理
export ANTHROPIC_BASE_URL="https://your-proxy.com"
```

或直接提供 OAuth bearer token：

```bash
export ANTHROPIC_AUTH_TOKEN="anthropic-oauth-or-proxy-bearer-token"
```

## Mock 对等 Harness

工作区现包含一个确定性 Anthropic 兼容 mock 服务和一个清洁环境 CLI harness，用于端到端对等检查。

```bash
cd rust/

# 运行脚本化的清洁环境 harness
./scripts/run_mock_parity_harness.sh

# 或手动启动 mock 服务
cargo run -p mock-anthropic-service -- --bind 127.0.0.1:0
```

Harness 覆盖：

- `streaming_text`、`read_file_roundtrip`、`grep_chunk_assembly`
- `write_file_allowed`、`write_file_denied`
- `multi_tool_turn_roundtrip`、`bash_stdout_roundtrip`
- `bash_permission_prompt_approved`、`bash_permission_prompt_denied`
- `plugin_tool_roundtrip`

主要制品：

- `crates/mock-anthropic-service/` — 可复用的 mock Anthropic 兼容服务
- `crates/rusty-claude-cli/tests/mock_parity_harness.rs` — 清洁环境 CLI harness
- `scripts/run_mock_parity_harness.sh` — 可重现包装器
- `scripts/run_mock_parity_diff.py` — 场景检查清单 + PARITY 映射运行器
- `mock_parity_scenarios.json` — 场景到 PARITY 的清单

## 功能

| 功能 | 状态 |
|------|------|
| Anthropic / OpenAI 兼容提供者流 + 流式传输 | ✅ |
| 通过 `ANTHROPIC_AUTH_TOKEN` 的 Bearer token 认证 | ✅ |
| 交互式 REPL (rustyline) | ✅ |
| 工具系统（bash、read、write、edit、grep、glob） | ✅ |
| Web 工具（search、fetch） | ✅ |
| 子 agent / agent 界面 | ✅ |
| 待办追踪 | ✅ |
| Notebook 编辑 | ✅ |
| CLAUDE.md / CLAW.md / AGENTS.md 项目记忆 | ✅ |
| 配置文件层次结构（`.claw.json` + 合并配置段） | ✅ |
| 权限系统 | ✅ |
| MCP 服务器生命周期 + 检查 | ✅ |
| 会话持久化 + 恢复 | ✅ |
| 成本 / 用量 / 统计界面 | ✅ |
| Git 集成 | ✅ |
| Markdown 终端渲染 (ANSI) | ✅ |
| 模型别名 (opus/sonnet/haiku) | ✅ |
| 直接 CLI 子命令（`status`、`sandbox`、`agents`、`mcp`、`skills`、`doctor`） | ✅ |
| 斜杠命令（含 `/skills`、`/agents`、`/mcp`、`/doctor`、`/plugin`、`/subagent`） | ✅ |
| 钩子（`/hooks`、配置支持的生命周期钩子） | ✅ |
| 插件管理界面 | ✅ |
| 技能清单 / 安装 / 卸载界面 | ✅ |
| 核心 CLI 界面的机器可读 JSON 输出 | ✅ |

## 模型别名

| 别名 | 解析为 |
|------|--------|
| `opus` | `claude-opus-4-7` |
| `sonnet` | `claude-sonnet-4-6` |
| `haiku` | `claude-haiku-4-5-20251213` |

## CLI 标志和命令

```text
claw [OPTIONS] [COMMAND]

标志：
  --model MODEL
  --output-format text|json
  --permission-mode MODE
  --cwd PATH, -C PATH, --directory PATH
  --dangerously-skip-permissions, --skip-permissions
  --allowedTools TOOLS
  --resume [SESSION.jsonl|session-id|latest]
  --version, -V

顶层命令：
  prompt <text>
  help, version, status, sandbox
  acp [serve]
  agents, mcp, skills
  system-prompt, init
```

## 工作区布局

```text
rust/
├── Cargo.toml              # 工作区根
├── Cargo.lock
└── crates/
    ├── api/                # 提供者客户端 + 流式传输 + 请求预检
    ├── commands/           # 共享斜杠命令注册表 + 帮助渲染
    ├── compat-harness/     # 兼容性/对等 harness 工具
    ├── mock-anthropic-service/ # 确定性本地 Anthropic 兼容 mock
    ├── plugins/            # 插件元数据、管理器、安装/启用/禁用界面
    ├── runtime/            # 会话、配置、权限、MCP、提示词、认证/运行时循环
    ├── rusty-claude-cli/   # 主 CLI 二进制文件 (`claw`)
    ├── telemetry/          # 会话追踪和用量遥测类型
    └── tools/              # 内置工具、技能解析、工具搜索、agent 运行时界面
```

### Crate 职责

- **api** — 提供者客户端、SSE 流式传输、请求/响应类型、认证
- **commands** — 斜杠命令定义、解析、帮助文本生成、JSON/文本命令渲染
- **compat-harness** — 兼容性和对等辅助工具
- **mock-anthropic-service** — 确定性 mock 用于 CLI 对等测试
- **plugins** — 插件元数据和生命周期管理
- **runtime** — 会话持久化、配置加载、权限策略、MCP 客户端、系统提示词组装
- **rusty-claude-cli** — REPL、一次性提示词、CLI 子命令、流式显示
- **telemetry** — 会话追踪事件和遥测
- **tools** — 工具规格和执行：Bash、文件操作、Web 搜索、Agent、Todo 等

## 统计

- **约 20K 行** Rust
- **9 个 crates** 在工作区
- **二进制名称：** `claw`
- **默认模型：** `claude-opus-4-7`
- **默认权限：** `workspace-write`

## 许可证

见仓库根目录。
