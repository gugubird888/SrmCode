# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

## 检测到的技术栈
- 语言：Rust。
- 框架：从支持的启动标记中未检测到。

## 验证
- 从仓库根目录运行 Rust 验证：`scripts/fmt.sh --check`；格式化使用 `scripts/fmt.sh`。从 `rust/` 运行 Rust clippy/测试：`cargo clippy --workspace --all-targets -- -D warnings`、`cargo test --workspace`
- `src/` 和 `tests/` 均存在；当行为变更时，同时更新这两个层面。

## 仓库结构
- `rust/` 包含 Rust 工作区和活跃的 CLI/运行时实现。
- `src/` 包含应与生成的指导和测试保持一致的源文件。
- `tests/` 包含应在代码变更时同步审查的验证层面。

## 工作约定
- 倾向于小的、可审查的变更，并保持生成的引导文件与实际仓库工作流对齐。
- 将共享默认值保留在 `.claude.json`；将 `.claude/settings.local.json` 留给本机级别的覆盖。
- 不要自动覆盖现有的 `CLAUDE.md` 内容；仅在仓库工作流变更时有意识地更新它。
