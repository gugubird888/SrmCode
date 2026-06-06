# Mock LLM 对等 Harness

此里程碑添加了一个确定性 Anthropic 兼容 mock 服务和一个可重现的 CLI harness，用于 Rust `claw` 二进制文件。

## 制品

- `crates/mock-anthropic-service/` — mock `/v1/messages` 服务
- `crates/rusty-claude-cli/tests/mock_parity_harness.rs` — 端到端清洁环境 harness
- `scripts/run_mock_parity_harness.sh` — 便捷包装器

## 场景

Harness 在新鲜的工作区和隔离的环境变量下运行这些脚本化场景：

1. `streaming_text`
2. `read_file_roundtrip`
3. `grep_chunk_assembly`
4. `write_file_allowed`
5. `write_file_denied`
6. `multi_tool_turn_roundtrip`
7. `bash_stdout_roundtrip`
8. `bash_permission_prompt_approved`
9. `bash_permission_prompt_denied`
10. `plugin_tool_roundtrip`
11. `auto_compact_triggered`
12. `token_cost_reporting`

## 运行

```bash
cd rust/
./scripts/run_mock_parity_harness.sh
```

行为检查清单 / 对等 diff：

```bash
cd rust/
python3 scripts/run_mock_parity_diff.py
```

场景到 PARITY 的映射位于 `mock_parity_scenarios.json`；通过 `python3 scripts/run_mock_parity_diff.py --no-run` 保持此清单与 harness 和 `PARITY.md` 对齐。

## 手动 Mock 服务器

```bash
cd rust/
cargo run -p mock-anthropic-service -- --bind 127.0.0.1:0
```

服务器打印 `MOCK_ANTHROPIC_BASE_URL=...`；将 `ANTHROPIC_BASE_URL` 指向该 URL 并使用任意非空 `ANTHROPIC_API_KEY`。
