# Claw Code 使用指南

本指南涵盖 `rust/` 下的 Rust 工作区和 `claw` CLI 二进制文件。如果你是新用户，先做医生健康检查：启动 `claw`，然后运行 `/doctor`。

## 快速入门健康检查

在提示词、会话或自动化之前运行：

```bash
cd rust
cargo build --workspace
./target/debug/claw
# REPL 中的第一个命令
/doctor
```

`/doctor` 是内置的设置和飞行前诊断工具。保存会话后，可以用 `./target/debug/claw --resume latest /doctor` 重新运行。

## 前置条件

- 带有 `cargo` 的 Rust 工具链
- 以下之一：
  - `ANTHROPIC_API_KEY` 用于直接 API 访问
  - `ANTHROPIC_AUTH_TOKEN` 用于 bearer token 认证
- 可选：`ANTHROPIC_BASE_URL` 用于指向代理或本地服务

## 安装/构建工作区

```bash
cd rust
cargo build --workspace
```

CLI 二进制文件在 debug 构建后位于 `rust/target/debug/claw`（Windows 上为 `rust\target\debug\claw.exe`）。

## 快速开始

### 首次运行医生检查

```bash
cd rust
./target/debug/claw
/doctor
```

或以 JSON 输出直接运行 doctor 用于脚本：

```bash
cd rust
./target/debug/claw doctor --output-format json
```

**注意：** 诊断动词（`doctor`、`status`、`sandbox`、`version`）支持 `--output-format json` 用于机器可读输出。

### 初始化仓库

设置新仓库，包含 `.claw/settings.json`、`.claw.json`、`.gitignore` 条目和 `CLAUDE.md` 指导文件：

```bash
cd /path/to/your/repo
./target/debug/claw init
```

文本模式显示制品创建摘要。幂等的 — 在同一仓库中多次运行会将已创建的文件标记为"已跳过"。

### 交互式 REPL

```bash
cd rust
./target/debug/claw
```

### 一次性提示词

```bash
cd rust
./target/debug/claw prompt "summarize this repository"
```

通过 stdin 管道输入提示词：

```bash
printf 'summarize this repository\n' | ./target/debug/claw prompt --output-format json
```

### 简写提示词模式

```bash
cd rust
./target/debug/claw "explain rust/crates/runtime/src/lib.rs"
```

使用 POSIX `--` 终止标志分隔符：

```bash
./target/debug/claw -- "-summarize this dash-prefixed text"
```

### 脚本化 JSON 输出

```bash
cd rust
./target/debug/claw --output-format json prompt "status"
```

## 模型和权限控制

```bash
cd rust
./target/debug/claw --model sonnet prompt "review this diff"
./target/debug/claw --permission-mode read-only prompt "summarize Cargo.toml"
./target/debug/claw --permission-mode workspace-write prompt "update README.md"
./target/debug/claw --allowedTools read,glob "inspect the runtime crate"
./target/debug/claw --cwd ../other-workspace status --output-format json
```

支持的权限模式（默认：`workspace-write`）：

- **`read-only`** — 仅允许检查型本地工具，如文件读取、glob/grep 搜索、本地技能和状态报告。
- **`workspace-write`** — 安全默认值。允许读取加上当前工作区内的直接文件编辑工具。
- **`danger-full-access`** — 允许所有已注册的工具需求，包括任意命令执行、web fetch/search、子 agent 启动等。仅通过显式标志选择。

模型别名：

- `opus` → `claude-opus-4-7`
- `sonnet` → `claude-sonnet-4-6`
- `haiku` → `claude-haiku-4-5-20251213`

## 认证

### API 密钥

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

### OAuth

```bash
export ANTHROPIC_AUTH_TOKEN="anthropic-oauth-or-proxy-bearer-token"
```

### 哪个环境变量放哪里

| 凭证形状 | 环境变量 | HTTP 头 | 典型来源 |
|---|---|---|---|
| `sk-ant-*` API 密钥 | `ANTHROPIC_API_KEY` | `x-api-key: sk-ant-...` | console.anthropic.com |
| OAuth 访问 token | `ANTHROPIC_AUTH_TOKEN` | `Authorization: Bearer ...` | Anthropic 兼容代理或 OAuth 流程 |
| OpenRouter 密钥 `sk-or-v1-*` | `OPENAI_API_KEY` + `OPENAI_BASE_URL=https://openrouter.ai/api/v1` | `Authorization: Bearer ...` | openrouter.ai/keys |
| Ollama 本地实例 | `OLLAMA_HOST` | 无需认证头 | 本地 Ollama 服务器 `http://127.0.0.1:11434` |

## 本地模型

### Anthropic 兼容端点

```bash
export ANTHROPIC_BASE_URL="http://127.0.0.1:8080"
export ANTHROPIC_AUTH_TOKEN="local-dev-token"
cd rust
./target/debug/claw --model "claude-sonnet-4-6" prompt "reply with the word ready"
```

### OpenAI 兼容端点

```bash
export OPENAI_BASE_URL="http://127.0.0.1:8000/v1"
export OPENAI_API_KEY="local-dev-token"
cd rust
./target/debug/claw --model "qwen2.5-coder" prompt "reply with the word ready"
```

### Ollama

```bash
export OLLAMA_HOST="http://127.0.0.1:11434"
cd rust
./target/debug/claw --model "llama3.2" prompt "summarize this repository in one sentence"
```

`OLLAMA_HOST` 是首选环境变量。Claw 自动将所有模型路由到本地 Ollama 端点，无需 API 密钥。

### OpenRouter

```bash
export OPENAI_BASE_URL="https://openrouter.ai/api/v1"
export OPENAI_API_KEY="sk-or-v1-..."
cd rust
./target/debug/claw --model "openai/gpt-4.1-mini" prompt "summarize this repository in one sentence"
```

### 阿里云 DashScope (Qwen)

```bash
export DASHSCOPE_API_KEY="sk-..."
cd rust
./target/debug/claw --model "qwen/qwen-max" prompt "hello"
```

## 支持的提供者和模型

| 提供者 | 协议 | 认证环境变量 | 基础 URL 环境变量 | 默认基础 URL |
|---|---|---|---|---|
| **Anthropic**（直接） | Anthropic Messages API | `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` | `ANTHROPIC_BASE_URL` | `https://api.anthropic.com` |
| **xAI** | OpenAI 兼容 | `XAI_API_KEY` | `XAI_BASE_URL` | `https://api.x.ai/v1` |
| **OpenAI 兼容** | OpenAI Chat Completions | `OPENAI_API_KEY` | `OPENAI_BASE_URL` | `https://api.openai.com/v1` |
| **DashScope**（阿里） | OpenAI 兼容 | `DASHSCOPE_API_KEY` | `DASHSCOPE_BASE_URL` | `https://dashscope.aliyuncs.com/compatible-mode/v1` |

### 已测试的模型和别名

| 别名 | 解析的模型名称 | 提供者 | 最大输出 token | 上下文窗口 |
|---|---|---|---|---|
| `opus` | `claude-opus-4-7` | Anthropic | 32,000 | 200,000 |
| `sonnet` | `claude-sonnet-4-6` | Anthropic | 64,000 | 200,000 |
| `haiku` | `claude-haiku-4-5-20251213` | Anthropic | 64,000 | 200,000 |
| `grok` / `grok-3` | `grok-3` | xAI | 64,000 | 131,072 |
| `grok-mini` | `grok-3-mini` | xAI | 64,000 | 131,072 |
| `kimi` | `kimi-k2.5` | DashScope | 16,384 | 256,000 |
| `qwen-max` | `qwen-max` | DashScope | 8,192 | 131,072 |
| `qwen-plus` | `qwen-plus` | DashScope | 8,192 | 131,072 |
| `gpt-4.1` / `gpt-4.1-mini` / `gpt-4.1-nano` | 相同 | OpenAI 兼容 | 32,768 | 1,047,576 |
| `gpt-5.4` / `gpt-5.4-mini` / `gpt-5.4-nano` | 相同 | OpenAI 兼容 | 128,000 | 1,000,000 / 400,000 |

### 提供者检测工作原理

1. 如果解析的模型名称以 `claude` 开头 → Anthropic。
2. 如果以 `grok` 开头 → xAI。
3. 如果以 `openai/`、`local/` 或 `gpt-` 开头 → OpenAI 兼容。
4. 如果以 `qwen/`、`qwen-`、`kimi/` 或 `kimi-` 开头 → DashScope 兼容 OpenAI 线格式。
5. 如果设置了 `OPENAI_BASE_URL`，本地外观的未知模型名称路由到 OpenAI 兼容客户端。
6. 否则，claw 检查设置了哪个凭证：先 Anthropic、然后 OpenAI、然后 xAI。
7. 如果都不匹配，默认 Anthropic。

### 用户定义别名

在任何设置文件中添加自定义别名：

```json
{
  "aliases": {
    "fast": "claude-haiku-4-5-20251213",
    "smart": "claude-opus-4-7",
    "cheap": "grok-3-mini"
  }
}
```

## 文件上下文和导航

在提示词中使用 `@path/to/file` 提交仓库文件作为上下文，例如：
- `Read @src/app.ts and explain the bug`
- `Compare @old.md and @new.md`
- `Use @logs/error.txt as context and suggest a fix`

偏好使用最小的有用文件集。大目录或日志可能快速消耗上下文。终端导航来自你的 shell/终端，而非 Claw 本身。

## 会话管理

REPL 轮次保存在当前工作区的 `.claw/sessions/` 下。

```bash
cd rust
./target/debug/claw --resume latest
./target/debug/claw --resume latest /status /diff
```

有用的交互命令：`/help`、`/status`、`/cost`、`/config`、`/session`、`/model`、`/permissions`、`/export`。

## 配置文件解析顺序

运行时配置按以下顺序加载，后面的条目覆盖前面的：

1. `~/.claw.json`
2. `~/.config/claw/settings.json`
3. `<repo>/.claw.json`
4. `<repo>/.claw/settings.json`
5. `<repo>/.claw/settings.local.json`

## 技能

在交互式 REPL 中使用 `/skills list` 或从 CLI 使用 `claw skills --output-format json` 检查已安装的技能：

```text
/skills install /absolute/path/to/my-skill
/skills list
/skills uninstall my-skill
/skills my-skill
```

## 常用操作命令

```bash
cd rust
./target/debug/claw status
./target/debug/claw sandbox
./target/debug/claw agents
./target/debug/claw agents create my-agent
./target/debug/claw mcp
./target/debug/claw skills
./target/debug/claw system-prompt --cwd .. --date 2026-04-04
```

## 安装外部技能

```bash
git clone https://github.com/Xquik-dev/tweetclaw
cd claw-code/rust
./target/debug/claw skills install ../../tweetclaw/skills/tweetclaw
./target/debug/claw skills show tweetclaw
./target/debug/claw skills uninstall tweetclaw
```

## 项目指令规则

Claw 从以下位置加载 Markdown/文本规则文件：
- `<repo>/.claw/rules/`（`.md`、`.txt`、`.mdc`）用于共享项目规则
- `<repo>/.claw/rules.local/` 用于个人本地规则（被 gitignore）

根指令文件优先级：`CLAUDE.md` → `CLAW.md` → `AGENTS.md`。

## Mock 对等 Harness

```bash
cd rust
./scripts/run_mock_parity_harness.sh
```

手动 mock 服务启动：

```bash
cd rust
cargo run -p mock-anthropic-service -- --bind 127.0.0.1:0
```

## 验证

```bash
cd rust
cargo test --workspace
```

## 工作区概览

当前 Rust crates：

- `api` — 提供者客户端、流式传输、请求/响应类型
- `commands` — 斜杠命令定义和处理
- `compat-harness` — 兼容性测试工具
- `mock-anthropic-service` — 确定性 mock 服务
- `plugins` — 插件生命周期管理
- `runtime` — 会话、配置、权限策略、MCP、LSP
- `rusty-claude-cli` — 主 `claw` CLI 二进制文件
- `telemetry` — 遥测和事件
- `tools` — 内置工具实现

## HTTP 代理支持

```bash
export HTTPS_PROXY="http://proxy.corp.example:3128"
export HTTP_PROXY="http://proxy.corp.example:3128"
export NO_PROXY="localhost,127.0.0.1,.corp.example"
```

## 常见问题

### Claw Code 是 Claude 专用的吗？

不。Claw Code 是一个 Claude-Code 形态的工作流/运行时，而非 Claude 专用产品。它可以面向 Anthropic、OpenAI 兼容、提供者路由和本地模型，具体取决于配置。

### Codex 是什么？

名称 "codex" 出现在 Claw Code 生态系统中，但**不**指 OpenAI Codex。在此项目中：
- **`oh-my-codex` (OmX)** 是工作流和插件层
- **`.codex/` 目录**是遗留查找路径
- **`CODEX_HOME`** 是可选环境变量
- `claw` **不**支持 OpenAI Codex 会话
