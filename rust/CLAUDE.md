# CLAUDE.md

本文件为 Claw Code (clawcode.dev) 在此仓库中工作时提供指导。

## 检测到的技术栈
- 语言：Rust。
- 框架：从支持的启动标记中未检测到。

## 验证
- 从仓库根目录，使用 `scripts/fmt.sh` 运行 Rust 格式化（或使用 `scripts/fmt.sh --check` 进行 CI 风格检查）。从此 `rust/` 目录，等效命令是 `../scripts/fmt.sh`。根级 `cargo fmt --manifest-path rust/Cargo.toml` 不是受支持的格式化命令。
- 从此 `rust/` 目录，使用 `cargo clippy --workspace --all-targets -- -D warnings` 和 `cargo test --workspace` 运行 Rust 验证。

## 工作约定
- 倾向于小的、可审查的变更，并保持生成的引导文件与实际仓库工作流对齐。
- 将共享默认值保留在 `.claw.json`；将 `.claw/settings.local.json` 留给本机级别的覆盖。
- 不要自动覆盖现有的 `CLAUDE.md` 内容；仅在仓库工作流变更时有意识地更新它。
