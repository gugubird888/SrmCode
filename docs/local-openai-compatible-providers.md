# 本地 OpenAI 兼容提供者和技能设置

本指南涵盖两个常见的离线/本地工作流：

1. 针对 OpenAI 兼容本地模型服务器（如 Ollama、llama.cpp 或 vLLM）运行 Claw
2. 从磁盘安装本地技能，使 Claw 无需网络即可发现它们

## Claw 不是 Claude 专用的

Claw Code 是一个 Claude-Code 形态的工作流/运行时，而非 Claude 专用产品。它直接支持 Anthropic，并且可以根据配置面向 OpenAI 兼容、提供者路由和本地模型。

## OpenAI 兼容路由基础

设置 `OPENAI_BASE_URL` 为服务器的 `/v1` 端点，并将 `OPENAI_API_KEY` 设为所需 token 或无害占位符：

```bash
export OPENAI_BASE_URL="http://127.0.0.1:11434/v1"
export OPENAI_API_KEY="local-dev-token"
claw --model "qwen3:latest" prompt "Reply exactly HELLO_WORLD_123"
```

路由注意事项：
- 使用 `openai/` 前缀进行 OpenAI 兼容网关路由
- 对于本地服务器，优先使用服务器报告的精确模型 ID
- 对于斜杠包含的本地 ID，使用 `local/` 前缀

## 原始 `/v1/chat/completions` 冒烟测试

在调试 Claw 之前，验证本地服务器是否响应预期的线格式：

```bash
curl -sS "$OPENAI_BASE_URL/chat/completions" \
  -H "Authorization: Bearer ${OPENAI_API_KEY:-local-dev-token}" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3:latest",
    "messages": [{"role": "user", "content": "Reply exactly HELLO_WORLD_123"}],
    "stream": false
  }'
```

预期结果：包含一条助手消息的 JSON 响应，内容为 `HELLO_WORLD_123`。

## Ollama

启动 Ollama 并拉取模型：

```bash
ollama pull qwen3:latest
ollama serve
```

在另一个 shell 中：

```bash
export OLLAMA_HOST="http://127.0.0.1:11434"
claw --model "qwen3:latest" prompt "Reply exactly HELLO_WORLD_123"
```

`OLLAMA_HOST` 是 Ollama 的首选环境变量。设置后无需 API 密钥。

## llama.cpp 服务器

```bash
llama-server -m ./models/qwen2.5-coder.gguf --host 127.0.0.1 --port 8080 --alias qwen2.5-coder
```

```bash
export OPENAI_BASE_URL="http://127.0.0.1:8080/v1"
export OPENAI_API_KEY="local-dev-token"
claw --model "qwen2.5-coder" prompt "Reply exactly HELLO_WORLD_123"
```

## vLLM 或其他 OpenAI 兼容服务器

```bash
vllm serve Qwen/Qwen2.5-Coder-7B-Instruct --host 127.0.0.1 --port 8000
```

```bash
export OPENAI_BASE_URL="http://127.0.0.1:8000/v1"
export OPENAI_API_KEY="local-dev-token"
claw --model "Qwen/Qwen2.5-Coder-7B-Instruct" prompt "Reply exactly HELLO_WORLD_123"
```

## 从磁盘安装本地技能

技能目录应包含一个带有前置元数据的 `SKILL.md` 文件：

```text
my-skill/
└── SKILL.md
```

```markdown
---
name: my-skill
description: 说明何时应使用此技能。
---

# 我的技能

Agent 的指令在此。
```

在交互式 REPL 中安装：

```text
/skills install /absolute/path/to/my-skill
/skills list
/skills my-skill
```

离线安装检查清单：
- 安装特定的技能目录，而非仓库根目录
- 保持前置元数据 `name` 与用户输入的目录名一致
- 安装后运行 `/skills list` 确认已发现的名称和源路径
- 如果技能调用失败且报 HTTP/提供者错误，先运行 `claw doctor` 验证提供者凭证

## 故障排除

| 症状 | 检查项 |
|---|---|
| Claw 仍然要求 Anthropic 凭证 | 使用显式的 OpenAI 兼容模型路由或移除无关的 Anthropic 环境变量 |
| 本地服务器报 `model not found` | 使用 Ollama/llama.cpp/vLLM 暴露的精确模型 ID |
| 纯提示词有效但工具失败 | 确认模型/服务器支持 OpenAI 兼容工具调用和响应格式 |
| 技能显示已安装但 `/skills <name>` 失败 | 检查 `/skills list` 中的已发现名称和来源；单独验证提供者凭证 |
