# 对等状态 — claw-code Rust 移植

最后更新：2026-04-03

## 摘要

- 规范文档：此顶层 `PARITY.md` 即 `rust/scripts/run_mock_parity_diff.py` 所消费的文件。
- 请求的 9 条 lane 检查点：**所有 9 条 lane 已合并到 `main`。**
- 当前 `main` HEAD：`ee31e00`（桩实现替换为真实的 AskUserQuestion + RemoteTrigger）。
- 仓库统计：**main 分支 292 次提交 / 所有分支 293 次提交**，**9 个 crates**，**48,599 行追踪 Rust 代码**，**2,568 行测试代码**，**3 位作者**，日期范围 **2026-03-31 → 2026-04-03**。
- Mock 对等 harness 统计：**12 个脚本化场景**，**21 个捕获的 `/v1/messages` 请求**。

## Mock 对等 harness — 里程碑 1

- [x] 确定性 Anthropic 兼容 mock 服务（`rust/crates/mock-anthropic-service`）
- [x] 可重现的清洁环境 CLI harness（`rust/crates/rusty-claude-cli/tests/mock_parity_harness.rs`）
- [x] 脚本化场景：`streaming_text`、`read_file_roundtrip`、`grep_chunk_assembly`、`write_file_allowed`、`write_file_denied`

## Mock 对等 harness — 里程碑 2（行为扩展）

- [x] 脚本化多工具轮次覆盖：`multi_tool_turn_roundtrip`
- [x] 脚本化 bash 覆盖：`bash_stdout_roundtrip`
- [x] 脚本化权限提示覆盖：`bash_permission_prompt_approved`、`bash_permission_prompt_denied`
- [x] 脚本化插件路径覆盖：`plugin_tool_roundtrip`
- [x] 行为 diff/检查清单运行器：`rust/scripts/run_mock_parity_diff.py`
- [x] 脚本化会话压缩元数据覆盖：`auto_compact_triggered`
- [x] 脚本化 token/成本 JSON 覆盖：`token_cost_reporting`

## Harness v2 行为检查清单

规范场景映射：`rust/mock_parity_scenarios.json`

- 多工具助手轮次
- Bash 流程往返
- 跨工具路径的权限执行
- 插件工具执行路径
- 文件工具 — harness 验证的流程
- Mock 对等 harness 验证的流式响应支持

## 9 条 Lane 检查点

| Lane | 状态 | 功能提交 | 合并提交 | 证据 |
|---|---|---|---|---|
| 1. Bash 验证 | 已合并 | `36dac6c` | `1cfd78a` | `rust/crates/runtime/src/bash_validation.rs` |
| 2. CI 修复 | 已合并 | `89104eb` | `f1969ce` | `rust/crates/runtime/src/sandbox.rs` |
| 3. 文件工具 | 已合并 | `284163b` | `a98f2b6` | `rust/crates/runtime/src/file_ops.rs` |
| 4. TaskRegistry | 已合并 | `5ea138e` | `21a1e1d` | `rust/crates/runtime/src/task_registry.rs` |
| 5. 任务连接 | 已合并 | `e8692e4` | `d994be6` | `rust/crates/tools/src/lib.rs` |
| 6. Team+Cron | 已合并 | `c486ca6` | `49653fe` | `rust/crates/runtime/src/team_cron_registry.rs` |
| 7. MCP 生命周期 | 已合并 | `730667f` | `cc0f92e` | `rust/crates/runtime/src/mcp_tool_bridge.rs` |
| 8. LSP 客户端 | 已合并 | `2d66503` | `d7f0dc6` | `rust/crates/runtime/src/lsp_client.rs` |
| 9. 权限执行 | 已合并 | `66283f4` | `336f820` | `rust/crates/runtime/src/permission_enforcer.rs` |

## 工具层面：main 上暴露 40 个工具规格

- `mvp_tool_specs()` 暴露 **40** 个工具规格。
- 核心执行：`bash`、`read_file`、`write_file`、`edit_file`、`glob_search`、`grep_search`。
- 产品工具包括：`WebFetch`、`WebSearch`、`TodoWrite`、`Skill`、`Agent`、`ToolSearch`、`NotebookEdit`、`Sleep`、`SendUserMessage`、`Config`、`EnterPlanMode`、`ExitPlanMode`、`REPL` 和 `PowerShell`。

## 已协调的旧 PARITY 检查清单

- [x] 路径遍历防护（符号链接跟踪、`../` 逃逸）
- [x] 读写大小限制
- [x] 二进制文件检测
- [x] 权限模式执行（只读 vs 工作区写入）
- [x] 配置合并优先级（用户 > 项目 > 本地）
- [x] 插件安装/启用/禁用/卸载流程
- [x] 零 `#[ignore]` 测试隐藏失败

## 仍待解决

- [ ] 端到端 MCP 运行时生命周期（注册表桥之外）
- [x] 输出截断（大 stdout/文件内容）
- [x] 会话压缩行为匹配
- [x] Token 计数 / 成本追踪准确性
- [x] Bash 验证 lane 已合并到 `main`
- [ ] 每次提交 CI 通过

## 迁移就绪

- [x] `PARITY.md` 保持维护且诚实
- [x] 9 条请求的 lane 已记录，含提交哈希和当前状态
- [x] 所有 9 条请求的 lane 已落地到 `main`
- [x] 零 `#[ignore]` 测试隐藏失败
- [ ] 每次提交 CI 通过
- [x] 代码库结构整洁，足够交接文档使用
