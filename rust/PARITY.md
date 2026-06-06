# 对等状态 — claw-code Rust 移植

最后更新：2026-04-03

## Mock 对等 Harness — 里程碑 1

- [x] 确定性 Anthropic 兼容 mock 服务（`rust/crates/mock-anthropic-service`）
- [x] 可重现的清洁环境 CLI harness（`rust/crates/rusty-claude-cli/tests/mock_parity_harness.rs`）
- [x] 脚本化场景：`streaming_text`、`read_file_roundtrip`、`grep_chunk_assembly`、`write_file_allowed`、`write_file_denied`

## Mock 对等 Harness — 里程碑 2（行为扩展）

- [x] 脚本化多工具轮次覆盖：`multi_tool_turn_roundtrip`
- [x] 脚本化 bash 覆盖：`bash_stdout_roundtrip`
- [x] 脚本化权限提示覆盖：`bash_permission_prompt_approved`、`bash_permission_prompt_denied`
- [x] 脚本化插件路径覆盖：`plugin_tool_roundtrip`
- [x] 行为 diff/检查清单运行器：`rust/scripts/run_mock_parity_diff.py`

## 已完成的行为对等工作

| Lane | 状态 | 功能提交 | 合并提交 | Diff 统计 |
|------|------|----------|----------|-----------|
| Bash 验证（9 个子模块） | ✅ 完成 | `36dac6c` | — (`jobdori/bash-validation-submodules`) | `1005 insertions` |
| CI 修复 | ✅ 完成 | `89104eb` | `f1969ce` | `22 insertions, 1 deletion` |
| 文件工具边界情况 | ✅ 完成 | `284163b` | `a98f2b6` | `195 insertions, 1 deletion` |
| TaskRegistry | ✅ 完成 | `5ea138e` | `21a1e1d` | `336 insertions` |
| 任务工具连接 | ✅ 完成 | `e8692e4` | `d994be6` | `79 insertions, 35 deletions` |
| Team + Cron 运行时 | ✅ 完成 | `c486ca6` | `49653fe` | `441 insertions, 37 deletions` |
| MCP 生命周期 | ✅ 完成 | `730667f` | `cc0f92e` | `491 insertions, 24 deletions` |
| LSP 客户端 | ✅ 完成 | `2d66503` | `d7f0dc6` | `461 insertions, 9 deletions` |
| 权限执行 | ✅ 完成 | `66283f4` | `336f820` | `357 insertions` |

## 工具层面：40/40（规格对等）

### 真实实现（行为对等 — 深度不同）

| 工具 | Rust 实现 | 行为说明 |
|------|-----------|----------|
| **bash** | `runtime::bash` 283 LOC | 子进程执行、超时、后台、沙箱 — 强对等。9/9 请求的验证子模块完成 |
| **read_file** | `runtime::file_ops` | offset/limit 读取 — 良好对等 |
| **write_file** | `runtime::file_ops` | 文件创建/覆盖 — 良好对等 |
| **edit_file** | `runtime::file_ops` | 旧/新字符串替换 — 良好对等 |
| **glob_search** | `runtime::file_ops` | glob 模式匹配 — 良好对等 |
| **grep_search** | `runtime::file_ops` | ripgrep 风格搜索 — 良好对等 |
| **WebFetch** | `tools` | URL 获取 + 内容提取 — 中等对等 |
| **WebSearch** | `tools` | 搜索查询执行 — 中等对等 |
| **TodoWrite** | `tools` | 待办/笔记持久化 — 中等对等 |
| **Skill** | `tools` | 技能发现/安装 — 中等对等 |
| **Agent** | `tools` | Agent 委托 — 中等对等 |
| **TaskCreate/Get/List/Stop/Update/Output** | `runtime::task_registry` + `tools` | 内存中任务生命周期 — 良好对等 |
| **TeamCreate/Delete** | `runtime::team_cron_registry` + `tools` | Team 生命周期 — 良好对等 |
| **CronCreate/Delete/List** | `runtime::team_cron_registry` + `tools` | Cron 条目管理 — 良好对等 |
| **LSP** | `runtime::lsp_client` + `tools` | 诊断、悬停、定义、引用 — 良好对等 |
| **MCP 工具** | `runtime::mcp_tool_bridge` + `tools` | MCP 工具调用桥 — 良好对等 |
| **AskUserQuestion** | `tools` | 桩 — 需要实时用户 I/O 集成 |
| **其他** | `tools` | ToolSearch、NotebookEdit、Sleep 等 — 中等对等 |

## 已完成的文件工具检查点

- [x] 路径遍历防护（符号链接跟踪、../ 逃逸）
- [x] 读写大小限制
- [x] 二进制文件检测
- [x] 权限模式执行（只读 vs 工作区写入）

## 运行时行为差距

- [x] 所有工具的权限执行
- [ ] 输出截断（大 stdout/文件内容）
- [ ] 会话压缩行为匹配
- [ ] Token 计数 / 成本追踪准确性
- [x] Mock 对等 harness 验证的流式响应支持

## 迁移就绪

- [x] `PARITY.md` 保持维护且诚实
- [ ] 零 `#[ignore]` 测试隐藏失败（仅允许 1 个：`live_stream_smoke_test`）
- [ ] 每次提交 CI 通过
- [ ] 代码库结构整洁，足够交接
