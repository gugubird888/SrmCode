# 模型兼容性指南

本文档描述 OpenAI 兼容提供者中的模型特定处理。添加新模型或提供者时，请参考本指南以确保正确兼容。

## 概述

`openai_compat.rs` 提供者将 Claw Code 的内部消息格式转换为 OpenAI 兼容的聊天完成请求。不同模型对以下方面有不同的要求：

- 工具结果消息字段（`is_error`）
- 采样参数（temperature、top_p 等）
- Token 限制字段（`max_tokens` vs `max_completion_tokens`）
- 基础 URL 路由
- 提供者特定的额外 body 参数

## 模型特定处理

### Kimi 模型（is_error 排除）

**受影响的模型：** `kimi-k2.5`、`kimi-k1.5`、`kimi-moonshot` 以及任何名称中包含 `kimi` 的模型（不区分大小写）

**行为：** `is_error` 字段从工具结果消息中**排除**。

**原因：** Kimi 模型（通过 Moonshot AI 和 DashScope）会拒绝 `is_error` 字段并返回 400 Bad Request。

**检测：**
```rust
fn model_rejects_is_error_field(model: &str) -> bool {
    let lowered = model.to_ascii_lowercase();
    let canonical = lowered.rsplit('/').next().unwrap_or(lowered.as_str());
    canonical.starts_with("kimi")
}
```

### 推理模型（调优参数剥离）

**受影响的模型：**
- OpenAI：`o1`、`o1-*`、`o3`、`o3-*`、`o4`、`o4-*`
- xAI：`grok-3-mini`
- 阿里云 DashScope：`qwen-qwq-*`、`qwq-*`、`qwen3-*-thinking`

**行为：** 从请求中**剥离**以下调优参数：`temperature`、`top_p`、`frequency_penalty`、`presence_penalty`

**原因：** 推理/思维链模型使用固定采样策略，会拒绝这些参数并返回 400 错误。

**检测：**
```rust
fn is_reasoning_model(model: &str) -> bool {
    let canonical = model.to_ascii_lowercase()
        .rsplit('/')
        .next()
        .unwrap_or(model);
    canonical.starts_with("o1")
        || canonical.starts_with("o3")
        || canonical.starts_with("o4")
        || canonical == "grok-3-mini"
        || canonical.starts_with("qwen-qwq")
        || canonical.starts_with("qwq")
        || (canonical.starts_with("qwen3") && canonical.contains("-thinking"))
}
```

### GPT-5（max_completion_tokens）

**受影响的模型：** 所有以 `gpt-5` 开头的模型

**行为：** 在请求负载中使用 `max_completion_tokens` 而非 `max_tokens`。

**原因：** GPT-5 模型需要 `max_completion_tokens` 字段。使用旧版 `max_tokens` 会导致请求验证失败。

### Qwen 和 Kimi 模型（DashScope 路由）

**受影响的模型：** 所有带有 `qwen` 或 `kimi` 前缀的模型

**行为：** 路由到 DashScope（`https://dashscope.aliyuncs.com/compatible-mode/v1`），而非其他提供者。已知路由前缀在发送前被剥离。

**认证：** 使用 `DASHSCOPE_API_KEY` 环境变量。

### 自定义网关 Slugs 和额外 Body 参数

**受影响的模型：** 通过 OpenAI 兼容提供者路由的斜杠包含的模型 ID，尤其是配置了 `OPENAI_BASE_URL` 的自定义网关（如 OpenRouter）。

**行为：**
- 默认 OpenAI API 和本地 OpenAI 兼容基础 URL 将 `openai/` 视为路由前缀，在线路上发送裸模型名称。
- 非本地自定义 OpenAI 兼容基础 URL 保留斜杠包含的 slugs（如 `openai/gpt-4.1-mini`）。
- `MessageRequest::extra_body` 在核心字段填充后透传自定义请求 JSON。
- 受保护的核心字段（`model`、`messages`、`stream`、`tools`、`tool_choice`、`max_tokens`、`max_completion_tokens`）不能通过 `extra_body` 覆盖。

## 实现细节

### 文件位置
所有模型特定逻辑位于：`rust/crates/api/src/providers/openai_compat.rs`

### 关键函数

| 函数 | 用途 |
|------|------|
| `model_rejects_is_error_field()` | 检测不支持工具结果中 `is_error` 的模型 |
| `is_reasoning_model()` | 检测需要调优参数剥离的推理模型 |
| `translate_message()` | 将内部消息转换为 OpenAI 格式 |
| `build_chat_completion_request()` | 构建完整请求负载 |
| `provider_diagnostics_for_model()` | 产生提供者/状态诊断信息 |

### 提供者前缀处理

所有模型检测函数在匹配前剥离提供者前缀：

```rust
let canonical = model.to_ascii_lowercase()
    .rsplit('/')
    .next()
    .unwrap_or(model);
```

## 添加新模型

添加新模型支持时：

1. **检查是否为推理模型** — 是否拒绝 temperature/top_p 参数？添加到 `is_reasoning_model()` 检测
2. **检查工具结果兼容性** — 是否拒绝 `is_error` 字段？添加到 `model_rejects_is_error_field()`
3. **检查 token 限制字段** — 是否需要 `max_completion_tokens`？更新 `max_tokens_key` 逻辑
4. **检查自定义网关行为** — 是否应保留斜杠包含的 ID？
5. **添加测试** — 检测函数的单元测试 + 集成测试
6. **更新本文档** — 将模型添加到受影响列表

## 测试

### 运行模型特定测试

```bash
cargo test --package api providers::openai_compat
cargo test --package api model_rejects_is_error_field
cargo test --package api reasoning_model
cargo test --package api gpt5
```

### 测试文件

- 单元测试：`rust/crates/api/src/providers/openai_compat.rs`
- 集成测试：`rust/crates/api/tests/openai_compat_integration.rs`

---

*最后更新：2026-05-15*
