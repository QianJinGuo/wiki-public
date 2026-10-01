---
title: "Introducing Grok on Amazon Bedrock — Grok 4.3 GA with Mantle Inference Engine"
created: 2026-07-24
updated: 2026-10-01
type: entity
tags: [xai, grok, grok-4.3, bedrock, aws, mantle, inference-engine, reasoning, agentic]
sources: [raw/articles/introducing-grok-on-amazon-bedrock]
confidence: 0.75
vxc: 49
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Introducing Grok on Amazon Bedrock — Grok 4.3 GA with Mantle Inference Engine

xAI's Grok 4.3 is now generally available on Amazon Bedrock, marking xAI's entry as a model provider on the platform. Grok 4.3 runs on Mantle, Amazon Bedrock's next-generation inference engine, and uses OpenAI-compatible APIs. ^[raw/articles/introducing-grok-on-amazon-bedrock.md]

## Key Capabilities

### Configurable Reasoning Effort
Grok 4.3 supports four effort levels controlled per-request via the `reasoning` parameter: `none` (disables reasoning), `low` (default), `medium`, and `high`. Higher effort improves accuracy on multi-step problems at the cost of more output tokens. Classification and extraction can run at `none` for low latency; contract analysis and case law tasks use `high` when depth matters. ^[raw/articles/introducing-grok-on-amazon-bedrock.md]

### Model Parameters
- **Context window**: 1 million tokens (for long documents and multi-turn sessions)
- **Input**: Text and image (PNG/JPEG)
- **Output**: Text
- **Defaults**: `temperature` = 0.7, `top_p` = 0.95, `max_completion_tokens` = 131072
- **Model ID**: `xai.grok-4.3`

### Mantle Inference Engine
Grok 4.3 runs on Mantle, which uses OpenAI-compatible APIs. The endpoint URL is Region-specific (e.g., `https://bedrock-mantle.us-west-2.api.aws/openai/v1`). Authentication supports both long-term Amazon Bedrock API keys (for exploration) and short-term bearer tokens from IAM credentials (for production). ^[raw/articles/introducing-grok-on-amazon-bedrock.md]

### Tool Calling & Structured Output
- Supports standard OpenAI tool-calling with JSON Schema parameter definitions
- `tool_choice: "auto"` lets the model decide when to call a tool
- Structured output via `json_schema` response format with strict mode (`additionalProperties: false`)
- One operational note: requests occasionally return 400 from automated content safety checks even on benign input — build a short retry into production calls ^[raw/articles/introducing-grok-on-amazon-bedrock.md]

### Stateful Conversations
The Responses API supports server-side conversation state via `store=True` and `previous_response_id`. The service retains each turn's reasoning and feeds it back automatically. Encrypted reasoning is available for stateless use (`store=False`) with `include=["reasoning.encrypted_content"]`. ^[raw/articles/introducing-grok-on-amazon-bedrock.md]

## Service Tiers & Regional Availability

| Tier | Description |
|------|-------------|
| Standard | Pay-per-token, no commitment |
| Priority | Preferential queue processing, higher per-token price |
| Flex | Lower-cost access, not time-sensitive |

Grok 4.3 uses in-Region inference only (no Geo or Global cross-Region inference at launch). Example Region: `us-west-2`. ^[raw/articles/introducing-grok-on-amazon-bedrock.md]

## Comparison with Grok 4.5

Grok 4.3 is the predecessor to Grok 4.5. While Grok 4.3 offers a 1M token context window and runs on Bedrock via Mantle, Grok 4.5 uses a 500K context window with 1.5T parameters on the V9 base. Grok 4.3 is positioned for production workloads that need the full 1M context at lower pricing ($1.25/$2.50 per million input/output tokens vs Grok 4.5's $2/$6). ^[raw/articles/introducing-grok-on-amazon-bedrock.md]

## Practical Guidance

1. **Match reasoning effort to task complexity**: Use `none` for classification/extraction, `low` for short factual lookups, `high` for planning steps and multi-step chains where early mistakes derail the task.
2. **Use short-term credentials in production**: Long-term API keys are convenient for exploration but should be deleted after testing. Short-term bearer tokens from IAM credentials keep access tied to your identity and expire automatically.
3. **Build retry logic for structured output**: The automated content safety check can return 400 on benign input — a short retry loop mitigates this in production.
4. **For image input, validate encoding**: Malformed or truncated base64 image payloads return a `validation_error` rather than a best-guess answer.
5. **Consider service tiers**: Standard for pay-per-token, Priority for guaranteed throughput, Flex for cost-sensitive batch workloads.

## 深度分析

**战略层：接入面正在向 OpenAI 事实标准收敛。** Grok 4.3 GA 的意义超出单模型上架——xAI 由此正式加入 Amazon Bedrock 模型供应商阵营，且通过 Mantle 推理引擎以 OpenAI 兼容 API（Chat Completions + Responses）暴露，而非 Bedrock Runtime 专有 API。这降低了既有 OpenAI SDK 应用的迁移成本，但也意味着 Bedrock 第三方模型的 API 粘性在减弱：换模型变成改 base_url 的操作。^[raw/articles/introducing-grok-on-amazon-bedrock.md:14-32]

**推理努力（reasoning effort）的经济学：把"思考多少"变成显式计费维度。** 四档 per-request 控制的本质是让一个模型覆盖从低延迟分类到深度合同分析的全谱系工作负载，替代以往"小模型路由 + 大模型兜底"的双模型架构。raw 实测很有说服力：`high` 档在 bat-and-ball 陷阱题上真的花费了 reasoning tokens 做代数推导而非给出直觉错误答案，`none` 档该字段为 0。xAI 宣称的 "2-10x intelligence per dollar" 定价叙事，正是围绕这个可调杠杆建立的——effort 档位是用户侧最直接的成本控制旋钮。^[raw/articles/introducing-grok-on-amazon-bedrock.md:91-122] 相关讨论见 [[entities/llm-thonking-reasoning-effort-security-triage]] 与 [[entities/codexclaude-code-推理-effort本质-就是往prompt里塞了一句话]]。

**生态位卡位：长上下文 + 成本敏感的生产工作负载。** 对比 Grok 4.5（500K 窗口、$2/$6 每百万 token），Grok 4.3 以 1M 窗口 + $1.25/$2.50 定价卡住"长文档 + 高吞吐推理"的位置，两者构成 xAI 在 Bedrock 上的价格-能力梯度而非简单替代关系。参照 [[entities/openai-models-codex-amazon-bedrock-ga]] 与 [[entities/gpt-56-sol-terra-luna-tiered-pricing-codex-merge-2026]]，frontier 模型在 Bedrock 上按档位分层定价已成平台常态，[[concepts/context-window-economics]] 的框架在这里直接适用。xAI 报告的 Omniscience（最低幻觉率）、Tau2 Telecom（工具调用）、Vals AI Case Law 三项第一则是在为"企业级准确性"叙事背书，注意这些都是 xAI 自选基准。^[raw/articles/introducing-grok-on-amazon-bedrock.md:20]

**风险面：三个 operational 信号。** (a) 自动内容安全检查偶发对良性输入返回 400——安全层在 API 边界而非模型层，调用方必须有重试预算；(b) 默认参数三处偏离 OpenAI 规范（`temperature` 0.7、`top_p` 0.95、`max_completion_tokens` 131072），直接迁移的应用若不显式设置会静默改变行为；(c) launch 时仅 in-Region 推理、无 Geo/Global cross-Region，多区域容灾要自行设计，且 `store=True` 的服务端状态留存涉及数据驻留合规。加上畸形图片返回 `validation_error` 而非 best-guess，这个 API 的边界行为整体偏"严格失败"而非"容错猜测"。^[raw/articles/introducing-grok-on-amazon-bedrock.md:36-44,193,223,245-251]

**Stateful 推理的两条路径：状态留在服务端还是加密外放。** Responses API 给出经典的分布式系统权衡：`store=True` + `previous_response_id` 让服务端自动回喂上轮推理（免管理，但有数据留存）；`store=False` + encrypted reasoning 把状态管理责任推回客户端，换取轮次不落盘。这是 agent 长会话架构里"stateless 计算 + 状态外置"模式在 API 层的直接体现，可与 [[concepts/agentic-workflow-patterns]]、[[concepts/context-management-agent-systems]] 互参，也呼应 [[concepts/inference-optimization]] 中关于推理开销显式化的趋势。^[raw/articles/introducing-grok-on-amazon-bedrock.md:93,227-245]

## 实践启示

1. **按任务复杂度分配 effort 并量化到 token**：分类/抽取/短查询用 `none`/`low`，规划、数学、多步链用 `high`；用 usage 块里的 `reasoning_tokens` 监控每档真实成本，在自己 workload 上 benchmark 出 high 不再值回票价的临界点——raw 结论部分也是这个建议。^[raw/articles/introducing-grok-on-amazon-bedrock.md:122,257]
2. **迁移自 OpenAI SDK 时显式覆盖三个默认值**：`temperature`、`top_p`、`max_completion_tokens` 在 Mantle 端点上的默认与 OpenAI 规范不同，在请求构造层集中断言这三个参数，避免静默行为漂移。^[raw/articles/introducing-grok-on-amazon-bedrock.md:36-44]
3. **凭据生命周期分级管理**：生产用 IAM 短期 bearer token（`aws-bedrock-token-generator`，自动过期、绑定 IAM 身份），长 API key 只用于探索且用完即删——把凭据清理纳入发布 checklist 而非事后审计。^[raw/articles/introducing-grok-on-amazon-bedrock.md:48,255]
4. **为 400 误报和图片校验设计防御**：结构化输出调用加短重试循环；图片上传前做 base64 完整性/格式校验，因为畸形 payload 会直接 `validation_error` 而不是尽力解读。^[raw/articles/introducing-grok-on-amazon-bedrock.md:193,223]
5. **长会话状态按合规要求选路径**：无数据留存约束时优先 `store=True` 享受服务端推理记忆；有约束时走 encrypted reasoning 客户端续传。当前仅 in-Region 推理，多区域部署需按 Region 粘住 Mantle base_url。^[raw/articles/introducing-grok-on-amazon-bedrock.md:93,245-251]

## Related Entities

- [[entities/grok-4-5-model-release-xai-2026-07]] — Grok 4.5 model release (successor model)
- [[entities/cursor让马斯克的grok45咸鱼翻身追平opus-48成本比glm52还低]] — Cursor + Grok 4.5 depth analysis
- [[entities/xai-dissolved-grok-colossus2-analysis]] — xAI Colossus cluster analysis
- [[entities/openai-models-codex-amazon-bedrock-ga]] — OpenAI models on Bedrock GA
- [[entities/gpt-56-sol-terra-luna-tiered-pricing-codex-merge-2026]] — GPT-5.6 on Bedrock

→ [[raw/articles/introducing-grok-on-amazon-bedrock|原文存档]]
