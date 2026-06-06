# Claw Code

<p align="center">
  <a href="https://github.com/code-yeongyu/lazycodex">
    <img src="https://img.shields.io/badge/LazyCodex-codex%20for%20no--brainers-111111?style=for-the-badge&logo=github&logoColor=white" alt="LazyCodex banner" />
  </a>
  <a href="https://github.com/Yeachan-Heo/gajae-code">
    <img src="https://img.shields.io/badge/Gajae--Code-red--claw%20agent%20harness-B22222?style=for-the-badge&logo=github&logoColor=white" alt="Gajae-Code banner" />
  </a>
</p>

<h3 align="center">从真正的螃蟹驱动的 harness 开始</h3>

<p align="center">
  <a href="https://github.com/code-yeongyu/lazycodex"><b>github.com/code-yeongyu/lazycodex</b></a>
  <br/>
  <a href="https://github.com/Yeachan-Heo/gajae-code"><b>github.com/Yeachan-Heo/gajae-code</b></a>
</p>

<p align="center">
  <a href="https://discord.gg/GtjhvgjnV">
    <img src="https://img.shields.io/badge/Discord-join%20the%20harness%20lab-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Join the harness lab on Discord" />
  </a>
  <a href="https://discord.gg/4Rt79F7dF">
    <img src="https://img.shields.io/badge/Discord-join%20the%20crab%20tank-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Join the crab tank on Discord" />
  </a>
</p>

> [!IMPORTANT]
> **Claw Code 不是这里严肃的生产项目。**
> 本仓库更接近于一个博物馆展品而非产品宣传——一个由螃蟹们维护的甲壳类制品，由 agent 清扫和标记，并根据上述 harness 自动维护。
>
> 如已在项目理念中所述，这不意味着像普通产品仓库那样手工操作。它是一个 **agent 管理的展品**：harness 规划、执行、验证、标记并保存制品，而螃蟹们保持鱼缸运转。
>
> 如果你想真正运行工作，从 **[LazyCodex](https://github.com/code-yeongyu/lazycodex)** 或 **[Gajae-Code](https://github.com/Yeachan-Heo/gajae-code)** 开始。如果你想检视 Claw Code 时刻的奇特小化石，请继续往下看。
>
> 关于此理念的更长的公开解释，请参阅[此处](https://x.com/realsigridjin/status/2039472968624185713)。

<p align="center">
  <a href="https://github.com/ultraworkers/claw-code">ultraworkers/claw-code</a>
  ·
  <a href="./USAGE.md">使用指南</a>
  ·
  <a href="./rust/README.md">Rust 工作区</a>
  ·
  <a href="./PARITY.md">对等状态</a>
  ·
  <a href="./ROADMAP.md">路线图</a>
  ·
  <a href="./CONTRIBUTING.md">贡献</a>
  ·
  <a href="./SECURITY.md">安全</a>
  ·
  <a href="https://discord.gg/5TUQKqFWd">UltraWorkers Discord</a>
</p>

<p align="center">
  <a href="https://star-history.com/#ultraworkers/claw-code&Date">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=ultraworkers/claw-code&type=Date&theme=dark" />
      <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=ultraworkers/claw-code&type=Date" />
      <img alt="Star history for ultraworkers/claw-code" src="https://api.star-history.com/svg?repos=ultraworkers/claw-code&type=Date" width="600" />
    </picture>
  </a>
</p>

<p align="center">
  <img src="assets/claw-hero.jpeg" alt="Claw Code" width="300" />
</p>

Claw Code 是 `claw` CLI agent harness 的公开 Rust 实现。规范实现位于 [`rust/`](./rust)，本仓库的当前真相来源是 **ultraworkers/claw-code**。

> [!IMPORTANT]
> 从 [`USAGE.md`](./USAGE.md) 开始，了解构建、认证、CLI、会话和对等 harness 工作流。文件提交/导航问题见[导航和文件上下文](./docs/navigation-file-context.md)。本地 OpenAI 兼容模型和离线技能安装见[本地 OpenAI 兼容提供者和技能设置](./docs/local-openai-compatible-providers.md)。Windows 用户可跳转到 [Windows 安装和发布快速入门](./docs/windows-install-release.md)。构建后首先运行 `claw doctor` 进行健康检查，使用 [`rust/README.md`](./rust/README.md) 了解 crate 级别的详细信息，阅读 [`PARITY.md`](./PARITY.md) 了解当前 Rust 移植检查点，以及 [`docs/container.md`](./docs/container.md) 了解容器优先工作流。
>
> **ACP / Zed 状态：** `claw-code` 尚未提供 ACP/Zed 守护进程或 JSON-RPC 入口点。运行 `claw acp`（或 `claw --acp`）获取当前状态，而非从源码布局猜测；`claw acp serve` 目前仅作为可发现性别名，返回状态并退出码 0，真正的 ACP 支持仍在 `ROADMAP.md` 中单独追踪。

## 当前仓库结构

- **`rust/`** — 规范 Rust 工作区和 `claw` CLI 二进制文件
- **`USAGE.md`** — 面向任务的当前产品使用指南
- **`PARITY.md`** — Rust 移植对等状态和迁移说明
- **`ROADMAP.md`** — 活跃路线图和清理待办事项
- **`src/` + `tests/`** — 配套 Python/参考工作区和审计辅助工具；非主要运行时

## 快速开始

> [!WARNING]
> **`cargo install claw-code` 会安装错误的东西。** crates.io 上的 `claw-code` crate 是一个已弃用的桩，安装的是 `claw-code-deprecated.exe` — 而非 `claw`。运行它只会打印 `"claw-code has been renamed to agent-code"`。**不要使用 `cargo install claw-code`。** 要么从源码构建（本仓库），要么安装上游二进制文件：
> ```bash
> cargo install agent-code   # 上游二进制文件 — 安装 'agent.exe' (Windows) / 'agent' (Unix)，而非 'agent-code'
> ```
> 本仓库 (`ultraworkers/claw-code`) 为**仅从源码构建** — 请遵循以下步骤。

```bash
# 1. 克隆并构建
git clone https://github.com/ultraworkers/claw-code
cd claw-code/rust
cargo build --workspace

# 2. 设置 API 密钥（Anthropic API 密钥 — 非 Claude 订阅）
export ANTHROPIC_API_KEY="sk-ant-..."

# 3. 验证一切正常
./target/debug/claw doctor

# 4. 运行提示词
./target/debug/claw prompt "say hello"
```

> [!NOTE]
> **Windows (PowerShell)：** 二进制文件是 `claw.exe`，而非 `claw`。使用 `.\target\debug\claw.exe` 或运行 `cargo run -- prompt "say hello"` 跳过路径查找。

### Windows 设置

**PowerShell 是受支持的 Windows 路径。** 使用任何你习惯的 shell。Windows 上常见的入门问题：

1. **先安装 Rust** — 从 <https://rustup.rs/> 下载并运行安装程序。完成后关闭并重新打开终端。
2. **验证 Rust 在 PATH 中：**
   ```powershell
   cargo --version
   ```
   如果失败，重新打开终端或运行 Rust 安装程序输出中的 PATH 设置，然后重试。
3. **克隆并构建**（适用于 PowerShell、Git Bash 或 WSL）：
   ```powershell
   git clone https://github.com/ultraworkers/claw-code
   cd claw-code/rust
   cargo build --workspace
   ```
4. **运行**（PowerShell — 注意 `.exe` 和反斜杠）：
   ```powershell
   $env:ANTHROPIC_API_KEY = "sk-ant-..."
   .\target\debug\claw.exe prompt "say hello"
   ```

对于发布 ZIP、PATH 设置、提供者切换和通知冒烟检查，参见 [`docs/windows-install-release.md`](./docs/windows-install-release.md)。

## 构建后：定位二进制文件并验证

运行 `cargo build --workspace` 后，`claw` 二进制文件已构建但**未**自动安装到系统。以下是在哪里找到它以及如何验证构建成功。

### 二进制文件位置

在 `claw-code/rust/` 中运行 `cargo build --workspace` 后：

**Debug 构建（默认，编译更快）：**
- **macOS/Linux：** `rust/target/debug/claw`
- **Windows：** `rust/target/debug/claw.exe`

**Release 构建（优化，编译较慢）：**
- **macOS/Linux：** `rust/target/release/claw`
- **Windows：** `rust/target/release/claw.exe`

### 验证构建成功

使用路径直接测试二进制文件：

```bash
# macOS/Linux (debug 构建)
./rust/target/debug/claw --help
./rust/target/debug/claw doctor

# Windows PowerShell (debug 构建)
.\rust\target\debug\claw.exe --help
.\rust\target\debug\claw.exe doctor
```

PowerShell 冒烟命令（无需真实凭证）：

```powershell
$env:CLAW_CONFIG_HOME = Join-Path $env:TEMP "claw config home"
New-Item -ItemType Directory -Force -Path $env:CLAW_CONFIG_HOME | Out-Null
Remove-Item Env:\ANTHROPIC_API_KEY, Env:\ANTHROPIC_AUTH_TOKEN, Env:\OPENAI_API_KEY -ErrorAction SilentlyContinue
.\rust\target\debug\claw.exe help
.\rust\target\debug\claw.exe status
.\rust\target\debug\claw.exe config env
.\rust\target\debug\claw.exe doctor
```

如果这些命令成功，则构建正常工作。`claw doctor` 是你的第一个健康检查 — 它验证你的 API 密钥、模型访问和工具配置。

### 可选：添加到 PATH

**选项 1：符号链接 (macOS/Linux)**
```bash
ln -s $(pwd)/rust/target/debug/claw /usr/local/bin/claw
```

**选项 2：使用 `cargo install`（所有平台）**
```bash
cd claw-code/rust
cargo install --path . --force
claw --help
```

**选项 3：更新 shell 配置文件 (bash/zsh)**
```bash
export PATH="$(pwd)/rust/target/debug:$PATH"
```

### 故障排除

- **"command not found: claw"** — 二进制文件在 `rust/target/debug/claw`，但不在 PATH 中。使用完整路径或按上述方式符号链接/安装。
- **"permission denied"** — 在 macOS/Linux 上可能需要 `chmod +x rust/target/debug/claw`。
- **Debug vs. release** — 如果构建慢，你在 debug 模式（默认值）。添加 `--release` 可获得更快的运行时，但构建本身需要 5–10 分钟。

> [!NOTE]
> **认证：** claw 需要 **API 密钥**（`ANTHROPIC_API_KEY`、`OPENAI_API_KEY` 等）— Claude 订阅登录不是受支持的认证路径。

## 文档地图

- [`USAGE.md`](./USAGE.md) — 快速命令、认证、会话、配置、对等 harness
- [`docs/navigation-file-context.md`](./docs/navigation-file-context.md) — 终端导航、滚动回看、`@path` 文件上下文、附件和密钥安全指南
- [`docs/local-openai-compatible-providers.md`](./docs/local-openai-compatible-providers.md) — Ollama/llama.cpp/vLLM 设置、Claw 多提供者定位和本地技能安装检查
- [`docs/windows-install-release.md`](./docs/windows-install-release.md) — PowerShell 优先安装、发布制品、提供者切换和 Windows/WSL 通知冒烟路径
- [`rust/README.md`](./rust/README.md) — crate 地图、CLI 界面、功能、工作区布局
- [`PARITY.md`](./PARITY.md) — Rust 移植的对等状态
- [`rust/MOCK_PARITY_HARNESS.md`](./rust/MOCK_PARITY_HARNESS.md) — 确定性 mock 服务 harness 详情
- [`ROADMAP.md`](./ROADMAP.md) — 活跃路线图和开放的清理工作
- [`docs/MODEL_COMPATIBILITY.md`](./docs/MODEL_COMPATIBILITY.md) — 模型兼容性（Kimi、推理模型、GPT-5、DashScope 等）
- [`CONTRIBUTING.md`](./CONTRIBUTING.md)、[`SECURITY.md`](./SECURITY.md)、[`SUPPORT.md`](./SUPPORT.md) — 贡献、漏洞报告、支持和社区政策
- [`LICENSE`](./LICENSE) — 本仓库的 MIT 许可证

## 生态系统

Claw Code 与更广泛的 UltraWorkers 工具链一起公开构建：

- [clawhip](https://github.com/Yeachan-Heo/clawhip)
- [oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)
- [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)
- [oh-my-codex](https://github.com/Yeachan-Heo/oh-my-codex)
- [gajae-code](https://github.com/Yeachan-Heo/gajae-code)
- [UltraWorkers Discord](https://discord.gg/5TUQKqFWd)

## 所有权/附属免责声明

- 本仓库**不**声称拥有原始 Claude Code 源代码的所有权。
- 本仓库**不隶属于、不受认可于、也不由 Anthropic 维护**。
