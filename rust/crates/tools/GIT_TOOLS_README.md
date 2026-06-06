# Git 感知的上下文工具

为 claw-code 添加五个原生 git 工具，提供结构化的只读仓库状态访问。这些工具用专门构建的工具定义替代通过 bash 的临时 `git` 命令，模型可以直接发现和调用。

## 工具

### GitStatus

显示工作树状态（分支、已暂存、未暂存、未跟踪）。等效于 `git status --short --branch`。

| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| `short` | boolean | 否 | `true` | 使用 `--short --branch` 格式输出简洁结果 |

### GitDiff

显示提交之间、索引和工作树的变更。支持暂存变更、特定路径、提交范围以及比较两个提交。

| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| `staged` | boolean | 否 | `false` | 显示暂存变更 |
| `commit` | string | 否 | — | 要 diff 的提交哈希/标签/分支 |
| `commit2` | string | 否 | — | 用于范围 diff 的第二个提交 |
| `path` | string | 否 | — | 限制 diff 的文件路径 |

### GitLog

显示提交历史。支持限制数量、按作者/日期/路径过滤以及 oneline 格式。

| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| `count` | integer | 否 | `20` | 返回的最大提交数 |
| `oneline` | boolean | 否 | `false` | 使用 `--oneline` 格式 |
| `author` | string | 否 | — | 按作者模式过滤 |
| `since` | string | 否 | — | 过滤自某日期以来的提交 |
| `until` | string | 否 | — | 过滤至某日期为止的提交 |
| `path` | string | 否 | — | 过滤的文件或目录路径 |

### GitShow

显示提交、标签或树对象及其 diff。支持在特定提交处显示特定文件和仅统计模式。

| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| `commit` | string | **是** | — | 要显示的提交哈希/标签/分支引用 |
| `path` | string | 否 | — | 在给定提交处仅显示此文件 |
| `stat` | boolean | 否 | `false` | 显示 diffstat 摘要而非完整 diff |

### GitBlame

显示每行文件最后被哪个修订版本和作者修改。支持行范围过滤。

| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| `path` | string | **是** | — | 要 blame 的文件路径 |
| `start_line` | integer | 否 | — | 行范围起始（基于 1） |
| `end_line` | integer | 否 | — | 行范围结束（基于 1） |

## 架构

所有五个工具遵循相同的模式：

1. **ToolSpec** — 定义工具名称、描述、JSON 输入 schema 和 `PermissionMode::ReadOnly`
2. **Input struct** — 派生 `Deserialize`，可选字段带 `#[serde(default)]`
3. **Run 函数** — 构建 git 参数，调用 `git_stdout()`，通过 `to_pretty_json()` 包装结果
4. **Dispatch** — 像所有其他工具一样在 `execute_tool_with_enforcer()` 中匹配

## 为什么需要原生 git 工具？

在此变更之前，模型必须使用 `bash` 工具进行 git 操作，存在若干缺点：

- **无结构化输出** — Bash 返回原始文本，模型必须解析
- **过度授权** — Bash 即使是只读 git 命令也需要 `DangerFullAccess`
- **无法发现** — 模型无法通过 `ToolSearch` 搜索支持 git 的工具
- **不一致** — 每次调用可能使用不同的标志或格式

使用原生 git 工具：

- 所有五个都是 `ReadOnly` — 在受限权限模式下安全
- 结构化 JSON 输出 — 一致、可解析的结果
- 可通过 `ToolSearch` 发现，关键词如 "git"、"diff"、"blame"
- 模型友好的描述说明何时使用每个工具而非 bash

## 测试

```bash
cd rust
cargo build --release
cargo test -p tools
```
