# Windows 安装和发布快速入门

本页面是 PowerShell 优先的路径，用于在 Windows 上安装、验证和安全切换提供者。

## 选择安装路径

### 方案 A：在 PowerShell 中从源码构建

```powershell
git clone https://github.com/ultraworkers/claw-code
Set-Location .\claw-code\rust
cargo build --workspace
.\target\debug\claw.exe --help
.\target\debug\claw.exe doctor
```

优化本地构建：

```powershell
Set-Location .\claw-code\rust
cargo build --workspace --release
.\target\release\claw.exe --help
```

### 方案 B：使用发布制品

```powershell
$Asset = "claw-windows-x64.exe"
$InstallRoot = "$env:LOCALAPPDATA\Programs\claw"
New-Item -ItemType Directory -Force $InstallRoot | Out-Null

# 从发布页面下载 $Asset 和 $Asset.sha256，然后验证：
$Actual = (Get-FileHash ".\$Asset" -Algorithm SHA256).Hash.ToLowerInvariant()
$Expected = (Get-Content ".\$Asset.sha256" | Select-Object -First 1).Split()[0].ToLowerInvariant()
if ($Actual -ne $Expected) { throw "checksum mismatch for $Asset" }

Copy-Item ".\$Asset" "$InstallRoot\claw.exe" -Force
& "$InstallRoot\claw.exe" --help
& "$InstallRoot\claw.exe" doctor
```

添加到 PATH：

```powershell
$InstallRoot = "$env:LOCALAPPDATA\Programs\claw"
[Environment]::SetEnvironmentVariable(
  "Path",
  [Environment]::GetEnvironmentVariable("Path", "User") + ";$InstallRoot",
  "User"
)
```

### 方案 C：WSL

```powershell
wsl --install
wsl
```

在 WSL 内：

```bash
git clone https://github.com/ultraworkers/claw-code
cd claw-code
./install.sh
```

## 首次运行健康检查

```powershell
Set-Location .\claw-code\rust
.\target\debug\claw.exe --help
.\target\debug\claw.exe doctor
.\target\debug\claw.exe status --output-format json
.\target\debug\claw.exe config --output-format json
```

## 安全凭证设置

当前 PowerShell 会话：

```powershell
$env:ANTHROPIC_API_KEY = "sk-ant-REPLACE_ME"
```

持久化：

```powershell
setx ANTHROPIC_API_KEY "sk-ant-REPLACE_ME"
```

移除会话本地密钥：

```powershell
Remove-Item Env:\ANTHROPIC_API_KEY -ErrorAction SilentlyContinue
```

## 安全提供者切换示例

### Anthropic 直接

```powershell
$env:ANTHROPIC_API_KEY = "sk-ant-REPLACE_ME"
Remove-Item Env:\OPENAI_BASE_URL -ErrorAction SilentlyContinue
Remove-Item Env:\OPENAI_API_KEY -ErrorAction SilentlyContinue
.\target\debug\claw.exe --model "sonnet" prompt "reply with ready"
```

### OpenAI 兼容网关或 OpenRouter

```powershell
Remove-Item Env:\ANTHROPIC_API_KEY -ErrorAction SilentlyContinue
$env:OPENAI_BASE_URL = "https://openrouter.ai/api/v1"
$env:OPENAI_API_KEY = "sk-or-v1-REPLACE_ME"
.\target\debug\claw.exe --model "openai/gpt-4.1-mini" prompt "reply with ready"
```

### 本地 OpenAI 兼容服务器

```powershell
Remove-Item Env:\ANTHROPIC_API_KEY -ErrorAction SilentlyContinue
$env:OPENAI_BASE_URL = "http://127.0.0.1:11434/v1"
$env:OPENAI_API_KEY = "local-dev-token"
.\target\debug\claw.exe --model "llama3.2" prompt "reply with ready"
```

### DashScope / Qwen

```powershell
Remove-Item Env:\ANTHROPIC_API_KEY -ErrorAction SilentlyContinue
$env:DASHSCOPE_API_KEY = "sk-REPLACE_ME"
.\target\debug\claw.exe --model "qwen-plus" prompt "reply with ready"
```

## 故障排除检查清单

- `claw` 未找到：在 Windows 上使用 `claw.exe` 或通过完整路径运行（`.\target\debug\claw.exe`）
- `cargo` 未找到：从 <https://rustup.rs/> 安装 Rust 后重新打开 PowerShell
- `401 Invalid bearer token`：将 `sk-ant-*` 值放入 `ANTHROPIC_API_KEY`，而非 `ANTHROPIC_AUTH_TOKEN`
- 选择了错误的提供者：添加显式模型前缀，如 `openai/gpt-4.1-mini`、`qwen-plus` 或 `grok`
