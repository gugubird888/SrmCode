# 为 Claw Code 做贡献

感谢你帮助改进 Claw Code。本仓库是一个 Rust 优先的 CLI 工作区，包含支持文档和兼容性 fixtures。

## 基本规则

- 保持变更小而可审查，并与具体的 issue 或行为相关联。
- 不要提交机密、API 密钥、含凭证的会话记录或生成的构建输出。
- 倾向于使用现有的 crate 边界和工具，而不是添加新依赖。
- 当面向用户的命令、配置键或提供者行为发生变更时，更新文档。
- 保持示例可直接复制使用。使用占位密钥（如 `sk-ant-...`），避免使用需要真实凭证的命令，除非文本明确说明。

## 本地设置

```bash
git clone https://github.com/ultraworkers/claw-code
cd claw-code/rust
cargo build --workspace
cargo test --workspace
```

在 Windows PowerShell 上，从相同的 `rust` 工作区构建，并使用 `.exe` 后缀运行二进制文件：

```powershell
cd claw-code\rust
cargo build --workspace
.\target\debug\claw.exe --help
```

## ROADMAP id 分配

在追加新的数字 ROADMAP 条目之前，先拉取/变基到最新的 `main`，从你要编辑的文件中分配 id，并在推送前运行重复 id 检查：

```bash
git pull --rebase
NEXT=$(scripts/roadmap-next-id.sh)
# 将 "${NEXT}. **...**" 追加到 ROADMAP.md
scripts/roadmap-check-ids.sh
```

## 创建 Pull Request 前的检查

运行与你的变更相关的最小测试，当涉及共享的运行时、CLI 或文档层面时再运行更广泛的检查：

```bash
cd rust
cargo fmt --all --check
cargo test --workspace
cargo clippy --workspace
```

## Pull Request 指南

- 描述变更面向用户的原因。
- 列出你执行的命令以及任何已知的不足。
- 标注 CLI 输出、JSON schema、插件契约、提供者行为或 Windows/PowerShell 示例的兼容性风险。
- 不要将无关的清理工作混入功能或修复 PR。

## 许可证

通过贡献，你同意你的贡献以项目的 [MIT 许可证](./LICENSE) 授权。
