# 导航和文件上下文指南

本指南回答 Claw Code 常见的"如何浏览输出？"和"如何提交文件？"问题。Claw 是一个 agent CLI，而非完整的文件管理器：终端导航来自你的 shell 或终端，而文件上下文通过提示词显式传递。

## 提示词和终端导航

使用终端的常规控件进行命令历史和长输出：

- `上` / `下` 键通常移动 shell 或 REPL 提示词历史。
- `Ctrl-r` 在大多数 shell 中搜索 shell 历史。
- 长命令输出通过终端滚动回看查看。在 tmux 中，按 `Ctrl-b [` 进入复制模式，然后使用方向键、PageUp/PageDown、搜索或鼠标。
- 如果输出太大无法舒适滚动，将其重定向到文件并将该文件作为上下文提供给 Claw：
  ```bash
  cargo test --workspace 2>&1 | tee logs/test-output.txt
  claw prompt "Use @logs/test-output.txt as context and summarize the failing tests."
  ```

## 使用 `@path` 提交仓库文件

使用 `@` 路径提及当前工作区中的文件：

```text
Read @src/app.ts and explain the bug.
Compare @old.md and @new.md.
Use @logs/error.txt as context and suggest a fix.
```

提示：

- 优先使用最小的有用文件集。
- 尽可能使用精确路径（`@rust/crates/runtime/src/lib.rs`）而非模糊描述。
- 对生成的日志，将其保存在临时或被忽略的目录（如 `logs/`）下并引用该文件。

## 机密和凭证安全

不要在提示词、issue 评论、截图或提交的文档中粘贴真实的 API 密钥、OAuth token、私有日志或客户数据。提交文件前：

- 将真实密钥替换为占位符，如 `sk-ant-REPLACE_ME`、`sk-or-v1-REPLACE_ME` 或 `local-dev-token`。
- 编辑掉 bearer token、cookie、会话 ID 和私有基础 URL。
- 优先使用最小复现而非完整生产日志。
- 将 `.env`、密钥文件和私有日志排除在 git 之外。
